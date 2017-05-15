<properties
    pageTitle="流分析输出：存储、分析选项 | Azure"
    description="了解有关设定流分析数据输出选项（包括 Power BI）目标，用于分析结果的信息。"
    keywords="数据转换, 分析结果, 数据存储选项"
    services="stream-analytics,documentdb,sql-database,event-hubs,service-bus,storage"
    documentationcenter=""
    author="jeffstokes72"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="ba6697ac-e90f-4be3-bafd-5cfcf4bd8f1f"
    ms.service="stream-analytics"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-services"
    ms.date="03/28/2017"
    wacn.date="05/15/2017"
    ms.author="jeffstok"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="f04c00a0930c165d004fb571e1df2a9fd8c97324"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="stream-analytics-outputs-options-for-storage-analysis"></a>流分析输出：存储、分析选项
创作流分析作业时，需考虑如何使用生成的数据。 如何查看流分析作业的结果？流分析作业的结果存储在何处？

为了启用多种应用程序模式，Azure 流分析提供了不同的选项来存储输出和查看分析结果。 这样可以轻松地查看作业输出，并可灵活地使用和存储作业输出，以便进行数据仓库操作和其他操作。 必须先存在作业中配置的输出，然后才能启动作业并开始事件的流动。 例如，如果你使用 Blob 存储作为输出，该作业将不会自动创建存储帐户。 在启动 ASA 作业之前，需要由用户创建该存储帐户。


## <a name="sql-database"></a>SQL 数据库
可以将 [Azure SQL 数据库](/home/features/sql-database/)用作本质上为关系型数据的输出，也可以将其用于所依赖的内容在关系数据库中托管的应用程序。 流分析作业将写入到 Azure SQL 数据库的现有表中。  请注意表架构必须与字段及其正从作业输出的类型完全匹配。 [Azure SQL 数据仓库](/documentation/services/sql-data-warehouse/)也可以通过 SQL 数据库输出选项指定为输出（此项为预览功能）。 下表列出了属性名称和用于创建 SQL 数据库输出的属性说明。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 |该名称是在查询中使用的友好名称，用于将查询输出定向到此数据库。 |
| 数据库 |数据库的名称（你正在向该数据库发送输出） |
| 服务器名称 |SQL 数据库服务器名称 |
| 用户名 |有权写入到数据库的用户名 |
| 密码 |用于连接到数据库的密码 |
| 表 |将写入输出的表名称。 表名称区分大小写，并且该表的架构应与字段数量以及作业输出正在生成的字段类型完全匹配。 |

> [AZURE.NOTE]
> 目前，流分析中的作业输出支持 Azure SQL 数据库产品/服务。 但是，不支持附加了数据库，运行 SQL Server 的 Azure 虚拟机。 这在将来的版本中可能会有所改变。
> 
> 

## <a name="blob-storage"></a>Blob 存储
Blob 存储提供了一种经济高效且可缩放的解决方案，用于在云中存储大量非结构化数据。  有关 Azure Blob 存储及其用法的简介，请参阅[如何使用 Blob](/documentation/articles/storage-dotnet-how-to-use-blobs/) 处的文档。

下表列出了用于创建 blob 输出的属性名称及其说明。

<table>
<tbody>
<tr>
<td>属性名称</td>
<td>说明</td>
</tr>
<tr>
<td>输出别名</td>
<td>该名称是在查询中使用的友好名称，用于将查询输出定向到此 blob 存储。</td>
</tr>
<tr>
<td>存储帐户</td>
<td>存储帐户的名称（你正在向该存储帐户发送输出）。</td>
</tr>
<tr>
<td>存储帐户密钥</td>
<td>与存储帐户关联的密钥。</td>
</tr>
<tr>
<td>存储容器</td>
<td>容器对存储在 Azure Blob 服务中的 blob 进行逻辑分组。 将 blob 上载到 Blob 服务时，必须为该 blob 指定一个容器。</td>
</tr>
<tr>
<td>路径前缀模式 [可选]</td>
<td>用于对指定容器中的 blob 进行编写的文件路径。<BR>在路径中，你可以选择使用以下 2 个变量的一个或多个实例指定 blob 写入的频率：<BR>{date}、{time}<BR>示例 1：cluster1/logs/{date}/{time}<BR>示例 2：cluster1/logs/{date}</td>
</tr>
<tr>
<td>日期格式 [可选]</td>
<td>如果在前缀路径中使用日期令牌，你可以选择组织文件所采用的日期格式。 示例：YYYY/MM/DD</td>
</tr>
<tr>
<td>时间格式 [可选]</td>
<td>如果在前缀路径中使用时间令牌，可指定组织文件所采用的时间格式。 目前唯一支持的值是 HH。</td>
</tr>
<tr>
<td>事件序列化格式</td>
<td>输出数据的序列化格式。  支持 JSON、CSV 和 Avro。</td>
</tr>
<tr>
<td>编码</td>
<td>如果是 CSV 或 JSON 格式，则必须指定一种编码格式。 目前只支持 UTF-8 这种编码格式。</td>
</tr>
<tr>
<td>分隔符</td>
<td>仅适用于 CSV 序列化。 流分析支持大量的常见分隔符以对 CSV 数据进行序列化。 支持的值为逗号、分号、空格、制表符和竖线。</td>
</tr>
<tr>
<td>格式</td>
<td>仅适用于 JSON 序列化。 分隔行指定了通过新行分隔各个 JSON 对象，从而格式化输出。 数组指定输出将被格式化为 JSON 对象的数组。</td>
</tr>
</tbody>
</table>

## <a name="event-hub"></a>事件中心
[事件中心](/home/features/event-hubs/)是具有高扩展性的发布-订阅事件引入器。 事件中心每秒可收集数百万个事件。  当流分析作业的输出将要成为另一个流式处理作业的输入时，可以将事件中心用作输出。

将事件中心数据流配置成输出时，需要使用几个参数。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 |该名称是在查询中使用的友好名称，用于将查询输出定向到此事件中心。 |
| 服务总线命名空间 |服务总线命名空间是包含一组消息传递实体的容器。 创建新的事件中心后，还创建了服务总线命名空间 |
| 事件中心 |事件中心输出的名称 |
| 事件中心策略名称 |可以在事件中心的“配置”选项卡上创建的共享访问策略。 每个共享访问策略都有名称、所设权限以及访问密钥 |
| 事件中心策略密钥 |用于验证对服务总线命名空间的访问权限的共享访问密钥 |
| 分区键列 [可选] |此列包含事件中心输出的分区键。 |
| 事件序列化格式 |输出数据的序列化格式。  支持 JSON、CSV 和 Avro。 |
| 编码 |对于 CSV 和 JSON，目前只支持 UTF-8 这种编码格式 |
| 分隔符 |仅适用于 CSV 序列化。 流分析支持大量的常见分隔符以对 CSV 格式的数据进行序列化。 支持的值为逗号、分号、空格、制表符和竖线。 |
| 格式 |仅适用于 JSON 类型。 分隔行指定了通过新行分隔各个 JSON 对象，从而格式化输出。 数组指定输出将被格式化为 JSON 对象的数组。 |

## <a name="table-storage"></a>表存储
[Azure 表存储](/documentation/articles/storage-introduction/)提供了具有高可用性且可大规模缩放的存储，因此应用程序可以自动缩放以满足用户需求。 表存储是 Microsoft 推出的 NoSQL 键/属性存储，适用于对架构的约束性较少的结构化数据。 Azure 表存储可用于持久地存储数据，方便进行高效的检索。

下表列出了用于创建表输出的属性名称及其说明。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 |该名称是在查询中使用的友好名称，用于将查询输出定向到此表存储。 |
| 存储帐户 |存储帐户的名称（你正在向该存储帐户发送输出）。 |
| 存储帐户密钥 |与存储帐户关联的访问密钥。 |
| 表名称 |表的名称。 如果表不存在，则会创建表。 |
| 分区键 |包含分区键的输出列的名称。 分区键是某个给定表中分区的唯一标识符，分区键构成了实体主键的第一部分。 分区键是一个最大为 1 KB 的字符串值。 |
| 行键 |包含行键的输出列的名称。 行键是某个给定分区中实体的唯一标识符。 行键构成了实体主键的第二部分。 行键是一个最大为 1 KB 的字符串值。 |
| 批大小 |批处理操作的记录数。 通常情况下，默认值对于大多数作业来说已经足够；若要修改此设置，请参阅[表批处理操作规范](https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tablebatchoperation.aspx)以获取详细信息。 |

## <a name="service-bus-queues"></a>服务总线队列
<!-- azure/hh367516.aspx redirect to service-bus-queues-topics-subscriptions-->
[服务总线队列](/documentation/articles/service-bus-queues-topics-subscriptions/)为一个或多个竞争使用方提供先入先出 (FIFO) 消息传递方式。通常情况下，接收方会按照消息添加到队列中的临时顺序来接收并处理消息，并且每条消息仅由一个消息使用方接收并处理。

下表列出了用于创建队列输出的属性名称及其说明。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 |该名称是在查询中使用的友好名称，用于将查询输出定向到此服务总线队列。 |
| 服务总线命名空间 |服务总线命名空间是包含一组消息传递实体的容器。 |
| 队列名称 |服务总线队列的名称。 |
| 队列策略名称 |在创建队列时，你还可以在“队列配置”选项卡上创建共享的访问策略。 每个共享访问策略都有名称、所设权限以及访问密钥。 |
| 队列策略密钥 |用于验证对服务总线命名空间的访问权限的共享访问密钥 |
| 事件序列化格式 |输出数据的序列化格式。  支持 JSON、CSV 和 Avro。 |
| 编码 |对于 CSV 和 JSON，目前只支持 UTF-8 这种编码格式 |
| 分隔符 |仅适用于 CSV 序列化。 流分析支持大量的常见分隔符以对 CSV 格式的数据进行序列化。 支持的值为逗号、分号、空格、制表符和竖线。 |
| 格式 |仅适用于 JSON 类型。 分隔行指定了通过新行分隔各个 JSON 对象，从而格式化输出。 数组指定输出将被格式化为 JSON 对象的数组。 |

## <a name="service-bus-topics"></a>服务总线主题
<!-- azure/hh367516.aspx redirect to service-bus-queues-topics-subscriptions -->
服务总线队列提供的是从发送方到接收方的一对一通信方法，而[服务总线主题](/documentation/articles/service-bus-queues-topics-subscriptions/)提供的则是一对多形式的通信。

下表列出了用于创建表输出的属性名称及其说明。

| 属性名称 | 说明 |
| --- | --- |
| 输出别名 |该名称是在查询中使用的友好名称，用于将查询输出定向到此服务总线主题。 |
| 服务总线命名空间 |服务总线命名空间是包含一组消息传递实体的容器。 创建新的事件中心后，还创建了服务总线命名空间 |
| 主题名称 |主题是消息传递实体，类似于事件中心和队列。 设计用于从多个不同的设备和服务收集事件流。 在创建主题时，还会为其提供特定的名称。 发送到主题的消息在创建订阅后才会提供给用户，因此请确保主题下存在一个或多个订阅 |
| 主题策略名称 |创建主题时，还可以在“主题配置”选项卡上创建共享的访问策略。 每个共享访问策略都有名称、所设权限以及访问密钥 |
| 主题策略密钥 |用于验证对服务总线命名空间的访问权限的共享访问密钥 |
| 事件序列化格式 |输出数据的序列化格式。  支持 JSON、CSV 和 Avro。 |
| 编码 |如果是 CSV 或 JSON 格式，则必须指定一种编码格式。 目前只支持 UTF-8 这种编码格式 |
| 分隔符 |仅适用于 CSV 序列化。 流分析支持大量的常见分隔符以对 CSV 格式的数据进行序列化。 支持的值为逗号、分号、空格、制表符和竖线。 |

## <a name="get-help"></a>获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureStreamAnalytics)

## <a name="next-steps"></a>后续步骤
我们已经向你介绍了流分析，这是一种托管服务，适用于对物联网的数据进行流式分析。若要了解有关此服务的详细信息，请参阅：

* [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
* [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
* [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
* [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)

<!--Link references-->
[stream.analytics.developer.guide]: /documentation/articles/stream-analytics-developer-guide/
[stream.analytics.scale.jobs]: /documentation/articles/stream-analytics-scale-jobs/
[stream.analytics.introduction]: /documentation/articles/stream-analytics-introduction/
[stream.analytics.get.started]: /documentation/articles/stream-analytics-get-started/
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: http://go.microsoft.com/fwlink/?LinkId=517301

<!-- Update_Description: update meta properties; wording update -->