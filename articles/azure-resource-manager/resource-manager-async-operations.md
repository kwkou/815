<properties
    pageTitle="Azure 异步操作 | Azure"
    description="介绍如何在 Azure 中跟踪异步操作。"
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
    ms.date="01/11/2017"
    wacn.date="01/25/2017"
    ms.author="tomfitz" />  


# 跟踪异步 Azure 操作
某些 Azure REST 操作以异步方式运行，因为操作无法快速完成。本主题介绍如何通过响应中返回的值跟踪异步操作的状态。

## 异步操作的状态代码
最初，异步操作返回下面的一个 HTTP 状态代码：

* 201 (已创建)
* 202 (已接受)

该操作在成功完成时返回下面的一个代码：

* 200 (正常)
* 204 (无内容)

请参阅 [REST API 文档](https://docs.microsoft.com/rest/api/)，了解要执行的操作的响应。

## 监视操作状态
异步 REST 操作返回标头值，用于确定操作的状态。可能有三个需要检查的标头值：

* `Azure-AsyncOperation` - URL，用于检查操作目前的状态。如果操作返回此值，则应始终使用它（而不是 Location）来跟踪操作的状态。
* `Location` - URL，用于确定操作的完成时间。只有在未返回 Azure-AsyncOperation 时，才使用此值。
* `Retry-After` - 在检查异步操作的状态之前等待的秒数。

但是，并非每个异步操作都会返回所有这些值。例如，可能需要对一个操作计算 Azure-AsyncOperation 标头值，对另一个操作计算 Location 标头值。

检索这些标头值与检索请求的任何标头值一样。例如，在 C# 中，可以使用以下代码从名为 `response` 的 `HttpWebResponse` 对象检索标头值：

    response.Headers.GetValues("Azure-AsyncOperation").GetValue(0)

## Azure-AsyncOperation 请求和响应

若要获取异步操作的状态，请向 Azure-AsyncOperation 标头值中的 URL 发送 GET 请求。

来自此操作的响应的正文包含有关操作的信息。以下示例显示从操作中返回的可能值：

    {
        "id": "{resource path from GET operation}",
        "name": "{operation-id}", 
        "status" : "Succeeded | Failed | Canceled | {resource provider values}", 
        "startTime": "2017-01-06T20:56:36.002812+00:00",
        "endTime": "2017-01-06T20:56:56.002812+00:00",
        "percentComplete": {double between 0 and 100 },
        "properties": {
            /* Specific resource provider values for successful operations */
        },
        "error" : { 
            "code": "{error code}",  
            "message": "{error description}" 
        }
    }

所有响应都只返回 `status`。当状态为“已失败”或“已取消”时，返回错误对象。所有其他值都是可选的；因此，收到的响应看起来可能不同于示例。

## provisioningState 值

用于创建、更新或删除（PUT、PATCH、DELETE）资源的操作通常返回 `provisioningState` 值。完成操作后，将返回以下三个值之一：

* 已成功
* 已失败
* 已取消

所有其他值表示该操作仍在运行。资源提供程序可以返回自定义的值，用于指示其状态。例如，当请求已收到且正在运行时，用户会收到“已接受”。

## 示例请求和响应

### 启动虚拟机（Azure-AsyncOperation 标头出现 202 响应）
此示例演示如何确定虚拟机的“启动”操作的状态。初始请求采用以下格式：

    POST 
    https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Compute/virtualMachines/{vm-name}/start?api-version=2016-03-30

它返回状态代码 202。在标头值中可以看到：

    Azure-AsyncOperation : https://management.chinacloudapi.cn/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2016-03-30

若要检查异步操作的状态，请向该 URL 发送另一请求。

    GET 
    https://management.chinacloudapi.cn/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2016-03-30

响应正文包含操作的状态：

    {
      "startTime": "2017-01-06T18:58:24.7596323+00:00",
      "status": "InProgress",
      "name": "9a062a88-e463-4697-bef2-fe039df73a02"
    }

### 部署资源（Azure-AsyncOperation 标头出现 201 响应）

此示例演示将资源部署到 Azure 时，如何确定“部署”操作的状态。初始请求采用以下格式：

    PUT
    https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/microsoft.resources/deployments/{deployment-name}?api-version=2016-09-01

它返回状态代码 201。响应的正文包括：

    "provisioningState":"Accepted",

在标头值中可以看到：

    Azure-AsyncOperation: https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2016-09-01

若要检查异步操作的状态，请向该 URL 发送另一请求。

    GET 
    https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2016-09-01

响应正文包含操作的状态：

    {"status":"Running"}

部署完成后，响应包含：

    {"status":"Succeeded"}

### 创建存储帐户（Location 和 Retry-After 标头出现 202 响应）

此示例演示如何确定存储帐户的“创建”操作的状态。初始请求采用以下格式：

    PUT
    https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}?api-version=2016-01-01

请求正文包含存储帐户的属性：

    { "location": "China East", "properties": {}, "sku": { "name": "Standard_LRS" }, "kind": "Storage" }

它返回状态代码 202。在标头值中会出现以下两个值：

    Location: https://management.chinacloudapi.cn/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2016-01-01
    Retry-After: 17

等待特定的秒数（在 Retry-After 中指定）后，查看异步操作的状态，方法是向该 URL 发送另一请求。

    GET 
    https://management.chinacloudapi.cn/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2016-01-01

如果请求仍在运行，则会收到状态代码 202。如果请求已完成，则会收到状态代码 200，且响应的正文包含已创建存储帐户的属性。

## 后续步骤

* 有关每个 REST 操作的文档，请参阅 [REST API 文档](https://docs.microsoft.com/rest/api/)。
* 有关通过 Resource Manager REST API 管理资源的信息，请参阅[使用 Resource Manager REST API](/documentation/articles/resource-manager-rest-api/)。
* 有关通过 Resource Manager REST API 部署模板的信息，请参阅[使用 Resource Manager 模板和 Resource Manager REST API 部署资源](/documentation/articles/resource-group-template-deploy-rest/)。

<!---HONumber=Mooncake_0120_2017-->
<!-- Update_Description: new article how to track the async operation in Azure -->