<properties
    pageTitle="Azure SQL 数据库 T-SQL 中不支持的语句 | Azure"
    description="在 Azure SQL 数据库中不完全支持的 Transact-SQL 语句"
    services="sql-database"
    documentationcenter=""
    author="BYHAM"
    manager="jhubbard"
    editor=""
    tags="" />
<tags
    ms.assetid="c05abd9e-28a7-4c97-9bdf-bc60d08fc92e"
    ms.service="sql-database"
    ms.custom="overview"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="data-management"
    ms.date="11/28/2016"
    wacn.date="01/20/2017"
    ms.author="rick.byham@microsoft.com" />  


# Azure SQL 数据库 Transact-SQL 差异
Microsoft SQL Server 和 Azure SQL 数据库都支持应用程序依赖的大多数 Transact-SQL 功能。例如，核心 SQL 组件（如数据类型、运算符、字符串、算术、逻辑和光标函数等）的工作方式与 SQL Server 没有差别。

## 为什么不支持某种 Transact-SQL
Azure SQL 数据库设计为将功能与 master 数据库和操作系统上的依赖项相隔离。因此，许多服务器级活动并不适用于 SQL 数据库。如果 Transact-SQL 语句用于配置服务器级别选项、操作系统组件或指定文件系统配置，这些语句将通常不可用。当用户数据库外部的功能必需时，通常可以某种其他方式从 SQL 数据库或从其他 Azure 功能或服务获得相应的替代方案。

例如，可以将 Always On 替换为活动异地复制。因此，SQL 数据库不支持与可用性组相关的任何 Transact-SQL 语句，也不支持与 Always On 相关的动态管理视图。

有关 SQL 数据库支持和不支持的功能列表，请参阅 [Azure SQL 数据库注意事项、准则和功能](/documentation/articles/sql-database-features/)。

SQL 数据库通常不支持 SQL Server 中已弃用的语法。

## SQL 数据库部分支持的 Transact-SQL 语法
SQL 数据库支持相应 SQL Server 2016 Transact-SQL 语句中存在的某些（但不是所有）参数。例如，`CREATE PROCEDURE` 语句可用，但 `CREATE PROCEDURE` 的所有选项不可用。在此处描述完整的语法可能会让人感到困惑和冗余。请参阅链接的语法主题，了解有关每个语句支持的区域的详细信息。

- 数据库：[CREATE](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx)/[ALTER DATABASE](https://msdn.microsoft.com/zh-cn/library/ms174269.aspx)
- 函数：[CREATE](https://msdn.microsoft.com/zh-cn/library/ms186755.aspx)/[ALTER FUNCTION](https://msdn.microsoft.com/zh-cn/library/ms186967.aspx)
- 登录名：[CREATE](https://msdn.microsoft.com/zh-cn/library/ms189751.aspx)/[ALTER LOGIN](https://msdn.microsoft.com/zh-cn/library/ms189828.aspx)
- 存储过程：[CREATE](https://msdn.microsoft.com/zh-cn/library/ms187926.aspx)/[ALTER PROCEDURE](https://msdn.microsoft.com/zh-cn/library/ms189762.aspx)
- 表：[CREATE](https://msdn.microsoft.com/zh-cn/library/dn305849.aspx)/[ALTER TABLE](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)
- 类型（自定义）：[CREATE TYPE](https://msdn.microsoft.com/zh-cn/library/ms175007.aspx)
- 用户：[CREATE](https://msdn.microsoft.com/zh-cn/library/ms173463.aspx)/[ALTER USER](https://msdn.microsoft.com/zh-cn/library/ms176060.aspx)
- 视图：[CREATE](https://msdn.microsoft.com/zh-cn/library/ms187956.aspx)/[ALTER VIEW](https://msdn.microsoft.com/zh-cn/library/ms173846.aspx)

## SQL 数据库不支持的 Transact-SQL 语法   
除了与 [Azure SQL 数据库注意事项、准则和功能](/documentation/articles/sql-database-features/)中所述的不支持功能相关的 Transact-SQL 语句外，也不支持以下语句和语句组。
- 系统对象的排序规则
- 相关的连接：终结点语句、`ORIGINAL_DB_NAME`。SQL 数据库不支持 Windows 身份验证，但支持类似的 Azure Active Directory 身份验证。某些身份验证类型要求使用最新版本的 SSMS。有关详细信息，请参阅[使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库](/documentation/articles/sql-database-aad-authentication/)。
- 使用三个或四个部分名称的跨数据库查询。（使用[弹性数据库查询](/documentation/articles/sql-database-elastic-query-overview/)支持只读跨数据库查询。）
- 跨数据库所有权链接、`TRUSTWORTHY` 设置
- `DATABASEPROPERTY` 改用 `DATABASEPROPERTYEX`。
- `EXECUTE AS LOGIN` 改用“EXECUTE AS USER”。
- 支持加密，但不支持可扩展密钥管理
- 事件：事件、事件通知、查询通知
- 与数据库文件定位、大小以及 Azure 自动管理的数据库文件相关的语法。
- 与通过 Azure 帐户管理的高可用性相关的语法。这包括备份、还原、Always On、数据库镜像、日志传送、恢复模式的语法。
- 依赖于在 SQL 数据库上不可用的日志读取器的语法：推送复制、更改数据捕获。SQL 数据库可以是推送复制项目的订阅服务器。
- 依赖于 SQL Server 代理或 MSDB 数据库的语法：警报、运算符、中央管理服务器。请改用脚本，如 Azure PowerShell。
- 函数：`fn_get_sql`、`fn_virtualfilestats`、`fn_virtualservernodes`
- 全局临时表
- 与硬件相关服务器设置相关的语法：内存、工作线程、CPU 相关性、跟踪标志，等等。改用服务级别。
- `HAS_DBACCESS`  

- `KILL STATS JOB`  

- `OPENQUERY`、`OPENROWSET`、`OPENDATASOURCE`、`BULK INSERT` 和由四部分构成的名称
- .NET Framework [CLR 与 SQL Server 的集成](http://msdn.microsoft.com/zh-cn/library/ms254963.aspx)
- 语义搜索
- 服务器凭据。改用数据库范围的凭据。
- 服务器级项目：服务器角色、`IS_SRVROLEMEMBER`、`sys.login_token`. `GRANT`、`REVOKE` 和 `DENY` 的服务器级权限不可用，某些权限已替换为数据库级权限。一些有用的服务器级 DMV 具有等效的数据库级 DMV。
- `SET REMOTE_PROC_TRANSACTIONS`  

- `SHUTDOWN`
- `sp_addmessage`  

- `sp_configure` 选项和 `RECONFIGURE`。可以通过 [ALTER DATABASE SCOPED CONFIGURATION](https://msdn.microsoft.com/zh-cn/library/mt629158.aspx) 使用某些选项。
- `sp_helpuser`  

- `sp_migrate_user_to_contained`  

- SQL Server 审核。改用 SQL 数据库审核。
- SQL Server 跟踪
- 跟踪标志。某些跟踪标志项已移至兼容模式。
- Transact-SQL 调试
- 触发器：服务器作用域或登录触发器
- `USE` 语句：若要将数据库上下文更改为不同的数据库，必须与新数据库建立新连接。

## 完整的 Transact-SQL 引用

有关 Transact-SQL 语法、用法和示例的详细信息，请参阅 SQL Server 联机丛书中的 [Transact-SQL 引用（数据库引擎）](https://msdn.microsoft.com/zh-cn/library/bb510741.aspx)。

### 有关“适用于”标记

Transact-SQL 参考包含从 SQL Server 2008 到最新版本的相关主题。主题标题下面有一个图标栏，其中列出了四个 SQL Server 平台，并指明了适用性。例如，SQL Server 2012 中引入了可用性组。[创建可用性组](https://msdn.microsoft.com/zh-cn/library/ff878399.aspx)主题指明该语句适用于 **SQL Server（从版本 2012 开始）。该语句不适用于 SQL Server 2008、SQL Server 2008 R2、Azure SQL 数据库、Azure SQL 数据仓库或并行数据仓库。

在某些情况下，产品中可能使用了某个主题的常规主旨，但产品之间存在细微的差异。在适当的情况下，我们会在主题的中间位置指出差异。

<!---HONumber=Mooncake_0116_2017-->
<!--update: update supported T-sql syntex -->