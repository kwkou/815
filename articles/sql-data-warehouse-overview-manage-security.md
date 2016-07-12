<properties
   pageTitle="保护 SQL 数据仓库中的数据库 | Azure"
   description="有关在开发解决方案时保护 Azure SQL 数据仓库中的数据库的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sahaj08"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/13/2016"
   wacn.date="06/20/2016"/>

# 保护 SQL 数据仓库中的数据库

本文逐步讲述有关保护 Azure SQL 数据仓库数据库的基本知识。具体而言，本文将帮助你了解如何使用相应的资源，在数据库中限制访问、保护数据和监视活动。

## 连接安全性

连接安全性是指如何使用防火墙规则和连接加密来限制和保护数据库连接。

服务器和数据库使用防火墙规则来拒绝源自未明确列入白名单的 IP 地址的连接企图。若要从应用程序或客户端计算机的公共 IP 地址进行连接，必须先使用 Azure 管理门户、REST API 或 PowerShell 创建服务器级防火墙规则。作为最佳实践，应该尽量通过服务器防火墙来限制允许的 IP 地址范围。若要从你的本地计算机访问 Azure SQL 数据仓库，请确保你的网络和本地计算机上的防火墙允许在 TCP 端口 1433 上的传出通信。有关详细信息，请参阅 [Azure SQL 数据库防火墙][]。

连接到 SQL 数据仓库时，可以在连接字符串中设置加密模式，以便对连接进行加密。为连接启用加密时，其语法因协议而异。导航到 Azure 门户中的数据库即可设置连接字符串。在“概要”下面，单击“显示数据库连接字符串”。


## 身份验证

身份验证是指连接到数据库时如何证明你的身份。SQL 数据仓库当前支持使用用户名和密码的 SQL Server 身份验证。Windows 身份验证以预览功能的形式提供。

在为数据库创建逻辑服务器时，你已指定一个包含用户名和密码的“服务器管理员”登录名。使用这些凭据，你可以使用数据库所有者（即“dbo”）的身份通过服务器上任何数据库的身份验证。

但是，作为最佳实践，组织的用户应使用不同的帐户进行身份验证。这样，你就可以限制授予应用程序的权限，并在应用程序代码容易受到 SQL 注入攻击的情况下降低恶意活动的风险。若要基于服务器登录名创建数据库用户，请执行以下操作：

首先，使用服务器管理员登录名连接到服务器上的 master 数据库，并创建新的服务器登录名。

    -- Connect to master database and create a login
    CREATE LOGIN ApplicationLogin WITH PASSWORD = 'strong_password';

然后，使用服务器管理员登录名连接到 SQL 数据仓库数据库，并基于刚刚创建的服务器登录名创建数据库用户。

    -- Connect to SQL DW database and create a database user
    CREATE USER ApplicationUser FOR LOGIN ApplicationLogin;


有关如何向 SQL 数据库进行身份验证的详细信息，请参阅 [Managing databases and logins in Azure SQL Database（在 Azure SQL 数据库中管理数据库和登录）][]和 [使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库][]。


## 授权

授权是指你可以在 Azure SQL 数据仓库数据库中执行哪些操作，这由你的用户帐户角色成员身份和权限来控制。作为最佳实践，你应向用户授予所需的最低权限。Azure SQL 数据仓库可让你在 T-SQL 中使用角色方便管理这种权限：

    EXEC sp_addrolemember 'db_datareader', 'ApplicationUser'; -- allows ApplicationUser to read data
    EXEC sp_addrolemember 'db_datawriter', 'ApplicationUser'; -- allows ApplicationUser to write data

用于连接的服务器管理员帐户是 db\_owner 所有者的成员，该帐户有权在数据库中执行任何操作。请保存此帐户，以便部署架构升级并执行其他管理操作。权限受到更多限制的“ApplicationUser”帐户可让用户使用应用程序所需的最低权限从应用程序连接到数据库。

有许多方式可以进一步限制用户通过 Azure SQL 数据库可以执行的操作：

- 精细[权限][]可让你控制可以对数据库中单个列、表、视图、过程和其他对象执行的操作。使用细化的权限可以进行最精细的控制，可以根据用户需要授予其最低权限。细化权限系统有点复杂，需要进行一定程度的学习才能有效使用。
- 除了 db\_datareader 和 db\_datawriter 以外的[数据库角色][]可用于创建权力较大的应用程序用户帐户或权力较小的管理帐户。内置的固定的数据库角色可以方便地用来授予权限，但可能会导致所授权限超出需要的情况。
- [存储过程][]可用于限制可对数据库执行的操作。

## 加密

Azure SQL 数据仓库将会帮助你通过使用[透明数据加密][]来加密处于“静止”状态或存储在数据库文件的数据，从而保护你的数据。必须是管理员或 dbmanager 角色的成员才能在 master 数据库中启用 TDE。若要加密你的数据库，请连接到服务器上的 master 数据库，并执行：

    ALTER DATABASE [AdventureWorks] SET ENCRYPTION ON;

你也可以通过 [Azure 门户][]中的数据库设置启用透明数据加密。有关详细信息，请参阅[透明数据加密 (TDE) 入门][]。

## 审核

审核和跟踪数据库事件可帮助你保持合规性，以及识别可疑活动。SQL 数据仓库审核可让你将数据库中的事件记录到 Azure 存储帐户中的审核日志。SQL 数据仓库审核还与 Microsoft Power BI 集成，以帮助向下钻取报告和分析数据。有关详细信息，请参阅 [SQL 数据库审核入门][]。

## 后续步骤
如需使用不同协议连接到 SQL 数据仓库的详细信息和示例，请参阅[连接到 SQL 数据仓库][]。

<!--Image references-->

<!--Article references-->
[连接到 SQL 数据仓库]: /documentation/articles/sql-data-warehouse-develop-connections/
[SQL 数据库审核入门]: /documentation/articles/sql-database-auditing-get-started/
[透明数据加密 (TDE) 入门]: /documentation/articles/sql-data-warehouse-encryption-tde-sql/

<!--MSDN references-->
[Azure SQL 数据库防火墙]: /documentation/articles/sql-database-firewall-configure/
[数据库角色]: https://msdn.microsoft.com/zh-cn/library/ms189121.aspx
[Managing databases and logins in Azure SQL Database（在 Azure SQL 数据库中管理数据库和登录）]: /documentation/articles/sql-database-manage-logins/
[权限]: https://msdn.microsoft.com/zh-cn/library/ms191291.aspx
[存储过程]: https://msdn.microsoft.com/zh-cn/library/ms190782.aspx
[透明数据加密]: https://msdn.microsoft.com/zh-cn/library/dn948096.aspx
[Azure 门户]: https://manage.windowsazure.cn/

<!--Other Web references-->
[Role-based access control in Azure Portal（Azure 门户中基于角色的访问控制）]: /documentation/articles/role-based-access-control-configure/

<!---HONumber=Mooncake_0613_2016-->