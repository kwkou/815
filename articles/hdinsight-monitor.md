<properties linkid="manage-services-hdinsight-howto-monitor-hdinsight" urlDisplayName="Monitor" pageTitle="Monitor HDInsight | Azure" metaKeywords="" description="Learn how to monitor an HDInsight cluster and view Hadoop job history through the Azure management portal." metaCanonical="" services="hdinsight" documentationCenter="" title="How to Monitor HDInsight" authors="jgao" solutions="" manager="paulettm" editor="mollybos" />

# 如何监视 HDInsight

在本主题中，你将了解如何监视 HDInsight 群集。

## 目录

-   [如何：监视 HDInsight 群集][]
-   [如何：查看 Hadoop 作业历史记录][]

## 如何：监视 HDInsight 群集

若要监视 HDInsight 群集以及其上运行的 Hadoop 作业的运行状况，你可以连接到 HDInsight 仪表板，然后单击“监视群集”磁贴。

![HDI.TileMonitorCluster][]

“监视”页与下面类似：

![HDI.MonitorPage][]

该页面的右侧指明了 Namenode 和作业跟踪器正在运行，并且有 4 个数据节点正在正常运行。

该页面的左侧显示了过去 30 分钟的映射精简度量值。你可以将监视窗口更改为 30 分钟、1 小时、3 小时、12 小时、1 天、2 天、1 周和 2 周。

## 如何：查看 Hadoop 作业历史记录

若要查看 Hadoop 作业历史记录，请连接到 HDInsight 仪表板，然后单击“作业历史记录”磁贴。

![HDI.TileJobHistory][]

该磁贴显示已经运行过的作业的数量；例如，前一张图指明 6 个作业有作业历史记录。“作业历史记录”页与下面类似：

![HDI.JobHistoryPage][]

## 另请参阅

-   [如何：管理 HDInsight][]
-   [如何：以编程方式部署 HDInsight 群集][]
-   [如何：定期在你的 HDInsight 群集上执行远程作业][]
-   [教程：Azure HDInsight 入门][]

  [如何：监视 HDInsight 群集]: #monitorcluster
  [如何：查看 Hadoop 作业历史记录]: #jobhistory
  [HDI.TileMonitorCluster]: ./media/hdinsight-monitor/HDI.TileMonitorCluster.PNG
  [HDI.MonitorPage]: ./media/hdinsight-monitor/HDI.MonitorPage.PNG
  [HDI.TileJobHistory]: ./media/hdinsight-monitor/HDI.TileJobHistory.PNG
  [HDI.JobHistoryPage]: ./media/hdinsight-monitor/HDI.JobHistoryPage.PNG
  [如何：管理 HDInsight]: /en-us/manage/services/hdinsight/howto-administer-hdinsight/
  [如何：以编程方式部署 HDInsight 群集]: /en-us/manage/services/hdinsight/howto-deploy-cluster/
  [如何：定期在你的 HDInsight 群集上执行远程作业]: /en-us/manage/services/hdinsight/howto-execute-jobs-programmatically/
  [教程：Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
