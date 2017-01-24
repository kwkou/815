<properties
    pageTitle="Azure SQL 数据库概述 | Azure"
    description="本页概述 Azure SQL 数据库。"
    services="sql-database"
    documentationcenter="na"
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.service="sql-database"
    ms.custom="single databases"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="data-management"
    ms.date="11/28/2016"
    wacn.date="01/20/2017"
    ms.author="carlrab" />  


# Azure SQL 数据库概述
本主题概述 Azure SQL 数据库。有关 Azure SQL 逻辑服务器的信息，请参阅[逻辑服务器](/documentation/articles/sql-database-server-overview/)。

## 什么是 Azure SQL 数据库？
Azure SQL 数据库中的每个数据库都与逻辑服务器相关联。数据库可以是：

- 单一数据库，具有其[自己的资源集](/documentation/articles/sql-database-what-is-a-dtu/#what-are-database-transaction-units-dtus) (DTU)
- 属于[弹性池](/documentation/articles/sql-database-elastic-pool/)的一部分，[共享一组资源](/documentation/articles/sql-database-what-is-a-dtu/#what-are-elastic-database-transaction-units-edtus) (eDTU)
- 属于[横向扩展的分片数据库集](/documentation/articles/sql-database-elastic-scale-introduction/#horizontal-and-vertical-scaling)的一部分，可以是单一数据库或入池数据库
- 属于参与[多租户 SaaS 设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)的一组数据库，其数据库可以是单一数据库和/或入池数据库

## 如何连接 Azure SQL 数据库并向其进行身份验证？

- **身份验证和授权**：Azure SQL 数据库支持 SQL 身份验证和 Azure Active Directory 身份验证（但存在某些限制）- 请参阅[使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)了解身份验证。可以通过服务器的 master 数据库或直接连接到用户数据库，连接到 Azure SQL 数据库并向其进行身份验证。有关详细信息，请参阅[在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins/)。不支持 Windows 身份验证。
- **TDS**：Azure SQL 数据库支持表格格式数据流 (TDS) 协议客户端版本 7.3 或更高版本。
- **TCP/IP**：只允许 TCP/IP 连接。
- **SQL 数据库防火墙**：为了保护用户数据，在用户指定哪些计算机具有访问权限之前，SQL 数据库防火墙将禁止所有对数据库服务器或其数据库的访问。请参阅[防火墙](/documentation/articles/sql-database-firewall-configure/)

## 支持哪些排序规则？
Azure SQL 数据库使用的默认数据库排序规则是 **SQL\_LATIN1\_GENERAL\_CP1\_CI\_AS**，其中 **LATIN1\_GENERAL** 是英语(美国)，**CP1** 是代码页 1252，**CI** 是不区分大小写，**AS** 是区分重音。无法改变 V12 数据库的排序规则。有关如何设置排序规则的详细信息，请参阅 [COLLATE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms184391.aspx)。

## 数据库对象有哪些命名要求？

所有新对象的名称都必须符合 SQL Server 标识符规则。有关详细信息，请参阅[标识符](https://msdn.microsoft.com/zh-cn/library/ms175874.aspx)。

## Azure SQL 数据库支持哪些功能？

有关支持的功能的信息，请参阅[功能](/documentation/articles/sql-database-features/)。另请参阅 [Azure SQL 数据库 Transact-SQL 差异](/documentation/articles/sql-database-transact-sql-information/)，从背景方面了解不支持某些类型的功能的原因。

## 如何管理 Azure SQL 数据库？

可以使用几种方法管理 Azure SQL 数据库逻辑服务器：
- [Azure 门户预览](/documentation/articles/sql-database-manage-portal/)
- [PowerShell](/documentation/articles/sql-database-manage-powershell/)
- [Transact-SQL](/documentation/articles/sql-database-manage-azure-ssms/)
- [REST](https://docs.microsoft.com/rest/api/sql/)

## 后续步骤

- 有关 Azure SQL 数据库服务的信息，请参阅[什么是 SQL 数据库？](/documentation/articles/sql-database-technical-overview/)
- 有关支持的功能的信息，请参阅[功能](/documentation/articles/sql-database-features/)
- 有关 Azure SQL 逻辑服务器的概述，请参阅 [SQL 数据库逻辑服务器概述](/documentation/articles/sql-database-server-overview/)
- 有关 Transact-SQL 支持和差异的信息，请参阅 [Azure SQL 数据库 Transact-SQL 差异](/documentation/articles/sql-database-transact-sql-information/)。
- 按**服务层**了解具体的资源配额和限制。有关服务层的概述，请参阅 [SQL 数据库服务层](/documentation/articles/sql-database-service-tiers/)。
- 如需与安全相关的指导原则，请参阅 [Azure SQL 数据库安全指导原则和限制](/documentation/articles/sql-database-security-guidelines/)。
- 有关驱动程序可用性和 SQL 数据库支持的信息，请参阅 [用于 SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries/)。

<!---HONumber=Mooncake_0116_2017-->