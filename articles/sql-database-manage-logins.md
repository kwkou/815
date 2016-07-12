<properties
   pageTitle="SQL 数据库安全管理 - 登录安全 | Azure"
   description="了解 SQL 数据库安全管理，特别是如何通过服务器级的主体帐户管理数据库的访问和登录安全。"
   keywords="sql 数据库安全,数据库安全管理,登录安全,数据库安全,数据库访问权限"
   services="sql-database"
   documentationCenter=""
   authors="BYHAM"
   manager="jhubbard"
   editor=""
   tags=""/>

<tags
   ms.service="sql-database"
   ms.date="03/22/2016"
   wacn.date="06/14/2016"/>

# SQL 数据库安全：管理数据库的访问和登录安全  

了解 SQL 数据库安全管理，特别是如何通过服务器级的主体帐户管理数据库的访问和登录安全。了解 SQL 数据库和本地 SQL Server 之间存在的登录安全选项方面的某些异同之处。请参阅 [Azure SQL 数据库教程：Azure SQL 数据库安全性入门](/documentation/articles/sql-database-get-started-security/)获取快速教程。

## 数据库预配和服务器级的主体登录名

在 Azure SQL 数据库中，当你注册该服务时，预配过程将创建一个 Azure SQL 数据库服务器、一个名为 **master** 的数据库以及作为你的 Azure SQL 数据库服务器的服务器级别主体的登录名。该登录名与本地 SQL Server 实例的服务器级别主体 (**sa**) 类似。

Azure SQL 数据库服务器级别主体帐户始终有权管理所有服务器级别和数据库级别的安全性。本主题介绍如何使用服务器级别主体和其他帐户以便在 SQL 数据库中管理登录名和数据库。

通过 Azure 基于角色的访问控制 (RBAC) 和 Azure 资源管理器 REST API 来访问 SQL 数据库的 Azure 用户从其 Azure 角色获得权限。这些角色提供对管理平面操作的访问权限，但不提供对数据平面操作的访问权限。这些管理平面操作包括能够读取各种属性和 SQL 数据库中的架构元素。并且允许创建、删除和配置某些与 SQL 数据库相关的服务器级别的功能。其中许多管理平面操作是在使用 Azure 门户时可以查看并配置的项。使用 RBAC 角色时，数据库内部的 Azure 角色成员的操作（例如列出表）由数据库引擎执行，这样角色就不会被 GRANT/REVOKE/DENY 语句的标准 SQL Server 权限系统影响。RBAC 角色不能够读取或更改数据，因为这些是数据平面操作。有关详细信息，请参阅 [RBAC：内置角色](/documentation/articles/role-based-access-built-in-roles/)。

> [AZURE.IMPORTANT] SQL 数据库 V12 允许用户使用包含的数据库用户在数据库中进行身份验证。包含的数据库用户不需要登录名。这将使数据库更易于移植，但会降低服务器级主体对数据库访问权限的控制能力。启用包含的数据库用户会产生重要的安全影响。有关详细信息，请参阅[包含的数据库用户 - 使你的数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)、[包含的数据库](https://technet.microsoft.com/zh-cn/library/ff929071.aspx)、[创建用户 (Transact-SQL)](https://technet.microsoft.com/zh-cn/library/ms173463.aspx) 和[使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)。

## SQL 数据库安全管理概述

SQL 数据库安全管理类似于 SQL Server 的本地实例的安全管理。在数据库级别管理安全性几乎完全相同，只是在可用参数上存在差异。因为 SQL 数据库可以扩展到一个或多个计算机，所以，Azure SQL 数据库为服务器级别管理使用不同的策略。下表介绍本地 SQL Server 的数据库安全管理与 Azure SQL 数据库中的安全管理之间的差异。

| 差异点 | 本地 SQL Server | Azure SQL 数据库 |
|------------------------------------------------|-----------------------------------------------------------------------------|--------------------------------------------------|
| 管理服务器级别安全性的位置 | SQL Server Management Studio 的“对象资源管理器”中的**安全性**文件夹 | **master** 数据库以及通过 Azure 门户 |
| Windows 身份验证 | Active Directory 标识 | Azure Active Directory 标识 |
| 用于创建登录名的服务器级别安全角色 | **securityadmin** 固定服务器角色 | **master** 数据库中的 **loginmanager** 数据库角色 |
| 用于管理登录名的命令 | CREATE LOGIN、ALTER LOGIN、DROP LOGIN | CREATE LOGIN、ALTER LOGIN、DROP LOGIN（存在一些参数限制并且你必须连接到 **master** 数据库。） |
| 显示所有登录名的视图 | sys.server\_principals | sys.sql\_logins（你必须连接到 **master** 数据库）|
| 用于创建数据库的服务器级别角色 | **dbcreator** 固定数据库角色 | **master** 数据库中的 **dbmanager** 数据库角色 |
| 用于创建数据库的命令 | CREATE DATABASE | CREATE DATABASE（存在一些参数限制并且你必须连接到 **master** 数据库。） |
| 列出所有数据库的视图 | sys.databases | sys.databases（你必须连接到 **master** 数据库。） |

## 服务器级别管理和 master 数据库

你的 Azure SQL 数据库服务器是用于定义一组数据库的抽象。与你的 Azure SQL 数据库服务器相关联的数据库可以驻留在 Microsoft 数据中心的单独物理计算机上。通过使用名为 **master** 的单一数据库对全部数据库执行服务器级别的管理。

**master** 数据库跟踪登录名以及哪些登录名有权创建数据库或其他登录名。只要你创建、更改或删除登录名或数据库，就必须连接到 **master** 数据库。**master** 数据库还具有 ``sys.sql_logins`` 和 ``sys.databases`` 视图，可供你查看登录名和数据库。

> [AZURE.NOTE] ``USE`` 命令不支持在数据库之间切换。直接建立与目标数据库的连接。

你可以在 Azure SQL 数据库中通过与用于 SQL Server 的本地实例相同的方式来管理用户和对象的数据库级别的安全性。仅在可用于相应命令的参数中存在差异。有关详细信息，请参阅 [Azure SQL 数据库安全指导原则和限制](/documentation/articles/sql-database-security-guidelines/)。

## 管理包含数据库用户

通过连接到数据库来使用服务器级别主体创建第一个包含数据库用户。使用 ``CREATE USER``、``ALTER USER`` 或 ``DROP USER`` 语句。以下示例将创建一个名为 user1 的用户。

```
CREATE USER user1 WITH password='<Strong_Password>';
```

> [AZURE.NOTE] 创建包含数据库用户时必须使用强密码。有关详细信息，请参阅[强密码](https://msdn.microsoft.com/zh-cn/library/ms161962.aspx)。

可以通过拥有 **ALTER ANY USER** 权限的任何用户创建额外的包含数据库用户。

作为一项预览功能，SQL 数据库 V12 支持将 Azure Active Directory 标识作为包含数据库用户。有关详细信息，请参阅[通过使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)。

Azure 建议对 SQL 数据库使用包含数据库用户。有关详细信息，请参阅[包含数据库用户 - 使你的数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)。

## 管理登录名

通过连接到 master 数据库来使用服务器级别主体登录名管理登录名。你可以使用 ``CREATE LOGIN``、``ALTER LOGIN`` 或 ``DROP LOGIN`` 语句。以下示例将创建一个名为 **login1** 的登录名：

```
-- first, connect to the master database
CREATE LOGIN login1 WITH password='<ProvidePassword>';
```

> [AZURE.NOTE] 创建登录名时必须使用强密码。有关详细信息，请参阅[强密码](https://msdn.microsoft.com/zh-cn/library/ms161962.aspx)。

#### 使用新登录名

为了使用你创建的登录名连接到 Azure SQL 数据库，你必须首先通过使用 ``CREATE USER`` 命令为每个登录名授予数据库级别权限。有关详细信息，请参阅下面的**授予登录名数据库访问权限**一节。

因为某些工具以不同方式实现表格格式数据流 (TDS)，所以，你可能需要使用 ``<login>@<server>`` 表示法将 Azure SQL 数据库服务器名称追加到连接字符串中的登录名。在这些情况下，使用 ``@`` 符号分隔登录名和 Azure SQL 数据库服务器名称。例如，如果你的登录名为 **login1**，并且你的 Azure SQL 数据库服务器的完全限定名称为 **servername.database.chinacloudapi.cn**，则你的连接字符串的用户名参数应为：**login1@servername**。此限制将限制你可为登录名选择的文本。有关详细信息，请参阅 [CREATE LOGIN (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms189751.aspx)。

## 向登录名授予服务器级别权限

为了让你使用除服务器级主体以外的登录名来管理服务器级安全，Azure SQL 数据库提供了两个安全角色：用于创建登录名的 **loginmanager**，以及用于创建数据库的 **dbmanager**。只有 **master** 数据库中的用户才能添加到这些数据库角色中。

> [AZURE.NOTE] 若要创建登录名或数据库，你必须连接到 **master** 数据库（这是 **master** 的逻辑表示形式）。

### loginmanager 角色

与用于 SQL Server 的本地实例的 **securityadmin** 固定服务器角色相似，Azure SQL 数据库中的 **loginmanager** 数据库角色有权创建登录名。只有服务器级别主体登录名（由预配过程创建）或 **loginmanager** 数据库角色的成员才能创建新的登录名。

### dbmanager 角色

Azure SQL 数据库 **dbmanager** 数据库角色与用于 SQL Server 的本地实例的 **dbcreator** 固定服务器角色相似。只有服务器级别主体登录名（由预配过程创建）或 **dbmanager** 数据库角色的成员才能创建数据库。某个用户成为 **dbmanager** 数据库角色的成员后，可以使用 Azure SQL 数据库 ``CREATE DATABASE`` 命令创建数据库，但该命令必须在 master 数据库中执行。有关详细信息，请参阅 [CREATE DATABASE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx)。

### 如何分配 SQL 数据库服务器级别角色

若要创建登录名以及可创建数据库或其他登录名的关联用户，请执行以下步骤：

1. 使用服务器级别主体登录名（由预配过程创建）的凭据或 **loginmanager** 数据库角色的现有成员的凭据连接到 **master** 数据库。
2. 使用 ``CREATE LOGIN`` 命令创建一个登录名。有关详细信息，请参阅 [CREATE LOGIN (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms189751.aspx)。
3. 在 master 数据库中使用 ``CREATE USER`` 命令为该登录名创建一个新用户。有关详细信息，请参阅 [CREATE USER (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms173463.aspx)。
4. 使用存储过程 ``sp_addrolememeber`` 将新用户添加到 **dbmanager** 数据库角色和/或 loginmanager 数据库角色。

以下代码示例演示了如何创建一个登录名 **login1** 以及名为 **login1User** 的相应数据库用户，该用户能够在连接到 **master** 数据库时创建数据库或其他登录名：

```
-- first, connect to the master database
CREATE LOGIN login1 WITH password='<ProvidePassword>';
CREATE USER login1User FROM LOGIN login1;
EXEC sp_addrolemember 'dbmanager', 'login1User';
EXEC sp_addrolemember 'loginmanager', 'login1User';
```

> [AZURE.NOTE] 创建登录名时必须使用强密码。有关详细信息，请参阅[强密码](https://msdn.microsoft.com/zh-cn/library/ms161962.aspx)。

## 授予对登录名的数据库访问权限

所有登录名都必须在 master 数据库中创建。在创建一个登录名后，你可以为该登录名在另一个数据库中创建用户帐户。Azure SQL 数据库通过与 SQL Server 的本地实例中相同的方式支持数据库角色。

若要在其他数据库中创建用户帐户，并且假设你尚未创建登录名或数据库，请执行以下步骤：

1. 连接到 **master** 数据库（使用具有 **loginmanager** 和 **dbmanager** 角色的登录名）。
2. 使用 ``CREATE LOGIN`` 命令创建一个新登录名。有关详细信息，请参阅 [CREATE LOGIN (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms189751.aspx)。不支持 Windows 身份验证。
3. 使用 ``CREATE DATABASE`` 命令创建一个新数据库。有关详细信息，请参阅 [CREATE DATABASE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx)。
4. 建立与该新数据库的连接（使用创建了该数据库的登录名）。
5. 使用 ``CREATE USER`` 命令在这个新数据库上创建一个新用户。有关详细信息，请参阅 [CREATE USER (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms173463.aspx)。

以下代码示例演示如何创建一个名为 **login1** 的登录名和一个名为 **database1** 的数据库：

```
-- first, connect to the master database
CREATE LOGIN login1 WITH password='<ProvidePassword>';
CREATE DATABASE database1;
```

> [AZURE.NOTE] 创建登录名时必须使用强密码。有关详细信息，请参阅[强密码](https://msdn.microsoft.com/zh-cn/library/ms161962.aspx)。

以下示例演示了如何在与登录名 **login1** 相对应的数据库 **database1** 中创建名为 **login1User** 的数据库用户。若要执行以下示例，首先必须使用在 database1 中拥有“更改任意用户”权限的登录名与该数据库建立新连接。以 **db\_owner** 角色成员连接的任何用户（例如创建该数据库的登录名）都拥有该权限。

```
-- Establish a new connection to the database1 database
CREATE USER login1User FROM LOGIN login1;
```

Azure SQL 数据库中的这一数据库级别权限模型与 SQL Server 的本地实例中的权限模型相同。有关详细信息，请参阅 SQL ServerSQL Server 联机丛书中的下列主题。

- [管理登录名、用户和架构操作指南主题](https://msdn.microsoft.com/zh-cn/library/aa337552.aspx)
- [第 2 课：配置对数据库对象的权限](https://msdn.microsoft.com/zh-cn/library/ms365345.aspx)

> [AZURE.NOTE] Azure SQL 数据库中与安全相关的 Transact-SQL 语句在可用的参数方面可能稍有不同。有关详细信息，请参阅“特定语句的联机丛书语法”。

## 查看登录名和数据库


若要查看你的 Azure SQL 数据库服务器上的登录名和数据库，请分别使用 master 数据库的 ``sys.sql_logins`` 和 ``sys.databases`` 视图。以下示例演示如何显示你的 Azure SQL 数据库服务器上所有登录名和数据库的列表。

```
-- first, connect to the master database
SELECT * FROM sys.sql_logins;
SELECT * FROM sys.databases;
```

## 另请参阅

[Azure SQL 数据库教程：Azure SQL 数据库安全性入门](/documentation/articles/sql-database-get-started-security/)
[Azure SQL 数据库安全指导原则和限制](/documentation/articles/sql-database-security-guidelines/)
[通过使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)

<!---HONumber=Mooncake_0606_2016-->
