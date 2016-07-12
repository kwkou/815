<properties
   pageTitle="使用 REST API 对部署进行故障排除 | Azure"
   description="介绍如何使用 Azure Resource Manager REST API 来检测和解决资源管理器部署的问题。"
   services="azure-resource-manager,virtual-machines"
   documentationCenter=""
   tags="top-support-issue"
   authors="tfitzmac"
   manager="timlt"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="03/21/2016"
   wacn.date="05/05/2016"/>

# 使用 Azure Resource Manager REST API 对资源组部署进行故障排除

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/resource-manager-troubleshoot-deployments-powershell/)
- [REST API](/documentation/articles/resource-manager-troubleshoot-deployments-rest/)

如果在将资源部署到 Azure 时发生错误，你需要进行故障排除。REST API 提供操作来让你查找错误并确定可能的解决方法。

可以通过查看审核日志或部署操作来对部署进行故障排除。本主题将演示这两种方法。

如果部署之前先验证模板和基础结构，则可以避免一些错误。有关详细信息，请参阅 [Deploy a resource group with Azure Resource Manager template（使用 Azure Resource Manager 模板部署资源组）](/documentation/articles/resource-group-template-deploy/)。

## 使用 REST API 进行故障排除

1. 使用[创建模板部署](https://msdn.microsoft.com/zh-cn/library/azure/dn790564.aspx)操作来部署资源。若要保留可能有助于调试的信息，请将 JSON 请求中的 **debugSetting** 属性设置为 **requestContent** 和/或 **responseContent**。 

        PUT https://manage.windowsazure.cn/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}
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

    默认情况下，**debugSetting** 值设置为 **none**。指定 **debugSetting** 值时，请仔细考虑在部署期间传递的信息类型。通过记录有关请求或响应的信息，可能会公开通过部署操作检索的机密数据。

2. 使用[获取有关模板部署的信息](https://msdn.microsoft.com/zh-cn/library/azure/dn790565.aspx)操作来获取有关部署的信息。

        GET https://manage.windowsazure.cn/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}

    在响应中，请特别注意 **provisioningState**、**correlationId** 和 **error** 元素。**correlationId** 可用于跟踪相关的事件，在与技术支持人员合作排查问题时非常有用。
    
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

3. 使用[列出所有模板部署操作](https://msdn.microsoft.com/zh-cn/library/azure/dn790518.aspx)操作来获取有关部署操作的信息。

        GET https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}/operations?$skiptoken={skiptoken}&api-version={api-version}

    响应根据部署期间在 **debugSetting** 属性中指定的内容包含请求和/或响应信息。
    
        {
          ...
          "properties": 
          {
            ...
            "request":{
              "content":{
                "location":"China East",
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

        GET https://manage.chinacloudapi.cn/subscriptions/{subscription-id}/providers/microsoft.insights/eventtypes/management/values?api-version={api-version}&$filter={filter-expression}&$select={comma-separated-property-names}


## 后续步骤

- 若要了解如何使用审核日志来监视其他类型的操作，请参阅 [使用 Resource Manager 执行审核操作](/documentation/articles/resource-group-audit/)。
- 若要在执行部署之前验证部署，请参阅 [使用 Azure Resource Manager 模板部署资源组](/documentation/articles/resource-group-template-deploy/)。

<!---HONumber=Mooncake_0425_2016-->