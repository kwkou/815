<properties
    pageTitle="在 HDInsight 中将 Hadoop Hive 与 Curl 配合使用 | Azure"
    description="了解如何使用 Curl 向 HDInsight 远程提交 Pig 作业。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="6ce18163-63b5-4df6-9bb6-8fcbd4db05d8"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="04/14/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="ff16f1e2766eaf0d733d5eac15cdacd61d1706b0"
    ms.lasthandoff="04/28/2017" />

# <a name="run-hive-queries-with-hadoop-in-hdinsight-using-rest"></a>使用 REST 在 HDInsight 中通过 Hadoop 运行 Hive 查询

[AZURE.INCLUDE [hive-selector](../../includes/hdinsight-selector-use-hive.md)]

了解如何使用 WebHCat REST API 在 Azure HDInsight 群集上通过 Hadoop 运行 Hive 查询。

[Curl](http://curl.haxx.se/) 用于演示如何使用原始 HTTP 请求与 HDInsight 交互。 [jq](http://stedolan.github.io/jq/) 实用工具用于处理从 REST 请求返回的 JSON 数据。

> [AZURE.NOTE]
> 如果已熟悉如何使用基于 Linux 的 Hadoop 服务器，但刚接触 HDInsight，请参阅 [HDInsight 上的基于 Linux 的 Hadoop 须知信息](/documentation/articles/hdinsight-hadoop-linux-information/)文档。

## <a id="curl"></a>运行 Hive 查询

> [AZURE.NOTE]
> 使用 cURL 或者与 WebHCat 进行任何其他形式的 REST 通信时，必须提供 HDInsight 群集管理员的用户名和密码以对请求进行身份验证。
> <p>
> 对本部分中的所有命令，请将 **USERNAME**替换为在群集上进行身份验证的用户，并将 **PASSWORD** 替换为用户帐户的密码。 将 **CLUSTERNAME** 替换为群集名称。
> <p>
> REST API 通过 [基本身份验证](http://en.wikipedia.org/wiki/Basic_access_authentication)进行保护。 为了有助于确保将凭据安全地发送到服务器，应始终使用安全 HTTP (HTTPS) 发出请求。

1. 在命令行中，使用以下命令验证你是否可以连接到 HDInsight 群集。

        curl -u USERNAME:PASSWORD -G https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/status

    将收到类似于以下文本的响应：

        {"status":"ok","version":"v1"}

    此命令中使用的参数如下：

    * **-u** - 用来对请求进行身份验证的用户名和密码。
    * **-G** - 指示此请求是一个 GET 操作。

    所有请求的 URL 开头都是 **https://CLUSTERNAME.azurehdinsight.cn/templeton/v1**。 路径 **/status** 指示请求将返回服务器的 WebHCat（也称为 Templeton）状态。 你还可以通过使用以下命令请求 Hive 的版本：

        curl -u USERNAME:PASSWORD -G https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/version/hive

    此请求返回的响应类似于以下文本：

        {"module":"hive","version":"0.13.0.2.1.6.0-2103"}

2. 使用以下命令创建名为 **log4jLogs** 的表：

        curl -u USERNAME:PASSWORD -d user.name=USERNAME -d execute="set+hive.execution.engine=tez;DROP+TABLE+log4jLogs;CREATE+EXTERNAL+TABLE+log4jLogs(t1+string,t2+string,t3+string,t4+string,t5+string,t6+string,t7+string)+ROW+FORMAT+DELIMITED+FIELDS+TERMINATED+BY+' '+STORED+AS+TEXTFILE+LOCATION+'/example/data/';SELECT+t4+AS+sev,COUNT(*)+AS+count+FROM+log4jLogs+WHERE+t4+=+'[ERROR]'+AND+INPUT__FILE__NAME+LIKE+'%25.log'+GROUP+BY+t4;" -d statusdir="/example/curl" https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/hive

    以下参数与此请求配合使用：

    * **-d** - 由于未使用 `-G`，请求将按默认使用 POST 方法。 `-d` 指定与请求一起发送的数据值。

        * **user.name** - 正在运行命令的用户。
        * **execute** - 要执行的 HiveQL 语句。
        * **statusdir** - 此作业的状态要写入到的目录。

        这些语句将执行以下操作：
    * **DROP TABLE** - 如果表已存在，则删除该表。
    * **CREATE EXTERNAL TABLE**：在 Hive 中创建新的“外部”表。 外部表仅在 Hive 中存储表定义。 数据将保留在原始位置。

        > [AZURE.NOTE]
        > 如果希望通过外部源更新基础数据，应使用外部表。 例如，使用自动化数据上传过程或其他 MapReduce 操作。
        > <p>
        > 删除外部表**不会**删除数据，只会删除表定义。

    * **ROW FORMAT** - 如何设置数据的格式。 每个日志中的字段都用空格分隔。
    * **STORED AS TEXTFILE LOCATION** - 数据的存储位置（example/data 目录），以及数据已存储为文本。
    * **SELECT** - 选择第 **t4** 列包含值 **[ERROR]** 的所有行的计数。 此语句返回的值为 **3**，因为有三行包含此值。

        > [AZURE.NOTE]
        > 请注意，在与 Curl 配合使用时，将用 `+` 字符替换 HiveQL 语句之间的空格。 如果带引号的值包含空格（例如分隔符），则不应替换为 `+`。

    * **INPUT__FILE__NAME LIKE '%25.log'** - 此语句对搜索设置了限制，仅使用以 .log 结尾的文件。
    
        > [AZURE.NOTE]
        > `%25` 是 % 的 URL 编码形式，因此实际条件是 `like '%.log'`。 % 必须是 URL 编码的，因为系统将其视为 URL 中的特殊字符。

        此命令应会返回可用来检查作业状态的作业 ID。

            {"id":"job_1415651640909_0026"}

3. 若要检查作业的状态，请使用以下命令：

        curl -G -u USERNAME:PASSWORD -d user.name=USERNAME https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/jobs/JOBID | jq .status.state

    将“JOBID”替换为上一步骤返回的值。 例如，如果返回值为 `{"id":"job_1415651640909_0026"}`，则 **JOBID** 将是 `job_1415651640909_0026`。

    如果作业已完成，状态将是 **SUCCEEDED**。

    > [AZURE.NOTE]
    > 此 Curl 请求返回含有作业相关信息的 JavaScript 对象表示法 (JSON) 文档。 jq 用于仅检索状态值。

4. 在作业的状态更改为“SUCCEEDED”后，可以从 Azure Blob 存储中检索作业的结果。 随查询一起传递的 `statusdir` 参数包含输出文件的位置；在本例中，位置为 **/example/curl**。 此地址在群集默认存储的 **example/curl** 目录中存储输出。

    可以使用 [Azure CLI](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli) 列出并下载这些文件。 有关将 Azure CLI 与 Azure 存储配合使用的详细信息，请参阅[将 Azure CLI 2.0 与 Azure 存储配合使用](/documentation/articles/storage-azure-cli/#create-and-manage-blobs)文档。

5. 使用以下语句创建名为 **errorLogs** 的新“内部”表：

        curl -u USERNAME:PASSWORD -d user.name=USERNAME -d execute="set+hive.execution.engine=tez;CREATE+TABLE+IF+NOT+EXISTS+errorLogs(t1+string,t2+string,t3+string,t4+string,t5+string,t6+string,t7+string)+STORED+AS+ORC;INSERT+OVERWRITE+TABLE+errorLogs+SELECT+t1,t2,t3,t4,t5,t6,t7+FROM+log4jLogs+WHERE+t4+=+'[ERROR]'+AND+INPUT__FILE__NAME+LIKE+'%25.log';SELECT+*+from+errorLogs;" -d statusdir="/example/curl" https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/hive

    这些语句将执行以下操作：

    * **CREATE TABLE IF NOT EXISTS** - 创建表（如果该表不存在）。 此语句创建内部表，该表存储在 Hive 数据仓库中，并完全由 Hive 管理。

        > [AZURE.NOTE]
        > 与外部表不同，删除内部表会同时删除基础数据。

    * **STORED AS ORC** - 以优化行纵栏表 (ORC) 格式存储数据。 ORC 是高度优化且有效的 Hive 数据存储格式。
    * **INSERT OVERWRITE ...SELECT** - 从包含 **[ERROR]** 的 **log4jLogs** 表中选择行，然后将数据插入 **errorLogs** 表中。
    * **SELECT** - 选择新 **errorLogs** 表中的所有行。

6. 使用返回的作业 ID 检查作业的状态。 成功后，如前所述使用 Azure CLI 下载并查看结果。 输出应包含三行，其中所有行都包含 **[ERROR]**。

## <a id="nextsteps"></a>后续步骤

有关将 Hive 与 HDInsight 配合使用的一般信息：

* [将 Hive 与 Hadoop on HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)

有关 HDInsight 上 Hadoop 的其他使用方法的信息：

* [将 Pig 与 Hadoop on HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

如果将 Tez 与 Hive 配合使用，请参阅以下文档以了解调试信息：

* [在基于 Linux 的 HDInsight 上使用 Ambari Tez 视图](/documentation/articles/hdinsight-debug-ambari-tez-view/)

有关在本文档中使用的 REST API 的详细信息，请参阅 [WebHCat 参考](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference)文档。

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

[hdinsight-provision]: /documentation/articles/hdinsight-hadoop-provision-linux-clusters/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/

[powershell-here-strings]: http://technet.microsoft.com/zh-cn/library/ee692792.aspx

<!--Update_Description: wording update-->