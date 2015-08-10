<properties 
	pageTitle="Azure Redis 缓存常见问题" 
	description="了解常见问题的答案，以及有关 Azure Redis 缓存的模式和最佳实践" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>

<tags ms.service="cache" ms.date="05/01/2015" wacn.date="06/16/2015"/>

# Azure Redis 缓存常见问题

了解常见问题的答案，以及有关 Azure Redis 缓存的模式和最佳实践。


## <a name="cache-size"></a>我应使用哪种 Redis 缓存产品和大小？

每款 Azure Redis 缓存产品在**大小**、**带宽**、**高可能性**和 **SLA** 方面提供不同的级别。

-	基本 SKU - 单个节点，不提供复制或 SLA，缓存大小为 250 MB 到 53 GB。
-	标准 SKU - 主要/辅助节点，提供自动复制，99.9% 的 SLA，缓存大小为 250 MB 到 53 GB。

如果你需要高可用性，请选择提供 99.9% SLA 的标准缓存产品。对于开发和原型设计或者不需要 SLA 的情况，基本产品可能合适。

缓存大小和带宽粗略对应于托管缓存的虚拟机的大小和带宽。基本和标准服务的 250 MB 缓存在特小型 (A0) 虚拟机上使用共享内核进行托管，而其他大小的缓存则使用专用内核进行托管。1 GB 缓存在小型 (A1) 虚拟机上托管，它有 1 个专用虚拟内核，用于为操作系统和 Redis 缓存提供服务。较大的缓存则在大型 VM 实例上托管，具有多个专用虚拟内核。

如果你的缓存具有高吞吐量，请选择 1 GB 或更大空间的服务，让你的缓存使用专用内核运行。1 GB 缓存在 1 个内核的虚拟机上托管。此内核用于为操作系统和缓存提供服务。大于 1 GB 的缓存在具有多个内核的虚拟机上运行，Redis 缓存使用不与操作系统共享的专用内核。

**Redis 是单线程的**，因此与仅使用两个内核相比，使用两个以上的内核并没有额外的优势，但**大型 VM 通常提供比小型 VM 更多的带宽**。如果缓存服务器或客户端达到带宽限制，客户端上会出现超时。

下表显示了在 Iaas VM 中使用 `redis-benchmark.exe` 针对 Azure Redis 缓存终结点测试各种大小的 Azure Redis 缓存时，所观测到的最大带宽值。请注意，这些值是没有保证的，并且我们不针对这些数字提供 SLA，但它们反映了典型的情况。你应该对自己的应用程序进行负载测试，以确定适合应用程序的缓存大小。

<table>
  <tr>
    <th>缓存名称</th>
    <th>缓存大小</th>
    <th>Get/秒（获取 1 KB 值的简单 GET 调用次数）</th>
    <th>带宽（Mb/秒）</th>
  </tr>
  <tr>
    <td>C0</td>
    <td>250 MB</td>
    <td>610</td>
    <td>5</td>
  </tr>
  <tr>
    <td>C1</td>
    <td>1 GB</td>
    <td>12,200</td>
    <td>100</td>
  </tr>
  <tr>
    <td>C2</td>
    <td>2.5 GB</td>
    <td>24,300</td>
    <td>200</td>
  </tr>
  <tr>
    <td>C3</td>
    <td>6 GB</td>
    <td>48,875</td>
    <td>400</td>
  </tr>
  <tr>
    <td>C4</td>
    <td>13 GB</td>
    <td>61,350</td>
    <td>500</td>
  </tr>
  <tr>
    <td>C5</td>
    <td>26 GB</td>
    <td>112,275</td>
    <td>1000</td>
  </tr>
  <tr>
    <td>C6</td>
    <td>53 GB</td>
    <td>153,219</td>
    <td>1000+</td>
  </tr>
</table>

有关下载 Redis 工具（例如 `redis-benchmark.exe`）的说明，请参阅[如何运行 Redis 命令？](#cache-commands)部分。


## <a name="cache-region"></a>我应该将缓存放在哪个区域？

为了获得最佳性能并最大程度地降低延迟，请在缓存客户端应用程序所在的区域放置 Azure Redis 缓存。


## <a name="cache-timeouts"></a>为何会出现超时？

超时发生在用来与 Redis 通信的客户端中。大多数情况下，Redis 服务器不会超时。将某个命令发送到 Redis 服务器后，该命令将会排队，Redis 服务器最终会提取该命令并执行它。但是，客户端在此过程中可能会超时，在这种情况下，会在调用端引发异常。有关排查超时问题的详细信息，请参阅[调查 Azure Redis 缓存的 StackExchange.Redis 超时异常](http://azure.microsoft.com/blog/2015/02/10/investigating-timeout-exceptions-in-stackexchange-redis-for-azure-redis-cache/)。


## <a name="cache-monitor"></a>如何监视缓存的运行状况和性能？

可以在 [Azure 门户](https://manage.windowsazure.cn)中监视 Windows Azure Redis 缓存实例。你可以查看度量值、将度量值图表固定到启动面板、自定义监视图表的日期和时间范围、在图表中添加和删除度量值，以及设置符合特定条件时发出的警报。借助这些工具，你可以监视 Azure Redis 缓存实例的运行状况，以及管理缓存应用程序。有关监视缓存的详细信息，请参阅[监视 Azure Redis 缓存](https://msdn.microsoft.com/zh-cn/library/azure/dn763945.aspx)。


## <a name="cache-disconnect"></a>客户端为何与缓存断开连接？

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


## <a name="cache-configuration"></a>StackExchange.Redis 配置选项有什么作用？

StackExchange.Redis 有很多选项。本部分将介绍一些常用设置。有关 StackExchange.Redis 选项的详细详细，请参阅 [StackExchange.Redis 配置](https://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md)。

<table>
  <tr>
    <th>配置选项</th>
    <th>说明</th>
    <th>建议</th>
  </tr>
  <tr>
    <td>AbortOnConnectFail</td>
    <td>如果设置为 true，则发生网络故障后不会重新建立连接。</td>
    <td>设置为 false，让 StackExchange.Redis 自动重新连接。</td>
  </tr>
  <tr>
    <td>ConnectRetry</td>
    <td>初始连接期间重试连接的次数。</td>
    <td></td>
  </tr>
  <tr>
    <td>ConnectTimeout</td>
    <td>连接操作的超时，以毫秒为单位。</td>
    <td></td>
  </tr>
</table>

在大多数情况下，使用客户端的默认值便已足够。你可以根据工作负载微调选项。

-	重试
	-	对于 ConnectRetry 和 ConnectTimeout，一般指导原则是快速失败并重试。这取决于工作负载，以及客户端发出 Redis 命令和接收响应平均花费的时间。
	-	让 StackExchange.Redis 自动重新连接，而不是检查连接状态，然后由你自己重新连接。**避免使用 ConnectionMultiplexer.IsConnected 属性**。
	-	雪球效应 - 有时，你可能会遇到这样的问题：不断地重试解决，但问题不断累积而永远无法恢复。在这种情况下，你应该根据 Microsoft 模式和实践小组发布的[一般重试指导原则](https://github.com/mspnp/azure-guidance/blob/master/Retry-General.md)中所述，考虑使用指数退让重试算法。
-	超时值
	-	根据工作负载相应地设置值。如果要存储较大值，应将超时设置为较大值。
		-	将 ABortOnConnectFail 设置为 false，让 StackExchange.Redis 为你重新连接。
-	使用应用程序的单个 ConnectionMultiplexer 实例。可以使用 LazyConnection 创建 Connection 属性返回的单个实例，如[使用 ConnectionMultiplexer 类连接到缓存](https://msdn.microsoft.com/zh-cn/library/azure/dn690521.aspx#Connect)中所示。
-	将 `ConnectionMultiplexer.ClientName` 属性设置为应用程序实例的唯一名称以进行诊断。
-	对自定义工作负载使用多个 `ConnectionMultiplexer` 实例。
	-	如果应用程序中的负载不同，你可以遵循此模型。例如：
		-	可以使用一个多路复用器来处理大键。 
		-	可以使用一个多路复用器来处理小键。 
		-	可为连接超时设置不同的值，并为使用的每个 ConnectionMultiplexer 设置重试逻辑。
		-	在每个多路复用器上设置 `ClientName` 属性以帮助进行诊断。 
		-	这可以更好地改进每个 `ConnectionMultiplexer` 的延迟。


## <a name="cache-redis-commands"></a>使用常见 Redis 命令时要注意哪些问题？

-	对于某些需要较长时间才能完成的 Redis 命令，在未了解这些命令造成的影响的情况下，你不应运行这些命令。
	-	例如，不要在生产环境中运行 [KEYS](http://redis.io/commands/keys) 命令，因为它可能需要很长时间才能返回，具体时间取决于键数。Redis 是单线程服务器，每次只能处理一个命令。如果在 KEYS 后面发出了其他命令，则这些命令只会在处理完 KEYS 命令后才会得到处理。
-	键大小 - 应使用小键/值还是大键/值？ 通常这取决于方案。如果你的方案需要较大的键，则你可以调整 ConnectionTimeout 和重试值，并调整重试逻辑。从 Redis 服务器的角度来看，值越小，性能就越好。
	-	但这并不意味着你不能 Redis 中存储较大值，只是要注意以下事项。延迟将会提高。如果你有一个较大的数据集和一个较小的数据集，则可以使用多个 ConnectionMultiplexer 实例，并根据[StackExchange.Redis 配置选项有什么作用](#cache-configuration)部分中所述，为每个实例配置一组不同的超时和重试值。



## <a name="cache-ssl"></a>何时应启用非 SSL 端口来连接 Redis？

Redis 服务器不能现成地支持 SSL，但 Azure Redis 缓存可提供此支持。如果你要连接到 Azure Redis 缓存并且客户端支持 SSL（如 StackExchange.Redis），则你应使用 SSL。

请注意，默认情况下，为新的 Azure Redis 缓存实例禁用了非 SSL 端口。如果客户端不支持 SSL，则你必须根据[在 Azure Redis 缓存中配置缓存](https://msdn.microsoft.com/zh-cn/library/azure/dn793612.aspx)一文中的[访问端口](https://msdn.microsoft.com/zh-cn/library/azure/dn793612.aspx#AccessPorts)部分中的说明启用非 SSL 端口。

`redis-cli` 等 Redis 工具对 SSL 端口不起作用，但是，你可以根据[适用于 Redis 预览版的 ASP.NET 会话状态提供程序通告](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx)中的说明，使用 `stunnel` 等实用程序安全地将这些工具连接到 SSL。

有关下载 Redis 工具的说明，请参阅[如何运行 Redis 命令？](#cache-commands)部分。


## <a name="cache-benchmarking"></a>如何制定基准和测试缓存的性能？

-	[启用缓存诊断](https://msdn.microsoft.com/zh-cn/library/azure/dn763945.aspx#EnableDiagnostics)，以便可以[监视](https://msdn.microsoft.com/zh-cn/library/azure/dn763945.aspx)缓存的运行状况。可以在门户中查看度量值，也可以使用所选的工具[下载和查看](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring)这些度量值。
-	可以使用 redis-benchmark.exe 对 Redis 服务器进行负载测试。
	-	确保负载测试客户端和 Redis 缓存位于同一区域。
-	使用 redis-cli.exe，并使用 INFO 命令监视缓存。
	-	如果你的负载导致出现大量内存碎片，则你应该扩展为更大的缓存大小。
-	有关下载 Redis 工具的说明，请参阅[如何运行 Redis 命令？](#cache-commands)部分。


## <a name="cache-commands"></a>如何运行 Redis 命令？

你可以使用 [Redis 命令](http://redis.io/commands#)中列出的任何命令。若要运行这些命令，可以使用以下工具。

-	下载 [Redis 命令行工具](https://github.com/MSOpenTech/redis/releases/download/win-2.8.19.1/redis-2.8.19.zip)。
-	使用 `redis-cli.exe` 连接到缓存。使用 -h 开关传入缓存终结点，如以下示例中所示使用 -a 传入键。
	-	`redis-cli -h <your cache name>.redis.cache.chinacloudapi.cn -a <key>`
-	请注意，Redis 命令行工具对 SSL 端口不起作用，但是，你可以根据[适用于 Redis 预览版的 ASP.NET 会话状态提供程序通告](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx)中的说明，使用 `stunnel` 等实用程序安全地将这些工具连接到 SSL。


## <a name="cache-common-patterns"></a>常见的缓存模式和注意事项有哪些？

-	Microsoft 模式和实践小组制定了以下指导原则。
	-	[缓存指导原则](https://github.com/mspnp/azure-guidance/blob/master/Caching.md)。
	-	[Azure 云应用程序设计和实施指导原则](https://github.com/mspnp/azure-guidance)
-	[使用 Azure Redis 缓存的常见缓存模式](/documentation/articles/cache-howto-common-cache-patterns)


## <a name="cache-reference"></a>Azure Redis 缓存为何不像某些其他 Azure 服务一样提供 MSDN 类库参考？

Windows Azure Redis 缓存基于主流开源 Redis 缓存，让你能够访问由世纪互联管理的安全专用 Redis 缓存。我们针对许多编程语言提供各种 [Redis 客户端](http://redis.io/clients)。每个客户端有自身的 API，用于通过 [Redis 命令](http://redis.io/commands)调用 Redis 缓存实例。

由于客户端各不相同，因此 MSDN 上未提供统一的类参考；而是每个客户端都在维护其自身的参考文档。除了参考文档以外，Azure.com 的 [Redis 缓存文档](/documentation/services/redis-cache/)页上提供了许多教程，说明如何使用不同的语言和缓存客户端开始使用 Azure Redis 缓存。

<!---HONumber=60-->