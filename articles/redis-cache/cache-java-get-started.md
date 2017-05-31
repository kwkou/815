<properties
    pageTitle="如何将 Azure Redis 缓存与 Java 配合使用 | Azure"
    description="开始将 Azure Redis 缓存与 Java 配合使用"
    services="redis-cache"
    documentationcenter=""
    author="steved0x"
    manager="douge"
    editor="" />
<tags
    ms.assetid="29275a5e-2e39-4ef2-804f-7ecc5161eab9"
    ms.service="cache"
    ms.devlang="java"
    ms.topic="hero-article"
    ms.tgt_pltfrm="cache-redis"
    ms.workload="tbd"
    ms.date="04/13/2017"
    wacn.date="05/31/2017"
    ms.author="sdanie" />

# 如何将 Azure Redis 缓存与 Java 配合使用
> [AZURE.SELECTOR]
- [.NET](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)
- [ASP.NET](/documentation/articles/cache-web-app-howto/)
- [Node.js](/documentation/articles/cache-nodejs-get-started/)
- [Java](/documentation/articles/cache-java-get-started/)
- [Python](/documentation/articles/cache-python-get-started/)

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

Azure Redis 缓存可让你访问 Azure.cn 管理的专用 Redis 缓存。可从 Azure 内部的任何应用程序访问缓存。

本主题说明如何将 Azure Redis 缓存与 Java 配合使用。

## 先决条件
[Jedis](https://github.com/xetorthio/jedis) - Redis 的 Java 客户端

本教程使用 Jedis，但你可以使用 [http://redis.io/clients](http://redis.io/clients) 中列出的任何 Java 客户端。

## 在 Azure 上创建 Redis 缓存
[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

## <a name="retrieve-the-host-name-and-access-keys"></a> 检索主机名和访问密钥
[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]

## 使用 SSL 安全地连接到缓存
最新版本的 [jedis](https://github.com/xetorthio/jedis) 支持使用 SSL 连接到 Azure Redis 缓存。以下示例显示了如何使用 SSL 终结点 6380 连接到 Azure Redis 缓存。将 `<name>` 替换为缓存名称，将 `<key>` 替换为主密钥或辅助密钥，如前面的[检索主机名和访问密钥](#retrieve-the-host-name-and-access-keys)部分中所述。

    boolean useSsl = true;
    /* In this line, replace <name> with your cache name: */
    JedisShardInfo shardInfo = new JedisShardInfo("<name>.redis.cache.chinacloudapi.cn", 6379, useSsl);
    shardInfo.setPassword("<key>"); /* Use your access key. */

> [AZURE.NOTE]
为新的 Azure Redis 缓存实例禁用了非 SSL 端口。如果使用的是不支持 SSL 的不同客户端，请参阅[如何启用非 SSL 端口](/documentation/articles/cache-configure/#access-ports)。
> 
> 

## 在缓存中添加一些内容并检索此内容
    package com.mycompany.app;
    import redis.clients.jedis.Jedis;
    import redis.clients.jedis.JedisShardInfo;

    public class App
    {
      public static void main( String[] args )
      {
        boolean useSsl = true;
        /* In this line, replace <name> with your cache name: */
        JedisShardInfo shardInfo = new JedisShardInfo("<name>.redis.cache.chinacloudapi.cn", 6379, useSsl);
        shardInfo.setPassword("<key>"); /* Use your access key. */
        Jedis jedis = new Jedis(shardInfo);
        jedis.set("foo", "bar");
        String value = jedis.get("foo");
      }
    }

## 后续步骤
* [启用缓存诊断](/documentation/articles/cache-how-to-monitor/#EnableDiagnostics)，以便可以[监视](/documentation/articles/cache-how-to-monitor/)缓存的运行状况。
* 阅读官方 [Redis 文档](http://redis.io/documentation)。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: add a note about non-ssl port-->