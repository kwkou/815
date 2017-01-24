<properties
    pageTitle="Azure SQL 数据库功能概述 | Azure"
    description="本页概述 Azure SQL 数据库逻辑服务器和数据库，并提供带有每个列出功能的链接的功能支持矩阵。"
    services="sql-database"
    documentationcenter="na"
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="d1a46fa4-53d2-4d25-a0a7-92e8f9d70828"
    ms.service="sql-database"
    ms.custom="overview"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="data-management"
    ms.date="11/28/2016"
    wacn.date="01/20/2017"
    ms.author="carlrab; jognanay" />  


# Azure SQL 数据库功能
本主题概述 Azure SQL 数据库逻辑服务器和数据库，并提供带有每个列出功能的链接的功能支持矩阵。

## 什么是 Azure SQL 数据库逻辑服务器？
Azure SQL 数据库逻辑服务器充当多个数据库的中心管理点。在 SQL 数据库中，服务器是一个逻辑构造，它不同于在本地环境中可能很熟悉的 SQL Server 实例。具体而言，SQL 数据库服务对数据库相对于其逻辑服务器的位置不做出任何保证，并且不公开任何实例级访问权限或功能。有关 Azure SQL 逻辑服务器的详细信息，请参阅[逻辑服务器](/documentation/articles/sql-database-server-overview/)。

## 什么是 Azure SQL 数据库？
Azure SQL 数据库中的每个数据库都与逻辑服务器相关联。数据库可以是：

- 单一数据库，具有其[自己的资源集](/documentation/articles/sql-database-what-is-a-dtu/#what-are-database-transaction-units-dtus) (DTU)
- 属于[共享一组资源](/documentation/articles/sql-database-what-is-a-dtu/#what-are-elastic-database-transaction-units-edtus)的[数据库池](/documentation/articles/sql-database-elastic-pool/) (eDTU)
- 属于[向外扩展的分片数据库集](/documentation/articles/sql-database-elastic-scale-introduction/#horizontal-and-vertical-scaling)，可以是单一数据库或入池数据库
- 属于参与[多租户 SaaS 设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)的一组数据库，其数据库可以是单一数据库和/或入池数据库

有关 Azure SQL 数据库的详细信息，请参阅 [SQL 数据库](/documentation/articles/sql-database-overview/)。

## 支持哪些功能？

下表列出了 Azure SQL 数据库和 SQL Server 的主要功能，指定其可支持性，并提供每个平台上有关功能的详细信息的链接。对于 Transact-SQL 功能，请打开表中的链接了解功能的类别。另请参阅 [Azure SQL 数据库 Transact-SQL 差异](/documentation/articles/sql-database-transact-sql-information/)，了解关于不支持某些类型的功能的原因的更多背景资料。


> [AZURE.TIP]
若要测试现有数据库与 Azure SQL 数据库的兼容性，请参阅[验证 Azure SQL 数据库兼容性](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/)。
>

| **功能** | **SQL Server** | **Azure SQL 数据库** | 
| --- | :---: | :---: | 
| 活动异地复制 | 不支持 - 请参阅 [AlwaysOn 可用性组](https://msdn.microsoft.com/zh-cn/library/hh510230.aspx) | [支持](/documentation/articles/sql-database-geo-replication-overview/)
| 始终加密 | [支持](https://msdn.microsoft.com/zh-cn/library/mt163865.aspx) | [支持](/documentation/articles/sql-database-always-encrypted/) |
| AlwaysOn 可用性组 | [支持](https://msdn.microsoft.com/zh-cn/library/hh510230.aspx) | 不支持 - 请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/) |
| 附加数据库 | [支持](https://msdn.microsoft.com/zh-cn/library/ms190209.aspx) | 不支持 |
| 应用程序角色 | [支持](https://msdn.microsoft.com/zh-cn/library/ms190998.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/ms190998.aspx) |
| 自动缩放 | 不支持 | [支持](/documentation/articles/sql-database-scale-up/) |
| Azure Active Directory | 不支持 | [支持](/documentation/articles/sql-database-aad-authentication/) |
| 审核 | [支持](https://msdn.microsoft.com/zh-cn/library/cc280386.aspx) | [支持](/documentation/articles/sql-database-auditing-get-started/) |
| BACPAC 文件（导出） | [支持](https://msdn.microsoft.com/zh-cn/library/hh213241.aspx) | [支持](/documentation/articles/sql-database-export/) |
| BACPAC 文件（导入） | [支持](https://msdn.microsoft.com/zh-cn/library/hh710052.aspx) | [支持](/documentation/articles/sql-database-import/) |
| 备份和还原语句 | [支持](https://msdn.microsoft.com/zh-cn/library/ff848768.aspx) | 不支持 |
| 内置函数 | [支持](https://msdn.microsoft.com/zh-cn/library/ms174318.aspx) | [大多数](https://msdn.microsoft.com/zh-cn/library/ms174318.aspx) |
| 更改数据捕获 | [支持](https://msdn.microsoft.com/zh-cn/library/cc645937.aspx) | 不支持 |
| 更改跟踪 | [支持](https://msdn.microsoft.com/zh-cn/library/bb933875.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/bb933875.aspx) |
| 排序规则语句 | [支持](https://msdn.microsoft.com/zh-cn/library/ff848763.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/ff848763.aspx) |
| 列存储索引 | [支持](https://msdn.microsoft.com/zh-cn/library/gg492088.aspx) | [仅限 Premium Edition](https://msdn.microsoft.com/zh-cn/library/gg492088.aspx) |
| 公共语言运行时 (CLR) | [支持](https://msdn.microsoft.com/zh-cn/library/ms131102.aspx) | 不支持 |
| 包含的数据库 | [支持](https://msdn.microsoft.com/zh-cn/library/ff929071.aspx) | 内置 |
| 包含用户 | [支持](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx) | [支持](/documentation/articles/sql-database-manage-logins/#non-administrator-users) |
| 控制流语言关键字 | [支持](https://msdn.microsoft.com/zh-cn/library/ms174290.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/ms174290.aspx) |
| 跨数据库查询 | [支持](https://msdn.microsoft.com/zh-cn/library/dn584627.aspx) | [弹性查询](/documentation/articles/sql-database-elastic-query-overview/) |
| 游标 | [支持](https://msdn.microsoft.com/zh-cn/library/ms181441.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/ms181441.aspx) | 
| 数据压缩 | [支持](https://msdn.microsoft.com/zh-cn/library/cc280449.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/cc280449.aspx) |
| 数据库备份 | [已向用户公开](https://msdn.microsoft.com/zh-cn/library/ms187048.aspx) | [内置](/documentation/articles/sql-database-automated-backups/) |
| 数据库邮件 | [支持](https://msdn.microsoft.com/zh-cn/library/ms189635.aspx) | 不支持 |
| 数据库镜像 | [支持](https://msdn.microsoft.com/zh-cn/library/ms189852.aspx) | 不支持 |
| 数据库配置选项 | [支持](https://msdn.microsoft.com/zh-cn/library/mt629158.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/mt629158.aspx) |
| Data Quality Services (DQS) | [支持](https://msdn.microsoft.com/zh-cn/library/ff877925.aspx) | 不支持 |
| 数据库快照 | [支持](https://msdn.microsoft.com/zh-cn/library/ms175158.aspx) | 不支持 |
| 数据类型 | [支持](https://msdn.microsoft.com/zh-cn/library/ms187752.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/ms187752.aspx) |  
| DBCC 语句 | [全部](https://msdn.microsoft.com/zh-cn/library/ms188796.aspx) | [某些](https://msdn.microsoft.com/zh-cn/library/ms188796.aspx) |
| DDL 语句 | [支持](https://msdn.microsoft.com/zh-cn/library/ff848799.aspx) | [大多数](https://msdn.microsoft.com/zh-cn/library/ff848799.aspx)
| DDL 触发器 | [支持](https://msdn.microsoft.com/zh-cn/library/ms175941.aspx) | [仅数据库](https://msdn.microsoft.com/zh-cn/library/ms175941.aspx) |
| 分布式事务 | [MS DTC](https://msdn.microsoft.com/zh-cn/library/ms131665.aspx) | 仅限受限制的 SQL 数据库内方案 |
| DML 语句 | [支持](https://msdn.microsoft.com/zh-cn/library/ff848766.aspx) | [大多数](https://msdn.microsoft.com/zh-cn/library/ff848766.aspx) |
| DML 触发器 | [支持](https://msdn.microsoft.com/zh-cn/library/ms178110.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/ms178110.aspx) |
| DMV | [全部](https://msdn.microsoft.com/zh-cn/library/ms188754.aspx) | [某些](https://msdn.microsoft.com/zh-cn/library/ms188754.aspx) |
| 弹性池 | 不支持 | [支持](/documentation/articles/sql-database-elastic-pool/) |
| 弹性作业 | 不支持 - 请参阅 [SQL Server 代理](https://msdn.microsoft.com/zh-cn/library/ms189237.aspx) | 不支持 | 
| 弹性查询 | 不支持 - 请参阅[跨数据库查询](https://msdn.microsoft.com/zh-cn/library/dn584627.aspx) | [支持](/documentation/articles/sql-database-elastic-query-overview/) |
| 事件通知 | [支持](https://msdn.microsoft.com/zh-cn/library/ms186376.aspx) | [支持](/documentation/articles/sql-database-insights-alerts-portal/) |
| 表达式 | [支持](https://msdn.microsoft.com/zh-cn/library/ms190286.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/ms190286.aspx) |
| 扩展的事件 | [支持](https://msdn.microsoft.com/zh-cn/library/bb630282.aspx) | [某些](/documentation/articles/sql-database-xevent-db-diff-from-svr/) |
| 扩展的存储过程 | [支持](https://msdn.microsoft.com/zh-cn/library/ms164627.aspx) | 不支持 |
| 文件组 | [支持](https://msdn.microsoft.com/zh-cn/library/ms189563.aspx#Anchor_2) | [仅限主要](https://msdn.microsoft.com/zh-cn/library/ms189563.aspx#Anchor_2) |
| 文件流 | [支持](https://msdn.microsoft.com/zh-cn/library/gg471497.aspx) | 不支持 |
| 全文搜索 | [支持](https://msdn.microsoft.com/zh-cn/library/ms142571.aspx) | [不支持第三方断字符](https://msdn.microsoft.com/zh-cn/library/ms142571.aspx) |
| 函数 | [支持](https://msdn.microsoft.com/zh-cn/library/ms174318.aspx) | [大多数](https://msdn.microsoft.com/zh-cn/library/ms174318.aspx) |
| 内存中优化 | [支持](https://msdn.microsoft.com/zh-cn/library/dn133186.aspx) | [仅限 Premium Edition](https://msdn.microsoft.com/zh-cn/library/dn133186.aspx) |
| 作业 | [SQL Server 代理](https://msdn.microsoft.com/zh-cn/library/ms189237.aspx) | 不支持 |
| JSON 数据支持 | [支持](https://msdn.microsoft.com/zh-cn/library/dn921897.aspx) | [支持](/documentation/articles/sql-database-json-features/) |
| 语言元素 | [支持](https://msdn.microsoft.com/zh-cn/library/ff848807.aspx) | [大多数](https://msdn.microsoft.com/zh-cn/library/ff848807.aspx) |  
| 链接的服务器 | [支持](https://msdn.microsoft.com/zh-cn/library/ms188279.aspx) | 不支持 - 请参阅[弹性查询](/documentation/articles/sql-database-elastic-query-horizontal-partitioning/) |
| 日志传送 | [支持](https://msdn.microsoft.com/zh-cn/library/ms187103.aspx) | 不支持 - 请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/) |
| 管理命令 | [支持](https://msdn.microsoft.com/zh-cn/library/ms190286.aspx)| [不支持](https://msdn.microsoft.com/zh-cn/library/ms190286.aspx) |
| Master Data Services (MDS) | [支持](https://msdn.microsoft.com/zh-cn/library/ff487003.aspx) | 不支持 |
| 批量导入时的最小日志记录 | [支持](https://msdn.microsoft.com/zh-cn/library/ms190422.aspx) | 不支持 |
| 修改系统数据 | [支持](https://msdn.microsoft.com/zh-cn/library/ms178028.aspx) | 不支持 |
| 联机索引操作 | [支持](https://msdn.microsoft.com/zh-cn/library/ms177442.aspx) | [受服务层限制的事务大小](https://msdn.microsoft.com/zh-cn/library/ms177442.aspx) |
| 运算符 | [支持](https://msdn.microsoft.com/zh-cn/library/ms174986.aspx) | [大多数](https://msdn.microsoft.com/zh-cn/library/ms174986.aspx) |
| 数据库时间点还原 | [支持](https://msdn.microsoft.com/zh-cn/library/ms179451.aspx) | [支持](/documentation/articles/sql-database-recovery-using-backups/#point-in-time-restore) |
| Polybase | [支持](https://msdn.microsoft.com/zh-cn/library/mt143171.aspx) | [不支持]
| 基于策略的管理 | [支持](https://msdn.microsoft.com/zh-cn/library/bb510667.aspx) | 不支持 |
| 谓词 | [支持](https://msdn.microsoft.com/zh-cn/library/ms189523.aspx) | [大多数](https://msdn.microsoft.com/zh-cn/library/ms189523.aspx)
| 资源调控器 | [支持](https://msdn.microsoft.com/zh-cn/library/bb933866.aspx) | [内置](/documentation/articles/sql-database-service-tiers/) |
| 从备份还原数据库 | [支持](https://msdn.microsoft.com/zh-cn/library/ms187048.aspx#anchor_6) | [仅限从内置备份](/documentation/articles/sql-database-recovery-using-backups/) |
| 行级别安全性 | [支持](https://msdn.microsoft.com/zh-cn/library/dn765131.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/dn765131.aspx) |
| 安全语句 | [支持](https://msdn.microsoft.com/zh-cn/library/ff848791.aspx) | [某些](https://msdn.microsoft.com/zh-cn/library/ff848791.aspx) |
| 语义搜索 | [支持](https://msdn.microsoft.com/zh-cn/library/gg492075.aspx) | 不支持 |
| 序列号 | [支持](https://msdn.microsoft.com/zh-cn/library/ff878058.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/ff878058.aspx) |
| 服务中转站 | [支持](https://msdn.microsoft.com/zh-cn/library/bb522893.aspx) | 不支持 |
| 服务器配置选项 | [支持](https://msdn.microsoft.com/zh-cn/library/ms189631.aspx) | 不支持 - 请参阅[数据库配置选项](https://msdn.microsoft.com/zh-cn/library/mt629158.aspx) |
| Set 语句 | [支持](https://msdn.microsoft.com/zh-cn/library/ms190356.aspx) | [大多数](https://msdn.microsoft.com/zh-cn/library/ms190356.aspx) 
| 空间 | [支持](https://msdn.microsoft.com/zh-cn/library/bb933790.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/bb933790.aspx) |
| SQL Server 代理 | [支持](https://msdn.microsoft.com/zh-cn/library/ms189237.aspx) | 不支持 |
| SQL Server Analysis Services (SSAS) | [支持](https://msdn.microsoft.com/zh-cn/library/bb522607.aspx) | 不支持 - 请参阅 [Azure Analysis Services](https://azure.microsoft.com/services/analysis-services/) |
| SQL Server 集成服务 (SSIS) | [支持](https://msdn.microsoft.com/zh-cn/library/ms141026.aspx) | 不支持 - 请参阅 [Azure 数据工厂](https://azure.microsoft.com/services/data-factory/) |
| SQL Server PowerShell | [支持](https://msdn.microsoft.com/zh-cn/library/hh245198.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/hh245198.aspx) |
| SQL Server 事件探查器 | [支持](https://msdn.microsoft.com/zh-cn/library/ms181091.aspx) | 不支持 - 请参阅[扩展事件](https://msdn.microsoft.com/zh-cn/library/ms181091.aspx) |
| SQL Server 复制 | [支持](https://msdn.microsoft.com/zh-cn/library/ms151198.aspx) | [仅限事务复制和快照复制订阅服务器](/documentation/articles/sql-database-cloud-migrate-compatible-using-transactional-replication/) |
| SQL Server Reporting Services (SSRS) | [支持](https://msdn.microsoft.com/zh-cn/library/ms159106.aspx) | 不支持 |
| 存储过程 | [支持](https://msdn.microsoft.com/zh-cn/library/ms190782.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/ms190782.aspx) |
| 系统存储函数 | [支持](https://msdn.microsoft.com/zh-cn/library/ff848780.aspx) | [某些](https://msdn.microsoft.com/zh-cn/library/ff848780.aspx) |
| 系统存储过程 | [支持](https://msdn.microsoft.com/zh-cn/library/ms187961.aspx) | [某些](https://msdn.microsoft.com/zh-cn/library/ms187961.aspx) |
| 系统表 | [支持](https://msdn.microsoft.com/zh-cn/library/ms179932.aspx) | [某些](https://msdn.microsoft.com/zh-cn/library/ms179932.aspx) |
| 系统视图 | [支持](https://msdn.microsoft.com/zh-cn/library/ms177862.aspx) | [某些](https://msdn.microsoft.com/zh-cn/library/ms177862.aspx)
| 表分区 | [支持](https://msdn.microsoft.com/zh-cn/library/ms190787.aspx) | [仅限主文件组](https://msdn.microsoft.com/zh-cn/library/ms190787.aspx) |
| 临时表 | [本地和全局](https://msdn.microsoft.com/zh-cn/library/ms174979.aspx#Anchor_4) | [仅限本地](https://msdn.microsoft.com/zh-cn/library/ms174979.aspx#Anchor_4) |
| 临时表 | [支持](https://msdn.microsoft.com/zh-cn/library/dn935015.aspx) | [支持](/documentation/articles/sql-database-temporal-tables/) |
| 事务语句 | [支持](https://msdn.microsoft.com/zh-cn/library/ms174377.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/ms174377.aspx) |
| 变量 | [支持](https://msdn.microsoft.com/zh-cn/library/ff848809.aspx) | | [支持](https://msdn.microsoft.com/zh-cn/library/ff848809.aspx) | 
| 透明数据加密 (TDE) | [支持](https://msdn.microsoft.com/zh-cn/library/bb934049.aspx) | [支持](https://msdn.microsoft.com/dn948096.aspx) |
| Windows Server 故障转移群集 | [支持](https://msdn.microsoft.com/zh-cn/library/hh270278.aspx) | 不支持 - 请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/) |
| XML 索引 | [支持](http://msdn.microsoft.com/zh-cn/library/bb934097.aspx) | [支持](http://msdn.microsoft.com/zh-cn/library/bb934097.aspx) |
| XML 语句 | [支持](https://msdn.microsoft.com/zh-cn/library/ff848798.aspx) | [支持](https://msdn.microsoft.com/zh-cn/library/ff848798.aspx) |

## 后续步骤

- 有关 Azure SQL 数据库服务的信息，请参阅[什么是 SQL 数据库？](/documentation/articles/sql-database-technical-overview/)
- 有关 Azure SQL 逻辑服务器的概述，请参阅 [SQL 数据库逻辑服务器概述](/documentation/articles/sql-database-server-overview/)
- 有关 Azure SQL 数据库的概述，请参阅 [SQL 数据库概述](/documentation/articles/sql-database-overview/)
- 有关 Transact-SQL 支持和差异的信息，请参阅 [Azure SQL 数据库 Transact-SQL 差异](/documentation/articles/sql-database-transact-sql-information/)。
- 基于**服务层**，了解有关特定资源配额和限制的信息。有关服务层的概述，请参阅 [SQL 数据库服务层](/documentation/articles/sql-database-service-tiers/)。
- 有关与安全相关的指导原则，请参阅 [Azure SQL 数据库安全指导原则和限制](/documentation/articles/sql-database-security-guidelines/)。
- 有关驱动程序可用性和 SQL 数据库支持的信息，请参阅 [用于 SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries/)。

<!---HONumber=Mooncake_0116_2017-->