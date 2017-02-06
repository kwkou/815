<properties
    pageTitle="使用 PowerShell 管理 HDInsight 中的 Hadoop 群集 | Azure"
    description="了解如何使用 Azure PowerShell 对 HDInsight 中的 Hadoop 群集执行管理任务。"
    services="hdinsight"
    editor="cgronlun"
    manager="jhubbard"
    tags="azure-portal"
    author="mumian"
    documentationcenter="" />
<tags
    ms.assetid="bfdfa754-18e5-4ef9-b0d6-2dbdcebc0283"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/15/2016"
    wacn.date="01/25/2017"
    ms.author="jgao" />

# 使用 Azure PowerShell 管理 HDInsight 中的 Hadoop 群集
[AZURE.INCLUDE [选择器](../../includes/hdinsight-portal-management-selector.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。在本文中，你将要学习如何通过使用 Windows PowerShell 借助于本地 Azure PowerShell 控制台来管理 Azure HDInsight 中的 Hadoop 群集。有关 HDInsight PowerShell cmdlet 的列表，请参阅 [HDInsight cmdlet 参考][hdinsight-powershell-reference]。

**先决条件**

在开始本文前，你必须具有以下项：

* **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。

## <a id="install-azure-powershell-10-and-greater"></a> 安装 Azure PowerShell
[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

如果已安装 Azure PowerShell 版本 0.9x，则必须卸载它然后再安装更新版本。

若要检查所安装的 PowerShell 版本，请执行以下操作：

    Get-Module *azure*

若要卸载旧版本，请运行控制面板中的“程序和功能”。

## 创建群集
请参阅[使用 Azure PowerShell 在 HDInsight 中创建基于 Linux 的群集](/documentation/articles/hdinsight-hadoop-create-linux-clusters-azure-powershell/)

## 列出群集
使用以下命令可列出当前订阅中的所有群集：

    Get-AzureRmHDInsightCluster

## 显示群集
使用以下命令可显示当前订阅中特定群集的详细信息：

    Get-AzureRmHDInsightCluster -ClusterName <Cluster Name>

## <a name="delete-clusters"></a> 删除群集
使用以下命令来删除群集：

    Remove-AzureRmHDInsightCluster -ClusterName <Cluster Name>

还可以通过删除包含群集的资源组来删除群集。请注意，这将删除包括默认存储帐户的组中的所有资源。

    Remove-AzureRmResourceGroup -Name <Resource Group Name>

## 缩放群集
使用群集缩放功能，可更改 Azure HDInsight 中运行的群集使用的辅助节点数，而无需重新创建群集。

> [AZURE.NOTE]
只支持使用 HDInsight 3.1.3 或更高版本的群集。如果不确定群集的版本，可以查看“属性”页面。请参阅[列出并显示群集](/documentation/articles/hdinsight-administer-use-portal-linux/#list-and-show-clusters)。
> 
> 

更改 HDInsight 支持的每种类型的群集所用数据节点数的影响：

* Hadoop
  
    可顺利增加正在运行的 Hadoop 群集中的辅助节点数，而不会影响任何挂起或运行中的作业。也可在操作进行中提交新作业。系统会正常处理失败的缩放操作，让群集始终保持正常运行状态。
  
    减少数据节点数目以缩减 Hadoop 群集时，系统会重新启动群集中的某些服务。这会导致所有正在运行和挂起的作业在缩放操作完成时失败。但是，可在操作完成后重新提交这些作业。
* HBase
  
    可在 HBase 群集运行时顺利添加或删除节点。完成缩放操作后的几分钟内，区域服务器将自动平衡。但也可手动平衡区域服务器，方法是登录到群集的头节点，然后在命令提示符窗口中运行以下命令：
  
        >pushd %HBASE_HOME%\bin
        >hbase shell
        >balancer
* Storm
  
    可在 Storm 群集运行时顺利添加或删除数据节点。但是，缩放操作成功完成后，需要重新平衡拓扑。
  
    可以使用两种方法来完成重新平衡操作：
  
  * Storm Web UI
  * 命令行界面 (CLI) 工具
    
    有关更多详细信息，请参阅 [Apache Storm 文档](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html)。
    
    HDInsight 群集上提供了 Storm Web UI：
    
    ![HDInsight storm 缩放重新平衡](./media/hdinsight-administer-use-management-portal/hdinsight.portal.scale.cluster.storm.rebalance.png)
    
    以下是有关如何使用 CLI 命令重新平衡 Storm 拓扑的示例：
    
        ## Reconfigure the topology "mytopology" to use 5 worker processes,
        ## the spout "blue-spout" to use 3 executors, and
        ## the bolt "yellow-bolt" to use 10 executors
        $ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10

若要使用 Azure PowerShell 更改 Hadoop 群集大小，请从客户端计算机运行以下命令：

    Set-AzureRmHDInsightClusterSize -ClusterName <Cluster Name> -TargetInstanceCount <NewSize>

## <a name="grant/revoke-access" id="grantrevoke-access"></a> 授予/撤消访问权限
HDInsight 群集提供以下 HTTP Web 服务（所有这些服务都有 REST 样式的终结点）：

* ODBC
* JDBC
* Ambari
* Oozie
* Templeton

默认情况下，将授权这些服务进行访问。你可以撤消/授予访问权限。若要撤消：

    Revoke-AzureRmHDInsightHttpServicesAccess -ClusterName <Cluster Name>

若要授予：

    $clusterName = "<HDInsight Cluster Name>"

    # Credential option 1
    $hadoopUserName = "admin"
    $hadoopUserPassword = "<Enter the Password>"
    $hadoopUserPW = ConvertTo-SecureString -String $hadoopUserPassword -AsPlainText -Force
    $credential = New-Object System.Management.Automation.PSCredential($hadoopUserName,$hadoopUserPW)

    # Credential option 2
    #$credential = Get-Credential -Message "Enter the HTTP username and password:" -UserName "admin"

    Grant-AzureRmHDInsightHttpServicesAccess -ClusterName $clusterName -HttpCredential $credential

> [AZURE.NOTE]
通过授予/撤消访问权限，你将重置群集用户名和密码。
> 
> 

也可以通过门户完成此操作。请参阅[使用 Azure 门户预览管理 HDInsight][hdinsight-admin-portal]。

## 更新 HTTP 用户凭据
这与[授予/撤消 HTTP 访问权限](#grant/revoke-access)是同一过程。如果已授予群集 HTTP 访问权限，必须先撤消该访问权限。然后再使用新的 HTTP 用户凭据授予访问权限。

## 查找默认存储帐户
以下 Powershell 脚本演示如何获取群集的默认存储帐户名称和默认存储帐户密钥。

    $clusterName = "<HDInsight Cluster Name>"

    $cluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroupName = $cluster.ResourceGroup
    $defaultStorageAccountName = ($cluster.DefaultStorageAccount).Replace(".blob.core.chinacloudapi.cn", "")
    $defaultBlobContainerName = $cluster.DefaultStorageContainer
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value
    $defaultStorageAccountContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey 

## 查找资源组
在 Resource Manager 模式下，每个 HDInsight 群集都属于一个 Azure 资源组。若要查找资源组：

    $clusterName = "<HDInsight Cluster Name>"

    $cluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroupName = $cluster.ResourceGroup

## 提交作业
**提交 MapReduce 作业**

请参阅[在基于 Windows 的 HDInsight 中运行 Hadoop MapReduce 示例](/documentation/articles/hdinsight-run-samples/)。

**提交 Hive 作业**

请参阅[使用 PowerShell 运行 Hive 查询](/documentation/articles/hdinsight-hadoop-use-hive-powershell/)

**提交 Pig 作业**

请参阅[使用 PowerShell 运行 Pig 作业](/documentation/articles/hdinsight-hadoop-use-pig-powershell/)。

**提交 Sqoop 作业**

请参阅[将 Sqoop 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-sqoop/)。

**提交 Oozie 作业**

请参阅[在 HDInsight 中将 Oozie 与 Hadoop 配合使用以定义和运行工作流](/documentation/articles/hdinsight-use-oozie/)。

## 将数据上传到 Azure Blob 存储
请参阅[将数据上传到 HDInsight][hdinsight-upload-data]。

## 另请参阅
* [HDInsight cmdlet 参考文档][hdinsight-powershell-reference]
* [使用 Azure 门户预览管理 HDInsight][hdinsight-admin-portal]
* [使用命令行借口管理 HDInsight][hdinsight-admin-cli]
* [创建 HDInsight 群集][hdinsight-provision]
* [将数据上传到 HDInsight][hdinsight-upload-data]
* [以编程方式提交 Hadoop 作业][hdinsight-submit-jobs]
* [Azure HDInsight 入门][hdinsight-get-started]

[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/

[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-provision-custom-options]: /documentation/articles/hdinsight-provision-clusters/#configuration
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/

[hdinsight-admin-cli]: /documentation/articles/hdinsight-administer-use-command-line/
[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-mapreduce]: /documentation/articles/hdinsight-use-mapreduce/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-flight]: /documentation/articles/hdinsight-analyze-flight-delay-data/

[hdinsight-powershell-reference]: https://msdn.microsoft.com/zh-cn/library/dn858087.aspx

[powershell-install-configure]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs

[image-hdi-ps-provision]: ./media/hdinsight-administer-use-powershell/HDI.PS.Provision.png

<!---HONumber=Mooncake_0120_2017-->