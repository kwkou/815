<properties
   pageTitle="Azure SQL 数据库 T-SQL 不支持的语句 | Azure"
   description="在 Azure SQL 数据库中不完全支持的 Transact-SQL 语句"
   services="sql-database"
   documentationCenter=""
   authors="BYHAM"
   manager="jeffreyg"
   editor=""
   tags=""/>

<tags
   ms.service="sql-database"
   ms.date="02/18/2016"
   wacn.date="06/14/2016"/>

# Azure SQL 数据库 Transact-SQL 的差异


Microsoft SQL Server 和 Azure SQL 数据库都支持应用程序依赖的大多数 Transact-SQL 功能。下面是应用程序支持的功能的不完整列表：

- 数据类型。
- 运算符。
- 字符串、算术、逻辑和游标函数。

但是，Azure SQL 数据库会将功能与 **master** 数据库上的任何依赖项相隔离。因此，许多服务器级活动并不适用于 SQL 数据库，也不受 SQL 数据库的支持。本主题将详细说明 SQL 数据库不完全支持的功能。

此外，在 SQL Server 中弃用的功能一般不受 SQL 数据库的支持。

## 升级到 SQL 数据库 V12

本主题介绍 SQL 数据库升级到免费版 SQL 数据库 V12 后可用的功能。有关 V12 的详细信息，请参阅 [SQL 数据库 V12 新增功能](/documentation/articles/sql-database-v12-whats-new/)。SQL 数据库 V12 增加了性能和可管理性改进功能，以及对其他功能的支持。增加的功能在下面列出，分为完全支持的功能以及部分支持的功能。

## SQL 数据库 V12 部分支持的功能

SQL 数据库 V12 支持相应 SQL Server 2016 Transact-SQL 语句中存在的某些（但不是所有）参数。例如，CREATE PROCEDURE 语句可用，但 CREATE PROCEDURE 的 WITH ENCRYPTION 选项不可用。请参阅链接的语法主题，以了解有关每个语句的受支持区域的详细信息。

- 数据库：[CREATE](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx)/[ALTER](https://msdn.microsoft.com/zh-cn/library/ms174269.aspx)
- DMV 通常适用于可用的功能
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
- 连接相关：终结点语句、ORIGINAL\_DB\_NAME。Windows 身份验证不可用于登录名或包含的数据库用户。
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
- 依赖于日志读取器的功能：复制、更改数据捕获。
- 依赖于 SQL Server 代理或 MSDB 数据库的功能：作业、警报、运算符、基于策略的管理、数据库邮件、中心管理服务器。
- FILESTREAM
- 函数：fn\_get\_sql、fn\_virtualfilestats、fn\_virtualservernodes
- 全局临时表
- 硬件相关的服务器设置：内存、工作线程、CPU 相关性、跟踪标志，等等。改用服务级别。
- HAS\_DBACCESS
- KILL STATS JOB
- 链接的服务器、OPENQUERY、OPENROWSET、OPENDATASOURCE、BULK INSERT、包含 3 个和 4 个部分的名称
- 主/目标服务器
- .NET Framework [CLR 与 SQL Server 的集成](http://msdn.microsoft.com/zh-cn/library/ms254963.aspx)
- 资源调控器
- 语义搜索
- 服务器凭据
- 服务器级项：服务器角色，IS\_SRVROLEMEMBER、sys.login\_token。服务器级权限不可用，有些权限已替换为数据库级权限。某些服务器级 DMV 不可用，有些 DMV 已替换为数据库级 DMV。
- Serverless express：localdb、用户实例
- 服务代理
- SET REMOTE\_PROC\_TRANSACTIONS
- 关机
- sp\_addmessage
- sp\_configure 选项和 RECONFIGURE
- sp\_helpuser
- sp\_migrate\_user\_to\_contained
- SQL Server 审核（改用 SQL 数据库审核）
- SQL Server 事件探查器
- SQL Server 跟踪
- 跟踪标志
- Transact-SQL 调试
- 触发器：服务器作用域或登录触发器
- USE 语句

## 完整 Transact-SQL 参考

有关 Transact-SQL 语法、用法和示例的详细信息，请参阅 SQL Server 联机丛书中的 [Transact-SQL 参考（数据库引擎）](https://msdn.microsoft.com/zh-cn/library/bb510741.aspx)。

### 有关“适用于”标记

Transact-SQL 参考包含从 SQL Server 2008 到最新版本的相关主题。主题标题下面通常有一个“适用于”行，其中列出了 SQL Server 版本，此外也可能列出了其他产品名称。通常，相同的“适用于”标记也会列出 Azure SQL 数据库。如果“适用于”中未列出 Azure SQL 数据库，则表示该主题内容不适用于 Azure SQL 数据库。有时你可能会看到“适用于”行列出了多个产品，每个产品都附带一个小图标用于指示该主题是否适用于每个产品。

 例如，SQL Server 2012 中引入了可用性组。**创建可用性组**主题表示它适用于 **SQL Server（SQL Server 2012 至当前版本）**，因为它不适用于 SQL Server 2008、SQL Server 2008 R2 或 Azure SQL 数据库。

在某些情况下，产品中可能使用了某个主题的常规主旨，但产品之间存在细微的差异。在适当的情况下，我们会在主题的中间位置指出差异。


<!---HONumber=Mooncake_0606_2016-->
