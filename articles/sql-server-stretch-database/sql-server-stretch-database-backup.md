<properties
    pageTitle="备份启用了延伸的数据库 | Azure"
    description="了解如何备份启用了延伸的数据库。"
    services="sql-server-stretch-database"
    documentationcenter=""
    author="Antvgski"
    manager="johnmac"
    editor="douglasl" />
<tags
    ms.assetid="a196f858-ef8f-47b5-b9db-bb7db98d48bd"
    ms.service="sql-server-stretch-database"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/23/2017"
    wacn.date="03/24/2017"
    ms.author="douglasl" />  


# 备份启用了延伸的数据库
数据库备用有助于从多种类型的故障、错误和灾难中恢复。

* 因此必须备份启用了延伸的 SQL Server 数据库。
* Azure 会自动备份 Stretch Database 从 SQL Server 迁移到 Azure 的远程数据。

> [AZURE.NOTE]
> 备份只是高可用性和业务连续性完整解决方案的一个部分。有关高可用性的详细信息，请参阅[高可用性解决方案](https://msdn.microsoft.com/zh-cn/library/ms190202.aspx)。
> 
> 

## 备份 SQL Server 数据
若要备份启用了延伸的 SQL Server 数据库，可继续使用当前所用的 SQL Server 备份方法。有关详细信息，请参阅 [SQL Server 数据库的备份和还原](https://msdn.microsoft.com/zh-cn/library/ms187048.aspx)。

对于启用了延伸的 SQL Server 数据库，其备份仅包括本地数据和备份运行时可进行迁移的合格数据。（合格数据是指尚未迁移，但将根据表的迁移设置迁移到 Azure 的数据。） 这称为**浅**备份，且不包括已迁移至 Azure 的数据。

## 备份远程 Azure 数据
Azure 会自动备份 Stretch Database 从 SQL Server 迁移到 Azure 的远程数据。

### Azure 通过自动备份降低了数据丢失的风险
Azure 上的 SQL Server Stretch Database 服务至少每 8 小时拍摄一次自动存储快照，以保护远程数据库。每张快照可保留 7 天，为你提供一段时间范围内可用的还原点。

### Azure 通过异地冗余降低了数据丢失的风险
Azure 数据库备份存储在异地冗余的 Azure 存储空间 (RA-GRS) 中，因此默认为异地冗余。异地冗余存储将数据复制到距主要区域数百英里以外的次要区域。在主要区域和次要区域，分别跨不同的容错域和升级域复制数据三次。即使某区域整个中断或发生灾难，造成某个 Azure 区域不可用，这仍可确保你的数据持久可用。

### <a name="stretchRPO"></a>Stretch Database 通过暂时保留迁移后的行来降低数据丢失的风险
在 Stretch Database 将符合条件的行从 SQL Server 迁移至 Azure 后，它可将这些行保留在临时表中至少 8 小时。如果还原 Azure 数据库的备份，Stretch Database 会使用临时表中保存的行来协调 SQL Server 和 Azure 数据库。

在还原 Azure 数据的备份后，必须运行存储过程 [sys.sp\_rda\_reauthorize\_db](https://msdn.microsoft.com/zh-cn/library/mt131016.aspx) 才可将启用了延伸的 SQL Server 数据库重新连接到远程 Azure 数据库。在运行 **sys.sp\_rda\_reauthorize\_db** 时，Stretch Database 会自动协调 SQL Server 和 Azure 数据库。

若要增加 Stretch Database 在临时表中暂时保留已迁移数据的小时数，请运行存储过程 [sys.sp\_rda\_set\_rpo\_duration](https://msdn.microsoft.com/zh-cn/library/mt707766.aspx) 并将小时数指定为大于 8 小时。若要决定要保留的数据量，请考虑以下因素：

* Azure 自动备份的频率（至少每 8 小时备份一次）。
* 出问题后识别问题和决定还原备份所需的时间。
* Azure 还原操作的持续时间。

> [AZURE.NOTE]
> 增加 Stretch Database 在临时表中暂时保留的数据量可增加 SQL Server 上所需的空间量。
> 
> 

若要查看 Stretch Database 当前在临时表中暂时保留的数据小时数，请运行存储过程 [sys.sp\_rda\_get\_rpo\_duration](https://msdn.microsoft.com/zh-cn/library/mt707767.aspx)。

## 另请参阅
[Stretch Database 的管理和故障排除](/documentation/articles/sql-server-stretch-database-manage/)

[sys.sp\_rda\_reauthorize\_db (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt131016.aspx)

[SQL Server 数据库的备份和还原](https://msdn.microsoft.com/zh-cn/library/ms187048.aspx)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description:update meta properties;wording update-->