<properties
   pageTitle="使用 REST API 和模板部署资源 | Azure"
   description="使用 Azure Resource Manager 和 Resource Manager REST API 将资源部署到 Azure。资源在 Resource Manager 模板中定义。"
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
   ms.date="07/11/2016"
   wacn.date="11/21/2016"
   ms.author="tomfitz"/>


# 使用 Resource Manager 模板和 Resource Manager REST API 部署资源

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/resource-group-template-deploy/)
- [Azure CLI](/documentation/articles/resource-group-template-deploy-cli/)
- [门户](/documentation/articles/resource-group-template-deploy-portal/)
- [REST API](/documentation/articles/resource-group-template-deploy-rest/)

本文介绍如何将 Resource Manager REST API 与 Resource Manager 模板配合使用向 Azure 部署资源。

> [AZURE.TIP] 有关在部署过程中调试错误的帮助，请参阅：
>
> - [使用 REST API 查看部署操作](/documentation/articles/resource-manager-troubleshoot-deployments-rest/)，以了解如何获取有助于排查错误的信息
> - [排查使用 Azure Resource Manager 将资源部署到 Azure 时的常见错误](/documentation/articles/resource-manager-common-deployment-errors/)，了解如何解决常见的部署错误

你的模板可以是本地文件或是可通过 URI 访问的外部文件。如果模板驻留在存储帐户中，你可以限制对该模板的访问，并在部署过程中提供共享访问签名 (SAS) 令牌。

[AZURE.INCLUDE [resource-manager-deployments](../../includes/resource-manager-deployments.md)]

## 使用 REST API 进行部署
1. 设置[常见参数和标头](https://msdn.microsoft.com/zh-cn/library/azure/8d088ecc-26eb-42e9-8acc-fe929ed33563#bk_common)，包括身份验证令牌。
2. 如果目前没有资源组，请创建资源组。提供订阅 ID、新资源组的名称，以及解决方案所需的位置。有关详细信息，请参阅[创建资源组](https://msdn.microsoft.com/zh-cn/library/azure/dn790525.aspx)。

        PUT https://management.chinacloudapi.cn/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>?api-version=2015-01-01
          <common headers>
          {
            "location": "China East",
            "tags": {
               "tagname1": "tagvalue1"
            }
          }
   
3. 在执行部署之前，然后通过运行[验证模板部署](https://msdn.microsoft.com/zh-cn/library/azure/dn790547.aspx)操作来验证部署。测试部署时，请提供与执行部署时所提供的完全相同的参数（如下一步中所示）。

3. 新建部署。提供订阅 ID、要部署的资源组的名称、部署的名称以及模板的链接。有关模板文件的信息，请参阅[参数文件](#parameter-file)。有关使用 REST API 创建资源组的详细信息，请参阅[创建模板部署](https://msdn.microsoft.com/zh-cn/library/azure/dn790564.aspx)。请注意，“mode”设置为“Incremental”。若要运行完整部署，请将 **mode** 设置为 **Complete**。使用完整模式时要小心，因为可能会无意中删除不在模板中的资源。
    
        PUT https://management.chinacloudapi.cn/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2015-01-01
          <common headers>
          {
            "properties": {
              "templateLink": {
                "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
                "contentVersion": "1.0.0.0",
              },
              "mode": "Incremental",
              "parametersLink": {
                "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
                "contentVersion": "1.0.0.0",
              }
            }
          }
   
      如果想要记录响应内容或/和请求内容，请在请求中包括 **debugSetting**。

        "debugSetting": {
          "detailLevel": "requestContent, responseContent"
        }

      可以将存储帐户设置为使用共享访问签名 (SAS) 令牌。有关详细信息，请参阅[使用共享访问签名委托访问权限](https://msdn.microsoft.com/zh-cn/library/ee395415.aspx)。

4. 获取模板部署的状态。有关详细信息，请参阅[获取有关模板部署的信息](https://msdn.microsoft.com/zh-cn/library/azure/dn790565.aspx)。

          GET https://management.chinacloudapi.cn/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2015-01-01
           <common headers>

[AZURE.INCLUDE [resource-manager-parameter-file](../../includes/resource-manager-parameter-file.md)]

## 后续步骤
- 有关通过 .NET 客户端库部署资源的示例，请参阅[使用 .NET 库和模板部署资源](/documentation/articles/virtual-machines-windows-csharp-template/)。
- 若要在模板中定义参数，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates#parameters)。
- 有关将解决方案部署到不同环境的指南，请参阅 [Azure 中的开发和测试环境](/documentation/articles/solution-dev-test-environments/)。
- 有关使用 KeyVault 引用来传递安全值的详细信息，请参阅[在部署期间传递安全值](/documentation/articles/resource-manager-keyvault-parameter/)。

<!---HONumber=Mooncake_1114_2016-->