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
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management"
   ms.date="09/24/2016"
   wacn.date="10/31/2016"/>  


# 对 Azure SQL 数据仓库进行身份验证

> [AZURE.SELECTOR]
- [安全性概述](/documentation/articles/sql-data-warehouse-overview-manage-security/)
- [身份验证](/documentation/articles/sql-data-warehouse-authentication/)
- [加密（门户）](/documentation/articles/sql-data-warehouse-encryption-tde/)
- [加密 (T-SQL)](/documentation/articles/sql-data-warehouse-encryption-tde-tsql/)

若要连接到 SQL 数据仓库，必须传入安全凭据进行身份验证。建立连接时，特定的连接设置已配置为建立查询会话的一部分。

有关安全性以及如何启用与数据仓库的连接的详细信息，请参阅[保护 SQL 数据仓库中的数据库][]。

## SQL 身份验证
若要连接到 SQL 数据仓库，必须提供以下信息：

- 完全限定的服务器名称
- 指定 SQL 身份验证
- 用户名
- 密码
- 默认数据库（可选）

默认情况下，连接连接到 *master* 数据库而不是用户数据库。若要连接到用户数据库，可以选择执行以下两项操作之一：

- 在 SSDT、SSMS 或应用程序连接字符串中将您的服务器注册到 SQL Server 对象资源管理器时指定默认数据库。例如，包含 ODBC 连接的 InitialCatalog 参数。
- 在 SSDT 中创建会话之前先突出显示用户数据库。

> [AZURE.NOTE] 不支持使用 Transact-SQL 语句 **USE MyDatabase;** 更改连接的数据库。有关使用 SSDT 连接到 SQL 数据仓库的指南，请参阅[使用 Visual Studio 进行查询][]一文。

## Azure Active Directory (AAD) 身份验证

[Azure Active Directory][What is Azure Active Directory] 身份验证是使用 Azure Active Directory (Azure AD) 中的标识连接到 Azure SQL 数据仓库的一种机制。通过 Azure Active Directory 身份验证，可以在一个中心位置中集中管理数据库用户和其他 Microsoft 服务的标识。集中 ID 管理提供单一位置用于管理 SQL 数据仓库用户，并简化权限管理。

### 优点

Azure Active Directory 的优点包括：

- 提供一个 SQL Server 身份验证的替代方法。
- 帮助阻止用户标识在数据库服务器之间激增。
- 允许在单一位置中轮换密码
- 使用外部 (AAD) 组管理数据库权限。
- 通过启用集成的 Windows 身份验证和 Azure Active Directory 支持的其他形式的身份验证来消除存储密码。
- 使用包含的数据库用户在数据库级别对标识进行身份验证。
- 支持对连接到 SQL 数据仓库的应用程序进行基于令牌的身份验证。
- 支持通过用于 SQL Server Management Studio 的 Active Directory 通用身份验证进行多重身份验证。

> [AZURE.NOTE] Azure Active Directory 仍然相对较新，具有某些限制。若要确保 Azure Active Directory 适用于环境，请参阅 [Azure AD features and limitations][]（Azure AD 功能和限制），尤其是那些需要额外考虑的内容。

### 配置步骤

按照这些步骤配置 Azure Active Directory 身份验证。

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

## 后续步骤

若要开始使用 Visual Studio 和其他应用程序查询数据仓库，请参阅[使用 Visual Studio 进行查询][]。

<!-- Article references -->
[保护 SQL 数据仓库中的数据库]: /documentation/articles/sql-data-warehouse-overview-manage-security/
[使用 Visual Studio 进行查询]: /documentation/articles/sql-data-warehouse-query-visual-studio/
[What is Azure Active Directory]: /documentation/articles/active-directory-whatis/
[Azure AD features and limitations]: /documentation/articles/sql-database-aad-authentication

<!---HONumber=Mooncake_1024_2016-->