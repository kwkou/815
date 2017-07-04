<properties
    pageTitle="Azure Resource Manager 请求限制 | Azure"
    description="介绍在达到订阅限制后如何对 Azure Resource Manager 请求进行限制。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="e1047233-b8e4-4232-8919-3268d93a3824"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/11/2017"
    wacn.date="02/10/2017"
    ms.author="tomfitz" />  


# 限制 Resource Manager 请求数
对于每个订阅和租户，Resource Manager 将读取请求数限制为 15,000/小时，将写入请求数限制为 1,200/小时。如果应用程序或脚本达到了这些限制，则需对请求数进行限制。本主题介绍如何在达到限制之前确定剩余的请求数，以及在达到限制后如何响应。

达到限制时，用户会收到 HTTP 状态代码“429 请求过多”。

请求数仅限于订阅或租户。如果订阅中有多个并发应用程序在进行请求，则会将这些应用程序发出的请求加在一起来确定剩余请求数。

订阅范围的请求是需要传递订阅 ID 的请求，例如在订阅中检索资源组。租户范围的请求是指不包括订阅 ID 的请求，例如检索有效的 Azure 位置。

## 剩余请求数
可以通过检查响应标头确定剩余请求数。每个请求都包含剩余读取和写入请求数的值。下表说明了各种响应标头，用户可以检查其中是否存在这些值：

| 响应标头 | 说明 |
| --- | --- |
| x-ms-ratelimit-remaining-subscription-reads |订阅范围的剩余读取数 |
| x-ms-ratelimit-remaining-subscription-writes |订阅范围的剩余写入数 |
| x-ms-ratelimit-remaining-tenant-reads |租户范围的剩余读取数 |
| x-ms-ratelimit-remaining-tenant-writes |租户范围的剩余写入数 |
| x-ms-ratelimit-remaining-subscription-resource-requests |订阅范围的剩余资源类型请求数。<br /><br />仅当某个服务重写了默认限制时，才会返回此标头值。Resource Manager 会添加此值而非订阅读取数或写入数。 |
| x-ms-ratelimit-remaining-subscription-resource-entities-read |订阅范围的剩余资源类型集合请求数。<br /><br />仅当某个服务重写了默认限制时，才会返回此标头值。此值提供剩余集合请求（列出资源）的数目。 |
| x-ms-ratelimit-remaining-tenant-resource-requests |租户范围的剩余资源类型请求数。<br /><br />此标头仅针对租户级别的请求添加，并且仅当某个服务重写了默认限制时添加。Resource Manager 添加此值而非租户读取数或写入数。 |
| x-ms-ratelimit-remaining-tenant-resource-entities-read |租户范围的剩余资源类型集合请求数。<br /><br />此标头仅针对租户级别的请求添加，并且仅当某个服务重写了默认限制时添加。 |

## 检索标头值
在代码或脚本中检索这些标头值与检索任何标头值无异。

例如，在 **C#** 中，可以使用以下代码从名为 **response** 的 **HttpWebResponse** 对象检索标头值：

    response.Headers.GetValues("x-ms-ratelimit-remaining-subscription-reads").GetValue(0)

在 **PowerShell** 中，可以通过 Invoke-WebRequest 操作检索标头值。

    $r = Invoke-WebRequest -Uri https://management.chinacloudapi.cn/subscriptions/{guid}/resourcegroups?api-version=2016-09-01 -Method GET -Headers $authHeaders
    $r.Headers["x-ms-ratelimit-remaining-subscription-reads"]

或者，如果需要查看剩余请求数以便进行调试，可在 **PowerShell** cmdlet 中提供 **-Debug** 参数。

    Get-AzureRmResourceGroup -Debug

这会返回许多值，包括以下响应值：

    ...
    DEBUG: ============================ HTTP RESPONSE ============================

    Status Code:
    OK

    Headers:
    Pragma                        : no-cache
    x-ms-ratelimit-remaining-subscription-reads: 14999
    ...

在 **Azure CLI** 中，可以使用更详细的选项检索标头值。

    azure group list -vv --json

这会返回许多值，包括以下对象：

    ...
    silly: returnObject
    {
      "statusCode": 200,
      "header": {
        "cache-control": "no-cache",
        "pragma": "no-cache",
        "content-type": "application/json; charset=utf-8",
        "expires": "-1",
        "x-ms-ratelimit-remaining-subscription-reads": "14998",
        ...

## 在发送下一请求之前等待
达到请求限制时，Resource Manager 会返回 **429** HTTP 状态代码以及标头中的 **Retry-After** 值。**Retry-After** 值指定应用程序在发送下一请求之前应等待（或睡眠）的秒数。如果在重试值所对应的时间尚未用完之前发送请求，则系统不会处理该请求，而会返回新的重试值。

## 后续步骤

* 有关限制和配额的详细信息，请参阅 [Azure 订阅和服务限制、配额与约束](/documentation/articles/azure-subscription-service-limits/)。
* 若要了解如何处理异步 REST 请求，请参阅[跟踪异步 Azure 操作](/documentation/articles/resource-manager-async-operations/)。

<!---HONumber=Mooncake_0206_2017-->
<!-- Update_Description: meta data;wording update -->