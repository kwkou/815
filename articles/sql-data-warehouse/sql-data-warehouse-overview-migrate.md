<properties
   pageTitle="将解决方案迁移到 SQL 数据仓库 | Azure"
   description="有关将解决方案转移到 Azure SQL 数据仓库平台的迁移指南。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="barbkess"
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
   ms.author="barbkess;jrj;sonyama"/>  


# 将解决方案迁移到 SQL 数据仓库
SQL 数据仓库是一种分布式数据库系统，可根据你的需要弹性缩放。为了保持性能与缩放性，SQL 数据仓库中未实现 SQL Server 的所有功能。以下迁移主题大致介绍了将解决方案迁移到 SQL 数据仓库时考虑的一些重要因素。由于针对缩放性设计数据仓库引入了不同的设计模式，因此传统的方法不一定最合适。因此，用户可能会发现，调整现有解决方案可确保充分利用 SQL 数据仓库所提供的分布式平台。

此外必须记住，SQL 数据仓库是基于 Azure 的平台。因此，迁移过程很有可能需将数据传输到云中。数据传输本身是一个话题，应予以谨慎考虑，尤其是随着卷增大时。数据传输和数据加载是不同的主题。

## 迁移指导
在开始迁移之前，请务必通读这些文章，确保了解一些产品差异和基本概念。

- [迁移架构][]
- [迁移数据][]
- [迁移代码][]

## 后续步骤

CAT（客户顾问团队）也有一些很好的通过博客发布的 SQL 数据仓库指南。请参阅他们的 [Migrating data to Azure SQL Data Warehouse in practice][]（在实践中将数据迁移到 Azure SQL 数据仓库）一文，了解有关迁移的更多指南。

<!--Image references-->


<!--Article references-->
[迁移架构]: /documentation/articles/sql-data-warehouse-migrate-schema/
[迁移数据]: /documentation/articles/sql-data-warehouse-migrate-data/
[迁移代码]: /documentation/articles/sql-data-warehouse-migrate-code/


<!--MSDN references-->


<!--Other Web references-->

[Migrating data to Azure SQL Data Warehouse in practice]: https://blogs.msdn.microsoft.com/sqlcat/2016/08/18/migrating-data-to-azure-sql-data-warehouse-in-practice/

<!---HONumber=Mooncake_1212_2016-->