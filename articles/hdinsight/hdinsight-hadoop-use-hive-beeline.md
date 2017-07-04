<properties
    pageTitle="将 Beeline 与 Apache Hive 配合使用 - Azure HDInsight | Azure"
    description="了解如何使用 Beeline 客户端通过 Hadoop on HDInsight 运行 Hive 查询。 Beeline 是用于通过 JDBC 操作 HiveServer2 的实用工具。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    keywords="beeline hive,hive beeline" />
<tags
    ms.assetid="3adfb1ba-8924-4a13-98db-10a67ab24fca"
    ms.service="hdinsight"
    ms.custom="hdinsightactive,hdiseo17may2017"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="04/05/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="ad1aacee73edca9a8067817974d0919ed14edc27"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="use-the-beeline-client-with-apache-hive"></a>将 Beeline 客户端与 Apache Hive 配合使用

了解如何使用 [Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline-NewCommandLineShell) 在 HDInsight 上运行 Hive 查询。

Beeline 是一个 Hive 客户端，包含在 HDInsight 群集的头节点上。 Beeline 使用 JDBC 连接到 HiveServer2，后者是 HDInsight 群集上托管的一项服务。 还可以使用 Beeline 通过 Internet 远程访问 Hive on HDInsight。 下表提供与 Beeline 结合使用的连接字符串：

| 运行 Beeline 的位置 | Parameters |
| --- | --- |
| 与头节点或边缘节点的 SSH 连接 | `-u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -n admin` |
| 群集外部 | `-u 'jdbc:hive2://clustername.azurehdinsight.cn:443/;ssl=true;transportMode=http;httpPath=/hive2' -n admin -p password` |

> [AZURE.NOTE]
> 将 `admin` 替换为群集的群集登录帐户。
><p>
> 将 `password` 替换为群集登录帐户的密码。
><p>
> 将 `clustername` 替换为 HDInsight 群集的名称。

## <a id="prereq"></a>先决条件

* 基于 Linux 的 HDInsight 上的 Hadoop 群集。

    > [AZURE.IMPORTANT]
    > Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 版本弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

* SSH 客户端或本地 Beeline 客户端。 本文档中的大多数步骤都假定从与群集的 SSH 会话使用 Beeline。 有关从群集外部运行 Beeline 的信息，请参阅[远程使用 Beeline](#remote) 部分。

    有关使用 SSH 的详细信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

## <a id="beeline"></a>使用 Beeline

1. 启动 Beeline 时，必须提供用于 HDInsight 群集上 HiveServer2 的连接字符串。 还必须提供用于群集登录的帐户名（通常为 `admin`）。 如果从群集外部运行命令，则还必须提供群集登录密码。 可使用下表查找要使用的连接字符串格式和参数：

    | 运行 Beeline 的位置 | Parameters |
    | --- | --- | --- |
    | 与头节点或边缘节点的 SSH 连接 | `-u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -n admin` |
    | 群集外部 | `-u 'jdbc:hive2://clustername.azurehdinsight.cn:443/;ssl=true;transportMode=http;httpPath=/hive2' -n admin -p password` |

    例如，可以使用以下命令从与群集的 SSH 会话启动 Beeline：

        beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -n admin

    此命令启动 Beeline 客户端，并连接到群集头节点上的 HiveServer2。 `-n` 参数用于提供群集登录帐户。 默认登录名是 `admin`。 如果在群集创建过程中使用了其他名称，则使用该名称而不是 `admin`。

    命令完成后，将出现 `jdbc:hive2://headnodehost:10001/>` 提示符。

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

    * `DROP TABLE` - 如果表存在，则将其删除。

    * `CREATE EXTERNAL TABLE` - 在 Hive 中创建一个**外部**表。 外部表只会在 Hive 中存储表定义。 数据将保留在原始位置。

    * `ROW FORMAT` - 如何设置数据的格式。 在此情况下，每个日志中的字段以空格分隔。

    * `STORED AS TEXTFILE LOCATION` - 数据存储位置和文件格式。

    * `SELECT` - 选择 **t4** 列包含值 **[ERROR]** 的所有行的计数。 此查询返回值 **3**，因为有三行包含此值。

    * `INPUT__FILE__NAME LIKE '%.log'` - Hive 会尝试向目录中的所有文件应用架构。 在此示例中，目录包含与架构不匹配的文件。 为防止结果中包含垃圾数据，此语句指示 Hive 应当仅返回以 .log 结尾的文件中的数据。

    > [AZURE.NOTE]
    > 如果希望通过外部源更新基础数据，应使用外部表。 例如，自动化数据上传进程或 MapReduce 操作。
    ><p>
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

## <a id="file"></a>使用 Beeline 运行 HiveQL 文件

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

        beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -n admin -i query.hql

    > [AZURE.NOTE]
    > `-i` 参数将启动 Beeline，运行 query.hql 文件中的语句。 查询完成后，会出现 `jdbc:hive2://headnodehost:10001/>` 提示符。 还可以使用 `-f` 参数运行文件，该参数在查询完成后会退出 Beeline。

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

## <a id="remote"></a>远程使用 Beeline

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

<!--Update_Description: wording update-->