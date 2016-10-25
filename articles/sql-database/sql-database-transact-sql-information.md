<properties
   pageTitle="Azure SQL 数据库 T-SQL 不支持的语句 | Azure"
   description="在 Azure SQL 数据库中不完全支持的 Transact-SQL 语句"
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
   ms.date="08/30/2016"
   wacn.date="10/17/2016"
   ms.author="rick.byham@microsoft.com"/>  


# Azure SQL 数据库 Transact-SQL 的差异


Microsoft SQL Server 和 Azure SQL 数据库都支持应用程序依赖的大多数 Transact-SQL 功能。下面是应用程序支持的功能的不完整列表：

- 数据类型。
- 运算符。
- 字符串、算术、逻辑和游标函数。

但是，Azure SQL 数据库会将功能与 **master** 数据库上的任何依赖项相隔离。因此，许多服务器级活动并不适用于 SQL 数据库，也不受 SQL 数据库的支持。在 SQL Server 中弃用的功能一般不受 SQL 数据库的支持。

> [AZURE.NOTE]
本主题介绍 SQL 数据库升级到最新版 SQL 数据库 V12 后可用的功能。有关 V12 的详细信息，请参阅 [SQL 数据库 V12 新增功能](/documentation/articles/sql-database-v12-whats-new/)。

以下部分列出了部分受支持的功能，以及完全不受支持的功能。


## SQL 数据库 V12 部分支持的功能

SQL 数据库 V12 支持相应 SQL Server 2016 Transact-SQL 语句中存在的某些（但不是所有）参数。例如，CREATE PROCEDURE 语句可用，但 CREATE PROCEDURE 的所有选项不可用。请参阅链接的语法主题，了解有关每个语句的受支持区域的详细信息。

- 数据库：[CREATE](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx)/[ALTER](https://msdn.microsoft.com/zh-cn/library/ms174269.aspx)
- DMV 通常适用于可用的功能。
- 函数：[CREATE](https://msdn.microsoft.com/zh-cn/library/ms186755.aspx)/[ALTER FUNCTION](https://msdn.microsoft.com/zh-cn/library/ms186967.aspx)
- [KILL](https://msdn.microsoft.com/zh-cn/library/ms173730.aspx)
- 登录名：[CREATE](https://msdn.microsoft.com/zh-cn/library/ms189751.aspx)/[ALTER LOGIN](https://msdn.microsoft.com/zh-cn/library/ms189828.aspx)
- 存储过程：[CREATE](https://msdn.microsoft.com/zh-cn/library/ms187926.aspx)/[ALTER PROCEDURE](https://msdn.microsoft.com/zh-cn/library/ms189762.aspx)
- 表：[CREATE](https://msdn.microsoft.com/zh-cn/library/dn305849.aspx)/[ALTER](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)
- 类型（自定义）：[CREATE TYPE](https://msdn.microsoft.com/zh-cn/library/ms175007.aspx)
- 用户：[CREATE](https://msdn.microsoft.com/zh-cn/library/ms173463.aspx)/[ALTER USER](https://msdn.microsoft.com/zh-cn/library/ms176060.aspx)
- 视图：[CREATE](https://msdn.microsoft.com/zh-cn/library/ms187956.aspx)/[ALTER VIEW](https://msdn.microsoft.com/zh-cn/library/ms173846.aspx)

## SQL 数据库不支持的功能

- 系统对象的排序规则
- 连接相关：终结点语句、ORIGINAL\_DB\_NAME。SQL 数据库不支持 Windows 身份验证，但支持类似的 Azure Active Directory 身份验证。某些身份验证类型要求使用最新版本的 SSMS。有关详细信息，请参阅 [Connecting to SQL Database or SQL Data Warehouse By Using Azure Active Directory Authentication](/documentation/articles/sql-database-aad-authentication/)（使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库）。
- 使用三个或四个部分名称的跨数据库查询。（使用[弹性数据库查询](/documentation/articles/sql-database-elastic-query-overview/)支持只读跨数据库查询。）
- 跨数据库所有权链接, TRUSTWORTHY 设置
- 数据收集器
- 数据库关系图
- 数据库邮件
- DATABASEPROPERTY（改用 DATABASEPROPERTYEX）
- EXECUTE AS 登录名
- 加密：可扩展密钥管理
- 事件：事件、事件通知、查询通知
- 与数据库文件定位、大小以及 Azure 自动管理的数据库文件相关的功能。
- 与通过 Azure 帐户管理的高可用性相关的功能：备份、还原、AlwaysOn、数据库镜像、日志传送、恢复模式。有关详细信息，请参阅“Azure SQL 数据库备份和还原”。
- 依赖于在 SQL 数据库上运行的日志读取器的功能：推送复制、更改数据捕获。
- 依赖于 SQL Server 代理或 MSDB 数据库的功能：作业、警报、运算符、基于策略的管理、数据库邮件、中心管理服务器。
- FILESTREAM
- 函数：fn\_get\_sql、fn\_virtualfilestats、fn\_virtualservernodes
- 全局临时表
- 硬件相关的服务器设置：内存、工作线程、CPU 相关性、跟踪标志，等等。改用服务级别。
- HAS\_DBACCESS
- KILL STATS JOB
- 链接的服务器、OPENQUERY、OPENROWSET、OPENDATASOURCE、BULK INSERT、包含四个部分的名称
- 主/目标服务器
- .NET Framework [CLR 与 SQL Server 的集成](http://msdn.microsoft.com/zh-cn/library/ms254963.aspx)
- 资源调控器
- 语义搜索
- 服务器凭据。改用数据库范围的凭据。
- 服务器级项：服务器角色，IS\_SRVROLEMEMBER、sys.login\_token。服务器级权限不可用，有些权限已替换为数据库级权限。某些服务器级 DMV 不可用，有些 DMV 已替换为数据库级 DMV。
- Serverless express：localdb、用户实例
- 服务代理
- SET REMOTE\_PROC\_TRANSACTIONS
- 关机
- sp\_addmessage
- sp\_configure 选项和 RECONFIGURE。可以通过 [ALTER DATABASE SCOPED CONFIGURATION](https://msdn.microsoft.com/zh-cn/library/mt629158.aspx) 使用某些选项。
- sp\_helpuser
- sp\_migrate\_user\_to\_contained
- SQL Server 审核。改用 SQL 数据库审核。
- SQL Server 事件探查器
- SQL Server 跟踪
- 跟踪标志。某些跟踪标志项已移至兼容模式。
- Transact-SQL 调试
- 触发器：服务器作用域或登录触发器
- USE 语句：若要将数据库上下文更改为不同的数据库，必须与新数据库建立新连接。


## 完整 Transact-SQL 参考

有关 Transact-SQL 语法、用法和示例的详细信息，请参阅 SQL Server 联机丛书中的 [Transact-SQL 参考（数据库引擎）](https://msdn.microsoft.com/zh-cn/library/bb510741.aspx)。

### 有关“适用于”标记

Transact-SQL 参考包含从 SQL Server 2008 到最新版本的相关主题。主题标题下面有一个图标栏，其中列出了四个 SQL Server 平台，并指明了适用性。例如，SQL Server 2012 中引入了可用性组。[创建可用性组](https://msdn.microsoft.com/zh-cn/library/ff878399.aspx)主题指明该语句适用于 **SQL Server（从版本 2012 开始）。该语句不适用于 SQL Server 2008、SQL Server 2008 R2、Azure SQL 数据库、Azure SQL 数据仓库或并行数据仓库。

在某些情况下，产品中可能使用了某个主题的常规主旨，但产品之间存在细微的差异。在适当的情况下，我们会在主题的中间位置指出差异。

<!---HONumber=Mooncake_1010_2016-->