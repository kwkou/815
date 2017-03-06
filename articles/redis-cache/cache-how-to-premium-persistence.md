<properties
    pageTitle="如何为高级 Azure Redis 缓存配置数据暂留"
    description="了解如何为高级层的 Azure Redis 缓存实例配置和管理数据暂留"
    services="redis-cache"
    documentationcenter=""
    author="steved0x"
    manager="douge"
    editor="" />
<tags
    ms.assetid="b01cf279-60a0-4711-8c5f-af22d9540d38"
    ms.service="cache"
    ms.workload="tbd"
    ms.tgt_pltfrm="cache-redis"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/09/2017"
    wacn.date="03/03/2017"
    ms.author="sdanie" />

# 如何为高级 Azure Redis 缓存配置数据暂留
Azure Redis 缓存具有不同的缓存产品（包括群集、持久性和虚拟网络支持等高级层功能），使缓存大小和功能的选择更加灵活。本文介绍如何配置高级 Azure Redis 缓存实例中的暂留。

有关其他高级缓存功能的信息，请参阅 [Azure Redis 缓存高级层简介](/documentation/articles/cache-premium-tier-intro/)。

## 什么是数据暂留？
Redis 暂留可让你保留存储在 Redis 中的数据。还可以获取快照并备份数据，以便在出现硬件故障时进行加载。这相对于基本层或标准层是一项巨大优势，因为基本层或标准层将所有数据存储在内存中，在出现故障的情况下，如果缓存节点停机，则可能导致数据丢失。

Azure Redis 缓存使用 [RDB 模型](http://redis.io/topics/persistence)提供的 Redis 持久性，允许将数据存储在 Azure 存储帐户中。配置暂留以后，Azure Redis 缓存会按照可配置的备份频率，将 Redis 缓存的快照以 Redis 二进制格式暂留在磁盘上。如果发生了灾难性事件，导致主缓存和副缓存都无法使用，则会使用最新快照重新构造缓存。

可以在创建缓存过程中通过“新建 Redis 缓存”边栏选项卡配置持久性；另外还可以在现有高级缓存的“资源”菜单上进行这项配置。

[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-premium-create.md)]

一旦选中某个高级定价层，则请单击“Redis 暂留”。

![Redis 暂留][redis-cache-persistence]  


以下部分中的步骤说明如何在新的高级缓存上配置 Redis 暂留。配置 Redis 持久性后，单击“创建”以创建具有 Redis 持久性的新高级版缓存。

## <a name="configure-redis-persistence"></a>配置 Redis 持久性
在“Redis 数据持久性”边栏选项卡上配置 Redis 持久性。对于新缓存，可以按前一部分中所述，在创建缓存过程中访问此边栏选项卡。对于现有缓存，可以从缓存的“资源”菜单访问“Redis 数据持久性”边栏选项卡。

![Redis 设置][redis-cache-settings]  


若要启用 Redis 持久性，请单击“已启用”来启用 RDB（Redis 数据库）备份。若要在以前启用的高级缓存上禁用 Redis 持久性，请单击“禁用”。

若要配置备份间隔，请选择下拉列表中的“备份频率”。选项包括“15 分钟”、“30 分钟”、“60 分钟”、“6 小时”、“12 小时”和“24 小时”。在上一个备份操作成功完成以后，此时间间隔就会开始倒计时，同时会启动新的备份。

单击“存储帐户”以选择要使用的存储帐户，然后从“存储密钥”下拉列表中选择要使用的“主密钥”或“辅助密钥”。必须选择与缓存处于相同区域的存储帐户，建议选择“高级存储”帐户，因为高级存储的吞吐量较高。

> [AZURE.IMPORTANT]
如果重新生成了持久性帐户的存储密钥，必须从“存储密钥”下拉列表中重新配置所需的密钥。
> 
> 

![Redis 暂留][redis-cache-persistence-selected]  


单击“确定”可保存暂留配置。

一旦备份频率间隔时间已过，则会启动下一次备份（或新缓存的首次备份）。

## 暂留常见问题
以下列表包含有关 Azure Redis 缓存暂留常见问题的解答。

* [能否在此前已创建的缓存的基础上启用保留？](#can-i-enable-persistence-on-a-previously-created-cache)
* [能否在创建缓存后更改备份频率？](#can-i-change-the-backup-frequency-after-i-create-the-cache)
* [为何我的备份频率为 60 分钟，而两次备份的间隔却超过 60 分钟？](#why-if-i-have-a-backup-frequency-of-60-minutes-there-is-more-than-60-minutes-between-backups)
* [进行新备份以后，旧备份会发生什么情况？](#what-happens-to-the-old-backups-when-a-new-backup-is-made)
* [如果我缩放到不同大小并还原了缩放操作之前生成的备份，会发生什么情况？](#what-happens-if-i-have-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation)

### <a name="can-i-enable-persistence-on-a-previously-created-cache"></a> 能否在以前创建的缓存上启用持久性？
是的，可以在创建缓存时或者在现有高级缓存上配置 Redis 暂留。

### <a name="can-i-change-the-backup-frequency-after-i-create-the-cache"></a> 能否在创建缓存后更改备份频率？
是的，可以在“Redis 数据持久性”边栏选项卡上更改备份频率。有关说明，请参阅[配置 Redis 持久性](#configure-redis-persistence)。

### <a name="why-if-i-have-a-backup-frequency-of-60-minutes-there-is-more-than-60-minutes-between-backups"></a> 为何我的备份频率为 60 分钟，而两次备份的间隔却超过 60 分钟？
在上一次备份过程成功完成之前，本次备份不会开始，其频率所对应的时间间隔也不会开始计算。如果备份频率为 60 分钟，而备份过程需要 15 分钟才能成功完成，则在上一次备份开始以后，要再过 75 分钟才会开始下一次备份。

### <a name="what-happens-to-the-old-backups-when-a-new-backup-is-made"></a> 进行新备份以后，旧备份会发生什么情况？
除最新备份外的所有备份都会自动删除。这种删除可能不会即刻发生，但旧备份是不会无限期保留下去的。

### <a name="what-happens-if-i-have-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation"></a> 如果我缩放到不同大小并还原了缩放操作之前生成的备份，会发生什么情况？
* 如果缩放到更大的大小，则没有任何影响。
* 如果缩放到更小的大小，并且自定义[数据库](/documentation/articles/cache-configure/#databases)设置大于新大小的[数据库限制](/documentation/articles/cache-configure/#databases)，则不会还原这些数据库中的数据。有关详细信息，请参阅[在缩放过程中，自定义数据库设置是否会受影响？](/documentation/articles/cache-how-to-scale/#is-my-custom-databases-setting-affected-during-scaling)
* 如果缩放到更小的大小，并且更小的大小空间不足，无法容纳上次备份的所有数据，则在还原过程中，通常会使用 [allkeys-lru](http://redis.io/topics/lru-cache) 逐出策略逐出密钥。

## 后续步骤
了解如何使用更多的高级缓存功能。

* [Azure Redis 缓存高级层简介](/documentation/articles/cache-premium-tier-intro/)

<!-- IMAGES -->


[redis-cache-new-cache-menu]: ./media/cache-how-to-premium-persistence/redis-cache-new-cache-menu.png

[redis-cache-premium-pricing-tier]: ./media/cache-how-to-premium-persistence/redis-cache-premium-pricing-tier.png

[redis-cache-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-persistence.png

[redis-cache-persistence-selected]: ./media/cache-how-to-premium-persistence/redis-cache-persistence-selected.png

[redis-cache-settings]: ./media/cache-how-to-premium-persistence/redis-cache-settings.png

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: wording update and move "how to create premium Redis Cache" to an include file-->