<properties
    pageTitle="使用 Beeline 来操作 HDInsight (Hadoop) 上的 Hive | Azure"
    description="了解如何使用 SSH 连接到 HDInsight 中的 Hadoop 群集，然后使用 Beeline 以交互方式提交 Hive 查询。 Beeline 是用于通过 JDBC 操作 HiveServer2 的实用工具。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="3adfb1ba-8924-4a13-98db-10a67ab24fca"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="04/05/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="0ce7089552bdf5a652fc10a23dfab1a13160f323"
    ms.lasthandoff="04/28/2017" />

# <a name="use-hive-with-hadoop-in-hdinsight-with-beeline"></a>通过 Beeline 将 Hive 与 HDInsight 中的 Hadoop 配合使用
[AZURE.INCLUDE [hive-selector](../../includes/hdinsight-selector-use-hive.md)]

了解如何使用 [Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline-NewCommandLineShell) 在 HDInsight 上运行 Hive 查询。

Beeline 是一个命令行工具，包含在 HDInsight 群集的头节点上。 它使用 JDBC 连接 HiveServer2，后者是 HDInsight 群集上托管的服务。 下表提供与 Beeline 结合使用的连接字符串：

| 运行 Beeline 的位置 | 连接字符串 | 其他参数 |
| --- | --- | --- |
| 到头节点的 SSH 连接 | `jdbc:hive2://localhost:10001/;transportMode=http` | `-n admin` |
| 边缘节点 | `jdbc:hive2://headnodehost:10001/;transportMode=http` | `-n admin` |
| 群集外部 | `jdbc:hive2://clustername.azurehdinsight.cn:443/;ssl=true;transportMode=http;httpPath=/hive2` | `-n admin -p password` |

> [AZURE.NOTE]
> 将“admin”替换为群集的群集登录帐户。
><p>
> 将“password”替换为群集登录帐户的密码。
><p>
> 将 `clustername` 替换为 HDInsight 群集的名称。

## <a id="prereq"></a>先决条件

* 基于 Linux 的 HDInsight 上的 Hadoop 群集。

    > [AZURE.IMPORTANT]
    > Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅[弃用 HDInsight 版本 3.2 和 3.3](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

* SSH 客户端。 有关使用 SSH 的详细信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

## <a id="ssh"></a>使用 SSH 进行连接

使用以下命令通过 SSH 连接到群集：

    ssh sshuser@myhdinsight-ssh.azurehdinsight.cn

将 `sshuser` 替换为群集的 SSH 帐户。 将 `myhdinsight` 替换为 HDInsight 群集的名称。

如果创建 HDInsight 群集时**提供了 SSH 身份验证密码**，则必须在系统提示时提供该密码。

如果创建 HDInsight 群集时**提供了 SSH 身份验证的证书密钥**，则可能需要指定客户端系统上的私钥位置：

    ssh -i ~/.ssh/mykey.key ssh@myhdinsight-ssh.azurehdinsight.cn

有关将 SSH 与 HDInsight 配合使用的详细信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

## <a id="beeline"></a>使用 Beeline 命令

1. 在 SSH 会话中，使用以下命令启动 Beeline：

        beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin

    此命令启动 Beeline 客户端，并连接到群集头节点上的 HiveServer2。 `-n` 参数用于提供群集登录帐户。 默认登录名是 `admin`。 如果在群集创建过程中使用了其他名称，则使用该名称而不是 `admin`。

    命令完成后，将出现 `jdbc:hive2://localhost:10001/>` 提示符。

2. Beeline 命令以 `!` 字符开头，例如，`!help` 将显示帮助。 但是，`!` 对于某些命令可以省略。 例如，`help` 也是有效的。

    有一个 `!sql`，用于执行 HiveQL 语句。 但是，由于 HiveQL 非常流行，因此可以省略前面的 `!sql`。 以下两个语句等效：

        !sql show tables;
        show tables;

    在新群集上，只会列出一个表：**hivesampletable**。

3. 使用以下命令显示 hivesampletable 的架构：

        describe hivesampletable;

    此命令返回以下信息：

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

    此信息描述表中的列。 我们可以对此数据执行某些查询，不过，暂时让我们改为创建全新的表来演示如何将数据载入 Hive 并应用架构。

4. 输入以下语句，以使用 HDInsight 群集随附的示例数据来创建名为 **log4jLogs** 的表：

        DROP TABLE log4jLogs;
        CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
        STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';
        SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;

    这些语句将执行以下操作：

    * **DROP TABLE** -如果表存在，则将其删除。

    * **CREATE EXTERNAL TABLE** - 在 Hive 中创建“外部”表。 外部表只会在 Hive 中存储表定义。 数据将保留在原始位置。

    * **ROW FORMAT** - 数据的设格式置方式。 在此情况下，每个日志中的字段以空格分隔。

    * **STORED AS TEXTFILE LOCATION** - 数据存储位置和文件格式。

    * **SELECT** - 选择第 **t4** 列包含值 **[ERROR]** 的所有行的计数。 此查询返回值 **3**，因为有三行包含此值。

    * **INPUT__FILE__NAME LIKE '%.log'** - 示例数据文件与其他文件一起存储。 此语句将查询范围限制为以 .log 结尾的文件中存储的数据。

    > [AZURE.NOTE]
    > 如果希望通过外部源更新基础数据，应使用外部表。 例如，自动化数据上传进程或 MapReduce 操作。
    > <p>
    > 删除外部表**不会**删除数据，只会删除表定义。

    此命令的输出类似于以下文本：

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

5. 若要退出 Beeline，请使用 `!exit`。

## <a id="file"></a>运行 HiveQL 文件

使用以下步骤创建文件，然后使用 Beeline 运行该文件。

1. 使用以下命令创建一个名为 **query.hql** 的文件：

        nano query.hql

2. 将以下文本用作文件的内容。 此查询创建名为 **errorLogs** 的新“内部”表：

        CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
        INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';

    这些语句将执行以下操作：

    * **CREATE TABLE IF NOT EXISTS** - 创建表（如果该表尚不存在）。 因为未使用 **EXTERNAL** 关键字，此语句创建内部表。 内部表存储在 Hive 数据仓库中，由 Hive 全权管理。
    * **STORED AS ORC**：以优化行纵栏表 (ORC) 格式存储数据。 ORC 格式是高度优化且有效的 Hive 数据存储格式。
    * **INSERT OVERWRITE ...SELECT** - 从包含 **[ERROR]** 的 **log4jLogs** 表中选择行，然后将数据插入 **errorLogs** 表中。

    > [AZURE.NOTE]
    > 与外部表不同，删除内部表会同时删除基础数据。

3. 若要保存文件，请使用 **Ctrl**+**_X**，然后输入 **Y**，最后按 **Enter**。

4. 使用以下命令以通过 Beeline 运行该文件。 将 **HOSTNAME** 替换为前面获取的头节点名称，将 **PASSWORD** 替换为 admin 帐户的密码：

        beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin -i query.hql

    > [AZURE.NOTE]
    > `-i` 参数将启动 Beeline，运行 query.hql 文件中的语句。 查询完成后，会出现 `jdbc:hive2://localhost:10001/>` 提示符。 还可以使用 `-f` 参数运行文件，该参数在查询完成后会退出 Beeline。

5. 若要验证是否已创建 **errorLogs** 表，请使用以下语句从 **errorLogs** 返回所有行：

        SELECT * from errorLogs;

    应返回三行数据，所有行都包含 t4 列中的 **[ERROR]**：

        +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
        | errorlogs.t1  | errorlogs.t2  | errorlogs.t3  | errorlogs.t4  | errorlogs.t5  | errorlogs.t6  | errorlogs.t7  |
        +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
        | 2012-02-03    | 18:35:34      | SampleClass0  | [ERROR]       | incorrect     | id            |               |
        | 2012-02-03    | 18:55:54      | SampleClass1  | [ERROR]       | incorrect     | id            |               |
        | 2012-02-03    | 19:25:27      | SampleClass4  | [ERROR]       | incorrect     | id            |               |
        +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
        3 rows selected (1.538 seconds)

## <a name="edge-nodes"></a>边缘节点

如果群集具有边缘节点，建议在使用 SSH 进行连接时始终使用边缘节点而非头节点。 若要通过指向边缘节点的 SSH 连接来启动 Beeline，请使用以下命令：

    beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -n admin

## <a name="remote-clients"></a>远程客户端

如果本地安装了 Beeline，或者通过 Docker 映像（如 [sutoiku/beeline](https://hub.docker.com/r/sutoiku/beeline/)）使用它，必须使用以下参数：

* __连接字符串__：`-u 'jdbc:hive2://clustername.azurehdinsight.cn:443/;ssl=true;transportMode=http;httpPath=/hive2'`

* __群集登录名__：`-n admin`

* __群集登录密码__`-p 'password'`

将连接字符串中的 `clustername` 替换为 HDInsight 群集名称。

将 `admin` 替换为群集登录名称，并将 `password` 替换为群集登录密码。

## <a id="summary"></a><a id="nextsteps"></a>后续步骤

有关 HDInsight 中 Hive 的更多常规信息，请参阅以下文档：

* [将 Hive 与 Hadoop on HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)

若要深入了解使用 Hadoop on HDInsight 的其他方法，请参阅以下文档：

* [将 Pig 与 Hadoop on HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

如果将 Tez 与 Hive 配合使用，请参阅以下文档：

* [在基于 Windows 的 HDInsight 上使用 Tez UI](/documentation/articles/hdinsight-debug-tez-ui/)
* [在基于 Linux 的 HDInsight 上使用 Ambari Tez 视图](/documentation/articles/hdinsight-debug-ambari-tez-view/)

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

[hdinsight-provision]: /documentation/articles/hdinsight-hadoop-provision-linux-clusters/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/

[powershell-here-strings]: http://technet.microsoft.com/zh-cn/library/ee692792.aspx