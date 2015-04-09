<properties title="Adding a Shard To an Elastic Scale Application" pageTitle="向弹性缩放应用程序添加分片" description="如何使用弹性缩放 API 将新分片添加到分片集。" metaKeywords="sharding scaling, Azure SQL Database sharding, elastic scale, shardmapmanager" services="sql-database" documentationCenter="" manager="jhubbard" authors="sidneyh@microsoft.com"/>

# 向弹性缩放应用程序添加分片 


### 添加新范围或键的分片  

对于已经存在的分片映射，应用程序通常仅需要添加新分片，以处理预期的新键或键范围数据。例如，由租户 ID 分片的应用程序可能需要为新的租户设置新分片，或者在每个新的月份开始之前，每月分片的数据可能需要设置新分片。 

如果新的键值范围还不是现有映射的组成部分，则添加新分片以及将新的键或范围关联到该分片非常简单。 

###示例：将分片及其范围添加到现有的分片映射
在以下示例中，创建了一个名为 **sample_shard_2** 的数据库以及其中所有必要的架构对象，用于保存范围 [300, 400)。  

    // sm is a RangeShardMap object.
    // Add a new shard to hold the range being added. 
    Shard shard2 = null; 

    if (!sm.TryGetShard(new ShardLocation(shardServer, "sample_shard_2"),out shard2)) 
    { 
        shard2 = sm.CreateShard(new ShardLocation(shardServer, "sample_shard_2"));  
    } 

    // Create the mapping and associate it with the new shard 
    sm.CreateRangeMapping(new RangeMappingCreationInfo<long> 
                            (new Range<long>(300, 400), shard2, MappingStatus.Online)); 


### 为现有范围的空部分添加分片  

在某些情况下，你可能已将某个范围映射到某个分片，并在该分片中填充了部分数据，但是，你现在希望将后续数据定向到其他分片。例如，你以前按日期范围分片，并且已在某个分片中分配了 50 天的数据，但从第 24 天开始，你希望以后的数据驻留在其他分片中。弹性缩放预览版中的[拆分-合并服务](/zh-cn/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/) 可以执行此操作，但是，如果不需要数据移动（例如，[25, 50) 天数范围 - 即，从第 25 天（含）到第 50 天（不含）- 的数据尚不存在），则你完全可以直接使用分片映射管理 API 执行此操作。

###示例：拆分范围并将空部分分配到新增的分片

已创建名为"sample_shard_2"的数据库以及其中所有必要的架构对象。  

 
    // sm is a RangeShardMap object.
    // Add a new shard to hold the range we will move 
    Shard shard2 = null; 

    if (!sm.TryGetShard(new ShardLocation(shardServer, "sample_shard_2"),out shard2)) 
    { 
    
        shard2 = sm.CreateShard(new ShardLocation(shardServer, "sample_shard_2"));  
    } 

    // Split the Range holding Key 25 

    sm.SplitMapping(sm.GetMappingForKey(25), 25); 

    // Map new range holding [25-50) to different shard: 
    // first take existing mapping offline 
    sm.MarkMappingOffline(sm.GetMappingForKey(25)); 
    // now map while offline to a different shard and take online 
    RangeMappingUpdate upd = new RangeMappingUpdate(); 
    upd.Shard = shard2; 
    sm.MarkMappingOnline(sm.UpdateMapping(sm.GetMappingForKey(25), upd)); 

**重要说明**：仅当你确定所更新映射的范围为空时，才使用此方法。上述方法不会在数据中检查要移动的范围，因此最好在代码中包含检查操作。如果要移动的范围中存在行，则实际数据分发将不匹配更新的分片映射。在这种情况下，请改用[拆分-合并服务](/zh-cn/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/) 来执行该操作。  


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]
