<properties
    pageTitle="从 HDFS 兼容的 Azure 存储查询数据 | Azure"
    description="了解如何从 Azure 存储查询数据，以存储分析结果。"
    keywords="blob 存储,hdfs,结构化数据,非结构化数据"
    services="hdinsight,storage"
    documentationcenter=""
    tags="azure-portal"
    author="mumian"
    manager="jhubbard"
    editor="cgronlun"
    translationtype="Human Translation" />
<tags
    ms.assetid="1d2e65f2-16de-449e-915f-3ffbc230f815"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="02/27/2017"
    wacn.date="05/08/2017"
    ms.author="jgao"
    ms.sourcegitcommit="9b66f16218093b3750001d881c49cd8ebd506b22"
    ms.openlocfilehash="7fb3c467e11e817d661de7f7828ba13edcb45bd8"
    ms.lasthandoff="04/29/2017" />

# <a name="use-hdfs-compatible-storage-with-hadoop-in-hdinsight"></a>将 HDFS 兼容的存储与 HDInsight 中的 Hadoop 配合使用

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

若要分析 HDInsight 群集中的数据，可以将数据存储在 Azure 存储中。 使用这两个存储选项都可以安全地删除用于计算的 HDInsight 群集，而不会丢失用户数据。

Hadoop 支持默认文件系统的概念。 默认文件系统意指默认方案和授权。 它还可用于解析相对路径。 在 HDInsight 群集创建过程中，可以指定 Azure 存储中的 Blob 容器作为默认文件系统；或者在 HDInsight 3.5 中，可以选择 Azure 存储作为默认文件系统。

本文介绍这两个存储选项处理 HDInsight 群集的方式。 有关创建 HDInsight 群集的详细信息，请参阅 [HDInsight 入门](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)。

## <a name="using-azure-storage-with-hdinsight-clusters"></a>将 Azure 存储与 HDInsight 群集配合使用

Azure 存储是一种稳健、通用的存储解决方案，它与 HDInsight 无缝集成。 HDInsight 可将 Azure 存储中的 Blob 容器用作群集的默认文件系统。 通过 Hadoop 分布式的文件系统 (HDFS) 界面，可以针对作为 Blob 存储的结构化或非结构化数据直接运行 HDInsight 中的整套组件。

> [AZURE.WARNING]
> 创建 Azure 存储帐户时，有几个选项可用。 下表介绍了 HDInsight 支持的选项：
> 
> | 存储帐户类型 | 存储层 | 受 HDInsight 支持 |
> | ------- | ------- | ------- |
> | 通用存储帐户 | 标准 | __是__ |
> | &nbsp; | 高级 | 否 |
> | Blob 存储帐户 | 热 | 否 |
> | &nbsp; | 冷 | 否 |

### <a id="architecture"></a> HDInsight 存储体系结构
下图提供了 HDInsight 存储体系结构的抽象视图：

![Hadoop 群集使用 HDFS API 来访问 Blob 存储中的结构化和非结构化数据，并在其中存储这些数据。](./media/hdinsight-hadoop-use-blob-storage/HDI.WASB.Arch.png "HDInsight Storage Architecture")

HDInsight 提供对在本地附加到计算节点的分布式文件系统的访问权限。 可使用完全限定 URI 访问该文件系统，例如：

    hdfs://<namenodehost>/<path>

另外，通过 HDInsight 还能访问 Azure 存储中存储的数据。 语法为：

    wasb[s]://<containername>@<accountname>.blob.core.chinacloudapi.cn/<path>

下面是将 Azure 存储帐户与 HDInsight 群集配合使用时的一些注意事项。

* **连接到群集的存储帐户中的容器：** 由于在创建过程中帐户名称和密钥将与群集相关联，因此你对这些容器中的 Blob 具有完全访问权限。

* **没有连接到群集的存储帐户中的公共容器或公共 Blob：** 你对这些容器中的 Blob 具有只读权限。

    > [AZURE.NOTE]
    > 利用公共容器，你可以获得该容器中可用的所有 Blob 的列表以及容器元数据。 利用公共 Blob，你仅在知道正确 URL 时才可访问 Blob。 有关详细信息，请参阅<a href="/documentation/articles/storage-manage-access-to-resources/">限制对容器和 Blob 的访问</a>。
    > 
    > 
* **没有连接到群集的存储帐户中的私有容器：** 你不能访问这些容器中的 Blob，除非在提交 WebHCat 作业时定义存储帐户。 本文后面对此做了解释。

创建过程中定义的存储帐户及其密钥存储在群集节点上的 %HADOOP/_HOME%/conf/core-site.xml 中。 HDInsight 的默认行为是使用 core-site.xml 文件中定义的存储帐户。 不建议直接编辑 core-site.xml 文件，因为群集头节点 (master) 可能会随时重新映像或迁移，对该文件所做的任何更改都不会保留。

多个 WebHCat 作业，包括 Hive、MapReduce、Hadoop 流和 Pig，都可以带有存储帐户和元数据的说明。 （它目前对带有存储帐户的 Pig 有效，但对元数据无效。）在本文的[使用 Azure PowerShell 访问 Blob](#powershell) 部分中，提供了此功能的示例。 有关详细信息，请参阅 [将 HDInsight 群集与备用存储帐户和元存储配合使用](http://social.technet.microsoft.com/wiki/contents/articles/23256.using-an-hdinsight-cluster-with-alternate-storage-accounts-and-metastores.aspx)。

Blob 可用于结构化和非结构化数据。 Blob 容器将数据存储为键值对，没有目录层次结构。 不过，可在键名称中使用斜杠字符 (/)，使其看起来像存储在目录结构中的文件。 例如，Blob 的键可以是 *input/log1.txt*。 不存在实际的 *input* 目录，但由于键名称中包含斜线字符，因此使其看起来像文件路径。

### <a id="benefits"></a>Azure 存储的优点
通过在 Azure 区域的存储帐户资源附近创建计算群集，使计算节点能够通过高速网络非常高效地访问 Azure 存储中的数据，从而减少了非并置计算群集和存储资源所产生的隐含性能成本。

在 Azure 存储而非 HDFS 中存储数据有几个好处：

* **数据重用和共享：** HDFS 中的数据位于计算群集内。 仅有权访问计算群集的应用程序才能通过 HDFS API 使用数据。 Azure 存储中的数据可通过 HDFS API 或 [Blob 存储 REST API][blob-storage-restAPI] 访问。 因此，可使用大量应用程序（包括其他 HDInsight 群集）和工具来生成和使用此类数据。
* **数据存档：** 通过在 Azure 存储中存储数据，可以安全地删除用于计算的 HDInsight 群集而不会丢失用户数据。
* **数据存储成本：** 与在 Azure 存储中存储数据相比，在 DFS 中长期存储数据的成本更高，因为计算群集的成本高于 Azure 存储的成本。 此外，由于数据无需在每次生成计算群集时重新加载，也为你节省了数据加载成本。
* **弹性扩展：** 尽管 HDFS 提供扩展文件系统，但规模由为群集创建的节点数量决定。 与依靠自动获得的 Azure 存储的弹性缩放功能相比，更改规模的过程可能更复杂。
* **异地复制：** 可对 Azure 存储进行异地复制。 尽管这可为你提供地理恢复和数据冗余功能，但针对异地复制位置的故障转移将大大影响你的性能，并且可能会产生额外成本。 因此，我们建议你仅在数据的价值值得你支付额外成本时才选择适当的地理复制。

某些 MapReduce 作业和包可能会产生中间结果，你并不想在 Azure 存储中存储这些结果。 在此情况下，你仍可以选择将数据存储在本地 HDFS 中。 实际上，HDInsight 在 Hive 作业和其他过程中会为其中某些中间结果使用 DFS。

> [AZURE.NOTE]
> 大多数 HDFS 命令（例如 <b>ls</b>、<b>copyFromLocal</b> 和 <b>mkdir</b>）仍按预期工作。 只有特定于本机 HDFS 实现（称作 DFS）的命令在 Azure 存储上会显示不同的行为，例如 <b>fschk</b> 和 <b>dfsadmin</b>。
> 
> 

### <a id="preparingblobstorage"></a> 创建 Blob 容器
若要使用 Blob，必须先创建 [Azure 存储帐户][azure-storage-create]。 在此过程中，可指定在其中创建存储帐户的 Azure 区域。 群集和存储帐户必须位于同一区域。 Hive 元存储 SQL Server 数据库和 Oozie 元存储 SQL Server 数据库也必须位于同一区域。

无论所创建的每个 Blob 位于何处，它都属于 Azure 存储帐户中的某个容器。 此容器可以是在 HDInsight 外部创建的现有的 Blob，也可以是为 HDInsight 群集创建的容器。

默认的 Blob 容器存储群集特定的信息，如作业历史记录和日志。 请不要多个 HDInsight 群集之间共享默认的 Blob 容器。 这可能会损坏作业历史记录。 建议对每个群集使用不同的容器，并将共享数据放入在所有相关群集的部署中指定的链接存储帐户，而不是放入默认存储帐户。 有关配置链接存储帐户的详细信息，请参阅 [创建 HDInsight 群集][hdinsight-creation]。 但是，在删除原始的 HDInsight 群集后，你可以重用默认存储容器。 对于 HBase 群集，实际上可以通过使用已删除的 HBase 群集使用的默认 Blob 容器创建新的 HBase 群集来保留 HBase 表架构和数据。

#### <a name="using-the-azure-portal-preview"></a>使用 Azure 门户预览
从门户创建 HDInsight 群集时，可通过以下选项提供存储帐户详细信息。 还可以指定是否要将附加存储帐户与该群集相关联，如果需要，请选择 Azure 存储 Blob 作为附加存储。

![HDInsight Hadoop - 创建数据源](./media/hdinsight-hadoop-use-blob-storage/hdinsight.provision.data.source.png)

> [AZURE.WARNING]
> 不支持在 HDInsight 群集之外的其他位置使用别的存储帐户。

#### <a name="using-azure-cli"></a>使用 Azure CLI
[AZURE.INCLUDE [use-latest-version](../../includes/hdinsight-use-latest-cli.md)]

如果 [已安装并配置 Azure CLI](/documentation/articles/cli-install-nodejs/)，则以下命令可以用于存储帐户和容器。

    azure storage account create <storageaccountname> --type LRS

> [AZURE.NOTE]
> `--type` 参数指示如何复制存储帐户。 有关详细信息，请参阅 [Azure 存储复制](/documentation/articles/storage-redundancy/)。 不要使用 ZRS，因为 ZRS 不支持页 blob、文件、表或队列。
> 
> 

系统会提示你指定创建存储帐户的地理区域。 应在计划创建 HDInsight 群集的同一区域中创建存储帐户。

创建存储帐户后，使用以下命令检索存储帐户密钥：

    azure storage account keys list <storageaccountname>

若要创建容器，请使用以下命令：

    azure storage container create <containername> --account-name <storageaccountname> --account-key <storageaccountkey>

#### <a name="using-azure-powershell"></a>使用 Azure PowerShell
如果 [已安装并配置 Azure PowerShell][powershell-install]，可从 Azure PowerShell 提示符使用以下命令来创建存储帐户和容器：

[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

    $SubscriptionID = "<Your Azure Subscription ID>"
    $ResourceGroupName = "<New Azure Resource Group Name>"
    $Location = "CHINA EAST"

    $StorageAccountName = "<New Azure Storage Account Name>"
    $containerName = "<New Azure Blob Container Name>"

    Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    Select-AzureRmSubscription -SubscriptionId $SubscriptionID

    # Create resource group
    New-AzureRmResourceGroup -name $ResourceGroupName -Location $Location

    # Create default storage account
    New-AzureRmStorageAccount -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -Location $Location -Type Standard_LRS 

    # Create default blob containers
    $storageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -StorageAccountName $StorageAccountName)[0].Value
    $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  
    New-AzureStorageContainer -Name $containerName -Context $destContext

### <a id="addressing"></a> 确定 Azure 存储中文件的地址
用于从 HDInsight 访问 Azure 存储中的文件的 URI 方案为：

    wasb[s]://<BlobStorageContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/<path>

URI 方案提供了使用 *wasb:* 前缀的未加密访问和使用 *wasbs* 的 SSL 加密访问。 建议尽量使用 *wasbs* ，即使在访问位于同一 Azure 区域内的数据时也是如此。

&lt;BlobStorageContainerName&gt; 标识 Azure 存储中 Blob 容器的名称。
&lt;StorageAccountName 标识&gt; Azure 存储帐户名。 完全限定域名 (FQDN) 是必需的。

如果既没有指定 &lt;BlobStorageContainerName&gt; 也没有指定 &lt;StorageAccountName&gt;，则使用默认文件系统。 对于默认文件系统中的文件，你可以使用相对路径或绝对路径。 例如，可以使用以下任一方式引用随 HDInsight 群集提供的 *hadoop-mapreduce-examples.jar* 文件：

    wasbs://mycontainer@myaccount.blob.core.chinacloudapi.cn/example/jars/hadoop-mapreduce-examples.jar
    wasbs:///example/jars/hadoop-mapreduce-examples.jar
    /example/jars/hadoop-mapreduce-examples.jar

> [AZURE.NOTE]
> 在 HDInsight 版本 2.1 和 1.6 群集中，文件名是 <i>hadoop-examples.jar</i>。
> 
> 

&lt;path&gt; 是文件或目录 HDFS 路径名。 由于 Azure 存储容器只是键值存储，因此没有真正的分层文件系统。 Blob 键中的斜杠字符 (/) 解释为目录分隔符。 例如， *hadoop-mapreduce-examples.jar* 的 Blob 名称是：

    example/jars/hadoop-mapreduce-examples.jar

> [AZURE.NOTE]
> 在 HDInsight 外部使用 Blob 时，大多数实用程序无法识别 WASB 格式，应改用基本路径格式，如 `example/jars/hadoop-mapreduce-examples.jar`。
> 
> 

### <a id="azurecli"></a> 使用 Azure CLI 访问 Blob
使用以下命令列出与 Blob 有关的命令：

    azure storage blob

**使用 Azure CLI 上传文件的示例**

    azure storage blob upload <sourcefilename> <containername> <blobname> --account-name <storageaccountname> --account-key <storageaccountkey>

**使用 Azure CLI 下载文件的示例**

    azure storage blob download <containername> <blobname> <destinationfilename> --account-name <storageaccountname> --account-key <storageaccountkey>

**使用 Azure CLI 删除文件的示例**

    azure storage blob delete <containername> <blobname> --account-name <storageaccountname> --account-key <storageaccountkey>

**使用 Azure CLI 列出文件的示例**

    azure storage blob list <containername> <blobname|prefix> --account-name <storageaccountname> --account-key <storageaccountkey>

### <a id="powershell" name="access-blobs-using-azure-powershell"></a> 使用 Azure PowerShell 访问 Blob
> [AZURE.NOTE]
> 本部分中的命令提供了使用 PowerShell 访问 Blob 中存储的数据的基本示例。 有关针对使用 HDInsight 自定义的功能更加全面的示例，请参阅 [HDInsight 工具](https://github.com/Blackmist/hdinsight-tools)。
> 
> 

使用以下命令列出与 Blob 有关的 cmdlet：

    Get-Command *blob*

![Blob 相关 PowerShell cmdlet 的列表。][img-hdi-powershell-blobcommands]

#### <a name="upload-files"></a>上载文件
请参阅 [将数据上传到 HDInsight][hdinsight-upload-data]。

#### <a name="download-files"></a>下载文件
以下脚本将一个块 Blob 下载到当前文件夹。 运行该脚本之前，请将该目录更改为你有写权限的文件夹。

    $resourceGroupName = "<AzureResourceGroupName>"
    $storageAccountName = "<AzureStorageAccountName>"   # The storage account used for the default file system specified at creation.
    $containerName = "<BlobStorageContainerName>"  # The default file system container has the same name as the cluster.
    $blob = "example/data/sample.log" # The name of the blob to be downloaded.

    # Use Add-AzureAccount -Environment AzureChinaCloud if you haven't connected to your Azure subscription
    Login-AzureRmAccount -EnvironmentName AzureChinaCloud 
    Select-AzureRmSubscription -SubscriptionID "<Your Azure Subscription ID>"

    Write-Host "Create a context object ... " -ForegroundColor Green
    $storageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName)[0].Value
    $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    Write-Host "Download the blob ..." -ForegroundColor Green
    Get-AzureStorageBlobContent -Container $ContainerName -Blob $blob -Context $storageContext -Force

    Write-Host "List the downloaded file ..." -ForegroundColor Green
    cat "./$blob"

如果提供资源组名称和群集名称，可以使用以下代码：

    $resourceGroupName = "<AzureResourceGroupName>"
    $clusterName = "<HDInsightClusterName>"
    $blob = "example/data/sample.log" # The name of the blob to be downloaded.

    $cluster = Get-AzureRmHDInsightCluster -ResourceGroupName $resourceGroupName -ClusterName $clusterName
    $defaultStorageAccount = $cluster.DefaultStorageAccount -replace '.blob.core.chinacloudapi.cn'
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccount)[0].Value
    $defaultStorageContainer = $cluster.DefaultStorageContainer
    $storageContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccount -StorageAccountKey $defaultStorageAccountKey 

    Write-Host "Download the blob ..." -ForegroundColor Green
    Get-AzureStorageBlobContent -Container $defaultStorageContainer -Blob $blob -Context $storageContext -Force

#### <a name="delete-files"></a>删除文件
    Remove-AzureStorageBlob -Container $containerName -Context $storageContext -blob $blob

#### <a name="list-files"></a>列出文件
    Get-AzureStorageBlob -Container $containerName -Context $storageContext -prefix "example/data/"

#### <a name="run-hive-queries-using-an-undefined-storage-account"></a>使用未定义的存储帐户运行 Hive 查询
此示例显示如何列出在创建过程中未定义的存储帐户的文件夹。

    $clusterName = "<HDInsightClusterName>"

    $undefinedStorageAccount = "<UnboundedStorageAccountUnderTheSameSubscription>"
    $undefinedContainer = "<UnboundedBlobContainerAssociatedWithTheStorageAccount>"

    $undefinedStorageKey = Get-AzureStorageKey $undefinedStorageAccount | %{ $_.Primary }

    Use-AzureRmHDInsightCluster $clusterName

    $defines = @{}
    $defines.Add("fs.azure.account.key.$undefinedStorageAccount.blob.core.chinacloudapi.cn", $undefinedStorageKey)

    Invoke-AzureRmHDInsightHiveJob -Defines $defines -Query "dfs -ls wasbs://$undefinedContainer@$undefinedStorageAccount.blob.core.chinacloudapi.cn/;"

### <a name="using-additional-storage-accounts"></a>使用其他存储帐户

创建 HDInsight 群集时，可以指定要与其关联的 Azure 存储帐户。 除了此存储帐户外，在创建过程中或群集创建完成后，还可以从同一 Azure 订阅或不同 Azure 订阅添加其他存储帐户。 有关添加其他存储帐户的说明，请参阅[创建 HDInsight 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)。

> [AZURE.WARNING]
> 不支持在 HDInsight 群集之外的其他位置使用别的存储帐户。

## <a name="next-steps"></a>后续步骤
本文已介绍如何将 HDFS 兼容的 Azure 存储与 HDInsight 配合使用。 这样，你便可以构建可缩放的长期存档数据获取解决方案，并使用 HDInsight 来解锁存储的结构化和非结构化数据中的信息。

有关详细信息，请参阅：

* [Azure HDInsight 入门][hdinsight-get-started]
* [将数据上传到 HDInsight][hdinsight-upload-data]
* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
* [使用 Azure 存储共享访问签名来限制使用 HDInsight 访问数据][hdinsight-use-sas]

[hdinsight-use-sas]: /documentation/articles/hdinsight-storage-sharedaccesssignature-permissions/
[powershell-install]: https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs
[hdinsight-creation]: /documentation/articles/hdinsight-hadoop-provision-linux-clusters/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/

[blob-storage-restAPI]: http://msdn.microsoft.com/zh-cn/library/azure/dd135733.aspx
[azure-storage-create]: /documentation/articles/storage-create-storage-account/

[img-hdi-powershell-blobcommands]: ./media/hdinsight-hadoop-use-blob-storage/HDI.PowerShell.BlobCommands.png
[img-hdi-quick-create]: ./media/hdinsight-hadoop-use-blob-storage/HDI.QuickCreateCluster.png
[img-hdi-custom-create-storage-account]: ./media/hdinsight-hadoop-use-blob-storage/HDI.CustomCreateStorageAccount.png

<!--Update_Description: wording update-->