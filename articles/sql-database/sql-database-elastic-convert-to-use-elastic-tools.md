<properties
    pageTitle="迁移要扩大的现有数据库 | Azure"
    description="创建分片映射管理器，将分片数据库转换为使用弹性数据库工具"
    services="sql-database"
    documentationcenter=""
    author="ddove"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="8c851d8e-8fd5-4327-89c1-9178b20ddd69"
    ms.service="sql-database"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-management"
    ms.date="10/24/2016"
    wacn.date="12/19/2016"
ms.author="ddove" />

# 迁移要扩展的现有数据库

使用 Azure SQL 数据库数据库工具（例如[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library/)）轻松管理现有的已扩大分片数据库。必须先转换现有的数据库集，然后才能使用[分片映射管理器](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。

## 概述
迁移现有的分片数据库：

1. 准备[分片映射管理器数据库](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。
2. 创建分片映射。
3. 准备各个分片。
4. 将映射添加到分片映射。

可以使用 [.NET Framework 客户端库](http://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client)或 [Azure SQL DB - 弹性数据库工具脚本](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db)中提供的 PowerShell 脚本来实现这些技巧。以下示例使用 PowerShell 脚本。

有关 ShardMapManager 的详细信息，请参阅[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。有关弹性数据库工具的概述，请参阅[弹性数据库功能概述](/documentation/articles/sql-database-elastic-scale-introduction/)。

## 准备分片映射管理器数据库
分片映射管理器是一个特殊的数据库，其中包含用于管理已扩展数据库的数据。可以使用现有数据库，或创建新的数据库。请注意：用作分片映射管理器的数据库应该是与分片不同的数据库。另请注意：PowerShell 脚本不会为你创建该数据库。

## 步骤 1：创建分片映射管理器

	# Create a shard map manager. 
	New-ShardMapManager -UserName '<user_name>' 
	-Password '<password>' 
	-SqlServerName '<server_name>' 
	-SqlDatabaseName '<smm_db_name>' 
	#<server_name> and <smm_db_name> are the server name and database name 
	# for the new or existing database that should be used for storing 
	# tenant-database mapping information.

### 检索分片映射管理器
创建后，可以使用此 cmdlet 检索分片映射管理器。每当需要使用 ShardMapManager 对象时，则需要执行此步骤。

	# Try to get a reference to the Shard Map Manager  
	$ShardMapManager = Get-ShardMapManager -UserName '<user_name>' 
	-Password '<password>' 
	-SqlServerName '<server_name>' 
	-SqlDatabaseName '<smm_db_name>' 

  
## 第 2 步：创建分片映射
必须选择要创建的分片映射类型。类型的选择取决于数据库架构：

1. 每个数据库一个租户（有关术语，请参阅[词汇表](/documentation/articles/sql-database-elastic-scale-glossary/)。）
2. 每个数据库多个租户（两种类型）：
	1. 列表映射
	2. 范围映射
 

对于单租户模型，创建“列表映射”分片映射。单租户模型将每个租户分配给一个数据库。这是适用于 SaaS 开发人员的有效模型，因为它可以简化管理。

![列表映射][1]

多租户模型将多个租户分配给单一数据库（可以跨多个数据库分布租户组）。如果希望每个租户具有较小的数据需求，请使用此模型。在此模型中，我们使用“范围映射”将一系列租户分配给数据库。

![范围映射][2]

或者你可以使用“列表映射”实现多租户数据库模型，以将多个租户分配给单一数据库。例如，DB1 用于存储租户 ID 1 和 5 的相关信息，而 DB2 用于存储租户 7 和租户 10 的数据。

![单一数据库上的多个租户][3]  


**根据你具体的选择，选择以下选项之一：**

### 选项 1：为列表映射创建分片映射
使用 ShardMapManager 对象创建分片映射。

	# $ShardMapManager is the shard map manager object. 
	$ShardMap = New-ListShardMap -KeyType $([int]) 
	-ListShardMapName 'ListShardMap' 
	-ShardMapManager $ShardMapManager 
 
 
### 选项 2：为范围映射创建分片映射
请注意：若要使用此映射模式，租户 ID 值需要为连续范围，并且可接受范围中存在间距，方法是只需在创建数据库时跳过范围即可。

	# $ShardMapManager is the shard map manager object 
	# 'RangeShardMap' is the unique identifier for the range shard map.  
	$ShardMap = New-RangeShardMap 
	-KeyType $([int]) 
	-RangeShardMapName 'RangeShardMap' 
	-ShardMapManager $ShardMapManager 

### 选项 3：单一数据库上的列表映射
设置此模式也要求创建列表映射，如步骤 2，选项 1 中所示。

## 步骤 3：准备各个分片
将每个分片（数据库）添加到分片映射管理器。此操作将准备用于存储映射信息的各个数据库。对每个分片执行此方法。
	 
	Add-Shard 
	-ShardMap $ShardMap 
	-SqlServerName '<shard_server_name>' 
	-SqlDatabaseName '<shard_database_name>'
	# The $ShardMap is the shard map created in step 2.
 

## 步骤 4：添加映射
添加映射的操作取决于所创建的分片映射种类。如果创建的是列表映射，则添加列表映射。如果创建的是范围映射，则添加范围映射。

### 选项 1：为列表映射映射数据
通过为每个租户添加列表映射来映射数据。

	# Create the mappings and associate it with the new shards 
	Add-ListMapping 
	-KeyType $([int]) 
	-ListPoint '<tenant_id>' 
	-ListShardMap $ShardMap 
	-SqlServerName '<shard_server_name>' 
	-SqlDatabaseName '<shard_database_name>' 

### 选项 2：为范围映射映射数据
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
对于每个租户，请运行 Add-ListMapping（上面的选项 1）。

## 检查映射
可以使用以下命令查询现有分片及其关联的映射的相关信息：

	# List the shards and mappings 
	Get-Shards -ShardMap $ShardMap 
	Get-Mappings -ShardMap $ShardMap 

## 摘要
完成设置后，便可以开始使用弹性数据库客户端库。还可以使用[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)和[多分片查询](/documentation/articles/sql-database-elastic-scale-multishard-querying/)。

## 后续步骤


从 [Azure SQL DB - 弹性数据库工具脚本](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db)获取 PowerShell 脚本。

GitHub 上也提供了这些工具：[Azure/elastic-db-tools](https://github.com/Azure/elastic-db-tools)。

使用拆分/合并工具在多租户模型与单租户模型之间来回移动数据。请参阅[拆分/合并工具](/documentation/articles/sql-database-elastic-scale-get-started/)。

## 其他资源
有关多租户软件即服务 (SaaS) 数据库应用程序的常见数据体系结构模式的信息，请参阅[包含 Azure SQL 数据库的多租户 SaaS 应用程序的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)。


<!--Image references-->

[1]: ./media/sql-database-elastic-convert-to-use-elastic-tools/listmapping.png
[2]: ./media/sql-database-elastic-convert-to-use-elastic-tools/rangemapping.png
[3]: ./media/sql-database-elastic-convert-to-use-elastic-tools/multipleonsingledb.png
 

<!---HONumber=Mooncake_1212_2016-->