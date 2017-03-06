<properties
    pageTitle="如何将 Azure Redis 缓存与 Node.js 配合使用 | Azure"
    description="开始将 Azure Redis 缓存与 Node.js 和 node_redis 配合使用。"
    services="redis-cache"
    documentationcenter=""
    author="steved0x"
    manager="douge"
    editor="v-lincan" />
<tags
    ms.assetid="06fddc95-8029-4a8d-83f5-ebd5016891d9"
    ms.service="cache"
    ms.devlang="nodejs"
    ms.topic="hero-article"
    ms.tgt_pltfrm="cache-redis"
    ms.workload="tbd"
    ms.date="02/10/2017"
    wacn.date="03/03/2017"
    ms.author="sdanie" />

# 如何将 Azure Redis 缓存与 Node.js 配合使用
> [AZURE.SELECTOR]
- [.NET](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)
- [ASP.NET](/documentation/articles/cache-web-app-howto/)
- [Node.js](/documentation/articles/cache-nodejs-get-started/)
- [Java](/documentation/articles/cache-java-get-started/)
- [Python](/documentation/articles/cache-python-get-started/)

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

Azure Redis 缓存允许访问 Microsoft 管理的安全、专用的 Redis 缓存。可从 Azure 内部的任何应用程序访问缓存。

本主题说明如何将 Azure Redis 缓存与 Node.js 配合使用。有关将 Azure Redis 缓存与 Node.js 配合使用的另一个示例，请参阅[在 Azure 网站中使用 Socket.IO 生成 Node.js 聊天应用程序](/documentation/articles/web-sites-nodejs-chat-app-socketio/)。

## 先决条件
安装 [node\_redis](https://github.com/mranney/node_redis)：

    npm install redis

本教程使用的是 [node\_redis](https://github.com/mranney/node_redis)。有关使用其他 Node.js 客户端的示例，请使用与 [Node.js Redis 客户端](http://redis.io/clients#nodejs)中列出的 Node.js 客户端对应的各个文档。

## 在 Azure 上创建 Redis 缓存
[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

## <a name="retrieve-the-host-name-and-access-keys"></a> 检索主机名和访问密钥
[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]

## 使用 SSL 安全地连接到缓存
最新版本的 [node\_redis](https://github.com/mranney/node_redis) 支持使用 SSL 连接到 Azure Redis 缓存。以下示例显示了如何使用 SSL 终结点 6380 连接到 Azure Redis 缓存。将 `<name>` 替换为缓存名称，将 `<key>` 替换为主密钥或辅助密钥，如前面的[检索主机名和访问密钥](#retrieve-the-host-name-and-access-keys)部分中所述。

     var redis = require("redis");

      // Add your cache name and access key.
    var client = redis.createClient(6380,'<name>.redis.cache.chinacloudapi.cn', {auth_pass: '<key>', tls: {servername: '<name>.redis.cache.chinacloudapi.cn'}});

> [AZURE.NOTE]
为新的 Azure Redis 缓存实例禁用了非 SSL 端口。如果使用的是不支持 SSL 的不同客户端，请参阅[如何启用非 SSL 端口](/documentation/articles/cache-configure/#access-ports)。
> 
> 

## 在缓存中添加一些内容并检索此内容
以下示例显示了如何连接到 Azure Redis 缓存实例，以及如何在缓存中存储并检索项目。有关将 Redis 与 [node\_redis](https://github.com/mranney/node_redis) 客户端一起使用的更多示例，请参阅 [http://redis.js.org/](http://redis.js.org/)。

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
* [启用缓存诊断](/documentation/articles/cache-how-to-monitor/#enable-cache-diagnostics)，以便可以[监视](/documentation/articles/cache-how-to-monitor/)缓存的运行状况。
* 阅读官方 [Redis 文档](http://redis.io/documentation)。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: add a note about none-ssl port-->