<properties
   pageTitle="使用 Azure Active Directory 身份验证连接到 SQL 数据仓库 | Azure"
   description="了解如何通过使用 Azure Active Directory 身份验证连接到 SQL 数据仓库。"
   services="sql-data-warehouse"
   documentationCenter=""
   authors="BYHAM"
   manager="jhubbard"
   editor=""
   tags=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/11/2016"
   wacn.date="05/30/2016"/>

# 使用 Azure Active Directory 身份验证连接到 SQL 数据仓库


Azure Active Directory 身份验证是使用 Azure Active Directory (Azure AD) 中的标识连接到 Azure SQL 数据仓库的一种机制。通过 Azure Active Directory 身份验证，可以在一个中心位置中集中管理数据库用户和其他 Microsoft 服务的标识。集中 ID 管理提供单一位置用于管理 SQL 数据仓库用户，并简化权限管理。包括如下优点：

- 提供一个 SQL Server 身份验证的替代方法。
- 帮助阻止用户标识在数据库服务器之间激增。
- 允许在单一位置中轮换密码
- 客户可以使用外部 (AAD) 组管理数据库权限。
- 它可以通过启用集成的 Windows 身份验证和 Azure Active Directory 支持的其他形式的身份验证来消除存储密码。
- Azure Active Directory 身份验证使用包含的数据库用户以数据库级别对标识进行身份验证。
- Azure Active Directory 支持对连接到 SQL 数据仓库的应用程序进行基于令牌的身份验证。


配置步骤包括配置和使用 Azure Active Directory 身份验证的以下过程。

1. 创建并填充 Azure Active Directory
2. 可选：关联或更改当前与你的 Azure 订阅关联的活动目录
3. 为 Azure SQL 数据仓库创建 Azure Active Directory 管理员。
4. 配置客户端计算机
5. 在映射到 Azure AD 标识的数据库中创建包含的数据库用户
6. 使用 Azure AD 标识连接到数据仓库

配置和使用 Azure Active Directory 身份验证的详细步骤与适用于 Azure SQL 数据库和 Azure SQL 数据仓库的步骤几乎完全相同。请遵循主题 [Connecting to SQL Database or SQL Data Warehouse By Using Azure Active Directory Authentication](/documentation/articles/sql-database-aad-authentication)（使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库）中的详细步骤。

对 Azure SQL 数据库与 Azure SQL 数据仓库使用 Azure Active Directory 身份验证的主要差别在于，必须使用 SQL Server Data Tools 而不是 SQL Server Management Studio 连接到 SQL 数据仓库。SQL 数据仓库要求至少安装 SQL Server Data Tools for Visual Studio 2015 的 2016 年 4 月版（版本 14.0.60311.1）。目前，Azure Active Directory 用户不会显示在 SSDT 对象资源管理器中。解决方法是在 [sys.database\_principals](https://msdn.microsoft.com/zh-cn/library/ms187328.aspx) 中查看这些用户。

<!---HONumber=Mooncake_0523_2016-->