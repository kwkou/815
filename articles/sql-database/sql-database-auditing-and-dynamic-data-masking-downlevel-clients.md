<properties
    pageTitle="Azure SQL 数据库的审核、TDS 重定向和 IP 终结点 | Azure"
    description="了解在 Azure SQL 数据库中实现表审核时，审核、TDS 重定向和 IP 终结点的变化。"
    services="sql-database"
    documentationcenter=""
    author="ronitr"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="4ef19ed1-e798-43a2-ad99-0e563f93ab53"
    ms.service="sql-database"
    ms.custom="secure and protect"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/05/2017"
    wacn.date="01/25/2017"
    ms.author="ronitr" />

# SQL 数据库 - 针对审核的下层客户端支持和 IP 终结点更改


对于支持 TDS 重定向的 SQL 客户端，可以自动使用 [SQL 数据库表审核](/documentation/articles/sql-database-auditing-get-started/)。请注意，使用 Blob 审核方法时，重定向不适用。

## <a id="subheading-1"></a>下层客户端支持
任何实现了 TDS 7.4 的客户端同样应当支持重定向。例外情况包括不完全支持重定向功能的 JDBC 4.0 以及未实现重定向的 Tedious（适用于 Node.JS）。

对于“下层客户端”，即支持 TDS 7.3 版和更低版本的客户端 — 应修改连接字符串中的服务器 FQDN：

连接字符串中的原始服务器 FQDN：<*服务器名称*>.database.chinacloudapi.cn

连接字符串中修改后的服务器 FQDN：<*服务器名称*>.database.**secure**.chinacloudapi.cn

“下层客户端”的部分列表包括：

- .NET 4.0 和更低版本，
- ODBC 10.0 和更低版本。
- JDBC（JDBC 虽然支持 TDS 7.4，但不完全支持 TDS 重定向功能）
- Tedious（适用于 Node.JS）

**注释：**上面的服务器 FDQN 修改可能还可用于应用 SQL Server 级别审核策略，而无需在每个数据库中进行配置（临时缓解）。

## <a id="subheading-2"></a>IP 终结点在启用审核时更改
请注意，启用表审核时，数据库的 IP 终结点将发生更改。如果具有严格的防火墙设置，请相应地更新这些防火墙设置。

新的数据库 IP 终结点将取决于数据库区域：

| 数据库区域 | 可能的 IP 终结点 |
|----------|---------------|
| 中国北部 | 139\.217.29.176、139.217.28.254 |
| 中国东部 | 42\.159.245.65、42.159.246.245 |

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: wording update-->