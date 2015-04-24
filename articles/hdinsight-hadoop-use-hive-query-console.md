<properties
   pageTitle="在 HDInsight 中使用 Hadoop Hive | Azure"
   description="了解如何通过基于 Web 的 HDInsight 查询控制台使用 Hive"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>
<tags ms.service="hdinsight"
    ms.date="04/03/2015"
    wacn.date="04/15/2015"
    />


# 使用查询控制台运行 Hive 查询

[AZURE.INCLUDE [hive-selector](../includes/hdinsight-selector-use-hive.md)]

在本文中，你将了解如何在浏览器中使用 HDInsight 查询控制台在 HDInsight Hadoop 群集上运行 Hive 查询。

> [AZURE.NOTE] 查询控制台只能在基于 Windows 的 HDInsight 群集上使用

## <a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* 基于 Windows 的 HDInsight Hadoop 群集

* 现代 Web 浏览器

## <a id="run"></a> 使用查询控制台运行 Hive 查询

1. 打开 <a href="https://manage.windowsazure.cn" target="_blank">Azure 管理门户</a>，并选择你的 HDInsight 群集。在页面底部选择"查询控制台"。出现提示时，请输入你在创建群集时使用的用户名和密码。

    > [AZURE.NOTE] 可以通过在浏览器中输入 **https://CLUSTERNAME.azurehdinsight.cn** 访问查询控制台。

2. 在页面顶部的链接中，选择"Hive 编辑器"。此时将显示一个窗体，你可以在其中输入要在 HDInsight 群集上运行的 HiveQL 语句。 
	
	![the hive editor](./media/hdinsight-hadoop-use-hive-query-console/queryconsole.png)
	
	将文本 `Select * from hivesampletable` 替换为以下 HiveQL 语句。

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

2. 选择"提交"。页面底部的"作业会话"应显示作业的详细信息。

3. 在"状态"字段变为"已完成"后，请选择作业对应的"查看详细信息"。在详细信息页面上，"作业输出"将包含 `[ERROR]	3`。你可以使用此字段下面的"下载"按钮下载包含作业的输出的文件。


## <a id="summary"></a>摘要

如你所见，查询控制台提供了简单的方法让你在 HDInsight 群集上运行 Hive 查询、监视作业状态，以及检索输出。 

若要了解有关使用查询控制台运行 Hive 查询的详细信息，请在查询控制台的顶部选择"入门"，然后使用示例。每个示例逐步演示了使用 Hive 分析数据的过程，包括示例中使用的 HiveQL 语句的解释。

## <a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 Hive 的一般信息。

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息。

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

[1]: /documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/

[hdinsight-sdk-documentation]:  http://msdn.microsoft.com/zh-cn/library/dn479185.aspx

[azure-purchase-options]: /pricing/purchase-options/
[azure-member-offers]: /pricing/member-offers/
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