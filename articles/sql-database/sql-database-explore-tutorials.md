<properties
    pageTitle="浏览 Azure SQL 数据库教程 | Azure"
    description="了解 SQL 数据库的特性和功能"
    keywords=""
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="04c0fd7f-d260-4e43-a4f0-41cdcd5e3786"
    ms.service="sql-database"
    ms.custom="overview"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-management"
    ms.date="12/08/2016"
    wacn.date="01/20/2017"
    ms.author="carlrab" />  


# 浏览 Azure SQL 数据库教程

以下链接可以转到每个列出的功能区域的概述，以及每个区域的简单的分步入门教程。有关在基于实际情况的完整解决方案中演示 SQL 数据库使用的解决方案范围内的快速入门，请参阅 [Azure SQL 数据库解决方案快速入门](/documentation/articles/sql-database-solution-quick-starts/)。

## 使用 SQL Server Management Studio
以下教程介绍如何使用 SQL Server Management Studio 管理和查询 Azure SQL 数据库。


> [AZURE.IMPORTANT] 建议始终使用最新版本的 Management Studio 以保持与 Azure 和 SQL 数据库的更新同步。[更新 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。


| 教程 | 说明 |
|---|---|---|
| [使用服务器级主体登录名连接到 Azure SQL 数据库](/documentation/articles/sql-database-get-started-security/#connect-to-azure-sql-database-using-sql-server-authentication)| 本教程介绍如何使用 SQL Server 级主体登录名连接到 Azure SQL 数据库。|
| [以用户身份连接到 Azure SQL 数据库](/documentation/articles/sql-database-get-started-security/#connect-to-azure-sql-database-as-a-user) | 本教程介绍如何使用数据库级用户帐户连接到 Azure SQL 数据库。|
||||

## 创建服务器、数据库和服务器级防火墙规则
在以下教程中，用户可以创建服务器、数据库和服务器级防火墙规则，并了解如何连接和查询服务器和数据库。

| 教程 | 说明 |
| --- | --- | --- |
| [开始使用 Azure 门户预览和 SQL Server Management Studio 了解 Azure SQL 数据库服务器、数据库和防火墙规则](/documentation/articles/sql-database-get-started/) | 在本教程中，将使用 Azure 门户预览和 SQL Server Management Studio 创建一个 Azure SQL 数据库逻辑服务器，查看其属性，创建服务器级防火墙，连接到 master 数据库，创建示例数据库，查看其属性，连接到示例数据库，并创建一个空数据库。 |
| [开始使用 Azure PowerShell 了解 Azure SQL 数据库服务器、数据库和防火墙规则](/documentation/articles/sql-database-get-started-powershell/) | 在本教程中，将使用 PowerShell 创建一个 Azure SQL 数据库逻辑服务器，查看其属性，创建服务器级防火墙，连接到 master 数据库，创建示例数据库，查看其属性，连接到示例数据库，并创建一个空数据库。 |
| [使用 C# 通过适用于 .NET 的 SQL 数据库库创建 SQL 数据库](/documentation/articles/sql-database-get-started-csharp/)| 在本教程中，将使用 C# 创建 SQL 数据库服务器、防火墙规则和 SQL 数据库。还将创建一个 Active Directory (AD) 应用程序和对 C# 应用进行身份验证所需的服务主体。 |

## 备份和恢复
在以下教程中，会将数据库还原到某个时间点、配置长期备份保留，并从 Azure 恢复服务保管库中的保留还原数据库。

| 教程 | 说明 |
| --- | --- | --- |
| [开始使用备份和还原进行数据保护和恢复](/documentation/articles/sql-database-get-started-backup-recovery/) | 在此教程中，将使用 Azure 门户预览将数据库还原到某个时间点、配置长期备份保留，并从 Azure 恢复服务保管库中的保留还原数据库。 |


##<a name="elastic-pools"></a> 弹性池

以下教程介绍如何使用[弹性池](/documentation/articles/sql-database-elastic-pool/)管理使用模式变化很大且不可预测的多个数据库的性能目标。

| 教程 | 说明 |
|---|---|---|
| [创建弹性池](/documentation/articles/sql-database-elastic-pool-create-portal/) | 本教程介绍如何创建 Azure SQL 数据库的可缩放的池。 |
| [监视弹性数据库](/documentation/articles/sql-database-elastic-pool-manage-portal/#elastic-database-monitoring) | 本教程介绍如何监视单个弹性数据库的潜在问题。 |
| [向池资源添加警报](/documentation/articles/sql-database-elastic-pool-manage-portal/#add-an-alert-to-a-pool-resource) | 本教程介绍如何向资源添加规则，以便在资源达到设置的利用率阈值时，向人员发送电子邮件或是向 URL 终结点发送警报字符串。 |
| [将数据库移入弹性池](/documentation/articles/sql-database-elastic-pool-manage-portal/#move-a-database-into-an-elastic-pool) | 本教程介绍如何将数据库移入弹性池。 |
| [将数据库移出弹性池](/documentation/articles/sql-database-elastic-pool-manage-portal/#move-a-database-out-of-an-elastic-pool) | 本教程介绍如何将数据库移出弹性池。 |
| [更改池的性能设置](/documentation/articles/sql-database-elastic-pool-manage-portal/#change-performance-settings-of-a-pool) | 本教程介绍如何调整池的性能和存储限制。 |
||||



## 弹性查询

以下教程介绍如何运行[弹性查询](/documentation/articles/sql-database-elastic-query-overview/)。

| 教程 | 说明 |
|---|---|---|
| [跨横向分区（分片）数据库查询）](/documentation/articles/sql-database-elastic-query-getting-started/) | 本教程介绍如何使用[弹性查询](/documentation/articles/sql-database-elastic-query-overview/)从位于横向分区（分片）数据库中的所有数据库创建报表 |
| [跨垂直分区数据库查询）](/documentation/articles/sql-database-elastic-query-getting-started-vertical/#create-database-objects) | 本教程介绍如何使用[弹性查询](/documentation/articles/sql-database-elastic-query-overview/)从位于纵向分区数据库中的所有数据库创建报表 |
| [迁移要横向扩展的现有数据库](/documentation/articles/sql-database-elastic-convert-to-use-elastic-tools/)| 本教程介绍如何横向扩展（分片）Azure SQL 数据库。 |
||||

## 性能优化

以下教程介绍如何优化[单一数据库的性能](/documentation/articles/sql-database-performance-guidance/)。有关多个数据库的性能优化，请参阅[弹性池](#elastic-pools)。

| 教程 | 说明 |
|---|---|---|
| [更改数据库的服务层和性能级别](/documentation/articles/sql-database-scale-up/#change-the-service-tier-and-performance-level-of-your-database) | 本教程介绍如何使用服务层扩展或缩减 Azure SQL 数据库的性能。 |
| [SQL 数据库顾问 Query Performance Insight](/documentation/articles/sql-database-performance/#performance-overview) | 本教程介绍如何打开和使用 SQL 数据库顾问 Query Performance Insight。|
| [SQL 数据库顾问性能建议](/documentation/articles/sql-database-advisor/#viewing-recommendations) | 本教程介绍如何查看和应用 SQL 数据库顾问性能建议。 |
| [查看 DTU 消耗量靠前的查询](/documentation/articles/sql-database-query-performance/#review-top-cpu-consuming-queries)| 本教程介绍如何使用 SQL 数据库顾问 Query Performance Insight 来查看 CPU 消耗量靠前的查询。|
| [查看单个查询的详细信息](/documentation/articles/sql-database-query-performance/#viewing-individual-query-details)| 本教程介绍如何使用 SQL 数据库顾问 Query Performance Insight 来查看单个查询性能的详细信息。|
||||

## SQL 数据库迁移和存档 

以下教程介绍如何[将现有的 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。

| 教程 | 说明 |
|---|---|---|
| [使用 SQL Server Data Tools for Visual Studio 检测兼容性问题](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/#detecting-compatibility-issues-using-sql-server-data-tools-for-visual-studio) | 本教程介绍如何使用 SQL Server Data Tools for Visual Studio 确定 Azure SQL 数据库的兼容性。 |
| [使用 SQL Server Data Tools for Visual Studio 解决兼容性问题](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/#fixing-compatibility-issues-using-sql-server-data-tools-for-visual-studio) | 本教程介绍如何使用 SQL Server Data Tools for Visual Studio 修复 Azure SQL 数据库的兼容性问题。 |
| [使用 SqlPackage.exe 确定 SQL 数据库兼容性](/documentation/articles/sql-database-cloud-migrate-determine-compatibility-sqlpackage/#using-sqlpackageexe) | 本教程介绍如何使用 SQLPackage.exe 命令行实用工具确定 Azure SQL 数据库的兼容性。|
| [使用 SSMS 确定 SQL 数据库兼容性](/documentation/articles/sql-database-cloud-migrate-determine-compatibility-ssms/#using-sql-server-management-studio) |本教程介绍如何使用 SQL Server Management Studio 确定 Azure SQL 数据库的兼容性。|
| [使用“将数据库部署到 Azure 数据库向导”将 SQL Server 数据库迁移到 SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-using-ssms-migration-wizard/#use-the-deploy-database-to-microsoft-azure-database-wizard) | 本教程介绍如何在 SQL Server Management Studio 中使用“将数据库部署到 Azure 数据库向导”将兼容的 SQL Server 数据库迁移到 Azure SQL 数据库。
| [使用 SSMS 将 SQL Server 数据库导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/) | 本教程介绍如何在 SQL Server Management Studio 中使用“导出数据层应用程序向导”将兼容的 SQL Server 数据库导出到 BACPAC 文件。|
| [使用 SqlPackage 将 SQL Server 数据库导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-sqlpackage/) | 本教程介绍如何使用 SQLPackage.exe 命令行实用工具将兼容的 SQL Server 数据库导出到 BACPAC 文件。|
| [使用 SSMS 将 BACPAC 文件导入 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/) | 本教程介绍如何在 SQL Server Management Studio 中使用“导出数据层应用程序向导”将数据库从 BACPAC 文件导入 Azure SQL 数据库。 |
| [使用 SqlPackage 将 BACPAC 文件导入 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage/#import-from-a-bacpac-file-into-azure-sql-database-using-sqlpackage) | 本教程介绍如何使用 SQLPackage 命令行实用工具将数据库从 BACPAC 文件导入 Azure SQL 数据库。 |
| [使用 Azure 门户预览将 BACPAC 文件导入 Azure SQL 数据库](/documentation/articles/sql-database-import/) | 本教程介绍如何使用 Azure 门户预览将数据库从存储在 Azure blob 中的 BACPAC 文件导入 Azure SQL 数据库。|
| [使用 PowerShell 将 BACPAC 文件导入 Azure SQL 数据库](/documentation/articles/sql-database-import-powershell/) | 本教程介绍如何使用 PowerShell 将数据库从 BACPAC 文件导入 Azure SQL 数据库。|
| [使用 Azure 门户预览存档 Azure SQL 数据库](/documentation/articles/sql-database-export/#export-your-database) | 本教程介绍如何使用 Azure 门户预览将 Azure SQL 数据库存档到 BACPAC 文件。 |
| [使用 PowerShell 存档 Azure SQL 数据库](/documentation/articles/sql-database-export-powershell/) | 本教程介绍如何使用 PowerShell 将 Azure SQL 数据库存档到 BACPAC 文件。 |
| [使用 Azure 门户预览复制 Azure SQL 数据库](/documentation/articles/sql-database-copy/#copy-your-sql-database) | 本教程介绍如何使用 Azure 门户预览复制 Azure SQL 数据库。 |
| [使用 PowerShell 复制 Azure SQL 数据库](/documentation/articles/sql-database-copy-powershell/#copy-your-sql-database) | 本教程介绍如何使用 PowerShell 复制 Azure SQL 数据库。 |
| [使用 Transact-SQL 复制 Azure SQL 数据库](/documentation/articles/sql-database-copy-transact-sql/#copy-your-sql-database) | 本教程介绍如何使用 Transact-SQL 复制 Azure SQL 数据库。 |
||||

##开发

以下教程介绍 [SQL 数据库开发](/documentation/articles/sql-database-develop-overview/)以及使用[连接库](/documentation/articles/sql-database-libraries/)的相关信息。

| 教程 | 说明 |
|---|---|---|
| [使用 .NET (C#) 连接到 SQL 数据库](/documentation/articles/sql-database-develop-dotnet-simple/) | 本教程介绍如何使用 C# 连接到 Azure SQL 数据库。 |
| [使用 Java 连接到 SQL 数据库](/documentation/articles/sql-database-develop-java-simple/) | 本教程介绍如何使用 Java 连接到 Azure SQL 数据库。 |
| [使用 Node.js 连接到 SQL 数据库](/documentation/articles/sql-database-develop-nodejs-simple/) | 本教程介绍如何使用 Node.js 连接到 Azure SQL 数据库。 |
| [使用 PHP 连接到 SQL 数据库](/documentation/articles/sql-database-develop-php-simple/) | 本教程介绍如何使用 PHP 连接到 Azure SQL 数据库。 |
| [使用 Python 连接到 SQL 数据库](/documentation/articles/sql-database-develop-python-simple/) | 本教程介绍如何使用 Python 连接到 Azure SQL 数据库。 |
| [使用 Ruby 连接到 SQL 数据库](/documentation/articles/sql-database-develop-ruby-simple/) | 本教程介绍如何使用 Ruby 连接到 Azure SQL 数据库。 |
||||
 
## 数据库身份验证和授权
以下教程介绍如何[创建和管理登录名和用户](/documentation/articles/sql-database-manage-logins/)。

| 教程 | 说明 |
|---|---|---|
| [使用 Azure 门户预览创建 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings/) | 本教程介绍如何使用 Azure 门户预览配置 SQL 数据库服务器级防火墙。 |
| [使用 Transact-SQL 创建数据库级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-tsql/#database-level-firewall-rules) | 本教程介绍如何使用 Transact-SQL 创建数据库级防火墙规则。|
| [使用 Transact-SQL 管理服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-tsql/#manage-server-level-firewall-rules-through-transact-sql) | 本教程介绍如何使用 Transact-SQL 管理 Azure SQL 数据库服务器级防火墙。|
| [使用 PowerShell 管理服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-powershell/#manage-firewall-rules-using-powershell) | 本教程介绍如何使用 PowerShell 管理 Azure SQL 数据库服务器级防火墙。|
| [使用 REST API 管理服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-rest/#manage-firewall-rules-using-the-service-management-rest-api) | 本教程介绍如何使用 RESET API 管理 Azure SQL 数据库服务器级防火墙。|
| [使用服务器级主体登录名连接到 Azure SQL 数据库](/documentation/articles/sql-database-get-started-security/#connect-to-azure-sql-database-using-a-server-level-principal-login)| 本教程介绍如何使用服务器级主体登录名连接到 Azure SQL 数据库。|
| [向登录名授予对数据库的访问权限](/documentation/articles/sql-database-manage-logins/#granting-database-access-to-a-login) | 本教程介绍如何向服务器级的登录名授予数据库访问权限。|
| [使用 SSMS 创建新数据库用户](/documentation/articles/sql-database-get-started-security/#create-new-database-user-using-ssms) | 本教程介绍如何使用 SSMS 在现有数据库中创建新的数据库用户。|
| [授予新的数据库用户 db\_owner 权限](/documentation/articles/sql-database-get-started-security/#grant-new-database-user-dbowner-permissions) | 本教程介绍如何授予现有数据库用户 db\_owner 权限。|
| [以用户身份连接到 Azure SQL 数据库](/documentation/articles/sql-database-get-started-security/#connect-to-azure-sql-database-as-a-user) | 本教程介绍如何使用数据库级用户帐户连接到 Azure SQL 数据库。|
||||

## 保护数据
以下教程介绍如何[保护 Azure SQL 数据库数据](/documentation/articles/sql-database-security/)。

| 教程 | 说明 |
|---|---|---|
| [使用 Azure 门户预览为数据库启用威胁检测](/documentation/articles/sql-database-threat-detection-get-started/#set-up-threat-detection-for-your-database) | 本教程介绍如何在 Azure 门户预览中为数据库设置威胁检测。|
| [使用“始终加密”保护敏感数据](/documentation/articles/sql-database-always-encrypted-azure-key-vault/) | 本教程介绍如何使用“始终加密”向导保护 Azure SQL 数据库中的敏感数据。|
| [使用透明数据加密保护敏感数据](https://msdn.microsoft.com/zh-cn/library/dn948096.aspx)| 本教程介绍如何使用透明数据加密保护敏感数据。|
| [加密一列数据](https://msdn.microsoft.com/zh-cn/library/ms179331.aspx)| 本教程介绍如何使用 Transact-SQL 加密一列数据。|
| [设置动态数据掩码](/documentation/articles/sql-database-dynamic-data-masking-get-started/#set-up-dynamic-data-masking-for-your-database-using-the-azure-portal) | 本教程介绍如何为 Azure SQL 数据库设置动态数据掩码。 |
||||

## 业务连续性
以下教程介绍如何使用[异地还原和活动异地复制](/documentation/articles/sql-database-business-continuity/)从错误中恢复过来，以保持业务连续性和实现查询扩展。

| 教程 | 说明 |
|---|---|---|
| [使用 Azure 门户预览将 Azure SQL 数据库还原到之前的时间点](/documentation/articles/sql-database-point-in-time-restore-portal/)| 本教程介绍如何使用 Azure 门户预览将数据库还原到较早的时间点。|
| [使用 PowerShell 将 Azure SQL 数据库还原到之前的时间点](/documentation/articles/sql-database-point-in-time-restore-powershell/) | 本教程介绍如何使用 PowerShell 将数据库还原到较早的时间点|
| [使用 Azure 门户预览还原已删除的 Azure SQL 数据库](/documentation/articles/sql-database-restore-deleted-database-portal/) | 本教程介绍如何使用 Azure 门户预览还原已删除的数据库。|
| [使用 PowerShell 还原已删除的 Azure SQL 数据库](/documentation/articles/sql-database-restore-deleted-database-powershell/) | 本教程介绍如何使用 PowerShell 还原已删除的数据库。|
| [使用 Azure 门户预览为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-portal/)| 本教程介绍如何使用 Azure 门户预览配置活动异地复制。|
| [使用 PowerShell 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-powershell/)| 本教程介绍如何使用 Transact-SQL 配置活动异地复制。|
| [使用 Transact-SQL 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-transact-sql/)| 本教程介绍如何使用 Transact-SQL 配置活动异地复制。|
| [使用 Azure 门户预览为 Azure SQL 数据库启动计划内或计划外的故障转移](/documentation/articles/sql-database-geo-replication-failover-portal/) | 本教程介绍如何使用 Azure 门户预览故障转移到异地复制的次要副本。|
| [使用 PowerShell 为 Azure SQL 数据库启动计划内或计划外的故障转移](/documentation/articles/sql-database-geo-replication-failover-powershell/) | 本教程介绍如何使用 PowerShell 故障转移到异地复制的次要副本。|
| [使用 Transact-SQL 为 Azure SQL 数据库启动计划内或计划外的故障转移](/documentation/articles/sql-database-geo-replication-failover-transact-sql/) | 本教程介绍如何使用 Transact-SQL 故障转移到异地复制的次要副本。|
||||

## 数据同步

本教程介绍如何进行[数据同步](http://download.microsoft.com/download/4/E/3/4E394315-A4CB-4C59-9696-B25215A19CEF/SQL_Data_Sync_Preview.pdf)。

| 教程 | 说明 |
|---|---|---| 
| [Azure SQL 数据同步入门（预览）](/documentation/articles/sql-database-get-started-sql-data-sync/) | 本教程介绍使用 Azure 经典门户进行 Azure SQL 数据同步的基础知识。 |
||||

## 后续步骤

[浏览 Azure SQL 数据库解决方案快速入门](/documentation/articles/sql-database-solution-quick-starts/)

<!---HONumber=Mooncake_0116_2017-->
<!--update: add sections("创建服务器、数据库和服务器级防火墙规则",备份和恢复); wording update-->