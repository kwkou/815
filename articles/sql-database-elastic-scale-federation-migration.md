<properties 
    pageTitle="联合迁移 | Azure" 
    description="概述了将使用“联合”功能构建的现有应用迁移到弹性数据库模型的步骤。" 
    services="sql-database" 
    documentationCenter="" 
    manager="jeffreyg" 
    authors="ddove" 
    editor=""/>

<tags 
    ms.service="sql-database"
    ms.date="11/04/2015" 
    wacn.date="01/05/2016"/>

# 联合迁移 

Azure SQL 数据库联合功能以及 Web/Business Edition 将于 2015 年 9 月停用。到那时，使用联合功能的应用程序将停止工作。为了确保迁移成功，强烈建议您尽快开始迁移工作，以便充分计划和执行。

如果你的应用程序尚未做好失去联合功能的准备，请使用[此处](https://support.microsoft.com/zh-cn/kb/3087180)提到的说明联系 Microsoft 支持部门。

本文档提供了联合迁移实用工具的上下文、示例和介绍，说明了如何成功地将当前联合应用程序无缝迁移到用于分片的[弹性数据库客户端库](http://go.microsoft.com/?linkid=9862592) API。该文档的目标是引导您完成建议的步骤来迁移联合应用程序，而无需任何数据移动。

将现有联合应用程序迁移到使用弹性数据库工具的应用程序需要三个主要步骤。

1. [从联合根创建分片映射管理器](#create-a-shard-map-manager-from-a-federation-root) 
2. [修改现有应用程序](#modify-the-existing-application)
3. [断开现有联合成员](#switch-out-existing-federation-members)
    

### 迁移示例工具
为协助此过程，我们已创建[联合迁移实用工具](http://go.microsoft.com/?linkid=9862613)。该工具将完成步骤 1 和 3。

## 从联合根创建分片映射管理器
迁移联合应用程序的第一步是将联合根的元数据克隆到分片映射管理器的结构中。

![将联合根克隆到分片映射管理器][1]
 
在测试环境中，从现有的联合应用程序开始。
 
使用**联合迁移实用工具**将联合根元数据克隆到弹性数据库客户端库的[分片映射管理器](/documentation/articles/sql-database-elastic-scale-shard-map-management/)的结构中。与联合根类似，分片映射管理器数据库是独立数据库，它包含了分片映射（即联合）、对分片的引用（即联合成员）以及各自的范围映射。

从联合根到分片映射管理器的克隆是元数据的复制和转换。不会在联合根上更改任何元数据。请注意，使用联合迁移实用工具克隆联合根是一项时间点操作，并且对联合根或分片映射进行的任何更改将不会反映在其他相应的数据存储中。如果在测试新 API 期间对联合根进行了更改，则联合迁移实用工具可用于刷新分片映射以表示当前状态。

![迁移现有应用以使用弹性数据库工具 API][2]

## 修改现有应用程序 

在分片映射管理器到位且已使用分片映射管理器注册联合成员和范围（通过迁移实用工具完成）后，开发人员可以修改现有联合应用程序以使用弹性数据库客户端库。如上图所示，通过这些 API 的应用程序连接将通过分片映射管理器路由到相应的联合成员（现在也是分片）。将联合成员映射到分片映射管理器使一个应用程序的两个版本（一个使用联合，另一个使用弹性数据库客户端库）能够并行执行以验证功能。

在迁移应用程序期间，将需要对现有应用程序进行两项核心修改。


#### 变化 1：实例化分片映射管理器对象： 

与联合不同，弹性数据库工具 API 通过 **ShardMapManager** 类与分片映射管理器进行交互。**ShardMapManager** 对象和分片映射的实例可按照如下所示来完成：
     
    //Instantiate ShardMapManger Object 
    ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(
                            connectionStringSMM, ShardMapManagerLoadPolicy.Lazy); 
    RangeShardMap<T> rangeShardMap = shardMapManager.GetRangeShardMap<T>(shardMapName) 
    
#### 变化 2：将连接路由到相应的分片 

对于联合，将使用 USE FEDERATION 命令建立与特定联合成员的连接，如下所示：

    USE FEDERATION CustomerFederation(cid=100) WITH RESET, FILTERING=OFF`

对于弹性数据库工具 API，将使用 **RangeShardMap** 类中的 **OpenConnectionForKey** 方法，通过[数据相关路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)建立与特定分片的连接。

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

本部分中的这些步骤是必要的，但是可能不适用于出现的所有迁移方案。有关详细信息，请参阅[弹性数据库工具的概念性概述](/documentation/articles/sql-database-elastic-scale-introduction/)和 [API 参考](http://go.microsoft.com/?linkid=9862604)。

## 断开现有联合成员 

![断开分片的联合成员][3]

将应用程序修改为包含弹性数据库工具 API 后，迁移联合应用程序的最后一步是 **SWITCH OUT** 联合成员（有关详细信息，请参阅 [ALTER FEDERATION（Azure SQL 数据库）] 的 MSDN 参考(http://msdn.microsoft.com/zh-cn/library/dn269988(v=sql.120).aspx))。针对特定联合成员发出 **SWITCH OUT** 的最终结果将导致删除所有联合约束和元数据，从而将联合成员呈现为常规 Azure SQL 数据库，与其他任何 Azure SQL 数据库无异。

请注意，针对联合成员发出 **SWITCH OUT** 是一个单向操作，无法撤消。执行后，无法将产生的数据库添加回联合，并且 USE FEDERATION 命令将不再适用于此数据库。

为了执行该切换，其他参数已添加到 ALTER FEDERATION 命令，以 SWITCH OUT 联合成员。虽然可对单个联合成员发出该命令，但是联合迁移实用工具提供了以编程方式遍历每个联合成员并执行切换操作的功能。

对所有现有联合成员执行切换后，便完成了应用程序的迁移。断开所有联合成员后，建议在根位置执行 DROP FEDERATION 操作。这不会影响以前的任何成员，但允许根数据库轻松地迁出 Web 和 Business Edition。
  
联合迁移实用工具提供了以下功能：

1.    执行联合根到分片映射管理器的克隆。开发人员可选择将现有的分片映射管理器放置在新 Azure SQL 数据库上（推荐）或放置在现有联合根数据库上。
2.    针对联合中的所有联合成员发出 SWITCH OUT。


## 功能比较

虽然弹性数据库工具提供了许多附加功能（例如，[多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying/)、[拆分和合并分片](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)、分片弹性、[客户端缓存](/documentation/articles/sql-database-elastic-scale-shard-map-management/)等），但是有几个在弹性数据库工具中不受支持的联合功能值得注意。
  
- **FILTERING=ON** 的使用。建立使用行级安全性 (RLS) 来执行行筛选。与联合中的筛选一样，RLS 会自动将一个谓词添加到对分片表执行的所有查询。有关详细信息，请参阅[具有弹性数据库工具和行级安全性的多租户应用程序](/documentation/articles/sql-database-elastic-tools-multi-tenant-row-level-security/)。 
 
 或者，你可以在针对分片发出的查询中生成筛选逻辑。例如：

        --Example of USE FEDERATION with FILTERING=ON
        USE FEDERATION CustomerFederation(cid=100) WITH RESET, FILTERING=ON 
        SELECT * FROM customer

 产生的结果如下：

        --Example of USE FEDERATION with filtering in the WHERE clause 
        USE FEDERATION CustomerFederation(cid=100) WITH RESET, FILTERING=OFF 
        SELECT * FROM customer WHERE CustomerId = 100 

- 弹性数据库工具**拆分**功能并不完全处于联机状态。在进行拆分操作期间，必须使每个单独的 shardlet 在移动期间处于脱机状态。
- 拆分功能要求手动预配数据库和手动管理架构。

## 其他注意事项

* Web 版和 Business Edition 以及联合功能将于 2015 年秋季停用。在联合应用程序迁移过程中，强烈建议对 Basic、Standard 和 Premium Edition 执行性能测试。 

* 对联合成员执行 SWITCH OUT 语句将使生成的数据库能够利用所有 Azure SQL 数据库功能（即新版本、备份、PITR、审核等）。

##迁移需要更多时间吗？ 
如果你的应用程序尚未做好失去联合功能的准备，请使用[此处](https://support.microsoft.com/zh-cn/kb/3087180)提到的说明联系 Microsoft 支持部门。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Anchors-->
[Create Shard Map Manager from a Federation Root]: #create-shard-map-manager
[Modify the Existing Application]: #Modify-the-Existing-Application
[Switch Out Existing Federation Members]: #Switch-Out-Existing-Federation-Members


<!--Image references-->
[1]: ./media/sql-database-elastic-scale-federation-migration/migrate-1.png
[2]: ./media/sql-database-elastic-scale-federation-migration/migrate-2.png
[3]: ./media/sql-database-elastic-scale-federation-migration/migrate-3.png
 

<!---HONumber=Mooncake_1221_2015-->
