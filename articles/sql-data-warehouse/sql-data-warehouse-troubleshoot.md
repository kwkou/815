<properties
   pageTitle="排查 Azure SQL 数据仓库问题 | Azure"
   description="排查 Azure SQL 数据仓库问题。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="07/18/2016"
   wacn.date="09/05/2016"/>

# 排查 Azure SQL 数据仓库问题

本主题列出了客户较常见的一些故障排除问题。

## 连接

| 问题 | 解决方法 |
| :----------------------------------| :---------------------------------------------- |
| CTAIP 错误 | 当登录名已在 SQL Server master 数据库中创建，但未在 SQL 数据仓库数据库中时，可能会出现此错误。如果你遇到此错误，请参阅[安全性概述][]一文。本文介绍如何在 master 中创建登录名，然后如何在 SQL 数据仓库数据库中创建用户。|
| 被防火墙阻止 |为了确保只有已知的 IP 地址可以访问数据库，Azure SQL 数据库受到服务器和数据库级别的防火墙保护。默认情况下，防火墙是安全的，这意味着，你需要显式启用单个 IP 地址或地址范围才能进行连接。若要配置防火墙的访问权限，请遵循[预配说明][]中的[为客户端 IP 配置服务器防火墙访问权限][]中所述的步骤。|
| 无法使用工具或驱动程序进行连接 | SQL 数据仓库建议使用 [Visual Studio 2013 或 2015][] 来查询数据。针对客户端连接，建议使用 [SQL Server Native Client 10/11 (ODBC)][]。|


## 工具

| 问题 | 解决方法 |
| :----------------------------------| :---------------------------------------------- |
| Visual Studio 对象资源管理器缺少 AAD 用户 | 这是已知问题。解决方法是在 [sys.database\_principals][] 中查看这些用户。若要详细了解如何将 Azure Active Directory 用于 SQL 数据仓库，请参阅[向 Azure SQL 数据仓库进行身份验证][]。|

## 性能

| 问题 | 解决方法 |
| :----------------------------------| :---------------------------------------------- |
| 查询性能故障排除 | 如果你要尝试对特定查询进行故障排除，请从 [Learning how to monitor your queries][]（学习如何监视查询）开始。|
| 查询性能和计划不佳通常是由于缺少统计信息 | 性能不佳的最常见原因是缺少数据表的统计信息。有关如何创建统计信息以及统计信息为何对性能至关重要的详细信息，请参阅[维护表的统计信息][Statistics]。|
| 低并发性/查询排队 | 若要了解如何利用并发性平衡内存分配，了解[工作负荷管理][]很重要。|
| 如何实施最佳做法 | 开始了解如何提高查询性能的最好地方是 [SQL 数据仓库最佳实践][]一文。|
| 如何通过缩放提高性能 | 有时，改进性能的解决方案是只需通过[缩放 SQL 数据仓库][]来提升计算能力。|
| 由于索引质量不佳导致查询性能不佳 | 有时，由于[列存储索引质量不佳][]，查询速度可能会减慢。有关详细信息以及如何[重建索引以提高段质量][]，请参阅本文。|

## 系统管理

| 问题 | 解决方法 |
| :----------------------------------| :---------------------------------------------- |
| 消息 40847：无法执行操作，因为服务器将超过 45000 这一允许的数据库吞吐量单元配额。 | 请减少要尝试创建的数据库的 [DWU][]，或者[请求增加配额][]。|
| 调查空间使用率 | 请参阅[表大小][]，了解系统的空间使用率。|
| 管理表的帮助 | 有关管理表的帮助，请参阅[表概述][Overview]一文。本文还包含指向更详细主题的链接，如[表数据类型][Data types]、[分布表][Distribute]、[为表编制索引][Index]、[将表分区][Partition]、[维护表统计信息][Statistics]和[临时表][Temporary]。|

## Polybase

| 问题 | 解决方法 |
| :----------------------------------| :---------------------------------------------- |
| UTF-8 错误 | 目前 PolyBase 仅支持加载已进行 UTF-8 编码的数据文件。有关如何解决此限制的指导，请参阅 [Working around the PolyBase UTF-8 requirement][]（绕开 PolyBase UTF-8 要求）。|
| 由于行大加载失败 | 目前 Polybase 不提供大型行支持。这意味着，如果你的表包含 VARCHAR(MAX)、NVARCHAR(MAX) 或 VARBINARY(MAX)，将无法使用外部表加载数据。目前仅支持通过 Azure 数据工厂（带 BCP）、Azure 流分析、SSIS、BCP 或 .NET SQLBulkCopy 类加载大型行。在未来版本中将添加对大型行的 PolyBase 支持。|
| 使用 bcp 加载包含 MAX 数据类型的表失败 | 有一个已知问题，它要求在某些情况下将 VARCHAR(MAX)、NVARCHAR(MAX) 或 VARBINARY(MAX) 放置在表的末尾。请尝试将 MAX 列移到表的末尾。|

## 与 SQL 数据库的差异

| 问题 | 解决方法 |
| :----------------------------------| :---------------------------------------------- |
| 不支持的 SQL 数据库功能 | 请参阅[不支持的表功能][]。|
| 不支持的 SQL 数据库数据类型 | 请参阅[不支持的数据类型][]。|
| DELETE 和 UPDATE 限制 | 请参阅 [UPDATE 解决方法][]、[DELETE 解决方法][]和[使用 CTAS 解决不支持的 UPDATE 和 DELETE 语法][]。 |
| 不支持 MERGE 语句 | 请参阅 [MERGE 解决方法][]。|
| 存储过程限制 | 请参阅[存储过程限制][]，了解存储过程的一些限制。|
| UDF 不支持 SELECT 语句 | 这是 UDF 的当前一项限制。有关我们支持的语法，请参阅 [CREATE FUNCTION][]。 |
'<--LocComment: 找不到页面，已打破“存储过程限制”。我已尝试修复“文章参考”中的链接 -->'

## 后续步骤

如果你找不到上述问题的解决方案，你可以尝试下面列出的一些其他资源。

- [博客]
- [功能请求]
- [CAT 团队博客]
- [MSDN 论坛]

<!--Image references-->

<!--Article references-->
[安全性概述]: /documentation/articles/sql-data-warehouse-overview-manage-security/
[Create support ticket]: /documentation/articles/sql-data-warehouse-get-started-create-support-ticket/
[缩放 SQL 数据仓库]: /documentation/articles/sql-data-warehouse-manage-compute-overview/
[DWU]: /documentation/articles/sql-data-warehouse-overview-what-is#data-warehouse-units
[请求增加配额]: /documentation/articles/sql-data-warehouse-get-started-create-support-ticket#request-quota-change
[Learning how to monitor your queries]: /documentation/articles/sql-data-warehouse-manage-monitor/
[预配说明]: /documentation/articles/sql-data-warehouse-get-started-provision-powershell/
[为客户端 IP 配置服务器防火墙访问权限]: /documentation/articles/sql-data-warehouse-get-started-provision-powershell/#create-a-new-azure-sql-server-level-firewall
[Visual Studio 2013 或 2015]: /documentation/articles/sql-data-warehouse-get-started-connect/
[SQL 数据仓库最佳实践]: /documentation/articles/sql-data-warehouse-best-practices/
[表大小]: /documentation/articles/sql-data-warehouse-tables-overview/#table-size-queries
[不支持的表功能]: /documentation/articles/sql-data-warehouse-tables-overview/#unsupported-table-features
[不支持的数据类型]: /documentation/articles/sql-data-warehouse-tables-data-types/#unsupported-data-types
[Overview]: /documentation/articles/sql-data-warehouse-tables-overview/
[Data types]: /documentation/articles/sql-data-warehouse-tables-data-types/
[Distribute]: /documentation/articles/sql-data-warehouse-tables-distribute/
[Index]: /documentation/articles/sql-data-warehouse-tables-index/
[Partition]: /documentation/articles/sql-data-warehouse-tables-partition/
[Statistics]: /documentation/articles/sql-data-warehouse-tables-statistics/
[Temporary]: /documentation/articles/sql-data-warehouse-tables-temporary/
[列存储索引质量不佳]: /documentation/articles/sql-data-warehouse-tables-index/#causes-of-poor-columnstore-index-quality
[重建索引以提高段质量]: /documentation/articles/sql-data-warehouse-tables-index/#rebuilding-indexes-to-improve-segment-quality
[工作负荷管理]: /documentation/articles/sql-data-warehouse-develop-concurrency/
[使用 CTAS 解决不支持的 UPDATE 和 DELETE 语法]: /documentation/articles/sql-data-warehouse-develop-ctas/#using-ctas-to-work-around-unsupported-features
[UPDATE 解决方法]: /documentation/articles/sql-data-warehouse-develop-ctas/#ansi-join-replacement-for-update-statements
[DELETE 解决方法]: /documentation/articles/sql-data-warehouse-develop-ctas/#ansi-join-replacement-for-delete-statements
[MERGE 解决方法]: /documentation/articles/sql-data-warehouse-develop-ctas/#replace-merge-statements
[存储过程限制]: /documentation/articles/sql-data-warehouse-develop-stored-procedures#limitations
[向 Azure SQL 数据仓库进行身份验证]: /documentation/articles/sql-data-warehouse-authentication/
[Working around the PolyBase UTF-8 requirement]: /documentation/articles/sql-data-warehouse-load-polybase-guide/#working-around-the-polybase-utf-8-requirement

<!--MSDN references-->
[SQL Server Native Client 10/11 (ODBC)]: https://msdn.microsoft.com/zh-cn/library/ms131415.aspx
[sys.database\_principals]: https://msdn.microsoft.com/zh-cn/library/ms187328.aspx
[CREATE FUNCTION]: https://msdn.microsoft.com/zh-cn/library/mt203952.aspx

<!--Other Web references-->

[博客]: https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/
[CAT 团队博客]: https://blogs.msdn.microsoft.com/sqlcat/tag/sql-dw/
[功能请求]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[MSDN 论坛]: https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureSQLDataWarehouse
[Stack Overflow forum]: http://stackoverflow.com/questions/tagged/azure-sqldw
[Twitter]: https://twitter.com/hashtag/SQLDW
[Videos]: https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse

<!---HONumber=Mooncake_0829_2016-->