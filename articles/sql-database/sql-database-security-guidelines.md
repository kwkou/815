<properties
   pageTitle="Azure SQL 数据库安全指导原则和限制 | Azure"
   description="了解与安全性相关的 Azure SQL 数据库指导原则和限制。"
   services="sql-database"
   documentationCenter=""
   authors="BYHAM"
   manager="jhubbard"
   editor=""
   tags=""/>

<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management"
   ms.date="08/02/2016"
   wacn.date="09/28/2016"
   ms.author="rickbyh"/>

# Azure SQL 数据库安全指导原则和限制

本主题介绍与安全性相关的 Azure SQL 数据库指导原则和限制。在管理你的 Azure SQL 数据库的安全性时应考虑以下几点：

## 虚拟 master 数据库的访问权限

通常只有管理员需要 master 数据库的访问权限。对每个用户数据库的例程访问应通过数据库用户进行，这些用户创建于每个数据库且不包含管理员。使用包含的数据库用户时，不需要在 master 数据库中创建登录名。有关详细信息，请参阅[包含数据库用户 - 使你的数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)。


## 防火墙

作用域为整个 Azure SQL Server 的 SQL Server 防火墙通常通过门户进行配置，且应该仅承认管理员使用的 IP 地址。连接到用户数据库，然后使用 [sp\_set\_database\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270010.aspx) Transact SQL 语句创建数据库范围的防火墙规则，此规则可打开各个数据库必要的 IP 地址范围。

只能通过 TCP 端口 1433 使用 Azure SQL 数据库服务。若要从计算机访问 SQL 数据库，请确保客户端计算机防火墙允许 TCP 端口 1433 上的传出 TCP 通信。如果其他应用程序不需要，则阻止 TCP 端口 1433 上的入站连接。

在连接过程中，来自 Azure 虚拟机的连接将重定向到每个辅助角色特有的不同 IP 地址和端口。该端口号在 11000 到 11999 的范围内。有关 TCP 端口的详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)。


## 身份验证

尽可能使用 Active Directory 身份验证（集成安全性）。有关配置 AD 身份验证的信息，请参阅[通过使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)和 SQL Server 联机丛书中的[选择身份验证模式](https://msdn.microsoft.com/zh-cn/library/ms144284.aspx)。

使用包含的数据库用户可提高可缩放性。有关详细信息，请参阅[包含的数据库用户 - 使你的数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)、[CREATE USER (Transact-SQL)](https://technet.microsoft.com/zh-cn/library/ms173463.aspx) 和[包含的数据库](https://technet.microsoft.com/zh-cn/library/ff929071.aspx)。

数据库引擎将关闭空闲超过 30 分钟的连接。该连接必须重新登录才可供使用。

连续与 SQL 数据库建立活动连接需要至少每隔 10 小时重新授权一次（由数据库引擎执行授权）。数据库引擎将尝试使用最初提交的密码重新授权，且不需要用户输入。出于性能原因，在 SQL 数据库中重置密码时，将不会对连接重新进行身份验证，即使该连接由于连接池而重置。这一点与本地 SQL Server 的行为不同。如果自从最初授权后密码已更改，则必须终止该连接，并使用新密码建立新连接。具有 KILL DATABASE CONNECTION 权限的用户可以使用 [KILL](https://msdn.microsoft.com/zh-cn/library/ms173730.aspx) 命令显式终止与 SQL 数据库的连接。

## 登录名和用户

在 SQL 数据库中管理登录名和用户时，存在一些限制。


- 执行``CREATE/ALTER/DROP DATABASE`` 语句时，必须连接到 **master** 数据库。- 不能更改或删除 master 数据库中与服务器级别主体登录名相对应的数据库用户。
- 美国英语为服务器级别主体登录名的默认语言。
- 若要访问 **master** 数据库，每个登录名都必须映射到 **master** 数据库中的某一用户帐户。**master** 数据库不支持包含的数据库用户。
- 只有服务器级别主体登录名和 **dbmanager** 数据库中 **master** 角色的成员才有权执行 ``CREATE DATABASE`` 和 ``DROP DATABASE`` 语句。
- 执行 ``CREATE/ALTER/DROP LOGIN`` 语句时，必须连接到 master 数据库。但不建议使用登录名。改用包含的数据库用户。
- 要连接到用户数据库，必须在连接字符串中提供数据库的名称。
- 只有服务器级别主体登录名和 **loginmanager** 数据库中 **master** 角色的成员才有权执行 ``CREATE LOGIN``、``ALTER LOGIN`` 和 ``DROP LOGIN`` 语句。
- 在 ADO.NET 应用程序中执行 ``CREATE/ALTER/DROP LOGIN`` 和 ``CREATE/ALTER/DROP DATABASE`` 语句时，不允许使用参数化命令。有关详细信息，请参阅[命令和参数](https://msdn.microsoft.com/zh-cn/library/ms254953.aspx)。
- 在执行 ``CREATE/ALTER/DROP DATABASE`` 和 ``CREATE/ALTER/DROP LOGIN`` 语句时，上述每个语句都必须是 Transact-SQL 批处理中的唯一语句。否则将会出错。例如，以下 Transact-SQL 将检查该数据库是否存在。如果该数据库存在，则调用 ``DROP DATABASE`` 语句以便删除该数据库。因为 ``DROP DATABASE`` 语句不是该批处理中的唯一语句，所以执行此 Transact-SQL 将导致错误。


		IF EXISTS (SELECT [name]
		           FROM   [sys].[databases]
		           WHERE  [name] = N'database_name')
		     DROP DATABASE [database_name];
		GO


- 在使用 ``FOR/FROM LOGIN`` 选项执行 ``CREATE USER`` 语句时，该语句必须是 Transact-SQL 批处理中的唯一语句。
- 在使用 ``WITH LOGIN`` 选项执行 ``ALTER USER`` 语句时，该语句必须是 Transact-SQL 批处理中的唯一语句。
- 若要执行 ``CREATE/ALTER/DROP`` 操作，用户需要对数据库拥有 ``ALTER ANY USER`` 权限。
- 在数据库角色的所有者尝试向该数据库角色添加其他数据库用户或者从该数据库角色中删除其他数据库用户时，可能会发生以下错误：“此数据库中不存在用户或角色 'Name'”。 在用户对所有者不可见时，将会发生此错误。若要解决此问题，请向角色所有者授予对该用户的 ``VIEW DEFINITION`` 权限。

有关登录名和用户的详细信息，请参阅[在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins/)。


## 另请参阅

[Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)

[如何：配置防火墙设置（Azure SQL 数据库）](/documentation/articles/sql-database-configure-firewall-settings-powershell/)

[在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins/)

[SQL Server 数据库引擎和 Azure SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)

<!---HONumber=Mooncake_0919_2016-->