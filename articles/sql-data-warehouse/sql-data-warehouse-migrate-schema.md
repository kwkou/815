<properties
   pageTitle="将架构迁移到 SQL 数据仓库 | Azure"
   description="有关在开发解决方案时将架构迁移到 Azure SQL 数据仓库的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="10/31/2016"
   wacn.date="12/19/2016"
   ms.author="jrj;barbkess;sonyama"/>  


# 将架构迁移到 SQL 数据仓库#
以下摘要有助于理解 SQL Server 与 SQL 数据仓库之间的差异，方便用户迁移数据库。

## 表迁移

在迁移表时，需熟悉 SQL 数据仓库表的表功能。[表概述][]是一个不错的起点。本文介绍了在创建表（例如表统计信息、分布、分区和索引）时需要考虑的最重要事项。同时还介绍了一些[不受支持的表功能][]和解决方法。

SQL 数据仓库支持常见的业务数据类型。请参阅[数据类型][]一文，了解一系列受支持的和[不受支持的数据类型][]。[数据类型][]一文还包含一个查询，用于标识[不受支持的数据类型][]。转换数据类型时，请确保参阅[数据类型最佳实践][]。

## 后续步骤
成功将数据库架构迁移到 SQL 数据仓库后，请继续阅读下列文章之一：

- [迁移数据][]
- [迁移代码][]

有关 SQL 数据仓库最佳实践的详细信息，请参阅[最佳实践][]一文。

<!--Image references-->


<!--Article references-->
[迁移代码]: /documentation/articles/sql-data-warehouse-migrate-code/
[迁移数据]: /documentation/articles/sql-data-warehouse-migrate-data/
[最佳实践]: /documentation/articles/sql-data-warehouse-best-practices/
[development overview]: /documentation/articles/sql-data-warehouse-overview-develop/
[不受支持的表功能]: /documentation/articles/sql-data-warehouse-tables-overview/
[数据类型]: /documentation/articles/sql-data-warehouse-tables-data-types/
[不受支持的数据类型]: /documentation/articles/sql-data-warehouse-tables-data-types/
[数据类型最佳实践]: /documentation/articles/sql-data-warehouse-tables-data-types/

<!--MSDN references-->


<!--Other Web references-->

<!---HONumber=Mooncake_1212_2016-->