<properties 
   pageTitle="如何配置 Azure Redis 缓存"
   description="了解 Azure Redis 缓存的默认 Redis 配置，并了解如何配置 Azure Redis 缓存实例"
   services="redis-cache"
   documentationCenter="na"
   authors="steved0x"
   manager="dwrede"
   editor="tysonn" />
<tags 
   ms.service="cache"
   ms.date="07/24/2015"
   wacn.date="08/29/2015" />

# 如何配置 Azure Redis 缓存

本主题介绍如何查看和更新 Azure Redis 缓存实例的配置，并介绍了 Azure Redis 缓存实例的默认 Redis 服务器配置。

## 配置 Redis 缓存设置

可以在 [Windows Azure 门户](http://manage.windowsazure.cn/)中使用“浏览”边栏选项卡访问缓存。

![Azure Redis 缓存浏览边栏选项卡](./media/cache-configure/IC796920.png)

单击“Redis 缓存”查看你的缓存。

![Azure Redis 缓存浏览缓存列表](./media/cache-configure/IC796921.png)

选择所需缓存以查看其属性。

![Redis 缓存的所有设置](./media/cache-configure/IC808312.png)

单击“设置”或“所有设置”以查看和配置缓存。

![Redis 缓存设置](./media/cache-configure/IC808313.png)

## 属性

单击“属性”查看有关缓存的信息，包括缓存终结点和端口。

![Redis 缓存属性](./media/cache-configure/IC808314.png)

## 访问密钥

单击“访问密钥”查看或重新生成缓存访问密钥。通过正在连接到缓存的客户端，从“属性”边栏选项卡将这些密钥与主机名和端口一起使用。

![Redis 缓存访问密钥](./media/cache-configure/IC808315.png)

## 访问端口

默认情况下，为新缓存禁用非 SSL 访问。要启用非 SSL 端口，则单击“访问端口”边栏选项卡，然后单击“否”。

![Redis 缓存访问端口](./media/cache-configure/IC808316.png)

## Diagnostics

单击“诊断”以配置用于存储缓存诊断的存储帐户。

![Redis 缓存诊断](./media/cache-configure/IC808317.png)

有关详细信息，请参阅[如何监视 Azure Redis 缓存](/documentation/articles/cache-how-to-monitor)。

## Maxmemory-policy 和 maxmemory-reserved

单击“Maxmemory 策略”为缓存配置内存策略。“maxmemory-policy”设置将为缓存配置逐出策略，“maxmemory-reserved”将为非缓存进程配置保留的内存。

![Redis 缓存 Maxmemory 策略](./media/cache-configure/IC808318.png)

“Maxmemory 策略”允许从以下逐出策略中进行选择。

-	volatile-lru（默认）。
-	allkeys-lru
-	volatile-random
-	allkeys-random
-	volatile-ttl
-	noeviction

有关 Maxmemory 策略的详细信息，请参阅[逐出策略](http://redis.io/topics/lru-cache#eviction-policies)。

“maxmemory-reserved”设置可为故障转移过程中的复制等非缓存操作配置保留的内存量 (MB)。碎片比率较高时也可使用此设置。设置此值让你能够在负载变化时具有更一致的 Redis 服务器体验。对于写入密集型工作负荷，应将此值设置为较高。为此类操作保留内存后，将无法存储缓存数据。

>[AZURE.IMPORTANT]“maxmemory-reserved”设置仅适用于标准缓存。

## 密钥空间通知（高级设置）

单击“高级设置”配置 Redis 密钥空间通知。密钥空间通知让客户端能够在发生特定事件时接收通知。

![Redis 缓存高级设置](./media/cache-configure/IC808319.png)

>[AZURE.IMPORTANT]密钥空间通知和“notify-keyspace-events”设置仅适用于标准缓存。

有关详细信息，请参阅 [Redis 密钥空间通知](http://redis.io/topics/notifications)。有关示例代码，请参阅 [Hello world](https://github.com/rustd/RedisSamples/tree/master/HelloWorld) 示例中的 [KeySpaceNotifications.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/KeySpaceNotifications.cs) 文件。

## 用户和标记

![Redis 缓存用户和标记](./media/cache-configure/IC808320.png)

门户中的“用户”部分对基于角色的访问控制 (RBAC) 提供支持，以帮助简单准确地满足组织的访问管理要求。有关详细信息，请参阅 [Windows Azure 预览门户中基于角色的访问控制](/documentation/articles/role-based-access-control-configure)。

“标记”部分可帮助你组织资源。有关详细信息，请参阅[使用标记来组织 Azure 资源](/documentation/articles/resource-group-using-tags)。

## 默认 Redis 服务器配置

新的 Azure Redis 缓存实例均已配置以下默认 Redis 配置值。

>[AZURE.NOTE]无法使用 `StackExchange.Redis.IServer.ConfigSet` 方法更改本部分中的设置。如果使用此部分中的任一命令调用此方法，将引发如下异常：
>
>`StackExchange.Redis.RedisServerException: ERR unknown command 'CONFIG'`
>  
>任何可配置值，如“最大内存策略”，均可通过门户进行配置。

|设置|默认值|说明|
|---|---|---|
|数据库|16|默认数据库是 DB 0，可使用 connection.GetDataBase(dbid) 对每个连接使用不同数据库，其中 dbid 是 0 到 15 之间的数字。|
|maxclients|10,000|这是同一时间内允许的最大已连接客户端数。一旦达到该限制，Redis 将在关闭所有新连接的同时发送“达到客户端最大数量”的错误。|
|maxmemory-policy|volatile-lru|Maxmemory 策略是达到 maxmemory（创建缓存时所选缓存服务的大小）时，Redis 将根据它选择要删除内容的设置。Azure Redis 缓存的默认设置为 volatile-lru，此设置使用 LRU 算法删除具有过期设置的密钥。可以在门户中配置此设置。有关详细信息，请参阅 [Maxmemory-policy 和 maxmemory-reserved](#maxmemory-policy-and-maxmemory-reserved)。|
|maxmemory-samples|3|LRU 和最小 TTL 算法不是精确算法而是近似算法（为了节省内存），因此你还可以选择示例大小进行检查。例如，对于默认设置，Redis 将检查三个密钥并选取最近使用较少的一个。|
|lua-time-limit|5,000|Lua 脚本的最大执行时间（以毫秒为单位）。如果达到最大执行时间，Redis 将记录达到最大允许时间后仍继续执行的脚本，并将开始在查询答复时出现错误。|
|lua-event-limit|500|这是脚本事件队列的最大大小。|
|client-output-buffer-limit normalclient-output-buffer-limit pubsub|0 0 032mb 8mb 60|客户端输出缓冲区限制可用于强制断开处于某种原因（一个常见原因是发布/订阅客户端处理消息的速度慢于发布者提供消息的速度）而未从服务器快速读取数据的客户端的连接。有关详细信息，请参阅 [http://redis.io/topics/clients](http://redis.io/topics/clients)。|

## Azure Redis 缓存中不支持 Redis 命令

>[AZURE.IMPORTANT]因为 Azure Redis 缓存实例的配置和管理是使用 Azure 门户完成的，所以禁用了以下命令。如果尝试调用它们，将收到一条类似于 `"(error) ERR unknown command"` 的错误消息。
>
>-	BGREWRITEAOF
>-	BGSAVE
>-	配置
>-	调试
>-	迁移
>-	保存
>-	关机
>-	SLAVEOF

有关 Redis 命令的详细信息，请参阅 [http://redis.io/commands](http://redis.io/commands)。

## Redis 控制台

可以使用 **Redis 控制台**向 Azure Redis 缓存实例安全地发布命令，此操作适用于标准缓存。要访问 Redis 控制台，则从“Redis 缓存”边栏选项卡单击“控制台”。

![Redis 控制台](./media/cache-configure/redis-console-menu.png)

>[AZURE.IMPORTANT]Redis 控制台仅适用于标准缓存。

要发布针对缓存实例的命令，只需将所需命令键入到控制台即可。

![Redis 控制台](./media/cache-configure/redis-console.png)

有关为 Azure Redis 缓存禁用的 Redis 命令列表，请参阅之前的 [Azure Redis 缓存中不支持 Redis 命令](#redis-commands-not-supported-in-azure-redis-cache)部分。有关 Redis 命令的详细信息，请参阅 [http://redis.io/commands](http://redis.io/commands)。

## 后续步骤
-	有关使用 Redis 命令的详细信息，请参阅[如何运行 Redis 命令？](/documentation/articles/cache-faq#how-can-i-run-redis-commands)

<!---HONumber=67-->