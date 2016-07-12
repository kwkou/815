<properties 
	pageTitle="SQL 数据库 V12 的新增功能 | Azure" 
	description="介绍云中使用 Azure SQL 数据库的业务系统在升级到版本 V12 后为何能够受益。" 
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jhubbard" 
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.date="03/21/2016" 
	wacn.date="04/18/2016"/>


# SQL 数据库 V12 的新增功能


本主题介绍 Azure SQL 数据库新版本 V12 相比版本 V11 具有的诸多优点。


我们继续向 V12 添加功能。 有关 SQL 数据库资源限制的最新信息：[Azure SQL 数据库资源限制](/documentation/articles/sql-database-resource-limits/)。


## 提高了与 SQL Server 的应用程序兼容性


SQL 数据库 V12 的主要目标是提高与 Microsoft SQL Server 2014 的兼容性，以及在发布新版本的 SQL Server 时保持此兼容性。另外，在编程性方面，V12 能够与 SQL Server 兼容。例如：


- 包括 [OVER](http://msdn.microsoft.com/zh-cn/library/ms189461.aspx) 的[开窗函数](http://msdn.microsoft.com/zh-cn/library/ms189798.aspx) 

- [XML 索引](http://msdn.microsoft.com/zh-cn/library/bb934097.aspx)和[选择性 XML 索引](http://msdn.microsoft.com/zh-cn/library/jj670104.aspx)

- [更改跟踪](http://msdn.microsoft.com/zh-cn/library/bb933875.aspx)

- [SELECT...INTO](http://msdn.microsoft.com/zh-cn/library/ms188029.aspx)

- [全文搜索](http://msdn.microsoft.com/zh-cn/library/ms142571.aspx)

- [更改数据库范围的配置 (TRANSACT-SQL)](http://msdn.microsoft.com/zh-cn/library/mt629158.aspx)

有关 SQL 数据库尚不支持的少部分功能，请参阅[此处](/documentation/articles/sql-database-transact-sql-information/)。


## 更高级的性能，全新的性能级别


在 V12 中，我们将分配给所有“高级”性能级别的数据库事务单位 (DTU) 提高了 25% 且不收取额外的费用。使用如下所述的新功能可以进一步提高性能：


- 支持内存中[列存储索引](http://msdn.microsoft.com/zh-cn/library/gg492153.aspx)。
- 使用 [TRUNCATE TABLE](http://msdn.microsoft.com/zh-cn/library/ms177570.aspx) 的相关增强功能，[按行进行表分区](http://msdn.microsoft.com/zh-cn/library/ms187802.aspx)。
- 可以使用动态管理视图 [(DMV)](http://msdn.microsoft.com/zh-cn/library/ms188754.aspx) 来帮助监视和优化性能。


### 可靠的性能


如果客户端程序连接到 SQL 数据库 V12，而客户端运行在 Azure 虚拟机 (VM) 上，则必须打开 VM 上的以下端口范围：

- 11000-11999
- 14000-14999


单击[此处](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)可了解有关 SQL 数据库 V12 的端口的详细信息。
SQL 数据库 V12 中的性能增强功能需要这些端口。


## 更好支持云 SaaS 供应商


我们只在 V12 中发布了新的标准性能级别 S3 与[弹性数据库池](/documentation/articles/sql-database-elastic-pool/)公共预览版。这是专门为云 SaaS 供应商设计的解决方案。使用弹性数据库池，你可以：


- 在数据库之间共享 DTU，以降低部署大量数据库所带来的成本。
- 执行[弹性数据库作业](/documentation/articles/sql-database-elastic-jobs-overview/)大规模管理数据库。


## 安全增强


对于在云中运营业务的任何人来说，安全性是一个主要考虑因素。V12 中发布的最新安全功能包括：


- [行级安全性](http://msdn.microsoft.com/zh-cn/library/dn765131.aspx) (RLS)
- [动态数据屏蔽](/documentation/articles/sql-database-dynamic-data-masking-get-started-portal/)
- [包含的数据库](http://msdn.microsoft.com/zh-cn/library/ff929188.aspx)
- 使用 GRANT、DENY、REVOKE 管理的[应用程序角色](http://msdn.microsoft.com/zh-cn/library/ms190998.aspx)
- [透明数据加密](http://msdn.microsoft.com/zh-cn/library/0bf7e8ff-1416-4923-9c4c-49341e208c62.aspx) (TDE)
- [通过使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)
 - SQL 数据库现在支持 Azure Active Directory 身份验证，该身份验证是使用 Azure Active Directory (Azure AD) 中的标识连接到 SQL 数据库的一种机制。通过 Azure Active Directory 身份验证，可以在一个中心位置中集中管理数据库用户和其他 Azure 服务的标识。
- [始终加密](https://msdn.microsoft.com/zh-cn/library/mt163865.aspx)（预览版）使得加密对于应用程序来说是透明的，从而让客户端能够加密客户端应用程序中的敏感数据，不必与 SQL 数据库共享加密密钥。


## 在需要恢复时提高业务连续性


V12 提供大幅提高的恢复点目标 (RPO) 与预计恢复时间 (ERT)：


| 业务连续性功能 | 早期版本 | V12 |
| :-- | :-- | :-- |
| 异地还原 | • RPO < 24 小时。<br/>• ERT < 12 小时。 | • RPO < 1 小时。<br/>• ERT < 12 小时。 |
| 标准异地复制 | • RPO < 30 分钟。<br/>• ERT < 2 小时。 | • RPO < 5 秒。<br/>• ERT < 30 秒。 |
| 活动异地复制 | • RPO < 5 分钟。<br/>• ERT < 1 小时。 | • RPO < 5 秒。<br/>• ERT < 30 秒。 |


有关详细信息，请参阅 [SQL 数据库业务连续](/documentation/articles/sql-database-business-continuity/)。


## 立即升级的更多原因


我们可以给出很多有说服力的理由来劝告客户立即从 V11 升级到 Azure SQL 数据库 V12：


- 与 V11 相比，SQL 数据库 V12 中增加了许多新的功能。
- 我们将持续在 V12 中新增功能，但是 V11 不会再新增功能。
- 大多数新功能都会先在 SQL 数据库 V12 中发布，然后再发布到 Microsoft SQL Server。


## 已开始使用 V12 了吗？


执行以下操作可以轻松判断你是否在早期版本的 SQL 数据库服务上运行数据库或逻辑服务器，是在数据库中运行 `SELECT @@version;` 语句，然后查看类似于下面的结果：


- **12**.0.2000.10 &nbsp; (版本 V12)
- **11**.0.9228.18 &nbsp; (版本 V11)


V12 数据库只能托管在 V12 逻辑服务器上。V12 服务器只能托管 V12 数据库。



## <a name="V12AzureSqlDbPreviewGaTable"></a> 正式版上市区域


- 2015 年 7 月 31，所有区域都已发布正式版 (GA)。
- V12 已于 2014 年 12 月发布，但仅以预览版提供。

<!---HONumber=Mooncake_0411_2016-->
