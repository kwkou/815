<properties
    pageTitle="在具有不同架构的云数据库中进行查询 | Azure"
    description="如何在垂直分区上设置跨数据库查询"    
    services="sql-database"
    documentationCenter=""  
    manager="jhubbard"
    authors="torsteng"/>

<tags
    ms.service="sql-database"
    ms.date="05/27/2016"
    wacn.date="07/18/2016" />

# 在具有不同架构的云数据库中进行查询。（预览）

![跨不同数据库中的表进行查询][1]

垂直分区的数据库在不同的数据库中使用不同的表集。这意味着不同数据库上的架构是不同的。例如，清单的所有表都位于一个数据库上，而与会计相关的所有表都位于第二个数据库上。

## 先决条件

* 用户必须拥有 ALTER ANY EXTERNAL DATA SOURCE 权限。此权限包含在 ALTER DATABASE 权限中。
* 引用基础数据源需要 ALTER ANY EXTERNAL DATA SOURCE 权限。

## 概述

**注意**：与水平分区不同，这些 DDL 语句并不依赖于通过弹性数据库客户端库定义包含分片映射的数据层。

1. [CREATE MASTER KEY](https://msdn.microsoft.com/zh-cn/library/ms174382.aspx)
2. [CREATE DATABASE SCOPED CREDENTIAL](https://msdn.microsoft.com/zh-cn/library/mt270260.aspx)
3. [CREATE EXTERNAL DATA SOURCE](https://msdn.microsoft.com/zh-cn/library/dn935022.aspx)
4. [CREATE EXTERNAL TABLE](https://msdn.microsoft.com/zh-cn/library/dn935021.aspx)


## 创建数据库范围的主密钥和凭据 

弹性查询使用此凭据连接到远程数据库。

    CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'password';
    CREATE DATABASE SCOPED CREDENTIAL <credential_name>  WITH IDENTITY = '<username>',  
    SECRET = '<password>'
    [;]
 
**注意** 请确保“<username>”中不包括任何“@servername”后缀。

## 创建外部数据源

语法：

    <External_Data_Source> ::=
    CREATE EXTERNAL DATA SOURCE <data_source_name> WITH 
               (TYPE = RDBMS,
                LOCATION = ’<fully_qualified_server_name>’,
                DATABASE_NAME = ‘<remote_database_name>’,  
                CREDENTIAL = <credential_name> 
                ) [;] 

**重要事项** TYPE 参数必须设置为**“RDBMS”**。

### 示例 

以下示例说明了如何使用 CREATE 语句创建外部数据源。

	CREATE EXTERNAL DATA SOURCE RemoteReferenceData 
	WITH 
	( 
		TYPE=RDBMS, 
		LOCATION='myserver.database.chinacloudapi.cn', 
		DATABASE_NAME='ReferenceData', 
		CREDENTIAL= SqlUser 
	); 
 
若要检索当前外部数据源的列表：

    select * from sys.external_data_sources; 

### 外部表 

语法：

	CREATE EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name . ] table_name  
    ( { <column_definition> } [ ,...n ])     
	{ WITH ( <rdbms_external_table_options> ) } 
	)[;] 
	
	<rdbms_external_table_options> ::= 
      DATA_SOURCE = <External_Data_Source>, 
      [ SCHEMA_NAME = N'nonescaped_schema_name',] 
      [ OBJECT_NAME = N'nonescaped_object_name',] 

### 示例  

	CREATE EXTERNAL TABLE [dbo].[customer]( 
		[c_id] int NOT NULL, 
		[c_firstname] nvarchar(256) NULL, 
		[c_lastname] nvarchar(256) NOT NULL, 
		[street] nvarchar(256) NOT NULL, 
		[city] nvarchar(256) NOT NULL, 
		[state] nvarchar(20) NULL, 
		[country] nvarchar(50) NOT NULL, 
	) 
	WITH 
	( 
	       DATA_SOURCE = RemoteReferenceData 
	); 

以下示例演示如何从当前数据库中检索外部表的列表：

	select * from sys.external_tables; 

### 备注

弹性查询将扩展现有的外部表语法以定义使用 RDBMS 类型的外部数据源的外部表。垂直分区的外部表定义涉及以下几个方面：

* **架构**：外部表 DDL 定义了你的查询可以使用的架构。外部表定义中提供的架构需要与存储实际数据的远程数据库中的表的架构相匹配。

* **远程数据库引用**：外部表 DDL 引用外部数据源。外部数据源指定存储实际表数据的远程数据库的逻辑服务器名称和数据库名称。

使用上一节中所述的外部数据源时，用于创建外部表的语法如下：

DATA\_SOURCE 子句定义用于外部表的外部数据源（即，在垂直分区情况下的远程数据库）。

使用 SCHEMA\_NAME 和 OBJECT\_NAME 子句可分别将外部表定义映射到远程数据库上不同架构中的表，或映射到具有不同名称的表。如果你要在远程数据库上为目录视图或 DMV 定义外部表，或者在远程表名已在本地使用的任何其他情况下，这两个子句很有用。

以下 DDL 语句从本地目录中删除现有的外部表定义。它不会影响远程数据库。

	DROP EXTERNAL TABLE [ [ schema_name ] . | schema_name. ] table_name[;]  

**CREATE/DROP EXTERNAL TABLE 的权限：**外部表 DDL 需要 ALTER ANY EXTERNAL DATA SOURCE 权限，在引用基础数据源时也需要该权限。

## 安全注意事项
有权访问外部表的用户在使用外部数据源定义中提供的凭据时自动获得对基础远程表的访问权。你应小心管理对外部表的访问权以避免通过外部数据源的凭据意外地提升权限。可以使用常规 SQL 权限来授予或撤消对外部表的访问权，就像它是常规表一样。


## 示例：正在查询垂直分区数据库 

以下查询执行订单和订单行的两个本地表和客户的远程表之间的三向联接。这是弹性查询的引用数据用例的示例：

	SELECT  	
	 c_id as customer,
	 c_lastname as customer_name,
	 count(*) as cnt_orderline, 
	 max(ol_quantity) as max_quantity,
	 avg(ol_amount) as avg_amount,
	 min(ol_delivery_d) as min_deliv_date
	FROM customer 
	JOIN orders 
	ON c_id = o_c_id
	JOIN  order_line 
	ON o_id = ol_o_id and o_c_id = ol_c_id
	WHERE c_id = 100


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

可以使用常规 SQL Server 连接字符串将 BI 和数据集成工具连接到 SQL 数据库服务器上已启用弹性查询并已定义外部表的数据库。请确保支持将 SQL Server 用作工具的数据源。然后可以引用弹性查询数据库及其外部表，就如同使用工具连接的任何其他 SQL Server 数据库一样。

## 最佳实践 
 
* 确保已通过在 SQL 数据库防火墙配置中启用对 Azure 服务的访问授予弹性查询终结点数据库访问远程数据库的权限。另请确保外部数据源定义中提供的凭据可以成功登录到远程数据库并有权访问远程表。  

* 弹性查询最适合大部分计算可以在远程数据库上完成的查询。使用可以在远程数据库或联接上求值的选择性筛选器谓词（可以完全在远程数据库上执行），通常可以获得最佳查询性能。其他查询模式可能需要从远程数据库加载大量数据并且可能会执行效果不佳。


## 后续步骤

若要查询水平分区的数据库（也称为分片数据库），请参阅[跨分片云数据库（水平分区）的查询](/documentation/articles/sql-database-elastic-query-horizontal-partitioning/)。

[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]


<!--Image references-->
[1]: ./media/sql-database-elastic-query-vertical-partitioning/verticalpartitioning.png


<!--anchors-->

<!---HONumber=Mooncake_0711_2016-->