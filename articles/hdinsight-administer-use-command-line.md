<properties
	pageTitle="使用 Azure CLI 管理 Hadoop 群集 | Azure"
	description="如何使用 Azure CLI 管理 HDInsight 中的 Hadoop 群集"
	services="hdinsight"
	editor="cgronlun"
	manager="paulettm"
	authors="mumian"
	tags="azure-portal"
	documentationCenter=""/>

<tags 
	ms.service="hdinsight" 
	ms.date="09/17/2015"
	wacn.date="10/22/2015"/>

# 使用 Azure 命令行界面 (Azure CLI) 管理 HDInsight 中的 Hadoop 群集

[AZURE.INCLUDE [选择器](../includes/hdinsight-portal-management-selector.md)]

了解如何使用 Azure CLI 管理 Azure HDInsight 中的 Hadoop 群集。Azure CLI 是以 Node.js 实现的。可以在支持 Node.js 的任意平台（包括 Windows、Mac 和 Linux）上使用它。

Azure CLI 是开放源代码。在 GitHub 中管理源代码（网址为 <a href= "https://github.com/azure/azure-xplat-cli">https://github.com/azure/azure-xplat-cli</a>）。

本文仅介绍如何将 Azure CLI 与 HDInsight 配合使用。有关如何使用 Azure CLI 的一般指导，请参阅[如何使用 Azure CLI][azure-command-line-tools]。


##先决条件

在开始阅读本文前，你必须具有：

- **一个 Azure 订阅**。请参阅[获取 Azure 免费试用版](/pricing/1rmb-trial/)。

- **Azure CLI** - 有关安装和配置信息，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli)。

##安装

如果尚未这样做，请使用[安装和配置 Azure CLI](/documentation/articles/xplat-cli) 文档来安装和配置 Azure CLI。

##设置 HDInsight 群集

[AZURE.INCLUDE [provisioningnote](../includes/hdinsight-provisioning.md)]


HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户，然后才能创建 HDInsight 群集。

在导入了 publishsettings 文件后，你可以使用以下命令创建一个存储帐户：

	azure account storage create [options] <StorageAccountName>


> [AZURE.NOTE]存储帐户必须与 HDInsight 共置于同一数据中心。


有关通过使用 Azure 门户创建 Azure 存储帐户的信息，请参阅[创建、管理或删除存储帐户][azure-create-storageaccount]。

如果你已有存储帐户但是不知道帐户名称和帐户密钥，则可以使用以下命令来检索该信息：

	-- Lists Storage accounts
	azure account storage list
	-- Shows a Storage account
	azure account storage show <StorageAccountName>
	-- Lists the keys for a Storage account
	azure account storage keys list <StorageAccountName>

有关使用 Azure 门户获取信息的详细信息，请参阅[创建、管理或删除存储帐户][azure-create-storageaccount]中的“查看、复制和重新生成存储访问密钥”部分。


如果容器不存在，可使用 **azure hdinsight cluster create** 命令创建它。如果选择预先创建容器，可以使用以下命令：

	azure storage container create --account-name <StorageAccountName> --account-key <StorageAccountKey> [ContainerName]
		
准备好存储帐户和 blob 容器后，你就可以创建群集了：

	azure hdinsight cluster create --clusterName <ClusterName> --storageAccountName <StorageAccountName> --storageAccountKey <storageAccountKey> --storageContainer <StorageContainer> --nodes <NumberOfNodes> --location <DataCenterLocation> --username <HDInsightClusterUsername> --clusterPassword <HDInsightClusterPassword>

![HDI.CLIClusterCreation][image-cli-clustercreation]

















##使用配置文件设置 HDInsight 群集
通常，你设置一个 HDInsight 群集，对其运行作业，然后删除该群集以降低成本。在命令行界面上，你可以选择将配置保存到文件，以便在每次设置群集时重用这些配置。
 
	azure hdinsight cluster config create <file>
	 
	azure hdinsight cluster config set <file> --clusterName <ClusterName> --nodes <NumberOfNodes> --location "<DataCenterLocation>" --storageAccountName "<StorageAccountName>.blob.core.chinacloudapi.cn" --storageAccountKey "<StorageAccountKey>" --storageContainer "<BlobContainerName>" --username "<Username>" --clusterPassword "<UserPassword>"
	 
	azure hdinsight cluster config storage add <file> --storageAccountName "<StorageAccountName>.blob.core.chinacloudapi.cn"
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




##后续步骤
在本文中，你已了解如何执行不同的 HDInsight 群集管理任务。若要了解更多信息，请参阅下列文章：

* [使用 Azure 门户管理 HDInsight][hdinsight-admin-portal]
* [使用 Azure PowerShell 管理 HDInsight][hdinsight-admin-powershell]
* [Azure HDInsight 入门][hdinsight-get-started]
* [如何使用 Azure CLI][azure-command-line-tools]


[azure-command-line-tools]: /documentation/articles/xplat-cli/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account/
[azure-purchase-options]: /pricing/overview/
[azure-trial]: /pricing/1rmb-trial/


[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal-v1/
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell/
[hdinsight-get-started]: /documentation/articles/hdinsight-get-started/

[image-cli-account-download-import]: ./media/hdinsight-administer-use-command-line/HDI.CLIAccountDownloadImport.png
[image-cli-clustercreation]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreation.png
[image-cli-clustercreation-config]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreationConfig.png
[image-cli-clusterlisting]: ./media/hdinsight-administer-use-command-line/HDI.CLIListClusters.png "列出并显示群集"

<!---HONumber=74-->