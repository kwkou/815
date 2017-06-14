<properties
    pageTitle="在 HDInsight 上安装或更新 Mono | Azure"
    description="了解如何将特定版本的 Mono 用于 HDInsight 群集。 Mono 用于在基于 Linux 的 HDInsight 群集上运行 .NET 应用程序。"
    services="hdinsight"
    documentationCenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.service="hdinsight"
    ms.devlang=""
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="05/01/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.custom="hdinsightactive"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="dc9405b1c193948b72944692c0bd1b25a221513c"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="install-or-update-mono-on-hdinsight"></a>在 HDInsight 上安装或更新 Mono

了解如何在 HDInsight 3.4 或更高版本上安装特定版本的 [Mono](https://www.mono-project.com)。

Mono 安装在 HDInsight 3.4 及更高版本上，用于运行 .NET 应用程序。 有关每个 HDInsight 版本包含的 Mono 版本的信息，请参阅 [HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning/)。 若要在群集上安装不同版本，请使用本文档中的脚本操作。 

## <a name="how-it-works"></a>工作原理

此脚本接受以下参数：

* __Mono 版本号__：要安装的 Mono 版本。 版本必须可从 [https://download.mono-project.com/repo/debian/dists/wheezy/snapshots/](https://download.mono-project.com/repo/debian/dists/wheezy/snapshots/) 获得。

该脚本安装以下 Mono 包：

* __mono-complete__

* __ca-certificates-mono__

## <a name="the-script"></a>脚本

__脚本位置__：[https://hdiconfigactions.blob.core.windows.net/install-mono/install-mono.bash](https://hdiconfigactions.blob.core.windows.net/install-mono/install-mono.bash)

__要求__：

* 该脚本必须应用在__头节点__和__工作节点__上。

## <a name="to-use-the-script"></a>使用脚本

有关如何在 HDInsight 中使用此脚本的信息，请参阅[使用脚本操作自定义基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/#apply-a-script-action-to-a-running-cluster)一文。 可以通过 Azure 门户、Azure PowerShell 或 Azure CLI 使用此脚本。

按照脚本操作文档进行操作时，请使用以下 URI：

    https://hdiconfigactions.blob.core.windows.net/install-mono/install-mono.bash

> [AZURE.NOTE]
> 使用此脚本配置 HDInsight 时，请将此脚本标记为 __Persisted__。 此设置允许 HDInsight 将脚本应用于通过缩放操作添加的工作节点。

## <a name="next-steps"></a>后续步骤

已学习如何在 HDInsight 上升级或安装 Mono 的特定版本。 有关如何将 .NET 应用程序与 HDInsight 上的 Mono 配合使用的详细信息，请参阅以下文档：

* [使用 .NET 流式处理 HDInsight 上的 MapReduce](/documentation/articles/hdinsight-hadoop-dotnet-csharp-mapreduce-streaming/)
* [在 HDInsight 中将 .NET 与 Hive 和 Pig 配合使用](/documentation/articles/hdinsight-hadoop-hive-pig-udf-dotnet-csharp/)
* [使用 HDInsight 上的 Storm 开发 C# 解决方案](/documentation/articles/hdinsight-storm-develop-csharp-visual-studio-topology/)
* [将 .NET 解决方案迁移到基于 Linux 的 HDInsight](/documentation/articles/hdinsight-hadoop-migrate-dotnet-to-linux/)

有关使用脚本操作的详细信息，请参阅[使用脚本操作自定义基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)