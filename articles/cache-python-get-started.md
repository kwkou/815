<properties
	pageTitle="如何将 Azure Redis 缓存与 Python 配合使用 | Azure"
	description="开始将 Azure Redis 缓存与 Python 配合使用"
	services="redis-cache"
	documentationCenter=""
	authors="steved0x"
	manager="dwrede"
	editor="v-lincan"/>

<tags
	ms.service="cache"
	ms.date="03/04/2016"
	wacn.date="04/26/2016"/>

# 如何将 Azure Redis 缓存与 Python 配合使用

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)
- [ASP.NET](/documentation/articles/cache-web-app-howto/)
- [Node.js](/documentation/articles/cache-nodejs-get-started/)
- [Java](/documentation/articles/cache-java-get-started/)
- [Python](/documentation/articles/cache-python-get-started/)

本主题说明如何开始将 Azure Redis 缓存与 Python 配合使用。


## 先决条件

安装 [redis-py](https://github.com/andymccurdy/redis-py)。


## 在 Azure 上创建 Redis 缓存

[AZURE.INCLUDE [redis-cache-create](../includes/redis-cache-create.md)]

## 获取 host name 和 access keys

[AZURE.INCLUDE [redis-cache-create](../includes/redis-cache-access-keys.md)]

## 启用非 SSL 终结点

[AZURE.INCLUDE [redis-cache-create](../includes/redis-cache-non-ssl-port.md)]

## 在缓存中添加一些内容并检索此内容

    >>> import redis
    >>> r = redis.StrictRedis(host='<name>.redis.cache.chinacloudapi.cn',
          port=6380, db=0, password='<key>', ssl=True)
    >>> r.set('foo', 'bar')
    True
    >>> r.get('foo')
    b'bar'

将 `<name>` 替换为你的缓存名称，将 `<key>` 替换为你的缓存访问密钥。


<!--Image references-->
[1]: ./media/cache-python-get-started/redis-cache-new-cache-menu.png
[2]: ./media/cache-python-get-started/redis-cache-cache-create.png

<!---HONumber=71-->