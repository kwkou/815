<properties
   pageTitle="将解决方案迁移到 SQL 数据仓库 | Azure"
   description="有关将解决方案转移到 Azure SQL 数据仓库平台的迁移指南。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/14/2016"
   wacn.date="04/11/2016"/>

# 将解决方案迁移到 SQL 数据仓库

SQL 数据仓库是一种分布式数据库系统，可根据你的需要弹性缩放。为了保持性能与缩放性，SQL 数据仓库中未实现 SQL Server 的所有功能。以下迁移主题大致介绍了将解决方案迁移到 SQL 数据仓库时考虑的一些重要因素。由于针对缩放性设计数据仓库引入了不同的设计模式，因此传统的方法不一定最合适。你可能发现需要调整解决方案，以确保充分利用 SQL 数据仓库所提供的分布式平台。

此外必须记住，SQL 数据仓库是基于 Azure 的平台。因此，迁移过程很有可能需将数据传输到云中。数据传输本身是一个话题，应予以谨慎考虑，尤其是随着卷增大时。此外，不应将数据传输与数据加载相混淆，后者又是另外一个主题。

## 迁移指导
在开始迁移之前，请务必通读这些文章，以确保了解一些产品差异和基本概念。

- [迁移架构][]
- [迁移数据][]
- [迁移代码][]
 
## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

你还可以查看 [Transact-SQL 参考][]，以获取更多的信息。

最后，请查看[加载概述][]，其中介绍了各种数据加载选项并提供了分步指导。

<!--Image references-->

<!--Article references-->
[迁移架构]: /documentation/articles/sql-data-warehouse-migrate-schema/
[迁移数据]: /documentation/articles/sql-data-warehouse-migrate-data/
[迁移代码]: /documentation/articles/sql-data-warehouse-migrate-code/

[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/
[加载概述]: /documentation/articles/sql-data-warehouse-overview-load/
[Transact-SQL 参考]: /documentation/articles/sql-data-warehouse-overview-migrate/

<!--MSDN references-->


<!--Other Web references-->

<!---HONumber=Mooncake_0307_2016-->