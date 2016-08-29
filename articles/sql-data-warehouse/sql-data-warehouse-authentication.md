<!-- Temporarily comment out connect overview, visual studio, nex time on -->
<properties
   pageTitle="对 Azure SQL 数据仓库进行身份验证 | Azure"
   description="对 Azure SQL 数据仓库进行的 Azure Active Directory (AAD) 和 SQL Server 身份验证。"
   services="sql-data-warehouse"
   documentationCenter=""
   authors="byham"
   manager="barbkess"
   editor=""
   tags=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="08/04/2016"
   wacn.date="08/29/2016" />

# 对 Azure SQL 数据仓库进行身份验证

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-connect-overview/)
- [身份验证](/documentation/articles/sql-data-warehouse-authentication/)
- [驱动程序](/documentation/articles/sql-data-warehouse-connection-strings/)

若要连接到 SQL 数据仓库，需要传入安全凭据进行身份验证。建立连接时，你还会看到特定的连接设置已配置为建立查询会话的一部分。

有关安全性以及如何启用与数据仓库的连接的详细信息，请参阅[保护 SQL 数据仓库中的数据库][]。

## SQL 身份验证
若要连接到 SQL 数据仓库，需要提供以下信息：

- 完全限定的服务器名称
- 指定 SQL 身份验证
- 用户名
- 密码
- 默认数据库（可选）

默认情况下，你的连接将连接到 master 数据库而不是用户数据库。若要连接到用户数据库，可以选择执行以下两项操作之一：

1. 在 SSDT、SSMS 或应用程序连接字符串中将您的服务器注册到 SQL Server 对象资源管理器时指定默认数据库。例如，包含 ODBC 连接的 InitialCatalog 参数。
2. 在 SSDT 中创建会话之前先突出显示用户数据库。

> [AZURE.NOTE] 有关使用 SSDT 连接到 SQL 数据仓库的指南，请参阅[使用 Visual Studio 进行查询][]一文。

再次强调，必须注意，不支持使用 Transact-SQL 语句 **USE<your DB>** 更改连接的数据库


## Azure Active Directory (AAD) 身份验证

[Azure Active Directory][What is Azure Active Directory] 身份验证是使用 Azure Active Directory (Azure AD) 中的标识连接到 Azure SQL 数据仓库的一种机制。通过 Azure Active Directory 身份验证，可以在一个中心位置中集中管理数据库用户和其他 Microsoft 服务的标识。集中 ID 管理提供单一位置用于管理 SQL 数据仓库用户，并简化权限管理。

### 优点

包括如下优点：

- 提供一个 SQL Server 身份验证的替代方法。
- 帮助阻止用户标识在数据库服务器之间激增。
- 允许在单一位置中轮换密码
- 客户可以使用外部 (AAD) 组管理数据库权限。
- 它可以通过启用集成的 Windows 身份验证和 Azure Active Directory 支持的其他形式的身份验证来消除存储密码。
- Azure Active Directory 身份验证使用包含的数据库用户以数据库级别对标识进行身份验证。
- Azure Active Directory 支持对连接到 SQL 数据仓库的应用程序进行基于令牌的身份验证。


### 配置步骤

配置步骤包括配置和使用 Azure Active Directory 身份验证的以下过程。

1. 创建并填充 Azure Active Directory
2. 可选：关联或更改当前与你的 Azure 订阅关联的活动目录
3. 为 Azure SQL 数据仓库创建 Azure Active Directory 管理员。
4. 配置客户端计算机
5. 在映射到 Azure AD 标识的数据库中创建包含的数据库用户
6. 使用 Azure AD 标识连接到数据仓库

目前，Azure Active Directory 用户不会显示在 SSDT 对象资源管理器中。解决方法是在 [sys.database\_principals](https://msdn.microsoft.com/zh-cn/library/ms187328.aspx) 中查看这些用户。
  
### 查看详细信息
- 完成详细步骤。配置和使用 Azure Active Directory 身份验证的详细步骤与适用于 Azure SQL 数据库和 Azure SQL 数据仓库的步骤几乎完全相同。请遵循主题[使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库](/documentation/articles/sql-database-aad-authentication/)中的详细步骤。
- 创建自定义数据库角色，并向角色添加用户。然后授予角色具体权限。有关详细信息，请参阅[数据库引擎权限入门](https://msdn.microsoft.com/zh-cn/library/mt667986.aspx)。

<!-- Article references -->
[保护 SQL 数据仓库中的数据库]: /documentation/articles/sql-data-warehouse-overview-manage-security/
[使用 Visual Studio 进行查询]: /documentation/articles/sql-data-warehouse-query-visual-studio/
[What is Azure Active Directory]: /documentation/articles/active-directory-whatis/

<!---HONumber=Mooncake_0822_2016-->