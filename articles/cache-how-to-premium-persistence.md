<properties 
	pageTitle="如何为高级 Azure Redis 缓存配置数据暂留" 
	description="了解如何为高级级别的 Azure Redis 缓存实例配置和管理数据暂留" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="cache"
	ms.date="03/04/2016"
	wacn.date="04/26/2016"/>

# 如何为高级 Azure Redis 缓存配置数据暂留

Azure Redis 缓存具有不同的缓存产品（包括新推出的高级层），使缓存大小和功能的选择更加灵活。

Azure Redis 缓存高级级别包括群集、暂留和虚拟网络支持。本文介绍如何配置高级 Azure Redis 缓存实例中的暂留。

有关其他高级缓存功能的信息，请参阅[如何配置高级 Azure Redis 缓存的群集功能](/documentation/articles/cache-how-to-premium-clustering/)和[如何配置高级 Azure Redis 缓存的虚拟网络支持](/documentation/articles/cache-how-to-premium-vnet/)。

## 什么是数据暂留？
Redis 暂留可让你保留存储在 Redis 中的数据。你还可获取快照并备份数据，以便在出现硬件故障时进行加载。这相对于基本级别或标准级别是一项巨大优势，因为基本级别或标准级别将所有数据存储在内存中，在出现故障的情况下，如果缓存节点停机，则可能导致数据丢失。

Azure Redis 缓存使用 [RDB 模型](http://redis.io/topics/persistence)提供的 Redis 持久性，允许将数据存储在 Azure 存储帐户中。配置暂留以后，Azure Redis 缓存会按照可配置的备份频率，将 Redis 缓存的快照以 Redis 二进制格式暂留在磁盘上。如果发生了灾难性事件，导致主缓存和副缓存都无法使用，则会使用最新快照重新构造缓存。

可以在创建缓存过程中通过“新建 Redis 缓存”边栏选项卡配置持久性；另外还可以在现有高级缓存的“设置”边栏选项卡上进行这项配置。

## 创建高级缓存

在 Azure 中国区，只能通过 Azure PowerShell 或 Azure CLI 管理 Redis 缓存


[AZURE.INCLUDE [azurerm-azurechinacloud-environment-parameter](../includes/azurerm-azurechinacloud-environment-parameter.md)]


使用以下 PowerShell 脚本创建具有 Redis 持久性的高级缓存：

	$VerbosePreference = "Continue"

	# Create a new cache with date string to make name unique. 
	$cacheName = "MovieCache" + $(Get-Date -Format ('ddhhmm')) 
	$location = "China North"
	$resourceGroupName = "Default-Web-WestUS"
	
	$movieCache = New-AzureRmRedisCache -Location $location -Name $cacheName  -ResourceGroupName $resourceGroupName -Size 6GB -Sku Premium -RedisConfiguration @{"rdb-backup-enabled"="true"; "rdb-backup-frequency"="60"; "rdb-backup-max-snapshot-count"="1"; "rdb-storage-connection-string"="DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey;EndpointSuffix=core.chinacloudapi.cn"}

以下部分中的步骤说明如何在新的高级缓存上配置 Redis 持久性。

## 配置 Redis 持久性

可以使用 **Set-AzureRmRedisCache** PowerShell 命令来配置 Redis 数据持久性：

	Set-AzureRmRedisCache -Name $cacheName  -ResourceGroupName $resourceGroupName -RedisConfiguration @{"rdb-backup-enabled"="true"; "rdb-backup-frequency"="60"; "rdb-backup-max-snapshot-count"="1"; "rdb-storage-connection-string"="DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey;EndpointSuffix=core.chinacloudapi.cn"}

在此 PowerShell 命令中可以看到，对于 **-RedisConfiguration** 参数，将“rdb-backup-enabled”设置为 true 可以启用 RDB，设置为 false 可以禁用 RDB。

若要配置备份间隔，可以将“rdb-backup-frequency”设置为 15（表示 **15 分钟**）、30（表示 **30 分钟**）、60（表示 **60 分钟**）、360（表示 **6 小时**）、720（表示 **12 小时**）或 1440（表示 **24 小时**）。在上一个备份操作成功完成以后，此时间间隔就会开始倒计时，同时会启动新的备份。

若要配置存储帐户，可以将“rdb-storage-connection-string”设置为 Azure 中国区的某个连接字符串。在上述命令中可以看到，你需要在连接字符串中指定 BlobEndpoint、QueueEndpoint 和 TableEndpoint。

>[AZURE.IMPORTANT]如果重新生成了持久性帐户的存储密钥，则必须更新“rdb-backup-frequency”。

一旦备份频率间隔时间已过，则会启动下一次备份（或新缓存的首次备份）。



## 保留常见问题

以下列表包含有关 Azure Redis 缓存保留常见问题的解答。

## 能否在此前已创建的缓存的基础上启用保留？

是的，可以在创建缓存时或者在现有高级缓存上配置 Redis 持久性。

## 能否在创建缓存后更改备份频率？

是的，可以在 Azure PowerShell 更改备份频率。以下是一个命令示例，通过修改 rdb-backup-frequency 可以改变备份频率。

	Set-AzureRmRedisCache -Name $cacheName  -ResourceGroupName $resourceGroupName -RedisConfiguration @{"rdb-backup-enabled"="true"; "rdb-backup-frequency"="60"; "rdb-backup-max-snapshot-count"="1"; "rdb-storage-connection-string"="DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey;EndpointSuffix=core.chinacloudapi.cn"}

## 为何我的备份频率为 60 分钟，而两次备份的间隔却超过 60 分钟？

在上一次备份过程成功完成之前，本次备份不会开始，其频率所对应的时间间隔也不会开始计算。如果备份频率为 60 分钟，而备份过程需要 15 分钟才能成功完成，则在上一次备份开始以后，要再过 75 分钟才会开始下一次备份。

## 进行新备份以后，旧备份会发生什么情况

除最新备份外的所有备份都会自动删除。这种删除可能不会即刻发生，但旧备份是不会无限期保留下去的。

## 后续步骤
了解如何使用更多的高级版缓存功能。

-	[如何为高级 Azure Redis 缓存配置群集功能](/documentation/articles/cache-how-to-premium-clustering/)
-	[如何为高级 Azure Redis 缓存配置虚拟网络支持](/documentation/articles/cache-how-to-premium-vnet/)
  
<!-- IMAGES -->

[redis-cache-new-cache-menu]: ./media/cache-how-to-premium-persistence/redis-cache-new-cache-menu.png

[redis-cache-premium-pricing-tier]: ./media/cache-how-to-premium-persistence/redis-cache-premium-pricing-tier.png

[redis-cache-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-persistence.png

[redis-cache-persistence-selected]: ./media/cache-how-to-premium-persistence/redis-cache-persistence-selected.png

[redis-cache-settings]: ./media/cache-how-to-premium-persistence/redis-cache-settings.png

<!---HONumber=Mooncake_0104_2016-->