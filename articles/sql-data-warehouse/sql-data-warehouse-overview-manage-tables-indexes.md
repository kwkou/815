<properties
   pageTitle="管理 Azure SQL 数据仓库中的表和索引 | Azure"
   description="在 Azure SQL 数据仓库中管理表和索引时的注意事项、最佳实践和任务的概述"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/02/2016"
   wacn.date="06/20/2016"/>

# 管理 Azure SQL 数据仓库中的表和索引

在 SQL 数据仓库中管理表和索引时的注意事项、最佳实践和任务的概述。



| 类别 | 任务或注意事项 | 说明 |
| :-----------------------| :---------------------------------------------- | :----------- |
| 列存储索引 | 在加载或插入数据后重新生成索引 | 默认情况下，SQL 数据仓库将每个表存储为聚集列存储索引。这样可以提高性能和数据压缩效果，尤其是对大型表来说。根据你将数据引入列存储索引的方式，可能无法使用列存储压缩来存储所有数据。发生这种情况时，你可能无法获得列存储索引本应提供的性能。<br/><br/>若要确保列存储索引处于正常运行状态，请参阅[管理列存储索引][]。 |
| 哈希分布表 | 验证数据是否均匀地分布在节点中 | 哈希分布表基本上可以说是在大型表上对联接和聚合操作进行优化的最佳方式。SQL 数据仓库根据指定的分布列来分发表。某些列是很好的分布键，而某些列则不是。你希望行能够均匀地分布。一定程度的不均匀或者说数据偏斜是允许的，但如果数据偏斜程度过大，则 SQL 数据仓库设计提供的大规模并行处理 (MPP) 优势将会丧失殆尽。<br/><br/>若要确保哈希分布表没有过多的数据偏斜，请参阅[管理分布式数据偏斜][]。 |





## 后续步骤

有关更多管理提示，请转到[管理概述][]。

<!--Image references-->

<!--Article references-->
[管理列存储索引]: /documentation/articles/sql-data-warehouse-manage-columnstore-indexes/
[管理分布式数据偏斜]: /documentation/articles/sql-data-warehouse-manage-distributed-data-skew/
[管理概述]: /documentation/articles/sql-data-warehouse-overview-manage/

<!--MSDN references-->


<!--Other Web references-->
<!---HONumber=Mooncake_0613_2016-->