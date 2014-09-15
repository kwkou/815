<properties linkid="manage-services-hdinsight-analyze-flight-delay-data" urlDisplayName="Analyze flight delay data with HDInsight" pageTitle="Analyze flight delay data using HDInsight | Azure" metaKeywords="" description="Learn how to upload data to HDInsight, how to process the data using Hive, and how to export the results to SQL Database using Sqoop." metaCanonical="" services="hdinsight" documentationCenter="" title="Analyze flight delay data using HDInsight" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />

# 使用 HDInsight 分析航班延误数据

Hive 提供了通过类似 SQL 的脚本语言（称作 *[HiveQL][]*）运行 MapReduce 作业的方法，此方法可用于对大量数据进行汇总、查询和分析。本教程演示如何使用 Hive 计算机场之间的平均延迟，以及如何使用 Sqoop 将结果导出到 SQL Database。

**先决条件：**

在开始阅读本教程前，你必须具有：

-   一个 Azure HDInsight 群集。有关设置 HDInsight 群集的信息，请参阅 [HDInsight 入门][]或[设置 HDInsight 群集][]。
-   已安装并已配置 Azure PowerShell 的工作站。有关说明，请参阅[安装和配置 Azure PowerShell][]。

估计完成时间：30 分钟

## 在本教程中

-   [准备教程][]
-   [创建并上载 HiveQL 脚本][]
-   [执行 HiveQL 脚本][]
-   [将输出导出到 Azure SQL Database][]
-   [后续步骤][]

## 准备教程

本教程将对你的工作站使用来自[美国研究与技术创新管理部门 - 运输统计局][] (RITA) 的航班准时表现数据。你将执行以下操作：

1.  使用 Web 浏览器从 RITA 将准时表现数据下载到你的工作站
2.  使用 Azure PowerShell 将数据上载到 HDInsight
3.  准备使用 Azure PowerShell 对 SQL Database 进行数据导出

**了解 HDInsight 存储**

HDInsight 将 Azure Blob 存储用于数据存储。它称为 *WASB* 或 *Azure 存储空间 - Blob*。WASB 是 Microsoft 在 Azure Blob 存储上的 HDFS 实现。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

设置 HDInsight 群集时，请将 Blob 存储容器指定为默认文件系统，就像在 HDFS 中一样。除了此容器外，你还可以在设置过程中从同一 Azure 存储帐户或不同 Azure 存储帐户添加其他容器。有关添加其他存储帐户的说明，请参阅[设置 HDInsight 群集][]。

为了简化本教程中使用的 PowerShell 脚本，所有文件都存储在默认文件系统容器（位于 */tutorials/flightdelays*）中。默认情况下，此容器与 HDInsight 群集同名。

WASB 语法为：

    wasb[s]://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/<路径>/<文件名>

> [WACOM.NOTE] HDInsight 群集 3.0 版只支持 *wasb://* 语法。较早的 *asv://* 语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.0 群集中不受支持，以后的版本将不会支持该语法。

> WASB 路径是虚拟路径。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

对于存储在默认文件系统容器中的文件，可以使用以下任一 URI 从 HDInsight 进行访问（以 flightdelays.hql 为例）：

    wasb://mycontainer@mystorageaccount.blob.core.chinacloudapi.cn/tutorials/flightdelays/flightdelays.hql
    wasb:///tutorials/flightdelays/flightdelays.hql
    /tutorials/flightdelays/flightdelays.hql

如果要从存储帐户直接访问该文件，则请注意，该文件的 Blob 名称是：

    tutorials/flightdelays/flightdelays.hql

下表列出了本教程中使用的文件：

<table border="1">
<tr><td><strong>文件</strong></td><td><strong>说明</strong></td></tr>
<tr><td>\tutotirals\flightdelays\data</td><td>Hive 的输入航班数据</td></tr>
<tr><td>\tutorials\flightdelays\output</td><td>Hive 的输出</td></tr>
<tr><td>\tutorials\flightdelays\flightdelays.hql</td><td>HiveQL 脚本文件</td></tr>
<tr><td>\tutorials\flightdelays\jobstatus</td><td>Hadoop 作业状态</td></tr>
</table>

**了解 Hive 内部表和外部表**

以下是你需要了解的有关 Hive 内部表和外部表的一些信息：

-   CREATE TABLE 命令创建内部表。数据文件必须位于默认容器中。
-   CREATE TABLE 命令会将数据文件移到 /hive/warehouse/ 文件夹中。
-   CREATE EXTERNAL TABLE 命令创建外部表。数据文件可以位于默认容器以外的位置。
-   CREATE EXTERNAL TABLE 命令不移动数据文件。
-   CREATE EXTERNAL TABLE 命令不允许在 LOCATION 中有任何文件夹。这是本教程生成 sample.log 文件的副本的原因。

有关详细信息，请参阅 [HDInsight：Hive 内部表和外部表简介][cindygross-hive-tables]。

> [WACOM.NOTE] HiveQL 语句之一可创建 Hive 外部表。Hive 外部表将数据文件保留在原始位置。Hive 内部表将数据文件移到 hive\\warehouse。Hive 外部表要求数据文件位于默认文件系统 WASB 容器中。如果你选择将航班数据文件存储在默认 Blob 容器以外的容器中，则必须使用 Hive 内部表。

**下载航班数据**

1.  浏览到[美国研究与技术创新管理部门 - 运输统计局][] (RITA)。
2.  在该页面上，选择以下值：

<table border="1">
<tr><th>名称</th><th>值</th></tr>
<tr><td>筛选年份</td><td>2012</td></tr>
<tr><td>筛选期间</td><td>1 月</td></tr>
<tr><td>字段：</td><td>*Year*、*FlightDate*、*UniqueCarrier*、*Carrier*、*FlightNum*、*OriginAirportID*、*Origin*、*OriginCityName*、*OriginState*、*DestAirportID*、*Dest*、*DestCityName*、*DestState*、*DepDelayMinutes*、*ArrDelay*、*ArrDelayMinutes*、*CarrierDelay*、*WeatherDelay*、*NASDelay*、*SecurityDelay*、*LateAircraftDelay*（清除其他所有字段）</td></tr>
</table>

3.  单击“下载” 。下载每个文件最多需要花费 15 分钟。
4.  将文件解压缩到 **C:\\Tutorials\\FlightDelays\\Data** 文件夹。每个文件均为 CSV 文件且大小约为 60 GB。
5.  将文件重命名为其包含的数据所对应的月份的名称。例如，将包含 1 月份数据的文件命名为 *January.csv*。
6.  重复步骤 2 和步骤 5，为 2012 年中的 12 个月分别下载一个对应的文件。你至少需要一个文件才能运行本教程。

**将航班延迟数据上载到 Azure Blob 存储**

1.  打开 Azure PowerShell。有关说明，请参阅[安装和配置 Azure PowerShell][]。
2.  运行以下命令以连接到 Azure 订阅：

        Add-AzureAccount

    系统将提示你输入 Azure 帐户凭据。

3.  设置前三个变量，然后运行命令。

        $subscriptionName = "<AzureSubscriptionName>"
        $storageAccountName = "<AzureStorageAccountName>"
        $ContainerName = "<BlobStorageContainerName>"

        $localFolder = "C:\Tutorials\FlightDelays\Data"
        $destFolder = "tutorials/flightdelays/data"

        $month = "January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"

    以下是变量及其说明：

    **变量名**

    **说明**

    \$subscriptionName

    你的 Azure 订阅名称。

    \$storageAccountName

    用于存储航班数据文件的 Azure 存储帐户。建议使用默认的存储帐户。

    \$containerName

    用于存储航班数据文件的 Azure Blob 存储容器。建议使用默认的 HDInsight 群集文件系统 Blob 容器。默认情况下，该容器与 HDInsight 群集同名。

    \$localFolder

    这是工作站上存储航班延误文件的文件夹。

    \$destFolder

    这是航班延误数据将上载到的 WASB 路径。Hadoop (HDInsight) 路径区分大小写。

    \$month

    如果你未下载全部 12 个文件，则需要更新此变量

4.  运行以下命令以上载并列出文件：

        # 创建存储上下文对象
        Select-AzureSubscription $subscriptionName
        $storageaccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}
        $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageaccountkey

        # 将本地工作站中的文件复制到 WASB        
        foreach ($item in $month){
        $fileName = "$localFolder\$item.csv"
        $blobName = "$destFolder/$item.csv"

        Write-Host "Copying $fileName to $blobName" -BackgroundColor Green

        Set-AzureStorageBlobContent -File $fileName -Container $dataContainerName -Blob $blobName -Context $destContext
        }

        # 在 HDinsight 上列出已上载的文件
        Get-AzureStorageBlob -Container $dataContainerName  -Context $destContext -Prefix $destFolder

如果你选择使用其他方法上载文件，请确保文件路径是 *tutorials/flightdelays/data*。用于访问文件的语法是：

    wasb://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/tutorials/flightdelays/data

*tutorials/flightdelays/data* 是你在上载文件时创建的虚拟文件夹。验证是否有 12 个文件，每个月对应一个文件。

**准备 SQL 数据库**

1.  打开 Azure PowerShell。
2.  运行以下命令以连接到 Azure 订阅：

        Add-AzureAccount

    系统将提示你输入 Azure 帐户凭据。

3.  设置以下脚本中的前六个变量，然后运行这些命令：

        # Azure 订阅名称
        $subscriptionName = "<AzureSubscriptionName>"

        # SQL 数据库服务器变量
        $sqlDatabaseServer = ""  # 如果已创建 Azure SQL 数据库服务器，则指定该服务器名称。否则使用 ""。
        $sqlDatabaseUsername = "<SQLDatabaseUserName>"
        $sqlDatabasePassword = "<SQLDatabasePassword>"
        $sqlDatabaseLocation = "<MicrosoftDataCenter>"   #例如，China East。

        # SQL 数据库变量
        $sqlDatabaseName = ""  # 如果已创建数据库，则指定该数据库名称。否则使用 ""，让脚本为你创建一个。
        $sqlDatabaseMaxSizeGB = 10

        #用于创建 AvgDelays 表的 SQL 查询字符串
        $sqlDatabaseTableName = "AvgDelays"
        $sqlCreateAvgDelaysTable = " CREATE TABLE [dbo].[$sqlDatabaseTableName](
        [origin_city_name] [nvarchar](50) NOT NULL,
        [weather_delay] float,
        CONSTRAINT [PK_$sqlDatabaseTableName] PRIMARY KEY CLUSTERED   
            (
        [origin_city_name] ASC
            )
            )"

    以下是变量及其说明：

    **变量名**

    **说明**

    \$subscriptionName

    你的 Azure 订阅名称。

    \$sqlDatabaseServer

    Sqoop 用于将数据导出到的 SQL Database 服务器名称。如果你将此项按原样保留，则此脚本将为你创建一个。否则，指定现有 SQL Database 或 SQL Server。

    \$sqlDatabaseUsername

    SQL Database/SQL Server 用户名。

    \$sqlDatabasePassword

    SQL Database/SQL Server 用户密码。

    \$sqlDatabaseLocation

    仅当你想让脚本为你创建 SQL Database 服务器时，才使用此变量。

    \$sqlDatabaseName

    Sqoop 用于将数据导出到的 SQL Database 名称。如果你将此项按原样保留，则此脚本将为你创建一个。否则，指定现有 SQL Database 或 SQL Server。

    \$sqlDatabaseMaxSizeGB

    仅当你想让脚本为你创建 SQL Database 时，才使用此变量。

4.  运行以下命令以创建 SQL Database 服务器/数据库/表。

        # 当有多个订阅时，选择当前 Azure 订阅
        Select-AzureSubscription $subscriptionName

        # 如果收到请求，则创建新的 Azure SQL Database
        if ([string]::IsNullOrEmpty($sqlDatabaseServer))
        {
        $sqlDatabaseServer = New-AzureSqlDatabaseServer -AdministratorLogin $sqlDatabaseUsername -AdministratorLoginPassword $sqlDatabasePassword -Location $sqlDatabaseLocation 
        Write-Host "The new SQL Database server is $sqlDatabaseServer."-BackgroundColor Green

        }
        else
        {
        Write-Host "Use an existing SQL Database server:$sqlDatabaseServer" -BackgroundColor Green
        }

        # 如果收到请求，则创建新的 SQL 数据库
        if ([string]::IsNullOrEmpty($sqlDatabaseName))
        {
        $sqlDatabaseName = "HDISqoop"
        $sqlDatabaseServerCredential = new-object System.Management.Automation.PSCredential($sqlDatabaseUsername, ($sqlDatabasePassword  | ConvertTo-SecureString -asPlainText -Force)) 
        $sqlDatabaseServerConnectionContext = New-AzureSqlDatabaseServerContext -ServerName $sqlDatabaseServer -Credential $sqlDatabaseServerCredential 

        $sqlDatabase = New-AzureSqlDatabase -ConnectionContext $sqlDatabaseServerConnectionContext -DatabaseName $sqlDatabaseName -MaxSizeGB $sqlDatabaseMaxSizeGB

        Write-Host "The new SQL Database is $sqlDatabaseName."-BackgroundColor Green

        }
        else
        {
        Write-Host "Use an existing SQL Database :$sqlDatabaseName" -BackgroundColor Green
        }

        #创建 AvgDelays 表
        $conn = New-Object System.Data.SqlClient.SqlConnection
        $conn.ConnectionString = "Data Source=$sqlDatabaseServer.database.windows.net;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseUsername;Password=$sqlDatabasePassword;Encrypt=true;Trusted_Connection=false;"
        $conn.open()
        $cmd = New-Object System.Data.SqlClient.SqlCommand
        $cmd.connection = $conn
        $cmd.commandtext = $sqlCreateAvgDelaysTable
        $cmd.executenonquery()

        $conn.close()

## 创建并上载 HiveQL 脚本

使用 Azure PowerShell，你可以一次运行多个 HiveQL 语句，或者将 HiveQL 语句打包到一个脚本文件中。在本教程中，你将创建 HiveQL 脚本。必须将脚本文件上载到 WASB。在下一节中，你将使用 Azure PowerShell 运行脚本文件。

HiveQL 脚本将执行以下操作：

1.  **删除 delays\_raw 表**（如果该表已存在）。
2.  **创建 delays\_raw 外部 Hive 表**，并将该表指向航班延误文件所在的 WASB 位置。此查询指定用“,”分隔字段并用“\\n”终止行。这在字段值包含**逗号时将导致出现问题，因为 Hive 无法区分逗号是字段分隔符还是字段值的一部分（在 ORIGIN\_CITY\_NAME 和 DEST\_CITY\_NAME 的字段值中属于此情况）。为了解决此问题，此查询将创建 TEMP 列来保存未正确拆分到列中的数据。
3.  **删除 delays 表**（如果该表已存在）；
4.  **创建 delays 表**。这适用于在进一步处理前清理数据。此查询将从“delays\_raw”表创建一个新表“delays”**。请注意，将不会复制 TEMP 列（如前所述），并且将使用 *substring* 函数从数据中删除引号标记。
5.  **计算平均天气延迟，并按城市名对结果进行分组。**它还会将结果输出到 WASB。请注意，该查询将从数据中删除撇号，并且将排除其中的 *weather\_delay* 值为 *null* 的行，这是必需操作，因为默认情况下，Sqoop（在本教程后面部分使用）不会适当地处理这些值。

如需 HiveQL 命令的完整列表，请参阅 [Hive 数据定义语言][HiveQL]。每条 HiveQL 命令必须以分号结尾。

**创建 HiveQL 脚本文件**

1.  打开 Azure PowerShell。
2.  运行以下命令以连接到 Azure 订阅：

        Add-AzureAccount

3.  设置前两个变量，然后运行命令。

        $storageAccountName = "<AzureStorageAccountName>"
        $containerName ="<BlobStorageContainerName>"

        $hqlLocalFileName = "C:\tutorials\flightdelays\flightdelays.hql"  
        $hqlBlobName = "tutorials/flightdelays/flightdelays.hql" 

        $srcDataFolder = "tutorials/flightdelays/data" 
        $dstDataFolder = "tutorials/flightdelays/output"

    以下是变量及其说明：

    **变量名**

    **说明**

    \$storageAccountName

    用于存储 HiveQL 脚本文件的 Azure 存储帐户。本教程中提供的 PowerShell 脚本需要航班数据文件和脚本文件都位于同一 Azure 存储帐户和 Blob 存储容器中。

    \$containerName

    用于存储 HiveQL 脚本文件的 Azure Blob 存储容器。本教程中提供的 PowerShell 脚本需要航班数据文件和脚本文件都位于同一 Azure 存储帐户和 Blob 存储容器中。

    \$hqlLocalFileName

    HiveQL 脚本在上载到 WASB 之前的本地文件名。为了简化 PowerShell 脚本，你将在本地编写文件，然后使用 Set-AzureStorageBlobContent cmdlet 将脚本文件上载到 HDInsight。

    \$hqlBlobName

    这是带有 WASB 上的路径的脚本文件名

    \$srcDataFolder

    这是 HiveQL 脚本从中拉取数据的 WASB 上的文件夹。

    \$dstDataFolder

    这是 HiveQL 脚本将输出发送到的 WASB 上的文件夹。稍后在本教程中，你将使用 Sqoop 将此文件夹中的数据导出到 Azure SQL Database。

4.  运行以下命令以定义 HiveQL 语句：

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
        "LOCATION 'wasb://$containerName@$storageAccountName.blob.core.chinacloudapi.cn/$srcDataFolder';"

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

        $hqlInsertLocal = "INSERT OVERWRITE DIRECTORY 'wasb://$containerName@$storageAccountName.blob.core.chinacloudapi.cn/$dstDataFolder' " +
        "SELECT regexp_replace(origin_city_name, '''', ''), " +
        "avg(weather_delay) " +
        "FROM delays " +
        "WHERE weather_delay IS NOT NULL " +
        "GROUP BY origin_city_name;"

        $hqlScript = $hqlDropDelaysRaw + $hqlCreateDelaysRaw + $hqlDropDelays + $hqlCreateDelays + $hqlInsertLocal

5.  运行以下命令以将 Hive 脚本文件写入工作站，并将它上载到 WASB：

        # 将 Hive 脚本写入一个本地文件
        $hqlScript | Out-File $hqlLocalFileName -Encoding ascii

        # 创建存储上下文对象
        $storageaccountkey = get-azurestoragekey $storageAccountName | %{$_.Primary}
        $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageaccountkey

        # 将本地工作站中的文件复制到 WASB
        Set-AzureStorageBlobContent -File $hqlLocalFileName -Container $containerName -Blob $hqlBlobName -Context $destContext

        # 列出 WASB 中的脚本文件
        Get-AzureStorageBlob -Container $containerName  -Context $destContext -Prefix $hqlBlobName

    输出应如下所示：

        Container Uri:https://xxxxxxxx.blob.core.chinacloudapi.cn/hdi0212v3

        Name                              BlobType   Length                            ContentType                       LastModified                      SnapshotTime
        ----                              --------   ------                            -----------                       ------------                      ------------
        tutorials/flightdelays/flightd...BlockBlob  1938                              application/octet-stream          2/12/2014 9:57:28 PM +00:00

## 执行 HiveQL 脚本

有几个 Azure PowerShell cmdlet 可用于运行 Hive。本教程使用的是 Invoke-Hive。有关其他方法，请参阅[将 Hive 与 HDInsight 配合使用][]。使用 Invoke-Hive，可以运行 HiveQL 语句或 HiveQL 脚本。你将使用已创建并已上载到 Azure Blob 存储的 HiveQL 脚本。

有一个已知的 Hive 路径问题。可在 [TechNet Wiki][] 上找到用于解决此问题的说明。

**使用 PowerShell 运行 Hive 查询**

1.  打开 Azure PowerShell。
2.  运行以下命令以更改当前目录：

    cd \\Tutorials\\FlightDelays\\

    这一步是必要的，因为你要将 Hive 输出的副本下载到工作站。默认情况下，你对 PowerShell 文件夹没有写入权限。

3.  运行以下命令以连接到 Azure 订阅：

        Add-AzureAccount

4.  设置前三个变量，然后运行这些命令：

        $clusterName = "<HDInsightClusterName>"
        $storageAccountName = "<AzureStorageAccountName>"
        $containerName = "<AzureBlobStorageContainerName>"
        $hqlScriptFile = "wasb://$containerName@$storageAccountName.blob.core.chinacloudapi.cn/tutorials/flightdelays/flightdelays.hql"
        $outputBlobName = "tutorials/flightdelays/output/000000_0"

    以下是变量及其说明：

    **变量名**

    **说明**

    \$clusterName

    将运行 Hive 脚本和 Sqoop 导出的 HDInsight 群集。

    \$storageAccountName

    用于存储 HiveQL 脚本的 Azure 存储帐户。请参阅[创建并上载 HiveQL 脚本][1]。

    \$containerName

    用于存储 HiveQL 脚本的 Azure Blob 存储容器。请参阅[创建并上载 HiveQL 脚本][1]。

    \$hqlScriptFile

    这是 HiveQL 脚本文件的 URI。

    \$outputBlobName

    这是 HiveQL 脚本输出文件。默认名称是 *000000\_0*。

5.  运行以下命令以调用 Hive

        Use-AzureHDInsightCluster $clusterName

        Invoke-Hive –File $hqlScriptFile

6.  运行以下命令以验证结果：

        $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
        $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

        Get-AzureStorageBlobContent -Container $ContainerName -Blob $outputBlobName -Context $storageContext 

        cat ".\$outputBlobName" | findstr /B "Ch"

    输出应如下所示：

        Champaign/Urbana 19.023255813953487
        Charleston 24.93975903614458
        Charleston/Dunbar 16.954545454545453
        Charlotte 16.88616981831665
        Charlotte Amalie 12.88888888888889
        Charlottesville 25.444444444444443
        Chattanooga 19.883561643835616
        Cheyenne 9.875
        Chicago 16.77321649680922
        Chico 12.340909090909092
        Christiansted 12.318181818181818

    在每个城市和延迟时间之间有一个分隔符，该分隔符在 PowerShell 输出窗口中不可见。该分隔符是“\\001”。当你运行 Sqoop 导出时，将使用此分隔符。

## 将 Hive 作业输出导出到 Azure SQL Database

最后的步骤是运行 Sqoop 导出，将数据导出到 SQL Database。在本教程的前面部分，你已创建 SQL Database 和 AvgDelays 表。

**将数据导出到 SQL Database**

1.  打开 Azure PowerShell。
2.  运行以下命令以连接到 Azure 订阅：

        Add-AzureAccount

3.  设置前五个变量，然后运行命令：

        $clusterName = "<HDInsightClusterName>"

        $sqlDatabaseServerName = "<SQLDatabaseServerName>"
        $sqlDatabaseUserName = "<SQLDatabaseUsername>"
        $sqlDatabasePassword = "<SQLDatabasePassword>"

        $sqlDatabaseName = "<SQLDatabaseName>" # 默认名称为“HDISqoop”
        $sqlDatabaseTableName = "AvgDelays" 

        $exportDir = "wasb://$containerName@$storageAccountName.blob.core.chinacloudapi.cn/tutorials/flightdelays/output"

        $sqlDatabaseConnectionString = "jdbc:sqlserver://$sqlDatabaseServerName.database.chinacloudapi.cn;user=$sqlDatabaseUserName@$sqlDatabaseServerName;password=$sqlDatabasePassword;database=$sqlDatabaseName"

    以下是变量及其说明：

    **变量名**

    **说明**

    \$clusterName

    HDInsight 群集名称。

    \$sqlDatabaseServer

    Sqoop 将数据导出到的 SQL Database 服务器。

    \$sqlDatabaseUsername

    SQL Database 用户名。

    \$sqlDatabasePassword

    SQL Database 用户密码。

    \$sqlDatabaseName

    Sqoop 将数据导出到的 SQL Database。默认名称是“HDISqoop”。

    \$sqlDatabaseTableName

    Sqoop 将数据导出到的 SQL Database。默认名称是 AvgDelays。这是先前在教程中创建的表。

    \$exportDir

    这是 Hive 输出文件位置。Sqoop 会将此位置中的文件导出到 SQL Database。

4.  运行以下命令以导出数据：

        $sqoopDef = New-AzureHDInsightSqoopJobDefinition -Command "export --connect $sqlDatabaseConnectionString --table $sqlDatabaseTableName --export-dir $exportDir --fields-terminated-by \001 "

        $sqoopJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $sqoopDef #-Debug -Verbose
        Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 -Job $sqoopJob

        Write-Host "Standard Error" -BackgroundColor Green
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $sqoopJob.JobId -StandardError
        Write-Host "Standard Output" -BackgroundColor Green
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $sqoopJob.JobId -StandardOutput

5.  连接到 SQL Database，并在“AvgDelays”**表中按城市查看平均航班延迟：

    ![HDI.FlightDelays.AvgDelays.Dataset][]

## 后续步骤

现在你已了解如何执行以下操作：将文件上载到 Blob 存储、使用 Blob 存储中的数据填充 Hive 表、运行 Hive 查询以及使用 Sqoop 将数据从 HDFS 导出到 Azure SQL Database。若要了解更多信息，请参阅下列文章：

-   [HDInsight 入门][]
-   [将 Hive 与 HDInsight 配合使用][]
-   [将 Oozie 与 HDInsight 配合使用][]
-   [将 Sqoop 与 HDInsight 配合使用][]
-   [将 Pig 与 HDInsight 配合使用][]
-   [为 HDInsight 开发 Java MapReduce 程序][]
-   [为 HDInsight 开发 C\# Hadoop 流程序][]

  [HiveQL]: https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL
  [HDInsight 入门]: ../hdinsight-get-started/
  [设置 HDInsight 群集]: ../hdinsight-provision-clusters/
  [安装和配置 Azure PowerShell]: ../install-configure-powershell/
  [准备教程]: #prepare
  [创建并上载 HiveQL 脚本]: #createscript
  [执行 HiveQL 脚本]: #executehqlscript
  [将输出导出到 Azure SQL Database]: #exportdata
  [后续步骤]: #nextsteps
  [美国研究与技术创新管理部门 - 运输统计局]: http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time
  [将 Azure Blob 存储与 HDInsight 配合使用]: ../hdinsight-use-blob-storage/
  [将 Hive 与 HDInsight 配合使用]: ../hdinsight-use-hive/
  [TechNet Wiki]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx
  [1]: #createScript
  [HDI.FlightDelays.AvgDelays.Dataset]: ./media/hdinsight-analyze-flight-delay-data/HDI.FlightDelays.AvgDelays.DataSet.png
  [将 Oozie 与 HDInsight 配合使用]: ../hdinsight-use-oozie/
  [将 Sqoop 与 HDInsight 配合使用]: ../hdinsight-use-sqoop/
  [将 Pig 与 HDInsight 配合使用]: ../hdinsight-use-pig/
  [为 HDInsight 开发 Java MapReduce 程序]: ../hdinsight-develop-deploy-java-mapreduce/
  [为 HDInsight 开发 C\# Hadoop 流程序]: ../hdinsight-hadoop-develop-deploy-streaming-jobs/
