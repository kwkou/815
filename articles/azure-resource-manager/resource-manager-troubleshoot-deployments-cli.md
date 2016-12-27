<properties
    pageTitle="使用 Azure CLI 查看部署操作 | Azure"
    description="介绍如何使用 Azure CLI 来检测 Resource Manager 部署的问题。"
    services="azure-resource-manager,virtual-machines"
    documentationcenter=""
    tags="top-support-issue"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />  

<tags
    ms.assetid="3001a585-ccf8-4534-a21d-aab282288243"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-multiple"
    ms.workload="infrastructure"
    ms.date="08/15/2016"
    wacn.date="12/26/2016"
    ms.author="tomfitz" />

# 使用 Azure CLI 查看部署操作
>[AZURE.SELECTOR]
[Portal](/documentation/articles/resource-manager-troubleshoot-deployments-portal/)
[PowerShell](/documentation/articles/resource-manager-troubleshoot-deployments-powershell/)
[Azure CLI](/documentation/articles/resource-manager-troubleshoot-deployments-cli/)
[REST API](/documentation/articles/resource-manager-troubleshoot-deployments-rest/)

如果你在将资源部署到 Azure 时收到错误，可能想要查看有关已执行的部署操作的更多详细信息。Azure CLI 提供命令来让你查找错误并确定可能的解决方法。

[AZURE.INCLUDE [resource-manager-troubleshoot-introduction](../../includes/resource-manager-troubleshoot-introduction.md)]

如果部署之前先验证模板和基础结构，则可以避免一些错误。你还可以在部署过程中记录其他请求和响应信息，以方便以后进行故障排除。若要了解有关验证和记录请求和响应信息的内容，请参阅 [Deploy a resource group with Azure Resource Manager template](/documentation/articles/resource-group-template-deploy-cli/)（使用 Azure Resource Manager 模板部署资源组）。

## 使用审核日志进行故障排除
[AZURE.INCLUDE [resource-manager-audit-limitations](../../includes/resource-manager-audit-limitations.md)]

若要查看部署相关的错误，请使用以下步骤：

1. 若要查看审计日志，请运行 **azure group log show** 命令。可以包含 **--last-deployment** 选项以只获取最近部署的日志。
   
        azure group log show ExampleGroup --last-deployment
2. **azure group log show** 命令返回很多信息。进行故障排除，你通常希望专注于失败的操作。以下脚本使用 **--json** 选项和 [jq](https://stedolan.github.io/jq/) JSON 实用工具，在日志中搜索部署失败。
   
        azure group log show ExampleGroup --json | jq '.[] | select(.status.value == "Failed")'
   
        {
        "claims": {
          "aud": "https://management.core.chinacloudapi.cn/",
          "iss": "https://sts.chinacloudapi.cn/<guid>/",
          "iat": "1442510510",
          "nbf": "1442510510",
          "exp": "1442514410",
          "ver": "1.0",
          "http://schemas.microsoft.com/identity/claims/tenantid": "{guid}",
          "http://schemas.microsoft.com/identity/claims/objectidentifier": "{guid}",
          "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress": "someone@example.com",
          "puid": "XXXXXXXXXXXXXXXX",
          "http://schemas.microsoft.com/identity/claims/identityprovider": "example.com",
          "altsecid": "1:example.com:XXXXXXXXXXX",
          "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "<hash string="">",
          "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "Tom",
          "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "FitzMacken",
          "name": "Tom FitzMacken",
          "http://schemas.microsoft.com/claims/authnmethodsreferences": "pwd",
          "groups": "{guid}",
          "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "example.com#someone@example.com",
          "wids": "{guid}",
          "appid": "{guid}",
          "appidacr": "0",
          "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
          "http://schemas.microsoft.com/claims/authnclassreference": "1",
          "ipaddr": "000.000.000.000"
        },
        "properties": {
          "statusCode": "Conflict",
          "statusMessage": "{"Code":"Conflict","Message":"Website with given name mysite already exists.","Target":null,"Details":[{"Message":"Website with given name
            mysite already exists."},{"Code":"Conflict"},{"ErrorEntity":{"Code":"Conflict","Message":"Website with given name mysite already exists.","ExtendedCode":
            "54001","MessageTemplate":"Website with given name {0} already exists.","Parameters":["mysite"],"InnerErrors":null}}],"Innererror":null}"
        },
        ...
   
    你可以看到 **properties** 在 json 中包含有关失败操作的信息。
   
    可以使用 **--verbose** 和 **-vv** 选项，在日志中查看更多信息。使用 **--verbose** 选项显示操作在 `stdout` 上进行的步骤。要查看完整的请求历史记录，请使用 **-vv** 选项。这些消息经常提供有关任何失败原因的重要线索。
3. 若要专注于失败条目的状态消息，请使用以下命令：
   
        azure group log show ExampleGroup --json | jq -r ".[] | select(.status.value == "Failed") | .properties.statusMessage"

## 使用部署操作进行故障排除
1. 使用 **azure group deployment show** 命令获取部署的总体状态。在下面的示例中，部署已失败。
   
        azure group deployment show --resource-group ExampleGroup --name ExampleDeployment
   
        info:    Executing command group deployment show
        + Getting deployments
        data:    DeploymentName     : ExampleDeployment
        data:    ResourceGroupName  : ExampleGroup
        data:    ProvisioningState  : Failed
        data:    Timestamp          : 2015-08-27T20:03:34.9178576Z
        data:    Mode               : Incremental
        data:    Name             Type    Value
        data:    ---------------  ------  ------------
        data:    siteName         String  ExampleSite
        data:    hostingPlanName  String  ExamplePlan
        data:    siteLocation     String  China North
        data:    sku              String  Free
        data:    workerSize       String  0
        info:    group deployment show command OK
2. 若要查看部署失败操作的消息，请使用：
   
        azure group deployment operation list --resource-group ExampleGroup --name ExampleDeployment --json  | jq ".[] | select(.properties.provisioningState == "Failed") | .properties.statusMessage.Message"

## 后续步骤
* 有关解决特定部署错误的帮助，请参阅 [Resolve common errors when deploying resources to Azure with Azure Resource Manager](/documentation/articles/resource-manager-common-deployment-errors/)（解决使用 Azure Resource Manager 将资源部署到 Azure 时的常见错误）。
* 若要了解如何使用审核日志来监视其他类型的操作，请参阅 [Audit operations with Resource Manager](/documentation/articles/resource-group-audit/)（使用 Resource Manager 执行审核操作）。
* 若要在执行部署之前验证部署，请参阅[使用 Azure Resource Manager 模板部署资源组](/documentation/articles/resource-group-template-deploy/)。

<!---HONumber=Mooncake_1219_2016-->