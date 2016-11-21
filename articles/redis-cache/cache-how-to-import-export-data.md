<properties 
	pageTitle="在 Azure Redis 缓存中导入和导出数据 | Azure" 
	description="了解如何使用高级 Azure Redis 缓存实例从 blob 存储导入数据，以及将数据导出到 blob 存储" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="douge" 
	editor=""/>  


<tags 
	ms.service="cache" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="cache-redis" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/15/2016" 
	wacn.date="11/21/2016" 
	ms.author="sdanie"/>  


# 在 Azure Redis 缓存中导入和导出数据

导入/导出是一种 Azure Redis 缓存数据管理操作，可用于通过从高级缓存导入 Redis 缓存数据库 (RDB) 快照以及将 Redis 缓存数据库 (RDB) 快照导出到 Azure 存储帐户中的页 blob 来相应地将数据导入到 Azure Redis 缓存以及从 Azure Redis 缓存导出数据。导入/导出允许在不同 Azure Redis 缓存实例之间进行迁移，或者在使用缓存之前在缓存中填充数据。

本文提供使用 Azure Redis 缓存导入和导出数据的指南，并提供常见问题的解答。

>[AZURE.IMPORTANT] 导入/导出处于预览状态，仅适用于[高级层](/documentation/articles/cache-premium-tier-intro/)缓存。

## 导入

导入可用于从任何云或环境中运行的任何 Redis 服务器引入与 Redis 兼容的 RDB 文件，包括在 Linux、Windows 上运行的 Redis 或任何云提供程序（如 Amazon Web Services 等）。导入数据是使用预先填充的数据创建缓存的简单方式。在导入过程中，Azure Redis 缓存从 Azure 存储将 RDB 文件加载到内存中，然后再将密钥插入到缓存中。

>[AZURE.NOTE] 在开始导入操作之前，请确保 Redis 数据库 (RDB) 文件已上载到 Azure 存储空间的页 blob 中，并与 Azure Redis 缓存实例在同一区域和订阅中。有关详细信息，请参阅 [Get started with Azure Blob storage](/documentation/articles/storage-dotnet-how-to-use-blobs/)（Azure Blob 存储入门）。如果已使用 [Azure Redis 缓存导出](#export)功能导出 RDB 文件，则 RDB 文件已存储在页 blob 中并已准备好进行导入。

1. 若要导入一个或多个导出的缓存 blob，请在 Azure 门户预览中[浏览到你的缓存](/documentation/articles/cache-configure/#configure-redis-cache-settings)，然后在缓存实例的“设置”边栏选项卡中单击“导入数据”。

    ![导入数据][cache-import-data]  

2. 单击“选择 Blob”，然后选择包含要导入的数据的存储帐户。

    ![选择存储帐户][cache-import-choose-storage-account]

3. 单击包含要导入的数据的容器。

    ![选择容器][cache-import-choose-container]

4. 通过单击 blob 名称左侧的区域选择要导入的一个或多个 blob，然后单击“选择”。

    ![选择 blob][cache-import-choose-blobs]

5. 单击“导入”以开始导入过程。

    >[AZURE.IMPORTANT] 在导入过程中，缓存客户端无法访问该缓存，并且将删除该缓存中的任何现有数据。

    ![导入][cache-import-blobs]  

    可以通过关注 Azure 门户预览中的通知或通过查看[审核日志](/documentation/articles/cache-configure/#support-amp-troubleshooting-settings)中的事件，来监视导入操作的进度。

    ![导入进度][cache-import-data-import-complete]  



## <a name="export"></a>导出

使用导出可以将 Azure Redis 缓存中存储的数据导出到与 Redis 兼容的 RDB 文件。可以使用此功能将一个 Azure Redis 缓存实例中的数据移到另一个 Azure Redis 缓存实例或另一个 Redis 服务器。在导出过程中，将在托管 Azure Redis 缓存服务器实例的 VM 上创建临时文件，并将该文件上载到指定的存储帐户。导出操作完成后，无论状态为成功还是失败，都会删除临时文件。

1. 若要将缓存的当前内容导出到存储，请在 Azure 门户预览中[浏览到你的缓存](/documentation/articles/cache-configure/#configure-redis-cache-settings)，然后在缓存实例的“设置”边栏选项卡中单击“导出数据”。

    ![选择存储容器][cache-export-data-choose-storage-container]  

2. 单击“选择存储容器”并选择所需的存储帐户。存储帐户必须与你的缓存在同一订阅和区域中。

    >[AZURE.IMPORTANT] 导入/导出适用于页 blob，经典存储帐户和 ARM 存储帐户都支持页 blob，但目前 [Blob 存储帐户](/documentation/articles/storage-blob-storage-tiers/#blob-storage-accounts)不支持页 blob。

    ![存储帐户][cache-export-data-choose-account]

3. 选择所需的 blob 容器，然后单击“选择”。若要使用新容器，请单击“添加容器”以先添加该容器，然后再从列表中选择该容器。

    ![选择存储容器][cache-export-data-container]

4. 键入 **Blob 名称前缀**，然后单击“导出”以开始导出过程。blob 名称前缀用于作为此导出操作生成的文件名称的前缀。

    ![导出][cache-export-data]  

    可以通过关注 Azure 门户预览中的通知或通过查看[审核日志](/documentation/articles/cache-configure/#support-amp-troubleshooting-settings)中的事件，来监视导出操作的进度。

    ![][cache-export-data-export-complete]  

    在导出过程中，缓存仍可供使用。


## 导入/导出常见问题

本部分包含有关导入/导出功能的常见问题。

-	[哪些定价层可以使用导入/导出？](#what-pricing-tiers-can-use-importexport)
-	[能否从任何 Redis 服务器导入数据？](#can-i-import-data-from-any-redis-server)
-	[在导入/导出操作期间能否使用我的缓存？](#will-my-cache-be-available-during-an-importexport-operation)
-	[能否对 Redis 群集使用导入/导出？](#can-i-use-importexport-with-redis-cluster)
-	[导入/导出如何可用于自定义数据库设置？](#how-does-importexport-work-with-a-custom-databases-setting)
-	[导入/导出与 Redis 持久性有何区别？](#how-is-importexport-different-from-redis-persistence)
-	[能否使用 PowerShell、CLI 或其他管理客户端自动执行导入/导出？](#can-i-automate-importexport-using-powershell-cli-or-other-management-clients)
-	[我在导入/导出操作期间收到超时错误。它意味着什么？](#i-received-a-timeout-error-during-my-importexport-operation.-what-does-it-mean)
-	[在将我的数据导出到 Azure Blob 存储时收到错误。发生了什么情况？](#i-got-an-error-when-exporting-my-data-to-azure-blob-storage.-what-happened)


### <a name="what-pricing-tiers-can-use-importexport"></a>哪些定价层可以使用导入/导出？

导入/导出仅在高级定价层中可用。

### <a name="can-i-import-data-from-any-redis-server"></a>能否从任何 Redis 服务器导入数据？

能，除了导入从 Azure Redis 缓存实例导出的数据外，还可以从任何云或环境中运行的任何 Redis 服务器导入 RDB 文件，如 Linux、Windows 或云提供程序（如 Amazon Web Services）。为此，请从所需的 Redis 服务器将 RDB 文件上载到 Azure 存储帐户中的页 blob，然后将其导入到高级 Azure Redis 缓存实例中。例如，你可能想要从生产缓存导出数据，然后将其导入到用作过渡环境的一部分的缓存，用于测试或迁移。

### <a name="will-my-cache-be-available-during-an-importexport-operation"></a>在导入/导出操作期间能否使用我的缓存？

-	**导出** - 缓存保持可用，你可以在导出操作过程中继续使用缓存。
-	**导入** - 在导入操作开始时，缓存即变为不可用，在导入操作完成后，缓存变为可供使用。


### <a name="can-i-use-importexport-with-redis-cluster"></a>能否对 Redis 群集使用导入/导出？

能，并且你可以在群集缓存和非群集缓存之间导入/导出。由于 Redis 群集[仅支持数据库 0](/documentation/articles/cache-how-to-premium-clustering/#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)，因此将不会导入 0 以外的数据库中的任何数据。导入群集缓存数据时，密钥会在群集的分片之间重新分发。

### <a name="how-does-importexport-work-with-a-custom-databases-setting"></a>如何在自定义数据库设置中使用导入/导出？

某些定价层具有不同的[数据库限制](/documentation/articles/cache-configure/#databases)，因此，如果在缓存创建过程中为 `databases` 设置配置了自定义值，则在导入时需注意一些注意事项。

-	当导入到的定价层的 `databases` 限制低于导出层的相应限制时：
	-	如果你使用的是默认 `databases` 数（对于所有定价层来说为 16），则不会丢失数据。
	-	如果你使用的是在你要导入到的层的限制内的自定义 `databases` 数，则不会丢失数据。
	-	如果你导出的数据包含超出新层限制的数据库中的数据，则将不会导入这些更高版本数据库中的数据。

### <a name="how-is-importexport-different-from-redis-persistence"></a>导入/导出与 Redis 持久性有何区别？

Azure Redis 缓存持久性可让你将 Redis 中存储的数据持久保存到 Azure 存储空间。配置暂留以后，Azure Redis 缓存会按照可配置的备份频率，将 Redis 缓存的快照以 Redis 二进制格式暂留在磁盘上。如果发生了灾难性事件，导致主缓存和副本缓存都无法使用，则会使用最新快照自动还原缓存数据。有关详细信息，请参阅 [How to configure data persistence for a Premium Azure Redis Cache](/documentation/articles/cache-how-to-premium-persistence/)（如何为高级 Azure Redis 缓存配置数据持久性）。

使用导入/导出，可以将数据引入 Azure Redis 缓存或从 Azure Redis 缓存中导出。它不使用 Redis 持久性配置备份和还原。


### <a name="can-i-automate-importexport-using-powershell-cli-or-other-management-clients"></a>能否使用 PowerShell、CLI 或其他管理客户端自动执行导入/导出？

能。如需 PowerShell 说明，请参阅[导入 Redis 缓存](/documentation/articles/cache-howto-manage-redis-cache-powershell/#to-import-a-redis-cache)和[导出 Redis 缓存](/documentation/articles/cache-howto-manage-redis-cache-powershell/#to-export-a-redis-cache)。



### <a name="i-received-a-timeout-error-during-my-importexport-operation.-what-does-it-mean"></a>导入/导出操作期间出现超时错误。它意味着什么？

如果你在发起操作前停留在“导入数据”或“导出数据”边栏选项卡的时间超过 15 分钟，则将收到如下错误。

    将数据导入缓存“contoso55”的请求失败，状态为“错误”，错误“无法使用提供的 SAS URI 之一”可能由以下原因所致：SAS 令牌结束时间 (se) 必须至少距离现在 1 小时；如果已指定开始时间 (st)，则开始时间必须在 15 分钟之前。

若要解决此问题，请在经过 15 分钟前发起导入或导出操作。

### <a name="i-got-an-error-when-exporting-my-data-to-azure-blob-storage.-what-happened"></a>将数据导出到 Azure Blob 存储时遇到错误。发生了什么情况？

导入/导出仅适用于以页 blob 形式存储的 RDB 文件。目前，不支持其他 blob 类型，包括带有热层和冷层的 blob 存储帐户。


## 后续步骤
了解如何使用更多的高级版缓存功能。

-	[Azure Redis 缓存高级层简介](/documentation/articles/cache-premium-tier-intro/)

  
<!-- IMAGES -->

[cache-settings-import-export-menu]: ./media/cache-how-to-import-export-data/cache-settings-import-export-menu.png
[cache-export-data-choose-account]: ./media/cache-how-to-import-export-data/cache-export-data-choose-account.png
[cache-export-data-choose-storage-container]: ./media/cache-how-to-import-export-data/cache-export-data-choose-storage-container.png
[cache-export-data-container]: ./media/cache-how-to-import-export-data/cache-export-data-container.png
[cache-export-data-export-complete]: ./media/cache-how-to-import-export-data/cache-export-data-export-complete.png
[cache-export-data]: ./media/cache-how-to-import-export-data/cache-export-data.png
[cache-import-data]: ./media/cache-how-to-import-export-data/cache-import-data.png
[cache-import-choose-storage-account]: ./media/cache-how-to-import-export-data/cache-import-choose-storage-account.png
[cache-import-choose-container]: ./media/cache-how-to-import-export-data/cache-import-choose-container.png
[cache-import-choose-blobs]: ./media/cache-how-to-import-export-data/cache-import-choose-blobs.png
[cache-import-blobs]: ./media/cache-how-to-import-export-data/cache-import-blobs.png
[cache-import-data-import-complete]: ./media/cache-how-to-import-export-data/cache-import-data-import-complete.png

<!---HONumber=Mooncake_1114_2016-->