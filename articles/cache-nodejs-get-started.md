<properties
	pageTitle="如何将 Azure Redis 缓存与 Node.js 配合使用 | Windows Azure"
	description="开始将 Azure Redis 缓存与 Node.js 和 node_redis 配合使用。"
	services="redis-cache"
	documentationCenter=""
	authors="steved0x"
	manager="dwrede"
	editor="v-lincan"/>

<tags
	ms.service="cache"
	ms.date="08/17/2015"
	wacn.date="01/21/2016"/>

# 如何将 Azure Redis 缓存与 Node.js 配合使用

Azure Redis 缓存可让你访问 WindowsAzure.cn 管理的、专用安全的 Redis 缓存。可从 Windows Azure 内部的任何应用程序访问你的缓存。

本主题说明如何开始将 Azure Redis 缓存与 Node.js 配合使用。有关将 Azure Redis 缓存与 Node.js 配合使用的另一个示例，请参阅[在 Azure 中使用 Socket.IO 生成 Node.js 聊天应用程序][]。


## 先决条件

安装 [node\_redis](https://github.com/mranney/node_redis)：

    npm install redis

本教程使用 [node\_redis](https://github.com/mranney/node_redis)，但你可以使用 [http://redis.io/clients](http://redis.io/clients) 中列出的任何 Node.js 客户端。

## 在 Azure 上创建 Redis 缓存

Windows Azure 中国目前只支持 PowerShell 或者 Azure CLI 对 Redis 缓存进行管理。

[AZURE.INCLUDE [azurerm-azurechinacloud-environment-parameter](../includes/azurerm-azurechinacloud-environment-parameter.md)]

使用以下的 PowerShell 脚本创建缓存：

	$VerbosePreference = "Continue"

	# Create a new cache with date string to make name unique. 
	$cacheName = "MovieCache" + $(Get-Date -Format ('ddhhmm')) 
	$location = "China North"
	$resourceGroupName = "Default-Web-ChinaNorth"
	
	$movieCache = New-AzureRmRedisCache -Location $location -Name $cacheName  -ResourceGroupName $resourceGroupName -Size 250MB -Sku Basic


## 启用非 SSL 终结点


你可以使用以下的 PowerShell 命令行启用非 SSL 终结点

	Set-AzureRmRedisCache -Name "<your cache name>" -ResourceGroupName "<your resource group name>" -EnableNonSslPort $true


## 在缓存中添加一些内容并检索此内容

	var redis = require("redis");

    // Add your cache name and access key.
	var client = redis.createClient(6379,'<name>.redis.cache.windows.net', {auth_pass: '<key>' });

	client.set("foo", "bar", function(err, reply) {
	    console.log(reply);
	});

	client.get("foo",  function(err, reply) {
	    console.log(reply);
	});


输出：

	OK
	bar


## 后续步骤

- 阅读官方 [Redis 文档](http://redis.io/documentation)。


<!--Image references-->
[1]: ./media/cache-nodejs-get-started/cache01.png
[2]: ./media/cache-nodejs-get-started/cache02.png
[3]: ./media/cache-nodejs-get-started/cache03.png
[4]: ./media/cache-nodejs-get-started/cache04.png

[在 Azure 中使用 Socket.IO 生成 Node.js 聊天应用程序]: /documentation/articles/web-sites-nodejs-chat-app-socketio

<!---HONumber=71-->