<properties
    pageTitle="在 HDInsight 中使用 Hadoop Pig | Azure"
    description="了解如何将 Pig 与 HDInsight 上的 Hadoop 配合使用。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="acfeb52b-4b81-4a7d-af77-3e9908407404"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="04/03/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="632c48b3ce48c9df9c2721608d244124cc156d6b"
    ms.lasthandoff="04/28/2017" />

# <a name="use-pig-with-hadoop-on-hdinsight"></a>将 Pig 与 Hadoop on HDInsight 配合使用

了解如何将 [Apache Pig](http://pig.apache.org/) 与 HDInsight 配合使用...

Pig 是一个平台，可以使用名为 *Pig Latin* 的过程语言为 Hadoop 创建程序。 Pig 可以替代 Java 来创建 *MapReduce* 解决方案，它已包括在 Azure HDInsight 中。 使用下表可以找出将 Pig 与 HDInsight 配合使用的各种方法：

| **使用此方法**，如果你想要... | ... **交互式** shell | ...**批处理** | ...使用此 **群集操作系统** | ...从此 **客户端操作系统** |
|:--- |:---:|:---:|:--- |:--- |
| [SSH](/documentation/articles/hdinsight-hadoop-use-pig-ssh/) |✔ |✔ |Linux |Linux、Unix、Mac OS X 或 Windows |
| [REST API](/documentation/articles/hdinsight-hadoop-use-pig-curl/) |&nbsp; |✔ |Linux 或 Windows |Linux、Unix、Mac OS X 或 Windows |
| [.NET SDK for Hadoop](/documentation/articles/hdinsight-hadoop-use-pig-dotnet-sdk/) |&nbsp; |✔ |Linux 或 Windows |Windows（暂时） |
| [Windows PowerShell](/documentation/articles/hdinsight-hadoop-use-pig-powershell/) |&nbsp; |✔ |Linux 或 Windows |Windows |
| [远程桌面](/documentation/articles/hdinsight-hadoop-use-pig-remote-desktop/)（HDInsight 3.2 和 3.3） |✔ |✔ |Windows |Windows |

> [AZURE.IMPORTANT]
> Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅[弃用 HDInsight 3.2 和 3.3](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

## <a id="why"></a>为何使用 Pig

在 Hadoop 中使用 MapReduce 来处理数据时，其中一项难题是仅使用映射和化简函数来实现处理逻辑。 进行复杂的处理时，你通常必须将处理分解成多个 MapReduce 操作，这些操作合在一起即可获得所需的结果。

使用 Pig，可以将处理定义成一系列转换，相关数据经过这些转换即可生成所需的输出。

使用 Pig Latin 语言，你可以通过一个或多个转换从原始输入描述数据流，以便生成所需的输出。 Pig Latin 程序遵循下述常规模式：

* **加载**：从文件系统中读取要操作的数据

* **转换**：操作该数据

* **转储或存储**：将数据输出到屏幕或将其存储后再进行处理

### <a name="user-defined-functions"></a>用户定义的函数

Pig Latin 还支持使用用户定义函数 (UDF) 来调用外部组件，以便实现难以在 Pig Latin 中建模的逻辑。

有关 Pig Latin 的详细信息，请参阅 [Pig Latin Reference Manual 1](http://pig.apache.org/docs/r0.7.0/piglatin_ref1.html)（Pig Latin 参考手册 1）和 [Pig Latin Reference Manual 2](http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html)（Pig Latin 参考手册 2）。

如需通过 Pig 使用 UDF 的示例，请参阅以下文档：

* [在 HDInsight 中通过 Pig 使用 DataFu](/documentation/articles/hdinsight-hadoop-use-pig-datafu-udf/) - DataFu 是由 Apache 维护的有用 UDF 的集合
* [在 HDInsight 中将 Python 与 Pig 和 Hive 配合使用](/documentation/articles/hdinsight-python/)
* [在 HDInsight 中将 C# 与 Hive 和 Pig 配合使用](/documentation/articles/hdinsight-hadoop-hive-pig-udf-dotnet-csharp/)

## <a id="data"></a>示例数据

HDInsight 提供各种示例数据集，它们存储在 `/example/data` 和 `/HdiSamples` 目录中。 这些目录位于群集的默认存储中。 本文档中的 Pig 示例使用来自 `/example/data/sample.log` 的 *log4j* 文件。

该文件中的每个日志都包含一行字段，其中包含一个 `[LOG LEVEL]` 字段，用于显示类型和严重性，例如：

    2012-02-03 20:26:41 SampleClass3 [ERROR] verbose detail for id 1527353937

在前面的示例中，日志级别为 ERROR。

> [AZURE.NOTE]
> 还可以使用 [Apache Log4j](http://zh.wikipedia.org/wiki/Log4j) 日志记录工具来生成 log4j 文件，然后将该文件上传到 Blob。 请参阅[将数据上传到 HDInsight](/documentation/articles/hdinsight-upload-data/) 以获取相关说明。 有关如何将 Azure 存储中的 Blob 与 HDInsight 配合使用的详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-use-blob-storage/)。

## <a id="job"></a>示例作业

下面的 Pig Latin 作业从 HDInsight 群集的默认存储加载 `sample.log` 文件。 然后，它会执行一系列转换，以便对输入数据中出现的每个日志级别进行计数。 结果会写入 STDOUT。

    LOGS = LOAD 'wasbs:///example/data/sample.log';
    LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
    FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
    GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
    FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
    RESULT = order FREQUENCIES by COUNT desc;
    DUMP RESULT;

下图概要显示每个转换对数据的影响。

![转换的图形表示形式][image-hdi-pig-data-transformation]

## <a id="run"></a>运行 Pig Latin 作业

HDInsight 可以使用各种方法来运行 Pig Latin 作业。 使用下表来确定哪种方法最适合你，然后访问此链接进行演练。

| **使用此方法** ，如果想要... | ... **交互式** shell | ...**批处理** | ...使用此**群集操作系统** | ...通过此**客户端** |
|:--- |:---:|:---:|:--- |:--- |
| [SSH](/documentation/articles/hdinsight-hadoop-use-pig-ssh/) |✔ |✔ |Linux |Linux、Unix、Mac OS X 或 Windows |
| [Curl](/documentation/articles/hdinsight-hadoop-use-pig-curl/) |&nbsp; |✔ |Linux 或 Windows |Linux、Unix、Mac OS X 或 Windows |
| [.NET SDK for Hadoop](/documentation/articles/hdinsight-hadoop-use-pig-dotnet-sdk/) |&nbsp; |✔ |Linux 或 Windows |Windows（暂时） |
| [Windows PowerShell](/documentation/articles/hdinsight-hadoop-use-pig-powershell/) |&nbsp; |✔ |Linux 或 Windows |Windows |
| [远程桌面](/documentation/articles/hdinsight-hadoop-use-pig-remote-desktop/)（HDInsight 3.2 和 3.3） |✔ |✔ |Windows |Windows |

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
> Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅[弃用 HDInsight 3.2 和 3.3](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

## <a name="pig-and-sql-server-integration-services"></a>Pig 和 SQL Server Integration Services

可以使用 SQL Server Integration Services (SSIS) 来运行 Pig 作业。 Azure Feature Pack for SSIS 提供适用于 HDInsight 上的 Pig 作业的以下组件。

* [Azure HDInsight Pig 任务][pigtask]

* [Azure 订阅连接管理器][connectionmanager]

在 [此处][ssispack]了解有关 Azure Feature Pack for SSIS 的详细信息。

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

[hdinsight-provision]: /documentation/articles/hdinsight-hadoop-provision-linux-clusters/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/#mapreduce-sdk

[Powershell-install-configure]: https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs

[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx

[image-hdi-pig-data-transformation]: ./media/hdinsight-use-pig/HDI.DataTransformation.gif