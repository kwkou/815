<properties 
	pageTitle="在 HDInsight 中使用基于时间的 Hadoop Oozie 协调器 | Azure" 
	description="在 HDInsight 中使用基于时间的 Hadoop Oozie 协调器（大数据服务）。了解如何定义 Oozie 工作流和协调器，并提交作业。"
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


# 将基于时间的 Oozie 协调器与 HDInsight 中的 Hadoop 配合使用以定义工作流和协调作业

在本文中，你将学习如何定义工作流和协调器，以及如何基于时间触发协调器作业。在阅读本文之前，先浏览[将 Oozie 与 HDInsight 配合使用][hdinsight-use-oozie]一文会很有用。若要了解 Azure 数据工厂，请参阅 [将 Pig 和 Hive 用于数据工厂][azure-data-factory-pig-hive]。

##<a id="whatisoozie"></a>什么是 Oozie

Apache Oozie 是一个管理 Hadoop 作业的工作流/协调系统。它与 Hadoop 堆栈集成，支持 Apache MapReduce、Apache Pig、Apache Hive 和 Apache Sqoop 的 Hadoop 作业。它也能用于安排特定于某系统的作业，例如 Java 程序或 shell 脚本。

下图显示将要实施的工作流：

![工作流关系图][img-workflow-diagram]

工作流包含两个操作：

1. Hive 操作运行 HiveQL 脚本以统计 log4j 日志文件中每个日志级类型的次数。每个 log4j 日志都包含一行字段，其中包含 [LOG LEVEL] 字段，可显示类型和严重性，例如：

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
	
2.  Sqoop 操作将 HiveQL 操作输出结果导出到 Azure SQL 数据库中的表。有关 Sqoop 的详细信息，请参阅[将 Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]。

> [AZURE.NOTE]有关 HDInsight 群集上支持的 Oozie 版本，请参阅 [HDInsight 提供的群集版本有哪些新功能？][hdinsight-versions]。

> [AZURE.NOTE]本教程适用于 HDInsight 群集版本 2.1 和 3.0。本文尚未在 HDInsight Emulator 上测试过。


##<a id="prerequisites"></a>先决条件

在开始阅读本教程前，你必须具有：

- **配备 Azure PowerShell 的工作站**。

	[AZURE.INCLUDE [upgrade-powershell](../includes/hdinsight-use-latest-powershell.md)]

- **一个 HDInsight 群集**。有关创建 HDInsight 群集的信息，请参阅[创建 HDInsight 群集][hdinsight-provision]或 [HDInsight 入门][hdinsight-get-started]。你将需要以下数据才能完成本教程：

	<table border = "1">
	<tr><th>群集属性</th><th>Windows PowerShell 变量名</th><th>值</th><th>说明</th></tr>
	<tr><td>HDInsight 群集名称</td><td>$clusterName</td><td></td><td>要在其中运行本教程的 HDInsight 群集。</td></tr>
	<tr><td>HDInsight 群集用户名</td><td>$clusterUsername</td><td></td><td>HDInsight 群集用户名。</td></tr>
	<tr><td>HDInsight 群集用户的密码 </td><td>$clusterPassword</td><td></td><td>HDInsight 群集用户的密码。</td></tr>
	<tr><td>Azure 存储帐户名称</td><td>$storageAccountName</td><td></td><td>可用于 HDInsight 群集的 Azure 存储帐户。在本教程中，使用在群集设置过程中指定的默认存储帐户。</td></tr>
	<tr><td>Azure Blob 容器名称</td><td>$containerName</td><td></td><td>在此示例中，使用用于默认 HDInsight 群集文件系统的 Azure Blob 存储容器。默认情况下，该容器与 HDInsight 群集同名。</td></tr>
	</table>

- **Azure SQL 数据库**。你必须为 SQL 数据库服务器配置防火墙规则以允许从你的工作站进行访问。有关创建 Azure SQL 数据库和配置防火墙的说明，请参阅 [Azure SQL 数据库入门][sqldatabase-get-started]。本文提供了用于创建本教程所需的 Azure SQL 数据库表的 Windows PowerShell 脚本。

	<table border = "1">
	<tr><th>SQL 数据库属性</th><th>Windows PowerShell 变量名</th><th>值</th><th>说明</th></tr>
	<tr><td>SQL 数据库服务器名称</td><td>$sqlDatabaseServer</td><td></td><td>Sqoop 要将数据导出到其中的 SQL 数据库服务器。</td></tr>
	<tr><td>SQL 数据库登录名</td><td>$sqlDatabaseLogin</td><td></td><td>SQL 数据库登录名。</td></tr>
	<tr><td>SQL 数据库登录密码</td><td>$sqlDatabaseLoginPassword</td><td></td><td>SQL 数据库登录密码。</td></tr>
	<tr><td>SQL 数据库名</td><td>$sqlDatabaseName</td><td></td><td>Sqoop 要将数据导出到其中的 Azure SQL 数据库。</td></tr>
	</table>

	> [AZURE.NOTE]默认情况下，可以从 Azure HDInsight 这样的 Azure 服务连接 Azure SQL 数据库。如果禁用了此防火墙设置，则必须从 Azure 经典管理门户启用它。有关创建 SQL 数据库和配置防火墙规则的说明，请参阅 [创建和配置 SQL 数据库][sqldatabase-create-configure]。


> [AZURE.NOTE]将值填充到表中。这将有助于学习本教程。


##<a id="defineworkflow"></a>定义 Oozie 工作流及相关 HiveQL 脚本

Oozie 工作流定义是用 hPDL（一种 XML 过程定义语言）编写的。默认的工作流文件名为 *workflow.xml*。你将在本地保存该工作流文件，并将在本教程后面使用 Azure PowerShell 将它部署到 HDInsight 群集。

该工作流中的 Hive 操作调用 HiveQL 脚本文件。此脚本文件包含三个 HiveQL 语句：

1. **DROP TABLE 语句**删除 log4j Hive 表（如果存在）。
2. **CREATE TABLE 语句**创建一个 log4j Hive 外部表，该表指向 log4j 日志文件的位置；
3.  **log4j 日志文件的位置**。字段分隔符为“,”。默认分行符为“\\n”。Hive 外部表用于在你想多次运行 Oozie 工作流的情况下避免数据文件从原始位置被删除。
3. **INSERT OVERWRITE 语句**从 log4j Hive 表统计每个日志级类型的次数，并将输出结果保存到 Azure Blob 存储位置。 

**注意**：有一个已知的 Hive 路径问题。你在提交 Oozie 作业时将会遇到这个问题。可在 TechNet Wiki 上找到用于解决此问题的说明：[HDInsight Hive 错误: 无法重命名][technetwiki-hive-error]。

**将 HiveQL 脚本文件定义为由工作流调用**

1. 创建一个内容如下的文本文件：

		DROP TABLE ${hiveTableName};
		CREATE EXTERNAL TABLE ${hiveTableName}(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION '${hiveDataFolder}';
		INSERT OVERWRITE DIRECTORY '${hiveOutputFolder}' SELECT t4 AS sev, COUNT(*) AS cnt FROM ${hiveTableName} WHERE t4 LIKE '[%' GROUP BY t4;

	该脚本中使用了三个变量：

	- ${hiveTableName}
	- ${hiveDataFolder}
	- ${hiveOutputFolder}
			
	工作流定义文件（本教程中的 workflow.xml）在运行时会将三个值传递到这个 HiveQL 脚本。
		
2. 使用 ANSI (ASCII) 编码将文件另存为 **C:\\Tutorials\\UseOozie\\useooziewf.hql**。（如果你的文本编辑器不提供此选项，则使用记事本。） 在本教程的后面，此脚本文件将被部署到 HDInsight 群集。



**定义工作流**

1. 创建一个内容如下的文本文件：

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

	该工作流中定义了两个操作。start-to 操作是 *RunHiveScript*。如果该操作运行*正常*，则下一个操作是 *RunSqoopExport*。

	RunHiveScript 有几个变量。在从工作站使用 Azure PowerShell 提交 Oozie 作业时，将会传递值。

	<table border = "1">
	<tr><th>工作流变量</th><th>说明</th></tr>
	<tr><td>${jobTracker}</td><td>指定 Hadoop 作业跟踪器的 URL。在 HDInsight 群集版本 2.0 和 3.0 上使用 <strong>jobtrackerhost:9010</strong>。</td></tr>
	<tr><td>${nameNode}</td><td>指定 Hadoop 名称节点的 URL。使用默认文件系统 wasb:// 地址，例如 <i>wasb://&lt;containerName>@&lt;storageAccountName>.blob.core.chinacloudapi.cn</i>。</td></tr>
	<tr><td>${queueName}</td><td>指定要将作业提交到的 queuename。使用“默认”。<strong></strong></td></tr>
	</table><table border = "1">
	<tr><th>Hive 操作变量</th><th>说明</th></tr>
	<tr><td>${hiveDataFolder}</td><td>Hive Create Table 命令的源目录。</td></tr>
	<tr><td>${hiveOutputFolder}</td><td>INSERT OVERWRITE 语句的输出文件夹。</td></tr>
	<tr><td>${hiveTableName}</td><td>引用 log4j 数据文件的 Hive 表的名称。</td></tr>
	</table><table border = "1">
	<tr><th>Sqoop 操作变量</th><th>说明</th></tr>
	<tr><td>${sqlDatabaseConnectionString}</td><td>SQL 数据库连接字符串。</td></tr>
	<tr><td>${sqlDatabaseTableName}</td><td>数据将要导出到的 Azure SQL 数据库表。</td></tr>
	<tr><td>${hiveOutputFolder}</td><td>Hive INSERT OVERWRITE 语句的输出文件夹。这是用于 Sqoop 导出 (export-dir) 的同一个文件夹。</td></tr>
	</table>

	有关 Oozie 工作流以及使用工作流操作的详细信息，请参阅 [Apache Oozie 4.0 文档][apache-oozie-400]（用于 HDInsight 群集版本 3.0）或 [Apache Oozie 3.3.2 文档][apache-oozie-332]（用于 HDInsight 群集版本 2.1）。

2. 使用 ANSI (ASCII) 编码将文件另存为 **C:\\Tutorials\\UseOozie\\workflow.xml**。（如果你的文本编辑器不提供此选项，则使用记事本。）

**定义协调器**

1. 创建一个内容如下的文本文件：

		<coordinator-app name="my_coord_app" frequency="${coordFrequency}" start="${coordStart}" end="${coordEnd}" timezone="${coordTimezone}" xmlns="uri:oozie:coordinator:0.4">
		   <action>
		      <workflow>
		         <app-path>${wfPath}</app-path>
		      </workflow>
		   </action>
		</coordinator-app>

	该定义文件中使用了五个变量：

	| 变量 | 说明 |
	| ------------------|------------ |
	| ${coordFrequency} | 作业暂停时间。频率总是用分钟来表示的。 |
	| ${coordStart} | 作业开始时间。 |
	| ${coordEnd} | 作业结束时间。 |
    | ${coordTimezone} | Oozie 在没有夏时制的固定时区（通常用 UTC 表示）处理协调器作业。此时区被称为“Oozie 处理时区”。 |
	| ${wfPath} | workflow.xml 的路径。如果该工作流文件名不是默认文件名 (workflow.xml)，则必须指定该名称。 |
	
2. 使用 ANSI (ASCII) 编码将文件另存为 **C:\\Tutorials\\UseOozie\\coordinator.xml**。（如果你的文本编辑器不提供此选项，则使用记事本。）
	
##<a id="deploy"></a>部署 Oozie 项目并准备教程

你将运行 Azure PowerShell 脚本来执行以下操作：

- 将 HiveQL 脚本 (useoozie.hql) 复制到 Azure Blob 存储 wasb:///tutorials/useoozie/useoozie.hql。
- 将 workflow.xml 复制到 wasb:///tutorials/useoozie/workflow.xml。
- 将 coordinator.xml 复制到 wasb:///tutorials/useoozie/coordinator.xml。
- 将数据文件 (/example/data/sample.log) 复制到 wasb:///tutorials/useoozie/data/sample.log。 
- 创建用于存储 Sqoop 导出数据的 Azure SQL 数据库表。表的名称为 *log4jLogCount*。

**了解 HDInsight 存储**

HDInsight 将 Azure Blob 存储用于数据存储。wasb:// 是 Microsoft 在 Azure Blob 存储中对 Hadoop 分布式文件系统 (HDFS) 的实施。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

设置 HDInsight 群集时，请将 Azure Blob 存储帐户和该帐户上的特定容器指定为默认文件系统，就像在 HDFS 中一样。除了此存储帐户外，在设置过程中，你还可以从同一 Azure 订阅或不同 Azure 订阅添加其他存储帐户。有关添加其他存储帐户的说明，请参阅[设置 HDInsight 群集][hdinsight-provision]。为了简化本教程中使用的 Azure PowerShell 脚本，所有文件都存储在默认文件系统容器（位于 */tutorials/useoozie*）中。默认情况下，此容器与 HDInsight 群集同名。语法为：

	wasb[s]://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/<path>/<filename>

> [AZURE.NOTE]HDInsight 群集 3.0 版只支持 *wasb://* 语法。较早的 *asv://* 语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.0 群集中不受支持。

> [AZURE.NOTE]wasb:// 路径是虚拟路径。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

存储在默认文件系统容器中的文件可以使用以下任一 URI 从 HDInsight 进行访问（以 workflow.xml 为例）：

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

1. 打开 Windows PowerShell ISE（在 Windows 8“开始”屏幕上，键入 **PowerShell_ISE**，然后单击“Windows PowerShell ISE”。有关详细信息，请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][powershell-start]。
2. 在底部窗格中，运行以下命令以连接到 Azure 订阅：

		Add-AzureAccount -Environment AzureChinaCloud

	系统将提示你输入 Azure 帐户凭据。这种添加订阅连接的方法会超时，12 个小时之后，你将需要再次运行该 cmdlet。

	> [AZURE.NOTE]如果你有多个 Azure 订阅，而默认订阅不是你想使用的，则请使用 <strong>Select-AzureSubscription</strong> cmdlet 来选择订阅。

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
		$hiveQLScript = "C:\Tutorials\UseOozie\useooziewf.hql"
		$workflowDefinition = "C:\Tutorials\UseOozie\workflow.xml"
		$coordDefinition =  "C:\Tutorials\UseOozie\coordinator.xml"
		
		# WASB folder for storing the Oozie tutorial files.
		$destFolder = "tutorials/useoozie"  # Do NOT use the long path here


	有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)部分。

3. 在脚本窗格中将以下内容追加到脚本：
		
		# Create a storage context object
		$storageaccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}
		$destContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageAccountName -StorageAccountKey $storageaccountkey
		
		function uploadOozieFiles()
		{		
		    Write-Host "Copy HiveQL script, workflow definition and coordinator definition ..." -ForegroundColor Green
			Set-AzureStorageBlobContent -File $hiveQLScript -Container $containerName -Blob "$destFolder/useooziewf.hql" -Context $destContext
			Set-AzureStorageBlobContent -File $workflowDefinition -Container $containerName -Blob "$destFolder/workflow.xml" -Context $destContext
			Set-AzureStorageBlobContent -File $coordDefinition -Container $containerName -Blob "$destFolder/coordinator.xml" -Context $destContext
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
			$conn.ConnectionString = "Data Source=$sqlDatabaseServer.database.chinacloudapi.cn;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseLogin;Password=$sqlDatabaseLoginPassword;Encrypt=true;Trusted_Connection=false;"
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

4. 单击“运行脚本”或按 **F5** 键以运行该脚本。输出结果将会类似于：

	![教程准备的输出结果][img-preparation-output]

##<a id="run"></a>运行 Oozie 项目

Azure PowerShell 目前不提供任何用于定义 Oozie 作业的 cmdlet。你可以使用 **Invoke-RestMethod** cmdlet 来调用 Oozie Web 服务。Oozie Web 服务 API 是 HTTP REST JSON API。有关 Oozie Web 服务 API 的详细信息，请参阅 [Apache Oozie 4.0 文档][apache-oozie-400]（用于 HDInsight 群集版本 3.0）或 [Apache Oozie 3.3.2 文档][apache-oozie-332]（用于 HDInsight 群集版本 2.1）。

**提交 Oozie 作业**

1. 打开 Windows PowerShell ISE（在 Windows 8“开始”屏幕上，键入 **PowerShell_ISE**，然后单击“Windows PowerShell ISE”。有关详细信息，请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][powershell-start]。

3. 将以下脚本复制到脚本窗格，然后设置前 14 个变量（不过，请跳过 **$storageUri**）。

		#HDInsight cluster variables
		$clusterName = "<HDInsightClusterName>"
		$clusterUsername = "<HDInsightClusterUsername>"
		$clusterPassword = "<HDInsightClusterUserPassword>"
		
		#Azure Blob storage (WASB) variables
		$storageAccountName = "<StorageAccountName>"
		$storageContainerName = "<BlobContainerName>"
		$storageUri="wasb://$storageContainerName@$storageAccountName.blob.core.chinacloudapi.cn"
		
		#Azure SQL database variables
		$sqlDatabaseServer = "<SQLDatabaseServerName>"
		$sqlDatabaseLogin = "<SQLDatabaseLoginName>"
		$sqlDatabaseLoginPassword = "<SQLDatabaseloginPassword>"
		$sqlDatabaseName = "<SQLDatabaseName>"  
		
		#Oozie WF/coordinator variables
		$coordStart = "2014-03-21T13:45Z"
		$coordEnd = "2014-03-21T13:45Z"
		$coordFrequency = "1440"	# in minutes, 24h x 60m = 1440m
		$coordTimezone = "UTC"	#UTC/GMT

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

	$coordstart 和 $coordend 是工作流的开始和结束时间。若要了解 UTC/GMT 时间，请在 bing.com 上搜索“utc 时间”。$coordFrequency 是指你想要运行工作流的频率（以分钟为单位）。

3. 将以下内容追加到脚本。这部分定义 Oozie 负载：
		
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
		       <name>oozie.coord.application.path</name>
		       <value>$oozieWFPath</value>
		   </property>

		   <property>
		       <name>wfPath</name>
		       <value>$oozieWFPath</value>
		   </property>

		   <property>
		       <name>coordStart</name>
		       <value>$coordStart</value>
		   </property>

		   <property>
		       <name>coordEnd</name>
		       <value>$coordEnd</value>
		   </property>

		   <property>
		       <name>coordFrequency</name>
		       <value>$coordFrequency</value>
		   </property>

		   <property>
		       <name>coordTimezone</name>
		       <value>$coordTimezone</value>
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
		       <value>admin</value>
		   </property>
		
		</configuration>
		"@

	>[AZURE.NOTE]与工作流提交负载文件相比，主要区别是变量 **oozie.coord.application.path**。在提交工作流作业时，你使用的是 **oozie.wf.application.path**。

4. 将以下内容追加到脚本。这部分检查 Oozie Web 服务状态：
			
		function checkOozieServerStatus()
		{
		    Write-Host "Checking Oozie server status..." -ForegroundColor Green
		    $clusterUriStatus = "https://$clusterName.azurehdinsight.cn:443/oozie/v2/admin/status"
		    $response = Invoke-RestMethod -Method Get -Uri $clusterUriStatus -Credential $creds -OutVariable $OozieServerStatus 
		    
		    $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
		    $oozieServerSatus = $jsonResponse[0].("systemMode")
		    Write-Host "Oozie server status is $oozieServerSatus..."
		
		    if($oozieServerSatus -notmatch "NORMAL")
		    {
		        Write-Host "Oozie server status is $oozieServerSatus...cannot submit Oozie jobs. Check the server status and re-run the job."
		        exit 1
		    }
		}
	
5. 将以下内容追加到脚本。这部分创建一项 Oozie 作业：

		function createOozieJob()
		{
		    # create Oozie job
		    Write-Host "Sending the following Payload to the cluster:" -ForegroundColor Green
		    Write-Host "`n--------`n$OoziePayload`n--------"
		    $clusterUriCreateJob = "https://$clusterName.azurehdinsight.cn:443/oozie/v2/jobs"
		    $response = Invoke-RestMethod -Method Post -Uri $clusterUriCreateJob -Credential $creds -Body $OoziePayload -ContentType "application/xml" -OutVariable $OozieJobName -debug -Verbose
		
		    $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
		    $oozieJobId = $jsonResponse[0].("id")
		    Write-Host "Oozie job id is $oozieJobId..."
		
		    return $oozieJobId
		}

	> [AZURE.NOTE]在提交工作流作业时，你必须在创建作业后进行另一次 Web 服务调用以启动该作业。在这种情况下，该协调器作业会按时间触发。该作业将自动启动。

6. 将以下内容追加到脚本。这部分检查 Oozie 作业状态：

		function checkOozieJobStatus($oozieJobId)
		{
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
		
		    Write-Host "$(Get-Date -format 'G'): $oozieJobId is in $JobStatus state!"
		    if($JobStatus -notmatch "SUCCEEDED")
		    {
		        Write-Host "Check logs at http://headnode0:9014/cluster for detais."
		        exit -1
		    }
		}

7. （可选）将以下内容追加到脚本。

		function listOozieJobs()
		{
		    Write-Host "Listing Oozie jobs..." -ForegroundColor Green
		    $clusterUriStatus = "https://$clusterName.azurehdinsight.cn:443/oozie/v2/jobs"
		    $response = Invoke-RestMethod -Method Get -Uri $clusterUriStatus -Credential $creds 
		    
		    write-host "Job ID                                   App Name        Status      Started                         Ended"
		    write-host "----------------------------------------------------------------------------------------------------------------------------------"
		    foreach($job in $response.workflows)
		    {
		        Write-Host $job.id "`t" $job.appName "`t" $job.status "`t" $job.startTime "`t" $job.endTime
		    }
		}

		function ShowOozieJobLog($oozieJobId)
		{
		    Write-Host "Showing Oozie job info..." -ForegroundColor Green
		    $clusterUriStatus = "https://$clusterName.azurehdinsight.cn:443/oozie/v2/job/$oozieJobId" + "?show=log"
		    $response = Invoke-RestMethod -Method Get -Uri $clusterUriStatus -Credential $creds 
		    write-host $response
		}

		function killOozieJob($oozieJobId)
		{
		    Write-Host "Killing the Oozie job $oozieJobId..." -ForegroundColor Green
		    $clusterUriStartJob = "https://$clusterName.azurehdinsight.cn:443/oozie/v2/job/" + $oozieJobId + "?action=kill" #Valid values for the 'action' parameter are 'start', 'suspend', 'resume', 'kill', 'dryrun', 'rerun', and 'change'.
		    $response = Invoke-RestMethod -Method Put -Uri $clusterUriStartJob -Credential $creds | Format-Table -HideTableHeaders -debug
		}
  
7. 将以下内容追加到脚本：

		checkOozieServerStatus
		# listOozieJobs
		$oozieJobId = createOozieJob($oozieJobId)
		checkOozieJobStatus($oozieJobId)
		# ShowOozieJobLog($oozieJobId)
		# killOozieJob($oozieJobId)

	如果要运行这些附加的功能，请删除这些 # 号。

7. 如果你的 HDinsight 群集是 2.1 版的，请将“https://$clusterName.azurehdinsight.cn:443/oozie/v2/”替换为“https://$clusterName.azurehdinsight.cn:443/oozie/v1/”。HDInsight 群集版本 2.1 不支持 Web 服务的版本 2。

7. 单击“运行脚本”或按 **F5** 键以运行该脚本。输出结果将会类似于：

	![教程运行工作流输出][img-runworkflow-output]

8. 连接到 SQL 数据库以查看导出的数据。

**检查作业错误日志**

若要解决工作流的疑难问题，可从群集头节点中的 C:\\apps\\dist\\oozie-3.3.2.1.3.2.0-05\\oozie-win-distro\\logs\\Oozie.log 位置找到 Oozie 日志文件。有关 RDP 的信息，请参阅[使用经典管理门户管理 HDInsight 群集][hdinsight-admin-portal]。

**重新运行教程**

若要重新运行该工作流，必须执行以下任务：

- 删除 Hive 脚本输出文件。
- 删除 log4jLogsCount 表中的数据。

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


##<a id="nextsteps"></a>后续步骤
在本教程中，你已经学习了如何定义 Oozie 工作流、Oozie 协调器，以及如何使用 Azure PowerShell 运行 Oozie 协调器作业。若要了解更多信息，请参阅下列文章：

- [开始使用 HDInsight][hdinsight-get-started]
- [将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]
- [使用 Azure PowerShell 管理 HDInsight][hdinsight-admin-powershell]
- [将数据上载到 HDInsight][hdinsight-upload-data]
- [将 Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]
- [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
- [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
- [为 HDInsight 开发 C# Hadoop 流作业][hdinsight-develop-streaming-jobs]
- [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-java-mapreduce]



[hdinsight-cmdlets-download]: http://go.microsoft.com/fwlink/?LinkID=325563


[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning-v1/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/
[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal-v1/


[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-get-started-emulator]: /documentation/articles/hdinsight-hadoop-emulator-get-started/
[hdinsight-develop-streaming-jobs]: /documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/
[hdinsight-develop-java-mapreduce]: /documentation/articles/hdinsight-develop-deploy-java-mapreduce/
[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/

[sqldatabase-create-configue]: /documentation/articles/sql-database-get-started/
[sqldatabase-get-started]: /documentation/articles/sql-database-get-started/

[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account/

[apache-hadoop]: http://hadoop.apache.org/
[apache-oozie-400]: http://oozie.apache.org/docs/4.0.0/
[apache-oozie-332]: http://oozie.apache.org/docs/3.3.2/

[powershell-download]: /downloads/
[powershell-about-profiles]: https://technet.microsoft.com/zh-cn/library/hh847857.aspx
[powershell-install-configure]: /documentation/articles/powershell-install-configure/
[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
[powershell-script]: https://technet.microsoft.com/library/dn425048.aspx

[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

[img-workflow-diagram]: ./media/hdinsight-use-oozie-coordinator-time/HDI.UseOozie.Workflow.Diagram.png
[img-preparation-output]: ./media/hdinsight-use-oozie-coordinator-time/HDI.UseOozie.Preparation.Output1.png
[img-runworkflow-output]: ./media/hdinsight-use-oozie-coordinator-time/HDI.UseOozie.RunCoord.Output.png

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx

<!---HONumber=71-->