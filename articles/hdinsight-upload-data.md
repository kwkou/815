<properties linkid="manage-services-hdinsight-howto-upload-data-to-hdinsight" urlDisplayName="Upload Data" pageTitle="Upload data to HDInsight | Azure" metaKeywords="" description="Learn how to upload and access data in HDInsight using Azure Storage Explorer, the interactive console, the Hadoop command line, or Sqoop." metaCanonical="" services="storage,hdinsight" documentationCenter="" title="Upload data to HDInsight" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />

# 将数据上传到 HDInsight

Azure HDInsight 在 Azure Blob 存储之上提供了一个功能完备的 Hadoop 分布式文件系统 (HDFS)。该系统设计为一个 HDFS 扩展，以便通过启用 Hadoop 生态系统中的整套组件以直接操作其管理的数据来为客户提供无缝体验。Azure Blob 存储和 HDFS 是独立的文件系统，并且已针对数据的存储和计算进行了优化。有关使用 Azure Blob 存储的益处，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

通常，为了执行 MapReduce 作业，会部署 Azure HDInsight 群集，一旦这些作业完成，就删除这些群集。在完成计算后将数据保存在 HDFS 群集中是一种成本很高的数据存储方法。对于将使用 HDInsight 处理的数据而言，Azure Blob 存储是一个高度可用的、可高度缩放的、大容量、低成本且可共享的存储选项。在 Blob 中存储数据，可以安全地释放用于计算的 HDInsight 群集而不丢失数据。

可以通过 [AzCopy][]、[Azure PowerShell][]、[Azure .NET 存储客户端库][]或资源管理器工具访问 Azure Blob 存储。下面是一些可用的工具：

-   [Azure 存储空间资源管理器][]
-   [Cloud Storage Studio 2][]
-   [CloudXplorer][]
-   [Azure 资源管理器][]
-   [Azure 资源管理器专业版][]

**必备条件**

在开始阅读本文前，请注意以下要求：

-   一个 Azure HDInsight 群集。有关说明，请参阅 [Azure HDInsight 入门][]或[设置 HDInsight 群集][]。

## 本文内容

-   [使用 AzCopy 将数据上传到 Blob 存储空间][]
-   [使用 Azure PowerShell 将数据上载到 Blob 存储][]
-   [使用 Azure 存储空间资源管理器将数据上传到 Blob 存储空间][]
-   [使用 Hadoop 命令行将数据上传到 Blob 存储空间][]
-   [使用 Sqoop 从 Azure SQL Database 将数据导入到 Blob 存储][]

## 使用 AzCopy 将数据上载到 Blob 存储

AzCopy 是一个命令行实用程序，用于简化将数据传入和传出 Azure 存储帐户的任务。你可以将它用作独立的工具，也可以将此实用程序融入到现有应用程序中。[下载 AzCopy][]。

AzCopy 语法为：

    AzCopy <Source> <Destination> [filePattern [filePattern...]][Options]

有关详细信息，请参阅 [AzCopy - 上载/下载 Azure Blob 的文件][AzCopy]

## 使用 Azure PowerShell 将数据上载到 Blob 存储

Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。你可以使用 Azure PowerShell 将数据上载到 Blob 存储，以便 MapReduce 作业可以处理数据。有关配置工作站以运行 Azure PowerShell 的信息，请参阅[安装和配置 Azure PowerShell][]。

**将本地文件上传到 Blob 存储**

1.  按[安装和配置 Azure PowerShell][] 中的说明打开 Azure PowerShell 控制台窗口。
2.  设置以下脚本中前五个变量的值：

        $subscriptionName = "<AzureSubscriptionName>"
        $storageAccountName = "<StorageAccountName>"
        $containerName = "<ContainerName>"

        $fileName ="<LocalFileName>"
        $blobName = "<BlobName>"

        # 获取存储帐户密钥
        Select-AzureSubscription $subscriptionName
        $storageaccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}

        # 创建存储上下文对象
        $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageaccountkey

        # 将本地工作站中的文件复制到 Blob 容器        
        Set-AzureStorageBlobContent -File $fileName -Container $containerName -Blob $blobName -context $destContext

3.  将脚本粘贴到 Azure PowerShell 控制台窗口中以运行此脚本。

Blob 存储容器将数据存储为键值对，没有目录层次结构。不过，可在键名称中使用“/”字符，使其看起来像存储在目录结构中的文件。例如，Blob 的键可以是 *input/log1.txt*。不存在实际的“input”目录，但由于键名称中包含“/”字符，因此使其看起来像一个文件路径。在前一个脚本中，你可以通过设置 \$blobname 变量为此文件提供一个文件夹结构。例如，*\$blobname="myfolder/myfile.txt"*。

使用 Azure 资源管理器工具，你会注意到一些 0 字节的文件。这些文件有两个作用：

-   在有空文件夹的情况下，它们充当文件夹存在的标记。Blob 存储足够聪明，可以了解如果存在名为 foo/bar 的 blob，就存在一个名为 foo 的文件夹。但如果你希望具有一个名为 foo 的空文件夹，则指明这一点的唯一方法是放入这个特殊的 0 字节文件。
-   它们具有 Hadoop 文件系统所需的一些特别的元数据，最明显的是文件夹的权限和所有者。

## 使用 Azure 存储资源管理器将数据上载到 Blob 存储

*Azure 存储资源管理器*是用于在 Azure 存储空间中检查和更改数据的有用工具。它是免费工具，可从 [][]<http://azurestorageexplorer.codeplex.com/>下载。

使用该工具之前，你必须知道你的 Azure 存储帐户名和帐户密钥。有关如何获取此信息的说明，请参见[管理存储帐户][]中的“如何：查看、复制和重新生成存储访问密钥”一节。

1.  运行 Azure 存储空间资源管理器。

    ![HDI.AzureStorageExplorer][]

2.  单击“添加帐户” 。在将帐户添加到 Azure 存储资源管理器后，你无需再执行该步骤。

    ![HDI.ASEAddAccount][]

3.  输入存储帐户名称 和存储帐户密钥 ，然后单击“添加存储帐户” 。可以添加多个存储帐户，每个帐户均将显示在一个选项卡上。
4.  在“存储类型”下 ，单击“Blob” 。

    ![HDI.ASEBlob][]

5.  从“容器”中， 单击与你的 HDInsight 群集关联的容器。创建 HDInsight 群集时，你必须指定一个容器。否则，群集创建过程将为你创建一个容器。
6.  在“Blob” 下，单击“上载” 。
7.  指定要上载的文件，然后单击“打开” 。

## 使用 Hadoop 命令行将数据上载到 Blob 存储

若要使用 Hadoop 命令行，你必须先使用远程桌面连接到群集。

1.  登录到[管理门户][]。
2.  单击“HDInsight” 。这将显示已部署的 Hadoop 群集的列表。
3.  单击要往其中上载数据的 HDInsight 群集。
4.  单击页面顶部的“配置” 。
5.  如果你尚未启用远程桌面，则单击“启用远程” ，并按说明操作。否则，跳至下一步。
6.  单击页面底部的“连接” 。
7.  单击“打开” 。
8.  输入你的凭据，然后单击“确定” 。
9.  单击“是” 。
10. 在桌面上单击“Hadoop 命令行” 。
11. 下面的示例演示如何将 davinci.txt 文件从 HDInsight 头节点上的本地文件系统复制到 /example/data 目录。

        hadoop dfs -copyFromLocal C:\temp\davinci.txt /example/data/davinci.txt

    由于默认文件系统在 Azure Blob 存储上，/example/datadavinci.txt 实际是在 Azure Blob 存储上。你也可以将该文件表示为：

        wasb:///example/data/davinci.txt 

    或

        wasbs://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/example/data/davinci.txt

    在使用 *wasbs* 时，完全限定的域名是必需的。

12. 使用以下命令列出已上传的文件：

        hadoop dfs -lsr /example/data

## 使用 Sqoop 将数据从 SQL Database/SQL Server 导入到 HDFS

Sqoop 是一种为在 Hadoop 和关系数据库之间传输数据而设计的工具。可以使用此工具将数据从关系数据库管理系统 (RDBMS)（如 SQL、MySQL 或 Oracle）中导入 Hadoop 分布式文件系统 (HDFS)，在 Hadoop 中使用 MapReduce 或 Hive 转换数据，然后再将数据导回到 RDBMS。有关详细信息，请参阅 [Sqoop 用户指南][]。

在导入数据前，你必须知道 Azure SQL Database 服务器名称、数据库帐户名称、帐户密码和数据库名称。

默认情况下，可以从 Azure HDInsight 这样的 Azure 服务连接 Azure SQL 数据库。如果禁用了此防火墙设置，则必须从 Azure 管理门户启用它。有关创建 SQL 数据库和配置防火墙规则的说明，请参阅[创建和配置 SQL Database][]。

以下过程使用 PowerShell 提交 Sqoop 作业。

**使用 Sqoop 和 PowerShell 将数据导入到 HDInsight**

1.  按[安装和配置 Azure PowerShell][] 中的说明打开 Azure PowerShell 控制台窗口。
2.  在以下脚本中设置前八个变量的值：

        $subscriptionName = "<AzureSubscriptionName>"
        $clusterName = "<HDInsightClusterName>"

        $sqlDatabaseServerName = "<SQLDatabaseServerName>"
        $sqlDatabaseUserName = "<SQLDatabaseDatabaseName>"
        $sqlDatabasePassword = "<SQLDatabasePassword>"
        $sqlDatabaseDatabaseName = "<SQLDatabaseDatabaseName>"
        $tableName = "<SQLDatabaseTableName>"

        $hdfsOutputDir = "<OutputPath>"  # 这是输出文件的 HDFS 路径，例如“/lineItemData”。

        Select-AzureSubscription $subscriptionName      
        $sqoopDef = New-AzureHDInsightSqoopJobDefinition -Command "import --connect jdbc:sqlserver://$sqlDatabaseServerName.database.chinacloudapi.cn;user=$sqlDatabaseUserName@$sqlDatabaseServerName;password=$sqlDatabasePassword;database=$sqlDatabaseDatabaseName --table $tableName --target-dir $hdfsOutputDir -m 1"

        $sqoopJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $sqoopDef #-Debug -Verbose
        Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 -Job $sqoopJob

        Write-Host "Standard Error" -BackgroundColor Green
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $sqoopJob.JobId -StandardError
        Write-Host "Standard Output" -BackgroundColor Green
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $sqoopJob.JobId -StandardOutput

3.  将脚本粘贴到 Azure PowerShell 控制台窗口中以运行此脚本。请参阅 [HDInsight 入门][Azure HDInsight 入门]，以获取用于从 Azure Blob 存储检索数据文件的 PowerShell 示例。

有关使用 Sqoop 的详细信息，请参阅[将 Sqoop 与 HDInsight 配合使用][]。

## 后续步骤

现在，你已了解如何将数据导入 HDInsight，请使用以下文章了解如何执行分析：

-   [Azure HDInsight 入门][]
-   [以编程方式提交 Hadoop 作业][]
-   [Hive 与 HDInsight 配合使用][]
-   [Pig 与 HDInsight 配合使用][]

  [将 Azure Blob 存储与 HDInsight 配合使用]: ../hdinsight-use-blob-storage/
  [AzCopy]: http://blogs.msdn.com/b/windowsazurestorage/archive/2012/12/03/azcopy-uploading-downloading-files-for-windows-azure-blobs.aspx
  [Azure PowerShell]: http://msdn.microsoft.com/zh-cn/library/azure/jj152841.aspx
  [Azure .NET 存储客户端库]: /en-us/develop/net/how-to-guides/blob-storage/
  [Azure 存储空间资源管理器]: http://azurestorageexplorer.codeplex.com/
  [Cloud Storage Studio 2]: http://www.cerebrata.com/Products/CloudStorageStudio/
  [CloudXplorer]: http://clumsyleaf.com/products/cloudxplorer
  [Azure 资源管理器]: http://www.cloudberrylab.com/free-microsoft-azure-explorer.aspx
  [Azure 资源管理器专业版]: http://www.cloudberrylab.com/microsoft-azure-explorer-pro.aspx
  [Azure HDInsight 入门]: ../hdinsight-get-started/
  [设置 HDInsight 群集]: ../hdinsight-provision-clusters/
  [使用 AzCopy 将数据上传到 Blob 存储空间]: #azcopy
  [使用 Azure PowerShell 将数据上载到 Blob 存储]: #powershell
  [使用 Azure 存储空间资源管理器将数据上传到 Blob 存储空间]: #storageexplorer
  [使用 Hadoop 命令行将数据上传到 Blob 存储空间]: #commandline
  [使用 Sqoop 从 Azure SQL Database 将数据导入到 Blob 存储]: #sqoop
  [下载 AzCopy]: http://aka.ms/WaCopy
  [安装和配置 Azure PowerShell]: ../install-configure-powershell/
  []: http://azurestorageexplorer.codeplex.com/ "Azure 存储资源管理器"
  [管理存储帐户]: ../storage-manage-storage-account/
  [HDI.AzureStorageExplorer]: ./media/hdinsight-upload-data/HDI.AzureStorageExplorer.png
  [HDI.ASEAddAccount]: ./media/hdinsight-upload-data/HDI.ASEAddAccount.png
  [HDI.ASEBlob]: ./media/hdinsight-upload-data/HDI.ASEBlob.png
  [管理门户]: https://manage.windowsazure.cn
  [Sqoop 用户指南]: http://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html
  [创建和配置 SQL Database]: ../sql-database-create-configure/
  [将 Sqoop 与 HDInsight 配合使用]: ../hdinsight-use-sqoop/
  [以编程方式提交 Hadoop 作业]: ../hdinsight-submit-hadoop-jobs-programmatically/
  [Hive 与 HDInsight 配合使用]: ../hdinsight-use-hive/
  [Pig 与 HDInsight 配合使用]: ../hdinsight-use-pig
