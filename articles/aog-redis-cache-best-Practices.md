<properties 
	pageTitle="Azure Redis最佳实践" 
	description="Azure Redis最佳实践" 
	services="redis-cache" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="redis-cache-aog" ms.date="" wacn.date="08/31/2016"/>
#Azure Redis最佳实践

下面我会给大家分享一些在使用Azure Redis过程中的最佳实践。 这些信息是基于我数以百计的Azure Redis的客户，在他们使用StackExchange.Redis的时候研究他们所看到的各种各样的错误所总结的。
##StackExchange.Redis

1.	设置AbortConnect 为false，然后让ConnectionMultiplexer自动重新连接， [点击这里](https://gist.github.com/JonCole/36ba6f60c274e89014dd#file-se-redis-setabortconnecttofalse-md)查看详细内容。
2.	重复使用ConnectionMultiplexer – 不要在每次请求的时候创建。另外我们强烈推荐您使用[Lazy&lt;ConnectionMultiplexer&gt;模式](https://www.azure.cn/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/#connect-to-the-cache)。 
3.	Redis最适合较小的数据，所以请考虑把更大的数据切成多个键值存储。在这个有关Redis的讨论中，100KB被认为是“大的数据”。这篇文章列出了Redis在存取大的数据时会引发的问题。
4.	配置[ThreadPool设置](https://gist.github.com/JonCole/e65411214030f0d823cb)来避免超时。
5.	将默认连接超时时间至少设为5秒。这样StackExchange.Redis才会在网络不佳的情况下有足够的时间来重新建立连接。
6.	注意与您正在运行的不同操作关联的性能成本。比如“KEYS”命令的时间复杂度为O(n), 我们应当尽量避免去使用它。在[redis.io 网站](http://redis.io/commands/)上有对于每一种操作的时间复杂度的详细介绍。

##配置和概念

1.	在生产系统中建议使用标准级别或者高级级别的Redis服务。基本级别的Redis服务是一个单节点系统，它不支持数据复制和SLA。另外，至少使用C1缓存。 C0缓存通常适用于开发或者测试环境。
2.	请记住Redis是一个内存数据存储系统。阅读[这篇文章](https://gist.github.com/JonCole/b6354d92a2d51c141490f10142884ea4#file-whathappenedtomydatainredis-md)以意识到一些数据会丢失的场景。
3.	开发您的系统以满足能处理连接因为[打补丁和故障转移](https://gist.github.com/JonCole/317fe03805d5802e31cfa37e646e419d#file-azureredis-patchingexplained-md)而丢失的情况

##性能测试

1.	在开发我们自己的性能测试工具之前，我们可以使用redis-benchmark.exe工具来了解一下可能的吞吐量。需要注意的是redis-benchmark不支持SSL, 所以我们需要在开始测试之前在[Azure门户管理界面中启动Non-SSL port](https://azure.microsoft.com/en-us/documentation/articles/cache-configure/#access-ports)。
2.	用来测试的客户端虚拟机必须要和Redis缓存实例在相同的区域中。
3.	我们建议使用Dv2系列虚拟机作为你的客户端，因为这个系列的虚拟机拥有更好的配置，这样我们能获得最佳的测试结果。
4.	确认您选择的用来测试客户端虚拟机在计算能力和带宽性能方面要优于你要测试的Redis缓存。
5.	如果您使用的是Windows，需要开启VRSS选项。[点击这里](https://technet.microsoft.com/zh-cn/library/dn383582%28v=ws.11%29.aspx)查看详细信息。
6.	高级级别的Redis实例拥有更低的网络延时和更高吞吐量，因为它们运行在更高配置的硬件上，比如CPU和网络。

##Redis-Benchmark示例

使用1K的负载来测试管道SET请求

    redis-benchmark.exe -h yourcache.redis.cache.windows.net -a yourAccesskey -t SET -n 1000000 -d 1024 -P 50
    
使用1K的负载来测试管道GET请求
>注: 先运行以上的SET测试来填充缓存

    redis-benchmark.exe -h yourcache.redis.cache.windows.net -a yourAccesskey -t GET -n 1000000 -d 1024 -P 50
