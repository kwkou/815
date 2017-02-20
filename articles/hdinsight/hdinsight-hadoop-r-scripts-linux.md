<!-- not suitable for Mooncake -->

<properties
    pageTitle="在基于 Linux 的 HDInsight 上安装 R | Azure"
    description="了解如何安装并使用 R 来自定义基于 Linux 的 Hadoop 群集。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun" />
<tags 
    ms.assetid="7b758492-87bf-4d82-8b8c-1664e7d177bd"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/09/2017"
    wacn.date="02/20/2017"
    ms.author="larryfr" />

# 在 HDInsight Hadoop 群集上安装并使用 R
你可以使用**脚本操作**群集自定义在 HDInsight 上 Hadoop 中的任何类型的群集上安装 R。这样，数据科学家和分析人员便可使用 R 部署功能强大的 MapReduce/YARN 编程框架，以便在 HDInsight 中部署的 Hadoop 群集上处理大量数据。

> [AZURE.IMPORTANT]
HDInsight 的 [HDInsight](/pricing/details/hdinsight/) 产品/服务包括 HDInsight 群集中的 R Server。这将允许 R 脚本使用 MapReduce 和 Spark 运行分布式计算。有关详细信息，请参阅[Get started using R Server on HDInsight](/documentation/articles/hdinsight-hadoop-r-server-get-started/)（开始使用 HDInsight 上的 R Server）。
> 
> 

## 什么是 R？
<a href="http://www.r-project.org/" target="_blank">统计计算的 R 项目</a>是一种用于统计计算的开放源代码语言和环境。R 提供了数百个内置统计函数及其自己的编程语言，该语言结合了各方面的函数编程和面向对象的编程。它还提供了各种图形功能。R 是面向各个领域最专业的统计学家和科学家的首选编程环境。

R 脚本可以在 HDInsight 中使用创建用于安装 R 环境的脚本操作自定义的 Hadoop 群集上运行。R 与 Azure Blob 存储 (WASB) 兼容，这样，存储在此的数据可以在 HDInsight 上使用 R 进行处理。

## 此脚本的用途
用于在 HDInsight 群集上安装 R 的脚本操作会安装以下提供基本 R 安装的 Ubuntu 包：

* [r-base](http://packages.ubuntu.com/precise/r-base)：基本 GNU R 包
* [r-base-dev](http://packages.ubuntu.com/precise/r-base-dev)：辅助 GNU R 包

同时会安装以下 RHadoop 包，提供与 MapReduce 和 HDFS 的集成：

* [rmr2](https://github.com/RevolutionAnalytics/rmr2)：允许 R 开发人员使用 Hadoop MapReduce
* [rhdfs](https://github.com/RevolutionAnalytics/rhdfs)：允许 R 开发人员使用 Hadoop HDFS（用于 HDInsight 的 WASB）

此外，还会安装以下 R 包：

| R 包 | 提供内容 |
| --- | --- |
| [rJava](https://cran.r-project.org/web/packages/rJava/index.html) |Java 接口的低级别 R。 |
| [Rcpp](https://cran.r-project.org/web/packages/Rcpp/index.html) |R 和 C++ 集成。 |
| [RJSONIO](https://cran.r-project.org/web/packages/RJSONIO/index.html) |将 R 对象序列化/反序列化为 JSON |
| [bitops](https://cran.r-project.org/web/packages/bitops/index.html) |整数向量的位运算函数。 |
| [digest](https://cran.r-project.org/web/packages/digest/index.html) |创建 R 对象的加密哈希摘要。 |
| [功能](https://cran.r-project.org/web/packages/functional/index.html) |柯里化、撰写和其他高阶函数 |
| [reshape2](https://cran.r-project.org/web/packages/reshape2/index.html) |灵活地重构及聚合数据。 |
| [stringr](https://cran.r-project.org/web/packages/stringr/index.html) |用于通用字符串操作的简单、一致包装。 |
| [plyr](https://cran.r-project.org/web/packages/plyr/index.html) |用于拆分、应用和合并数据的工具。 |
| [caTools](https://cran.r-project.org/web/packages/caTools/index.html) |用于移动窗口统计信息、GIF、Base64、ROC AUC 等的工具。 |
| [stringdist](https://cran.r-project.org/web/packages/stringdist/index.html) |近似字符串匹配和字符串距离函数。 |

## 使用脚本操作安装 R
以下脚本操作用于在 HDInsight 群集上安装 R。https://hdiconfigactions.blob.core.windows.net/linuxrconfigactionv01/r-installer-v01.sh

本部分提供有关如何在使用 Azure 门户预览创建新群集时使用脚本的说明。

> [AZURE.NOTE]
Azure PowerShell、Azure CLI、HDInsight .NET SDK 或 Azure Resource Manager 模板也可用于应用脚本操作。你也可以将脚本操作应用于已在运行的群集。有关详细信息，请参阅 [Customize HDInsight clusters with Script Actions](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)（使用脚本操作自定义 HDInsight 群集）。
> 
> 

1. 使用 [Provision Linux-based HDInsight clusters](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)（预配基于 Linux 的 HDInsight 群集）中的步骤开始预配群集，但不要完成预配。
2. 在“可选配置”边栏选项卡上，选择“脚本操作”，并提供以下信息：
   
    * **名称**：输入脚本操作的友好名称。
    * **脚本 URI**：https://hdiconfigactions.blob.core.windows.net/linuxrconfigactionv01/r-installer-v01.sh
    * **标头**：选中此选项
    * **辅助角色**：选中此选项
    * **ZOOKEEPER**：选中此选项以在 Zookeeper 节点上安装。
    * **参数**：将此字段留空
3. 在“脚本操作”的底部，使用“选择”按钮保存配置。最后，使用“可选配置”边栏选项卡底部的“选择”按钮保存可选配置信息。
4. 根据 [Provision Linux-based HDInsight clusters](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)（预配基于 Linux 的 HDInsight 群集）中所述继续预配群集。

## 运行 R 脚本
在群集完成预配后，执行以下步骤，使用 R 在 群集上执行 MapReduce 操作。

1. 使用 SSH 连接到 HDInsight 群集：
   
        ssh USERNAME@CLUSTERNAME-ed-ssh.azurehdinsight.cn
   
    有关如何在 HDInsight 中使用 SSH 的详细信息，请参阅以下文档：
   
    * [在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)
    * [在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)
2. 在 `username@hn0-CLUSTERNAME:~$` 提示符中输入以下命令，启动交互式 R 会话：
   
        R
3. 输入以下 R 程序。这会生成数字 1 到 100 ，然后将它们乘以 2。
   
        library(rmr2)
        ints = to.dfs(1:100)
        calc = mapreduce(input = ints, map = function(k, v) cbind(v, 2*v))
   
    第一行会调用 RHadoop 库 rmr2，该库用于 MapReduce 操作。
   
    第二行会生成值 1 到 100，然后使用 `to.dfs` 将它们存储到 Hadoop 文件系统。
   
    第三行会使用 rmr2 提供的功能创建 MapReduce 进程并开始处理。随着处理操作开始，应看到多个行滚动过去。
4. 接下来，请使用以下命令查看存储 MapReduce 输出的临时路径：
   
        print(calc())
   
    路径应该类似 `/tmp/file5f615d870ad2`。若要查看实际输出，请使用以下命令：
   
        print(from.dfs(calc))
   
    输出应如下所示：
   
        [1,]  1 2
        [2,]  2 4
        .
        .
        .
        [98,]  98 196
        [99,]  99 198
        [100,] 100 200
5. 若要退出 R，请使用以下命令：
   
        q()

## 后续步骤
* [Install and use Hue on HDInsight clusters](/documentation/articles/hdinsight-hadoop-hue-linux/)（在 HDInsight 群集上安装并使用 Hue）。Hue 是一种 Web UI，可让你轻松创建、运行及保存 Pig 和 Hive 作业，以及浏览 HDInsight 群集的默认存储。
* [在 HDInsight 群集上安装 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install/)。使用群集自定义在 HDInsight Hadoop 群集上安装 Giraph。Giraph 可让你使用 Hadoop 执行图形处理，并可以在 Azure HDInsight 上使用。
* [在 HDInsight 群集上安装 Solr](/documentation/articles/hdinsight-hadoop-solr-install/)。使用群集自定义在 HDInsight Hadoop 群集上安装 Solr。Solr 允许你对存储的数据执行功能强大的搜索操作。
* [Install Hue on HDInsight clusters](/documentation/articles/hdinsight-hadoop-hue-linux/)（在 HDInsight 群集上安装 Hue）。使用群集自定义在 HDInsight Hadoop 群集上安装 Hue。Hue 是用来与 Hadoop 群集交互的一系列 Web 应用程序。

[hdinsight-cluster-customize]: /documentation/articles/hdinsight-hadoop-customize-cluster-linux/

<!---HONumber=Mooncake_0213_2017-->