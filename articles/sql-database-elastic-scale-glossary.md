<properties title="Azure Elastic Scale Glossary" pageTitle="Azure 灵活扩展词汇表" description="用于 Azure SQL Database 的灵活扩展功能的术语解释" metaKeywords="sharding,elastic scale, Azure SQL DB sharding" services="sql-database" documentationCenter="" manager="jhubbard" authors="sidneyh@microsoft.com"/>

#灵活扩展词汇表
为 Azure SQL Database 的灵活扩展功能定义了以下术语。

![Elastic Scale terms][1]

**数据库**：Azure SQL Database。 

**数据依赖路由**：使应用程序能够连接到给定了特定分片键的分片的功能。与**多分片查询**进行比较。

**全局分片映射**：**分片集**内分片键及其各自分片之间的映射组。GSM 存储在**分片映射管理器**中。与**局部分片映射**进行比较。

**列表分片映射**：在其中单独映射分片键的分片映射。与**范围分片映射**进行比较。   

**局部分片映射**：局部分片映射（存储在分片上）包含驻留在该分片上的 shardlet 的映射。


**多分片查询**：能够针对多个分片发出查询；使用 UNION ALL 语义（也称为"扇出查询"）返回结果集。与**数据依赖路由**进行比较。

**范围分片映射**：分片分发策略在其中基于多个连续值范围的分片映射。 


**引用表**：未进行分片，但在分片间进行复制的表。 

**分片**：用于存储分片数据集中的数据的 Azure SQL Database。 

**分片灵活性** (SE)：执行**水平扩展**和**垂直扩展**的功能。

**分片表**：已进行分片的表，即在基于其分片键值的分片中分发其数据。 

**分片键**：确定如何在分片上分发数据的列值。值类型可以是以下值之一：int、bigint、varbinary 或 uniqueidentifier。 

**分片集**：属于分片映射管理器中相同分片映射的分片集合。  

**Shardlet**：与分片上分片键的单个值相关联的所有数据。当重新分发分片表时，shardlet 是可能的数据移动的最小单元。 

**分片映射**：分片键及其各自分片之间的映射集。

**分片映射管理器**：包含分片映射、分片位置和一个或多个分片集的映射的管理对象和数据存储。

![Mappings][2]


##动词

**水平扩展**：通过将分片添加到分片映射或删除分片，向外（或向内）扩展分片集合的行为。

**合并**：将 shardlet 从两个分片移动到一个分片，并且相应地更新分片映射的行为。

**Shardlet 移动**：将单个 shardlet 移动到不同分片的行为。 

**分片**：基于分片键对多个数据库上的结构相同的数据进行水平分区的行为。

**拆分**：将几个 shardlet 从一个分片移动到另一个（通常是新的）分片的行为。由用户提供的作为拆分点的分片键。

**垂直扩展**：向上（或向下）扩展单个分片的性能水平的行为。例如，将分片从标准版更改为高级版（出于性能原因需要此更改）。 

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]  

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-glossary/glossary.png
[2]: ./media/sql-database-elastic-scale-glossary/mappings.png


