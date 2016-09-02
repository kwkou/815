<properties 
	pageTitle="DocumentDB 规模和性能测试 | Azure" 
	description="了解如何使用 Azure DocumentDB 执行规模和性能测试"
	keywords="性能测试"
	services="documentdb" 
	authors="arramac" 
	manager="jhubbard" 
	editor="" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.date="07/18/2016" 
	wacn.date="08/08/2016"/>

# 使用 Azure DocumentDB 进行性能和规模测试
性能和规模测试是应用程序开发过程中的关键步骤。对许多应用程序而言，数据库层对整体性能和可缩放性具有相当重大的影响，因此是性能测试的关键组件。[Azure DocumentDB](/documentation/services/documentdb/) 是为了实现弹性缩放和性能可预测而构建的，因此非常适合需要高性能数据库层的应用程序。

要针对其 DocumentDB 工作负荷实施性能测试套件或针对高性能应用程序方案评估 DocumentDB 的开发人员可以参考本文。本文重点演示隔离的数据库性能测试，但也提供适用于生产应用程序的最佳实践。

阅读本文后，你将能够回答以下问题：

- 在哪里可以找到可用于 Azure DocumentDB 性能测试的示例 .NET 客户端应用程序？
- 有哪些关键因素会影响到对 Azure DocumentDB 发出的请求的端到端性能？
- 如何使用 Azure DocumentDB 从客户端应用程序实现高吞吐量级别？

若要开始处理代码，请从 [DocumentDB 性能测试示例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark)下载项目。

## 关键的客户端配置选项
DocumentDB 是一个快速、弹性的分布式数据库，可以在提供延迟与吞吐量保证的情况下无缝缩放。使用 DocumentDB 时，无需对体系结构进行重大更改或编写复杂的代码就能缩放数据库层。扩展和缩减操作就像执行单个 API 调用或 SDK 方法调用一样简单。但是，进行大规模测试时，请务必注意，需要通过网络调用来访问 DocumentDB。如果你正在编写独立的客户端应用程序以测试 DocumentDB 的性能，必须适当地配置该应用程序，以抵消网络延迟对性能度量的影响。

为了获得 DocumentDB 的最佳端到端性能，请考虑使用以下客户端配置选项：

- **增加线程/任务的数量**：由于对 DocumentDB 的调用是通过网络进行的，因此你可能需要改变请求的并行度，以便最大程度地减少客户端应用程序等待请求的时间。例如，如果使用 .NET 的[任务并行库](https://msdn.microsoft.com//library/dd460717.aspx)，请创建大约几百个读取或写入 DocumentDB 的任务。
- **在同一个 Azure 区域中测试**：尽可能地从部署在同一 Azure 区域中的虚拟机或 App Service 进行测试。通过大致的比较发现，在同一区域中对 DocumentDB 的调用可在 1-2 毫秒内完成，而美国西岸和美国东岸之间的延迟则大于 50 毫秒。
- **增加每台主机的 System.Net MaxConnections**：默认情况下，DocumentDB 请求是通过 HTTPS/REST 发出的，并受制于每个主机名或 IP 地址的默认连接限制。可能需要将此值设置为较大的值 (100-1000)，以便客户端库能够同时利用多个连接来访问 DocumentDB。在 .NET 中，此参数为 [ServicePointManager.DefaultConnectionLimit](https://msdn.microsoft.com/library/system.net.servicepointmanager.defaultconnectionlimit.aspx)。
- **打开服务器端 GC**：在某些情况下，降低垃圾收集的频率可能会有帮助。在 .NET 中，应将 [gcServer](https://msdn.microsoft.com/library/ms229357.aspx) 设置为 true。
- **使用直接连接**：为获得最佳性能，请使用[直接连接](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionmode.aspx)。
- **按 RetryAfter 间隔实现退让**：在性能测试期间，应该增加负载，直到系统对小部分请求进行限制为止。如果受到限制，客户端应用程序应按照服务器指定的重试间隔在限制时退让。这可确保最大程度地减少等待重试的时间。请参阅 [RetryAfter](https://msdn.microsoft.com/library/microsoft.azure.documents.documentclientexception.retryafter.aspx)。
- **扩大客户端工作负荷**：如果以高吞吐量级别（> 50,000 RU/秒）进行测试，客户端应用程序可能成为瓶颈，因为计算机的 CPU 或网络利用率将达到上限。如果达到此限制，可以将客户端应用程序扩展到多个服务器，以进一步推送 DocumentDB 帐户。

## 入门
最快的入门方法是根据以下步骤中所述，编译并运行下面的 .NET 示例。你也可以查看源代码，然后在自己的客户端应用程序中实施类似的配置。

**步骤 1：**从 [DocumentDB 性能测试示例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark)下载项目，或创建 Github 存储库的分支。

**步骤 2：**在 App.config 中修改 EndpointUrl、AuthorizationKey、CollectionThroughput 和 DocumentTemplate（可选）的设置。

> [AZURE.NOTE] 以高吞吐量预配集合之前，请参阅[定价页](/pricing/details/documentdb/)以估算每个集合的成本。DocumentDB 根据存储和吞吐量单独按小时计费，因此你可以通过在测试后删除或降低 DocumentDB 集合的吞吐量来节省成本。

**步骤 3：**从命令行编译并运行控制台应用。你应会看到如下输出：

	Summary:
	---------------------------------------------------------------------
	Endpoint: https://docdb-scale-demo.documents.azure.com:443/
	Collection : db.testdata at 50000 request units per second
	Document Template*: Player.json
	Degree of parallelism*: 500
	---------------------------------------------------------------------

	DocumentDBBenchmark starting...
	Creating database db
	Creating collection testdata
	Creating metric collection metrics
	Retrying after sleeping for 00:03:34.1720000
	Starting Inserts with 500 tasks
	Inserted 661 docs @ 656 writes/s, 6860 RU/s (18B max monthly 1KB reads)
	Inserted 6505 docs @ 2668 writes/s, 27962 RU/s (72B max monthly 1KB reads)
	Inserted 11756 docs @ 3240 writes/s, 33957 RU/s (88B max monthly 1KB reads)
	Inserted 17076 docs @ 3590 writes/s, 37627 RU/s (98B max monthly 1KB reads)
	Inserted 22106 docs @ 3748 writes/s, 39281 RU/s (102B max monthly 1KB reads)
	Inserted 28430 docs @ 3902 writes/s, 40897 RU/s (106B max monthly 1KB reads)
	Inserted 33492 docs @ 3928 writes/s, 41168 RU/s (107B max monthly 1KB reads)
	Inserted 38392 docs @ 3963 writes/s, 41528 RU/s (108B max monthly 1KB reads)
	Inserted 43371 docs @ 4012 writes/s, 42051 RU/s (109B max monthly 1KB reads)
	Inserted 48477 docs @ 4035 writes/s, 42282 RU/s (110B max monthly 1KB reads)
	Inserted 53845 docs @ 4088 writes/s, 42845 RU/s (111B max monthly 1KB reads)
	Inserted 59267 docs @ 4138 writes/s, 43364 RU/s (112B max monthly 1KB reads)
	Inserted 64703 docs @ 4197 writes/s, 43981 RU/s (114B max monthly 1KB reads)
	Inserted 70428 docs @ 4216 writes/s, 44181 RU/s (115B max monthly 1KB reads)
	Inserted 75868 docs @ 4247 writes/s, 44505 RU/s (115B max monthly 1KB reads)
	Inserted 81571 docs @ 4280 writes/s, 44852 RU/s (116B max monthly 1KB reads)
	Inserted 86271 docs @ 4273 writes/s, 44783 RU/s (116B max monthly 1KB reads)
	Inserted 91993 docs @ 4299 writes/s, 45056 RU/s (117B max monthly 1KB reads)
	Inserted 97469 docs @ 4292 writes/s, 44984 RU/s (117B max monthly 1KB reads)
	Inserted 99736 docs @ 4192 writes/s, 43930 RU/s (114B max monthly 1KB reads)
	Inserted 99997 docs @ 4013 writes/s, 42051 RU/s (109B max monthly 1KB reads)
	Inserted 100000 docs @ 3846 writes/s, 40304 RU/s (104B max monthly 1KB reads)

	Summary:
	---------------------------------------------------------------------
	Inserted 100000 docs @ 3834 writes/s, 40180 RU/s (104B max monthly 1KB reads)
	---------------------------------------------------------------------
	DocumentDBBenchmark completed successfully.


**步骤 4（如有必要）：**工具报告的吞吐量（RU/秒）应该等于或大于预配的集合吞吐量。如果情况并非如此，以较小的增量提高 DegreeOfParallelism 可帮助达到该限制。如果客户端应用的吞吐量达到持平状态，则在相同或不同计算机上启动多个应用程序实例可帮助在各个不同的实例中达到预配的限制。如需有关此步骤的帮助，请通过 [Ask DocumentDB](askdocdb@microsoft.com) 或[在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单来与我们联系。

应用处于运行状态后，你可以尝试不同的[索引编制策略](/documentation/articles/documentdb-indexing-policies/)和[一致性级别](/documentation/articles/documentdb-consistency-levels/)，以了解它们对吞吐量和延迟的影响。你也可以查看源代码，然后在自己的测试套件或生产应用程序中实施类似的配置。

## 摘要
在本文中，我们探讨了如何在 DocumentDB 中使用 .NET 控制台应用程序来执行性能和规模测试，另外，了解了可从 Azure DocumentDB 获得最佳性能的关键配置选项。有关使用 DocumentDB 的其他信息，请参阅下面的链接。

* [DocumentDB performance testing sample（DocumentDB 性能测试示例）](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark)
* [Server-side partitioning in DocumentDB（DocumentDB 中的服务器端分区）](/documentation/articles/documentdb-partition-data/)
* [DocumentDB 集合和性能级别](/documentation/articles/documentdb-performance-levels/)
* [MSDN 上的 DocumentDB .NET SDK 文档](https://msdn.microsoft.com/library/azure/dn948556.aspx)
* [DocumentDB .NET samples（DocumentDB .NET 示例）](https://github.com/Azure/azure-documentdb-net)

<!---HONumber=Mooncake_0801_2016-->