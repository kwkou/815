<properties
    pageTitle="什么是 Apache Hive 和 HiveQL- Azure HDInsight | Azure"
    description="Apache Hive 是适用于 Hadoop 的数据仓库系统。 可以使用 HiveQL（类似于 Transact-SQL）查询 Hive 中存储的数据。 本文档介绍了如何在 Azure HDInsight 中使用 Hive 和 HiveQL。"
    keywords="hiveql,什么是 hive,hadoop hiveql,如何使用 hive,了解 hive,hive 是什么"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="2c10f989-7636-41bf-b7f7-c4b67ec0814f"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="05/12/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="88f72151e94d825dd159e1d932eac312df0d1875"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="what-is-apache-hive-and-hiveql-on-azure-hdinsight"></a>什么是 Azure HDInsight 中的 Apache Hive 和 HiveQL

[Apache Hive](http://hive.apache.org/) 是适用于 Hadoop 的数据仓库系统。 使用 Hive 可以汇总、查询和分析数据。 Hive 查询使用 HiveQL 编写，它是类似于 SQL 的查询语言。

Hive 允许你在很大程度上未结构化的数据上投影结构。 定义结构后，可以使用 HiveQL 来查询这些数据，而无需具备 Java 或 MapReduce 方面的知识。

HDInsight 提供了已针对特定工作负荷进行了优化的多种群集类型。 以下群集类型最常用于 Hive 查询：

* __交互式 Hive__：提供[低延迟分析处理 (LLAP)](https://cwiki.apache.org/confluence/display/Hive/LLAP) 功能的 Hadoop 群集，可改善交互式查询的响应时间。 有关详细信息，请参阅 [HDInsight 中的交互式 Hive 入门](/documentation/articles/hdinsight-hadoop-use-interactive-hive/)文档。

* __Hadoop__：针对批处理工作负荷进行优化的 Hadoop 群集。 有关详细信息，请参阅 [HDInsight 中的 Hadoop 入门](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)文档。

* __Spark__：Apache Spark 提供了用于使用 Hive 的内置功能。 有关详细信息，请参阅 [Spark on HDInsight 入门](/documentation/articles/hdinsight-apache-spark-jupyter-spark-sql/)文档。

* __HBase__：HiveQL 可用于查询 HBase 中存储的数据。 有关详细信息，请参阅 [HBase on HDInsight 入门](/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/)文档。

## <a name="how-to-use-hive"></a>如何使用 Hive

使用下表来了解如何将 Hive 与 HDInsight 配合使用：

| **使用此方法**，如果想要... | ... **交互式** shell | ...**批处理** | ...使用此 **群集操作系统** | ...从此 **客户端操作系统** |
|:--- |:---:|:---:|:--- |:--- |
| [Hive 视图](/documentation/articles/hdinsight-hadoop-use-hive-ambari-view/) |✔ |✔ |Linux |任何（基于浏览器） |
| [Beeline 客户端](/documentation/articles/hdinsight-hadoop-use-hive-beeline/) |✔ |✔ |Linux |Linux、Unix、Mac OS X 或 Windows |
| [REST API](/documentation/articles/hdinsight-hadoop-use-hive-curl/) |&nbsp; |✔ |Linux 或 Windows* |Linux、Unix、Mac OS X 或 Windows |
| [用于 Visual Studio 的 HDInsight 工具](/documentation/articles/hdinsight-hadoop-use-hive-visual-studio/) |&nbsp; |✔ |Linux 或 Windows* |Windows |
| [Windows PowerShell](/documentation/articles/hdinsight-hadoop-use-hive-powershell/) |&nbsp; |✔ |Linux 或 Windows* |Windows |

> [AZURE.IMPORTANT]
> \* Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 3.3 弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。
><p>
> 如果使用基于 Windows 的 HDInsight 群集，可以通过浏览器或[远程桌面](/documentation/articles/hdinsight-hadoop-use-hive-remote-desktop/)使用[查询控制台](/documentation/articles/hdinsight-hadoop-use-hive-query-console/)来运行 Hive 查询。

## <a name="hiveql-language-reference"></a>HiveQL 语言参考

[语言手册 (https://cwiki.apache.org/confluence/display/Hive/LanguageManual)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual) 中提供了 HiveQL 语言参考。

## <a name="hive-and-data-structure"></a>Hive 和数据结构

Hive 知道如何处理结构化和半结构化数据。 例如，其中的字段由特定字符分隔的文本文件。 以下 HiveQL 语句基于空格分隔的数据创建一个表：

    CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE LOCATION '/example/data/';

Hive 还支持对复杂或不规则的结构化数据使用自定义**序列化程序/反序列化程序 (SerDe)**。 有关详细信息，请参阅[如何将自定义 JSON SerDe 与 HDInsight 配合使用](http://blogs.msdn.com/b/bigdatasupport/archive/2014/06/18/how-to-use-a-custom-json-serde-with-microsoft-azure-hdinsight.aspx)文档。

有关 Hive 支持的文件格式的详细信息，请参阅[语言手册 (https://cwiki.apache.org/confluence/display/Hive/LanguageManual)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual)

## <a name="hive-internal-tables-vs-external-tables"></a>Hive 内部表与外部表

使用 Hive 可以创建两种类型的表：

* __内部__：数据存储在 Hive 数据仓库中。 数据仓库位于群集的默认存储中的 `/hive/warehouse/` 上。

    在以下情况下使用内部表：

    * 数据是临时性的。
    * 希望 Hive 管理表和数据的生命周期。

* __外部__：数据存储在数据仓库外部。 数据可以存储在可由群集访问的任何存储中。

    在以下情况下使用外部表：

    * 数据也在 Hive 外部使用。 例如，另一个进程（不会锁定文件）会更新数据文件。
    * 数据需要保留在基础位置，即使在删除表之后。
    * 需要一个自定义位置，例如非默认存储帐户。
    * 由 Hive 以外的某个程序管理数据格式、位置等。

有关详细信息，请参阅 [Hive 内部和外部表简介][cindygross-hive-tables]博客文章。

## <a name="user-defined-functions-udf"></a>用户定义函数 (UDF)

还可以通过**用户定义函数 (UDF)** 扩展 Hive。 UDF 允许你实现 HiveQL 中不容易建模的功能或逻辑。 有关将 UDF 与 Hive 配合使用的示例，请参阅以下文档：

* [将 Java 用户定义函数与 Hive 配合使用](/documentation/articles/hdinsight-hadoop-hive-java-udf/)

* [将 Python 用户定义函数与 Hive 和 Pig 配合使用](/documentation/articles/hdinsight-python/)

* [将 C# 用户定义函数与 Hive 和 Pig 配合使用](/documentation/articles/hdinsight-hadoop-hive-pig-udf-dotnet-csharp/)

* [如何将自定义的 Hive 用户定义函数添加到 HDInsight](http://blogs.msdn.com/b/bigdatasupport/archive/2014/01/14/how-to-add-custom-hive-udfs-to-hdinsight.aspx)

* [用于将日期/时间格式转换为 Hive 时间戳的 Hive 用户定义函数示例](https://github.com/Azure-Samples/hdinsight-java-hive-udf)

## <a id="data"></a>示例数据

Hive on HDInsight 中预先加载了名为 `hivesampletable` 的内部表。 HDInsight 还提供了可与 Hive 配合使用的示例数据集。 这些数据集存储在 `/example/data` 和 `/HdiSamples` 目录中。 这些目录存在于群集的默认存储中。

## <a id="job"></a>Hive 查询示例

以下 HiveQL 语句将各列投影到 `/example/data/sample.log` 文件上：

    set hive.execution.engine=tez;
    DROP TABLE log4jLogs;
    CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE LOCATION '/example/data/';
    SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;

在上例中，HiveQL 语句执行以下操作：

* `set hive.execution.engine=tez;`：将执行引擎设置为使用 Tez。 使用 Tez 而不是 MapReduce 可以提高查询性能。 有关 Tez 的详细信息，请参阅[使用 Apache Tez 提高性能](#usetez)部分。

    > [AZURE.NOTE]
    > 仅在使用基于 Windows 的 HDInsight 群集时需要此语句。 Tez 是用于基于 Linux 的 HDInsight 的默认执行引擎。

* `DROP TABLE`：如果该表已存在，则删除它。

* `CREATE EXTERNAL TABLE`：在 Hive 中创建一个新的**外部**表。 外部表只会在 Hive 中存储表定义。 数据以原始格式的形式保留在原始位置中。

* `ROW FORMAT`：告知 Hive 如何设置数据的格式。 在此情况下，每个日志中的字段以空格分隔。

* `STORED AS TEXTFILE LOCATION`：让 Hive 知道数据的存储位置（`example/data` 目录），并且数据已存储为文本。 数据可以在一个文件中，也可以分散在目录的多个文件内。

* `SELECT`：选择 **t4** 列中包含值 **[ERROR]** 的所有行的计数。 此语句返回的值为 **3**，因为有三行包含此值。

* `INPUT__FILE__NAME LIKE '%.log'` - Hive 会尝试向目录中的所有文件应用架构。 在这种情况下，目录包含与架构不匹配的文件。 为防止结果中包含垃圾数据，此语句指示 Hive 应当仅返回以 .log 结尾的文件中的数据。

> [AZURE.NOTE]
> 如果希望通过外部源更新基础数据，应使用外部表。 例如，自动化数据上传过程或 MapReduce 操作。
><p>
> 删除外部表**不会**删除数据，只会删除表定义。

若要创建**内部**表而非外部表，请使用以下 HiveQL：

    set hive.execution.engine=tez;
    CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
    STORED AS ORC;
    INSERT OVERWRITE TABLE errorLogs
    SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]';

这些语句将执行以下操作：

* `CREATE TABLE IF NOT EXISTS`：如果该表不存在，则创建该表。 由于未使用 **EXTERNAL** 关键字，因此，此语句创建一个内部表。 该表存储在 Hive 数据仓库中，并完全由 Hive 管理。

* `STORED AS ORC`：以优化的行纵栏式 (ORC) 格式存储数据。 ORC 是高度优化且有效的 Hive 数据存储格式。

* `INSERT OVERWRITE ... SELECT`：从包含 **[ERROR]** 的 **log4jLogs** 表中选择行，然后将数据插入到 **errorLogs** 表中。

> [AZURE.NOTE]
> 与外部表不同，删除内部表会同时删除基础数据。

## <a name="improve-hive-query-performance"></a>提高 Hive 查询性能

### <a id="usetez"></a>Apache Tez

[Apache Tez](http://tez.apache.org) 是让数据密集型应用程序（例如 Hive）能够大规模高效运行的框架。 默认情况下，已经为基于 Linux 的 HDInsight 群集启用了 Tez。

> [AZURE.NOTE]
> 对于基于 Windows 的 HDInsight 群集来说，Tez 目前默认处于关闭状态，因此必须启用。 若要充分利用 Tez，你必须设置 Hive 查询的以下值：
><p>
> `set hive.execution.engine=tez;`
><p>
> Tez 是用于基于 Linux 的 HDInsight 群集的默认引擎。

[Hive on Tez 设计文档](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez)包含有关实现选项和优化配置的详细信息。

为了帮助使用 Tez 调试运行的作业，HDInsight 提供了以下 Web UI，使你可以查看 Tez 作业的详细信息：

* [在基于 Linux 的 HDInsight 上使用 Ambari Tez 视图](/documentation/articles/hdinsight-debug-ambari-tez-view/)

* [在基于 Windows 的 HDInsight 上使用 Tez UI](/documentation/articles/hdinsight-debug-tez-ui/)

### <a name="low-latency-analytical-processing-llap"></a>低延迟分析处理 (LLAP)

[LLAP](https://cwiki.apache.org/confluence/display/Hive/LLAP)（有时称为 Live Long and Process）是 Hive 2.0 中的一项新功能，可对查询进行内存中缓存。 LLAP 能够大幅加快 Hive 查询的速度，[在某些情况下，速度比 Hive 1.x 要快 2.6 倍](https://hortonworks.com/blog/announcing-apache-hive-2-1-25x-faster-queries-much/)。

HDInsight 在交互式 Hive 群集类型中提供 LLAP。 有关详细信息，请参阅[交互式 Hive 入门](/documentation/articles/hdinsight-hadoop-use-interactive-hive/)文档。

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

## <a name="hive-jobs-and-sql-server-integration-services"></a>Hive 作业和 SQL Server Integration Services

可以使用 SQL Server Integration Services (SSIS) 来运行 Hive 作业。 Azure Feature Pack for SSIS 提供适用于 HDInsight 上的 Hive 作业的以下组件。

* [Azure HDInsight Hive 任务][hivetask]

* [Azure 订阅连接管理器][connectionmanager]

在 [此处][ssispack]了解有关 Azure Feature Pack for SSIS 的详细信息。

## <a id="nextsteps"></a>后续步骤

现在，你已了解什么是 Hive，以及如何将它与 HDInsight 中的 Hadoop 配合使用，请使用以下链接来学习 Azure HDInsight 的其他用法。

* [将数据上传到 HDInsight][hdinsight-upload-data]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
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

[hdinsight-provision]: /documentation/articles/hdinsight-hadoop-provision-linux-clusters/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/

[Powershell-install-configure]: https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs
[powershell-here-strings]: http://technet.microsoft.com/zh-cn/library/ee692792.aspx

[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

<!--Update_Description: wording update-->