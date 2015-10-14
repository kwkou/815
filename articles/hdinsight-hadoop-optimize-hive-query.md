<properties
   pageTitle="优化 Hive 查询以便在 HDInsight 中更快地执行 | Windows Azure"
   description="了解如何为 HDInsight 中的 Hadoop 优化 Hive 查询。"
   services="hdinsight"
   documentationCenter=""
   authors="rashimg"
   manager="mwinkle"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
   ms.service="hdinsight"
   ms.date="07/28/2015"
   wacn.date="10/03/2015"/>


# 了解如何使用 HDInsight 优化 Hive 查询

我们的大多数客户都在使用 Hive 作为在 HDInsight 中执行查询的接口。Hive 既可用于即席查询，也能在管道中使用。

最常见的问题之一是，如何更快地运行查询。由于 Hadoop 必须满足不同规模的用户需求（从小型数据集到 PB 量级的数据查询），现成的 Hadoop 已经过优化，以确保查询能够成功完成执行 - 但速度不一定更快。

如果你已开始阅读有关 Hive 优化的主题，很快就会发现，你可以使用几十种甚至几百种优化方法。在本教程中，我们将探讨可以用来加快查询运行速度的最常用优化方法。我们将从体系结构和查询级别进行讲解。

##优化方法 1：扩大

扩大是指增加群集中可部署的节点数。之所以增加节点能够带来帮助，是因为你可以并行运行更多的任务，从而可以运行更多的映射器和化简器。在 HDInsight 中，可通过两种方式增加扩大的数目：

1. 在创建群集的过程中，可以选择要在多少个节点中运行查询。下图显示了这种方法。

	![scaleout\_1][image-hdi-optimize-hive-scaleout_1]

2. 如果你已启动并运行了群集，则无需删除然后重新创建该群集，也能增加扩展数目。如下所示。

	![scaleout\_1][image-hdi-optimize-hive-scaleout_2]

有关 HDInsight 支持的不同 VM 的详细信息，请单击[价格](/home/features/hdinsight/#price)链接。

##优化方法 2：启用 Tez

[Apache Tez](http://hortonworks.com/hadoop/tez/) 是 MapReduce 的备用执行引擎，可使 Hive 具有交互性。Tez 不仅能够提高 MapReduce 引擎的速度，而且还能保留 MapReduce 扩展到 PB 数据量级的能力。Apache Hive 和 Apache Pig 使用 Tez。下图显示了 Hive 可以在 HDInsight 平台中的 MapReduce 或交互式 Tez 上运行。

![tez\_1][image-hdi-optimize-hive-tez_1]

Tez 的速度比 MapReduce 引擎更快，原因如下：

1. 以单个作业执行定向无圈图 (DAG) - 在 MapReduce 引擎中，表达的 DAG 要求每组映射器后接一组化简器。这会导致针对每个 Hive 查询运行多个 MapReduce 作业。Tez 没有这种局限性，它可以将复杂的 DAG 作为一个作业进行处理，因此将作业启动开销降到最低。
2. 避免不必要的写入 - 由于要为 MapReduce 引擎的同一 Hive 查询运行多个作业，每个作业的输出将写入 HDFS 以用作暂存数据。而 Tez 最大程度地减少了对每个 Hive 查询运行的作业数，因此能够避免不必要的写入。
3. 最大程度地降低启动延迟 - Tez 可以减少需要启动的映射器数目，同时还能提高优化吞吐量，因此，更有利于最小化启动延迟。
4. 重复使用容器 - Tez 会尽可能地重复使用容器，以确保降低由于启动容器而产生的延迟。
5. 连续优化技术 - 传统上，优化是在编译阶段完成的。但是，这可以提供有关输入的详细信息，以便在运行时更好地进行优化。Tez 使用连续优化技术，从而可以在运行时阶段进一步优化计划。

有关这些概念的详细信息，请单击[此处](http://hortonworks.com/hadoop/tez/)

你可以通过在查询的前面加上以下设置作为前缀，来执行 Tez 支持的任何 Hive 查询：

`set hive.execution.engine=tez;`

如果你希望 Tez 按默认支持你的群集，可以使用以下 PowerShell 脚本创建该群集。请注意，此脚本只在创建群集时才起作用，而并不可用于将已在运行的群集转换为 Tez 按默认支持的群集。

    $location = "China East"  [Change this to location of your cluster]

    $storageAccountName = "Your_Blob_Storage_name"   [Change this to your blob storage name]
    $storageContainerName = "Your_Blob_Storage_Container_name" [Change this to your storage container name]

    $hdiDataNodes = 32 [Change this to number of nodes you want]
    $hdiName = "Your_Cluster_Name"    [Change this to the name of your cluster. Make sure this is unique]
    $hdiVersion = "3.1" [Choose version of HDI cluster]

    $hdiUserName = "username" [Choose a username]
    $hdiPassword = "password" [Choose password - make sure it is 8 characters, has atleast 1 number, 1 special character and 1 uppercase letter]  

    $storageAccountKey = "Your_storage_key" [List the storage key associated with your storage account above]

    $hdiSecurePassword = ConvertTo-SecureString $hdiPassword -AsPlainText -Force
    $hdiCredential = New-Object System.Management.Automation.PSCredential($hdiUserName, $hdiSecurePassword)

    $hiveConfig = new-object 'Microsoft.WindowsAzure.Management.HDInsight.Cmdlet.DataObjects.AzureHDInsightHiveConfiguration'
    $hiveConfig.Configuration = @{ "hive.execution.engine"="tez" }

    New-AzureHDInsightClusterConfig -ClusterSizeInNodes $hdiDataNodes -HeadNodeVMSize Standard_D14 -DataNodeVMSize Standard_D14 |
    Set-AzureHDInsightDefaultStorage -StorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey -StorageContainerName $storageContainerName |
    Add-AzureHDInsightConfigValues -Hive $hiveConfig |
    New-AzureHDInsightCluster -Name $hdiName -Location $location -Credential $hdiCredential -Version $hdiVersion -Debug

##优化方法 3：分区

默认情况下，Hive 查询会扫描整个表。对于“表扫描”之类的查询，这非常有利；但是，对于只需扫描少量数据的查询（例如，使用筛选进行查询），这会不必要地延长运行时间。由于 Hive 查询的大部分瓶颈都是 I/O 操作造成的，因此，如果可以减少需要读取的数据量，则会降低查询的总体延迟。

Hive 分区利用了此方法。分区可使 Hive 访问一部分数据，以便只会处理必要的数据量。Hive 中实现这种优势的途径是，重新组织原始数据，以便为每个分区创建一个新目录 - 用户将在此目录中定义一个分区。例如，假设我们要根据“年份”列为表分区。那么，将为每个年份创建一个新目录。下图详细演示了分区的概念。

![partitioning][image-hdi-optimize-hive-partitioning_1]

关于分区的某些要点：1.不要分区不足 - 根据包含少量值的列进行分区可能会导致创建很少的分区，以致无法减少需要读取的数据量。例如，根据性别分区只会创建两个分区 - 一个针对男性，一个针对女性 - 最多只会将延迟降低一半。2.不要过度分区 - 另一种极端是，根据包含唯一值（例如 userid）的列进行分区会导致创建多个分区，从而给命名节点带来很大的压力，因为它必须处理大量的目录。这是另一种极端做法。3.避免数据偏斜 - 明智选择分区键，以便所有分区的大小均等。例如，根据“州”分区可能会导致“加利福尼亚州”的记录数几乎是“佛蒙特州”的 30 倍，因为两个州的人口有差异。

若要创建分区表，请使用以下代码段中所示的 *Partitioned By* 子句。

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

静态分区表示已在相应目录中创建了分片数据，你可以请求根据目录位置在 Hive 中手动分区。以下代码段对此做了演示。

    INSERT OVERWRITE TABLE lineitem_part
    PARTITION (L_SHIPDATE = ‘5/23/1996 12:00:00 AM’)
    SELECT * FROM lineitem 
    WHERE lineitem.L_SHIPDATE = ‘5/23/1996 12:00:00 AM’

    ALTER TABLE lineitem_part ADD PARTITION (L_SHIPDATE = ‘5/23/1996 12:00:00 AM’))
    LOCATION ‘wasb://sampledata@ignitedemo.blob.core.chinacloudapi.cn/partitions/5_23_1996/'

动态分区表示你希望 Hive 自动为你创建分区。由于我们已经基于暂存表创建了分区表，我们需要做的就是将数据插入分区表，如下所示：

    SET hive.exec.dynamic.partition = true;
    SET hive.exec.dynamic.partition.mode = nonstrict;
    INSERT INTO TABLE lineitem_part
    PARTITION (L_SHIPDATE)
    SELECT L_ORDERKEY as L_ORDERKEY, L_PARTKEY as L_PARTKEY , 
    	 L_SUPPKEY as L_SUPPKEY, L_LINENUMBER as L_LINENUMBER,
     	 L_QUANTITY as L_QUANTITY, L_EXTENDEDPRICE as L_EXTENDEDPRICE,
    	 L_DISCOUNT as L_DISCOUNT, L_TAX as L_TAX, L_RETURNFLAG as 	 	 L_RETURNFLAG, L_LINESTATUS as L_LINESTATUS, L_SHIPDATE as 	 	 L_SHIPDATE_PS, L_COMMITDATE as L_COMMITDATE, L_RECEIPTDATE as 	 L_RECEIPTDATE, L_SHIPINSTRUCT as L_SHIPINSTRUCT, L_SHIPMODE as 	 L_SHIPMODE, L_COMMENT as L_COMMENT, L_SHIPDATE as L_SHIPDATE FROM lineitem;

有关分区的更多详细信息，请单击[此处](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-PartitionedTables)。

##优化方法 4：使用 ORCFile 格式
Hive 支持使用不同的文件格式来执行查询。下面是其中的一部分格式及其最佳用例：


- 文本格式：这是默认的文件格式，适用于大多数情况
- Avro 格式：非常适合互操作方案
- ORC/Parquet：最适合用于提高性能

ORC（优化行纵栏式）格式是存储 Hive 数据的高效方式。与其他格式相比，ORC 具有以下优点：支持复杂类型（包括 DateTime）和复杂的半结构化类型 - 高达 70% 的压缩率 - 每隔 10,000 行编制索引一次并允许跳过行

可在[此处](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC)阅读有关 ORC 格式的详细信息。

若要启用 ORC 格式，请先使用 *Stored as ORC* 子句创建一个表，如下所示

    CREATE TABLE lineitem_orc_part
    	(L_ORDERKEY INT, L_PARTKEY INT,L_SUPPKEY INT, L_LINENUMBER INT,
    	 L_QUANTITY DOUBLE, L_EXTENDEDPRICE DOUBLE, L_DISCOUNT DOUBLE,
    	 L_TAX DOUBLE, L_RETURNFLAG STRING, L_LINESTATUS STRING,
    	 L_SHIPDATE_PS STRING, L_COMMITDATE STRING, L_RECEIPTDATE 	 	 STRING, L_SHIPINSTRUCT STRING, L_SHIPMODE STRING, L_COMMENT 	 STRING)
    PARTITIONED BY(L_SHIPDATE STRING)
    STORED AS ORC;

接下来，使用以下代码从暂存表向 ORC 表插入数据：

    INSERT INTO TABLE lineitem_orc
    SELECT L_ORDERKEY as L_ORDERKEY, L_PARTKEY as L_PARTKEY , 
    	 L_SUPPKEY as L_SUPPKEY, L_LINENUMBER as L_LINENUMBER,
     	 L_QUANTITY as L_QUANTITY, L_EXTENDEDPRICE as L_EXTENDEDPRICE,
    	 L_DISCOUNT as L_DISCOUNT, L_TAX as L_TAX, L_RETURNFLAG as 	 	 L_RETURNFLAG, L_LINESTATUS as L_LINESTATUS, L_SHIPDATE as 	 	 L_SHIPDATE, L_COMMITDATE as L_COMMITDATE, L_RECEIPTDATE as 	 	 L_RECEIPTDATE, L_SHIPINSTRUCT as L_SHIPINSTRUCT, L_SHIPMODE as 	 L_SHIPMODE, L_COMMENT as L_COMMENT
    FROM lineitem;

使用 ORC 时，你应会看到，数据压缩率大约为 50-80%，同时，运行时执行时间会明显下降。

##优化方法 5：向量化
向量化是 Hive 的一项功能，可以减少常见查询操作的 CPU 使用率。向量化不是一次处理一行，而是一次性处理 1024 行。这意味着，简单的操作可以更快地完成，因为需要运行的内部代码更少。可以在[此处](https://cwiki.apache.org/confluence/display/Hive/Vectorized+Query+Execution)阅读有关向量化的详细信息。

若要启用向量化，请在 Hive 查询的前面加上以下设置作为前缀：

    set hive.vectorized.execution.enabled = true;


##其他优化方法

你还可以考虑使用其他一些高级优化方法，其中包括装桶、联接和增加化简器，我们将在以后的文章中予以介绍。

##摘要
总之，你应该根据具体的情景，考虑使用上述优化方法来加快查询运行速度。

HDInsight 正在致力于进一步优化 Hadoop，且不会明显增大贵方的工作量。一旦我们有分享的内容，就会在本教程中更新更多的详情。

[image-hdi-optimize-hive-scaleout_1]: ./media/hdinsight-hadoop-optimize-hive-query/scaleout_1.png
[image-hdi-optimize-hive-scaleout_2]: ./media/hdinsight-hadoop-optimize-hive-query/scaleout_2.png
[image-hdi-optimize-hive-tez_1]: ./media/hdinsight-hadoop-optimize-hive-query/tez_1.png
[image-hdi-optimize-hive-partitioning_1]: ./media/hdinsight-hadoop-optimize-hive-query/partitioning_1.png

<!---HONumber=71-->