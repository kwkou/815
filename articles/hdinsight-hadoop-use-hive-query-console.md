<properties
   pageTitle="在 HDInsight 中的查询控制台上使用 Hadoop Hive | Azure"
   description="了解如何在浏览器中使用基于 Web 的查询控制台在 HDInsight Hadoop 群集上运行 Hive 查询。"
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

# 使用查询控制台运行 Hive 查询

[AZURE.INCLUDE [hive-selector](../includes/hdinsight-selector-use-hive.md)]

在本文中，你将了解如何在浏览器中使用 HDInsight 查询控制台在 HDInsight Hadoop 群集上运行 Hive 查询。

> [AZURE.NOTE] 查询控制台只能在基于 Windows 的 HDInsight 群集上使用。


##<a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* 基于 Windows 的 HDInsight Hadoop 群集

* 现代 Web 浏览器

##<a id="run"></a>使用查询控制台运行 Hive 查询

1. 打开 <a href="https://manage.windowsazure.cn" target="_blank">Azure 经典管理门户</a>，并选择你的 HDInsight 群集。在页面底部选择“查询控制台”。出现提示时，输入你在创建群集时使用的用户名和密码。

2. 在页面顶部的链接中，选择“Hive 编辑器”。此时将显示一个窗体，你可以在其中输入要在 HDInsight 群集上运行的 HiveQL 语句。

	![hive 编辑器](./media/hdinsight-hadoop-use-hive-query-console/queryconsole.png)

	将文本 `Select * from hivesampletable` 替换为以下 HiveQL 语句：

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
    * **SELECT**：选择其列 **t4** 包含值 **[ERROR]** 的所有行计数。这应会返回值 **3**，因为有三个行包含此值。
    * **INPUT\_\_FILE\_\_NAME LIKE '%.log'** - 告诉 Hive，我们只应返回以 .log 结尾的文件中的数据。此项将搜索限定于包含数据的 sample.log 文件，使搜索不会返回与所定义架构不符的其他示例数据文件中的数据。

2. 单击“提交”。页面底部的“作业会话”应显示作业的详细信息。

3. 当“状态”字段更改为“已完成”时，请选择与作业对应的“查看详细信息”。在详细信息页上，“作业输出”包含 `[ERROR]	3`。你可以使用此字段下面的“下载”按钮下载包含作业的输出的文件。


##<a id="summary"></a>摘要

如你所见，查询控制台提供了简单的方法让你在 HDInsight 群集上运行 Hive 查询，监视作业状态，以及检索输出。

若要了解有关使用 Hive 查询控制台运行 Hive 作业的详细信息，请选择查询控制台顶部的“入门”，然后使用提供的示例。每个示例演练使用 Hive 分析数据的过程，包括有关本示例中使用的 HiveQL 语句的说明。

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


[img-hdi-hive-powershell-output]: ./media/hdinsight-use-hive/HDI.Hive.PowerShell.Output.png

<!---HONumber=Mooncake_0411_2016-->