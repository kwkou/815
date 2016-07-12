<properties
   pageTitle="Azure SQL 数据库安全指导原则和限制 | Azure"
   description="了解与安全性相关的 Azure SQL 数据库指导原则和限制。"
   services="sql-database"
   documentationCenter=""
   authors="BYHAM"
   manager="jeffreyg"
   editor=""
   tags=""/>

<tags
   ms.service="sql-database"
   ms.date="02/16/2016"
   wacn.date="06/14/2016"/>

# Azure SQL 数据库安全指导原则和限制

本主题介绍与安全性相关的 Azure SQL 数据库指导原则和限制。在管理你的 Azure SQL 数据库的安全性时应考虑以下几点：

## 防火墙

只能通过 TCP 端口 1433 使用 Azure SQL 数据库服务。若要从计算机访问 SQL 数据库，请确保你的防火墙允许 TCP 端口 1433 上的传出 TCP 通信。在连接过程中，来自 Azure 虚拟机的连接将重定向到每个辅助角色特有的不同 IP 地址和端口。该端口号在 11000 到 11999 的范围内。

在你首次连接到 Azure SQL 数据库服务器之前，必须使用 [Azure 管理门户](https://manage.windowsazure.cn)来配置 Azure SQL 数据库防火墙。你将需要创建一个服务器级防火墙设置，该设置允许从你的计算机或 Azure 连接到 Azure SQL 数据库服务器。如果要控制对 Azure SQL 数据库服务器内某些数据库的访问，请为相应数据库创建数据库级防火墙规则。有关详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)。

## 连接加密和证书验证

在 SQL 数据库和你的应用程序之间的所有通信始终都要求加密 (SSL)。如果你的客户端应用程序未在连接时验证证书，则与 SQL 数据库的连接可能容易受到“中间人”攻击。

若要使用应用程序代码或工具验证证书，需显式请求一个加密的连接并且不信任服务器证书。即使你的应用程序代码或工具未请求加密的连接，它们仍将会收到加密的连接。但是，它们可能不会验证服务器证书，因此将容易受到“中间人”攻击。

若要使用 ADO.NET 应用程序代码验证证书，请在数据库连接字符串中设置 ``Encrypt=True`` 和 ``TrustServerCertificate=False``。有关详细信息，请参阅[代码示例：使用 ADO.NET 连接到 Azure SQL 数据库](https://msdn.microsoft.com/zh-cn/library/azure/ee336243.aspx)。

SQL Server Management Studio 也支持证书验证。在“连接到服务器”对话框中的“连接属性”选项卡上单击“加密连接”。

> [AZURE.NOTE] 在 SQL Server 2008 R2 之前的版本中，SQL Server Management Studio 不支持与 SQL 数据库建立连接。

尽管从 SQL 数据库开始 SQLCMD 支持 SQL Server 2008，但在 SQL Server 2008 R2 之前的版本中不支持证书验证。若要使用从 SQL Server 2008 R2 中开始的 SQLCMD 来验证证书，请使用 ``-N`` 命令行选项并且不使用 ``-C`` 选项。通过使用 -N 选项，SQLCMD 请求加密的连接。不使用 ``-C`` 选项时，SQLCMD 不会隐式信任服务器证书并且强制验证证书。

有关补充技术信息，请参阅 TechNet Wiki 站点上的文章 [Azure SQL 数据库连接安全性](http://social.technet.microsoft.com/wiki/contents/articles/2951.windows-azure-sql-database-connection-security.aspx#comment-4847)。

## 身份验证

Active Directory 身份验证（集成安全性）在 SQL 数据库 V12 中以预览版的形式提供。有关如何配置 AD 身份验证的信息，请参阅[通过使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)。如果与使用预览版，则用户在每次连接到 SQL 数据库时都必须提供凭据（登录名和密码）。有关 SQL Server 身份验证的详细信息，请参阅 SQL Server 联机丛书中的[选择身份验证模式](https://msdn.microsoft.com/zh-cn/library/ms144284.aspx)。

[SQL 数据库 V12](/documentation/articles/sql-database-v12-whats-new/) 允许用户使用包含的数据库用户在数据库中进行身份验证。有关详细信息，请参阅[包含的数据库用户 - 使你的数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)、[CREATE USER (Transact-SQL)](https://technet.microsoft.com/zh-cn/library/ms173463.aspx) 和[包含的数据库](https://technet.microsoft.com/zh-cn/library/ff929071.aspx)。

> [AZURE.NOTE] Azure 建议使用包含的数据库用户提高可缩放性。

数据库引擎将关闭空闲超过 30 分钟的连接。该连接必须重新登录才可供使用。

连续与 SQL 数据库建立活动连接需要至少每隔 10 小时重新授权一次（由数据库引擎执行授权）。数据库引擎将尝试使用最初提交的密码重新授权，且不需要用户输入。出于性能原因，在 SQL 数据库中重置密码时，将不会对连接重新进行身份验证，即使该连接由于连接池而重置。这一点与本地 SQL Server 的行为不同。如果自从最初授权后密码已更改，则必须终止该连接，并使用新密码建立新连接。具有 KILL DATABASE CONNECTION 权限的用户可以使用 [KILL](https://msdn.microsoft.com/zh-cn/library/ms173730.aspx) 命令显式终止与 SQL 数据库的连接。

## 登录名和用户

在 SQL 数据库中管理登录名和用户时，存在一些限制。

对于服务器级别主体登录名，以下限制适用：

- 不能更改或删除 master 数据库中与服务器级别主体登录名相对应的数据库用户。 
- 尽管服务器级别主体登录名不是 **dbmanager** 数据库中两个数据库角色 **loginmanager** 和 **master** 的成员，但它具有向这两个角色授予的所有权限。

> [AZURE.NOTE] 此登录名在服务器预配期间创建并且类似于 SQL Server 实例中的 **sa** 登录名。

对于所有登录名，以下限制均适用：

- 默认语言为美国英语。
- 若要访问 **master** 数据库，每个登录名都必须映射到 **master** 数据库中的某一用户帐户。**master** 数据库不支持包含的数据库用户。
- 如果未在连接字符串中指定数据库，则将默认连接到 **master** 数据库。
- 在执行 ``CREATE/ALTER/DROP LOGIN`` 和 ``CREATE/ALTER/DROP DATABASE`` 语句时，必须连接到 **master** 数据库。 
- 在 ADO.NET 应用程序中执行 ``CREATE/ALTER/DROP LOGIN`` 和 ``CREATE/ALTER/DROP DATABASE`` 语句时，不允许使用参数化命令。有关详细信息，请参阅[命令和参数](https://msdn.microsoft.com/zh-cn/library/ms254953.aspx)。
- 在执行 ``CREATE/ALTER/DROP DATABASE`` 和 ``CREATE/ALTER/DROP LOGIN`` 语句时，上述每个语句都必须是 Transact-SQL 批处理中的唯一语句。否则将会出错。例如，以下 Transact-SQL 将检查该数据库是否存在。如果该数据库存在，则调用 ``DROP DATABASE`` 语句以便删除该数据库。因为 ``DROP DATABASE`` 语句不是该批处理中的唯一语句，所以执行此 Transact-SQL 将导致错误。


		IF EXISTS (SELECT [name]
		           FROM   [sys].[databases]
		           WHERE  [name] = N'database_name')
		     DROP DATABASE [database_name];
		GO


- 在使用 ``FOR/FROM LOGIN`` 选项执行 ``CREATE USER`` 语句时，该语句必须是 Transact-SQL 批处理中的唯一语句。
- 在使用 ``WITH LOGIN`` 选项执行 ``ALTER USER`` 语句时，该语句必须是 Transact-SQL 批处理中的唯一语句。
- 只有服务器级别主体登录名和 **dbmanager** 数据库中 **master** 角色的成员才有权执行 ``CREATE DATABASE`` 和 ``DROP DATABASE`` 语句。
- 只有服务器级别主体登录名和 **loginmanager** 数据库中 **master** 角色的成员才有权执行 ``CREATE LOGIN``、``ALTER LOGIN`` 和 ``DROP LOGIN`` 语句。
- 若要执行 ``CREATE/ALTER/DROP`` 操作，用户需要对数据库拥有 ``ALTER ANY USER`` 权限。
- 在数据库角色的所有者尝试向该数据库角色添加其他数据库用户或者从该数据库角色中删除其他数据库用户时，可能会发生以下错误：“此数据库中不存在用户或角色 'Name'”。 在用户对所有者不可见时，将会发生此错误。若要解决此问题，请向角色所有者授予对该用户的 ``VIEW DEFINITION`` 权限。 

有关登录名和用户的详细信息，请参阅[在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins/)。

## 安全最佳实践

请考虑以下这些要点，以免你的 Azure SQL 数据库应用程序受到安全威胁：

- 始终使用最新的更新：在连接到 SQL 数据库时，始终使用工具和库的最新版本来防止安全漏洞。
- 阻止 TCP 端口 1433 上的入站连接：应用程序只需 TCP 端口 1433 上的出站连接即可与 SQL 数据库通信。如果该计算机上任何其他应用程序不需要入站通信，则请确保你的防火墙继续阻止 TCP 端口 1433 上的入站连接。
- 防止注入安全漏洞：为了确保应用程序不存在 SQL 注入安全漏洞，请尽可能使用参数化查询。此外，请确保首先通查代码并且运行渗透测试，再部署你的应用程序。


## 另请参阅

[Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)

[在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins/)

[SQL Server 数据库引擎和 Azure SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)

<!---HONumber=Mooncake_0606_2016-->
