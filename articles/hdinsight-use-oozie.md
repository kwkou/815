<properties
	pageTitle="在 HDInsight 中使用 Hadoop Oozie | Azure"
	description="在 HDInsight 中使用大数据服务 Hadoop Oozie。了解如何定义 Oozie 工作流，并提交 Oozie 作业。"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="03/04/2016"
	wacn.date="05/24/2016"/>


# 在 HDInsight 中将 Oozie 与 Hadoop 配合使用以定义和运行工作流

[AZURE.INCLUDE [oozie-selector](../includes/hdinsight-oozie-selector.md)]

了解如何定义工作流，以及如何在 HDInsight 上运行工作流。若要了解 Oozie 协调器，请参阅[将基于时间的 Hadoop Oozie 协调器与 HDInsight 配合使用][hdinsight-oozie-coordinator-time]。

Apache Oozie 是一个管理 Hadoop 作业的工作流/协调系统。它与 Hadoop 堆栈集成，支持 Apache MapReduce、Apache Pig、Apache Hive 和 Apache Sqoop 的 Hadoop 作业。它也能用于安排特定于某系统的作业，例如 Java 程序或 shell 脚本。

你要根据本教程中的说明实现的工作流包含两个操作：

![工作流关系图][img-workflow-diagram]

1. Hive 操作运行 HiveQL 脚本以统计 log4j 文件中每个日志级类型的次数。每个 log4j 文件都包含一行字段，其中包含 [LOG LEVEL] 字段用于显示类型和严重性，例如：

		2012-02-03 18:35:34 SampleClass6 [INFO] everything normal for id 577725851
		2012-02-03 18:35:34 SampleClass4 [FATAL] system problem at id 1991281254
		2012-02-03 18:35:34 SampleClass3 [DEBUG] detail for id 1304807656
		...

	该 Hive 脚本的输出结果类似于：

		[DEBUG] 434
		[ERROR] 3
		[FATAL] 1
		[INFO]  96
		[TRACE] 816
		[WARN]  4

	有关 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]。

2.  Sqoop 操作将 HiveQL 输出结果导出到 Azure SQL 数据库中的表。有关 Sqoop 的详细信息，请参阅[将 Hadoop Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]。

> [AZURE.NOTE]有关在 HDInsight 群集上支持的 Oozie 版本，请参阅 [HDInsight 提供的 Hadoop 群集版本有哪些新增功能？][hdinsight-versions]。

###<a name="prerequisites"></a>先决条件

在开始阅读本教程前，你必须具有：

- **配备 Azure PowerShell 的工作站**。

	[AZURE.INCLUDE [upgrade-powershell](../includes/hdinsight-use-latest-powershell.md)]

	若要执行 Windows PowerShell 脚本，必须以管理员身份运行并将执行策略设为 *RemoteSigned*。有关详细信息，请参阅[运行 Windows PowerShell 脚本][powershell-script]。

##定义 Oozie 工作流及相关 HiveQL 脚本

Oozie 工作流定义是用 hPDL（一种 XML 过程定义语言）编写的。默认的工作流文件名为 *workflow.xml*。以下是你要在本教程中使用的工作流文件。

	<workflow-app name="useooziewf" xmlns="uri:oozie:workflow:0.2">
		<start to = "RunHiveScript"/>

		<action name="RunHiveScript">
			<hive xmlns="uri:oozie:hive-action:0.2">
				<job-tracker>${jobTracker}</job-tracker>
				<name-node>${nameNode}</name-node>
				<configuration>
					<property>
						<name>mapred.job.queue.name</name>
						<value>${queueName}</value>
					</property>
				</configuration>
				<script>${hiveScript}</script>
				<param>hiveTableName=${hiveTableName}</param>
				<param>hiveDataFolder=${hiveDataFolder}</param>
				<param>hiveOutputFolder=${hiveOutputFolder}</param>
			</hive>
			<ok to="RunSqoopExport"/>
			<error to="fail"/>
		</action>

		<action name="RunSqoopExport">
			<sqoop xmlns="uri:oozie:sqoop-action:0.2">
				<job-tracker>${jobTracker}</job-tracker>
				<name-node>${nameNode}</name-node>
				<configuration>
					<property>
						<name>mapred.compress.map.output</name>
						<value>true</value>
					</property>
				</configuration>
			<arg>export</arg>
			<arg>--connect</arg>
			<arg>${sqlDatabaseConnectionString}</arg>
			<arg>--table</arg>
			<arg>${sqlDatabaseTableName}</arg>
			<arg>--export-dir</arg>
			<arg>${hiveOutputFolder}</arg>
			<arg>-m</arg>
			<arg>1</arg>
			<arg>--input-fields-terminated-by</arg>
			<arg>"\001"</arg>
			</sqoop>
			<ok to="end"/>
			<error to="fail"/>
		</action>

		<kill name="fail">
			<message>Job failed, error message[${wf:errorMessage(wf:lastErrorNode())}] </message>
		</kill>

		<end name="end"/>
	</workflow-app>

该工作流中定义了两个操作。start-to 操作是 *RunHiveScript*。如果该操作成功运行，则下一个操作是 *RunSqoopExport*。

RunHiveScript 有几个变量。在从工作站使用 Azure PowerShell 提交 Oozie 作业时，将会传递值。

<table border = "1">
<tr><th>工作流变量</th><th>说明</th></tr>
<tr><td>${jobTracker}</td><td>指定 Hadoop 作业跟踪器的 URL。在 HDInsight 版本 3.0 和 2.1 中使用 <strong>jobtrackerhost:9010</strong>。</td></tr>
<tr><td>${nameNode}</td><td>指定 Hadoop 名称节点的 URL。请使用默认文件系统地址，例如 <i>wasb://&lt;containerName>@&lt;storageAccountName>.blob.core.chinacloudapi.cn</i>。</td></tr>
<tr><td>${queueName}</td><td>指定要将作业提交到的队列名称。使用<strong>默认值</strong>。</td></tr>
</table>

<table border = "1">
<tr><th>Hive 操作变量</th><th>说明</th></tr>
<tr><td>${hiveDataFolder}</td><td>指定 Hive Create Table 命令的源目录。</td></tr>
<tr><td>${hiveOutputFolder}</td><td>指定 INSERT OVERWRITE 语句的输出文件夹。</td></tr>
<tr><td>${hiveTableName}</td><td>指定引用 log4j 数据文件的 Hive 表的名称。</td></tr>
</table>

<table border = "1">
<tr><th>Sqoop 操作变量</th><th>说明</th></tr>
<tr><td>${sqlDatabaseConnectionString}</td><td>指定 Azure SQL 数据库连接字符串。</td></tr>
<tr><td>${sqlDatabaseTableName}</td><td>指定要将数据导出到的 Azure SQL 数据库表。</td></tr>
<tr><td>${hiveOutputFolder}</td><td>指定 Hive INSERT OVERWRITE 语句的输出文件夹。这是用于 Sqoop 导出 (export-dir) 的同一个文件夹。</td></tr>
</table>

有关 Oozie 工作流和使用工作流操作的详细信息，请参阅 [Apache Oozie 4.0 文档][apache-oozie-400]（适用于 HDInsight 3.0 版）或 [Apache Oozie 3.3.2 文档][apache-oozie-332]（适用于 HDInsight 2.1 版）。

2. 使用 ANSI (ASCII) 编码将文件另存为 **C:\\Tutorials\\UseOozie\\workflow.xml**。（如果你的文本编辑器不提供此选项，则使用记事本。）
	
##部署 Oozie 项目并准备教程

你将运行 Windows PowerShell 脚本来执行以下操作：

- 将 HiveQL 脚本 (useoozie.hql) 复制到 Azure 存储空间 (wasb:///tutorials/useoozie/useoozie.hql)。
- 将 workflow.xml 复制到 wasb:///tutorials/useoozie/workflow.xml。
- 将数据文件 (/example/data/sample.log) 复制到 wasb:///tutorials/useoozie/data/sample.log。 
- 创建用于存储 Sqoop 导出数据的 Azure SQL 数据库表。表的名称为 *log4jLogCount*。

**了解 HDInsight 存储**

HDInsight 使用 Azure 存储空间中的 Blob 来存储数据。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

设置 HDInsight 群集时，请将 Azure 存储帐户和该帐户上的特定 Blob 容器指定为默认文件系统，就像在 HDFS 中一样。除了此存储帐户外，在设置过程中，你还可以从同一 Azure 订阅或不同 Azure 订阅添加其他存储帐户。有关添加其他存储帐户的详细信息，请参阅[在 HDInsight 中预配 Hadoop 群集][hdinsight-provision]。为了简化本教程中使用的 Windows PowerShell 脚本，所有文件都存储在默认文件存储容器（位于 */tutorials/useoozie*）中。默认情况下，此容器与 HDInsight 群集同名。语法为：

	wasb[s]://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/<path>/<filename>

> [AZURE.NOTE]HDInsight 版本 3.0 群集仅支持 **wasb://* 语法。较早的 **asv://* 语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.0 群集中不受支持。

> [AZURE.NOTE]wasb:// 路径是虚拟路径。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

存储在默认文件系统容器中的文件可以使用以下任一 URI 从 HDInsight 进行访问（以下代码使用 workflow.xml 为例）：

	wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn/tutorials/useoozie/workflow.xml
	wasb:///tutorials/useoozie/workflow.xml
	/tutorials/useoozie/workflow.xml

如果要从存储帐户直接访问该文件，则请注意，该文件的 Blob 名称是：

	tutorials/useoozie/workflow.xml

**了解 Hive 内部表和外部表**

以下是你需要了解的有关 Hive 内部表和外部表的一些信息：

- CREATE TABLE 命令创建内部表，也称为托管表。数据文件必须位于默认容器中。
- CREATE TABLE 命令将数据文件移动到默认容器中的 /hive/warehouse/<TableName> 文件夹。
- CREATE EXTERNAL TABLE 命令创建外部表。数据文件可以位于默认容器以外的位置。
- CREATE EXTERNAL TABLE 命令不移动数据文件。
- CREATE EXTERNAL TABLE 命令不允许 LOCATION 子句中指定的文件夹下有任何子文件夹。这是本教程生成 sample.log 文件的副本的原因。

有关详细信息，请参阅 [HDInsight：Hive 内部和外部表简介][cindygross-hive-tables]。

**准备教程**

1. 打开 Windows PowerShell ISE。（在 Windows 8 的“开始”屏幕上键入 **PowerShell_ISE**，然后单击“Windows PowerShell ISE”。有关详细信息，请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][powershell-start]）。
2. 在底部窗格中，运行以下命令以连接到 Azure 订阅：

		Add-AzureAccount -Environment AzureChinaCloud

	系统将提示你输入 Azure 帐户凭据。这种添加订阅连接的方法会超时，12 个小时之后，你将需要再次运行该 cmdlet。

	> [AZURE.NOTE]如果你有多个 Azure 订阅，而默认订阅不是你想使用的，则请使用 <strong>Select-AzureSubscription</strong> cmdlet 来选择正确的订阅。

3. 将以下脚本复制到脚本窗格，然后设置前六个变量：
			
		# WASB variables
		$storageAccountName = "<StorageAccountName>"
		$containerName = "<BlobStorageContainerName>"
		
		# SQL database variables
		$sqlDatabaseServer = "<SQLDatabaseServerName>"  
		$sqlDatabaseLogin = "<SQLDatabaseLoginName>"
		$sqlDatabaseLoginPassword = "SQLDatabaseLoginPassword>"
		$sqlDatabaseName = "<SQLDatabaseName>"  
		$sqlDatabaseTableName = "log4jLogsCount"
		
		# Oozie files for the tutorial	
		$workflowDefinition = "C:\Tutorials\UseOozie\workflow.xml"
		$hiveQLScript = "C:\Tutorials\UseOozie\useooziewf.hql"
		
		# WASB folder for storing the Oozie tutorial files.
		$destFolder = "tutorials/useoozie"  # Do NOT use the long path here


	有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)部分。

3. 在脚本窗格中将以下内容追加到脚本：
		
		# Create a storage context object
		$storageaccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}
		$destContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageAccountName -StorageAccountKey $storageaccountkey
		
		function uploadOozieFiles()
		{		
		    Write-Host "Copy workflow definition and HiveQL script file ..." -ForegroundColor Green
			Set-AzureStorageBlobContent -File $workflowDefinition -Container $containerName -Blob "$destFolder/workflow.xml" -Context $destContext
			Set-AzureStorageBlobContent -File $hiveQLScript -Container $containerName -Blob "$destFolder/useooziewf.hql" -Context $destContext
		}
				
		function prepareHiveDataFile()
		{
			Write-Host "Make a copy of the sample.log file ... " -ForegroundColor Green
			Start-CopyAzureStorageBlob -SrcContainer $containerName -SrcBlob "example/data/sample.log" -Context $destContext -DestContainer $containerName -destBlob "$destFolder/data/sample.log" -DestContext $destContext
		}
				
		function prepareSQLDatabase()
		{
			# SQL query string for creating log4jLogsCount table
			$cmdCreateLog4jCountTable = " CREATE TABLE [dbo].[$sqlDatabaseTableName](
				    [Level] [nvarchar](10) NOT NULL,
				    [Total] float,
				CONSTRAINT [PK_$sqlDatabaseTableName] PRIMARY KEY CLUSTERED   
				(
				[Level] ASC
				)
				)"
				
			#Create the log4jLogsCount table
		    Write-Host "Create Log4jLogsCount table ..." -ForegroundColor Green
			$conn = New-Object System.Data.SqlClient.SqlConnection
			$conn.ConnectionString = "Data Source=$sqlDatabaseServer.devdatabase.chinacloudapi.cn;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseLogin;Password=$sqlDatabaseLoginPassword;Encrypt=true;Trusted_Connection=false;"
			$conn.open()
			$cmd = New-Object System.Data.SqlClient.SqlCommand
			$cmd.connection = $conn
			$cmd.commandtext = $cmdCreateLog4jCountTable
			$cmd.executenonquery()
				
			$conn.close()
		}
				
		# upload workflow.xml, coordinator.xml, and ooziewf.hql
		uploadOozieFiles;
				
		# make a copy of example/data/sample.log to example/data/log4j/sample.log
		prepareHiveDataFile;
		
		# create log4jlogsCount table on SQL database
		prepareSQLDatabase;

4. 单击“运行脚本”或按 **F5** 键以运行该脚本。输出将类似于下面：

	![教程准备的输出结果][img-preparation-output]

##运行 Oozie 项目

Azure PowerShell 目前不提供任何用于定义 Oozie 作业的 cmdlet。你可以使用 **Invoke-RestMethod** cmdlet 来调用 Oozie Web 服务。Oozie Web 服务 API 是 HTTP REST JSON API。有关 Oozie Web 服务 API 的详细信息，请参阅 [Apache Oozie 4.0 文档][apache-oozie-400]（用于 HDInsight 版本 3.0）或 [Apache Oozie 3.3.2 文档][apache-oozie-332]（用于 HDInsight 版本 2.1）。

**提交 Oozie 作业**

1. 打开 Windows PowerShell ISE。（在 Windows 8 的“开始”屏幕上键入 **PowerShell_ISE**，然后单击“Windows PowerShell ISE”。有关详细信息，请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][powershell-start]）。

3. 将以下脚本复制到脚本窗格，然后设置前 10 个变量（跳过 $storageUri 变量）。

		#HDInsight cluster variables
		$clusterName = "<HDInsightClusterName>"
		$clusterUsername = "<HDInsightClusterUsername>"
		$clusterPassword = "<HDInsightClusterUserPassword>"
		
		#Azure Blob storage variables
		$storageAccountName = "<StorageAccountName>"
		$storageContainerName = "<BlobContainerName>"
		$storageUri="wasb://$storageContainerName@$storageAccountName.blob.core.chinacloudapi.cn"
		
		#Azure SQL database variables
		$sqlDatabaseServer = "<SQLDatabaseServerName>"
		$sqlDatabaseLogin = "<SQLDatabaseLoginName>"
		$sqlDatabaseLoginPassword = "<SQLDatabaseloginPassword>"
		$sqlDatabaseName = "<SQLDatabaseName>"  
		
		#Oozie WF variables
		$oozieWFPath="$storageUri/tutorials/useoozie"  # The default name is workflow.xml. And you don't need to specify the file name.
		$waitTimeBetweenOozieJobStatusCheck=10
		
		#Hive action variables
		$hiveScript = "$storageUri/tutorials/useoozie/useooziewf.hql"
		$hiveTableName = "log4jlogs"
		$hiveDataFolder = "$storageUri/tutorials/useoozie/data"
		$hiveOutputFolder = "$storageUri/tutorials/useoozie/output"
		
		#Sqoop action variables
		$sqlDatabaseConnectionString = "jdbc:sqlserver://$sqlDatabaseServer.database.chinacloudapi.cn;user=$sqlDatabaseLogin@$sqlDatabaseServer;password=$sqlDatabaseLoginPassword;database=$sqlDatabaseName"
		$sqlDatabaseTableName = "log4jLogsCount"

		$passwd = ConvertTo-SecureString $clusterPassword -AsPlainText -Force
		$creds = New-Object System.Management.Automation.PSCredential ($clusterUsername, $passwd)


	有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)部分。

3. 将以下内容追加到脚本。此脚本定义 Oozie 负载。
		
		#OoziePayload used for Oozie web service submission
		$OoziePayload =  @"
		<?xml version="1.0" encoding="UTF-8"?>
		<configuration>
		
		   <property>
		       <name>nameNode</name>
		       <value>$storageUrI</value>
		   </property>
		
		   <property>
		       <name>jobTracker</name>
		       <value>jobtrackerhost:9010</value>
		   </property>
		
		   <property>
		       <name>queueName</name>
		       <value>default</value>
		   </property>
		
		   <property>
		       <name>oozie.use.system.libpath</name>
		       <value>true</value>
		   </property>
		
		   <property>
		       <name>hiveScript</name>
		       <value>$hiveScript</value>
		   </property>
		
		   <property>
		       <name>hiveTableName</name>
		       <value>$hiveTableName</value>
		   </property>
		
		   <property>
		       <name>hiveDataFolder</name>
		       <value>$hiveDataFolder</value>
		   </property>
		
		   <property>
		       <name>hiveOutputFolder</name>
		       <value>$hiveOutputFolder</value>
		   </property>
		
		   <property>
		       <name>sqlDatabaseConnectionString</name>
		       <value>";$sqlDatabaseConnectionString";</value>
		   </property>
		
		   <property>
		       <name>sqlDatabaseTableName</name>
		       <value>$SQLDatabaseTableName</value>
		   </property>
		
		   <property>
		       <name>user.name</name>
		       <value>$clusterUsername</value>
		   </property>
		
		   <property>
		       <name>oozie.wf.application.path</name>
		       <value>$oozieWFPath</value>
		   </property>
		
		</configuration>
		"@
		
4. 将以下内容追加到脚本。此脚本检查 Oozie Web 服务状态。
			
	    Write-Host "Checking Oozie server status..." -ForegroundColor Green
	    $clusterUriStatus = "https://$clusterName.azurehdinsight.cn:443/oozie/v2/admin/status"
	    $response = Invoke-RestMethod -Method Get -Uri $clusterUriStatus -Credential $creds -OutVariable $OozieServerStatus 
	    
	    $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
	    $oozieServerSatus = $jsonResponse[0].("systemMode")
	    Write-Host "Oozie server status is $oozieServerSatus..."
	
5. 将以下内容追加到脚本。这部分创建并启动一项 Oozie 作业：

	    # create Oozie job
	    Write-Host "Sending the following Payload to the cluster:" -ForegroundColor Green
	    Write-Host "`n--------`n$OoziePayload`n--------"
	    $clusterUriCreateJob = "https://$clusterName.azurehdinsight.cn:443/oozie/v2/jobs"
	    $response = Invoke-RestMethod -Method Post -Uri $clusterUriCreateJob -Credential $creds -Body $OoziePayload -ContentType "application/xml" -OutVariable $OozieJobName #-debug
	
	    $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
	    $oozieJobId = $jsonResponse[0].("id")
	    Write-Host "Oozie job id is $oozieJobId..."
	
	    # start Oozie job
	    Write-Host "Starting the Oozie job $oozieJobId..." -ForegroundColor Green
	    $clusterUriStartJob = "https://$clusterName.azurehdinsight.cn:443/oozie/v2/job/" + $oozieJobId + "?action=start"
	    $response = Invoke-RestMethod -Method Put -Uri $clusterUriStartJob -Credential $creds | Format-Table -HideTableHeaders #-debug
		
6. 将以下内容追加到脚本。此脚本检查 Oozie 作业状态。

	    # get job status
	    Write-Host "Sleeping for $waitTimeBetweenOozieJobStatusCheck seconds until the job metadata is populated in the Oozie metastore..." -ForegroundColor Green
	    Start-Sleep -Seconds $waitTimeBetweenOozieJobStatusCheck
	
	    Write-Host "Getting job status and waiting for the job to complete..." -ForegroundColor Green
	    $clusterUriGetJobStatus = "https://$clusterName.azurehdinsight.cn:443/oozie/v2/job/" + $oozieJobId + "?show=info"
	    $response = Invoke-RestMethod -Method Get -Uri $clusterUriGetJobStatus -Credential $creds 
	    $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
	    $JobStatus = $jsonResponse[0].("status")
	
	    while($JobStatus -notmatch "SUCCEEDED|KILLED")
	    {
	        Write-Host "$(Get-Date -format 'G'): $oozieJobId is in $JobStatus state...waiting $waitTimeBetweenOozieJobStatusCheck seconds for the job to complete..."
	        Start-Sleep -Seconds $waitTimeBetweenOozieJobStatusCheck
	        $response = Invoke-RestMethod -Method Get -Uri $clusterUriGetJobStatus -Credential $creds 
	        $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
	        $JobStatus = $jsonResponse[0].("status")
	    }
	
	    Write-Host "$(Get-Date -format 'G'): $oozieJobId is in $JobStatus state!" -ForegroundColor Green

7. 如果你的 HDinsight 群集是 2.1 版的，请将“https://$clusterName.azurehdinsight.cn:443/oozie/v2/”替换为“https://$clusterName.azurehdinsight.cn:443/oozie/v1/”。HDInsight 群集版本 2.1 不支持 Web 服务的版本 2。

8. 单击“运行脚本”或按 **F5** 键以运行该脚本。输出将类似于下面：

	![教程运行工作流输出][img-runworkflow-output]

8. 连接到 Azure SQL 数据库以查看导出的数据。

**检查作业错误日志**

若要对工作流进行故障排除，可从群集头节点找到 Oozie 日志文件，位置是 *C:\\apps\\dist\\oozie-3.3.2.1.3.2.0-05\\oozie-win-distro\\logs\\Oozie.log* 或 *C:\\apps\\dist\\oozie-4.0.0.2.0.7.0-1528\\oozie-win-distro\\logs\\Oozie.log*。有关 RDP 的信息，请参阅[在 HDInsight 中使用 Azure 经典管理门户管理 Hadoop 群集][hdinsight-admin-portal]。

**重新运行教程**

若要重新运行该工作流，必须删除以下内容：

- Hive 脚本输出文件
- log4jLogsCount 表中的数据

这是你可以使用的一个示例 Windows PowerShell 脚本：

	$storageAccountName = "<AzureStorageAccountName>"
	$containerName = "<ContainerName>"
	
	#SQL database variables
	$sqlDatabaseServer = "<SQLDatabaseServerName>"
	$sqlDatabaseLogin = "<SQLDatabaseLoginName>"
	$sqlDatabaseLoginPassword = "<SQLDatabaseLoginPassword>"
	$sqlDatabaseName = "<SQLDatabaseName>"
	$sqlDatabaseTableName = "log4jLogsCount"
	
	Write-host "Delete the Hive script output file ..." -ForegroundColor Green
	$storageaccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}
	$destContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageAccountName -StorageAccountKey $storageaccountkey
	Remove-AzureStorageBlob -Context $destContext -Blob "tutorials/useoozie/output/000000_0" -Container $containerName
	
	Write-host "Delete all the records from the log4jLogsCount table ..." -ForegroundColor Green
	$conn = New-Object System.Data.SqlClient.SqlConnection
	$conn.ConnectionString = "Data Source=$sqlDatabaseServer.database.chinacloudapi.cn;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseLogin;Password=$sqlDatabaseLoginPassword;Encrypt=true;Trusted_Connection=false;"
	$conn.open()
	$cmd = New-Object System.Data.SqlClient.SqlCommand
	$cmd.connection = $conn
	$cmd.commandtext = "delete from $sqlDatabaseTableName"
	$cmd.executenonquery()
	
	$conn.close()


##后续步骤
在本教程中，你已经学习了如何定义 Oozie 工作流，以及如何使用 Windows PowerShell 运行 Oozie 作业。若要了解更多信息，请参阅下列文章：

- [将基于时间的 Oozie 协调器与 HDInsight 配合使用][hdinsight-oozie-coordinator-time]
- [将 Hadoop 与 HDInsight 中的 Hive 配合使用以分析手机使用情况][hdinsight-get-started]
- [将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]
- [使用 PowerShell 管理 HDInsight][hdinsight-admin-powershell]
- [在 HDInsight 中上载 Hadoop 作业的数据][hdinsight-upload-data]
- [将 Sqoop 与 HDInsight 中的 Hadoop 配合使用][hdinsight-use-sqoop]
- [将 Hive 与 HDInsight 上的 Hadoop 配合使用][hdinsight-use-hive]
- [将 Pig 与 HDInsight 上的 Hadoop 配合使用][hdinsight-use-pig]
- [为 HDInsight 开发 C# Hadoop 流作业][hdinsight-develop-streaming-jobs]
- [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-mapreduce]


[hdinsight-cmdlets-download]: http://go.microsoft.com/fwlink/?LinkID=325563




[hdinsight-oozie-coordinator-time]: /documentation/articles/hdinsight-use-oozie-coordinator-time/
[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning-v1/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/
[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal-v1/


[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-use-mapreduce]: /documentation/articles/hdinsight-use-mapreduce/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-get-started-emulator]: /documentation/articles/hdinsight-hadoop-emulator-get-started/

[hdinsight-develop-streaming-jobs]: /documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/
[hdinsight-develop-mapreduce]: /documentation/articles/hdinsight-develop-deploy-java-mapreduce/

[sqldatabase-create-configue]: /documentation/articles/sql-database-get-started/
[sqldatabase-get-started]: /documentation/articles/sql-database-get-started/

[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account/

[apache-hadoop]: http://hadoop.apache.org/
[apache-oozie-400]: http://oozie.apache.org/docs/4.0.0/
[apache-oozie-332]: http://oozie.apache.org/docs/3.3.2/

[powershell-download]: /zh-cn/downloads/#cmd-line-tools
[powershell-about-profiles]: https://technet.microsoft.com/zh-cn/library/hh847857.aspx
[powershell-install-configure]: /documentation/articles/powershell-install-configure/
[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
[powershell-script]: https://technet.microsoft.com/library/dn425048.aspx

[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

[img-workflow-diagram]: ./media/hdinsight-use-oozie/HDI.UseOozie.Workflow.Diagram.png
[img-preparation-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.Preparation.Output1.png
[img-runworkflow-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.RunWF.Output.png

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx

<!---HONumber=Mooncake_0104_2016-->