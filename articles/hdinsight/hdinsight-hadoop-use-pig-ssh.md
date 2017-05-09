<properties
    pageTitle="在 HDInsight 群集上配合使用 Hadoop Pig 和 SSH | Azure"
    description="了解如何使用 SSH 连接到基于 Linux 的 Hadoop 群集，然后使用 Pig 命令以交互方式或以批处理作业形式运行 Pig Latin 语句。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="b646a93b-4c51-4ba4-84da-3275d9124ebe"
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
    ms.openlocfilehash="ea0811520e8fcb9be25887acd9191d7da9f7a57c"
    ms.lasthandoff="04/28/2017" />

# <a name="run-pig-jobs-on-a-linux-based-cluster-with-the-pig-command-ssh"></a>使用 Pig 命令 (SSH) 在基于 Linux 的群集上运行 Pig 作业

[AZURE.INCLUDE [pig-selector](../../includes/hdinsight-selector-use-pig.md)]

了解如何以交互方式从 HDInsight 群集的 SSH 连接中运行 Pig 作业。 可以使用 Pig Latin 编程语言来描述应用到输入数据以生成所需输出的转换。

> [AZURE.IMPORTANT]
> 本文档中的步骤要求使用基于 Linux 的 HDInsight 群集。 Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

## <a id="ssh"></a>使用 SSH 进行连接

使用 SSH 连接到 HDInsight 群集。 以下示例连接到名为 **myhdinsight** 的群集，以作为名为 **sshuser** 的帐户：

    ssh sshuser@myhdinsight-ssh.azurehdinsight.cn

如果创建 HDInsight 群集时**提供了 SSH 身份验证的证书密钥**，则可能需要指定客户端系统上的私钥位置。

    ssh sshuser@myhdinsight-ssh.azurehdinsight.cn -i ~/mykey.key

如果在创建 HDInsight 群集时**提供了 SSH 身份验证密码**，请在系统提示时提供该密码。

有关将 SSH 与 HDInsight 配合使用的详细信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

## <a id="pig"></a>使用 Pig 命令

1. 连接后，请使用以下命令启动 Pig 命令行接口 (CLI)：

        pig

    稍后，你应该会看到 `grunt>` 提示符。

2. 输入以下语句：

        LOGS = LOAD '/example/data/sample.log';

    此命令会将 sample.log 文件的内容载入到 LOGS。 可以使用以下语句查看该文件的内容：

        DUMP LOGS;

3. 接下来，应用正则表达式以使用以下语句仅提取每条记录的日志记录级别，从而转换数据：

        LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;

    转换后，可以使用 **DUMP** 来查看数据。 在这种情况下，使用 `DUMP LEVELS;`。

4. 使用下表中的语句继续应用转换：

    | Pig Latin 语句 | 语句的作用 |
    | ---- | ---- |
    | `FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;` | 删除包含日志级别 null 值的行，并将结果存储到 `FILTEREDLEVELS` 中。 |
    | `GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;` | 按日志级别对行进行分组，并将结果存储到 `GROUPEDLEVELS` 中。 |
    | `FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;` | 创建包含每个唯一日志级别值及其发生次数的数据集。 该数据集将存储到 `FREQUENCIES` 中。 |
    | `RESULT = order FREQUENCIES by COUNT desc;` | 按计数为日志级别排序（降序），并存储到 `RESULT` 中。 |

    > [AZURE.TIP]
    > 使用 `DUMP` 查看每个步骤后的转换结果。

5. 你也可以使用 `STORE` 语句保存转换结果。 例如，以下语句将 `RESULT` 保存到群集的默认存储的 `/example/data/pigout` 目录：

        STORE RESULT into '/example/data/pigout';

    > [AZURE.NOTE]
    > 该数据存储在名为 `part-nnnnn` 的文件的指定目录中。 如果该目录已存在，则将收到错误。

6. 若要退出 grunt 提示符，请输入以下语句：

        QUIT;

### <a name="pig-latin-batch-files"></a>Pig Latin 批处理文件

你也可以使用 Pig 命令运行文件中包含的 Pig Latin。

1. 退出 grunt 提示符之后，请使用以下命令将 STDIN 发送到名为 `pigbatch.pig` 的文件中。 此文件创建于 SSH 用户帐户的主目录中。

        cat > ~/pigbatch.pig

2. 输入或粘贴以下行，然后在完成后按 Ctrl+D。

        LOGS = LOAD '/example/data/sample.log';
        LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
        FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
        GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
        FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
        RESULT = order FREQUENCIES by COUNT desc;
        DUMP RESULT;

3. 使用以下命令，以使用 Pig 命令运行 `pigbatch.pig` 文件。

        pig ~/pigbatch.pig

    批处理作业完成后，将看到以下输出：

        (TRACE,816)
        (DEBUG,434)
        (INFO,96)
        (WARN,11)
        (ERROR,6)
        (FATAL,2)

## <a id="nextsteps"></a>后续步骤

有关 HDInsight 中 Pig 的常规信息，请参阅以下文档：

* [将 Pig 与 Hadoop on HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)

有关使用 HDInsight 上 Hadoop 的其他方式的详细信息，请参阅以下文档：

* [将 Hive 与 Hadoop on HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!--Update_Description: wording update-->