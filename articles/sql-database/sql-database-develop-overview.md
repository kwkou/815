<properties
	pageTitle="SQL 数据库开发概述 | Azure"
	description="了解连接到 SQL 数据库的应用程序的可用连接库和最佳实践。"
	services="sql-database"
	documentationCenter=""
	authors="annemill"
	manager="jhubbard"
	editor="genemi"/>


<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/17/2016"
	wacn.date="12/26/2016"
	ms.author="annemill"/>  


# SQL 数据库开发概述
本文逐步讲解开发人员在编写代码以连接到 Azure SQL 数据库时应考虑的基本注意事项。

## 语言和平台
针对各种编程语言和平台提供了代码示例。可在以下位置找到代码示例的链接：

* 详细信息：[用于 SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries/)

## 资源限制
Azure SQL 数据库使用两种不同的机制管理可用于数据库的资源：资源调控和强制限制。

* 详细信息：[Azure SQL 数据库资源限制](/documentation/articles/sql-database-resource-limits/)

## 安全性
Azure SQL 数据库提供资源，用于在 SQL 数据库中限制访问、保护数据和监视活动。

* 详细信息：[保护 SQL 数据库的安全](/documentation/articles/sql-database-security/)

## 身份验证
* Azure SQL 数据库支持 SQL Server 身份验证用户和登录名，以及 [Azure Active Directory 身份验证](/documentation/articles/sql-database-aad-authentication/)用户和登录名。
* 需要指定一个特定的数据库，而不是默认使用 *master* 数据库。
* 无法在 SQL 数据库上使用 Transact-SQL **USE myDatabaseName;** 语句切换到其他数据库。
* 详细信息：[SQL 数据库安全：管理数据库的访问和登录安全](/documentation/articles/sql-database-manage-logins/)

## 复原能力
如果在连接到 SQL 数据库时发生暂时性错误，则代码应重试调用。建议让重试逻辑使用退让逻辑，这样就不会因为多个客户端同时重试而对 SQL 数据库造成混乱。

* 代码示例：有关演示重试逻辑的代码示例，请在以下位置参阅所选语言的示例：[用于 SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries/)
* 详细信息：[SQL 数据库客户端程序的错误消息](/documentation/articles/sql-database-develop-error-messages/)

## 管理连接
* 在客户端连接逻辑中，将默认超时替换为 30 秒。默认值 15 秒对于依赖于 Internet 的连接而言太短。
* 如果正在使用[连接池](http://msdn.microsoft.com/zh-cn/library/8xx3tyca.aspx)，请确保在程序不活跃地使用连接时立即将其关闭，且不要准备重用它。

## 网络注意事项
* 在托管客户端程序的计算机上，确保防火墙允许端口 1433 上的传出 TCP 通信。详细信息：[配置 Azure SQL 数据库防火墙](/documentation/articles/sql-database-configure-firewall-settings/)
* 如果客户端程序连接到 SQL 数据库 V12，而客户端在 Azure 虚拟机 (VM) 上运行，则必须打开 VM 上的某些端口范围。详细信息：[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)
* 与 Azure SQL 数据库 V12 建立的客户端连接有时会绕过代理直接与数据库交互。除 1433 以外的端口变得非常重要。详细信息：[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)

## 具有 Elastic Scale 的数据分片
Elastic Scale 简化了扩大（和缩小）过程。

* [包含 Azure SQL 数据库的多租户 SaaS 应用程序的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)
* [依赖于数据的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)
* [Azure SQL 数据库 Elastic Scale 预览版入门](/documentation/articles/sql-database-elastic-scale-get-started/)

## 后续步骤

浏览所有 [SQL 数据库的功能](/home/features/sql-database/)。

<!---HONumber=Mooncake_Quality_Review_1215_2016-->