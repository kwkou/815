<properties
   pageTitle="浏览 SQL 数据库教程"
   description="了解 SQL 数据库的特性和功能"
   keywords=""
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="05/04/2016"
   wacn.date="06/14/2016"/>
   
# 浏览 SQL 数据库教程

以下链接可以转到每个列出的功能区域概述以及每个区域的快速入门教程。

## 使用 SQL Server Management Studio

在以下教程中，你将学习如何使用 SQL Server Management Studio 管理和查询 Azure SQL 数据库。

| 教程 | 说明 |
|---|---|---|
| [使用服务器级主体登录名连接到 Azure SQL 数据库](/documentation/articles/sql-database-get-started-security/#connect-to-azure-sql-database-using-a-server-level-principal-login)| 在本教程中，你将学习如何使用服务器级主体登录名连接到 Azure SQL 数据库。|
| [以用户身份连接到 Azure SQL 数据库](/documentation/articles/sql-database-get-started-security/#connect-to-azure-sql-database-as-a-user) | 在本教程中，你将学习如何使用数据库级用户帐户连接到 Azure SQL 数据库。|
||||

## 弹性池

在以下教程中，你将学习如何使用[弹性池](/documentation/articles/sql-database-elastic-pool/)管理使用模式变化很大且不可预测的多个数据库的性能目标。

| 教程 | 说明 |
|---|---|---|
| 创建弹性池 | 在本教程中，你将学习如何创建 Azure SQL 数据库的可缩放的池。 |
| 监视弹性数据库 | 在本教程中，你将学习如何监视单个弹性数据库的潜在问题。 |
| 向池资源添加警报 | 在本教程中，你将学习如何向资源添加规则，以便在资源达到设置的利用率阈值时，向人员发送电子邮件或是向 URL 终结点发送警报字符串。 |
| 将数据库移入弹性池 | 在本教程中，你将学习如何将数据库移入弹性池。 |
| 将数据库移出弹性池 | 在本教程中，你将学习如何将数据库移出弹性池。 |
| 更改池的性能设置   | 在本教程中，你将学习如何调整池的性能和存储限制。 |
||||

## 弹性数据库作业

在以下教程中，你将学习如何使用[弹性数据库作业](/documentation/articles/sql-database-elastic-jobs-overview/)。

| 教程 | 说明 |
|---|---|---|
| [弹性数据库工具入门](/documentation/articles/sql-database-elastic-scale-get-started/) | 在本教程中，你将学习如何使用简单的分片应用程序来使用弹性数据库工具的功能。 |
| [Azure SQL 数据库弹性作业入门](/documentation/articles/sql-database-elastic-jobs-getting-started/) | 在本教程中，你将学习如何创建和管理用于管理一组相关数据库的作业。 | 
||||

## 弹性查询

在以下教程中，你将学习如何运行[弹性查询](/documentation/articles/sql-database-elastic-query-overview/)。

| 教程 | 说明 |
|---|---|---|
| [跨横向分区（分片）数据库查询）](/documentation/articles/sql-database-elastic-query-getting-started/) | 在本教程中，你将学习如何使用[弹性查询](/documentation/articles/sql-database-elastic-query-overview/)从位于横向分区（分片）数据库中的所有数据库创建报表 |
| [跨垂直分区数据库查询）](/documentation/articles/sql-database-elastic-query-getting-started-vertical/#create-database-objects) | 在本教程中，你将学习如何使用[弹性查询](/documentation/articles/sql-database-elastic-query-overview/)从位于垂直分区（分片）数据库中的所有数据库创建报表 |
| [迁移要横向扩展的现有数据库](/documentation/articles/sql-database-elastic-convert-to-use-elastic-tools/)| 在本教程中，你将学习如何横向扩展（分片）Azure SQL 数据库。 |
||||

## 性能优化

在以下教程中，你将学习如何优化[单一数据库的性能](/documentation/articles/sql-database-performance-guidance/)。有关多个数据库的性能优化，请参阅[弹性池](#elastic-pools)。

| 教程 | 说明 |
|---|---|---|
| 更改数据库的服务层和性能级别 | 在本教程中，你将学习如何使用服务层扩展或缩减 Azure SQL 数据库的性能。 |
| SQL 数据库顾问 Query Performance Insight| 在本教程中，你将学习如何打开和使用 SQL 数据库顾问 Query Performance Insight。|
| SQL 数据库顾问性能建议 | 在本教程中，你将学习如何查看和应用 SQL 数据库顾问性能建议。 |
| 查看 DTU 消耗量靠前的查询 | 在本教程中，你将学习如何使用 SQL 数据库顾问 Query Performance Insight 来查看 CPU 消耗量靠前的查询。|
| 查看单个查询的详细信息 | 在本教程中，你将学习如何使用 SQL 数据库顾问 Query Performance Insight 来查看单个查询性能的详细信息。|
||||

## SQL 数据库迁移和存档 

在以下教程中，你将学习如何[将现有的 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。

| 教程 | 说明 |
|---|---|---|
| [使用 SQL Server Data Tools for Visual Studio 检测兼容性问题](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/#detecting-compatibility-issues-using-sql-server-data-tools-for-visual-studio) | 在本教程中，你将学习如何使用 SQL Server Data Tools for Visual Studio 确定 Azure SQL 数据库的兼容性。 |
| [使用 SQL Server Data Tools for Visual Studio 解决兼容性问题](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/#fixing-compatibility-issues-using-sql-server-data-tools-for-visual-studio) | 在本教程中，你将学习如何使用 SQL Server Data Tools for Visual Studio 修复 Azure SQL 数据库的兼容性问题。 |
| [使用 SqlPackage.exe 确定 SQL 数据库兼容性](/documentation/articles/ql-database-cloud-migrate-determine-compatibility-sqlpackage/#using-sqlpackageexe) | 在本教程中，你将学习如何使用 SQLPackage.exe 命令行实用工具确定 Azure SQL 数据库的兼容性。|
| [使用 SSMS 确定 SQL 数据库兼容性](/documentation/articles/sql-database-cloud-migrate-determine-compatibility-ssms/#using-sql-server-management-studio) |在本教程中，你将学习如何使用 SQL Server Management Studio 确定 Azure SQL 数据库的兼容性。|
| [使用“将数据库部署到 Azure 数据库”向导将 SQL Server 数据库迁移到 SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-using-ssms-migration-wizard/#use-the-deploy-database-to-microsoft-azure-database-wizard) | 在本教程中，你将学习如何在 SQL Server Management Studio 中使用“将数据库部署到 Azure 数据库”向导将兼容的 SQL Server 数据库迁移到 Azure SQL 数据库。
| [使用 SSMS 将 SQL Server 数据库导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/) | 在本教程中，你将学习如何在 SQL Server Management Studio 中使用“导出数据层应用程序”向导将兼容的 SQL Server 数据库导出到 BACPAC 文件。|
| [使用 SqlPackage 将 SQL Server 数据库导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-sqlpackage/) | 在本教程中，你将学习如何使用 SQLPackage.exe 命令行实用工具将兼容的 SQL Server 数据库导出到 BACPAC 文件。|
| [使用 PowerShell 将 BACPAC 文件导入 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/) | 在本教程中，你将学习如何在 SQL Server Management Studio 中使用“导出数据层应用程序”向导将数据库从 BACPAC 文件导入 Azure SQL 数据库。 |
| [使用 SqlPackage 将 BACPAC 文件导入 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage/#import-from-a-bacpac-file-into-azure-sql-database-using-sqlpackage) | 在本教程中，你将学习如何使用 SQLPackage 命令行实用工具将数据库从 BACPAC 文件导入 Azure SQL 数据库。 |
| 使用 Azure 门户将 BACPAC 文件导入 Azure SQL 数据库 | 在本教程中，你将学习如何使用 Azure 门户将数据库从存储在 Azure blob 中的 BACPAC 文件导入 Azure SQL 数据库。|
| [使用 PowerShell 将 BACPAC 文件导入 Azure SQL 数据库](/documentation/articles/sql-database-import-powershell/) | 在本教程中，你将学习如何使用 PowerShell 将数据库从 BACPAC 文件导入 Azure SQL 数据库。|
| 使用 Azure 门户存档 Azure SQL 数据库 | 在本教程中，你将学习如何使用 Azure 门户将 Azure SQL 数据库存档到 BACPAC 文件。 |
| [使用 PowerShell 存档 Azure SQL 数据库](/documentation/articles/sql-database-export-powershell/) | 在本教程中，你将学习如何使用 Azure 门户将 Azure SQL 数据库存档到 BACPAC 文件。 |
| 使用 Azure 门户复制 Azure SQL 数据库 | 在本教程中，你将学习如何使用 Azure 门户复制 Azure SQL 数据库。 |
| [使用 PowerShell 复制 Azure SQL 数据库](/documentation/articles/sql-database-copy-powershell/#copy-your-sql-database) | 在本教程中，你将学习如何使用 PowerShell 复制 Azure SQL 数据库。 |
| [使用 Transact-SQL 复制 Azure SQL 数据库](/documentation/articles/sql-database-copy-transact-sql/#copy-your-sql-database) | 在本教程中，你将学习如何使用 Transact-SQL 复制 Azure SQL 数据库。 |
||||

##开发

在以下教程中，你将了解 [Azure SQL 数据库的客户端连接](/documentation/articles/sql-database-connect-central-recommendations/)和使用[连接库](/documentation/articles/sql-database-libraries/)。

| 教程 | 说明 |
|---|---|---|
| [使用 .NET (C#) 连接到 SQL 数据库](/documentation/articles/sql-database-develop-dotnet-simple/) | 在本教程中，你将学习如何使用 C# 连接到 Azure SQL 数据库。 |
| [使用 Java 连接到 SQL 数据库](/documentation/articles/sql-database-develop-java-simple/) | 在本教程中，你将学习如何使用 Java 连接到 Azure SQL 数据库。 |
| [使用 Node.js 连接到 SQL 数据库](/documentation/articles/sql-database-develop-nodejs-simple/) | 在本教程中，你将学习如何使用 Node.js 连接到 Azure SQL 数据库。 |
| [使用 PHP 连接到 SQL 数据库](/documentation/articles/sql-database-develop-php-simple/) | 在本教程中，你将学习如何使用 PHP 连接到 Azure SQL 数据库。 |
| [使用 Python 连接到 SQL 数据库](/documentation/articles/sql-database-develop-python-simple/) | 在本教程中，你将学习如何使用 Python 连接到 Azure SQL 数据库。 |
| [使用 Ruby 连接到 SQL 数据库](/documentation/articles/sql-database-develop-ruby-simple/) | 在本教程中，你将学习如何使用 Ruby 连接到 Azure SQL 数据库。 |
||||
 
## 数据库访问

在以下教程中，你将学习如何[创建和管理登录名和用户](/documentation/articles/sql-database-manage-logins/)。

| 教程 | 说明 |
|---|---|---|
| 使用 Azure 门户创建 Azure SQL 数据库服务器级防火墙规则 | 在本教程中，你将学习如何使用 Azure 门户配置 SQL 数据库服务器级防火墙。 |
| [使用 Transact-SQL 创建数据库级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-tsql/#database-level-firewall-rules) | 在本教程中，你将学习如何使用 Transact-SQL 创建数据库级防火墙规则。|
| [使用 Transact-SQL 管理服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-tsql/#manage-server-level-firewall-rules-through-transact-sql) | 在本教程中，你将学习如何使用 Transact-SQL 管理 Azure SQL 数据库服务器级防火墙。|
| [使用 PowerShell 管理服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-powershell/#manage-firewall-rules-using-powershell) | 在本教程中，你将学习如何使用 PowerShell 管理 Azure SQL 数据库服务器级防火墙。|
| [使用 REST API 管理服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-rest/#manage-firewall-rules-using-the-service-management-rest-api) | 在本教程中，你将学习如何使用 RESET API 管理 Azure SQL 数据库服务器级防火墙。|
| [使用服务器级主体登录名连接到 Azure SQL 数据库](/documentation/articles/sql-database-get-started-security/#connect-to-azure-sql-database-using-a-server-level-principal-login)| 在本教程中，你将学习如何使用服务器级主体登录名连接到 Azure SQL 数据库。|
| [授予对登录名的数据库访问权限](/documentation/articles/sql-database-manage-logins/#granting-database-access-to-a-login) | 在本教程中，你将学习如何授予对服务器级登录名的数据库访问权限。|
| [使用 SSMS 创建新数据库用户](/documentation/articles/sql-database-get-started-security/#create-new-database-user-using-ssms) | 在本教程中，你将学习如何使用 SSMS 在现有数据库中创建新的数据库用户。|
| [授予新的数据库用户 db\_owner 权限](/documentation/articles/sql-database-get-started-security/#grant-new-database-user-dbowner-permissions) | 在本教程中，你将学习如何授予现有数据库用户 db\_owner 权限。|
| [以用户身份连接到 Azure SQL 数据库](/documentation/articles/sql-database-get-started-security/#connect-to-azure-sql-database-as-a-user) | 在本教程中，你将学习如何使用数据库级用户帐户连接到 Azure SQL 数据库。|
||||


## 数据安全

在以下教程中，你将学习如何[保护 Azure SQL 数据库数据](/documentation/articles/sql-database-security/)。

| 教程 | 说明 |
|---|---|---|
| 使用 Azure 门户为数据库启用威胁检测 | 在本教程中，你将学习在 Azure 门户中如何为数据库设置威胁检测。|
| [使用“始终加密”保护敏感数据](/documentation/articles/sql-database-always-encrypted-azure-key-vault/) | 在本教程中，你将使用“始终加密”向导保护 Azure SQL 数据库中的敏感数据。|
| [使用透明数据加密保护敏感数据](https://msdn.microsoft.com/zh-cn/library/dn948096.aspx)| 在本教程中，你将学习如何使用透明数据加密保护敏感数据。|
| [加密一列数据](https://msdn.microsoft.com/zh-cn/library/ms179331.aspx)| 在本教程中，你将学习如何使用 Transact-SQL 加密一列数据。|
| 设置动态数据掩码 | 在本教程中，你将学习如何为 Azure SQL 数据库设置动态数据掩码。 |
||||

## 业务连续性和查询横向扩展

在以下教程中，你将学习如何使用[异地还原和活动异地复制](/documentation/articles/sql-database-business-continuity/)从错误中恢复过来，以保持业务连续性和实现查询横向扩展。

| 教程 | 说明 |
|---|---|---|
| 使用 Azure 门户将 Azure SQL 数据库还原到之前的时间点 | 在本教程中，你将学习如何使用 Azure 门户将数据库还原到较早的时间点。|
| [使用 PowerShell 将 Azure SQL 数据库还原到之前的时间点](/documentation/articles/sql-database-point-in-time-restore-powershell/) | 在本教程中，你将学习如何使用 PowerShell 将数据库还原到较早的时间点|
| 使用 Azure 门户还原已删除的 Azure SQL 数据库 | 在本教程中，你将学习如何使用 Azure 门户还原已删除的数据库。|
| [使用 PowerShell 还原已删除的 Azure SQL 数据库](/documentation/articles/sql-database-restore-deleted-database-powershell/) | 在本教程中，你将学习如何使用 PowerShell 还原已删除的数据库。|
| 使用 Azure 门户为 Azure SQL 数据库配置异地复制 | 在本教程中，你将学习如何使用 Azure 门户配置活动异地复制。|
| [使用 PowerShell 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-powershell/)| 在本教程中，你将学习如何使用 Transact-SQL 配置活动异地复制。|
| [使用 Transact-SQL 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-transact-sql/)| 在本教程中，你将学习如何使用 Transact-SQL 配置活动异地复制。|
| 使用 Azure 门户为 Azure SQL 数据库启动计划内或计划外的故障转移 | 在本教程中，你将学习如何使用 Azure 门户故障转移到异地复制的次要副本。|
| [使用 PowerShell 为 Azure SQL 数据库启动计划内或计划外的故障转移](/documentation/articles/sql-database-geo-replication-failover-powershell/) | 在本教程中，你将学习如何使用 PowerShell 故障转移到异地复制的次要副本。|
| [使用 Transact-SQL 为 Azure SQL 数据库启动计划内或计划外的故障转移](/documentation/articles/sql-database-geo-replication-failover-transact-sql/) | 在本教程中，你将学习如何使用 Transact-SQL 故障转移到异地复制的次要副本。|
||||

## 数据同步

在本教程中，你将学习如何进行[数据同步](http://download.microsoft.com/download/4/E/3/4E394315-A4CB-4C59-9696-B25215A19CEF/SQL_Data_Sync_Preview.pdf)。

| 教程 | 说明 |
|---|---|---| 
| [Azure SQL 数据同步入门（预览版）](/documentation/articles/sql-database-get-started-sql-data-sync/) | 在本教程中，你将了解使用 Azure 管理门户的 Azure SQL 数据同步的基础知识。 |
||||

<!---HONumber=Mooncake_0530_2016-->
