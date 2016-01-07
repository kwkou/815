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
	ms.date="11/03/2015"
	wacn.date="01/07/2016"/>

# 使用 Azure CLI 管理 HDInsight 中的 Hadoop 群集

[AZURE.INCLUDE [选择器](../includes/hdinsight-portal-management-selector.md)]

了解如何使用 [Azure 命令行界面](/documentation/articles/xplat-cli-install)管理 Azure HDInsight 中的 Hadoop 群集。Azure CLI 是以 Node.js 实现的。可以在支持 Node.js 的任意平台上使用它。

Azure CLI 是开放源代码。在 GitHub 中管理源代码（网址为 <a href= "https://github.com/azure/azure-xplat-cli">https://github.com/azure/azure-xplat-cli</a>）。

本文仅介绍如何将 Azure CLI 与 HDInsight 配合使用。有关如何使用 Azure CLI 的常规指南，请参阅[安装和配置 Azure CLI][azure-command-line-tools]。


##先决条件

在开始阅读本文前，你必须具有：

- **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。

- **Azure CLI** - 有关安装和配置信息，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install)。
- 使用以下命令**连接到 Azure**：

		azure login -e AzureChinaCloud

	有关使用公司或学校帐户进行身份验证的详细信息，请参阅[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect)。
	
- 使用以下命令**切换到 Azure 资源管理器模式**：

		azure config mode arm


##创建群集

[AZURE.INCLUDE [provisioningnote](../includes/hdinsight-provisioning.md)]

在创建 HDInsight 群集之前，你必须拥有 Azure 资源管理 (ARM) 和 Azure Blob 存储帐户。若要创建 HDInsight 群集，你必须指定以下信息：

- **Azure 资源组**：必须在 Azure 资源组中创建一个 Data Lake 分析帐户。那么，你可以使用 Azure 资源管理器以组的方式处理应用程序中的资源。你可以通过一个协调的操作为应用程序部署、更新或删除所有资源。 

	若要列出订阅中的资源组：
	
		azure group list 
	
	若要创建新的资源组：
	
		azure group create -n "<Resource Group Name>" -l "<Azure Location>"

- **HDInsight 群集名称**

- **位置**：支持 HDInsight 群集的 Azure 数据中心之一。有关支持的位置的列表，请参阅 [HDInsight 定价](/home/features/hdinsight/#price)。

- **默认存储帐户**：HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户，然后才能创建 HDInsight 群集。

	若要创建新的 Azure 存储帐户：
	
		azure storage account create "<Azure Storage Account Name>" -g "<Resource Group Name>" -l "<Azure Location>" --type LRS

> [AZURE.NOTE]存储帐户必须与 HDInsight 共置于同一数据中心。
	> 存储帐户类型不能为 ZRS，因为 ZRS 不支持表。
	
	如果你已有存储帐户但是不知道帐户名称和帐户密钥，则可以使用以下命令来检索该信息：
	
		-- Lists Storage accounts
		azure storage account list
		-- Shows a Storage account
		azure storage account show "<Storage Account Name>"
		-- Lists the keys for a Storage account
		azure storage account keys list "<Storage Account Name>" -g "<Resource Group Name>"

有关使用 Azure 管理门户获取信息的详细信息，请参阅[创建、管理或删除存储帐户][azure-create-storageaccount]中的“查看、复制和重新生成存储访问密钥”部分。

- **(可选)默认 Blob 容器**：如果容器不存在，可使用 **azure hdinsight cluster create** 命令创建它。如果选择预先创建容器，可以使用以下命令：

	azure storage container create --account-name "<Storage Account Name>" --account-key <StorageAccountKey> [ContainerName]

准备好存储帐户和 blob 容器后，你就可以创建群集了：

	azure hdinsight cluster create --clusterName <ClusterName> --storageAccountName "<Storage Account Name>" --storageAccountKey <storageAccountKey> --storageContainer <StorageContainer> --nodes <NumberOfNodes> --location <DataCenterLocation> --username <HDInsightClusterUsername> --clusterPassword <HDInsightClusterPassword>

![HDI.CLIClusterCreation][image-cli-clustercreation]

















##使用配置文件创建群集
通常，你创建一个 HDInsight 群集，对其运行作业，然后删除该群集以降低成本。在命令行界面上，你可以选择将配置保存到文件，以便在每次创建群集时可以重用这些配置。

> [AZURE.NOTE]元存储配置不可用于 HBase 群集类型。

	azure hdinsight cluster config create <file>

	azure hdinsight cluster config set <file> --clusterName <ClusterName> --nodes <NumberOfNodes> --location "<DataCenterLocation>" --storageAccountName ""<Storage Account Name>".blob.core.chinacloudapi.cn" --storageAccountKey "<StorageAccountKey>" --storageContainer "<BlobContainerName>" --username "<Username>" --clusterPassword "<UserPassword>"

	azure hdinsight cluster config storage add <file> --storageAccountName ""<Storage Account Name>".blob.core.chinacloudapi.cn"
	       --storageAccountKey "<StorageAccountKey>"

	azure hdinsight cluster config metastore set <file> --type "hive" --server "<SQLDatabaseName>.database.chinacloudapi.cn"
	       --database "<HiveDatabaseName>" --user "<Username>" --metastorePassword "<UserPassword>"

	azure hdinsight cluster config metastore set <file> --type "oozie" --server "<SQLDatabaseName>.database.chinacloudapi.cn"
	       --database "<OozieDatabaseName>" --user "<SQLUsername>" --metastorePassword "<SQLPassword>"

	azure hdinsight cluster create --config <file>



![HDI.CLIClusterCreationConfig][image-cli-clustercreation-config]


##列出并显示群集详细信息
使用以下命令来列出和显示群集详细信息：

	azure hdinsight cluster list
	azure hdinsight cluster show <ClusterName>

![HDI.CLIListCluster][image-cli-clusterlisting]


##删除群集
使用以下命令来删除群集：

	azure hdinsight cluster delete <ClusterName>

##缩放群集

若要使用 Azure PowerShell 更改 Hadoop 群集大小，请从客户端计算机运行以下命令：

	Set-AzureHDInsightClusterSize -ClusterSizeInNodes <NewSize> -name <clustername>

##后续步骤
在本文中，你已了解如何执行不同的 HDInsight 群集管理任务。若要了解更多信息，请参阅下列文章：

* [使用 Azure 管理门户管理 HDInsight][hdinsight-admin-portal]
* [使用 Azure PowerShell 管理 HDInsight][hdinsight-admin-powershell]
* [Azure HDInsight 入门][hdinsight-get-started]
* [如何使用 Azure CLI][azure-command-line-tools]


[azure-command-line-tools]: /documentation/articles/xplat-cli
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

<!---HONumber=Mooncake_1207_2015-->