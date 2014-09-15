<properties linkid="manage-services-hdinsight-administer-hdinsight-using-powershell" urlDisplayName="HDInsight Administration" pageTitle="Administer HDInsight using PowerShell | Azure" metaKeywords="hdinsight, hdinsight administration, hdinsight administration azure" description="Learn how to perform administrative tasks for the HDInsight clusters using PowerShell." services="hdinsight" umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" title="Administer HDInsight using PowerShell" authors="bradsev" />

# 使用 PowerShell 管理 HDInsight

Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。在本文中，你将了解如何通过使用 Windows PowerShell 借助于本地 Azure PowerShell 控制台来管理 HDInsight 群集。有关 HDInsight PowerShell cmdlet 的列表，请参阅 [HDInsight cmdlet 参考][]。

**先决条件：**

在开始阅读本文前，你必须具有：

-   Azure 订阅。Azure 是基于订阅的平台。HDInsight PowerShell cmdlet 通过订阅执行任务。有关获取订阅的详细信息，请参阅[购买选项][]、[成员优惠][azure-member-offers] 或[免费试用][]。

-   配备 Azure PowerShell 的工作站。有关说明，请参阅[安装和配置 Azure PowerShell][]。

## 本文内容

-   [设置群集][]
-   [列出并显示群集][]
-   [删除群集][]
-   [授予/撤消 HTTP 服务访问权限][]
-   [提交 MapReduce 作业][]
-   [提交 Hive 作业][]
-   [将数据上载到 Blob 存储][]
-   [从 Blob 存储下载 MapReduce 输出数据][]

## 设置 HDInsight 群集

HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户和存储容器，然后才能创建 HDInsight 群集。

**创建 Azure 存储帐户**

在导入了 publishsettings 文件后，你可以使用以下命令创建一个存储帐户：

    # 创建 Azure 存储帐户
    $storageAccountName = "<StorageAcccountName>"
    $location = "<Microsoft data center>"           # 例如，“China East”

    New-AzureStorageAccount -StorageAccountName $storageAccountName -Location $location

> [WACOM.NOTE] 存储帐户必须与 HDInsight 群集位于同一数据中心。目前，只能在以下数据中心内设置 HDInsight 群集：

> -   亚洲东南部
> -   欧洲北部
> -   欧洲西部
> -   美国东部
> -   美国西部

有关使用管理门户创建 Azure 存储帐户的信息，请参阅[如何创建存储帐户][]。

如果你已有存储帐户但是不知道帐户名称和帐户密钥，可以使用以下命令来检索该信息：

    # 列出当前订阅的存储帐户
    Get-AzureStorageAccount
    # 列出存储帐户的密钥
    Get-AzureStorageKey <StorageAccountName>

有关使用管理门户获取信息的详细信息，请参阅[如何管理存储帐户][]中的*如何：查看、复制和重新生成存储访问密钥*一节。

**创建 Azure 存储容器**

PowerShell 不能在 HDInsight 设置过程中创建 Blob 容器。你可以使用以下脚本创建一个容器：

    $storageAccountName = "<StorageAccountName>"
    $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
    $containerName="<ContainerName>"

    # 创建存储上下文对象
    $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName 
    -StorageAccountKey $storageAccountKey  

    # 创建 Blob 存储容器
    New-AzureStorageContainer -Name $containerName -Context $destContext

**设置群集**

准备好存储帐户和 blob 容器后，你就可以创建群集了。

    $storageAccountName = "<StorageAccountName>"
    $containerName = "<ContainerName>"

    $clusterName = "<HDInsightClusterName>"
    $location = "<MicrosoftDataCenter>"
    $clusterNodes = <ClusterSizeInNodes>

    # 获取存储帐户密钥
    $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }

    # 新建 HDInsight 群集
    New-AzureHDInsightCluster -Name $clusterName -Location $location -DefaultStorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainerName $containerName  -ClusterSizeInNodes $clusterNodes

下面的屏幕快照显示脚本执行：

![HDI.PS.Provision][]

## 列出并显示群集详细信息

使用以下命令来列出和显示群集详细信息：

**列出当前订阅中的所有群集**

    Get-AzureHDInsightCluster 

**显示当前订阅中特定群集的详细信息**

    Get-AzureHDInsightCluster -Name <ClusterName> 

## 删除群集

使用以下命令来删除群集：

    Remove-AzureHDInsightCluster -Name <ClusterName> 

## 授予/撤消 HTTP 服务访问权限

HDInsight 群集提供以下 HTTP Web 服务（所有这些服务都有 REST 样式的终结点）：

-   ODBC
-   JDBC
-   Ambari
-   Oozie
-   Templeton

默认情况下，将授权这些服务进行访问。你可以撤消/授予访问权限。下面是一个示例：

    Revoke-AzureHDInsightHttpServicesAccess -Name hdiv2 -Location "China East"

在此示例中，*hdiv2* 是 HDInsight 群集名称。

> [WACOM.NOTE] 授予/撤消访问权限时，你将重设群集用户的用户名和密码。

也可以使用 Windows Azure 管理门户完成此操作。请参阅[使用管理门户管理 HDInsight][]。

## 提交 MapReduce 作业

HDInsight 群集分发附带一些 MapReduce 示例。其中一个示例是计算源文件中的单词频率。

**提交 MapReduce 作业**

下面的 PowerShell 脚本提交单词计数示例作业：

    $clusterName = "<HDInsightClusterName>"            

    # 定义 MapReduce 作业
    $wordCountJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-examples.jar" -ClassName "wordcount" -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data/WordCountOutput"

    # 运行作业并显示标准错误 
    $wordCountJobDefinition | Start-AzureHDInsightJob -Cluster $clusterName | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | %{ Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $_.JobId -StandardError}

> [WACOM.NOTE] *hadoop-examples.jar* 随版本 2.1 HDInsight 群集提供。该文件在版本 3.0 HDInsight 群集上已被重命名为 *hadoop-mapreduce.jar*。

有关 WASB 前缀的信息，请参阅 [将 Azure Blob 存储用于 HDInsight][hdinsight-
storage]。

**下载 MapReduce 作业输出**

下面的 PowerShell 脚本从上一个过程中检索 MapReduce 作业输出：

    $storageAccountName = "<StorageAccountName>"   
    $containerName = "<ContainerName>"             
        
    # 创建存储帐户上下文对象
    $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
    $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    # 将输出下载到本地计算机
    Get-AzureStorageBlobContent -Container $ContainerName -Blob example/data/WordCountOutput/part-r-00000 -Context $storageContext -Force

    # 显示输出
    cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"

有关开发和运行 MapReduce 作业的详细信息，请参阅[将 MapReduce 与 HDInsight 配合使用][]。

## 提交 Hive 作业

HDInsight 群集分发附带称作 *hivesampletable* 的示例 Hive 表。你可以使用 HiveQL“show tables;”列出群集上的 Hive 表。

**提交 Hive 作业**

下面的脚本提交一个 hive 作业以便列出 Hive 表：

    $clusterName = "<HDInsightClusterName>"               

    # HiveQL 查询
    $querystring = @"
    SHOW TABLES;
    SELECT * FROM hivesampletable 
    WHERE Country='United Kingdom'
    LIMIT 10;
    "@

    Use-AzureHDInsightCluster -Name $clusterName
    Invoke-Hive $querystring

该 Hive 作业将首先显示在该群集上创建的 Hive 表，并且显示从 hivesampletable 返回的数据。

有关使用 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用][]。

## 将数据上载到 Blob 存储

请参阅[将数据上载到 HDInsight][]。

## 从 Blob 存储下载 MapReduce 输出

请参阅本文中的[提交 MapReduce 作业][]一节。

## 另请参阅

-   [HDInsight Cmdlet 参考文档][HDInsight cmdlet 参考]
-   [使用管理门户管理 HDInsight][]
-   [使用命令行接口管理 HDInsight][]
-   [配置 HDInsight 群集][]
-   [将数据上载到 HDInsight][]
-   [以编程方式提交 Hadoop 作业][]
-   [Azure HDInsight 入门][]

  [HDInsight cmdlet 参考]: http://msdn.microsoft.com/zh-cn/library/azure/dn479228.aspx
  [购买选项]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [免费试用]: https://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [安装和配置 Azure PowerShell]: /en-us/documentation/articles/install-configure-powershell/
  [设置群集]: #provision
  [列出并显示群集]: #listshow
  [删除群集]: #delete
  [授予/撤消 HTTP 服务访问权限]: #httpservices
  [提交 MapReduce 作业]: #mapreduce
  [提交 Hive 作业]: #hive
  [将数据上载到 Blob 存储]: #upload
  [从 Blob 存储下载 MapReduce 输出数据]: #download
  [如何创建存储帐户]: /en-us/manage/services/storage/how-to-create-a-storage-account/
  [如何管理存储帐户]: /en-us/manage/services/storage/how-to-manage-a-storage-account/
  [HDI.PS.Provision]: ./media/hdinsight-administer-use-powershell/HDI.PS.Provision.png
  [使用管理门户管理 HDInsight]: /en-us/documentation/articles/hdinsight-administer-use-management-portal/
  [将 MapReduce 与 HDInsight 配合使用]: /en-us/documentation/articles/hdinsight-use-mapreduce/
  [将 Hive 与 HDInsight 配合使用]: /en-us/documentation/articles/hdinsight-use-hive/
  [将数据上载到 HDInsight]: /en-us/documentation/articles/hdinsight-upload-data/
  [使用命令行接口管理 HDInsight]: /en-us/documentation/articles/hdinsight-administer-use-command-line/
  [配置 HDInsight 群集]: /en-us/documentation/articles/hdinsight-provision-clusters/
  [以编程方式提交 Hadoop 作业]: /en-us/documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
  [Azure HDInsight 入门]: /en-us/documentation/articles/hdinsight-get-started/
