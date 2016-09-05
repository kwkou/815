<properties 
	pageTitle="如何排查 Azure Redis 缓存问题 | Azure" 
	description="了解如何解决 Azure Redis 缓存的常见问题。" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="douge" 
	editor=""/>

<tags
	ms.service="cache"
	ms.date="08/09/2016"
	wacn.date="09/05/2016"/>

# 如何排查 Azure Redis 缓存问题

本文提供的指南适用于排查以下类别的 Azure Redis 缓存问题。

-	[客户端故障排除](#client-side-troubleshooting) - 此部分指导你如何确定和解决连接到 Azure Redis 缓存的应用程序引发的问题。
-	[服务器端故障排除](#server-side-troubleshooting) - 此部分指导你如何确定和解决 Azure Redis 缓存服务器端引发的问题。
-	[StackExchange.Redis 超时异常](#stackexchangeredis-timeout-exceptions) - 此部分说明如何在使用 StackExchange.Redis 客户端时排查问题。


>[AZURE.NOTE] 本指南中的多个故障排除步骤包括了运行 Redis 命令和监视各种性能指标的说明。如需更多信息和说明，请参阅[其他信息](#additional-information)部分的文章。

## <a name="client-side-troubleshooting"></a> 客户端故障排除


此部分讨论如何排查因客户端应用程序出现状况而发生的问题。

-	[客户端内存压力](#memory-pressure-on-the-client)
-	[流量激增](#burst-of-traffic)
-	[客户端 CPU 使用率过高](#high-client-cpu-usage)
-	[超出客户端带宽](#client-side-bandwidth-exceeded)
-	[请求/响应大小过大](#large-requestresponse-size)
-	[我在 Redis 中的数据发生了什么情况？](#what-happened-to-my-data-in-redis)

### <a name="memory-pressure-on-the-client"></a> 客户端内存压力

#### 问题

客户端计算机上出现的内存压力会导致各种性能问题，这些问题可能会延迟对 Redis 实例所发送的无延迟数据的处理。出现内存压力时，系统通常必须将数据从物理内存以分页方式转移到磁盘上的虚拟内存。此 *分页错误* 导致系统的性能显著下降。

#### 度量 

1.	监视计算机上的内存使用情况，确保所用内存不超出可用内存。
2.	监视 `Page Faults/Sec` 性能计数器。大多数系统在正常运行时也会出现某些分页错误，因此需注意此分页错误性能计数器中出现的峰值，与这些峰值对应的是超时。

#### 解决方法

升级客户端，使客户端 VM 大小更大，内存更多，或者对内存使用模式进行深入分析以减少内存消耗。


### <a name="burst-of-traffic"></a> 流量激增

#### 问题

流量激增时，如果 `ThreadPool` 设置不佳，则可能导致对 Redis 服务器已发送但尚未在客户端上使用的数据的处理出现延迟。

#### 度量 

使用[此类](https://github.com/JonCole/SampleCode/blob/master/ThreadPoolMonitor/ThreadPoolLogger.cs)代码监视 `ThreadPool` 统计信息随时间变化的情况。也可查看 StackExchange.Redis 发出的 `TimeoutException` 消息。下面是一个示例：

    System.TimeoutException: Timeout performing EVAL, inst: 8, mgr: Inactive, queue: 0, qu: 0, qs: 0, qc: 0, wr: 0, wq: 0, in: 64221, ar: 0, 
    IOCP: (Busy=6,Free=999,Min=2,Max=1000), WORKER: (Busy=7,Free=8184,Min=2,Max=8191)

在上面的消息中，有几个需要注意的问题：

 1. 请注意，在 `IOCP` 部分和 `WORKER` 部分，`Busy` 值大于 `Min` 值。这意味着你的 `ThreadPool` 设置需要调整。
 2. 也可参看 `in: 64221`。这表示已在内核套接字层收到了 64211 字节，但这些字节尚未被应用程序（例如 StackExchange.Redis）读取。这通常意味着，你的应用程序从网络读取数据的速度没有服务器向你发送数据的速度快。

#### 解决方法

配置 [ThreadPool 设置](https://gist.github.com/JonCole/e65411214030f0d823cb)，确保线程池在流量激增的情况下快速进行扩展。


### <a name="high-client-cpu-usage"></a> 客户端 CPU 使用率过高

#### 问题

客户端 CPU 使用率过高表示系统跟不上所要求执行的工作的进度。这意味着，客户端可能无法及时处理 Redis 发出的响应，即使 Redis 发送响应的速度很快。

#### 度量

通过 Azure 门户或关联的性能计数器监视系统范围的 CPU 使用率。请注意，不要监视 *进程* CPU，因为即使单个进程的 CPU 使用率低，系统的总体 CPU 使用率也可能高。注意与超时相对应的 CPU 使用率峰值。由于 CPU 使用率高，你可能还会在 `TimeoutException` 错误消息中看到 `in: XXX` 值高，如[流量激增](#burst-of-traffic)部分所述。

>[AZURE.NOTE] StackExchange.Redis 1.1.603 及更高版本在 `TimeoutException` 错误消息中包括了 `local-cpu` 指标。确保使用最新版本的 [StackExchange.Redis NuGet 包](https://www.nuget.org/packages/StackExchange.Redis/)。我们会不断对代码中的 Bug 进行修正，以便更好地应对超时情况。因此，请务必使用最新的版本。

#### 解决方法

通过升级提高 VM 大小和 CPU 容量，或者调查清楚导致 CPU 峰值的原因。



### <a name="client-side-bandwidth-exceeded"></a> 超出客户端带宽

#### 问题

不同大小的客户端计算机对于可提供的网络带宽存在不同的限制。如果客户端超出可用带宽，则客户端的数据处理速度将赶不上服务器的数据发送速度。这可能导致超时。

#### 度量

使用[此类](https://github.com/JonCole/SampleCode/blob/master/BandWidthMonitor/BandwidthLogger.cs)代码监视带宽使用随时间变化的情况。请注意，在对权限有限制的某些环境（例如 Azure 网站）中，此代码可能无法成功运行。

#### 解决方法 

增加客户端 VM 大小或减少网络带宽消耗。


### <a name="large-requestresponse-size"></a> 请求/响应大小过大

#### 问题

请求/响应过大可能导致超时。例如，假设你在客户端上配置的超时值为 1 秒。你的应用程序同时请求两个密钥（例如“A”和“B”）（使用相同的物理网络连接）。大多数客户端支持对请求进行“管道操作”，使得请求“A”和“B”在线路上是依次发送到服务器的，不需等待响应。服务器会按相同顺序将响应发送回来。如果响应“A”过大，则可能会消耗掉后续请求的大部分超时时间。

以下示例演示了这种情况。在这种情况下，请求“A”和“B”的发送速度很快，服务器开始发送响应“A”和“B”的速度也很快，但由于数据传输需要时间，“B”被其他请求挡在了后面，在服务器响应速度很快的情况下也会超时。

    |-------- 1 Second Timeout (A)----------|
    |-Request A-|
         |-------- 1 Second Timeout (B) ----------|
         |-Request B-|
                |- Read Response A --------|
                                           |- Read Response B-| (**TIMEOUT**)



#### 度量

这种情况很难进行度量。基本说来，这种情况下你必须对客户端代码进行检测，以便跟踪大型请求和响应。

#### 解决方法

1.	Redis 适用于大量的小型值，而不适用于少量的大型值。首选解决方案是将数据分解成较小的相关值。请参阅 [What is the ideal value size range for redis? Is 100KB too large?](https://groups.google.com/forum/#!searchin/redis-db/size/redis-db/n7aa2A4DZDs/3OeEPHSQBAAJ)（Redis 的理想值大小范围是多少？100KB 是否过大？）这个帖子，以详细了解为何我们建议你使用较小值。
2.	增加 VM 的大小（针对客户端和 Redis 缓存服务器），以便提高带宽能力，减少大型响应的数据传输时间。请注意，不能只提高服务器的带宽，也不能只提高客户端的带宽。测量带宽使用情况，将其与当前 VM 大小所对应的带宽能力进行比较。
3.	增加所使用的 `ConnectionMultiplexer` 对象数，以及通过不同连接进行的轮询请求数。


### <a name="what-happened-to-my-data-in-redis"></a> 我在 Redis 中的数据发生了什么情况？

#### 问题

我本以为某些数据会出现在我的 Azure Redis 缓存实例中，但这些数据似乎并不在那里。

##### 解决方法

请参阅 [What happened to my data in Redis?](https://gist.github.com/JonCole/b6354d92a2d51c141490f10142884ea4#file-whathappenedtomydatainredis-md)（我在 Redis 中的数据发生了什么情况？），了解可能的原因和解决方法。


## <a name="server-side-troubleshooting"></a> 服务器端故障排除

此部分讨论如何排查因缓存服务器出现状况而发生的问题。

-	[服务器上的内存压力](#memory-pressure-on-the-server)
-	[CPU 使用率/服务器负载过高](#high-cpu-usage-server-load)
-	[超出服务器端带宽](#server-side-bandwidth-exceeded)

### <a name="memory-pressure-on-the-server"></a> 服务器上的内存压力

#### 问题

服务器端的内存压力会导致各种性能问题，从而延缓对请求的处理。出现内存压力时，系统通常必须将数据从物理内存以分页方式转移到磁盘上的虚拟内存。此 *分页错误* 导致系统的性能显著下降。这种内存压力可能有多个原因：

1.	缓存中填满了数据。
2.	Redis 出现大量内存碎片 - 大多数情况下是因为存储了大型对象（Redis 适用于小型对象 - 如需详细信息，请参阅 [What is the ideal value size range for redis? Is 100KB too large?](https://groups.google.com/forum/#!searchin/redis-db/size/redis-db/n7aa2A4DZDs/3OeEPHSQBAAJ)（Redis 的理想值大小范围是多少？100KB 是否过大？）这篇帖子）。

#### 度量

Redis 公开了两个指标，你可以通过这两个指标来确定此问题。第一个是 `used_memory`，另一个是 `used_memory_rss`。可以在 Azure 门户中或者通过 [Redis INFO](http://redis.io/commands/info) 命令获取[这些指标](/documentation/articles/cache-how-to-monitor/#available-metrics-and-reporting-intervals)。

#### 解决方法

你可以通过多个可能的更改来确保内存的正常使用：

1. [配置内存策略](/documentation/articles/cache-configure/#maxmemory-policy-and-maxmemory-reserved)，对密钥设置过期时间。请注意，如果存在内存碎片，则这样设置还不够。
2. [配置 maxmemory-reserved 值](/documentation/articles/cache-configure/#maxmemory-policy-and-maxmemory-reserved)，该值应足够大，可以抵消内存碎片造成的影响。
3. 将大型缓存对象分解成小型相关对象。
4. [扩展](/documentation/articles/cache-how-to-scale/)到更大型的缓存大小。
5. 如果你使用的是[启用了 Redis 群集的高级缓存](/documentation/articles/cache-how-to-premium-clustering/)，则可[增加分片数](/documentation/articles/cache-how-to-premium-clustering/#change-the-cluster-size-on-a-running-premium-cache)。

### <a name="high-cpu-usage-server-load"></a> CPU 使用率/服务器负载过高

#### 问题

CPU 使用率高可能意味着，客户端可能无法及时处理 Redis 发出的响应，即使 Redis 发送响应的速度很快。

#### 度量

通过 Azure 门户或关联的性能计数器监视系统范围的 CPU 使用率。请注意，不要监视 *进程* CPU，因为即使单个进程的 CPU 使用率低，系统的总体 CPU 使用率也可能高。注意与超时相对应的 CPU 使用率峰值。

#### 解决方法

通过[扩展](/documentation/articles/cache-how-to-scale/)提高缓存层大小和 CPU 容量，或者调查清楚导致 CPU 峰值的原因。

### <a name="server-side-bandwidth-exceeded"></a> 超出服务器端带宽

#### 问题

不同大小的缓存实例对于可提供的网络带宽存在不同的限制。如果服务器超出可用带宽，则数据将无法快速发送到客户端。这可能导致超时。

#### 度量

你可以监视 `Cache Read` 指标，该指标是指在指定的报告时间间隔内从缓存读取的数据量，以每秒兆字节数（MB/秒）为单位。此值对应于该缓存使用的网络带宽。如果你要针对服务器端网络带宽限制设置警报，可使用此 `Cache Read` 计数器来创建这些警报。将你的读数与[此表](/documentation/articles/cache-faq/#cache-performance)中的值进行比较，了解各种缓存定价层和大小所遵循的带宽限制。

#### 解决方法

如果你一直很接近定价层和缓存大小所遵循的最大带宽，则可考虑[扩展](/documentation/articles/cache-how-to-scale/)到网络带宽更高的定价层或大小（使用[此表](/documentation/articles/cache-faq/#cache-performance)中的值作为指导）。


## <a name="stackexchangeredis-timeout-exceptions"></a> StackExchange.Redis 超时异常

StackExchange.Redis 使用名为 `synctimeout` 的配置设置进行同步操作，该设置的默认值为 1000 毫秒。如果同步调用未在规定时间内完成，StackExchange.Redis 客户端会引发类似于以下示例的超时错误。

	System.TimeoutException: Timeout performing MGET 2728cc84-58ae-406b-8ec8-3f962419f641, inst: 1,mgr: Inactive, queue: 73, qu=6, qs=67, qc=0, wr=1/1, in=0/0 IOCP: (Busy=6, Free=999, Min=2,Max=1000), WORKER (Busy=7,Free=8184,Min=2,Max=8191)


此错误消息中包含的指标可以为你指出问题的原因和可能的解决方法。下表包含有关错误消息指标的详细信息。

| 错误消息指标 | 详细信息 |
|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| inst | 在上一个时间片中：发出了 0 个命令 |
| mgr | 套接字管理器正在执行 `socket.select`，也就是说，它在请求 OS 指示一个需要执行某些操作的套接字；大致说来：读取器并没有主动从网络读取内容，因为它认为不需执行任何操作 |
| 队列 | 总共有 73 个正在进行的操作 |
| qu | 正在进行的操作中，有 6 个操作位于未发送队列中，尚未写入到出站网络 |
| qs | 正在进行的操作中，有 67 个操作已发送给服务器，但尚未得到响应。响应可能为 `Not yet sent by the server` 或 `sent by the server but not yet processed by the client.` |
| qc | 正在进行的操作中，有 0 个操作已经有回复，但尚未标记为完成，因为正在完成循环中进行等待 |
| wr | 存在活动的写入器（这意味着系统不会忽略这 6 个尚未发送的请求）字节/活动写入器 |
| 位于 | 没有活动的读取器，NIC 字节/活动读取器上没有可供读取的字节 |


### 调查步骤

1. 请确保你在使用 StackExchange.Redis 客户端时，按照以下模式进行连接，这是一种最佳做法。


	    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
	    {
	        return ConnectionMultiplexer.Connect("cachename.redis.cache.chinacloudapi.cn,abortConnect=false,ssl=true,password=...");
	
	    });
	
	    public static ConnectionMultiplexer Connection
	    {
	        get
	        {
	            return lazyConnection.Value;
	        }
	    }


    有关详细信息，请参阅 [Connect to the cache using StackExchange.Redis](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/#connect-to-the-cache)（使用 StackExchange.Redis 连接到缓存）。

2. 请确保你的 Azure Redis 缓存和客户端应用程序位于 Azure 中的同一区域。例如，如果你的缓存位于中国东部但客户端位于中国北部，而且请求没有在 `synctimeout` 时间间隔内完成，则可能会出现超时；或者，如果你是从本地开发计算机进行调试，则也可能会出现超时。

    强烈建议你将缓存和客户端置于同一 Azure 区域。如果你的方案包括跨区域调用，则应将 `synctimeout` 时间间隔设置为比默认的 1000 毫秒时间间隔更高的值，方法是在连接字符串中增加一个 `synctimeout` 属性。以下示例显示的是 StackExchange.Redis 缓存连接字符串代码段，其中的 `synctimeout` 为 2000 毫秒。

        synctimeout=2000,cachename.redis.cache.chinacloudapi.cn,abortConnect=false,ssl=true,password=...

3. 确保使用最新版本的 [StackExchange.Redis NuGet 包](https://www.nuget.org/packages/StackExchange.Redis/)。我们会不断对代码中的 Bug 进行修正，以便更好地应对超时情况。因此，请务必使用最新的版本。

4. 如果请求受服务器或客户端上的带宽限制的约束，则需要更长时间才能完成，因此会导致超时。若要了解超时是否是服务器上的网络带宽造成的，请参阅[超出服务器端带宽](#server-side-bandwidth-exceeded)。若要了解超时是否是客户端网络带宽造成的，请参阅[超出客户端带宽](#client-side-bandwidth-exceeded)。

6. 你的操作是否占用了服务器或客户端上的大量 CPU？
	-	看看你的操作是否占用了客户端上的大量 CPU，如果是的话，则可能会导致请求无法在 `synctimeout` 时间间隔内得到处理，从而导致超时。改用更大型客户端或者将负载分散也许有助于控制这种情况。
	-	看看你的操作是否占用了服务器上的大量 CPU，方法是监视 `CPU` [缓存性能指标](/documentation/articles/cache-how-to-monitor/#available-metrics-and-reporting-intervals)。如果请求传入时 Redis 处于 CPU 被大量占用的情况，则可能会导致这些请求超时。为了解决此问题，你可以将负载分散到高级缓存的多个分片中，也可以升级缓存大小或定价层。有关详细信息，请参阅 [Server Side Bandwidth Exceeded](#server-side-bandwidth-exceeded)（超出服务器端带宽）。

7. 是否存在需要在服务器上进行长时间处理的命令？ 长时间运行的命令需要在 Redis 服务器上进行长时间的处理，可能会导致超时。下面是长时间运行的命令的一些示例：密钥数量很大的 `mget`、`keys *` 或编写质量差的 lua 脚本。可以使用 redis-cli 客户端或 [Redis 控制台](/documentation/articles/cache-configure/#redis-console)连接到 Azure Redis 缓存实例，然后运行 [SlowLog](http://redis.io/commands/slowlog) 命令，看是否有请求的处理时间超出正常。Redis 服务器和 StackExchange.Redis 适合处理多个小型请求，而不适合处理寥寥数个大型请求。将数据拆分成更小的块可能会解决问题。

    若要了解如何使用 redis-cli 和 stunnel 连接到 Azure Redis 缓存 SSL 终结点，请参阅博客文章：[Announcing ASP.NET Session State Provider for Redis Preview Release](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx)（宣布推出适用于 Redis 的 ASP.NET 会话状态提供程序预览版）。有关详细信息，请参阅 [SlowLog](http://redis.io/commands/slowlog)。

8. Redis 服务器负载过高可能会导致超时。可以通过监视 `Redis Server Load` [缓存性能指标](/documentation/articles/cache-how-to-monitor/#available-metrics-and-reporting-intervals)来监视服务器负载。服务器负载值为 100（最大值）表示 Redis 服务器正忙于处理请求，没有空闲时间。若要查看某些请求是否占用了服务器的全部处理能力，请按上一段中的说明运行 SlowLog 命令。有关详细信息，请参阅 [High CPU usage / Server Load](#high-cpu-usage-server-load)（CPU 使用率/服务器负载过高）。

9. 客户端上是否存在其他可能导致网络故障的事件？ 查看客户端（Web 角色、辅助角色或 Iaas VM）上是否存在如下事件：向上或向下缩放客户端实例数目、部署新版客户端或启用自动缩放。在我们的测试中，我们发现进行自动缩放或向上/向下缩放可能导致出站网络连接失去连接数秒钟的时间。StackExchange.Redis 代码可以灵活应对此类事件，并且会重新连接。在这个重新连接的时间内，队列中的请求可能会超时。

10. 在向 Redis 缓存发出数个小型请求之前，是否存在导致超时的大型请求？ 错误消息中的参数 `qs` 会告诉你，多少请求从客户端发送到了服务器，但尚未进行响应处理。此值可能会持续增加，因为 StackExchange.Redis 使用单个 TCP 连接，一次只能读取一个响应。即使第一个操作超时，也不会阻止数据通过服务器进行传输，在此操作完成之前，系统会阻止其他请求，导致超时。降低超时概率的一种解决方案是确保缓存对于工作负荷来说足够大，并将大的值拆分成较小的块。另一种可能的解决方案是使用客户端中的 `ConnectionMultiplexer` 对象池，在发送新请求时选择负载最小的 `ConnectionMultiplexer`。这样可以防止因为某个请求超时而导致其他请求也超时。

11. 如果你使用的是 `RedisSessionStateprovider`，请确保你已经正确设置了重试超时。`retrytimeoutInMilliseconds` 应高于 `operationTimeoutinMilliseonds`，否则不会进行重试。在下面的示例中，`retrytimeoutInMilliseconds` 设置为 3000。有关详细信息，请参阅 [Azure Redis 缓存的 ASP.NET 会话状态提供程序](/documentation/articles/cache-aspnet-session-state-provider/)和 [How to use the configuration parameters of Session State Provider and Output Cache Provider](https://github.com/Azure/aspnet-redis-providers/wiki/Configuration)（如何使用会话状态提供程序和输出缓存提供程序的配置参数）。


		<add
		name="AFRedisCacheSessionStateProvider"
		type="Microsoft.Web.Redis.RedisSessionStateProvider"
		host="enbwcache.redis.cache.chinacloudapi.cn"
		port="6380"
		accessKey="…"
		ssl="true"
		databaseId="0"
		applicationName="AFRedisCacheSessionState"
		connectionTimeoutInMilliseconds = "5000"
		operationTimeoutInMilliseconds = "1000"
		retryTimeoutInMilliseconds="3000" />


12. 通过[监视](/documentation/articles/cache-how-to-monitor/#available-metrics-and-reporting-intervals) `Used Memory RSS` 和 `Used Memory`，了解 Azure Redis 缓存服务器上的内存使用情况。如果实施了逐出策略，则当 `Used_Memory` 达到缓存大小时，Redis 就会开始逐出密钥。理想情况下，`Used memory` 应只稍高于 `Used Memory RSS`。差异过大意味着会出现内存碎片（内部或外部）。如果 `Used Memory RSS` 小于 `Used Memory`，则意味着部分缓存内存已被操作系统更换。如果发生这种情况，则会出现明显的延迟。由于 Redis 无法控制如何将其分配内容映射到内存页，`Used Memory RSS` 过高通常是由于内存使用剧增的缘故。当 Redis 释放内存以后，内存会送回给分配器，而分配器不一定会将内存送回给系统。`Used Memory` 值与操作系统所报告的内存消耗量可能存在差异。这可能是因为内存由 Redis 使用并释放后，并未送回给系统。为了减少内存问题，可执行以下步骤。
    -	通过升级提高缓存大小，消除系统的内存限制。
    -	对密钥设置过期时间，以便主动逐出过旧的值。
    -	监视 `used_memory_rss` 缓存指标。当此值接近缓存大小时，就可能会出现性能问题。将数据分布到多个分片（如果你使用的是高级缓存），或者通过升级来提高缓存大小。

    有关详细信息，请参阅 [Memory Pressure on the server](#memory-pressure-on-the-server)（服务器上的内存压力）。

## <a name="additional-information"></a> 其他信息

-	[我应使用哪种 Redis 缓存产品和大小？](/documentation/articles/cache-faq/#what-redis-cache-offering-and-size-should-i-use)
-	[如何制定基准和测试缓存的性能？](/documentation/articles/cache-faq/#how-can-i-benchmark-and-test-the-performance-of-my-cache)
-	[如何运行 Redis 命令？](/documentation/articles/cache-faq/#how-can-i-run-redis-commands)
-	[如何监视 Azure Redis 缓存](/documentation/articles/cache-how-to-monitor/)

<!---HONumber=Mooncake_0829_2016-->