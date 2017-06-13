<properties
    pageTitle="在 Azure HDInsight 中创建 Hadoop、HBase、Kafka、Storm 或 Spark 群集 | Azure"
    description="了解如何使用浏览器、Azure CLI、Azure PowerShell、REST 或 SDK 在 Linux 上创建适用于 HDInsight 的 Hadoop、HBase、Storm 或 Spark 群集。"
    keywords="hadoop 群集设置, kafka 群集设置, spark 群集设置, hbase 群集设置, storm 群集设置, 什么是 hadoop 群集"
    services="hdinsight"
    documentationcenter=""
    author="mumian"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="23a01938-3fe5-4e2e-8e8b-3368e1bbe2ca"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="05/01/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="c0e4c1bc713a158fb9c89764fac8eaf4cf1099a6"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="create-hadoop-clusters-in-hdinsight"></a>在 HDInsight 中创建 Hadoop 群集
[AZURE.INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

Hadoop 群集由用于对群集中的任务进行分布式处理的多个虚拟机（节点）组成。 Azure HDInsight 对各个节点的安装和配置的实现细节进行抽象，因此你只需提供常规配置信息。 本文将介绍这些配置设置。

## <a name="basic-configurations"></a>基本配置

通过 [Azure 门户预览](https://portal.azure.cn)，可以使用“快速创建”或“自定义”来创建 HDInsight 群集。 本部分介绍了在“快速创建”选项中使用的基本配置设置。 自定义选项包括以下额外配置：

- [应用程序](#install-hdinsight-applications)
- [群集大小](#configure-cluster-size)
- 高级设置

  - [脚本操作](#customize-clusters-using-script-action)
  - [虚拟网络](#use-virtual-network)

### <a name="subscription"></a>订阅 
每个 HDInsight 群集与一个 Azure 订阅绑定。

### <a name="resource-group-name"></a>资源组名称
可以借助 [Azure Resource Manager](/documentation/articles/resource-group-overview/) 以组（称为 Azure 资源组）的形式处理应用程序中的资源。 可以通过单个协调的操作来部署、更新、监视或删除应用程序的所有资源。

### <a name="cluster-name"></a>群集名称
群集名称用于标识群集。 群集名称必须全局唯一，并且遵守以下命名准则：

* 字段必须是包含 3 到 63 个字符的字符串。
* 字段只能包含字母、数字和连字符。

### <a name="cluster-types"></a> 群集类型
Azure HDInsight 目前提供了以下几种群集类型，每种类型都具有一组用于提供特定功能的组件：

| 群集类型 | 功能 |
| --- | --- |
| [Hadoop](/documentation/articles/hdinsight-hadoop-introduction/) |查询和分析（批处理作业） |
| [HBase](/documentation/articles/hdinsight-hbase-overview/) |NoSQL 数据存储 |
| [Storm](/documentation/articles/hdinsight-storm-overview/) |实时事件处理 |
| [Spark](/documentation/articles/hdinsight-apache-spark-overview/) |内存中处理、交互式查询、微批流处理 |
| [交互式 Hive（预览版）](/documentation/articles/hdinsight-hadoop-use-interactive-hive/) |更快的交互式 Hive 查询的内存中缓存 |

每个群集类型在群集中具有自身的节点数目、在群集中使用自身的节点术语，对每个节点类型具有默认的 VM 大小。 下表中的括号内列出了每个节点类型的节点数目。

| 类型 | Nodes | 图表 |
| --- | --- | --- |
| Hadoop |头节点 (2)，数据节点 (1+) |![HDInsight Hadoop 群集节点](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-hadoop-roles.png) |
| HBase |头服务器 (2)，区域服务器 (1+)，主控/ZooKeeper 节点 (3) |![HDInsight HBase 群集节点](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-hbase-roles.png) |
| Storm |Nimbus 节点 (2)，监督程序服务器 (1+)，ZooKeeper 节点 (3) |![HDInsight Storm 群集节点](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-storm-roles.png) |
| Spark |头节点 (2)，辅助角色节点 (1+)，ZooKeeper 节点 (3)（对于 A1 ZooKeeper VM 大小免费） |![HDInsight Spark 群集节点](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-spark-roles.png) |

下表列出了 HDInsight 的默认 VM 大小：

<!-- need to be verified -->

| 群集类型 | Hadoop | HBase | Storm | Spark |
| --- | --- | --- | --- | --- |
| 头：默认 VM 大小 |D3 v2 |D3 v2 |A3 |D12 v2 |
| 头：建议的 VM 大小 |D3 v2、D4 v2、D12 v2 |D3 v2、D4 v2、D12 v2 |A3、A4、A5 |D12 v2、D13 v2、D14 v2 |
| 辅助角色：默认 VM 大小 |D3 v2 |D3 v2 |D3 v2 |Windows：D12 v2；Linux：D4 v2 |
| 辅助角色：建议的 VM 大小 |D3 v2、D4 v2、D12 v2 |D3 v2、D4 v2、D12 v2 |D3 v2、D4 v2、D12 v2 |Windows：D12 v2、D13 v2、D14 v2；Linux：D4 v2、D12 v2、D13 v2、D14 v2 |
| Zookeeper：默认 VM 大小 | |A3 |A2 | |
| Zookeeper：建议的 VM 大小 | |A3、A4、A5 |A2、A3、A4 | |

> [AZURE.NOTE]
> 头称为 Storm 群集类型的 *Nimbus* 。 辅助角色称为 HBase 群集类型的“区域”以及 Storm 群集类型的“监督程序”。

> [AZURE.IMPORTANT]
> 如果计划使用 32 个以上的工作节点（在创建群集时配置或者是在创建之后通过扩展群集来配置），必须选择至少具有 8 个核心和 14 GB RAM 的头节点大小。
>
>

可以使用[脚本操作](#customize-clusters-using-script-action)向这些基本类型添加其他组件，如 Hue。

> [AZURE.IMPORTANT]
> HDInsight 群集有各种类型，分别与针对其优化群集的工作负荷或技术相对应。 没有任何方法支持创建组合多种类型的群集，如一个群集同时具有 Storm 和 HBase 类型。
>
>

如果你的解决方案需要分布在多种 HDInsight 群集类型上的技术，则应创建 Azure 虚拟网络，并创建虚拟网络中所需的群集类型。 此配置允许群集以及部署到群集的任何代码直接相互通信。

有关将 Azure 虚拟网络与 HDInsight 配合使用的详细信息，请参阅[使用 Azure 虚拟网络扩展 HDInsight](/documentation/articles/hdinsight-extend-hadoop-virtual-network/)。

有关在一个 Azure 虚拟网络中使用两种群集类型的示例，请参阅[使用 Storm 和 HBase 分析传感器数据](/documentation/articles/hdinsight-storm-sensor-data-analysis/)。

### <a name="operating-system"></a>操作系统
可以在 Linux 或 Windows 中创建 HDInsight 群集。  有关 OS 版本的详细信息，请参阅[支持的 HDInsight 版本](/documentation/articles/hdinsight-component-versioning/#supported-hdinsight-versions)。 HDInsight 3.4 及更高版本使用 Ubuntu Linux 发行版作为群集的基础 OS。 

### <a name="version"></a>Version
此选项用于确定该群集所需的 HDInsight 版本。 有关详细信息，请参阅[支持的 HDInsight 版本](/documentation/articles/hdinsight-component-versioning/#supported-hdinsight-versions)。

### <a name="credentials"></a>凭据
使用 HDInsight 群集时，可以在群集创建期间配置两个用户帐户：

* HTTP 用户。 默认的用户名为 *admin*。 它使用 Azure 门户预览的基本配置。 有时称为“群集用户”。
* SSH 用户（Linux 群集）。 通过 SSH 连接到群集时使用此用户。 有关详细信息，请参阅 [Use SSH with HDInsight](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)（对 HDInsight 使用 SSH）。

### <a name="location"></a>位置
群集位置不需要显式指定。 群集共享相同的默认存储位置。 有关受支持区域的列表，请单击 [HDInsight 定价](/pricing/details/hdinsight/)中的“区域”下拉列表。

### <a name="storage"></a>存储

原始 Hadoop 分布式文件系统 (HDFS) 在群集上使用许多本地磁盘。 HDInsight 使用 [Azure 存储](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/#azure-storage)中的 Blob。

![HDInsight 存储](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-cluster-creation-storage.png)

#### <a name="azure-storage"></a>Azure 存储
Azure 存储是一种稳健、通用的存储解决方案，它与 HDInsight 无缝集成。 通过 HDFS 界面，可以针对 Blob 中存储的结构化或非结构化数据直接运行 HDInsight 中的整套组件。 将数据存储在 Azure 存储中有助于安全删除用于计算的 HDInsight 群集，而不会丢失用户数据。 HDInsight 仅支持 __常规用途__ Azure 存储帐户。 它目前不支持 __Blob 存储__帐户类型。

在配置期间，请指定一个 Azure 存储帐户和该 Azure 存储帐户中的一个 blob 容器。 群集使用该 Blob 容器作为默认存储位置。 也可以选择指定群集可访问的其他 Azure 存储帐户（链接的存储）。 群集还可以访问任何配置有完全公共读取权限或仅限对 blob 的公共读取权限的 blob 容器。

建议不要使用默认 Blob 容器来存储业务数据。 最佳做法是每次使用之后删除默认 Blob 容器以降低存储成本。 请注意，默认容器包含应用程序日志和系统日志。 请确保在删除该容器之前检索日志。

不支持对多个群集共享一个 Blob 容器。

有关使用 Azure 存储帐户的详细信息，请参阅[将 Azure 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-use-blob-storage/)。

#### <a name="use-additional-storage"></a>附加存储

在某些情况下，可能会向群集添加其他存储。 例如，为不同地理区域或不同服务创建了多个 Azure 存储帐户，但想要使用 HDInsight 分析所有这些帐户。

创建 HDInsight 群集时或创建群集后，可以添加存储帐户。  请参阅[使用脚本操作自定义基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)。

有关辅助 Azure 存储帐户的详细信息，请参阅[将 Azure 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-use-blob-storage/)。

#### <a name="use-hiveoozie-metastore"></a>Hive 元存储

如果想要在删除 HDInsight 群集后保留 Hive 表，建议使用自定义元存储。 这样，便可以将该元存储附加到另HDInsight 群集。

> [AZURE.IMPORTANT]
> 为一个 HDInsight 群集版本创建的 HDInsight 元存储不能在不同的 HDInsight 群集版本之间共享。 有关 HDInsight 版本的列表，请参阅[支持的 HDInsight 版本](/documentation/articles/hdinsight-component-versioning/#supported-hdinsight-versions)。

元存储包含 Hive 元数据，例如 Hive 表、分区、架构和列。 元存储可帮助保留 Hive 元数据，因此在创建新群集时，不需要重新创建 Hive 表。 默认情况下，Hive 使用嵌入的 Azure SQL 数据库存储此信息。 删除群集时，嵌入式数据库无法保留元数据。 在配置了 Hive 元存储的 HDInsight 群集中创建 Hive 表时，使用相同的 Hive 元存储重新创建群集会保留这些表。

元存储配置并非适用于所有群集类型。 例如，它不适用于 HBase 或 Kafka 群集。

> [AZURE.IMPORTANT]
> 创建自定义元存储时，请勿使用包含短划线、连字符或空格的数据库名称。 否则可能导致群集创建过程失败。

> [AZURE.WARNING]
> Hive 元存储不支持 Azure SQL 仓库。

#### <a name="oozie-metastore"></a>Oozie 元存储

在使用 Oozie 时若要提高性能，请使用自定义元存储。 如果想在删除群集后，访问 Oozie 作业数据，也可使用自定义元存储。 如果不打算使用 Oozie，或仅间歇性使用 Oozie，无需创建自定义元存储。

> [AZURE.IMPORTANT]
> 无法重用自定义 Oozie 元存储。 若要使用自定义 Oozie 元存储，必须在创建 HDInsight 群集时提供一个空的 Azure SQL 数据库。

元存储配置并非适用于所有群集类型。 例如，它不适用于 HBase 或 Kafka 群集。

> [AZURE.WARNING]
> Oozie 元存储不支持 Azure SQL 仓库。

## <a name="install-hdinsight-applications"></a>Install HDInsight applications

HDInsight 应用程序是用户可以在基于 Linux 的 HDInsight 群集上安装的应用程序。 这些应用程序可能是 Microsoft、独立软件供应商 (ISV) 或你自己开发的。 有关详细信息，请参阅[在 Azure HDInsight 上安装第三方 Hadoop 应用程序](/documentation/articles/hdinsight-apps-install-applications/)。

大多数 HDInsight 应用程序安装在空边缘节点上。  空边缘节点是安装并配置了与头节点中相同的客户端工具的 Linux 虚拟机。 可以使用该边缘节点来访问群集、测试客户端应用程序和托管客户端应用程序。 有关详细信息，请参阅[在 HDInsight 中使用空边缘节点](/documentation/articles/hdinsight-apps-use-edge-node/)。

## <a name="configure-cluster-size"></a>配置群集大小

客户需根据群集的生存期，支付这些节点的使用费。 创建群集后便开始计费，删除群集后停止计费。 无法取消分配群集或将其置于暂停状态。

不同群集类型具有不同的节点类型、节点数和节点大小。 例如，Hadoop 群集类型具有两个头节点和四个数据节点（默认值），而 Storm 群集类型具有两个 Nimbus 节点、三个 ZooKeeper 节点和四个 supervisor 节点（默认值）。 HDInsight 群集的成本取决于节点数和节点的虚拟机大小。 例如，如果你知道将执行需要大量内存的操作，则可能要选择具有更多内存的计算资源。 为便于学习，建议使用一个数据节点。 有关 HDInsight 定价的详细信息，请参阅 [HDInsight 定价](/pricing/details/hdinsight/)。

> [AZURE.NOTE]
> 群集大小限制因 Azure 订阅而异。 要提高限制的大小，请联系计费支持人员。
><p>
> 群集使用的节点不视为虚拟机，因为用于节点的虚拟机映像是 HDInsight 服务的实现细节。 节点使用的计算核心会计入可供订阅使用的计算核心总数。 创建 HDInsight 群集时，可在“群集大小”边栏选项卡的摘要部分中查看可用核心数以及群集要使用的核心数。
>

使用 Azure 门户预览配置群集时，可通过“节点定价层”边栏选项卡查看节点大小  。 还可以查看不同节点大小的相关成本。 以下屏幕截图显示了基于 Linux 的 Hadoop 群集的选项。

![HDInsight VM 节点大小](./media/hdinsight-hadoop-provision-linux-clusters/hdinsight-node-sizes.png)

下表显示 HDInsight 群集支持的大小和它们提供的容量。

### <a name="standard-tier-a-series"></a>标准层：A 系列
在经典部署模型中，某些 VM 大小在 PowerShell 和命令行接口 (CLI) 中稍有不同。

* Standard_A3 是大型
* Standard_A4 是超大型

| 大小 | CPU 核心数 | 内存 | NIC 数（最大值） | 最大 磁盘大小 | 最大 数据磁盘（每个 1023 GB） | 最大 IOPS（每个磁盘 500 次） |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_A3\大型 |4 |7 GB |2 |临时磁盘 = 285 GB |8 |8x500 |
| Standard_A4\超大型 |8 |14 GB |4 |临时磁盘 = 605 GB |16 |16x500 |
| Standard_A6 |4 |28 GB |2 |临时磁盘 = 285 GB |8 |8x500 |
| Standard_A7 |8 |56 GB |4 |临时磁盘 = 605 GB |16 |16x500 |

### <a name="standard-tier-d-series"></a>标准层：D 系列
| 大小 | CPU 核心数 | 内存 | NIC 数（最大值） | 最大 磁盘大小 | 最大 数据磁盘（每个 1023 GB） | 最大 IOPS（每个磁盘 500 次） |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_D3 |4 |14 GB |4 |临时磁盘 (SSD) = 200 GB |8 |8x500 |
| Standard_D4 |8 |28 GB |8 |临时磁盘 (SSD) = 400 GB |16 |16x500 |
| Standard_D12 |4 |28 GB |4 |临时磁盘 (SSD) = 200 GB |8 |8x500 |
| Standard_D13 |8 |56 GB |8 |临时磁盘 (SSD) = 400 GB |16 |16x500 |
| Standard_D14 |16 |112 GB |8 |临时磁盘 (SSD) = 800 GB |32 |32x500 |

### <a name="standard-tier-dv2-series"></a>标准层：Dv2 系列
| 大小 | CPU 核心数 | 内存 | NIC 数（最大值） | 最大 磁盘大小 | 最大 数据磁盘（每个 1023 GB） | 最大 IOPS（每个磁盘 500 次） |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_D3_v2 |4 |14 GB |4 |临时磁盘 (SSD) = 200 GB |8 |8x500 |
| Standard_D4_v2 |8 |28 GB |8 |临时磁盘 (SSD) = 400 GB |16 |16x500 |
| Standard_D12_v2 |4 |28 GB |4 |临时磁盘 (SSD) = 200 GB |8 |8x500 |
| Standard_D13_v2 |8 |56 GB |8 |临时磁盘 (SSD) = 400 GB |16 |16x500 |
| Standard_D14_v2 |16 |112 GB |8 |临时磁盘 (SSD) = 800 GB |32 |32x500 |

有关在计划使用这些资源时要考虑的部署注意事项，请参阅[虚拟机的大小](/documentation/articles/virtual-machines-windows-sizes/)。 有关不同大小的定价信息，请参阅 [HDInsight 定价](/pricing/details/hdinsight/)。   

> [AZURE.IMPORTANT]
> 如果计划使用 32 个以上的工作节点（在创建群集时配置或者是在创建之后通过扩展群集来配置），必须选择至少具有 8 个核心和 14 GB RAM 的头节点大小。
>
>

创建群集后便开始计费，删除群集后停止计费。 有关定价的详细信息，请参阅 [HDInsight 定价详细信息](/pricing/details/hdinsight/)。

> [AZURE.WARNING]
> 不支持在 HDInsight 群集之外的其他位置使用别的存储帐户。

## <a name="use-virtual-network"></a>使用虚拟网络
使用 [Azure 虚拟网络](/documentation/services/networking/)可创建包含解决方案所需的资源的安全持久性网络。 有关将 HDInsight 与虚拟网络配合使用的详细信息（包括虚拟网络的特定配置要求），请参阅[使用 Azure 虚拟网络扩展 HDInsight 功能](/documentation/articles/hdinsight-extend-hadoop-virtual-network/)。

## <a name="customize-clusters-using-script-action"></a>使用脚本操作自定义群集

你可以在创建期间通过使用脚本安装其他组件或自定义群集配置。 此类脚本可通过 **脚本操作**调用，脚本操作是一种配置选项，可通过 Azure 门户预览、HDInsight Windows PowerShell cmdlet 或 HDInsight .NET SDK 使用。 有关详细信息，请参阅[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)。

某些本机 Java 组件（如 Mahout 和 Cascading）可以在群集上作为 Java 存档 (JAR) 文件运行。 可以通过 Hadoop 作业提交机制将这些 JAR 文件分发到 Azure 存储，然后提交到 HDInsight 群集。 有关详细信息，请参阅[以编程方式提交 Hadoop 作业](/documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/)。

> [AZURE.NOTE]
> 如果在将 JAR 文件部署到 HDInsight 群集或调用 HDInsight 群集上的 JAR 文件时遇到问题，请联系 [Azure.cn 支持](/support/contact/)。
><p>
> HDInsight 不支持级联，因此不符合 Azure.cn 技术支持的条件。 有关支持的组件的列表，请参阅 [HDInsight 提供的群集版本有哪些新功能？](/documentation/articles/hdinsight-component-versioning/)。
>
>

在创建过程中，有时需要配置以下配置文件：

* clusterIdentity.xml
* core-site.xml
* gateway.xml
* hbase-env.xml
* hbase-site.xml
* hdfs-site.xml
* hive-env.xml
* hive-site.xml
* mapred-site
* oozie-site.xml
* oozie-env.xml
* storm-site.xml
* tez-site.xml
* webhcat-site.xml
* yarn-site.xml

有关详细信息，请参阅[使用 Bootstrap 自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-bootstrap/)。

## <a name="cluster-creation-methods"></a>群集创建方法
在本文中，你了解了有关创建基于 Linux 的 HDInsight 群集的基本信息。 使用下表查找具体信息，了解如何使用最合适的方法创建群集。

| 群集创建方法 | Web 浏览器 | 命令行 | REST API | SDK | Linux、Mac OS X 或 Unix | Windows |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| [Azure 门户预览](/documentation/articles/hdinsight-hadoop-create-linux-clusters-portal/) |✔ |&nbsp; |&nbsp; |&nbsp; |✔ |✔ |
| [Azure CLI](/documentation/articles/hdinsight-hadoop-create-linux-clusters-azure-cli/) |&nbsp; |✔ |&nbsp; |&nbsp; |✔ |✔ |
| [Azure PowerShell](/documentation/articles/hdinsight-hadoop-create-linux-clusters-azure-powershell/) |&nbsp; |✔ |&nbsp; |&nbsp; |✔ |✔ |
| [cURL](/documentation/articles/hdinsight-hadoop-create-linux-clusters-curl-rest/) |&nbsp; |✔ |✔ |&nbsp; |✔ |✔ |
| [.NET SDK](/documentation/articles/hdinsight-hadoop-create-linux-clusters-dotnet-sdk/) |&nbsp; |&nbsp; |&nbsp; |✔ |✔ |✔ |
| [Azure Resource Manager 模板](/documentation/articles/hdinsight-hadoop-create-linux-clusters-arm-templates/) |&nbsp; |✔ |&nbsp; |&nbsp; |✔ |✔ |

## <a name="troubleshoot"></a>故障排除

如果在创建 HDInsight 群集时遇到问题，请参阅[访问控制要求](/documentation/articles/hdinsight-administer-use-portal-linux/#create-clusters)。

## <a name="next-steps"></a>后续步骤

- [什么是 HDInsight](/documentation/articles/hdinsight-hadoop-introduction/)。
- [Hadoop 教程：开始使用 HDInsight 中的 Hadoop](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)。

<!--Update_Description: add "basic configuration", "storage configuration", and "troubleshoot"-->