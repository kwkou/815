<properties
    pageTitle="在 Azure 解决方案中链接相关资源 | Azure"
    description="在 Azure Resource Manager 的不同资源组中的相关资源之间创建链接。"
    services="azure-resource-manager"
    documentationcenter=""
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="0738d072-c093-4cf1-a790-de13025d9d60"
    ms.service="azure-resource-manager"
    ms.workload="multiple"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="08/01/2016"
    wacn.date="03/03/2017"
    ms.author="tomfitz" />  


# 从不同资源组链接相关资源
在部署期间，你可以将一个资源标记为依赖于另一个资源，但该生命周期在部署时结束。部署后，依赖资源之间没有任何标识关系。Resource Manager 提供了一个称为资源链接的功能来建立资源之间的永久关系。

通过资源链接，可以记录跨资源组的关系。例如，将数据库的生命周期驻留在一个资源组中，将应用程序的不同于前者的生命周期驻留在另一个资源组中是一种常见现象。应用将连接到数据库，因此你需要在应用和数据库之间标记链接。

所有链接资源必须属于同一订阅。每个资源可以链接到 50 个其他资源。查询相关资源的唯一方法是通过 REST API。如果删除或移动任何链接资源，链接所有者必须清理剩余的链接。删除链接到其他资源的资源时，你**不会**收到警告。

## 模板中的链接
若要在模板中定义链接，请包括将源资源的资源提供程序命名空间和类型与 **/providers/links** 组合在一起的资源类型。该名称必须包含源资源的名称。你提供目标资源的资源 ID。下面的示例将在网站与存储帐户之间建立链接。

    {
      "type": "Microsoft.Web/sites/providers/links",
      "apiVersion": "2015-01-01",
      "name": "[concat(variables('siteName'),'/Microsoft.Resources/SiteToStorage')]",
      "dependsOn": [ "[variables('siteName')]" ],
      "properties": {
        "targetId": "[resourceId('Microsoft.Storage/storageAccounts','storagecontoso')]",
        "notes": "This web site uses the storage account to store user information."
      }
    }


## 使用 REST API 进行链接
若要定义已部署的资源之间的链接，请运行：

    PUT https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/{provider-namespace}/{resource-type}/{resource-name}/providers/Microsoft.Resources/links/{link-name}?api-version={api-version}

将 {subscription-id} 替换为订阅 ID。将 {resource-group}、{provider-namespace}、{resource-type} 和 {resource-name} 替换为标识链接中的第一个资源的值。将 {link-name} 替换为要创建的链接名称。将 2015-01-01 用于 api-version。

在该请求中，包括定义链接中第二个资源的一个对象：

    {
        "name": "{new-link-name}",
        "properties": {
            "targetId": "subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/{provider-namespace}/{resource-type}/{resource-name}/",
            "notes": "{link-description}",
        }
    }

Properties 元素包含第二个资源的标识符。

可以使用以下命令在订阅中查询链接：

    https://management.chinacloudapi.cn/subscriptions/{subscription-id}/providers/Microsoft.Resources/links?api-version={api-version}

有关更多示例，包括如何检索有关链接的信息，请参阅[链接资源](https://docs.microsoft.com/rest/api/resources/resourcelinks)。

## 后续步骤
* 您还可以使用标记来组织您的资源。若要了解有关标记资源的信息，请参阅[使用标记来组织你的资源](/documentation/articles/resource-group-using-tags/)。
* 有关如何创建模板并定义要部署的资源的说明，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/)。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description:update meta properties; wording update-->