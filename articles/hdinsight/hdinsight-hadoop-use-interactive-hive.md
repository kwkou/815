<!-- not suitable for Mooncake -->


<properties
	pageTitle="在 HDInsight 中使用交互式 Hive | Azure"
	description="了解如何在 HDInsight 中使用交互式 Hive（基于 LLAP 的 Hive）。"
	keywords=""
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian" 
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/05/2016"
	wacn.date="02/06/2017"
	ms.author="jgao"/>  



# 在 HDInsight（预览版）中使用交互式 Hive

交互式 Hive（也称为 [Live Long and Process](https://cwiki.apache.org/confluence/display/Hive/LLAP)）是一个新的 HDInsight [群集类型](/documentation/articles/hdinsight-provision-clusters/#cluster-types)。交互式 Hive 允许在内存中缓存，这使 Hive 查询具有更好的交互性和更加快速。该新功能通过与 R 服务的深度集成，凭借内存缓存（使用 Hive 和 Spark）和高级分析功能，使 HDInsight 成为云中的性能最高、最灵活和开放的大数据解决方案。

交互式 Hive 群集与 Hadoop 群集不同。它只包含 Hive 服务。

> [AZURE.NOTE] 将很快从该群集类型中删除 MapReduce、Pig、Sqoop、Oozie 和其他服务。只能通过 Ambari Hive 视图、Beeline 和 Hive ODBC 访问交互式 Hive 群集中的 Hive 服务。无法通过 Hive 控制台、Templeton、Azure CLI 和 Azure PowerShell 访问它。


 


## 创建交互式 Hive 群集

只有基于 Linux 的群集支持交互式 Hive 群集。有关创建 HDInsight 群集的信息，请参阅[在 HDInsight 中创建基于 Linux 的 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters/)。


## 从交互式 Hive 执行 Hive

执行 Hive 查询的方式有多种：

- 使用 Ambari Hive 视图运行 Hive

- 使用 Beeline 运行 Hive

	有关在 HDInsight 上使用 Beeline 的信息，请参阅[在 HDInsight 中通过 Beeline 使用 Hive 和 Hadoop](/documentation/articles/hdinsight-hadoop-use-hive-beeline/)。

	可以在头节点或空的边缘节点中使用 Beeline。建议在空的边缘节点中使用 Beeline。有关创建具有空的边缘节点的 HDInsight 群集的信息，请参阅[在 HDInsight 中使用空的边缘节点](/documentation/articles/hdinsight-apps-use-edge-node/)。

- 使用 Hive ODBC 运行 Hive

	有关使用 Hive ODBC 的信息，请参阅[使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 Hadoop](/documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/)。

**若要查找 JDBC 连接字符串：**

1.	使用以下 URL 登录到 Ambari：https://<ClusterName>.AzureHDInsight.net。
2.	在左侧菜单中，单击“Hive”。
3.	单击突出显示图标以复制 URL：

	![HDInsight Hadoop 交互式 Hive LLAP JDBC](./media/hdinsight-hadoop-use-interactive-hive/hdinsight-hadoop-use-interactive-hive-jdbc.png)  


## 另请参阅
-	[在 HDInsight 中创建基于 Linux 的 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters/)：了解如何在 HDInsight 中创建交互式 Hive 群集。
-	[在 HDInsight 中通过 Beeline 使用 Hive 和 Hadoop](/documentation/articles/hdinsight-hadoop-use-hive-beeline/)：了解如何使用 Beeline 提交 Hive 查询。
-	[使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 Hadoop](/documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/)：了解如何将 Excel 连接到 Hive。

<!---HONumber=Mooncake_1107_2016-->