<properties
    pageTitle="使用 Azure CLI 在 HDInsight 中创建基于 Windows 的 Hadoop 群集"
    description="了解如何使用 Azure CLI 创建 Azure HDInsight 群集。"
    services="hdinsight"
    documentationcenter=""
    tags="azure-portal"
    author="mumian"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="c766544c-c16f-4bfa-89d0-592325d24250"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="09/02/2016"
    wacn.date="01/25/2017"
    ms.author="jgao" />

# 使用 Azure CLI 在 HDInsight 中创建基于 Windows 的 Hadoop 群集
[AZURE.INCLUDE [选择器](../../includes/hdinsight-selector-create-clusters.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

了解如何使用 Azure CLI 创建 HDInsight 群集。有关其他群集创建工具和功能，请单击本页面顶部的相应选项卡，或参阅[群集创建方法](/documentation/articles/hdinsight-provision-clusters/#cluster-creation-methods)。

## 先决条件：
[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

在开始按照本文中的说明操作之前，你必须具有以下内容：

* **Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* **Azure CLI**。
  
    [AZURE.INCLUDE [use-latest-version](../../includes/hdinsight-use-latest-cli.md)]

### 访问控制要求
[AZURE.INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## 连接到 Azure
使用以下命令连接到 Azure：

    azure login -e AzureChinaCloud

有关使用公司或学校帐户进行身份验证的详细信息，请参阅[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)。

使用以下命令切换到 ARM 模式：

    azure config mode arm

要获取帮助，请使用 **-h** 开关。例如：

    azure hdinsight cluster create -h

## 创建群集
在创建 HDInsight 群集之前，你必须拥有 Azure 资源管理 (ARM) 和 Azure Blob 存储帐户。若要创建 HDInsight 群集，必须指定以下信息：

* **Azure 资源组**：必须在 Azure 资源组中创建一个 Data Lake 分析帐户。那么，你可以使用 Azure 资源管理器以组的方式处理应用程序中的资源。你可以通过一个协调的操作为应用程序部署、更新或删除所有资源。
  
    若要列出订阅中的资源组：
  
        azure group list
  
    若要创建新的资源组：
  
        azure group create -n "<Resource Group Name>" -l "<Azure Location>"
* **HDInsight 群集名称**
* **位置**：支持 HDInsight 群集的 Azure 数据中心之一。有关支持位置的列表，请参阅 [HDInsight 定价](/pricing/details/hdinsight/)。
* **默认存储帐户**：HDInsight 使用 Azure Blob 存储容器作为默认文件系统。需要先拥有 Azure 存储帐户，然后才能创建 HDInsight 群集。
  
    要创建新 Azure 存储帐户：
  
        azure storage account create "<Azure Storage Account Name>" -g "<Resource Group Name>" -l "<Azure Location>" --type LRS
  
    > [AZURE.NOTE]
    存储帐户必须与 HDInsight 共置于同一数据中心。
    存储帐户类型不能为 ZRS，因为 ZRS 不支持表。
    > 
    > 
  
    有关使用 Azure 门户预览创建 Azure 存储帐户的信息，请参阅[创建、管理或删除存储帐户][azure-create-storageaccount]。
  
    如果已拥有存储帐户但不知道帐户名称和帐户密钥，可使用以下命令检索相关信息：
  
        -- Lists Storage accounts
        azure storage account list
        -- Shows a Storage account
        azure storage account show "<Storage Account Name>"
        -- Lists the keys for a Storage account
        azure storage account keys list "<Storage Account Name>" -g "<Resource Group Name>"
  
    有关使用 Azure 门户预览获取信息的详细信息，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/#manage-your-storage-account)的“管理存储帐户”部分。
* **(可选)默认 Blob 容器**：如果容器不存在，可使用 **azure hdinsight cluster create** 命令创建。如果选择预先创建容器，可以使用以下命令：
  
        azure storage container create --account-name "<Storage Account Name>" --account-key <Storage Account Key> [ContainerName]

准备好存储帐户后，即可创建群集：

    azure hdinsight cluster create -g <Resource Group Name> -c <HDInsight Cluster Name> -l <Location> --osType <Windows | Linux> --version <Cluster Version> --clusterType <Hadoop | HBase | Spark | Storm> --workerNodeCount 2 --headNodeSize Large --workerNodeSize Large --defaultStorageAccountName <Azure Storage Account Name>.blob.core.chinacloudapi.cn --defaultStorageAccountKey "<Default Storage Account Key>" --defaultStorageContainer <Default Blob Storage Container> --userName admin --password "<HTTP User Password>" --rdpUserName <RDP Username> --rdpPassword "<RDP User Password" --rdpAccessExpiry "03/01/2016"

## 使用配置文件创建群集
通常，你创建一个 HDInsight 群集，对其运行作业，然后删除该群集以降低成本。在命令行界面上，你可以选择将配置保存到文件，以便在每次创建群集时可以重用这些配置。

    azure hdinsight config create [options ] <Config File Path> <overwirte>
    azure hdinsight config add-config-values [options] <Config File Path>
    azure hdinsight config add-script-action [options] <Config File Path>

示例：创建一个配置文件，其中包含创建群集时要运行的脚本操作。

    azure hdinsight config create "C:\myFiles\configFile.config"
    azure hdinsight config add-script-action --configFilePath "C:\myFiles\configFile.config" --nodeType HeadNode --uri <Script Action URI> --name myScriptAction --parameters "-param value"
    azure hdinsight cluster create --configurationPath "C:\myFiles\configFile.config"

## 使用脚本操作创建群集
创建一个配置文件，其中包含创建群集时要运行的脚本操作。

    azure hdinsight config create "C:\myFiles\configFile.config"
    azure hdinsight config add-script-action --configFilePath "C:\myFiles\configFile.config" --nodeType HeadNode --uri <scriptActionURI> --name myScriptAction --parameters "-param value"

使用脚本操作创建群集

    azure hdinsight cluster create -g myarmgroup01 -l chinanorth -y Windows --clusterType Hadoop --version 3.2 --defaultStorageAccountName mystorageaccount --defaultStorageAccountKey <defaultStorageAccountKey> --defaultStorageContainer mycontainer --userName admin --password <clusterPassword> --sshUserName sshuser --sshPassword <sshPassword> --workerNodeCount 1 -configurationPath "C:\myFiles\configFile.config" myNewCluster01

有关脚本操作的常规信息，请参阅[使用脚本操作 (Linux) 自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster/)。

## 使用 ARM 模板创建群集
可以通过调用 ARM 模板使用 CLI 创建群集。请参阅[使用 Azure CLI 进行部署](/documentation/articles/hdinsight-hadoop-create-windows-clusters-arm-templates/#deploy-with-azure-cli)。

## 另请参阅
* [Azure HDInsight 入门](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/) - 了解如何开始使用 HDInsight 群集
* [以编程方式提交 Hadoop 作业](/documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/) - 了解如何以编程方式将作业提交到 HDInsight
* [使用 Azure CLI 管理 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-administer-use-command-line/)
* [将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 服务管理配合使用](/documentation/articles/virtual-machines-command-line-tools/)

<!---HONumber=Mooncake_0120_2017-->