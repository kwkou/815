<properties
   pageTitle="SQL 数据仓库（门户）中的透明数据加密 | Azure"
   description="SQL 数据仓库中的透明数据加密 (TDE)"
   services="sql-data-warehouse"
   documentationCenter=""
   authors="ronortloff"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.workload="data-management"
   ms.tgt_pltfrm="na"
   ms.devlang="na"
   ms.topic="article"
   ms.date="10/31/2016" 
   wacn.date="12/12/2016"
   ms.author="rortloff;barbkess;sonyama"/>  


# SQL 数据仓库中的透明数据加密 (TDE) 入门

> [AZURE.SELECTOR]
- [安全性概述](/documentation/articles/sql-data-warehouse-overview-manage-security/)
- [身份验证](/documentation/articles/sql-data-warehouse-authentication/)
- [加密（门户）](/documentation/articles/sql-data-warehouse-encryption-tde/)
- [加密 (T-SQL)](/documentation/articles/sql-data-warehouse-encryption-tde-tsql/)

## 所需的权限
若要启用透明数据加密 (TDE)，用户必须是管理员或 dbmanager 角色的成员。

## 启用加密
若要为 SQL 数据仓库启用 TDE，请遵循以下步骤：

1. 在 [Azure 门户预览](https://portal.azure.cn/)中打开数据库
2. 在数据库边栏选项卡中，单击“设置”按钮
3. 选择“透明数据加密”选项 
![][1]
4. 选择“打开”设置 
![][2]
5. 选择“保存” 
![][3]  

## 禁用加密
若要为 SQL 数据仓库禁用 TDE，请遵循以下步骤：

1. 在 [Azure 门户预览](https://portal.azure.cn/)中打开数据库
2. 在数据库边栏选项卡中，单击“设置”按钮
3. 选择“透明数据加密”选项 
![][1]
4. 选择“关闭”设置 
![][4]
5. 选择“保存” 
![][5]  

##加密 DMV

可使用下列 DMV 来确认加密：

- [sys.databases]
- [sys.dm\_pdw\_nodes\_database\_encryption\_keys]

<!--MSDN references-->

[Transparent Data Encryption (TDE)]: https://msdn.microsoft.com/zh-cn/library/bb934049.aspx
[sys.databases]: http://msdn.microsoft.com/zh-cn/library/ms178534.aspx
[sys.dm\_pdw\_nodes\_database\_encryption\_keys]: https://msdn.microsoft.com/zh-cn/library/mt203922.aspx

<!--Image references-->
[1]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings.png
[2]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-on.png
[3]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-save.png
[4]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-off.png
[5]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-save2.png

<!--Link references-->

<!---HONumber=Mooncake_1205_2016-->