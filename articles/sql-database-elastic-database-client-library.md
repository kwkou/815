<properties
    pageTitle="构建可缩放的云数据库 | Azure"
    description="使用弹性数据库客户端库构建可缩放的 .NET 数据库应用"
    services="sql-database"
    documentationCenter=""
    manager="jhubbard"
    authors="ddove"
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="04/26/2016"
    wacn.date="06/14/2016"/>

# 构建可缩放的云数据库

使用 SQL Azure 数据库的可缩放工具和功能，可以轻松地扩大数据库。特别是，你可以使用**弹性数据库客户端库**来创建和管理扩大的数据库。此功能可让你使用成百上千个 Azure SQL 数据库，轻松地开发分区应用程序。

若要安装该库，请转到 [Microsoft.Azure.SqlDatabase.ElasticScale.Client](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client)。

## 文档
1. [弹性数据库工具入门](/documentation/articles/sql-database-elastic-scale-get-started/)
* [弹性数据库功能](/documentation/articles/sql-database-elastic-scale-introduction/)
* [分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)
* [迁移要扩大的现有数据库](/documentation/articles/sql-database-elastic-convert-to-use-elastic-tools/)
* [依赖于数据的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)
* [多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying/)
* [使用弹性数据库工具添加分片](/documentation/articles/sql-database-elastic-scale-add-a-shard/)
* [具有弹性数据库工具和行级安全性的多租户应用程序](/documentation/articles/sql-database-elastic-tools-multi-tenant-row-level-security/)
* [升级客户端库应用](/documentation/articles/sql-database-elastic-scale-upgrade-client-library/) 
* [弹性查询概述](/documentation/articles/sql-database-elastic-query-overview/)
* [弹性数据库工具词汇表](/documentation/articles/sql-database-elastic-scale-glossary/)
* [将弹性数据库客户端库与实体框架配合使用](/documentation/articles/sql-database-elastic-scale-use-entity-framework-applications-visual-studio/)
* [弹性数据库客户端库与 Dapper](/documentation/articles/sql-database-elastic-scale-working-with-dapper/)
* [拆分-合并工具](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)
* [弹性数据库工具常见问题](/documentation/articles/sql-database-elastic-scale-faq/)

## 客户端功能

无论对于开发人员还是管理员，使用*分区*扩大应用程序都存在挑战。客户端库通过提供工具让开发人员和管理员管理扩大的数据库，简化了管理任务。在典型的示例中，有许多称为“分片”的数据库要管理。客户并置于同一数据库，并且每位客户（单租户方案）一个数据库。客户端库包含下列功能：

1.  **分片映射管理**：创建一个称为“分片映射管理器”的特殊数据库。分片映射管理是一种使应用程序能够管理其分片相关元数据的功能。开发人员可使用此功能将数据库注册为分片（描述各个分片键或键范围到这些数据库的映射），并随着数据库的数量和组成发展来维护此元数据，以反映容量更改。如果不使用弹性数据库客户端库，实现分片时你必须花费大量时间来编写管理代码。有关详细信息，请参阅[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。

* **数据相关的路由**：假设将一个请求传入应用程序。基于请求的分区键值，应用程序必须根据该键值判断正确的数据库。接着，它会与数据库建立连接来处理请求。借助数据依赖路由，您能够通过对应用程序的分片映射的单个简单调用打开连接。数据相关的路由是基础结构代码的另一个区域，现在它由弹性数据库客户端库中的功能所取代。有关详细信息，请参阅[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)。

* **多分片查询 (MSQ)**：当一个请求涉及多个（或所有）分片时，多分片查询将生效。多分片查询在所有分片或一组分片上执行相同的 T-SQL 代码。使用 UNION ALL 语义，将参与分片中的结果合并到一个总结果集中。该功能是通过该客户端库处理多个任务公开的，其中包括连接管理、线程管理、故障处理和中间结果处理。MSQ 最多可以查询数百个分片。有关详细信息，请参阅[多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying/)。

一般而言，当使用弹性数据库的客户提交具有其自己的语义的局部分片操作（与跨分片操作相对）时，预期可获取完整的 T-SQL 功能。

## 后续步骤

尝试运行用于演示客户端功能的[示例应用](/documentation/articles/sql-database-elastic-scale-get-started/)。

若要安装该库，请转到[弹性数据库客户端库](http://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client)。

有关使用拆分/合并工具的说明，请参阅[拆分/合并工具概述](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)。

[弹性数据库客户端库的源代码现已发布！](https://azure.microsoft.com/blog/elastic-database-client-library-is-now-open-sourced)

使用[弹性查询](/documentation/articles/sql-database-elastic-query-overview/)。

[GitHub](https://github.com/Azure/elastic-db-tools) 以开源软件的形式提供该库。


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Anchors-->
<!--Image references-->
[1]: ./media/sql-database-elastic-database-client-library/glossary.png


<!---HONumber=Mooncake_0530_2016-->
