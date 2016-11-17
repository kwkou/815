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
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="07/21/2016" 
	ms.author="arramac"
   	wacn.date="10/18/2016"/>  


# 使用 Azure DocumentDB 进行性能和规模测试
性能和规模测试是应用程序开发过程中的关键步骤。对许多应用程序而言，数据库层对整体性能和可缩放性具有相当重大的影响，因此是性能测试的关键组件。[Azure DocumentDB](/home/features/documentdb/) 是为了实现弹性缩放和性能可预测而构建的，因此非常适合需要高性能数据库层的应用程序。

要针对其 DocumentDB 工作负荷实施性能测试套件或针对高性能应用程序方案评估 DocumentDB 的开发人员可以参考本文。本文重点演示隔离的数据库性能测试，但也提供适用于生产应用程序的最佳实践。

阅读本文后，你将能够回答以下问题：

- 在哪里可以找到可用于 Azure DocumentDB 性能测试的示例 .NET 客户端应用程序？
- 如何使用 Azure DocumentDB 从客户端应用程序实现高吞吐量级别？

若要开始处理代码，请从 [DocumentDB 性能测试示例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark)下载项目。

> [AZURE.NOTE] 此应用程序的目标是演示使用少数客户端计算机从 DocumentDB 中提取更好性能的最佳做法。生成此应用程序的目的不是为了演示服务的峰值容量如何可以无限扩展。

如果正在寻找用于提高 DocumentDB 性能的客户端配置选项，请参阅 [DocumentDB 性能提示](/documentation/articles/documentdb-performance-tips/)。

## 运行性能测试应用程序
最快的入门方法是根据以下步骤中所述，编译并运行下面的 .NET 示例。你也可以查看源代码，然后在自己的客户端应用程序中实施类似的配置。

**步骤 1：**从 [DocumentDB 性能测试示例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark)下载项目，或创建 Github 存储库的分支。

**步骤 2：**在 App.config 中修改 EndpointUrl、AuthorizationKey、CollectionThroughput 和 DocumentTemplate（可选）的设置。

> [AZURE.NOTE] 为集合预配高吞吐量之前，请参阅[定价页](/pricing/details/documentdb/)以估算每个集合的成本。DocumentDB 根据存储和吞吐量单独按小时计费，因此你可以通过在测试后删除或降低 DocumentDB 集合的吞吐量来节省成本。

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


**步骤 4（如有必要）：**工具报告的吞吐量（RU/秒）应该等于或大于预配的集合吞吐量。如果情况并非如此，以较小的增量提高 DegreeOfParallelism 可帮助达到该限制。如果客户端应用的吞吐量达到持平状态，则在相同或不同计算机上启动多个应用程序实例可帮助在各个不同的实例中达到预配的限制。如果需要此步骤的帮助，请撰写电子邮件并将其发送到 askdocdb@microsoft.com 或填写支持票证。

让应用处于运行状态后，可以尝试不同的[编制索引策略](/documentation/articles/documentdb-indexing-policies/)和[一致性级别](/documentation/articles/documentdb-consistency-levels/)，以了解它们对吞吐量和延迟的影响。你也可以查看源代码，然后在自己的测试套件或生产应用程序中实施类似的配置。

## 后续步骤
本文介绍了如何使用 .NET 控制台应用对 DocumentDB 执行性能和规模测试。有关使用 DocumentDB 的其他信息，请参阅下面的链接。

* [DocumentDB performance testing sample（DocumentDB 性能测试示例）](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark)
* [用于提高 DocumentDB 性能的客户端配置选项](/documentation/articles/documentdb-performance-tips/)
* [Server-side partitioning in DocumentDB（DocumentDB 中的服务器端分区）](/documentation/articles/documentdb-partition-data/)
* [DocumentDB 集合和性能级别](/documentation/articles/documentdb-performance-levels/)
* [MSDN 上的 DocumentDB .NET SDK 文档](https://msdn.microsoft.com/zh-cn/library/azure/dn948556.aspx)
* [DocumentDB .NET samples（DocumentDB .NET 示例）](https://github.com/Azure/azure-documentdb-net)
* [DocumentDB 性能提示博客](https://azure.microsoft.com/blog/2015/01/20/performance-tips-for-azure-documentdb-part-1-2/)

<!---HONumber=Mooncake_1010_2016-->
