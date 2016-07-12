<properties
   pageTitle="使用 Azure CLI 在 HDInsight 中创建基于 Windows 的 Hadoop 群集"
   	description="了解如何使用 Azure CLI 创建 Azure HDInsight 的群集。"
   services="hdinsight"
   documentationCenter=""
   tags="azure-portal"
   authors="mumian"
   manager="paulettm"
   editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="03/08/2016"
	wacn.date="04/12/2016"/>

# 使用 Azure CLI 在 HDInsight 中创建基于 Windows 的 Hadoop 群集

[AZURE.INCLUDE [选择器](../includes/hdinsight-selector-create-clusters.md)]

了解如何使用 Azure CLI 创建 HDInsight 群集。有关其他群集创建工具和功能，请单击本页面顶部的相应选项卡，或参阅[群集创建方法](/documentation/articles/hdinsight-provision-clusters-v1/#cluster-creation-methods)。

###先决条件：

[AZURE.INCLUDE [delete-cluster-warning](../includes/hdinsight-delete-cluster-warning.md)]

在开始按照本文中的说明操作之前，你必须具有以下内容：

- **Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
- **Azure CLI** - 有关安装和配置信息，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)。

##连接到 Azure

使用以下命令连接到 Azure：

	azure login -e AzureChinaCloud -u <your account>

有关使用公司或学校帐户进行身份验证的详细信息，请参阅[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)。

若要获得帮助，请使用 **-h** 开关。例如：

	azure hdinsight cluster create -h
	
##创建群集

必须先获取 Azure Blob 存储帐户，然后才能创建 HDInsight 群集。若要创建 HDInsight 群集，必须指定以下信息：

- **HDInsight 群集名称**

- **位置**：支持 HDInsight 群集的 Azure 数据中心之一。有关支持的位置的列表，请参阅 [HDInsight 定价](/home/features/hdinsight/pricing/)。

- **默认存储帐户**：HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户，然后才能创建 HDInsight 群集。

	若要创建新的 Azure 存储帐户：
	
		azure storage account create "<Azure Storage Account Name>" -l "<Azure Location>" --type LRS

	> [AZURE.NOTE] 存储帐户必须与 HDInsight 共置于同一数据中心。存储帐户类型不能为 ZRS，因为 ZRS 不支持表。

	有关使用 Azure 经典管理门户创建 Azure 存储帐户的信息，请参阅 [创建、管理或删除存储帐户][azure-create-storageaccount]。
	
	如果你已有存储帐户但是不知道帐户名称和帐户密钥，则可以使用以下命令来检索该信息：
	
		-- Lists Storage accounts
		azure storage account list
		-- Shows a Storage account
		azure storage account show "<Storage Account Name>"
		-- Lists the keys for a Storage account
		azure storage account keys list "<Storage Account Name>"

	有关使用 Azure 经典管理门户获取信息的详细信息，请参阅 [创建、管理或删除存储帐户][azure-create-storageaccount] 中的“查看、复制和重新生成存储访问密钥”部分。

- **(可选)默认 Blob 容器**：如果容器不存在，可使用 **azure hdinsight cluster create** 命令创建它。如果选择预先创建容器，可以使用以下命令：

	azure storage container create --account-name "<Storage Account Name>" --account-key <Storage Account Key> [ContainerName]

准备好存储帐户后，你就可以创建群集了：

    
    azure hdinsight cluster create -c <HDInsight Cluster Name> -l <Location> --osType Windows --version <Cluster Version> --clusterType <Hadoop | HBase | Spark | Storm> --workerNodeCount 2 --headNodeSize Large --workerNodeSize Large --defaultStorageAccountName <Azure Storage Account Name>.blob.core.chinacloudapi.cn --defaultStorageAccountKey "<Default Storage Account Key>" --defaultStorageContainer <Default Blob Storage Container> --userName admin --password "<HTTP User Password>" --rdpUserName <RDP Username> --rdpPassword "<RDP User Password" --rdpAccessExpiry "03/01/2016"

## 另请参阅

- [Azure HDInsight 入门](/documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/) - 了解如何开始使用你的 HDInsight 群集
- [以编程方式提交 Hadoop 作业](/documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/) - 了解如何以编程方式将作业提交到 HDInsight
- [使用 Azure CLI 管理 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-administer-use-command-line/)
- [将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 服务管理配合使用](/documentation/articles/virtual-machines-command-line-tools/)

<!---HONumber=Mooncake_0215_2016-->