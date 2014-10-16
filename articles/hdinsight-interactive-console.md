<properties linkid="manage-services-hdinsight-howto-work-with-the-interactive-console" urlDisplayName="Interactive Console" pageTitle="How to work with the interactive console in HDInsight | Azure" metaKeywords="" description="In this guide, you'll learn how to perform common tasks such as file upload, run jobs, and visualize data using the interactive console for HDInsight Service." metaCanonical="" services="hdinsight" documentationCenter="" title="HDInsight Interactive JavaScript and Hive Consoles" authors="bradsev" solutions="" manager="paulettm" editor="cgronlun" />

# HDInsight 交互式 JavaScript 和 Hive 控制台

Microsoft Azure HDInsight 服务附带了适用于 JavaScript 和 Hive 的交互式控制台。这些控制台提供了简单的交互式读取-计算-打印循环 (REPL) 体验，用户可在其中输入表达式，计算这些表达式，然后立即查询并显示 MapReduce 作业的结果。JavaScript 控制台执行 Pig Latin 语句。 Hive 控制台将计算 Hive 查询语言 (Hive QL) 语句。这两种类型的语句已编译为 MapReduce 程序。与远程连接到 Hadoop 群集的头节点并直接处理 MapReduce 程序相比，在这些控制台上管理 Hadoop 作业要简单得多。

**JavaScript 控制台**：一个提供 Hadoop 生态系统的流畅接口的命令外壳。流畅接口使用链接指令的方法，此方法将序列中某个调用的上下文中继到该序列中的后续调用。JavaScript 控制台提供：

-   对 Hadoop 群集、其资源以及 Hadoop 分布式文件系统 (HDFS) 命令的访问。
-   对数据传入和传出 Hadoop 群集的管理和操作。
-   一个流畅接口，此接口将计算 Pig Latin 和 JavaScript 语句来定义一系列 MapReduce 程序，从而创建数据处理工作流。

**Hive 控制台**：Hive 是一个基于 Hadoop 构建的数据仓库框架，它提供了数据管理、查询和分析功能。它使用 HiveQL（一种 SQL 行话）查询存储在 Hadoop 群集中的数据。Hive 控制台提供：

-   对 Hadoop 群集、其资源和 HDFS 命令的访问。
-   对 Hive 框架的实现，此实现可在 Hadoop 群集上执行 HiveQL 语句。
-   一种用于 HDFS 的关系数据库模型，利用它，你可以与分布式文件系统中存储的数据交互，就像那些数据是存储在表中一样。
    JavaScript 控制台使用 Pig Latin（一种数据流语言），Hive 控制台使用 HiveQL（一种查询语言）。

Pig（和 JavaScript 控制台）往往是那些更熟悉脚本方法的人员的首选，其中会使用一系列链接的（或流畅的）转换来定义数据处理工作流。如果你有高度非结构化的数据，它也是一个不错的选择。

Hive（及其控制台）往往是那些更熟悉 SQL 和关系数据库环境的人员的首选。在 Hive 中使用架构和表抽象意味着，所获得的体验将非常接近通常在 RDBMS 中获得的体验。

Pig 和 Hive 提供编译成 MapReduce 程序的更高级别的语言，这些程序用 Java 编写并在 HDFS 上运行。若要实现真正的精确控制或高性能，你需要直接编写 MapReduce 程序。

## 在本教程中

-   [使用 JavaScript 控制台运行 MapReduce 作业][]
-   [使用 JavaScript 控制台以图形形式显示结果][]
-   [使用 Hive 控制台将结果导出到 Hive 表][]
-   [使用 Hive 控制台查询 Hive 表中的数据][]

## 使用 JavaScript 控制台运行 MapReduce 作业

在本节中，你使用 JavaScript 控制台运行 HDInsight 服务附带的 WordCount 示例。此处运行的 JavaScript 查询使用在由交互式控制台提供的 Pig 上分层的流畅 API。此处分析的文本文件是 Project Gutenberg 电子书版本的《莱奥纳多.达.芬奇笔记》(The Notebooks of Leonardo Da Vinci)**。指定一个筛选器，以使 MapReduce 作业的结果仅包含 10 个最常出现的词。

1.  登录到[管理门户][]。
2.  单击“HDInsight” 。这将显示已部署的 Hadoop 群集的列表。
3.  单击你要连接到的 HDInsight 群集的名称。
4.  单击“管理群集” 。
5.  输入你的凭据，然后单击“登录” 。
6.  从 HDInsight 门户中，单击“示例” 。

    ![HDI.Tiles.Samples][hdi-tiles-samples]

7.  从“Hadoop 示例库” 页中，单击“WordCount” 磁贴。
8.  从右上位置单击 **WordCount.js**，将文件保存到本地目录中，例如 ../downloads 文件夹。

    ![HDI.JsConsole.WordCountDownloads][]

9.  单击左上角的 **Azure HDInsight**，以返回到群集仪表板页面。
10. 单击“交互式群集” 以显示 JavaScript 控制台。

    ![HDI.Tiles.InteractiveConsole][hdi-tiles-interactive-console]

11. 单击右上角的 **JavaScript**。
12. 运行以下命令：

    fs.put()

13. 将以下参数输入“上载文件” 窗口：

    -   **源：** \_..\\downloads\\Wordcount.js
    -   **目标：** ./WordCount.js/

    ![HDI.JsConsole.UploadJs][hdi-jsconsole-upload]

    浏览 WordCount.js 文件的位置。这将需要完整的本地路径。作为 HDFS 中相对地址的一部分，目标路径的开头需要有一个点。

14. 单击**“上载”**按钮。

15. 运行下列命令以列出文件并显示内容：

        #ls
        #cat WordCount.js

    ![HDI.JsConsole.JsCode][hdi-jsconsole-jscode]

    请注意，在化简函数中计算某个词的出现次数之前，JavaScript 映射函数会使用“toLowerCase()”方法从文本中删除大写字母。

16. 运行以下命令，以列出将由 WordCount MapReduce 作业处理的数据文件：

        #ls /example/data/gutenberg

17. 运行以下命令以执行 MapReduce 程序：

        pig.from("/example/data/gutenberg/davinci.txt").mapReduce("/user/admin/WordCount.js", "word, count:long").orderBy("count DESC").take(10).to("DaVinciTop10Words")

    将 admin 替换为当前登录用户名。

    注意这些指令是如何使用圆点运算符连接起来的，输出文件称为“DaVinciTop10Words”**。在下一部分中，你将访问此输出文件。

    一旦完成，你将看到以下内容：

        pig.from("/example/data/gutenberg/davinci.txt").mapReduce("/user/admin/WordCount.js", "word, count:long").orderBy("count DESC").take(10).to("DaVinciTop10Words")
        2013-04-25 18:54:28,116 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - Success!(View Log)

18. 若要查看作业进度，请滚动到右侧并单击“查看日志” 。如果作业未能完成，则该日志还将提供诊断信息。作业完成后，你将在日志结尾看到以下消息：

        [main] INFO org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - Success! followed by a link to the log file.

19. 运行以下命令以列出输出文件：

        #ls

    注意创建了一个 DavinciTop10Words 文件夹。

## 使用 JavaScript 控制台以图形形式显示结果

在上一部分中，你运行了 MapReduce 作业从一个文本文件中检索出现频率排前 10 位的单词。输出文件为 ./DaVinciTop10Words。

1.  运行下列命令以在 DaVinciTop10Words 目录中显示结果：

        file = fs.read("DaVinciTop10Words")

    结果与下面类似：

        js> file=fs.read("DaVinciTop10Words")
        the 22966
        of  11228
        and 8428
        in  5737
        to  5296
        a   4791
        is  4261
        it  3073
        that    2903
        which   2544

2.  运行以下命令将此文件的内容解析到数据文件中：

        data = parse(file.data, "word, count:long")

3.  运行以下命令以绘制数据的图表

        graph.bar(data)

    ![HDI.JsConsole.BarGraphTop10Words][hdi-jsconsole-bargraph-top10words]

## 使用 Hive 控制台将结果导出到 Hive 表

本节介绍 Hive 交互式控制台。你将从 MapReduce 作业输出中创建 Hive 表。下一节介绍如何查询此表中的数据。

**创建 Hive 表**

1.  单击右上角的“Hive”打开 Hive 控制台。

2.  输入以下命令，根据存储在“DaVinciTop10Words”文件夹中的 WordCount 示例输出结果，创建一个名为 *DaVinciWordCountTable* 的两列的表：

        CREATE EXTERNAL TABLE DaVinciWordCountTable     
        (word STRING,
        count INT)  
        ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'  
        STORED AS TEXTFILE LOCATION '/user/admin/DaVinciTop10Words';    

    将 admin 替换为登录用户名。

    请注意，此表将作为 EXTERNAL 表创建以使目标文件夹独立于该表。另请注意，你只需指定输出文件所在的文件夹，不需指定文件名本身。

3.  单击左下角的**“计算”**。

4.  输入以下命令，以确认已创建包含两个列的表：

        SHOW TABLES;
        DESCRIBE DaVinciWordCountTable;

5.  单击“计算” 。

	![HDI.Hive.ShowDescribeTable][hdi-hive-showdescribetable]

## 使用 Hive 控制台查询 Hive 表中的数据

1.  运行以下命令，以查询出现次数排名前 10 的单词：

        SELECT word, count
        FROM DaVinciWordCountTable
        ORDER BY count DESC LIMIT 10

    此查询的结果为：

        SELECT word, count FROM DaVinciWordCountTable ORDER BY count DESC LIMIT 10
        the 22966
        of  11228
        and 8428
        in  5737
        to  5296
        a   4791
        is  4261
        it  3073
        that    2903
        which   2544

## 后续步骤

你已了解如何从交互式 JavaScript 控制台运行 Hadoop 作业，以及如何使用该控制台从作业中检查结果。此外，你已了解如何使用交互式 Hive 控制台创建并查询包含 MapReduce 程序中的输出的表，来检查和处理 Hadoop 作业的结果。你已查看控制台中使用的 Pig Latin 和 Hive QL 语句的示例。最后，你已了解如何使用 Hadoop 群集简化 JavaScript 和 Hive 控制台的 REPL 交互式特性。若要了解更多信息，请参阅下列文章：

-   [将 Pig 与 HDInsight 配合使用][]
-   [将 Hive 与 HDInsight 配合使用][]
-   [将 MapReduce 与 HDInsight 配合使用][]

[hdi-hive-showdescribetable]:./media/hdinsight-interactive-console/HDI.Hive.ShowDescribeTable.PNG "Hive Table Confirmation")

  [使用 JavaScript 控制台运行 MapReduce 作业]: #runjob
  [使用 JavaScript 控制台以图形形式显示结果]: #displayresults
  [使用 Hive 控制台将结果导出到 Hive 表]: #createhivetable
  [使用 Hive 控制台查询 Hive 表中的数据]: #queryhivetable
  [管理门户]: https://manage.windowsazure.cn
  [hdi-tiles-samples]: ./media/hdinsight-interactive-console/HDI.TileSamples.PNG
  [HDI.JsConsole.WordCountDownloads]: ./media/hdinsight-interactive-console/HDI.JsConsole.WordCountDownloads.PNG
  [hdi-tiles-interactive-console]: ./media/hdinsight-interactive-console/HDI.TileInteractiveConsole.PNG
  [hdi-jsconsole-upload]: ./media/hdinsight-interactive-console/HDI.JsConsole.UploadJs.PNG
  [hdi-jsconsole-jscode]: ./media/hdinsight-interactive-console/HDI.JsConsole.JsCode.PNG
  [hdi-jsconsole-bargraph-top10words]: ./media/hdinsight-interactive-console/HDI.JsConsole.BarGraphTop10Words.PNG
  [将 Pig 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-pig-with-hdinsight/
  [将 Hive 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-hive-with-hdinsight/
  [将 MapReduce 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-mapreduce-with-hdinsight/
