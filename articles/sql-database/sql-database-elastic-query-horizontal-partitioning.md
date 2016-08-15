<properties
    pageTitle="跨扩展云数据库进行报告（预览）| Azure"
    description="如何对横向分区设置弹性查询"    
    services="sql-database"
    documentationCenter=""  
    manager="jhubbard"
    authors="torsteng"/>

<tags
    ms.service="sql-database"
    ms.date="05/27/2016"
    wacn.date="07/18/2016" />

# 跨扩展云数据库进行报告（预览）

![跨分片进行查询][1]

分片数据库跨扩展数据层分布行。所有分区（也称为横向分区）数据库的架构都是一样的。使用弹性查询，可以创建跨分片数据库中的所有数据库的报表。

有关快速入门，请参阅 [Reporting across scaled-out cloud databases（跨扩展云数据库进行报告）](/documentation/articles/sql-database-elastic-query-getting-started)。

对于非分片数据库，请参阅 [Query across cloud databases with different schemas（跨具有不同架构的云数据库进行查询）](/documentation/articles/sql-database-elastic-query-vertical-partitioning)。

 
## 先决条件

* 使用弹性数据库客户端库创建分片映射。请参阅 [Shard map management（分片映射管理）](/documentation/articles/sql-database-elastic-scale-shard-map-management/)。或者使用 [Get started with elastic database tools（弹性数据库工具入门）](/documentation/articles/sql-database-elastic-scale-get-started/)中的示例应用程序。
* 你也可以参阅 [Migrate existing databases to scaled-out databases（将现有数据库迁移到扩展数据库）](/documentation/articles/sql-database-elastic-convert-to-use-elastic-tools/)。
* 用户必须拥有 ALTER ANY EXTERNAL DATA SOURCE 权限。此权限包含在 ALTER DATABASE 权限中。
* 引用基础数据源需要 ALTER ANY EXTERNAL DATA SOURCE 权限。

## 概述

这些语句在弹性查询数据库中创建元数据表示形式的分片数据层。


1. [CREATE MASTER KEY](https://msdn.microsoft.com/zh-cn/library/ms174382.aspx)
2. [CREATE DATABASE SCOPED CREDENTIAL](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx)
3. [CREATE EXTERNAL DATA SOURCE](https://msdn.microsoft.com/zh-cn/library/dn935022.aspx)
4. [CREATE EXTERNAL TABLE](https://msdn.microsoft.com/zh-cn/library/dn935021.aspx)

## 1\.1 创建数据库范围的主密钥和凭据 

弹性查询使用此凭据连接到远程数据库。

    CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'password';
    CREATE DATABASE SCOPED CREDENTIAL <credential_name>  WITH IDENTITY = '<username>',  
    SECRET = '<password>'
    [;]
 
**注意** 请确保“<username>”中不包括任何“@servername”后缀。

## 1\.2 创建外部数据源

语法：

	<External_Data_Source> ::=    
	CREATE EXTERNAL DATA SOURCE <data_source_name> WITH                               	           
			(TYPE = SHARD_MAP_MANAGER,
                   	LOCATION = '<fully_qualified_server_name>',
			DATABASE_NAME = ‘<shardmap_database_name>',
			CREDENTIAL = <credential_name>, 
			SHARD_MAP_NAME = ‘<shardmapname>’ 
                   ) [;] 

### 示例 

	CREATE EXTERNAL DATA SOURCE MyExtSrc 
	WITH 
	( 
		TYPE=SHARD_MAP_MANAGER,
		LOCATION='myserver.database.chinacloudapi.cn', 
		DATABASE_NAME='ShardMapDatabase', 
		CREDENTIAL= SMMUser, 
		SHARD_MAP_NAME='ShardMap' 
	);
 
检索当前外部数据源的列表：

	select * from sys.external_data_sources; 

外部数据源引用分片映射。然后，弹性查询使用外部数据源和基础分片映射枚举参与数据层的数据库。
在弹性查询处理过程中，相同的凭据用于读取分片映射和访问上分片的数据。

## 1\.3 创建外部表 
 
语法：

	CREATE EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name. ] table_name  
        ( { <column_definition> } [ ,...n ])     
	    { WITH ( <sharded_external_table_options> ) }
	) [;]  
	
	<sharded_external_table_options> ::= 
      DATA_SOURCE = <External_Data_Source>,       
	  [ SCHEMA_NAME = N'nonescaped_schema_name',] 
      [ OBJECT_NAME = N'nonescaped_object_name',] 
      DISTRIBUTION = SHARDED(<sharding_column_name>) | REPLICATED |ROUND_ROBIN

**示例**

	CREATE EXTERNAL TABLE [dbo].[order_line]( 
		 [ol_o_id] int NOT NULL, 
		 [ol_d_id] tinyint NOT NULL,
		 [ol_w_id] int NOT NULL, 
		 [ol_number] tinyint NOT NULL, 
		 [ol_i_id] int NOT NULL, 
		 [ol_delivery_d] datetime NOT NULL, 
		 [ol_amount] smallmoney NOT NULL, 
		 [ol_supply_w_id] int NOT NULL, 
		 [ol_quantity] smallint NOT NULL, 
		 [ol_dist_info] char(24) NOT NULL 
	) 
	
	WITH 
	( 
		DATA_SOURCE = MyExtSrc, 
	 	SCHEMA_NAME = 'orders', 
	 	OBJECT_NAME = 'order_details', 
		DISTRIBUTION=SHARDED(ol_w_id)
	); 

从当前数据库中检索外部表的列表：

	SELECT * from sys.external_tables; 

要删除外部表：

	DROP EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name. ] table_name[;]

### 备注

DATA\_SOURCE 子句定义了用于外部表的外部数据源（分片映射）。

SCHEMA\_NAME 和 OBJECT\_NAME 子句将外部表定义映射到不同架构的表。如果省略，则假定远程对象的架构是“dbo”，并假定其名称与所定义的外部表名称相同。如果远程表的名称已在你要在其中创建外部表的数据库中使用，那么该做法很有用。例如，你想要定义外部表以在扩展数据层上获取目录视图或 DMV 的聚合视图。由于目录视图和 DMV 已在本地存在，因此不能在外部表定义中使用其名称。而是改用不同名称，并在 SCHEMA\_NAME 和/或 OBJECT\_NAME 子句中使用目录视图或 DMV 的名称。（请参阅下面的示例。）

DISTRIBUTION 子句指定用于此表的数据分布。查询处理器利用 DISTRIBUTION 子句中提供的信息来构建最有效的查询计划。

1. **SHARDED** 表示数据在各数据库之间横向分区。数据分布的分区键为 **<sharding\_column\_name>** 参数。
2. **REPLICATED** 表示每个数据库都存在表的相同副本。你要负责确保各数据库上的副本是相同的。
3. **ROUND\_ROBIN** 表示使用依赖于应用程序的分布方法对表进行横向分区。

**数据层引用**：外部表 DDL 引用外部数据源。外部数据源指定分片映射，后者为外部表提供在数据层中找到所有数据库所需的信息。


### 安全注意事项 

有权访问外部表的用户在使用外部数据源定义中提供的凭据时自动获得对基础远程表的访问权。避免通过外部数据源的凭据进行不必要的权限提升。将外部表当作常规表，在其中使用 GRANT 或 REVOKE。

定义外部数据源和外部表后，可以对外部表使用完整的 T-SQL。

## 示例：查询横向分区的数据库 

下面的查询在仓库、订单和订单行之间执行三向联接，并使用多个聚合和选择性筛选器。它假定 (1) 执行横向分区（分片），(2) 仓库、订单和订单行按仓库 ID 列进行分片，弹性查询可以在分片上并置联接，以及并行处理分片上成本较高部分的查询。

	select  
		 w_id as warehouse,
		 o_c_id as customer,
		 count(*) as cnt_orderline,
		 max(ol_quantity) as max_quantity,
		 avg(ol_amount) as avg_amount, 
		 min(ol_delivery_d) as min_deliv_date
	from warehouse 
	join orders 
	on w_id = o_w_id
	join order_line 
	on o_id = ol_o_id and o_w_id = ol_w_id 
	where w_id > 100 and w_id < 200 
	group by w_id, o_c_id 
 
## 远程 T-SQL 执行的存储过程：sp\_execute\_remote

弹性查询还引入了一个存储过程，以便提供对分片的直接访问。该存储过程名为 [sp\_execute \_remote](https://msdn.microsoft.com/zh-cn/library/mt703714)，可用于执行远程存储过程或远程数据库上的 T-SQL 代码。它采用了以下参数：

* 数据源名称 (nvarchar)：RDBMS 类型的外部数据源名称。
* 查询 (nvarchar)：要在每个分片上执行的 T-SQL 查询。
* 参数声明 (nvarchar) - 可选：在查询参数（如 sp\_executesql）中使用的参数的字符串（包含数据类型定义）。
* 参数值列表 - 可选：以逗号分隔的参数值（如 sp\_executesql）的列表。

Sp\_execute\_remote 使用调用参数中提供的外部数据源在远程数据库上执行给定的 T-SQL 语句。它使用外部数据源的凭据连接到分片映射管理器数据库和远程数据库。

示例：

	EXEC sp_execute_remote
		N'MyExtSrc',
		N'select count(w_id) as foo from warehouse' 

## 工具的连接  

可以使用常规 SQL Server 连接字符串将应用程序、BI 和数据集成工具连接到具有外部表定义的数据库。请确保支持将 SQL Server 用作工具的数据源。然后引用弹性查询数据库就像连接到工具的任何其他 SQL Server 数据库一样，并像使用本地表一样从工具或应用程序使用外部表。

## 最佳实践 

* 确保已向弹性查询终结点数据库授予通过 SQL 数据库防火墙访问分片映射数据库和所有分片的权限。

* 验证或强制执行由外部表定义的数据分布。如果实际的数据分布不同于表定义中指定的分布，你的查询可能会产生意外的结果。

* 当分片键上的谓词允许安全地从处理中排除某些分片时，弹性查询当前不执行分片消除。

* 弹性查询最适合大部分计算可以在分片上完成的查询。使用可以在分片或联接上通过分区键求值的选择性筛选器谓词（可以在所有分片上以分区对齐方式执行），通常可以获得最佳查询性能。其他查询模式可能需要从分片将大量数据加载到头节点并且可能会执行效果不佳

[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-query-horizontal-partitioning/horizontalpartitioning.png
<!--anchors-->

<!---HONumber=Mooncake_0711_2016-->