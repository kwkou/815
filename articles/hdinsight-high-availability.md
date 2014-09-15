<properties linkid="manage-services-hdinsight-high-availability" urlDisplayName="HDInsight High Availability" pageTitle="Availability and reliability of HDInsight Clusters | Azure" metaKeywords="hdinsight, hadoop, hdinsight hadoop, hadoop azure" description="HDInsight deploys highly available and reliable clusters." services="HDInsight" umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" title="Availability and reliability of HDInsight clusters" authors="bradsev" />

# HDInsight 群集的可用性和可靠性

## 介绍

已将第二个头节点添加到 HDInsight 所部署的 Hadoop 群集，以增加管理工作负载所需的服务的可用性和可靠性。Hadoop 群集的标准实现通常具有单个头节点。这些群集设计为顺畅管理工作节点故障，不过在头节点上运行的主控服务的任何停机都会导致群集停止工作。

![][]

HDInsight 通过添加辅助头节点 (Head Node1) 去除了此单点故障。[ZooKeeper][] 节点 (ZK) 已经添加，用于头节点的 Leader Election，以确保工作节点和网关 (GW) 知道当活动头节点 (HeadNode0) 变为不活动时何时故障转移到辅助头节点 (Head Node1)。

## 如何检查活动头节点上的服务状态

若要确定哪个头节点处于活动状态，以及检查该头节点上运行的服务的状态，你必须使用远程桌面协议 (RDP) 连接到 Hadoop 群集。在 Azure 中，默认情况下会禁用远程连接到群集的功能，所以必须先启用该功能。有关如何在门户中进行此操作的说明，请参阅[使用 RDP 连接到 HDInsight 群集][]。
一旦远程连接到群集，请双击桌面上的“Hadoop 服务可用状态” 图标，以便通过相关状态来了解 Namenode、Jobtracker、Templeton、Oozieservice、Metastore 和 Hiveserver2 服务正在运行哪个头节点，而在 HDI 3.0 上，则可通过相关状态来了解 Namenode、Resource Manager、History Server、Templeton、Oozieservice、Metastore 和 Hiveserver2 服务正在运行哪个头节点。

![][1]

## 如何访问辅助性头节点上的日志文件

当辅助性头节点变为活动头节点时，可以通过浏览 JobTracker UI 来访问辅助性头节点上的作业日志，就像是在主活动节点上操作一样。若要访问作业跟踪器，你必须按前一节中的说明使用远程桌面协议 (RDP) 连接到 Hadoop 群集。一旦远程连接到了该群集，则请双击位于桌面上的“Hadoop 名称节点 ”图标，然后单击“名称节点日志” ，即可转到辅助性头节点上的日志目录。

![][2]

## 如何配置头节点的大小

默认情况下，头节点被分配为大型 VM。管理在群集上运行的大多数 Hadoop 作业只需要这种大小。但是，有的方案可能要求为头节点设置特大型 VM。例如，当群集需要管理大量的小型 Oozie 作业时。

使用 Azure PowerShell cmdlet 或 HDInsight SDK 均可配置特大型 VM。

使用 PowerShell 创建和设置群集的内容记录在[使用 PowerShell 管理 HDInsight][] 中。配置特大型头节点需要将 `-HeadNodeVMSize ExtraLarge` 参数添加到此代码中使用的 `New-AzureHDInsightcluster` cmdlet。

    # 在 Azure PowerShell 中创建新的 HDInsight 群集
    # 已配置特大型头节点 VM
    New-AzureHDInsightCluster -Name $clusterName -Location $location -HeadNodeVMSize ExtraLarge -DefaultStorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainerName $containerName  -ClusterSizeInNodes $clusterNodes

SDK 的情形是类似的。使用 SDK 创建和设置群集的内容记录在[使用 HDInsight .NET SDK][] 中。配置特大型头节点需要将 `HeadNodeSize = NodeVMSize.ExtraLarge` 参数添加到此代码中使用的 `ClusterCreateParameters()` 方法。

    # 使用 HDInsight SDK 创建新的 HDInsight 群集
    # 已配置特大型头节点 VM
    ClusterCreateParameters clusterInfo = new ClusterCreateParameters()
    {
    Name = clustername,
    Location = location,
    HeadNodeSize = NodeVMSize.ExtraLarge,
    DefaultStorageAccountName = storageaccountname,
    DefaultStorageAccountKey = storageaccountkey,
    DefaultStorageContainer = containername,
    UserName = username,
    Password = password,
    ClusterSizeInNodes = clustersize
    };

**参考**

-   [ZooKeeper][]
-   [使用 RDP 连接到 HDInsight 群集][]
-   [使用 HDInsight .NET SDK][]

  []: http://i.imgur.com/jrUmrH4.png
  [ZooKeeper]: http://zookeeper.apache.org/
  [使用 RDP 连接到 HDInsight 群集]: http://azure.microsoft.com/zh-cn/documentation/articles/hdinsight-administer-use-management-portal/#rdp
  [1]: http://i.imgur.com/MYTkCHW.png
  [2]: http://i.imgur.com/eL6jzgB.png
  [使用 PowerShell 管理 HDInsight]: http://azure.microsoft.com/zh-cn/documentation/articles/hdinsight-administer-use-powershell/
  [使用 HDInsight .NET SDK]: http://azure.microsoft.com/zh-cn/documentation/articles/hdinsight-provision-clusters/#sdk
