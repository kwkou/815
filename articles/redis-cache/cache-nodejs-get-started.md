<properties
	pageTitle="如何将 Azure Redis 缓存与 Node.js 配合使用 | Azure"
	description="开始将 Azure Redis 缓存与 Node.js 和 node_redis 配合使用。"
	services="redis-cache"
	documentationCenter=""
	authors="steved0x"
	manager="douge"
	editor="v-lincan"/>  


<tags
	ms.service="cache"
	ms.date="05/31/2016"
	wacn.date="09/05/2016"/>

# 如何将 Azure Redis 缓存与 Node.js 配合使用

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)
- [ASP.NET](/documentation/articles/cache-web-app-howto/)
- [Node.js](/documentation/articles/cache-nodejs-get-started/)
- [Java](/documentation/articles/cache-java-get-started/)
- [Python](/documentation/articles/cache-python-get-started/)

Azure Redis 缓存可让你访问 Azure.cn 管理的、专用安全的 Redis 缓存。可从 Azure 内部的任何应用程序访问你的缓存。

本主题说明如何将Azure Redis 缓存与 Node.js 配合使用。有关将 Azure Redis 缓存与 Node.js 配合使用的另一个示例，请参阅[在 Azure 网站中使用 Socket.IO 生成 Node.js 聊天应用程序](/documentation/articles/web-sites-nodejs-chat-app-socketio/)。


## 先决条件

安装 [node\_redis](https://github.com/mranney/node_redis)：

    npm install redis

本教程使用 [node\_redis](https://github.com/mranney/node_redis)，但你可以使用 [http://redis.io/clients](http://redis.io/clients) 中列出的任何 Node.js 客户端。

## 在 Azure 上创建 Redis 缓存

[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

## 检索主机名和访问密钥

[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]


## 启用非 SSL 终结点

某些 Redis 客户端不支持 SSL，默认情况下，[为新的 Azure Redis 缓存实例禁用了非 SSL 端口](/documentation/articles/cache-configure/#access-ports)。在编写本文时，[node\_redis](https://github.com/mranney/node_redis) 客户端不支持 SSL。

[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-non-ssl-port.md)]


## 在缓存中添加一些内容并检索此内容

	  var redis = require("redis");
	
	  // Add your cache name and access key.
	var client = redis.createClient(6380,'<name>.redis.cache.chinacloudapi.cn', {auth_pass: '<key>', tls: {servername: '<name>.redis.cache.chinacloudapi.cn'}});
	
	client.set("key1", "value", function(err, reply) {
		    console.log(reply);
		});
	
	client.get("key1",  function(err, reply) {
		    console.log(reply);
		});

输出：

	OK
	value


## 后续步骤

- [启用缓存诊断](/documentation/articles/cache-how-to-monitor/#enable-cache-diagnostics)，以便可以[监视](/documentation/articles/cache-how-to-monitor/)缓存的运行状况。
- 阅读官方 [Redis 文档](http://redis.io/documentation)。

<!---HONumber=Mooncake_0829_2016-->