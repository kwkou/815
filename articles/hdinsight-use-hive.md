<properties linkid="manage-services-hdinsight-howto-hive" urlDisplayName="Use Hive with HDInsight" pageTitle="Use Hive with HDInsight | Azure" metaKeywords="" description="Learn how to use Hive with HDInsight. You'll use a log file as input into an HDInsight table, and use HiveQL to query the data and report basic statistics." metaCanonical="" services="hdinsight" documentationCenter="" title="Use Hive with HDInsight" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />

# Hive 与 HDInsight 配合使用

[Apache Hive][] 提供了通过类似 SQL 的脚本语言（称作 *HiveQL*）运行 MapReduce 作业的方法，此方法可用于对大量数据进行汇总、查询和分析。在本文中，你将使用 HiveQL 来查询 Apache log4j 日志文件中的数据并报告基本统计信息。

**先决条件：**

-   你必须已经设置了 **HDInsight 群集**。有关如何使用 Azure 门户进行此设置的演练，请参阅 [HDInsight 入门][]。有关可用于创建这种群集的各种其他方法的说明，请参阅[设置 HDInsight 群集][]。

-   你的工作站上必须已经安装了 **Azure PowerShell**。有关如何进行此安装的说明，请参阅[安装和配置 Azure PowerShell][]。

**估计完成时间：** 30 分钟

## 本文内容

-   [Hive 用例][]
-   [将数据文件上载到 Azure Blob 存储][]
-   [使用 PowerShell 运行 Hive 查询][]
-   [后续步骤][]

## Hive 用例

数据库比较适合需要管理可能有低延迟查询的较小数据集的情况。但对于包含以 TB 为单位的数据的大型数据集而言，传统的 SQL 数据库并不是理想的解决方案。数据库管理员习惯上调规模来处理这些较大的数据集，这样就会随着数据库负载增加和性能下降而采购更大型的硬件。

Hive 通过允许用户在查询大型数据集时横向扩展来解决与大数据有关联的问题。Hive 使用 MapReduce 在多个节点间并行查询数据，从而在数量随负载增加而不断增加的主机间分布数据库。

Hive 和 HiveQL 还提供了备用方法，以便在查询数据时用 Java 编写 MapReduce 作业。它提供简单的类似 SQL 的包装器，允许用 HiveQL 编写查询，这些查询之后由 HDInsight 为你编译到 MapReduce 并在群集上运行。

利用 Hive，熟悉 MapReduce 框架的程序员还将能够插入其自定义映射器和简化器，以便执行 HiveQL 语言的内置功能无法支持的更复杂的分析。

Hive 最适合批处理大量不可变数据（例如 Web 日志）。但它不适用于需要非常快的响应速度的事务应用程序，如数据库管理系统。Hive 已针对可伸缩性（可将多台计算机动态添加到 Hadoop 群集）、扩展性（在 MapReduce 框架内，具有其他编程接口）和容错进行了优化。延迟并不是设计时重点要考虑的问题。

一般而言，应用程序会在日志文件中保存错误、异常和其他代码类问题，以便管理员能够使用日志文件中的这些数据来检查可能出现的问题并生成与错误或其他问题（如性能）相关的指标。这些日志文件通常都很大，并且包含着丰富数据，为了提高该应用程序的智能水平，必须对这些数据进行处理和挖掘。

所以，日志文件就是大数据的典型范例。HDInsight 提供了 Hive 数据仓库系统，该系统有助于简化数据摘要、即席查询，以及对 Azure Blob 存储这样的 Hadoop 兼容文件系统中存储的这些大数据集进行分析。

## 将数据文件上载到 Blob 存储

HDInsight 使用 Azure Blob 存储容器作为默认文件系统。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

在本文中，你要使用一个 log4j 示例文件，它是随 *\\example\\data\\sample.log* 中存储的 HDInsight 群集分发的。该文件中的每个日志都包含一行字段，其中包含一个 `[LOG LEVEL]` 字段，可显示类型和严重性。例如：

    2012-02-03 20:26:41 SampleClass3 [ERROR] verbose detail for id 1527353937 

要访问文件，请使用以下语法：

    wasb://<containerName>@<AzureStorageName>.blob.core.chinacloudapi.cn

例如：

    wasb://mycontainer@mystorage.blob.core.chinacloudapi.cn/example/data/sample.log

将 *mycontainer* 替换为容器名称，将 *mystorage* 替换为 Blob 存储帐户名。

由于文件存储在默认文件系统中，你也可以使用以下命令来访问文件：

    wasb:///example/data/sample.log
    /example/data/sample.log

若要生成你自己的 log4j 文件，请使用 [Apache Log4j][] 日志记录实用程序。有关将数据上载到 Azure Blob 存储的信息，请参阅[将数据上载到 HDInsight][]。

## 使用 PowerShell 运行 Hive 查询

在上一节中，你将一个名为 sample.log 的 log4j 文件上载到了默认文件系统容器中。在本节中，你将运行 HiveQL 来创建一个 hive 表，将数据载入到该 hive 表，然后查询这些数据来了解错误日志的数量。

本文提供了有关使用 Azure PowerShell cmdlet 运行 Hive 查询的说明。在学习本节之前，必须先按照本主题开始时**先决条件**一节中的说明，设置本地环境并配置到 Azure 的连接。

使用 **Start-AzureHDInsightJob** cmdlet 或 **Invoke-Hive** cmdlet 均可在 PowerShell 中运行 Hive 查询。

**要使用 Start-AzureHDInsightJob 运行 Hive 查询**

1.  打开 Azure PowerShell 控制台窗口。在[安装和配置 Azure PowerShell][] 中可找到说明。
2.  运行以下命令以连接到 Azure 订阅：

        Add-AzureAccount

    系统将提示你输入 Azure 帐户凭据。

3.  设置以下脚本中的变量并运行它：

        # 提供 Azure 订阅名称，以及用于默认 HDInsight 文件系统的 Azure 存储帐户和容器。
        $subscriptionName = "<SubscriptionName>"
        $storageAccountName = "<AzureStorageAccountName>"
        $containerName = "<AzureStorageContainerName>"

        # 提供要在其中运行该 Hive 作业的 HDInsight 群集的名称
        $clusterName = "<HDInsightClusterName>"

4.  运行以下脚本以定义 HiveQL 查询：

        # HiveQL 查询
        # 使用内部表选项。 
        $queryString = "DROP TABLE log4jLogs;" +
        "CREATE TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ';" +
        "LOAD DATA INPATH 'wasb://$containerName@$storageAccountName.blob.core.chinacloudapi.cn/example/data/sample.log' OVERWRITE INTO TABLE log4jLogs;" +
        "SELECT t4 AS sev, COUNT(*) AS cnt FROM log4jLogs WHERE t4 = '[ERROR]' GROUP BY t4;"

        # 使用外部表选项。 
        $queryString = "DROP TABLE log4jLogs;" +
        "CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION 'wasb://$containerName@$storageAccountName.blob.core.chinacloudapi.cn/example/data/';" +
        "SELECT t4 AS sev, COUNT(*) AS cnt FROM log4jLogs WHERE t4 = '[ERROR]' GROUP BY t4;"

    LOAD DATA HiveQL 命令将导致数据文件移动到 \\hive\\warehouse\\ 文件夹。DROP TABLE 命令将用于删除表和数据文件。如果你使用内部表选项并要再次运行脚本，必须再次上传 sample.log 文件。如果想保留数据文件，必须按该脚本中显示的那样使用 CREATE EXTERNAL TABLE 命令。

    对于数据文件位于不同容器或存储帐户的情况，还可以使用外部表。

    首先使用 DROP TABLE，以防再次运行脚本和 log4jlogs 表已存在。

5.  运行以下脚本来创建 Hive 作业定义：

        # 创建 Hive 作业定义 
        $hiveJobDefinition = New-AzureHDInsightHiveJobDefinition -Query $queryString 

    还可以使用 -File 开关来指定 HDFS 上的 HiveQL 脚本文件。

6.  运行以下脚本来提交 Hive 作业：

        # 将作业提交到群集 
        Select-AzureSubscription $subscriptionName
        $hiveJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $hiveJobDefinition

7.  运行以下脚本来等待 Hive 作业完成：

        # 等待 Hive 作业完成
        Wait-AzureHDInsightJob -Job $hiveJob -WaitTimeoutInSeconds 3600

8.  运行以下脚本以打印标准输出结果：
9.  # 打印 Hive 作业的标准错误和标准输出结果。
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $hiveJob.JobId -StandardOutput

    ![HDI.HIVE.PowerShell][]

    结果为：

        [ERROR] 3

**使用 Invoke-Hive 提交 Hive 查询**

1.  打开 Azure PowerShell 控制台窗口。
2.  运行以下命令以连接到 Azure 订阅：

        Add-AzureAccount

    系统将提示你输入 Azure 帐户凭据。

3.  设置该变量，然后运行它：

        $clusterName = "<HDInsightClusterName>"

4.  运行以下脚本以调用 HiveQL 查询：

        Use-AzureHDInsightCluster $clusterName 
        $response = Invoke-Hive -Query @"
        SELECT * FROM hivesampletable
        WHERE devicemake LIKE "HTC%"
        LIMIT 10; 
        "@

        Write-Host $response

    输出为：

    ![PowerShell Invoke-Hive 输出结果][]

    对于比较长的 HiveQL 查询，推荐使用 PowerShell Here-Strings 或 HiveQL 脚本文件。下面的示例显示了如何使用 Invoke-Hive cmdlet 来运行 HiveQL 脚本文件。HiveQL 脚本文件必须上载到 WASB。

        Invoke-Hive -File "wasb://<ContainerName>@<StorageAccountName>/<Path>/query.hql"

    有关 Here-Strings 的详细信息，请参阅[使用 Windows PowerShell Here-Strings][]。

## 后续步骤

Hive 可以简化使用类似 SQL 的查询语言进行数据查询的步骤，而随 HDInsight 提供的其他组件也提供了数据移动和转换这样的补充功能。若要了解更多信息，请参阅下列文章：

-   [Azure HDInsight 入门][HDInsight 入门]
-   [使用 HDInsight 分析航班延误数据][]
-   [将 Oozie 与 HDInsight 配合使用][]
-   [以编程方式提交 Hadoop 作业][]
-   [将数据上传到 HDInsight][将数据上载到 HDInsight]
-   [Pig 与 HDInsight 配合使用][]
-   [Azure HDInsight SDK 文档][]

  [Apache Hive]: http://hive.apache.org/
  [HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [设置 HDInsight 群集]: /en-us/manage/services/hdinsight/provision-hdinsight-clusters/
  [安装和配置 Azure PowerShell]: /en-us/documentation/articles/install-configure-powershell/
  [Hive 用例]: #usage
  [将数据文件上载到 Azure Blob 存储]: #uploaddata
  [使用 PowerShell 运行 Hive 查询]: #runhivequeries
  [后续步骤]: #nextsteps
  [将 Azure Blob 存储与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/howto-blob-store
  [Apache Log4j]: http://en.wikipedia.org/wiki/Log4j
  [将数据上载到 HDInsight]: /en-us/manage/services/hdinsight/howto-upload-data-to-hdinsight/
  [HDI.HIVE.PowerShell]: ./media/hdinsight-use-hive/HDI.HIVE.PowerShell.png
  [PowerShell Invoke-Hive 输出结果]: ./media/hdinsight-use-hive/HDI.Hive.PowerShell.Output.png
  [使用 Windows PowerShell Here-Strings]: http://technet.microsoft.com/en-us/library/ee692792.aspx
  [使用 HDInsight 分析航班延误数据]: /en-us/documentation/articles/hdinsight-analyze-flight-delay-data/
  [将 Oozie 与 HDInsight 配合使用]: /en-us/documentation/articles/hdinsight-use-oozie/
  [以编程方式提交 Hadoop 作业]: /en-us/manage/services/hdinsight/submit-hadoop-jobs-programmatically/
  [Pig 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-pig-with-hdinsight/
  [Azure HDInsight SDK 文档]: http://msdn.microsoft.com/zh-cn/library/dn469975.aspx
