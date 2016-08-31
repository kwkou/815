<properties 
	pageTitle="Redis Cache 常见问题解答" 
	description="Redis Cache 常见问题解答" 
	services="redis-cache" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="redis-cache-aog" ms.date="" wacn.date="08/31/2016"/>
# Redis Cache 常见问题解答

当客户连接azure redis时，有时候会遇到一些错误，通常情况下我们需要排查两个方面，确认该错误是来源于client端，或者是server端。我们就这两方面经常遇到的问题做一个简单的总结。

## Client端常见问题

在client端往往会遇到一些连接超时问题，以下是关于该问题常见的一些原因。

### 内存压力

**问题描述：**如果client端机器某段时间的内存使用量很大，会导致很多性能问题，该机器会出现处理数据缓慢延迟，使得server端接收数据也发生延迟。当内存压力达到一定的程度，系统会不得不将本应该在物理内存处理的数据转移到虚拟内存中去处理，这会导致非常明显的系统运行缓慢。

**监控该问题**

1.	监控机器的内存使用情况，避免超过可用内存。
2.	监控Page Faults/Sec的性能计数器。大多数情况下，甚至正常的操作，系统都会出现page faults，因此通过监控Page Faults/Sec的性能计数器能有效的观察到峰值异常情况的发生。

**解决方法：**
可以升级虚拟机到更高的型号从而获取到更多内存，或者改善内存使用模式从而减少内存消耗。

### 突发流量

**问题描述：**当网络流量突然增加并且ThreadPool设置不当会导致client端处理从redis server端发出来的数据变得缓慢。

**监控该问题：**

可以通过代码监控统计ThreadPool的数据，可以参考[链接](https://github.com/JonCole/SampleCode/blob/master/ThreadPoolMonitor/ThreadPoolLogger.cs)。也可以通过检查StackExchange.Redis 发出的异常超时报错信息。一个简单的例子如下：

	System.TimeoutException: Timeout performing EVAL, inst: 8, mgr: Inactive, queue: 0, qu: 0, qs: 0, qc: 0, wr: 0, wq: 0, in: 64221, ar: 0, 
	IOCP: (Busy=6,Free=999,Min=2,Max=1000), WORKER: (Busy=7,Free=8184,Min=2,Max=8191)

通过上面的例子，可以看到几个比较有趣的问题：

1.	在IOCP和WORKER部分，注意到busy的值要比Min的值大，这就表明了您的ThreadPool需要做一些调整了。
2.	对于in: 64221，表明从kernel socket层有64211 bytes的数据被接收到，但是还没有被应用(e.g. StackExchange.Redis)做读取处理。这说明了您的应用做数据的读取速度跟不上网络上数据的传输速度。

**解决方法：**配置您的[ThreadPool](https://gist.github.com/JonCole/e65411214030f0d823cb) 以确保ThreadPool可以在网络流量突发的情况下快速增加。

### 高CPU使用量

**问题描述：**
当出现高cpu使用率问题时表明client端的系统已经处理不过来应该执行的任务。从redis的角度来看，这意味着client很有可能因为没能及时处理redis server端发过来的请求而导致相应请求失败。

**监控该问题：**客户可以通过监控azure管理门户上提供的监控服务观察系统的cpu使用率。需要注意的一点是尽量避免监控某个进程的cpu，因为可能某个单一的进程cpu利用率很低，但是整体系统的cpu利用率却很高。同时也需要注意在发生超时时观察是否cpu也出现了利用率的峰值情况。如果是由于cpu高利用率的原因，在超时的异常报错信息中可以看到"in: XXX"该项的值会比较高(与突发流量部分类似)。
注意：StackExchange.Redis 1.1.603版本或者更高版本在超时发生时，会给出"local-cpu"使用率的信息，这将有助于我们准备判断是否cpu使用率会影响系统的性能。

**解决方法：**升级虚拟机到更高的型号，保证更高的cpu资源，或者可以调查是什么原因导致的cpu突然升高。

### Client端的带宽超限

**问题描述：**不同型号的client在网络带宽方面有不同的限制，如果client机器的网络带宽超过了限制，将会导致client端处理数据的速度跟不上server端发送数据的速度，从而会有超时发生。

**监控该问题：**可以通过[代码](https://github.com/JonCole/SampleCode/blob/master/BandWidthMonitor/BandwidthLogger.cs)的方式来监控带宽使用量。需要注意的一点是该代码在某些有特定限制的环境下无法成功运行，比如azure websites。

**解决方法：**可以通过升级虚拟机型号获取更多的带宽资源 或者减少网络带宽消耗。

### 大的请求/响应

**问题描述：**大的请求和响应也会导致超时。举个例子，假设我们设置的超时时限为1s，我们的应用在物理网络下同时请求两个key(e.g. A and B)。大多数的client端都支持“流水线”处理请求，也就是A和B以一个接着一个的方式被发送给server端，不需要中间等待响应，而Server端会以同样的顺序发送响应。而如果A的响应请求很大，这就会占据很多超时时限的时间。

接下来我们来说明一下这个问题，首先请求A和请求B很快被发送出去，server端很快开始发送A请求响应和B请求响应，然而，在处理A请求响应和数据传输期间，B请求是被阻塞的，所以当A响应处理完之后，即使server端可以很快的对B响应，还是发生了超时。

	|-------- 1 Second Timeout (A)----------|
	|-Request A-|
	     |-------- 1 Second Timeout (B) ----------|
	     |-Request B-|
	            |- Read Response A --------|
	                                       |- Read Response B-| (**TIMEOUT**)

**监控该问题：**这个问题相对比较难监控，通常可以通过在code中做特殊的设置来检索一些比较大的请求和响应，从而有效追踪该问题。

**解决方法：**

1.	使用redis时推荐用大量的小数据，而不是少量的大数据，因此我们建议客户将大数据分割成小数据使用，更多的一些关于为什么要使用小数据的资料可以参考[链接](https://groups.google.com/forum/#!searchin/redis-db/size/redis-db/n7aa2A4DZDs/3OeEPHSQBAAJ)。
2.	通过提高虚拟机型号(client端和redis server端)来增加带宽容量，这样可以减少在传输大数据时候的传送时间。需要注意的一点是，只增加client端或者server端一端的带宽是不够的，因此客户需要评估自己的带宽使用量，然后保证与客户的虚拟机带宽容量匹配。
3.	增加对象ConnectionMultiplexer的数量和轮询请求数量。如果选择这样做，客户需要注意到的是确保不要为每个request都创建一个新的ConnectionMultiplexer，因为这样会大大的消耗机器性能。


## Server端常见问题

造成Server端的延迟原因有以下几种情况：

### 内存压力

**问题描述：**Server端的内存压力会导致各种性能问题，同时会引起server端处理请求缓慢。当内存压力达到一定的程度，系统会不得不将本应该在物理内存处理的数据转移到虚拟内存中去处理，这会导致非常明显的系统运行缓慢。导致系统压力的原因主要有以下几种：

1.	Cache已经被用户用完。
2.	Redis里存在大内存碎片，这种情况往往是由于客户存储了大的数据对象导致的问题(Redis的最优使用是存放小的数据对象，详细信息可以参考[文档](https://groups.google.com/forum/))

**监控该问题：**redis 对外公开两个数据可以帮助客户鉴别该问题。一个数据是used_memory，另外一个是used_memory_rss。通过azure portal或者通过[redis info](http://redis.io/commands/info)命令行，可以获得该数据。

**解决方法：**客户可以做几种改变从而有效的保持内存使用健康。

1.	[配置内存使用规则](https://azure.microsoft.com/en-us/documentation/articles/cache-configure/#maxmemory-policy-and-maxmemory-reserved)以及设置key的过期时间。需要注意的一点是如果客户redis中存在碎片，可能这样做效果不是很明显。
2.	[配置最大存储保留值](https://azure.microsoft.com/en-us/documentation/articles/cache-configure/#maxmemory-policy-and-maxmemory-reserved)从而使得有足够大的空间来放存储碎片。
3.	将大的cache对象分割成一个个对应的小的cache对象。
4.	升级redis到更大的cache。
	
### 高cpu/server负载大

**问题描述：**高cpu使用率意味着，即使在redis server端可以很快的发送响应请求的条件下，server端也没办法及时的处理从redis client端发送过来的响应请求。  

**监控该问题：**通过azure管理门户或者其他的一些性能计数器来监控整个系统的cpu使用率。需要注意的一点的尽量避免监控某个进程的cpu，因为可能某个单一的进程cpu利用率很低，但是整体系统的cpu利用率却很高。同时也需要注意在发生超时时观察cpu是否也出现了利用率的峰值情况。

**解决方法：**升级虚拟机到更高的型号，保证更高的cpu资源，或者可以调查是什么原因导致的cpu突然升高。

### Server端的带宽超限

**问题描述：**不同型号的redis cache在网络带宽上的限制不同。如果redis sever端超过了网络带宽的限制，数据将不会被快速的传到client端。server端无法快速将数据传到client端将会导致超过的发生。

**监控问题：**Azure管理门户上会有一些监控诊断数据(Cache Reads/Sec)，可以利用这些数据来检查server端网络的使用率。其对应单位为Bytes/sec。

**解决方法：**可以升级更高的cache型号。

### 运行一些消耗大的操作/脚本

**问题描述：**并不是所有的redis操作都是完全一样的，有些操作相对其他的操作的消耗比较大。关于每种操作的时间消耗复杂性，您可以参考[这里](http://redis.io/commands/)。我们强烈推荐客户检查运行在redis上的相关命令并且尝试去理解每条命令对redis性能的影响。举个例子，命令KEYS是我们最常见的客户使用的命令，但是客户往往却不知道这是一条时间消耗为O(N)数量级类型的命令，因此我们应该避免使用。

**监控问题：**[检查slow 日志](http://redis.io/commands/slowlog)

**解决方法：**避免使用那些时间消耗大的命令或者通过升级redis到更高版本来获取到更多的cpu资源。

