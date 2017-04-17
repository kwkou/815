<properties
    pageTitle="Azure PowerShell 脚本 - 配置数据库审核和威胁检测 | Azure"
    description="Azure PowerShell 脚本示例 - 使用 PowerShell 配置 SQL 数据库审核和威胁检测"
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
    ms.openlocfilehash="b9062753dcef4046adae2a66a6c61998ca56031d"
    ms.lasthandoff="04/07/2017" />

# <a name="configure-sql-database-auditing-and-threat-detection-using-powershell"></a>使用 PowerShell 配置 SQL 数据库审核和威胁检测

此示例 PowerShell 脚本配置 SQL 数据库审核和威胁检测。 

在运行此脚本前，请确保已使用 `Add-AzureRmAccount` cmdlet 创建与 Azure 的连接。  

## <a name="sample-script"></a>示例脚本

    # Set an admin login and password for your database
    $adminlogin = "ServerAdmin"
    $password = "ChangeYourAdminPassword1"
    # The logical server name has to be unique in the system
    $servername = "server-$($(Get-AzureRMContext).Subscription.SubscriptionId)"
    # The storage account name has to be unique in the system
    $storageaccountname = $("sql$($(Get-AzureRMContext).Subscription.SubscriptionId)").substring(0,23).replace("-", "")
    # Specify the email recipients for the threat detection alerts
    $notificationemailreceipient = "changeto@your.email;changeto@your.email"

    # Create a new resource group
    New-AzureRmResourceGroup -Name "myResourceGroup" -Location "China East"

    # Create a new server with a system wide unique server name
    New-AzureRmSqlServer -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -Location "China East" `
        -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))

    # Create a blank database with S0 performance level
    New-AzureRmSqlDatabase  -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -DatabaseName "MySampleDatabase" `
        -RequestedServiceObjectiveName "S0"

    # Create a new Storage Account 
    New-AzureRmStorageAccount -ResourceGroupName "myResourceGroup" `
        -AccountName $storageaccountname `
        -Location "China East" `
        -Type "Standard_LRS"

    # Set an auditing policy
    Set-AzureRmSqlDatabaseAuditingPolicy -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -DatabaseName "MySampleDatabase" `
        -StorageAccountName $storageaccountname `

    # Set a threat detection policy
    Set-AzureRmSqlDatabaseThreatDetectionPolicy -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -DatabaseName "MySampleDatabase" `
        -StorageAccountName $storageaccountname `
        -NotificationRecipientsEmails $notificationemailreceipient `
        -EmailAdmins $False

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

    Remove-AzureRmResourceGroup -ResourceGroupName "myResourceGroup"

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [New-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/new-azurermresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzureRmSqlServer](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqlserver) | 创建用于托管数据库或弹性池的逻辑服务器。 |
| [New-AzureRmSqlDatabase](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqldatabase) | 在逻辑服务器中创建数据库作为单一数据库或入池数据库。 |
| [New-AzureRmStorageAccount](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.storage/v2.4.0/new-azurermstorageaccount) | 创建存储帐户。 |
| [Set-AzureRmSqlDatabaseAuditingPolicy](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/set-azurermsqldatabaseauditingpolicy) | 设置数据库的审核策略。 |
| [Set-AzureRmSqlDatabaseThreatDetectionPolicy](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/set-azurermsqldatabasethreatdetectionpolicy) | 在数据库上设置威胁检测策略。 |
| [Remove-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/remove-azurermresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/)。

可以在 [Azure SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-powershell-samples/)中找到更多 SQL 数据库 PowerShell 脚本示例。