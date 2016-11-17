<properties 
	pageTitle="DocumentDB 性能提示 | Azure" 
	description="了解用于提高 Azure DocumentDB 数据库性能的客户端配置选项"
	keywords="如何提高数据库性能"
	services="documentdb" 
	authors="mimig1" 
	manager="jhubbard" 
	editor="" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="07/29/2016" 
	wacn.date="08/29/2016" 
	ms.author="mimig"/>

# DocumentDB 性能提示

Azure DocumentDB 是一个快速、弹性的分布式数据库，可以在提供延迟与吞吐量保证的情况下无缝缩放。使用 DocumentDB 时，无需对体系结构进行重大更改或编写复杂的代码就能缩放数据库。扩展和缩减操作就像执行单个 API 调用或 [SDK 方法调用](/documentation/articles/documentdb-performance-levels/#changing-performance-levels-using-the-net-sdk)一样简单。但是，由于 DocumentDB 是通过网络调用访问的，因此你可以通过客户端优化来获得最高性能。

如果你有“如何改善数据库性能”的疑问，请考虑以下选项。

## 联网
<a id="direct-connection"></a>

1. **连接策略：使用直接连接模式**
    
    客户端连接到 Azure DocumentDB 的方式对性能有重大影响（尤其对观察到的客户端延迟而言）。有两个重要配置设置可用于配置客户端连接策略 - 连接*模式*和[连接*协议*](#connection-protocol)。两种可用模式：

    1. 网关模式（默认）
    2. 直接模式

    由于 DocumentDB 是分布式存储系统，DocumentDB 资源（如集合）在大量的计算机之间分区，并且每个分区都经过复制以提供高可用性。物理地址转换逻辑以资源形式保留在内部提供的路由表中。

    在网关模式下，DocumentDB 网关计算机执行此路由，因此可让客户端代码变得简单且精简。客户端应用程序对 DocumentDB 网关计算机发出请求，计算机将请求中的逻辑 URI 转换为后端节点的物理地址，然后相应地转发请求。相反，直接模式客户端必须维护（且定期刷新）此路由表的副本，然后直接连接到后端 DocumentDB 节点。

    网关模式受所有 SDK 平台的支持并已配置为默认设置。如果应用程序在有严格防火墙限制的企业网络中运行，则网关模式是最佳选择，因为它使用标准 HTTPS 端口与单个终结点。但是，对于性能的影响是每次读取或写入 DocumentDB 数据时，网关模式都涉及到额外的网络跃点。因此，直接模式因为网络跃点较少，可以提供更好的性能。

2. **连接策略：使用 TCP 协议**

    利用直接模式时，有两个可用的协议选项：

    - TCP
    - HTTPS

    DocumentDB 提供基于 HTTPS 的简单开放 RESTful 编程模型。此外，它提供高效的 TCP 协议，该协议在其通信模型中也是 RESTful，可通过 .NET 客户端 SDK 获得。为了获得最佳性能，请尽可能使用 TCP 协议。

    连接模式是在构造 DocumentClient 实例期间使用 ConnectionPolicy 参数配置的。如果使用直接模式，则也可以在 ConnectionPolicy 参数中设置协议。

        var serviceEndpoint = new Uri("https://contoso.documents.net");
        var authKey = new "your authKey from Azure Mngt Portal";
        DocumentClient client = new DocumentClient(serviceEndpoint, authKey, 
        new ConnectionPolicy
        {
            ConnectionMode = ConnectionMode.Direct,
            ConnectionProtocol = Protocol.Tcp
        });

    由于只有直接模式支持 TCP，因此如果使用网关模式，HTTPS 协议始终用来与网关通信，并忽略 ConnectionPolicy 中的 Protocol 值。

    ![DocumentDB 连接策略演示](./media/documentdb-performance-tips/azure-documentdb-connection-policy.png)

3. **调用 OpenAsync 以避免首次请求时启动延迟**

    默认情况下，第一个请求因为必须提取地址路由表而有较高的延迟。为了避免首次请求时的这种启动延迟，应该在初始化期间调用 OpenAsync() 一次，如下所示。

        await client.OpenAsync();
<a id="same-region"></a>
4. **将客户端并置在同一个 Azure 区域以提高性能**

    如果可能，请将任何调用 DocumentDB 的应用程序放在与 DocumentDB 数据库相同的区域中。通过大致的比较发现，在同一区域中对 DocumentDB 的调用可在 1-2 毫秒内完成，而中国东部和中国北部之间的延迟则大于 50 毫秒。根据请求采用的路由，各项请求从客户端传递到 Azure 数据中心边界时的此类延迟可能有所不同。确保调用方应用程序位于与预配的 DocumentDB 终结点相同的 Azure 区域中，有可能会实现最低的延迟。

    ![DocumentDB 连接策略演示](./media/documentdb-performance-tips/azure-documentdb-same-region.png) 
    
<a id="increase-threads"></a>
5. **增加线程/任务数目**

    由于对 DocumentDB 的调用是通过网络执行的，因此你可能需要改变请求的并行度，以便最大程度地减少客户端应用程序等待请求的时间。例如，如果使用 .NET 的[任务并行库](https://msdn.microsoft.com/zh-cn/library/dd460717.aspx)，请创建大约几百个读取或写入 DocumentDB 的任务。

## SDK 用法

1. **安装最新的 SDK**

    DocumentDB SDK 正在不断改进，以求提供最佳性能。请参阅 [DocumentDB SDK](/documentation/articles/documentdb-sdk-dotnet/) 页来了解最新的 SDK 并查看改进项目。

2. **在应用程序生存期内使用单一实例 DocumentDB 客户端**
  
    请注意，每个 DocumentClient 实例都是线程安全的，在直接模式下运行时可执行高效的连接管理和地址缓存。若要通过 DocumentClient 获得高效的连接管理和更好的性能，建议在应用程序生存期内对每个 AppDomain 使用单个 DocumentClient 实例。
    <a id="max-connection"></a>
3. **增加每台主机的 System.Net MaxConnections**

    默认情况下，DocumentDB 请求是通过 HTTPS/REST 发出的，并受制于每个主机名或 IP 地址的默认连接限制。可能需要将此值设置为较大的值 (100-1000)，以便客户端库能够同时利用多个连接来访问 DocumentDB。在 .NET SDK 1.8.0 和更高版本中，[ServicePointManager.DefaultConnectionLimit](https://msdn.microsoft.com/zh-cn/library/system.net.servicepointmanager.defaultconnectionlimit.aspx) 的默认值为 50，若要更改此值，可将 [Documents.Client.ConnectionPolicy.MaxConnectionLimit](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.connectionpolicy.maxconnectionlimit.aspx) 设置为更大的值。

4. **打开服务器端 GC**
    
    在某些情况下，降低垃圾收集的频率可能会有帮助。在 .NET 中，应将 [gcServer](https://msdn.microsoft.com/zh-cn/library/ms229357.aspx) 设置为 true。

5. **按 RetryAfter 间隔实现退让**
 
    在性能测试期间，应该增加负载，直到系统对小部分请求进行限制为止。如果受到限制，客户端应用程序应按照服务器指定的重试间隔在限制时退让。这可确保最大程度地减少等待重试的时间。重试策略支持包含在 DocumentDB [.NET](/documentation/articles/documentdb-sdk-dotnet/) 和 [Java](/documentation/articles/documentdb-sdk-java/) 1.8.0 和更高版本中和 [Python](/documentation/articles/documentdb-sdk-python/) 1.9.0 或更高版本中。有关详细信息，请参阅 [Exceeding reserved throughput limits](/documentation/articles/documentdb-request-units/#exceeding-reserved-throughput-limits)（超过保留的吞吐量限制）和 [RetryAfter](https://msdn.microsoft.com/zh-cn/library/microsoft.azure.documents.documentclientexception.retryafter.aspx)。

6. **增大客户端工作负荷**

    如果以高吞吐量级别（> 50,000 RU/秒）进行测试，客户端应用程序可能成为瓶颈，因为计算机的 CPU 或网络利用率将达到上限。如果达到此限制，可以将客户端应用程序扩展到多个服务器，以进一步推送 DocumentDB 帐户。

7. **缓存较低读取延迟的文档 URI**

    尽可能缓存文档 URI 以获得最佳读取性能。
<a id="tune-page-size"></a>
8. **调整查询/读取源的页面大小以获得更好的性能**

    使用读取源功能（例如 ReadDocumentFeedAsync）执行批量文档读取时，或发出 DocumentDB SQL 查询时，如果结果集太大，则以分段方式返回结果。默认情况下，将以包括 100 个项的块或 1 MB 大小的块返回结果（以先达到的限制为准）。

    若要减少检索所有适用结果所需的网络往返次数，可以使用 x-ms-max-item-count 请求标头将页面大小最大增加到 1000。在只需要显示几个结果的情况下（例如，用户界面或应用程序 API 一次只返回 10 个结果），也可以将页面大小缩小为 10，以降低读取和查询所耗用的吞吐量。

    也可以使用可用的 DocumentDB SDK 设置页面大小。例如：
    
        IQueryable<dynamic> authorResults = client.CreateDocumentQuery(documentCollection.SelfLink, "SELECT p.Author FROM Pages p WHERE p.Title = 'About Seattle'", new FeedOptions { MaxItemCount = 1000 });


## 索引策略

1. **使用延迟索引加快高峰时的引入速率**

    DocumentDB 可让你在集合级别指定索引编制策略，该策略可让你选择是否要对集合中的文档自动编制索引。此外，你还可以在同步（一致）和异步（延迟）索引更新之间进行选择。默认情况下，每次在集合中插入、替换或删除文档时同步更新索引。这使查询可以与文档读取采用相同的[一致性级别](/documentation/articles/documentdb-consistency-levels/)，而不会因同步索引出现任何延迟。
    
    当发生数据写入喷发，你想要延长时间来分摊编制内容索引所需的任务时，可以考虑延迟索引。这样，你便可以有效地使用预配的吞吐量，并在高峰时以最少的延迟为写入请求提供服务。但请务必注意，启用延迟索引功能后，无论为 DocumentDB 帐户配置的一致性级别为何，查询结果最终仍保持一致。

    因此，一致索引模式（IndexingPolicy.IndexingMode 设置为 Consistent）会在每次写入时产生最高请求单位费用，而延迟索引模式（IndexingPolicy.IndexingMode 设置为 Lazy）和无索引（IndexingPolicy.Automatic 设置为 False）在写入时的索引成本为零。

2. **从索引中排除未使用的路径以加快写入速度**

    DocumentDB 的索引策略还可让你使用索引路径（IndexingPolicy.IncludedPaths 和 IndexingPolicy.ExcludedPaths）指定要在索引中包括或排除的文档路径。在事先知道查询模式的方案中，使用索引路径可改善写入性能并降低索引存储空间，因为索引成本与索引的唯一路径数目直接相关。例如，以下代码演示了如何使用“*”通配符从索引中排除文档的整个部分（也称为子树）。

        var collection = new DocumentCollection { Id = "excludedPathCollection" };
        collection.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
        collection.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/nonIndexedContent/*");
        collection = await client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("db"), excluded);

    有关详细信息，请参阅 [DocumentDB indexing policies](/documentation/articles/documentdb-indexing-policies/)（DocumentDB 索引策略）。

## 吞吐量
<a id="measure-rus"></a>

1. **测量和优化较低的每秒请求单位使用量**

    DocumentDB 提供一组丰富的数据库操作，包括 UDF 的关系和层次查询，存储的程序和触发 – 所有都在数据库集合的文档上操作。与这些操作关联的成本取决于完成操作所需的 CPU、IO 和内存。与考虑和管理硬件资源不同的是，你可以考虑将请求单位 (RU) 作为所需资源的单个措施，以执行各种数据库操作和服务应用程序请求。

    系统根据你购买的容量单位数目，为每个数据库帐户预配[请求单位](/documentation/articles/documentdb-request-units/)。请求单位消耗以每秒速率评估。如果应用程序的速率超过了为其帐户预配的请求单位速率，则会受到限制，直到该速率降到帐户的保留级别以下。如果应用程序需要更高的吞吐量，你可以购买额外容量单位。

    查询的复杂性会影响操作使用的请求单位数量。谓词数、谓词性质、UDF 数目和源数据集的大小都会影响查询操作的成本。

    若要测量任何操作（创建、更新或删除）的开销，请检查 x-ms-request-charge header 标头（或同等的 .NET SDK 中 ResourceResponse<T> 或 FeedResponse<T> 中的 RequestCharge 属性）来测量这些操作占用的请求单位数。

        // Measure the performance (request units) of writes
        ResourceResponse<Document> response = await client.CreateDocumentAsync(collectionSelfLink, myDocument);
        Console.WriteLine("Insert of document consumed {0} request units", response.RequestCharge);
        // Measure the performance (request units) of queries
        IDocumentQuery<dynamic> queryable = client.CreateDocumentQuery(collectionSelfLink, queryString).AsDocumentQuery();
        while (queryable.HasMoreResults)
             {
                  FeedResponse<dynamic> queryResponse = await queryable.ExecuteNextAsync<dynamic>();
                  Console.WriteLine("Query batch consumed {0} request units", queryResponse.RequestCharge);
             }
        
    在此标头中返回的请求费用是预配吞吐量的一小部分（即 2000 RU/秒）。例如，如果上述查询返回 1000 个 1KB 文档，则操作成本是 1000。因此在一秒内，服务器在限制后续请求之前，只接受两个此类请求。有关详细信息，请参阅 [Request units](/documentation/articles/documentdb-request-units/)（请求单位）和 [request unit calculator](https://www.documentdb.com/capacityplanner)（请求单位计算器）。

2. **处理速率限制/请求速率太大**

    当客户端尝试超过帐户保留的吞吐量时，服务器的性能不会降低，并且不使用超过保留级别的吞吐量容量。服务器将会抢先结束 RequestRateTooLarge（HTTP 状态代码 429）的请求并返回 x-ms-retry-after-ms 标头，该标头指示重试请求前用户必须等待的时间数量（以毫秒为单位）。
 
        HTTP Status 429,
        Status Line: RequestRateTooLarge
        x-ms-retry-after-ms :100

    SDK 全部都会隐式捕获此响应，并遵循服务器指定的 retry-after 标头，然后重试请求。除非多个客户端同时访问你的帐户，否则下次重试就会成功。

    如果存在多个高于请求速率的请求操作，则客户端当前在内部设置为 9 的默认重试计数可能无法满足需要；在此情况下，客户端就会向应用程序引发 DocumentClientException，其状态代码为 429。可以通过在 ConnectionPolicy 实例上设置 RetryOptions 来更改默认重试计数。默认情况下，如果请求继续以高于请求速率的方式运行，则在 30 秒的累积等待时间后将返回 DocumentClientException 和状态代码 429。即使当前的重试计数小于最大重试计数（默认值 9 或用户定义的值），也会发生这种情况。

    尽管自动重试行为有助于改善大多数应用程序的复原能力和可用性，但是在执行性能基准测试时可能会造成冲突（尤其是在测量延迟时）。如果实验达到服务器限制并导致客户端 SDK 静默重试，则客户端观测到的延迟将会剧增。若要避免性能实验期间出现延迟高峰，可以测量每个操作返回的费用，并确保请求以低于保留请求速率的方式运行。有关详细信息，请参阅 [Request units](/documentation/articles/documentdb-request-units/)（请求单位）。
   
3. **针对小型文档进行设计以提高吞吐量**

    给定操作的请求费用（即请求处理成本）与文档大小直接相关。大型文档的操作成本高于小型文档的操作成本。

## 一致性级别

1. **使用较弱的一致性级别来改善读取延迟**

    在优化 DocumentDB 应用程序性能时，另一个需要考虑的重要因素是一致性级别。一致性级别的选择会同时影响到读取和写入性能。你可以配置数据库帐户的默认一致性级别，所选的一致性级别将应用到 DocumentDB 帐户中的所有集合（跨所有数据库）。在写入操作方面，更改一致性级别所造成的影响体现在请求延迟。使用更强的一致性级别时，将增大写入延迟。另一方面，一致性级别对读取操作的影响体现在吞吐量。更弱的一致性级别可让客户端实现更高的读取吞吐量。

    所有对用户定义的资源发出的读取和查询，默认都会使用数据库帐户上指定的默认一致性级别。不过，你可以指定 x-ms-consistency-level 请求标头，来降低特定读取/查询请求的一致性级别。有关详细信息，请参阅 [Consistency levels in DocumentDB](/documentation/articles/documentdb-consistency-levels/)（DocumentDB 中的一致性级别）。

## 后续步骤

有关用于评估 DocumentDB 以使用少量客户端计算机实现高性能的示例应用程序，请参阅 [Performance and Scale Testing with Azure DocumentDB](/documentation/articles/documentdb-performance-testing/)（使用 Azure DocumentDB 进行性能和规模测试）。

此外，若要了解如何设计应用程序以实现缩放和高性能的详细信息，请参阅 [Partitioning and scaling in Azure DocumentDB](/documentation/articles/documentdb-partition-data/)（Azure DocumentDB 中的分区和缩放）。

<!---HONumber=Mooncake_0822_2016-->