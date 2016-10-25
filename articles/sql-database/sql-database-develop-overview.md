<properties
	pageTitle="SQL 数据库开发概述 | Azure"
	description="了解可用于连接到 SQL 数据库的连接库和最佳实践。"
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
	wacn.date="10/17/2016"
	ms.author="annemill"/>  


# SQL 数据库开发概述
本文逐步讲解开发人员在编写代码以连接到 Azure SQL 数据库时应考虑的基本注意事项。

## 语言和平台
为各种编程语言和平台提供了代码示例。可在以下位置找到代码示例的链接：

* 详细信息：[Connection libraries for SQL Database and SQL Server（用于 SQL 数据库和 SQL Server 的连接库）](/documentation/articles/sql-database-libraries)

## 资源限制
Azure SQL 数据库使用两种不同的机制管理可用于数据库的资源：资源调控和强制限制。

* 详细信息：[Azure SQL Database resource limits（Azure SQL 数据库资源限制）](/documentation/articles/sql-database-resource-limits)

## 安全性
Azure SQL 数据库提供用于在 SQL 数据库中限制访问、保护数据和监视活动的资源。

* 详细信息：[Securing your SQL Database（保护 SQL 数据库）](/documentation/articles/sql-database-security)

## 身份验证
* Azure SQL 数据库支持 SQL Server 身份验证用户和登录名，以及 [Azure Active Directory 身份验证](/documentation/articles/sql-database-aad-authentication/)用户和登录名。
* 需要指定一个特定的数据库，而不要默认使用 *master* 数据库。
* 你无法使用 SQL 数据库上的 Transact-SQL **USE myDatabaseName;** 语句切换到其他数据库。
* 详细信息：[SQL Database security: Manage database access and login security（SQL 数据库安全：管理数据库的访问和登录安全）](/documentation/articles/sql-database-manage-logins)

## 复原能力
如果在连接到 SQL 数据库时发生暂时性错误，你的代码应重试调用。建议让重试逻辑使用退让逻辑，这样就不会因为多个客户端同时重试而对 SQL 数据库造成混乱。

* 代码示例：有关演示重试逻辑的代码示例，请在以下位置参阅所选语言的示例：[Connection libraries for SQL Database and SQL Server（用于 SQL 数据库和 SQL Server 的连接库）](/documentation/articles/sql-database-libraries)
* 详细信息：[Error messages for SQL Database client programs（SQL 数据库客户端程序的错误消息）](/documentation/articles/sql-database-develop-error-messages)

## 管理连接
* 在你的客户端连接逻辑中，将默认超时替换为 30 秒。默认值 15 秒对于依赖于 Internet 的连接而言太短。
* 如果你在使用[连接池](http://msdn.microsoft.com/zh-cn/library/8xx3tyca.aspx)，请确保在程序不活跃地使用连接时将其关闭，而不是准备重用它。

## 网络注意事项
* 在托管你的客户端程序的计算机上，确保防火墙允许端口 1433 上的传出 TCP 通信。详细信息：[Configure an Azure SQL Database firewall（配置 Azure SQL 数据库防火墙）](/documentation/articles/sql-database-configure-firewall-settings-powershell)
* 如果客户端程序连接到 SQL 数据库 V12，而客户端运行在 Azure 虚拟机 (VM) 上，则必须打开 VM 上的某些端口范围。详细信息：[Ports beyond 1433 for ADO.NET 4.5 and SQL Database V12（用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口）](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12)
* 与 Azure SQL 数据库 V12 建立的客户端连接有时会绕过代理直接与数据库交互。除 1433 以外的端口变得非常重要。详细信息：[Ports beyond 1433 for ADO.NET 4.5 and SQL Database V12（用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口）](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12)

## 数据分片和弹性缩放
弹性缩放简化了扩展（和缩减）过程。

* [包含 Azure SQL 数据库的多租户 SaaS 应用程序的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)
* [依赖于数据的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)
* [Azure SQL 数据库 Elastic Scale 预览版入门](/documentation/articles/sql-database-elastic-scale-get-started/)

## 后续步骤

浏览所有 [SQL 数据库功能](/home/features/sql-database/)。

<!---HONumber=Mooncake_1010_2016-->