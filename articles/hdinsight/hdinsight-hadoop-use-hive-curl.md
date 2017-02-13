<properties
    pageTitle="在 HDInsight 中将 Hadoop Hive 与 Curl 配合使用 | Azure"
    description="了解如何使用 Curl 向 HDInsight 远程提交 Pig 作业。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="6ce18163-63b5-4df6-9bb6-8fcbd4db05d8"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="09/07/2016"
    wacn.date="01/25/2017"
    ms.author="larryfr" />

# 使用 Curl 在 HDInsight 中以 Hadoop 运行 Hive 查询
[AZURE.INCLUDE [hive-selector](../../includes/hdinsight-selector-use-hive.md)]

本文介绍如何使用 Curl 在 Azure HDInsight 群集上对 Hadoop 运行 Hive 查询。

本文档使用 Curl 演示如何通过使用原始 HTTP 请求来与 HDInsight 交互，以便运行、监视和检索 Hive 查询的结果。要执行这些操作，需要使用 HDInsight 群集提供的 WebHCat REST API（前称 Templeton）。

> [AZURE.NOTE]
如果你已熟悉如何使用基于 Linux 的 Hadoop 服务器，但刚接触 HDInsight，请参阅[基于 Linux 的 HDInsight 上的 Hadoop 须知信息](/documentation/articles/hdinsight-hadoop-linux-information/)。

## <a id="prereq"></a>先决条件
要完成本文中的步骤，需要：

* HDInsight 群集上的 Hadoop
* [Curl](http://curl.haxx.se/)
* [jq](http://stedolan.github.io/jq/)

## <a id="curl"></a>通过使用 Curl 运行 Hive 查询
> [AZURE.NOTE]
使用 Curl 或者与 WebHCat 进行任何其他形式的 REST 通信时，必须提供 HDInsight 群集管理员用户名和密码对请求进行身份验证。此外，还必须使用群集名称作为用来向服务器发送请求的统一资源标识符 (URI) 的一部分。
><p> 
> 对本部分中的所有命令，请将 **USERNAME** 替换为在群集上进行身份验证的用户，并将 **PASSWORD** 替换为用户帐户的密码。将 **CLUSTERNAME** 替换为群集名称。
><p> 
> REST API 通过[基本身份验证](http://en.wikipedia.org/wiki/Basic_access_authentication)进行保护。你始终应该使用安全 HTTP (HTTPS) 来发出请求，以确保安全地将凭据发送到服务器。
> 
> 

1. 在命令行中，使用以下命令验证是否可以连接到 HDInsight 群集。
   
        curl -u USERNAME:PASSWORD -G https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/status
   
    你应会收到类似于下面的响应：
   
        {"status":"ok","version":"v1"}
   
    此命令中使用的参数如下：
   
    * **-u** - 用来对请求进行身份验证的用户名和密码。
    * **-G** - 指出这是 GET 请求。
     
    所有请求的 URL 开头 (**https://CLUSTERNAME.azurehdinsight.cn/templeton/v1**) 都是一样的。路径 **/status** 指示请求将返回服务器的 WebHCat（也称为 Templeton）状态。还可以通过使用以下命令请求 Hive 的版本：
     
        curl -u USERNAME:PASSWORD -G https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/version/hive
     
    这应该会返回如下响应：
     
        {"module":"hive","version":"0.13.0.2.1.6.0-2103"}
2. 使用以下命令创建名为 **log4jLogs** 的新表：
   
        curl -u USERNAME:PASSWORD -d user.name=USERNAME -d execute="set+hive.execution.engine=tez;DROP+TABLE+log4jLogs;CREATE+EXTERNAL+TABLE+log4jLogs(t1+string,t2+string,t3+string,t4+string,t5+string,t6+string,t7+string)+ROW+FORMAT+DELIMITED+FIELDS+TERMINATED+BY+' '+STORED+AS+TEXTFILE+LOCATION+'wasbs:///example/data/';SELECT+t4+AS+sev,COUNT(*)+AS+count+FROM+log4jLogs+WHERE+t4+=+'[ERROR]'+AND+INPUT__FILE__NAME+LIKE+'%25.log'+GROUP+BY+t4;" -d statusdir="wasbs:///example/curl" https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/hive
   
    此命令中使用的参数如下：
   
    * **-d** - 由于未使用 `-G`，请求默认为 POST 方法。`-d` 指定与请求一起发送的数据值。
     
        * **user.name** - 正在运行命令的用户。
        * **execute** - 要执行的 HiveQL 语句。
        * **statusdir** - 此作业的状态要写入到的目录。
     
        这些语句可执行以下操作：
    * **DROP TABLE** - 删除表和数据文件（如果该表已存在）。
    * **CREATE EXTERNAL TABLE** - 在 Hive 中创建新“外部”表。外部表仅在 Hive 中存储表定义。数据将保留在原始位置。
     
        > [AZURE.NOTE]
        如果希望以外部源更新基础数据（例如自动化数据上载过程），或以其他 MapReduce 操作更新基础数据，但希望 Hive 查询始终使用最新数据时，必须使用外部表。
        ><p> 
        > 删除外部表**不会**删除数据，只会删除表定义。
        > 
        > 
    * **ROW FORMAT** - 告知 Hive 如何设置数据的格式。在本例中，每个日志中的字段以空格分隔。
    * **STORED AS TEXTFILE LOCATION** - 让 Hive 知道数据的存储位置（example/data 目录），并且数据已存储为文本。
    * **SELECT** - 选择第 **t4** 列包含值 **[ERROR]** 的所有行的计数。这应会返回值 **3**，因为有三个行包含此值。
     
        > [AZURE.NOTE]
        请注意，在与 Curl 配合使用时，将用 `+` 字符替换 HiveQL 语句之间的空格。如果带引号的值包含空格（例如分隔符），则不应替换为 `+`。
        > 
        > 
    * **INPUT\_\_FILE\_\_NAME LIKE '%25.log'** - 此项会对搜索进行限制，仅使用以 .log 结尾的文件。如果此项不存在，Hive 会尝试搜索此目录及其子目录中的所有文件，包括不符合为此表定义的列架构的文件。
     
        > [AZURE.NOTE]
        请注意，%25 是 % 的 URL 编码形式，因此实际条件是 `like '%.log'`。% 必须是 URL 编码的，因为系统将其视为 URL 中的特殊字符。
        > 
        > 
     
        此命令应会返回可用来检查作业状态的作业 ID。
     
            {"id":"job_1415651640909_0026"}
3. 若要检查作业的状态，请使用以下命令。将 **JOBID** 替换为上一步骤返回的值。例如，如果返回值为 `{"id":"job_1415651640909_0026"}`，则 **JOBID** 将是 `job_1415651640909_0026`。
   
        curl -G -u USERNAME:PASSWORD -d user.name=USERNAME https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/jobs/JOBID | jq .status.state
   
    如果作业已完成，状态将是 **SUCCEEDED**。
   
    > [AZURE.NOTE]
    此 Curl 请求返回具有作业相关信息的 JavaScript 对象表示法 (JSON) 文档；使用 jq 可以仅检索状态值。
    > 
    > 
4. 在作业的状态更改为 **SUCCEEDED** 后，可以从 Azure Blob 存储中检索作业的结果。随查询一起传递的 `statusdir` 参数包含输出文件的位置；在本例中为 **wasbs:///example/curl**。此地址会将作业的输出存储在 HDInsight 群集所用的默认存储容器的 **example/curl** 目录中。
   
    可以使用 [Azure CLI](/documentation/articles/xplat-cli-install/) 列出并下载这些文件。例如，若要列出 **example/curl** 中的文件，请使用以下命令：
   
        azure storage blob list <container-name> example/curl
   
    若要下载文件，请使用以下命令：
   
        azure storage blob download <container-name> <blob-name> <destination-file>
   
    > [AZURE.NOTE]
    必须使用 `-a` 和 `-k` 参数指定包含 Blob 的存储帐户名称，或者设置 **AZURE\_STORAGE\_ACCOUNT** 和 **AZURE\_STORAGE\_ACCESS\_KEY** 环境变量。请参阅 <a href="/documentation/articles/hdinsight-upload-data" target="\_blank" 了解详细信息。
    > 
    > 
5. 使用以下语句创建名为 **errorLogs** 的新“内部”表：
   
        curl -u USERNAME:PASSWORD -d user.name=USERNAME -d execute="set+hive.execution.engine=tez;CREATE+TABLE+IF+NOT+EXISTS+errorLogs(t1+string,t2+string,t3+string,t4+string,t5+string,t6+string,t7+string)+STORED+AS+ORC;INSERT+OVERWRITE+TABLE+errorLogs+SELECT+t1,t2,t3,t4,t5,t6,t7+FROM+log4jLogs+WHERE+t4+=+'[ERROR]'+AND+INPUT__FILE__NAME+LIKE+'%25.log';SELECT+*+from+errorLogs;" -d statusdir="wasbs:///example/curl" https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/hive
   
    这些语句可执行以下操作：
   
    * **CREATE TABLE IF NOT EXISTS** - 创建表（如果该表不存在）。由于未使用 **EXTERNAL** 关键字，因此这是一个内部表，它存储在 Hive 数据仓库中并完全由 Hive 管理。
     
        > [AZURE.NOTE]
        与外部表不同，删除内部表会同时删除基础数据。
        > 
        > 
    * **STORED AS ORC** - 以优化行纵栏表 (ORC) 格式存储数据。这是高度优化且有效的 Hive 数据存储格式。
    * **INSERT OVERWRITE ...SELECT** - 从包含 **[ERROR]** 的 **log4jLogs** 表中选择行，然后将数据插入 **errorLogs** 表中。
    * **SELECT** - 选择新 **errorLogs** 表中的所有行。

6. 使用返回的作业 ID 检查作业的状态。成功后，如前所述使用 Azure CLI 下载并查看结果。输出应包含三行，其中所有行都包含 **[ERROR]**。

## <a id="summary"></a>摘要

如本文档中所示，可以使用原始 HTTP 请求来运行、监视和查看 HDInsight 群集上的 Hive 作业的结果。

有关本文中使用的 REST 接口的详细信息，请参阅 <a href="https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference" target="_blank">WebHCat 参考</a>。

## <a id="nextsteps"></a>后续步骤

有关将 Hive 与 HDInsight 配合使用的一般信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

有关 HDInsight 上 Hadoop 的其他使用方法的信息：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

如果将 Tez 与 Hive 配合使用，请参阅以下文档，了解调试信息：

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



[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/

[powershell-here-strings]: http://technet.microsoft.com/zh-cn/library/ee692792.aspx

<!---HONumber=Mooncake_0120_2017-->