<properties
    pageTitle="Azure PowerShell 脚本 - 导入 bacpac SQL 数据库 | Azure"
    description="Azure PowerShell 脚本示例 - 使用 PowerShell 从 bacpac 导入 SQL 数据库"
    services="sql-database"
    documentationcenter="sql-database"
    author="janeng"
    manager="jstrauss"
    editor="carlrab"
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="sql-database"
    ms.custom="sample"
    ms.devlang="PowerShell"
    ms.topic="article"
    ms.tgt_pltfrm="sql-database"
    ms.workload="database"
    ms.date="03/07/2017"
    wacn.date="04/17/2017"
    ms.author="janeng"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="e8c26bd7df9703fcc26a90c62c096d39409ed27d"
    ms.lasthandoff="04/07/2017" />

# <a name="import-from-a-bacpac-into-a-sql-database-using-powershell"></a>使用 PowerShell 从 bacpac 导入 SQL 数据库

此示例 PowerShell 脚本从 bacpac 导入数据库。  

在运行此脚本前，请确保已使用 `Add-AzureRmAccount` cmdlet 创建与 Azure 的连接。

## <a name="sample-script"></a>示例脚本

    # Set an admin login and password for your database
    $adminlogin = "ServerAdmin"
    $password = "ChangeYourAdminPassword1"
    # The logical server name has to be unique in the system
    $servername = "server-$($(Get-AzureRMContext).Subscription.SubscriptionId)"
    # The storage account name has to be unique in the system
    $storageaccountname = $("sql$($(Get-AzureRMContext).Subscription.SubscriptionId)").substring(0,23).replace("-", "")
    # The ip address range that you want to allow to access your DB
    $startip = "0.0.0.0"
    $endip = "255.255.255.255"

    # Create a new resource group
    New-AzureRmResourceGroup -Name "myResourceGroup" -Location "China East"

    # Create a new Storage Account 
    New-AzureRmStorageAccount -ResourceGroupName "myResourceGroup" `
        -AccountName $storageaccountname `
        -Location "China East" `
        -Type "Standard_LRS"

    # Create a new storage container 
    New-AzureStorageContainer -Name "importsample" `
        -Context $(New-AzureStorageContext -StorageAccountName $storageaccountname `
            -StorageAccountKey $(Get-AzureRmStorageAccountKey -ResourceGroupName "myResourceGroup" -StorageAccountName $storageaccountname).Value[0])

    # Download sample database from Github
    Invoke-WebRequest -Uri "https://github.com/Microsoft/sql-server-samples/releases/download/wide-world-importers-v1.0/WideWorldImporters-Standard.bacpac" -OutFile "sample.bacpac"

    # Upload sample database into storage container
    Set-AzureStorageBlobContent -Container "importsample" `
        -File "sample.bacpac" `
        -Context $(New-AzureStorageContext -StorageAccountName $storageaccountname `
            -StorageAccountKey $(Get-AzureRmStorageAccountKey -ResourceGroupName "myResourceGroup" -StorageAccountName $storageaccountname).Value[0])

    # Create a new server with a system wide unique server name
    New-AzureRmSqlServer -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -Location "China East" `
        -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList "ServerAdmin", $(ConvertTo-SecureString -String "ASecureP@assw0rd" -AsPlainText -Force))

    # Open up the server firewall so we can connect
    New-AzureRmSqlServerFirewallRule -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -FirewallRuleName "AllowAll" -StartIpAddress $startip -EndIpAddress $endip

    # Import bacpac
    $importRequest = New-AzureRmSqlDatabaseImport -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -DatabaseName "MyImportSample" `
        -DatabaseMaxSizeBytes "262144000" `
        -StorageKeyType "StorageAccessKey" `
        -StorageKey $(Get-AzureRmStorageAccountKey -ResourceGroupName "myResourceGroup" -StorageAccountName $storageaccountname).Value[0] `
        -StorageUri "http://$storageaccountname.blob.core.chinacloudapi.cn/importsample/sample.bacpac" `
        -Edition "Standard" `
        -ServiceObjectiveName "S0" `
        -AdministratorLogin "ServerAdmin" `
        -AdministratorLoginPassword $(ConvertTo-SecureString -String "ASecureP@assw0rd" -AsPlainText -Force)

    # Check import status and wait for the import to complete
    $importStatus = Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink
    [Console]::Write("Importing")
    while ($importStatus.Status -eq "InProgress")
    {
        $importStatus = Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink
        [Console]::Write(".")
        Start-Sleep -s 10
    }
    [Console]::WriteLine("")
    $importStatus

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

    Remove-AzureRmResourceGroup -ResourceGroupName "myResourceGroup"

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [New-AzureRmResourceGroup]() | 创建用于存储所有资源的资源组。 |
| [New-AzureRmSqlServer]() | 创建用于托管 SQL 数据库的逻辑服务器。 |
| [New-AzureRmSqlServerFirewallRule]() | 创建一个防火墙规则，以允许从输入的 IP 地址范围访问服务器上的所有 SQL 数据库。 |
| [New-AzureRmSqlDatabase]() | 在逻辑服务器中创建 SQL 数据库。 |
| [Remove-AzureRmResourceGroup]() | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/)。

可以在 [Azure SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-powershell-samples/)中找到更多 SQL 数据库 PowerShell 脚本示例。