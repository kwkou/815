<properties
   	pageTitle="使用 Azure PowerShell 在 HDInsight 中的 Linux 上创建 Hadoop、HBase、Storm 或 Spark 群集 | Azure"
   	description="了解如何使用 Azure PowerShell 在 HDInsight 中的 Linux 上创建 Hadoop、HBase、Storm 或 Spark 群集。"
   	services="hdinsight"
   	documentationCenter=""
   	authors="nitinme"
   	manager="jhubbard"
   	editor="cgronlun"
	tags="azure-portal"/>

<tags
   	ms.service="hdinsight"
   	ms.devlang="na"
   	ms.topic="article"
   	ms.tgt_pltfrm="na"
   	ms.workload="big-data"
   	ms.date="10/05/2016"
   	wacn.date="02/06/2017"
   	ms.author="nitinme"/>

# 使用 Azure PowerShell 在 HDInsight 中创建基于 Linux 的群集

[AZURE.INCLUDE [选择器](../../includes/hdinsight-selector-create-clusters.md)]

Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。本文档介绍如何使用 Azure PowerShell 创建基于 Linux 的 HDInsight 群集。此外，还提供了示例脚本。

> [AZURE.NOTE] Azure PowerShell 仅在 Windows 客户端上可用。如果使用的是 Linux、Unix 或 Mac OS X 客户端，请参阅 [Create a Linux-based HDInsight cluster using Azure CLI](/documentation/articles/hdinsight-hadoop-create-linux-clusters-azure-cli/)（使用 Azure CLI 创建基于 Linux 的 HDInsight 群集），了解如何使用 Azure CLI 创建群集。

## 先决条件
开始执行此过程之前请做好以下准备：

- Azure 订阅。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。

- Azure PowerShell。有关将 Azure PowerShell 与 HDInsight 配合使用的详细信息，请参阅 [Administer HDInsight using PowerShell](/documentation/articles/hdinsight-administer-use-powershell/)（使用 PowerShell 管理 HDInsight）。有关 HDInsight Windows PowerShell cmdlet 的列表，请参阅 [HDInsight cmdlet reference](https://msdn.microsoft.com/zh-cn/library/azure/dn858087.aspx)（HDInsight cmdlet 参考）。

    [AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

### 访问控制要求

[AZURE.INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## 创建群集

[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

若要使用 Azure PowerShell 创建 HDInsight 群集，必须完成以下过程：

- 创建 Azure 资源组
- 创建 Azure 存储帐户
- 创建 Azure Blob 容器
- 创建 HDInsight 群集

创建 Linux 群集必须设置的两个最重要参数是用于指定 OS 类型和 SSH 用户详细信息的参数：

- 确保将 **-OSType** 参数指定为 **Linux**。
- 若要在群集上对远程会话使用 SSH，可以指定 SSH 用户密码或 SSH 公钥。如果你同时指定 SSH 用户密码和 SSH 公钥，则将忽略该密钥。如果你要对远程会话使用 SSH 密钥，则必须在出现提示时指定空 SSH 密码。有关如何将 SSH 与 HDInsight 配合使用的详细信息，请参阅以下文章之一：

    * [在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)
    * [在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)

以下脚本演示了如何创建新群集：

    $token ="<SpecifyAnUniqueString>"

    $resourceGroupName = $token + "rg"      # Provide a Resource Group name
    $clusterName = $token
    $defaultStorageAccountName = $token + "store"   # Provide a Storage account name
    $defaultStorageContainerName = $token + "container"
    $location = "China East"     # Change the location if needed
    $clusterNodes = 1           # The number of nodes in the HDInsight cluster

    # Sign in to Azure
    Login-AzureRmAccount

    # Select the subscription to use if you have multiple subscriptions
    #$subscriptionID = "<SubscriptionName>"        # Provide your Subscription Name
    #Select-AzureRmSubscription -SubscriptionId $subscriptionID

    # Create an Azure Resource Group
    New-AzureRmResourceGroup -Name $resourceGroupName -Location $location

    # Create an Azure Storage account and container used as the default storage
    New-AzureRmStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -StorageAccountName $defaultStorageAccountName `
        -Location $location `
        -Type Standard_LRS
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -Name $defaultStorageAccountName -ResourceGroupName $resourceGroupName)[0].Key1
    $destContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey
    New-AzureStorageContainer -Name $defaultStorageContainerName -Context $destContext

    # Create an HDInsight cluster
    $credentials = Get-Credential -Message "Enter Cluster user credentials" -UserName "admin"
    $sshCredentials = Get-Credential -Message "Enter SSH user credentials"

    # The location of the HDInsight cluster must be in the same data center as the Storage account.
    $location = Get-AzureRmStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $defaultStorageAccountName | %{$_.Location}

    New-AzureRmHDInsightCluster `
        -ClusterName $clusterName `
        -ResourceGroupName $resourceGroupName `
        -HttpCredential $credentials `
        -Location $location `
        -DefaultStorageAccountName "$defaultStorageAccountName.blob.core.chinacloudapi.cn" `
        -DefaultStorageAccountKey $defaultStorageAccountKey `
        -DefaultStorageContainer $defaultStorageContainerName  `
        -ClusterSizeInNodes $clusterNodes `
        -ClusterType Hadoop `
        -OSType Linux `
        -Version "3.5" `
        -SshCredential $sshCredentials

你为 **$clusterCredentials** 指定的值用于创建群集的 Hadoop 用户帐户。使用此帐户连接到群集。

为 **$sshCredentials** 指定的值用于创建群集的 SSH 用户。使用此帐户在群集上启动远程 SSH 会话和运行作业。

> [AZURE.IMPORTANT] 在此脚本中，必须指定群集中要包含的工作节点数。如果计划使用 32 个以上的辅助角色节点（在创建群集时配置或者是在创建之后通过扩展群集来配置），则还必须指定至少具有 8 个核心和 14 GB RAM 的头节点大小。
><p>
> 有关节点大小和相关费用的详细信息，请参阅 [HDInsight pricing](/pricing/details/hdinsight/)（HDInsight 定价）。

创建群集可能需要 20 分钟。

下面的示例演示了如何添加其他存储帐户：

    # Create another storage account used as additional storage account
    $additionalStorageAccountName = $token + "store2"
    New-AzureRmStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $additionalStorageAccountName -Location $location -Type Standard_LRS
    $additionalStorageAccountKey = (Get-AzureRmStorageAccountKey -Name $additionalStorageAccountName -ResourceGroupName $resourceGroupName)[0].Value

    $config = New-AzureRmHDInsightClusterConfig
    Add-AzureRmHDInsightStorage -Config $config -StorageAccountName "$additionalStorageAccountName.blob.core.chinacloudapi.cn" -StorageAccountKey $additionalStorageAccountKey

    # Create a new HDInsight cluster
    New-AzureRmHDInsightCluster `
        -ClusterName $clusterName `
        -ResourceGroupName $resourceGroupName `
        -HttpCredential $credentials `
        -Location $location `
        -DefaultStorageAccountName "$defaultStorageAccountName.blob.core.chinacloudapi.cn" `
        -DefaultStorageAccountKey $defaultStorageAccountKey `
        -DefaultStorageContainer $defaultStorageContainerName  `
        -ClusterSizeInNodes $clusterNodes `
        -ClusterType Hadoop `
        -OSType Linux `
        -Version "3.5" `
        -SshCredential $sshCredentials `
        -Config $config

## 自定义群集

- 请参阅 [Customize HDInsight clusters using Bootstrap](/documentation/articles/hdinsight-hadoop-customize-cluster-bootstrap/#use-azure-powershell)（使用 Bootstrap 自定义 HDInsight 群集）。
- 请参阅 [Customize Windows-based HDInsight clusters using Script Action](/documentation/articles/hdinsight-hadoop-customize-cluster/#call-scripts-using-azure-powershell)（使用脚本操作自定义基于 Windows 的 HDInsight 群集）。

## 删除群集

[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## 后续步骤

成功创建 HDInsight 群集后，请通过以下资源了解如何使用群集。

### Hadoop 群集

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

### HBase 群集

* [Get started with HBase on HDInsight（HBase on HDInsight 入门）](/documentation/articles/hdinsight-hbase-tutorial-get-started/)
* [Develop Java applications for HBase on HDInsight（为 HBase on HDInsight 开发 Java 应用程序）](/documentation/articles/hdinsight-hbase-build-java-maven-linux/)

### Storm 群集

* [Develop Java topologies for Storm on HDInsight（为 Storm on HDInsight 开发 Java 拓扑）](/documentation/articles/hdinsight-storm-develop-java-topology/)
* [Use Python components in Storm on HDInsight（在 Storm on HDInsight 中使用 Python 组件）](/documentation/articles/hdinsight-storm-develop-python-topology/)
* [Deploy and monitor topologies with Storm on HDInsight（使用 Storm on HDInsight 部署和监视拓扑）](/documentation/articles/hdinsight-storm-deploy-monitor-topology/)

### Spark 群集

* [使用 Scala 创建独立的应用程序](/documentation/articles/hdinsight-apache-spark-create-standalone-application/)
* [使用 Livy 在 Spark 群集中远程运行作业](/documentation/articles/hdinsight-apache-spark-livy-rest-interface/)
* [Spark 和 BI：使用 HDInsight 中的 Spark 和 BI 工具执行交互式数据分析](/documentation/articles/hdinsight-apache-spark-use-bi-tools/)
* [Spark 和机器学习：使用 HDInsight 中的 Spark 预测食品检查结果](/documentation/articles/hdinsight-apache-spark-machine-learning-mllib-ipython/)
* [Spark 流式处理：使用 HDInsight 中的 Spark 生成实时流式处理应用程序](/documentation/articles/hdinsight-apache-spark-eventhub-streaming/)

<!---HONumber=Mooncake_1107_2016-->