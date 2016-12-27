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
    ms.date="11/16/2016"
    wacn.date="12/26/2016"
    ms.author="tomfitz" />  


# 使用 Resource Manager 模板和 Azure PowerShell 部署资源
> [AZURE.SELECTOR]
* [Azure PowerShell](/documentation/articles/powershell-azure-resource-manager/)
* [Azure CLI](/documentation/articles/xplat-cli-azure-resource-manager/)
* [门户](/documentation/articles/resource-group-portal/)
* [REST API](/documentation/articles/resource-manager-rest-api/)

本主题介绍如何将 Azure PowerShell 与 Resource Manager 模板配合使用向 Azure 部署资源。你的模板可以是本地文件或是可通过 URI 访问的外部文件。如果模板驻留在存储帐户中，你可以限制对该模板的访问，并在部署过程中提供共享访问签名 (SAS) 令牌。

> [AZURE.TIP]
有关在部署过程中调试错误的帮助，请参阅：
> 
> * [使用 Azure PowerShell 查看部署操作](/documentation/articles/resource-manager-troubleshoot-deployments-powershell/)，了解如何获取有助于排查错误的信息
> * [排查使用 Azure Resource Manager 将资源部署到 Azure 时的常见错误](/documentation/articles/resource-manager-common-deployment-errors/)，了解如何解决常见的部署错误
> 
> 

## 快速部署步骤
若要快速开始进行部署，请使用以下命令：

    New-AzureRmResourceGroup -Name ExampleResourceGroup -Location "China East"
    New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile c:\MyTemplates\example.json -TemplateParameterFile c:\MyTemplates\example.params.json

这些命令创建资源组，并将模板部署到该资源组。模板文件和参数文件都是本地文件。如果该操作成功，则一切准备就绪，可以部署资源了。不过，可以使用更多选项来指定要部署的资源。本文其余部分介绍了部署过程中可用的所有选项。

[AZURE.INCLUDE [resource-manager-deployments](../../includes/resource-manager-deployments.md)]

## 部署
1. 登录到你的 Azure 帐户。

        Add-AzureRmAccount -EnvironmentName AzureChinaCloud

     将返回你的帐户的摘要。

        Environment           : AzureChinaCloud
        Account               : someone@example.partner.onmschina.cn
        TenantId              : {guid}
        SubscriptionId        : {guid}
        SubscriptionName      : Example Subscription
        CurrentStorageAccount :

2. 如果有多个订阅，请使用 **Set-AzureRmContext** 命令提供要用于部署的订阅 ID。

        Set-AzureRmContext -SubscriptionID <YourSubscriptionId>

3. 部署模板时，必须指定将包含已部署资源的资源组。如果有要部署到的现有资源组，可以跳过此步骤，然后使用该资源组。
   
    若要创建资源组，请提供资源组的名称和位置。提供资源组的位置是因为资源组存储与资源有关的元数据。出于合规性原因，你可能会想要指定该元数据的存储位置。一般情况下，建议指定大部分资源将驻留的位置。使用相同位置可简化模板。

        New-AzureRmResourceGroup -Name ExampleResourceGroup -Location "China East"
   
    将返回新资源组的摘要。
   
            ResourceGroupName : ExampleResourceGroup
            Location          : chinaeast
            ProvisioningState : Succeeded
            Tags              :
            Permissions       :
                 Actions  NotActions
                 =======  ==========
                 *
            ResourceId        : /subscriptions/######/resourceGroups/ExampleResourceGroup

4. 在执行部署前，可以验证部署设置。使用 **Test-AzureRmResourceGroupDeployment** cmdlet 可以在创建实际资源之前找出问题。下面的示例演示如何验证部署。

       Test-AzureRmResourceGroupDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate>

5. 若要将资源部署到资源组，请运行 **New-AzureRmResourceGroupDeployment** 命令并提供所需的参数。参数包括部署的名称、资源组的名称、模板的路径或 URL，以及方案所需的任何其他参数。如果未指定 **Mode** 参数，将使用 **Incremental** 的默认值。若要运行完整部署，请将 **Mode** 设置为 **Complete**。使用完整模式时要小心，因为可能会无意中删除不在模板中的资源。
   
     若要部署本地模板，请使用 **TemplateFile** 参数：

       New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile c:\MyTemplates\example.json

    若要部署外部模板，请使用 **TemplateUri** 参数：

       New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateUri https://raw.githubusercontent.com/exampleuser/MyTemplates/master/example.json

    The preceding two examples do not include parameter values. You learn about the options for passing parameter values in the [Parameters](#parameters) section. For now, you are prompted to provide parameter values with the following syntax:

       cmdlet New-AzureRmResourceGroupDeployment at command pipeline position 1
       Supply values for the following parameters:
       (Type !? for Help.)
       firstParameter: <type here>

    After the resources have been deployed, you see a summary of the deployment. The summary includes a **ProvisioningState**, which indicates whether the deployment succeeded.

       DeploymentName    : ExampleDeployment
       ResourceGroupName : ExampleResourceGroup
       ProvisioningState : Succeeded
       Timestamp         : 4/14/2015 7:00:27 PM
       Mode              : Incremental

6. 如果要记录可能帮助你排查任何部署错误的有关部署的其他信息，请使用 **DeploymentDebugLogLevel** 参数。你可以指定在对部署操作进行日志记录时记录请求内容或/和响应内容。

       New-AzureRmResourceGroupDeployment -Name ExampleDeployment -DeploymentDebugLogLevel All -ResourceGroupName ExampleResourceGroup -TemplateFile <PathOrLinkToTemplate>

    有关使用此调试内容解决部署问题的详细信息，请参阅[使用 Azure PowerShell 对资源组部署进行故障排除](/documentation/articles/resource-manager-troubleshoot-deployments-powershell/)。

## 使用 SAS 令牌部署专用模板
可以将模板添加到存储帐户，并在部署过程中使用 SAS 令牌链接到这些模板。

> [AZURE.IMPORTANT]
通过执行以下步骤，只有帐户所有者可以访问包含模板的 blob。但是，如果为 blob 创建 SAS 令牌，则拥有该 URI 的任何人都可以访问 blob。如果其他用户截获了该 URI，则此用户可以访问该模板。使用 SAS 令牌是限制对模板的访问的好方法，但不应直接在模板中包括密码等敏感数据。
> 
> 

### 将专用模板添加到存储帐户
以下步骤用于为模板设置存储帐户：

1. 创建资源组。

       New-AzureRmResourceGroup -Name ManageGroup -Location "China East"

2. 创建存储帐户。存储帐户名称必须在 Azure 中唯一，因此，请为帐户提供自己的名称。

       New-AzureRmStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates -Type Standard_LRS -Location "China East"

3. 设置当前存储帐户。

       Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates

4. 创建容器。权限设置为“关闭”，这意味着只有所有者可以访问该容器。

       New-AzureStorageContainer -Name templates -Permission Off

5. 将模板添加到该容器。

       Set-AzureStorageBlobContent -Container templates -File c:\Azure\Templates\azuredeploy.json

### 在部署期间提供 SAS 令牌
若要在存储帐户中部署专用模板，请检索 SAS 令牌，并将其包括在模板的 URI 中。

1. 如果已更改当前存储帐户，请将当前存储帐户设置为包含你的模板的存储帐户。

       Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates

2. 创建具有读取权限和到期时间的 SAS 令牌来限制访问。检索包括 SAS 令牌的模板的完整 URI。

       $templateuri = New-AzureStorageBlobSASToken -Container templates -Blob azuredeploy.json -Permission r -ExpiryTime (Get-Date).AddHours(2.0) -FullUri

3. 通过提供包括 SAS 令牌的 URI 来部署该模板。

       New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleResourceGroup -TemplateUri $templateuri

有关将 SAS 令牌与链接模板配合使用的示例，请参阅[将已链接的模版与 Azure Resource Manager 配合使用](/documentation/articles/resource-group-linked-templates/)。

## <a name="parameters"></a> 参数

可以使用以下选项提供参数值：
   
- 内联参数。在 cmdlet 中包括各个参数名称（例如，**-myParameterName**。）

       New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate> -myParameterName "parameterValue"

- 参数对象。包括 **-TemplateParameterObject** 参数。

       $parameters = @{"<ParameterName>"="<Parameter Value>"}
       New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate> -TemplateParameterObject $parameters

- 本地参数文件。包括 **-TemplateParameterFile** 参数。

       New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate> -TemplateParameterFile c:\MyTemplates\example.params.json

- 外部参数文件。包括 **-TemplateParameterUri** 参数。

       New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateUri <LinkToTemplate> -TemplateParameterUri https://raw.githubusercontent.com/exampleuser/MyTemplates/master/example.params.json

<a name="parameter-file"></a>

[AZURE.INCLUDE [resource-manager-parameter-file](../../includes/resource-manager-parameter-file.md)]

如果你的模板包括的一个参数与 PowerShell 命令中的某个参数同名，系统会提示你提供该参数的值。Azure PowerShell 在提供模板中的参数时包含后缀 **FromTemplate**。例如，模板中名为 **ResourceGroupName** 的参数与 [New-AzureRmResourceGroupDeployment](https://docs.microsoft.com/powershell/resourcemanager/azurerm.resources/v3.3.0/new-azurermresourcegroupdeployment) cmdlet 中的 **ResourceGroupName** 参数冲突。系统会提示你提供 **ResourceGroupNameFromTemplate** 的值。通常，不应将参数命名为与用于部署操作的参数的名称相同以避免这种混乱。

## 参数优先级

可以在同一部署操作中使用内联参数和本地参数文件。例如，可以在本地参数文件中指定某些值，并在部署期间添加其他内联值。如果同时为本地参数文件中的参数和内联参数提供值，则内联值优先。

但是，使用外部参数文件时，不能传递是内联值的或本地文件中的其他值。如果在 **TemplateParameterUri** 参数中指定参数文件，则将忽略所有内联参数。提供外部文件中的所有参数值。如果模板包括参数文件中无法包括的敏感值，可将该值添加到密钥保管库，或者动态地以内联方式提供所有参数值。

## 后续步骤
* 有关通过 .NET 客户端库部署资源的示例，请参阅 [Deploy resources using .NET libraries and a template](/documentation/articles/virtual-machines-windows-csharp-template/)（使用 .NET 库和模板部署资源）。
* 若要在模板中定义参数，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/#parameters)。
* 有关将解决方案部署到不同环境的指南，请参阅 [Azure 中的开发和测试环境](/documentation/articles/solution-dev-test-environments/)。
* 如需了解企业如何使用 Resource Manager 对订阅进行有效管理，请参阅 [Azure 企业机架 - 规范性订阅管理](/documentation/articles/resource-manager-subscription-governance/)。
* 有关自动化部署的四部分系列教程，请参阅[将应用程序自动部署到 Azure 虚拟机](/documentation/articles/virtual-machines-windows-dotnet-core-1-landing/)。此系列教程介绍了应用程序体系结构、访问与安全性、可用性与缩放性，以及应用程序部署。

<!---HONumber=Mooncake_1219_2016-->