<properties
   pageTitle="用于链接资源的资源管理器模板 | Microsoft Azure"
   description="介绍用于通过模板在相关资源之间部署链接的资源管理器架构。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="01/04/2016"
   wacn.date="02/26/2016"/>

# 资源链接模板架构

在两个资源之间创建链接。该链接将应用于称为源资源的资源。链接中的第二个资源称为目标资源。

## 架构格式

若要创建链接，请将以下架构添加到模板的资源节中。
    
    {
        "type": enum,
        "apiVersion": "2015-01-01",
        "name": string,
        "dependsOn": [ array values ],
        "properties":
        {
            "targetId": string,
            "notes": string
        }
    }



## 值

下表描述了需要在架构中设置的值。

| Name | 类型 | 必选 | 允许的值 | 说明 |
| ---- | ---- | -------- | ---------------- | ----------- |
| type | 枚举 | 是 | **{namespace}/{type}/providers/links** | 要创建的资源类型。{namespace} 和 {type} 值是指源资源的提供程序命名空间和资源类型。 |
| apiVersion | 枚举 | 是 | **2015-01-01** | 要用于创建该资源的 API 版本。 |  
| name | 字符串 | 是 | **{resouce}/Microsoft.Resources/{linkname}**<br /><br />最多为 64 个字符<br />它不能包含 <、>、%、&、? 或任何控制字符。 | 一个值，用于指定源资源的名称和链接名称。 |
| dependsOn | 数组 | 否 | 资源名称或资源唯一标识符的逗号分隔列表。 | 此链接依赖的资源的集合。如果要链接的资源部署在同一模板中，请在此元素中包含这些资源名称以确保它们先部署。 | 
| properties | 对象 | 是 | （下面显示） | 一个对象，标识要链接到的资源，以及有关该链接的说明。 |  

### 属性对象

| Name | 类型 | 必选 | 允许的值 | 说明 |
| ------- | ---- | ---------------- | -------- | ----------- |
| targetId | 字符串 | 是 | | 要链接到的目标资源的标识符。 |
| 说明 | 字符串 | 否 | 512 个字符 | 该锁的说明。 |


## 如何使用链接资源

当资源在部署后仍存在依赖关系时，可以在这两个资源之间应用链接。例如，应用可能会连接到不同资源组中的数据库。可以通过创建从应用到数据库的链接来定义该依赖关系。链接使你能够记录两个资源之间的关系。稍后，你或你组织中的其他人可以针对链接查询资源以了解该资源如何使用其他资源。

所有链接资源必须属于同一订阅。每个资源可以链接到 50 个其他资源。如果删除或移动任何链接资源，链接所有者必须清理剩余的链接。

若要通过 REST 使用链接，请参阅[链接的资源](https://msdn.microsoft.com/zh-cn/library/azure/mt238499.aspx)。

使用以下 Azure PowerShell 命令可查看订阅中的所有链接。你可以提供其他参数来限制结果。

    Get-AzureRmResource -ResourceType Microsoft.Resources/links -isCollection -OutputObjectFormat New

## 示例

以下示例将只读锁应用于 Web 应用。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "hostingPlanName": {
                "type": "string"
            }
        },
        "variables": {
            "siteName": "[concat('site',uniqueString(resourceGroup().id))]"
        },
        "resources": [
            {
                "apiVersion": "2015-08-01",
                "type": "Microsoft.Web/serverfarms",
                "name": "[parameters('hostingPlanName')]",
                "location": "[resourceGroup().location]",
                "sku": {
                    "tier": "Free",
                    "name": "f1",
                    "capacity": 0
                },
                "properties": {
                    "numberOfWorkers": 1
                }
            },
            {
                "apiVersion": "2015-08-01",
                "name": "[variables('siteName')]",
                "type": "Microsoft.Web/sites",
                "location": "[resourceGroup().location]",
                "dependsOn": [ "[parameters('hostingPlanName')]" ],
                "properties": {
                    "serverFarmId": "[parameters('hostingPlanName')]"
                }
            },
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
        ],
        "outputs": {}
    }


## 后续步骤

- 有关模板结构的信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)。

<!---HONumber=Mooncake_0215_2016-->