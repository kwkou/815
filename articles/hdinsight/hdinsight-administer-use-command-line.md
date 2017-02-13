<properties
    pageTitle="使用 Azure CLI 管理 Hadoop 群集 | Azure"
    description="如何使用 Azure CLI 管理 HDInsight 中的 Hadoop 群集"
    services="hdinsight"
    editor="cgronlun"
    manager="jhubbard"
    author="mumian"
    tags="azure-portal"
    documentationcenter="" />
<tags
    ms.assetid="4f26c79f-8540-44bd-a470-84722a9e4eca"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/15/2016"
    wacn.date="01/25/2017"
    ms.author="jgao" />

# 使用 Azure CLI 管理 HDInsight 中的 Hadoop 群集
[AZURE.INCLUDE [选择器](../../includes/hdinsight-portal-management-selector.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

了解如何使用 [Azure 命令行接口](/documentation/articles/xplat-cli-install/)管理 Azure HDInsight 中的 Hadoop 群集。Azure CLI 是以 Node.js 实现的。可以在支持 Node.js 的任意平台（包括 Windows、Mac 和 Linux）上使用它。

本文仅介绍如何将 Azure CLI 与 HDInsight 配合使用。有关如何使用 Azure CLI 的常规指南，请参阅[安装和配置 Azure CLI][azure-command-line-tools]。

## 先决条件
在开始阅读本文前，你必须具有：

* **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* **Azure CLI** - 有关安装和配置信息，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)。
* 使用以下命令**连接到 Azure**：
  
        azure login -e AzureChinaCloud
  
    有关使用公司或学校帐户进行身份验证的详细信息，请参阅[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)。
* 使用以下命令**切换到 Azure 资源管理器模式**：
  
        azure config mode arm

若要获得帮助，请使用 **-h** 开关。例如：

    azure hdinsight cluster create -h

## 创建群集
请参阅[使用 Azure CLI 在 HDInsight 中创建基于 Linux 的群集](/documentation/articles/hdinsight-hadoop-create-linux-clusters-azure-cli/)。

## 列出并显示群集详细信息
使用以下命令列出并显示群集详细信息：

    azure hdinsight cluster list
    azure hdinsight cluster show <Cluster Name>

![HDI.CLIListCluster][image-cli-clusterlisting]

## <a name="delete-clusters"></a> 删除群集
使用以下命令来删除群集：

    azure hdinsight cluster delete <Cluster Name>

还可以通过删除包含群集的资源组来删除群集。请注意，这将删除包括默认存储帐户的组中的所有资源。

    azure group delete <Resource Group Name>

## <a name="scale-clusters"></a> 缩放群集
若要更改 Hadoop 群集大小，请执行以下操作：

    azure hdinsight cluster resize [options] <clusterName> <Target Instance Count>

## <a name="enabledisable-http-access-for-a-cluster"></a> 启用/禁用对群集的 HTTP 访问
    azure hdinsight cluster enable-http-access [options] <Cluster Name> <userName> <password>
    azure hdinsight cluster disable-http-access [options] <Cluster Name>

## 启用/禁用对群集的 RDP 访问
      azure hdinsight cluster enable-rdp-access [options] <Cluster Name> <rdpUserName> <rdpPassword> <rdpExpiryDate>
      azure hdinsight cluster disable-rdp-access [options] <Cluster Name>

## 后续步骤
在本文中，已了解如何执行不同的 HDInsight 群集管理任务。要了解更多信息，请参阅下列文章：

* [使用 Azure 门户预览管理 HDInsight][hdinsight-admin-portal]
* [使用 Azure PowerShell 管理 HDInsight][hdinsight-admin-powershell]
* [Azure HDInsight 入门][hdinsight-get-started]
* [如何使用 Azure CLI][azure-command-line-tools]

[azure-command-line-tools]: /documentation/articles/xplat-cli-install/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account/
[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/

[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal/
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/

[image-cli-account-download-import]: ./media/hdinsight-administer-use-command-line/HDI.CLIAccountDownloadImport.png
[image-cli-clustercreation]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreation.png
[image-cli-clustercreation-config]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreationConfig.png
[image-cli-clusterlisting]: ./media/hdinsight-administer-use-command-line/HDI.CLIListClusters.png "列出并显示群集"

<!---HONumber=Mooncake_0120_2017-->