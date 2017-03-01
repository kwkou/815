<properties
    pageTitle="使用 bootstrap 自定义 HDInsight 群集 | Azure"
    description="了解如何使用 Bootstrap 自定义 HDInsight 群集。"
    services="hdinsight"
    documentationcenter=""
    author="mumian"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="ab2ebf0c-e961-4e95-8151-9724ee22d769"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="09/02/2016"
    wacn.date="01/25/2017"
    ms.author="jgao" />

# 使用 Bootstrap 自定义 HDInsight 群集

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

有时，你可能需要配置配置文件，包括：

* clusterIdentity.xml
* core-site.xml
* gateway.xml
* hbase-env.xml
* hbase-site.xml
* hdfs-site.xml
* hive-env.xml
* hive-site.xml
* mapred-site
* oozie-site.xml
* oozie-env.xml
* storm-site.xml
* tez-site.xml
* webhcat-site.xml
* yarn-site.xml

群集无法保留重置映像所造成的更改。有关重置映像的详细信息，请参阅[由于操作系统升级而重新启动角色实例](http://blogs.msdn.com/b/kwill/archive/2012/09/19/role-instance-restarts-due-to-os-upgrades.aspx)。若要在群集生存期保留更改，你可以在创建过程中使用 HDInsight 群集自定义。这是更改群集的配置，以及发生这些 Azure 重置映像、重新引导和重新启动事件后保留配置的推荐方法。这些配置更改将在服务启动之前应用，因此无需重新启动服务。

Bootstrap 的使用方式有 3 种：

* 使用 Azure PowerShell
  
    [AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]
* 使用 .NET SDK
* 使用 Azure Resource Manager 模板

有关在创建时在 HDInsight 群集上安装其他组件的信息，请参阅：

* [使用脚本操作 (Linux) 自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)
* [使用脚本操作自定义 HDInsight 群集 (Windows)](/documentation/articles/hdinsight-hadoop-customize-cluster/)

## <a name="use-azure-powershell"></a> 使用 Azure PowerShell
以下 PowerShell 代码将自定义 Hive 配置：

    # hive-site.xml configuration
    $hiveConfigValues = @{ "hive.metastore.client.socket.timeout"="90" }

    $config = New-AzureRmHDInsightClusterConfig `
        | Set-AzureRmHDInsightDefaultStorage `
            -StorageAccountName "$defaultStorageAccountName.blob.core.chinacloudapi.cn" `
            -StorageAccountKey $defaultStorageAccountKey `
        | Add-AzureRmHDInsightConfigValues `
            -HiveSite $hiveConfigValues 

    New-AzureRmHDInsightCluster `
        -ResourceGroupName $existingResourceGroupName `
        -ClusterName $clusterName `
        -Location $location `
        -ClusterSizeInNodes $clusterSizeInNodes `
        -ClusterType Hadoop `
        -OSType Windows `
        -Version "3.2" `
        -HttpCredential $httpCredential `
        -Config $config 

可在[附录 A](#appx-a:-powershell-sample) 中找到完整的有效 PowerShell 脚本。

**若要验证更改，请执行以下操作：**

1. 登录到 [Azure 门户预览](https://portal.azure.cn)。
2. 在左窗格中，单击“浏览”，然后单击“HDInsight 群集”。
3. 单击刚刚使用 PowerShell 脚本创建的群集。
4. 单击边栏选项卡顶部的“仪表板”打开 Ambari UI。
5. 在左侧菜单中，单击“Hive”。
6. 单击“摘要”中的 **HiveServer2**。
7. 单击“配置”选项卡。
8. 在左侧菜单中，单击“Hive”。
9. 单击“高级”选项卡。
10. 向下滚动，然后展开“高级 hive 站点”。
11. 在此部分中查找 **hive.metastore.client.socket.timeout**。

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

## 使用 .NET SDK
请参阅[使用 .NET SDK 在 HDInsight 中创建基于 Linux 的群集](/documentation/articles/hdinsight-hadoop-create-linux-clusters-dotnet-sdk/#use-bootstrap)。

## 使用 Resource Manager 模板
可以在 Resource Manager 模板中使用 bootstrap：

    "configurations": {
        …
        "hive-site": {
            "hive.metastore.client.connect.retry.delay": "5",
            "hive.execution.engine": "mr",
            "hive.security.authorization.manager": "org.apache.hadoop.hive.ql.security.authorization.DefaultHiveAuthorizationProvider"
        }
    }

![hdinsight hadoop, 自定义群集, bootstrap, azure resource manager 模板](./media/hdinsight-hadoop-customize-cluster-bootstrap/hdinsight-customize-cluster-bootstrap-arm.png)  


## 另请参阅
* [在 HDInsight 中创建 Hadoop 群集][hdinsight-provision-cluster]说明了如何使用其他自定义选项创建 HDInsight 群集。
* [为 HDInsight 开发脚本操作脚本][hdinsight-write-script]
* [在 HDInsight 群集上安装并使用 Spark][hdinsight-install-spark]
* [在 HDInsight 群集上安装并使用 R][hdinsight-install-r]
* [在 HDInsight 群集上安装并使用 Solr](/documentation/articles/hdinsight-hadoop-solr-install/)
* [在 HDInsight 群集上安装并使用 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install/)

[hdinsight-install-spark]: /documentation/articles/hdinsight-hadoop-spark-install/
[hdinsight-install-r]: /documentation/articles/hdinsight-hadoop-r-scripts/
[hdinsight-write-script]: /documentation/articles/hdinsight-hadoop-script-actions/
[hdinsight-provision-cluster]: /documentation/articles/hdinsight-provision-clusters/
[powershell-install-configure]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs

[img-hdi-cluster-states]: ./media/hdinsight-hadoop-customize-cluster/HDI-Cluster-state.png "群集创建期间的阶段"

## <a name="appx-a:-powershell-sample"></a> 附录 A：PowerShell 示例
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

    $location = "China East"
    #endregion

    # Treat all errors as terminating
    $ErrorActionPreference = "Stop"

    ####################################
    # Connect to Azure
    ####################################
    #region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
    try{Get-AzureRmContext}
    catch{Login-AzureRmAccount -EnvironmentName AzureChinaCloud}
    #endregion

    #region - Create an HDInsight cluster
    ####################################
    # Create dependent components
    ####################################
    Write-Host "Creating a resource group ..." -ForegroundColor Green
    New-AzureRmResourceGroup `
        -Name  $resourceGroupName `
        -Location $location

    Write-Host "Creating the default storage account and default blob container ..."  -ForegroundColor Green
    New-AzureRmStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $defaultStorageAccountName `
        -Location $location `
        -Type Standard_GRS

    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey `
                                    -ResourceGroupName $resourceGroupName `
                                    -Name $defaultStorageAccountName)[0].Value
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

    $config = New-AzureRmHDInsightClusterConfig `
        | Set-AzureRmHDInsightDefaultStorage `
            -StorageAccountName "$defaultStorageAccountName.blob.core.chinacloudapi.cn" `
            -StorageAccountKey $defaultStorageAccountKey `
        | Add-AzureRmHDInsightConfigValues `
            -HiveSite $hiveConfigValues 

    ####################################
    # Create an HDInsight cluster
    ####################################
    $httpPW = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
    $httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$httpPW)

    $sshPW = ConvertTo-SecureString -String $sshPassword -AsPlainText -Force
    $sshCredential = New-Object System.Management.Automation.PSCredential($sshUserName,$sshPW)

    New-AzureRmHDInsightCluster `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $hdinsightClusterName `
        -Location $location `
        -ClusterSizeInNodes 1 `
        -ClusterType Hadoop `
        -OSType Linux `
        -Version "3.5" `
        -HttpCredential $httpCredential `
        -SshCredential $sshCredential `
        -Config $config

    ####################################
    # Verify the cluster
    ####################################
    Get-AzureRmHDInsightCluster -ClusterName $hdinsightClusterName

    #endregion

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->