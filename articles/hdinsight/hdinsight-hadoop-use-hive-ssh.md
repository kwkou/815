<properties
   pageTitle="在 HDInsight 中将 Hadoop Hive 与 SSH 配合使用 | Azure"
   description="学习如何使用 SSH 连接到 HDInsight 中的 Hadoop 群集，然后使用 Hive 命令行界面以交互方式提交 Hive 查询。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
   ms.service="hdinsight" 
   ms.date="07/06/2015"
   wacn.date="02/06/2017"/>

#通过 SSH 将 Hive 与 HDInsight 中的 Hadoop 配合使用

[AZURE.INCLUDE [hive-selector](../../includes/hdinsight-selector-use-hive.md)]

在本文中，你将学习如何使用安全外壳 (SSH) 连接到 HDInsight 群集上的 Hadoop，然后使用 Hive 命令行界面 (CLI) 以交互方式提交 Hive 查询。

> [AZURE.NOTE]如果你已熟悉如何使用基于 Linux 的 Hadoop 服务器，但刚接触 HDInsight，请参阅[基于 Linux 的 HDInsight 上的 Hadoop 须知信息](/documentation/articles/hdinsight-hadoop-linux-information/)。

##<a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* 基于 Linux 的 HDInsight 上的 Hadoop 群集。

* SSH 客户端。SSH 客户端上应该装有 Linux、Unix 和 Mac OS。Windows 用户必须下载 [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 之类的客户端。

##<a id="ssh"></a>使用 SSH 进行连接

使用 SSH 命令连接到 HDInsight 群集的完全限定域名 (FQDN)。FQDN 是你为群集指定的名称后接 **.azurehdinsight.cn**。例如，以下命令将连接到名为 **myhdinsight** 的群集：

	ssh admin@myhdinsight-ssh.azurehdinsight.cn

如果你在创建 HDInsight 群集时**提供了 SSH 身份验证的证书密钥**，则可能需要指定客户端系统上的私钥位置。

	ssh admin@myhdinsight-ssh.azurehdinsight.cn -i ~/mykey.key

如果你在创建 HDInsight 群集时**提供了 SSH 身份验证的密码**，则需要根据提示提供该密码。

有关将 SSH 与 HDInsight 配合使用的详细信息，请参阅[在 Linux、OS X 和 Unix 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

###PuTTY（基于 Windows 的客户端）

Windows 未提供内置的 SSH 客户端。建议使用可从 [http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 下载的 **PuTTY**。

有关使用 PuTTY 的详细信息，请参阅[从 Windows 在 HDInsight 上配合使用 SSH 与基于 Linux 的 Hadoop](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)。

##<a id="hive"></a>使用 Hive 命令

2. 连接后，请使用以下命令启动 Hive 命令行界面 (CLI)。

        hive

3. 在 CLI 中输入以下语句，以使用示例数据创建名为 **log4jLogs** 的新表：

        DROP TABLE log4jLogs;
        CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
        STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';
        SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' GROUP BY t4;

    这些语句将执行以下操作：

    * **DROP TABLE** - 删除表和数据文件（如果该表已存在）。
    * **CREATE EXTERNAL TABLE** - 在 Hive 中创建新的“外部”表。外部表只会在 Hive 中存储表定义。数据将保留在原始位置。
    * **ROW FORMAT** - 告知 Hive 如何设置数据的格式。在此情况下，每个日志中的字段以空格分隔。
    * **STORED AS TEXTFILE LOCATION** - 让 Hive 知道数据的存储位置（example/data 目录），并且数据已存储为文本。
    * **SELECT** - 选择第 **t4** 列包含值 **[ERROR]** 的所有行的计数。这应会返回值 **3**，因为有三个行包含此值。

    > [AZURE.NOTE]当你预期以外部源更新基础数据（例如自动化数据上载过程），或以其他 MapReduce 操作更新基础数据，但希望 Hive 查询始终使用最新数据时，必须使用外部表。
    ><p>
    > 删除外部表**不会**删除数据，只会删除表定义。

4. 使用以下语句可创建名为 **errorLogs** 的新“内部”表：

        CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
        INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]';

    这些语句将执行以下操作：

    * **CREATE TABLE IF NOT EXISTS** - 创建表（如果该表尚不存在）。由于未使用 **EXTERNAL** 关键字，因此这是一个“内部”表，它存储在 Hive 数据仓库中并完全受 Hive 的管理。
    * **STORED AS ORC** - 以优化行纵栏表 (ORC) 格式存储数据。这是高度优化且有效的 Hive 数据存储格式。
    * **INSERT OVERWRITE ...SELECT**：从包含 **[ERROR]** 的 **log4jLogs** 表中选择行，然后将数据插入 **errorLogs** 表中。

    若要验证是否只将列 t4 中包含 **[ERROR]** 的行存储到了 **errorLogs** 表，请使用以下语句从 **errorLogs** 返回所有行：

        SELECT * from errorLogs;

    应返回三行数据，所有行都包含列 t4 中的 **[ERROR]**。

    > [AZURE.NOTE]与外部表不同，删除内部表会同时删除基础数据。

##<a id="summary"></a>摘要

如你所见，Hive 命令提供了简单的方法让你以交互方式在 HDInsight 群集上运行 Hive 查询、监视作业状态，以及检索输出。

##<a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 Hive 的一般信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx


[apache-tez]: http://tez.apache.org
[apache-hive]: http://hive.apache.org/
[apache-log4j]: http://zh.wikipedia.org/wiki/Log4j
[hive-on-tez-wiki]: https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez
[import-to-excel]: /documentation/articles/hdinsight-connect-excel-power-query/


[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/
[hdinsight-analyze-flight-data]: /documentation/articles/hdinsight-analyze-flight-delay-data/

[putty]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html

[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage/

[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-get-started]: /documentation/articles/hdinsight-get-started/

[Powershell-install-configure]: /documentation/articles/install-configure-powershell/
[powershell-here-strings]: http://technet.microsoft.com/zh-cn/library/ee692792.aspx

[image-hdi-hive-powershell]: ./media/hdinsight-use-hive/HDI.HIVE.PowerShell.png
[img-hdi-hive-powershell-output]: ./media/hdinsight-use-hive/HDI.Hive.PowerShell.Output.png
[image-hdi-hive-architecture]: ./media/hdinsight-use-hive/HDI.Hive.Architecture.png

<!---HONumber=71-->