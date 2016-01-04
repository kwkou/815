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
	ms.date="10/01/2015"
	wacn.date="01/04/2016"/>

# 如何为高级 Azure Redis 缓存配置数据暂留

Azure Redis 缓存具有不同的缓存产品（包括新推出的高级级别，目前为预览版），使缓存大小和功能的选择更加灵活。

Azure Redis 缓存高级级别包括群集、暂留和虚拟网络支持。本文介绍如何配置高级 Azure Redis 缓存实例中的暂留。

有关其他高级缓存功能的信息，请参阅[如何配置高级 Azure Redis 缓存的群集功能](/documentation/articles/cache-how-to-premium-clustering)和[如何配置高级 Azure Redis 缓存的虚拟网络支持](/documentation/articles/cache-how-to-premium-vnet)。

>[AZURE.NOTE]Azure Redis 缓存高级级别目前处于预览状态。在预览期间，暂留不能与群集功能或虚拟网络一起使用。

## 什么是数据暂留？
Redis 暂留可让你保留存储在 Redis 中的数据。你还可获取快照并备份数据，以便在出现硬件故障时进行加载。这相对于基本级别或标准级别是一项巨大优势，因为基本级别或标准级别将所有数据存储在内存中，在出现故障的情况下，如果缓存节点停机，则可能导致数据丢失。

Azure Redis 缓存提供的 Redis 暂留功能允许将数据存储在 Azure 存储帐户中。公共预览版支持 [RDB 模型](http://redis.io/topics/persistence)，并且很快就会支持 [AOF](http://redis.io/topics/persistence)。

## 数据暂留
配置暂留以后，Azure Redis 缓存会按照可配置的备份频率，将 Redis 缓存的快照以 Redis 二进制格式暂留在磁盘上。如果发生了灾难性事件，导致主缓存和副缓存都无法使用，则会使用最新快照重新构造缓存。

Windows Azure 中国目前只支持 PowerShell 或者 Azure CLI 对 Redis 缓存进行管理。

[AZURE.INCLUDE [azurerm-azurechinacloud-environment-parameter](../includes/azurerm-azurechinacloud-environment-parameter.md)]

使用以下的 PowerShell 脚本创建一个已激活数据暂留的高级缓存：

	$VerbosePreference = "Continue"

	# Create a new cache with date string to make name unique. 
	$cacheName = "MovieCache" + $(Get-Date -Format ('ddhhmm')) 
	$location = "China North"
	$resourceGroupName = "Default-Web-WestUS"
	
	$movieCache = New-AzureRmRedisCache -Location $location -Name $cacheName  -ResourceGroupName $resourceGroupName -Size 6GB -Sku Premium -RedisConfiguration @{"rdb-backup-enabled"="true"; "rdb-backup-frequency"="60"; "rdb-backup-max-snapshot-count"="1"; "rdb-storage-connection-string"="DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey;EndpointSuffix=core.chinacloudapi.cn"}

这一段脚本可以建立一个6GB，“备份频率”为60分钟的高级缓存。

“备份频率”选项包括“60 分钟”、“6 小时”、“12 小时”和“24 小时”。在上一个备份操作成功完成以后，此时间间隔就会开始倒计时，同时会启动新的备份。建议使用“高级存储”帐户，因为高级存储的吞吐量更大，关于如何建立“高级存储”，请参阅[高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage-preview-portal)，同样地， Windows Azure 中国目前只支持用 PowerShell 管理高级存储。

>[AZURE.NOTE]AOF 在高级级别预览期间不可用，但缓存团队会争取尽快增加此功能。有关 RDB 和 AOF 以及各自优势的详细信息，请参阅 [Redis 暂留](http://redis.io/topics/persistence)。

创建完缓存后，一旦备份频率间隔时间已过，则会启动第一次备份。

## 保留常见问题

以下列表包含有关 Azure Redis 缓存保留常见问题的解答。

## 能否在此前已创建的缓存的基础上启用保留？

在预览期间，你只能在高级缓存的创建过程中启用和配置保留。在公共预览期间，不允许从基本/标准版扩展到高级版，但很快就会允许。

## 能否在创建缓存后更改备份频率？

在预览期间，你只能在缓存创建过程中配置保留。若要更改保留配置，请先删除缓存，然后使用所需保留配置创建新的缓存。这是预览版存在的限制，很快会推出相应的支持。

## 为何我的备份频率为 60 分钟，而两次备份的间隔却超过 60 分钟？

在上一次备份过程成功完成之前，本次备份不会开始，其频率所对应的时间间隔也不会开始计算。如果备份频率为 60 分钟，而备份过程需要 15 分钟才能成功完成，则在上一次备份开始以后，要再过 75 分钟才会开始下一次备份。

## 进行新备份以后，旧备份会发生什么情况

除最新备份外的所有备份都会自动删除。这种删除可能不会即刻发生，但旧备份是不会无限期保留下去的。

## 后续步骤
了解如何使用更多的高级版缓存功能。

-	[如何为高级 Azure Redis 缓存配置群集功能](/documentation/articles/cache-how-to-premium-clustering)
-	[如何为高级 Azure Redis 缓存配置虚拟网络支持](/documentation/articles/cache-how-to-premium-vnet)
  
<!-- IMAGES -->

[redis-cache-new-cache-menu]: ./media/cache-how-to-premium-persistence/redis-cache-new-cache-menu.png

[redis-cache-premium-pricing-tier]: ./media/cache-how-to-premium-persistence/redis-cache-premium-pricing-tier.png

[redis-cache-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-persistence.png

[redis-cache-persistence-selected]: ./media/cache-how-to-premium-persistence/redis-cache-persistence-selected.png

<!---HONumber=79-->