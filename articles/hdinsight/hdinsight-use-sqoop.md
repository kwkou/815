<properties
    pageTitle="使用 Azure HDInsight (Hadoop) 运行 Apache Sqoop 作业 | Azure"
    description="了解如何从工作站使用 Azure PowerShell 在 Hadoop 群集和 Azure SQL 数据库之间运行 Sqoop 导入和导出。"
    editor="cgronlun"
    manager="jhubbard"
    services="hdinsight"
    documentationcenter=""
    tags="azure-portal"
    author="mumian" />
<tags
    ms.assetid="2fdcc6b7-6ad5-4397-a30b-e7e389b66c7a"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/15/2016"
    wacn.date="03/28/2017"
    ms.author="jgao" />  

# 将 Sqoop 与 HDInsight 中的 Hadoop 配合使用
[AZURE.INCLUDE [sqoop-selector](../../includes/hdinsight-selector-use-sqoop.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

了解如何使用 HDInsight 中的 Sqoop 在 HDInsight 群集和 Azure SQL 数据库或 SQL Server 数据库之间进行导入和导出。

虽然选择 Hadoop 处理日志和文件等非结构化和半结构化的数据是理所当然的事，但可能还需要处理存储在关系数据库中的结构化数据。

[Sqoop][sqoop-user-guide-1.4.4] 是一种专用于在 Hadoop 群集和关系数据库之间传输数据的工具。可以使用此工具将数据从关系数据库管理系统 (RDBMS)（如 SQL Server、MySQL 或 Oracle）中导入到 Hadoop 分布式文件系统 (HDFS)，在 Hadoop 中使用 MapReduce 或 Hive 转换数据，然后将数据导回 RDBMS。在本教程中，SQL Server 数据库将用于关系数据库。

有关 HDInsight 群集上支持的 Sqoop 版本，请参阅 [HDInsight 提供的群集版本有哪些新增功能？][hdinsight-versions]。

## 了解方案
HDInsight 群集附带了某些示例数据。以后会用到以下两个示例：

* 位于 */example/data/sample.log* 的 log4j 日志文件。以下日志会从该文件中提取出来：
  
        2012-02-03 18:35:34 SampleClass6 [INFO] everything normal for id 577725851
        2012-02-03 18:35:34 SampleClass4 [FATAL] system problem at id 1991281254
        2012-02-03 18:35:34 SampleClass3 [DEBUG] detail for id 1304807656
        ...
* 一个名为 *hivesampletable* 的 Hive 表，它引用位于 */hive/warehouse/hivesampletable* 中的数据文件。该表包含一些移动设备数据。
  
    | 字段 | 数据类型 |
    | --- | --- |
    | clientid |字符串 |
    | querytime |字符串 |
    | market |字符串 |
    | deviceplatform |字符串 |
    | devicemake |字符串 |
    | devicemodel |字符串 |
    | state |字符串 |
    | country |字符串 |
    | querydwelltime |double |
    | sessionid |bigint |
    | sessionpagevieworder |bigint |

用户需要首先将 *sample.log* 和 *hivesampletable* 导出到 Azure SQL 数据库或 SQL Server，然后使用以下路径将包含移动设备数据的表导回 HDInsight：

    /tutorials/usesqoop/importeddata

## <a name="create-cluster-and-sql-database"></a> 创建群集和 SQL 数据库
本部分演示如何使用 Azure 门户预览和 Azure Resource Manager 模板创建群集、SQL 数据库和 SQL 数据库架构，以便运行教程。可以在 [Azure 快速启动模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-linux-with-sql-database/)中找到模板。Resource Manager 模板调用 bacpac 包，将表架构部署到 SQL 数据库。bacpac 包位于公共 Blob 容器 https://hditutorialdata.blob.core.windows.net/usesqoop/SqoopTutorial-2016-2-23-11-2.bacpac 中。如果想要私有容器用于 bacpac 文件，请使用模板中的以下值：
   
        "storageKeyType": "Primary",
        "storageKey": "<TheAzureStorageAccountKey>",

若要使用 Azure PowerShell 创建群集和 SQL 数据库，请参阅[附录 A](#appendix-a---a-powershell-sample)。

1. 单击以下图像，在 Azure 门户预览中打开 Resource Manager 模板。
   
    <a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-with-sql-database%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-use-sqoop/deploy-to-azure.png" alt="Deploy to Azure"></a>

    >[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；把允许的地域改成“China North”和“China East”；把 HDInsight Linux 版本改为 Azure 中国所支持的 3.5。

2. 输入以下属性：

    - **订阅**：输入 Azure 订阅。
    - **资源组**：创建新的 Azure 资源组或选择现有的资源组。资源组用于管理。它是对象的容器。
    - **位置**：选择区域。
    - **ClusterName**：为将创建的 Hadoop 群输入名称。
    - **群集登录名和密码**：默认登录名是 admin。
    - **SSH 用户名和密码**。
    - **SQL 数据库服务器登录名和密码**。
    - **\_artifacts 位置**：使用默认值，除非你要在其他位置使用自己的 backpac 文件。
    - **\_artifacts 位置 Sas 令牌**：将其留空。
    - **Bacpac 文件名**：使用默认值，除非你要使用自己的 backpac 文件。
     
    以下值在变量部分中硬编码：
     
    | 默认存储帐户名 | <CluterName>store |
    | --- | --- |
    | Azure SQL 数据库服务器名称 |<ClusterName>dbserver |
    | Azure SQL 数据库名称 |<ClusterName>db |
     
    请记下这些值。本教程后面的步骤中将会用到它们。

3\.单击“确定”保存参数。

4\. 在“自定义部署”边栏选项卡中，单击“资源组”下拉框，然后单击“新建”以创建新资源组。资源组是对群集、依赖存储帐户和其他链接资源进行分组的容器。

5\. 单击“法律条款”，然后单击“创建”。

6\. 单击“创建”。你会看到一个标题为“为模板部署提交部署”的新磁贴。创建群集和 SQL 数据库大约需要 20 分钟时间。

如果选择使用现有的 Azure SQL 数据库或 Microsoft SQL Server

* **Azure SQL 数据库**：必须为 Azure SQL 数据库服务器配置防火墙规则，允许从工作站进行访问。有关创建 Azure SQL 数据库和配置防火墙的说明，请参阅 [Azure SQL 数据库入门][sqldatabase-get-started]。
  
    > [AZURE.NOTE]
    默认情况下，可以从 Azure HDInsight 这样的 Azure 服务连接 Azure SQL 数据库。如果禁用此防火墙设置，则必须从 Azure 门户预览启用。有关创建 Azure SQL 数据库和配置防火墙规则的说明，请参阅[创建和配置 SQL 数据库][sqldatabase-create-configue]。
    > 
    > 
    <a name="sql_server_condition"></a>
* **SQL Server**：如果 HDInsight 群集与 SQL Server 位于 Azure 中的同一虚拟网络，则可以使用本文中的步骤将数据导入或导出 SQL Server 数据库。
  
    > [AZURE.NOTE]
    HDInsight 仅支持基于位置的虚拟网络，并且当前不适用于基于地缘组的虚拟网络。
    > 
    > 
  
    * 若要创建和配置虚拟网络，请参阅[使用 Azure 门户预览创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-arm-pportal/)。
    
        * 在数据中心使用 SQL Server 时，必须将虚拟网络配置为*站点到站点*或*点到站点*。
      
            > [AZURE.NOTE]
            对于**点到站点**虚拟网络，SQL Server 必须运行 VPN 客户端配置应用程序，该应用程序可从 Azure 虚拟网络配置的“仪表板”中获得。
            > 
            > 
        * 在 Azure 虚拟机上使用 SQL Server 时，如果托管 SQL Server 的虚拟机是 HDInsight 所在虚拟网络的成员，则可以使用任何虚拟网络配置。
    * 若要在虚拟网络上创建 HDInsight 群集，请参阅[使用自定义选项在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)
    
    > [AZURE.NOTE]
    SQL Server 还必须允许身份验证。必须使用 SQL Server 登录名来完成本文中的步骤。
    > 
    > 

## 运行 Sqoop 作业
HDInsight 可以使用各种方法运行 Sqoop 作业。使用下表来确定哪种方法最适合你，然后访问此链接进行演练。

| **使用此方法**，如果想要... | ...**交互式** shell | ...**批处理** | ...使用此**群集操作系统** | ...从此**客户端操作系统** |
|:--- |:---:|:---:|:--- |:--- |
| [SSH](/documentation/articles/hdinsight-use-sqoop-mac-linux/) |✔ |✔ |Linux |Linux、Unix、Mac OS X 或 Windows |
| [.NET SDK for Hadoop](/documentation/articles/hdinsight-hadoop-use-sqoop-dotnet-sdk/) |&nbsp; |✔ |Linux 或 Windows |Windows（暂时） |
| [Azure PowerShell](/documentation/articles/hdinsight-hadoop-use-sqoop-powershell/) |&nbsp; |✔ |Linux 或 Windows |Windows |

## 限制
* 批量导出 - 在基于 Linux 的 HDInsight 上，用于将数据导出到 Microsoft SQL Server 或 Azure SQL 数据库的 Sqoop 连接器目前不支持批量插入。
* 批处理 - 在基于 Linux 的 HDInsight 上，如果执行插入时使用 `-batch` 开关，Sqoop 将执行多次插入而不是批处理插入操作。

## 后续步骤
现在你已了解如何使用 Sqoop。若要了解更多信息，请参阅以下文章：

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 Oozie 与 HDInsight 配合使用][hdinsight-use-oozie]：在 Oozie 工作流中使用 Sqoop 操作。
* [使用 HDInsight 分析外部测试版延迟数据][hdinsight-analyze-flight-data]：使用 Hive 分析外部测试版延迟数据，然后使用 Sqoop 将数据导出到 Azure SQL 数据库。
* [将数据上传到 HDInsight][hdinsight-upload-data]：了解将数据上传到 HDInsight/Azure Blob 存储的其他方法。

## <a name="appendix-a---a-powershell-sample"></a> 附录 A - PowerShell 示例
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
   
    检查数据库和表的最简单方法是使用 Visual Studio。可以使用 Azure 门户预览检查数据库服务器和数据库。
4. 创建 HDInsight 群集。
   
    若要检查群集，可以使用 Azure 门户预览或 Azure PowerShell。
5. 预处理源数据文件。
   
    在本教程中，一个 log4j log 文件（带分隔符的文件）和一个 Hive 表将导出到 Azure SQL 数据库。带分隔符的文件名为 */example/data/sample.log*。在本教程前面介绍了几个 log4j 日志的示例。在日志文件中，有一些空行和一些类似下面这样的行：
   
        java.lang.Exception: 2012-02-03 20:11:35 SampleClass2 [FATAL] unrecoverable system problem at id 609774657
            at com.osa.mocklogger.MockLogger$2.run(MockLogger.java:83)
   
    对于使用此数据的其他示例来说，这是没有问题的，但要将数据导入到 Azure SQL 数据库或 SQL Server 中，必须删除这些异常。如果存在空字符串，或者元素数量比 Azure SQL 数据库表中所定义字段数量要少的行，Sqoop 导出将会失败。log4jlogs 表有 7 个字符串类型的字段。
   
    此过程将在群集上创建新文件：tutorials/usesqoop/data/sample.log。若要检查修改后的数据文件，可以使用 Azure 门户预览、Azure 存储资源管理器工具或 Azure PowerShell。[HDInsight 入门][hdinsight-get-started]中有一个关于使用 Azure PowerShell 下载文件并显示文件内容的代码示例。
6. 将数据文件导出到 Azure SQL 数据库。
   
    源文件为 tutorials/usesqoop/data/sample.log。数据导出到的表的名称为 log4jlogs。
   
    > [AZURE.NOTE]
    除了连接字符串信息，此部分中的步骤还应适用于 Azure SQL 数据库或 SQL Server。这些步骤已经过以下配置测试：
    ><p> 
    ><p> *Azure 虚拟网络点到站点配置**：虚拟网络已将 HDInsight 群集连接到专用数据中心的 SQL Server。有关详细信息，请参阅[在管理门户中配置点到站点 VPN](/documentation/articles/vpn-gateway-point-to-site-create/)。
    <p> * **Azure HDInsight 3.1**：有关在虚拟网络上创建群集的信息，请参阅[使用自定义选项在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)。
    <p> *SQL Server 2014**：已配置为允许身份验证和运行 VPN 客户端配置包，可以安全地连接到虚拟网络。
    > 
    > 
7. 将 Hive 表导出到 Azure SQL 数据库。
8. 将 mobiledata 表导入 HDInsight 群集。
   
    若要检查修改后的数据文件，可以使用 Azure 门户预览、Azure 存储资源管理器工具或 Azure PowerShell。[HDInsight 入门][hdinsight-get-started]中有一个关于使用 Azure PowerShell 下载文件并显示文件内容的代码示例。

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
    $location = "China East" # used by all Azure services defined in this tutorial

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
    New-AzureRmStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $defaultStorageAccountName `
        -Location $location `
        -Type Standard_LRS

    # Create the default Blob container
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey `
                                    -ResourceGroupName $resourceGroupName `
                                    -Name $defaultStorageAccountName)[0].Value
    $defaultStorageAccountContext = New-AzureStorageContext `
                                        -StorageAccountName $defaultStorageAccountName `
                                        -StorageAccountKey $defaultStorageAccountKey 
    New-AzureStorageContainer `
        -Name $defaultBlobContainerName `
        -Context $defaultStorageAccountContext 

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
        -DefaultStorageContainer $defaultBlobContainerName 

    # Validate the cluster
    Get-AzureRmHDInsightCluster -ClusterName $hdinsightClusterName
    #endregion

    #region - pre-process the source file

    Write-Host "Preprocessing the source file ..." -ForegroundColor Green

    # This procedure creates a new file with $destBlobName
    $sourceBlobName = "example/data/sample.log"
    $destBlobName = "tutorials/usesqoop/data/sample.log"

    # Define the connection string
    $storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=$defaultStorageAccountName;AccountKey=$defaultStorageAccountKey;EndpointSuffix=core.chinacloudapi.cn"

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
    $sqoopDef = New-AzureRmHDInsightSqoopJobDefinition `
        -Command "export --connect $connectionString --table $tableName_log4j --export-dir $exportDir_log4j --input-fields-terminated-by \0x20 -m 1"
    $sqoopJob = Start-AzureRmHDInsightJob `
                    -ClusterName $hdinsightClusterName `
                    -HttpCredential $httpCredential `
                    -JobDefinition $sqoopDef #-Debug -Verbose
    Wait-AzureRmHDInsightJob `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $hdinsightClusterName `
        -HttpCredential $httpCredential `
        -JobId $sqoopJob.JobId

    Write-Host "Standard Error" -BackgroundColor Green
    Get-AzureRmHDInsightJobOutput -ResourceGroupName $resourceGroupName -ClusterName $hdinsightClusterName -DefaultStorageAccountName $defaultStorageAccountName -DefaultStorageAccountKey $defaultStorageAccountKey -DefaultContainer $defaultBlobContainerName -HttpCredential $httpCredential -JobId $sqoopJob.JobId -DisplayOutputType StandardError
    Write-Host "Standard Output" -BackgroundColor Green
    Get-AzureRmHDInsightJobOutput -ResourceGroupName $resourceGroupName -ClusterName $hdinsightClusterName -DefaultStorageAccountName $defaultStorageAccountName -DefaultStorageAccountKey $defaultStorageAccountKey -DefaultContainer $defaultBlobContainerName -HttpCredential $httpCredential -JobId $sqoopJob.JobId -DisplayOutputType StandardOutput

    #endregion

    #region - export a Hive table

    $tableName_mobile = "mobiledata"
    $exportDir_mobile = "/hive/warehouse/hivesampletable"

    $sqoopDef = New-AzureRmHDInsightSqoopJobDefinition `
        -Command "export --connect $connectionString --table $tableName_mobile --export-dir $exportDir_mobile --fields-terminated-by \t -m 1"
    $sqoopJob = Start-AzureRmHDInsightJob `
                    -ClusterName $hdinsightClusterName `
                    -HttpCredential $httpCredential `
                    -JobDefinition $sqoopDef #-Debug -Verbose

    Wait-AzureRmHDInsightJob `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $hdinsightClusterName `
        -HttpCredential $httpCredential `
        -JobId $sqoopJob.JobId

    Write-Host "Standard Error" -BackgroundColor Green
    Get-AzureRmHDInsightJobOutput `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $hdinsightClusterName `
        -DefaultStorageAccountName $defaultStorageAccountName `
        -DefaultStorageAccountKey $defaultStorageAccountKey `
        -DefaultContainer $defaultBlobContainerName `
        -HttpCredential $httpCredential `
        -JobId $sqoopJob.JobId `
        -DisplayOutputType StandardError

    Write-Host "Standard Output" -BackgroundColor Green
    Get-AzureRmHDInsightJobOutput `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $hdinsightClusterName `
        -DefaultStorageAccountName $defaultStorageAccountName `
        -DefaultStorageAccountKey $defaultStorageAccountKey `
        -DefaultContainer $defaultBlobContainerName `
        -HttpCredential $httpCredential `
        -JobId $sqoopJob.JobId `
        -DisplayOutputType StandardOutput

    #endregion

    #region - import a database

    $targetDir_mobile = "/tutorials/usesqoop/importeddata/"

    $sqoopDef = New-AzureRmHDInsightSqoopJobDefinition `
        -Command "import --connect $connectionString --table $tableName_mobile --target-dir $targetDir_mobile --fields-terminated-by \t --lines-terminated-by \n -m 1"

    $sqoopJob = Start-AzureRmHDInsightJob `
                    -ClusterName $hdinsightClusterName `
                    -HttpCredential $httpCredential `
                    -JobDefinition $sqoopDef #-Debug -Verbose

    Wait-AzureRmHDInsightJob `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $hdinsightClusterName `
        -HttpCredential $httpCredential `
        -JobId $sqoopJob.JobId

    Write-Host "Standard Error" -BackgroundColor Green
    Get-AzureRmHDInsightJobOutput `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $hdinsightClusterName `
        -DefaultStorageAccountName $defaultStorageAccountName `
        -DefaultStorageAccountKey $defaultStorageAccountKey `
        -DefaultContainer $defaultBlobContainerName `
        -HttpCredential $httpCredential `
        -JobId $sqoopJob.JobId `
        -DisplayOutputType StandardError

    Write-Host "Standard Output" -BackgroundColor Green
    Get-AzureRmHDInsightJobOutput `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $hdinsightClusterName `
        -DefaultStorageAccountName $defaultStorageAccountName `
        -DefaultStorageAccountKey $defaultStorageAccountKey `
        -DefaultContainer $defaultBlobContainerName `
        -HttpCredential $httpCredential `
        -JobId $sqoopJob.JobId `
        -DisplayOutputType StandardOutput

    #endregion


[azure-management-portal]: https://portal.azure.cn/

[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning/
[hdinsight-provision]: /documentation/articles/hdinsight-hadoop-provision-linux-clusters/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-analyze-flight-data]: /documentation/articles/hdinsight-analyze-flight-delay-data/
[hdinsight-use-oozie]: /documentation/articles/hdinsight-use-oozie/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/

[sqldatabase-get-started]: /documentation/articles/sql-database-get-started/
[sqldatabase-create-configue]: /documentation/articles/sql-database-get-started/

[powershell-start]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx
[powershell-install]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs
[powershell-script]: https://technet.microsoft.com/zh-cn/library/dn425048.aspx

[sqoop-user-guide-1.4.4]: https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->