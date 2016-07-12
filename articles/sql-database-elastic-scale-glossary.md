<properties 
    pageTitle="弹性数据库工具词汇表 | Azure" 
    description="弹性数据库工具所用术语的解释" 
    services="sql-database" 
    documentationCenter="" 
    manager="jeffreyg" 
    authors="ddove" 
    editor=""/>

<tags 
    ms.service="sql-database" 
    ms.date="02/01/2016" 
    wacn.date="06/14/2016"/>

# 弹性数据库工具词汇表
Azure SQL 数据库的[弹性数据库工具](/documentation/articles/sql-database-elastic-scale-introduction/)的术语定义如下。这些工具用于管理[分片映射](/documentation/articles/sql-database-elastic-scale-shard-map-management/)，包括[客户端库](/documentation/articles/sql-database-elastic-database-client-library/)、[拆分/合并工具](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)、[弹性池](/documentation/articles/sql-database-elastic-pool/)和[查询](/documentation/articles/sql-database-elastic-query-overview/)。

这些术语用于[使用弹性数据库工具添加分片](/documentation/articles/sql-database-elastic-scale-add-a-shard/)和[使用 RecoveryManager 类解决分片映射问题](/documentation/articles/sql-database-elastic-database-recovery-manager/)。

![灵活扩展术语][1]

**数据库**：Azure SQL 数据库。

**数据相关的路由**：使应用程序能够连接到给定了特定分片键的分片的功能。请参阅[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)。与**[多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying/)**进行比较。

**全局分片映射**：**分片集**内分片键及其各自分片之间的映射。全局分片映射存储在**分片映射管理器**中。与**局部分片映射**进行比较。

**列表分片映射**：在其中单独映射分片键的分片映射。与**范围分片映射**进行比较。

**局部分片映射**：局部分片映射（存储在分片上）包含驻留在该分片上的 shardlet 的映射。

**多分片查询**：能够针对多个分片发出查询；使用 UNION ALL 语义（也称为“扇出查询”）返回结果集。与**数据相关的路由**进行比较。

**范围分片映射**：分片分发策略在其中基于多个连续值范围的分片映射。

**引用表**：未进行分片，但在分片间进行复制的表。例如，可以在引用表中存储邮政编码。

**分片**：用于存储分片数据集中的数据的 Azure SQL 数据库。

**分片弹性**：执行**横向缩放**和**纵向缩放**的能力。

**分片表**：已进行分片的表，即在基于其分片键值的分片中分发其数据。

**分片键**：确定如何在分片上分发数据的列值。值类型可以是下列其中一项：**int**、**bigint**、**varbinary** 或 **uniqueidentifier**。

**分片集**：属于分片映射管理器中相同分片映射的分片集合。

**Shardlet**：与分片上分片键的单个值相关联的所有数据。当重新分发分片表时，shardlet 是可能的数据移动的最小单元。

**分片映射**：分片键及其各自分片之间的映射集。

**分片映射管理器**：包含分片映射、分片位置和一个或多个分片集的映射的管理对象和数据存储。

![映射][2]


##动词

**横向缩放**：通过将分片添加到分片映射或删除分片，向外（或向内）扩展分片集合的行为。

![横向缩放与纵向缩放][3]

**合并**：将 shardlet 从两个分片移动到一个分片，并且相应地更新分片映射的行为。

**Shardlet 移动**：将单个 shardlet 移动到不同分片的行为。

**分片**：基于分片键对多个数据库上的结构相同的数据进行水平分区的行为。

**拆分**：将几个 shardlet 从一个分片移动到另一个（通常是新的）分片的行为。由用户提供的作为拆分点的分片键。

**纵向缩放**：向上（或向下）缩放单个分片的性能水平的行为。例如，将分片从标准版更改为高级版（这会导致需要更多的计算资源）。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-glossary/glossary.png
[2]: ./media/sql-database-elastic-scale-glossary/mappings.png
[3]: ./media/sql-database-elastic-scale-glossary/h_versus_vert.png
 
<!---HONumber=Mooncake_0606_2016-->
