<properties
    pageTitle="在 HDInsight 中使用基于时间的 Hadoop Oozie 协调器 | Azure"
    description="在 HDInsight 中使用基于时间的 Hadoop Oozie 协调器（大数据服务）。了解如何定义 Oozie 工作流和协调器，以及如何提交作业。"
    services="hdinsight"
    documentationcenter=""
    tags="azure-portal"
    author="mumian"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="00c3a395-d51a-44ff-af2d-1f116c4b1c83"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/15/2016"
    wacn.date="01/25/2017"
    ms.author="jgao" />  


# 将基于时间的 Oozie 协调器与 HDInsight 中的 Hadoop 配合使用以定义工作流和协调作业

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

在本文中，将学习如何定义工作流和协调器，以及如何基于时间触发协调器作业。阅读本文前，浏览[将 Oozie 与 HDInsight 配合使用][hdinsight-use-oozie]很有帮助。除了 Oozie，还可以使用 Azure 数据工厂来计划作业。

> [AZURE.NOTE]
本文需要基于 Windows 的 HDInsight 群集。有关在基于 Linux 的群集上使用 Oozie 的信息（包括基于时间的作业），请参阅[在基于 Linux 的 HDInsight 上将 Oozie 与 Hadoop 配合使用以定义和运行工作流](/documentation/articles/hdinsight-use-oozie-linux-mac/)

## <a id="whatisoozie"></a> 什么是 Oozie
Apache Oozie 是一个管理 Hadoop 作业的工作流/协调系统。该系统与 Hadoop 堆栈集成，支持 Apache MapReduce、Apache Pig、Apache Hive 和 Apache Sqoop 的 Hadoop 作业。此外，还可用于调度系统特定作业，如 Java 程序或 shell 脚本。

下图显示将要实现的工作流：

![工作流关系图][img-workflow-diagram]  


工作流包含两个操作：

1. Hive 操作运行 HiveQL 脚本，以统计 log4j 日志文件中每个日志级类型的次数。每个 log4j 日志文件都包含一行字段，其中包含显示类型和严重性的 [LOG LEVEL] 字段，例如：

        2012-02-03 18:35:34 SampleClass6 [INFO] everything normal for id 577725851
        2012-02-03 18:35:34 SampleClass4 [FATAL] system problem at id 1991281254
        2012-02-03 18:35:34 SampleClass3 [DEBUG] detail for id 1304807656
        ...

    Hive 脚本的输出结果类似如下：

        [DEBUG] 434
        [ERROR] 3
        [FATAL] 1
        [INFO]  96
        [TRACE] 816
        [WARN]  4

    有关 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]。
2. Sqoop 操作将 HiveQL 操作输出结果导出到 Azure SQL 数据库中的表。有关 Sqoop 的详细信息，请参阅[将 Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]。

> [AZURE.NOTE]
有关 HDInsight 群集上支持的 Oozie 版本，请参阅 [HDInsight 提供的群集版本有哪些新功能？][hdinsight-versions]。
>
>

## <a id="prerequisites"></a>先决条件
开始阅读本教程前的必要准备：

* **配备 Azure PowerShell 的工作站**。

    > [AZURE.IMPORTANT]
    Azure PowerShell 对于使用 Azure Service Manager 管理 HDInsight 资源的支持已**弃用**，将于 2017 年 1 月 1 日删除。本文档中的步骤使用的是与 Azure Resource Manager 兼容的新 HDInsight cmdlet。
    ><p>
    > 请按照 [Install and configure Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)（安装和配置 Azure PowerShell）中的步骤安装最新版本的 Azure PowerShell。如果你的脚本需要修改才能使用与 Azure Resource Manager 兼容的新 cmdlet，请参阅[迁移到适用于 HDInsight 群集的基于 Azure Resource Manager 的开发工具](/documentation/articles/hdinsight-hadoop-development-using-azure-resource-manager/)，了解详细信息。

* **HDInsight 群集**。有关创建 HDInsight 群集的信息，请参阅[创建 HDInsight 群集][hdinsight-provision]或 [HDInsight 入门][hdinsight-get-started]。完成本教程需要以下数据：

    <table border = "1">
    <tr><th>群集属性</th><th>Windows PowerShell 变量名</th><th>值</th><th>说明</th></tr>
    <tr><td>HDInsight 群集名称</td><td>$clusterName</td><td></td><td>运行本教程的 HDInsight 群集。</td></tr>
    <tr><td>HDInsight 群集用户名</td><td>$clusterUsername</td><td></td><td>HDInsight 群集用户名。</td></tr>
    <tr><td>HDInsight 群集用户密码 </td><td>$clusterPassword</td><td></td><td>HDInsight 群集用户密码。</td></tr>
    <tr><td>Azure 存储帐户名称</td><td>$storageAccountName</td><td></td><td>可用于 HDInsight 群集的 Azure 存储帐户。在本教程中，使用在群集设置过程中指定的默认存储帐户。</td></tr>
    <tr><td>Azure Blob 容器名称</td><td>$containerName</td><td></td><td>在此示例中，使用用于默认 HDInsight 群集文件系统的 Azure Blob 存储容器。默认情况下，该容器与 HDInsight 群集同名。</td></tr>
    </table>
* **Azure SQL 数据库**。必须为 SQL 数据库服务器配置防火墙规则，以允许从工作站访问。有关创建 Azure SQL 数据库和配置防火墙的说明，请参阅 [Azure SQL 数据库入门][sqldatabase-get-started]。本文提供用于创建本教程所需 Azure SQL 数据库表的 Windows PowerShell 脚本。

    <table border = "1">
    <tr><th>SQL 数据库属性</th><th>Windows PowerShell 变量名</th><th>值</th><th>说明</th></tr>
    <tr><td>SQL 数据库服务器名称</td><td>$sqlDatabaseServer</td><td></td><td>Sqoop 将数据导出到的 SQL 数据库服务器。</td></tr>
    <tr><td>SQL 数据库登录名</td><td>$sqlDatabaseLogin</td><td></td><td>SQL 数据库登录名。</td></tr>
    <tr><td>SQL 数据库登录密码</td><td>$sqlDatabaseLoginPassword</td><td></td><td>SQL 数据库登录密码。</td></tr>
    <tr><td>SQL 数据库名</td><td>$sqlDatabaseName</td><td></td><td>Sqoop 将数据导出到的 Azure SQL 数据库。</td></tr>
    </table>

    > [AZURE.NOTE]
    默认情况下，可从 Azure 服务（如 Azure HDInsight）连接 Azure SQL 数据库。如果禁用此防火墙设置，则必须从 Azure 门户预览启用。有关创建 SQL 数据库和配置防火墙规则的说明，请参阅[创建和配置 SQL 数据库][sqldatabase-get-started]。

> [AZURE.NOTE]
填写表中的值。这将有助于学习本教程。

## <a id="defineworkflow"></a> 定义 Oozie 工作流和相关 HiveQL 脚本
Oozie 工作流定义以 hPDL（XML 过程定义语言）编写。默认的工作流文件名为 *workflow.xml*。可在本地保存工作流文件，并在本教程后面使用 Azure PowerShell 将其部署到 HDInsight 群集。

工作流中的 Hive 操作调用 HiveQL 脚本文件。此脚本文件包含三个 HiveQL 语句：

1. **DROP TABLE 语句**删除 log4j Hive 表（如果存在）。
2. **CREATE TABLE 语句**创建 log4j Hive 外部表，指向 log4j 日志文件位置；
3. **log4j 日志文件位置**。字段分隔符为“,”。默认换行符为“\\n”。如果要多次运行 Oozie 工作流，可使用 Hive 外部表避免从原始位置删除数据文件。
4. **INSERT OVERWRITE 语句**从 log4j Hive 表统计每个日志级类型的次数，并将输出结果保存到 Azure Blob 存储位置。

> [AZURE.NOTE]
有一个已知的 Hive 路径问题。将会在提交 Oozie 作业时遇到此问题。可在 TechNet Wiki 上找到解决此问题的说明：[HDInsight Hive 错误:无法重命名][technetwiki-hive-error]。

**定义由工作流调用的 HiveQL 脚本文件**

1. 创建包含以下内容的文本文件：

        DROP TABLE ${hiveTableName};
        CREATE EXTERNAL TABLE ${hiveTableName}(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION '${hiveDataFolder}';
        INSERT OVERWRITE DIRECTORY '${hiveOutputFolder}' SELECT t4 AS sev, COUNT(*) AS cnt FROM ${hiveTableName} WHERE t4 LIKE '[%' GROUP BY t4;

    在脚本中使用三个变量：

    * ${hiveTableName}
    * ${hiveDataFolder}
    * ${hiveOutputFolder}

    工作流定义文件（本教程中的 workflow.xml）在运行时会将三个值传递到此 HiveQL 脚本。
2. 使用 ANSI (ASCII) 编码将文件另存为 **C:\\Tutorials\\UseOozie\\useooziewf.hql**。（如果文本编辑器不提供此选项，可使用记事本。） 在本教程后面，会将此脚本文件部署到 HDInsight 群集。

**定义工作流**

1. 创建包含以下内容的文本文件：

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

    在工作流中定义两个操作。start-to 操作是 *RunHiveScript*。如果该操作运行*正常*，则执行下一个操作 *RunSqoopExport*。

    RunHiveScript 包含几个变量。在从工作站使用 Azure PowerShell 提交 Oozie 作业时，将会传递值。

    工作流变量

    <table border = "1">
    <tr><th>工作流变量</th><th>说明</th></tr>
    <tr><td>${jobTracker}</td><td>指定 Hadoop 作业跟踪器的 URL。在 HDInsight 群集 2.0 版和 3.0 版上使用 <strong>jobtrackerhost:9010</strong>。</td></tr>
    <tr><td>${nameNode}</td><td>指定 Hadoop 名称节点的 URL。使用默认文件系统 wasbs:// 地址，例如 <i>wasbs://&lt;containerName>@&lt;storageAccountName>.blob.core.chinacloudapi.cn</i>。</td></tr>
    <tr><td>${queueName}</td><td>指定作业要提交到的队列名称。使用“默认值”。<strong></strong></td></tr>
    </table>

    Hive 操作变量

    <table border = "1">
    <tr><th>Hive 操作变量</th><th>说明</th></tr>
    <tr><td>${hiveDataFolder}</td><td>Hive Create Table 命令的源目录。</td></tr>
    <tr><td>${hiveOutputFolder}</td><td>INSERT OVERWRITE 语句的输出文件夹。</td></tr>
    <tr><td>${hiveTableName}</td><td>引用 log4j 数据文件的 Hive 表的名称。</td></tr>
    </table>

    Sqoop 操作变量

    <table border = "1">
    <tr><th>Sqoop 操作变量</th><th>说明</th></tr>
    <tr><td>${sqlDatabaseConnectionString}</td><td>SQL 数据库连接字符串。</td></tr>
    <tr><td>${sqlDatabaseTableName}</td><td>将数据导出到的 Azure SQL 数据库表。</td></tr>
    <tr><td>${hiveOutputFolder}</td><td>Hive INSERT OVERWRITE 语句的输出文件夹。该文件夹是用于 Sqoop 导出 (export-dir) 的同一个文件夹。</td></tr>
    </table>

    有关 Oozie 工作流和使用工作流操作的详细信息，请参阅 [Apache Oozie 4.0 文档][apache-oozie-400]（对于 HDInsight 群集 3.0 版）或 [Apache Oozie 3.3.2 文档][apache-oozie-332]（对于 HDInsight 群集 2.1 版）。

1. 使用 ANSI (ASCII) 编码将文件另存为 **C:\\Tutorials\\UseOozie\\workflow.xml**。（如果文本编辑器不提供此选项，可使用记事本。）

**定义协调器**

1. 创建包含以下内容的文本文件：

        <coordinator-app name="my_coord_app" frequency="${coordFrequency}" start="${coordStart}" end="${coordEnd}" timezone="${coordTimezone}" xmlns="uri:oozie:coordinator:0.4">
            <action>
                <workflow>
                    <app-path>${wfPath}</app-path>
                </workflow>
            </action>
        </coordinator-app>

    定义文件中使用五个变量：

    | 变量 | 说明 |
    | --- | --- |
    | ${coordFrequency} |作业暂停时间。频率始终用分钟表示。 |
    | ${coordStart} |作业开始时间。 |
    | ${coordEnd} |作业结束时间。 |
    | ${coordTimezone} |Oozie 在没有夏时制的固定时区（通常用 UTC 表示）处理协调器作业。此时区被称为“Oozie 处理时区”。 |
    | ${wfPath} |workflow.xml 的路径。如果工作流文件名不是默认文件名 (workflow.xml)，则必须指定该名称。 |
2. 使用 ANSI (ASCII) 编码将文件另存为 **C:\\Tutorials\\UseOozie\\coordinator.xml**。（如果文本编辑器不提供此选项，可使用记事本。）

## <a id="deploy"></a> 部署 Oozie 项目并准备教程
运行 Azure PowerShell 脚本执行以下操作：

* 将 HiveQL 脚本 (useoozie.hql) 复制到 Azure Blob 存储 wasbs:///tutorials/useoozie/useoozie.hql。
* 将 workflow.xml 复制到 wasbs:///tutorials/useoozie/workflow.xml。
* 将 coordinator.xml 复制到 wasbs:///tutorials/useoozie/coordinator.xml。
* 将数据文件 (/example/data/sample.log) 复制到 wasbs:///tutorials/useoozie/data/sample.log。
* 创建用于存储 Sqoop 导出数据的 Azure SQL 数据库表。表名为 *log4jLogCount*。

**了解 HDInsight 存储**

HDInsight 使用 Azure Blob 存储进行数据存储。wasbs:// 是 Microsoft 在 Azure Blob 存储中实现 Hadoop 分布式文件系统 (HDFS)。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

设置 HDInsight 群集时，将 Azure Blob 存储帐户和该帐户的特定容器指定为默认文件系统，如在 HDFS 中。除此存储帐户以外，在预配过程中还可以从相同或不同 Azure 订阅添加其他存储帐户。有关添加其他存储帐户的说明，请参阅[设置 HDInsight 群集][hdinsight-provision]。为简化本教程中使用的 Azure PowerShell 脚本，将所有文件都存储在默认文件系统容器（位于 */tutorials/useoozie*）中。默认情况下，此容器与 HDInsight 群集同名。语法为：

    wasb[s]://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/<path>/<filename>

> [AZURE.NOTE]
HDInsight 群集版本 3.0 仅支持 *wasb://* 语法。较早的 *asv://* 语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.0 群集中不受支持。
><p>
> wasb:// 路径是虚拟路径。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。

存储在默认文件系统容器中的文件，可使用以下任一 URI 从 HDInsight 进行访问（以 workflow.xml 为例）：

    wasbs://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn/tutorials/useoozie/workflow.xml
    wasbs:///tutorials/useoozie/workflow.xml
    /tutorials/useoozie/workflow.xml

如果要从存储帐户直接访问文件，则文件的 Blob 名称为：

    tutorials/useoozie/workflow.xml

**了解 Hive 内部表和外部表**

以下是需要了解的有关 Hive 内部表和外部表的一些信息：

* CREATE TABLE 命令创建内部表，也称为托管表。数据文件必须位于默认容器中。
* CREATE TABLE 命令将数据文件移动到默认容器中的 /hive/warehouse/<TableName> 文件夹。
* CREATE EXTERNAL TABLE 命令创建外部表。数据文件可以位于默认容器以外的位置。
* CREATE EXTERNAL TABLE 命令不移动数据文件。
* CREATE EXTERNAL TABLE 命令不允许 LOCATION 子句中指定的文件夹下有任何子文件夹。这是本教程生成 sample.log 文件副本的原因。

有关详细信息，请参阅 [HDInsight：Hive 内部和外部表简介][cindygross-hive-tables]。

**准备教程**

1. 打开 Windows PowerShell ISE（在 Windows 8“开始”屏幕上，键入 **PowerShell\_ISE**，然后单击“Windows PowerShell ISE”。有关详细信息，请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][powershell-start]。
2. 在底部窗格中，运行以下命令连接到 Azure 订阅：

        Add-AzureAccount -Environment AzureChinaCloud

    系统将提示输入 Azure 帐户凭据。添加订阅连接的此方法会超时，12 小时后必须重新运行 cmdlet。

    > [AZURE.NOTE]
    如果拥有多个 Azure 订阅，而默认订阅不是要使用的订阅，则可以使用 <strong>Select-AzureSubscription</strong> 选择订阅。

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

    有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)一节。

4. 在脚本窗格中将以下内容追加到脚本：

        # Create a storage context object
        $storageaccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}
        $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageaccountkey

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

5. 单击“运行脚本”或按 **F5** 运行脚本。输出结果类似如下：

    ![教程准备的输出结果][img-preparation-output]  


## <a id="run"></a> 运行 Oozie 项目
目前，Azure PowerShell 不提供用于定义 Oozie 作业的任何 cmdlet。可以使用 **Invoke-RestMethod** cmdlet 调用 Oozie Web 服务。Oozie Web 服务 API 是 HTTP REST JSON API。有关 Oozie Web 服务 API 的详细信息，请参阅 [Apache Oozie 4.0 文档][apache-oozie-400]（对于 HDInsight 群集 3.0 版）或 [Apache Oozie 3.3.2 文档][apache-oozie-332]（对于 HDInsight 群集 2.1 版）。

**提交 Oozie 作业**

1. 打开 Windows PowerShell ISE（在 Windows 8“开始”屏幕上，键入 **PowerShell\_ISE**，然后单击“Windows PowerShell ISE”。有关详细信息，请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][powershell-start]。
2. 将以下脚本复制到脚本窗格，然后设置前 14 个变量（但是，跳过 **$storageUri**）。

        #HDInsight cluster variables
        $clusterName = "<HDInsightClusterName>"
        $clusterUsername = "<HDInsightClusterUsername>"
        $clusterPassword = "<HDInsightClusterUserPassword>"

        #Azure Blob storage (WASB) variables
        $storageAccountName = "<StorageAccountName>"
        $storageContainerName = "<BlobContainerName>"
        $storageUri="wasbs://$storageContainerName@$storageAccountName.blob.core.chinacloudapi.cn"

        #Azure SQL database variables
        $sqlDatabaseServer = "<SQLDatabaseServerName>"
        $sqlDatabaseLogin = "<SQLDatabaseLoginName>"
        $sqlDatabaseLoginPassword = "<SQLDatabaseloginPassword>"
        $sqlDatabaseName = "<SQLDatabaseName>"

        #Oozie WF/coordinator variables
        $coordStart = "2014-03-21T13:45Z"
        $coordEnd = "2014-03-21T13:45Z"
        $coordFrequency = "1440"    # in minutes, 24h x 60m = 1440m
        $coordTimezone = "UTC"    #UTC/GMT

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

    有关这些变量的详细说明，请参阅本教程中的[先决条件](#prerequisites)一节。

    $coordstart 和 $coordend 是工作流的开始和结束时间。要了解 UTC/GMT 时间，请在 bing.com 上搜索“utc 时间”。$coordFrequency 是要运行工作流的频率（以分钟为单位）。
3. 将以下内容追加到脚本。此部分定义 Oozie 有效负载：

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

    > [AZURE.NOTE]
    与工作流提交有效负载文件相比，主要区别是变量 **oozie.coord.application.path**。提交工作流作业时，使用 **oozie.wf.application.path**。

4. 将以下内容追加到脚本。此部分检查 Oozie Web 服务状态：

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

5. 将以下内容追加到脚本。此部分创建 Oozie 作业：

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

    > [AZURE.NOTE]
    提交工作流作业时，必须在创建作业后进行另一次 Web 服务调用以启动作业。在这种情况下，协调器作业会按时间触发。作业将自动启动。

6. 将以下内容追加到脚本。此部分检查 Oozie 作业状态：

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

8. 将以下内容追加到脚本：

        checkOozieServerStatus
        # listOozieJobs
        $oozieJobId = createOozieJob($oozieJobId)
        checkOozieJobStatus($oozieJobId)
        # ShowOozieJobLog($oozieJobId)
        # killOozieJob($oozieJobId)

    如果要运行附加功能，请删除 # 号。
9. 如果 HDinsight 群集版本为 2.1，请将“https://$clusterName.azurehdinsight.cn:443/oozie/v2/”替换为“https://$clusterName.azurehdinsight.cn:443/oozie/v1/”。HDInsight 群集 2.1 版不支持 2 版的 Web 服务。
10. 单击“运行脚本”或按 **F5** 运行脚本。输出结果类似如下：

    ![教程运行工作流输出][img-runworkflow-output]  

11. 连接到 SQL 数据库以查看导出的数据。

**检查作业错误日志**

要解决工作流的疑难问题，可从群集头节点中的 C:\\apps\\dist\\oozie-3.3.2.1.3.2.0-05\\oozie-win-distro\\logs\\Oozie.log 位置找到 Oozie 日志文件。有关 RDP 的信息，请参阅[使用 Azure 门户预览管理 HDInsight 群集][hdinsight-admin-portal]。

**重新运行教程**

要重新运行工作流，必须执行以下任务：

* 删除 Hive 脚本输出文件。
* 删除 log4jLogsCount 表中的数据。

以下是可以使用的示例 Windows PowerShell 脚本：

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
在本教程中，已经学习如何定义 Oozie 工作流，Oozie 协调器，以及如何使用 Azure PowerShell 运行 Oozie 协调器作业。要了解更多信息，请参阅下列文章：

* [开始使用 HDInsight][hdinsight-get-started]
* [将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]
* [使用 Azure PowerShell 管理 HDInsight][hdinsight-admin-powershell]
* [将数据上载到 HDInsight][hdinsight-upload-data]
* [将 Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]
* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
* [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-java-mapreduce]

[hdinsight-cmdlets-download]: http://go.microsoft.com/fwlink/?LinkID=325563

[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal/

[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop/
[hdinsight-provision]: /documentation/articles/hdinsight-hadoop-provision-linux-clusters/
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-develop-java-mapreduce]: /documentation/articles/hdinsight-develop-deploy-java-mapreduce-linux/
[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/

[sqldatabase-get-started]: /documentation/articles/sql-database-get-started/

[azure-management-portal]: https://portal.azure.cn/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account/

[apache-hadoop]: http://hadoop.apache.org/
[apache-oozie-400]: http://oozie.apache.org/docs/4.0.0/
[apache-oozie-332]: http://oozie.apache.org/docs/3.3.2/

[powershell-download]: /downloads/
[powershell-about-profiles]: https://technet.microsoft.com/zh-cn/library/hh847857.aspx
[powershell-install-configure]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs
[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
[powershell-script]: https://technet.microsoft.com/zh-cn/library/dn425048.aspx

[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

[img-workflow-diagram]: ./media/hdinsight-use-oozie-coordinator-time/HDI.UseOozie.Workflow.Diagram.png
[img-preparation-output]: ./media/hdinsight-use-oozie-coordinator-time/HDI.UseOozie.Preparation.Output1.png
[img-runworkflow-output]: ./media/hdinsight-use-oozie-coordinator-time/HDI.UseOozie.RunCoord.Output.png

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->