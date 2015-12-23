<properties
   pageTitle="使用资源管理器模板部署资源 | Windows Azure"
   services="azure-resource-manager"
   description="使用 Azure 资源管理器将资源部署到 Azure。模板是一个 JSON 文件，可以从门户、PowerShell、适用于 Mac、Linux 和 Windows 的 Azure 命令行界面以及 REST 使用。"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="11/13/2015"
   wacn.date="12/17/2015"/>

# 使用 Azure 资源管理器模板部署应用程序

本主题介绍如何使用 Azure 资源管理器模板向 Azure 部署应用程序。它将说明如何使用 Azure PowerShell、Azure CLI、REST API 或 Windows Azure 门户部署应用程序。

有关资源管理器的简介，请参阅 [Azure 资源管理器概述](/documentation/articles/resource-group-overview)。若要了解有关创建模板的信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)。

在使用模板部署应用程序时，可以提供参数值来自定义如何创建资源。以内联方式或者在参数文件中指定这些参数的值。


## 使用 PowerShell 进行部署

[AZURE.INCLUDE [powershell-preview-inline-include](../includes/powershell-preview-inline-include.md)]


1. 登录到你的 Azure 帐户。提供凭据后，该命令将返回有关你的帐户的信息。

    早于 Azure PowerShell 1.0 预览版：

        PS C:\> Switch-AzureMode AzureResourceManager
        ...
        PS C:\> Add-AzureAccount

        Id                             Type       ...
        --                             ----    
        someone@example.com            User       ...   

    Azure PowerShell 1.0 预览版：

         PS C:\> Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)

         Evironment : AzureCloud
         Account    : someone@example.com
         ...


2. 如果你有多个订阅，请使用 **Select-AzureRmSubscription** 命令提供要用于部署的订阅 ID。

        PS C:\> Select-AzureRmSubscription -SubscriptionID <YourSubscriptionId>

3. 如果目前没有资源组，请使用 **New-AzureRmResourceGroup** 命令创建新的资源组。提供资源组的名称，以及解决方案所需的位置。将返回新资源组的摘要。

        PS C:\> New-AzureRmResourceGroup -Name ExampleResourceGroup -Location "China East"
   
        ResourceGroupName : ExampleResourceGroup
        Location          : westus
        ProvisioningState : Succeeded
        Tags              :
        Permissions       :
                    Actions  NotActions
                    =======  ==========
                    *
        ResourceId        : /subscriptions/######/resourceGroups/ExampleResourceGroup

5. 若要为资源组创建新部署，请运行 **New-AzureRmResourceGroupDeployment** 命令并提供所需的参数。参数包括部署的名称、资源组的名称、所创建模板的路径，以及方案所需的任何其他参数。
   
     可以使用以下选项提供参数值：
   
     - 使用内联参数。

            PS C:\> New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathOrLinkToTemplate> -myParameterName "parameterValue"

     - 使用参数对象。

            PS C:\> $parameters = @{"<ParameterName>"="<Parameter Value>"}
            PS C:\> New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathOrLinkToTemplate> -TemplateParameterObject $parameters

     - 使用参数文件。有关模板文件的信息，请参阅[参数文件](./#parameter-file)。

            PS C:\> New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile <PathOrLinkToTemplate> -TemplateParameterFile <PathOrLinkToParameterFile>

     部署资源组后，你将看到部署摘要。

          DeploymentName    : ExampleDeployment
          ResourceGroupName : ExampleResourceGroup
          ProvisioningState : Succeeded
          Timestamp         : 4/14/2015 7:00:27 PM
          Mode              : Incremental
          ...

6. 获取有关部署错误的信息。

        PS C:\> Get-AzureRmResourceGroupDeployment -ResourceGroupName ExampleResourceGroup -Name ExampleDeployment

## 使用适用于 Mac、Linux 和 Windows 的 Azure CLI 进行部署

如果你以前没有对资源管理器使用过 Azure CLI，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理配合使用](/documentation/articles/xplat-cli-azure-resource-manager)。

1. 登录到你的 Azure 帐户。提供凭据后，该命令将返回你的登录结果。

        azure login -e AzureCinaCloud -u <username> -p <password>
  
        ...
        info:    login command OK

2. 如果你有多个订阅，请提供要用于部署的订阅 ID。

        azure account set <YourSubscriptionNameOrId>

3. 切换到 Azure 资源管理器模块。你将收到新模式确认。

        azure config mode arm
   
        info:     New mode is arm

4. 如果目前没有资源组，请创建新的资源组。提供资源组的名称，以及解决方案所需的位置。将返回新资源组的摘要。

        azure group create -n ExampleResourceGroup -l "China East"
   
        info:    Executing command group create
        + Getting resource group ExampleResourceGroup
        + Creating resource group ExampleResourceGroup
        info:    Created resource group ExampleResourceGroup
        data:    Id:                  /subscriptions/####/resourceGroups/ExampleResourceGroup
        data:    Name:                ExampleResourceGroup
        data:    Location:            westus
        data:    Provisioning State:  Succeeded
        data:    Tags:
        data:
        info:    group create command OK

5. 若要为资源组创建新部署，请运行以下命令并提供所需的参数。参数包括部署的名称、资源组的名称、所创建模板的路径，以及方案所需的任何其他参数。
   
     可以使用以下选项提供参数值：

     - 使用内联参数和本地模板。每个参数采用以下格式：`"ParameterName": { "value": "ParameterValue" }`。以下示例显示带转义符的参数。

             azure group deployment create -f <PathToTemplate> -p "{"ParameterName":{"value":"ParameterValue"}}" -g ExampleResourceGroup -n ExampleDeployment

     - 使用内联参数和模板链接。

             azure group deployment create --template-uri <LinkToTemplate> -p "{"ParameterName":{"value":"ParameterValue"}}" -g ExampleResourceGroup -n ExampleDeployment

     - 使用参数文件。有关模板文件的信息，请参阅[参数文件](#parameter-file)。
    
             azure group deployment create -f <PathToTemplate> -e <PathToParameterFile> -g ExampleResourceGroup -n ExampleDeployment

     部署资源组后，你将看到部署摘要。
  
           info:    Executing command group deployment create
           + Initializing template configurations and parameters
           + Creating a deployment
           ...
           info:    group deployment create command OK


6. 获取有关最新部署的信息。

         azure group log show -l ExampleResourceGroup

7. 获取有关部署错误的详细信息。
      
         azure group log show -l -v ExampleResourceGroup

## 使用 REST API 进行部署
1. 设置[常见参数和标头](https://msdn.microsoft.com/zh-cn/library/azure/8d088ecc-26eb-42e9-8acc-fe929ed33563#bk_common)，包括身份验证令牌。
2. 如果目前没有资源组，请创建新的资源组。提供订阅 ID、新资源组的名称，以及解决方案所需的位置。有关详细信息，请参阅[创建资源组](https://msdn.microsoft.com/zh-cn/library/azure/dn790525.aspx)。

         PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>?api-version=2015-01-01
           <common headers>
           {
             "location": "China East",
             "tags": {
               "tagname1": "tagvalue1"
             }
           }
   
3. 创建新的资源组部署。提供你的订阅 ID、要部署的资源组的名称、部署的名称以及模板的位置。有关模板文件的信息，请参阅[参数文件](#parameter-file)。有关使用 REST API 创建资源组的详细信息，请参阅[创建模板部署](https://msdn.microsoft.com/zh-cn/library/azure/dn790564.aspx)。
    
         PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2015-01-01
            <common headers>
            {
              "properties": {
                "templateLink": {
                  "uri": "http://mystorageaccount.blob.core.chinacloudapi.cn/templates/template.json",
                  "contentVersion": "1.0.0.0",
                },
                "mode": "Incremental",
                "parametersLink": {
                  "uri": "http://mystorageaccount.blob.core.chinacloudapi.cn/templates/parameters.json",
                  "contentVersion": "1.0.0.0",
                }
              }
            }
   
4. 获取模板部署的状态。有关详细信息，请参阅[获取有关模板部署的信息](https://msdn.microsoft.com/zh-cn/library/azure/dn790565.aspx)。

         GET https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2015-01-01
           <common headers>

## 使用 Visual Studio 进行部署

使用 Visual Studio，可以创建资源组项目，并通过用户界面将其部署到 Azure。可选择要在项目中包括的资源类型，这些资源将自动添加到资源管理器模板中。该项目还提供了用于部署模板的 PowerShell 脚本。

有关将 Visual Studio 用于资源组的简介，请参阅[通过 Visual Studio 创建和部署 Azure 资源组](/documentation/articles/vs-azure-tools-resource-groups-deployment-projects-create-deploy)


## 参数文件

如果要在部署期间使用参数文件将参数值传递给模板，你需要使用类似于下例的格式创建一个 JSON 文件。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "webSiteName": {
                "value": "ExampleSite"
            },
            "webSiteHostingPlanName": {
                "value": "DefaultPlan"
            },
            "webSiteLocation": {
                "value": "China East"
            }
       }
    }

参数文件的大小不能超过 64 KB。

## 后续步骤
- 有关通过 .NET 客户端库部署资源的示例，请参阅[使用 .NET 库和模板部署资源](/documentation/articles/arm-template-deployment)
- 有关进一步了解部署应用程序的示例，请参阅[按可预见的方式在 Azure 中预配和部署微服务](/documentation/articles/app-service-deploy-complex-application-predictably)
- 若要了解 Azure 资源管理器模板的节，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates)。
- 有关可在 Azure 资源管理器模板中使用的函数列表，请参阅[模板函数](/documentation/articles/resource-group-template-functions)。

 

<!---HONumber=Mooncake_1207_2015-->