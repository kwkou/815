<properties
	pageTitle="在 HDInsight 中使用 Hadoop Sqoop | Azure"
	description="学习如何从工作站使用 Azure PowerShell 在 Hadoop 群集和 Azure SQL 数据库之间运行 Sqoop 导入和导出。"
	editor="cgronlun"
	manager="paulettm"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian"/>

<tags
	ms.service="hdinsight"
	ms.date="04/06/2016"
	wacn.date="05/24/2016"/>

#将 Sqoop 与 HDInsight 中的 Hadoop 配合使用

[AZURE.INCLUDE [sqoop-selector](../includes/hdinsight-selector-use-sqoop.md)]

了解如何使用 HDInsight 中的 Sqoop 在 HDInsight 群集和 Azure SQL 数据库或 SQL Server 数据库之间进行导入和导出。

虽然自然而然地选用 Hadoop 处理如日志和文件等非结构化和半结构化的数据，但可能还需要处理存储在关系数据库中的结构化数据。

[Sqoop][sqoop-user-guide-1.4.4] 是一种为在 Hadoop 群集和关系数据库之间传输数据而设计的工具。可以使用此工具将数据从关系数据库管理系统 (RDBMS)（如 SQL Server、MySQL 或 Oracle）中导入到 Hadoop 分布式文件系统 (HDFS)，在 Hadoop 中使用 MapReduce 或 Hive 转换数据，然后回过来将数据导出到 RDBMS。在本教程中，你要为你的关系数据库使用 SQL Server 数据库。

有关 HDInsight 群集上支持的 Sqoop 版本，请参阅 [HDInsight 提供的群集版本有哪些新功能？][hdinsight-versions]。

##了解方案

HDInsight 群集带有某些示例数据。你将会使用以下两个示例：

- 位于 */example/data/sample.log* 的 log4j 日志文件。以下日志会从该文件中提取出来：

		2012-02-03 18:35:34 SampleClass6 [INFO] everything normal for id 577725851
		2012-02-03 18:35:34 SampleClass4 [FATAL] system problem at id 1991281254
		2012-02-03 18:35:34 SampleClass3 [DEBUG] detail for id 1304807656
		...

- 一个名为 *hivesampletable* 的 Hive 表，它引用位于 */hive/warehouse/hivesampletable* 中的数据文件。该表包含一些移动设备数据。

    | 字段 | 数据类型 |
    | ----- | --------- |
    | clientid | 字符串 |
    | querytime | 字符串 |
    | market | 字符串 |
    | deviceplatform | 字符串 |
    | devicemake | 字符串 |
    | devicemodel | 字符串 |
    | state | 字符串 |
    | country | 字符串 |
    | querydwelltime | double |
    | sessionid | bigint |
    | sessionpagevieworder | bigint |

你需要首先将 *sample.log* 和 *hivesampletable* 导出到 Azure SQL 数据库或 SQL Server，然后使用以下路径将包含移动设备数据的表导回 HDInsight：

	/tutorials/usesqoop/importeddata

##<a name="create-cluster-and-sql-database"></a> 创建群集和 SQL 数据库

在 [Azure 经典管理门户](https://manage.windowsazure.cn)上创建 SQL 数据库。

1. 登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 单击“新建”>“数据服务”>“Sql 数据库”>“自定义创建”。
3. 为数据库输入名称，并为数据库选择“服务层”、“性能级别”、“排序规则”和“服务器”。
4. 如果选择为数据库创建新的 SQL 数据库服务器，请单击“下一步”。如果选择现有服务，请确保它满足[以下条件](#sql_server_condition)，然后单击“完成”。
5. 输入登录用户名和密码，然后为 SQL Server 选择区域（如果选择创建新服务器）。

> [AZURE.NOTE] SQl Server 的资源组名称是“Default-Sql-chinaeast”或“Default-Sql-chinanorth”，具体取决于 SQL Server 的区域。

在 [Azure 经典管理门户](https://manage.windowsazure.cn)上创建群集。

1. 登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 单击“新建”>“数据服务”>“HDInsight”>“Hadoop”。
3. 为群集输入名称、HTTP 密码，然后选择“群集大小”和“存储帐户”。
4. 单击“创建 HDINSIGHT 群集”。

如果选择使用现有的 Azure SQL 数据库或 Microsoft SQL Server

- **Azure SQL 数据库**：你必须为 Azure SQL 数据库服务器配置防火墙规则以允许从你的工作站进行访问。有关创建 Azure SQL 数据库和配置防火墙的说明，请参阅 [Azure SQL 数据库入门][sqldatabase-get-started]。 

    > [AZURE.NOTE] 默认情况下，可以从 Azure HDInsight 这样的 Azure 服务连接 Azure SQL 数据库。如果禁用了此防火墙设置，则必须从 Azure 经典管理门户启用它。有关创建 Azure SQL 数据库和配置防火墙规则的说明，请参阅[创建和配置 SQL 数据库][sqldatabase-create-configue]。

    <a name="sql_server_condition"></a>

- **SQL Server**：如果你的 HDInsight 群集与 SQL Server 位于 Azure 中的同一虚拟网络，你可以使用本文中的步骤对 SQL Server 数据库执行数据导入和导出操作。

    > [AZURE.NOTE] HDInsight 仅支持基于位置的虚拟网络，并且当前不适用于基于地缘组的虚拟网络。

    * 若要创建和配置虚拟网络，请参阅[虚拟网络配置任务](/home/features/virtual-machines/)。

        * 在数据中心使用 SQL Server 时，必须将虚拟网络配置为站点到站点或点到站点。

            > [AZURE.NOTE] 对于**点到站点**虚拟网络，SQL Server 必须运行 Azure 虚拟网络配置的“仪表板”中提供的 VPN 客户端配置应用程序。

        * 当你在 Azure 虚拟机上使用 SQL Server 时，如果托管 SQL Server 的虚拟机是 HDInsight 所在虚拟网络的成员，则可以使用任何虚拟网络配置。

    * 若要在虚拟网络上创建 HDInsight 群集，请参阅[使用自定义选项在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters-v1/)

    > [AZURE.NOTE] SQL Server 还必须允许身份验证。必须使用 SQL Server 登录名来完成此文章中的步骤。
	

## 运行 Sqoop 作业

HDInsight 可以使用各种方法运行 Sqoop 作业。使用下表来确定哪种方法最适合你，然后按链接进行演练。

| **使用此方法**，如果你想要... | ...**交互式** shell | ...**批处理** | ...使用此**群集操作系统** | ...从此**客户端操作系统** |
|:--------------------------------------------------------------|:---------------------------:|:-----------------------:|:------------------------------------------|:-----------------------------------------|
| [.NET SDK for Hadoop](/documentation/articles/hdinsight-hadoop-use-sqoop-dotnet-sdk/) | &nbsp; | ✔ | Windows | Windows（暂时） |
| [Azure PowerShell](/documentation/articles/hdinsight-hadoop-use-sqoop-powershell/) | &nbsp; | ✔ | Windows | Windows |

##后续步骤

现在你已经学习了如何使用 Sqoop。若要了解更多信息，请参阅以下文章：

- [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
- [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
- [将 Oozie 与 HDInsight 配合使用][hdinsight-use-oozie]：在 Oozie 工作流中使用 Sqoop 操作。
- [使用 HDInsight 分析航班延误数据][hdinsight-analyze-flight-data]：使用 Hive 分析航班延误数据，然后使用 Sqoop 将数据导出到 Azure SQL 数据库。
- [将数据上载到 HDInsight][hdinsight-upload-data]：了解将数据上载到 HDInsight/Azure Blob 存储的其他方法。


## 附录 A - PowerShell 示例

PowerShell 示例将执行以下步骤：

1. 连接到 Azure。
2. 创建 Azure 资源组。有关详细信息，请参阅[将 Azure PowerShell 与 Azure 资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager/)
3. 创建一个 Azure SQL 数据库服务器、一个 Azure SQL 数据库和两个表。 

	如果改用 SQL Server，请使用以下语句来创建表：
	
		CREATE TABLE [dbo].[log4jlogs](
		 [t1] [nvarchar](50),
		 [t2] [nvarchar](50),
		 [t3] [nvarchar](50),
		 [t4] [nvarchar](50),
		 [t5] [nvarchar](50),
		 [t6] [nvarchar](50),
		 [t7] [nvarchar](50))

		CREATE TABLE [dbo].[mobiledata](
		 [clientid] [nvarchar](50),
		 [querytime] [nvarchar](50),
		 [market] [nvarchar](50),
		 [deviceplatform] [nvarchar](50),
		 [devicemake] [nvarchar](50),
		 [devicemodel] [nvarchar](50),
		 [state] [nvarchar](50),
		 [country] [nvarchar](50),
		 [querydwelltime] [float],
		 [sessionid] [bigint],
		 [sessionpagevieworder][bigint])

	检查数据库和表的最简单方法是使用 Visual Studio。可以使用 Azure 经典管理门户检查数据库服务器和数据库。

4. 创建 HDInsight 群集。

	若要检查群集，可以使用 Azure 经典管理门户或 Azure PowerShell。

5. 预处理源数据文件。

	在本教程中，你要将一个 log4j log 文件（带分隔符的文件）和一个 Hive 表导出到 Azure SQL 数据库。带分隔符的文件名为 */example/data/sample.log*。在本教程前面，你看到了几个 log4j 日志的示例。在日志文件中，有一些空行和一些类似下面这样的行：
	
		java.lang.Exception: 2012-02-03 20:11:35 SampleClass2 [FATAL] unrecoverable system problem at id 609774657
			at com.osa.mocklogger.MockLogger$2.run(MockLogger.java:83)
	
	对于其他使用此数据的示例来说，这是没有问题的，但我们必须删除这些异常，然后才能将内容导入到 Azure SQL 数据库或 SQL Server 中。如果有空字符串，或者有其元素数量比 Azure SQL 数据库表中所定义字段数量要少的行，Sqoop 导出将会失败。log4jlogs 表有 7 个字符串类型的字段。

	此过程将在群集上创建新文件：tutorials/usesqoop/data/sample.log。若要检查修改后的数据文件，可以使用 Azure 经典管理门户、Azure 存储资源管理器工具或 Azure PowerShell。[HDInsight 入门][hdinsight-get-started]中有一个关于使用 Azure PowerShell 下载文件并显示文件内容的代码示例。

6. 将数据文件导出到 Azure SQL 数据库。

	源文件为 tutorials/usesqoop/data/sample.log。数据导出到的表的名称为 log4jlogs。
	
	> [AZURE.NOTE] 除了连接字符串信息，此节中的步骤还应适用于 Azure SQL 数据库或 SQL Server。这些步骤已使用以下配置测试过：
	><p> * **Azure 虚拟网络点到站点配置**：虚拟网络已将 HDInsight 群集连接到专用数据中心的 SQL Server。
	><p> * **Azure HDInsight 3.1**：有关在虚拟网络上创建群集的信息，请参阅[使用自定义选项在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters-v1/)。
	><p> * **SQL Server 2014**：已配置为允许身份验证和运行 VPN 客户端配置包，可以安全地连接到虚拟网络。

7. 将 Hive 表导出到 Azure SQL 数据库。

8. 将 mobiledata 表导入 HDInsight 群集。

	若要检查修改后的数据文件，可以使用 Azure 经典管理门户、Azure 存储资源管理器工具或 Azure PowerShell。[HDInsight 入门][hdinsight-get-started]中有一个关于使用 Azure PowerShell 下载文件并显示文件内容的代码示例。


### PowerShell 示例

	# Prepare an Azure SQL database to be used by the Sqoop tutorial
	
	#region - provide the following values
	
	$subscriptionID = "<Enter your Azure Subscription ID>"
	
	$sqlDatabaseLogin = "<Enter a SQL Database Login name>" #SQL Database server login
	$sqlDatabasePassword = "<Enter a Password>"
	
	$httpUserName = "admin"  #HDInsight cluster username
	$httpPassword = "<Enter a Password>"
	
	# used for creating Azure service names
	$nameToken = "<Enter an alias>" 
	$namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")
	#endregion
	
	#region - variables
	
	# Resource group variables
	$resourceGroupName = $namePrefix + "rg"
	$location = "China East 2" # used by all Azure services defined in this tutorial
	
	# SQL database varialbes
	$sqlDatabaseServerName = $namePrefix + "sqldbserver"
	$sqlDatabaseName = $namePrefix + "sqldb"
	$sqlDatabaseConnectionString = "Data Source=$sqlDatabaseServerName.database.chinacloudapi.cn;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseLogin;Password=$sqlDatabasePassword;Encrypt=true;Trusted_Connection=false;"
	$sqlDatabaseMaxSizeGB = 10
	
	# Used for retrieving external IP address and creating firewall rules
	$ipAddressRestService = "http://bot.whatismyipaddress.com"
	$fireWallRuleName = "UseSqoop"
	
	# Used for creating tables and clustered indexes
	$cmdCreateLog4jTable = "CREATE TABLE [dbo].[log4jlogs](
		[t1] [nvarchar](50),
		[t2] [nvarchar](50),
		[t3] [nvarchar](50),
		[t4] [nvarchar](50),
		[t5] [nvarchar](50),
		[t6] [nvarchar](50),
		[t7] [nvarchar](50))"
	
	$cmdCreateLog4jClusteredIndex = "CREATE CLUSTERED INDEX log4jlogs_clustered_index on log4jlogs(t1)"
	
	$cmdCreateMobileTable = " CREATE TABLE [dbo].[mobiledata](
	[clientid] [nvarchar](50),
	[querytime] [nvarchar](50),
	[market] [nvarchar](50),
	[deviceplatform] [nvarchar](50),
	[devicemake] [nvarchar](50),
	[devicemodel] [nvarchar](50),
	[state] [nvarchar](50),
	[country] [nvarchar](50),
	[querydwelltime] [float],
	[sessionid] [bigint],
	[sessionpagevieworder][bigint])"
	
	$cmdCreateMobileDataClusteredIndex = "CREATE CLUSTERED INDEX mobiledata_clustered_index on mobiledata(clientid)"
	
	# HDInsight variables
	$hdinsightClusterName = $namePrefix + "hdi"
	$defaultStorageAccountName = $namePrefix + "store"
	$defaultBlobContainerName = $hdinsightClusterName
	#endregion
	
	# Treat all errors as terminating
	$ErrorActionPreference = "Stop"
	
	#region - Connect to Azure subscription
	Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
	try{Get-AzureRmContext}
	catch{Login-AzureRmAccount -EnvironmentName AzureChinaCloud}

    try{Get-AzureSubscription}
	catch{Add-AzureAccount -Environment AzureChinaCloud}
	#endregion
	
	#region - Create Azure resouce group
	Write-Host "`nCreating an Azure resource group ..." -ForegroundColor Green
	try{
		Get-AzureRmResourceGroup -Name $resourceGroupName
	}
	catch{
		New-AzureRmResourceGroup -Name $resourceGroupName -Location $location
	}
	#endregion
	
	#region - Create Azure SQL database server
	Write-Host "`nCreating an Azure SQL Database server ..." -ForegroundColor Green
	try{
		Get-AzureRmSqlServer -ServerName $sqlDatabaseServerName -ResourceGroupName $resourceGroupName}
	catch{
		Write-Host "`nCreating SQL Database server ..."  -ForegroundColor Green
	
		$sqlDatabasePW = ConvertTo-SecureString -String $sqlDatabasePassword -AsPlainText -Force
		$credential = New-Object System.Management.Automation.PSCredential($sqlDatabaseLogin,$sqlDatabasePW)
	
		$sqlDatabaseServerName = (New-AzureRmSqlServer `
									-ResourceGroupName $resourceGroupName `
									-ServerName $sqlDatabaseServerName `
									-SqlAdministratorCredentials $credential `
									-Location $location).ServerName
		Write-Host "`tThe new SQL database server name is $sqlDatabaseServerName." -ForegroundColor Cyan
	
		Write-Host "`nCreating firewall rule, $fireWallRuleName ..." -ForegroundColor Green
		$workstationIPAddress = Invoke-RestMethod $ipAddressRestService
		New-AzureRmSqlServerFirewallRule `
			-ResourceGroupName $resourceGroupName `
			-ServerName $sqlDatabaseServerName `
			-FirewallRuleName "$fireWallRuleName-workstation" `
			-StartIpAddress $workstationIPAddress `
			-EndIpAddress $workstationIPAddress
	
		#To allow other Azure services to access the server add a firewall rule and set both the StartIpAddress and EndIpAddress to 0.0.0.0. 
		#Note that this allows Azure traffic from any Azure subscription to access the server.
		New-AzureRmSqlServerFirewallRule `
			-ResourceGroupName $resourceGroupName `
			-ServerName $sqlDatabaseServerName `
			-FirewallRuleName "$fireWallRuleName-Azureservices" `
			-StartIpAddress "0.0.0.0" `
			-EndIpAddress "0.0.0.0"
	}
	
	#endregion
	
	#region - Create and validate Azure SQL database
	Write-Host "`nCreating an Azure SQL database ..." -ForegroundColor Green
	
	try {
		Get-AzureRmSqlDatabase `
			-ResourceGroupName $resourceGroupName `
			-ServerName $sqlDatabaseServerName `
			-DatabaseName $sqlDatabaseName
	}
	catch {
		Write-Host "`nCreating SQL Database, $sqlDatabaseName ..."  -ForegroundColor Green
		New-AzureRMSqlDatabase `
			-ResourceGroupName $resourceGroupName `
			-ServerName $sqlDatabaseServerName `
			-DatabaseName $sqlDatabaseName `
			-Edition "Standard" `
			-RequestedServiceObjectiveName "S1"
	}
	
	#endregion
	
	#region - Create tables
	Write-Host "Creating the log4jlogs table and the mobiledata table ..." -ForegroundColor Green
	
	$conn = New-Object System.Data.SqlClient.SqlConnection
	$conn.ConnectionString = $sqlDatabaseConnectionString
	$conn.Open()
	
	# Create the log4jlogs table and index
	$cmd = New-Object System.Data.SqlClient.SqlCommand
	$cmd.Connection = $conn
	$cmd.CommandText = $cmdCreateLog4jTable
	$ret = $cmd.ExecuteNonQuery()
	$cmd.CommandText = $cmdCreateLog4jClusteredIndex
	$cmd.ExecuteNonQuery()
	
	# Create the mobiledata table and index
	$cmd.CommandText = $cmdCreateMobileTable
	$cmd.ExecuteNonQuery()
	$cmd.CommandText = $cmdCreateMobileDataClusteredIndex
	$cmd.ExecuteNonQuery()
	
	$conn.close()
	
	#endregion
	
	
	#region - Create HDInsight cluster
	
	Write-Host "Creating the HDInsight cluster and the dependent services ..." -ForegroundColor Green
	
	# Create the default storage account
	New-AzureStorageAccount `
		-StorageAccountName $defaultStorageAccountName `
		-Location $location `
		-Type Standard_LRS
	
	# Create the default Blob container
	$defaultStorageAccountKey = Get-AzureStorageAccountKey `
									-StorageAccountName $defaultStorageAccountName |  %{ $_.primary }
	$defaultStorageAccountContext = New-AzureStorageContext `
										-StorageAccountName $defaultStorageAccountName `
										-StorageAccountKey $defaultStorageAccountKey 
	New-AzureStorageContainer `
		-Name $defaultBlobContainerName `
		-Context $defaultStorageAccountContext 
	
	# Create the HDInsight cluster
	$pw = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
	$httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$pw)
	
	New-AzureHDInsightCluster `
		-Name $HDInsightClusterName `
		-Location $location `
		-ClusterType Hadoop `
		-OSType Windows `
		-ClusterSizeInNodes 2 `
		-Credential $httpCredential `
		-DefaultStorageAccountName "$defaultStorageAccountName.blob.core.chinacloudapi.cn" `
		-DefaultStorageAccountKey $defaultStorageAccountKey `
		-DefaultStorageContainerName $defaultBlobContainerName 
	
	# Validate the cluster
	Get-AzureHDInsightCluster -Name $hdinsightClusterName
	#endregion
	
	#region - pre-process the source file
	
	Write-Host "Preprocessing the source file ..." -ForegroundColor Green
	
	# This procedure creates a new file with $destBlobName
	$sourceBlobName = "example/data/sample.log"
	$destBlobName = "tutorials/usesqoop/data/sample.log"
	
	# Define the connection string
	$storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=$defaultStorageAccountName;AccountKey=$defaultStorageAccountKey"
	
	# Create block blob objects referencing the source and destination blob.
	$storageAccount = [Microsoft.WindowsAzure.Storage.CloudStorageAccount]::Parse($storageConnectionString)
	$storageClient = $storageAccount.CreateCloudBlobClient();
	$storageContainer = $storageClient.GetContainerReference($defaultBlobContainerName)
	$sourceBlob = $storageContainer.GetBlockBlobReference($sourceBlobName)
	$destBlob = $storageContainer.GetBlockBlobReference($destBlobName)
	
	# Define a MemoryStream and a StreamReader for reading from the source file
	$stream = New-Object System.IO.MemoryStream
	$stream = $sourceBlob.OpenRead()
	$sReader = New-Object System.IO.StreamReader($stream)
	
	# Define a MemoryStream and a StreamWriter for writing into the destination file
	$memStream = New-Object System.IO.MemoryStream
	$writeStream = New-Object System.IO.StreamWriter $memStream
	
	# Pre-process the source blob
	$exString = "java.lang.Exception:"
	while(-Not $sReader.EndOfStream){
		$line = $sReader.ReadLine()
		$split = $line.Split(" ")
	
		# remove the "java.lang.Exception" from the first element of the array
		# for example: java.lang.Exception: 2012-02-03 19:11:02 SampleClass8 [WARN] problem finding id 153454612
		if ($split[0] -eq $exString){
			#create a new ArrayList to remove $split[0]
			$newArray = [System.Collections.ArrayList] $split
			$newArray.Remove($exString)
	
			# update $split and $line
			$split = $newArray
			$line = $newArray -join(" ")
		}
	
		# remove the lines that has less than 7 elements
		if ($split.count -ge 7){
			write-host $line
			$writeStream.WriteLine($line)
		}
	}
	
	# Write to the destination blob
	$writeStream.Flush()
	$memStream.Seek(0, "Begin")
	$destBlob.UploadFromStream($memStream)
	
	#endregion
	
	#region - export a log file from the cluster to the SQL database
	
	Write-Host "Preprocessing the source file ..." -ForegroundColor Green
	
	$tableName_log4j = "log4jlogs"
	
	# Connection string for Azure SQL Database.
	# Comment if using SQL Server
	$connectionString = "jdbc:sqlserver://$sqlDatabaseServerName.database.chinacloudapi.cn;user=$sqlDatabaseLogin@$sqlDatabaseServerName;password=$sqlDatabasePassword;database=$sqlDatabaseName"
	# Connection string for SQL Server.
	# Uncomment if using SQL Server.
	#$connectionString = "jdbc:sqlserver://$sqlDatabaseServerName;user=$sqlDatabaseLogin;password=$sqlDatabasePassword;database=$sqlDatabaseName"
	
	$exportDir_log4j = "/tutorials/usesqoop/data"
	
	# Submit a Sqoop job
	$sqoopDef = New-AzureHDInsightSqoopJobDefinition `
		-Command "export --connect $connectionString --table $tableName_log4j --export-dir $exportDir_log4j --input-fields-terminated-by \0x20 -m 1"
	$sqoopJob = Start-AzureHDInsightJob `
					-Cluster $hdinsightClusterName `
					-Credential $httpCredential `
					-JobDefinition $sqoopDef #-Debug -Verbose
	Wait-AzureHDInsightJob `
		-Cluster $hdinsightClusterName `
		-Credential $httpCredential `
		-JobId $sqoopJob.JobId
	
	Write-Host "Standard Error" -BackgroundColor Green
	Get-AzureHDInsightJobOutput -Cluster $hdinsightClusterName -JobId $sqoopJob.JobId -StandardError
	Write-Host "Standard Output" -BackgroundColor Green
	Get-AzureHDInsightJobOutput -Cluster $hdinsightClusterName -JobId $sqoopJob.JobId -StandardOutput
	
	#endregion
	
	#region - export a Hive table
	
	$tableName_mobile = "mobiledata"
	$exportDir_mobile = "/hive/warehouse/hivesampletable"
	
	$sqoopDef = New-AzureHDInsightSqoopJobDefinition `
		-Command "export --connect $connectionString --table $tableName_mobile --export-dir $exportDir_mobile --fields-terminated-by \t -m 1"
	$sqoopJob = Start-AzureHDInsightJob `
					-Cluster $hdinsightClusterName `
					-Credential $httpCredential `
					-JobDefinition $sqoopDef #-Debug -Verbose
	
	Wait-AzureHDInsightJob `
		-Cluster $hdinsightClusterName `
		-Credential $httpCredential `
		-JobId $sqoopJob.JobId
	
	Write-Host "Standard Error" -BackgroundColor Green
	Get-AzureHDInsightJobOutput `
		-Cluster $hdinsightClusterName `
		-JobId $sqoopJob.JobId `
		-StandardError
	
	Write-Host "Standard Output" -BackgroundColor Green
	Get-AzureHDInsightJobOutput `
		-Cluster $hdinsightClusterName `
		-JobId $sqoopJob.JobId `
		-StandardOutput
	
	#endregion
	
	#region - import a database
	
	$targetDir_mobile = "/tutorials/usesqoop/importeddata/"
	
	$sqoopDef = New-AzureHDInsightSqoopJobDefinition `
		-Command "import --connect $connectionString --table $tableName_mobile --target-dir $targetDir_mobile --fields-terminated-by \t --lines-terminated-by \n -m 1"
	
	$sqoopJob = Start-AzureHDInsightJob `
					-Cluster $hdinsightClusterName `
					-Credential $httpCredential `
					-JobDefinition $sqoopDef #-Debug -Verbose
	
	Wait-AzureHDInsightJob `
		-Cluster $hdinsightClusterName `
		-Credential $httpCredential `
		-JobId $sqoopJob.JobId
	
	Write-Host "Standard Error" -BackgroundColor Green
	Get-AzureHDInsightJobOutput `
		-Cluster $hdinsightClusterName `
		-JobId $sqoopJob.JobId `
		-StandardError
	
	Write-Host "Standard Output" -BackgroundColor Green
	Get-AzureHDInsightJobOutput `
		-Cluster $hdinsightClusterName `
		-JobId $sqoopJob.JobId `
		-StandardOutput
	
	#endregion



[azure-management-portal]: https://manage.windowsazure.cn/

[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning-v1/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-analyze-flight-data]: /documentation/articles/hdinsight-analyze-flight-delay-data/
[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/

[sqldatabase-get-started]: /documentation/articles/sql-database-get-started/
[sqldatabase-create-configue]: /documentation/articles/sql-database-get-started/

[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
[powershell-install]: /documentation/articles/powershell-install-configure/
[powershell-script]: https://technet.microsoft.com/zh-cn/library/dn425048.aspx

[sqoop-user-guide-1.4.4]: https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html

<!---HONumber=Mooncake_0516_2016-->