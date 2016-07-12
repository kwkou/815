<properties
	pageTitle="数据连接：事件流中的数据流输入 | Azure"
	description="了解如何设置连接到流分析的名为“输入”的数据连接。输入包括来自事件的数据流，也包括引用数据。"
	keywords="数据流、数据连接、事件流"
	services="stream-analytics"
	documentationCenter=""
	authors="jeffstokes72"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="stream-analytics"
	ms.date="05/03/2016"
	wacn.date="05/30/2016"/>
# 数据连接：了解从事件到流分析的数据流输入

连接到流分析的数据连接是数据源提供的事件所组成的数据流。这称为“输入”。 流分析与 Azure 数据流源（事件中心、IoT 中心和 Blob 存储）进行第一类集成，这些数据流源可能与你的分析作业来自同一个 Azure 订阅，也可能来自不同的 Azure 订阅。

## 数据输入类型：数据流和引用数据
将数据推送到数据源时，流分析作业会使用该数据并对其进行实时处理。输入分为两种不同类型：数据流输入和引用数据输入。

### 数据流输入
数据流是一段时间内不受限制的事件序列。流分析作业必须包含至少一种可供作业使用和转换的数据流输入。Blob 存储、事件中心和 IoT 中心均可作为数据流输入源。事件中心用于从多个设备和服务（例如传感器提供的社交媒体活动源、股票交易信息或数据）收集事件流。IoT 中心经过优化以从物联网 (IoT) 方案中连接的设备收集数据。Blob 存储可用作按流的形式引入大量数据的输入源。

### 引用数据
流分析支持称为引用数据的第二类输入。此类数据为辅助数据，处于静态或者缓慢变化状态，通常用于执行关联性操作和查找操作。目前只支持使用 Azure Blob 存储作为引用数据的输入源。引用数据源 Blob 存在 100MB 的大小限制。
	若要了解如何创建引用数据输入，请参阅[使用引用数据](/documentation/articles/stream-analytics-use-reference-data/)

## 通过事件中心创建数据流输入

[Azure 事件中心](/services/event-hubs/)是具有高扩展性的发布-订阅事件引入器。事件中心每秒可收集数百万个事件，使你能够处理和分析互连设备与应用程序生成的海量数据。事件中心是最常见的流分析输入。事件中心和流分析一起为客户提供端到端的解决方案以进行实时分析。事件中心允许客户实时将事件输入到 Azure 中，流分析作业可实时处理这些事件。例如，客户可以将 Web 点击操作、传感器读数、联机日志事件发送到事件中心，然后创建流分析作业，将事件中心用作输入数据流，以便进行实时筛选、聚合和关联操作。

需要注意的是，来自流分析中事件中心的事件默认时间戳是事件到达事件中心的时间戳，即 EventEnqueuedUtcTime。若要在事件负载中使用时间戳以流方式处理数据，必须使用 [TIMESTAMP BY](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx) 关键字。

### 使用者组

应对每个流分析事件中心输入进行配置，使之拥有自己的使用者组。如果作业包含自联接或多个输入，部分输入可能会由下游的多个读取器读取，这会影响单个使用者组中的读取器数目。为了避免超出针对事件中心设置的每个分区每个使用者组 5 个读取器的限制，最好是为每个流分析作业指定一个使用者组。请注意还有一项限制，即每个事件中心最多只能有 20 个使用者组。有关详细信息，请参阅[事件中心编程指南](/documentation/articles/event-hubs-programming-guide/)。

## 将事件中心配置为输入数据流

下表在属性说明中介绍了事件中心输入选项卡中的每个属性：

| 属性名称 | 说明 |
|------|------|
| 输入别名 | 一个友好名称会用于作业查询，以便引用此输入 |
| 服务总线命名空间 | 服务总线命名空间是包含一组消息传递实体的容器。当你创建新的事件中心后，你还创建了 Service Bus 命名空间。 |
| 事件中心 | 事件中心输入的名称。 |
| 事件中心策略名称 | 可以在事件中心的“配置”选项卡上创建的共享访问策略。每个共享访问策略都会有名称、所设权限以及访问密钥。 |
| 事件中心策略密钥 | 用于验证对服务总线命名空间的访问权限的共享访问密钥。 |
| 事件中心使用者组（可选） | 可以从事件中心引入数据的使用者组。如果未指定，流分析作业将使用默认使用者组从事件中心引入数据。建议为每个流分析作业使用不同的使用者组。 |
| 事件序列化格式 | 为确保查询按预计的方式运行，流分析需要了解你对传入数据流使用哪种序列化格式（JSON、CSV 或 Avro）。 |
| 编码 | 目前只支持 UTF-8 这种编码格式。 |

当你的数据来自事件中心源时，你可以在流分析查询中访问几个元数据字段。下表列出了这些字段及其说明。

| 属性 | 说明 |
|------|------|
| EventProcessedUtcTime | 流分析处理事件的日期和时间。 |
| EventEnqueuedUtcTime | 事件中心收到事件的日期和时间。 |
| PartitionId | 输入适配器的从零开始的分区 ID。 |

例如，你可以编写类似以下的查询：

````
SELECT
	EventProcessedUtcTime,
	EventEnqueuedUtcTime,
	PartitionId
FROM Input
````

## 创建 IoT 中心数据流输入

Azure Iot 中心是已针对 IoT 进行优化，具有高度可缩放性的发布-订阅事件引入器。
需要注意的是，来自流分析中 IoT 中心的事件默认时间戳是事件到达 IoT 中心的时间戳，即 EventEnqueuedUtcTime。若要在事件负载中使用时间戳以流方式处理数据，必须使用 [TIMESTAMP BY](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx) 关键字。

### 使用者组

应对每个流分析 IoT 中心输入进行配置，使之拥有自己的使用者组。如果作业包含自联接或多个输入，部分输入可能会由下游的多个读取器读取，这会影响单个使用者组中的读取器数目。为了避免超出针对 IoT 中心设置的每个分区每个使用者组 5 个读取器的限制，最好是为每个流分析作业指定一个使用者组。

## 将 IoT 中心配置为数据流输入

下表在属性说明中介绍了 IoT 中心输入选项卡中的每个属性：

| 属性名称 | 说明 |
|------|------|
| 输入别名 | 一个友好名称会用于作业查询，以便引用此输入。 |
| IoT 中心 | IoT 中心是包含一组消息实体的容器。 |
| 终结点 | IoT 中心终结点的名称。 |
| 共享访问策略名称 | 用于提供对 IoT 中心的访问权限的共享访问策略。每个共享访问策略都会有名称、所设权限以及访问密钥。 |
| 共享访问策略密钥 | 用于验证对 IoT 中心的访问权限的共享访问密钥。 |
| 使用者组（可选） | 可以从 IoT 中心引入数据的使用者组。如果未指定，流分析作业将使用默认使用者组从 IoT 中心引入数据。建议为每个流分析作业使用不同的使用者组。 |
| 事件序列化格式 | 为确保查询按预计的方式运行，流分析需要了解你对传入数据流使用哪种序列化格式（JSON、CSV 或 Avro）。 |
| 编码 | 目前只支持 UTF-8 这种编码格式。 |

当你的数据来自 IoT 中心源时，你可以在流分析查询中访问很少元数据字段。下表列出了这些字段及其说明。

| 属性 | 说明 |
|------|------|
| EventProcessedUtcTime | 处理事件的日期和时间。 |
| EventEnqueuedUtcTime | IoT 中心收到事件的日期和时间。 |
| PartitionId | 输入适配器的从零开始的分区 ID。 |
| IoTHub.MessageId | 用于关联 IoT 中心内的双向通信。 |
| IoTHub.CorrelationId | 用于 IoT 中心内的消息响应和反馈。 |
| IoTHub.ConnectionDeviceId | 经过身份验证的 ID，用于发送此消息、由 IoT 中心在服务绑定的消息上加盖标记。 |
| IoTHub.ConnectionDeviceGenerationId | 经过验证的设备的 generationId，用于发送此消息、由 IoT 中心在服务绑定的消息上加盖标记。 |
| IoTHub.EnqueuedTime | IoT 中心收到消息的时间。 |
| IoTHub.StreamId | 由发送方设备添加的自定义事件属性。 |

## 创建 Blob 存储数据流输入

对于需要将大量非结构化数据存储在云中的情况，Blob 存储提供了一种经济高效且可伸缩的解决方案。通常情况下，可以将 [Blob 存储](/services/storage/)中的数据视为“静态”数据，但这些数据可以作为数据流由流分析进行处理。流分析使用 Blob 存储输入的一种常见情况是进行日志处理，即首先从某个系统捕获遥测数据，然后根据需要对这些数据进行分析和处理以提取有意义的数据。

需要注意的是，流分析中 Blob 存储事件的默认时间戳是上次修改 blob 的时间戳，即 *isBlobLastModifiedUtcTime*。若要在事件负载中使用时间戳以流方式处理数据，必须使用 [TIMESTAMP BY](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx) 关键字。

> [AZURE.NOTE] 流分析不支持将内容添加到现有 Blob。流分析只会查看 Blob 一次，在这项读取操作后所做的任何更改都不会得到处理。最佳实践是一次性上载所有数据，而不要在 Blob 存储中添加其他任何事件。

下表在属性说明中介绍了 Blob 存储输入选项卡中的每个属性：

<table>
<tbody>
<tr>
<td>属性名称</td>
<td>说明</td>
</tr>
<tr>
<td>输入别名</td>
<td>一个友好名称会用于作业查询，以便引用此输入。</td>
</tr>
<tr>
<td>存储帐户</td>
<td>存储 blob 文件的存储帐户的名称。</td>
</tr>
<tr>
<td>存储帐户密钥</td>
<td>与存储帐户关联的密钥。</td>
</tr>
<tr>
<td>存储容器
</td>
<td>容器对存储在 Azure Blob 服务中的 blob 进行逻辑分组。将 blob 上载到 Blob 服务时，必须为该 blob 指定一个容器。</td>
</tr>
<tr>
<td>路径前缀模式 [可选]</td>
<td>用于对指定容器中的 blob 进行定位的文件路径。在路径中，你可以选择指定一个或多个使用以下 3 个变量的实例：<BR>{date}、{time}、<BR>{partition}<BR>示例 1：cluster1/logs/{date}/{time}/{partition}<BR>示例 2：cluster1/logs/{date}<P>请注意，“*”不是路径前缀允许使用的值。仅允许使用有效的 <a HREF="https://msdn.microsoft.com/zh-cn/library/azure/dd135715.aspx">Azure blob 字符</a>。</td>
</tr>
<tr>
<td>日期格式 [可选]</td>
<td>如果在前缀路径中使用日期令牌，你可以选择组织文件所采用的日期格式。示例：YYYY/MM/DD</td>
</tr>
<tr>
<td>时间格式 [可选]</td>
<td>如果在前缀路径中使用时间令牌，你可以选择组织文件所采用的时间格式。目前唯一支持的值是 HH。</td>
</tr>
<tr>
<td>事件序列化格式</td>
<td>为确保查询按预计的方式运行，流分析需要了解你对传入数据流使用哪种序列化格式（JSON、CSV 或 Avro）。</td>
</tr>
<tr>
<td>编码</td>
<td>对于 CSV 和 JSON，目前只支持 UTF-8 这种编码格式。</td>
</tr>
<tr>
<td>分隔符</td>
<td>流分析支持大量的常见分隔符以对 CSV 格式的数据进行序列化。支持的值为逗号、分号、空格、制表符和竖线。</td>
</tr>
</tbody>
</table>

当你的数据来自 Blob 存储源时，你可以在流分析查询中访问几个元数据字段。下表列出了这些字段及其说明。

| 属性 | 说明 |
|------|------|
| BlobName | 提供事件的输入 blob 的名称。 |
| EventProcessedUtcTime | 流分析处理事件的日期和时间。 |
| BlobLastModifiedUtcTime | 上次修改 blob 的日期和时间。 |
| PartitionId | 输入适配器的从零开始的分区 ID。 |

例如，你可以编写类似以下的查询：

````
SELECT
	BlobName,
	EventProcessedUtcTime,
	BlobLastModifiedUtcTime
FROM Input
````


## 获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureStreamAnalytics)

## 后续步骤
你已经了解了 Azure 中针对流分析作业的数据连接选项。若要了解流分析的更多内容，请参阅：

- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)

<!--Link references-->
[stream.analytics.developer.guide]: /documentation/articles/stream-analytics-developer-guide/
[stream.analytics.scale.jobs]: /documentation/articlesstream-analytics-scale-jobs
[stream.analytics.introduction]: /documentation/articlesstream-analytics-introduction
[stream.analytics.get.started]: /documentation/articlesstream-analytics-get-started
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: http://go.microsoft.com/fwlink/?LinkId=517301

<!---HONumber=Mooncake_0314_2016-->