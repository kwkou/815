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
   ms.date="01/07/2016"
   wacn.date="03/17/2016"/>

# 保护 SQL 数据仓库中的数据库

本文逐步讲述有关保护 Azure SQL 数据仓库数据库的基本知识。具体而言，本文将帮助你了解如何使用相应的资源，在数据库中限制访问、保护数据和监视活动。

## 连接安全性

连接安全性是指如何使用防火墙规则和连接加密来限制和保护数据库连接。

服务器和数据库使用防火墙规则来拒绝源自未明确列入白名单的 IP 地址的连接企图。若要允许应用程序或客户端计算机的公共 IP 地址尝试连接到新数据库，你必须先使用 Azure 管理门户、REST API 或 PowerShell 创建服务器级防火墙规则。作为最佳实践，应该尽量通过服务器防火墙来限制允许的 IP 地址范围。有关详细信息，请参阅 [Azure SQL 数据库防火墙][]。


## 身份验证

身份验证是指连接到数据库时如何证明你的身份。SQL 数据仓库当前支持使用用户名和密码的 SQL 身份验证。

在为数据库创建逻辑服务器时，你已指定一个包含用户名和密码的“服务器管理员”登录名。使用这些凭据，你可以使用数据库所有者（即“dbo”）的身份通过服务器上任何数据库的身份验证。

但是，作为最佳实践，组织的用户应使用不同的帐户进行身份验证。这样，你就可以限制授予应用程序的权限，并在应用程序代码容易受到 SQL 注入攻击的情况下降低恶意活动的风险。若要基于服务器登录名创建数据库用户，请执行以下操作：

首先，使用服务器管理员登录名连接到服务器上的 master 数据库，并创建新的服务器登录名。

```
-- Connect to master database and create a login
CREATE LOGIN ApplicationLogin WITH PASSWORD = 'strong_password';

```

然后，使用服务器管理员登录名连接到 SQL 数据仓库数据库，并基于刚刚创建的服务器登录名创建数据库用户。

```

-- Connect to SQL DW database and create a database user
CREATE USER ApplicationUser FOR LOGIN ApplicationLogin;

```

有关在 SQL 数据库上进行身份验证的详细信息，请参阅[在 Azure SQL 数据库中管理数据库和登录名][]。


## 授权

授权是指你可以在 Azure SQL 数据仓库数据库中执行哪些操作，这由你的用户帐户角色成员身份和权限来控制。作为最佳实践，你应向用户授予所需的最低权限。Azure SQL 数据仓库可让你在 T-SQL 中使用角色方便管理这种权限：

```
EXEC sp_addrolemember 'db_datareader', 'ApplicationUser'; -- allows ApplicationUser to read data
EXEC sp_addrolemember 'db_datawriter', 'ApplicationUser'; -- allows ApplicationUser to write data
```

用于连接的服务器管理员帐户是 db\_owner 所有者的成员，该帐户有权在数据库中执行任何操作。请保存此帐户，以便部署架构升级并执行其他管理操作。权限受到更多限制的“ApplicationUser”帐户可让用户使用应用程序所需的最低权限从应用程序连接到数据库。

有许多方式可以进一步限制用户通过 Azure SQL 数据库可以执行的操作：

- 除了 db\_datareader 和 db\_datawriter 以外的[数据库角色][]可用于创建权力较大的应用程序用户帐户或权力较小的管理帐户。
- 精细[权限][]可让你控制可以对数据库中单个列、表、视图、过程和其他对象执行的操作。
- [存储过程][]可用于限制可对数据库执行的操作。

从 Azure 管理门户或使用 Azure 资源管理器 API 管理数据库和逻辑服务器的操作将会根据你的门户用户帐户的角色分配进行控制。有关此主题的详细信息，请参阅 [Azure 预览管理门户中基于角色的访问控制][]。



## 加密

Azure SQL 数据仓库将会帮助你通过使用[透明数据加密][]来加密处于“静止”状态或存储在数据库文件的数据，从而保护你的数据。若要加密你的数据库，请连接到服务器上的 master 数据库，并执行：


```

ALTER DATABASE [AdventureWorks] SET ENCRYPTION ON;

```

你也可以通过 [Azure 管理门户][] 中的数据库设置启用透明数据加密。



## 审核

审核和跟踪数据库事件可帮助你保持合规性，以及识别可疑活动。SQL 数据仓库审核可让你将数据库中的事件记录到 Azure 存储帐户中的审核日志。SQL 数据仓库审核还与 Microsoft Power BI 集成，以帮助向下钻取报告和分析数据。有关详细信息，请参阅 [SQL 数据库审核入门][]。



## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop


<!--MSDN references-->
[Azure SQL 数据库防火墙]: https://msdn.microsoft.com/zh-cn/library/ee621782.aspx
[数据库角色]: https://msdn.microsoft.com/zh-cn/library/ms189121.aspx
[在 Azure SQL 数据库中管理数据库和登录名]: https://msdn.microsoft.com/zh-cn/library/ee336235.aspx
[权限]: https://msdn.microsoft.com/zh-cn/library/ms191291.aspx
[存储过程]: https://msdn.microsoft.com/zh-cn/library/ms190782.aspx
[透明数据加密]: http://go.microsoft.com/fwlink/?LinkId=526242
[SQL 数据库审核入门]: /documentation/articles/sql-database-auditing-get-started
[Azure 管理门户]: https://manage.windowsazure.cn

<!--Other Web references-->
[Azure 门户中基于角色的访问控制]: /documentation/articles/role-based-access-control-configure

<!---HONumber=Mooncake_0307_2016-->