<properties
    pageTitle="在基于 Linux 的 HDInsight (Hadoop) 上安装并使用 Giraph | Azure"
    description="了解如何使用脚本操作在基于 Linux 的 HDInsight 群集上安装 Giraph。 脚本操作可让你通过更改群集配置或安装服务和实用工具，在创建期间自定义群集。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="9fcac906-8f06-4002-9fe8-473e42f8fd0f"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/08/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="24b735ad828670041ff636a7fed491319eece2ba"
    ms.lasthandoff="04/28/2017" />

# <a name="install-giraph-on-hdinsight-hadoop-clusters-and-use-giraph-to-process-large-scale-graphs"></a>在 HDInsight Hadoop 群集上安装 Giraph 并使用 Giraph 处理大型图形

你可以通过使用 **脚本操作** 自定义群集，在 Azure HDInsight 上 Hadoop 中的任何一种群集上安装 Giraph。

本主题介绍如何使用脚本操作安装 Giraph。 一旦你已安装 Giraph，你还将了解如何将 Giraph 用于大多数典型应用程序，也就是处理大型图形。

> [AZURE.IMPORTANT]
> 本文档中的步骤需要使用 Linux 的 HDInsight 群集。 Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 在 Windows 上即将弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

## <a name="whatis"></a>什么是 Giraph？

[Apache Giraph](http://giraph.apache.org/) 可让你使用 Hadoop 执行图形处理，并可以在 Azure HDInsight 上使用。 图形可为对象之间的关系建模，例如，为 Internet 之类的大型网络上的路由器之间的连接建模，或者为社交网络上的人物之间的关系建模（有时称为社交图形）。 通过图形处理，可以推理图形中对象之间的关系，例如：

* 根据当前的关系识别潜在的朋友。

* 识别网络中两台计算机之间的最短路由。

* 计算网页的排名。

> [AZURE.WARNING]
> 完全支持通过 HDInsight 群集提供的组件，Azure 支持部门将帮助找出并解决与这些组件相关的问题。
> <p>
> 自定义组件（如 Giraph）可获得合理范围的支持，以帮助你进一步排查问题。 这可能会促进解决问题，或要求使用可用的开源技术渠道，在渠道中可找到该技术的深厚的专业知识。 有许多可以使用的社区站点，例如：[HDInsight 的 MSDN 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hdinsight)和 [Azure CSDN](http://azure.csdn.net)。 此外，Apache 项目在 [http://apache.org](http://apache.org) 上提供了项目站点，例如 [Hadoop](http://hadoop.apache.org/)。

## <a name="what-the-script-does"></a>脚本功能

此脚本可执行以下操作：

* 将 Giraph 安装到 `/usr/hdp/current/giraph`

* 将 `giraph-examples.jar` 文件复制到群集的默认存储 (WASB)：`/example/jars/giraph-examples.jar`

## <a name="install"></a>使用脚本操作安装 Giraph

用于在 HDInsight 群集上安装 Giraph 的示例脚本位于下列位置。

    https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh

本部分说明如何在通过 Azure 门户预览创建群集时使用示例脚本。

> [AZURE.NOTE]
> Azure PowerShell、Azure CLI、HDInsight .NET SDK 或 Azure Resource Manager 模板也可用于应用脚本操作。 你也可以将脚本操作应用于已在运行的群集。 有关详细信息，请参阅[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)。

1. 使用[创建基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-create-linux-clusters-portal/)中的步骤开始创建群集，但是不完成创建。

2. 在“可选配置”边栏选项卡上，选择“脚本操作”，并提供以下信息：

    * **名称**：输入脚本操作的友好名称。

    * **脚本 URI**：https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh

    * **标头**：选中此选项

    * **辅助角色**：不选择此选项

    * **ZOOKEEPER**：不选择此选项

    * **参数**：将此字段留空

3. 在“脚本操作”的底部，使用“选择”按钮保存配置。 最后，使用“可选配置”边栏选项卡底部的“选择”按钮保存可选配置信息。

4. 根据[创建基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-create-linux-clusters-portal/)中所述继续创建群集。

## <a name="usegiraph"></a>如何在 HDInsight 中使用 Giraph？

完成群集创建之后，便可执行以下步骤来运行 Giraph 随附的 SimpleShortestPathsComputation 示例。 这会使用基本 [Pregel](http://people.apache.org/~edwardyoon/documents/pregel.pdf) 实现来查找图形中对象之间最短的路径。

1. 使用 SSH 连接到 HDInsight 群集：

        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn

    有关信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

2. 使用以下命令创建名为 **tiny_graph.txt** 的新文件：

        nano tiny_graph.txt

    使用以下项作为此文件的内容：

        [0,0,[[1,1],[3,3]]]
        [1,0,[[0,1],[2,2],[3,1]]]
        [2,0,[[1,2],[4,4]]]
        [3,0,[[0,3],[1,1],[4,4]]]
        [4,0,[[3,4],[2,4]]]

    此数据使用 `[source_id, source_value,[[dest_id], [edge_value],...]]` 格式描述定向图形中对象之间的关系。 每一行代表 `source_id` 对象与一个或多个 `dest_id` 对象之间的关系。 `edge_value`（或权重）可被视为 `source_id` 和 `dest\_id` 之间的连接强度或距离。

    使用表示对象间距离的值（或权重）绘制图形后，上述数据可能与下面类似。

    ![tiny_graph.txt 中的对象绘制为圆圈，线条表示对象之间的不同距离](./media/hdinsight-hadoop-giraph-install-linux/giraph-graph.png)

3. 若要保存文件，请使用 **Ctrl+X**，然后输入“Y”，最后按 **Enter** 以接受文件名。

4. 使用以下命令将数据存储到 HDInsight 群集的主存储中：

        hdfs dfs -put tiny_graph.txt /example/data/tiny_graph.txt

5. 使用以下命令运行 SimpleShortestPathsComputation 示例。

        yarn jar /usr/hdp/current/giraph/giraph-examples.jar org.apache.giraph.GiraphRunner org.apache.giraph.examples.SimpleShortestPathsComputation -ca mapred.job.tracker=headnodehost:9010 -vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat -vip /example/data/tiny_graph.txt -vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat -op /example/output/shortestpaths -w 2

    下表介绍了与此命令搭配使用的参数：

    | 参数 | 作用 |
    | --- | --- |
    | `jar /usr/hdp/current/giraph/giraph-examples.jar` |包含示例的 jar 文件。 |
    | `org.apache.giraph.GiraphRunner` |用于启动示例的类。 |
    | `org.apache.giraph.examples.SimpleShortestPathsCoputation` |使用的示例。 在此例中，它会计算图形中 ID 1 和其他所有 ID 之间的最短路径。 |
    | `-ca mapred.job.tracker=headnodehost:9010` |群集的头节点。 |
    | `-vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFromat` |用于输入数据的输入格式。 |
    | `-vip /example/data/tiny_graph.txt` |输入数据文件。 |
    | `-vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat` |输出格式。 在此例中，ID 和值是纯文本。 |
    | `-op /example/output/shortestpaths` |输出位置。 |
    | `-w 2` |要使用的辅助角色数目。 此例中为 2。 |

    有关这些参数以及与 Giraph 示例搭配使用的其他参数的详细信息，请参阅 [Giraph quickstart](http://giraph.apache.org/quick_start.html)（Giraph 快速入门）。

6. 作业完成后，其结果存储在 **/example/out/shotestpaths** 目录。 创建的输出文件名称以 **part-m-** 开头，结尾的数字表示第一个文件、第二个文件，依此类推。 使用以下命令查看输出：

        hdfs dfs -text /example/output/shortestpaths/*

    输出应如下所示：

        0    1.0
        4    5.0
        2    2.0
        1    0.0
        3    1.0

    SimpleShortestPathComputation 示例硬编码为从对象 ID 1 开始查找与其他对象间的最短路径。 因此，输出应显示为 `destination_id distance`，其中，distance 为对象 ID 1 与目标 ID 的边缘之间的行程值（或权重）。

    在可视化此数据的情况下，可以通过行经 ID 1 与所有其他对象之间的最短路径来验证结果。 请注意，ID 1 和 ID 4 之间的最短路径为 5。 这是从 <span style="color:orange">ID 1 到 ID 3</span>，然后再从 <span style="color:red">ID 3 到 ID 4</span> 的总距离。

    ![将对象绘制为圆圈，并绘制对象之间的最短路径](./media/hdinsight-hadoop-giraph-install-linux/giraph-graph-out.png)

## <a name="next-steps"></a>后续步骤

* [在 HDInsight 群集上安装并使用 Hue](/documentation/articles/hdinsight-hadoop-hue-linux/)。 Hue 是一种 Web UI，可让你轻松创建、运行及保存 Pig 和 Hive 作业，以及浏览 HDInsight 群集的默认存储。

* [在 HDinsight 群集上安装 R](/documentation/articles/hdinsight-hadoop-r-scripts-linux/)：说明如何使用群集自定义在 HDInsight Hadoop 群集上安装并使用 R。 R 是一种用于统计计算的开放源代码语言和环境。 它提供了数百个内置统计函数及其自己的编程语言，可结合各方面的函数编程和面向对象的编程。 它还提供了各种图形功能。

* [在 HDInsight 群集上安装 Solr](/documentation/articles/hdinsight-hadoop-solr-install-linux/)。 使用群集自定义在 HDInsight Hadoop 群集上安装 Solr。 Solr 允许你对存储的数据执行功能强大的搜索操作。