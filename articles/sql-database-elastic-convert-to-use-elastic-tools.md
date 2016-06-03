<properties
   pageTitle="转换现有数据库以使用弹性数据库工具"
   description="通过创建分片映射管理器来转换分片数据库，以使用弹性数据库工具"
   services="sql-database"
   documentationCenter=""
   authors="SilviaDoomra"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="04/01/2016"
   wacn.date="05/16/2016"/>

# 转换现有数据库以使用弹性数据库工具

如果你有现有的分片扩展解决方案，可以结合本文所述的技巧来利用弹性数据库工具（如[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library)和[拆分/合并工具](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge)）。

你可以使用 [.NET Framework 客户端库](http://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client)或者 [Azure SQL 数据库 - 弹性数据库工具脚本](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db)中提供的 PowerShell 脚本来实现这些技巧。以下示例使用 PowerShell 脚本。

请注意，必须先创建数据库，再运行 Add-Shard 和 New-ShardMapManager cmdlet。这些 cmdlet 不会创建数据库。

有四个步骤：

1. 准备分片映射管理器数据库。
2. 创建分片映射。
3. 准备各个分片。  
2. 将映射添加到分片映射。

有关 ShardMapManager 的详细信息，请参阅 [Shard map management（分片映射管理）](/documentation/articles/sql-database-elastic-scale-shard-map-management)。有关弹性数据库工具的概述，请参阅 [Elastic Database features overview（弹性数据库功能概述）](/documentation/articles/sql-database-elastic-scale-introduction)。

## 准备分片映射管理器数据库
可以使用新的或现有的数据库作为分片映射管理器。

## 步骤 1：创建分片映射管理器
请注意，用作分片映射管理器的数据库不应是与分片相同的数据库。

	# Create a shard map manager. 
	New-ShardMapManager -UserName '<user_name>' 
	-Password '<password>' 
	-SqlServerName '<server_name>' 
	-SqlDatabaseName '<smm_db_name>' 
	#<server_name> and <smm_db_name> are the server name and database name 
	# for the new or existing database that should be used for storing 
	# tenant-database mapping information.

### 检索分片映射管理器

创建后，可以使用此 cmdlet 检索分片映射管理器。每当需要使用 ShardMapManager 对象时，就需要执行此步骤。

	# Try to get a reference to the Shard Map Manager  
	$ShardMapManager = Get-ShardMapManager -UserName '<user_name>' 
	-Password '<password>' 
	-SqlServerName '<server_name>' 
	-SqlDatabaseName '<smm_db_name>' 

  
## 步骤 2：创建分片映射

选择创建以下模型之一：

1. 每个数据库一个租户 
2. 每个数据库多个租户（两种类型）：
	3. 范围映射
	4. 列表映射
 

如果你使用单租户数据库模型，请使用列表映射。单租户模型将每个租户分配一个数据库。这是适用于 SaaS 开发人员的有效模式，因为它可以简化管理。

![列表映射][1]

相比之下，多租户数据库模型将数个租户分配给单一数据库（你可以跨多个数据库分布租户组。）如果预期到每个租户的数据量很小，这是可行的模型。在此模型中，我们使用范围映射将某个范围的租户分配给数据库。
 

![范围映射][2]

你也可以使用列表映射来实现多租户数据库模型，以将多个租户分配给单一数据库。例如，DB1 用于存储租户 ID 1 和 5 的相关信息，而 DB2 用于存储租户 7 和租户 10 的数据。

![单一数据库上的多个租户][3]


## 步骤 2，选项 1：为列表映射创建分片映射
使用 ShardMapManager 对象创建分片映射。

	# $ShardMapManager is the shard map manager object. 
	$ShardMap = New-ListShardMap -KeyType $([int]) 
	-ListShardMapName 'ListShardMap' 
	-ShardMapManager $ShardMapManager 
 
 
## 步骤 2，选项 2：为范围映射创建分片映射

请注意，若要使用此映射模式，租户 ID 值需是连续范围，并且可接受范围中有间距，方法为只在创建数据库时跳过范围。

	# $ShardMapManager is the shard map manager object 
	# 'RangeShardMap' is the unique identifier for the range shard map.  
	$ShardMap = New-RangeShardMap 
	-KeyType $([int]) 
	-RangeShardMapName 'RangeShardMap' 
	-ShardMapManager $ShardMapManager 

## 步骤 2，选项 3：单一数据库上的列表映射
设置此模式也需要创建列表映射，如步骤 2，选项 1 中所示。

## 步骤 3：准备各个分片

将每个分片（数据库）添加到分片映射管理器。这将准备用于存储映射信息的各个数据库。对每个分片执行此方法。
	 
	Add-Shard 
	-ShardMap $ShardMap 
	-SqlServerName '<shard_server_name>' 
	-SqlDatabaseName '<shard_database_name>'
	# The $ShardMap is the shard map created in step 2.
 

## 步骤 4：添加映射

添加映射的操作取决于创建的分片映射种类。如果你已创建列表映射，请添加列表映射。如果你已创建范围映射，请添加范围映射。

### 步骤 4，选项 1：映射列表映射的数据

通过为每个租户添加列表映射来映射数据。

	# Create the mappings and associate it with the new shards 
	Add-ListMapping 
	-KeyType $([int]) 
	-ListPoint '<tenant_id>' 
	-ListShardMap $ShardMap 
	-SqlServerName '<shard_server_name>' 
	-SqlDatabaseName '<shard_database_name>' 

### 步骤 4，选项 2：映射范围映射的数据

添加所有租户 ID 范围的范围映射 – 数据库关联：

	# Create the mappings and associate it with the new shards 
	Add-RangeMapping 
	-KeyType $([int]) 
	-RangeHigh '5' 
	-RangeLow '1' 
	-RangeShardMap $ShardMap 
	-SqlServerName '<shard_server_name>' 
	-SqlDatabaseName '<shard_database_name>' 


### 步骤 4，选项 3：映射单一数据库上多个租户的数据

对于每个租户，运行 Add-ListMapping（上面的选项 1）。


## 检查映射

可以使用以下命令来查询现有分片及其关联的映射的信息：

	# List the shards and mappings 
	Get-Shards -ShardMap $ShardMap 
	Get-Mappings -ShardMap $ShardMap 

## 摘要

完成设置后，可以开始使用弹性数据库客户端库。你还可以使用[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing)和[多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying)。

## 后续步骤


从 [Azure SQL 数据库 - 弹性数据库工具脚本](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db)获取 PowerShell 脚本。

GitHub 上也提供了这些工具：[Azure/elastic-db-tools](https://github.com/Azure/elastic-db-tools)。

使用拆分/合并工具在多租户模型与单租户模型之间来回移动数据。请参阅 [Split merge tool（拆分/合并工具）](/documentation/articles/sql-database-elastic-scale-get-started)。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-convert-to-use-elastic-tools/listmapping.png
[2]: ./media/sql-database-elastic-convert-to-use-elastic-tools/rangemapping.png
[3]: ./media/sql-database-elastic-convert-to-use-elastic-tools/multipleonsingledb.png
 

<!---HONumber=Mooncake_0503_2016-->
