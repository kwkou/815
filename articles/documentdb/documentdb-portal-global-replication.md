<properties
	pageTitle="DocumentDB 全局数据库复制 | Azure"
	description="了解如何通过 Azure 门户预览管理 DocumentDB 帐户的全局复制。"
	services="documentdb"
    keywords="全局数据库, 复制"
	documentationCenter=""
	authors="mimig1"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="documentdb"
	ms.date="08/15/2016"
	wacn.date="08/29/2016"/>  


# 如何使用 Azure 门户预览执行 DocumentDB 全局数据库复制

了解如何使用 Azure 门户预览复制多个区域中的数据，使 Azure DocumentDB 中的数据具有全局可用性。

有关 DocumentDB 中全局数据库复制工作原理的信息，请参阅 [Distribute data globally with DocumentDB](/documentation/articles/documentdb-distribute-data-globally)（使用 DocumentDB 全局分发数据）。有关以编程方式执行全局数据库复制的信息，请参阅 [Developing with multi-region DocumentDB accounts](/documentation/articles/documentdb-developing-with-multiple-regions/)（使用多区域 DocumentDB 帐户进行开发）。

> [AZURE.NOTE] DocumentDB 数据库全局分发功能已正式推出，所有新建的 DocumentDB 帐户将自动启用该功能。我们正在努力为所有现有帐户启用全局分发，但在此之前，如果你要为你的帐户启用全局分发，请[与支持部门联系](/support/contact/)，我们将会帮助你启用。

## <a id="addregion"></a>添加全局数据库区域

为数据库帐户选择默认的一致性级别后，可以关联一个或多个区域（具体取决于所选的默认一致性级别和全局分发需求）。

1. 在 [Azure 门户预览](https://portal.azure.cn/)的跳转栏中，单击“DocumentDB 帐户”。
2. 在“DocumentDB 帐户”边栏选项卡中，选择要修改的数据库帐户。
3. 在帐户边栏选项卡上的菜单中，单击“添加/删除区域”。
    ![在“DocumentDB 帐户”>“添加/删除区域”下添加区域][1]

5. 在“添加/删除区域”边栏选项卡中，选择要添加或删除的区域，然后单击“确定”。添加区域会产生费用，有关详细信息，请参阅[定价页](/pricing/details/documentdb/)或 [Distribute data globally with DocumentDB](/documentation/articles/documentdb-distribute-data-globally/)（使用 DocumentDB 全局分发数据）。

### 选择全局数据库区域

在配置两个或更多个区域时，根据区域对选择区域。

具体而言，在配置多个区域时，请确保从每个配对区域列中选择相同数目的区域（使用 +/-1 可更改为奇数/偶数）。

如果只为灾难恢复方案配置两个区域，请务必遵循本指南。如果要配置两个以上的区域，则遵循本指南是不错的做法，但只要某些选定的区域遵守这种配对，则就没有必要遵循本指南。

## <a id="next"></a>后续步骤

阅读 [Consistency levels in DocumentDB](/documentation/articles/documentdb-consistency-levels/)（DocumentDB 中的一致性级别），了解如何管理全局复制帐户的一致性。

有关 DocumentDB 中全局数据库复制工作原理的信息，请参阅 [Distribute data globally with DocumentDB](/documentation/articles/documentdb-distribute-data-globally/)（使用 DocumentDB 全局分发数据）。有关以编程方式复制多个区域中的数据的信息，请参阅 [Developing with multi-region DocumentDB accounts](/documentation/articles/documentdb-developing-with-multiple-regions/)（使用多区域 DocumentDB 帐户进行开发）。

<!--Image references-->
[1]: ./media/documentdb-portal-global-replication/documentdb-account-blade.png
[2]: ./media/documentdb-portal-global-replication/documentdb-add-region.png
[3]: ./media/documentdb-portal-global-replication/documentdb_change_write_region-1.png
[4]: ./media/documentdb-portal-global-replication/documentdb_change_write_region-2.png

<!--Reference style links - using these makes the source content way more readable than using inline links-->

[bcdr]: /documentation/articles/best-practices-availability-paired-regions/
[consistency]: /documentation/articles/documentdb-consistency-levels/
[azureregions]: https://azure.microsoft.com/zh-cn/regions/#services
[offers]: /pricing/details/documentdb/

<!---HONumber=Mooncake_0822_2016-->