<properties
    pageTitle="用于分片的弹性数据库查询（水平分区）| Azure"
    description="如何对水平分区设置弹性查询"    
    services="sql-database"
    documentationCenter=""  
    manager="jeffreyg"
    authors="torsteng"/>

<tags
    ms.service="sql-database"
    ms.date="01/28/2016"
    wacn.date="06/14/2016" />

# 用于分片的弹性数据库查询（水平分区）

本文档说明如何为水平分区方案设置弹性数据库查询以及如何执行查询。有关水平分区方案的定义，请参阅[弹性数据库查询概述（预览版）](/documentation/articles/sql-database-elastic-query-overview/)。

![跨分片进行查询][1]

该功能是 Azure SQL [数据库弹性数据库功能集](/documentation/articles/sql-database-elastic-scale-introduction/)的一部分。
 
## 创建数据库对象

弹性数据库查询扩展 T-SQL 语法以引用数据层，这些数据层使用分片（或水平分区）将数据分布到多个数据库上。本部分概述了与分片表上的弹性查询关联的 DDL 语句。这些语句在弹性查询数据库中创建元数据表示形式的分片数据层。运行这些语句的先决条件是使用弹性数据库客户端库创建分片映射。有关详细信息，请参阅[分片映射管理](/documentation/articles/sql-database-elastic-scale-shard-map-management/)；或使用以下主题中的示例创建一个分片映射：[弹性数据库工具入门](/documentation/articles/sql-database-elastic-scale-get-started/)。

定义弹性数据库查询的数据库对象依赖于以下 T-SQL 语句，下面将针对水平分区方案对这些语句进行进一步说明：

* [CREATE MASTER KEY](https://msdn.microsoft.com/zh-cn/library/ms174382.aspx) 

* [CREATE DATABASE SCOPED CREDENTIAL](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx)

* [CREATE/DROP EXTERNAL DATA SOURCE](https://msdn.microsoft.com/zh-cn/library/dn935022.aspx)

* [CREATE/DROP EXTERNAL TABLE](https://msdn.microsoft.com/zh-cn/library/dn935021.aspx)

### 1\.1 数据库范围的主密钥和凭据 

凭据表示弹性查询将用于连接到 Azure SQL 数据库中的远程数据库的用户 ID 和密码。若要创建所需的主密钥和凭据，请使用以下语法：

    CREATE MASTER KEY ENCRYPTION BY PASSWORD = ’password’;
    CREATE DATABASE SCOPED CREDENTIAL <credential_name>  WITH IDENTITY = ‘<username>’,  
    SECRET = ‘<password>’
    [;]

或者若要删除凭据和密钥，请使用以下语法：

    DROP DATABASE SCOPED CREDENTIAL <credential_name>;  
    DROP MASTER KEY;   

 
**注意** 请确保 *< username>* 中不包括任何“@servername”后缀。

### 1\.2 外部数据源

通过定义外部数据源提供有关分片映射和数据层的信息。外部数据源引用分片映射。然后，弹性查询使用外部数据源和基础分片映射枚举参与数据层的数据库。用于创建外部数据源的语法定义如下：

	<External_Data_Source> ::=    
	CREATE EXTERNAL DATA SOURCE <data_source_name> WITH                               	           
			(TYPE = SHARD_MAP_MANAGER,
                   	LOCATION = '<fully_qualified_server_name>',
			DATABASE_NAME = ‘<shardmap_database_name>',
			CREDENTIAL = <credential_name>, 
			SHARD_MAP_NAME = ‘<shardmapname>’ 
                   ) [;] 
 
或者若要删除外部数据源：

	DROP EXTERNAL DATA SOURCE <data_source_name>[;] 

#### CREATE/DROP EXTERNAL DATA SOURCE 的权限 

用户必须拥有 ALTER ANY EXTERNAL DATA SOURCE 权限。此权限包含在 ALTER DATABASE 权限中。

**示例**

以下示例说明了如何使用 CREATE 语句创建外部数据源。

	CREATE EXTERNAL DATA SOURCE MyExtSrc 
	WITH 
	( 
		TYPE=SHARD_MAP_MANAGER,
		LOCATION='myserver.database.chinacloudapi.cn', 
		DATABASE_NAME='ShardMapDatabase', 
		CREDENTIAL= SMMUser, 
		SHARD_MAP_NAME='ShardMap' 
	);
 
你可以从以下目录视图检索当前外部数据源的列表：

	select * from sys.external_data_sources; 

请注意，在弹性查询处理过程中，相同的凭据用于读取分片映射和访问上分片的数据。

### 1\.3 外部表 
 
弹性查询将扩展外部表 DDL 以引用水平分区到多个数据库的外部表。外部表定义涵盖以下方面：

* **架构**：外部表 DDL 定义了你的查询可以使用的架构。外部表定义中提供的架构需要与存储实际数据的分片上的表的架构匹配。 

* **数据分布**：外部表 DDL 定义了用于将数据分布到数据层上的数据分布。请注意 Azure SQL 数据库不会针对分片上的实际分布验证外部表上定义的分布。你负责确保分片上的实际数据分布与外部表定义相匹配。

* **数据层引用**：外部表 DDL 引用外部数据源。外部数据源指定分片映射，后者为外部表提供在数据层中找到所有数据库所需的信息。

使用上一节中所述的外部数据源时，用于创建和删除外部表的语法如下：

	CREATE EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name. ] table_name  
        ( { <column_definition> } [ ,...n ])     
	    { WITH ( <sharded_external_table_options> ) }
	) [;]  
	
	<sharded_external_table_options> ::= 
      DATA_SOURCE = <External_Data_Source>,       
	  [ SCHEMA_NAME = N'nonescaped_schema_name',] 
      [ OBJECT_NAME = N'nonescaped_object_name',] 
      DISTRIBUTION = SHARDED(<sharding_column_name>) | REPLICATED |ROUND_ROBIN

DATA\_SOURCE 子句定义用于外部表的外部数据源（在水平分区情况下的分片映射）。

使用 SCHEMA\_NAME 和 OBJECT\_NAME 子句可分别将外部表定义映射到分片上不同架构中的表，或映射到具有不同名称的表。如果省略，则假定远程对象的架构是“dbo”，并假定其名称与所定义的外部表名称相同。

当远程表的名称已在你要在其中创建外部表的数据库中使用时，SCHEMA\_NAME 和 OBJECT\_NAME 子句特别有用。此问题的一个示例是当你要定义外部表以在扩大的数据层上获取目录视图或 DMV 的聚合视图时。由于目录视图和 DMV 已在本地存在，因此不能在外部表定义中使用其名称。而是改用不同名称，并在 SCHEMA\_NAME 和/或 OBJECT\_NAME 子句中使用目录视图或 DMV 的名称。（请参阅下面的示例。）

DISTRIBUTION 子句指定用于此表的数据分布：

* SHARDED 表示此表的数据将水平分区到分片映射中的数据库上。数据分布的分区键在 <sharding_column_name> 参数中捕获。  

* REPLICATED 表示表的相同副本将存在于分片映射中的每个数据库上。Azure SQL 数据库不维护表的副本。你负责确保各数据库上的副本是相同的。

* ROUND\_ROBIN 表示将使用水平分区对表进行分布。但是，已使用应用程序相关的分布。

查询处理器利用 DISTRIBUTION 子句中提供的信息来构建最有效的查询计划。

使用以下语句删除外部表：

	DROP EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name. ] table_name[;]  

**CREATE/DROP EXTERNAL TABLE 的权限：**需要 ALTER ANY EXTERNAL DATA SOURCE 权限，在引用基础数据源时也需要该权限。

**安全注意事项：**有权访问外部表的用户在使用外部数据源定义中提供的凭据时自动获得对基础远程表的访问权。你应小心管理对外部表的访问权以避免通过外部数据源的凭据意外地提升权限。可以使用常规 SQL 权限来授予或撤消对外部表的访问权，就像它是常规表一样。

**示例**：以下示例说明如何创建外部表：

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

以下示例演示如何从当前数据库中检索外部表的列表：

	select * from sys.external_tables; 

## 查询 

### 2\.1 完全保真的 T-SQL 查询 

定义外部数据源和外部表后，现在可以对外部表使用完整的 T-SQL。

**水平分区的示例：**下面的查询执行仓库、订单和订单行之间的三向联接，并使用多个聚合和选择性筛选器。它假定进行 (1) 水平分区（分片），(2) 仓库、订单和订单行按仓库 ID 列共享，并且弹性查询可以排列分片上的联接，且并行处理分片上的查询的成本高昂部分。

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
 
### 2\.2 存储过程 SP\_EXECUTE\_FANOUT 

弹性查询还引入了一个存储过程，以便提供对分片的直接访问。该存储过程名为 sp\_execute\_fanout 并采用以下参数：

* 服务器名称 (nvarchar)：托管分片映射的逻辑服务器的完全限定名称。 
* 分片映射数据库名称 (nvarchar)：分片映射数据库的名称。 
* 用户名 (nvarchar)：用于登录到分片映射数据库的用户名。 
* 密码 (nvarchar)：用户的密码。 
* 分片映射名称 (nvarchar)：要用于查询的分片映射的名称。可以在 \_ShardManagement.ShardMapsGlobal 表中找到该名称，它是使用[弹性数据库工具入门](/documentation/articles/sql-database-elastic-scale-get-started/)中的示例应用创建数据库时使用的默认名称。应用中的默认名称为“CustomerIDShardMap”。
*  查询：要在每个分片上执行的 T-SQL 查询。 
*  参数声明 (nvarchar) - 可选：在查询参数（如 sp\_executesql）中使用的参数的字符串（包含数据类型定义）。 
*  参数值列表 - 可选：以逗号分隔的参数值（如 sp\_executesql）的列表  

sp\_execute\_fanout 使用调用参数中提供的分片映射信息在注册到分片映射的所有分片上执行给定的 T-SQL 语句。使用 UNION ALL 语义合并任何结果。结果还包括附加的“virtual”列，其中包含分片名称。

请注意，使用相同的凭据连接到分片映射数据库和分片。

示例：

	sp_execute_fanout 
		N'myserver.database.chinacloudapi.cn', 
		N'ShardMapDb', 
		N'myuser', 
		N'MyPwd', 
		N'ShardMap', 
		N'select count(w_id) as foo from warehouse' 

## 工具的连接  

可以使用常规 SQL Server 连接字符串将应用程序、BI 和数据集成工具连接到具有外部表定义的数据库。请确保支持将 SQL Server 用作工具的数据源。然后引用弹性查询数据库就像连接到工具的任何其他 SQL Server 数据库一样，并像使用本地表一样从工具或应用程序使用外部表。

## 最佳实践 

* 确保已向弹性查询终结点数据库授予通过 SQL 数据库防火墙访问分片映射数据库和所有分片的权限。  

* 弹性查询不验证或强制执行由外部表定义的数据分布。如果实际的数据分布不同于表定义中指定的分布，你的查询可能会产生意外的结果。

* 当分片键上的谓词允许安全地从处理中排除某些分片时，弹性查询当前不执行分片消除。

* 弹性查询最适合大部分计算可以在分片上完成的查询。使用可以在分片或联接上通过分区键求值的选择性筛选器谓词（可以在所有分片上以分区对齐方式执行），通常可以获得最佳查询性能。其他查询模式可能需要从分片将大量数据加载到头节点并且可能会执行效果不佳


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]


<!--Image references-->
[1]: ./media/sql-database-elastic-query-horizontal-partitioning/horizontalpartitioning.png
<!--anchors-->

<!---HONumber=Mooncake_0606_2016-->