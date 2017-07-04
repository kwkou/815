<properties
    pageTitle="为 Stretch Database 启用透明数据加密 - Azure | Azure"
    description="为 Azure 上的 SQL Server Stretch Database 启用透明数据加密 (TDE)"
    services="sql-server-stretch-database"
    documentationcenter=""
    author="douglaslMS"
    manager="barbkess"
    editor="" />
<tags
    ms.assetid="a44ed8f5-b416-4c41-9b1e-b7271f10bdc3"
    ms.service="sql-server-stretch-database"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="06/14/2016"
    wacn.date="03/24/2017"
    ms.author="douglasl" />  


# 为 Azure 上的 Stretch Database 启用透明数据加密 (TDE)
> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-server-stretch-database-encryption-tde/)
- [TSQL](/documentation/articles/sql-server-stretch-database-tde-tsql/)

透明数据加密 (TDE) 无需更改应用程序，即可对静止的数据库、关联的备份和事务日志执行实时加密和解密，帮助防止恶意活动的威胁。

TDE 使用称为数据库加密密钥的对称密钥来加密整个数据库的存储。数据库加密密钥由内置服务器证书保护。内置服务器证书对每个 Azure 服务器都是唯一的。Microsoft 每隔 90 天自动轮换这些证书至少一次。有关 TDE 的一般描述，请参阅[透明数据加密 (TDE)]。

##启用加密

对于存储从启用延伸的 SQL Server 数据库迁移的数据的 Azure 数据库，若要启用 TDE，请执行以下操作：

1. 在 [Azure 门户](https://portal.azure.cn)中打开数据库
2. 在数据库边栏选项卡中，单击“设置”按钮
3. 选择“透明数据加密”选项 
    ![][1]
4. 选择“打开”设置，请然后选择“保存”
    ![][2]


##禁用加密

对于存储从启用延伸的 SQL Server 数据库迁移的数据的 Azure 数据库，若要禁用 TDE，请执行以下操作：

1. 在 [Azure 门户](https://portal.azure.cn)中打开数据库
2. 在数据库边栏选项卡中，单击“设置”按钮
3. 选择“透明数据加密”选项
4. 选择“关闭”设置，请然后选择“保存”




<!--Anchors-->

[透明数据加密 (TDE)]: https://msdn.microsoft.com/zh-cn/library/bb934049.aspx


<!--Image references-->
[1]: ./media/sql-server-stretch-database-encryption-tde/stretchtde1.png
[2]: ./media/sql-server-stretch-database-encryption-tde/stretchtde2.png


<!--Link references-->

<!---HONumber=Mooncake_0320_2017-->