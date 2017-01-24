<properties
    pageTitle="使用 PowerShell 设置新的 SQL 数据库 | Azure"
    description="了解如何使用 PowerShell 创建 SQL 数据库。可以通过 PowerShell cmdlet 管理常见的数据库设置任务。"
    keywords="新建 sql 数据库,数据库设置"
    services="sql-database"
    documentationcenter=""
    author="stevestein"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="7d99869b-cec5-4583-8c1c-4c663f4afd4d"
    ms.service="sql-database"
    ms.custom="single databases"
    ms.devlang="NA"
    ms.topic="hero-article"
    ms.tgt_pltfrm="powershell"
    ms.workload="data-management"
    ms.date="12/09/2016"
    wacn.date="01/20/2017"
    ms.author="sstein" />  


# 开始使用 Azure PowerShell 了解 Azure SQL 数据库服务器、数据库和防火墙规则

本入门教程介绍如何使用 PowerShell 完成以下任务：

* 创建新的 Azure 资源组
* 创建 Aure SQL 逻辑服务器
* 查看 Azure SQL Server 属性
* 创建服务器级防火墙规则
* 将 AdventureWorksLT 示例数据库创建为单一数据库
* 查看 AdventureWorksLT 示例数据库属性
* 创建空的单一数据库

在本教程中，你还将：

* 连接到逻辑服务器及其 master 数据库
* 查看 master 数据库属性
* 连接到示例数据库
* 查看用户数据库属性

完成本教程后，用户会有一个示例数据库和一个空的数据库在 Azure 资源组中运行并连接到逻辑服务器。此外还会配置一项服务器级防火墙规则，目的是让服务器级主体从指定 IP 地址（或 IP 地址范围）登录到服务器。

**时间估计**：本教程大约需要 30 分钟（假定用户已满足先决条件）。


> [AZURE.TIP]
也可以使用 [Azure 门户预览](/documentation/articles/sql-database-get-started/)执行入门教程中的上述相同任务。
>


## 先决条件

* 需要一个 Azure 帐户。可以[注册 Azure 1 元试用帐户](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

* 必须使用帐户登录，该帐户是订阅所有者或参与者角色的成员。有关基于角色的访问控制 (RBAC) 的详细信息，请参阅[开始在 Azure 门户预览中进行访问管理](/documentation/articles/role-based-access-control-what-is/)。

* 需要 Azure Blob 存储中的 AdventureWorksLT 示例数据库 .bacpac 文件

### 请下载 AdventureWorksLT 示例数据库 .bacpac 文件，将其保存在 Azure Blob 存储中

本教程通过从 Azure 存储导入 .bacpac 文件来创建新的 AdventureWorksLT 数据库。第一步是获取 AdventureWorksLT.bacpac 的副本，然后将其上载到 Blob 存储。以下步骤是让示例数据库做好导入准备：

1. [下载 AdventureWorksLT.bacpac](https://sqldbbacpacs.blob.core.windows.net/bacpacs/AdventureWorksLT.bacpac) 并使用 .bacpac 文件扩展名对其进行保存。
2. [创建存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account) - 可以使用默认设置创建存储帐户。
3. 创建新的“容器”，方法是：浏览到存储帐户，选择“Blob”，然后单击“+容器”。
4. 将 .bacpac 文件上载到存储帐户中的 Blob 容器。可以使用容器页面顶部的“上载”按钮，也可以[使用 AzCopy](/documentation/articles/storage-use-azcopy/#blob-upload)。
5. 保存 AdventureWorksLT.bacpac 以后，需要本教程后面的导入代码片段的 URL 和存储帐户密钥。
   * 选择 bacpac 文件并复制 URL。该 URL 将类似于 https://{storage-account-name}.blob.core.chinacloudapi.cn/{container-name}/AdventureWorksLT.bacpac。在存储帐户页面上，单击“访问密钥”，然后复制“key1”。


[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]


## 使用 Azure PowerShell 创建新的逻辑 SQL Server

用户需要包含服务器的资源组，因此第一步是创建新的资源组和服务器（[New-AzureRmResourceGroup](https://docs.microsoft.com/powershell/resourcemanager/azurerm.resources/v3.3.0/new-azurermresourcegroup)、[New-AzureRmSqlServer](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.3.0/new-azurermsqlserver)），或者获取现有资源组和服务器（[Get-AzureRmResourceGroup](https://docs.microsoft.com/powershell/resourcemanager/azurerm.resources/v3.3.0/get-azurermresourcegroup)、[Get-AzureRmSqlServer](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.3.0/get-azurermsqlserver)）的引用。以下代码片段将创建资源组和 Azure SQL Server（如果尚不存在）：

如需有效 Azure 位置和字符串格式的列表，请参阅下面的[帮助器代码片段](#helper-snippets)部分。

	# Create new, or get existing resource group
	############################################

	$resourceGroupName = "{resource-group-name}"
	$resourceGroupLocation = "{resource-group-location}"

	$myResourceGroup = Get-AzureRmResourceGroup -Name $resourceGroupName -ea SilentlyContinue

	if(!$myResourceGroup)
	{
	   Write-Output "Creating resource group: $resourceGroupName"
	   $myResourceGroup = New-AzureRmResourceGroup -Name $resourceGroupName -Location $resourceGroupLocation
	}
	else
	{
	   Write-Output "Resource group $resourceGroupName already exists:"
	}
	$myResourceGroup



	# Create a new, or get existing server
	######################################

	$serverName = "{server-name}"
	$serverVersion = "12.0"
	$serverLocation = $resourceGroupLocation
	$serverResourceGroupName = $resourceGroupName

	$serverAdmin = "{server-admin}"
	$serverAdminPassword = "{server-admin-password}"

	$securePassword = ConvertTo-SecureString –String $serverAdminPassword –AsPlainText -Force
	$serverCreds = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $serverAdmin, $securePassword

	$myServer = Get-AzureRmSqlServer -ServerName $serverName -ResourceGroupName $serverResourceGroupName -ea SilentlyContinue
	if(!$myServer)
	{
	   Write-Output "Creating SQL server: $serverName"
	   $myServer = New-AzureRmSqlServer -ResourceGroupName $serverResourceGroupName -ServerName $serverName -Location $serverLocation -ServerVersion $serverVersion -SqlAdministratorCredentials $serverCreds
	}
	else
	{
	   Write-Output "SQL server $serverName already exists:"
	}
	$myServer




## 使用 Azure PowerShell 查看逻辑 SQL Server 属性


	#$serverResourceGroupName = "{resource-group-name}"
	#$serverName = "{server-name}"

	$myServer = Get-AzureRmSqlServer -ServerName $serverName -ResourceGroupName $serverResourceGroupName 

	Write-Host "Server name: " $myServer.ServerName
	Write-Host "Fully qualified server name: $serverName.database.chinacloudapi.cn"
	Write-Host "Server location: " $myServer.Location
	Write-Host "Server version: " $myServer.ServerVersion
	Write-Host "Server administrator login: " $myServer.SqlAdministratorLogin



## 使用 Azure PowerShell 创建服务器级防火墙规则

若要设置防火墙规则，需知道公共 IP 地址。可以使用所选浏览器获取自己的 IP 地址（询问“我的 IP 地址是什么”）。有关详细信息，请参阅[防火墙规则](/documentation/articles/sql-database-firewall-configure/)。

下面使用 [Get-AzureRmSqlServerFirewallRule](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.3.0/get-azurermsqlserverfirewallrule) 和 [New-AzureRmSqlServerFirewallRule](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.3.0/new-azurermsqlserverfirewallrule) cmdlet 获取引用或创建新的规则。就此代码片段来说，如果规则已存在，则只会获取该规则的引用，不会更新起始和结束 IP 地址。始终可以修改 **else** 子句，使用 [Set-AzureRmSqlServerFirewallRule](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.3.0/set-azurermsqlserverfirewallrule) 执行创建或更新功能。

> [AZURE.NOTE] 
可以针对一个 IP 地址或整个地址范围打开服务器上的 SQL 数据库防火墙。打开防火墙后，SQL 管理员和用户就能够登录到他们具有有效凭据的服务器上的任何数据库。
>


	#$serverName = "{server-name}"
	#$serverResourceGroupName = "{resource-group-name}"
	$serverFirewallRuleName = "{server-firewall-rule-name}"
	$serverFirewallStartIp = "{server-firewall-rule-startIp}"
	$serverFirewallEndIp = "{server-firewall-rule-endIp}"

	$myFirewallRule = Get-AzureRmSqlServerFirewallRule -FirewallRuleName $serverFirewallRuleName -ServerName $serverName -ResourceGroupName $serverResourceGroupName -ea SilentlyContinue

	if(!$myFirewallRule)
	{
	   Write-host "Creating server firewall rule: $serverFirewallRuleName"
	   $myFirewallRule = New-AzureRmSqlServerFirewallRule -ResourceGroupName $serverResourceGroupName -ServerName $serverName -FirewallRuleName $serverFirewallRuleName -StartIpAddress $serverFirewallStartIp -EndIpAddress $serverFirewallEndIp
	}
	else
	{
	   Write-host "Server firewall rule $serverFirewallRuleName already exists:"
	}
	$myFirewallRule



## 使用 Azure PowerShell 连接到 SQL Server

让我们针对 master 数据库运行快速查询，确保我们能够连接到服务器。以下代码片段使用 [.NET Framework Provider for SQL Server (System.Data.SqlClient)](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient(v=vs.110).aspx) 连接和查询服务器的 master 数据库。该代码片段根据我们在前面的代码片段中使用的变量生成连接字符串。将占位符替换为 SQL Server 管理员和密码（在前述步骤中，你在创建服务器时用过）。



	#$serverName = "{server-name}"
	#$serverAdmin = "{server-admin}"
	#$serverAdminPassword = "{server-admin-password}"
	$databaseName = "master"

	$connectionString = "Server=tcp:" + $serverName + ".database.chinacloudapi.cn" + ",1433;Initial Catalog=" + $databaseName + ";Persist Security Info=False;User ID=$serverAdmin;Password=$serverAdminPassword" + ";MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"



	$connection = New-Object System.Data.SqlClient.SqlConnection
	$connection.ConnectionString = $connectionString
	$connection.Open()
	$command = New-Object System.Data.SQLClient.SQLCommand("select * from sys.objects", $connection)

	$command.Connection = $connection
	$reader = $command.ExecuteReader()


	$sysObjects = ""
	while ($reader.Read()) {
	    $sysObjects += $reader["name"] + "`n"
	}
	$sysObjects

	$connection.Close()



## 使用 Azure PowerShell 创建新的 AdventureWorksLT 示例数据库

以下代码片段使用 [New-AzureRmSqlDatabaseImport](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.3.0/new-azurermsqldatabaseimport) cmdlet 导入 AdventureWorksLT 示例数据库的 bacpac。bacpac 位于 Azure Blob 存储中。运行导入 cmdlet 以后，可以使用 [Get-AzureRmSqlDatabaseImportExportStatus](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.3.0/get-azurermsqldatabaseimportexportstatus) cmdlet 监视导入操作的进度。$storageUri 是此前已上载到门户的 bacpac 文件的 URL 属性，应类似于：https://{storage-account}.blob.core.windows.net/{container}/AdventureWorksLT.bacpac


	#$resourceGroupName = "{resource-group-name}"
	#$serverName = "{server-name}"

	$databaseName = "AdventureWorksLT"
	$databaseEdition = "Basic"
	$databaseServiceLevel = "Basic"

	$storageKeyType = "StorageAccessKey"
	$storageUri = "{storage-uri}" # URL of bacpac file you uploaded to your storage account
	$storageKey = "{storage-key}" # key1 in the Access keys setting of your storage account

	$importRequest = New-AzureRmSqlDatabaseImport –ResourceGroupName $resourceGroupName –ServerName $serverName –DatabaseName $databaseName –StorageKeytype $storageKeyType –StorageKey $storageKey -StorageUri $storageUri –AdministratorLogin $serverAdmin –AdministratorLoginPassword $securePassword –Edition $databaseEdition –ServiceObjectiveName $databaseServiceLevel -DatabaseMaxSizeBytes 5000000


	Do {
	     $importStatus = Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink
	     Write-host "Importing database..." $importStatus.StatusMessage
	     Start-Sleep -Seconds 30
	     $importStatus.Status
	   }
	   until ($importStatus.Status -eq "Succeeded")
	$importStatus




## 使用 Azure PowerShell 查看数据库属性

创建数据库后，查看其部分属性。


	#$resourceGroupName = "{resource-group-name}"
	#$serverName = "{server-name}"
	#$databaseName = "{database-name}"

	$myDatabase = Get-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $serverName -DatabaseName $databaseName


	Write-Host "Database name: " $myDatabase.DatabaseName
	Write-Host "Server name: " $myDatabase.ServerName
	Write-Host "Creation date: " $myDatabase.CreationDate
	Write-Host "Database edition: " $myDatabase.Edition
	Write-Host "Database performance level: " $myDatabase.CurrentServiceObjectiveName
	Write-Host "Database status: " $myDatabase.Status


## 使用 Azure PowerShell 连接和查询示例数据库

让我们针对 AdventureWorksLT 数据库运行快速查询，确保我们能够连接。以下代码片段使用 [.NET Framework Provider for SQL Server (System.Data.SqlClient)](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient(v=vs.110).aspx) 连接和查询数据库。该代码片段根据我们在前面的代码片段中使用的变量生成连接字符串。将占位符替换为 SQL Server 管理员密码。


	#$serverName = {server-name}
	#$serverAdmin = "{server-admin}"
	#$serverAdminPassword = "{server-admin-password}"
	#$databaseName = {database-name}

	$connectionString = "Server=tcp:" + $serverName + ".database.chinacloudapi.cn" + ",1433;Initial Catalog=" + $databaseName + ";Persist Security Info=False;User ID=$serverAdmin;Password=$serverAdminPassword" + ";MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"


	$connection = New-Object System.Data.SqlClient.SqlConnection
	$connection.ConnectionString = $connectionString
	$connection.Open()
	$command = New-Object System.Data.SQLClient.SQLCommand("select * from sys.objects", $connection)

	$command.Connection = $connection
	$reader = $command.ExecuteReader()


	$sysObjects = ""
	while ($reader.Read()) {
	    $sysObjects += $reader["name"] + "`n"
	}
	$sysObjects

	$connection.Close()


##<a name="create-a-sql-database-powershell-script"></a> 使用 Azure PowerShell 创建新的空数据库


	#$resourceGroupName = {resource-group-name}
	#$serverName = {server-name}

	$blankDatabaseName = "blankdb"
	$blankDatabaseEdition = "Basic"
	$blankDatabaseServiceLevel = "Basic"


	$myBlankDatabase = New-AzureRmSqlDatabase -DatabaseName $blankDatabaseName -ServerName $serverName -ResourceGroupName $resourceGroupName -Edition $blankDatabaseEdition -RequestedServiceObjectiveName $blankDatabaseServiceLevel
	$myBlankDatabase



## 完成用于创建服务器、防火墙规则和数据库的 Azure PowerShell 脚本




	# Sign in to Azure and set the subscription to work with
	########################################################

	$SubscriptionId = "{subscription-id}"

	Add-AzureRmAccount -EnvironmentName AzureChinaCloud
	Set-AzureRmContext -SubscriptionId $SubscriptionId

	# User variables
	################

	$myResourceGroupName = "{resource-group-name}"
	$myResourceGroupLocation = "{resource-group-location}"

	$myServerName = "{server-name}"
	$myServerVersion = "12.0"
	$myServerLocation = $myResourceGroupLocation
	$myServerResourceGroupName = $myResourceGroupName
	$myServerAdmin = "{server-admin}"
	$myServerAdminPassword = "{server-admin-password}" 

	$myServerFirewallRuleName = "{server-firewall-rule-name}"
	$myServerFirewallStartIp = "{start-ip}"
	$myServerFirewallEndIp = "{end-ip}"

	$myDatabaseName = "AdventureWorksLT"
	$myDatabaseEdition = "Basic"
	$myDatabaseServiceLevel = "Basic"

	$myStorageKeyType = "StorageAccessKey"
	# Get these values from your Azure storage account:
	$myStorageUri = "{http://your-storage-account.blob.core.chinacloudapi.cn/your-container/AdventureWorksLT.bacpac}"
	$myStorageKey = "{your-storage-key}"


	# Create new, or get existing resource group
	############################################

	$resourceGroupName = $myResourceGroupName
	$resourceGroupLocation = $myResourceGroupLocation

	$myResourceGroup = Get-AzureRmResourceGroup -Name $resourceGroupName -ea SilentlyContinue

	if(!$myResourceGroup)
	{
	   Write-host "Creating resource group: $resourceGroupName"
	   $myResourceGroup = New-AzureRmResourceGroup -Name $resourceGroupName -Location $resourceGroupLocation
	}
	else
	{
	   Write-host "Resource group $resourceGroupName already exists:"
	}
	$myResourceGroup


	# Create a new, or get existing server
	######################################

	$serverName = $myServerName
	$serverVersion = $myServerVersion
	$serverLocation = $myServerLocation
	$serverResourceGroupName = $myServerResourceGroupName

	$serverAdmin = $myServerAdmin
	$serverAdminPassword = $myServerAdminPassword

	$securePassword = ConvertTo-SecureString –String $serverAdminPassword –AsPlainText -Force
	$serverCreds = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $serverAdmin, $securePassword

	$myServer = Get-AzureRmSqlServer -ServerName $serverName -ResourceGroupName $serverResourceGroupName -ea SilentlyContinue
	if(!$myServer)
	{
	   Write-host "Creating SQL server: $serverName"
	   $myServer = New-AzureRmSqlServer -ResourceGroupName $serverResourceGroupName -ServerName $serverName -Location $serverLocation -ServerVersion $serverVersion -SqlAdministratorCredentials $serverCreds
	}
	else
	{
	   Write-host "SQL server $serverName already exists:"
	}
	$myServer


	# View server properties
	##########################

	$resourceGroupName = $myResourceGroupName
	$serverName = $myServerName

	$myServer = Get-AzureRmSqlServer -ServerName $serverName -ResourceGroupName $serverResourceGroupName 

	Write-Host "Server name: " $myServer.ServerName
	Write-Host "Fully qualified server name: $serverName.database.chinacloudapi.cn"
	Write-Host "Server location: " $myServer.Location
	Write-Host "Server version: " $myServer.ServerVersion
	Write-Host "Server administrator login: " $myServer.SqlAdministratorLogin


	# Create or update server firewall rule
	#######################################

	$serverFirewallRuleName = $myServerFirewallRuleName
	$serverFirewallStartIp = $myServerFirewallStartIp
	$serverFirewallEndIp = $myServerFirewallEndIp

	$myFirewallRule = Get-AzureRmSqlServerFirewallRule -FirewallRuleName $serverFirewallRuleName -ServerName $serverName -ResourceGroupName $serverResourceGroupName -ea SilentlyContinue

	if(!$myFirewallRule)
	{
	   Write-host "Creating server firewall rule: $serverFirewallRuleName"
	   $myFirewallRule = New-AzureRmSqlServerFirewallRule -ResourceGroupName $serverResourceGroupName -ServerName $serverName -FirewallRuleName $serverFirewallRuleName -StartIpAddress $serverFirewallStartIp -EndIpAddress $serverFirewallEndIp
	}
	else
	{
	   Write-host "Server firewall rule $serverFirewallRuleName already exists:"
	}
	$myFirewallRule


	# Connect to the server and master database
	###########################################
	$databaseName = "master"

	$connectionString = "Server=tcp:" + $serverName + ".database.chinacloudapi.cn" + ",1433;Initial Catalog=" + $databaseName + ";Persist Security Info=False;User ID=" + $myServer.SqlAdministratorLogin + ";Password=" + $myServerAdminPassword + ";MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"


	$connection = New-Object System.Data.SqlClient.SqlConnection
	$connection.ConnectionString = $connectionString
	$connection.Open()
	$command = New-Object System.Data.SQLClient.SQLCommand("select * from sys.objects", $connection)

	$command.Connection = $connection
	$reader = $command.ExecuteReader()

	$sysObjects = ""
	while ($reader.Read()) {
	    $sysObjects += $reader["name"] + "`n"
	}
	$sysObjects

	$connection.Close()


	# Create the AdventureWorksLT database from a bacpac
	####################################################

	$resourceGroupName = $myResourceGroupName
	$serverName = $myServerName

	$databaseName = $myDatabaseName
	$databaseEdition = $myDatabaseEdition
	$databaseServiceLevel = $myDatabaseServiceLevel

	$storageKeyType = $myStorageKeyType
	$storageUri = $myStorageUri
	$storageKey = $myStorageKey

	$importRequest = New-AzureRmSqlDatabaseImport –ResourceGroupName $resourceGroupName –ServerName $serverName –DatabaseName $databaseName –StorageKeytype $storageKeyType –StorageKey $storageKey -StorageUri $storageUri –AdministratorLogin $serverAdmin –AdministratorLoginPassword $securePassword –Edition $databaseEdition –ServiceObjectiveName $databaseServiceLevel -DatabaseMaxSizeBytes 5000000

	Do {
	     $importStatus = Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink
	     Write-host "Importing database..." $importStatus.StatusMessage
	     Start-Sleep -Seconds 30
	     $importStatus.Status
	   }
	   until ($importStatus.Status -eq "Succeeded")
	$importStatus


	# View database properties
	##########################

	$resourceGroupName = $myResourceGroupName
	$serverName = $myServerName
	$databaseName = $myDatabaseName

	$myDatabase = Get-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $serverName -DatabaseName $databaseName

	Write-Host "Database name: " $myDatabase.DatabaseName
	Write-Host "Server name: " $myDatabase.ServerName
	Write-Host "Creation date: " $myDatabase.CreationDate
	Write-Host "Database edition: " $myDatabase.Edition
	Write-Host "Database performance level: " $myDatabase.CurrentServiceObjectiveName
	Write-Host "Database status: " $myDatabase.Status


	# Query the database
	####################

	$connectionString = "Server=tcp:" + $serverName + ".database.chinacloudapi.cn" + ",1433;Initial Catalog=" + $databaseName + ";Persist Security Info=False;User ID=" + $myServer.SqlAdministratorLogin + ";Password=" + $myServerAdminPassword + ";MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"

	$connection = New-Object System.Data.SqlClient.SqlConnection
	$connection.ConnectionString = $connectionString
	$connection.Open()
	$command = New-Object System.Data.SQLClient.SQLCommand("select * from sys.objects", $connection)

	$command.Connection = $connection
	$reader = $command.ExecuteReader()

	$sysObjects = ""
	while ($reader.Read()) {
	    $sysObjects += $reader["name"] + "`n"
	}
	$sysObjects

	$connection.Close()


	# Create a blank database
	#########################

	$blankDatabaseName = "blankdb"
	$blankDatabaseEdition = "Basic"
	$blankDatabaseServiceLevel = "Basic"


	$myBlankDatabase = New-AzureRmSqlDatabase -DatabaseName $blankDatabaseName -ServerName $serverName -ResourceGroupName $resourceGroupName -Edition $blankDatabaseEdition -RequestedServiceObjectiveName $blankDatabaseServiceLevel
	$myBlankDatabase




> [AZURE.TIP]
可以删除不使用的数据库，这样可以节省一些学习费用。“基本”版数据库可以在 7 天内还原。但是，请勿删除服务器。如果这样做，则无法恢复服务器或其已删除的数据库。

##<a name="helper-snippets"></a> 帮助器代码片段


	# Get a list of Azure regions where you can create SQL resources

	$sqlRegions = (Get-AzureRmLocation | Where-Object { $_.Providers -eq "Microsoft.Sql" })
	foreach ($region in $sqlRegions)
	{
	   $region.Location
	}

	# Clean up
	# Delete a resource group (and all contained resources)
	Remove-AzureRmResourceGroup -Name {resource-group-name}


> [AZURE.TIP]
可以删除不使用的数据库，这样可以节省一些学习费用。“基本”版数据库可以在七天内还原。但是，请勿删除服务器。如果这样做，则无法恢复服务器或其已删除的数据库。
>

## 后续步骤
完成这第一个入门教程并使用某些示例数据创建数据库以后，还可以查看很多其他的教程，这些教程介绍如何构建你在本教程中学习到的内容。

* 若要开始了解 Azure SQL 数据库安全性，请参阅[安全入门](/documentation/articles/sql-database-get-started-security/)。
* 如果熟悉 Excel，请学习如何[使用 Excel 连接到 Azure 中的 SQL 数据库](/documentation/articles/sql-database-connect-excel/)。
* 如果已准备好开始编写代码，请在[用于 SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries/)中选择编程语言。
* 若要将本地 SQL Server 数据库移到 Azure，请参阅[将数据库迁移到 SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。
* 若要使用 BCP 命令行工具将 CSV 文件中的某些数据载入新表，请参阅[使用 BCP 将 CSV 文件中的数据载入 SQL 数据库](/documentation/articles/sql-database-load-from-csv-with-bcp/)。
* 若要开始创建表和其他对象，请参阅[创建表](https://msdn.microsoft.com/zh-cn/library/ms365315.aspx)中的“创建表”主题。

## 其他资源
[什么是 SQL 数据库？](/documentation/articles/sql-database-technical-overview/)

<!---HONumber=Mooncake_0116_2017-->
<!--update: whole content update include steps, powershell scripts, code samples-->