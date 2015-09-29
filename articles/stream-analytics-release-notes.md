<properties 
	pageTitle="流分析发行说明 | Windows Azure" 
	description="流分析通用版说明" 
	services="stream-analytics" 
	documentationCenter="" 
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.date="08/20/2015" 
	wacn.date="09/15/2015"/>

#Microsoft 流分析发行说明

## 流分析 08/20/2015 版说明 ##

此版本包含以下更新。

标题|说明
---|---
增加了 LAST 函数 |现在，你可以在流分析作业中使用 [LAST](http://msdn.microsoft.com/zh-cn/library/mt421186.aspx) 函数来检索给定时间框架内某个事件流中的最新事件。
新的数组函数|现在，你可以使用数组函数 [GetArrayElement](http://msdn.microsoft.com/zh-cn/library/mt270218.aspx)、[GetArrayElements](http://msdn.microsoft.com/zh-cn/library/mt298451.aspx) 和 [GetArrayLength](http://msdn.microsoft.com/zh-cn/library/mt270226.aspx)。
新的记录函数|现在，你可以使用记录函数 [GetRecordProperties](http://msdn.microsoft.com/zh-cn/library/mt270221.aspx) 和 [GetRecordPropertyValue](http://msdn.microsoft.com/zh-cn/library/mt270220.aspx)。

## 流分析 07/30/2015 版说明 ##

此版本包含以下更新。

标题|说明
---|---
Power BI Org Id 与 Azure Id 脱偶联|此功能允许任何 Azure 帐户类型（Live Id 或 Org Id）下任何 ASA 作业的 [Power BI 输出](/documentation/articles/stream-analytics-power-bi-dashboard)。此外，你还可以将一个 Org Id 用于 Azure 帐户，将另一个用于 Power BI 输出的授权。
支持 Service Bus 队列输出|[Service Bus 队列](/documentation/articles/stream-analytics-connect-data-event-outputs#service-bus-queues)输出现可用于流分析作业。
支持 Service Bus 主题输出|[Service Bus 主题](/documentation/articles/stream-analytics-connect-data-event-outputs#service-bus-topics)输出现可用于流分析作业。

## 流分析 07/09/2015 版说明 ##

此版本包含以下更新。


标题|说明
---|---
自定义 Blob 输出分区|Blob 存储输出现在允许你指定写入输出 blob 的频率，以及输出数据路径文件夹的结构和格式。 

## 流分析 05/03/2015 版说明 ##

此版本包含以下更新。


标题|说明
---|---
增大“无序容许度窗口”的最大值|“无序容许度窗口”的最大大小现在为 59:59 (MM:SS)
JSON 输出格式：行分隔或数组|现在，当你输出到 Blob 存储或事件中心时，可以选择将输出作为 JSON 对象的数组，也可以使用新行来分隔 JSON 对象。 

## 流分析 04/16/2015 版说明 ##


标题|说明
---|---
Azure 存储帐户配置中的延迟|在某个区域首次创建流分析作业时，系统会提示你创建一个新的存储帐户，或者指定一个现有的帐户来监视该区域的流分析作业。由于配置监视方面的延迟，在 30 分钟内在同一区域创建另一个流分析作业时，系统会提示你指定第二个存储帐户，而不是在“监视存储帐户”下拉列表中显示最近配置的帐户。为了避免创建不需要的存储帐户，在某个区域首次创建作业后，可等待 30 分钟，然后再预配该区域的其他作业。
作业升级|目前，流分析不支持实时编辑正在运行的作业的定义或配置。若要更改正在运行的作业的输入、输出、查询、规模或配置，你必须先停止该作业。
从输入源推断出数据类型|如果未使用 CREATE TABLE 语句，则可从输入格式推断输入类型，例如，CSV 中的所有字段都是字符串。若要避免出现类型不匹配错误，需使用 CAST 函数将字段显式转换为正确的类型。
缺失字段将输出为 null 值|引用输入源中不存在的字段将导致输出事件中出现 null 值。
WITH 语句必须位于 SELECT 语句前面|在你的查询中，SELECT 必须紧跟 WITH 语句中定义的子查询。
内存不足问题|流式处理分析作业时，如果作业对无序事件的容忍度较大，而且/或者查询很复杂，需要保留大量的状态，则可能会导致作业在运行时出现内存不足的问题，使得作业重新启动。可以在作业的操作日志中查看启动和停止操作。若要避免出现这种行为，可以将查询横向扩展到多个分区。在未来的版本中，为了解决这种受限问题，将会降低受影响作业的性能，而不是重新启动它们。
如果 blob 输入很大而没有负载时间戳，则可能导致内存不足问题|如果没有通过 TIMESTAMP BY 指定时间戳字段，则在使用 Blob 存储中的大型文件时，可能导致流分析作业崩溃。若要避免出现此问题，请将每个 blob 的大小控制在 10MB 以下。
SQL 数据库事件数量限制|使用 SQL 数据库作为输出目标时，如果输出数据量过大，则可能导致流分析作业超时。若要解决此问题，一方面可以通过使用聚合或筛选器运算符来减少输出量，另一方面可以改为选择 Azure Blob 存储或事件中心作为输出目标。
PowerBI 数据集只能包含一个表|PowerBI 不支持在给定数据集中设置多个表。

## 获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)

## 后续步骤

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)
 

<!---HONumber=69-->