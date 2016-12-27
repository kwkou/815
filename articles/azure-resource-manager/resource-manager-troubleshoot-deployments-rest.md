<properties
    pageTitle="使用 REST API 查看部署操作 | Azure"
    description="介绍如何使用 Azure Resource Manager REST API 来检测 Resource Manager 部署的问题。"
    services="azure-resource-manager,virtual-machines"
    documentationcenter=""
    tags="top-support-issue"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />  

<tags
    ms.assetid="af2abbef-5454-45d3-b5cf-4ae10347e630"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-multiple"
    ms.workload="infrastructure"
    ms.date="06/13/2016"
    wacn.date="12/26/2016"
    ms.author="tomfitz" />  


# 使用 Azure Resource Manager REST API 查看部署操作
>[AZURE.SELECTOR]
- [门户](/documentation/articles/resource-manager-troubleshoot-deployments-portal/)
- [PowerShell](/documentation/articles/resource-manager-troubleshoot-deployments-powershell/)
- [Azure CLI](/documentation/articles/resource-manager-troubleshoot-deployments-cli/)
- [REST API](/documentation/articles/resource-manager-troubleshoot-deployments-rest/)

如果你在将资源部署到 Azure 时收到错误，可能想要查看有关已执行的部署操作的更多详细信息。REST API 提供操作来让你查找错误并确定可能的解决方法。

[AZURE.INCLUDE [resource-manager-troubleshoot-introduction](../../includes/resource-manager-troubleshoot-introduction.md)]

如果部署之前先验证模板和基础结构，则可以避免一些错误。你还可以在部署过程中记录其他请求和响应信息，以方便以后进行故障排除。若要了解有关验证和记录请求和响应信息的内容，请参阅 [Deploy a resource group with Azure Resource Manager template](/documentation/articles/resource-group-template-deploy-rest/)（使用 Azure Resource Manager 模板部署资源组）。

## 使用 REST API 进行故障排除
1. 使用[创建模板部署](https://docs.microsoft.com/rest/api/resources/deployments#Deployments_CreateOrUpdate)操作来部署资源。若要保留可能有助于调试的信息，请将 JSON 请求中的 **debugSetting** 属性设置为 **requestContent** 和/或 **responseContent**。
   
        PUT https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}
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
              },
              "debugSetting": {
                "detailLevel": "requestContent, responseContent"
              }
            }
          }
   
    默认情况下，**debugSetting** 值设置为 **none**。指定 **debugSetting** 值时，请仔细考虑要在部署期间传入的信息类型。通过记录有关请求或响应的信息，可能会公开通过部署操作检索的机密数据。
2. 使用[获取有关模板部署的信息](https://docs.microsoft.com/rest/api/resources/deployments#Deployments_Get)操作来获取有关部署的信息。
   
        GET https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}
   
    在响应中，请特别注意 **provisioningState**、**correlationId** 和 **error** 元素。**correlationId** 可用于跟踪相关事件，在与技术支持人员合作排查问题时非常有用。
   
        { 
          ...
          "properties": {
            "provisioningState":"Failed",
            "correlationId":"d5062e45-6e9f-4fd3-a0a0-6b2c56b15757",
            ...
            "error":{
              "code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see http://aka.ms/arm-debug for usage details.",
              "details":[{"code":"Conflict","message":"{\r\n  "error": {\r\n    "message": "Conflict",\r\n    "code": "Conflict"\r\n  }\r\n}"}]
            }  
          }
        }
3. 使用[列出所有模板部署操作](https://docs.microsoft.com/rest/api/resources/deployments#Deployments_List)操作来获取有关部署操作的信息。
   
        GET https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}/operations?$skiptoken={skiptoken}&api-version={api-version}
   
    响应根据部署期间在 **debugSetting** 属性中指定的内容，将包含请求和/或响应信息。
   
        {
          ...
          "properties": 
          {
            ...
            "request":{
              "content":{
                "location":"China North",
                "properties":{
                  "accountType": "Standard_LRS"
                }
              }
            },
            "response":{
              "content":{
                "error":{
                  "message":"Conflict","code":"Conflict"
                }
              }
            }
          }
        }
4. 使用[列出订阅中的管理事件](https://msdn.microsoft.com/zh-cn/library/azure/dn931934.aspx)操作，从审核日志中获取部署的事件。
   
        GET https://management.chinacloudapi.cn/subscriptions/{subscription-id}/providers/microsoft.insights/eventtypes/management/values?api-version={api-version}&$filter={filter-expression}&$select={comma-separated-property-names}

## 后续步骤
* 有关解决特定部署错误的帮助，请参阅 [Resolve common errors when deploying resources to Azure with Azure Resource Manager](/documentation/articles/resource-manager-common-deployment-errors/)（解决使用 Azure Resource Manager 将资源部署到 Azure 时的常见错误）。
* 若要了解如何使用审核日志来监视其他类型的操作，请参阅 [Audit operations with Resource Manager](/documentation/articles/resource-group-audit/)（使用 Resource Manager 执行审核操作）。
* 若要在执行部署之前验证部署，请参阅 [Deploy a resource group with Azure Resource Manager template](/documentation/articles/resource-group-template-deploy/)（使用 Azure Resource Manager 模板部署资源组）。

<!---HONumber=Mooncake_1219_2016-->