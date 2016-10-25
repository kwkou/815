<properties
	pageTitle="如何执行管理任务，例如重置管理员密码 | Azure"
	description="介绍如何在 SQL 数据库中执行常见管理任务。例如，重置管理员密码，授予和删除访问权限。"
	services="sql-database"
	documentationCenter=""
	authors="v-shysun"
	manager="felixwu"
	editor=""
	keywords="重置管理员密码"/>  

<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/13/2016"
	wacn.date="10/17/2016"
	ms.author="v-shysun"/>  


# 如何在 Azure SQL 数据库中执行重置管理员密码等常见管理任务
使用本主题了解授予和删除对 Azure SQL 数据库的访问权限的快速步骤。如需更完整的信息，请参阅：

- [在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins/)
- [保护 SQL 数据库](/documentation/articles/sql-database-security/)
- [SQL Server 数据库引擎和 Azure SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)


[AZURE.INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## 重置逻辑服务器的管理员密码

- 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中，单击“SQL Server”，从列表中选择服务器，然后单击“重置密码”。

## 帮助确保只允许经过授权的 IP 地址访问服务器
- 请参阅[如何：在 SQL 数据库上配置防火墙设置](/documentation/articles/sql-database-configure-firewall-settings-powershell/)。

## 在用户数据库中创建包含的数据库用户
- 使用 [CREATE USER](https://msdn.microsoft.com/zh-cn/library/ms173463.aspx) 语句，并参阅[包含的数据库用户 - 使你的数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)。

## 使用 Azure Active Directory 对包含的数据库用户进行身份验证
- 请参阅[使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)。

## 在虚拟 master 数据库中为高权限用户创建其他登录名
-使用 [CREATE LOGIN](https://msdn.microsoft.com/zh-cn/library/ms189751.aspx) 语句，并参阅[管理 Azure SQL 数据库中的数据库和登录名](/documentation/articles/sql-database-manage-logins/)的“管理登录名”部分以了解详细信息。

<!---HONumber=Mooncake_1010_2016-->