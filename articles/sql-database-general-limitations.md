<properties 
   pageTitle="Azure SQL 数据库的一般性限制和指导原则"
   description="本页介绍 Azure SQL 数据库的某些一般性限制，以及互操作性和支持方面的问题。"
   services="sql-database"
   documentationCenter="na"
   authors="rothja"
   manager="jeffreyg"
   editor="monicar" />
<tags 
   ms.service="sql-database"
   ms.date="11/16/2015"
   wacn.date="12/22/2015" />

# Azure SQL 数据库的一般性限制和指导原则

本主题提供 Azure SQL 数据库的一般性限制与指导原则。若要全面了解配额、资源管理和支持，请参阅本主题末尾的[其他资源](/documentation/articles/#additional-guidelines)。

## 连接

 - 不支持 Windows 身份验证。请参阅[管理 Azure SQL 数据库的数据库和登录名](sql-database-manage-logins)。 

 - Microsoft Azure SQL 数据库支持表格格式数据流 (TDS) 协议客户端 7.3 版或更高版本。

 - 只允许 TCP/IP 连接。

 - 不支持 SQL Server 2008 SQL Server 浏览器，因为 Microsoft Azure SQL 数据库没有动态端口，只有端口 1433。

## SQL Server 代理/作业

Microsoft Azure SQL 数据库不支持 SQL Server 代理或作业。不过，你可以在本地 SQL Server 上运行 SQL Server 代理并连接到 Microsoft Azure SQL 数据库。

## 事务

Microsoft Azure SQL 数据库不支持分布式事务，分布式事务是影响若干资源的事务。有关详细信息，请参阅[分布式事务 (ADO.NET)](https://msdn.microsoft.com/zh-cn/library/ms254973.aspx)。SQL 数据库可能不会跨故障转移保留当前数据库 (DBTS) 的未提交的 timestamp 值。

> [AZURE.NOTE]在某些情况下，事务会自动升级为分布式事务。有关详细信息，请参阅[与 SQL Server 的系统事务集成](https://msdn.microsoft.com/zh-cn/library/ms172070.aspx)。

## SQL Server 排序规则支持

Microsoft Azure SQL 数据库使用的默认数据库排序规则是 **SQL\_LATIN1\_GENERAL\_CP1\_CI\_AS**。其中，**LATIN1\_GENERAL** 是英语（美国），**CP1** 是代码页 1252，**CI** 是不区分大小写，**AS** 是区分重音。可以使用 Transact-SQL 改变 V12 数据库的排序规则。有关如何设置定序的详细信息，请参阅 [COLLATE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms184391.aspx)。

## 命名要求

出于安全原因，不允许使用某些用户名。不能使用下列名称：

 - **admin** 
 - **administrator** 
 - **guest** 
 - **root** 
 - **sa** 

所有新对象的名称都必须符合 SQL Server 标识符规则。有关详细信息，请参阅[标识符](https://msdn.microsoft.com/zh-cn/library/ms175874.aspx)。

此外，登录名和用户名不得包含 \\ 字符（不支持 Windows 身份验证）。

## 其他指导原则

- 除了本文所述的一般性限制外，SQL 数据库还会根据**服务层**实施特定的配额与限制。有关服务层的概述，请参阅 [SQL 数据库服务层](/documentation/articles/sql-database-service-tiers)。

- 有关其他 Azure SQL 数据库限制，请参阅 [Azure SQL 数据库资源限制](/documentation/articles/sql-database-resource-limits)。

- 有关与安全相关的指导原则，请参阅 [Azure SQL 数据库安全指导原则和限制](/documentation/articles/sql-database-security-guidelines)。

- 与兼容性相关的另一个方面是，Azure SQL 数据库有本地版本的 SQL Server，例如 SQL Server 2014。Azure SQL 数据库的最新版本 V12 已在此方面做出诸多改善。有关详细信息，请参阅 [SQL 数据库 V12 的新增功能](/documentation/articles/sql-database-v12-whats-new)。

- 有关驱动程序可用性和 SQL 数据库支持的信息，请参阅 [用于 SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries)。

<!---HONumber=Mooncake_1207_2015-->