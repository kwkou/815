<properties 
	pageTitle="Azure Redis 缓存常见问题" 
	description="了解常见问题的答案，以及有关 Azure Redis 缓存的模式和最佳实践" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="cache"
	ms.date="04/20/2016"
	wacn.date="06/29/2016"/>

# Azure Redis 缓存常见问题

了解常见问题的答案，以及有关 Azure Redis 缓存的模式和最佳实践。

##<a name="cache-size" id="what-redis-cache-offering-and-size-should-i-use"></a> 我应使用哪种 Redis 缓存产品和大小？
每款 Azure Redis 缓存产品在**大小**、**带宽**、**高可能性**和 **SLA** 方面提供不同的级别。

以下是选择缓存产品的注意事项。

-	**内存**：基本级别和标准级别提供 250 MB - 53 GB。高级层最多提供 530 GB。有关详细信息，请参阅 [Azure Redis 缓存定价](/home/features/redis-cache/pricing/)。
-	**网络性能**：如果你的工作负荷需要高的吞吐量，则可使用高级级别，该级别提供比标准级别或基本级别更高的带宽。另外，在每个级别中，缓存大小越大，带宽越高，因为是由基础 VM 托管缓存。有关详细信息，请参阅[下表](#cache-performance)。
-	**吞吐量**：高级级别提供的可用吞吐量最大。如果缓存服务器或客户端达到带宽限制，客户端上会出现超时。有关详细信息，请参阅下表。
-	**高可用性/SLA**：Azure Redis 缓存保证标准/高级缓存在至少 99.9% 的时间内都可用。若要了解有关 SLA 的详细信息，请参阅 [Azure Redis 缓存定价](/home/features/redis-cache/pricing/)。SLA 仅涉及与缓存终结点的连接。SLA 不涉及对数据丢失的防护。我们建议使用高级级别中的 Redis 数据暂留功能来增加灵活性，防止数据丢失。
-	**Redis 数据持久性**：高级级别允许你将缓存数据暂留在 Azure 存储帐户中。在基本/标准缓存中，所有数据只存储在内存中。如果底层基础结构出现问题，可能会导致数据丢失。我们建议使用高级级别中的 Redis 数据暂留功能来增加灵活性，防止数据丢失。Azure Redis 缓存提供可在 Redis 暂留中使用的 RDB 和 AOF（即将推出）选项。有关详细信息，请参阅[如何为高级 Azure Redis 缓存配置持久性](/documentation/articles/cache-how-to-premium-persistence/)。
-	**Redis 群集**：如果你想要创建大于 53 GB 的缓存，或者想要将数据通过分片的方式分散到多个 Redis 节点中，则可以使用在高级级别中提供的 Redis 群集功能。每个节点都包含一个主/副缓存对，目的是提高可用性。有关详细信息，请参阅[如何为高级 Azure Redis 缓存配置群集功能](/documentation/articles/cache-how-to-premium-clustering/)。
-	**增强的安全性和独立性**：Azure 虚拟网络 (VNET) 部署为 Azure Redis 缓存提供增强的安全性和隔离性，并提供子网、访问控制策略和进一步限制访问的其他功能。有关详细信息，请参阅[如何为高级 Azure Redis 缓存配置虚拟网络支持](/documentation/articles/cache-how-to-premium-vnet/)。
-	**配置 Redis**：在标准级别和高级级别，你都可以针对 Keyspace 通知来配置 Redis。
-	**客户端连接的最大数量**：高级级别提供的可以连接到 Redis 的客户端数量是最大的，缓存大小越大，连接数量越大。[有关详细信息，请参阅定价页](/home/features/redis-cache/pricing/)。
-	**专用 Redis 服务器核心**：高级级别的所有缓存大小都有针对 Redis 的专用核心。在基本级别/标准级别，C1 大小及以上有针对 Redis 服务器的专用核心。
-	**Redis 是单线程的**，因此与仅使用两个内核相比，使用两个以上的内核并没有额外的优势，但大型 VM 通常提供比小型 VM 更高的带宽。如果缓存服务器或客户端达到带宽限制，客户端上会出现超时。
-	**性能改进**：相对于基本级别或标准级别，高级级别的缓存部署在处理器速度更快且性能更高的硬件上。高级级别缓存的吞吐量更高，延迟更低。

<a name="cache-performance"></a>下表显示了在 Iaas VM 中使用 `redis-benchmark.exe` 针对 Azure Redis 缓存终结点测试各种大小的标准缓存和高级缓存时，所观测到的最大带宽值。请注意，这些值是没有保证的，并且我们不针对这些数字提供 SLA，但它们反映了典型的情况。你应该对自己的应用程序进行负载测试，以确定适合应用程序的缓存大小。

我们可以从此表得出以下结论。

-	如果缓存大小相同，则高级级别中的吞吐量高于标准级别。例如，就 6 GB 缓存来说，P1 的吞吐量为 140K RPS，而 C3 的吞吐量为 49K。
-	启用 Redis 群集功能时，增加群集中分片（节点）的数量会导致吞吐量线性提高。例如，如果你创建了一个包含 10 个分片的 P4 群集，则可用吞吐量为 250K *10 = 2.5 百万 RPS
-	如果增加密钥大小，则高级级别的吞吐量要高于标准级别。

| 定价层 | 大小 | 可用带宽 | 1 KB 密钥大小 |
|----------------------|--------|----------------------------|--------------------------------|
| **标准缓存大小** | &nbsp; |**每秒兆位 (Mbps)** | **每秒请求数 (RPS)** |
| C0 | 250 MB | 5 | 600 |
| C1 | 1 GB | 100 | 12200 |
| C2 | 2\.5 GB | 200 | 24000 |
| C3 | 6 GB | 400 | 49000 |
| C4 | 13 GB | 500 | 61000 |
| C5 | 26 GB | 1000 | 115000 |
| C6 | 53 GB | 2000 | 150000 |
| **高级缓存大小** | &nbsp; | &nbsp; | **每分片每秒请求数 (RPS)** |
| P1 | 6 GB | 1000 | 140000 |
| P2 | 13 GB | 2000 | 220000 |
| P3 | 26 GB | 2000 | 220000 |
| P4 | 53 GB | 4000 | 250000 |


有关下载 Redis 工具（例如 `redis-benchmark.exe`）的说明，请参阅[如何运行 Redis 命令？](#cache-commands)部分。

##<a name="cache-region"></a> 我应该将缓存放在哪个区域？

为了获得最佳性能并最大程度地降低延迟，请在缓存客户端应用程序所在的区域放置 Azure Redis 缓存。

##<a name="cache-billing"></a> Azure Redis 缓存如何计费？

[此处](/home/features/redis-cache/pricing/)提供了 Azure Redis 缓存定价。定价页列出了每小时费率。缓存按分钟计费，从创建缓存时开始，到删除缓存时为止。没有提供用于停止或暂停缓存的计费选项。

##<a name="cache-timeouts"></a> 为何会出现超时？

超时发生在用来与 Redis 通信的客户端中。大多数情况下，Redis 服务器不会超时。将某个命令发送到 Redis 服务器后，该命令将会排队，Redis 服务器最终会提取该命令并执行它。但是，客户端在此过程中可能会超时，在这种情况下，会在调用端引发异常。有关排查超时问题的详细信息，请参阅[调查 Azure Redis 缓存的 StackExchange.Redis 超时异常](https://azure.microsoft.com/blog/2015/02/10/investigating-timeout-exceptions-in-stackexchange-redis-for-azure-redis-cache/)。

##<a name="cache-monitor"></a> 如何监视缓存的运行状况和性能？

目前，Azure 中国区仅支持使用 Azure PowerShell 和 Azure CLI 管理 Redis 缓存。因此，你无法以图形方式监视缓存。

##<a name="cache-disconnect"></a> 客户端为何与缓存断开连接？

下面是缓存断开连接的一些常见原因。

-	客户端的原因
	-	已重新部署客户端应用程序。
	-	客户端应用程序执行了缩放操作。
		-	对于云服务或 Web Apps，原因可能在于自动缩放。
	-	客户端上的网络层已更改。
	-	客户端中或客户端与服务器之间的网络节点中发生暂时性错误。
	-	已达到带宽阈值限制。
	-	占用大量 CPU 的操作花费了太长时间才完成。
-	服务器端的原因
	-	在标准缓存产品上，Azure Redis 缓存服务启动了从主节点到辅助节点的故障转移。
	-	Azure 正在修补已部署缓存的实例
		-	原因可能是 Redis 服务器更新或常规 VM 维护。

##<a name="cache-configuration"></a> StackExchange.Redis 配置选项有什么作用？

StackExchange.Redis 有很多选项。本部分将介绍一些常用设置。有关 StackExchange.Redis 选项的详细详细，请参阅 [StackExchange.Redis 配置](https://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md)。

配置选项|说明|建议
---|---|---
AbortOnConnectFail|如果设置为 true，则发生网络故障后不会重新建立连接。|设置为 false，让 StackExchange.Redis 自动重新连接。
ConnectRetry|初始连接期间重试连接的次数。||
ConnectTimeout|连接操作的超时，以毫秒为单位。|

在大多数情况下，使用客户端的默认值便已足够。你可以根据工作负载微调选项。

-	重试
	-	对于 ConnectRetry 和 ConnectTimeout，一般指导原则是快速失败并重试。这取决于工作负载，以及客户端发出 Redis 命令和接收响应平均花费的时间。
	-	让 StackExchange.Redis 自动重新连接，而不是检查连接状态，然后由你自己重新连接。**避免使用 ConnectionMultiplexer.IsConnected 属性**。
	-	雪球效应 - 有时，你可能会遇到这样的问题：不断地重试解决，但问题不断累积而永远无法恢复。在这种情况下，你应该根据 Azure.cn 模式和实践小组发布的[一般重试指导原则](https://github.com/mspnp/azure-guidance/blob/master/Retry-Policies.md)中所述，考虑使用指数退让重试算法。
-	超时值
	-	根据工作负载相应地设置值。如果要存储较大值，应将超时设置为较大值。
		-	将 ABortOnConnectFail 设置为 false，让 StackExchange.Redis 为你重新连接。
-	使用应用程序的单个 ConnectionMultiplexer 实例。可以使用 LazyConnection 创建 Connection 属性返回的单个实例，如[使用 ConnectionMultiplexer 类连接到缓存](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/#working-with-caches)中所示。
-	将 `ConnectionMultiplexer.ClientName` 属性设置为应用程序实例的唯一名称以进行诊断。
-	对自定义工作负载使用多个 `ConnectionMultiplexer` 实例。
	-	如果应用程序中的负载不同，你可以遵循此模型。例如：
		-	可以使用一个多路复用器来处理大键。 
		-	可以使用一个多路复用器来处理小键。 
		-	可为连接超时设置不同的值，并为使用的每个 ConnectionMultiplexer 设置重试逻辑。
		-	在每个多路复用器上设置 `ClientName` 属性以帮助进行诊断。 
		-	这可以更好地改进每个 `ConnectionMultiplexer` 的延迟。

##<a name="threadpool"></a> 有关线程池增长的重要详细信息

CLR 线程池具有两种类型的线程 —“辅助角色”和“I/O 完成端口”（又称为 IOCP）线程。

-	对于诸如处理 `Task.Run(…)` 或 `ThreadPool.QueueUserWorkItem(…)` 方法这类事务，使用辅助角色线程。需要在后台线程上进行工作时，CLR 中的各种组件也会使用这些线程。
-	进行异步 IO（例如从网络进行读取）时，使用 IOCP 线程。  

线程池按需提供新的辅助角色线程或 I/O 完成线程（没有任何限制），直到它达到每种线程类型的“最小值”设置。默认情况下，最小线程数设置为系统上的处理器数。

一旦现有（忙碌）线程数达到“最小”线程数，线程池便会将插入新线程的速率限制为每 500 毫秒一个线程。这意味着，如果系统中出现需要 IOCP 线程的突发工作，则它会非常快速地处理该工作。但是，如果突发工作多于配置的“最小值”设置，则在处理某些工作时会出现一定的延迟，因为线程池会等待发生以下两种情况之一。

1. 一个现有线程释放，以便处理工作。
1. 在 500 毫秒内没有任何现有线程释放，因此会创建一个新线程。

基本上，这意味着忙碌线程数大于最小线程数，在应用程序处理网络流量之前可能需要付出 500 毫秒延迟。此外请务必注意，当现有线程保持空闲状态的时间超过 15 秒（基于我记得的内容）时，会清理它，并且这种增长和收缩的循环可能会重复。

如果我们考虑一个来自 StackExchange.Redis（内部版本 1.0.450 或更高版本）的示例错误消息，会看到它现在会打印线程池统计信息（请参阅下面的 IOCP 和辅助角色详细信息）。

	System.TimeoutException: Timeout performing GET MyKey, inst: 2, mgr: Inactive, 
	queue: 6, qu: 0, qs: 6, qc: 0, wr: 0, wq: 0, in: 0, ar: 0, 
	IOCP: (Busy=6,Free=994,Min=4,Max=1000), 
	WORKER: (Busy=3,Free=997,Min=4,Max=1000)

在上面的示例中，可以看到对于 IOCP 线程有 6 个忙碌线程，而系统配置为允许最少 4 个线程。在这种情况下，客户端可能会遇到两个 500 毫秒延迟，因为 6 > 4。

请注意，如果 IOCP 或辅助角色线程受到限制，则 StackExchange.Redis 可以会超时。

### 建议

考虑到此信息，我们强烈建议客户将 IOCP 和辅助角色线程的最小配置值设置为大于默认值。我们无法提供有关此值应是多少的通用指导，因为一个应用程序的合适值对于另一个应用程序会太高/低。此设置还可能会影响复杂应用程序其他部分的性能，因此每个客户需要按照其特定需求来微调此设置。开始时设置为 200 或 300 会比较好，随后可进行测试并根据需要进行调整。

如何配置此设置：

-	在 ASP.NET 中，可在 web.config 中的 `<processModel>` 配置元素下使用[“minIoThreads”配置设置][]。如果在 Azure 网站内部运行，则此设置不会通过配置选项进行公开。但是，应仍然能够通过 global.asax.cs 中的 Application\_Start 方法，以编程方式设置对此进行设置（请参阅下文）。

> **重要说明：**此配置元素中指定的值是按核心设置。例如，如果具有 4 核计算机，并且希望 minIOThreads 设置在运行时为 200，则使用 `<processModel minIoThreads="50"/>`。

-	在 ASP.NET 外部，可使用 [ThreadPool.SetMinThreads(…)](https://msdn.microsoft.com/zh-cn/library/system.threading.threadpool.setminthreads.aspx) API。

##<a name="server-gc"></a> 启用服务器 GC 以便在使用 StackExchange.Redis 时在客户端上获取更多吞吐量

启用服务器 GC 可以在使用 StackExchange.Redis 时优化客户端并提供更好的性能和吞吐量。有关服务器 GC 以及如何启用它的详细信息，请参阅以下文章。

-	[启用服务器 GC](https://msdn.microsoft.com/zh-cn/library/ms229357.aspx)
-	[垃圾回收基础](https://msdn.microsoft.com/zh-cn/library/ee787088.aspx)
-	[垃圾回收和性能](https://msdn.microsoft.com/zh-cn/library/ee851764.aspx)

##<a name="cache-redis-commands"></a> 使用常见 Redis 命令时要注意哪些问题？

-	对于某些需要较长时间才能完成的 Redis 命令，在未了解这些命令造成的影响的情况下，你不应运行这些命令。
	-	例如，不要在生产环境中运行 [KEYS](http://redis.io/commands/keys) 命令，因为它可能需要很长时间才能返回，具体时间取决于键数。Redis 是单线程服务器，每次只能处理一个命令。如果在 KEYS 后面发出了其他命令，则这些命令只会在处理完 KEYS 命令后才会得到处理。
-	键大小 - 应使用小键/值还是大键/值？ 通常这取决于方案。如果你的方案需要较大的键，则你可以调整 ConnectionTimeout 和重试值，并调整重试逻辑。从 Redis 服务器的角度来看，值越小，性能就越好。
	-	但这并不意味着你不能 Redis 中存储较大值，只是要注意以下事项。延迟将会提高。如果你有一个较大的数据集和一个较小的数据集，则可以使用多个 ConnectionMultiplexer 实例，并根据[StackExchange.Redis 配置选项有什么作用](#cache-configuration)部分中所述，为每个实例配置一组不同的超时和重试值。


##<a name="cache-ssl"></a> 何时应启用非 SSL 端口来连接 Redis？

Redis 服务器不能现成地支持 SSL，但 Azure Redis 缓存可提供此支持。如果你要连接到 Azure Redis 缓存并且客户端支持 SSL（如 StackExchange.Redis），则你应使用 SSL。

请注意，默认情况下，为新的 Azure Redis 缓存实例禁用了非 SSL 端口。如果客户端不支持 SSL，则必须启用非 SSL 端口。

`redis-cli` 等 Redis 工具对 SSL 端口不起作用，但是，你可以根据[适用于 Redis 预览版的 ASP.NET 会话状态提供程序通告](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx)中的说明，使用 `stunnel` 等实用程序安全地将这些工具连接到 SSL。

有关下载 Redis 工具的说明，请参阅[如何运行 Redis 命令？](#cache-commands)部分。

##<a name="cache-benchmarking"></a> 如何制定基准和测试缓存的性能？

-	你还可以使用所选的工具[下载和查看](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring)这些度量值。
-	可以使用 redis-benchmark.exe 对 Redis 服务器进行负载测试。
	-	确保负载测试客户端和 Redis 缓存位于同一区域。
-	使用 redis-cli.exe，并使用 INFO 命令监视缓存。
	-	如果你的负载导致出现大量内存碎片，则你应该扩展为更大的缓存大小。
-	有关下载 Redis 工具的说明，请参阅[如何运行 Redis 命令？](#cache-commands)部分。

##<a name="cache-commands"></a> 如何运行 Redis 命令？

你可以使用 [Redis 命令](http://redis.io/commands#)中列出的任何命令。可以配合多个选项来运行 Redis 命令。

-	你可以使用 Redis 命令行工具。若要使用这些选项，请执行以下步骤。
	-	下载 [Redis 命令行工具](https://github.com/MSOpenTech/redis/releases/)。
	-	使用 `redis-cli.exe` 连接到缓存。使用 -h 开关传入缓存终结点，如以下示例中所示使用 -a 传入键。
		-	`redis-cli -h <your cache name>.redis.cache.chinacloudapi.cn -a <key>`
	-	请注意，Redis 命令行工具对 SSL 端口不起作用，但是，你可以根据[适用于 Redis 预览版的 ASP.NET 会话状态提供程序通告](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx)中的说明，使用 `stunnel` 等实用程序安全地将这些工具连接到 SSL。

##<a name="cache-common-patterns"></a> 常见的缓存模式和注意事项有哪些？

-	Azure.cn 模式和实践小组制定了以下指导原则。
	-	[Azure 云应用程序设计和实施指导原则](https://github.com/mspnp/azure-guidance)
-	[使用 Azure Redis 缓存的常见缓存模式](/documentation/articles/cache-howto-common-cache-patterns/)

##<a name="cache-reference"></a> Azure Redis 缓存为何不像某些其他 Azure 服务一样提供 MSDN 类库参考？

Azure Redis 缓存基于主流开源 Redis 缓存，让你能够访问由 Azure.cn 管理的安全专用 Redis 缓存。我们针对许多编程语言提供各种 [Redis 客户端](http://redis.io/clients)。每个客户端有自身的 API，用于通过 [Redis 命令](http://redis.io/commands)调用 Redis 缓存实例。

由于客户端各不相同，因此 MSDN 上未提供统一的类参考；而是每个客户端都在维护其自身的参考文档。除了参考文档以外，Azure.com 的 [Redis 缓存文档](/documentation/services/redis-cache/)页上提供了许多教程，说明如何使用不同的语言和缓存客户端开始使用 Azure Redis 缓存。


## 哪种 Azure 缓存产品适合我？

>[AZURE.IMPORTANT] Azure.cn 建议所有新开发使用 Azure Redis 缓存。

Azure 缓存当前具有三种产品：

-	Azure Redis Cache
-	Azure 托管缓存服务
-	Azure 角色中缓存

>[AZURE.IMPORTANT]我们特此宣布将在 2016 年 11 月 30 日停用 Azure 托管缓存服务和 Azure 角色中缓存。我们建议你迁移到 Azure Redis 缓存，以便为这次停用做好准备。
><p>自该服务的正式版推出以来，Azure Redis 缓存一直是 Azure 中建议使用的缓存解决方案，而且它现在可以在所有 Azure 区域使用，包括中国和美国政府。由于这种广泛可用性，我们宣布即将停用托管缓存服务和角色中缓存服务。
><p>自 2015 年 11 月 30 日宣布之后，现有的客户最多仍可使用托管缓存服务和角色中缓存服务 12 个月，两者的服务终止日期将在 2016 年 11 月 30 日结束。在此日期之后，将关闭托管缓存服务，并且不再支持角色中缓存服务。
><p>我们将在 2016 年 2 月 1 日之后发布的第一个 Azure SDK 版本中去除对创建新角色中缓存的支持。不过，客户可以打开包含角色中缓存的现有项目。
><p>在此期间，我们建议所有现有托管缓存服务和角色中缓存服务客户迁移到 Azure Redis 缓存。Azure Redis 缓存提供更多的功能以及更高的总体价值。

### Azure Redis Cache
Azure Redis 缓存已正式发布，最大大小为 53 GB，且其可用性 SLA 为 99.9%。全新[高级层](/documentation/articles/cache-premium-tier-intro/)提供的最大大小为 530 GB，且支持群集、VNET 和持久性，并附带 99.9% SLA。

Azure Redis 缓存使客户能够使用 Azure.cn 管理的安全专用 Redis 缓存。有了此产品，你可以利用 Redis 提供的丰富功能集和生态系统，并可以从 Azure.cn 获得可靠的托管和监控。

与仅处理键/值对的传统缓存不同，Redis 因其高性能的数据类型而受欢迎。Redis 还支持对这些类型运行原子操作，如在字符串后面追加内容；递增哈希中的值；推送到列表；计算交集、并集和差集，或者获取排序集中排名最高的成员。其他功能包括支持事务、发布/订阅、Lua 脚本、具有有限生存时间的键和配置设置，使 Redis 在行为上更类似于传统缓存。

Redis 取得成功的另一个重要方面是围绕它构建了健康而充满活力的开放源生态系统。这反映在可通过多种语言使用各种不同的 Redis 客户端。这样一来，在 Azure 内部生成的几乎任何工作负荷都可以使用此缓存。

有关如何开始使用 Azure Redis 缓存的详细信息，请参阅[如何使用 Azure Redis 缓存](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)和 [Azure Redis 缓存文档](/documentation/services/redis-cache/)。

### 托管缓存服务
托管缓存服务已安排在 2016 年 11 月 30 日停用。

### 角色中缓存
角色中缓存已安排在 2016 年 11 月 30 日停用。

[“minIoThreads”配置设置]: https://msdn.microsoft.com/zh-cn/library/vstudio/7w2sway1(v=vs.100).aspx

<!---HONumber=Mooncake_0321_2016-->