<!-- not suitable for Mooncake -->

<properties
    pageTitle="Azure HDInsight 上的 R Server 简介 | Azure"
    description="获取 HDInsight 上的 R Server 简介。了解如何使用 R Server 创建用于大数据分析的应用程序。"
    services="hdinsight"
    documentationcenter=""
    author="jeffstokes72"
    manager="jhubbard"
    editor="cgronlun" />
<tags 
    ms.assetid="6dc21bf5-4429-435f-a0fb-eea856e0ea96"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/09/2017"
    wacn.date="01/25/2017"
    ms.author="jeffstok" />

#HDInsight 上的 R Server 和开放源代码 R 功能简介

自 Azure HDInsight 推出后，可以在 Azure 中创建 HDInsight 群集时选择使用 Microsoft R Server。这项新功能可让数据科研人员、统计人员和 R 程序员根据需要访问 HDInsight 上可缩放的分布式分析方法。

可依据手头的项目和任务调整群集大小，不再需要群集时将它解除。由于这些群集属于 Azure HDInsight 的一部分，提供企业级全年无休支持、99.9% 运行时间 SLA，并且能够灵活地与 Azure 生态系统中的其他组件集成。

HDInsight 上的 R Server 提供最新的功能，可针对载入 Azure Blob 或 Data Lake 存储的几乎任何大小的数据集执行基于 R 的分析。由于 R Server 基于开放源代码的 R 构建，因此你构建的基于 R 的应用程序可以利用超过 8000 个任意开放源代码 R 包，以及 ScaleR（R Server 附带的 Microsoft 的大数据分析包）的例程。

群集的边缘节点为连接到群集和运行 R 脚本提供了便捷的位置。使用边缘节点，你可以选择跨边缘节点服务器的各个核心运行 ScaleR 的并行化分布式函数。你还可以选择通过使用 ScaleR 的 Hadoop Map Reduce 或 Spark 计算上下文跨群集的各个节点运行这些函数。

可以下载分析后生成的模型或预测，以便在本地使用。也可以在 Azure 中的其他位置操作这些模型或预测。

## HDInsight 上的 R 入门
若要在 HDInsight 群集中包括 R Server，必须在使用 Azure 门户预览创建 HDInsight 群集时选择 R Server 群集类型。R Server 群集类型包括群集数据节点以及作为基于 R Server 的分析登录区域的边缘节点上的 R Server。请参阅 [Getting Started with R Server on HDInsight](/documentation/articles/hdinsight-hadoop-r-server-get-started/)（HDInsight 上的 R Server 入门）了解创建群集的详细演练。

## 了解数据存储选项
HDInsight 群集的 HDFS 文件系统的默认存储可以与 Azure 存储帐户或 Azure Data Lake Store 相关联。这可确保在分析过程中，上载到群集存储的任何数据均会持久保存。对于所选择的将数据传输到存储的选项有各种工具，包括存储帐户的基于门户的上载工具和 [AzCopy](/documentation/articles/storage-use-azcopy/) 实用程序。

无论你选择 Azure Blob 还是 Data Lake 作为群集的主存储，都可以在群集预配过程中选择添加对附加 Blob 和 Data Lake Store 的访问权限。请参阅 [HDInsight 上的 R Server 入门](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-r-server-get-started)，了解有关向其他帐户添加访问权限的信息，并请参阅补充的[适用于 HDInsight 上的 R Server 的 Azure 存储选项](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-r-server-storage)一文，了解如何使用多个存储帐户。

你也可以将 [Azure 文件](/documentation/articles/storage-how-to-use-files-linux/)服务用作边缘节点上的存储选项。Azure 文件可让你将 Azure 存储空间中创建的文件共享装载到 Linux 文件系统。有关适用于 HDInsight 群集上 R Server 的数据存储选项的详细信息，请参阅 [Azure Storage options for R Server on HDInsight clusters](/documentation/articles/hdinsight-hadoop-r-server-storage/)（适用于 HDInsight 群集上 R Server 的 Azure 存储选项）。

## 访问群集上的 R Server
创建包含 R Server 的群集后，如果已选择在预配过程中包括 RStudio Server 或者稍后已添加它，则可以使用浏览器连接到边缘节点上的 R Server，或使用 SSH/PuTTY 访问 R 控制台。有关创建群集后安装 RStudio Server 的详细信息，请参阅 [Installing RStudio Server on HDInsight clusters](/documentation/articles/hdinsight-hadoop-r-server-install-r-studio/)（在 HDInsight 群集上安装 RStudio Server）。

## 开发和运行 R 脚本
创建和运行的 R 脚本可以使用 8000 多种开放源代码 R 包中的任何一种，此外，还可以使用 ScaleR 库中的并行化分布式例程。一般而言，使用边缘节点上的 R Server 运行的脚本将在该节点上的 R 解释程序内运行，但调用计算上下文设置为 Hadoop Map Reduce (RxHadoopMR) 或 Spark (RxSpark) 的 ScaleR 函数的这些步骤除外。

在这种情况下，函数将以分布方式跨与引用数据关联的群集的数据（任务）节点运行。有关不同计算上下文选项的详细信息，请参阅 [Compute context options for R Server on HDInsight](/documentation/articles/hdinsight-hadoop-r-server-compute-contexts/)（适用于 HDInsight 上的 R Server 的计算上下文选项）。

## <a name="operationalize-a-model"></a> 操作模型
完成数据建模后，可以在 Azure 中和本地操作模型，以便针对新数据执行预测。此过程称为评分。以下提供了一些示例：

### <a name="scoring-in-hdinsight"></a> 在 HDInsight 中评分
若要在 HDInsight 中评分，可以针对已载入存储帐户的新数据文件编写调用模型的 R 函数以进行预测。然后将预测保存回到存储帐户。可以根据需要在群集的边缘节点上运行该例程，或使用计划作业来进行。

### 本地评分
若要在创建模型之后进行本地评分，可以在 R 中序列化模型，将其下载，将其反序列化，然后使用它进行新数据评分。可以使用前面[在 HDInsight 中评分](#scoring-in-hdinsight)所述的方法，或使用 [DeployR](https://deployr.revolutionanalytics.com/) 进行新数据评分。

## 维护群集
### 安装和维护 R 包
由于 R 脚本的大多数步骤在边缘节点上运行，因此边缘节点上需要有大部分使用的 R 包。若要在边缘节点上安装其他 R 包，可以在 R 中使用常用的 `install.packages()` 方法。

在大多数情况下，如果你只需要从 ScaleR 库跨群集使用例程，则不需要在数据节点上安装其他 R 包。但是，可能需要其他包才能支持在数据节点上使用 **rxExec** 或 **RxDataStep** 执行。

在这种情况下，可以在创建群集之后，使用脚本操作来安装其他包。有关详细信息，请参阅 [Creating an HDInsight cluster with R Server](/documentation/articles/hdinsight-hadoop-r-server-get-started/)（创建包含 R Server 的 HDInsight 群集）。

### 更改 Hadoop Map Reduce 内存设置
可以在运行 Map Reduce 作业时修改群集，以更改 R Server 的可用内存量。若要修改群集，可以使用通过群集的 Azure 门户门户预览选项卡提供的 Apache Ambari UI。有关如何访问群集的 Ambari UI 的说明，请参阅 [Manage HDInsight clusters using the Ambari Web UI](/documentation/articles/hdinsight-hadoop-manage-ambari/)（使用 Ambari Web UI 管理 HDInsight 群集）。

也可以在 **RxHadoopMR** 的调用中使用 Hadoop 参数更改 R Server 可用的内存量，如下所示：

    hadoopSwitches = "-libjars /etc/hadoop/conf -Dmapred.job.map.memory.mb=6656"  

### 缩放你的群集
可以通过门户扩展或缩减现有群集。通过缩放，可以获得更多的容量来完成较大的处理任务，或者在群集空闲时缩减容量。有关如何缩放群集的说明，请参阅 [Manage HDInsight clusters](/documentation/articles/hdinsight-administer-use-portal-linux/)（管理 HDInsight 群集）。

### 维护系统
在非工作时间，系统将在 HDInsight 群集的基础 Linux VM 上执行维护，以应用 OS 修补程序和其他更新。通常，维护操作将在星期一和星期四凌晨 3:30（基于 VM 的本地时间）完成。执行更新时，每次应该只有不到四分之一的群集会受影响。

由于头节点是冗余的，且并非所有数据节点都受影响，因此在此时间段运行的所有作业可能会变慢。但是，这些作业应该都可运行完成。除非发生需要重建群集的灾难性故障，否则任何自定义软件或本地数据在这些维护事件中都将保留。

## 了解适用于 HDInsight 群集上 R Server 的 IDE 选项
HDInsight 群集的 Linux 边缘节点是基于 R 的分析的登录区域。最新版本的 HDInsight 提供了一个默认选项，用于在边缘节点上安装 [RStudio Server](https://www.rstudio.com/products/rstudio-server/) 的社区版作为基于浏览器的 IDE。使用 RStudio Server 作为 IDE 来开发和执行 R 脚本，与仅使用 R 控制台相比，可以大幅提高生产力。如果选择不在创建群集时添加 RStudio Server，而想在以后添加，则请参阅[在 HDInsight 群集上安装 R Studio Server](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-r-server-install-r-studio)。+

另一个完整的 IDE 选项是安装桌面 IDE（如 Microsoft 最近公布的 [R Tools for Visual Studio](https://www.visualstudio.com/features/rtvs-vs.aspx) (RTVS)、RStudio、或 Walware 的基于 Eclipse 的 [StatET](http://www.walware.de/goto/statet)）并通过使用远程 Map Reduce 或 Spark 计算上下文访问群集。

最后，通过 SSH 或 PuTY 连接后，只需在 Linux 命令提示符处键入 **R** 即可访问边缘节点上的 R Server 控制台。如果在另一个窗口中运行 R 脚本开发的文本编辑器，可根据需要将脚本部分剪切并粘贴到 R 控制台，以使用增强的控制台界面。

## 了解定价
包含 R Server 的 HDInsight 群集的相关费用结构与标准 HDInsight 群集的费用结构类似。这些费用以各种名称、数据和边缘节点的基础 VM 大小为基准，加上的核心运行小时数附加费。有关 HDInsight 定价和 30 天试用版的可用性的详细信息，请参阅 [HDInsight pricing](/pricing/details/hdinsight/)（HDInsight 定价）。

## 后续步骤
单击以下链接详细了解如何使用 HDInsight 群集上的 R Server。

* [Getting Started with R Server on HDInsight（HDInsight 上的 R Server 入门）](/documentation/articles/hdinsight-hadoop-r-server-get-started/)
* [将 RStudio Server 添加到 HDInsight（如果未在群集创建过程中安装）](/documentation/articles/hdinsight-hadoop-r-server-install-r-studio/)
* [Compute context options for R Server on HDInsight（适用于 HDInsight 上的 R Server 的计算上下文选项）](/documentation/articles/hdinsight-hadoop-r-server-compute-contexts/)
* [Azure Storage options for R Server on HDInsight（适用于 HDInsight 上的 R Server 的 Azure 存储选项）](/documentation/articles/hdinsight-hadoop-r-server-storage/)

<!---HONumber=Mooncake_0120_2017-->