<properties 
	pageTitle="关于存储分析度量值" 
	description="了解如何使用存储分析（包括事务度量值和容量度量值）、如何存储度量值，以及如何访问度量值数据。" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor="cgronlun"/>
<tags ms.service="storage"
    ms.date=""
    wacn.date="04/15/2015"
    />










# 关于存储分析度量值

## 概述

存储分析可存储一些度量值，这些度量值包括有关存储服务请求的聚合事务统计信息和容量数据。在 API 操作级别以及存储服务级别报告事务，并在存储服务级别报告容量。度量值数据可用于分析存储服务使用情况，诊断对存储服务所发出请求的问题以及提高使用服务的应用程序的性能。

若要使用存储分析，必须为每个要监视的服务单独启用它。可以从 [Azure 管理门户](https://manage.windowsazure.cn/)启用它；有关详细信息，请参阅[如何监视存储帐户](http://www.windowsazure.cn/manage/services/storage/how-to-monitor-a-storage-account/)。还可以通过 REST API 或客户端库以编程方式启用存储分析。[使用"获取 Blob 服务属性"、"获取队列服务属性"](https://msdn.microsoft.com/zh-cn/library/hh452239.aspx)和["获取表服务属性"操作为每个服务启用存储分析](https://msdn.microsoft.com/zh-cn/library/hh452238.aspx)。

## 事务度量值

对于每个存储服务和请求的 API 操作，将按小时或分钟为间隔记录一组可靠的数据，其中包括入口/出口、可用性、错误和分类请求百分比。可以在[存储分析度量值表架构](https://msdn.microsoft.com/zh-cn/library/hh343264.aspx)主题中查看事务详细信息的完整列表。

在两个级别记录事务数据 - 服务级别和 API 操作级别。在服务级别，汇总所有请求的 API 操作的统计信息将每小时写入一次表实体，即使未向服务发出请求也是如此。在 API 操作级别，仅当在该小时内请求操作时才将统计信息写入实体。

例如，如果对 BLOB 服务执行 **GetBlob** 操作，则存储分析度量值将记录请求并将其包含在 BLOB 服务以及 **GetBlob** 操作的聚合数据中。但是，如果在一小时内未请求 **GetBlob** 操作，则不会将实体写入该操作的`$MetricsTransactionsBlob`。

为用户请求和存储分析本身发出的请求记录事务度量值。例如，将记录存储分析写入日志和表实体的请求。有关如何对这些请求进行计费的详细信息，请参阅[存储分析和计费](https://msdn.microsoft.com/zh-cn/library/hh360997.aspx)

## 容量度量值

>[AZURE.NOTE] 目前，容量度量值仅适用于 BLOB 服务。以后版本的存储分析将提供表服务和队列服务的容量度量值。

每天记录存储帐户的 BLOB 服务的容量数据，并写入两个表实体。一个实体提供用户数据的统计信息，另一个实体提供有关存储分析所使用的 `$logs` Blob 容器的统计信息。 `$MetricsCapacityBlob` 表包含以下统计信息：

- **Capacity**：存储帐户的 Blob 服务使用的存储量（字节）。

- **ContainerCount**：存储帐户的 Blob 服务中的 Blob 容器数。

- **ObjectCount**：存储帐户的 Blob 服务中的提交和未提交的块或页 Blob 数量。

有关容量度量值的详细信息，请参阅"存储分析度量值表架构"。

## 如何存储度量值

每个存储服务的所有度量数据都存储在为该服务保留的三个表中：一个表存储事务信息，一个表存储分钟事务信息，还有一个表存储容量信息。事务和分钟事务信息由请求和响应数据组成，而容量信息由存储使用情况数据组成。存储帐户的 Blob 服务的小时度量值、分钟度量值和容量可在按下表所述命名的表中访问。

| 度量值级别                      	| 表名                                                                                                                 	| 支持的版本                                                                                                                       	|
|------------------------------------	|-----------------------------------------------------------------------------------------------------------------------------	|----------------------------------------------------------------------------------------------------------------------------------------------	|
| 每小时度量值，主位置   	|  $MetricsTransactionsBlob <br/>$MetricsTransactionsTable <br/> $MetricsTransactionsQueue                                              	| 仅限 2013-08-15 之前的版本。虽然仍然支持这些名称，但还是建议你改为使用下面列出的表。 	|
| 每小时度量值，主位置   	| $MetricsHourPrimaryTransactionsBlob <br/>$MetricsHourPrimaryTransactionsTable <br/>$MetricsHourPrimaryTransactionsQueue             	| 所有版本，包括 2013-08-15                                                                                                           	|
| 分钟度量值，主位置   	| $MetricsMinutePrimaryTransactionsBlob <br/>$MetricsMinutePrimaryTransactionsTable <br/>$MetricsMinutePrimaryTransactionsQueue       	| 所有版本，包括 2013-08-15                                                                                                           	|
| 每小时度量值，辅助位置 	| $MetricsHourSecondaryTransactionsBlob <br/>$MetricsHourSecondaryTransactionsTable <br/>$MetricsHourSecondaryTransactionsQueue       	| 所有版本，包括 2013-08-15。必须启用读访问的地域冗余复制。                                                   	|
| 分钟度量值，辅助位置 	| $MetricsMinuteSecondaryTransactionsBlob <br/>$MetricsMinuteSecondaryTransactionsTable <br/>$MetricsMinuteSecondaryTransactionsQueue 	| 所有版本，包括 2013-08-15。必须启用读访问的地域冗余复制。                                                   	|
| 容量（仅限 Blob 服务）       	| $MetricsCapacityBlob                                                                                                        	| 所有版本，包括 2013-08-15                                                                                                           	|



为存储帐户启用存储分析时，将自动创建这些表。这些表通过存储帐户的命名空间进行访问，例如： `https://<accountname>.table.core.chinacloudapi.cn/Tables("$MetricsTransactionsBlob")`


## 访问度量值数据

度量值表中的所有数据都可以使用表服务 API 进行访问，包括 Azure 托管库提供的 .NET API。存储帐户管理员可以读取和删除表实体，但不能创建或更新表实体。

## 后续步骤
### 概念
[启用和配置存储分析](https://msdn.microsoft.com/zh-cn/library/hh360996.aspx)

[存储分析度量值表架构](https://msdn.microsoft.com/zh-cn/library/hh343264.aspx)

[存储分析记录的操作和状态消息](https://msdn.microsoft.com/zh-cn/library/hh343260.aspx)

[关于存储分析日志记录](https://msdn.microsoft.com/zh-cn/library/hh343262.aspx)

### 其他资源

[如何监视存储帐户](http://www.windowsazure.cn/manage/services/storage/how-to-monitor-a-storage-account/)

<!--HONumber=50-->