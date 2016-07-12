<properties 
	pageTitle="分片映射管理 | Azure" 
	description="如何使用弹性数据库客户端库 ShardMapManager" 
	services="sql-database" 
	documentationCenter="" 
	manager="jeffreyg" 
	authors="ddove" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="02/05/2016" 
	wacn.date="06/14/2016"/>

# 分片映射管理

在分片数据库环境中，[**分片映射**](/documentation/articles/sql-database-elastic-scale-glossary/)将维护相关信息，以便应用程序可以根据**分片键**的值连接到相应的数据库。了解如何构建这些映射对于分片映射管理至关重要。对于 Azure SQL 数据库，请使用[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library/)的 [ShardMapManager 类](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.aspx)来管理分片映射。
 

## 分片映射
 
### 支持的分片键的 .Net 类型

灵活扩展支持将以下 .Net Framework 类型用作分片键：

* integer
* long
* guid
* 字节  
* datetime
* timespan
* datetimeoffset

### 列表和范围分片映射
使用**各个分片键值的列表**或**分片键值的范围**可构造分片映射。

###列表分片映射
**分片**包含 **shardlet**，shardlet 到分片的映射由分片映射维护。**列表分片映射**是可标识 shardlet 的单独键值和可用作分片的数据库之间的关联项。“列表映射”是可以映射到同一个数据库的显式且不同的键值。例如，键 1 映射到数据库 A，键值 3 和 6 都引用数据库 B。

| 键 | 分片位置 |
|-----|----------------|
| 1 | Database\_A |
| 3 | Database\_B |
| 4 | Database\_C |
| 6 | Database\_B |
| ... | ... |
 

### 范围分片映射 
在**范围分片映射**中，键范围由 **[Low Value, High Value)** 对描述，其中 *Low Value* 是范围中的最小键，而 *High Value* 是第一个大于范围的值。

例如，**[0, 100)** 包括所有大于或等于 0 且小于 100 的整数。请注意，多个范围可指向同一数据库，并且支持多个不连续的范围（例如，在下面的示例中，[100,200) 和 [400,600) 可同时指向数据库 C。）

| 键 | 分片位置 |
|-----------|----------------|
| [1,50) | Database\_A |
| [50,100) | Database\_B |
| [100,200) | Database\_C |
| [400,600) | Database\_C |
| ... | ...            

上面所示的每个表都是 **ShardMap** 对象的概念性示例。每一行都是单个 **PointMapping**（适用于列表分片映射）对象或 **RangeMapping**（适用于范围分片映射）对象的简化示例。

## 分片映射管理器 

在客户端库中，分片映射管理器是分片映射的集合。由 ShardMapManager 实例管理的数据保存在以下三个位置中：

1. 全局分片映射 (GSM)：可为其所有的分片映射指定某个数据库以用作存储库。将自动创建特殊的表和存储过程以管理信息。这通常是小型数据库且可轻松进行访问，但不应用于满足应用程序的其他需求。这些表位于名为 **\_\_ShardManagement** 的特殊架构中。

2. 局部分片映射 (LSM)：修改指定为分片的每个数据库，以包含多个小表和特殊存储过程，其中包括特定于该分片的分片映射信息并对其进行管理。对于 GSM 中的信息而言，该信息是冗余的，但应用程序通过该信息可验证缓存的分片映射信息，而无需将所有负载置于 GSM 上；应用程序可使用 LSM 确定缓存的映射是否仍然有效。与每个分片上的 LSM 对应的表也位于架构 **\_\_ShardManagement** 中。

3. **应用程序缓存**：每个用于访问 **ShardMapManager** 对象的应用程序实例都可维护其映射的本地内存中缓存。它存储最近检索到的路由信息。

## 构造 ShardMapManager

ShardMapManager 对象使用[工厂](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.aspx)模式构造的。通过 [ShardMapManagerFactory.GetSqlShardMapManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager.aspx) 方法可获取具有 ConnectionString 形式的凭据（包括用于保存 GSM 的服务器名称和数据库名称），并返回 ShardMapManager 的实例。

在应用程序的初始化代码内，每个应用域只应实例化 **ShardMapManager** 一次。**ShardMapManager** 可包含任意数量的分片映射。尽管对于许多应用程序而言，单个分片映射可能是足够的，但有时针对不同的架构或出于特定目的，需使用不同的数据库集，在这些情况下多个分片可能更合适。

在此代码中，应用程序尝试使用 [TryGetSqlShardMapManager 方法](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager.aspx)打开现有的 ShardMapManager。如果表示全局 ShardMapManager (GSM) 的对象尚未存在于数据库内，则客户端库将在此处使用 [CreateSqlShardMapManager 方法](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.createsqlshardmapmanager.aspx)创建这些对象。

    // Try to get a reference to the Shard Map Manager 
 	// via the Shard Map Manager database.  
    // If it doesn't already exist, then create it. 
    ShardMapManager shardMapManager; 
    bool shardMapManagerExists = ShardMapManagerFactory.TryGetSqlShardMapManager(
                                        connectionString, 
                                        ShardMapManagerLoadPolicy.Lazy, 
                                        out shardMapManager); 

    if (shardMapManagerExists) 
     { 
        Console.WriteLine("Shard Map Manager already exists");
    } 
    else
    {
        // Create the Shard Map Manager. 
        ShardMapManagerFactory.CreateSqlShardMapManager(connectionString);
        Console.WriteLine("Created SqlShardMapManager"); 

        shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(
            connectionString, 
            ShardMapManagerLoadPolicy.Lazy);

        // The connectionString contains server name, database name, and admin credentials 
        // for privileges on both the GSM and the shards themselves.
    } 
 
你可以使用 Powershell 作为替代方法来创建新的分片映射管理器。[此处](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db)提供了一个示例。

### 分片映射管理凭据

通常，用于管理和操作分片映射的应用程序不同于那些使用分片映射路由连接的应用程序。

对于管理分片映射（添加或更改分片、分片映射等）的应用程序，你必须使用“在 GSM 数据库和用作分片的每个数据库上都具有读/写权限的凭据”实例化 **ShardMapManager**。在输入或更改分片映射信息时，这些凭据必须允许编写 GSM 和 LSM 中的表，以及在新分片上创建 LSM 表。

### 仅元数据受影响 

用于填充或更改 **ShardMapManager** 数据的方法不会更改存储在分片本身中的用户数据。例如，类似于 **CreateShard**、**DeleteShard**、**UpdateMapping** 等的方法仅影响分片映射元数据，而不会删除、添加或更改分片中所包含的用户数据。但是，这些方法旨在与您执行的单独操作结合使用，以创建或删除实际数据库，或者将行从一个分片移动到另一个分片，以使分片环境恢复均衡。（弹性数据库工具附带的**拆分-合并**工具将使用这些 API 并安排在分片之间移动实际数据。） 请参阅[使用弹性数据库拆分/合并工具进行缩放](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)。

## 填充分片映射示例
 
下面显示了用于填充特定分片映射的操作的示例序列。此代码将执行下列步骤：

1. 将在分片映射管理器中创建新的分片映射。 
2. 将两个不同分片的元数据添加到分片映射。 
3. 将添加各种键范围映射，并且将显示分片映射的整体内容。 

编写的代码可在发生错误时重新运行该方法。每个请求将测试是否存在某个分片或映射，然后尝试创建该分片或映射。该代码假设已在由字符串 shardServer 引用的服务器中创建名为 sample\_shard\_0、sample\_shard\_1 和 sample\_shard\_2 的数据库。

    public void CreatePopulatedRangeMap(ShardMapManager smm, string mapName) 
        {            
            RangeShardMap<long> sm = null; 

            // check if shardmap exists and if not, create it 
            if (!smm.TryGetRangeShardMap(mapName, out sm)) 
            { 
                sm = smm.CreateRangeShardMap<long>(mapName); 
            } 

            Shard shard0 = null, shard1=null; 
            // Check if shard exists and if not, 
            // create it (Idempotent / tolerant of re-execute) 
            if (!sm.TryGetShard(new ShardLocation(
	                                 shardServer, 
	                                 "sample_shard_0"), 
	                                 out shard0)) 
            { 
                Shard0 = sm.CreateShard(new ShardLocation(
	                                        shardServer, 
	                                        "sample_shard_0")); 
            } 

            if (!sm.TryGetShard(new ShardLocation(
	                                shardServer, 
	                                "sample_shard_1"), 
	                                out shard1)) 
            { 
                Shard1 = sm.CreateShard(new ShardLocation(
	                                         shardServer, 
	                                        "sample_shard_1"));  
            } 

            RangeMapping<long> rmpg=null; 

            // Check if mapping exists and if not,
            // create it (Idempotent / tolerant of re-execute) 
            if (!sm.TryGetMappingForKey(0, out rmpg)) 
            { 
                sm.CreateRangeMapping(
	                      new RangeMappingCreationInfo<long>
	                      (new Range<long>(0, 50), 
	                      shard0, 
	                      MappingStatus.Online)); 
            } 

            if (!sm.TryGetMappingForKey(50, out rmpg)) 
            { 
                sm.CreateRangeMapping(
	                     new RangeMappingCreationInfo<long> 
                         (new Range<long>(50, 100), 
	                     shard1, 
	                     MappingStatus.Online)); 
            } 

            if (!sm.TryGetMappingForKey(100, out rmpg)) 
            { 
                sm.CreateRangeMapping(
	                     new RangeMappingCreationInfo<long>
                         (new Range<long>(100, 150), 
	                     shard0, 
	                     MappingStatus.Online)); 
            } 

            if (!sm.TryGetMappingForKey(150, out rmpg)) 
            { 
                sm.CreateRangeMapping(
	                     new RangeMappingCreationInfo<long> 
                         (new Range<long>(150, 200), 
	                     shard1, 
	                     MappingStatus.Online)); 
            } 

            if (!sm.TryGetMappingForKey(200, out rmpg)) 
            { 
               sm.CreateRangeMapping(
	                     new RangeMappingCreationInfo<long> 
                         (new Range<long>(200, 300), 
	                     shard0, 
	                     MappingStatus.Online)); 
            } 

            // List the shards and mappings 
            foreach (Shard s in sm.GetShards()
                         .OrderBy(s => s.Location.DataSource)
                         .ThenBy(s => s.Location.Database))
            { 
               Console.WriteLine("shard: "+ s.Location); 
            } 

            foreach (RangeMapping<long> rm in sm.GetMappings()) 
            { 
                Console.WriteLine("range: [" + rm.Value.Low.ToString() + ":" 
                        + rm.Value.High.ToString()+ ")  ==>" +rm.Shard.Location); 
            } 
        } 
 
使用 PowerShell 脚本也可达到相同的结果。[此处](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db)提供了某些 PowerShell 示例。

填充完分片映射后，即可创建或改编数据访问应用程序，以便使用这些映射。在需要更改**映射布局**之前，无需重新填充或操作映射。

## 依赖于数据的路由 

分片映射管理器主要由需要数据库连接的应用程序用来执行特定于应用的数据操作。这些连接必须与正确的数据库关联。这称为**数据相关的路由**。对于这些应用程序，通过使用在 GSM 数据库上具有只读访问权限的凭据，实例化来自工厂的分片映射管理器对象。以后，单独的连接请求将提供连接相应分片数据库时所需的凭据。

请注意，这些应用程序（使用通过具有只读权限的凭据打开的 ShardMapManager）将无法更改映射。为了满足这些需求，请创建特定于管理的应用程序或 PowerShell 脚本，以提供如前所述的更高级别权限的凭据。请参阅[用于访问弹性数据库客户端库的凭据](/documentation/articles/sql-database-elastic-scale-manage-credentials/)。

有关详细信息，请参阅[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)。

## 修改分片映射 

可采用不同方式更改分片映射。以下所有方法都可修改用于描述分片及其映射的元数据，但这些方法不以物理方式修改分片内的数据，也不创建或删除实际数据库。下面所述的分片映射上的某些操作可能需要与以物理方式移动数据或添加和删除用作分片的数据库的管理操作进行协调。

这些方法作为构建基块一同工作，以便在分片的数据库环境中修改数据的总体分发情况。

* 若要添加或删除分片：请使用 [Shardmap 类](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.aspx)的 **[CreateShard](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.createshard.aspx)** 和 **[DeleteShard](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.deleteshard.aspx)**。 
    
    若要执行这些操作，表示目标分片的服务器和数据库必须已经存在。这些方法不会对数据库本身产生任何影响，仅对分片映射上的元数据产生影响。

* 若要创建或删除映射到分片的点或范围：请使用 [RangeShardMapping 类](https://msdn.microsoft.com/zh-cn/library/azure/dn807318.aspx)的 **[CreateRangeMapping](https://msdn.microsoft.com/zh-cn/library/azure/dn841993.aspx)** 和 **[DeleteMapping](https://msdn.microsoft.com/zh-cn/library/azure/dn824200.aspx)**，以及 [ListShardMap](https://msdn.microsoft.com/zh-cn/library/azure/dn842123.aspx) 的 **[CreatePointMapping](https://msdn.microsoft.com/zh-cn/library/azure/dn807218.aspx)**。
    
    许多不同的点或范围可映射到相同的分片。这些方法仅影响元数据，而不会影响已显示在分片中的任何数据。如果为了与 **DeleteMapping** 操作保持一致而需要将数据从数据库中删除，你将需要单独执行这些操作，但需要结合使用这些方法。

* 将现有的范围拆分为两个，或将相邻的范围合并为一个：请使用 **[SplitMapping](https://msdn.microsoft.com/zh-cn/library/azure/dn824205.aspx)** and **[MergeMappings](https://msdn.microsoft.com/zh-cn/library/azure/dn824201.aspx)**。

    请注意，拆分和合并操作**不更改键值要映射到的分片**。拆分操作可将现有的范围拆分为两个部分，但在映射到相同分片时同时保留这两个部分。对在已映射到相同分片的两个相邻的范围进行合并操作，从而可将其合并到单个范围中。若要在分片之间移动点或范围本身，需要将 **UpdateMapping** 与移动的实际数据结合使用，才能进行协调。当需要移动数据时，你可以使用弹性数据库工具中随附的**拆分/合并**服务，以将分片映射的更改与数据移动相协调。

* 将单独的点或范围重新映射（或移动）到不同的分片：请使用 [UpdateMapping](https://msdn.microsoft.com/zh-cn/library/azure/dn824207.aspx)。

    由于可能需要将数据从一个分片移动到另一个分片，以便与 **UpdateMapping** 操作保持一致，因此你将需要单独执行此移动，但需要结合使用这些方法。

* 若要在联机和脱机状态下执行映射：请使用 **[MarkMappingOffline](https://msdn.microsoft.com/zh-cn/library/azure/dn824202.aspx)** 和 **[MarkMappingOnline](https://msdn.microsoft.com/zh-cn/library/azure/dn807225.aspx)** 来控制映射的联机状态。

    仅当映射处于“脱机”状态时才允许在分片映射上进行某些操作，其中包括 **UpdateMapping** 和 **DeleteMapping**。当映射处于脱机状态时，基于该映射中所包含的键的数据依赖请求将返回一个错误。此外，当范围首次处于脱机状态时，所有到受影响的分片的连接都将自动终止，以防止因范围的更改而导致查询出现不一致或不完整的结果。

映射是 .Net 中的不可变对象。以上会更改映射的所有方法也会使代码中任何对映射的引用失效。为了更轻松地执行操作序列来更改映射的状态，所有会更改映射的方法都将返回新的映射引用，以便能够链接操作。例如，若要在 shardmap sm 中删除包含索引键 25 的现有映射，可以执行以下命令：

        sm.DeleteMapping(sm.MarkMappingOffline(sm.GetMappingForKey(25)));

## 添加分片 

对于已经存在的分片映射，应用程序通常仅需要添加新分片，以处理预期的新键或键范围数据。例如，由租户 ID 分片的应用程序可能需要为新的租户设置新分片，或者在每个新的月份开始之前，每月分片的数据可能需要设置新分片。

如果新的键值范围还不是现有映射的组成部分且无需移动数据，则添加新分片以及将新的键或范围关联到该分片非常简单。有关添加新分片的详细信息，请参阅[添加新分片](/documentation/articles/sql-database-elastic-scale-add-a-shard/)。

但是，在需要移动数据的情况下，需要拆分/合并工具并结合使用必要的分片映射更新，才能安排在分片之间移动数据。有关使用拆分/合并工具的详细信息，请参阅[拆分/合并概述](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]
 

<!---HONumber=Mooncake_0606_2016-->
