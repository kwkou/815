<properties 
	pageTitle="如何为高级 Azure Redis 缓存配置数据暂留" 
	description="了解如何为高级级别的 Azure Redis 缓存实例配置和管理数据暂留" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="cache-redis" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="08/09/2016" 
	wacn.date="09/05/2016" 
	ms.author="sdanie"/>

# 如何为高级 Azure Redis 缓存配置数据暂留

Azure Redis 缓存具有不同的缓存产品（包括新推出的高级层），使缓存大小和功能的选择更加灵活。

Azure Redis 缓存高级级别包括群集、暂留和虚拟网络支持。本文介绍如何配置高级 Azure Redis 缓存实例中的暂留。

有关其他高级缓存功能的信息，请参阅[如何配置高级 Azure Redis 缓存的群集功能](/documentation/articles/cache-how-to-premium-clustering/)和[如何配置高级 Azure Redis 缓存的虚拟网络支持](/documentation/articles/cache-how-to-premium-vnet/)。

## 什么是数据暂留？
Redis 暂留可让你保留存储在 Redis 中的数据。你还可获取快照并备份数据，以便在出现硬件故障时进行加载。这相对于基本级别或标准级别是一项巨大优势，因为基本级别或标准级别将所有数据存储在内存中，在出现故障的情况下，如果缓存节点停机，则可能导致数据丢失。

Azure Redis 缓存使用 [RDB 模型](http://redis.io/topics/persistence)提供的 Redis 持久性，允许将数据存储在 Azure 存储帐户中。配置暂留以后，Azure Redis 缓存会按照可配置的备份频率，将 Redis 缓存的快照以 Redis 二进制格式暂留在磁盘上。如果发生了灾难性事件，导致主缓存和副缓存都无法使用，则会使用最新快照重新构造缓存。

可以在创建缓存过程中通过“新建 Redis 缓存”边栏选项卡配置持久性；另外还可以在现有高级缓存的“设置”边栏选项卡上进行这项配置。

## 创建高级缓存

若要创建缓存并配置持久性，请登录到 [Azure 门户预览](https://portal.azure.cn)，然后单击“新建”>“数据 + 存储”>“Redis 缓存”。

![创建 Redis 缓存][redis-cache-new-cache-menu]  


若要配置暂留，请首先在“选择定价层”边栏选项卡中选择一个“高级”缓存。

![选择你的定价层][redis-cache-premium-pricing-tier]

一旦选中某个高级定价层，则请单击“Redis 暂留”。

![Redis 暂留][redis-cache-persistence]  


以下部分中的步骤说明如何在新的高级缓存上配置 Redis 持久性。配置 Redis 持久性后，单击“创建”以创建具有 Redis 持久性的新高级版缓存。

## <a name="configure-redis-persistence"></a>配置 Redis 持久性

在“Redis 数据持久性”边栏选项卡上配置 Redis 持久性。对于新缓存，可以按前一部分中所述，在创建缓存过程中访问此边栏选项卡。对于现有缓存，可以从缓存的“设置”边栏选项卡访问“Redis 数据持久性”边栏选项卡。

![Redis 设置][redis-cache-settings]

若要启用 Redis 持久性，请单击“已启用”来启用 RDB（Redis 数据库）备份。若要在以前启用的高级缓存上禁用 Redis 持久性，请单击“禁用”。

若要配置备份间隔，请选择下拉列表中的“备份频率”。选项包括“15 分钟”、“30 分钟”、“60 分钟”、“6 小时”、“12 小时”和“24 小时”。在上一个备份操作成功完成以后，此时间间隔就会开始倒计时，同时会启动新的备份。

单击“存储帐户”以选择要使用的存储帐户，然后从“存储密钥”下拉列表中选择要使用的“主密钥”或“辅助密钥”。你必须选择与缓存处于相同区域的存储帐户，建议选择“高级存储”帐户，因为高级存储的吞吐量较高。

>[AZURE.IMPORTANT] 如果重新生成持久性帐户的存储密钥，就必须从“存储密钥”下拉列表中重新选择所需的密钥。

![Redis 暂留][redis-cache-persistence-selected]  


单击“确定”可保存暂留配置。

一旦备份频率间隔时间已过，则会启动下一次备份（或新缓存的首次备份）。



## 保留常见问题

以下列表包含有关 Azure Redis 缓存保留常见问题的解答。

-	[能否在此前已创建的缓存的基础上启用保留？](#can-i-enable-persistence-on-a-previously-created-cache)
-	[能否在创建缓存后更改备份频率？](#can-i-change-the-backup-frequency-after-i-create-the-cache)
-	[为何我的备份频率为 60 分钟，而两次备份的间隔却超过 60 分钟？](#why-if-i-have-a-backup-frequency-of-60-minutes-there-is-more-than-60-minutes-between-backups)
-	[进行新备份以后，旧备份会发生什么情况？](#what-happens-to-the-old-backups-when-a-new-backup-is-made)
-	[如果我缩放到不同大小并还原了缩放操作之前生成的备份，会发生什么情况？](#what-happens-if-i-have-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation)

### <a name="can-i-enable-persistence-on-a-previously-created-cache"></a> 能否在此前已创建的缓存的基础上启用保留？

是的，可以在创建缓存时或者在现有高级缓存上配置 Redis 持久性。

### <a name="can-i-change-the-backup-frequency-after-i-create-the-cache"></a> 能否在创建缓存后更改备份频率？

是的，可以在“Redis 数据持久性”边栏选项卡上更改备份频率。有关说明，请参阅[配置 Redis 持久性](#configure-redis-persistence)。

### <a name="why-if-i-have-a-backup-frequency-of-60-minutes-there-is-more-than-60-minutes-between-backups"></a> 为何我的备份频率为 60 分钟，而两次备份的间隔却超过 60 分钟？

在上一次备份过程成功完成之前，本次备份不会开始，其频率所对应的时间间隔也不会开始计算。如果备份频率为 60 分钟，而备份过程需要 15 分钟才能成功完成，则在上一次备份开始以后，要再过 75 分钟才会开始下一次备份。

### <a name="what-happens-to-the-old-backups-when-a-new-backup-is-made"></a> 进行新备份以后，旧备份会发生什么情况？

除最新备份外的所有备份都会自动删除。这种删除可能不会即刻发生，但旧备份是不会无限期保留下去的。

### <a name="what-happens-if-i-have-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation"></a> 如果我缩放到不同大小并还原了缩放操作之前生成的备份，会发生什么情况？

-	如果缩放到更大的大小，则没有任何影响。
-	如果缩放到更小的大小，并且你的自定义[数据库](/documentation/articles/cache-configure/#databases)设置大于新大小的[数据库限制](/documentation/articles/cache-configure/#databases)，则不会还原这些数据库中的数据。
-	如果缩放到更小的大小，并且更小的大小空间不足，无法容纳上次备份的所有数据，则在还原过程中，通常会使用 [allkeys-lru](http://redis.io/topics/lru-cache) 逐出策略逐出密钥。

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

<!---HONumber=Mooncake_0829_2016-->