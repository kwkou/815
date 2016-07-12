<properties
   pageTitle="在 HDInsight 中将 Hadoop Pig 与 Curl 配合使用 | Azure"
   description="学习如何使用 Curl 在 Azure HDInsight 中的 Hadoop 群集上运行 Pig Latin 作业。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="03/18/2016"
	wacn.date="04/26/2016"/>

#使用 Curl 配合 HDInsight 上的 Hadoop 运行 Pig 作业

[AZURE.INCLUDE [pig-selector](../includes/hdinsight-selector-use-pig.md)]

在本文档中，你将学习如何使用 Curl 在 Azure HDInsight 群集上运行 Pig Latin 作业。可以使用 Pig Latin 编程语言来描述应用到输入数据以生成所需输出的转换。

本文档使用 Curl 演示如何使用原始 HTTP 请求来与 HDInsight 交互，以便运行、监视和检索 Pig 作业的结果。若要执行这些操作，需要使用 HDInsight 群集提供的 WebHCat REST API（前称 Templeton）。

##<a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* Azure HDInsight（HDInsight 上的 Hadoop）群集（基于 Windows）

* [Curl](http://curl.haxx.se/)

* [jq](http://stedolan.github.io/jq/)

##<a id="curl"></a>通过使用 Curl 运行 Pig 作业

> [AZURE.NOTE]使用 Curl 或者与 WebHCat 进行任何其他形式的 REST 通信时，必须提供 HDInsight 群集管理员用户名和密码对请求进行身份验证。此外，还必须使用群集名称作为用来向服务器发送请求的统一资源标识符 (URI) 的一部分。
> <p>对本部分中的所有命令，请将 **USERNAME** 替换为在群集上进行身份验证群集的用户，并将 **PASSWORD** 替换为用户帐户的密码。将 **CLUSTERNAME** 替换为群集名称。
> <p>REST API 通过[基本访问身份验证](http://en.wikipedia.org/wiki/Basic_access_authentication)进行保护。你始终应该使用安全 HTTP (HTTPS) 来发出请求，以确保安全地将凭据发送到服务器。

1. 在命令行中，使用以下命令验证你是否可以连接到 HDInsight 群集。

        curl -u USERNAME:PASSWORD -G https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/status

    你应会收到类似于下面的响应：

        {"status":"ok","version":"v1"}

    此命令中使用的参数如下：

    * **-u**：用来对请求进行身份验证的用户名和密码。
    * **-G**：指示这是 GET 请求

    所有请求的 URL 开头 ****https://CLUSTERNAME.azurehdinsight.cn/templeton/v1** 都是一样的。路径 **/status** 指示请求是要返回服务器的 WebHCat（也称为 Templeton）状态。

2. 使用以下代码将 Pig Latin 作业提交到群集：

        curl -u USERNAME:PASSWORD -d user.name=USERNAME -d execute="LOGS=LOAD+'wasb:///example/data/sample.log';LEVELS=foreach+LOGS+generate+REGEX_EXTRACT($0,'(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)',1)+as+LOGLEVEL;FILTEREDLEVELS=FILTER+LEVELS+by+LOGLEVEL+is+not+null;GROUPEDLEVELS=GROUP+FILTEREDLEVELS+by+LOGLEVEL;FREQUENCIES=foreach+GROUPEDLEVELS+generate+group+as+LOGLEVEL,COUNT(FILTEREDLEVELS.LOGLEVEL)+as+count;RESULT=order+FREQUENCIES+by+COUNT+desc;DUMP+RESULT;" -d statusdir="wasb:///example/pigcurl" https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/pig

    此命令中使用的参数如下：

    * **-d**：由于未使用 `-G`，请求默认为 POST 方法。`-d` 指定与请求一起发送的数据值。

        * **user.name**：正在运行命令的用户
        * **execute**：要执行的 Pig Latin 语句
        * **statusdir**：此作业的状态要写入到的目录

    > [AZURE.NOTE]请注意，在与 Curl 配合使用时，将使用 `+` 字符替换 Pig Latin 语句中的空格。

    此命令应会返回可用来检查作业状态的作业 ID，例如：

        {"id":"job_1415651640909_0026"}

3. 若要检查作业的状态，请使用以下命令。将 **JOBID** 替换为上一步骤返回的值。例如，如果返回值为 `{"id":"job_1415651640909_0026"}`，则 **JOBID** 将是 `job_1415651640909_0026`。

        curl -G -u USERNAME:PASSWORD -d user.name=USERNAME https://CLUSTERNAME.azurehdinsight.cn/templeton/v1/jobs/JOBID | jq .status.state

	如果作业已完成，状态将是 **SUCCEEDED**。

    > [AZURE.NOTE]此 Curl 请求返回具有作业相关信息的 JavaScript 对象表示法 (JSON) 文档；使用 jq 可以仅检索状态值。

##<a id="results"></a>查看结果

在作业的状态更改为 **SUCCEEDED** 后，你可以从 Azure Blob 存储中检索作业的结果。随查询一起传递的 `statusdir` 参数包含输出文件的位置；在本例中为 ****wasb:///example/pigcurl**。此地址会将作业的输出存储在 HDInsight 群集所用的默认存储容器的 **example/pigcurl** 目录中。

可以使用 [Azure CLI](/documentation/articles/xplat-cli-install/) 列出并下载这些文件。例如，若要列出 **example/pigcurl** 中的文件，请使用以下命令：

	azure storage blob list <container-name> example/pigcurl

若要下载文件，请使用以下命令：

	azure storage blob download <container-name> <blob-name> <destination-file>

> [AZURE.NOTE]你必须使用 `-a` 和 `-k` 参数指定包含 Blob 的存储帐户名称，或者设置 **AZURE\_STORAGE\_ACCOUNT** 和 **AZURE\_STORAGE\_ACCESS\_KEY** 环境变量。

##<a id="summary"></a>摘要

如本文档中所示，你可以使用原始 HTTP 请求来运行、监视和查看 HDInsight 群集上的 Pig 作业的结果。

有关本文中使用的 REST 接口的详细信息，请参阅 [WebHCat 参考](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference)。

##<a id="nextsteps"></a>后续步骤

有关 HDInsight 上的 Pig 的一般信息：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=Mooncake_1207_2015-->