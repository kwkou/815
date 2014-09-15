<properties linkid="hdinsight-use-oozie-with-hdinsight" urlDisplayName="Use Oozie with HDInsight" pageTitle="Use Oozie with HDInsight | Azure" metaKeywords="" description="Use Oozie with HDInsight, a big data solution. Learn how to define an Oozie workflow, and submit an Oozie job." metaCanonical="" services="hdinsight" documentationCenter="" title="Use Oozie with HDInsight" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />


# 将 Oozie 与 HDInsight 配合使用

学习如何在 HDInsight 上定义工作流以及如何运行该工作流。若要了解 Oozie 协调器，请参阅[将基于时间的 Oozie 协调器与 HDInsight 配合使用][hdinsight-oozie-coordinator-time]。



**估计完成时间：** 40 分钟

## 本文内容

1. [什么是 Oozie](#whatisoozie)
2. [先决条件](#prerequisites)
3. [定义 Oozie 工作流文件](#defineworkflow)
4. [部署 Oozie 项目并准备教程](#deploy)
5. [运行工作流](#run)
6. [后续步骤](#nextsteps)

## <a id="whatisoozie"></a>什么是 Oozie

Apache Oozie 是一个管理 Hadoop 作业的工作流/协调系统。它与 Hadoop 堆栈集成，支持 Apache MapReduce、Apache Pig、Apache Hive 和 Apache Sqoop 的 Hadoop 作业。它也能用于安排特定于某系统的作业，例如 Java 程序或 shell 脚本。

你要实现的工作流包含两个操作：

![工作流关系图][img-workflow-diagram]

1. Hive 操作运行 HiveQL 脚本以统计 log4j 日志文件中每个日志级类型的次数。每个 log4j 日志都包含一行字段，其中包含 [LOG LEVEL] 字段，可显示类型和严重性。例如：

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

	有关 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用][hdinsight-hive]。

2. Sqoop 操作将 HiveQL 操作输出结果导出到 Azure SQL 数据库中的表。有关 Sqoop 的详细信息，请参阅[将 Sqoop 与 HDInsight 配合使用][hdinsight-sqoop]。

> [WACOM.NOTE] 有关 HDInsight 群集上支持的 Oozie 版本，请参阅 [HDInsight 提供的群集版本有哪些新功能？][hdinsight-versions]。

> [WACOM.NOTE] 本教程适用于 HDInsight 群集版本 2.1 和 3.0。本文尚未在 HDInsight Emulator 上测试过。


##<a id="prerequisites"></a>先决条件

在开始阅读本教程前，你必须具有：

- 已安装并已配置 Azure PowerShell 的**工作站**。有关说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。若要执行 PowerShell 脚本，必须以管理员身份运行 Azure PowerShell 并将执行策略设为*RemoteSigned*。请参阅[运行 Windows PowerShell 脚本][powershell-script]。
- **HDInsight 群集**。有关创建 HDInsight 群集的信息，请参阅[设置 HDInsight 群集][hdinsight-provision]或 [HDInsight 入门][hdinsight-get-started]。你将需要以下数据才能完成本教程：

	<table border = "1">
	<tr><th>群集属性</th><th>PowerShell 变量名</th><th>值</th><th>说明</th></tr>
	<tr><td>HDInsight 群集名称</td><td>$clusterName</td><td></td><td>要在其中运行本教程的 HDInsight 群集。</td></tr>
	<tr><td>HDInsight 群集用户名</td><td>$clusterUsername</td><td></td><td>HDInsight 群集用户的用户名。 </td></tr>
	<tr><td>HDInsight 群集用户的密码</td><td>$clusterPassword</td><td></td><td>HDInsight 群集用户的密码。</td></tr>
	<tr><td>Azure 存储帐户名称</td><td>$storageAccountName</td><td></td><td>可用于 HDInsight 群集的 Azure 存储帐户。在本教程中，使用在群集设置过程中指定的默认存储帐户。</td></tr>
	<tr><td>Azure Blob 容器名称</td><td>$containerName</td><td></td><td>在此示例中，使用用于默认 HDInsight 群集文件系统的 Azure Blob 存储容器。默认情况下，该容器与 HDInsight 群集同名。</td></tr>
	</table>

- **Azure SQL Database**。你必须为 SQL Database 服务器配置防火墙规则以允许从你的工作站进行访问。有关创建 SQL 数据库和配置防火墙的说明，请参阅[使用 Azure SQL 数据库入门][sqldatabase-get-started]。本文提供了用于创建本教程所需的 SQL 数据库表的 PowerShell 脚本。

	<table border = "1">
	<tr><th>SQL 数据库属性</th><th>PowerShell 变量名</th><th>值</th><th>说明</th></tr>
	<tr><td>SQL 数据库服务器名称</td><td>$sqlDatabaseServer</td><td></td><td>Sqoop 要将数据导出到其中的 SQL Database 服务器。 </td></tr>
	<tr><td>SQL 数据库登录名</td><td>$sqlDatabaseLogin</td><td></td><td>SQL Database 登录名。</td></tr>
	<tr><td>SQL 数据库登录密码</td><td>$sqlDatabaseLoginPassword</td><td></td><td>SQL Database 登录密码。</td></tr>
	<tr><td>SQL 数据库名</td><td>$sqlDatabaseName</td><td></td><td>Sqoop 要将数据导出到其中的 Azure SQL Database。 </td></tr>
	</table>

	> [WACOM.NOTE] 默认情况下，可以从 Azure HDInsight 这样的 Azure 服务连接 Azure SQL 数据库。如果禁用了此防火墙设置，则必须从 Azure 管理门户启用它。有关创建 SQL 数据库和配置防火墙规则的说明，请参阅[创建和配置 SQL Database][sqldatabase-create-configue]。


> [WACOM.NOTE] 将值填入表。这将有助于学习本教程。


## <a id="defineworkflow"></a>定义 Oozie 工作流及相关 HiveQL 脚本

Oozie 工作流定义是用 hPDL（一种 XML 过程定义语言）编写的。默认的工作流文件名为 *workflow.xml*。你将在本地保存该工作流文件，并将在本教程后面使用 Azure PowerShell 将它部署到 HDInsight 群集。

该工作流中的 Hive 操作调用 HiveQL 脚本文件。此脚本文件包含三个 HiveQL 语句：

1. **DROP TABLE 语句**删除 log4j Hive 表（如果存在）。
2. **CREATE TABLE 语句**创建指向 log4j 日志文件位置
3. 的 log4j Hive 外部表。字段分隔符为“,”。默认分行符为“\n”。Hive 外部表用于在你想多次运行 Oozie 工作流的情况下避免数据文件从原始位置被删除。
4. **INSERT OVERWRITE 语句**从 log4j Hive 表统计每个日志级类型的次数，并将输出结果保存到 Azure 储存空间 - Blob (WASB) 位置。

有一个已知的 Hive 路径问题。你在提交 Oozie 作业时将会遇到这个问题。可在 [TechNet Wiki][technetwiki-hive-error] 上找到用于解决此问题的说明。

**将 HiveQL 脚本文件定义为由工作流调用：**

1. 创建一个内容如下的文本文件：

		DROP TABLE ${hiveTableName};
		CREATE EXTERNAL TABLE ${hiveTableName}(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION '${hiveDataFolder}';
		INSERT OVERWRITE DIRECTORY '${hiveOutputFolder}' SELECT t4 AS sev, COUNT(*) AS cnt FROM ${hiveTableName} WHERE t4 LIKE '[%' GROUP BY t4;

	该脚本中使用了三个变量：

	- ${hiveTableName}
	- ${hiveDataFolder}
	- ${hiveOutputFolder}
			
	工作流定义文件（本教程中的 workflow.xml）在运行时会将三个值传递到这个 HiveQL 脚本。
		
2. 将该文件另存为 **C:\Tutorials\UseOozie\useooziewf.hql**，采用 **ANSI(ASCII)** 编码（如果你的文本编辑器不提供该选项，请使用记事本）。在本教程的后面，此脚本文件将被部署到 HDInsight 群集。



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

	该工作流中定义了两个操作。start-to 操作是 *RunHiveScript*。如果该操作运行 *ok*，则下一个操作是 *RunSqoopExport*。

	RunHiveScript 有几个变量。在从工作站使用 Azure PowerShell 提交 Oozie 作业时，将会传递值。

	<table border = "1">
	<tr><th>工作流变量</th><th>说明</th></tr>
	<tr><td>${jobTracker}</td><td>指定 hadoop 作业跟踪器的 URL。在 HDInsight 群集版本 2.0 和 3.0 上使用 <strong>jobtrackerhost:9010</strong>。</td></tr>
	<tr><td>${nameNode}</td><td>指定 hadoop namenode 的 URL。使用默认文件系统 WASB 地址。例如，<i>wasb://&lt;containerName&gt;@&lt;storageAccountName&gt;.blob.core.chinacloudapi.cn</i>。</td></tr>
	<tr><td>${queueName}</td><td>指定要将作业提交到的 queuename。使用<strong>默认值</strong>。</td></tr>
	</table>

	<table border = "1">
	<tr><th>Hive 操作变量</th><th>说明</th></tr>
	<tr><td>${hiveDataFolder}</td><td>Hive Create Table 命令的源目录。</td></tr>
	<tr><td>${hiveOutputFolder}</td><td>INSERT OVERWRITE 语句的输出文件夹。</td></tr>
	<tr><td>${hiveTableName}</td><td>引用 log4j 数据文件的 Hive 表的名称。</td></tr>
	</table>

	<table border = "1">
	<tr><th>Sqoop 操作变量</th><th>说明</th></tr>
	<tr><td>${sqlDatabaseConnectionString}</td><td>SQL Database 连接字符串。</td></tr>
	<tr><td>${sqlDatabaseTableName}</td><td>数据将要导出到的 SQL Database 表。</td></tr>
	<tr><td>${hiveOutputFolder}</td><td>Hive INSERT OVERWRITE 语句的输出文件夹。这是用于 Sqoop Export export-dir 的同一个文件夹。</td></tr>
	</table>

	有关 Oozie 工作流以及使用工作流操作的详细信息，请参阅 [Apache Oozie 4.0 文档][apache-oozie-400]（用于 HDInsight 群集版本 3.0）或 [Apache Oozie 3.3.2 文档][apache-oozie-332]（用于 HDInsight 群集版本 2.1）。

2. 将该文件另存为 **C:\Tutorials\UseOozie\workflow.xml**，采用 ANSI(ASCII) 编码（如果你的文本编辑器不提供该选项，请使用记事本）。
	
## <a id="deploy"></a>部署 Oozie 项目并准备教程

你将运行 Azure PowerShell 脚本来执行以下操作：

- 将 HiveQL 脚本 (useoozie.hql) 复制到 Azure Blob 存储 wasb:///tutorials/useoozie/useoozie.hql。
- 将 workflow.xml 复制到 wasb:///tutorials/useoozie/workflow.xml。
- 将数据文件 (/example/data/sample.log) 复制到 wasb:///tutorials/useoozie/data/sample.log。
- 创建用于存储 Sqoop 导出数据的 SQL Database 表。表的名称为 *log4jLogCount*。

**了解 HDInsight 存储**

HDInsight 将 Azure Blob 存储用于数据存储。它称为 *WASB* 或 *Azure 存储空间 - Blob*。WASB 是 Microsoft 在 Azure Blob 存储上的 HDFS 实现。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

设置 HDInsight 群集时，请将 Azure 存储帐户和该帐户上的特定 Blob 存储容器指定为默认文件系统，就像在 HDFS 中一样。除了此存储帐户外，在设置过程中，你还可以从同一 Azure 订阅或不同 Azure 订阅添加其他存储帐户。有关添加其他存储帐户的说明，请参阅[设置 HDInsight 群集][hdinsight-provision]。为了简化本教程中使用的 PowerShell 脚本，所有文件都存储在默认文件系统容器（位于 */tutorials/useoozie*）中。默认情况下，此容器与 HDInsight 群集同名。
WASB 语法是：

	wasb[s]://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/<路径>/<文件名>

> [WACOM.NOTE] HDInsight 群集 3.0 版只支持 *wasb://* 语法。较早的 *asv://* 语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.0 群集中不受支持，以后的版本将不会支持该语法。

> [WACOM.NOTE] WASB 路径是虚拟路径。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

存储在默认文件系统容器中的文件可以使用以下任一 URI 从 HDInsight 进行访问（以 workflow.xml 为例）：

	wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn/tutorials/useoozie/workflow.xml
	wasb:///tutorials/useoozie/workflow.xml
	/tutorials/useoozie/workflow.xml

如果要从存储帐户直接访问该文件，则请注意，该文件的 Blob 名称是：

	tutorials/useoozie/workflow.xml

**了解 Hive 内部表和外部表**

以下是你需要了解的有关 Hive 内部表和外部表的一些信息：

- CREATE TABLE 命令创建内部表，也称为托管表。数据文件必须位于默认容器中。
- CREATE TABLE 命令将数据文件移动到默认容器上的 /hive/warehouse/ 文件夹。
- CREATE EXTERNAL TABLE 命令创建外部表。数据文件可以位于默认容器以外的位置。
- CREATE EXTERNAL TABLE 命令不移动数据文件。
- CREATE EXTERNAL TABLE 命令不允许 LOCATION 子句中指定的文件夹下有任何子文件夹。这是本教程生成 sample.log 文件的副本的原因。

有关详细信息，请参阅 [HDInsight：Hive 内部表和外部表简介][cindygross-hive-tables]。

**准备教程**

1. 打开 Windows PowerShell ISE（在 Windows 8“开始”屏幕上，键入 **PowerShell_ISE**，然后单击 **Windows PowerShell ISE**。请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][powershell-start]）。
2. 在底部窗格中，运行以下命令以连接到 Azure 订阅：

		Add-AzureAccount

	系统将提示你输入 Azure 帐户凭据。这种添加订阅连接的方法会超时，12 个小时之后，你将需要再次运行该 cmdlet。

	> [WACOM.NOTE] 如果你有多个 Azure 订阅，而默认订阅不是你想使用的，则请使用 <strong>Select-AzureSubscription</strong> cmdlet 来选择正确的订阅。

3. 将以下脚本复制到脚本窗格，然后设置前六个变量

		# WASB 变量
		$storageAccountName = "<StorageAccountName>"
		$containerName = "<BlobStorageContainerName>"
		
		# SQL 数据库变量
		$sqlDatabaseServer = "<SQLDatabaseServerName>"  
		$sqlDatabaseLogin = "<SQLDatabaseLoginName>"
		$sqlDatabaseLoginPassword = "SQLDatabaseLoginPassword>"
		$sqlDatabaseName = "<SQLDatabaseName>"  
		$sqlDatabaseTableName = "log4jLogsCount"
		
		# 用于教程的 Oozie 文件  
		$workflowDefinition = "C:\Tutorials\UseOozie\workflow.xml"
		$hiveQLScript = "C:\Tutorials\UseOozie\useooziewf.hql"
		
		# 用于存储 Oozie 教程文件的 WASB 文件夹。
		$destFolder = "tutorials/useoozie"  # 此处请勿使用长路径


	有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)一节。

4. 在脚本窗格中将以下内容追加到脚本：
		
		# 创建存储上下文对象
		$storageaccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}
		$destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageaccountkey
		
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
			# 用于创建 log4jLogsCount 表的 SQL 查询字符串
			$cmdCreateLog4jCountTable = " CREATE TABLE [dbo].[$sqlDatabaseTableName](
				    [Level] [nvarchar](10) NOT NULL,
				    [Total] float,
				CONSTRAINT [PK_$sqlDatabaseTableName] PRIMARY KEY CLUSTERED   
				(
				[Level] ASC
				)
				)"
				
			#创建 log4jLogsCount 表
		    Write-Host "Create Log4jLogsCount table ..." -ForegroundColor Green
			$conn = New-Object System.Data.SqlClient.SqlConnection
			$conn.ConnectionString = "Data Source=$sqlDatabaseServer.database.windows.net;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseLogin;Password=$sqlDatabaseLoginPassword;Encrypt=true;Trusted_Connection=false;"
			$conn.open()
			$cmd = New-Object System.Data.SqlClient.SqlCommand
			$cmd.connection = $conn
			$cmd.commandtext = $cmdCreateLog4jCountTable
			$cmd.executenonquery()
				
			$conn.close()
		}
				
		# 上载 workflow.xml、coordinator.xml 和 ooziewf.hql
		uploadOozieFiles;
				
		# 将 example/data/sample.log 复制一份到 example/data/log4j/sample.log
		prepareHiveDataFile;

		# 在 SQL 数据库上创建 log4jlogsCount 表
		prepareSQLDatabase;

5. 单击**运行脚本** 或按 **F5** 键以运行该脚本。输出应如下所示：

	![教程准备的输出结果][img-preparation-output]

## <a id="run"></a>运行 Oozie 项目

Azure PowerShell 目前不提供任何用于定义 Oozie 作业的 cmdlet。你可以使用
Invoke-RestMethod PowerShell cmdlet 来调用 Oozie Web 服务。Oozie Web 服务 API 是 HTTP REST JSON API。有关 Oozie Web 服务 API 的详细信息，请参阅 [Apache Oozie 4.0 文档][apache-oozie-400]（用于 HDInsight 群集版本 3.0）或 [Apache Oozie 3.3.2 文档][apache-oozie-332]（用于 HDInsight 群集版本 2.1）。

**提交 Oozie 作业**

1. 打开 Windows PowerShell ISE（在 Windows 8“开始”屏幕上，键入 **PowerShell_ISE**，然后单击 **Windows PowerShell ISE**。请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][powershell-start]）。

2. 将以下脚本复制到脚本窗格，然后设置前 10 个变量（跳过第 6 个：$storageUri）。

		#HDInsight 群集变量
		$clusterName = "<HDInsightClusterName>"
		$clusterUsername = "<HDInsightClusterUsername>"
		$clusterPassword = "<HDInsightClusterUserPassword>"
		
		#Azure Blob 存储 (WASB) 变量
		$storageAccountName = "<StorageAccountName>"
		$storageContainerName = "<BlobContainerName>"
		$storageUri="wasb://$storageContainerName@$storageAccountName.blob.core.chinacloudapi.cn"
		
		#Azure SQL 数据库变量
		$sqlDatabaseServer = "<SQLDatabaseServerName>"
		$sqlDatabaseLogin = "<SQLDatabaseLoginName>"
		$sqlDatabaseLoginPassword = "<SQLDatabaseloginPassword>"
		$sqlDatabaseName = "<SQLDatabaseName>"  
		
		#Oozie WF 变量
		$oozieWFPath="$storageUri/tutorials/useoozie"  # 默认名称为 workflow.xml。不需指定文件名。
		$waitTimeBetweenOozieJobStatusCheck=10
		
		#Hive 操作变量
		$hiveScript = "$storageUri/tutorials/useoozie/useooziewf.hql"
		$hiveTableName = "log4jlogs"
		$hiveDataFolder = "$storageUri/tutorials/useoozie/data"
		$hiveOutputFolder = "$storageUri/tutorials/useoozie/output"
		
		#Sqoop 操作变量
		$sqlDatabaseConnectionString = "jdbc:sqlserver://$sqlDatabaseServer.database.chinacloudapi.cn;user=$sqlDatabaseLogin@$sqlDatabaseServer;password=$sqlDatabaseLoginPassword;database=$sqlDatabaseName"
		$sqlDatabaseTableName = "log4jLogsCount"

		$passwd = ConvertTo-SecureString $clusterPassword -AsPlainText -Force
		$creds = New-Object System.Management.Automation.PSCredential ($clusterUsername, $passwd)


	有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)一节。

3. 将以下内容追加到脚本。这部分定义 Oozie 负载：
		
		#OoziePayload 用于 Oozie Web 服务提交
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
		       <value>&quot;$sqlDatabaseConnectionString&quot;</value>
		   </property>
		
		   <property>
		       <name>sqlDatabaseTableName</name>
		       <value>$SQLDatabaseTableName</value>
		   </property>
		
		   <property>
		       <name>user.name</name>
		       <value>admin</value>
		   </property>
		
		   <property>
		       <name>oozie.wf.application.path</name>
		       <value>$oozieWFPath</value>
		   </property>
		
		</configuration>
		"@
		
4. 将以下内容追加到脚本。这部分检查 Oozie Web 服务状态：
			
	    Write-Host "Checking Oozie server status..." -ForegroundColor Green
	    $clusterUriStatus = "https://$clusterName.hdinsightservices.cn:443/oozie/v2/admin/status"
	    $response = Invoke-RestMethod -Method Get -Uri $clusterUriStatus -Credential $creds -OutVariable $OozieServerStatus 
	    
	    $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
	    $oozieServerSatus = $jsonResponse[0].("systemMode")
	    Write-Host "Oozie server status is $oozieServerSatus..."
	
5. 将以下内容追加到脚本。这部分创建并启动一项 Oozie 作业：

	    # 创建 Oozie 作业
	    Write-Host "Sending the following Payload to the cluster:" -ForegroundColor Green
	    Write-Host "`n--------`n$OoziePayload`n--------"
	    $clusterUriCreateJob = "https://$clusterName.hdinsightservices.cn:443/oozie/v2/jobs"
	    $response = Invoke-RestMethod -Method Post -Uri $clusterUriCreateJob -Credential $creds -Body $OoziePayload -ContentType "application/xml" -OutVariable $OozieJobName #-debug
	
	    $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
	    $oozieJobId = $jsonResponse[0].("id")
	    Write-Host "Oozie job id is $oozieJobId..."
	
	    # 启动 Oozie 作业
	    Write-Host "Starting the Oozie job $oozieJobId..."-ForegroundColor Green
	    $clusterUriStartJob = "https://$clusterName.hdinsightservices.cn:443/oozie/v2/job/" + $oozieJobId + "?action=start"
	    $response = Invoke-RestMethod -Method Put -Uri $clusterUriStartJob -Credential $creds | Format-Table -HideTableHeaders #-debug

6. 将以下内容追加到脚本。这部分检查 Oozie 作业状态：

	    # 获取作业状态
	    Write-Host "Sleeping for $waitTimeBetweenOozieJobStatusCheck seconds until the job metadata is populated in the Oozie metastore..." -ForegroundColor Green
	    Start-Sleep -Seconds $waitTimeBetweenOozieJobStatusCheck
	
	    Write-Host "Getting job status and waiting for the job to complete..." -ForegroundColor Green
	    $clusterUriGetJobStatus = "https://$clusterName.hdinsightservices.cn:443/oozie/v2/job/" + $oozieJobId + "?show=info"
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

7. 如果你的 HDinsight 群集是 2.1 版的，请将“https://$clusterName.hdinsightservices.cn:443/oozie/v2/”替换为“https://$clusterName.hdinsightservices.cn:443/oozie/v1/”。HDInsight 群集版本 2.1 不支持 Web 服务的版本 2。

8. 单击**运行脚本** 或按 **F5** 键以运行该脚本。输出结果将会类似于：

	![教程运行工作流输出][img-runworkflow-output]

9. 连接到 SQL Database 以查看导出的数据。

**检查作业错误日志**

若要对工作流进行故障排除，可从群集头节点找到 Oozie 日志文件，位置是
*C:\apps\dist\oozie-3.3.2.1.3.2.0-05\oozie-win-distro\logs\Oozie.log* 或 *C:\apps\dist\oozie-4.0.0.2.0.7.0-1528\oozie-win-distro\logs\Oozie.log*。有关 RDP 的信息，请参阅[使用管理门户管理 HDInsight 群集][hdinsight-admin-portal]。

**重新运行教程**

若要重新运行该工作流，必须执行以下操作：

- 删除 Hive 脚本输出文件
- 删除 log4jLogsCount 表中的数据

这是你可以使用的一个示例 PowerShell 脚本：

	$storageAccountName = "<AzureStorageAccountName>"
	$containerName = "<ContainerName>"
	
	# SQL 数据库变量
	$sqlDatabaseServer = "<SQLDatabaseServerName>"
	$sqlDatabaseLogin = "<SQLDatabaseLoginName>"
	$sqlDatabaseLoginPassword = "<SQLDatabaseLoginPassword>"
	$sqlDatabaseName = "<SQLDatabaseName>"
	$sqlDatabaseTableName = "log4jLogsCount"
	
	Write-host "Delete the Hive script output file ..." -ForegroundColor Green
	$storageaccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}
	$destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageaccountkey
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


## <a id="nextsteps"></a>后续步骤
在本教程中，你已经学习了如何定义 Oozie 工作流，以及如何使用 Azure PowerShell 运行 Oozie 作业。若要了解更多信息，请参阅下列文章：

- [将基于时间的 Oozie 协调器与 HDInsight 配合使用][hdinsight-oozie-coordinator-time]
- [HDInsight 入门][hdinsight-get-started]
- [HDInsight Emulator 入门][hdinsight-emulator]
- [将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]
- [使用 PowerShell 管理 HDInsight][hdinsight-admin-powershell]
- [将数据上传到 HDInsight][hdinsight-upload-data]
- [将 Sqoop 与 HDInsight 配合使用][hdinsight-sqoop]
- [Hive 与 HDInsight 配合使用][hdinsight-hive]
- [Pig 与 HDInsight 配合使用][hdinsight-pig]
- [为 HDInsight 开发 C\# Hadoop 流作业][hdinsight-develop-streaming]
- [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-mapreduce]





[hdinsight-oozie-coordinator-time]: ../hdinsight-use-oozie-coordinator-time/
[hdinsight-versions]:  /en-us/documentation/articles/hdinsight-component-versioning/
[hdinsight-storage]: /en-us/documentation/articles/hdinsight-use-blob-storage/
[hdinsight-get-started]: /en-us/documentation/articles/hdinsight-get-started/
[hdinsight-admin-portal]: /en-us/documentation/articles/hdinsight-administer-use-management-portal/


[hdinsight-sqoop]: ../hdinsight-use-sqoop/
[hdinsight-provision]: /en-us/documentation/articles/hdinsight-provision-clusters/

[hdinsight-admin-powershell]: /en-us/documentation/articles/hdinsight-administer-use-powershell/

[hdinsight-upload-data]: /en-us/documentation/articles/hdinsight-upload-data/

[hdinsight-mapreduce]: /en-us/documentation/articles/hdinsight-use-mapreduce/
[hdinsight-hive]: /en-us/documentation/articles/hdinsight-use-hive/

[hdinsight-pig]: /en-us/documentation/articles/hdinsight-use-pig/

[hdinsight-cmdlets-download]: http://go.microsoft.com/fwlink/?LinkID=325563
[hdinsight-storage]: /en-us/documentation/articles/hdinsight-use-blob-storage/

[hdinsight-emulator]: /en-us/documentation/articles/hdinsight-get-started-emulator/

[hdinsight-develop-streaming]: /en-us/documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/
[hdinsight-develop-mapreduce]: /en-us/documentation/articles/hdinsight-develop-deploy-java-mapreduce/

[sqldatabase-create-configue]: ../sql-database-create-configure/
[sqldatabase-get-started]: ../sql-database-get-started/

[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: /en-us/manage/services/storage/how-to-create-a-storage-account/ 

[apache-hadoop]: http://hadoop.apache.org/
[apache-oozie-400]: http://oozie.apache.org/docs/4.0.0/
[apache-oozie-332]: http://oozie.apache.org/docs/3.3.2/

[powershell-download]: http://www.windowsazure.cn/zh-cn/downloads/#cmd-line-tools
[powershell-about-profiles]: http://go.microsoft.com/fwlink/?LinkID=113729
[powershell-install-configure]: /en-us/manage/install-and-configure-windows-powershell/
[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
[powershell-script]: http://technet.microsoft.com/zh-cn/library/ee176949.aspx

[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

[img-workflow-diagram]: ./media/hdinsight-use-oozie/HDI.UseOozie.Workflow.Diagram.png
[img-preparation-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.Preparation.Output1.png  
[img-runworkflow-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.RunWF.Output.png 

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx
