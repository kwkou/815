<properties linkid="manage-services-hdinsight-howto-blob-store" urlDisplayName="Blob Storage with HDInsight" pageTitle="Use Blob storage with HDInsight | Azure" metaKeywords="" description="Learn how HDInsight uses Blob storage as the underlying data store for HDFS and how you can query data from the store." metaCanonical="" services="storage,hdinsight" documentationCenter="" title="Use Azure Blob storage with HDInsight" authors="jgao" solutions="" manager="paulettm" editor="mollybos" />

# 将 Azure Blob 存储与 HDInsight 配合使用

Azure HDInsight 支持使用 Hadoop 分布式文件系统 (HDFS) 和 Azure Blob 存储来存储数据。Blob 存储是一个可靠的通用 Azure 存储解决方案。Blob 存储提供了一个功能全面的 HDFS 接口，该接口通过支持 Hadoop 生态系统中的整套组件直接操作（默认情况下）数据来为客户提供无缝体验。Blob 存储不只是一个低成本解决方案，通过将数据存储在 Blob 存储中，可以安全地删除用于计算的 HDInsight 群集而不会丢失用户数据。

> [WACOM.NOTE] HDInsight 群集版本 3.0 及将来的更高版本都不支持 *asv://* 语法。这就意味着提交到 HDInsight 群集版本 3.0 的任何显式使用“asv://”语法的作业都将会失败。应改用 *wasb://* 语法。而且，提交到任何 HDInsight 群集版本 3.0 的作业，如果是使用现有元存储创建的，而该元存储包含对使用 asv:// 语法的资源的显式引用，则这些作业也会失败。这些元存储将需要使用 wasb:// 重新创建以确定资源地址。

> [WACOM.NOTE] HDInsight 目前只支持块 Blob。

> [WACOM.NOTE]
> 大多数 HDFS 命令仍按预期效果工作，比如 **ls**、**copyFromLocal**、**mkdir**，等等。只有特定于本机 HDFS 实现（称作 DFS）的命令在 Azure Blob 存储上会显示不同的行为，比如 **fschk** 和 **dfsadmin**。

有关设置 HDInsight 群集的信息，请参阅 [HDInsight 入门][]或[设置 HDInsight 群集][]。

## 本文内容

-   [HDInsight 存储空间体系结构][]
-   [Azure Blob 存储的优点][]
-   [为 Blob 存储空间准备容器][]
-   [确定 Blob 存储空间中文件的地址][]
-   [使用 PowerShell 访问 Blob][]
-   [后续步骤][]

## HDInsight 存储体系结构

下图提供了 HDInsight 存储空间体系结构的抽象视图：

![HDI.ASVArch][]

HDInsight 提供对在本地附加到计算节点的分布式文件系统的访问。可使用完全限定 URI 访问该文件系统。例如：

    hdfs://<namenodehost>/<path>

另外，HDInsight 提供了访问 Blob 存储中所存储数据的功能。访问 Blob 存储的语法是：

    wasb[s]://<containername>@<accountname>.blob.core.chinacloudapi.cn/<path>

Hadoop 支持默认文件系统的概念。默认文件系统意指默认方案和授权；它还可用于解析相对路径。在 HDInsight 设置过程中，请将 Azure 存储帐户和该帐户上的特定 Blob 存储容器指定为默认文件系统。

除了此存储帐户外，在设置过程中，你还可以从同一 Azure 订阅或不同 Azure 订阅添加其他存储帐户。有关添加其他存储帐户的说明，请参阅[设置 HDInsight 群集][]。

-   **连接到群集的存储帐户中的容器：**由于帐户名称和密钥存储在 *core-site.xml* 中，所以你具有那些窗口中的 Blob 的完全访问权限。
-   **没有连接到群集的存储帐户中的公共容器或公共 Blob：**你对容器中的 Blob 具有只读权限。

    > [WACOM.NOTE]
    >  \> 利用公共容器，你可以获得该容器中所有可用 Blob 的列表以及容器元数据。利用公共 Blob，你仅在知道正确 URL 时才可访问 Blob。有关详细信息，请参阅[限制对容器和 Blob 的访问][]。

-   **没有连接到群集的存储帐户中的私有容器：**你不能访问容器中的这些 Blob，除非在提交 WebHCat 作业时定义存储帐户。本文后面对此进行了解释。

设置过程中定义的存储帐户及其密钥存储在 %HADOOP\_HOME%/conf/core-site.xml 中。HDInsight 的默认行为是使用 core-site.xml 文件中定义的存储帐户。不推荐编辑 core-site.xml 文件，因为群集头节点 (master) 可能会随时重新映像或迁移，对那些文件所做的任何更改都将会丢失。

多个 WebHCat 作业，包括 Hive、MapReduce、Hadoop 流和 Pig，都可以带有存储帐户和元数据的说明（它目前对带有存储帐户的 Pig 有效，但对元数据无效）。在本文的[使用 PowerShell 访问 Blob][] 一节中，提供了此功能的示例。有关详细信息，请参阅[将 HDInsight 群集与备用存储帐户和元存储配合使用][]。

Blob 存储容器将数据存储为键值对，没有目录层次结构。不过，可在键名称中使用“/”字符，使其看起来像存储在目录结构中的文件。例如，Blob 的键可以是 *input/log1.txt*。不存在实际的 *input* 目录，但由于键名称中包含“/”字符，因此使其看起来像一个文件路径。

## Azure Blob 存储的优点

通过在 Azure 数据中心的存储帐户资源附近设置计算群集，使计算节点能够通过高速网络非常高效地访问 Blob 存储中的数据，从而减少了不并置计算和存储所产生的隐含性能成本。

在 Blob 存储而非 HDFS 中存储数据有几个好处：

-   **数据重用和共享：**HDFS 中的数据位于计算群集内。仅有权访问计算群集的应用程序才能通过 HDFS API 使用数据。Blob 存储中的数据可通过 HDFS API 或 [Blob 存储 REST API][] 访问。因此，可使用大量应用程序（包括其他 HDInsight 群集）和工具来生成和使用此类数据。
-   **数据存档：** 通过在 Blob 存储中存储数据，可以安全地删除用于计算的 HDInsight 群集而不会丢失用户数据。
-   **数据存储成本：**与在 Blob 存储中存储数据相比，在 DFS 中长期存储数据的成本更高，因为计算群集的成本高于 Blob 存储容器的成本。此外，由于数据无需在每次生成计算群集时重新加载，也为你节省了数据加载成本。
-   **灵活向外扩展：**尽管 HDFS 为你提供了向外扩展文件系统，但缩放将由你为群集设置的节点的数量决定。与依靠你自动获得的 Blob 存储的弹性缩放功能相比，更改缩放的过程可能会更复杂。
-   **地理复制：**可通过 Azure 门户对你的 Blob 存储容器进行地理复制。虽然这可为你提供地理恢复和数据冗余功能，但针对地理复制位置的故障转移将大大影响你的性能，并且可能会产生额外成本。因此，我们建议你仅在数据的价值值得你支付额外成本时才选择适当的地理复制。

某些 MapReduce 作业和包可能会产生中间结果，你并不想在 Blob 存储容器中存储这些结果。在此情况下，你仍可以选择将数据存储在本地 HDFS 中。实际上，HDInsight 在 Hive 作业和其他过程中会为其中某些中间结果使用 DFS。

## 为 Blob 存储准备容器

若要使用 blob，必须先创建 [Azure 存储帐户][]。创建过程中，你要指定 Azure 数据中心，它将存储你使用此帐户创建的对象。群集和存储帐户都必须承载于同一数据中心（Hive 元存储 SQL 数据库和 Oozie 元存储 SQL 数据库也必须位于同一数据中心）。无论所创建的每个 blob 位于何处，它都属于存储帐户中的某个容器。此容器可以是在 HDInsight 外部创建的现有的 Blob 存储容器，也可以是为 HDInsight 群集创建的容器。

### 使用管理门户为 HDInsight 创建 Blob 容器

在 Azure 管理门户中设置 HDInsight 群集时，有两个选项：“快速创建”**和“自定义创建”**。快速创建选项要求提前创建好 Azure 存储帐户。有关说明，请参阅[如何创建存储帐户][Azure 存储帐户]。

使用快速创建选项时，你可以选择现有存储帐户。设置过程将创建一个与 HDInsight 群集同名的新容器。如果已存在同名的容器，将使用 -，例如 myHDIcluster-1。此容器用作默认文件系统。

![HDI.QuickCreate][]

使用自定义创建时，对于默认存储空间帐户，你可以选择以下选项之一：

-   使用现有存储
-   创建新存储
-   使用其他订阅中的存储。

你还可以选择创建你自己的 Blob 容器或使用现有的容器。

![HDI.CustomCreateStorageAccount][]

### 使用 Azure PowerShell 创建容器。

[Azure PowerShell][] 可用于创建 Blob 容器。下面是 PowerShell 脚本示例：

    $subscriptionName = "<SubscriptionName>"   # Azure 订阅名称
    $storageAccountName = "<AzureStorageAccountName>" # 你将要创建的存储帐户
    $containerName="<BlobContainerToBeCreated>" # 你将要创建的 Blob 容器名称

    # 连接到你的 Azure 帐户并选择当前订阅
    Add-AzureAccount # 该连接将在几小时后到期。
    Select-AzureSubscription $subscriptionName #只在你有多个订阅时需要

    # 创建存储上下文对象
    $storageAccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}
    $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    # 创建 Blob 存储容器
    New-AzureStorageContainer -Name $containerName -Context $destContext 

## 确定 Blob 存储中文件的地址

用于访问 Blob 存储中的文件的 URI 方案为：

    wasb[s]://<BlobStorageContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/<path>

> [WACOM.NOTE] 用于在存储模拟器上确定文件地址的语法（在 HDInsight 模拟器上运行）是 *wasb:*
> //\<ContainerName\>@storageemulator (//\<ContainerName\> at storageemulator)
> .

URI 方案提供了使用 *wasb:*前缀的未加密访问和使用 *wasbs* 的 SSL 加密访问。建议尽量使用 *wasbs*，即使在访问位于同一 Azure 数据中心内的数据时也是如此。

\<BlobStorageContainerName\> 标识 Blob 存储容器的名称。
\<StorageAccountName\> 标识 Azure 存储帐户名。完全限定域名 (FQDN) 是必需的。

如果既没有指定 \<BlobStorageContainerName\> 也没有指定 \<StorageAccountName\>，则会使用默认文件系统。对于默认文件系统中的文件，你可以使用相对路径或绝对路径。例如，可以使用以下任一方式引用随 HDInsight 群集提供的 hadoop-mapreduce-examples.jar 文件：

    wasb://mycontainer@myaccount.blob.core.chinacloudapi.cn/example/jars/hadoop-mapreduce-examples.jar
    wasb:///example/jars/hadoop-mapreduce-examples.jar
    /example/jars/hadoop-mapreduce-examples.jar

> [WACOM.NOTE] 在 HDInsight 群集版本 1.6 和 2.1 上，文件名是 *hadoop-examples.jar*。

\<path\> 是文件或目录 HDFS 路径名。由于 Blob 存储容器只是键值存储，因此没有真正的分层文件系统。Blob 键中的“/”解释为目录分隔符。例如，*hadoop-mapreduce-examples.jar* 的 Blob 名称是：

    example/jars/hadoop-mapreduce-examples.jar

## 使用 PowerShell 访问 Blob

有关在你的工作站上安装和配置 Azure PowerShell 的信息，请参阅[安装和配置 Azure PowerShell][Azure PowerShell]。你可以使用 Azure PowerShell 控制台窗口或 PowerShell\_ISE 来运行 PowerShell cmdlet。

使用以下命令列出与 Blob 有关的 cmdlet：

    Get-Command *blob*

![Blob.PowerShell.cmdlets][]

**用于上载文件的 PowerShell 示例**

请参阅[将数据上载到 HDInsight][]。

**用于下载文件的 PowerShell 示例**

以下脚本将一个块 Blob 下载到当前文件夹。运行该脚本之前，请将该目录更改为你有写权限的文件夹。

    $storageAccountName = "<AzureStorageAccountName>"   # 用于在设置时指定的默认文件系统的存储帐户。
    $containerName = "<BlobStorageContainerName>"  # 默认文件系统容器与群集同名。
    $blob = "example/data/sample.log" # 要下载的 Blob 的名称。

    # 如果你没有连接到 Azure 订阅，则使用 Add-AzureAccount
    #Add-AzureAccount # 该连接在 12 小时内有效

    # 如果你有多个订阅，则使用这两个命令
    #$subscriptionName = "<SubscriptionName>"       
    #Select-AzureSubscription $subscriptionName

    Write-Host "Create a context object ..." -ForegroundColor Green
    $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
    $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    Write-Host "Download the blob ..."-ForegroundColor Green
    Get-AzureStorageBlobContent -Container $ContainerName -Blob $blob -Context $storageContext -Force

    Write-Host "List the downloaded file ..."-ForegroundColor Green
    cat "./$blob"

**用于删除文件的 PowerShell 示例**

以下脚本显示如何删除文件。
\$storageAccountName = "" \# 用于在设置时指定的默认文件系统的存储帐户。
\$containerName = "" \# 默认文件系统容器与群集同名。
\$blob = "example/data/sample.log" \# 要下载的 Blob 的名称。

    # 如果你没有连接到 Azure 订阅，则使用 Add-AzureAccount
    #Add-AzureAccount # 该连接在 12 小时内有效

    # 如果你有多个订阅，则使用这两个命令
    #$subscriptionName = "<SubscriptionName>"       
    #Select-AzureSubscription $subscriptionName

    Write-Host "Create a context object ..." -ForegroundColor Green
    $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
    $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    Write-Host "Delete the blob ..."-ForegroundColor Green
    Remove-AzureStorageBlob -Container $containerName -Context $storageContext -blob $blob 

**用于列出文件夹中文件的 PowerShell 示例**

以下脚本显示如何列出“文件夹”内的文件。下一个示例显示如何使用 Invoke-Hive cmdlet 来执行 dfs ls 命令以列出文件夹。

    $storageAccountName = "<AzureStorageAccountName>"   # 用于在设置时指定的默认文件系统的存储帐户。
    $containerName = "<BlobStorageContainerName>"  # 默认文件系统容器与群集同名。
    $blobPrefix = "example/data/"

    # 如果你没有连接到 Azure 订阅，则使用 Add-AzureAccount
    #Add-AzureAccount # 该连接在 12 小时内有效

    # 如果你有多个订阅，则使用这两个命令
    #$subscriptionName = "<SubscriptionName>"       
    #Select-AzureSubscription $subscriptionName

    Write-Host "Create a context object ..." -ForegroundColor Green
    $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
    $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    Write-Host "List the files in $blobPrefix ..."
    Get-AzureStorageBlob -Container $containerName -Context $storageContext -prefix $blobPrefix

**用于访问未定义的存储帐户的 PowerShell 示例**

此示例显示如何列出在设置过程中未定义的存储帐户的文件夹。
\$clusterName = ""

    $undefinedStorageAccount = "<UnboundedStorageAccountUnderTheSameSubscription>"
    $undefinedContainer = "<UnboundedBlobContainerAssociatedWithTheStorageAccount>"

    $undefinedStorageKey = Get-AzureStorageKey $undefinedStorageAccount | %{ $_.Primary }

    Use-AzureHDInsightCluster $clusterName

    $defines = @{}
    $defines.Add("fs.azure.account.key.$undefinedStorageAccount.blob.core.chinacloudapi.cn", $undefinedStorageKey)

    Invoke-Hive -Defines $defines -Query "dfs -ls wasb://$undefinedContainer@$undefinedStorageAccount.blob.core.chinacloudapi.cn/;"

## 后续步骤

在本文中，你学习了如何将 Blob 存储与 HDInsight 配合使用，并已了解 Blob 存储是 HDInsight 的一个基本组件。这样，你就可以使用 Azure Blob 存储来生成可伸缩的长期存档数据获取解决方案，并使用 HDInsight 来解锁所存储数据内的信息。

若要了解更多信息，请参阅下列文章：

-   [Azure HDInsight 入门][HDInsight 入门]
-   [将数据上传到 HDInsight][将数据上载到 HDInsight]
-   [Hive 与 HDInsight 配合使用][]
-   [Pig 与 HDInsight 配合使用][]

  [HDInsight 入门]: ../hdinsight-get-started/
  [设置 HDInsight 群集]: ../hdinsight-provision-clusters/
  [HDInsight 存储空间体系结构]: #architecture
  [Azure Blob 存储的优点]: #benefits
  [为 Blob 存储空间准备容器]: #preparingblobstorage
  [确定 Blob 存储空间中文件的地址]: #addressing
  [使用 PowerShell 访问 Blob]: #powershell
  [后续步骤]: #nextsteps
  [HDI.ASVArch]: ./media/hdinsight-use-blob-storage/HDI.ASVArch.png "HDInsight 存储体系结构"
  [限制对容器和 Blob 的访问]: http://msdn.microsoft.com/zh-cn/library/azure/dd179354.aspx
  [将 HDInsight 群集与备用存储帐户和元存储配合使用]: http://social.technet.microsoft.com/wiki/contents/articles/23256.using-an-hdinsight-cluster-with-alternate-storage-accounts-and-metastores.aspx
  [Blob 存储 REST API]: http://msdn.microsoft.com/zh-cn/library/azure/dd135733.aspx
  [Azure 存储帐户]: ../storage-create-storage-account/
  [HDI.QuickCreate]: ./media/hdinsight-use-blob-storage/HDI.QuickCreateCluster.png
  [HDI.CustomCreateStorageAccount]: ./media/hdinsight-use-blob-storage/HDI.CustomCreateStorageAccount.png
  [Azure PowerShell]: ../install-configure-powershell/
  [Blob.PowerShell.cmdlets]: ./media/hdinsight-use-blob-storage/HDI.PowerShell.BlobCommands.png
  [将数据上载到 HDInsight]: ../hdinsight-upload-data/
  [Hive 与 HDInsight 配合使用]: ../hdinsight-use-hive/
  [Pig 与 HDInsight 配合使用]: ../hdinsight-use-pig/
