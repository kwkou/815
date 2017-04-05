<properties
    pageTitle="为基于 Linux 的 HDInsight 群集配置 OS 修补计划 - Azure | Azure"
    description="了解如何为基于 Linux 的 HDInsight 群集配置 OS 修补计划。"
    services="hdinsight"
    documentationcenter=""
    author="bhanupr"
    manager="asadk"
    editor="bhanupr" />
<tags
    ms.assetid=""
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="03/14/2017"
    wacn.date="03/31/2017"
    ms.author="bhanupr" />  


# 如何为基于 Linux 的 HDInsight 群集配置 OS 修补计划
有时需要重新启动 HDInsight 群集中的虚拟机，以便安装重要的安全修补程序。自 2016 年 8 月 1 日起，将按以下计划重新启动新的基于 Linux 的 HDInsight 群集（3.4 或更高版本）：

1. 安装修补程序时，群集中的虚拟机最多只能在 30 天内重新启动一次。
2. 重新启动在 UTC 时间晚上 12 点开始。
3. 重新启动过程在群集中各个虚拟机之间交错进行，因此群集在重新启动过程中仍然可用。
4. 对于新创建的群集，第一次重新启动的时间不会早于群集创建日期之后的 30 天。

可以使用本文中介绍的脚本操作将 OS 修补计划修改如下：
1. 启用或禁用自动重新启动
2. 设置重新启动的频率（两次重新启动相隔的天数）
3. 设置在星期几重新启动

> [AZURE.NOTE]
此脚本操作将仅适用于在 2016 年 8 月 1 日以后创建的基于 Linux 的 HDInsight 群集。修补程序在 VM 重新启动后才生效。
>

## 如何使用此脚本 

使用此脚本时，需要以下信息：
1. 脚本位置：https://hdiconfigactions.blob.core.windows.net/linuxospatchingrebootconfigv01/os-patching-reboot-config.sh。HDInsight 使用该 URI 查找和运行群集的所有虚拟机上的该脚本。
  
2. 脚本所适用的群集节点类型：头节点、辅助角色节点、zookeeper。

 > [AZURE.NOTE]
 此脚本必须适用于群集中的所有节点类型。如果此脚本不适用于某个节点类型，则该节点类型的虚拟机会继续使用以前的修补计划。
>

3.  参数：此脚本接受三个数字参数：

    | 参数 | 定义 |
    | --- | --- |
    | 启用/禁用自动重新启动 |0 或 1。值为 0 将禁用自动重新启动，值 为 1 将启用自动重新启动。 |
    | 频率 |7 到 90（含）。在重新启动虚拟机之前需等待的天数，适用于需要重新启动的修补程序。 |
    | 星期几 |1 到 7（含）。值为 1 表示应在星期一重新启动，值为 7 表示应在星期日重新启动。例如，使用参数 1 60 2 时，至多每隔 60 天就会在星期二自动重新启动。 |
    | 持久性 |向现有群集应用脚本操作时，可将脚本标记为持久性脚本。通过缩放操作将新的辅助角色节点添加到群集时，会应用持久性脚本。 |

> [AZURE.NOTE]
在应用到现有群集时，必须将该脚本标记为持久性脚本。否则，任何通过缩放操作创建的新节点都会使用默认的修补计划。如果在群集创建过程中应用脚本，则会自动将其变为持久性脚本。
>

## 后续步骤

有关使用脚本操作的具体步骤，请参阅[使用脚本操作自定义基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)的以下部分：

* [在创建群集期间使用脚本操作](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/#use-a-script-action-during-cluster-creation)
* [将脚本操作应用到正在运行的群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/#apply-a-script-action-to-a-running-cluster)

<!---HONumber=Mooncake_0327_2017-->