<properties
    pageTitle="在 HDInsight 中使用 Hadoop Pig | Azure"
    description="了解如何将 Pig 与 HDInsight 上的 Hadoop 配合使用。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="acfeb52b-4b81-4a7d-af77-3e9908407404"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/12/2017"
    wacn.date="01/25/2017"
    ms.author="larryfr" />  


# 将 Pig 与 HDInsight 上的 Hadoop 配合使用
[AZURE.INCLUDE [pig-selector](../../includes/hdinsight-selector-use-pig.md)]

[Apache Pig](http://pig.apache.org/) 是一个可以使用名为 *Pig Latin* 的过程语言为 Hadoop 创建程序的平台。Pig 可以替代 Java 来创建 *MapReduce* 解决方案，并且已在 Azure HDInsight 中随附。

本文介绍了如何将 Pig 与 HDInsight 配合使用。

## <a id="why"></a>为何使用 Pig？
在 Hadoop 中使用 MapReduce 来处理数据时，其中一项难题是仅使用一个 map 和 reduce 函数来实现处理逻辑。进行复杂的处理时，通常必须将处理分解成多个 MapReduce 操作，这些操作合在一起即可获得所需结果。

然而，使用 Pig 可以将处理定义成一系列转换，相关数据经过这些转换即可生成所需输出，不必仅使用 map 和 reduce 函数。

通过 Pig Latin 语言，可以描述从原始输入开始，然后经过一个或多个转换，最终生成所需输出的数据流。Pig Latin 程序遵循下述常规模式：

* **加载**：从文件系统中读取要操作的数据
* **转换**：操作该数据
* **转储或存储**：将数据输出到屏幕或将其存储后等待处理

Pig Latin 还支持使用用户定义函数 (UDF) 来调用外部组件，以便实现难以在 Pig Latin 中建模的逻辑。

有关 Pig Latin 的详细信息，请参阅 [Pig Latin 参考手册 1](http://pig.apache.org/docs/r0.7.0/piglatin_ref1.html) 和 [Pig Latin 参考手册 2](http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html)。

如需通过 Pig 使用 UDF 的示例，请参阅以下文档：

* [在 HDInsight 中通过 Pig 使用 DataFu](/documentation/articles/hdinsight-hadoop-use-pig-datafu-udf/) - DataFu 是由 Apache 维护的有用 UDF 的集合
* [在 HDInsight 中将 Python 与 Pig 和 Hive 配合使用](/documentation/articles/hdinsight-python/)
* [在 HDInsight 中将 C# 与 Hive 和 Pig 配合使用](/documentation/articles/hdinsight-hadoop-hive-pig-udf-dotnet-csharp/)

## <a id="data"></a>关于示例数据
本示例使用 *log4j* 示例文件，该文件存储在 Blob 存储容器的 **/example/data/sample.log** 中。该文件中的每个日志都由一行字段组成，其中包含一个用于显示类型和严重性的 `[LOG LEVEL]` 字段，例如：

    2012-02-03 20:26:41 SampleClass3 [ERROR] verbose detail for id 1527353937

在前面的示例中，日志级别为 ERROR。

> [AZURE.NOTE]
你还可以使用 [Apache Log4j](http://zh.wikipedia.org/wiki/Log4j) 日志记录工具来生成 log4j 文件，然后将该文件上载到 Blob。请参阅[将数据上载到 HDInsight](/documentation/articles/hdinsight-upload-data/) 以获取相关说明。有关如何将 Azure 存储空间中的 Blob 用于 HDInsight 的详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-use-blob-storage/)。
> 
> 

示例数据存储在 Azure Blob 存储中，HDInsight 可以将该存储用作 Hadoop 群集的默认文件系统。HDInsight 可以使用 **wasb** 前缀来访问存储在 Blob 中的文件。例如，若要访问 sample.log 文件，可使用以下语法：

    wasbs:///example/data/sample.log

由于 WASB 是 HDInsight 的默认存储，也可以使用 Pig Latin 中的 **/example/data/sample.log** 来访问该文件。

> [AZURE.NOTE]
语法 **wasbs:///** 用于访问存储在 HDInsight 群集的默认存储容器中的文件。若在预配群集时指定了其他存储帐户，并且想要访问存储在这些帐户中的文件，可以通过指定容器名称和存储帐户地址来访问数据，例如：**wasbs://mycontainer@mystorage.blob.core.chinacloudapi.cn/example/data/sample.log**。
> 
> 

## <a id="job"></a>关于示例作业
下面的 Pig Latin 作业从 HDInsight 群集的默认存储加载 **sample.log** 文件。然后，它会执行一系列转换，对输入数据中出现的每个日志级别进行计数。结果转储到 STDOUT。

    LOGS = LOAD 'wasbs:///example/data/sample.log';
    LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
    FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
    GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
    FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
    RESULT = order FREQUENCIES by COUNT desc;
    DUMP RESULT;

下同详细显示了每个转换对数据的影响。

![转换的图形表示形式][image-hdi-pig-data-transformation]  


## <a id="run"></a>运行 Pig Latin 作业
HDInsight 可以使用各种方法来运行 Pig Latin 作业。使用下表来确定哪种方法最适合你，然后访问此链接进行演练。

| **使用此方法**，如果想要... | ...**交互式** shell | ...**批处理** | ...使用此**群集操作系统** | ...从此**客户端操作系统** |
|:--- |:---:|:---:|:--- |:--- |
| [SSH](/documentation/articles/hdinsight-hadoop-use-pig-ssh/) |✔ |✔ |Linux |Linux、Unix、Mac OS X 或 Windows |
| [Curl](/documentation/articles/hdinsight-hadoop-use-pig-curl/) |&nbsp; |✔ |Linux 或 Windows |Linux、Unix、Mac OS X 或 Windows |
| [.NET SDK for Hadoop](/documentation/articles/hdinsight-hadoop-use-pig-dotnet-sdk/) |&nbsp; |✔ |Linux 或 Windows |Windows（暂时） |
| [Windows PowerShell](/documentation/articles/hdinsight-hadoop-use-pig-powershell/) |&nbsp; |✔ |Linux 或 Windows |Windows |
| [远程桌面](/documentation/articles/hdinsight-hadoop-use-pig-remote-desktop/) |✔ |✔ |Windows |Windows |

> [AZURE.IMPORTANT]
Linux 是在 HDInsight 3.4 或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

## 使用本地 SQL Server Integration Services 在 Azure HDInsight 上运行 Pig 作业
也可以使用 SQL Server Integration Services (SSIS) 运行 Pig 作业。Azure Feature Pack for SSIS 提供以下组件，用于 HDInsight 上的 Pig 作业。

* [Azure HDInsight Pig 任务][pigtask]
* [Azure 订阅连接管理器][connectionmanager]

在[此处][ssispack]了解有关 Azure Feature Pack for SSIS 的详细信息。

## <a id="nextsteps"></a>后续步骤
现在，你已了解如何将 Pig 与 HDInsight 配合使用，请使用以下链接来学习 Azure HDInsight 的其他用法。

* [将数据上载到 HDInsight][hdinsight-upload-data]
* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [将 Sqoop 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-sqoop/)
* [将 Oozie 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-oozie/)
* [将 MapReduce 作业与 HDInsight 配合使用][hdinsight-use-mapreduce]

[apachepig-home]: http://pig.apache.org/
[putty]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
[curl]: http://curl.haxx.se/
[pigtask]: http://msdn.microsoft.com/zh-cn/library/mt146781(v=sql.120).aspx
[connectionmanager]: http://msdn.microsoft.com/zh-cn/library/mt146773(v=sql.120).aspx
[ssispack]: http://msdn.microsoft.com/zh-cn/library/mt146770(v=sql.120).aspx

[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/

[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell/

[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-mapreduce]: /documentation/articles/hdinsight-use-mapreduce/

[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/#mapreduce-sdk

[Powershell-install-configure]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs

[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx

[image-hdi-pig-data-transformation]: ./media/hdinsight-use-pig/HDI.DataTransformation.gif

<!---HONumber=Mooncake_0120_2017-->