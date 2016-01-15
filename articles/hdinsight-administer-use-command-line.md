<properties
	pageTitle="使用 Azure CLI 管理 Hadoop 群集 | Windows Azure"
	description="如何使用 Azure CLI 管理 HDInsight 中的 Hadoop 群集"
	services="hdinsight"
	editor="cgronlun"
	manager="paulettm"
	authors="mumian"
	tags="azure-portal"
	documentationCenter=""/>

<tags
	ms.service="hdinsight"
	ms.date="12/16/2015"
	wacn.date="01/14/2016"/>

# 使用 Azure CLI 管理 HDInsight 中的 Hadoop 群集

[AZURE.INCLUDE [选择器](../includes/hdinsight-portal-management-selector.md)]

了解如何使用 [Azure 命令行界面](/documentation/articles/xplat-cli-install)管理 Azure HDInsight 中的 Hadoop 群集。Azure CLI 是以 Node.js 实现的。可以在支持 Node.js 的任意平台上使用它。

本文仅介绍如何将 Azure CLI 与 HDInsight 配合使用。有关如何使用 Azure CLI 的常规指南，请参阅[安装和配置 Azure CLI][azure-command-line-tools]。

##先决条件

在开始阅读本文前，你必须具有：

- **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
- **Azure CLI** - 有关安装和配置信息，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install)。
- 使用以下命令**连接到 Azure**：

		azure login -e AzureChinaCloud

	有关使用公司或学校帐户进行身份验证的详细信息，请参阅[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect)。

若要获得帮助，请使用 **-h** 开关。例如：

	azure hdinsight cluster create -h
	
##创建群集

[AZURE.INCLUDE [provisioningnote](../includes/hdinsight-provisioning.md)]

若要创建 HDInsight 群集，你必须指定以下信息：


- **HDInsight 群集名称**

- **位置**：支持 HDInsight 群集的 Azure 数据中心之一。有关支持的位置的列表，请参阅 [HDInsight 定价](/home/features/hdinsight/#price)。

- **默认存储帐户**：HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户，然后才能创建 HDInsight 群集。

	若要创建新的 Azure 存储帐户：
	
		azure storage account create "<Azure Storage Account Name>" -l "<Azure Location>" --type LRS

	> [AZURE.NOTE]存储帐户必须与 HDInsight 共置于同一数据中心。存储帐户类型不能为 ZRS，因为 ZRS 不支持表。
	
	如果你已有存储帐户但是不知道帐户名称和帐户密钥，则可以使用以下命令来检索该信息：
	
		-- Lists Storage accounts
		azure storage account list
		-- Shows a Storage account
		azure storage account show "<Storage Account Name>"
		-- Lists the keys for a Storage account
		azure storage account keys list "<Storage Account Name>"

	有关使用 Azure 管理门户获取信息的详细信息，请参阅[创建、管理或删除存储帐户][azure-create-storageaccount]中的“查看、复制和重新生成存储访问密钥”部分。

- **(可选)默认 Blob 容器**：如果容器不存在，可使用 **azure hdinsight cluster create** 命令创建它。如果选择预先创建容器，可以使用以下命令：

	azure storage container create --account-name "<Storage Account Name>" --account-key <Storage Account Key> [ContainerName]

准备好存储帐户和 blob 容器后，你就可以创建群集了：

	azure hdinsight cluster create --clusterName <ClusterName> --storageAccountName "<Storage Account Name>" --storageAccountKey <storageAccountKey> --storageContainer <StorageContainer> --nodes <NumberOfNodes> --location <DataCenterLocation> --username <HDInsightClusterUsername> --clusterPassword <HDInsightClusterPassword>

![HDI.CLIClusterCreation][image-cli-clustercreation]

##列出并显示群集详细信息
使用以下命令来列出和显示群集详细信息：

	azure hdinsight cluster list
	azure hdinsight cluster show <Cluster Name>

![HDI.CLIListCluster][image-cli-clusterlisting]


##删除群集
使用以下命令来删除群集：

	azure hdinsight cluster delete <Cluster Name>

##缩放群集

若要使用 Azure PowerShell 更改 Hadoop 群集大小，请从客户端计算机运行以下命令：

	Set-AzureHDInsightClusterSize -ClusterSizeInNodes <NewSize> -name <clustername>

##后续步骤
在本文中，你已了解如何执行不同的 HDInsight 群集管理任务。若要了解更多信息，请参阅下列文章：

* [使用 Azure 管理门户管理 HDInsight][hdinsight-admin-portal]
* [使用 Azure PowerShell 管理 HDInsight][hdinsight-admin-powershell]
* [Azure HDInsight 入门][hdinsight-get-started]
* [如何使用 Azure CLI][azure-command-line-tools]


[azure-command-line-tools]: /documentation/articles/xplat-cli-install
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account
[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/


[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal-v1
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1

[image-cli-account-download-import]: ./media/hdinsight-administer-use-command-line/HDI.CLIAccountDownloadImport.png
[image-cli-clustercreation]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreation.png
[image-cli-clustercreation-config]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreationConfig.png
[image-cli-clusterlisting]: ./media/hdinsight-administer-use-command-line/HDI.CLIListClusters.png "列出并显示群集"

<!---HONumber=Mooncake_0104_2016-->