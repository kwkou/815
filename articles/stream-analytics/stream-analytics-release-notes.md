<properties
    pageTitle="流分析发行说明 | Azure"
    description="流分析发行说明"
    services="stream-analytics"
    documentationcenter=""
    author="jeffstokes72"
    manager="jhubbard"
    editor="cgronlun"
    translationtype="Human Translation" />
<tags
    ms.assetid="5e59f893-cd2c-43fb-9eca-c146ce637203"
    ms.service="stream-analytics"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-services"
    ms.date="03/06/2017"
    wacn.date="04/17/2017"
    ms.author="jeffstok"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="ec3a83d16914ba22b89ca3c6c5e29e7975c24d66"
    ms.lasthandoff="04/07/2017" />

# <a name="stream-analytics-release-notes"></a>流分析发行说明
## <a name="notes-for-02012017-release-of-stream-analytics"></a>流分析 02/01/2017 版说明
此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| 引入了 JavaScript 用户定义的函数 (UDF) |[Java 用户定义的函数](/documentation/articles/stream-analytics-javascript-user-defined-functions/)现已推出，可在创建查询时提高灵活性。 |
| 适用于 Visual Studio 和流分析的工具简介 |[用于 Visual Studio 的工具](/documentation/articles/stream-analytics-tools-for-visual-studio/)现已推出，它们是一套可用于调试的极佳实用工具。 |
| 引入了地理空间函数 |[地理空间函数](http://msdn.microsoft.com/zh-cn/library/mt778980(Azure.100).aspx)现已推出正式版。 |
<!--| 诊断日志记录简介 |[诊断日志记录](/documentation/articles/stream-analytics-job-diagnostic-logs/) 现在已发布，因此故障诊断选项更多。 |-->
<!-- stream-analytics-job-diagnostic-logs not available in Azure.cn-->

<!--
## <a name="notes-for-04152016-release-of-stream-analytics"></a>流分析 04/15/2016 版说明
此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| Power BI 输出的公开发行版 |[Power BI 输出](/documentation/articles/stream-analytics-power-bi-dashboard/)现已正式发布。 Power BI 的 90 天授权有效期已去除。 有关需要续订授权的方案的详细信息，请参阅创建 Power BI 仪表板的“续订授权”[](/documentation/articles/stream-analytics-power-bi-dashboard/#renew-authorization)部分。 |
-->
<!-- stream-analytics-power-bi-dashboard not available in Azure.cn-->
## <a name="notes-for-03032016-release-of-stream-analytics"></a>流分析 03/03/2016 版说明
此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| 新的流分析查询语言项 |SAQL 现在具有 [GetType](https://msdn.microsoft.com/zh-cn/library/azure/mt643890.aspx "GetType MSDN 页")、[TRY_CAST](https://msdn.microsoft.com/zh-cn/library/azure/mt643735.aspx "TRY_CAST MSDN 页") 和 [REGEXMATCH](https://msdn.microsoft.com/zh-cn/library/azure/mt643891.aspx "REGEXMATCH MSDN 页")。 |

<!--
## <a name="notes-for-12102015-release-of-stream-analytics"></a>流分析 12/10/2015 版说明
此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| REST API 版本更新 |REST API 版本已更新至 2015-10-01。 可以在 MSDN 上的[流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)和[流分析中的机器学习集成](/documentation/articles/stream-analytics-how-to-configure-azure-machine-learning-endpoints-in-stream-analytics/)中找到详细信息。 |
| Azure 机器学习集成 |在此版本中，提供了对 Azure 机器学习用户定义函数的支持。 有关详细信息，请参阅[教程](/documentation/articles/stream-analytics-machine-learning-integration-tutorial/)以及[常规博客公告](http://blogs.technet.com/b/machinelearning/archive/2015/12/10/apply-azure-ml-as-a-function-in-azure-stream-analytics.aspx)。 |
-->
<!-- above available in Azure.cn-->
## <a name="notes-for-11122015-release-of-stream-analytics"></a>流分析 11/12/2015 版说明
此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| SELECT 的新行为 |流分析中的 SELECT 已扩展，以允许 * 作为嵌套记录的属性访问器。 有关更多信息，请查阅 [http://msdn.microsoft.com/zh-cn/library/mt622759.aspx“复杂数据类型”](http://msdn.microsoft.com/zh-cn/library/mt622759.aspx "复杂数据类型")。 |

## <a name="notes-for-10222015-release-of-stream-analytics"></a>流分析 10/22/2015 版说明

此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| 其他查询语言功能 |流分析通过包括以下功能扩展了查询语言：[ABS](https://msdn.microsoft.com/zh-cn/library/azure/mt574054.aspx)、[CEILING](https://msdn.microsoft.com/zh-cn/library/azure/mt605286.aspx)、[EXP](https://msdn.microsoft.com/zh-cn/library/azure/mt605289.aspx)、[FLOOR](https://msdn.microsoft.com/zh-cn/library/azure/mt605240.aspx)、[POWER](https://msdn.microsoft.com/zh-cn/library/azure/mt605287.aspx)、[SIGN](https://msdn.microsoft.com/zh-cn/library/azure/mt605290.aspx)、[SQUARE](https://msdn.microsoft.com/zh-cn/library/azure/mt605288.aspx) 和 [SQRT](https://msdn.microsoft.com/zh-cn/library/azure/mt605238.aspx)。 |
| 去除了聚合限制 |此版本去除了在一个查询中最多有 15 个聚合的限制。 现在，对每个查询的聚合数没有限制。 |
| 添加了 GROUP BY System.Timestamp 功能 |[GROUP BY](https://msdn.microsoft.com/zh-cn/library/azure/dn835023.aspx) 函数现在允许使用 window_type 或 [System.Timestamp](https://msdn.microsoft.com/zh-cn/library/azure/mt598501.aspx)。 |
| 添加了 OFFSET 用于翻转窗口和跳跃窗口 |默认情况下，[翻转](https://msdn.microsoft.com/zh-cn/library/azure/dn835055.aspx)和[跳跃](https://msdn.microsoft.com/zh-cn/library/azure/dn835041.aspx)窗口将根据零点时间 (1/1/0001 12:00:00 AM UTC) 进行调整。 新（可选）参数“offsetsize”允许指定自定义偏移量（或调整）。 |

## <a name="notes-for-09292015-release-of-stream-analytics"></a>流分析 09/29/2015 版说明

此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| Azure IoT 套件公共预览版 |流分析包含在 Azure IoT 套件的公共预览版中。 |
| Azure 门户集成 |除了在 Azure 管理门户中继续存在，流分析现在还集成在 [Azure 门户](http://portal.azure.cn)中。 请注意，门户预览版中的流分析功能目前是 Azure 管理门户中提供的功能子集，不支持浏览器内查询测试、Power BI 输出配置，以及在有权访问的订阅中浏览到或新建输入和输出资源。 |
| 支持 DocumentDB 输出 |流分析作业现在可以输出到 [DocumentDB](/home/features/documentdb/)。 |
| 支持 IoT 中心输入 |流分析作业现在可以采集来自 IoT 中心的数据。 |
| 用于异类事件的 TIMESTAMP BY |当单个数据流中包含多个其时间戳在不同字段中的事件类型时，你现在可以对表达式使用 [TIMESTAMP BY](http://msdn.microsoft.com/zh-cn/library/mt573293.aspx) ，以为每个用例指定不同时间戳字段。 |

<!--
## <a name="notes-for-09102015-release-of-stream-analytics"></a>流分析 09/10/2015 版说明

此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| 支持 PowerBI 组 |为了实现与其他 Power BI 用户共享数据，流分析作业现在可以写入到 Power BI 帐户中的 [PowerBI 组](/documentation/articles/stream-analytics-define-outputs/#power-bi)。 |
-->
<!-- above available in Azure.cn-->
## <a name="notes-for-08202015-release-of-stream-analytics"></a>流分析 08/20/2015 版说明

此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| 增加了 LAST 函数 |现在，你可以在流分析作业中使用 [LAST](http://msdn.microsoft.com/zh-cn/library/mt421186.aspx) 函数来检索给定时间框架内某个事件流中的最新事件。 |
| 新的数组函数 |现在，你可以使用数组函数 [GetArrayElement](http://msdn.microsoft.com/zh-cn/library/mt270218.aspx)、[GetArrayElements](http://msdn.microsoft.com/zh-cn/library/mt298451.aspx) 和 [GetArrayLength](http://msdn.microsoft.com/zh-cn/library/mt270226.aspx)。 |
| 新的记录函数 |现在，你可以使用记录函数 [GetRecordProperties](http://msdn.microsoft.com/zh-cn/library/mt270221.aspx) 和 [GetRecordPropertyValue](http://msdn.microsoft.com/zh-cn/library/mt270220.aspx)。 |

## <a name="notes-for-07302015-release-of-stream-analytics"></a>流分析 07/30/2015 版说明
此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| 支持 Service Bus 队列输出 |[服务总线队列](/documentation/articles/stream-analytics-define-outputs/#service-bus-queues)输出现可用于流分析作业。 |
<!--| Power BI Org Id 与 Azure Id 脱偶联 |此功能允许任何 Azure 帐户类型（Live Id 或 Org Id）下任何 ASA 作业的 [Power BI 输出](/documentation/articles/stream-analytics-power-bi-dashboard/)。 此外，你还可以将一个 Org Id 用于 Azure 帐户，将另一个用于 Power BI 输出的授权。 |-->
<!--| 支持服务总线主题输出 |[服务总线主题](/documentation/articles/stream-analytics-define-outputs/#service-bus-topics)输出现在可用于流分析作业。 |-->

## <a name="notes-for-07092015-release-of-stream-analytics"></a>流分析 07/09/2015 版说明
此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| 自定义 Blob 输出分区 |Blob 存储输出现在允许你指定写入输出 blob 的频率，以及输出数据路径文件夹的结构和格式。 |

## <a name="notes-for-05032015-release-of-stream-analytics"></a>流分析 05/03/2015 版说明
此版本包含以下更新。

| 标题 | 说明 |
| --- | --- |
| 增大“无序容许度窗口”的最大值 |“无序容许度窗口”的最大大小现在为 59:59 (MM:SS) |
| JSON 输出格式：行分隔或数组 |现在，当你输出到 Blob 存储或事件中心时，可以选择将输出作为 JSON 对象的数组，也可以使用新行来分隔 JSON 对象。 |

## <a name="notes-for-04162015-release-of-stream-analytics"></a>流分析 04/16/2015 版说明

| 标题 | 说明 |
| --- | --- |
| Azure 存储帐户配置中的延迟 |在某个区域首次创建流分析作业时，系统会提示你创建一个新的存储帐户，或者指定一个现有的帐户来监视该区域的流分析作业。 由于配置监视方面的延迟，在 30 分钟内在同一区域创建另一个流分析作业时，系统会提示你指定第二个存储帐户，而不是在“监视存储帐户”下拉列表中显示最近配置的帐户。 为了避免创建不需要的存储帐户，在某个区域首次创建作业后，可等待 30 分钟，然后再预配该区域的其他作业。 |
| 作业升级 |目前，流分析不支持实时编辑正在运行的作业的定义或配置。 若要更改正在运行的作业的输入、输出、查询、规模或配置，你必须先停止该作业。 |
| 从输入源推断出数据类型 |如果未使用 CREATE TABLE 语句，则可从输入格式推断输入类型，例如，CSV 中的所有字段都是字符串。 若要避免出现类型不匹配错误，需使用 CAST 函数将字段显式转换为正确的类型。 |
| 缺失字段将输出为 null 值 |引用输入源中不存在的字段将导致输出事件中出现 null 值。 |
| WITH 语句必须位于 SELECT 语句前面 |在你的查询中，SELECT 必须紧跟 WITH 语句中定义的子查询。 |
| 内存不足问题 |流式处理分析作业时，如果作业对无序事件的容忍度较大，而且/或者查询很复杂，需要保留大量的状态，则可能会导致作业在运行时出现内存不足的问题，使得作业重新启动。 可以在作业的操作日志中查看启动和停止操作。 若要避免出现这种行为，可以将查询横向扩展到多个分区。 在未来的版本中，为了解决这种受限问题，将会降低受影响作业的性能，而不是重新启动它们。 |
| 如果 blob 输入很大而没有负载时间戳，则可能导致内存不足问题 |如果没有通过 TIMESTAMP BY 指定时间戳字段，则在使用 Blob 存储中的大型文件时，可能导致流分析作业崩溃。 若要避免出现此问题，请将每个 blob 的大小控制在 10MB 以下。 |
| SQL 数据库事件数量限制 |使用 SQL 数据库作为输出目标时，如果输出数据量过大，则可能导致流分析作业超时。 若要解决此问题，一方面可以通过使用聚合或筛选器运算符来减少输出量，另一方面可以改为选择 Azure Blob 存储或事件中心作为输出目标。 |
| PowerBI 数据集只能包含一个表 |PowerBI 不支持在给定数据集中设置多个表。 |

## <a name="get-help"></a>获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureStreamAnalytics)

## <a name="next-steps"></a>后续步骤

* [Azure 流分析简介](/documentation/articles/stream-analytics-introduction/)
* [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
* [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
* [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
* [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)

<!--Update_Description: new article about the release notes of Stream Analytics service-->