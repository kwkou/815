<properties
    pageTitle="了解什么是 Hive 以及如何使用 HiveQL | Azure"
    description="了解 Apache Hive 以及如何将它与 HDInsight 中的 Hadoop 配合使用。选择如何运行 Hive 作业，以及如何使用 HiveQL 分析示例 Apache log4j 文件。"
    keywords="hiveql,什么是 hive"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="2c10f989-7636-41bf-b7f7-c4b67ec0814f"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/12/2017"
    wacn.date="01/25/2017"
    ms.author="larryfr" />

# 将 Hive 和 HiveQL 与 HDInsight 中的 Hadoop 配合使用以分析示例 Apache log4j 文件
[AZURE.INCLUDE [hive-selector](../../includes/hdinsight-selector-use-hive.md)]

本教程介绍了如何在 HDInsight 上使用 Hadoop 中的 Apache Hive，以及选择如何运行 Hive 作业。此外，还将介绍 HiveQL 以及如何分析一个示例 Apache log4j 文件。

## <a id="why"></a>什么是 Hive，为何要使用它？
[Apache Hive](http://hive.apache.org/) 是适用于 Hadoop 的数据仓库系统，可以实现通过 HiveQL（类似于 SQL 的查询语言）来进行数据汇总、查询和数据分析。Hive 可用于以交互方式浏览数据，或者创建可重用的批处理作业。

Hive 可以实现将结构投影到很大程度上未结构化的数据上。在定义结构后，就算不了解 Java 或 MapReduce，也可以使用 Hive 来查询这些数据。通过 **HiveQL**（Hive 查询语言），你可以使用类似于 T-SQL 的语句编写查询。

Hive 知道如何处理结构化和半结构化数据，例如字段以特定字符分隔的文本文件。Hive 还支持适用于复杂或不规则的结构化数据的自定义**序列化程序/反序列化程序 (SerDe)**。有关详细信息，请参阅[如何将自定义 JSON SerDe 与 HDInsight 配合使用](http://blogs.msdn.com/b/bigdatasupport/archive/2014/06/18/how-to-use-a-custom-json-serde-with-microsoft-azure-hdinsight.aspx)。

## 用户定义的函数 (UDF)
还可以通过**用户定义的函数 (UDF)** 扩展 Hive。通过 UDF 可实现 HiveQL 中不容易建模的功能或逻辑。有关将 UDF 与 Hive 配合使用的示例，请参阅：

* [将 Java 用户定义的函数与 Hive 配合使用](/documentation/articles/hdinsight-hadoop-hive-java-udf/)
* [在 HDInsight 中将 Python 与 Hive 和 Pig 配合使用](/documentation/articles/hdinsight-python/)
* [在 HDInsight 中将 C# 与 Hive 和 Pig 配合使用](/documentation/articles/hdinsight-hadoop-hive-pig-udf-dotnet-csharp/)
* [如何将自定义 Hive UDF 添加到 HDInsight](http://blogs.msdn.com/b/bigdatasupport/archive/2014/01/14/how-to-add-custom-hive-udfs-to-hdinsight.aspx)
* [将日期/时间格式转换为 Hive 时间戳的自定义 Hive UDF 示例](https://github.com/Azure-Samples/hdinsight-java-hive-udf)

## Hive 内部表与外部表
以下是你需要了解的有关 Hive 内部表和外部表的一些信息：

* **CREATE TABLE** 命令创建内部表。数据文件必须位于默认容器中。
* **CREATE TABLE** 命令将数据文件移动到 /hive/warehouse/<表名> 文件夹中。
* **CREATE EXTERNAL TABLE** 命令创建外部表。数据文件可以位于默认容器以外的位置。
* **CREATE EXTERNAL TABLE** 命令不移动数据文件。
* **CREATE EXTERNAL TABLE** 命令不允许在 LOCATION 中有任何文件夹。这是本教程生成 sample.log 文件副本的原因。

有关详细信息，请参阅 [HDInsight：Hive 内部表和外部表简介][cindygross-hive-tables]。

## <a id="data"></a>关于示例数据（一个 Apache log4j 文件）
本示例使用 *log4j* 示例文件，该文件存储在 Blob 存储容器的 **/example/data/sample.log** 中。该文件中的每个日志都由一行字段组成，其中包含一个用于显示类型和严重性的 `[LOG LEVEL]` 字段，例如：

    2012-02-03 20:26:41 SampleClass3 [ERROR] verbose detail for id 1527353937

在前面的示例中，日志级别为 ERROR。

> [AZURE.NOTE]
也可以使用 [Apache Log4j](http://zh.wikipedia.org/wiki/Log4j) 日志记录工具生成 log4j 文件，然后将该文件上传到 Blob 容器。请参阅[将数据上传到 HDInsight](/documentation/articles/hdinsight-upload-data/) 以了解相关说明。有关如何将 Azure Blob 存储与 HDInsight 配合使用的详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-use-blob-storage/)。
> 
> 

示例数据存储在 Azure Blob 存储中，HDInsight 将其用作默认文件系统。HDInsight 可以使用 **wasb** 前缀来访问存储在 Blob 中的文件。例如，若要访问 sample.log 文件，可使用以下语法：

    wasbs:///example/data/sample.log

Azure Blob 存储是 HDInsight 的默认存储，因此也可以使用 HiveQL 中的 **/example/data/sample.log** 访问该文件。

> [AZURE.NOTE]
语法 **wasbs:///** 用于访问存储在 HDInsight 群集的默认存储容器中的文件。如果你在预配群集时指定了其他存储帐户，并且想要访问存储在这些帐户中的文件，可以通过指定容器名称和存储帐户地址来访问数据，例如： **wasbs://mycontainer@mystorage.blob.core.chinacloudapi.cn/example/data/sample.log**。
> 
> 

## <a id="job"></a>示例作业：将列投影到分隔数据
以下 HiveQL 语句将列投影到 **wasbs:///example/data** 目录中存储的分隔数据：

    set hive.execution.engine=tez;
    DROP TABLE log4jLogs;
    CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';
    SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;

在上例中，HiveQL 语句执行以下操作：

* **set hive.execution.engine=tez;**：设置执行引擎以使用 Tez。使用 Tez 而不是 MapReduce 可以提高查询性能。有关 Tez 的详细信息，请参阅[使用 Apache Tez 提高性能](#usetez)部分。
  
    > [AZURE.NOTE]
    只有在使用基于 Windows 的 HDInsight 群集时，才需要此语句；对于基于 Linux 的 HDInsight，Tez 是默认的执行引擎。
    > 
    > 
* **DROP TABLE**：删除表和数据文件（如果该表已存在）。
* **CREATE EXTERNAL TABLE**：在 Hive 中创建新的**外部**表。外部表只会在 Hive 中存储表定义；数据以原始格式保留在原始位置。
* **ROW FORMAT**：告知 Hive 如何设置数据的格式。在此情况下，每个日志中的字段以空格分隔。
* **STORED AS TEXTFILE LOCATION**：告知 Hive 数据的存储位置（example/data 目录），以及数据已存储为文本。数据可以在一个文件中，也可以分散在目录的多个文件内。
* **SELECT**：选择 **t4** 列表包含值 **[ERROR]** 的所有行的计数。这应会返回值 **3**，因为有三个行包含此值。
* **INPUT\_\_FILE\_\_NAME LIKE '%.log'** - 告诉 Hive，我们只应返回以 .log 结尾的文件中的数据。此项将搜索限定于包含数据的 sample.log 文件，并阻止搜索返回与所定义架构不匹配的其他示例数据文件中的数据。

> [AZURE.NOTE]
当你预期以外部源更新基础数据（例如自动化数据上载过程），或以其他 MapReduce 操作更新基础数据，但希望 Hive 查询始终使用最新数据时，必须使用外部表。
><p> 
> 删除外部表**不会**删除数据，只会删除表定义。
> 
> 

创建外部表后，使用以下语句创建**内部**表。

    set hive.execution.engine=tez;
    CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
    STORED AS ORC;
    INSERT OVERWRITE TABLE errorLogs
    SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]';

这些语句可执行以下操作：

* **CREATE TABLE IF NOT EXISTS**：创建表（如果该表尚不存在）。由于未使用 **EXTERNAL** 关键字，因此这是一个内部表，存储在 Hive 数据仓库中并完全受 Hive 管理。
* **STORED AS ORC**：以优化行纵栏表 (ORC) 格式存储数据。这是高度优化且有效的 Hive 数据存储格式。
* **INSERT OVERWRITE ...SELECT**：从包含 **[ERROR]** 的 **log4jLogs** 表中选择行，然后将数据插入 **errorLogs** 表中。

> [AZURE.NOTE]
与外部表不同，删除内部表会同时删除基础数据。
> 
> 

## <a id="usetez"></a>使用 Apache Tez 提高性能
[Apache Tez](http://tez.apache.org) 是让数据密集型应用程序（例如 Hive）能够大规模高效运行的框架。在最新版的 HDInsight 中，Hive 支持在 Tez 上运行。默认情况下，已经为基于 Linux 的 HDInsight 群集启用了 Tez。

> [AZURE.NOTE]
对于基于 Windows 的 HDInsight 群集来说，Tez 目前默认处于关闭状态，因此必须启用。若要充分利用 Tez，你必须设置 Hive 查询的以下值：
><p> 
> ```set hive.execution.engine=tez;```  

><p> 
> 你可为每个查询提交此值，只需将它放置在查询的开头即可。你也可以在创建群集时设置配置值，而在群集上将此值默认为打开。可以在[预配 HDInsight 群集](/documentation/articles/hdinsight-provision-clusters/)中找到详细信息。
> 
> 

[Tez 上的 Hive 设计文档](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez)包含有关实现选项和优化配置的详细信息。

为了帮助使用 Tez 调试运行的作业，HDInsight 提供了以下 Web UI，用于查看 Tez 作业的详细信息：

* [在基于 Windows 的 HDInsight 上使用 Tez UI](/documentation/articles/hdinsight-debug-tez-ui/)
* [Use the Ambari Tez view on Linux-based HDInsight（在基于 Linux 的 HDInsight 上使用 Ambari Tez 视图）](/documentation/articles/hdinsight-debug-ambari-tez-view/)

## <a id="run"></a>选择如何运行 HiveQL 作业
HDInsight 可以使用各种方法运行 HiveQL 作业。使用下表来确定哪种方法最适合你，然后访问此链接进行演练。

| **使用此方法**，如果想要... | ...**交互式** shell | ...**批处理** | ...使用此**群集操作系统** | ...从此**客户端操作系统** |
|:--- |:---:|:---:|:--- |:--- |
| [Hive 视图](/documentation/articles/hdinsight-hadoop-use-hive-ambari-view/) |✔ |✔ |Linux |任何（基于浏览器） |
| [Beeline 命令（来自 SSH 会话）](/documentation/articles/hdinsight-hadoop-use-hive-beeline/) |✔ |✔ |Linux |Linux、Unix、Mac OS X 或 Windows |
| [Hive 命令（来自 SSH 会话）](/documentation/articles/hdinsight-hadoop-use-hive-ssh/) |✔ |✔ |Linux |Linux、Unix、Mac OS X 或 Windows |
| [Curl](/documentation/articles/hdinsight-hadoop-use-hive-curl/) |&nbsp; |✔ |Linux 或 Windows |Linux、Unix、Mac OS X 或 Windows |
| [查询控制台](/documentation/articles/hdinsight-hadoop-use-hive-query-console/) |&nbsp; |✔ |Windows |任何（基于浏览器） |
| [HDInsight tools for Visual Studio](/documentation/articles/hdinsight-hadoop-use-hive-visual-studio/) |&nbsp; |✔ |Linux 或 Windows |Windows |
| [Windows PowerShell](/documentation/articles/hdinsight-hadoop-use-hive-powershell/) |&nbsp; |✔ |Linux 或 Windows |Windows |
| [远程桌面](/documentation/articles/hdinsight-hadoop-use-hive-remote-desktop/) |✔ |✔ |Windows |Windows |

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
Linux 是在 HDInsight 3.4 或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

## 使用本地 SQL Server Integration Services 在 Azure HDInsight 上运行 Hive 作业
也可以使用 SQL Server Integration Services (SSIS) 运行 Hive 作业。Azure Feature Pack for SSIS 提供以下组件，用于 HDInsight 上的 Hive 作业。

* [Azure HDInsight Hive 任务][hivetask]
* [Azure 订阅连接管理器][connectionmanager]

在[此处][ssispack]了解有关 Azure Feature Pack for SSIS 的详细信息。

## <a id="nextsteps"></a>后续步骤
现在，你已了解什么是 Hive，以及如何将它与 HDInsight 中的 Hadoop 配合使用，请使用以下链接来学习 Azure HDInsight 的其他用法。

* [将数据上传到 HDInsight][hdinsight-upload-data]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
* [将 Sqoop 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-sqoop/)
* [将 Oozie 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-oozie/)
* [将 MapReduce 作业与 HDInsight 配合使用][hdinsight-use-mapreduce]

[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx

[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/

[apache-tez]: http://tez.apache.org
[apache-hive]: http://hive.apache.org/
[apache-log4j]: http://zh.wikipedia.org/wiki/Log4j
[hive-on-tez-wiki]: https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez
[import-to-excel]: /documentation/articles/hdinsight-connect-excel-power-query/
[hivetask]: http://msdn.microsoft.com/zh-cn/library/mt146771(v=sql.120).aspx
[connectionmanager]: http://msdn.microsoft.com/zh-cn/library/mt146773(v=sql.120).aspx
[ssispack]: http://msdn.microsoft.com/zh-cn/library/mt146770(v=sql.120).aspx

[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/
[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/
[hdinsight-analyze-flight-data]: /documentation/articles/hdinsight-analyze-flight-delay-data/
[hdinsight-use-mapreduce]: /documentation/articles/hdinsight-use-mapreduce/

[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/

[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/

[Powershell-install-configure]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs
[powershell-here-strings]: http://technet.microsoft.com/zh-cn/library/ee692792.aspx

[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->