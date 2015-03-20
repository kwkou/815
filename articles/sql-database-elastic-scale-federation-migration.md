<properties title="Federations Migration" pageTitle="联合迁移。" description="概述了将使用“联合”功能构建的现有应用迁移到灵活扩展模型的步骤。" metaKeywords="sharding scaling, federations, Azure SQL DB sharding, Elastic Scale" services="sql-database" documentationCenter=""  manager="jhubbard" authors="sidneyh"/>

#联合迁移 

Azure SQL Database 联合功能以及 Web/企业版将于 2015 年 9 月停用。在那时，使用联合功能的应用程序将停止工作。为了确保迁移成功，强烈建议您尽快开始迁移工作，以便充分计划和执行。本文档提供了联合迁移实用工具的上下文、示例和介绍，说明了如何成功地将当前联合应用程序无缝迁移到 [Azure SQL DB 灵活扩展预览 API](/zh-cn/documentation/articles/sql-database-elastic-scale-introduction/)。该文档的目标是引导您完成建议的步骤来迁移联合应用程序，而无需移动任何数据。

将现有联合应用程序迁移到使用灵活扩展 API 的应用程序需要三个主要步骤。

1. [从联合根创建分片映射管理器]
2. [修改现有应用程序]
3. [断开现有联合成员]
    

### 迁移示例工具
为协助此过程，我们已创建[联合迁移实用工具](http://go.microsoft.com/?linkid=9862613)。该工具将完成步骤 1 和 3。 

## <a name="create-shard-map-manager"></a>从联合根创建分片映射管理器
迁移联合应用程序的第一步是将联合根的元数据克隆到分片映射管理器的结构中。 

![Clone the federation root to the shard map manager][1]
 
在测试环境中，从现有的联合应用程序开始。
 
使用**联合迁移实用工具**将联合根元数据克隆到灵活扩展[分片映射管理器]的结构中(/zh-cn/documentation/articles/sql-database-elastic-scale-shard-map-management/)。与联合根类似，分片映射管理器数据库是独立数据库，它包含了分片映射（即联合）、对分片的引用（即联合成员）以及各自的范围映射。 

从联合根到分片映射管理器的克隆是元数据的复制和转换。不会在联合根上更改任何元数据。请注意，使用联合迁移实用工具克隆联合根是一项时间点操作，并且对联合根或分片映射进行的任何更改将不会反映在其他相应的数据存储中。如果在测试新 API 期间对联合根进行了更改，则联合迁移实用工具可用于刷新分片映射以表示当前状态。 

![Migrate the existing app to use the Elastic Scale APIs][2]

## 修改现有应用程序 

在分片映射管理器到位且已使用分片映射管理器注册联合成员和范围（通过迁移实用工具完成）后，开发人员可以修改现有联合应用程序以使用灵活扩展 API。如上图所示，通过灵活扩展 API 的应用程序连接将通过分片映射管理器路由到相应的联合成员（现在也是分片）。将联合成员映射到分片映射管理器使一个应用程序的两个版本（一个使用联合，另一个使用灵活扩展）能够并行执行以验证功能。   

在迁移应用程序期间，将需要对现有应用程序进行两项核心修改。


#### 更改 1：实例化分片映射管理器对象： 

与联合不同，灵活扩展 API 通过 **ShardMapManager** 类与分片映射管理器进行交互。**ShardMapManager** 对象和分片映射的实例可按照如下所示来完成：
     
    //Instantiate ShardMapManger Object 
    ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(
                            connectionStringSMM, ShardMapManagerLoadPolicy.Lazy); 
    RangeShardMap<T> rangeShardMap = shardMapManager.GetRangeShardMap<T>(shardMapName) 
    
#### 更改 2：将连接路由到相应的分片 

对于联合，将使用 USE FEDERATION 命令建立与特定联合成员的连接，如下所示：  

    USE FEDERATION CustomerFederation(cid=100) WITH RESET, FILTERING=OFF`

借助灵活扩展 API，可在 **RangeShardMap** 类上使用 **OpenConnectionForKey** 方法，以通过[数据依赖路由](/zh-cn/documentation/articles/sql-database-elastic-scale-data-dependent-routing/) 建立与特定分片的连接。 

    //Connect and issue queries on the shard with key=100 
    using (SqlConnection conn = rangeShardMap.OpenConnectionForKey(100, csb))  
    { 
         using (SqlCommand cmd = new SqlCommand()) 
         { 
            cmd.Connection = conn; 
            cmd.CommandText = "SELECT * FROM customer";
     
            using (SqlDataReader dr = cmd.ExecuteReader()) 
            { 
                  //Perform action on dr 
            } 
        } 
    }

本部分中的这些步骤是必要的，但是可能不适用于出现的所有迁移方案。有关详细信息，请参阅[灵活扩展的概念性概述](/zh-cn/documentation/articles/sql-database-elastic-scale-introduction/) 和 [API 参考](http://msdn.microsoft.com/zh-cn/library/azure/dn765902.aspx)。

## 断开现有联合成员 

![Switch out the federation members for the shards][3]

将应用程序修改为包含灵活扩展 API 后，迁移联合应用程序的最后一步是 **SWITCH OUT** 联合成员（有关详细信息，请参阅 [ALTER FEDERATION (Azure SQL Database)](http://msdn.microsoft.com/zh-cn/library/dn269988\(v=sql.120\).aspx 的 MSDN 参考）。针对特定联合成员发出 **SWITCH OUT** 的最终结果将导致删除所有联合约束和元数据，从而将联合成员呈现为常规 Azure SQL Database，与其他任何 Azure SQL Database 无异。  

请注意，针对联合成员发出 **SWITCH OUT** 是一个单向操作，无法撤消。执行后，无法将产生的数据库添加回联合，并且 USE FEDERATION 命令将不再适用于此数据库。 

为了执行该切换，其他参数已添加到 ALTER FEDERATION 命令，以 SWITCH OUT 联合成员。虽然可对单个联合成员发出该命令，但是联合迁移实用工具提供了以编程方式遍历每个联合成员并执行切换操作的功能。 

对所有现有联合成员执行切换后，便完成了应用程序的迁移。  
联合迁移实用工具提供了以下功能： 

1.    执行联合根到分片映射管理器的克隆。开发人员可选择将现有的分片映射管理器放置在新 Azure SQL Database 上（推荐）或放置在现有联合根数据库上。
2.    针对联合中的所有联合成员发出 SWITCH OUT。


##功能比较  
尽管灵活扩展提供了许多其他功能（例如，[多分片查询](/zh-cn/documentation/articles/sql-database-elastic-scale-multishard-querying/)、[拆分和合并分片](/zh-cn/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)、[分片灵活性](/zh-cn/documentation/articles/sql-database-elastic-scale-elasticity/)、[客户端缓存](/zh-cn/documentation/articles/sql-database-elastic-scale-shard-map-management/)等），但需要关注几个联合功能，因为它们在灵活扩展中不受支持。
  

- **FILTERING=ON** 的使用。灵活扩展当前不支持行级筛选。一种解决方法是将筛选逻辑构建到针对分片发出的查询中，如下所示： 

        --Example of USE FEDERATION with FILTERING=ON
        USE FEDERATION CustomerFederation(cid=100) WITH RESET, FILTERING=ON 
        SELECT * FROM customer

Yields the same result as:

        --Example of USE FEDERATION with filtering in the WHERE clause 
        USE FEDERATION CustomerFederation(cid=100) WITH RESET, FILTERING=OFF 
        SELECT * FROM customer WHERE CustomerId = 100 

- 灵活扩展**拆分**功能并不完全处于联机状态。在进行拆分操作期间，必须使每个单独的 shardlet 在移动期间都处于脱机状态。
- 灵活扩展拆分功能要求手动设置数据库和手动管理架构。

## 其他注意事项

* Web 版和企业版以及联合功能将于 2015 年秋季停用。作为联合应用程序迁移的一部分，强烈建议对基础版、标准版和高级版执行性能测试。 

* 对联合成员执行 SWITCH OUT 语句将使生成的数据库能够利用所有 Azure SQL Database 功能（即新版本、备份、PITR、审核等）。 

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Anchors-->
[从联合根创建分片映射管理器]:#create-shard-map-manager
[修改现有应用程序]:#Modify-the-Existing-Application
[断开现有联合成员]:#Switch-Out-Existing-Federation-members


<!--Image references-->
[1]: ./media/sql-database-elastic-scale-federation-migration/migrate-1.png
[2]: ./media/sql-database-elastic-scale-federation-migration/migrate-2.png
[3]: ./media/sql-database-elastic-scale-federation-migration/migrate-3.png
