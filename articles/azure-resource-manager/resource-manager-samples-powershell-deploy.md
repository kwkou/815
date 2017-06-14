<properties
    pageTitle="Azure PowerShell 脚本示例 - 部署模板 | Azure"
    description="用于部署 Azure Resource Manager 模板的示例脚本。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid=""
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="04/19/2017"
    wacn.date="06/05/2017"
    ms.author="v-yeche"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="7d908b4172d3c9891e9bd5223b70d35b889690b3"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="azure-resource-manager-template-deployment---powershell-script"></a>Azure Resource Manager 模板部署 - PowerShell 脚本

此脚本将 Resource Manager 模板部署到订阅中的资源组。

[AZURE.INCLUDE [sample-powershell-install](../../includes/sample-powershell-install.md)]

[AZURE.INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

    <#
     .SYNOPSIS
        Deploys a template to Azure

     .DESCRIPTION
        Deploys an Azure Resource Manager template

     .PARAMETER subscriptionId
        The subscription id where the template will be deployed.

     .PARAMETER resourceGroupName
        The resource group where the template will be deployed. Can be the name of an existing or a new resource group.

     .PARAMETER resourceGroupLocation
        Optional, a resource group location. If specified, will try to create a new resource group in this location. If not specified, assumes resource group is existing.

     .PARAMETER deploymentName
        The deployment name.

     .PARAMETER templateFilePath
        Optional, path to the template file. Defaults to template.json.

     .PARAMETER parametersFilePath
        Optional, path to the parameters file. Defaults to parameters.json. If file is not found, will prompt for parameter values based on template.
    #>

    param(
     [Parameter(Mandatory=$True)]
     [string]
     $subscriptionId,

     [Parameter(Mandatory=$True)]
     [string]
     $resourceGroupName,

     [string]
     $resourceGroupLocation,

     [Parameter(Mandatory=$True)]
     [string]
     $deploymentName,

     [string]
     $templateFilePath = "template.json",

     [string]
     $parametersFilePath = "parameters.json"
    )

    <#
    .SYNOPSIS
        Registers RPs
    #>
    Function RegisterRP {
        Param(
            [string]$ResourceProviderNamespace
        )

        Write-Host "Registering resource provider '$ResourceProviderNamespace'";
        Register-AzureRmResourceProvider -ProviderNamespace $ResourceProviderNamespace;
    }

    #******************************************************************************
    # Script body
    # Execution begins here
    #******************************************************************************
    $ErrorActionPreference = "Stop"

    # sign in
    Write-Host "Logging in...";
    Login-AzureRmAccount -EnvironmentName AzureChinaCloud;

    # select subscription
    Write-Host "Selecting subscription '$subscriptionId'";
    Select-AzureRmSubscription -SubscriptionID $subscriptionId;

    # Register RPs
    $resourceProviders = @();
    if($resourceProviders.length) {
        Write-Host "Registering resource providers"
        foreach($resourceProvider in $resourceProviders) {
            RegisterRP($resourceProvider);
        }
    }

    #Create or check for existing resource group
    $resourceGroup = Get-AzureRmResourceGroup -Name $resourceGroupName -ErrorAction SilentlyContinue
    if(!$resourceGroup)
    {
        Write-Host "Resource group '$resourceGroupName' does not exist. To create a new resource group, please enter a location.";
        if(!$resourceGroupLocation) {
            $resourceGroupLocation = Read-Host "resourceGroupLocation";
        }
        Write-Host "Creating resource group '$resourceGroupName' in location '$resourceGroupLocation'";
        New-AzureRmResourceGroup -Name $resourceGroupName -Location $resourceGroupLocation
    }
    else{
        Write-Host "Using existing resource group '$resourceGroupName'";
    }

    # Start the deployment
    Write-Host "Starting deployment...";
    if(Test-Path $parametersFilePath) {
        New-AzureRmResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile $templateFilePath -TemplateParameterFile $parametersFilePath;
    } else {
        New-AzureRmResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile $templateFilePath;
    }

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令以删除资源组及其所有资源。

    Remove-AzureRmResourceGroup -Name myResourceGroup

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建部署。 表中的每一项均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [Register-AzureRmResourceProvider](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.resources/register-azurermresourceprovider) | 注册资源提供程序，以便将其资源类型部署到订阅。  |
| [Get-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.resources/get-azurermresourcegroup) | 获取资源组。  |
| [New-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.resources/new-azurermresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzureRmResourceGroupDeployment](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.resources/new-azurermresourcegroupdeployment) | 将 Azure 部署添加到资源组。  |
| [Remove-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 删除资源组及其中包含的所有资源。 |

## <a name="next-steps"></a>后续步骤
* 有关部署模板的简介，请参阅[使用 Resource Manager 模板和 Azure PowerShell 部署资源](/documentation/articles/resource-group-template-deploy/)。
* 有关部署需要 SAS 令牌的模板的信息，请参阅[使用 SAS 令牌部署专用模板](/documentation/articles/resource-manager-powershell-sas-token/)。
* 若要在模板中定义参数，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/#parameters)。
* 有关企业可如何使用 Resource Manager 有效管理订阅的指南，请参阅 [Azure 企业基架 - 出于合规目的监管订阅](/documentation/articles/resource-manager-subscription-governance/)。

<!--Update_Description:new article about deploying the resource manager with powershell-->
