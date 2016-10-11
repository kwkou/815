<properties
   pageTitle="SQL 数据仓库中的透明数据加密 (TDE) 入门| Azure"
   description="SQL 数据仓库中的透明数据加密 (TDE) 入门"
   services="sql-data-warehouse"
   documentationCenter=""
   authors="ronortloff"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="06/07/2016" 
   wacn.date="10/10/2016"/>

# SQL 数据仓库中的透明数据加密 (TDE) 入门
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/sql-data-warehouse-encryption-tde/)
- [TSQL](/documentation/articles/sql-data-warehouse-encryption-tde-tsql/)

Azure SQL 数据仓库透明数据加密 (TDE) 无需更改应用程序，即可对静止的数据库、关联的备份和事务日志执行实时加密和解密，帮助防止恶意活动的威胁。

TDE 使用称为数据库加密密钥的对称密钥来加密整个数据库的存储。在 SQL 数据库中，数据库加密密钥由内置服务器证书保护。内置服务器证书对每个 SQL 数据库服务器都是唯一的。Microsoft 每隔 90 天自动轮换这些证书至少一次。SQL 数据仓库使用的加密算法为 AES-256。有关 TDE 的一般描述，请参阅[透明数据加密 (TDE)]。

##启用加密

若要为 SQL 数据仓库启用 TDE，请遵循以下步骤：

1. 在 [Azure 门户预览](https://portal.azure.cn/)中打开数据库
2. 在数据库边栏选项卡中，单击“设置”按钮
3. 选择“透明数据加密”选项 ![][1]
4. 选择“打开”设置 ![][2]
5. 选择“保存” ![][3]  

##禁用加密

若要为 SQL 数据仓库禁用 TDE，请遵循以下步骤：

1. 在 [Azure 门户预览](https://portal.azure.cn/)中打开数据库
2. 在数据库边栏选项卡中，单击“设置”按钮
3. 选择“透明数据加密”选项 ![][1]
4. 选择“关闭”设置 ![][4]
5. 选择“保存” ![][5]  

##加密 DMV

可使用下列 DMV 来确认加密：

- [sys.databases]
- [sys.dm\_pdw\_nodes\_database\_encryption\_keys]

<!--MSDN references-->
[透明数据加密 (TDE)]: https://msdn.microsoft.com/zh-cn/library/bb934049.aspx
[sys.databases]: http://msdn.microsoft.com/zh-cn/library/ms178534.aspx
[sys.dm\_pdw\_nodes\_database\_encryption\_keys]: https://msdn.microsoft.com/zh-cn/library/mt203922.aspx

<!--Image references-->
[1]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings.png
[2]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-on.png
[3]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-save.png
[4]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-off.png
[5]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-save2.png

<!--Link references-->

<!---HONumber=Mooncake_0627_2016-->