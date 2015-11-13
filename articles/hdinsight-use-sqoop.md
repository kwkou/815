<properties 
	pageTitle="在 HDInsight 中使用 Hadoop Sqoop | Azure" 
	description="学习如何从工作站使用 Azure PowerShell 在 Hadoop 群集和 Azure SQL 数据库之间运行 Sqoop 导入和导出。" 
	editor="cgronlun" 
	manager="paulettm" 
	services="hdinsight" 
	documentationCenter="" 
	authors="mumian"/>

<tags
	ms.service="hdinsight"
	ms.date="07/28/2015"
	wacn.date="10/03/2015" />

#将 Sqoop 与 HDInsight 中的 Hadoop 配合使用 (Windows)

[AZURE.INCLUDE [sqoop-selector](../includes/hdinsight-selector-use-sqoop.md)]

学习如何从工作站使用 Azure PowerShell 和 HDInsight .NET SDK 在 HDInsight 群集和 Azure SQL 数据库或 SQL Server 数据库之间运行 Sqoop 进行导入和导出。

##什么是 Sqoop？

虽然自然而然地选用 Hadoop 处理如日志和文件等非结构化和半结构化的数据，但可能还需要处理存储在关系数据库中的结构化数据。

[Sqoop][sqoop-user-guide-1.4.4] 是一种为在 Hadoop 群集和关系数据库之间传输数据而设计的工具。可以使用此工具将数据从关系数据库管理系统 (RDBMS)（如 SQL Server、MySQL 或 Oracle）中导入到 Hadoop 分布式文件系统 (HDFS)，在 Hadoop 中使用 MapReduce 或 Hive 转换数据，然后回过来将数据导出到 RDBMS。在本教程中，你要为你的关系数据库使用 SQL Server 数据库。

有关 HDInsight 群集上支持的 Sqoop 版本，请参阅 [HDInsight 提供的群集版本有哪些新功能？][hdinsight-versions]。




##先决条件

在开始阅读本教程前，你必须具有：

- **配备 Azure PowerShell 的工作站**。请参阅[安装和使用 Azure PowerShell][powershell-install]。若要执行 Azure PowerShell 脚本，必须以管理员身份运行 Azure PowerShell 并将执行策略设为 *RemoteSigned*。请参阅[运行 Windows PowerShell 脚本][powershell-script]。

- **Azure HDInsight 群集**：有关群集预配的说明，请参阅[开始使用 HDInsight][hdinsight-get-started] 或[预配 HDInsight 群集][hdinsight-provision]。你需要以下数据才能完成本教程：

	<table border="1">
	<tr><th>群集属性</th><th>Azure PowerShell 变量名</th><th>值</th><th>说明</th></tr>
	<tr><td>HDInsight 群集名称</td><td>$clusterName</td><td></td><td>HDInsight 群集名称。</td></tr>
	<tr><td>Azure 存储帐户名</td><td>$storageAccountName</td><td></td><td>可用于 HDInsight 群集的 Azure 存储帐户。在本教程中，使用在群集设置过程中指定的默认存储帐户。</td></tr>
	<tr><td>Azure Blob 容器名称</td><td>$containerName</td><td></td><td>在此示例中，使用用于默认 HDInsight 群集文件系统的 Blob 的名称。默认情况下，该容器与 HDInsight 群集同名。</td></tr>
	</table>

- **Azure SQL 数据库**：你必须为 Azure SQL 数据库服务器配置防火墙规则以允许从你的工作站进行访问。有关创建 Azure SQL 数据库和配置防火墙的说明，请参阅 [Azure SQL 数据库入门][sqldatabase-get-started]。本文提供了用于创建本教程所需的 Azure SQL 数据库表的 Windows PowerShell 脚本。

	<table border="1">
	<tr><th>Azure SQL 数据库属性</th><th>Azure PowerShell 变量名</th><th>值</th><th>说明</th></tr>
	<tr><td>Azure SQL 数据库服务器名称</td><td>$sqlDatabaseServer</td><td></td><td>Sqoop 要将数据导出到其中或从中导入数据的 Azure SQL 数据库服务器。</td></tr>
	<tr><td>Azure SQL 数据库登录名</td><td>$sqlDatabaseLogin</td><td></td><td>你的 Azure SQL 数据库登录名。</td></tr>
	<tr><td>Azure SQL 数据库登录密码</td><td>$sqlDatabasePassword</td><td></td><td>你的 Azure SQL 数据库登录密码。</td></tr>
	<tr><td>Azure SQL 数据库名称</td><td>$sqlDatabaseName</td><td></td><td>Sqoop 要将数据导出到其中或从中导入数据的 Azure SQL 数据库。</td></tr>
	</table>

	> [AZURE.NOTE]默认情况下，可以从 Azure HDInsight 这样的 Azure 服务连接 Azure SQL 数据库。如果禁用了此防火墙设置，则必须从 Azure 门户启用它。有关创建 Azure SQL 数据库和配置防火墙规则的说明，请参阅[创建和配置 SQL 数据库][sqldatabase-create-configue]。

* **SQL Server**：如果你的 HDInsight 群集与 SQL Server 位于 Azure 中的同一虚拟网络，你可以使用本文中的步骤对 SQL Server 数据库执行数据导入和导出操作。

	> [AZURE.NOTE]HDInsight 仅支持基于位置的虚拟网络，并且当前不适用于基于地缘组的虚拟网络。

	* 若要创建和配置虚拟网络，请参阅[虚拟网络配置任务](http://msdn.microsoft.com/zh-cn/library/azure/jj156206.aspx)。

		* 在数据中心使用 SQL Server 时，必须将虚拟网络配置为*站点到站点*或*点到站点*。

			> [AZURE.NOTE]对于**点到站点**虚拟网络，SQL Server 必须运行 Azure 虚拟网络配置的“仪表板”中提供的 VPN 客户端配置应用程序。

		* 当你在 Azure 虚拟机上使用 SQL Server 时，如果托管 SQL Server 的虚拟机是 HDInsight 所在虚拟网络的成员，则可以使用任何虚拟网络配置。

	* 若要在虚拟网络上预配 HDInsight 群集，请参阅[使用自定义选项在 HDInsight 中预配 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters)

	> [AZURE.NOTE]SQL Server 还必须允许身份验证。必须使用 SQL Server 登录名来完成此文章中的步骤。

	<table border="1">
<tr><th>SQL Server 数据库属性</th><th>Azure PowerShell 变量名</th><th>值</th><th>说明</th></tr>
<tr><td>SQL Server 名称</td><td>$sqlDatabaseServer</td><td></td><td>Sqoop 要将数据导出到其中或从中导入数据的 SQL Server。</td></tr>
<tr><td>SQL Server 登录名</td><td>$sqlDatabaseLogin</td><td></td><td>你的 SQL Server 登录名。</td></tr>
<tr><td>SQL Server 登录密码</td><td>$sqlDatabasePassword</td><td></td><td>你的 SQL Server 登录密码。</td></tr>
<tr><td>SQL Server 数据库名称</td><td>$sqlDatabaseName</td><td></td><td>Sqoop 要将数据导出到其中或从中导入数据的 SQL Server 数据库。</td></tr>
</table>


> [AZURE.NOTE]将值填充到以前的表中。这将有助于学习本教程。

##了解方案
HDInsight 群集带有某些示例数据。你将会使用以下两个示例：

- 位于 */example/data/sample.log* 的 log4j 日志文件。以下日志会从该文件中提取出来：

		2012-02-03 18:35:34 SampleClass6 [INFO] everything normal for id 577725851
		2012-02-03 18:35:34 SampleClass4 [FATAL] system problem at id 1991281254
		2012-02-03 18:35:34 SampleClass3 [DEBUG] detail for id 1304807656
		...

- 一个名为 *hivesampletable* 的 Hive 表，它引用位于 */hive/warehouse/hivesampletable* 中的数据文件。该表包含一些移动设备数据。Hive 表架构为：

	<table border="1">
<tr><th>字段</th><th>数据类型</th></tr>
<tr><td>clientid</td><td>字符串</td></tr>
<tr><td>querytime</td><td>字符串</td></tr>
<tr><td>market</td><td>字符串</td></tr>
<tr><td>deviceplatform</td><td>字符串</td></tr>
<tr><td>devicemake</td><td>字符串</td></tr>
<tr><td>devicemodel</td><td>字符串</td></tr>
<tr><td>state</td><td>字符串</td></tr>
<tr><td>country</td><td>字符串</td></tr>
<tr><td>querydwelltime</td><td>double</td></tr>
<tr><td>sessionid</td><td>bigint</td></tr>
<tr><td>sessionpagevieworder</td><td>bigint</td></tr>
</table>

你需要首先将 *sample.log* 和 *hivesampletable* 导出到 Azure SQL 数据库或 SQL Server，然后使用以下路径将包含移动设备数据的表导回 HDInsight：

	/tutorials/usesqoop/importeddata

###了解 HDInsight 存储

HDInsight 将 Azure Blob 存储用于数据存储。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

设置 HDInsight 群集时，请将 Azure 存储帐户和该帐户上的特定 Blob 存储容器指定为默认文件系统，像在 HDFS 中一样。除了此存储帐户外，在设置过程中，你还可以从同一 Azure 订阅或不同 Azure 订阅添加其他存储帐户。

有关添加其他存储帐户的说明，请参阅[设置 HDInsight 群集][hdinsight-provision]。为了简化本教程中使用的 Windows PowerShell 脚本，所有文件都存储在默认文件系统容器（位于 */tutorials/usesqoop*）中。默认情况下，此容器与 HDInsight 群集同名。语法为：

	wasb[s]://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/<path>/<filename>

> [AZURE.NOTE]HDInsight 群集 3.0 版只支持 **wasb://* 语法。较早的 **asv://* 语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.0 群集中不受支持。

> [AZURE.NOTE]**wasb://* 路径是虚拟路径。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

存储在默认文件系统 Blob 中的文件可以使用以下任一 URI 从 HDInsight 进行访问（以下示例使用 sample.log）：

	wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn/example/data/sample.log
	wasb:///example/data/sample.log
	/example/data/sample.log

如果要从存储帐户直接访问该文件，则请注意，该文件的 Blob 名称是：

	example/data/sample.log


##准备教程

你将在 Azure SQL 数据库或 SQL Server 中创建两个表。在本教程中，这些在以后将由 Sqoop 用来进行导出。你还需要先处理 sample.log 文件，然后 Sqoop 才能处理它们。

###创建 SQL 表

**对于 Azure SQL 数据库**

1. 打开 Windows PowerShell ISE（在 Windows 8 中的“开始”屏幕上键入 **PowerShell\_ISE**，然后单击“Windows PowerShell ISE”。请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][powershell-start]）。

2. 将以下脚本复制到脚本窗格，然后设置前四个变量：
		
		#SQL database variables
		$sqlDatabaseServer = "<SQLDatabaseServerName>" 
		$sqlDatabaseLogin = "<SQLDatabaseUsername>"
		$sqlDatabasePassword = "<SQLDatabasePassword>"
		$sqlDatabaseName = "<SQLDatabaseName>" 

		$sqlDatabaseConnectionString = "Data Source=$sqlDatabaseServer.database.chinacloudapi.cn;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseLogin;Password=$sqlDatabasePassword;Encrypt=true;Trusted_Connection=false;"

	有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)部分。

3. 将以下脚本追加到脚本窗格中。这些是定义两个表及其群集索引的 SQL 语句。Azure SQL 数据库要求群集的索引。

		# SQL query strings for creating tables and clustered indexes
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

4. 将以下脚本追加到脚本窗格中以便运行 SQL 命令：

		Write-Host "Connect to the SQL Database ..." -ForegroundColor Green
		$conn = New-Object System.Data.SqlClient.SqlConnection
		$conn.ConnectionString = $sqlDatabaseConnectionString
		$conn.Open()
		
		Write-Host "Create log4j table and clustered index ..." -ForegroundColor Green
		$cmd = New-Object System.Data.SqlClient.SqlCommand
		$cmd.Connection = $conn
		$cmd.CommandText = $cmdCreateLog4jTable
		$ret = $cmd.ExecuteNonQuery()
		$cmd.CommandText = $cmdCreateLog4jClusteredIndex
		$cmd.ExecuteNonQuery()
		
		Write-Host "Create log4j table and clustered index ..." -ForegroundColor Green
		$cmd.CommandText = $cmdCreateMobileTable
		$cmd.ExecuteNonQuery()
		$cmd.CommandText = $cmdCreateMobileDataClusteredIndex
		$cmd.ExecuteNonQuery()
		
		Write-Host "Close connection ..." -ForegroundColor Green		
		$conn.close()
		
		Write-Host "Done" -ForegroundColor Green
	
5. 单击“运行脚本”或按 **F5** 键以运行该脚本。
6. 使用 [Azure 门户][azure-management-portal]来检查表和群集索引。

**对于 SQL Server**

1. 打开 **SQL Server Management Studio** 并连接到 SQL Server。

2. 创建名为 **sqoopdb** 的新数据库。

3. 选择 **sqoopdb** 数据库，然后从 SQL Server Management Studio 顶部的功能区选择“新建查询”。

4. 在查询窗口中输入以下信息：

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

5. 单击 **F5**，或在功能区中选择 **! Execute** 以运行查询。在查询下会显示以下消息：

		Command(s) completed successfully.

6. 关闭 SQL Server Management Studio。

###生成数据

在本教程中，你要将一个 log4j log 文件（带分隔符的文件）和一个 Hive 表导出到 Azure SQL 数据库。分隔的文件名为 */example/data/sample.log*。在本教程前面，你看到了几个 log4j 日志的示例。在日志文件中，有一些空行和一些类似下面这样的行：

	java.lang.Exception: 2012-02-03 20:11:35 SampleClass2 [FATAL] unrecoverable system problem at id 609774657
		at com.osa.mocklogger.MockLogger$2.run(MockLogger.java:83)

对于其他使用此数据的示例来说，这是没有问题的，但我们必须删除这些异常，然后才能将内容导入到 Azure SQL 数据库或 SQL Server 中。如果有空字符串，或者有其元素数量比 Azure SQL 数据库表中所定义字段数量要少的行，Sqoop 导出将会失败。log4jlogs 表有 7 个字符串类型的字段。

**处理 sample.log 文件**

1. 打开 Windows PowerShell ISE。
2. 在底部窗格中，运行以下命令以连接到 Azure 订阅：

		Add-AzureAccount -Environment AzureChinaCloud

	系统将提示你输入 Azure 帐户凭据。这种添加订阅连接的方法会超时，12 个小时之后，你将需要再次登录。

	> [AZURE.NOTE]如果你有多个 Azure 订阅，而默认订阅不是你想使用的，则请使用 <strong>Select-AzureSubscription</strong> cmdlet 来选择正确的订阅。

3. 将以下脚本复制到脚本窗格，然后设置前两个变量。
		
		$storageAccountName = "<AzureStorageAccountName>"
		$containerName = "<BlobContainerName>"
		
		$sourceBlobName = "example/data/sample.log"
		$destBlobName = "tutorials/usesqoop/data/sample.log"

	有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)部分。
 
4. 将以下脚本追加到脚本窗格中：

		# Define the connection string
		$storageAccountKey = get-azurestoragekey $storageAccountName | %{$_.Primary}
		$storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=$storageAccountName;AccountKey=$storageAccountKey"
		
		# Create block blob objects referencing the source and destination blob.
		$storageAccount = [Microsoft.WindowsAzure.Storage.CloudStorageAccount]::Parse($storageConnectionString)
		$storageClient = $storageAccount.CreateCloudBlobClient();
		$storageContainer = $storageClient.GetContainerReference($containerName)
		$sourceBlob = $storageContainer.GetBlockBlobReference($sourceBlobName)
		$destBlob = $storageContainer.GetBlockBlobReference($destBlobName)
		
		# Define a MemoryStream and a StreamReader for reading from the source file
		$stream = New-Object System.IO.MemoryStream
		$stream = $sourceBlob.OpenRead()
		$sReader = New-Object System.IO.StreamReader($stream)
		
		# Define a MemoryStream and a StreamWriter for writing into the destination file
		$memStream = New-Object System.IO.MemoryStream
		$writeStream = New-Object System.IO.StreamWriter $memStream
		
		# process the source blob
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

5. 单击“运行脚本”或按 **F5** 键以运行该脚本。
6. 若要检查修改后的数据文件，可以使用 Azure 门户、Azure 存储资源管理器工具或 Azure PowerShell。[HDInsight 入门][hdinsight-get-started]中有一个关于使用 Azure PowerShell 下载文件并显示文件内容的代码示例。


##使用 PowerShell 来运行 Sqoop 导出

在本节中，你将使用 Azure PowerShell 来运行 Sqoop 导出命令，以将一个 Hive 表和一个数据文件都导出到 Azure SQL 数据库或 SQL Server。下一节会提供一个 HDInsight .NET 示例。

> [AZURE.NOTE]除了连接字符串信息，此节中的步骤还应适用于 Azure SQL 数据库或 SQL Server。这些步骤已使用以下配置测试过：
> 
> * **Azure 虚拟网络点到站点配置**：虚拟网络已将 HDInsight 群集连接到专用数据中心的 SQL Server。有关详细信息，请参阅[在管理门户中配置点到站点 VPN](/documentation/articles/vpn-gateway-point-to-site-create)。
> * **Azure HDInsight 3.1**：有关在虚拟网络上创建群集的信息，请参阅[在 HDInsight 中使用自定义选项预配 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters)。
> * **SQL Server 2014**：已配置为允许身份验证和运行 VPN 客户端配置包，可以安全地连接到虚拟网络。

**导出 log4j 日志文件**

1. 打开 Windows PowerShell ISE。
2. 在底部窗格中，运行以下命令以连接到 Azure 订阅：

		Add-AzureAccount -Environment AzureChinaCloud

	系统将提示你输入 Azure 帐户凭据。

3. 将以下脚本复制到脚本窗格，然后设置前七个变量：

		# Define the cluster variables
		$clusterName = "<HDInsightClusterName>"
		$storageAccountName = "<AzureStorageAccount>"
		$containerName = "<BlobStorageContainerName>"
		
		# Define the SQL database variables
		$sqlDatabaseServerName = "<SQLDatabaseServerName>"
		$sqlDatabaseLogin = "<SQLDatabaseUsername>"
		$sqlDatabasePassword = "<SQLDatabasePassword>"
		$databaseName = "<SQLDatabaseName>"

		$tableName_log4j = "log4jlogs"
		
		# Connection string for Azure SQL Database.
		# Comment if using SQL Server
		$connectionString = "jdbc:sqlserver://$sqlDatabaseServerName.database.chinacloudapi.cn;user=$sqlDatabaseLogin@$sqlDatabaseServerName;password=$sqlDatabasePassword;database=$databaseName"
		# Connection string for SQL Server.
		# Uncomment if using SQL Server.
		#$connectionString = "jdbc:sqlserver://$sqlDatabaseServerName;user=$sqlDatabaseLogin;password=$sqlDatabasePassword;database=$databaseName"
		
		$exportDir_log4j = "/tutorials/usesqoop/data"
	
	有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)部分。

	请注意，$exportDir\_log4j 没有指定 sample.log 文件的文件名。Sqoop 将从该文件夹下的所有文件中导出数据。

4. 将以下脚本追加到脚本窗格中：

		# Submit a Sqoop job
		$sqoopDef = New-AzureHDInsightSqoopJobDefinition -Command "export --connect $connectionString --table $tableName_log4j --export-dir $exportDir_log4j --input-fields-terminated-by \0x20 -m 1"
		$sqoopJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $sqoopDef #-Debug -Verbose
		Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 -Job $sqoopJob
		
		Write-Host "Standard Error" -BackgroundColor Green
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $sqoopJob.JobId -StandardError
		Write-Host "Standard Output" -BackgroundColor Green
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $sqoopJob.JobId -StandardOutput

	请注意，字段分隔符为 **\\0x20**，它是空格。该分隔符在 Azure PowerShell 脚本的 sample.log 文件中定义。若要了解有关 **-m 1** 的信息，请参阅 [Sqoop 用户指南][sqoop-user-guide-1.4.4]。

5. 单击“运行脚本”或按 **F5** 键以运行该脚本。
6. 使用 [Azure 门户][azure-management-portal]检查导出的数据。

**导出 hivesampletable Hive 表**

1. 打开 Windows PowerShell ISE。
2. 在底部窗格中，运行以下命令以连接到 Azure 订阅：

		Add-AzureAccount -Environment AzureChinaCloud

	系统将提示你输入 Azure 帐户凭据。

3. 将以下脚本复制到脚本窗格，然后设置前七个变量：

		# Define the cluster variables
		$clusterName = "<HDInsightClusterName>"
		$storageAccountName = "<AzureStorageAccount>"
		$containerName = "<BlobStorageContainerName>"
		
		# Define the SQL database variables
		$sqlDatabaseServerName = "<SQLDatabaseServerName>"
		$sqlDatabaseLogin = "<SQLDatabaseUsername>"
		$sqlDatabasePassword = "SQLDatabasePassword>"
		$databaseName = "SQLDatabaseName"

		$tableName_mobile = "mobiledata"
		
		# Connection string for Azure SQL Database.
		# Comment if using SQL Server
		$connectionString = "jdbc:sqlserver://$sqlDatabaseServerName.database.chinacloudapi.cn;user=$sqlDatabaseLogin@$sqlDatabaseServerName;password=$sqlDatabasePassword;database=$databaseName"
		# Connection string for SQL Server.
		# Uncomment if using SQL Server
		#$connectionString = "jdbc:sqlserver://$sqlDatabaseServerName;user=$sqlDatabaseLogin;password=$sqlDatabasePassword;database=$databaseName"
		
		$exportDir_mobile = "/hive/warehouse/hivesampletable"
	
	有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)部分。

4. 将以下脚本追加到脚本窗格中：
		
		$sqoopDef = New-AzureHDInsightSqoopJobDefinition -Command "export --connect $connectionString --table $tableName_mobile --export-dir $exportDir_mobile --fields-terminated-by \t -m 1"
		
		
		$sqoopJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $sqoopDef #-Debug -Verbose
		Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 -Job $sqoopJob
		
		Write-Host "Standard Error" -BackgroundColor Green
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $sqoopJob.JobId -StandardError
		Write-Host "Standard Output" -BackgroundColor Green
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $sqoopJob.JobId -StandardOutput

5. 单击“运行脚本”或按 **F5** 键以运行该脚本。
6. 使用 [Azure 门户][azure-management-portal]检查导出的数据。



##使用 HDInsight .NET SDK 来运行 Sqoop 导出

下面是一个使用 HDInsight .NET SDK 运行 Sqoop 导出的 C# 示例。有关使用 HDInsight .NET SDK 的常规信息，请参阅[以编程方式提交 Hadoop 作业][hdinsight-submit-jobs]。


	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Text;
	using System.Threading.Tasks;
	using System.IO;
	using System.Threading;
	using System.Security.Cryptography.X509Certificates;
	using Microsoft.WindowsAzure.Management.HDInsight;
	using Microsoft.Hadoop.Client;
	
	namespace sqoopSDKSample
	{
	    class Program
	    {
	        static void Main(string[] args)
	        {
	            // Set the variables
	            string subscriptionID = "<AzureSubscriptionID>";
	            string clusterName = "<HDInsightClusterName>";
	            string certFriendlyName = "<AzureCertificateFriendlyName>";
	            string sqlDatabaseServerName = "<SQLDatabaseServerName>";
	            string sqlDatabaseLogin = "<SQLDatabaseLogin>";
	            string sqlDatabaseLoginPassword = "<SQLDatabaseLoginPassword>";
	            string sqlDatabaseDatabaseName = "hdisqoop";
	            string sqlDatabaseTableName = "log4jlogs";
	
	            cmdExport = @"export";
				// Connection string for using Azure SQL Database.
				// Comment if using SQL Server
	            cmdExport = cmdExport + @" --connect jdbc:sqlserver://" + sqlDatabaseServerName + ".database.chinacloudapi.cn;user=" + sqlDatabaseLogin + "@" + sqlDatabaseServerName + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
				// Connection string for using SQL Server.
				// Uncomment if using SQL Server
				//cmdExport = cmdExport + @" --connect jdbc:sqlserver://" + sqlDatabaseServerName + ";user=" + sqlDatabaseLogin + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
	            cmdExport = cmdExport + @" --table " + sqlDatabaseTableName;
	            cmdExport = cmdExport + @" --export-dir /tutorials/usesqoop/data";
	            cmdExport = cmdExport + @" --input-fields-terminated-by \0x20 -m 1";
	
	            SqoopJobCreateParameters sqoopJobDefinition = new SqoopJobCreateParameters()
	            {
	                Command = cmdExport,
	                StatusFolder = "/tutorials/usesqoop/jobStatus"
	            };
	
	            // Get the certificate object from certificate store using the friendly name to identify it
	            X509Store store = new X509Store();
	            store.Open(OpenFlags.ReadOnly);
	            X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>().First(item => item.FriendlyName == certFriendlyName);
	            JobSubmissionCertificateCredential creds = new JobSubmissionCertificateCredential(new Guid(subscriptionID), cert, clusterName);
	
	            // Submit the Hive job
	            var jobClient = JobSubmissionClientFactory.Connect(creds);
	            JobCreationResults jobResults = jobClient.CreateSqoopJob(sqoopJobDefinition);
	
	            // Wait for the job to complete
	            WaitForJobCompletion(jobResults, jobClient);
	
	            // Print the Hive job output
	            System.IO.Stream stream = jobClient.GetJobErrorLogs(jobResults.JobId);
	
	            StreamReader reader = new StreamReader(stream);
	            Console.WriteLine(reader.ReadToEnd());
	
	            Console.WriteLine("Press ENTER to continue.");
	            Console.ReadLine();
	        }
	
	        private static void WaitForJobCompletion(JobCreationResults jobResults, IJobSubmissionClient client)
	        {
	            JobDetails jobInProgress = client.GetJob(jobResults.JobId);
	            while (jobInProgress.StatusCode != JobStatusCode.Completed && jobInProgress.StatusCode != JobStatusCode.Failed)
	            {
	                jobInProgress = client.GetJob(jobInProgress.JobId);
	                Thread.Sleep(TimeSpan.FromSeconds(10));
	            }
	        }
	    }
	}

若要执行脚本文件，可将：

	Command = cmdExport,

 替换为：

	File = "/tutorials/usesqoop/sqoopexport.txt",

脚本文件必须位于 Azure Blob 存储中。




##使用 Azure PowerShell 来运行 Sqoop 导入

在本节中，你要将 log4j 日志（已导出到 Azure SQL 数据库）导回到 HDInsight 中。

1. 打开 Windows PowerShell ISE。
2. 在底部窗格中，运行以下命令以连接到 Azure 订阅：

		Add-AzureAccount -Environment AzureChinaCloud

	系统将提示你输入 Azure 帐户凭据。

3. 将以下脚本复制到脚本窗格，然后设置前七个变量：

		# Define the cluster variables
		$clusterName = "<HDInsightClusterName>"
		$storageAccountName = "<AzureStorageAccount>"
		$containerName = "<BlobStorageContainerName>"
		
		# Define the SQL database variables
		$sqlDatabaseServerName = "<SQLDatabaseServerName>"
		$sqlDatabaseLogin = "<SQLDatabaseUsername>"
		$sqlDatabasePassword = "SQLDatabasePassword>"
		$databaseName = "SQLDatabaseName"

		$tableName_log4j = "log4jlogs"
		
		# Connection string for Azure SQL Database
		# Comment if using SQL Server
		$connectionString = "jdbc:sqlserver://$sqlDatabaseServerName.database.chinacloudapi.cn;user=$sqlDatabaseLogin@$sqlDatabaseServerName;password=$sqlDatabasePassword;database=$databaseName"
		# Connection string for SQL Server
		# Uncomment if using SQL Server
		#$connectionString = "jdbc:sqlserver://$sqlDatabaseServerName;user=$sqlDatabaseLogin;password=$sqlDatabasePassword;database=$databaseName"
		
		$tableName_mobile = "mobiledata"
		$targetDir_mobile = "/tutorials/usesqoop/importeddata/"
	
	有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)部分。

4. 将以下脚本追加到脚本窗格中：
	
		$sqoopDef = New-AzureHDInsightSqoopJobDefinition -Command "import --connect $connectionString --table $tableName_mobile --target-dir $targetDir_mobile --fields-terminated-by \t --lines-terminated-by \n -m 1"
		
		$sqoopJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $sqoopDef #-Debug -Verbose
		Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 -Job $sqoopJob
		
		Write-Host "Standard Error" -BackgroundColor Green
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $sqoopJob.JobId -StandardError
		Write-Host "Standard Output" -BackgroundColor Green
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $sqoopJob.JobId -StandardOutput

5. 单击“运行脚本”或按 **F5** 键以运行该脚本。
6. 若要检查修改后的数据文件，可以使用 Azure 门户、Azure 存储资源管理器工具或 Azure PowerShell。[HDInsight 入门][hdinsight-get-started]中有一个关于使用 Azure PowerShell 下载文件并显示文件内容的代码示例。

##后续步骤

现在你已经学习了如何使用 Sqoop。若要了解更多信息，请参阅以下文章：

- [将 Oozie 与 HDInsight 配合使用][hdinsight-use-oozie]：在 Oozie 工作流中使用 Sqoop 操作。
- [使用 HDInsight 分析航班延误数据][hdinsight-analyze-flight-data]：使用 Hive 分析航班延误数据，然后使用 Sqoop 将数据导出到 Azure SQL 数据库。
- [将数据上载到 HDInsight][hdinsight-upload-data]：了解将数据上载到 HDInsight/Azure Blob 存储的其他方法。


 

[azure-management-portal]: https://manage.windowsazure.cn/

[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/
[hdinsight-get-started]: /documentation/articles/hdinsight-get-started/
[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage/
[hdinsight-analyze-flight-data]: /documentation/articles/hdinsight-analyze-flight-delay-data/
[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/

[sqldatabase-get-started]: /documentation/articles/sql-database-get-started/
[sqldatabase-create-configue]: /documentation/articles/sql-database-create-configure/

[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
[powershell-install]: /documentation/articles/install-configure-powershell
[powershell-script]: http://technet.microsoft.com/zh-cn/library/ee176949.aspx

[sqoop-user-guide-1.4.4]: https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html

<!---HONumber=71-->