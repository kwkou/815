<properties
	pageTitle="排查 Azure SQL 数据库权限和访问问题"
	description="排查常见权限、访问、用户和登录问题的快速步骤"
	services="sql-database"
	documentationCenter=""
	authors="v-shysun"
	manager="msmets"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="03/28/2016"
	wacn.date="06/14/2016"/>

#排查常见的 Azure SQL 数据库权限和访问问题
使用本主题了解授予和删除对 Azure SQL 数据库的访问权限的快速步骤。如需更完整的信息，请参阅：

- [在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins/)
- [保护 SQL 数据库](/documentation/articles/sql-database-security/)
- [SQL Server 数据库引擎和 Azure SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)

##更改逻辑服务器的管理密码
- 在 [Azure 门户](https://manage.windowsazure.cn)中，单击“SQL Server”，从列表中选择服务器，然后单击“重置密码”。
##帮助确保只允许经过授权的 IP 地址访问服务器

##在用户数据库中创建包含的数据库用户
- 使用 [CREATE USER](https://msdn.microsoft.com/zh-cn/library/ms173463.aspx) 语句，并参阅[包含的数据库用户 - 使你的数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)。

## 使用 Azure Active Directory 对包含的数据库用户进行身份验证
- 请参阅[使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)。

## 在虚拟 master 数据库中为高权限用户创建其他登录名
- 使用 [CREATE LOGIN](https://msdn.microsoft.com/zh-cn/library/ms189751.aspx) 语句，并参阅[管理 Azure SQL 数据库中的数据库和登录名](/documentation/articles/sql-database-manage-logins/)的“管理登录名”部分以了解详细信息。

<!---HONumber=Mooncake_0606_2016-->
