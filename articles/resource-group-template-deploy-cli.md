<properties
   pageTitle="使用 Azure CLI 和模板部署资源 | Azure"
   description="使用 Azure Resource Manager 和 Azure CLI 将资源部署到 Azure。资源在 Resource Manager 模板中定义。"
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
   wacn.date="11/21/2016"
   ms.author="tomfitz"/>  


# 使用 Resource Manager 模板和 Azure CLI 部署资源

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/resource-group-template-deploy/)
- [Azure CLI](/documentation/articles/resource-group-template-deploy-cli/)
- [门户](/documentation/articles/resource-group-template-deploy-portal/)
- [REST API](/documentation/articles/resource-group-template-deploy-rest/)

本主题介绍如何将 Azure CLI 与 Resource Manager 模板配合使用向 Azure 部署资源。

> [AZURE.TIP] 有关在部署过程中调试错误的帮助，请参阅：
>
> - [使用 Azure CLI 查看部署操作](/documentation/articles/resource-manager-troubleshoot-deployments-cli/)，了解如何获取有助于排查错误的信息
> - [排查使用 Azure Resource Manager 将资源部署到 Azure 时的常见错误](/documentation/articles/resource-manager-common-deployment-errors/)，了解如何解决常见的部署错误

你的模板可以是本地文件或是可通过 URI 访问的外部文件。如果模板驻留在存储帐户中，你可以限制对该模板的访问，并在部署过程中提供共享访问签名 (SAS) 令牌。

## 快速部署步骤

本文介绍了部署过程中可用的所有不同选项。但是，通常只需要两个简单的命令。若要快速开始进行部署，请使用以下命令：

    azure group create -n ExampleResourceGroup -l "China East"
    azure group deployment create -f <PathToTemplate> -e <PathToParameterFile> -g ExampleResourceGroup -n ExampleDeployment

若要了解有关更适合于你的应用场景的部署选项的详细信息，请继续阅读本文。

[AZURE.INCLUDE [resource-manager-deployments](../includes/resource-manager-deployments.md)]

## 使用 Azure CLI 进行部署

如果你以前没有对资源管理器使用过 Azure CLI，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理配合使用](/documentation/articles/xplat-cli-azure-resource-manager/)。

1. 登录到你的 Azure 帐户。提供凭据后，该命令将返回你的登录结果。

        azure login -e AzureChinaCloud
  
        ...
        info:    login command OK

2. 如果你有多个订阅，请提供要用于部署的订阅 ID。

        azure account set <YourSubscriptionNameOrId>

3. 切换到 Azure 资源管理器模块。将收到新模式确认。

        azure config mode arm
   
        info:     New mode is arm

4. 如果目前没有资源组，请创建资源组。提供资源组的名称，以及解决方案所需的位置。需要提供资源组的位置，因为资源组存储与资源有关的元数据。出于合规性原因，你可能会想要指定该元数据的存储位置。一般情况下，建议指定大部分资源将驻留的位置。使用相同位置可简化模板。

        azure group create -n ExampleResourceGroup -l "China East"

     将返回新资源组的摘要。
   
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

5. 在执行部署之前先运行 **azure group template validate** 命令验证部署。测试部署时，请提供与执行部署时所提供的完全相同的参数（如下一步中所示）。

        azure group template validate -f <PathToTemplate> -p "{"ParameterName":{"value":"ParameterValue"}}" -g ExampleResourceGroup

5. 若要为资源组部署资源，请运行以下命令并提供所需的参数。参数包括部署的名称、资源组的名称、模板的路径或 URL，以及方案所需的任何其他参数。
   
     可以使用以下三个选项提供参数值：

     1. 使用内联参数和本地模板。每个参数采用以下格式：`"ParameterName": { "value": "ParameterValue" }`。以下示例显示带转义符的参数。

            azure group deployment create -f <PathToTemplate> -p "{"ParameterName":{"value":"ParameterValue"}}" -g ExampleResourceGroup -n ExampleDeployment

     2. 使用内联参数和模板链接。

            azure group deployment create --template-uri <LinkToTemplate> -p "{"ParameterName":{"value":"ParameterValue"}}" -g ExampleResourceGroup -n ExampleDeployment

     3. 使用参数文件。有关模板文件的信息。
    
            azure group deployment create -f <PathToTemplate> -e <PathToParameterFile> -g ExampleResourceGroup -n ExampleDeployment

     通过上述三种方法之一部署资源后，你将看到部署的摘要。
  
        info:    Executing command group deployment create
        + Initializing template configurations and parameters
        + Creating a deployment
        ...
        info:    group deployment create command OK

     若要运行完整部署，请将 **mode** 设置为 **Complete**。使用此模式时要小心，因为可能会无意中删除不在模板中的资源。

        azure group deployment create --mode Complete -f <PathToTemplate> -e <PathToParameterFile> -g ExampleResourceGroup -n ExampleDeployment

6. 如果要记录可能帮助你排查任何部署错误的有关部署的其他信息，请使用 **debug-setting** 参数。你可以指定在对部署操作进行日志记录时记录请求内容或/和响应内容。

        azure group deployment create --debug-setting All -f <PathToTemplate> -e <PathToParameterFile> -g ExampleResourceGroup -n ExampleDeployment

## 使用 SAS 令牌从存储空间部署模板

可以将模板添加到存储帐户，并在部署过程中使用 SAS 令牌链接到这些模板。

> [AZURE.IMPORTANT] 通过执行以下步骤，只有帐户所有者可以访问包含模板的 blob。但是，如果为 blob 创建 SAS 令牌，则拥有该 URI 的任何人都可以访问 blob。如果其他用户截获了该 URI，则此用户可以访问该模板。使用 SAS 令牌是限制对模板的访问的好方法，但不应直接在模板中包括密码等敏感数据。

### 将专用模板添加到存储帐户

以下步骤用于为模板设置存储帐户：

1. 创建资源组。

        azure group create -n "ManageGroup" -l "chinaeast"

2. 创建存储帐户。存储帐户名称必须在 Azure 中唯一，因此，请为帐户提供自己的名称。

        azure storage account create -g ManageGroup -l "westus" --sku-name LRS --kind Storage storagecontosotemplates

3. 为存储帐户和密钥设置变量。

        export AZURE_STORAGE_ACCOUNT=storagecontosotemplates
        export AZURE_STORAGE_ACCESS_KEY={storage_account_key}

4. 创建容器。权限设置为“关闭”，这意味着只有所有者可以访问该容器。

        azure storage container create --container templates -p Off 
        
4. 将模板添加到该容器。

        azure storage blob upload --container templates -f c:\Azure\Templates\azuredeploy.json
        
### 在部署期间提供 SAS 令牌

若要在存储帐户中部署专用模板，请检索 SAS 令牌，并将其包括在模板的 URI 中。

1. 创建具有读取权限和到期时间的 SAS 令牌来限制访问。设置到期时间以允许足够的时间来完成部署。检索包括 SAS 令牌的模板的完整 URI。

        expiretime=$(date -I'minutes' --date "+30 minutes")
        fullurl=$(azure storage blob sas create --container templates --blob azuredeploy.json --permissions r --expiry $expiretimetime --json  | jq ".url")

2. 通过提供包括 SAS 令牌的 URI 来部署该模板。

        azure group deployment create --template-uri $fullurl -g ExampleResourceGroup

有关将 SAS 令牌与链接模板配合使用的示例，请参阅[将已链接的模版与 Azure Resource Manager 配合使用](/documentation/articles/resource-group-linked-templates/)。

[AZURE.INCLUDE [resource-manager-parameter-file](../includes/resource-manager-parameter-file)]

## 后续步骤
- 有关通过 .NET 客户端库部署资源的示例，请参阅[使用 .NET 库和模板部署资源](/documentation/articles/virtual-machines-windows-csharp-template/)。
- 若要在模板中定义参数，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates#parameters)。
- 有关将解决方案部署到不同环境的指南，请参阅 [Azure 中的开发和测试环境](/documentation/articles/solution-dev-test-environments/)。
- 有关使用 KeyVault 引用来传递安全值的详细信息，请参阅[在部署期间传递安全值](/documentation/articles/resource-manager-keyvault-parameter/)。

<!---HONumber=Mooncake_1114_2016-->