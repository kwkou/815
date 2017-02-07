<properties
    pageTitle="使用 DocumentDB 全局分发数据 | Azure"
    description="了解如何通过 Azure DocumentDB（一种完全托管的 NoSQL 数据库服务），使用全局数据库进行全球范围的异地复制、故障转移和数据恢复。"
    services="documentdb"
    documentationcenter=""
    author="kiratp"
    manager="jhubbard"
    editor="" />  

<tags
    ms.assetid="ba5ad0cc-aa1f-4f40-aee9-3364af070725"
    ms.service="documentdb"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/16/2016"
    wacn.date="02/06/2017"
    ms.author="kipandya" />

# 使用 DocumentDB 全局分发数据
> [AZURE.NOTE]
DocumentDB 数据库全局分发功能已正式推出，所有新建的 DocumentDB 帐户将自动启用该功能。我们正在努力为所有现有帐户启用全局分发，但在此之前，如果要为帐户启用全局分发，请[与支持部门联系](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)，我们可以帮助你启用。
> 
> 

Azure DocumentDB 旨在满足由数百万个全球分布式设备组成的 IoT 应用程序，以及向全球各地的用户提供快速响应体验的 Internet 级应用程序的需求。这些数据库系统面临着这样的挑战：允许多个地理区域的用户以较低的延迟访问应用程序数据，同时还要履行明确规定的数据一致性和可用性保证。作为全局分布式数据库系统，DocumentDB 可通过提供完全托管的多区域数据库帐户来简化全局数据分发。这些帐户在一致性、可用性和性能之间提供明确的折衷，并且全部附带了相应的保证。所提供的 DocumentDB 数据库帐户具有以下特点：高可用性、10 毫秒以下的延迟、多个[妥善定义的一致性级别][consistency]、使用多宿主 API 实现透明的区域性故障转移，以及在全球范围内弹性缩放吞吐量和存储。

  
## 配置多区域帐户
通过 [Azure 门户预览](/documentation/articles/documentdb-portal-global-replication/)，在不到一分钟内就能将 DocumentDB 帐户配置为在全球范围内进行缩放。你只需在多个妥善定义的受支持一致性级别中选择适当的一致性级别，然后将任意数目的 Azure 区域与你的数据库帐户相关联。DocumentDB 一致性级别在特定的一致性保证与性能之间提供明确的折衷。

![DocumentDB 提供多个妥善定义的（宽松）一致性模型供你选择][1]  


DocumentDB 提供多个妥善定义的（宽松）一致性模型供你选择。

根据应用程序所需的一致性保证选择适当的一致性级别。DocumentDB 可跨所有指定的区域自动复制你的数据，并针对你为数据库帐户选择的一致性提供保证。

## 使用多区域故障转移
Azure DocumentDB 能够跨多个 Azure 区域以透明方式故障转移数据库帐户 — 新的[多宿主 API][developingwithmultipleregions] 保证应用可以继续使用逻辑终结点，并且不受故障转移的干扰。故障转移由你自己控制，在发生任何范围的故障状况时（包括应用程序、基础结构、服务或区域性故障，不管是实际的还是模拟的），可让你灵活复位数据库帐户。在发生 DocumentDB 区域性故障时，该服务将以透明方式故障转移你的数据库帐户，然后，你的应用程序可继续访问数据而不会失去可用性。DocumentDB 提供 [99\.99% 的可用性 SLA][sla]，但也可以测试应用程序的端到端可用性属性，方法是[以编程方式][arm]或者通过 Azure 门户预览模拟区域性故障。

## 在全球范围内缩放 <a name="scaling-across-the-planet"></a>
DocumentDB 允许你在数据库帐户关联的所有区域中，以任何规模针对每个 DocumentDB 集合全局独立预配吞吐量以及使用存储。DocumentDB 集合将在数据库帐户关联的所有区域之间自动进行全局分发和管理。

为每个 DocumentDB 集合购买的吞吐量和使用的存储将以均等方式在所有区域之间自动预配。这样，你的应用程序便可以在全球范围内无缝缩放，而你[只需为每个小时内使用的吞吐量和存储付费][pricing]。例如，如果你为某个 DocumentDB 集合预配了 200 万个 RU，则与数据库帐户关联的每个区域将获得该集合的 200 万个 RU。下面对此做了演示。

![跨四个区域缩放 DocumentDB 集合的吞吐量][2]

DocumentDB 保证读取延迟小于 10 毫秒，写入延迟小于 15 毫秒，级别为 P99。读取请求永不跨过数据中心边界，可保证[所选的一致性要求][consistency]。写入请求在经客户端确认之前，始终在本地提交仲裁。每个数据库帐户都配置了写入区域优先级。指定了最高优先级的区域将充当帐户的当前写入区域。所有 SDK 以透明方式将数据库帐户写入请求路由到当前写入区域。

最后，由于 DocumentDB 完全是[架构不可知的][vldb]，因此，你永远无需担心如何跨多个数据中心管理/更新架构或辅助索引。当应用程序和数据模型不断演变时，[SQL 查询][sqlqueries]可继续正常工作。

## 启用全局分发
可以通过将一个或多个 Azure 区域与 DocumentDB 数据库帐户相关联，在本地分发或者全局分发数据。可以随时在数据库帐户中添加或删除区域。若要允许通过门户进行全局分发，请参阅[如何使用 Azure 门户预览执行 DocumentDB 全局数据库复制](/documentation/articles/documentdb-portal-global-replication/)。若要以编程方式进行全局分发，请参阅[使用多区域 DocumentDB 帐户进行开发](/documentation/articles/documentdb-developing-with-multiple-regions/)。

## 后续步骤
在以下文章中了解有关使用 DocumentDB 全局分发数据的详细信息：

- [Provisioning throughput and storage for a collection（为集合预配吞吐量和存储）][throughputandstorage]
- [Multi-homing APIs via REST. .NET, Java, Python, and Node SDKs（通过 REST、.NET、Java、Python 和 Node SDK 使用多宿主 API）][developingwithmultipleregions]
- [Consistency Levels in DocumentDB（DocumentDB 中的一致性级别）][consistency]
- [可用性 SLA][sla]
- [Managing database account（管理数据库帐户）][manageaccount]

[1]: ./media/documentdb-distribute-data-globally/consistency-tradeoffs.png
[2]: ./media/documentdb-distribute-data-globally/collection-regions.png

<!--Reference style links - using these makes the source content way more readable than using inline links-->
[pcolls]: /documentation/articles/documentdb-partition-data/
[consistency]: /documentation/articles/documentdb-consistency-levels/
[consistencytradeooffs]:/documentation/articles/documentdb-consistency-levels/#consistency-levels-and-tradeoffs/
[developingwithmultipleregions]: /documentation/articles/documentdb-developing-with-multiple-regions/
[createaccount]: /documentation/articles/documentdb-create-account/
[manageaccount]: /documentation/articles/documentdb-manage-account/
[manageaccount-consistency]: /documentation/articles/documentdb-manage-account/#consistency/
[throughputandstorage]: /documentation/articles/documentdb-manage/
[arm]: /documentation/articles/documentdb-automation-resource-manager-cli/
[regions]: https://azure.microsoft.com/regions/
[pricing]: /pricing/details/documentdb/
[sla]: /support/legal/
[vldb]: http://www.vldb.org/pvldb/vol8/p1668-shukla.pdf
[sqlqueries]: /documentation/articles/documentdb-sql-query/

<!---HONumber=Mooncake_Quality_Review_0125_2017-->
