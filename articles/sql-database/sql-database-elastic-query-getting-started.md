<properties
	pageTitle="跨已扩展的云数据库（水平分区）进行报告 | Azure"
	description="如何使用跨数据库数据库查询"
	services="sql-database"
	documentationCenter=""  
	manager="jhubbard"
	authors="sidneyh"/>

<tags
	ms.service="sql-database"
	ms.workload="sql-database"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/23/2016"
	wacn.date="12/26/2016"
	ms.author="SilviaDoomra" />

# 跨已扩展的云数据库进行报告（预览）

可以使用[弹性查询](/documentation/articles/sql-database-elastic-query-overview/)从单个连接点中的多个 Azure SQL 数据库中创建报告。必须对数据库进行水平分区（也称为“分片”）。

如果拥有现有的数据库，请参阅[将现有数据库迁移到已扩展的数据库](/documentation/articles/sql-database-elastic-convert-to-use-elastic-tools/)。

若要了解需要查询的 SQL 对象，请参阅[跨水平分区的数据库进行查询](/documentation/articles/sql-database-elastic-query-horizontal-partitioning/)。

## 先决条件

下载并运行[弹性数据库工具示例入门](/documentation/articles/sql-database-elastic-scale-get-started/)。

## 使用示例应用创建分片映射管理器

在此处，将创建分片映射管理器以及多个分片，然后将数据插入到分片中。如果分片中正好设置了分片数据，则可以跳过下面的步骤，直接转到下一部分。

1. 生成并运行**弹性数据库工具入门**示例应用程序。按照步骤一直执行到[下载并运行示例应用](/documentation/articles/sql-database-elastic-scale-get-started/#Getting-started-with-elastic-database-tools)部分中的步骤 7 为止。在步骤 7 结束时，会显示以下的命令提示符：

	![命令提示符][1]

2.  在命令窗口中键入“1”，然后按 **Enter**。此操作将创建分片映射管理器，并将两个分片添加到服务器。然后键入“3”并按 **Enter**；重复该操作四次。此操作将在分片中插入示例数据行。
3.  [Azure 门户预览](https://portal.azure.cn)应在 v12 服务器中显示三个新的数据库：

	![Visual Studio 确认][2]

	目前，支持通过弹性数据库客户端库跨数据库进行查询。例如，在命令窗口中使用第 4 个选项。来自多分片查询的结果始终是所有分片结果的 **UNION ALL**。

	在下一部分中，我们将创建支持更丰富的跨分片数据查询的示例数据库终结点。

## 创建弹性查询数据库

1. 打开 [Azure 门户预览](https://portal.azure.cn)并登录。
2. 在与分片设置相同的服务器中创建新的 Azure SQL 数据库。将数据库命名为“ElasticDBQuery”。

	![Azure 门户和定价层][3]

	注意：可以使用现有的数据库。如果这样做，该数据库不能是要对其执行查询的某一个分片。此数据库将用于为弹性数据库查询创建元数据对象。


## 创建数据库对象

### 数据库范围的主密钥和凭据

可使用它们连接到分片映射管理器和分片：

1. 在 Visual Studio 中打开 SQL Server Management Studio 或 SQL Server Data Tools。
2. 连接到 ElasticDBQuery 数据库，并执行以下 T-SQL 命令：

		CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<password>';

		CREATE DATABASE SCOPED CREDENTIAL ElasticDBQueryCred
		WITH IDENTITY = '<username>',
		SECRET = '<password>';

	“username”和“password”应与[弹性数据库工具入门](/documentation/articles/sql-database-elastic-scale-get-started/)中[下载并运行示例应用](/documentation/articles/sql-database-elastic-scale-get-started/#Getting-started-with-elastic-database-tools)的步骤 6 中所使用的登录信息相同。

### 外部数据源

若要创建外部数据源，请对 ElasticDBQuery 数据库执行以下命令：

	CREATE EXTERNAL DATA SOURCE MyElasticDBQueryDataSrc WITH
      (TYPE = SHARD_MAP_MANAGER,
      LOCATION = '<server_name>.database.chinacloudapi.cn',
      DATABASE_NAME = 'ElasticScaleStarterKit_ShardMapManagerDb',
	  CREDENTIAL = ElasticDBQueryCred,
 	  SHARD_MAP_NAME = 'CustomerIDShardMap'
    ) ;

 如果使用弹性数据库工具示例创建了分片映射和分片映射管理器，“CustomerIDShardMap”则是分片映射的名称。但是，如果为此示例使用了自定义设置，则它应该是在应用程序中所选择的分片映射名称。

### 外部表

通过对 ElasticDBQuery 数据库执行以下命令，创建与分片上的客户表匹配的外部表：

	CREATE EXTERNAL TABLE [dbo].[Customers]
	( [CustomerId] [int] NOT NULL,
	  [Name] [nvarchar](256) NOT NULL,
	  [RegionId] [int] NOT NULL)
	WITH
	( DATA_SOURCE = MyElasticDBQueryDataSrc,
      DISTRIBUTION = SHARDED([CustomerId])
	) ;

## 执行示例弹性数据库 T-SQL 查询

在定义了外部数据源和外部表后，便可以对外部表使用完整的 T-SQL。

对 ElasticDBQuery 数据库执行以下查询：

	select count(CustomerId) from [dbo].[Customers]

用户将注意到，查询会从所有分片聚合结果并输出以下内容：

![输出详细信息][4]

## 将弹性数据库查询结果导入 Excel

 可以将查询结果导入到 Excel 文件。

1. 启动 Excel 2013。
2. 	导航到“数据”功能区。
3. 	单击“从其他源”，然后单击“从 SQL Server”。

	![从其他源导入 Excel][5]
4. 	在“数据连接向导”中，键入服务器名称和登录凭据。然后，单击“下一步”。
5. 	在对话框“选择包含所需数据的数据库”中，选择“ElasticDBQuery”数据库。
6. 	在列表视图中选择“客户”表并单击“下一步”。然后单击“完成”。
7. 	在“导入数据”表单中的“请选择该数据在工作簿中的显示方式”下，选择“表”，然后单击“确定”。

存储在不同分片中、来自“客户”表的所有行将填入 Excel 工作表。

## 后续步骤
现在可以使用 Excel 的强大数据可视化功能。可以使用包含服务器名称、数据库名称和凭据的连接字符串，将 BI 和数据集成工具连接到弹性查询数据库。请确保支持将 SQL Server 用作工具的数据源。可以引用弹性查询数据库和外部表，就如同引用使用工具连接的任何其他 SQL Server 数据库和 SQL Server 表一样。

### 成本
使用弹性数据库查询功能不会产生额外的费用。

有关定价信息，请参阅 [SQL 数据库定价详细信息](/pricing/details/sql-database/)。


[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-query-getting-started/cmd-prompt.png
[2]: ./media/sql-database-elastic-query-getting-started/portal.png
[3]: ./media/sql-database-elastic-query-getting-started/tiers.png
[4]: ./media/sql-database-elastic-query-getting-started/details.png
[5]: ./media/sql-database-elastic-query-getting-started/exel-sources.png
<!--anchors-->

<!---HONumber=Mooncake_Quality_Review_1215_2016-->