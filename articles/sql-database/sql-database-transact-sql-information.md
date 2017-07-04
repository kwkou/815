<properties
    pageTitle="解析 T-SQL 差异 - 迁移 - Azure SQL 数据库 | Azure"
    description="在 Azure SQL 数据库中不完全支持的 Transact-SQL 语句"
    services="sql-database"
    documentationcenter=""
    author="BYHAM"
    manager="jhubbard"
    editor=""
    tags=""
    translationtype="Human Translation" />
<tags
    ms.assetid="c05abd9e-28a7-4c97-9bdf-bc60d08fc92e"
    ms.service="sql-database"
    ms.custom="overview"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-management"
    ms.date="03/17/2017"
    wacn.date="04/17/2017"
    ms.author="rickbyh"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="f28a2d818b688998e4ccd3aba922a163e61685d7"
    ms.lasthandoff="04/07/2017" />

# <a name="resolving-transact-sql-differences-during-migration-to-sql-database"></a>解析迁移到 SQL 数据库的过程中的 Transact-SQL 差异   
从 SQL Server [将数据库迁移](/documentation/articles/sql-database-cloud-migrate/)到 Azure SQL Server 时，可能会发现需要对数据库进行一些重新设计才能迁移 SQL Server。 本主题提供相关指南来帮助你执行此重新设计和了解重新设计是必需的基本原因。 若要检测不兼容性，请使用 [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595)。

## <a name="overview"></a>概述
Microsoft SQL Server 和 Azure SQL 数据库都完全支持应用程序使用的大多数 Transact-SQL 功能。 例如，核心 SQL 组件（如数据类型、运算符、字符串、算术、逻辑和光标函数等）在 SQL Server 和 SQL 数据库中的工作方式相同。 但是，DDL（数据定义语言）和 DML（数据操作语言）元素中的一些 T-SQL 差异导致存在仅部分受支持的 T-SQL 语句和查询（我们将在本主题中后面的内容中介绍）。

此外，还有一些功能和语法根本不受支持，因为 Azure SQL 数据库旨在将功能与 master 数据库和操作系统的依赖项隔离。 因此，大多数服务器级活动不适用于 SQL 数据库。 T-SQL 语句和选项在配置服务器级选项、操作系统组件或指定文件系统配置时不可用。 需要此类功能时，通常是以某种其他方式从 SQL 数据库或从其他 Azure 功能或服务获取相应的替代项。 

例如，高可用性内置于 Azure 中，因此不需要配置 Always On（尽管可能需要配置活动异地复制，以便在发生灾难时更快地恢复）。 因此，SQL 数据库不支持与可用性组相关的 T-SQL 语句，也不支持与 Always On 相关的动态管理视图。

有关 SQL 数据库支持和不支持的功能的列表，请参阅 [Azure SQL 数据库功能比较](/documentation/articles/sql-database-features/)。 此页上的列表对该“准则和功能”主题进行了补充，并重点介绍了 Transact-SQL 语句。

## <a name="transact-sql-syntax-statements-with-partial-differences"></a>具有部分差异的 Transact-SQL 语法语句
核心 DDL（数据定义语言）语句可用，但某些 DDL 语句具有与磁盘放置和不支持功能相关的扩展。 

- CREATE 和 ALTER DATABASE 语句具有超过 36 个的选项。 这些语句包括文件定位、FILESTREAM 以及仅适用于 SQL Server 的服务中转站选项。 如果在迁移前创建数据库，这可能不是问题，但如果要迁移用于创建数据库的 T-SQL 代码，应将 [CREATE DATABASE（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx)与 [CREATE DATABASE (SQL Server Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms176061.aspx) 中的 SQL Server 语法进行比较，以确保所用的所有选项都受支持。 Azure SQL 数据库的 CREATE DATABASE 语句还具有服务目标和仅适用于 SQL 数据库的弹性缩放选项。
- CREATE 和 ALTER TABLE 语句具有不能在 SQL 数据库上使用的 FileTable 选项，因为不支持 FILESTREAM。
- SQL 数据库支持 CREATE 和 ALTER login 语句，但未提供所有选项。 要使数据库更易于移植，SQL 数据库建议尽可能使用包含的数据库用户，而不是使用登录名。 有关详细信息，请参阅 [CREATE/ALTER LOGIN](https://msdn.microsoft.com/zh-cn/library/ms189828.aspx) 和[控制和授予数据库访问权限](https://docs.microsoft.com/azure/sql-database/sql-database-manage-logins)。

## <a name="transact-sql-syntax-not-supported-in-sql-database"></a>SQL 数据库不支持的 Transact-SQL 语法   
除了与 [Azure SQL 数据库功能比较](/documentation/articles/sql-database-features/)中所述的不支持功能相关的 Transact-SQL 语句外，也不支持以下语句和语句组。 因此，如果要迁移的数据库使用以下任一功能，请重新设计 T-SQL 以消除这些 T-SQL 功能和语句。

- 系统对象的排序规则
- 相关的连接：终结点语句、 `ORIGINAL_DB_NAME`。 SQL 数据库不支持 Windows 身份验证，但支持类似的 Azure Active Directory 身份验证。 某些身份验证类型要求使用最新版本的 SSMS。 有关详细信息，请参阅[使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库](/documentation/articles/sql-database-aad-authentication/)。
- 使用三个或四个部分名称的跨数据库查询。 （使用[弹性数据库查询](/documentation/articles/sql-database-elastic-query-overview/)支持只读跨数据库查询。）
- 跨数据库所有权链接, `TRUSTWORTHY` 设置
- `DATABASEPROPERTY` 改用 `DATABASEPROPERTYEX`。
- `EXECUTE AS LOGIN` 改用“EXECUTE AS USER”。
- 支持加密，但可扩展密钥管理除外
- 事件：事件、事件通知、查询通知
- 文件定位：与数据库文件定位、大小以及 Azure 自动管理的数据库文件相关的语法。
- 高可用性：与通过 Azure 帐户管理的高可用性相关的语法。 这包括备份、还原、Always On、数据库镜像、日志传送、恢复模式的语法。
- 日志读取器：依赖于在 SQL 数据库上不可用的日志读取器的语法：推送复制、更改数据捕获。 SQL 数据库可以是推送复制项目的订阅服务器。
- 函数：`fn_get_sql`、`fn_virtualfilestats`、`fn_virtualservernodes`
- 全局临时表
- 硬件：与硬件相关的服务器设置（例如，内存、工作线程、CPU 相关性、跟踪标志）有关的语法。 改用服务级别。
- `HAS_DBACCESS`
- `KILL STATS JOB`
- `OPENQUERY`、`OPENROWSET`、`OPENDATASOURCE` 和由四部分构成的名称
- .NET Framework：CLR 与 SQL Server 集成
- 语义搜索
- 服务器凭据：改用[数据库范围的凭据](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx)。
- 服务器级项目：服务器角色、`IS_SRVROLEMEMBER`、`sys.login_token`。 `GRANT`、`REVOKE` 和 `DENY` 的服务器级权限不可用，某些权限已替换为数据库级权限。 一些有用的服务器级 DMV 具有等效的数据库级 DMV。
- `SET REMOTE_PROC_TRANSACTIONS`
- `SHUTDOWN`
- `sp_addmessage`
- `sp_configure` 选项和 `RECONFIGURE`。 可以通过 [ALTER DATABASE SCOPED CONFIGURATION](https://msdn.microsoft.com/zh-cn/library/mt629158.aspx) 使用某些选项。
- `sp_helpuser`
- `sp_migrate_user_to_contained`
- SQL Server 代理：依赖于 SQL Server 代理或 MSDB 数据库的语法：警报、运算符、中央管理服务器。 改用脚本，如 Azure PowerShell。
- SQL Server 审核：改用 SQL 数据库审核。
- SQL Server 跟踪
- 跟踪标志：某些跟踪标志项已移至兼容模式。
- Transact-SQL 调试
- 触发器：服务器作用域或登录触发器
- `USE` 语句：若要将数据库上下文更改为不同的数据库，必须与新数据库建立新连接。

## <a name="full-transact-sql-reference"></a>完整的 Transact-SQL 引用
有关 Transact-SQL 语法、用法和示例的详细信息，请参阅 SQL Server 联机丛书中的 [Transact-SQL 参考（数据库引擎）](https://msdn.microsoft.com/zh-cn/library/bb510741.aspx)。 

### <a name="about-the-applies-to-tags"></a>有关“适用于”标记
Transact-SQL 参考包含从 SQL Server 2008 到最新版本的相关主题。 主题标题下面有一个图标栏，其中列出了四个 SQL Server 平台，并指明了适用性。 例如，SQL Server 2012 中引入了可用性组。 [CREATE AVAILABILTY GROUP](https://msdn.microsoft.com/zh-cn/library/ff878399.aspx) 主题指明该语句适用于 **SQL Server（从版本 2012 开始）**。 该语句不适用于 SQL Server 2008、SQL Server 2008 R2、Azure SQL 数据库、Azure SQL 数据仓库或并行数据仓库。

在某些情况下，产品中可能使用了某个主题的常规主旨，但产品之间存在细微的差异。 在适当的情况下，我们会在主题的中间位置指出差异。 在某些情况下，产品中可能使用了某个主题的常规主旨，但产品之间存在细微的差异。 在适当的情况下，我们会在主题的中间位置指出差异。 例如，CREATE TRIGGER 主题在 SQL 数据库中可用。 但服务器级触发器的 **ALL SERVER** 选项指示不能在 SQL 数据库中使用服务器级触发器。 请改用数据库级触发器。

## <a name="next-steps"></a>后续步骤

有关 SQL 数据库支持和不支持的功能的列表，请参阅 [Azure SQL 数据库功能比较](/documentation/articles/sql-database-features/)。 此页上的列表对该“准则和功能”主题进行了补充，并重点介绍了 Transact-SQL 语句。
<!--Update_Description: refine overview description; update the list of unsupported T-SQL-->