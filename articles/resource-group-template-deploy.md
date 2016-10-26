<!-- Remove Portal, Temp remove REST API -->
<properties
   pageTitle="使用 PowerShell 和模板部署资源 | Azure"
   description="使用 Azure Resource Manager 和 Azure PowerShell 将资源部署到 Azure。资源在 Resource Manager 模板中定义。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>  


<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="08/15/2016"
   wacn.date="10/24/2016"/>  


# 使用 Resource Manager 模板和 Azure PowerShell 部署资源

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/resource-group-template-deploy/)
- [Azure CLI](/documentation/articles/resource-group-template-deploy-cli/)
- [门户](/documentation/articles/resource-group-template-deploy-portal/)
- [REST API](/documentation/articles/resource-group-template-deploy-rest/)

本主题介绍如何将 Azure PowerShell 与 Resource Manager 模板配合使用向 Azure 部署资源。

> [AZURE.TIP] 有关在部署过程中调试错误的帮助，请参阅：
>
> - [使用 Azure PowerShell 查看部署操作](/documentation/articles/resource-manager-troubleshoot-deployments-powershell/)，了解如何获取有助于排查错误的信息
> - [排查使用 Azure Resource Manager 将资源部署到 Azure 时的常见错误](/documentation/articles/resource-manager-common-deployment-errors/)，了解如何解决常见的部署错误

你的模板可以是本地文件或是可通过 URI 访问的外部文件。如果模板驻留在存储帐户中，你可以限制对该模板的访问，并在部署过程中提供共享访问签名 (SAS) 令牌。

## 快速部署步骤

本文介绍了部署过程中可用的所有不同选项。但是，通常只需要两个简单的命令。若要快速开始进行部署，请使用以下命令：

    New-AzureRmResourceGroup -Name ExampleResourceGroup -Location "China East"
    New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate> -TemplateParameterFile <PathToParameterFile>

若要了解有关更适合于你的应用场景的部署选项的详细信息，请继续阅读本文。

[AZURE.INCLUDE [resource-manager-deployments](../includes/resource-manager-deployments.md)]

## 使用 PowerShell 进行部署

1. 登录到你的 Azure 帐户。

        Add-AzureRmAccount -EnvironmentName AzureChinaCloud

     将返回你的帐户的摘要。

        Environment : AzureCloud
        Account     : someone@example.com
        ...

2. 如果有多个订阅，请使用 **Set-AzureRmContext** 命令提供要用于部署的订阅 ID。

        Set-AzureRmContext -SubscriptionID <YourSubscriptionId>

3. 通常，在部署新模板时，需要创建资源组以包含资源。如果有要部署到的现有资源组，可以跳过此步骤，然后使用该资源组。

     若要创建资源组，请提供资源组的名称和位置。需要提供资源组的位置，因为资源组存储与资源有关的元数据。出于合规性原因，你可能会想要指定该元数据的存储位置。一般情况下，建议指定大部分资源将驻留的位置。使用相同位置可简化模板。

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

5. 若要将资源部署到资源组，请运行 **New-AzureRmResourceGroupDeployment** 命令并提供所需的参数。参数包括部署的名称、资源组的名称、创建的模板的路径或 URL，以及方案所需的任何其他参数。如果未指定 **Mode** 参数，则使用默认值 **Incremental**。若要运行完整部署，请将 **Mode** 设置为 **Complete**。使用完整模式时要小心，因为可能会无意中删除不在模板中的资源。

     若要部署本地模板，请使用 **TemplateFile** 参数：

        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate>

     若要部署外部模板，请使用 **TemplateUri** 参数：

        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateUri <LinkToTemplate>
   
     可以使用以下选项提供参数值：
   
     1. 使用内联参数。

            New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate> -myParameterName "parameterValue"

     2. 使用参数对象。

            $parameters = @{"<ParameterName>"="<Parameter Value>"}
            New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate> -TemplateParameterObject $parameters

     3. 使用本地参数文件。有关模板文件的信息，请参阅[参数文件](#parameter-file)。

            New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathToTemplate> -TemplateParameterFile <PathToParameterFile>

     4. 使用外部参数文件。有关模板文件的信息，请参阅[参数文件](#parameter-file)。

            New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateUri <LinkToTemplate> -TemplateParameterUri <LinkToParameterFile>

        使用外部参数文件时，不能传递是内联值的或本地文件中的其他值。有关详细信息，请参阅[参数优先级](#parameter-precendence)。

     部署资源组后，你将看到部署摘要。

        DeploymentName    : ExampleDeployment
        ResourceGroupName : ExampleResourceGroup
        ProvisioningState : Succeeded
        Timestamp         : 4/14/2015 7:00:27 PM
        Mode              : Incremental
        ...

     如果你的模板包括的一个参数与 PowerShell 命令中的某个参数同名，系统会提示你提供该参数的值。模板中的参数会包含后缀 **FromTemplate**。例如，模板中名为 **ResourceGroupName** 的参数与 [New-AzureRmResourceGroupDeployment](https://msdn.microsoft.com/zh-cn/library/azure/mt679003.aspx) cmdlet 中的 **ResourceGroupName** 参数冲突。系统会提示你提供 **ResourceGroupNameFromTemplate** 的值。通常，不应将参数命名为与用于部署操作的参数的名称相同以避免这种混乱。

6. 如果要记录可能帮助你排查任何部署错误的有关部署的其他信息，请使用 **DeploymentDebugLogLevel** 参数。你可以指定在对部署操作进行日志记录时记录请求内容或/和响应内容。

        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -DeploymentDebugLogLevel All -ResourceGroupName ExampleResourceGroup -TemplateFile <PathOrLinkToTemplate>
        
     有关使用此调试内容解决部署问题的详细信息，请参阅[使用 Azure PowerShell 对资源组部署进行故障排除](/documentation/articles/resource-manager-troubleshoot-deployments-powershell/)。

## 使用 SAS 令牌从存储空间部署模板

可以将模板添加到存储帐户，并在部署过程中使用 SAS 令牌链接到这些模板。

> [AZURE.IMPORTANT] 通过执行以下步骤，只有帐户所有者可以访问包含模板的 blob。但是，如果为 blob 创建 SAS 令牌，则拥有该 URI 的任何人都可以访问 blob。如果其他用户截获了该 URI，则此用户可以访问该模板。使用 SAS 令牌是限制对模板的访问的好方法，但不应直接在模板中包括密码等敏感数据。

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

[AZURE.INCLUDE [resource-manager-parameter-file](../includes/resource-manager-parameter-file)]

## 参数优先级

可以在同一部署操作中使用内联参数和本地参数文件。例如，可以在本地参数文件中指定某些值，并在部署期间添加其他内联值。如果同时为本地参数文件中的参数和内联参数提供值，则内联值优先。

但是，不能将内联参数用于外部参数文件。如果在 **TemplateParameterUri** 参数中指定参数文件，则将忽略所有内联参数。必须提供外部文件中的所有参数值。如果模板包括参数文件中无法包括的敏感值，可将该值添加到密钥保管库，然后在外部参数文件中引用该密钥保管库，或者自动以内联方式提供所有参数值。

有关使用 KeyVault 引用来传递安全值的详细信息，请参阅[在部署期间传递安全值](/documentation/articles/resource-manager-keyvault-parameter/)。

## 后续步骤
- 有关通过 .NET 客户端库部署资源的示例，请参阅[使用 .NET 库和模板部署资源](/documentation/articles/virtual-machines-windows-csharp-template/)。
- 若要在模板中定义参数，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/#parameters)。
- 有关将解决方案部署到不同环境的指南，请参阅 [Microsoft Azure 中的开发和测试环境](/documentation/articles/solution-dev-test-environments/)。

<!---HONumber=Mooncake_1017_2016-->