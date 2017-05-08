<properties
    pageTitle="为基于 Linux 的 HDInsight 群集配置 OS 修补计划 - Azure | Azure"
    description="了解如何为基于 Linux 的 HDInsight 群集配置 OS 修补计划。"
    services="hdinsight"
    documentationcenter=""
    author="bprakash"
    manager="asadk"
    editor="bprakash"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="03/21/2017"
    wacn.date="05/08/2017"
    ms.author="bhanupr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="c23a62a837b44f4c2073afa47c503760ff73959b"
    ms.lasthandoff="04/28/2017" />

# <a name="os-patching-for-hdinsight"></a>针对 HDInsight 的 OS 修补 
作为托管的 Hadoop 服务，HDInsight 负责修补 HDInsight 群集使用的基础 VM 的 OS。 自 2016 年 8 月 1 日起，我们已经为基于 Linux 的 HDInsight 群集（版本 3.4 或更高版本）更改了来宾 OS 修补策略。 新策略的目标是显著减少由于修补导致的重启次数。 新策略将继续修补 Linux 群集上的虚拟机 (VM)，开始时间为每个星期一或星期四的凌晨 0 点 (UTC)，采用在任何给定群集中跨节点进行交错修补的方式。 但是，任何给定 VM 最多只会因来宾 OS 修补而 30 天重启一次。 此外，自新创建群集的创建日期起 30 天内，该群集不会发生第一次重启。 重启 VM 后，修补程序即可生效。

## <a name="how-to-configure-the-os-patching-schedule-for-linux-based-hdinsight-clusters"></a>如何为基于 Linux 的 HDInsight 群集配置 OS 修补计划
需不定期重启 HDInsight 群集中的虚拟机，以便安装重要的安全修补程序。 自 2016 年 8 月 1 日起，已使用以下计划重启新的基于 Linux 的 HDInsight 群集（版本 3.4 或更高版本）：

1. 群集中的虚拟机最多只能在 30 天内重启一次以进行修补。
2. 重新启动在 UTC 时间晚上 12 点开始。
3. 重新启动过程在群集中各个虚拟机之间交错进行，因此群集在重新启动过程中仍然可用。
4. 对于新创建的群集，第一次重新启动的时间不会早于群集创建日期之后的 30 天。

可以使用本文中介绍的脚本操作将 OS 修补计划修改如下：
1. 启用或禁用自动重新启动
2. 设置重新启动的频率（两次重新启动相隔的天数）
3. 设置在星期几重新启动

## <a name="how-to-use-the-script"></a>如何使用此脚本 

使用此脚本需要以下信息：
1. 脚本位置：https://hdiconfigactions.blob.core.windows.net/linuxospatchingrebootconfigv01/os-patching-reboot-config.sh。
     HDInsight 使用此 URI 在群集中的所有虚拟机上查找并运行脚本。

2. 脚本所适用的群集节点类型：头节点、辅助角色节点、zookeeper。 此脚本必须适用于群集中的所有节点类型。 如果此脚本不适用于某个节点类型，则该节点类型的虚拟机会继续使用以前的修补计划。

3.  参数：此脚本接受三个数字参数：

    | 参数 | 定义 |
    | --- | --- |
    | 启用/禁用自动重新启动 |0 或 1。 值为 0 将禁用自动重新启动，值 为 1 将启用自动重新启动。 |
    | 频率 |7 到 90（含）。 在重新启动虚拟机之前需等待的天数，适用于需要重新启动的修补程序。 |
    | 星期几 |1 到 7（含）。 值为 1 表示应在星期一重新启动，值为 7 表示应在星期日重新启动。例如，使用参数 1 60 2 时，至多每隔 60 天就会在星期二自动重新启动。 |
    | 持久性 |向现有群集应用脚本操作时，可将脚本标记为持久性脚本。 通过缩放操作将新的辅助角色节点添加到群集时，会应用持久性脚本。 |

> [AZURE.NOTE]
> 将其应用于现有群集时，必须将此脚本标记为持久化。 否则，通过缩放操作创建的任何新节点都将使用默认修补计划。
如果在群集创建过程中应用该脚本，则其会自动持久化。
>

## <a name="next-steps"></a>后续步骤

若要了解使用脚本操作的具体步骤，请参阅[使用脚本操作自定义基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)中的以下部分:

* [在创建群集期间使用脚本操作](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/#use-a-script-action-during-cluster-creation)
* [将脚本操作应用到正在运行的群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/#apply-a-script-action-to-a-running-cluster)