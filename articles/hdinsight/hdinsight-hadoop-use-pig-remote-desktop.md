<properties
    pageTitle="在 HDInsight 中将 Pig 与远程桌面配合使用 | Azure"
    description="了解如何使用 Pig 命令从到基于 Windows 的 HDInsight Hadoop 群集的远程桌面连接运行 Pig Latin 语句。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="e034a286-de0f-465f-8bf1-3d085ca6abed"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/17/2017"
    wacn.date="03/10/2017"
    ms.author="larryfr" />

# 从远程桌面连接运行 Pig 作业
[AZURE.INCLUDE [pig-selector](../../includes/hdinsight-selector-use-pig.md)]

本文档演练了如何使用 Pig 命令从到基于 Windows 的 HDInsight 群集的远程桌面连接运行 Pig Latin 语句。Pig Latin 允许通过描述数据转换创建 MapReduce 应用程序，而不是创建映射和化简函数。

> [AZURE.IMPORTANT]
远程桌面只能在使用 Windows 作为操作系统的 HDInsight 群集上使用。Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。
><p>
> 有关 HDInsight 3.4 或更高版本，请参阅[将 Pig 与 HDInsight 和 SSH 配合使用](/documentation/articles/hdinsight-hadoop-use-pig-ssh/)，了解如何通过命令行直接在群集上以交互方式运行 Pig 作业。

## <a id="prereq"></a>先决条件
若要完成本文中的步骤，你将需要以下各项：

* 基于 Windows 的 HDInsight（HDInsight 上的 Hadoop）群集
* 运行 Windows 10、Windows 8 或 Windows 7 的客户端计算机

## <a id="connect"></a>使用远程桌面进行连接
为 HDInsight 群集启用远程桌面，然后根据[使用 RDP 连接到 HDInsight 群集](/documentation/articles/hdinsight-administer-use-management-portal/#connect-to-clusters-using-rdp)中的说明连接到该群集。

## <a id="pig"></a>使用 Pig 命令
1. 在建立远程桌面连接后，通过使用桌面上的图标启动 **Hadoop 命令行**。
2. 使用以下方法启动 Pig 命令：

        %pig_home%\bin\pig

    系统将为你提供 `grunt>` 提示符。
3. 输入以下语句：

        LOGS = LOAD 'wasbs:///example/data/sample.log';

    此命令会将 sample.log 文件的内容加载到 LOGS 文件中。你可以通过使用以下命令查看该文件的内容：

        DUMP LOGS;
4. 通过应用正则表达式从每个记录中仅提取日志记录级别来转换数据：

        LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;

    转换后，你可以使用 **DUMP** 查看数据。在本例中为 `DUMP LEVELS;`。
5. 使用以下语句继续应用转换。使用 `DUMP` 查看每个步骤后的转换结果。

    <table>
    <tr>
    <th>语句</th><th>作用</th>
    </tr>
    <tr>
    <td>FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;</td><td>删除包含日志级别 Null 值的行，并将结果存储到 FILTEREDLEVELS。</td>
    </tr>
    <tr>
    <td>GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;</td><td>按日志级别对行进行分组，并将结果存储到 GROUPEDLEVELS。</td>
    </tr>
    <tr>
    <td>FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;</td><td>创建一组新的数据，其中包含每个唯一日志级别值和其发生次数。此数据将存储到 FREQUENCIES</td>
    </tr>
    <tr>
    <td>RESULT = order FREQUENCIES by COUNT desc;</td><td>按计数为日志级别排序（降序），并存储到 RESULT</td>
    </tr>
    </table>
6. 你也可以使用 `STORE` 语句保存转换结果。例如，以下命令将 `RESULT` 保存到群集的默认存储容器上的 **/example/data/pigout** 目录：

        STORE RESULT into 'wasbs:///example/data/pigout'

    > [AZURE.NOTE]
    数据将存储到文件中名为 **part-nnnnn** 的指定目录。如果该目录已存在，你将收到错误消息。
    >
    >
7. 若要退出 grunt 提示符，请输入以下语句。

        QUIT;

### Pig Latin 批处理文件
你也可以使用 Pig 命令运行文件中包含的 Pig Latin。

1. 退出 grunt 提示符之后，请打开“记事本”，并在 **%PIG\_HOME%** 目录中创建名为 **pigbatch.pig** 的新文件。
2. 在 **pigbatch.pig** 文件中键入或粘贴以下行，然后保存它：

        LOGS = LOAD 'wasbs:///example/data/sample.log';
        LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
        FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
        GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
        FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
        RESULT = order FREQUENCIES by COUNT desc;
        DUMP RESULT;
3. 使用以下命令，使用 pig 命令运行 **pigbatch.pig** 文件。

        pig %PIG_HOME%\pigbatch.pig

    在批处理作业完成后，你应该会看到以下输出，该输出应该与先前步骤中使用 `DUMP RESULT;` 时相同：

        (TRACE,816)
        (DEBUG,434)
        (INFO,96)
        (WARN,11)
        (ERROR,6)
        (FATAL,2)

## <a id="summary"></a>摘要
如你所见，Pig 命令允许你以交互方式运行 MapReduce 操作，或运行存储在批处理文件中的 Pig Latin 作业。

## <a id="nextsteps"></a>后续步骤
有关 HDInsight 中的 Pig 的一般信息：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned-->