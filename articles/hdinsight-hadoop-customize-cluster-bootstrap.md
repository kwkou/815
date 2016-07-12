<properties
	pageTitle="使用 Bootstrap 自定义 HDInsight 群集 | Azure"
	description="了解如何使用 Bootstrap 自定义 HDInsight 群集。"
	services="hdinsight"
	documentationCenter=""
	authors="mumian"
	manager="paulettm"
	editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="03/08/2016"
	wacn.date="05/24/2016"/>

# 使用 Bootstrap 自定义 HDInsight 群集

有时，你可能需要配置配置文件，包括：

- clusterIdentity.xml
- core-site.xml
- gateway.xml
- hbase-env.xml
- hbase-site.xml
- hdfs-site.xml
- hive-env.xml
- hive-site.xml
- mapred-site
- oozie-site.xml
- oozie-env.xml
- storm-site.xml
- tez-site.xml
- webhcat-site.xml
- yarn-site.xml

群集无法保留重置映像所造成的更改。有关重置映像的详细信息，请参阅[由于操作系统升级而重新启动角色实例](http://blogs.msdn.com/b/kwill/archive/2012/09/19/role-instance-restarts-due-to-os-upgrades.aspx)。若要在群集生存期保留更改，你可以在创建过程中使用 HDInsight 群集自定义。这是更改群集的配置，以及发生这些 Azure 重置映像、重新引导和重新启动事件后保留配置的推荐方法。这些配置更改将在服务启动之前应用，因此无需重新启动服务。

Bootstrap 的使用方式有 2 种：

- 使用 Azure PowerShell

	[AZURE.INCLUDE [upgrade-powershell](../includes/hdinsight-use-latest-powershell.md)]

- 使用 .NET SDK

有关在创建时在 HDInsight 群集上安装其他组件的信息，请参阅：

- [使用脚本操作自定义 HDInsight 群集 (Windows)](/documentation/articles/hdinsight-hadoop-customize-cluster-v1/)

##<a name="use-azure-powershell"></a> 使用 Azure PowerShell

以下 PowerShell 代码将自定义 Hive 配置：

	# hive-site.xml configuration
	$hiveConfigValues = @{ "hive.metastore.client.socket.timeout"="90" }
	
	$config = New-AzureHDInsightClusterConfig `
		| Set-AzureHDInsightDefaultStorage `
			-StorageAccountName "$defaultStorageAccountName.blob.core.chinacloudapi.cn" `
			-StorageAccountKey $defaultStorageAccountKey `
		| Add-AzureHDInsightConfigValues `
			-Hive $hiveConfigValues 
	
	New-AzureHDInsightCluster `
		-Name $clusterName `
		-Location $location `
		-ClusterSizeInNodes $clusterSizeInNodes `
		-ClusterType Hadoop `
		-Version "3.2" `
		-Credential $httpCredential `
		-Config $config 

可在[附录 A](#appx-a:-powershell-sample) 中找到完整的有效 PowerShell 脚本。

下面是有关自定义其他配置文件的更多示例：

	# hdfs-site.xml configuration
	$HdfsConfigValues = @{ "dfs.blocksize"="64m" } #default is 128MB in HDI 3.0 and 256MB in HDI 2.1

	# core-site.xml configuration
	$CoreConfigValues = @{ "ipc.client.connect.max.retries"="60" } #default 50

	# mapred-site.xml configuration
	$MapRedConfigValues = @{ "mapreduce.task.timeout"="1200000" } #default 600000

	# oozie-site.xml configuration
	$OozieConfigValues = @{ "oozie.service.coord.normal.default.timeout"="150" }  # default 120

有关详细信息，请参阅 Azim Uddin 的标题为[自定义 HDInsight 群集创建](http://blogs.msdn.com/b/bigdatasupport/archive/2014/04/15/customizing-hdinsight-cluster-provisioning-via-powershell-and-net-sdk.aspx)的博客。

## 另请参阅

- [在 HDInsight 中创建 Hadoop 群集][hdinsight-provision-cluster]说明了如何使用其他自定义选项来创建 HDInsight 群集。
- [为 HDInsight 开发脚本操作脚本][hdinsight-write-script]
- [在 HDInsight 群集上安装并使用 R][hdinsight-install-r]
- [在 HDInsight 群集上安装并使用 Solr](/documentation/articles/hdinsight-hadoop-solr-install-v1/)
- [在 HDInsight 群集上安装并使用 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install-v1/)

[hdinsight-install-r]: /documentation/articles/hdinsight-hadoop-r-scripts/
[hdinsight-write-script]: /documentation/articles/hdinsight-hadoop-script-actions/
[hdinsight-provision-cluster]: /documentation/articles/hdinsight-provision-clusters-v1/
[powershell-install-configure]: /documentation/articles/install-configure-powershell/


[img-hdi-cluster-states]: ./media/hdinsight-hadoop-customize-cluster-v1/HDI-Cluster-state.png "群集创建过程中的阶段"

##<a name="appx-a:-powershell-sample"></a> 附录 A：PowerShell 示例

此 PowerShell 脚本将创建一个 HDInsight 群集并自定义 Hive 设置：

    ####################################
    # Set these variables
    ####################################
    #region - used for creating Azure service names
    $nameToken = "<ENTER AN ALIAS>" 
    #endregion

    #region - cluster user accounts
    $httpUserName = "admin"  #HDInsight cluster username
    $httpPassword = "<ENTER A PASSWORD>" #"<Enter a Password>"

    $sshUserName = "sshuser" #HDInsight ssh user name
    $sshPassword = "<ENTER A PASSWORD>" #"<Enter a Password>"
    #endregion

    ####################################
    # Service names and varialbes
    ####################################
    #region - service names
    $namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")

    $resourceGroupName = $namePrefix + "rg"
    $hdinsightClusterName = $namePrefix + "hdi"
    $defaultStorageAccountName = $namePrefix + "store"
    $defaultBlobContainerName = $hdinsightClusterName

    $location = "China East 2"
    #endregion

    # Treat all errors as terminating
    $ErrorActionPreference = "Stop"

    ####################################
    # Connect to Azure
    ####################################
    #region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
    try{Get-AzureContext}
    catch{Add-AzureAccount -Environment AzureChinaCloud}
    #endregion

    Write-Host "Creating the default storage account and default blob container ..."  -ForegroundColor Green
    New-AzureStorageAccount `
        -StorageAccountName $defaultStorageAccountName `
        -Location $location `
        -Type Standard_GRS

    $defaultStorageAccountKey = Get-AzureStorageAccountKey `
                                    -StorageAccountName $defaultStorageAccountName |  %{ $_.Primary }
    $defaultStorageContext = New-AzureStorageContext `
                                    -StorageAccountName $defaultStorageAccountName `
                                    -StorageAccountKey $defaultStorageAccountKey
    New-AzureStorageContainer `
        -Name $defaultBlobContainerName `
        -Context $defaultStorageContext #use the cluster name as the container name

    ####################################
    # Create a configuration object
    ####################################
    $hiveConfigValues = @{ "hive.metastore.client.socket.timeout"="90" }
        
    $config = New-AzureHDInsightClusterConfig `
        | Set-AzureHDInsightDefaultStorage `
            -StorageAccountName "$defaultStorageAccountName.blob.core.chinacloudapi.cn" `
            -StorageAccountKey $defaultStorageAccountKey `
        | Add-AzureHDInsightConfigValues `
            -Hive $hiveConfigValues 

    ####################################
    # Create an HDInsight cluster
    ####################################
    $httpPW = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
    $httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$httpPW)

    $sshPW = ConvertTo-SecureString -String $sshPassword -AsPlainText -Force
    $sshCredential = New-Object System.Management.Automation.PSCredential($sshUserName,$sshPW)

    New-AzureHDInsightCluster `
        -Name $hdinsightClusterName `
        -Location $location `
        -ClusterSizeInNodes 1 `
        -ClusterType Hadoop `
        -Version "3.2" `
        -Credential $httpCredential `
        -Config $config

    ####################################
    # Verify the cluster
    ####################################
    Get-AzureHDInsightCluster -Name $hdinsightClusterName

    #endregion

<!---HONumber=Mooncake_0215_2016-->