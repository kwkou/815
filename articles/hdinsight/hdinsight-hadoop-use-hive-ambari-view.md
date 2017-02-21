<properties
    pageTitle="使用 Ambari 视图来操作 HDInsight (Hadoop) 上的 Hive | Azure"
    description="了解如何从 Web 浏览器中使用 Hive 视图来提交 Hive 查询。Hive 视图是随基于 Linux 的 HDInsight 群集提供的 Ambari Web UI 的一部分。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags 
    ms.assetid="1abe9104-f4b2-41b9-9161-abbc43de8294"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="10/28/2016"
    wacn.date="02/20/2017"
    ms.author="larryfr" />

# 将 Hive 视图与 HDInsight 中的 Hadoop 配合使用
[AZURE.INCLUDE [hive-selector](../../includes/hdinsight-selector-use-hive.md)]

Ambari 是一个管理和监控实用程序，随基于 Linux 的 HDInsight 群集一起提供。通过 Ambari 提供的功能之一是可用于运行 Hive 查询的 Web UI。这就是“Hive 视图”，它是随 HDInsight 群集一起提供的 Ambari 视图的一部分。

> [AZURE.NOTE]
Ambari 有很多功能，本文档中不就此进行讨论。有关详细信息，请参阅 [Manage HDInsight clusters by using the Ambari Web UI](/documentation/articles/hdinsight-hadoop-manage-ambari/)（使用 Ambari Web UI 管理 HDInsight 群集）。
> 
> 

## 先决条件
* 基于 Linux 的 HDInsight 群集。有关创建新群集的信息，请参阅[开始使用基于 Linux 的 HDInsight](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)。

## 打开 Hive 视图
可从 Azure 门户预览中获取 Ambari 视图；选择 HDInsight 群集，然后从“快速链接”部分选择“Ambari 视图”。

![“快速链接”部分](./media/hdinsight-hadoop-use-hive-ambari-view/quicklinks.png)  


也可以通过在 Web 浏览器中转到 https://CLUSTERNAME.azurehdinsight.cn 直接导航到 Ambari（其中 **CLUSTERNAME** 是 HDInsight 群集的名称），然后从页面菜单中选择一组正方形（在页面左侧“管理员”链接和按钮旁边），以列出可用视图。选择“Hive 视图”。

![选择 Ambari 视图](./media/hdinsight-hadoop-use-hive-ambari-view/selecthiveview.png)。

> [AZURE.NOTE]
访问 Ambari 时，系统将提示对该网站进行身份验证。请输入创建群集时使用的管理员（默认为 `admin`）帐户名和密码。
> 
> 

你应看到类似于下面的页面：

![Hive 视图页面的图像，其中包含“查询编辑器”部分](./media/hdinsight-hadoop-use-hive-ambari-view/hiveview.png)  


## 查看表
在页面的“数据库资源管理器”部分中，选择“数据库”选项卡上的“默认”条目。这将显示默认数据库中的表列表。对于新的 HDInsight 群集，只应存在一个表；**hivesampletable**。

![扩展了默认数据库的数据库资源管理器](./media/hdinsight-hadoop-use-hive-ambari-view/databaseexplorer.png)  


由于通过本文档中的步骤添加了新表，所以可以使用“数据库资源管理器”右上角的刷新图标刷新可用表的列表。

## <a name="hivequery"></a>查询编辑器
从 Hive 视图中使用以下步骤对群集中包含的数据运行 Hive 查询。

1. 在此页面的“查询编辑器”中，将以下 HiveQL 语句粘贴到工作表中：
   
        DROP TABLE log4jLogs;
        CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
        STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';
        SELECT t4 AS sev, COUNT(*) AS cnt FROM log4jLogs WHERE t4 = '[ERROR]' GROUP BY t4;
   
    这些语句可执行以下操作：
   
    * **DROP TABLE** - 删除表和数据文件（如果该表已存在）。
    * **CREATE EXTERNAL TABLE** - 在 Hive 中创建新的外部表。外部表仅在 Hive 中存储表定义；数据会保留在原始位置。
    * **ROW FORMAT** - 告知 Hive 如何设置数据的格式。在本例中，每个日志中的字段以空格分隔。
    * **STORED AS TEXTFILE LOCATION** - 告知 Hive 数据的存储位置（example/data 目录），且数据已存储为文本。
    * **SELECT** - 选择 t4 列包含值 [ERROR] 的所有行的计数。
     
     > [AZURE.NOTE]
     外部表应在希望通过外部源（例如自动化数据上传过程）或其他 MapReduce 操作更新基础数据时使用，但始终最好通过 Hive 查询使用最新数据。删除外部表*不会*删除数据，只会删除表定义。
     > 
     > 
2. 使用查询编辑器底部的“执行”按钮启动查询。它应会变为橙色，文本将更改为“停止执行”。“查询进程结果”部分应出现在“查询编辑器”下方，并显示有关作业的信息。
   
    > [AZURE.IMPORTANT]
    某些浏览器可能无法正确刷新日志或结果信息。如果运行一个作业，而它看上去似乎会永久运行下去而不更新日志或返回结果，请尝试改为使用 Mozilla FireFox 或 Google Chrome。
    > 
    > 
3. 查询完成后，“查询进程结果”部分将显示该操作的结果。“停止执行”按钮也将变回绿色“执行”按钮。“结果”选项卡应包含以下信息：
   
        sev       cnt
        [ERROR]   3
   
    “日志”选项卡可用于查看作业创建的日志记录信息。如果查询出现问题，可以使用此方法进行故障排除。
   
    > [AZURE.TIP]
    请注意“查询进程结果”部分左上角的“保存结果”下拉对话框；可使用此下载结果，或将其作为 CSV 文件保存到 HDInsight 存储中。
    > 
    > 
4. 选择此查询的前四行，然后选择“执行”。请注意，作业完成时没有结果。这是因为在选择了查询的一部分时使用“执行”按钮只会运行所选语句。在这种情况下，所选内容不包括会从表中检索行的最后一条语句。如果只选择该行并使用“执行”，应该会看到预期的结果。
5. 使用“查询编辑器”底部的“新建工作表”按钮创建新工作表。在新工作表中，输入以下 HiveQL 语句：
   
        CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
        INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]';
   
    这些语句可执行以下操作：
   
    * **CREATE TABLE IF NOT EXISTS** - 创建表（如果该表不存在）。由于未使用 **EXTERNAL** 关键字，因此这是一个内部表，它存储在 Hive 数据仓库中并完全由 Hive 管理。与外部表不同，删除内部表会同时删除基础数据。
    * **STORED AS ORC** - 以优化行纵栏表 (ORC) 格式存储数据。这是高度优化且有效的 Hive 数据存储格式。
    * **INSERT OVERWRITE ...SELECT** - 从包含 [ERROR] 的 **log4jLogs** 表中选择行，然后将数据插入 **errorLogs** 表中。
     
     使用“执行”按钮运行此查询。“结果”选项卡不会包含任何信息，因为此查询未返回任何行，但状态应显示为“已成功”。

### Hive 设置
选择编辑器右侧的“设置”图标。

![图标](./media/hdinsight-hadoop-use-hive-ambari-view/settings.png)  


设置可用于更改各种 Hive 设置，例如将 Hive 的执行引擎从 Tez（默认）更改为 MapReduce。

### Visual explain
选择编辑器右侧的“Visual Explain”图标。

![图标](./media/hdinsight-hadoop-use-hive-ambari-view/visualexplainicon.png)  


这是查询的“Visual Explain”视图，可以帮助理解复杂查询的流程。可以使用“查询编辑器”中的“说明”按钮查看此视图的文本等效内容。

![Visual explain 图像](./media/hdinsight-hadoop-use-hive-ambari-view/visualexplain.png)  


### Tez
选择编辑器右侧的“Tez”图标。

![图标](./media/hdinsight-hadoop-use-hive-ambari-view/tez.png)  


这将显示 Tez 用于此查询的有向无环图 (DAG)（如果有可用的）。如果想要查看过去已运行的查询的 DAG，或调试 Tez 进程，请改用 [Tez 视图](/documentation/articles/hdinsight-debug-ambari-tez-view/)。

### 通知
选择编辑器右侧的“通知”图标。

![图标](./media/hdinsight-hadoop-use-hive-ambari-view/notifications.png)  


通知是运行查询时生成的消息。例如，当提交查询或发生错误时，会收到通知。

## 已保存的查询
1. 在“查询编辑器”中，创建一个新工作表，并输入以下查询：
   
        SELECT * from errorLogs;
   
    执行查询以验证其是否正常工作。结果将如下所示：
   
        errorlogs.t1     errorlogs.t2     errorlogs.t3     errorlogs.t4     errorlogs.t5     errorlogs.t6     errorlogs.t7
        2012-02-03     18:35:34     SampleClass0     [ERROR]     incorrect     id     
        2012-02-03     18:55:54     SampleClass1     [ERROR]     incorrect     id     
        2012-02-03     19:25:27     SampleClass4     [ERROR]     incorrect     id
2. 使用编辑器底部的“另存为”按钮。将此查询命名为“错误日志”，然后选择“确定”。请注意，工作表的名称将更改为 **Errorlogs**。
3. 选择 Hive 视图页面顶部的“已保存的查询”选项卡。请注意，**Errorlogs** 现列为已保存的查询。它会保留在此列表中，直至被删除。选择名称将在“查询编辑器”中打开查询。

## 查询历史记录
可使用 Hive 视图顶部的“历史记录”按钮查看先前运行过的查询。现在使用该按钮，并选择之前运行过的一些查询。选定一个查询后，它会在“查询编辑器”中打开。

## 用户定义的函数 (UDF)
还可以通过**用户定义的函数 (UDF)** 扩展 Hive。通过 UDF 可实现 HiveQL 中不容易建模的功能或逻辑。

虽然可以在查询中将 UDF 添加为 HiveQL 语句的一部分，但也可使用 Hive 视图顶部的 UDF 选项卡声明并保存一组可用于“查询编辑器”的 UDF。

将 UDF 添加到 Hive 视图后，“查询编辑器”底部会显示一个“插入 UDF”按钮。选择此选项卡将显示 Hive 视图中定义的 UDF 的下拉列表。选择一个 UDF 可向查询添加 HiveQL 语句以启用 UDF。

例如，如果定义了一个具有以下属性的 UDF：

* 资源名称：myudfs
* 资源路径：wasbs:///myudfs.jar
* UDF 名称：myawesomeudf
* UDF 类名称：com.myudfs.Awesome

使用“插入 UDF”按钮将显示一个名为 **myudfs** 的条目，以及另一个为该资源定义的每个 UDF 的下拉列表。在本例中，为 **myawesomeudf**。选择此条目将在查询的开头添加以下内容：

    add jar wasbs:///myudfs.jar;

    create temporary function myawesomeudf as 'com.myudfs.Awesome';

然后便可在查询中使用 UDF。例如，`SELECT myawesomeudf(name) FROM people;`。

有关如何将 UDF 与 HDInsight 上的 Hive 配合使用的详细信息，请参阅以下资源：

* [在 HDInsight 中将 Python 与 Hive 和 Pig 配合使用](/documentation/articles/hdinsight-python/)
* [如何将自定义 Hive UDF 添加到 HDInsight](http://blogs.msdn.com/b/bigdatasupport/archive/2014/01/14/how-to-add-custom-hive-udfs-to-hdinsight.aspx)

## <a id="nextsteps"></a>后续步骤
有关 HDInsight 中的 Hive 的一般信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

有关 HDInsight 上 Hadoop 的其他使用方法的信息：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=Mooncake_0213_2017-->