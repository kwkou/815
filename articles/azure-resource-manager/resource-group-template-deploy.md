<properties
    pageTitle="使用 PowerShell 和模板部署资源 | Azure"
    description="使用 Azure Resource Manager 和 Azure PowerShell 将资源部署到 Azure。资源在 Resource Manager 模板中定义。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="55903f35-6c16-4c6d-bf52-dbf365605c3f"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/15/2017"
    wacn.date="03/31/2017"
    ms.author="tomfitz" />  


# 使用 Resource Manager 模板和 Azure PowerShell 部署资源
> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/resource-group-template-deploy/)
- [Azure CLI](/documentation/articles/resource-group-template-deploy-cli/)
- [门户](/documentation/articles/resource-group-template-deploy-portal/)
- [REST API](/documentation/articles/resource-group-template-deploy-rest/)

本主题介绍如何将 Azure PowerShell 与 Resource Manager 模板配合使用向 Azure 部署资源。你的模板可以是本地文件或是可通过 URI 访问的外部文件。如果模板驻留在存储帐户中，你可以限制对该模板的访问，并在部署过程中提供共享访问签名 (SAS) 令牌。

## <a name="deploy"></a> 部署
* 若要快速开始部署，请使用以下命令部署带内联参数的本地模板：

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        Set-AzureRmContext -SubscriptionID {your-subscription-ID}
        New-AzureRmResourceGroup -Name ExampleGroup -Location "China East"
        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile c:\MyTemplates\storage.json -storageNamePrefix contoso -storageSKU Standard_GRS

    部署可能需要几分钟才能完成。完成之后，你将看到一条包含以下结果的消息：

        ProvisioningState       : Succeeded

* 仅当要使用非默认订阅时，才需要 `Set-AzureRmContext` cmdlet。若要查看所有订阅及其 ID，请使用：

        Get-AzureRmSubscription

* 若要部署外部模板，请使用 **TemplateUri** 参数：

        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateUri https://raw.githubusercontent.com/exampleuser/MyTemplates/master/storage.json -storageNamePrefix contoso -storageSKU Standard_GRS

* 若要在文件中传递参数值，请使用 **TemplateParameterFile** 参数：

        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile c:\MyTemplates\storage.json -TemplateParameterFile c:\MyTemplates\storage.parameters.json

  参数文件必须采用以下格式：

        {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
            "storageNamePrefix": {
                "value": "contoso"
            },
            "storageSKU": {
                "value": "Standard_GRS"
            }
         }
        }

[AZURE.INCLUDE [resource-manager-deployments](../../includes/resource-manager-deployments.md)]

若要使用完整模式，请使用 **Mode** 参数：

    New-AzureRmResourceGroupDeployment -Mode Complete -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile c:\MyTemplates\storage.json 

## 使用 SAS 令牌部署专用模板
可以将模板添加到存储帐户，并在部署过程中使用 SAS 令牌链接到这些模板。

> [AZURE.IMPORTANT]
> 通过执行以下步骤，只有帐户所有者可以访问包含模板的 blob。但是，如果为 blob 创建 SAS 令牌，则拥有该 URI 的任何人都可以访问 blob。如果其他用户截获了该 URI，则此用户可以访问该模板。使用 SAS 令牌是限制对模板的访问的好方法，但不应直接在模板中包括密码等敏感数据。
> 
> 

### 将专用模板添加到存储帐户
以下示例设置专用存储帐户容器并上载模板：

    New-AzureRmResourceGroup -Name ManageGroup -Location "China East"
    New-AzureRmStorageAccount -ResourceGroupName ManageGroup -Name {your-unique-name} -Type Standard_LRS -Location "China East"
    Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name {your-unique-name}
    New-AzureStorageContainer -Name templates -Permission Off
    Set-AzureStorageBlobContent -Container templates -File c:\MyTemplates\storage.json

### 在部署期间提供 SAS 令牌
若要在存储帐户中部署专用模板，请生成 SAS 令牌，并将其包括在模板的 URI 中。设置到期时间以允许足够的时间来完成部署。

    Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name {your-unique-name}
    $templateuri = New-AzureStorageBlobSASToken -Container templates -Blob storage.json -Permission r -ExpiryTime (Get-Date).AddHours(2.0) -FullUri
    New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateUri $templateuri

有关将 SAS 令牌与链接模板配合使用的示例，请参阅[将已链接的模版与 Azure Resource Manager 配合使用](/documentation/articles/resource-group-linked-templates/)。

## <a name="parameters"></a> 参数
   
如果你的模板包括的一个参数与 PowerShell 命令中的某个参数同名，系统会提示你提供该参数的值。Azure PowerShell 在提供模板中的参数时包含后缀 **FromTemplate**。例如，模板中名为 **ResourceGroupName** 的参数与 [New-AzureRmResourceGroupDeployment](https://docs.microsoft.com/powershell/resourcemanager/azurerm.resources/v3.3.0/new-azurermresourcegroupdeployment) cmdlet 中的 **ResourceGroupName** 参数冲突。系统会提示你提供 **ResourceGroupNameFromTemplate** 的值。通常，不应将参数命名为与用于部署操作的参数的名称相同以避免这种混乱。

可以在同一部署操作中使用内联参数和本地参数文件。例如，可以在本地参数文件中指定某些值，并在部署期间添加其他内联值。如果同时为本地参数文件中的参数和内联参数提供值，则内联值优先。

但是，使用外部参数文件时，不能传递是内联值的或本地文件中的其他值。如果在 **TemplateParameterUri** 参数中指定参数文件，则将忽略所有内联参数。提供外部文件中的所有参数值。如果模板包括参数文件中无法包括的敏感值，可将该值添加到密钥保管库，或者动态地以内联方式提供所有参数值。

## 调试

如果要记录可能帮助你排查任何部署错误的有关部署的其他信息，请使用 **DeploymentDebugLogLevel** 参数。你可以指定在对部署操作进行日志记录时记录请求内容或/和响应内容。

    New-AzureRmResourceGroupDeployment -Name ExampleDeployment -DeploymentDebugLogLevel All -ResourceGroupName ExampleGroup -TemplateFile storage.json

若要获取有关失败的部署操作的详细信息，请使用：

    (Get-AzureRmResourceGroupDeploymentOperation -DeploymentName ExampleDeployment -ResourceGroupName ExampleGroup).Properties | Where-Object ProvisioningState -eq Failed

有关解决常见部署错误的提示，请参阅[排查使用 Azure Resource Manager 时的常见 Azure 部署错误](/documentation/articles/resource-manager-common-deployment-errors/)。

## 完整部署脚本

以下示例显示用于部署[导出模板](/documentation/articles/resource-manager-export-template/)功能生成的模板的 PowerShell 脚本：

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

## 后续步骤
* 有关通过 .NET 客户端库部署资源的示例，请参阅 [Deploy resources using .NET libraries and a template](/documentation/articles/virtual-machines-windows-csharp-template/)（使用 .NET 库和模板部署资源）。
* 若要在模板中定义参数，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/#parameters)。
<!--* 有关将解决方案部署到不同环境的指南，请参阅 [Development and test environments in Azure](/documentation/articles/solution-dev-test-environments/)（Azure 中的开发和测试环境）。-->
* 如需了解企业如何使用 Resource Manager 对订阅进行有效管理，请参阅 [Azure 企业机架 - 规范性订阅管理](/documentation/articles/resource-manager-subscription-governance/)。
* 有关自动化部署的四部分系列教程，请参阅[将应用程序自动部署到 Azure 虚拟机](/documentation/articles/virtual-machines-windows-dotnet-core-1-landing/)。此系列教程介绍了应用程序体系结构、访问与安全性、可用性与缩放性，以及应用程序部署。

<!---HONumber=Mooncake_0327_2017-->
<!-- Update_Description:update meta properties; adjust the location of the quick start deployment; add debug and whole deployment feature -->