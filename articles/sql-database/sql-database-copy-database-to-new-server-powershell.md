<properties
    pageTitle="Azure PowerShell 脚本 - 将 SQL 数据库复制到新服务器 | Azure"
    description="Azure PowerShell 脚本示例 - 使用 PowerShell 将 SQL 数据库复制到新服务器"
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
    ms.openlocfilehash="70d82c900bfd80736cbea5fb79958b1f226d5bfb"
    ms.lasthandoff="04/07/2017" />

# <a name="copy-a-sql-database-to-a-new-server-using-powershell"></a>使用 PowerShell 将 SQL 数据库复制到新服务器

此示例 PowerShell 脚本在新服务器中创建现有数据库的副本。 

在运行此脚本前，请确保已使用 `Add-AzureRmAccount` cmdlet 创建与 Azure 的连接。

## <a name="copy-a-database-to-a-new-server"></a>将数据库复制到新服务器

    # Set an admin login and password for your database
    $adminlogin = "ServerAdmin"
    $password = "ChangeYourAdminPassword1"
    # The logical server names have to be unique in the system
    $sourceserver = "source-server-$($(Get-AzureRMContext).Subscription.SubscriptionId)"
    $targetserver = "target-server-$($(Get-AzureRMContext).Subscription.SubscriptionId)"

    # Create new, or get existing resource group
    New-AzureRmResourceGroup -Name "myResourceGroup" -Location "China East"


    # Create a new server with a system wide unique server name
    New-AzureRmSqlServer -ResourceGroupName "myResourceGroup" `
        -ServerName $sourceserver `
        -Location "China East" `
        -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
    New-AzureRmSqlServer -ResourceGroupName "myResourceGroup" `
        -ServerName "target-server-$($(Get-AzureRMContext).Subscription.SubscriptionId)" `
        -Location "China East" `
        -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))


    # Create a blank database in the source-server
    New-AzureRmSqlDatabase  -ResourceGroupName "myResourceGroup" `
        -ServerName $sourceserver `
        -DatabaseName "MySampleDatabase" -RequestedServiceObjectiveName "S0"


    # Copy source database to target server 
    New-AzureRmSqlDatabaseCopy -ResourceGroupName "myResourceGroup" `
        -ServerName $sourceserver `
        -DatabaseName "MySampleDatabase" `
        -CopyResourceGroupName "myResourceGroup" `
        -CopyServerName $targetserver `
        -CopyDatabaseName "CopyOfMySampleDatabase"

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
| [New-AzureRmSqlDatabaseCopy](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqldatabasecopy) | 创建当前使用快照的数据库副本。 |
| [Remove-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/remove-azurermresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/)。

可以在 [Azure SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-powershell-samples/)中找到更多 SQL 数据库 PowerShell 脚本示例。