<properties
    pageTitle="使用 HDInsight 中的 Hadoop 分析航班延误数据 | Azure"
    description="了解如何使用单个 Windows PowerShell 脚本来创建 HDInsight 群集、运行 Hive 作业、运行 Sqoop 作业和删除群集。"
    services="hdinsight"
    documentationcenter=""
    author="mumian"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="00e26aa9-82fb-4dbe-b87d-ffe8e39a5412"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/06/2017"
    wacn.date="03/24/2017"
    ms.author="jgao" />  


# 使用 HDInsight 中的 Hive 分析航班延误数据

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

Hive 提供了通过类似 SQL 的脚本语言（称为 *[HiveQL][hadoop-hiveql]*）运行 Hadoop MapReduce 作业的方法，此方法可用于对大量数据进行汇总、查询和分析。

> [AZURE.IMPORTANT]
本文档中的步骤要求使用基于 Windows 的 HDInsight 群集。Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。有关适用于基于 Linux 的群集的步骤，请参阅[使用 HDInsight (Linux) 中的 Hive 分析航班延误数据](/documentation/articles/hdinsight-analyze-flight-delay-data-linux/)。

Azure HDInsight 的主要优势之一就是隔离数据存储和计算。HDInsight 将 Azure Blob 存储用于数据存储。典型的作业包含三部分：

1. **将数据存储在 Azure Blob 存储中。** 例如，将天气数据、传感器数据、Web 日志以及此示例中的航班延误数据保存到 Azure Blob 存储中。
2. **运行作业。** 该处理数据时，可运行 Windows PowerShell 脚本（或客户端应用程序）以创建 HDInsight 群集、运行作业，然后删除该群集。作业将输出数据保存到 Azure Blob 存储。甚至在删除该群集后，输出数据也会保留。这样，你仅为已使用的内容付费。
3. **从 Azure Blob 存储检索输出**，或在此教程中将数据导出到 Azure SQL 数据库。

下图演示了本教程的方案和结构：

![HDI.FlightDelays.flow][img-hdi-flightdelays-flow]  


请注意，图中的编号对应于章节标题。**M** 代表主进程。**A** 代表附录中的内容。

本教程的主要部分说明如何使用单个 Windows PowerShell 脚本执行以下任务：

* 创建 HDInsight 群集。
* 在群集上运行 Hive 作业，以计算机场的平均延迟。航班延误数据会存储在 Azure Blob 存储帐户中。
* 运行 Sqoop 作业将 Hive 作业输出导出至 Azure SQL 数据库。
* 删除 HDInsight 群集。

在附录中，你可以找到有关上载航班延误数据、创建/上载 Hive 查询字符串和针对 Sqoop 作业准备 Azure SQL 数据库的说明。

> [AZURE.NOTE]
本文档中的步骤特定于基于 Windows 的 HDInsight 群集。有关适用于基于 Linux 的群集的步骤，请参阅[使用 HDInsight (Linux) 中的 Hive 分析航班延误数据](/documentation/articles/hdinsight-analyze-flight-delay-data-linux/)

### <a id="prerequisite"></a>先决条件
要阅读本教程，必须具备以下项：

* **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* **配备 Azure PowerShell 的工作站**。

    > [AZURE.IMPORTANT]
    Azure PowerShell 对于使用 Azure Service Manager 管理 HDInsight 资源的支持已**弃用**，将于 2017 年 1 月 1 日删除。本文档中的步骤使用的是与 Azure Resource Manager 兼容的新 HDInsight cmdlet。
    ><p>
    > 请按照 [Install and configure Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)（安装和配置 Azure PowerShell）中的步骤安装最新版本的 Azure PowerShell。如果你的脚本需要修改才能使用与 Azure Resource Manager 兼容的新 cmdlet，请参阅[迁移到适用于 HDInsight 群集的基于 Azure Resource Manager 的开发工具](/documentation/articles/hdinsight-hadoop-development-using-azure-resource-manager/)，了解详细信息。

**本教程中使用的文件**

本教程将使用来自[美国研究与技术创新管理部门 - 运输统计局或 RITA][rita-website] 的航班准时表现数据。数据的副本已上传至具有公共 Blob 访问权限的 Azure Blob 存储容器。PowerShell 脚本的一部分将数据从公共 blob 容器复制到你的群集的默认 blob 容器。HiveQL 脚本也会复制到同一 Blob 容器。如果想要了解如何将数据获取/上载到你自己的存储帐户，以及如何创建/上载 HiveQL 脚本文件，请参阅[附录 A](#appendix-a) 和[附录 B](#appendix-b)。

下表列出了本教程中使用的文件：

<table border="1">
<tr><th>文件</th><th>说明</th></tr>
<tr><td>wasbs://flightdelay@hditutorialdata.blob.core.windows.net/flightdelays.hql</td><td>Hive 作业所用的 HiveQL 脚本文件。此脚本已上传到具有公共访问权限的 Azure Blob 存储帐户。<a href="#appendix-b">附录 B</a> 提供了有关准备此文件以及将其上传到你自己的 Azure Blob 存储帐户的说明。</td></tr>
<tr><td>wasbs://flightdelay@hditutorialdata.blob.core.windows.net/2013Data</td><td>Hive 作业的输入的数据。这些数据已上传到具有公共访问权限的 Azure Blob 存储帐户。<a href="#appendix-a">附录 A</a> 提供了有关获取数据以及将数据上传到你自己的 Azure Blob 存储帐户的说明。</td></tr>
<tr><td>\tutorials\flightdelays\output</td><td>Hive 作业的输出路径。默认容器用于存储输出数据。</td></tr>
<tr><td>\tutorials\flightdelays\jobstatus</td><td>默认容器上的 Hive 作业状态文件夹。</td></tr>
</table>

## <a name="runjob"></a> 创建群集并运行 Hive/Sqoop 作业
Hadoop MapReduce 属于批处理。运行 Hive 作业时，最具成本效益的方法是为作业创建群集，并在作业完成之后删除作业。以下脚本覆盖了整个过程。有关创建 HDInsight 群集和运行 Hive 作业的详细信息，请参阅[在 HDInsight 中创建 Hadoop 群集][hdinsight-provision]和[将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]。

**使用 Azure PowerShell 运行 Hive 查询**

1. 按照[附录 C](#appendix-c) 中的说明，为 Sqoop 作业输出创建 Azure SQL 数据库和表。
2. 打开 Windows PowerShell ISE 并运行以下脚本：

        $subscriptionID = "<Azure Subscription ID>"
        $nameToken = "<Enter an Alias>"

        ###########################################
        # You must configure the follwing variables
        # for an existing Azure SQL Database
        ###########################################
        $existingSqlDatabaseServerName = "<Azure SQL Database Server>"
        $existingSqlDatabaseLogin = "<Azure SQL Database Server Login>"
        $existingSqlDatabasePassword = "<Azure SQL Database Server login password>"
        $existingSqlDatabaseName = "<Azure SQL Database name>"

        $localFolder = "E:\Tutorials\Downloads" # A temp location for copying files.
        $azcopyPath = "C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy" # depends on the version, the folder can be different

        ###########################################
        # (Optional) configure the following variables
        ###########################################

        $namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")

        $resourceGroupName = $namePrefix + "rg"
        $location = "CHINA EAST"

        $HDInsightClusterName = $namePrefix + "hdi"
        $httpUserName = "admin"
        $httpPassword = "<Enter the Password>"

        $defaultStorageAccountName = $namePrefix + "store"
        $defaultBlobContainerName = $HDInsightClusterName # use the cluster name

        $existingSqlDatabaseTableName = "AvgDelays"
        $sqlDatabaseConnectionString = "jdbc:sqlserver://$existingSqlDatabaseServerName.database.chinacloudapi.cn;user=$existingSqlDatabaseLogin@$existingSqlDatabaseServerName;password=$existingSqlDatabaseLogin;database=$existingSqlDatabaseName"

        $hqlScriptFile = "/tutorials/flightdelays/flightdelays.hql"

        $jobStatusFolder = "/tutorials/flightdelays/jobstatus"

        ###########################################
        # Login
        ###########################################
        try{
            $acct = Get-AzureRmSubscription
        }
        catch{
            Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        }
        Select-AzureRmSubscription -SubscriptionID $subscriptionID

        ###########################################
        # Create a new HDInsight cluster
        ###########################################

        # Create ARM group
        New-AzureRmResourceGroup -Name $resourceGroupName -Location $location

        # Create the default storage account
        New-AzureRmStorageAccount -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName -Location $location -Type Standard_LRS

        # Create the default Blob container
        $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value
        $defaultStorageAccountContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey
        New-AzureStorageContainer -Name $defaultBlobContainerName -Context $defaultStorageAccountContext

        # Create the HDInsight cluster
        $pw = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
        $httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$pw)

        New-AzureRmHDInsightCluster `
            -ResourceGroupName $resourceGroupName `
            -ClusterName $HDInsightClusterName `
            -Location $location `
            -ClusterType Hadoop `
            -OSType Windows `
            -ClusterSizeInNodes 2 `
            -HttpCredential $httpCredential `
            -DefaultStorageAccountName "$defaultStorageAccountName.blob.core.chinacloudapi.cn" `
            -DefaultStorageAccountKey $defaultStorageAccountKey `
            -DefaultStorageContainer $existingDefaultBlobContainerName

        ###########################################
        # Prepare the HiveQL script and source data
        ###########################################

        # Create the temp location
        New-Item -Path $localFolder -ItemType Directory -Force

        # Download the sample file from Azure Blob storage
        $context = New-AzureStorageContext -StorageAccountName "hditutorialdata" -Anonymous
        $blobs = Get-AzureStorageBlob -Container "flightdelay" -Context $context
        #$blobs | Get-AzureStorageBlobContent -Context $context -Destination $localFolder

        # Upload data to default container

        $azcopycmd = "cmd.exe /C '$azcopyPath\azcopy.exe' /S /Source:'$localFolder' /Dest:'https://$defaultStorageAccountName.blob.core.chinacloudapi.cn/$defaultBlobContainerName/tutorials/flightdelays' /DestKey:$defaultStorageAccountKey"

        Invoke-Expression -Command:$azcopycmd

        ###########################################
        # Submit the Hive job
        ###########################################
        Use-AzureRmHDInsightCluster -ClusterName $HDInsightClusterName -HttpCredential $httpCredential
        $response = Invoke-AzureRmHDInsightHiveJob `
                        -Files $hqlScriptFile `
                        -DefaultContainer $defaultBlobContainerName `
                        -DefaultStorageAccountName $defaultStorageAccountName `
                        -DefaultStorageAccountKey $defaultStorageAccountKey `
                        -StatusFolder $jobStatusFolder

        write-Host $response

        ###########################################
        # Submit the Sqoop job
        ###########################################
        $exportDir = "wasbs://$defaultBlobContainerName@$defaultStorageAccountName.blob.core.chinacloudapi.cn/tutorials/flightdelays/output"

        $sqoopDef = New-AzureRmHDInsightSqoopJobDefinition `
                        -Command "export --connect $sqlDatabaseConnectionString --table $sqlDatabaseTableName --export-dir $exportDir --fields-terminated-by \001 "
        $sqoopJob = Start-AzureRmHDInsightJob `
                        -ResourceGroupName $resourceGroupName `
                        -ClusterName $hdinsightClusterName `
                        -HttpCredential $httpCredential `
                        -JobDefinition $sqoopDef #-Debug -Verbose

        Wait-AzureRmHDInsightJob `
            -ResourceGroupName $resourceGroupName `
            -ClusterName $HDInsightClusterName `
            -HttpCredential $httpCredential `
            -WaitTimeoutInSeconds 3600 `
            -Job $sqoopJob.JobId

        Get-AzureRmHDInsightJobOutput `
            -ResourceGroupName $resourceGroupName `
            -ClusterName $hdinsightClusterName `
            -HttpCredential $httpCredential `
            -DefaultContainer $existingDefaultBlobContainerName `
            -DefaultStorageAccountName $defaultStorageAccountName `
            -DefaultStorageAccountKey $defaultStorageAccountKey `
            -JobId $sqoopJob.JobId `
            -DisplayOutputType StandardError

        ###########################################
        # Delete the cluster
        ###########################################
        Remove-AzureRmHDInsightCluster -ResourceGroupName $resourceGroupName -ClusterName $hdinsightClusterName

3. 连接到 SQL 数据库，并在 AvgDelays 表中按城市查看平均航班延迟：

    ![HDI.FlightDelays.AvgDelays.Dataset][image-hdi-flightdelays-avgdelays-dataset]  


- - -

## <a id="appendix-a"></a>附录 A - 将航班延迟数据上载到 Azure Blob 存储
上载数据文件和 HiveQL 脚本文件（请参阅[附录 B](#appendix-b)）需要进行规划。思路是在创建 HDInsight 群集和运行 Hive 作业之前存储数据文件和 HiveQL 文件。可以使用两个选项：

* **使用将由 HDInsight 群集用作默认文件系统的同一 Azure 存储帐户。** 由于 HDInsight 群集将具有存储帐户访问密钥，因此你无需进行任何其他更改。
* **使用与 HDInsight 群集默认文件系统不同的 Azure 存储帐户。** 如果选择了此项，你必须修改[创建 HDInsight 群集和运行 Hive/Sqoop 作业](#runjob)中的 Windows PowerShell 脚本的创建部分，以链接该存储帐户作为额外的存储帐户。有关说明，请参阅[在 HDInsight 中创建 Hadoop 群集][hdinsight-provision]。这样，HDInsight 群集就会知道存储帐户的访问密钥。

> [AZURE.NOTE]
数据文件的 WASB 路径会在 HiveQL 脚本文件中进行硬编码。必须相应地进行更新。

**下载航班数据**

1. 浏览到[美国研究与技术创新管理部门 - 运输统计局][rita-website]。
2. 在该页面上，选择以下值：

    <table border="1">
    <tr><th>名称</th><th>值</th></tr>
    <tr><td>筛选年份</td><td>2013 </td></tr>
    <tr><td>筛选期间</td><td>1 月</td></tr>
    <tr><td>字段</td><td>*Year*、*FlightDate*、*UniqueCarrier*、*Carrier*、*FlightNum*、*OriginAirportID*、*Origin*、*OriginCityName*、*OriginState*、*DestAirportID*、*Dest*、*DestCityName*、*DestState*、*DepDelayMinutes*、*ArrDelay*、*ArrDelayMinutes*、*CarrierDelay*、*WeatherDelay*、*NASDelay*、*SecurityDelay*、*LateAircraftDelay*（清除其他所有字段）</td></tr>
    </table>
3. 单击“下载”。
4. 将文件解压缩到 **C:\\Tutorials\\FlightDelay\\2013Data** 文件夹。每个文件均为 CSV 文件且大小约为 60GB。
5. 将文件重命名为其包含的数据所对应的月份的名称。例如，将包含 1 月份数据的文件命名为 *January.csv*。
6. 重复步骤 2 和步骤 5 为 2013 年中的 12 个月分别下载一个对应的文件。完成本教程至少需要一个文件。

**将航班延迟数据上载到 Azure Blob 存储**

1. 准备参数：

    <table border="1">
    <tr><th>变量名</th><th>说明</th></tr>
    <tr><td>$storageAccountName</td><td>要将数据上传到的 Azure 存储帐户。</td></tr>
    <tr><td>$blobContainerName</td><td>要将数据上传到的 Blob 容器。</td></tr>
    </table>
2. 打开 Azure PowerShell ISE。
3. 将以下脚本粘贴到脚本窗格中：

        [CmdletBinding()]
        Param(

            [Parameter(Mandatory=$True,
                        HelpMessage="Enter the Azure storage account name for creating a new HDInsight cluster. If the account doesn't exist, the script will create one.")]
            [String]$storageAccountName,

            [Parameter(Mandatory=$True,
                        HelpMessage="Enter the Azure blob container name for creating a new HDInsight cluster. If not specified, the HDInsight cluster name will be used.")]
            [String]$blobContainerName
        )

        #Region - Variables
        $localFolder = "C:\Tutorials\FlightDelay\2013Data"  # The source folder
        $destFolder = "tutorials/flightdelay/2013data"     #The blob name prefix for the files to be uploaded
        #EndRegion

        #Region - Connect to Azure subscription
        Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
        try{Get-AzureRmContext}
        catch{Login-AzureRmAccount -EnvironmentName AzureChinaCloud}
        #EndRegion

        #Region - Validate user input
        Write-Host "`nValidating the Azure Storage account and the Blob container..." -ForegroundColor Green
        # Validate the Storage account
        if (-not (Get-AzureRmStorageAccount|Where-Object{$_.StorageAccountName -eq $storageAccountName}))
        {
            Write-Host "The storage account, $storageAccountName, doesn't exist." -ForegroundColor Red
            exit
        }
        else{
            $resourceGroupName = (Get-AzureRmStorageAccount|Where-Object{$_.StorageAccountName -eq $storageAccountName}).ResourceGroupName
        }

        # Validate the container
        $storageAccountKey = (Get-AzureRmStorageAccountKey -StorageAccountName $storageAccountName -ResourceGroupName $resourceGroupName)[0].Value
        $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

        if (-not (Get-AzureStorageContainer -Context $storageContext |Where-Object{$_.Name -eq $blobContainerName}))
        {
            Write-Host "The Blob container, $blobContainerName, doesn't exist" -ForegroundColor Red
            Exit
        }
        #EngRegion

        #Region - Copy the file from local workstation to Azure Blob storage
        if (test-path -Path $localFolder)
        {
            foreach ($item in Get-ChildItem -Path $localFolder){
                $fileName = "$localFolder\$item"
                $blobName = "$destFolder/$item"

                Write-Host "Copying $fileName to $blobName" -ForegroundColor Green

                Set-AzureStorageBlobContent -File $fileName -Container $blobContainerName -Blob $blobName -Context $storageContext
            }
        }
        else
        {
            Write-Host "The source folder on the workstation doesn't exist" -ForegroundColor Red
        }

        # List the uploaded files on HDInsight
        Get-AzureStorageBlob -Container $blobContainerName  -Context $storageContext -Prefix $destFolder
        #EndRegion

4. 按 **F5** 运行脚本。

如果选择使用其他方法上传文件，请确保文件路径是 tutorials/flightdelay/data。用于访问文件的语法是：

    wasbs://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/tutorials/flightdelay/data

路径 tutorials/flightdelay/data 是上传文件时创建的虚拟文件夹。验证是否有 12 个文件，每个月对应一个文件。

> [AZURE.NOTE]
必须更新 Hive 查询，才能从新位置进行读取。
><p>
> 必须配置容器访问权限，使其成为公用，或者将存储帐户绑定到 HDInsight 群集。否则，Hive 查询字符串将无法访问数据文件。

- - -

## <a id="appendix-b"></a>附录 B - 创建并上载 HiveQL 脚本
使用 Azure PowerShell，可一次运行多个 HiveQL 语句，或者将 HiveQL 语句打包到一个脚本文件中。本部分说明如何创建 HiveQL 脚本，以及如何使用 Azure PowerShell 将脚本上传到 Azure Blob 存储。Hive 要求 HiveQL 脚本必须存储在 Azure Blob 存储中。

HiveQL 脚本将执行以下操作：

1. **删除 delays\_raw 表**（如果该表已存在）。
2. **创建 delays\_raw 外部 Hive 表**，并将该表指向航班延误文件所在的 Blob 存储位置。此查询指定用“,”分隔字段并用“\\n”终止行。这在字段值包含逗号时将导致出现问题，因为 Hive 无法区分逗号是字段分隔符还是字段值的一部分（在 ORIGIN\_CITY\_NAME 和 DEST\_CITY\_NAME 的字段值中属于此情况）。为了解决此问题，此查询将创建 TEMP 列来保存未正确拆分到列中的数据。
3. **删除 delays 表**（如果该表已存在）。
4. **创建 delays 表**。这适用于在进一步处理前清理数据。此查询将从 delays\_raw 表创建一个新表 *delays*。请注意，将不会复制 TEMP 列（如前所述），并且将使用 **substring** 函数从数据中删除引号标记。
5. **计算平均天气延迟，并按城市名对结果进行分组。** 它还会将结果输出到 Blob 存储。请注意，查询将从数据中删除撇号，并且将排除 **weather\_delay** 的值为 null 的行。由于本教程中稍后使用的 Sqoop 在默认情况下无法适当地处理这些值，因此这是必要的。

如需 HiveQL 命令的完整列表，请参阅 [Hive 数据定义语言][hadoop-hiveql]。每条 HiveQL 命令必须以分号结尾。

**创建 HiveQL 脚本文件**

1. 准备参数：

    <table border="1">
    <tr><th>变量名</th><th>说明</th></tr>
    <tr><td>$storageAccountName</td><td>要将 HiveQL 脚本上传到的 Azure 存储帐户。</td></tr>
    <tr><td>$blobContainerName</td><td>要将 HiveQL 脚本上传到的 Blob 容器。</td></tr>
    </table>
2. 打开 Azure PowerShell ISE。
3. 将以下脚本复制并粘贴到脚本窗格中：

        [CmdletBinding()]
        Param(

            # Azure Blob storage variables
            [Parameter(Mandatory=$True,
                        HelpMessage="Enter the Azure storage account name for creating a new HDInsight cluster. If the account doesn't exist, the script will create one.")]
            [String]$storageAccountName,

            [Parameter(Mandatory=$True,
                        HelpMessage="Enter the Azure blob container name for creating a new HDInsight cluster. If not specified, the HDInsight cluster name will be used.")]
            [String]$blobContainerName
        )

        #region - Define variables
        # Treat all errors as terminating
        $ErrorActionPreference = "Stop"

        # The HiveQL script file is exported as this file before it's uploaded to Blob storage
        $hqlLocalFileName = "e:\tutorials\flightdelay\flightdelays.hql"

        # The HiveQL script file will be uploaded to Blob storage as this blob name
        $hqlBlobName = "tutorials/flightdelay/flightdelays.hql"

        # These two constants are used by the HiveQL script file
        #$srcDataFolder = "tutorials/flightdelay/data"
        $dstDataFolder = "/tutorials/flightdelay/output"
        #endregion

        #Region - Connect to Azure subscription
        Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
        try{Get-AzureRmContext}
        catch{Login-AzureRmAccount -EnvironmentName AzureChinaCloud}
        #EndRegion

        #Region - Validate user input
        Write-Host "`nValidating the Azure Storage account and the Blob container..." -ForegroundColor Green
        # Validate the Storage account
        if (-not (Get-AzureRmStorageAccount|Where-Object{$_.StorageAccountName -eq $storageAccountName}))
        {
            Write-Host "The storage account, $storageAccountName, doesn't exist." -ForegroundColor Red
            exit
        }
        else{
            $resourceGroupName = (Get-AzureRmStorageAccount|Where-Object{$_.StorageAccountName -eq $storageAccountName}).ResourceGroupName
        }

        # Validate the container
        $storageAccountKey = (Get-AzureRmStorageAccountKey -StorageAccountName $storageAccountName -ResourceGroupName $resourceGroupName)[0].Value
        $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

        if (-not (Get-AzureStorageContainer -Context $storageContext |Where-Object{$_.Name -eq $blobContainerName}))
        {
            Write-Host "The Blob container, $blobContainerName, doesn't exist" -ForegroundColor Red
            Exit
        }
        #EngRegion

        #region - Validate the file and file path

        # Check if a file with the same file name already exists on the workstation
        Write-Host "`nvalidating the folder structure on the workstation for saving the HQL script file ..."  -ForegroundColor Green
        if (test-path $hqlLocalFileName){

            $isDelete = Read-Host 'The file, ' $hqlLocalFileName ', exists.  Do you want to overwirte it? (Y/N)'

            if ($isDelete.ToLower() -ne "y")
            {
                Exit
            }
        }

        # Create the folder if it doesn't exist
        $folder = split-path $hqlLocalFileName
        if (-not (test-path $folder))
        {
            Write-Host "`nCreating folder, $folder ..." -ForegroundColor Green

            new-item $folder -ItemType directory
        }
        #end region

        #region - Write the Hive script into a local file
        Write-Host "`nWriting the Hive script into a file on your workstation ..." `
                    -ForegroundColor Green

        $hqlDropDelaysRaw = "DROP TABLE delays_raw;"

        $hqlCreateDelaysRaw = "CREATE EXTERNAL TABLE delays_raw (" +
                "YEAR string, " +
                "FL_DATE string, " +
                "UNIQUE_CARRIER string, " +
                "CARRIER string, " +
                "FL_NUM string, " +
                "ORIGIN_AIRPORT_ID string, " +
                "ORIGIN string, " +
                "ORIGIN_CITY_NAME string, " +
                "ORIGIN_CITY_NAME_TEMP string, " +
                "ORIGIN_STATE_ABR string, " +
                "DEST_AIRPORT_ID string, " +
                "DEST string, " +
                "DEST_CITY_NAME string, " +
                "DEST_CITY_NAME_TEMP string, " +
                "DEST_STATE_ABR string, " +
                "DEP_DELAY_NEW float, " +
                "ARR_DELAY_NEW float, " +
                "CARRIER_DELAY float, " +
                "WEATHER_DELAY float, " +
                "NAS_DELAY float, " +
                "SECURITY_DELAY float, " +
                "LATE_AIRCRAFT_DELAY float) " +
            "ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' " +
            "LINES TERMINATED BY '\n' " +
            "STORED AS TEXTFILE " +
            "LOCATION 'wasbs://flightdelay@hditutorialdata.blob.core.windows.net/2013Data';"

        $hqlDropDelays = "DROP TABLE delays;"

        $hqlCreateDelays = "CREATE TABLE delays AS " +
            "SELECT YEAR AS year, " +
                "FL_DATE AS flight_date, " +
                "substring(UNIQUE_CARRIER, 2, length(UNIQUE_CARRIER) -1) AS unique_carrier, " +
                "substring(CARRIER, 2, length(CARRIER) -1) AS carrier, " +
                "substring(FL_NUM, 2, length(FL_NUM) -1) AS flight_num, " +
                "ORIGIN_AIRPORT_ID AS origin_airport_id, " +
                "substring(ORIGIN, 2, length(ORIGIN) -1) AS origin_airport_code, " +
                "substring(ORIGIN_CITY_NAME, 2) AS origin_city_name, " +
                "substring(ORIGIN_STATE_ABR, 2, length(ORIGIN_STATE_ABR) -1)  AS origin_state_abr, " +
                "DEST_AIRPORT_ID AS dest_airport_id, " +
                "substring(DEST, 2, length(DEST) -1) AS dest_airport_code, " +
                "substring(DEST_CITY_NAME,2) AS dest_city_name, " +
                "substring(DEST_STATE_ABR, 2, length(DEST_STATE_ABR) -1) AS dest_state_abr, " +
                "DEP_DELAY_NEW AS dep_delay_new, " +
                "ARR_DELAY_NEW AS arr_delay_new, " +
                "CARRIER_DELAY AS carrier_delay, " +
                "WEATHER_DELAY AS weather_delay, " +
                "NAS_DELAY AS nas_delay, " +
                "SECURITY_DELAY AS security_delay, " +
                "LATE_AIRCRAFT_DELAY AS late_aircraft_delay " +
            "FROM delays_raw;"

        $hqlInsertLocal = "INSERT OVERWRITE DIRECTORY '$dstDataFolder' " +
            "SELECT regexp_replace(origin_city_name, '''', ''), " +
                "avg(weather_delay) " +
            "FROM delays " +
            "WHERE weather_delay IS NOT NULL " +
            "GROUP BY origin_city_name;"

        $hqlScript = $hqlDropDelaysRaw + $hqlCreateDelaysRaw + $hqlDropDelays + $hqlCreateDelays + $hqlInsertLocal

        $hqlScript | Out-File $hqlLocalFileName -Encoding ascii -Force
        #endregion

        #region - Upload the Hive script to the default Blob container
        Write-Host "`nUploading the Hive script to the default Blob container ..." -ForegroundColor Green

        # Create a storage context object
        $storageAccountKey = (Get-AzureRmStorageAccountKey -StorageAccountName $storageAccountName -ResourceGroupName $resourceGroupName)[0].Value
        $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

        # Upload the file from local workstation to Blob storage
        Set-AzureStorageBlobContent -File $hqlLocalFileName -Container $blobContainerName -Blob $hqlBlobName -Context $destContext
        #endregion

        Write-host "`nEnd of the PowerShell script" -ForegroundColor Green

    该脚本中使用了以下变量：

    * **$hqlLocalFileName** - 该脚本会先将 HiveQL 脚本文件保存在本地，然后才上载到 Blob 存储。这是文件名。默认值是 <u>C:\\tutorials\\flightdelay\\flightdelays.hql</u>。
    * **$hqlBlobName** - 这是 Azure Blob 存储中使用的 HiveQL 脚本文件 Blob 名称。默认值是 tutorials/flightdelay/flightdelays.hql。因为文件会直接写入 Azure Blob 存储，所以 Blob 名称的开头不是“/”。如果你要从 Blob 存储访问文件，必须在文件名的开头添加“/”。
    * **$srcDataFolder** 和 **$dstDataFolder** - = "tutorials/flightdelay/data" = "tutorials/flightdelay/output"

- - -
## <a id="appendix-c"></a>附录 C - 针对 Sqoop 作业输出准备 Azure SQL 数据库
**准备 SQL 数据库（将此部分与 Sqoop 脚本合并）**

1. 准备参数：

    <table border="1">
    <tr><th>变量名</th><th>说明</th></tr>
    <tr><td>$sqlDatabaseServerName</td><td>Azure SQL 数据库服务器的名称。不输入任何值会创建新的服务器。</td></tr>
    <tr><td>$sqlDatabaseUsername</td><td>Azure SQL 数据库服务器登录名。如果 $sqlDatabaseServerName 是现有的服务器，登录名和登录密码将用来向服务器进行身份验证。否则会创建新的服务器。</td></tr>
    <tr><td>$sqlDatabasePassword</td><td>Azure SQL 数据库服务器登录密码。</td></tr>
    <tr><td>$sqlDatabaseLocation</td><td>只有在创建新的 Azure 数据库服务器时才会使用此值。</td></tr>
    <tr><td>$sqlDatabaseName</td><td>Sqoop 作业的 AvgDelays 表的 SQL 数据库。保留空白会创建名为 HDISqoop 的数据库。Sqooop 作业输出的表名称为 AvgDelays。</td></tr>
    </table>
2. 打开 Azure PowerShell ISE。
3. 将以下脚本复制并粘贴到脚本窗格中：

        [CmdletBinding()]
        Param(

            # Azure Resource group variables
            [Parameter(Mandatory=$True,
                    HelpMessage="Enter the Azure resource group name. It will be created if it doesn't exist.")]
            [String]$resourceGroupName,

            # SQL database server variables
            [Parameter(Mandatory=$True,
                    HelpMessage="Enter the Azure SQL Database Server Name. It will be created if it doesn't exist.")]
            [String]$sqlDatabaseServer,

            [Parameter(Mandatory=$True,
                    HelpMessage="Enter the Azure SQL Database admin user.")]
            [String]$sqlDatabaseLogin,

            [Parameter(Mandatory=$True,
                    HelpMessage="Enter the Azure SQL Database admin user password.")]
            [String]$sqlDatabasePassword,

            [Parameter(Mandatory=$True,
                    HelpMessage="Enter the region to create the Database in.")]
            [String]$sqlDatabaseLocation,   #For example, China North.

            # SQL database variables
            [Parameter(Mandatory=$True,
                    HelpMessage="Enter the database name. It will be created if it doesn't exist.")]
            [String]$sqlDatabaseName # specify the database name if you have one created. Otherwise use "" to have the script create one for you.
        )

        # Treat all errors as terminating
        $ErrorActionPreference = "Stop"

        #region - Constants and variables

        # IP address REST service used for retrieving external IP address and creating firewall rules
        [String]$ipAddressRestService = "http://bot.whatismyipaddress.com"
        [String]$fireWallRuleName = "FlightDelay"

        # SQL database variables
        [String]$sqlDatabaseMaxSizeGB = 10

        #SQL query string for creating AvgDelays table
        [String]$sqlDatabaseTableName = "AvgDelays"
        [String]$sqlCreateAvgDelaysTable = " CREATE TABLE [dbo].[$sqlDatabaseTableName](
                    [origin_city_name] [nvarchar](50) NOT NULL,
                    [weather_delay] float,
                CONSTRAINT [PK_$sqlDatabaseTableName] PRIMARY KEY CLUSTERED
                (
                    [origin_city_name] ASC
                )
                )"
        #endregion

        #Region - Connect to Azure subscription
        Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
        try{Get-AzureRmContext}
        catch{Login-AzureRmAccount -EnvironmentName AzureChinaCloud}
        #EndRegion

        #region - Create and validate Azure resouce group
        try{
            Get-AzureRmResourceGroup -Name $resourceGroupName
        }
        catch{
            New-AzureRmResourceGroup -Name $resourceGroupName -Location $sqlDatabaseLocation
        }

        #EndRegion

        #region - Create and validate Azure SQL database server
        try{
            Get-AzureRmSqlServer -ServerName $sqlDatabaseServer -ResourceGroupName $resourceGroupName}
        catch{
            Write-Host "`nCreating SQL Database server ..."  -ForegroundColor Green

            $sqlDatabasePW = ConvertTo-SecureString -String $sqlDatabasePassword -AsPlainText -Force
            $credential = New-Object System.Management.Automation.PSCredential($sqlDatabaseLogin,$sqlDatabasePW)

            $sqlDatabaseServer = (New-AzureRmSqlServer -ResourceGroupName $resourceGroupName -ServerName $sqlDatabaseServer -SqlAdministratorCredentials $credential -Location $sqlDatabaseLocation).ServerName
            Write-Host "`tThe new SQL database server name is $sqlDatabaseServer." -ForegroundColor Cyan

            Write-Host "`nCreating firewall rule, $fireWallRuleName ..." -ForegroundColor Green
            $workstationIPAddress = Invoke-RestMethod $ipAddressRestService
            New-AzureRmSqlServerFirewallRule -ResourceGroupName $resourceGroupName -ServerName $sqlDatabaseServer -FirewallRuleName "$fireWallRuleName-workstation" -StartIpAddress $workstationIPAddress -EndIpAddress $workstationIPAddress

            #To allow other Azure services to access the server add a firewall rule and set both the StartIpAddress and EndIpAddress to 0.0.0.0. Note that this allows Azure traffic from any Azure subscription to access the server.
            New-AzureRmSqlServerFirewallRule -ResourceGroupName $resourceGroupName -ServerName $sqlDatabaseServer -FirewallRuleName "$fireWallRuleName-Azureservices" -StartIpAddress "0.0.0.0" -EndIpAddress "0.0.0.0"
        }

        #endregion

        #region - Create and validate Azure SQL database

        try {
            Get-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $sqlDatabaseServer -DatabaseName $sqlDatabaseName
        }
        catch {
            Write-Host "`nCreating SQL Database, $sqlDatabaseName ..."  -ForegroundColor Green
            New-AzureRMSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $sqlDatabaseServer -DatabaseName $sqlDatabaseName -Edition "Standard" -RequestedServiceObjectiveName "S1"
        }

        #endregion

        #region -  Execute an SQL command to create the AvgDelays table

        Write-Host "`nCreating SQL Database table ..."  -ForegroundColor Green
        $conn = New-Object System.Data.SqlClient.SqlConnection
        $conn.ConnectionString = "Data Source=$sqlDatabaseServer.database.chinacloudapi.cn;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseLogin;Password=$sqlDatabasePassword;Encrypt=true;Trusted_Connection=false;"
        $conn.open()
        $cmd = New-Object System.Data.SqlClient.SqlCommand
        $cmd.connection = $conn
        $cmd.commandtext = $sqlCreateAvgDelaysTable
        $cmd.executenonquery()

        $conn.close()

        Write-host "`nEnd of the PowerShell script" -ForegroundColor Green

    > [AZURE.NOTE]
    该脚本使用具象状态传输 (REST) 服务 http://bot.whatismyipaddress.com 来检索外部 IP 地址。IP 地址用于创建 SQL 数据库服务器的防火墙规则。

    该脚本中使用的某些变量：

    * **$ipAddressRestService** - 默认值为 http://bot.whatismyipaddress.com。这是用来获取外部 IP 地址的公共 IP 地址 REST 服务。可根据需要使用其他服务。使用此服务检索的外部 IP 地址将用于创建 Azure SQL 数据库服务器的防火墙规则，使你能够从工作站访问数据库（通过 Windows PowerShell 脚本）。
    * **$fireWallRuleName** - 这是 Azure SQL 数据库服务器的防火墙规则名称。默认名称为 <u>FlightDelay</u>。可根据需要对其进行重命名。
    * **$sqlDatabaseMaxSizeGB** - 只有在创建新的 Azure SQL 数据库服务器时才会使用此值。默认值为 10GB。10GB 对于本教程来说已足够。
    * **$sqlDatabaseName** - 只有在创建新的 Azure SQL 数据库时才会使用此值。默认值为 HDISqoop。如果将它重命名，则必须相应地更新 Sqoop Windows PowerShell 脚本。
4. 按 **F5** 运行脚本。
5. 验证脚本输出。确保已成功运行脚本。

## <a id="nextsteps"></a>后续步骤
现在你已了解如何执行以下操作：将文件上传到 Azure Blob 存储、使用 Azure Blob 存储中的数据填充 Hive 表、运行 Hive 查询以及使用 Sqoop 将数据从 HDFS 导出到 Azure SQL 数据库。若要了解更多信息，请参阅下列文章：

* [HDInsight 入门][hdinsight-get-started]
* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [将 Oozie 与 HDInsight 配合使用][hdinsight-use-oozie]
* [将 Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
* [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-mapreduce]

[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/

[rita-website]: http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time
[powershell-install-configure]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs

[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/
[hdinsight-develop-mapreduce]: /documentation/articles/hdinsight-develop-deploy-java-mapreduce-linux/

[hadoop-hiveql]: https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL
[hadoop-shell-commands]: http://hadoop.apache.org/docs/r0.18.3/hdfs_shell.html

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx

[image-hdi-flightdelays-avgdelays-dataset]: ./media/hdinsight-analyze-flight-delay-data/HDI.FlightDelays.AvgDelays.DataSet.png
[img-hdi-flightdelays-run-hive-job-output]: ./media/hdinsight-analyze-flight-delay-data/HDI.FlightDelays.RunHiveJob.Output.png
[img-hdi-flightdelays-flow]: ./media/hdinsight-analyze-flight-delay-data/HDI.FlightDelays.Flow.png

<!---HONumber=Mooncake_0320_2017-->