<properties
	pageTitle="使用 HDInsight 中的 Hadoop 分析航班延误数据 | Azure"
	description="了解如何使用一个 Windows PowerShell 脚本来创建 HDInsight 群集、运行 Hive 作业、运行 Sqoop 作业和删除群集。"
	services="hdinsight"
	documentationCenter=""
	authors="mumian"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="03/04/2016"
	wacn.date="04/26/2016"/>

#使用 HDInsight 中的 Hive 分析航班延误数据

Hive 提供了通过类似 SQL 的脚本语言（称为 *[HiveQL][hadoop-hiveql]*）运行 Hadoop MapReduce 作业的方法，此方法可用于对大量数据进行汇总、查询和分析。

Azure HDInsight 的主要优势之一就是隔离数据存储和计算。HDInsight 将 Azure Blob 存储用于数据存储。典型的作业包含 3 部分：

1. **将数据存储在 Azure Blob 存储中。** 这可能是一个连续的过程。例如，将天气数据、传感器数据、Web 日志以及此示例中的航班延误数据保存到 Azure Blob 存储中。
2. **运行作业。** 该处理数据时，你可以运行 Windows PowerShell 脚本（或客户端应用程序）以创建 HDInsight 群集、运行作业，然后删除该群集。作业将输出数据保存到 Azure Blob 存储。甚至在删除该群集后，输出数据也会保留。这样，你仅为已使用的内容付费。
3. **从 Azure Blob 存储检索输出**，或在此教程中将数据导出到 Azure SQL 数据库。

下图演示了本教程的方案和结构：

![HDI.FlightDelays.flow][img-hdi-flightdelays-flow]

**注意**：图中的编号对应于章节标题。**M** 代表主进程。**A** 代表附录中的内容。

教程的主要部分说明如何使用一个 PowerShell 脚本来执行以下操作：

- 创建 HDInsight 群集。
- 在群集上运行 Hive 作业，以计算机场的平均延迟。航班延误数据会存储在 Azure Blob 存储帐户中。
- 运行 Sqoop 作业将 Hive 作业输出导出至 Azure SQL 数据库。
- 删除 HDInsight 群集。

在附录中，你可以找到有关上载航班延误数据、创建/上载 Hive 查询字符串和针对 Sqoop 作业准备 Azure SQL 数据库的说明。

###<a id="prerequisite"></a> 先决条件

在开始阅读本教程前，你必须具有：

- **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。

- **配备 Azure PowerShell 的工作站**。

	[AZURE.INCLUDE [upgrade-powershell](../includes/hdinsight-use-latest-powershell.md)]

**本教程中使用的文件**

本教程将使用来自[美国研究与技术创新管理部门 - 运输统计局或 RITA][rita-website] 的航班准时表现数据。数据的副本已上载至具有公共 Blob 访问权限的 Azure Blob 存储容器。PowerShell 脚本的一部分将数据从公共 blob 容器复制到你的群集的默认 blob 容器。HiveQL 脚本也会复制到同一 Blob 容器。如果想要了解如何将数据获取/上载到你自己的存储帐户，以及如何创建/上载 HiveQL 脚本文件，请参阅[附录 A](#appendix-a) 和[附录 B](#appendix-b)。

下表列出了本教程中使用的文件：

<table border="1">
<tr><th>文件</th><th>说明</th></tr>
<tr><td>wasb://flightdelay@hditutorialdata.blob.core.windows.net/flightdelays.hql</td><td>你要运行的 Hive 作业所用的 HiveQL 脚本文件。此脚本已上载到具有公共访问权限的 Azure Blob 存储帐户。<a href="#appendix-b">附录 B</a> 提供了有关准备此文件以及将其上载到你自己的 Azure Blob 存储帐户的说明。</td></tr>
<tr><td>wasb://flightdelay@hditutorialdata.blob.core.windows.net/2013Data</td><td>Hive 作业的输入的数据。这些数据已上载到具有公共访问权限的 Azure Blob 存储帐户。<a href="#appendix-a">附录 A</a> 提供了有关获取数据以及将数据上载到你自己的 Azure Blob 存储帐户的说明。</td></tr>
<tr><td>\tutorials\flightdelays\output</td><td>Hive 作业的输出路径。默认容器用于存储输出数据。</td></tr>
<tr><td>\tutorials\flightdelays\jobstatus</td><td>默认容器上的 Hive 作业状态文件夹。</td></tr>
</table>


##<a name="runjob"></a> 创建群集并运行 Hive/Sqoop 作业

Hadoop MapReduce 属于批处理。运行 Hive 作业时，最具成本效益的方法是为作业创建群集，并在作业完成之后删除作业。以下脚本覆盖了整个过程。有关创建 HDInsight 群集和运行 Hive 作业的详细信息，请参阅[在 HDInsight 中创建 Hadoop 群集][hdinsight-provision]和[将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]。

**使用 Azure PowerShell 运行 Hive 查询**

1. 按照[附录 C](#appendix-c) 中的说明，为 Sqoop 作业输出创建 Azure SQL 数据库和表。
2. 准备参数：

	<table border="1">
<tr><th>变量名</th><th>说明</th></tr>
<tr><td>$hdinsightClusterName</td><td>HDInsight 群集名称。如果群集不存在，脚本会使用输入的名称创建一个群集。</td></tr>
<tr><td>$storageAccountName</td><td>将用作默认存储帐户的 Azure 存储帐户。只有在脚本需要创建 HDInsight 群集时才需要此值。如果你已为 $hdinsightClusterName 指定了现有的 HDInsight 群集名称，请保留空白。如果具有输入值的存储帐户不存在，则脚本会使用该名称创建帐户。</td></tr>
<tr><td>$blobContainerName</td><td>将用于默认文件系统的 Blob 容器。如果你将它保留空白，将使用 $hdinsightClusterName 值。</td></tr>
<tr><td>$sqlDatabaseServerName</td><td>Azure SQL 数据库服务器名称。必须是现有的服务器。有关创建服务器的信息，请参阅<a href="#appendix-c">附录 C</a>。</td></tr>
<tr><td>$sqlDatabaseUsername</td><td>Azure SQL 数据库服务器登录名。</td></tr>
<tr><td>$sqlDatabasePassword</td><td>Azure SQL 数据库服务器登录密码。</td></tr>
<tr><td>$sqlDatabaseName</td><td>Sqoop 将数据导出到的 SQL 数据库。默认名称是 HDISqoop。Sqooop 作业输出的表名称为 AvgDelays。</td></tr>
</table>
3. 打开 Windows PowerShell 集成脚本环境 (ISE)。
4. 将以下脚本复制并粘贴到脚本窗格中：

		[CmdletBinding()]
		Param(

		    # HDInsight cluster variables
		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the HDInsight cluster name. If the cluster doesn't exist, the script will create one.")]
		    [String]$hdinsightClusterName,

		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the Azure storage account name for creating a new HDInsight cluster. If the account doesn't exist, the script will create one.")]
		    [AllowEmptyString()]
		    [String]$storageAccountName,

		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the Azure blob container name for creating a new HDInsight cluster. If not specified, the HDInsight cluster name will be used.")]
		    [AllowEmptyString()]
		    [String]$blobContainerName,

		    #SQL database server variables
		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the Azure SQL Database Server Name where to export data.")]
		    [String]$sqlDatabaseServerName,  # specify the Azure SQL database server name where you want to export data to.

		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the Azure SQL Database login username.")]
		    [String]$sqlDatabaseUsername,

		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the Azure SQL Database login user password.")]
		    [String]$sqlDatabasePassword,

		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the database name where data will be exported to.")]
		    [String]$sqlDatabaseName  # the default value is HDISqoop
		)

		# Treat all errors as terminating
		$ErrorActionPreference = "Stop"

		#region - HDInsight cluster variables
		[int]$clusterSize = 1                # One data node is sufficient for this tutorial
		[String]$location = "China North"     # For better performance, choose a datacenter near you
		[String]$hadoopUserLogin = "admin"   # Use "admin" as the Hadoop login name
		[String]$hadoopUserpw = "Pass@word1" # Use "Pass@word1" as the Hadoop login password

		[Bool]$isNewCluster = $false      # Indicates whether a new HDInsight cluster is created by the script  
		                                  # If this variable is true, then the script can optionally delete the cluster after running the Hive and Sqoop jobs

		[Bool]$isNewStorageAccount = $false

		$storageAccountName = $storageAccountName.ToLower() # Storage account names must be between 3 and 24 characters in length and use numbers and lower-case letters only.
		#endregion

		#region - Hive job variables
		[String]$hqlScriptFile = "wasb://flightdelay@hditutorialdata.blob.core.windows.net/flightdelays.hql" # The HiveQL script is located in a public Blob container. Update this URI if you want to use your own script file.

		[String]$jobStatusFolder = "/tutorials/flightdelays/jobstatus" # The script saves both the output data and the job status file to the default container.
		                                                               # The output data path is set in the HiveQL file.

		#[String]$jobOutputBlobName = "tutorials/flightdelays/output/000000_0" # This is the output file of the Hive job. The path is set in the HiveQL script.
		#endregion

		#region - Sqoop job variables
		[String]$sqlDatabaseTableName = "AvgDelays"
		[String]$sqlDatabaseConnectionString = "jdbc:sqlserver://$sqlDatabaseServerName.database.chinacloudapi.cn;user=$sqlDatabaseUserName@$sqlDatabaseServerName;password=$sqlDatabasePassword;database=$sqlDatabaseName"
		#endregion Constants and variables

		#region - Connect to Azure subscription
		Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
		Write-Host "`tCurrent system time: " (get-date) -ForegroundColor Yellow
		if (-not (Get-AzureAccount)){ Add-AzureAccount -Environment AzureChinaCloud}
		#endregion

		#region - Validate user input, and provision HDInsight cluster if needed
		Write-Host "`nValidating user input ..." -ForegroundColor Green

		# Both the Azure SQL database server and database must exist
		if (-not (Get-AzureSqlDatabaseServer|Where-Object{$_.ServerName -eq $sqlDatabaseServerName})){
		    Write-host "The Azure SQL database server, $sqlDatabaseServerName doesn't exist." -ForegroundColor Red
		    Exit
		}
		else
		{
		    if (-not ((Get-AzureSqlDatabase -ServerName $sqlDatabaseServerName)|Where-Object{$_.Name -eq $sqlDatabaseName})){
		        Write-host "The Azure SQL database, $sqlDatabaseName doesn't exist." -ForegroundColor Red
		        Exit
		    }
		}

		if (Test-AzureName -Service -Name $hdinsightClusterName)     # If it is an existing HDInsight cluster ...
		{
		    Write-Host "`tThe HDInsight cluster, $hdinsightClusterName, exists. This cluster will be used to run the Hive job." -ForegroundColor Cyan

		    #region - Retrieve the default Storage account/container names if the cluster exists
		    # The Hive job output will be stored in the default container. The
		    # information is used to download a copy of the output file from
		    # Blob storage to workstation for the validation purpose.
		    Write-Host "`nRetrieving the HDInsight cluster default storage account information ..." `
		                -ForegroundColor Green

		    $hdi = Get-AzureHDInsightCluster -Name $HDInsightClusterName

		    # Use the default Storage account and the default container even if the names are different from the user input
		    $storageAccountName = $hdi.DefaultStorageAccount.StorageAccountName `
		                            -replace ".blob.core.chinacloudapi.cn"
		    $blobContainerName = $hdi.DefaultStorageAccount.StorageContainerName

		    Write-Host "`tThe default storage account for the cluster is $storageAccountName." `
		                -ForegroundColor Cyan
		    Write-Host "`tThe default Blob container for the cluster is $blobContainerName." `
		                -ForegroundColor Cyan
		    #endregion
		}
		else     #If the cluster doesn't exist, a new one will be provisioned
		{
		    if ([string]::IsNullOrEmpty($storageAccountName))
		    {
		        Write-Host "You must provide a storage account name" -ForegroundColor Red
		        EXit
		    }
		    else
		    {
		        # If the container name is not specified, use the cluster name as the container name
		        if ([string]::IsNullOrEmpty($blobContainerName))
		        {
		            $blobContainerName = $hdinsightClusterName
		        }
		        $blobContainerName = $blobContainerName.ToLower()

		        #region - Provision HDInsight cluster
		        # Create an Azure Storage account if it doesn't exist
		        if (-not (Get-AzureStorageAccount|Where-Object{$_.Label -eq $storageAccountName}))
		        {
		            Write-Host "`nCreating the Azure storage account, $storageAccountName ..." -ForegroundColor Green
		            if (-not (New-AzureStorageAccount -StorageAccountName $storageAccountName.ToLower() -Location $location)){
		                Write-Host "Error creating the storage account, $storageAccountName" -ForegroundColor Red
		                Exit
		            }
		            $isNewStorageAccount = $True
		        }

		        # Create a Blob container used as the default container
		        $storageAccountKey = get-azurestoragekey -StorageAccountName $storageAccountName | %{$_.Primary}
		        $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

		        if (-not (Get-AzureStorageContainer -Context $storageContext |Where-Object{$_.Name -eq $blobContainerName}))
		        {
		            Write-Host "`nCreating the Azure Blob container, $blobContainerName ..." -ForegroundColor Green
		            if (-not (New-AzureStorageContainer -name $blobContainerName -Context $storageContext)){
		                Write-Host "Error creating the Blob container, $blobContainerName" -ForegroundColor Red
		                Exit
		            }
		        }

		        # Create a new HDInsight cluster
		        Write-Host "`nProvisioning the HDInsight cluster, $hdinsightClusterName ..." -ForegroundColor Green
		        Write-Host "`tCurrent system time: " (get-date) -ForegroundColor Yellow
		        $hadoopUserPassword = ConvertTo-SecureString -String $hadoopUserpw -AsPlainText -Force
		        $credential = New-Object System.Management.Automation.PSCredential($hadoopUserLogin,$hadoopUserPassword)
		        if (-not $credential)
		        {
		            Write-Host "Error creating the PSCredential object" -ForegroundColor Red
		            Exit
		        }

		        if (-not (New-AzureHDInsightCluster -Name $hdinsightClusterName -Location $location -Credential $credential -DefaultStorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainerName $blobContainerName -ClusterSizeInNodes $clusterSize)){
		            Write-Host "Error provisioning the cluster, $hdinsightClusterName." -ForegroundColor Red
		            Exit
		        }
		        Else
		        {
		            $isNewCluster = $True
		        }
		        #endregion
		    }
		}
		#endregion

		#region - Submit Hive job
		Write-Host "`nSubmitting the Hive job ..." -ForegroundColor Green
		Write-Host "`tCurrent system time: " (get-date) -ForegroundColor Yellow

		Use-AzureHDInsightCluster $HDInsightClusterName
		$response = Invoke-Hive –File $hqlScriptFile -StatusFolder $jobStatusFolder

		Write-Host "`nThe Hive job status" -ForegroundColor Cyan
		Write-Host "---------------------------------------------------------" -ForegroundColor Cyan
		write-Host $response
		Write-Host "---------------------------------------------------------" -ForegroundColor Cyan
		#endregion

		#region - Run Sqoop job
		Write-Host "`nSubmitting the Sqoop job ..." -ForegroundColor Green
		Write-Host "`tCurrent system time: " (get-date) -ForegroundColor Yellow

		[String]$exportDir = "wasb://$blobContainerName@$storageAccountName.blob.core.chinacloudapi.cn/tutorials/flightdelays/output"


		$sqoopDef = New-AzureHDInsightSqoopJobDefinition -Command "export --connect $sqlDatabaseConnectionString --table $sqlDatabaseTableName --export-dir $exportDir --fields-terminated-by \001 "
		$sqoopJob = Start-AzureHDInsightJob -Cluster $hdinsightClusterName -JobDefinition $sqoopDef #-Debug -Verbose
		Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 -Job $sqoopJob

		Write-Host "Standard Error" -BackgroundColor Green
		Get-AzureHDInsightJobOutput -Cluster $hdinsightClusterName -JobId $sqoopJob.JobId -StandardError
		Write-Host "Standard Output" -BackgroundColor Green
		Get-AzureHDInsightJobOutput -Cluster $hdinsightClusterName -JobId $sqoopJob.JobId -StandardOutput
		#endregion

		#region - Delete the HDInsight cluster
		if ($isNewCluster -eq $True)
		{
		    $isDelete = Read-Host 'Do you want to delete the HDInsight Hadoop cluster ' $hdinsightClusterName '? (Y/N)'

		    if ($isDelete.ToLower() -eq "y")
		    {
		        Write-Host "`nDeleting the HDInsight cluster ..." -ForegroundColor Green
		        Write-Host "`tCurrent system time: " (get-date) -ForegroundColor Yellow
		        Remove-AzureHDInsightCluster -Name $hdinsightClusterName
		    }
		}
		#endregion

		#region - Delete the Storage account
		if ($isNewStorageAccount -eq $True)
		{
		    $isDelete = Read-Host 'Do you want to delete the Azure storage account ' $storageAccountName '? (Y/N)'

		    if ($isDelete.ToLower() -eq "y")
		    {
		        Write-Host "`nDeleting the Azure storage account ..." -ForegroundColor Green
		        Write-Host "`tCurrent system time: " (get-date) -ForegroundColor Yellow
		        Remove-AzureStorageAccount -StorageAccountName $storageAccountName
		    }
		}
		#endregion

		Write-Host "End of the PowerShell script" -ForegroundColor Green
		Write-Host "`tCurrent system time: " (get-date) -ForegroundColor Yellow

5. 按 **F5** 运行脚本。输出应如下所示：

	![HDI.FlightDelays.RunHiveJob.output][img-hdi-flightdelays-run-hive-job-output]

6. 连接到 SQL 数据库，并在 AvgDelays 表中按城市查看平均航班延迟：

	![HDI.FlightDelays.AvgDelays.Dataset][image-hdi-flightdelays-avgdelays-dataset]



---
##<a id="appendix-a"></a>附录 A - 将航班延误数据上载到 Azure Blob 存储
上载数据文件和 HiveQL 脚本文件（请参阅[附录 B](#appendix-b)）需要进行规划。思路是在创建 HDInsight 群集和运行 Hive 作业之前存储数据文件和 HiveQL 文件。可以使用两个选项：

- **使用将由 HDInsight 群集用作默认文件系统的同一 Azure 存储帐户。** 由于 HDInsight 群集将具有存储帐户访问密钥，因此你无需进行任何其他更改。
- **使用与 HDInsight 群集默认文件系统不同的 Azure 存储帐户。** 如果选择了此项，你必须修改[创建 HDInsight 群集和运行 Hive/Sqoop 作业](#runjob)中的 Windows PowerShell 脚本的创建部分，以链接该存储帐户作为额外的存储帐户。有关说明，请参阅[在 HDInsight 中创建 Hadoop 群集][hdinsight-provision]。这样，HDInsight 群集就会知道存储帐户的访问密钥。

>[AZURE.NOTE]数据文件的 WASB 路径会在 HiveQL 脚本文件中进行硬编码。你必须相应地更新该路径。

**下载航班数据**

1. 浏览到[美国研究与技术创新管理部门 - 运输统计局][rita-website]。
2. 在该页面上，选择以下值：

	<table border="1">
	<tr><th>Name</th><th>值</th></tr>
	<tr><td>筛选年份</td><td>2013 </td></tr>
	<tr><td>筛选期间</td><td>1 月</td></tr>
	<tr><td>字段</td><td>*Year*、*FlightDate*、*UniqueCarrier*、*Carrier*、*FlightNum*、*OriginAirportID*、*Origin*、*OriginCityName*、*OriginState*、*DestAirportID*、*Dest*、*DestCityName*、*DestState*、*DepDelayMinutes*、*ArrDelay*、*ArrDelayMinutes*、*CarrierDelay*、*WeatherDelay*、*NASDelay*、*SecurityDelay*、*LateAircraftDelay*（清除其他所有字段）</td></tr>
	</table>

3. 单击“下载”。
4. 将文件解压缩到 **C:\\Tutorials\\FlightDelays\\Data** 文件夹。每个文件均为 CSV 文件且大小约为 60GB。
5.	将文件重命名为其包含的数据所对应的月份的名称。例如，将包含 1 月份数据的文件命名为 *January.csv*。
6. 重复步骤 2 和步骤 5 为 2013 年中的 12 个月分别下载一个对应的文件。完成本教程到少要有一个文件。  

**将航班延迟数据上载到 Azure Blob 存储**

1. 准备参数：

	<table border="1">
	<tr><th>变量名</th><th>说明</th></tr>
	<tr><td>$storageAccountName</td><td>要将数据上载到的 Azure 存储帐户。</td></tr>
	<tr><td>$blobContainerName</td><td>要将数据上载到的 Blob 容器。</td></tr>
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
		$localFolder = "C:\Tutorials\FlightDelays\Data"  # The source folder
		$destFolder = "tutorials/flightdelays/data"     #The blob name prefix for the files to be uploaded
		#EndRegion

		#Region - Connect to Azure subscription
		Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
		if (-not (Get-AzureAccount)){ Add-AzureAccount -Environment AzureChinaCloud}
		#EndRegion

		#Region - Validate user input
		# Validate the Storage account
		if (-not (Get-AzureStorageAccount|Where-Object{$_.Label -eq $storageAccountName}))
		{
		    Write-Host "The storage account, $storageAccountName, doesn't exist." -ForegroundColor Red
		    exit
		}

		# Validate the container
		$storageAccountKey = get-azurestoragekey -StorageAccountName $storageAccountName | %{$_.Primary}
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

如果你选择使用其他方法上载文件，请确保文件路径是 tutorials/flightdelay/data。用于访问文件的语法是：

	wasb://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/tutorials/flightdelay/data

路径 tutorials/flightdelay/data 是你在上载文件时创建的虚拟文件夹。验证是否有 12 个文件，每个月对应一个文件。

>[AZURE.NOTE]你必须更新 Hive 查询，以从新位置读取。

> 你必须配置容器访问权限，使其成为公用，或者将存储帐户绑定到 HDInsight 群集。否则，Hive 查询字符串将无法访问数据文件。

---
##<a id="appendix-b"></a>附录 B - 创建并上载 HiveQL 脚本

使用 Azure PowerShell，你可以一次运行多个 HiveQL 语句，或者将 HiveQL 语句打包到一个脚本文件中。本部分说明如何创建 HiveQL 脚本，以及使用 Azure PowerShell 将脚本上载到 Azure Blob 存储。Hive 要求 HiveQL 脚本必须存储在 Azure Blob 存储中。

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
	<tr><td>$storageAccountName</td><td>要将 HiveQL 脚本上载到的 Azure 存储帐户。</td></tr>
	<tr><td>$blobContainerName</td><td>要将 HiveQL 脚本上载到的 Blob 容器。</td></tr>
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
		$hqlLocalFileName = "C:\tutorials\flightdelays\flightdelays.hql"

		# The HiveQL script file will be uploaded to Blob storage as this blob name
		$hqlBlobName = "tutorials/flightdelays/flightdelays.hql"

		# These two constants are used by the HiveQL script file
		#$srcDataFolder = "tutorials/flightdelays/data"
		$dstDataFolder = "/tutorials/flightdelays/output"
		#endregion

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

		#region - Add the Azure account
		Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
		$azureAccounts= Get-AzureAccount
		if (! $azureAccounts)
		{
		    Add-AzureAccount -Environment AzureChinaCloud
		}
		#endregion

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
		    "LOCATION 'wasb://flightdelay@hditutorialdata.blob.core.windows.net/2013Data';"

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
		$storageAccountKey = get-azurestoragekey $storageAccountName | %{$_.Primary}
		$destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

		# Upload the file from local workstation to Blob storage
		Set-AzureStorageBlobContent -File $hqlLocalFileName -Container $blobContainerName -Blob $hqlBlobName -Context $destContext
		#endregion

		Write-host "`nEnd of the PowerShell script" -ForegroundColor Green

	该脚本中使用了以下变量：

	- **$hqlLocalFileName** - 该脚本会先将 HiveQL 脚本文件保存在本地，然后才上载到 Blob 存储。这是文件名。默认值是 <u>C:\\tutorials\\flightdelays\\flightdelays.hql</u>。
	- **$hqlBlobName** - 这是 Azure Blob 存储中使用的 HiveQL 脚本文件 Blob 名称。默认值是 tutorials/flightdelay/flightdelays.hql。因为文件会直接写入 Azure Blob 存储，所以 Blob 名称的开头不是“/”。如果你要从 Blob 存储访问文件，必须在文件名的开头添加“/”。
	- **$srcDataFolder** 和 **$dstDataFolder** - = "tutorials/flightdelay/data" 
	= "tutorials/flightdelays/output"


---
##<a id="appendix-c"></a>附录 C - 针对 Sqoop 作业输出准备 Azure SQL 数据库
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

		    # SQL database server variables
		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the Azure SQL Database Server Name to use an existing one. Enter nothing to create a new one.")]
		    [AllowEmptyString()]
		    [String]$sqlDatabaseServer,  # Specify the Azure SQL database server name if you have one created. Otherwise use "".

		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the Azure SQL Database admin user.")]
		    [String]$sqlDatabaseUsername,

		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the Azure SQL Database admin user password.")]
		    [String]$sqlDatabasePassword,

		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the region to create the Database in.")]
		    [AllowEmptyString()]
		    [String]$sqlDatabaseLocation,   #For example, China North.

		    # SQL database variables
		    [Parameter(Mandatory=$True,
		               HelpMessage="Enter the database name if you have created one. Enter nothing to create one.")]
		    [AllowEmptyString()]
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

		#region - Add the Azure account
		Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
		$azureAccounts= Get-AzureAccount
		if (! $azureAccounts)
		{
		Add-AzureAccount -Environment AzureChinaCloud
		}
		#endregion

		#region - Create and validate Azure SQL database server
		if ([string]::IsNullOrEmpty($sqlDatabaseServer))
		{
			Write-Host "`nCreating SQL Database server ..."  -ForegroundColor Green
			$sqlDatabaseServer = (New-AzureSqlDatabaseServer -AdministratorLogin $sqlDatabaseUsername -AdministratorLoginPassword $sqlDatabasePassword -Location $sqlDatabaseLocation).ServerName
		    Write-Host "`tThe new SQL database server name is $sqlDatabaseServer." -ForegroundColor Cyan

		    Write-Host "`nCreating firewall rule, $fireWallRuleName ..." -ForegroundColor Green
		    $workstationIPAddress = Invoke-RestMethod $ipAddressRestService
		    New-AzureSqlDatabaseServerFirewallRule -ServerName $sqlDatabaseServer -RuleName "$fireWallRuleName-workstation" -StartIpAddress $workstationIPAddress -EndIpAddress $workstationIPAddress
		    New-AzureSqlDatabaseServerFirewallRule -ServerName $sqlDatabaseServer -RuleName "$fireWallRuleName-Azureservices" -AllowAllAzureServices
		}
		else
		{
		    $dbServer = Get-AzureSqlDatabaseServer -ServerName $sqlDatabaseServer
		    if (! $dbServer)
		    {
		        throw "The Azure SQL database server, $sqlDatabaseServer, doesn't exist!"
		    }
		    else
		    {
			    Write-Host "`nUse an existing SQL Database server, $sqlDatabaseServer" -ForegroundColor Green
		    }
		}
		#endregion

		#region - Create and validate Azure SQL database
		if ([string]::IsNullOrEmpty($sqlDatabaseName))
		{
			Write-Host "`nCreating SQL Database, HDISqoop ..."  -ForegroundColor Green

			$sqlDatabaseName = "HDISqoop"
			$sqlDatabaseServerCredential = new-object System.Management.Automation.PSCredential($sqlDatabaseUsername, ($sqlDatabasePassword  | ConvertTo-SecureString -asPlainText -Force))

		    $sqlDatabaseServerConnectionContext = New-AzureSqlDatabaseServerContext -ServerName $sqlDatabaseServer -Credential $sqlDatabaseServerCredential

			$sqlDatabase = New-AzureSqlDatabase -ConnectionContext $sqlDatabaseServerConnectionContext -DatabaseName $sqlDatabaseName -MaxSizeGB $sqlDatabaseMaxSizeGB
		}
		else
		{
		    $db = Get-AzureSqlDatabase -ServerName $sqlDatabaseServer -DatabaseName $sqlDatabaseName
		    if (! $db)
		    {
		        throw "The Azure SQL database server, $sqlDatabaseServer, doesn't exist!"
		    }
		    else
		    {
			    Write-Host "`nUse an existing SQL Database, $sqlDatabaseName" -ForegroundColor Green
		    }
		}
		#endregion

		#region -  Execute an SQL command to create the AvgDelays table

		Write-Host "`nCreating SQL Database table ..."  -ForegroundColor Green
		$conn = New-Object System.Data.SqlClient.SqlConnection
		$conn.ConnectionString = "Data Source=$sqlDatabaseServer.database.chinacloudapi.cn;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseUsername;Password=$sqlDatabasePassword;Encrypt=true;Trusted_Connection=false;"
		$conn.open()
		$cmd = New-Object System.Data.SqlClient.SqlCommand
		$cmd.connection = $conn
		$cmd.commandtext = $sqlCreateAvgDelaysTable
		$cmd.executenonquery()

		$conn.close()

		Write-host "`nEnd of the PowerShell script" -ForegroundColor Green

	>[AZURE.NOTE]该脚本使用具象状态传输 (REST) 服务 http://bot.whatismyipaddress.com 来检索外部 IP 地址。IP 地址用于创建 SQL 数据库服务器的防火墙规则。

	该脚本中使用的某些变量：

	- **$ipAddressRestService** - 默认值为 http://bot.whatismyipaddress.com。这是用来获取外部 IP 地址的公共 IP 地址 REST 服务。如果需要，你可以使用其他服务。使用此服务检索的外部 IP 地址将用于创建 Azure SQL 数据库服务器的防火墙规则，使你能够从工作站访问数据库（通过 Windows PowerShell 脚本）。
	- **$fireWallRuleName** - 这是 Azure SQL 数据库服务器的防火墙规则名称。默认名称为 <u>FlightDelay</u>。如果需要，你可以将它重命名。
	- **$sqlDatabaseMaxSizeGB** - 只有在创建新的 Azure SQL 数据库服务器时才会使用此值。默认值为 10GB。10GB 对于本教程来说已足够。
	- **$sqlDatabaseName** - 只有在创建新的 Azure SQL 数据库时才会使用此值。默认值为 HDISqoop。如果将它重命名，则必须相应地更新 Sqoop Windows PowerShell 脚本。

4. 按 **F5** 运行脚本。
5. 验证脚本输出。确保已成功运行脚本。

##<a id="nextsteps"></a>后续步骤
现在你已了解如何执行以下操作：将文件上载到 Azure Blob 存储、使用 Azure Blob 存储中的数据填充 Hive 表、运行 Hive 查询以及使用 Sqoop 将数据从 HDFS 导出到 Azure SQL 数据库。若要了解更多信息，请参阅下列文章：

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
[powershell-install-configure]: /documentation/articles/powershell-install-configure/

[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/
[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/
[hdinsight-develop-streaming]: /documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/
[hdinsight-develop-mapreduce]: /documentation/articles/hdinsight-develop-deploy-java-mapreduce/

[hadoop-hiveql]: https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL
[hadoop-shell-commands]: http://hadoop.apache.org/docs/r0.18.3/hdfs_shell.html

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx

[image-hdi-flightdelays-avgdelays-dataset]: ./media/hdinsight-analyze-flight-delay-data/HDI.FlightDelays.AvgDelays.DataSet.png
[img-hdi-flightdelays-run-hive-job-output]: ./media/hdinsight-analyze-flight-delay-data/HDI.FlightDelays.RunHiveJob.Output.png
[img-hdi-flightdelays-flow]: ./media/hdinsight-analyze-flight-delay-data/HDI.FlightDelays.Flow.png

<!---HONumber=Mooncake_1207_2015-->