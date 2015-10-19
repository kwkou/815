<properties
	pageTitle="扩展流分析作业以增加吞吐量 | Windows Azure"
	description="了解如何通过配置输入分区、细化查询定义和设置作业流式处理单位来扩展流分析作业。"
	keywords="analytics jobs,data stream,data streaming"
	services="stream-analytics"
	documentationCenter=""
	authors="jeffstokes72"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="stream-analytics"
	ms.date="08/19/2015"
	wacn.date="09/15/2015"/>

# 扩展 Azure 流分析作业以增加吞吐量 #

了解如何计算流分析作业的*流式处理单位*，以及如何通过配置输入分区、细化查询定义和设置作业流式处理单位来扩展流分析作业。

## 流分析作业的组成部分有哪些？ ##
Azure 流分析作业定义包括输入、查询和输出。输入是作业读取数据流的地方，查询是用于转换数据输入流的一种方式，而输出则是作业将作业结果发送到的地方。

若要对数据进行流式处理，作业需要至少一个输入源。可以将数据流输入源存储在 Azure Service Bus 事件中心或 Azure Blob 存储中。有关详细信息，请参阅 [Azure 流分析简介](/documentation/articles/stream-analytics-introduction)、[Azure 流分析入门](/documentation/articles/stream-analytics-get-started)<!--和 [Azure 流分析开发人员指南](/documentation/articles/stream-analytics-developer-guide)-->。

## 配置流式处理单位 ##
流式处理单位 (SU) 代表执行 Azure 流分析作业的资源和能力。在已经对 CPU、内存以及读取和写入速率进行测量的情况下，可以使用 SU 来描述相对的事件处理能力。每个流式处理单位大致相当于 1MB/秒的吞吐量。

选择特定作业所需的 SU 数目时，得根据输入的分区配置以及为作业定义的查询来决定。在使用 Azure 门户选择作业的流式处理单位数时，你最多可以选择定额数。默认情况下，每个 Azure 订阅的定额为最多 50 个流式处理单位，这适用于特定区域的所有分析作业。若要提高订阅的流式处理单位数，请联系 [Microsoft 支持](http://support.microsoft.com)。

作业能够使用的流式处理单位数取决于输入的分区配置以及为作业定义的查询。另请注意，必须使用有效的流单位值。有效值以 1、3、6 开始，往上再按 6 递增，如下所示。

![Azure 流分析流单位规模][img.stream.analytics.streaming.units.scale]

本文将说明如何计算和优化查询，以便增加分析作业的吞吐量。

## 计算作业的最大流式处理单位数 ##
流分析作业所能使用的流式处理单位总数取决于为作业定义的查询中的步骤数，以及每一步的分区数。

### 查询中的步骤 ###
查询可以有一个或多个步骤。每一步都是一个使用 WITH 关键字定义的子查询。位于 WITH 关键字外的唯一查询也计为一步，例如以下查询中的 SELECT 语句：

	WITH Step1 AS (
		SELECT COUNT(*) AS Count, TollBoothId
		FROM Input1 Partition By PartitionId
		GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
	)

	SELECT SUM(Count) AS Count, TollBoothId
	FROM Step1
	GROUP BY TumblingWindow(minute,3), TollBoothId

前面的查询有两步。

> [AZURE.NOTE]此示例查询将在本文后面部分介绍。

### 对步骤进行分区 ###

对步骤进行分区需要下列条件：

- 输入源必须进行分区。有关详细信息，请参阅 <!--[-->Azure 流分析开发人员指南<!--](/documentation/articles/stream-analytics-developer-guide)-->和<!--[-->事件中心编程指南<!--](/documentation/articles/azure-event-hubs-developer-guide)-->。
- 查询的 SELECT 语句必须从进行了分区的输入源读取。
- 步骤中的查询必须有 **Partition By** 关键字

对查询进行分区时，输入事件将在独立的分区组中进行处理和聚合，而输出事件则是针对每个组生成的。如果需要对聚合进行组合，则必须创建另一个不分区的步骤来进行聚合。

### 计算作业的最大流式处理单位数 ###

所有未分区的步骤加在一起可以进行扩展，最多可以扩展到每个流分析作业 6 个流式处理单位。若要添加更多流式处理单位，必须对步骤进行分区。每个分区可以有 6 个流式处理单位。

<table border="1">
<tr><th>作业的查询</th><th>作业的最大流式处理单位数</th></td>

<tr><td>
<ul>
<li>该查询包含一个步骤。</li>
<li>该步骤未分区。</li>
</ul>
</td>
<td>6</td></tr>

<tr><td>
<ul>
<li>输入数据流被分为 3 个分区。</li>
<li>该查询包含一个步骤。</li>
<li>该步骤已分区。</li>
</ul>
</td>
<td>18</td></tr>

<tr><td>
<ul>
<li>该查询包含两个步骤。</li>
<li>这两个步骤都未分区。</li>
</ul>
</td>
<td>6</td></tr>



<tr><td>
<ul>
<li>数据流输入被分为 3 个分区。</li>
<li>该查询包含两个步骤。输入步骤进行了分区，第二个步骤未分区。</li>
<li>SELECT 语句从已分区输入中读取数据。</li>
</ul>
</td>
<td>24（18 个用于已分区步骤 + 6 个用于未分区步骤）</td></tr>
</table>

### 缩放示例 ###
以下查询计算三分钟时段内通过收费站（总共三个收费亭）的车辆数。此查询可以扩展到 6 个流式处理单位。

	SELECT COUNT(*) AS Count, TollBoothId
	FROM Input1
	GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId

若要对查询使用更多的流式处理单位，必须对数据流输入和查询进行分区。如果将数据流分区设置为 3，则可将以下修改的查询扩展到 18 个流式处理单位：

	SELECT COUNT(*) AS Count, TollBoothId
	FROM Input1 Partition By PartitionId
	GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId

对查询进行分区后，将在独立的分区组中处理和聚合输入事件。此外，还会为每个组生成输出事件。在数据流输入中，当**“分组方式”**字段不是“分区键”时，进行分区可能会导致某些意外的结果。例如，在前面的示例查询中，TollBoothId 字段不是 Input1 的“分区键”。可以将 TollBooth #1 提供的数据分布到多个分区中。

将通过流分析对每个 Input1 分区分开进行处理，并会在相同的翻转窗口中为同一收费亭创建多个有关已通过车辆计数的记录。如果不能更改输入分区键，则可通过添加额外的不分区步骤来解决此问题，例如：

	WITH Step1 AS (
		SELECT COUNT(*) AS Count, TollBoothId
		FROM Input1 Partition By PartitionId
		GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
	)

	SELECT SUM(Count) AS Count, TollBoothId
	FROM Step1
	GROUP BY TumblingWindow(minute, 3), TollBoothId

此查询可以扩展到 24 个流式处理单位。

>[AZURE.NOTE]如果要联接两个流，请确保流是按进行联接的列的分区键分区的，并且两个流中的分区数目是相同的。


## 配置流分析作业分区 ##

**调整作业流式处理单位的步骤**

1. 登录到[管理门户](https://manage.windowsazure.cn)。
2. 单击左窗格中的**“流分析”**。
3. 单击想要缩放的流分析作业。
4. 单击页面顶部的**“缩放”**。

![Azure 流分析流单位规模][img.stream.analytics.streaming.units.scale]


## 监视作业性能 ##

使用管理门户时，你可以跟踪作业的吞吐量（以事件数/秒为单位）：

![Azure 流分析监视作业][img.stream.analytics.monitor.job]

计算预计的工作负荷吞吐量（以事件数/秒为单位）。如果吞吐量少于预期，则可调整输入分区和查询，并可为作业添加额外的流式处理单位。

## 基于规模的 ASA 吞吐量 - Raspberry Pi 方案 ##


若要了解在典型方案中 ASA 如果根据多个流式处理单位的处理吞吐量进行伸缩，我们在这里进行了一个试验，将传感器数据（客户端）发送到事件中心，由 ASA 对其进行处理，然后将警报或统计信息作为输出发送到另一数据中心。

客户端将综合性的传感器数据发送到事件中心，事件中心再以 JSON 格式将数据发送给 ASA，数据输出也采用 JSON 格式。下面是示例数据看起来的样子：

    {"devicetime":"2014-12-11T02:24:56.8850110Z","hmdt":42.7,"temp":72.6,"prss":98187.75,"lght":0.38,"dspl":"R-PI Olivier's Office"}

查询：“关灯时发送警报”

    SELECT AVG(lght),
	 “LightOff” as AlertText
	FROM input TIMESTAMP
	BY devicetime
	 WHERE
		lght< 0.05 GROUP BY TumblingWindow(second, 1)

衡量吞吐量：在这种情况下，吞吐量是指由 ASA 在固定的时间（10 分钟）内处理的输入数据的量。若要使输入数据达到最佳的处理吞吐量，必须对数据流输入和查询进行分区。此外，还需在查询中添加 COUNT()，以便衡量所处理的输入事件数。为了确保 ASA 不会单纯地等待输入事件的到来，输入事件中心的每个分区已预先加载了足够的输入数据（大约 300MB）。

下面是结果，可以发现流式处理单位数和事件中心的相应分区计数都增加了。

<table border="1">
<tr><th>输入分区数</th><th>输出分区数</th><th>流式处理单位数</th><th>持续的吞吐量
</th></td>

<tr><td>12</td>
<td>12</td>
<td>6</td>
<td>4.06 MB/秒</td>
</tr>

<tr><td>12</td>
<td>12</td>
<td>12</td>
<td>8.06 MB/秒</td>
</tr>

<tr><td>48</td>
<td>48</td>
<td>48</td>
<td>38.32 MB/秒</td>
</tr>

<tr><td>192</td>
<td>192</td>
<td>192</td>
<td>172.67 MB/秒</td>
</tr>

<tr><td>480</td>
<td>480</td>
<td>480</td>
<td>454.27 MB/秒</td>
</tr>

<tr><td>720</td>
<td>720</td>
<td>720</td>
<td>609.69 MB/秒</td>
</tr>
</table>

![img.stream.analytics.perfgraph][img.stream.analytics.perfgraph]

## 获取帮助 ##
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)。


## 后续步骤 ##

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)


<!--Image references-->

[img.stream.analytics.monitor.job]: ./media/stream-analytics-scale-jobs/StreamAnalytics.job.monitor.png
[img.stream.analytics.configure.scale]: ./media/stream-analytics-scale-jobs/StreamAnalytics.configure.scale.png
[img.stream.analytics.perfgraph]: ./media/stream-analytics-scale-jobs/perf.png
[img.stream.analytics.streaming.units.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsStreamingUnitsExample.jpg

<!--Link references-->

[microsoft.support]: http://support.microsoft.com
[azure.management.portal]: http://manage.windowsazure.cn
[azure.event.hubs.developer.guide]: http://msdn.microsoft.com/zh-cn/library/azure/dn789972.aspx

[stream.analytics.developer.guide]: /documentation/articles/stream-analytics-developer-guide
[stream.analytics.introduction]: /documentation/articles/stream-analytics-introduction
[stream.analytics.get.started]: /documentation/articles/stream-analytics-get-started
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: http://go.microsoft.com/fwlink/?LinkId=517301
 

<!---HONumber=69-->