<properties
    pageTitle="Azure PowerShell 与资源管理器 | Azure"
    description="介绍如何使用 Azure PowerShell 将作为资源组的多个资源部署到 Azure。"
    services="azure-resource-manager"
    documentationcenter=""
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="b33b7303-3330-4af8-8329-c80ac7e9bc7f"
    ms.service="azure-resource-manager"
    ms.workload="multiple"
    ms.tgt_pltfrm="powershell"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/05/2016"
    wacn.date="01/06/2017"
    ms.author="tomfitz" />  

# 使用 PowerShell 和 Resource Manager 管理 Azure 资源
> [AZURE.SELECTOR]
- [门户](/documentation/articles/resource-group-portal/)
- [Azure CLI](/documentation/articles/xplat-cli-azure-resource-manager/)
- [Azure PowerShell](/documentation/articles/powershell-azure-resource-manager/)
- [REST API](/documentation/articles/resource-manager-rest-api/)

本主题介绍如何使用 Azure PowerShell 和 Azure Resource Manager 管理解决方案。如果不熟悉 Resource Manager，请参阅 [Resource Manager 概述](/documentation/articles/resource-group-overview/)。本主题重点介绍管理任务。用户将能够：

1. 创建资源组
2. 将资源添加到资源组
3. 向资源添加标记
4. 根据名称或标记值查询资源
5. 向资源应用和删除锁
6. 从资源组创建 Resource Manager 模板
7. 删除资源组

## Azure PowerShell 入门

如果未安装 Azure PowerShell，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。

如果过去安装了 Azure PowerShell，但最近未更新它，请考虑安装最新版本。可通过用于安装的方法更新版本。例如，如果使用 Web 平台安装程序，请再次启动它以查找更新。

若要检查 Azure 资源模块的版本，请使用以下 cmdlet：

    Get-Module -ListAvailable -Name AzureRm.Resources | Select Version

本主题已针对版本 3.3.0 更新。如果使用更旧的版本，体验可能与本主题中所示步骤不完全相同。有关此版本中 cmdlet 的文档，请参阅 [AzureRM.Resources 模块](https://docs.microsoft.com/powershell/resourcemanager/azurerm.resources/v3.3.0/azurerm.resources)。

## 登录到 Azure 帐户
处理解决方案之前，必须登录到帐户。

若要登录到 Azure 帐户，请使用 **Add-AzureRmAccount -EnvironmentName AzureChinaCloud** cmdlet。

    Add-AzureRmAccount -EnvironmentName AzureChinaCloud

该 cmdlet 将提示您提供您的 Azure 帐户的登录凭据。登录后它会下载你的帐户设置，供 Azure PowerShell 使用。

该 cmdlet 将返回有关你的帐户和要用于任务的订阅的信息。

    Environment           : AzureCloud
    Account               : example@contoso.com
    TenantId              : {guid}
    SubscriptionId        : {guid}
    SubscriptionName      : Example Subscription One
    CurrentStorageAccount :

如果有多个订阅，可切换到其他订阅。首先，请看你的帐户的所有订阅。

    Get-AzureRmSubscription

它将返回已启用和已禁用的订阅。

    SubscriptionName : Example Subscription One
    SubscriptionId   : {guid}
    TenantId         : {guid}
    State            : Enabled

    SubscriptionName : Example Subscription Two
    SubscriptionId   : {guid}
    TenantId         : {guid}
    State            : Enabled

    SubscriptionName : Example Subscription Three
    SubscriptionId   : {guid}
    TenantId         : {guid}
    State            : Disabled

若要切换到其他订阅，请使用 **Set-AzureRmContext** cmdlet 提供订阅名称。

    Set-AzureRmContext -SubscriptionName "Example Subscription Two"

## 创建资源组
将任何资源部署到订阅之前，必须先创建将包含资源的资源组。

若要创建资源组，请使用 **New-AzureRmResourceGroup** cmdlet。该命令使用 **Name** 参数来指定资源组的名称，并使用 **Location** 参数来指定其位置。

    New-AzureRmResourceGroup -Name TestRG1 -Location "China East"

输入格式如下：

    ResourceGroupName : TestRG1
    Location          : chinaeast
    ProvisioningState : Succeeded
    Tags              :
    ResourceId        : /subscriptions/{guid}/resourceGroups/TestRG1

如果稍后需要检索资源组，请使用以下 cmdlet：

    Get-AzureRmResourceGroup -ResourceGroupName TestRG1

若要获取订阅中的所有资源组，请勿指定名称：

    Get-AzureRmResourceGroup

## 将资源添加到资源组
若要将资源添加到资源组中，可使用 **New-AzureRmResource** cmdlet 或特定于要创建的资源类型的 cmdlet（例如 **New-AzureRmStorageAccount**）。使用特定于资源类型的 cmdlet 可能更轻松，因为它包含新资源组所需属性的参数。若要使用 **New-AzureRmResource**，必须了解将不会提示而设置所有属性。

但是，通过 cmdlet 添加资源可能导致将来出现混乱，因为新的资源不存在于 Resource Manager 模板中。Azure 建议在 Resource Manager 模板中定义 Azure 解决方案的基础结构。通过模板，可以可靠地重复部署解决方案。本主题不演示如何将 Resource Manager 模板部署到订阅。有关详细信息，请参阅[使用 Resource Manager 模板和 Azure PowerShell 部署资源](/documentation/articles/resource-group-template-deploy/)。本主题使用 PowerShell cmdlet 创建存储帐户，但稍后从资源组生成模板。

以下 cmdlet 可创建存储帐户。请勿使用示例所示的名称，而是为存储帐户提供唯一名称。此名称必须为 3 到 24 个字符，只能使用数字和小写字母。如果使用示例所示名称，将收到错误，因为该名称被使用。

    New-AzureRmStorageAccount -ResourceGroupName TestRG1 -AccountName mystoragename -Type "Standard_LRS" -Location "China East"

如果稍后需要检索此资源组，请使用以下 cmdlet：

    Get-AzureRmResource -ResourceName mystoragename -ResourceGroupName TestRG1

## 添加标记

标记可用于根据属性组织资源。例如，可能有不同资源组中的多项资源属于同一部门。可对这些资源应用部门标签和值，将其标记为属于同一类别。也可标记资源是用于生产环境还是测试环境。在本主题中，只对一项资源应用标记，但在你的环境最好向所有资源应用标记。

以下 cmdlet 将向你的存储帐户应用两个标记：

    Set-AzureRmResource -Tag @{ Dept="IT"; Environment="Test" } -ResourceName mystoragename -ResourceGroupName TestRG1 -ResourceType Microsoft.Storage/storageAccounts

各个标记作为单个对象更新。若要向已包含标记的资源添加标记，请首先检索现有标记。将新标记添加到包含现有标记的对象，并将所有标记重新应用到资源。

    $tags = (Get-AzureRmResource -ResourceName mystoragename -ResourceGroupName TestRG1).Tags
    $tags += @{Status="Approved"}
    Set-AzureRmResource -Tag $tags -ResourceName mystoragename -ResourceGroupName TestRG1 -ResourceType Microsoft.Storage/storageAccounts

## 搜索资源

使用 **Find-AzureRmResource** cmdlet 可按不同搜索条件检索资源。

* 若要按名称获取资源，请提供 **ResourceNameContains** 参数：

        Find-AzureRmResource -ResourceNameContains mystoragename

* 若要获取资源组中的所有资源，请提供 **ResourceGroupNameContains** 参数：

        Find-AzureRmResource -ResourceGroupNameContains TestRG1

* 若要获取具有某个标记名称和值的所有资源，请提供 **TagName** 和 **TagValue** 参数：

        Find-AzureRmResource -TagName Dept -TagValue IT

* 若要获取具有特定资源类型的所有资源，请提供 **ResourceType** 参数：

        Find-AzureRmResource -ResourceType Microsoft.Storage/storageAccounts

## 锁定资源

需要确保不会意外删除或修改关键资源时，请对资源应用锁定。可指定 **CanNotDelete** 或 **ReadOnly**。

若要创建或删除管理锁，必须有权执行 `Microsoft.Authorization/*` 或 `Microsoft.Authorization/locks/*` 操作。在内置角色中，只有“所有者”和“用户访问管理员”有权执行这些操作。

若要应用锁定，请使用以下 cmdlet：

    New-AzureRmResourceLock -LockLevel CanNotDelete -LockName LockStorage -ResourceName mystoragename -ResourceType Microsoft.Storage/storageAccounts -ResourceGroupName TestRG1

上例中，在删除锁之前，无法删除锁定的资源。若要删除所，请使用：

    Remove-AzureRmResourceLock -LockName LockStorage -ResourceName mystoragename -ResourceType Microsoft.Storage/storageAccounts -ResourceGroupName TestRG1

有关设置锁的详细信息，请参阅[使用 Azure Resource Manager 锁定资源](/documentation/articles/resource-group-lock-resources/)。

## 导出 Resource Manager 模板
对于现有的资源组（通过 PowerShell 或门户等其他方法部署），可以查看资源组的 Resource Manager 模板。导出模板有两个好处：

1. 由于模板中定义了所有基础结构，因此将来可以轻松地自动完成解决方案的部署。
2. 可以查看代表解决方案的 JavaScript 对象表示法 (JSON)，以此熟悉模板语法。

> [AZURE.NOTE]
导出模板功能处于预览状态，并非所有的资源类型目前都支持导出模板。尝试导出模板时，你可能会看到一个错误，指出未导出某些资源。如果需要，你可以在下载模板之后，在模板中手动定义这些资源。
>
>

若要查看资源组的模板，请运行 **Export-AzureRmResourceGroup** cmdlet。

    Export-AzureRmResourceGroup -ResourceGroupName TestRG1 -Path c:\Azure\Templates\Downloads\TestRG1.json

可使用许多选项和方案导出 Resource Manager 模板。有关详细信息，请参阅[从现有资源导出 Azure Resource Manager 模板](/documentation/articles/resource-manager-export-template/)。

## 删除资源或资源组
可以删除资源或资源组。删除资源组时，还会删除该资源组中的所有资源。

* 若要从资源组中删除资源，请使用 **Remove-AzureRmResource** cmdlet。此 cmdlet 将删除该资源，但不会删除该资源组。

        Remove-AzureRmResource -ResourceName mystoragename -ResourceType Microsoft.Storage/storageAccounts -ResourceGroupName TestRG1

* 若要删除资源组及其所有资源，请使用 **Remove-AzureRmResourceGroup** cmdlet。

        Remove-AzureRmResourceGroup -Name TestRG1

使用这两个 cmdlet，都会要求你确认要删除的资源或资源组。如果操作成功删除资源或资源组，将返回 **True**。

## 使用 Azure 自动化运行 Resource Manager 脚本

本主题演示如何通过 Azure PowerShell 对资源执行基本操作。如果使用更高级的管理方案，通常需要创建脚本，然后按需或按计划重复使用该脚本。通过 [Azure 自动化](/documentation/articles/automation-intro/)，可自动执行用于管理 Azure 解决方案的常用脚本。

以下主题演示如何使用 Azure 自动化、Resource Manager 和 PowerShell 来有效执行管理任务：

- 有关创建 Runbook 的信息，请参阅[我的第一个 Runbook](/documentation/articles/automation-first-runbook-textual/)。
- 有关使用脚本库的信息，请参阅 [Azure 自动化的 Runbook 和模块库](/documentation/articles/automation-runbook-gallery/)。

## 后续步骤
* 若要了解如何创建资源管理器模板，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。
* 若要了解部署模板，请参阅[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy/)。
* 可以将现有资源移动到新的资源组。有关示例，请参阅[将资源移动到新的资源组或订阅中](/documentation/articles/resource-group-move-resources/)。
* 如需了解企业如何使用 Resource Manager 对订阅进行有效管理，请参阅 [Azure 企业机架 - 规范性订阅管理](/documentation/articles/resource-manager-subscription-governance/)。

<!---HONumber=Mooncake_0103_2017-->