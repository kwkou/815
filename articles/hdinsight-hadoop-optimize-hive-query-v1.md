<properties
   pageTitle="优化 Hive 查询以便在 HDInsight 中更快地执行 | Azure"
   description="了解如何在 HDInsight 中优化 Hive 查询"
   services="hdinsight"
   documentationCenter=""
   authors="rashimg"
   manager="mwinkle"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.date="07/28/2015"
   wacn.date="01/15/2016"/>


# 在 Hdinsight 中优化 Hadoop 的 Hive 查询

默认情况下，不会为了性能而优化 Hadoop 群集。本文介绍可对查询应用的几种最常见 Hive 性能优化方法。

##向外缩放辅助节点

增加群集中的辅助节点数目，即可利用更多并行运行的映射器和化简器。在 HDInsight 中，可通过两种方式增加扩大的数目：

- 在预配时，可以使用 Azure 经典管理门户、Azure PowerShell 或跨平台命令行界面指定辅助节点的数目。有关详细信息，请参阅[设置 HDInsight 群集](/documentation/articles/hdinsight-provision-clusters-v1/)。以下屏幕显示了 Azure 门户预览上的辅助节点配置：

	![scaleout\_1][image-hdi-optimize-hive-scaleout_1]
- 在运行时，你也可以向外缩放群集，而无需重建群集。如下所示。![scaleout\_1][image-hdi-optimize-hive-scaleout_2]

有关 HDInsight 支持的不同虚拟机的详细信息，请参阅 [HDInsight 定价](/home/features/hdinsight/pricing/)。

##启用 Tez

[Apache Tez](http://hortonworks.com/hadoop/tez/) 是 MapReduce 引擎的替代执行引擎：

![tez_1][image-hdi-optimize-hive-tez_1]


Tez 速度更快，因为：

- 在 MapReduce 引擎中以单个作业执行定向无圈图 (DAG)，表达的 DAG 要求每组映射器后接一组化简器。这会导致针对每个 Hive 查询运行多个 MapReduce 作业。Tez 没有这种局限性，它可以将复杂的 DAG 作为一个作业进行处理，因此将作业启动开销降到最低。
- **避免不必要的写入**。由于要为 MapReduce 引擎的同一 Hive 查询运行多个作业，每个作业的输出将写入 HDFS 以用作暂存数据。而 Tez 最大程度地减少了对每个 Hive 查询运行的作业数，因此能够避免不必要的写入。
- **最大程度地降低启动延迟**。Tez 可以减少需要启动的映射器数目，同时还能提高优化吞吐量，因此，更有利于最小化启动延迟。
- **重复使用容器**。Tez 会尽可能地重复使用容器，以确保降低由于启动容器而产生的延迟。
- **连续优化技术**。传统上，优化是在编译阶段完成的。但是，这可以提供有关输入的详细信息，以便在运行时更好地进行优化。Tez 使用连续优化技术，从而可以在运行时阶段进一步优化计划。

有关这些概念的详细信息，请单击[此处](http://hortonworks.com/hadoop/tez/)

你可以通过在查询的前面加上以下设置作为前缀，来执行 Tez 支持的任何 Hive 查询：

	set hive.execution.engine=tez;

必须在预配时启用 Tez。以下 Azure PowerShell 脚本示例用于预配已启用 Tez 的 Hadoop 群集：


	$clusterName = "[HDInsightClusterName]"
	$location = "[AzureDataCenter]" #i.e. China North
	$dataNodes = 32 # number of worker nodes in the cluster

	$defaultStorageAccountName = "[DefaultStorageAccountName]"
	$defaultStorageContainerName = "[DefaultBlobContainerName]"
	$defaultStorageAccountKey = $defaultStorageAccountKey = Get-AzureStorageKey $defaultStorageAccountName.ToLower() | %{ $_.Primary }

	$hdiUserName = "[HTTPUserName]"
	$hdiPassword = "[HTTPUserPassword]"

	$hdiSecurePassword = ConvertTo-SecureString $hdiPassword -AsPlainText -Force
	$hdiCredential = New-Object System.Management.Automation.PSCredential($hdiUserName, $hdiSecurePassword)

	$hiveConfig = new-object 'Microsoft.WindowsAzure.Management.HDInsight.Cmdlet.DataObjects.AzureHDInsightHiveConfiguration'
	$hiveConfig.Configuration = @{ "hive.execution.engine"="tez" }

	New-AzureHDInsightClusterConfig -ClusterSizeInNodes $dataNodes -HeadNodeVMSize Standard_D14 -DataNodeVMSize Standard_D14 |
	Set-AzureHDInsightDefaultStorage -StorageAccountName "$defaultStorageAccountName.blob.core.chinacloudapi.cn" -StorageAccountKey $defaultStorageAccountKey -StorageContainerName $defaultStorageContainerName |
	Add-AzureHDInsightConfigValues -Hive $hiveConfig |
	New-AzureHDInsightCluster -Name $clusterName -Location $location -Credential $hdiCredential

## Hive 分区

I/O 操作是运行 Hive 查询的主要性能瓶颈。如果可以减少需要读取的数据量，即可改善性能。默认情况下，Hive 查询扫描整个 Hive 表。对于表扫描之类的查询，这非常有利；但是，对于只需扫描少量数据的查询（例如，使用筛选进行查询），这会产生不必要的开销。使用 Hive 分区，Hive 查询只需访问 Hive 表中必要的数据量。

Hive 分区的实现方法是将未经处理的数据刷新成新的目录，而每个分区区都有自己的目录 - 其中的分区由用户定义。下图说明如何根据*年*列来分区 Hive 表。每年都会创建新的目录。

![partitioning][image-hdi-optimize-hive-partitioning_1]

一些分区注意事项：

- **不要分区不足** - 根据包含少量值的列进行分区可能会导致创建很少的分区。例如，根据性别（男性和女性）分区只会创建两个分区，最多只会将延迟降低一半。

- **不要过度分区** - 另一种极端是，根据包含唯一值（例如 userid）的列进行分区会导致创建多个分区，从而给群集命名节点带来很大的压力，因为它必须处理大量的目录。这是另一种极端做法。

- **避免数据偏斜** - 明智选择分区键，以便所有分区的大小均等。例如，根据*州*分区可能会导致“加利福尼亚州”的记录数几乎是“佛蒙特州”的 30 倍，因为两个州的人口有差异。

若要创建分区表，请使用 *Partitioned By* 子句：

    CREATE TABLE lineitem_part
    	(L_ORDERKEY INT, L_PARTKEY INT, L_SUPPKEY INT,L_LINENUMBER INT,
    	 L_QUANTITY DOUBLE, L_EXTENDEDPRICE DOUBLE, L_DISCOUNT DOUBLE,
    	 L_TAX DOUBLE, L_RETURNFLAG STRING, L_LINESTATUS STRING,
    	 L_SHIPDATE_PS STRING, L_COMMITDATE STRING, L_RECEIPTDATE 	  	 STRING, L_SHIPINSTRUCT STRING, L_SHIPMODE STRING,
    	 L_COMMENT STRING)
    PARTITIONED BY(L_SHIPDATE STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    STORED AS TEXTFILE;

创建分区表后，可以创建静态分区或动态分区。

- **静态分区**表示已在相应目录中创建了分片数据，你可以请求根据目录位置在 Hive 中手动分区。以下代码段对此做了演示。

	    INSERT OVERWRITE TABLE lineitem_part
	    PARTITION (L_SHIPDATE = ‘5/23/1996 12:00:00 AM’)
	    SELECT * FROM lineitem 
	    WHERE lineitem.L_SHIPDATE = ‘5/23/1996 12:00:00 AM’

	    ALTER TABLE lineitem_part ADD PARTITION (L_SHIPDATE = ‘5/23/1996 12:00:00 AM’))
	    LOCATION ‘wasb://sampledata@ignitedemo.blob.core.chinacloudapi.cn/partitions/5_23_1996/'

- **动态分区**表示你希望 Hive 自动为你创建分区。由于我们已经基于暂存表创建了分区表，我们需要做的就是将数据插入分区表，如下所示：

	    SET hive.exec.dynamic.partition = true;
	    SET hive.exec.dynamic.partition.mode = nonstrict;
	    INSERT INTO TABLE lineitem_part
	    PARTITION (L_SHIPDATE)
	    SELECT L_ORDERKEY as L_ORDERKEY, L_PARTKEY as L_PARTKEY , 
	    	 L_SUPPKEY as L_SUPPKEY, L_LINENUMBER as L_LINENUMBER,
	     	 L_QUANTITY as L_QUANTITY, L_EXTENDEDPRICE as L_EXTENDEDPRICE,
	    	 L_DISCOUNT as L_DISCOUNT, L_TAX as L_TAX, L_RETURNFLAG as 	 	 L_RETURNFLAG, L_LINESTATUS as L_LINESTATUS, L_SHIPDATE as 	 	 L_SHIPDATE_PS, L_COMMITDATE as L_COMMITDATE, L_RECEIPTDATE as 	 L_RECEIPTDATE, L_SHIPINSTRUCT as L_SHIPINSTRUCT, L_SHIPMODE as 	 L_SHIPMODE, L_COMMENT as L_COMMENT, L_SHIPDATE as L_SHIPDATE FROM lineitem;

有关更多详细信息，请参阅[分区表](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-PartitionedTables)。

##使用 ORCFile 格式

Hive 支持不同的文件格式。例如：

- **文本**：这是默认的文件格式，适用于大多数情况
- **Avro**：非常适合互操作方案
- **ORC/Parquet**：最适合用于提高性能

ORC（优化行纵栏式）格式是存储 Hive 数据的高效方式。与其他格式相比，ORC 具有以下优点：

- 支持复杂类型（包括 DateTime）和复杂的半结构化类型
- 高达 70% 的压缩率
- 每隔 10,000 行编制索引一次并允许跳过行
- 大幅减少运行时执行时间

若要启用 ORC 格式，请先使用 *Stored as ORC* 子句创建一个表：

    CREATE TABLE lineitem_orc_part
    	(L_ORDERKEY INT, L_PARTKEY INT,L_SUPPKEY INT, L_LINENUMBER INT,
    	 L_QUANTITY DOUBLE, L_EXTENDEDPRICE DOUBLE, L_DISCOUNT DOUBLE,
    	 L_TAX DOUBLE, L_RETURNFLAG STRING, L_LINESTATUS STRING,
    	 L_SHIPDATE_PS STRING, L_COMMITDATE STRING, L_RECEIPTDATE STRING,
		 L_SHIPINSTRUCT STRING, L_SHIPMODE STRING, L_COMMENT 	 STRING)
    PARTITIONED BY(L_SHIPDATE STRING)
    STORED AS ORC;

接下来，从暂存表向 ORC 表插入数据。例如：

    INSERT INTO TABLE lineitem_orc
    SELECT L_ORDERKEY as L_ORDERKEY, 
           L_PARTKEY as L_PARTKEY , 
    	   L_SUPPKEY as L_SUPPKEY,
		   L_LINENUMBER as L_LINENUMBER,
     	   L_QUANTITY as L_QUANTITY, 
		   L_EXTENDEDPRICE as L_EXTENDEDPRICE,
    	   L_DISCOUNT as L_DISCOUNT,
		   L_TAX as L_TAX,
           L_RETURNFLAG as L_RETURNFLAG,
		   L_LINESTATUS as L_LINESTATUS,
		   L_SHIPDATE as L_SHIPDATE,
		   L_COMMITDATE as L_COMMITDATE,
		   L_RECEIPTDATE as L_RECEIPTDATE, 
		   L_SHIPINSTRUCT as L_SHIPINSTRUCT,
		   L_SHIPMODE as L_SHIPMODE,
		   L_COMMENT as L_COMMENT
    FROM lineitem;

可在[此处](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC)阅读有关 ORC 格式的详细信息。

##向量化

向量化可让 Hive 以批的形式同时处理 1024 行，而不是一次处理一行。这意味着，简单的操作可以更快地完成，因为需要运行的内部代码更少。

若要启用向量化，请在 Hive 查询的前面加上以下设置作为前缀：

    set hive.vectorized.execution.enabled = true;

有关详细信息，请参阅[向量化查询执行](https://cwiki.apache.org/confluence/display/Hive/Vectorized+Query+Execution)。


##其他优化方法

你还可以考虑使用其他一些高级优化方法，例如：

- **Hive 装桶：**将大型数据集群集化或分段以优化查询性能的技术。
- **联接优化：**Hive 的查询执行计划优化，可改善联接的效率并减少用户提示的需要。有关详细信息，请参阅[联接优化](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+JoinOptimization#LanguageManualJoinOptimization-JoinOptimization)。
- **增加化简器**

##<a id="nextsteps"></a>后续步骤
在本文中，你已学习了几种常见的 Hive 查询优化方法。若要了解更多信息，请参阅下列文章：

- [使用 HDInsight 中的 Hive 分析航班延误数据](/documentation/articles/hdinsight-analyze-flight-delay-data/)
- [使用 HDInsight 中 Hadoop上的 Hive 查询控制台分析传感器数据](/documentation/articles/hdinsight-hive-analyze-sensor-data/)


[image-hdi-optimize-hive-scaleout_1]: ./media/hdinsight-hadoop-optimize-hive-query-v1/scaleout_1.png
[image-hdi-optimize-hive-scaleout_2]: ./media/hdinsight-hadoop-optimize-hive-query-v1/scaleout_2.png
[image-hdi-optimize-hive-tez_1]: ./media/hdinsight-hadoop-optimize-hive-query-v1/tez_1.png
[image-hdi-optimize-hive-partitioning_1]: ./media/hdinsight-hadoop-optimize-hive-query-v1/partitioning_1.png

<!---HONumber=71-->