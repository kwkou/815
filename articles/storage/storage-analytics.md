<properties
    pageTitle="使用 Azure 存储分析收集日志和指标数据 | Azure"
    description="使用存储分析，可以跟踪所有存储服务的指标数据，还可针对 Blob、队列和表存储收集日志。"
    services="storage"
    documentationcenter=""
    author="robinsh"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="7894993b-ca42-4125-8f17-8f6dfe3dca76"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="03/03/2017"
    wacn.date="03/31/2017"
    ms.author="robinsh" />

# 存储分析

Azure 存储分析执行日志记录并为存储帐户提供指标数据。可以使用此数据为存储帐户跟踪请求、分析使用趋势和诊断问题。

若要使用存储分析，必须为每个要监视的服务单独启用它。可以从 [Azure 门户预览](https://portal.azure.cn/)中启用它。有关详细信息，请参阅[在 Azure 门户预览中监视存储帐户](/documentation/articles/storage-monitor-storage-account/)。还可以通过 REST API 或客户端库以编程方式启用存储分析。使用[获取 Blob 服务属性](https://msdn.microsoft.com/zh-cn/library/hh452239.aspx)、[获取队列服务属性](https://msdn.microsoft.com/zh-cn/library/hh452243.aspx)、[获取表服务属性](https://msdn.microsoft.com/zh-cn/library/hh452238.aspx)和[获取文件服务属性](https://msdn.microsoft.com/zh-cn/library/mt427369.aspx)操作来为每个服务启用存储分析。

聚合数据存储在已知 Blob（对于日志记录）和已知表（对于指标）中，可以使用 Blob 服务和表服务 API 对其进行访问。

存储分析对于存储的数据量有 20 TB 的限制，这与存储帐户的总限制无关。有关计费和数据保留策略的详细信息，请参阅[存储分析和计费](https://msdn.microsoft.com/zh-cn/library/hh360997.aspx)。有关存储帐户限制的详细信息，请参阅 [Azure 存储可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。

有关使用存储分析及其他工具来识别、诊断和排查 Azure 存储相关问题的深入指导，请参阅[监视、诊断和排查 Azure 存储问题](/documentation/articles/storage-monitoring-diagnosing-troubleshooting/)。

## 关于存储分析日志记录
存储分析记录有关成功和失败的存储服务请求的详细信息。可以使用该信息监视各个请求和诊断存储服务问题。将尽量记录请求。

仅当存在存储服务活动时，才会创建日志条目。例如，如果存储帐户的 Blob 服务中存在活动，而表或队列服务中没有活动，则仅创建与 Blob 服务有关的日志。

存储分析日志记录不可用于 Azure 文件服务。

### 记录经过身份验证的请求
将记录以下类型的已经过身份验证的请求：

* 成功的请求。
* 失败的请求，包括超时、限制、网络、授权和其他错误。
* 使用共享访问签名 (SAS) 的请求，包括失败和成功的请求。
* 分析数据的请求。

不会记录存储分析本身发出的请求，如创建或删除日志。[存储分析记录的操作和状态消息](https://msdn.microsoft.com/zh-cn/library/hh343260.aspx)及[存储分析日志格式](https://msdn.microsoft.com/zh-cn/library/hh343259.aspx)主题中提供了所记录数据的完整列表。

### 记录匿名请求
将记录以下类型的匿名请求：

* 成功的请求。
* 服务器错误。
* 客户端和服务器的超时错误。
* 失败的 GET 请求，错误代码为 304（未修改）。

不会记录所有其他失败的匿名请求。[存储分析记录的操作和状态消息](https://msdn.microsoft.com/zh-cn/library/hh343260.aspx)及[存储分析日志格式](https://msdn.microsoft.com/zh-cn/library/hh343259.aspx)主题中提供了所记录数据的完整列表。

### 如何存储日志
所有日志以块 Blob 的形式存储在一个名为 $logs 的容器中，为存储帐户启用存储分析时将自动创建该容器。$logs 容器位于存储帐户的 Blob 命名空间中，例如：`http://<accountname>.blob.core.chinacloudapi.cn/$logs`。启用存储分析后无法删除该容器，但可以删除其内容。

>[Azure.NOTE] 执行容器列出操作（例如 [ListContainers](https://msdn.microsoft.com/zh-cn/library/azure/dd179352.aspx) 方法）时，不会显示 $logs 容器。必须直接访问该容器。例如，可以使用 [ListBlobs](https://msdn.microsoft.com/zh-cn/library/azure/dd135734.aspx) 方法访问 `$logs` 容器中的 Blob。记录请求时，存储分析以块形式上传中间结果。存储分析定期提交这些块，并将其作为 Blob 提供。

同一小时内创建的日志中可能存在重复的记录。可以通过检查 RequestId 和操作编号来确定记录是否为重复记录。

### 日志命名约定
用以下格式写入每个日志。

    <service-name>/YYYY/MM/DD/hhmm/<counter>.log

下表说明了日志名称中的每个属性。

| 属性 | 说明 |
| --- | --- |
| <service-name> |存储服务的名称例如：blob、table 或 queue。 |
| YYYY |用四位数表示的日志年份。例如：2011。 |
| MM |用两位数表示的日志月份。例如：07。 |
| DD |用两位数表示的日志月份。例如：07。 |
| hh |用两位数表示的日志起始小时，采用 24 小时 UTC 格式。例如：18。 |
| mm |用两位数表示的日志起始分钟。存储分析最新版中不支持此值，其值始终为 00。 |
| <counter> |从零开始的计数器，用六位数字表示 1 小时内为存储服务生成的日志 Blob 数。此计数器从 000000 开始。例如：000001。 |

可使用以下完整示例日志名称组合前述示例。

    blob/2011/07/31/1800/000001.log

可使用以下示例 URI 访问前述日志。

    https://<accountname>.blob.core.chinacloudapi.cn/$logs/blob/2011/07/31/1800/000001.log 

记录存储请求时，生成的日志名称与完成请求的操作时间（小时）关联。例如，如果在 2011 年 7 月 31 日下午 6:30 完成 GetBlob 请求，则会写入具有以下前缀的日志：`blob/2011/07/31/1800/`

### 日志元数据
所有日志 Blob 与可用于确定 Blob 包含哪些日志记录数据的元数据一起存储。下表说明了每个元数据属性。

| 属性 | 说明 |
| --- | --- |
| LogType |描述日志是否包含与读取、写入或删除操作有关的信息。该值可能包含一种类型，也可能包含所有三种类型的组合并用逗号隔开。示例 1：write；示例 2：read,write；示例 3：read,write,delete。 |
| StartTime |日志中的项的最早时间，采用 YYYY-MM-DDThh:mm:ssZ 形式。例如：2011-07-31T18:21:46Z。 |
| EndTime |日志中的项的最晚时间，采用 YYYY-MM-DDThh:mm:ssZ 形式。例如：2011-07-31T18:22:09Z。 |
| LogVersion |日志格式的版本。目前唯一支持的值是 1.0。 |

下表显示了使用前述示例的完整示例元数据。

* LogType=write
* StartTime=2011-07-31T18:21:46Z
* EndTime=2011-07-31T18:22:09Z
* LogVersion=1.0

### 访问日志记录数据
可以使用 Blob 服务 API（包括 Azure 托管库提供的 .NET API）访问 `$logs` 容器中的所有数据。存储帐户管理员可以读取和删除日志，但不能创建或更新日志。在查询日志时，可以使用日志的元数据和日志名称。给定小时的日志可能顺序不正确，但元数据始终指定日志中日志条目的时间范围。因此，搜索特定日志时，可以使用日志名称和元数据的组合。

## 关于存储分析指标
存储分析可存储一些指标，这些指标包括有关存储服务请求的聚合事务统计信息和容量数据。在 API 操作级别以及存储服务级别报告事务，并在存储服务级别报告容量。指标数据可用于分析存储服务使用情况，诊断对存储服务所发出请求的问题，提高使用服务的应用程序的性能。

若要使用存储分析，必须为每个要监视的服务单独启用它。可以从 [Azure 门户预览](https://portal.azure.cn/)中启用它。有关详细信息，请参阅[在 Azure 门户预览中监视存储帐户](/documentation/articles/storage-monitor-storage-account/)。还可以通过 REST API 或客户端库以编程方式启用存储分析。使用**获取服务属性**操作为每项服务启用存储分析。

### 事务指标
对于每个存储服务和请求的 API 操作，将以小时或分钟为间隔记录一组可靠的数据，其中包括入口/出口、可用性、错误和分类请求百分比。可以在[存储分析指标表架构](https://msdn.microsoft.com/zh-cn/library/hh343264.aspx)主题中查看事务详细信息的完整列表。

在两个级别记录事务数据 – 服务级别和 API 操作级别。在服务级别，汇总所有请求的 API 操作的统计信息将每小时写入一次表中条目，即使未向服务发出请求也是如此。在 API 操作级别，仅当在该小时内请求操作时才将统计信息写入条目。

例如，如果对 Blob 服务执行 **GetBlob** 操作，则存储分析指标将记录请求并将其包含在 Blob 服务以及 **GetBlob** 操作的聚合数据中。但是，如果在一小时内未请求 **GetBlob** 操作，则不会将条目写入该操作的 `$MetricsTransactionsBlob`。

为用户请求和存储分析本身发出的请求记录事务指标。例如，将记录存储分析写入日志和表中条目的请求。有关如何对这些请求进行计费的详细信息，请参阅[存储分析和计费](https://msdn.microsoft.com/zh-cn/library/hh360997.aspx)

### 容量指标

>[AZURE.NOTE] 目前，容量指标仅适用于 Blob 服务。未来版本的存储分析将提供表服务和队列服务的容量指标。

每天记录存储帐户的 Blob 服务的容量数据，并写入两个表中条目。一个条目提供用户数据的统计信息，另一个条目提供有关存储分析所使用的 `$logs` Blob 容器的统计信息。`$MetricsCapacityBlob` 表包含以下统计信息：

* **Capacity**：存储帐户的 Blob 服务使用的存储量（字节）。
* **ContainerCount**：存储帐户的 Blob 服务中的 Blob 容器数。
* **ObjectCount**：存储帐户的 Blob 服务中的提交和未提交的块或页 Blob 数量。

有关容量指标的详细信息，请参阅[存储分析指标表架构](https://msdn.microsoft.com/zh-cn/library/hh343264.aspx)。

###<a name="how-metrics-are-stored"></a> 如何存储指标
每个存储服务的所有度量数据都存储在为该服务保留的三个表中：一个表存储事务信息，一个表存储分钟事务信息，还有一个表存储容量信息。事务和分钟事务信息由请求和响应数据组成，而容量信息由存储使用情况数据组成。存储帐户的 Blob 服务的小时指标、分钟指标和容量可在下表所述名称的表中访问。

| 指标级别 | 表名 | 支持的版本 |
| --- | --- | --- |
| 小时指标，主位置 |$MetricsTransactionsBlob <br/>$MetricsTransactionsTable <br/> $MetricsTransactionsQueue |仅限 2013-08-15 之前的版本。虽然仍然支持这些名称，但还是建议你改为使用下面列出的表。 |
| 小时指标，主位置 |$MetricsHourPrimaryTransactionsBlob <br/>$MetricsHourPrimaryTransactionsTable <br/>$MetricsHourPrimaryTransactionsQueue |所有版本，包括 2013-08-15。 |
| 分钟指标，主位置 |$MetricsMinutePrimaryTransactionsBlob <br/>$MetricsMinutePrimaryTransactionsTable <br/>$MetricsMinutePrimaryTransactionsQueue |所有版本，包括 2013-08-15。 |
| 小时指标，辅助位置 |$MetricsHourSecondaryTransactionsBlob <br/>$MetricsHourSecondaryTransactionsTable <br/>$MetricsHourSecondaryTransactionsQueue |所有版本，包括 2013-08-15。必须启用读访问的地域冗余复制。 |
| 分钟指标，辅助位置 |$MetricsMinuteSecondaryTransactionsBlob <br/>$MetricsMinuteSecondaryTransactionsTable <br/>$MetricsMinuteSecondaryTransactionsQueue |所有版本，包括 2013-08-15。必须启用读访问的地域冗余复制。 |
| 容量（仅限 Blob 服务） |$MetricsCapacityBlob |所有版本，包括 2013-08-15。 |


为存储帐户启用存储分析时，将自动创建这些表。这些表通过存储帐户的命名空间进行访问，例如：`https://<accountname>.table.core.chinacloudapi.cn/Tables("$MetricsTransactionsBlob")`

### 访问指标数据
指标表中的所有数据都可以使用表服务 API 进行访问，包括 Azure 托管库提供的 .NET API。存储帐户管理员可以读取和删除表中条目，但不能创建或更新表中条目。

## 存储分析计费
所有度量数据是由存储帐户服务写入的。因此，存储分析执行的每个写入操作都是计费的。此外，度量数据使用的存储量也是计费的。

存储分析执行的以下操作都是计费的：

* 为日志记录创建 Blob 的请求。
* 为度量创建表中条目的请求。

如果配置了数据保留策略，则存储分析删除以前的日志记录和度量数据时，不会对删除事务收取费用。不过，从客户端删除事务将进行计费。有关保留策略的详细信息，请参阅[设置存储分析数据保留策略](https://msdn.microsoft.com/zh-cn/library/azure/hh343263.aspx)。

### 了解计费请求

向帐户的存储服务发出的每个请求是应计费或不计费的。存储分析记录向服务发出的每个请求，包括指示如何处理请求的状态消息。同样，存储分析存储服务及其 API 操作的度量数据，包括某些状态消息的百分比和计数。总之，这些功能可以帮助分析计费请求、对应用程序进行改进，以及诊断服务请求相关问题。有关计费的详细信息，请参阅[了解 Azure 存储计费 - 带宽、事务和容量](http://blogs.msdn.com/b/windowsazurestorage/archive/2010/07/09/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity.aspx)。

查看存储分析数据时，可以使用[存储分析记录的操作和状态消息](https://msdn.microsoft.com/zh-cn/library/azure/hh343260.aspx)主题中的表来确定计费的请求。然后，可以将日志和度量数据与状态消息进行比较，查看是否对特定请求收取了费用。也可以使用前述主题中的表来调查存储服务或各个 API 操作的可用性。

## 后续步骤
### 设置存储分析
- [在 Azure 门户预览中监视存储帐户](/documentation/articles/storage-monitor-storage-account/)
- [启用和配置存储分析](https://msdn.microsoft.com/zh-cn/library/hh360996.aspx)

### 存储分析日志记录  
- [关于存储分析日志记录](https://msdn.microsoft.com/zh-cn/library/hh343262.aspx)
- [存储分析日志格式](https://msdn.microsoft.com/zh-cn/library/hh343259.aspx)
- [存储分析记录的操作和状态消息](https://msdn.microsoft.com/zh-cn/library/hh343260.aspx)

### 存储分析指标
- [关于存储分析指标](https://msdn.microsoft.com/zh-cn/library/hh343258.aspx)
- [存储分析指标表架构](https://msdn.microsoft.com/zh-cn/library/hh343264.aspx)
- [存储分析记录的操作和状态消息](https://msdn.microsoft.com/zh-cn/library/hh343260.aspx)

<!---HONumber=Mooncake_0327_2017-->