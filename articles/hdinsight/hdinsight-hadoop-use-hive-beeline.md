<!-- not suitable for Mooncake -->


<properties
   pageTitle="使用 Beeline 来操作 HDInsight (Hadoop) 上的 Hive | Azure"
   description="了解如何使用 SSH 连接到 HDInsight 中的 Hadoop 群集，然后使用 Beeline 以交互方式提交 Hive 查询。Beeline 是用于通过 JDBC 操作 HiveServer2 的实用工具。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="jhubbard"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="10/10/2016"
   wacn.date="02/06/2017"
   ms.author="larryfr"/>  


#通过 Beeline 将 Hive 与 HDInsight 中的 Hadoop 配合使用

[AZURE.INCLUDE [hive-selector](../../includes/hdinsight-selector-use-hive.md)]

在本文中，你将学习如何使用安全外壳 (SSH) 连接到基于 Linux 的 HDInsight 群集，然后使用 [Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline-NewCommandLineShell) 命令行工具以交互方式提交 Hive 查询。

> [AZURE.NOTE] Beeline 使用 JDBC 连接到 Hive。有关将 JDBC 与 Hive 配合使用的详细信息，请参阅 [Connect to Hive on Azure HDInsight using the Hive JDBC driver](/documentation/articles/hdinsight-connect-hive-jdbc-driver/)（使用 Hive JDBC 驱动程序连接到 Azure HDInsight 上的 Hive）。

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

##<a id="beeline"></a>使用 Beeline 命令

1. 连接后，使用以下命令来启动 Beeline：

        beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin

    随后将会启动 Beeline 客户端并连接到 JDBC url。此处使用 `localhost`，是因为 HiveServer2 在群集中的两个头节点上运行，而我们将直接在主头节点上运行 Beeline。
    
    完成该命令后，将出现 `jdbc:hive2://localhost:10001/>` 提示符。

3. Beeline 命令通常以 `!` 字符开头，例如，`!help` 将显示帮助。但是，通常可以省略 `!`。例如，`help` 也是有效的。

    查看帮助时，你会看到用于执行 HiveQL 语句的 `!sql`。但是，由于 HiveQL 非常流行，因此你可以省略前面的 `!sql`。以下两个语句的结果完全相同；显示当前可通过 Hive 获取的表：
    
        !sql show tables;
        show tables;
    
    在新群集上，只会列出一个表：__hivesampletable__。

4. 使用以下命令可以显示 hivesampletable 的架构：

        describe hivesampletable;
        
    此命令将返回以下信息：
    
        +-----------------------+------------+----------+--+
        |       col_name        | data_type  | comment  |
        +-----------------------+------------+----------+--+
        | clientid              | string     |          |
        | querytime             | string     |          |
        | market                | string     |          |
        | deviceplatform        | string     |          |
        | devicemake            | string     |          |
        | devicemodel           | string     |          |
        | state                 | string     |          |
        | country               | string     |          |
        | querydwelltime        | double     |          |
        | sessionid             | bigint     |          |
        | sessionpagevieworder  | bigint     |          |
        +-----------------------+------------+----------+--+

    这将显示表中的列。我们可以对此数据执行某些查询，不过，暂时让我们改为创建全新的表来演示如何将数据载入 Hive 并应用架构。
    
5. 输入以下语句，以使用 HDInsight 群集随附的示例数据来创建名为 **log4jLogs** 的新表：

        DROP TABLE log4jLogs;
        CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
        STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';
        SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;

    这些语句将执行以下操作：

    * **DROP TABLE** - 删除表和数据文件（如果该表已存在）。
    * **CREATE EXTERNAL TABLE** - 在 Hive 中创建新的“外部”表。外部表只会在 Hive 中存储表定义。数据将保留在原始位置。
    * **ROW FORMAT** - 告知 Hive 如何设置数据的格式。在此情况下，每个日志中的字段以空格分隔。
    * **STORED AS TEXTFILE LOCATION** - 让 Hive 知道数据的存储位置（example/data 目录），并且数据已存储为文本。
    * **SELECT** - 选择第 **t4** 列包含值 **[ERROR]** 的所有行的计数。这应会返回值 **3**，因为有三个行包含此值。
    * **INPUT\_\_FILE\_\_NAME LIKE '%.log'** - 告诉 Hive，我们只应返回以 .log 结尾的文件中的数据。通常，在使用 hive 查询时，同一个文件夹中只包含具有相同架构的数据，不过此示例日志文件将以其他数据格式存储。

    > [AZURE.NOTE] 当你预期以外部源更新基础数据（例如自动化数据上载过程），或以其他 MapReduce 操作更新基础数据，但希望 Hive 查询始终使用最新数据时，必须使用外部表。
    >
    > 删除外部表**不会**删除数据，只会删除表定义。
    
    此命令的输出应与以下内容相似：
    
        INFO  : Tez session hasn't been created yet. Opening session
        INFO  :
        
        INFO  : Status: Running (Executing on YARN cluster with App id application_1443698635933_0001)
        
        INFO  : Map 1: -/-      Reducer 2: 0/1
        INFO  : Map 1: 0/1      Reducer 2: 0/1
        INFO  : Map 1: 0/1      Reducer 2: 0/1
        INFO  : Map 1: 0/1      Reducer 2: 0/1
        INFO  : Map 1: 0/1      Reducer 2: 0/1
        INFO  : Map 1: 0(+1)/1  Reducer 2: 0/1
        INFO  : Map 1: 0(+1)/1  Reducer 2: 0/1
        INFO  : Map 1: 1/1      Reducer 2: 0/1
        INFO  : Map 1: 1/1      Reducer 2: 0(+1)/1
        INFO  : Map 1: 1/1      Reducer 2: 1/1
        +----------+--------+--+
        |   sev    | count  |
        +----------+--------+--+
        | [ERROR]  | 3      |
        +----------+--------+--+
        1 row selected (47.351 seconds)

4. 若要退出 Beeline，请使用 `!quit`。

##<a id="file"></a>运行 HiveQL 文件

Beeline 还可用于运行包含 HiveQL 语句的文件。使用以下步骤创建文件，然后使用 Beeline 运行该文件。

1. 使用以下命令创建一个名为 __query.hql__ 的新文件：

        nano query.hql
        
2. 编辑器打开后，使用以下内容作为该文件的内容。此查询将创建名为 **errorLogs** 的新“内部”表：

        CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
        INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';

    这些语句将执行以下操作：

    * **CREATE TABLE IF NOT EXISTS** - 创建表（如果该表尚不存在）。由于未使用 **EXTERNAL** 关键字，因此这是一个“内部”表，它存储在 Hive 数据仓库中并完全受 Hive 的管理。
    * **STORED AS ORC** - 以优化行纵栏表 (ORC) 格式存储数据。这是高度优化且有效的 Hive 数据存储格式。
    * **INSERT OVERWRITE ...SELECT** - 从包含 **[ERROR]** 的 **log4jLogs** 表中选择行，然后将数据插入 **errorLogs** 表中。
    
    > [AZURE.NOTE] 与外部表不同，删除内部表会同时删除基础数据。
    
3. 若要保存文件，请使用 __Ctrl__+___X_\_，然后输入 __Y__，最后按 __Enter\_\_。

4. 使用以下命令以通过 Beeline 运行该文件。将 __HOSTNAME__ 替换为前面获取的头节点名称，将 __PASSWORD__ 替换为 admin 帐户的密码：

        beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin -i query.hql

    > [AZURE.NOTE] `-i` 参数将启动 Beeline，运行 query.hql 文件中的语句，并在出现 `jdbc:hive2://localhost:10001/>` 提示符时保留在 Beeline 中。你也可以使用 `-f` 参数运行某个文件，以便在处理该文件后返回到 Bash。

5. 若要验证是否已创建 **errorLogs** 表，请使用以下语句从 **errorLogs** 返回所有行：

        SELECT * from errorLogs;

    应返回三行数据，所有行都包含列 t4 中的 **[ERROR]**：
    
        +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
        | errorlogs.t1  | errorlogs.t2  | errorlogs.t3  | errorlogs.t4  | errorlogs.t5  | errorlogs.t6  | errorlogs.t7  |
        +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
        | 2012-02-03    | 18:35:34      | SampleClass0  | [ERROR]       | incorrect     | id            |               |
        | 2012-02-03    | 18:55:54      | SampleClass1  | [ERROR]       | incorrect     | id            |               |
        | 2012-02-03    | 19:25:27      | SampleClass4  | [ERROR]       | incorrect     | id            |               |
        +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
        3 rows selected (1.538 seconds)

## 有关 Beeline 连接的详细信息

本文档中的步骤使用 `localhost` 连接到群集头节点上运行的 HiveServer2。还可以使用主机名或头节点完全限定的域名，此时需要其他步骤进行处理（查找主机名或 FQDN 的步骤）。使用来自头节点的 Beeline 时，使用 `localhost` 就足够了。

如果群集中具有安装有 Beeline 的边缘节点，则需要使用主机名或头节点的 FQDN 进行连接。

如果在群集外部的客户端上安装有 Beeline，则可以使用以下命令进行连接。将 __CLUSTERNAME__ 替换为 HDInsight 群集的名称。将 __PASSWORD__ 替换为管理员（HTTP 登录）帐户的密码。

    beeline -u 'jdbc:hive2://CLUSTERNAME.azurehdinsight.cn:443/default;ssl=true?hive.server2.transport.mode=http;hive.server2.thrift.http.path=hive2' -n admin -p PASSWORD

请注意，此时的参数/URI 与直接在头节点上运行或者从群集内的边缘节点运行时的不同。这是因为从 Internet 到群集的连接使用通过端口 443 路由流量的公用网关。此外，通过端口 443 上的公用网关公开一些其他服务，以便此时的 URI 与直接连接时的不同。从 Internet 连接时，还必须通过提供密码来验证该会话。

##<a id="summary"></a><a id="nextsteps"></a>后续步骤

如你所见，Beeline 命令提供了简单的方法让你以交互方式在 HDInsight 群集上运行 Hive 查询。

有关 HDInsight 中的 Hive 的一般信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

如果将 Tez 与 Hive 配合使用，请参阅以下文档以了解调试信息：

* [在基于 Windows 的 HDInsight 上使用 Tez UI](/documentation/articles/hdinsight-debug-tez-ui/)

* [Use the Ambari Tez view on Linux-based HDInsight（在基于 Linux 的 HDInsight 上使用 Ambari Tez 视图）](/documentation/articles/hdinsight-debug-ambari-tez-view/)

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

[putty]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html


[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/


[powershell-here-strings]: http://technet.microsoft.com/zh-cn/library/ee692792.aspx

<!---HONumber=Mooncake_1107_2016-->