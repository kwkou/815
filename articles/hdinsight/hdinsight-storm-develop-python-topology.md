<properties
    pageTitle="使用 Python 组件的 Apache Storm - Azure HDInsight | Azure"
    description="了解如何创建使用 Python 组件的 Apache Storm 拓扑。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    keywords="apache storm python" />
<tags
    ms.assetid="edd0ec4f-664d-4266-910c-6ecc94172ad8"
    ms.service="hdinsight"
    ms.custom="hdinsightactive,hdiseo17may2017"
    ms.devlang="python"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="05/12/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="752bab0a597a5ad14194b560a9a486ec4114fe7f"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="develop-apache-storm-topologies-using-python-on-hdinsight"></a>在 HDInsight 上使用 Python 开发 Apache Storm 拓扑

了解如何创建使用 Python 组件的 Apache Storm 拓扑。 Apache Storm 支持多种语言，甚至允许将几种语言的组件组合到一个拓扑中。 借助 Flux 框架（通过 Storm 0.10.0 引入），可以轻松地创建使用 Python 组件的解决方案。

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
> 本文档中的信息已使用 Storm on HDInsight 3.5 进行测试。 Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 弃用日期](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

此项目的代码位于 [https://github.com/Azure-Samples/hdinsight-python-storm-wordcount](https://github.com/Azure-Samples/hdinsight-python-storm-wordcount)。

## <a name="prerequisites"></a>先决条件

* Python 2.7 或更高版本

* Java JDK 1.8 或更高版本

* Maven 3

* （可选）本地 Storm 开发环境。 仅当想要在本地运行拓扑时，才需要本地 Storm 环境。 有关详细信息，请参阅[设置开发环境](http://storm.apache.org/releases/1.0.1/Setting-up-development-environment.html)。

## <a name="storm-multi-language-support"></a>Storm 多语言支持

Apache Storm 设计为与使用任何编程语言编写的组件配合使用。 组件必须了解如何使用 Storm 的 Thrift 定义。 对于 Python，会以 Apache Storm 项目的一部分提供模块，让用户可以轻松与 Storm 进行交互。 可以在 [https://github.com/apache/storm/blob/master/storm-multilang/python/src/main/resources/resources/storm.py](https://github.com/apache/storm/blob/master/storm-multilang/python/src/main/resources/resources/storm.py) 上找到此模块。

Storm 是在 Java 虚拟机 (JVM) 上运行的 Java 进程。 使用其他语言编写的组件作为子进程执行。 Storm 使用通过 stdin/stdout 发送的 JSON 消息与这些子进程通信。 有关组件间通信的更多详细信息，请参阅 [Multi-lang Protocol](https://storm.apache.org/documentation/Multilang-protocol.html)（多语言协议）文档。

## <a name="python-with-the-flux-framework"></a>使用 Flux 框架的 Python

借助 Flux 框架，可独立于组件定义 Storm 拓扑。 Flux 框架使用 YAML 定义 Storm 拓扑。 下面的文本举例说明如何在 YAML 文档中引用 Python 组件：

    # Spout definitions
    spouts:
      - id: "sentence-spout"
        className: "org.apache.storm.flux.wrappers.spouts.FluxShellSpout"
        constructorArgs:
          # Command line
          - ["python", "sentencespout.py"]
          # Output field(s)
          - ["sentence"]
        # parallelism hint
        parallelism: 1

类 `FluxShellSpout` 用于启动实现 spout 的 `sentencespout.py` 脚本。

Flux 需要 Python 脚本位于包含拓扑的 jar 文件内的 `/resources` 目录中。 因此，此示例将 Python 脚本存储在 `/multilang/resources` 目录中。 `pom.xml` 使用以下 XML 包含此文件：

    <!-- include the Python components -->
    <resource>
        <directory>${basedir}/multilang</directory>
        <filtering>false</filtering>
    </resource>

如前所述，存在实现 Storm 的 Thrift 定义的 `storm.py` 文件。 Flux 框架在生成项目时自动包含 `storm.py`，无需额外执行操作。

## <a name="build-the-project"></a>生成项目

在项目的根目录中，使用以下命令：

    mvn clean compile package

此命令可创建 `target/WordCount-1.0-SNAPSHOT.jar` 文件，其中包含已编译的拓扑。

## <a name="run-the-topology-locally"></a>在本地运行拓扑

若要在本地运行拓扑，请使用以下命令：

    storm jar WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux -l -R /topology.yaml

> [AZURE.NOTE]
> 此命令要求提供本地 Storm 开发环境。 有关详细信息，请参阅[设置开发环境](http://storm.apache.org/releases/1.0.1/Setting-up-development-environment.html)

拓扑启动后，它会向本地控制台发出类似如下文本的信息：

    24302 [Thread-25-sentence-spout-executor[4 4]] INFO  o.a.s.s.ShellSpout - ShellLog pid:2436, name:sentence-spout Emiting the cow jumped over the moon
    24302 [Thread-30] INFO  o.a.s.t.ShellBolt - ShellLog pid:2438, name:splitter-bolt Emitting the
    24302 [Thread-28] INFO  o.a.s.t.ShellBolt - ShellLog pid:2437, name:counter-bolt Emitting years:160
    24302 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=the, count=599}
    24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=seven, count=302}
    24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=dwarfs, count=143}
    24303 [Thread-25-sentence-spout-executor[4 4]] INFO  o.a.s.s.ShellSpout - ShellLog pid:2436, name:sentence-spout Emiting the cow jumped over the moon
    24303 [Thread-30] INFO  o.a.s.t.ShellBolt - ShellLog pid:2438, name:splitter-bolt Emitting cow
    24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=four, count=160}

若要停止拓扑，请使用 __Ctrl + C__。

## <a name="run-the-storm-topology-on-hdinsight"></a>在 HDInsight 上运行 Storm 拓扑

1. 使用以下命令将 `WordCount-1.0-SNAPSHOT.jar` 文件复制到 Storm on HDInsight 群集：

        scp target\WordCount-1.0-SNAPSHOT.jar sshuser@mycluster-ssh.azurehdinsight.cn

    将 `sshuser` 替换为群集的 SSH 用户。 将 `mycluster` 替换为群集名称。 系统可能会提示输入 SSH 用户的密码。

    有关使用 SSH 和 SCP 的详细信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

2. 文件上传后，使用 SSH 连接到群集：

        ssh sshuser@mycluster-ssh.azurehdinsight.cn

3. 在 SSH 会话中，使用以下命令在群集上启动拓扑：

        storm jar WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux -r -R /topology.yaml

3. Storm UI 可以用于查看群集上的拓扑。 Storm UI 位于 https://mycluster.azurehdinsight.cn/stormui。 将 `mycluster` 替换为群集名称。

> [AZURE.NOTE]
> 启动后，Storm 拓扑会一直运行，直到被停止。 若要停止拓扑，可使用以下方法之一：
> <p>* 从命令行运行 `storm kill TOPOLOGYNAME` 命令
> <p>* Storm UI 中的“终止”按钮。

## <a name="next-steps"></a>后续步骤

请参阅以下文档，了解配合使用 Python 和 HDInsight 的其他方式：

* [如何使用 Python 流式处理 MapReduce 作业](/documentation/articles/hdinsight-hadoop-streaming-python/)
* [如何在 Pig 和 Hive 中使用 Python 用户定义函数 (UDF) ](/documentation/articles/hdinsight-python/)

<!--Update_Description: wording update-->