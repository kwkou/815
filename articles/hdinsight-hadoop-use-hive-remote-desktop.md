<properties
   pageTitle="在 HDInsight 中将 Hadoop Hive 与远程桌面配合使用 | Azure"
   description="学习如何通过使用远程桌面连接到 HDInsight 中的 Hadoop 群集，然后通过使用 Hive 命令行界面运行 Hive 查询。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="05/03/2016"
	wacn.date="06/29/2016"/>

# 通过远程桌面将 Hive 与 HDInsight 上的 Hadoop 配合使用

[AZURE.INCLUDE [hive-selector](../includes/hdinsight-selector-use-hive.md)]

在本文中，你将学习如何通过使用远程桌面连接到 HDInsight 群集，然后通过使用 Hive 命令行界面 (CLI) 运行 Hive 查询。

> [AZURE.NOTE] 本文档未详细描述示例中使用的 HiveQL 语句的作用。有关此示例中使用的 HiveQL 的详细信息，请参阅[将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)。

##<a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* 基于 Windows 的 HDInsight（HDInsight 上的 Hadoop）群集

* 运行 Windows 10、Window 8 或 Windows 7 的客户端计算机

##<a id="connect"></a>使用远程桌面进行连接

为 HDInsight 群集启用远程桌面，然后根据[使用 RDP 连接到 HDInsight 群集](/documentation/articles/hdinsight-administer-use-management-portal-v1/#rdp)中的说明连接到该群集。

##<a id="hive"></a>使用 Hive 命令

连接到 HDInsight 群集的桌面之后，请执行以下步骤来使用 Hive：

1. 从 HDInsight 桌面启动“Hadoop 命令行”。

2. 输入以下命令启动 Hive CLI：

        %hive_home%\bin\hive

    在启动 CLI 后，你将会看到 Hive CLI 提示符：`hive>`。

3. 在 CLI 中输入以下语句，以使用示例数据创建名为 **log4jLogs** 的新表：

        set hive.execution.engine=tez;
        DROP TABLE log4jLogs;
        CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
        STORED AS TEXTFILE LOCATION 'wasb:///example/data/';
        SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;

    这些语句将执行以下操作：

    * **DROP TABLE**：删除表和数据文件（如果该表已存在）。

    * **CREATE EXTERNAL TABLE**：在 Hive 中创建新的“外部”表。外部表仅在 Hive 中存储表定义；数据会保留在原始位置。

		> [AZURE.NOTE] 当你预期以外部源更新基础数据（例如自动化数据上载过程），或以其他 MapReduce 操作更新基础数据，但希望 Hive 查询始终使用最新数据时，必须使用外部表。
		> <p>删除外部表**不会**删除数据，只会删除表定义。

	* **ROW FORMAT**：告知 Hive 如何设置数据的格式。在此情况下，每个日志中的字段以空格分隔。

    * **STORED AS TEXTFILE LOCATION**：让 Hive 知道数据的存储位置（example/data 目录），并且数据已存储为文本。

    * **SELECT** - 选择第 **t4** 列包含值 **[ERROR]** 的所有行的计数。这应会返回值 **3**，因为有三个行包含此值。

    * **INPUT\_\_FILE\_\_NAME LIKE '%.log'** - 告诉 Hive，我们只应返回以 .log 结尾的文件中的数据。此项将搜索限定于包含数据的 sample.log 文件，使搜索不会返回与所定义架构不符的其他示例数据文件中的数据。


4. 使用以下语句创建名为 **errorLogs** 的新“内部”表：

        CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
        INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';

    这些语句将执行以下操作：

    * **CREATE TABLE IF NOT EXISTS**：创建表（如果该表尚不存在）。由于未使用 **EXTERNAL** 关键字，因此这是一个内部表，它存储在 Hive 数据仓库中并完全受 Hive 的管理。

		> [AZURE.NOTE] 与**外部**表不同，删除内部表会同时删除基础数据。

    * **STORED AS ORC**：以优化行纵栏表 (ORC) 格式存储数据。这是高度优化且有效的 Hive 数据存储格式。

    * **INSERT OVERWRITE ...SELECT**：从包含 **[ERROR]** 的 **log4jLogs** 表中选择行，然后将数据插入 **errorLogs** 表中。

    若要验证是否只将第 t4 列中包含 **[ERROR]** 的行存储到了 **errorLogs** 表，请使用以下语句从 **errorLogs** 返回所有行：

        SELECT * from errorLogs;

    应返回三行数据，所有行都包含列 t4 中的 **[ERROR]**。

##<a id="summary"></a>摘要

如你所见，Hive 命令提供了简单的方法让你以交互方式在 HDInsight 群集上运行 Hive 查询、监视作业状态，以及检索输出。

##<a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 Hive 的一般信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

如果将 Tez 与 Hive 配合使用，请参阅以下文档以了解调试信息：

* [在基于 Windows 的 HDInsight 上使用 Tez UI](/documentation/articles/hdinsight-debug-tez-ui/)

[1]: /documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/

[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx

[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/

[apache-tez]: http://tez.apache.org
[apache-hive]: http://hive.apache.org/
[apache-log4j]: http://zh.wikipedia.org/wiki/Log4j
[hive-on-tez-wiki]: https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez
[import-to-excel]: /documentation/articles/hdinsight-connect-excel-power-query/


[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/
[hdinsight-analyze-flight-data]: /documentation/articles/hdinsight-analyze-flight-delay-data/



[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/

[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/

[Powershell-install-configure]: /documentation/articles/powershell-install-configure/
[powershell-here-strings]: http://technet.microsoft.com/zh-cn/library/ee692792.aspx


<!---HONumber=Mooncake_0411_2016-->