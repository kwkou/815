<properties
	pageTitle="如何将 Azure Redis 缓存与 Java 配合使用"
	description="开始将 Azure Redis 缓存与 Java 配合使用"
	services="redis-cache"
	documentationCenter=""
	authors="steved0x"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="cache"
	ms.date="03/04/2016"
	wacn.date="04/26/2016"/>

# 如何将 Azure Redis 缓存与 Java 配合使用

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)
- [ASP.NET](/documentation/articles/cache-web-app-howto/)
- [Node.js](/documentation/articles/cache-nodejs-get-started/)
- [Java](/documentation/articles/cache-java-get-started/)
- [Python](/documentation/articles/cache-python-get-started/)

Azure Redis 缓存可让你访问 Azure.cn 管理的、专用安全的 Redis 缓存。可从 Azure 内部的任何应用程序访问你的缓存。

本主题说明如何开始将 Azure Redis 缓存与 Java 配合使用。

## 先决条件

[Jedis](https://github.com/xetorthio/jedis) - Redis 的 Java 客户端

本教程使用 Jedis，但你可以使用 [http://redis.io/clients](http://redis.io/clients) 中列出的任何 Java 客户端。


## 在 Azure 上创建 Redis 缓存

[AZURE.INCLUDE [redis-cache-create](../includes/redis-cache-create.md)]

## 获取 host name 和 access keys

[AZURE.INCLUDE [redis-cache-create](../includes/redis-cache-access-keys.md)]

## 启用非 SSL 终结点

[AZURE.INCLUDE [redis-cache-create](../includes/redis-cache-non-ssl-port.md)]

## 在缓存中添加一些内容并检索此内容

	package com.mycompany.app;
	import redis.clients.jedis.Jedis;
	import redis.clients.jedis.JedisShardInfo;

	/* Make sure your turn on non SSL port in Azure Redis using the Configuration section in the Azure portal */
	public class App
	{
	  public static void main( String[] args )
	  {
        /* In this line, replace <name> with your cache name: */
	    JedisShardInfo shardInfo = new JedisShardInfo("<name>.redis.cache.chinacloudapi.cn", 6379);
	    shardInfo.setPassword("<key>"); /* Use your access key. */
	    Jedis jedis = new Jedis(shardInfo);
     	jedis.set("foo", "bar");
     	String value = jedis.get("foo");
	  }
	}


## 后续步骤

- 阅读官方 [Redis 文档](http://redis.io/documentation)。


<!--Image references-->
[1]: ./media/cache-java-get-started/cache01.png
[2]: ./media/cache-java-get-started/cache02.png
[3]: ./media/cache-java-get-started/cache03.png
[4]: ./media/cache-java-get-started/cache04.png

<!---HONumber=71-->