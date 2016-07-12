<properties 
	pageTitle="SQL 数据库对审核的下层客户端支持 | Azure" 
	description="了解 SQL 数据库对审核的下层客户端支持。" 
	services="sql-database" 
	documentationCenter="" 
	authors="nadavhelfman" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database"
	ms.date="11/12/2015"
	wacn.date="05/16/2016"/>
 
# SQL 数据库 - 对审核的下层客户端支持


对于支持 TDS 重定向的 SQL 客户端，可以自动使用[审核](/documentation/articles/sql-database-auditing-get-started/)。

任何实现了 TDS 7.4 的客户端同样应当支持重定向。例外情况包括不完全支持重定向功能的 JDBC 4.0 以及未实现重定向的 Tedious（适用于 Node.JS）。

对于“下层客户端”，即支持 TDS 7.3 版和更低版本的客户端 — 应修改连接字符串中的服务器 FQDN：

连接字符串中的原始服务器 FQDN：<*服务器名称*>.database.chinacloudapi.cn

连接字符串中修改后的服务器 FQDN：<*服务器名称*>.database.**secure**.chinacloudapi.cn

“下层客户端”的部分列表包括：

- .NET 4.0 和更低版本，
- ODBC 10.0 和更低版本。
- JDBC（JDBC 虽然支持 TDS 7.4，但不完全支持 TDS 重定向功能）
- Tedious（适用于 Node.JS）

**注释：**上面的服务器 FDQN 修改可能还可用于应用 SQL Server 级别的审核策略，而无需在每个数据库中进行配置（临时缓解）。

<!---HONumber=Mooncake_1207_2015-->