<properties
   pageTitle="使用 Azure PowerShell 在 HDInsight 中创建基于 Windows 的 Hadoop 群集 | Azure"
   	description="了解如何使用 Azure PowerShell 创建 Azure HDInsight 的群集。"
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

# 使用 Azure PowerShell 在 HDInsight 中创建基于 Windows 的 Hadoop 群集

[AZURE.INCLUDE [选择器](../includes/hdinsight-selector-create-clusters.md)]

了解如何使用 Azure PowerShell 创建 HDInsight 群集。Azure PowerShell 是一个模块，提供用于通过 Windows PowerShell 管理 Azure 的 cmdlet。有关其他群集创建工具和功能，请单击本页面顶部的相应选项卡，或参阅[群集创建方法](/documentation/articles/hdinsight-provision-clusters-v1/#cluster-creation-methods)。


###先决条件：

[AZURE.INCLUDE [delete-cluster-warning](../includes/hdinsight-delete-cluster-warning.md)]

在开始按照本文中的说明操作之前，你必须具有以下内容：

- Azure 订阅创建新存储帐户。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
- Azure PowerShell。参阅[安装 Azure PowerShell 1.0](/documentation/articles/hdinsight-administer-use-powershell/#install-azure-powershell-10-and-greater)。



## 创建群集
Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。本部分提供有关如何通过使用 Azure PowerShell 创建 HDInsight 群集的说明。有关配置工作站以运行 HDInsight Windows Powershell cmdlet 的信息，请参阅[安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。有关将 Azure PowerShell 与 HDInsight 配合使用的详细信息，请参阅[使用 PowerShell 管理 HDInsight](/documentation/articles/hdinsight-administer-use-powershell/)。有关 HDInsight Windows PowerShell cmdlet 的列表，请参阅 [HDInsight cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn858087.aspx)。


通过使用 Azure PowerShell 创建 HDInsight 群集需要执行以下过程：

    ####################################
    # Set these variables
    ####################################
    #region - used for creating Azure service names
    $nameToken = "<Enter an Alias>" 
    #endregion

    #region - cluster user accounts
    $httpUserName = "admin"  #HDInsight cluster username
    $httpPassword = "<Enter a Password>"
    #endregion

    ###########################################
    # Service names and varialbes
    ###########################################
    #region - service names
    $namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")

    $hdinsightClusterName = $namePrefix + "hdi"
    $defaultStorageAccountName = $namePrefix + "store"
    $defaultBlobContainerName = $hdinsightClusterName

    $location = "China East"
    $clusterSizeInNodes = 1
    #endregion

    # Treat all errors as terminating
    $ErrorActionPreference = "Stop"

    ###########################################
    # Connect to Azure
    ###########################################
    #region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
    try{Get-AzureContext}
    catch{Add-AzureAccount -Environment AzureChinaCloud}
    #endregion

    ###########################################
    # Preapre default storage account and container
    ###########################################
    New-AzureStorageAccount `
        -StorageAccountName $defaultStorageAccountName `
        -Type Standard_GRS `
        -Location $location

    $defaultStorageAccountKey = Get-AzureStorageKey `
                                    -StorageAccountName $defaultStorageAccountName |  %{ $_.Primary }
    $defaultStorageContext = New-AzureStorageContext `
                                    -StorageAccountName $defaultStorageAccountName `
                                    -StorageAccountKey $defaultStorageAccountKey
    New-AzureStorageContainer `
        -Name $hdinsightClusterName -Context $defaultStorageContext

    ###########################################
    # Create the cluster
    ###########################################

    $httpPW = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
    $httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$httpPW)

    New-AzureHDInsightCluster `
        -Name $hdinsightClusterName `
        -Location $location `
        -ClusterSizeInNodes $clusterSizeInNodes `
        -ClusterType Hadoop `
        -Version "3.2" `
        -Credential $httpCredential `
        -DefaultStorageAccountName "$defaultStorageAccountName.blob.core.chinacloudapi.cn" `
        -DefaultStorageAccountKey $defaultStorageAccountKey `
        -DefaultStorageContainerName $hdinsightClusterName 

    ####################################
    # Verify the cluster
    ####################################
    Get-AzureHDInsightCluster -Name $hdinsightClusterName 

##自定义群集

- 参阅[使用 Bootstrap 自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-bootstrap/#use-azure-powershell)。
- 参阅[使用脚本操作自定义基于 Windows 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-v1/#call-scripts-using-azure-powershell)。


##后续步骤
在本文中，你已经学习了几种创建 HDInsight 群集的方法。若要了解更多信息，请参阅下列文章：

* [Azure HDInsight 入门](/documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/) - 了解如何开始使用你的 HDInsight 群集
* [以编程方式提交 Hadoop 作业](/documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/) - 了解如何以编程方式将作业提交到 HDInsight
* [使用 PowerShell 管理 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-administer-use-powershell/) - 了解如何通过 Azure PowerShell 使用 HDInsight
* [Azure HDInsight SDK 文档][hdinsight-sdk-documentation] - 探索 HDInsight SDK




[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx
[azure-preview-portal]: https://manage.windowsazure.cn
[connectionmanager]: http://msdn.microsoft.com/zh-cn/library/mt146773(v=sql.120).aspx
[ssispack]: http://msdn.microsoft.com/zh-cn/library/mt146770(v=sql.120).aspx
[ssisclustercreate]: http://msdn.microsoft.com/zh-cn/library/mt146774(v=sql.120).aspx
[ssisclusterdelete]: http://msdn.microsoft.com/zh-cn/library/mt146778(v=sql.120).aspx

<!---HONumber=Mooncake_0215_2016-->