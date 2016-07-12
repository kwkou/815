<properties
    pageTitle="适用于跨数据库查询的弹性数据库查询（垂直分区）| Azure"
    description="如何在垂直分区上设置跨数据库查询"    
    services="sql-database"
    documentationCenter=""  
    manager="jhubbard"
    authors="torsteng"/>

<tags
    ms.service="sql-database"
    ms.date="04/11/2016"
    wacn.date="06/14/2016" />

# 适用于跨数据库查询的弹性数据库查询（垂直分区）

本文档说明如何为跨数据库查询方案（垂直分区）设置弹性查询，以及如何执行查询。有关垂直分区方案的定义，请参阅 [Azure SQL 数据库弹性数据库查询概述（预览版）](/documentation/articles/sql-database-elastic-query-overview/)。

![跨不同数据库中的表进行查询][1]

## 创建数据库对象

对于垂直分区方案，弹性查询将扩展当前 T-SQL DDL 以引用存储在远程数据库中的表。本部分概述了用于配置弹性查询以透明地访问远程表的 DDL 语句。这些 DDL 语句允许在本地数据库中创建元数据表示形式的远程表。

**注意**：与水平分区不同，这些 DDL 语句并不依赖于通过弹性数据库客户端库定义包含分片映射的数据层。

定义弹性数据库查询的数据库对象依赖于以下 T-SQL 语句，下面将针对垂直分区方案对这些语句进行进一步说明：

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
    
若要删除凭据：
    
    DROP DATABASE SCOPED CREDENTIAL <credential_name>;  
    DROP MASTER KEY;   

 
请确保 *< username>* 中不包括任何“@servername”后缀。

### 1\.2 外部数据源

通过定义外部数据源为弹性查询提供有关远程数据库的信息。用于创建和删除外部数据源的语法定义如下：

    <External_Data_Source> ::=
    CREATE EXTERNAL DATA SOURCE <data_source_name> WITH 
               (TYPE = RDBMS,
                LOCATION = ’<fully_qualified_server_name>’,
                DATABASE_NAME = ‘<remote_database_name>’,  
                CREDENTIAL = <credential_name> 
                ) [;] 

请注意将此数据源定义为 RDBMS 的 TYPE 参数。

可以使用以下语句删除外部数据源：

    DROP EXTERNAL DATA SOURCE <data_source_name>[;]

#### CREATE/DROP EXTERNAL DATA SOURCE 的权限 

用户必须拥有 ALTER ANY EXTERNAL DATA SOURCE 权限。此权限包含在 ALTER DATABASE 权限中。

**示例**

以下示例说明了如何使用 CREATE 语句创建外部数据源。

	CREATE EXTERNAL DATA SOURCE RemoteReferenceData 
	WITH 
	( 
		TYPE=RDBMS, 
		LOCATION='myserver.database.chinacloudapi.cn', 
		DATABASE_NAME='ReferenceData', 
		CREDENTIAL= SqlUser 
	); 
 
若要从以下目录视图中检索当前外部数据源的列表，请执行以下命令：

    select * from sys.external_data_sources; 

### 1\.3 外部表 

弹性查询将扩展现有的外部表语法以定义使用 RDBMS 类型的外部数据源的外部表。垂直分区的外部表定义涉及以下几个方面：

* **架构**：外部表 DDL 定义了你的查询可以使用的架构。外部表定义中提供的架构需要与存储实际数据的远程数据库中的表的架构相匹配。 

* **远程数据库引用**：外部表 DDL 引用外部数据源。外部数据源指定存储实际表数据的远程数据库的逻辑服务器名称和数据库名称。

使用上一节中所述的外部数据源时，用于创建外部表的语法如下：

	CREATE EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name . ] table_name  
    ( { <column_definition> } [ ,...n ])     
	{ WITH ( <rdbms_external_table_options> ) } 
	)[;] 
	
	<rdbms_external_table_options> ::= 
      DATA_SOURCE = <External_Data_Source>, 
      [ SCHEMA_NAME = N'nonescaped_schema_name',] 
      [ OBJECT_NAME = N'nonescaped_object_name',] 

DATA\_SOURCE 子句定义用于外部表的外部数据源（即，在垂直分区情况下的远程数据库）。

使用 SCHEMA\_NAME 和 OBJECT\_NAME 子句可分别将外部表定义映射到远程数据库上不同架构中的表，或映射到具有不同名称的表。如果你要在远程数据库上为目录视图或 DMV 定义外部表，或者在远程表名已在本地使用的任何其他情况下，这两个子句很有用。

以下 DDL 语句从本地目录中删除现有的外部表定义。它不会影响远程数据库。

	DROP EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name. ] table_name[;]  

**CREATE/DROP EXTERNAL TABLE 的权限：**外部表 DDL 需要 ALTER ANY EXTERNAL DATA SOURCE 权限，在引用基础数据源时也需要该权限。

**安全注意事项：**有权访问外部表的用户在使用外部数据源定义中提供的凭据时自动获得对基础远程表的访问权。你应小心管理对外部表的访问权以避免通过外部数据源的凭据意外地提升权限。可以使用常规 SQL 权限来授予或撤消对外部表的访问权，就像它是常规表一样。


 **示例**：以下示例说明如何创建外部表：

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

## 查询

### 2\.1 完全保真的 T-SQL 查询 

定义外部数据源和外部表后，可以对外部表使用完整的 T-SQL。

**垂直分区的示例**：以下查询执行订单和订单行的两个本地表和客户的远程表之间的三向联接。这是弹性查询的引用数据用例的示例：

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

  
## 工具的连接

可以使用常规 SQL Server 连接字符串将 BI 和数据集成工具连接到 SQL 数据库服务器上已启用弹性查询并已定义外部表的数据库。请确保支持将 SQL Server 用作工具的数据源。然后可以引用弹性查询数据库及其外部表，就如同使用工具连接的任何其他 SQL Server 数据库一样。

## 最佳实践 
 
* 确保已通过在 SQL 数据库防火墙配置中启用对 Azure 服务的访问授予弹性查询终结点数据库访问远程数据库的权限。另请确保外部数据源定义中提供的凭据可以成功登录到远程数据库并有权访问远程表。  

* 弹性查询最适合大部分计算可以在远程数据库上完成的查询。使用可以在远程数据库或联接上求值的选择性筛选器谓词（可以完全在远程数据库上执行），通常可以获得最佳查询性能。其他查询模式可能需要从远程数据库加载大量数据并且可能会执行效果不佳。


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]


<!--Image references-->
[1]: ./media/sql-database-elastic-query-vertical-partitioning/verticalpartitioning.png


<!--anchors-->

<!---HONumber=Mooncake_0606_2016-->
