<properties
   pageTitle="在 HDInsight 中使用 Hive | Azure"
   description="了解如何通过 Visual Studio 将 Hive 与 HDInsight 配合使用。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>
<tags ms.service="hdinsight"
    ms.date="04/03/2015"
    wacn.date="04/15/2015"
    />

# 使用 HDInsight Tools for Visual Studio 运行 Hive 查询

[AZURE.INCLUDE [hive-selector](../includes/hdinsight-selector-use-hive.md)]

在本文中，你将学习如何使用 HDInsight Tools for Visual Studio，将 Hive 查询远程提交到 HDInsight 群集。

> [AZURE.NOTE] 本文档未详细描述示例中使用的 HiveQL 语句的作用。有关此示例中使用的 HiveQL 的详细信息，请参阅<a href="/documentation/articles/hdinsight-use-hive/" target="_blank">将 Hive 与 HDInsight 上的 Hadoop 配合使用</a>。

## <a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* Azure HDInsight（HDInsight 上的 Hadoop）群集（基于 Linux 或 Windows）

* Visual Studio 2012 <a href="http://www.microsoft.com/zh-CN/download/details.aspx?id=39305" target="_blank">Update 4</a> <!--Visual Studio 2013 <a href="https://www.microsoft.com/zh-CN/download/details.aspx?id=43721" target="_blank">Update 3</a> or <a href="https://www.microsoft.com/zh-CN/download/details.aspx?id=43722" target="_blank">Visual Studio Express 2013</a>-->

## <a id="run"></a> 使用 HDInsight Tools for Visual Studio 运行 Hive 查询

1. 打开"Visual Studio"，然后选择"新建"、"项目"、"HDInsight"，最后选择"Hive 应用程序"。提供此项目的名称。

2. 打开使用此项目创建的 **Script.hql** 文件，并在其中粘贴以下 HiveQL 语句。

        DROP TABLE log4jLogs;
        CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
        STORED AS TEXTFILE LOCATION 'wasb:///example/data/';
        SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' GROUP BY t4;

    这些语句将执行以下操作

    * **DROP TABLE** - 删除表和数据文件（如果表已存在）
    * **CREATE EXTERNAL TABLE** - 在 Hive 中创建新的"外部"表。外部表只会在 Hive 中存储表定义 - 数据会保留在原始位置。

        > [AZURE.NOTE] 当你预期以外部源更新基础数据（例如自动化数据上载过程），或以其他 MapReduce 操作更新基础数据，但希望 Hive 查询始终使用最新数据时，必须使用外部表。
        >
        > 删除外部表**不会**删除数据，只会删除表定义。

    * **ROW FORMAT** - 告知 Hive 如何设置数据的格式。在此情况下，每个日志中的字段以空格分隔
    * **STORED AS TEXTFILE LOCATION** - 让 Hive 知道数据的存储位置（example/data 目录），并且数据已存储为文本
    * **SELECT** - 选择其列 **t4** 包含值 **[ERROR]** 的所有行计数。这应会返回值 **3**，因为有三个行包含此值

3. 从工具栏中，选择你要用于此查询的"HDInsight 群集"，然后选择"提交"以 Hive 作业形式运行语句。"Hive 作业摘要"将会出现并显示有关正在运行的作业的信息。在"作业状态"更改为"已完成"之前，使用"刷新"链接刷新作业信息。

4. 使用"作业输出"链接查看此作业的输出。它应该会显示 `[ERROR] 3`，这是 SELECT 语句返回的值。

5. 你也可以运行 Hive 查询，而无需创建项目。使用"服务器资源管理器"，依次展开"Azure"和"HDInsight"，右键单击 HDInsight 服务器，然后选择"编写 Hive 查询"。

6. 在出现的 **temp.hql** 文档中添加以下 HiveQL 语句。

        CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, 
        t7 string) STORED AS ORC;
        INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]';

    这些语句将执行以下操作。

    * **CREATE TABLE IF NOT EXISTS** - 创建表（如果尚不存在）。由于未使用 **EXTERNAL** 关键字，因此这是一个"内部"表，它存储在 Hive 数据仓库中并完全受 Hive 的管理。

        > [AZURE.NOTE] 与 **EXTERNAL** 表不同，删除内部表会同时删除基础数据。

    * **STORED AS ORC** - 以优化行纵栏表 (ORC) 格式存储数据。这是高度优化且有效的 Hive 数据存储格式
    * **INSERT OVERWRITE ...SELECT** - 从包含 **[ERROR]** 的 **log4jLogs** 表中选择行，然后将数据插入 **errorLogs** 表

7. 从工具栏中，选择"提交"对应的下拉式列表来运行作业。使用"作业状态"确定作业是否已成功完成。

8. 若要确认作业是否已完成并已创建新表，请使用"服务器资源管理器"并依次展开"Azure"、"HDInsight"、你的 HDInsight 群集、"Hive 数据库"和"default"。你应该会同时看到 **errorLogs** 和 **log4jLogs** 表。

## <a id="summary"></a>摘要

如你所见，HDInsight Tools for Visual Studio 提供了简单的方法让你在 HDInsight 群集上运行 Hive 查询、监视作业状态，以及检索输出。

## <a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 Hive 的一般信息。

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息。

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

有关 HDInsight Tools for Visual Studio 的详细信息。

* [HDInsight Tools for Visual Studio 入门](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/)


[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx

[azure-purchase-options]: /pricing/overview/
[azure-free-trial]: /pricing/1rmb-trial/

[apache-tez]: http://tez.apache.org
[apache-hive]: http://hive.apache.org/
[apache-log4j]: http://zh.wikipedia.org/wiki/Log4j
[hive-on-tez-wiki]: https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez
[import-to-excel]: /documentation/articles/hdinsight-connect-excel-power-query/


[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/
[hdinsight-analyze-flight-data]: /documentation/articles/hdinsight-analyze-flight-delay-data/



[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage

[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-get-started]: /documentation/articles/hdinsight-get-started/

[Powershell-install-configure]: /documentation/articles/install-configure-powershell/
[powershell-here-strings]: http://technet.microsoft.com/zh-cn/library/ee692792.aspx

[image-hdi-hive-powershell]: ./media/hdinsight-use-hive/HDI.HIVE.PowerShell.png
[img-hdi-hive-powershell-output]: ./media/hdinsight-use-hive/HDI.Hive.PowerShell.Output.png
[image-hdi-hive-architecture]: ./media/hdinsight-use-hive/HDI.Hive.Architecture.png

<!--HONumber=50-->