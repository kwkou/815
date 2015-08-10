<properties 
   pageTitle="如何将 Azure Redis 缓存与 Java 配合使用" 
   description="开始将 Azure Redis 缓存与 Java 配合使用" 
   services="redis-cache" 
   documentationCenter="" 
   authors="MikeWasson" 
   manager="wpickett" 
   editor=""/>

<tags
   ms.service="cache"
   ms.date="04/30/2015"
   wacn.date="06/26/2015"/>

# 如何将 Azure Redis 缓存与 Java 配合使用 

Azure Redis 缓存可让你访问世纪互联管理的、专用安全的 Redis 缓存。可从 Windows Azure 内部的任何应用程序访问你的缓存。

本主题说明如何开始将 Azure Redis 缓存与 Java 配合使用。


## 先决条件

[Jedis](https://github.com/xetorthio/jedis) - Redis 的 Java 客户端

本教程使用 Jedis，但你可以使用 [http://redis.io/clients](http://redis.io/clients) 中列出的任何 Java 客户端。


## 在 Azure 上创建 Redis 缓存

在 [Azure 管理门户](http://manage.windowsazure.cn)中，单击“新建”，然后选择“Redis 缓存”。

  ![][1]

输入 DNS 主机名。该名称的格式为 `<name>.redis.cache.chinacloudapi.cn`。单击“创建”。

  ![][2]


创建缓存后，在门户中单击它以查看缓存设置。单击“密钥”下的链接，然后复制主密钥。稍后需要使用此密钥进行身份验证。

  ![][4]


## 启用非 SSL 终结点


单击“端口”下的链接，然后单击“只允许通过 SSL 访问”旁边的“否”。这样就会为缓存启用非 SSL 端口。Jedis 客户端当前不支持 SSL。

  ![][3]


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

- [启用缓存诊断](https://msdn.microsoft.com/zh-cn/library/azure/dn763945.aspx#EnableDiagnostics)，以便可以[监视](https://msdn.microsoft.com/zh-cn/library/azure/dn763945.aspx)缓存的运行状况。 
- 阅读官方 [Redis 文档](http://redis.io/documentation)。


<!--Image references-->
[1]: ./media/cache-java-get-started/cache01.png
[2]: ./media/cache-java-get-started/cache02.png
[3]: ./media/cache-java-get-started/cache03.png
[4]: ./media/cache-java-get-started/cache04.png

<!---HONumber=61-->