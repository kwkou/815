<properties
	pageTitle="有关 SQL 数据库服务的所有主题 | Azure"
	description="位于 /documentation/articles/ 的有关 Azure SQL 数据库服务的所有主题的表格，包括标题和描述。"
	services="sql-database"
	documentationCenter=""
	authors="MightyPen"
	manager="jhubbard"
	editor="MightyPen"/>  


<tags
	ms.service="sql-database"
	ms.workload="sql-database"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/05/2016"
	wacn.date="12/26/2016"
	ms.author="genemi"/>  



# 有关 Azure SQL 数据库服务的所有主题

本主题列出的每个主题都可以直接应用于 Azure **SQL 数据库**服务。可以使用 **Ctrl+F** 来搜索此网页的关键字，以便查找当前感兴趣的主题。




## 新建

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 1 | [Azure SQL 数据库中的 JSON 功能入门](/documentation/articles/sql-database-json-features/) | 使用 Azure SQL 数据库可以分析、查询数据，以 JavaScript 对象表示法 (JSON) 设置数据格式。 |
| 2 | [SQL 数据库和 SQL 数据仓库针对 Azure AD MFA 的 SSMS 支持](/documentation/articles/sql-database-ssms-mfa-authentication/) | 将多重身份验证与 SSMS 共同用于 SQL 数据库和 SQL 数据仓库。 |


## 更新的文章

本部分列出最近做了重大更新的文章。每篇更新的文章都会显示大致的代码片段，说明添加的 Markdown 文本。这些文章是在 **2016-07-26** 到 **2016-08-21** 期间更新的。

| &nbsp; | 文章 | 更新的文本、代码片段 |
| --: | :-- | :-- |
| 3 | [使用 PowerShell 将 Azure SQL 数据库存档为 BACPAC 文件](/documentation/articles/sql-database-export-powershell/) | $subscriptionId = "YOUR AZURE SUBSCRIPTION ID" Login-AzureRmAccount -EnvironmentName AzureChinaCloud Set-AzureRmContext -SubscriptionId $subscriptionId Database to export $DatabaseName = "DATABASE-NAME" $ResourceGroupName = "RESOURCE-GROUP-NAME" $ServerName = "SERVER-NAME" $serverAdmin = "ADMIN-NAME" $serverPassword = "ADMIN-PASSWORD" $securePassword = ConvertTo-SecureString ΓÇôString $serverPassword ΓÇôAsPlainText -Force $creds = New-Object ΓÇôTypeName System.Management.Automation.PSCredential ΓÇôArgumentList $serverAdmin, $securePassword Generate a unique filename for the BACPAC $bacpacFilename = $DatabaseName + (Get-Date).ToString("yyyyMMddHHmm") + ".bacpac" Storage account info for the BACPAC $BaseStorageUri = "https://STORAGE-NAME.blob.core.chinacloudapi.cn/BLOB-CONTAINER-NAME/" $BacpacUri = $BaseStorageUri + $bacpacFilename $StorageKeytype = "StorageAccessKey" $StorageKey = "YOUR STORAGE KEY" $exportRequest = New-AzureRmSqlDatabaseExpo |
| 4 | [SQL 数据库选项和性能：了解每个服务层中可用的功能](/documentation/articles/sql-database-service-tiers/) | **选择服务层**若要确定服务层，请先确定数据库是独立数据库还是弹性池的一部分。**选择独立数据库的服务层**若要确定独立数据库的服务层，请先确定所需的数据库功能以选择 SQL 数据库版本：/ 数据库大小（“基本”层最多为 5 GB、“标准”层最多为 250 GB、“高级”层最多为 500 GB 到 1 TB - 具体取决于性能级别）/ 数据库备份保留期限（“基本”层为 7 天、“标准”层为 35 天、“高级”层为 35 天）。确定 SQL 数据库版本之后，便可以确定数据库的性能级别（DTU 数目）。可以根据实际经验猜测，然后动态扩展或缩减 (/documentation/articles/sql-database-scale-up/)。还可以使用 DTU 计算器 (http://dtucalculator.azurewebsites.net/) 估算所需的 DTU 数目。** C |





## 入门

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 5 | [使用具有隔离和效率的 Azure SQL 数据库构建多租户应用](/documentation/articles/sql-database-build-multi-tenant-apps/) | 了解 SQL 数据库如何构建多租户应用 |
| 5 | [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/) | 了解如何通过使用 SQL Server Management Studio (SSMS) 连接到 Azure 上的 SQL 数据库。然后，使用 Transact-SQL (T-SQL) 运行示例查询。 |
| 6 | [使用 .NET (C#) 连接到 SQL 数据库](/documentation/articles/sql-database-develop-dotnet-simple/) | 使用本快速入门教程中的示例代码可以生成一个包含 C# 代码并由云中强大的 Azure SQL 数据库关系数据库支持的新式应用程序。 |
| 7 | [使用 Azure 门户创建新的弹性数据库池](/documentation/articles/sql-database-elastic-pool-create-portal/) | 如何将可缩放的弹性数据库池添加到 SQL 数据库配置，以简化多个数据库的管理和资源共享。 |
| 8 | [浏览 Azure SQL 数据库教程](/documentation/articles/sql-database-explore-tutorials/) | 了解 SQL 数据库的特性和功能 |
| 9 | [SQL 数据库教程：使用 Azure 门户预览在几分钟内创建一个 SQL 数据库](/documentation/articles/sql-database-get-started/) | 了解如何设置 SQL 数据库逻辑服务器、服务器防火墙规则、SQL 数据库、示例性数据、与客户端工具连接、配置用户和数据库防火墙规则。 |
| 10 | [Azure SQL 数据库提供安全和保护](/documentation/articles/sql-database-helps-secures-and-protects/) | 了解 SQL 数据库如何帮助提供安全和保护 |
| 11 | [Azure SQL 数据库会自行学习和进行适应性调整](/documentation/articles/sql-database-learn-and-adapt/) | 了解 SQL 数据库如何进行学习和适应性调整 |
| 12 | [选择云 SQL Server 选项：Azure SQL (PaaS) 数据库或 Azure VM 上的 SQL Server (IaaS)](/documentation/articles/sql-database-paas-vs-sql-server-iaas/) | 了解哪个云 SQL Server 选项适合应用程序：Azure SQL (PaaS) 数据库或 Azure 虚拟机上的云中 SQL Server。 |
| 13 | [Azure SQL 数据库动态缩放规模](/documentation/articles/sql-database-scale-on-the-fly/) | 了解 Azure SQL 数据库如何动态缩放规模 |
| 14 | [浏览 Azure SQL 数据库解决方案快速入门](/documentation/articles/sql-database-solution-quick-starts/) | 了解 Azure SQL 数据库解决方案 |
| 15 | [Azure SQL 数据库在你的环境中正常工作](/documentation/articles/sql-database-works-in-your-environment/) | 了解 SQL 数据库如何提供帮助、安全和保护 |



## 构建应用

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 16 | [排查、诊断和防止 SQL 数据库中的 SQL 连接错误和暂时性错误](/documentation/articles/sql-database-connectivity-issues/) | 了解如何排查、诊断和防止 Azure SQL 数据库中的 SQL 连接错误或暂时性错误。 |
| 17 | [使用 Visual Studio 连接到 SQL 数据库](/documentation/articles/sql-database-connect-query/) | 用 C# 编写用于查询和连接到 SQL 数据库的程序。有关 IP 地址、连接字符串、安全登录和免费 Visual Studio 的信息。 |
| 18 | [用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/) | 从 ADO.NET 到 Azure SQL 数据库 V12 的客户端连接有时会绕过代理直接与数据库交互。除 1433 以外的端口变得非常重要。 |
| 19 | [SQL 数据库客户端应用程序的 SQL 错误代码：数据库连接错误和其他问题](/documentation/articles/sql-database-develop-error-messages/) | 了解有关 SQL 数据库客户端应用程序的 SQL 错误代码，例如常见的数据库连接错误、数据库复制问题和常规错误。 |
| 20 | [在 Windows上使用 PHP 连接到 SQL 数据库](/documentation/articles/sql-database-develop-php-simple/) | 演示一个示例 PHP 程序，该程序可以从 Windows 客户端连接到 Azure SQL 数据库，并与客户端所需的软件组件建立链接。 |
| 21 | [试用 SQL 数据库：使用 C# 通过适用于 .NET 的 SQL 数据库库创建 SQL 数据库](/documentation/articles/sql-database-get-started-csharp/) | 尝试使用 SQL 数据库开发 SQL 和 C# 应用，并使用适用于 .NET 的 SQL 数据库库以 C# 创建 Azure SQL 数据库。 |
| 22 | [排查 Azure SQL 数据库的连接问题](/documentation/articles/sql-database-troubleshoot-common-connection-issues/) | 识别和解决 Azure SQL 数据库常见连接错误的步骤。 |
| 23 | [连接到 SQL 数据库时发生“服务器上的数据库当前不可用”错误](/documentation/articles/sql-database-troubleshoot-connection/) | 对应用程序连接到 SQL 数据库时发生的“服务器上的数据库当前不可用”错误进行排查。 |



## 开发

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 24 | [获取客户端 ID 和密钥以便从代码连接到 SQL 数据库](/documentation/articles/sql-database-client-id-keys/) | 获取客户端 ID 和密钥以便从代码访问 SQL 数据库。 |
| 25 | [在 Windows 上配合使用 Java 和 JDBC 连接到 SQL 数据库](/documentation/articles/sql-database-develop-java-simple/) | 演示了一个可以用来连接到 Azure SQL 数据库的 Java 代码示例。该示例使用 JDBC，并在 Windows 客户端计算机上运行。 |
| 26 | [使用 Node.js 连接到 SQL 数据库](/documentation/articles/sql-database-develop-nodejs-simple/) | 演示了一个可以用来连接到 Azure SQL 数据库的 Node.js 代码示例。 |
| 27 | [SQL 数据库开发概述](/documentation/articles/sql-database-develop-overview/) | 了解可用于连接到 SQL 数据库的连接库和最佳实践。 |
| 28 | [使用 Python 连接到 SQL 数据库](/documentation/articles/sql-database-develop-python-simple/) | 演示可用于连接到 Azure SQL 数据库的 Python 代码示例。 |
| 29 | [使用 Ruby 连接到 SQL 数据库](/documentation/articles/sql-database-develop-ruby-simple/) | 提供可运行的用于连接到 Azure SQL 数据库的 Ruby 代码示例。 |
| 30 | [Azure SQL 数据库中的临时表入门](/documentation/articles/sql-database-temporal-tables/) | 了解如何开始使用 Azure SQL 数据库中的临时表。 |



## 性能和缩放

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 31 | [SQL 数据库顾问](/documentation/articles/sql-database-advisor/) | Azure SQL 数据库顾问为现有 SQL 数据库提供建议，这样可以提高当前的查询性能。 |
| 32 | [SQL 数据库顾问](/documentation/articles/sql-database-advisor-portal/) | 可以在 Azure 门户预览中使用 Azure SQL 数据库顾问查看和实施为现有 SQL 数据库提供的建议，这些建议可以提高当前的查询性能。 |
| 33 | [Azure SQL 数据库基准检验概述](/documentation/articles/sql-database-benchmark-overview/) | 本主题介绍在 Azure SQL 数据库的性能测量中使用的 Azure SQL 数据库基准检验。 |
| 34 | [何时使用弹性数据库池？](/documentation/articles/sql-database-elastic-pool-guidance/) | 弹性数据库池是由一组弹性数据库共享的可用资源集合。本文提供相关的指导来帮助你评估是否适合对一组数据库使用弹性数据库池。 |
| 35 | [弹性数据库工具入门](/documentation/articles/sql-database-elastic-scale-get-started/) | 大致介绍 Azure SQL 数据库的弹性数据库工具功能，包括易于使用的示例应用。 |
| 36 | [SQL 数据库常见问题](/documentation/articles/sql-database-faq/) | 客户就云数据库、Azure SQL 数据库、Microsoft 的关系数据库管理系统 (RDBMS) 和云中“数据库即服务”经常提出的问题的解答。 |
| 38 | SQL 数据库中的 In-Memory（预览版）入门 | SQL In-Memory 技术大幅提升了事务和分析工作负荷的性能。了解如何利用这些技术。 |
| 39 | 使用 In-Memory OLTP（预览版）改善 SQL 数据库中的应用程序性能 | 利用 In-Memory OLTP 改善现有 SQL 数据库中的事务性能。 |
| 40 | [监视 In-Memory OLTP 存储](/documentation/articles/sql-database-in-memory-oltp-monitoring/) | 估算和监视 XTP 内存中存储用量与容量；解决容量错误 41823 |
| 41 | [在 Azure SQL 数据库中操作 Query Store](/documentation/articles/sql-database-operate-query-store/) | 了解如何在 Azure SQL 数据库中操作 Query Store |
| 42 | [SQL 数据库性能见解](/documentation/articles/sql-database-performance/) | Azure SQL 数据库提供的性能工具有助于发现可以提高当前查询性能的各个方面。 |
| 43 | [Azure SQL 数据库的单一数据库性能指导](/documentation/articles/sql-database-performance-guidance/) | 本主题提供了一些指导，帮助你确定此预览版提供的哪个服务层适合你的应用程序，并提供了一些应用程序优化建议，让你充分利用 Azure SQL 数据库。 |
| 44 | [Azure SQL 数据库 Query Performance Insight](/documentation/articles/sql-database-query-performance/) | 查询性能监视可以识别 Azure SQL 数据库中 CPU 消耗最大的查询。 |
| 45 | [更改 SQL 数据库的服务层和性能级别（定价层）](/documentation/articles/sql-database-scale-up/) | “更改 Azure SQL 数据库的服务层和性能级别”介绍如何扩展或缩减 SQL 数据库。更改 Azure SQL 数据库的定价层。 |
| 46 | [在 Azure SQL 数据库中监视数据库性能](/documentation/articles/sql-database-single-database-monitor/) | 了解使用 Azure 工具和动态管理视图监视数据库时可用的选项。 |
| 47 | [SQL 数据库性能优化提示](/documentation/articles/sql-database-troubleshoot-performance/) | 有关通过评估和改进来优化 Azure SQL 数据库性能的提示。 |
| 48 | [如何使用批处理来改善 SQL 数据库应用程序的性能](/documentation/articles/sql-database-use-batching-to-improve-performance/) | 本主题提供有关数据库批处理操作大幅改善 Azure SQL 数据库应用程序速度和缩放性的证据。尽管这些批处理方法适用于任何 SQL Server 数据库，但本文的重点是 Azure。 |



## 扩展工具

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 49 | [包含 Azure SQL 数据库的多租户 SaaS 应用程序的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/) | 本文介绍云环境中运行的多租户数据库应用程序需要考虑的要求和通用数据体系结构模式，以及与这些模式相关的各种利弊。此外，还说明了 Azure SQL 数据库服务及其弹性数据库池和弹性工具如何在不造成损害的情况下满足这些要求。 |
| 50 | [迁移要扩大的现有数据库](/documentation/articles/sql-database-elastic-convert-to-use-elastic-tools/) | 通过创建分片映射管理器来转换分片数据库，以使用弹性数据库工具 |
| 51 | [构建可缩放的云数据库](/documentation/articles/sql-database-elastic-database-client-library/) | 使用弹性数据库客户端库构建可缩放的 .NET 数据库应用 |
| 52 | [分片映射管理器的性能计数器](/documentation/articles/sql-database-elastic-database-perf-counters/) | ShardMapManager 类和数据相关的路由性能计数器 |
| 53 | [使用弹性数据库工具添加分片](/documentation/articles/sql-database-elastic-scale-add-a-shard/) | 如何使用弹性缩放 API 将新分片添加到分片集。 |
| 54 | [部署拆分/合并服务](/documentation/articles/sql-database-elastic-scale-configure-deploy-split-and-merge/) | 使用弹性数据库工具进行拆分与合并 |
| 55 | [依赖于数据的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/) | 如何将 .NET 应用中的 ShardMapManager 类用于数据相关的路由（Azure SQL 数据库弹性数据库的一项功能） |
| 56 | [弹性数据库工具常见问题](/documentation/articles/sql-database-elastic-scale-faq/) | 有关 Azure SQL 数据库弹性缩放的常见问题。 |
| 57 | [弹性数据库工具词汇表](/documentation/articles/sql-database-elastic-scale-glossary/) | 弹性数据库工具所用术语的解释 |
| 58 | [使用 Azure SQL 数据库进行扩展](/documentation/articles/sql-database-elastic-scale-introduction/) | 软件即服务 (SaaS) 开发人员可以使用这些工具轻松地在云中创建可缩放的弹性数据库 |
| 59 | [用于访问弹性数据库客户端库的凭据](/documentation/articles/sql-database-elastic-scale-manage-credentials/) | 如何为弹性数据库应用设置正确的凭据级别（从管理员到只读权限） |
| 60 | [多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying/) | 使用弹性数据库客户端库运行跨分片查询。 |
| 61 | [在扩展云数据库之间移动数据](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/) | 介绍如何使用弹性数据库 API 通过自托管服务来操作分片和移动数据。 |
| 62 | [使用分片映射管理器扩展数据库](/documentation/articles/sql-database-elastic-scale-shard-map-management/) | 如何使用弹性数据库客户端库 ShardMapManager |
| 63 | [拆分/合并安全配置](/documentation/articles/sql-database-elastic-scale-split-merge-security-configuration/) | 设置用于加密的 x409 证书 |
| 64 | [升级应用以使用最新的弹性数据库客户端库](/documentation/articles/sql-database-elastic-scale-upgrade-client-library/) | 使用 Nuget 升级应用和库 |
| 65 | [将弹性数据库客户端库与实体框架配合使用](/documentation/articles/sql-database-elastic-scale-use-entity-framework-applications-visual-studio/) | 将弹性数据库客户端库和实体框架用于数据库编码 |
| 66 | [将弹性数据库客户端库与 Dapper 配合使用](/documentation/articles/sql-database-elastic-scale-working-with-dapper/) | 将弹性数据库客户端库与 Dapper 配合使用。 |
| 67 | [具有弹性数据库工具和行级安全性的多租户应用程序](/documentation/articles/sql-database-elastic-tools-multi-tenant-row-level-security/) | 了解如何将弹性数据库工具和行级安全性一起使用，在 Azure SQL 数据库上构建具有高度可缩放的数据层、支持多租户分片的应用程序。 |







## 弹性池

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 74 | [什么是 Azure 弹性数据库池？](/documentation/articles/sql-database-elastic-pool/) | 使用池管理成百上千个数据库可通过池分发一组性能单位的价格。可随心所欲地移入或移出数据。 |
| 75 | [使用 C# 创建新的弹性数据库池](/documentation/articles/sql-database-elastic-pool-create-csharp/) | 使用 C# 数据库开发技术在 Azure SQL 数据库中创建可缩放的弹性数据库池，以便可以在多个数据库之间共享资源。 |
| 76 | [使用 PowerShell 创建新的弹性数据库池](/documentation/articles/sql-database-elastic-pool-create-powershell/) | 了解如何通过创建可缩放的弹性数据库池，使用 PowerShell 向外缩放 Azure SQL 数据库资源以管理多个数据库。 |
| 77 | [C# 数据库开发：为 SQL 数据库创建和配置弹性数据库池](/documentation/articles/sql-database-elastic-pool-csharp/) | 使用 C# 数据库开发技术创建 Azure SQL 数据库弹性数据库池，以便可以在多个数据库之间共享资源。 |
| 78 | [用于识别适用于弹性数据库池的数据库的 PowerShell 脚本](/documentation/articles/sql-database-elastic-pool-database-assessment-powershell/) | 弹性数据库池是由一组弹性数据库共享的可用资源集合。本文档提供 Powershell 脚本来帮助你评估是否适合对一组数据库使用弹性数据库池。 |
| 79 | [使用 C# 监视和管理弹性数据库池](/documentation/articles/sql-database-elastic-pool-manage-csharp/) | 使用 C# 数据库开发技术来管理 Azure SQL 数据库弹性数据库池。 |
| 80 | [使用 Azure 门户预览监视和管理弹性数据库池](/documentation/articles/sql-database-elastic-pool-manage-portal/) | 了解如何使用 Azure 门户和 SQL 数据库的内置智能来管理、监视可缩放的弹性数据库池并正确调整其大小，以优化数据库性能和管理成本。 |
| 81 | [使用 PowerShell 监视和管理弹性数据库池](/documentation/articles/sql-database-elastic-pool-manage-powershell/) | 了解如何使用 PowerShell 管理弹性数据库池。 |
| 82 | [使用 Transact-SQL 监视和管理弹性数据库池](/documentation/articles/sql-database-elastic-pool-manage-tsql/) | 使用 T-SQL 在弹性池中创建 Azure SQL 数据库。或使用 T-SQL 将数据库移入和移出池。 |
| 83 | [弹性数据库池计费和定价信息](/documentation/articles/sql-database-elastic-pool-price/) | 特定于弹性数据库池的定价信息。 |



## 弹性查询

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 84 | [跨扩展云数据库进行报告（预览）](/documentation/articles/sql-database-elastic-query-getting-started/) | 如何使用跨数据库的数据库查询 |
| 85 | [跨数据库查询（纵向分区）入门（预览）](/documentation/articles/sql-database-elastic-query-getting-started-vertical/) | 如何在垂直分区数据库中使用弹性数据库查询 |
| 86 | [跨扩展云数据库进行报告（预览）](/documentation/articles/sql-database-elastic-query-horizontal-partitioning/) | 如何对横向分区设置弹性查询 |
| 87 | [Azure SQL 数据库弹性数据库查询概述（预览）](/documentation/articles/sql-database-elastic-query-overview/) | 弹性查询功能概述 |
| 88 | [在具有不同架构的云数据库中进行查询（预览）](/documentation/articles/sql-database-elastic-query-vertical-partitioning/) | 如何在垂直分区上设置跨数据库查询 |



## 弹性事务

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 89 | [跨云数据库的分布式事务](/documentation/articles/sql-database-elastic-transactions-overview/) | Azure SQL 数据库的弹性数据库事务概述 |



## 备份和恢复

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 90 | [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/) | 了解 SQL 数据库的内置备份，借助该备份可将 Azure SQL 数据库回滚到以前的时间点，或者将数据库复制到地理区域中的新数据库（最多 35 天）。 |
| 91 | [使用 Azure SQL 数据库确保业务连续性](/documentation/articles/sql-database-business-continuity/) | 了解 Azure SQL 数据库如何支持云业务连续性和数据库恢复，以及如何帮助保持运行任务关键型云应用程序。 |
| 94 | [如何从 Azure SQL 数据库备份中还原单个表](/documentation/articles/sql-database-cloud-migrate-restore-single-table-azure-backup/) | 了解如何从 Azure SQL 数据库备份中还原单个表。 |
| 95 | [使用 SQL 数据库中的活动异地复制功能为云灾难恢复设计应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/) | 了解如何使用 Azure SQL 数据库中应用程序数据备份的异地复制功能为业务连续性规划设计云灾难恢复解决方案。 |
| 96 | [还原 Azure SQL 数据库或故障转移到辅助数据库](/documentation/articles/sql-database-disaster-recovery/) | 了解在发生区域性的数据中心中断或故障后，如何使用 Azure SQL 数据库活动异地复制和异地还原功能来恢复数据库。 |
| 97 | [执行灾难恢复演练](/documentation/articles/sql-database-disaster-recovery-drills/) | 了解有关使用 Azure SQL 数据库执行灾难恢复演练，帮助任务关键型业务应用程序弹性应对故障和中断的指导和最佳实践。 |
| 98 | [使用 SQL 数据库弹性池的应用程序的灾难恢复策略](/documentation/articles/sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool/) | 了解如何通过选择合适的故障转移模式来设计可实现灾难恢复的云解决方案。 |
| 99 | [使用 RecoveryManager 类解决分片映射问题](/documentation/articles/sql-database-elastic-database-recovery-manager/) | 使用 RecoveryManager 类解决分片映射问题 |
| 100 | [导入 BACPAC 文件以创建新的 Azure SQL 数据库](/documentation/articles/sql-database-import/) | 通过导入现有的 BACPAC 文件创建新的 Azure SQL 数据库。 |
| 101 | [使用 Azure 门户预览将 Azure SQL 数据库还原到之前的时间点](/documentation/articles/sql-database-point-in-time-restore-portal/) | 将 Azure SQL 数据库还原到之前的时间点。 |
| 102 | [使用 PowerShell 将 Azure SQL 数据库还原到之前的时间点](/documentation/articles/sql-database-point-in-time-restore-powershell/) | 将 Azure SQL 数据库还原到之前的时间点 |
| 104 | [使用自动数据库备份恢复 Azure SQL 数据库](/documentation/articles/sql-database-recovery-using-backups/) | 了解有关时间点还原的信息，它可使你将 Azure SQL 数据库回滚到之前的时间点（最多 35 天）。 |
| 105 | [使用 Azure 门户预览还原已删除的 Azure SQL 数据库](/documentation/articles/sql-database-restore-deleted-database-portal/) | 还原已删除的 Azure SQL 数据库（Azure 门户）。 |
| 106 | [使用 PowerShell 还原已删除的 Azure SQL 数据库](/documentation/articles/sql-database-restore-deleted-database-powershell/) | 还原已删除的 Azure SQL 数据库 (PowerShell)。 |
| 107 | [将数据库还原到以前的时间点、还原已删除的数据库，或者在数据中心服务中断的情况下进行恢复](/documentation/articles/sql-database-troubleshoot-backup-and-restore/) | 了解如何在发生错误和服务中断时，使用 Azure SQL 数据库中的备份和副本恢复云数据库。 |



## 迁移

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 110 | [使用 SqlPackage 将 SQL Server 数据库导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-sqlpackage/) | Azure SQL 数据库, 数据库迁移, 导出数据库, 导出 BACPAC 文件, sqlpackage |
| 111 | [使用 SQL Server Management Studio 将 SQL Server 数据库导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/) | Azure SQL 数据库, 数据库迁移, 导出数据库, 导出 BACPAC 文件, 导出数据层应用程序向导 |
| 112 | [使用 SqlPackage 从 BACPAC 文件导入到 SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage/) | Azure SQL 数据库, 数据库迁移, 导入数据库, 导入 BACPAC 文件, sqlpackage |
| 113 | [使用 SSMS 从 BACPAC 导入到 SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/) | Azure SQL 数据库, 数据库部署, 数据库迁移, 导入数据库, 导出数据库, 迁移向导 |
| 114 | [使用“将数据库部署到 Azure 数据库向导”将 SQL Server 数据库迁移到 SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-using-ssms-migration-wizard/) | Azure SQL 数据库, 数据库迁移, Azure 数据库向导 |
| 115 | [使用事务复制将 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-using-transactional-replication/) | Azure SQL 数据库，数据库迁移，导入数据库，事务复制 |
| 116 | [使用 SqlPackage.exe 确定 SQL 数据库兼容性](/documentation/articles/sql-database-cloud-migrate-determine-compatibility-sqlpackage/) | Azure SQL 数据库, 数据库迁移, SQL 数据库兼容性, SqlPackage |
| 117 | [在迁移到 Azure SQL 数据库之前，使用 SQL Server Management Studio 确定 SQL 数据库的兼容性](/documentation/articles/sql-database-cloud-migrate-determine-compatibility-ssms/) | Azure SQL 数据库, 数据库迁移, SQL 数据库兼容性, 导出数据层应用程序向导 |
| 118 | [在迁移到 Azure SQL 数据库之前使用 SQL Azure 迁移向导解决 SQL Server 数据库兼容性问题](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues/) | Azure SQL 数据库, 数据库迁移, 兼容性, SQL Azure 迁移向导 |
| 119 | [使用 SQL Server Data Tools for Visual Studio 将 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/) | Azure SQL 数据库, 数据库迁移, 兼容性, SQL Azure 迁移向导, SSDT |
| 120 | [在迁移到 SQL 数据库之前使用 SQL Server Management Studio 解决 SQL Server 数据库兼容性问题](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssms/) | Azure SQL 数据库, 数据库迁移, 兼容性, SQL Azure 迁移向导 |
| 121 | [使用 PowerShell 将 Azure SQL 数据库存档到 BACPAC 文件](/documentation/articles/sql-database-export-powershell/) | 使用 PowerShell 将 Azure SQL 数据库存档到 BACPAC 文件 |
| 122 | [使用 PowerShell 导入 BACPAC 文件以创建新的 Azure SQL 数据库](/documentation/articles/sql-database-import-powershell/) | 使用 PowerShell 导入 BACPAC 文件以创建新的 Azure SQL 数据库 |
| 123 | [将数据从 CSV 载入 Azure SQL 数据仓库（平面文件）](/documentation/articles/sql-database-load-from-csv-with-bcp/) | 对于较小的数据，请使用 bcp 将数据导入到 Azure SQL 数据库。 |
| 124 | [在服务器之间或订阅之间移动数据库，以及将数据库移入和移出 Azure](/documentation/articles/sql-database-troubleshoot-moving-data/) | 在 Azure SQL 数据库中复制、移动和迁移数据与数据库的快速步骤。 |



## 移入和移出数据

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 125 | [将 SQL Server 数据库迁移到云中的 SQL 数据库](/documentation/articles/sql-database-cloud-migrate/) | 了解如何将本地 SQL Server 数据库迁移到云中的 Azure SQL 数据库。执行数据库迁移之前，使用数据库迁移工具测试兼容性。 |
| 126 | [复制 Azure SQL 数据库](/documentation/articles/sql-database-copy/) | 创建 Azure SQL 数据库的副本 |
| 127 | [使用 Azure 门户预览将 Azure SQL 数据库存档到 BACPAC 文件](/documentation/articles/sql-database-export/) | 使用 Azure 门户预览将 Azure SQL 数据库存档到 BACPAC 文件 |



## 安全

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 128 | [使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库](/documentation/articles/sql-database-aad-authentication/) | 了解如何通过使用 Azure Active Directory 身份验证连接到 SQL 数据库 |
| 129 | [始终加密 - 使用数据库加密保护 SQL 数据库中的敏感数据并将加密密钥存储在 Windows 证书存储中](/documentation/articles/sql-database-always-encrypted/) | 在数分钟内保护 SQL 数据库中的敏感数据。 |
| 130 | [始终加密 - 使用数据加密保护 SQL 数据库中的敏感数据并将加密密钥存储在 Azure 密钥保管库中](/documentation/articles/sql-database-always-encrypted-azure-key-vault/) | 在数分钟内保护 SQL 数据库中的敏感数据。 |
| 131 | [SQL 数据库 - 针对审核的下层客户端支持和 IP 终结点更改](/documentation/articles/sql-database-auditing-and-dynamic-data-masking-downlevel-clients/) | 了解有关针对 SQL 数据库审核的下层客户端支持和 IP 终结点更改的信息。 |
| 132 | [使用 PowerShell 配置 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-powershell/) | 了解如何为访问 Azure SQL 数据库的 IP 地址配置防火墙。 |
| 133 | [使用 REST API 配置 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-rest/) | 了解如何为访问 Azure SQL 数据库的 IP 地址配置防火墙。 |
| 134 | [使用 T-SQL 配置 Azure SQL 数据库服务器级和数据库级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-tsql/) | 了解如何为访问 Azure SQL 数据库的 IP 地址配置防火墙。 |
| 135 | [SQL 数据库动态数据掩码入门（Azure 门户预览）](/documentation/articles/sql-database-dynamic-data-masking-get-started/) | 如何开始在 Azure 门户预览中使用 SQL 数据库动态数据掩码 |
| 136 | [SQL 数据库动态数据掩码入门（Azure 经典管理门户）](/documentation/articles/sql-database-dynamic-data-masking-get-started-portal/) | 如何开始在 Azure 经典门户中使用 SQL 数据库动态数据掩码 |
| 137 | [配置 Azure SQL 数据库防火墙规则 - 概述](/documentation/articles/sql-database-firewall-configure/) | 了解如何配置具有服务器级和数据库级防火墙规则的 SQL 数据库防火墙，以管理访问权限。 |
| 138 | [SQL 数据库教程：使用 Azure 门户预览创建 SQL 数据库用户帐户以访问和管理数据库](/documentation/articles/sql-database-get-started-security/) | 了解如何创建用户帐户来访问和管理数据库。 |
| 139 | [SQL 数据库身份验证和授权：授予访问权限](/documentation/articles/sql-database-manage-logins/) | 了解 SQL 数据库安全管理，特别是如何通过服务器级的主体帐户管理数据库的访问和登录安全。 |
| 140 | [Azure SQL 数据库安全指南和限制](/documentation/articles/sql-database-security-guidelines/) | 了解与安全相关的 Azure SQL 数据库指南和限制。 |
| 141 | [SQL 数据库威胁检测入门](/documentation/articles/sql-database-threat-detection-get-started/) | 如何开始在 Azure 门户中使用 SQL 数据库威胁检测 |
| 142 | [如何在 Azure SQL 数据库中执行重置管理员密码等常见管理任务](/documentation/articles/sql-database-troubleshoot-permissions/) | 介绍如何在 SQL 数据库中执行常见管理任务。例如，重置管理员密码，授予和删除访问权限。 |



## 管理数据库

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 144 | [SQL 数据库审核入门](/documentation/articles/sql-database-auditing-get-started/) | SQL 数据库审核入门 |
| 145 | [使用 Azure 门户预览配置 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings/) | 了解如何为访问 Azure SQL Server 的 IP 地址配置防火墙。 |
| 146 | [使用 Azure 门户预览复制 Azure SQL 数据库](/documentation/articles/sql-database-copy-portal/) | 创建 Azure SQL 数据库的副本 |
| 147 | [使用 Azure 自动化管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-automation/) | 了解如何使用 Azure 自动化服务来管理大规模的 Azure SQL 数据库。 |
| 148 | [概述：SQL 数据库的管理工具](/documentation/articles/sql-database-manage-overview/) | 比较管理 Azure SQL 数据库的工具和选项 |
| 149 | [使用动态管理视图监视 Azure SQL 数据库](/documentation/articles/sql-database-monitoring-with-dmvs/) | 了解如何通过使用动态管理视图监视 Azure SQL 数据库来检测并诊断常见性能问题。 |
| 150 | [保护你的 SQL 数据库](/documentation/articles/sql-database-security/) | 了解有关 Azure SQL 数据库和 SQL Server 安全性的信息，包括云与本地 SQL Server 在身份验证、授权、连接安全性、加密和合规性方面的差异。 |



## 扩展事件

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 151 | [SQL 数据库中扩展事件的事件文件目标代码](/documentation/articles/sql-database-xevent-code-event-file/) | 提供一个双阶段代码示例的 PowerShell 和 Transact-SQL，该示例演示 Azure SQL 数据库的扩展事件中的事件文件目标。此方案的特定部分需要使用 Azure 存储。 |
| 152 | [SQL 数据库中扩展事件的环形缓冲区目标代码](/documentation/articles/sql-database-xevent-code-ring-buffer/) | 提供一个 Transact-SQL 代码示例，以帮助你快速轻松地在 Azure SQL 数据库中使用环形缓冲区目标。 |
| 153 | [SQL 数据库中的扩展事件](/documentation/articles/sql-database-xevent-db-diff-from-svr/) | 介绍 Azure SQL 数据库中的扩展事件 (XEvents)，以及这些事件会话与 Microsoft SQL Server 中的事件会话有怎样的细微差别。 |



## 异地复制

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 154 | [使用 Azure 门户预览为 Azure SQL 数据库启动计划内或计划外故障转移](/documentation/articles/sql-database-geo-replication-failover-portal/) | 使用 Azure 门户预览为 Azure SQL 数据库启动计划内或计划外故障转移 |
| 155 | [使用 PowerShell 为 Azure SQL 数据库启动计划内或计划外故障转移](/documentation/articles/sql-database-geo-replication-failover-powershell/) | 使用 PowerShell 为 Azure SQL 数据库启动计划内或计划外故障转移 |
| 156 | [使用 Transact-SQL 为 Azure SQL 数据库启动计划内或计划外故障转移](/documentation/articles/sql-database-geo-replication-failover-transact-sql/) | 使用 Transact-SQL 为 Azure SQL 数据库启动计划内或计划外故障转移 |
| 157 | [概述：SQL 数据库的活动异地复制](/documentation/articles/sql-database-geo-replication-overview/) | 可以使用活动异地复制在任何 Azure 数据中心设置 4 个数据库副本。 |
| 158 | [使用 Azure 门户预览为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-portal/) | 使用 Azure 门户预览为 Azure SQL 数据库配置异地复制 |
| 159 | [使用 PowerShell 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-powershell/) | 使用 PowerShell 为 Azure SQL 数据库配置活动异地复制 |
| 160 | [灾难恢复后如何管理 Azure SQL 数据库安全性](/documentation/articles/sql-database-geo-replication-security-config/) | 本主题介绍在数据库还原或故障转移后进行安全管理时的安全注意事项。 |
| 161 | [使用 Transact-SQL 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-transact-sql/) | 使用 Transact-SQL 为 Azure SQL 数据库配置异地复制 |
| 162 | [使用 Azure 门户预览从异地冗余备份中异地还原 Azure SQL 数据库](/documentation/articles/sql-database-geo-restore-portal/) | 从异地冗余备份中异地还原 Azure SQL 数据库（Azure 门户预览）。 |
| 163 | [使用 PowerShell 从异地冗余备份中还原 Azure SQL 数据库](/documentation/articles/sql-database-geo-restore-powershell/) | 从异地冗余备份中将 Azure SQL 数据库还原到新的服务器中 |



## 升级

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 164 | [已改善 Azure SQL 数据库中兼容级别为 130 的查询性能](/documentation/articles/sql-database-compatibility-level-query-performance-130/) | 用于确定哪种兼容级别最适合 Azure SQL 数据库或 Microsoft SQL Server 上的数据库的工具和步骤 |
| 165 | [使用 SQL 数据库活动异地复制管理云应用程序的滚动升级](/documentation/articles/sql-database-manage-application-rolling-upgrade/) | 了解如何使用 Azure SQL 数据库异地复制来支持云应用程序的在线升级。 |
| 166 | [使用 Azure 门户预览升级到 Azure SQL 数据库 V12](/documentation/articles/sql-database-upgrade-server-portal/) | 介绍如何使用 Azure 门户预览升级到 Azure SQL 数据库 V12，包括如何升级 Web 和企业数据库，以及如何升级 V11 服务器并将其数据库直接迁移到弹性数据库池。 |
| 167 | [使用 PowerShell 升级到 Azure SQL 数据库 V12](/documentation/articles/sql-database-upgrade-server-powershell/) | 介绍如何使用 PowerShell 升级到 Azure SQL 数据库 V12，包括如何升级 Web 和企业数据库，以及如何升级 V11 服务器并将其数据库直接迁移到弹性数据库池。 |
| 168 | [规划和准备升级到 SQL 数据库 V12](/documentation/articles/sql-database-v12-plan-prepare-upgrade/) | 介绍升级到 Azure SQL 数据库 V12 版本所涉及的准备工作和限制。 |
| 169 | [SQL 数据库 V12 中的新增功能](/documentation/articles/sql-database-v12-whats-new/) | 介绍云中使用 Azure SQL 数据库的业务系统在升级到版本 V12 后为何能够受益。 |
| 170 | [Web 和 Business Edition 停用常见问题](/documentation/articles/sql-database-web-business-sunset-faq/) | 了解 Azure SQL Web 和企业数据库何时停用，并了解新服务层的特性和功能。 |



## 工具

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 171 | [使用 PowerShell 管理 Azure SQL 数据库](/documentation/articles/sql-database-command-line-tools/) | 使用 PowerShell 管理 Azure SQL 数据库。 |
| 172 | [SQL 数据库教程：将 Excel 连接到 Azure SQL 数据库并创建报表](/documentation/articles/sql-database-connect-excel/) | 了解如何将 Microsoft Excel 连接到云中的 Azure SQL 数据库。将数据导入 Excel 以进行报告和数据探索。 |
| 173 | [使用 PowerShell cmdlet 创建新的 SQL 数据库并执行常见的数据库设置任务](/documentation/articles/sql-database-get-started-powershell/) | 了解如何使用 PowerShell 创建新的 SQL 数据库。可以通过 PowerShell cmdlet 管理常见的数据库设置任务。 |
| 174 | [Azure SQL 数据同步入门（预览）](/documentation/articles/sql-database-get-started-sql-data-sync/) | 本教程帮助你开始使用 Azure SQL 数据同步（预览）。 |
| 175 | [使用 SQL Server Management Studio 管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-azure-ssms/) | 了解如何使用 SQL Server Management Studio 管理 SQL 数据库服务器和数据库。 |
| 176 | [使用 Azure 门户预览管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-portal/) | 了解如何使用 Azure 门户预览管理云中的关系数据库。 |
| 177 | [使用 PowerShell 更改 SQL 数据库的服务层和性能级别（定价层）](/documentation/articles/sql-database-scale-up-powershell/) | “更改 Azure SQL 数据库的服务层和性能级别”介绍如何使用 PowerShell 扩展和缩减 SQL 数据库。使用 PowerShell 更改 Azure SQL 数据库定价层。 |
| 178 | 在 Azure RemoteApp 中使用 SQL Server Management Studio 连接到 SQL 数据库 | 通过本教程了解如何在连接到 SQL 数据库时使用 Azure RemoteApp 中的 SQL Server Management Studio 进行安全和性能操作 |



## 引用

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 179 | [用于 SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries/) | 列出每个客户端程序可以用来连接到 Azure SQL 数据库或 Microsoft SQL Server 的驱动程序的最低版本号。提供一个链接，通过它可查看由社区而不是 Microsoft 发布的驱动程序的版本信息。 |
| 180 | [Azure SQL 数据库资源限制](/documentation/articles/sql-database-resource-limits/) | 本页介绍 Azure SQL 数据库的一些常见资源限制。 |
| 181 | [Azure SQL 数据库 Transact-SQL 的差异](/documentation/articles/sql-database-transact-sql-information/) | Azure SQL 数据库中不完全支持的 Transact-SQL 语句 |



## 杂项

| &nbsp; | 标题 | 说明 |
| --: | :-- | :-- |
| 182 | [使用 PowerShell 复制 Azure SQL 数据库](/documentation/articles/sql-database-copy-powershell/) | 使用 PowerShell 创建 Azure SQL 数据库的副本 |
| 183 | [使用 Transact-SQL 复制 Azure SQL 数据库](/documentation/articles/sql-database-copy-transact-sql/) | 使用 Transact-SQL 创建 Azure SQL 数据库的副本 |
| 184 | [Azure SQL 数据库的一般性限制和指导原则](/documentation/articles/sql-database-general-limitations/) | 本页介绍 Azure SQL 数据库的某些一般性限制，以及互操作性和支持方面的问题。 |
| 185 | [SQL 数据库定价层建议](/documentation/articles/sql-database-service-tier-advisor/) | 在 Azure 门户预览中更改定价层时，提供的定价层建议会推荐最适合用于运行现有 Azure SQL 数据库工作负荷的层。定价层描述 SQL 数据库的服务层和性能级别。 |


&nbsp;

<hr/>

<a name="extras_append"></a>

## 附加信息

<a name="extra_related_articles"></a>

### 其他 Azure 服务的相关文章

- [在 Azure 中备份 Azure App Service 应用](/documentation/articles/web-sites-backup/)

<!---HONumber=Mooncake_Quality_Review_1215_2016-->