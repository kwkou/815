<properties
    pageTitle="以编程方式访问 Hadoop YARN 应用程序日志 | Azure"
    description="以编程方式访问 HDInsight 中 Hadoop 群集上的应用程序日志。"
    services="hdinsight"
    documentationcenter=""
    tags="azure-portal"
    author="mumian"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="0198d6c9-7767-4682-bd34-42838cf48fc5"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/06/2017"
    wacn.date="03/10/2017"
    ms.author="jgao" />

# 在基于 Windows 的 HDInsight 上访问 YARN 应用程序日志
本主题介绍如何访问在 Azure HDInsight 中基于 Windows 的 Hadoop 群集上完成的 YARN (Yet Another Resource Negotiator) 应用程序日志

> [AZURE.IMPORTANT]
本文档中的信息仅适用于基于 Windows 的 HDInsight 群集。Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。有关在基于 Linux 的 HDInsight 群集上访问 YARN 日志的信息，请参阅 [Access YARN application logs on Linux-based Hadoop on HDInsight](/documentation/articles/hdinsight-hadoop-access-yarn-app-logs-linux/)（在 HDInsight 上基于 Linux 的 Hadoop 中访问 YARN 应用程序日志）
> 

### 先决条件
* 基于 Windows 的 HDInsight 群集。请参阅[在 HDInsight 中创建基于 Windows 的 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters/)。

## YARN Timeline Server
<a href="http://hadoop.apache.org/docs/r2.4.0/hadoop-yarn/hadoop-yarn-site/TimelineServer.html" target="_blank">YARN Timeline Server</a> 通过两个不同接口提供完成应用程序的通用信息，以及架构特定应用程序信息。具体而言：

* 已通过 3.1.1.374 版或更高版本启用存储和检索 HDInsight 群集的通用应用程序信息。
* Timeline Server 的架构特定应用程序信息组件目前不适用于 HDInsight 群集。

通用应用程序信息包含以下类型的数据：

* 应用程序 ID（应用程序的唯一标识符）
* 启动应用程序的用户
* 尝试完成应用程序的相关信息
* 任何给定应用程序尝试所用的容器

在 HDInsight 群集上，Azure Resource Manager 将此信息存储到默认存储帐户默认容器中的历史记录存储。可以通过 REST API 检索有关完成应用程序的此类通用数据：

    GET on https://<cluster-dns-name>.azurehdinsight.cn/ws/v1/applicationhistory/apps

## <a name="YARNAppsAndLogs"></a> YARN 应用程序和日志
YARN 通过将资源管理与应用程序计划/监视分离，支持多种编程模型（MapReduce 是其中之一）。这可通过全局 *ResourceManager* (RM)、按辅助节点 *NodeManagers* (NM) 和按应用程序 *ApplicationMasters* (AM) 实现。按应用程序 AM 与 RM 协商用于运行应用程序的资源（CPU、内存、磁盘、网络）。RM 与 NM 合作授予这些资源（以*容器*形式授予）。AM 负责跟踪 RM 为其分配容器的进度。根据应用程序性质，应用程序可能需要多个容器。

此外，每个应用程序可能包含多个*应用程序尝试*，以便在应用程序崩溃或因 AM 与 RM 之间通信中断时完成应用程序。因此，容器是授予应用程序的特定尝试。在某种意义上，容器提供 YARN 应用程序运行的基本工作单位的内容，而在容器的上下文中完成的所有工作都在分配给该容器的单个辅助节点上运行。请参阅 [YARN 的概念][YARN-concepts]，以获取更多参考信息。

应用程序日志（和关联容器日志）对于调试有问题的 Hadoop 应用程序至关重要。YARN 提供一个良好的框架，通过使用[日志聚合][log-aggregation]功能收集、聚合和储应用程序日志。日志聚合功能让访问应用程序日志更具确定性，因为该功能可聚合辅助节点上所有容器的日志，并在应用程序完成后，将它们按每个辅助节点一个聚合日志的方式存储在默认文件系统上。应用程序可能使用数百或数千个容器，但在单个辅助节点上运行的所有容器的日志将始终聚合成单个文件，从而为应用程序所用的每个辅助节点生成一个日志。在 HDInsight 群集（3.0 版和更高版本）上默认启用日志聚合，并且可在群集的默认容器中找到聚合日志，位置如下：

    wasbs:///app-logs/<user>/logs/<applicationId>

在该位置中，*user* 是启动应用程序的用户名，*applicationId* 是 YARN RM 分配的应用程序唯一标识符。

无法直接阅读聚合日志，因为它们是以 [TFile][T-file]（由容器编制索引的[二进制格式][binary-format]）编写。YARN 提供 CLI 工具，针对感兴趣的应用程序或容器，将这些日志转储成纯文本。可以直接在群集节点上（通过 RDP 连接到节点后）运行以下 YARN 命令之一，以纯文本格式查看这些日志：

    yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application>
    yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application> -containerId <containerId> -nodeAddress <worker-node-address>

## YARN ResourceManager UI
YARN ResourceManager UI 在群集头节点上运行，可以通过 Azure 门户预览仪表板进行访问：

1. 登录到 [Azure 门户预览](https://portal.azure.cn/)。
2. 在左侧菜单中，依次单击“浏览”、“HDInsight 群集”，单击要访问其 YARN 应用程序日志的基于 Windows 的群集。
3. 在顶部菜单中，单击“仪表板”。你将看到在新的浏览器标签页上打开一个称为“HDInsight 查询控制台”的页面。
4. 在“HDInsight 查询控制台”中，单击“Yarn UI”。

[YARN-timeline-server]: http://hadoop.apache.org/docs/r2.4.0/hadoop-yarn/hadoop-yarn-site/TimelineServer.html
[log-aggregation]: http://hortonworks.com/blog/simplifying-user-logs-management-and-access-in-yarn/
[T-file]: https://issues.apache.org/jira/secure/attachment/12396286/TFile%20Specification%2020081217.pdf
[binary-format]: https://issues.apache.org/jira/browse/HADOOP-3315
[YARN-concepts]: http://hortonworks.com/blog/apache-hadoop-yarn-concepts-and-applications/

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned-->