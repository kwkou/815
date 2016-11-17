<properties
   pageTitle="资源锁的资源管理器模板 | Azure"
   description="介绍用于通过模板部署资源锁的资源管理器架构。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="04/05/2016"
   wacn.date="05/05/2016"/>

# 资源锁模板架构

在资源及其子资源上创建新锁。

## 架构格式

若要创建锁，请将以下架构添加到模板的资源节中。
    
    {
        "type": enum,
        "apiVersion": "2015-01-01",
        "name": string,
        "dependsOn": [ array values ],
        "properties":
        {
            "level": enum,
            "notes": string
        }
    }



## 值

下表描述了需要在架构中设置的值。

| 名称 | 值 |
| ---- | ---- | 
| type | 枚举<br />必需<br />**{namespace}/{type}/providers/locks**（对于资源）或<br />**Microsoft.Authorization/locks**（对于资源组）<br /><br />要创建的资源类型。 |
| apiVersion | 枚举<br />必需<br />**2015-01-01**<br /><br />要用于创建该资源的 API 版本。 |  
| name | 字符串<br />必需<br />**{resource}/Microsoft.Authorization/{lockname}**（对于资源）或<br />**{lockname}**（对于资源组）<br />最多为 64 个字符，且不能包含 <、>、%、&、? 或任何控制字符。<br /><br />一个值，同时指定要锁定的资源和锁的名称。 |
| dependsOn | 数组<br />可选<br />资源名称或资源唯一标识符的逗号分隔列表。<br /><br />此锁所依赖的资源的集合。如果要锁定的资源部署在同一模板中，请在此元素中包含该资源名称以确保该资源先部署。 | 
| properties | 对象<br />必需<br />[properties 对象](#properties)<br /><br />一个对象，用于标识锁的类型，以及有关锁的说明。 |  

<a id="properties"></a>
### 属性对象

| 名称 | 值 |
| ------- | ---- |
| 级别 | 枚举<br />必需<br />**CannotDelete**<br /><br />要应用于作用域的锁的类型。CanNotDelete 允许修改，但阻止删除。 |
| 说明 | 字符串<br />可选<br />最多为 512 个字符<br /><br />锁的描述。 |


## 如何使用锁资源

将此资源添加到你的模板可防止对资源执行指定的操作。该锁将应用于所有用户和组。通常情况下，你只在有限的持续时间内应用锁，例如，当某个进程正在运行时，你想要确保组织中的某人不会无意中修改或删除某个资源。

若要创建或删除管理锁，你必须有权访问 **Microsoft.Authorization/*** 或 **Microsoft.Authorization/locks/*** 操作。在内置角色中，只有**所有者**和**用户访问管理员**有权执行这些操作。

锁将应用于指定的资源和任何子资源。

你可以使用 PowerShell 命令 **Remove-AzureRmResourceLock** 或 REST API 的[删除操作](https://msdn.microsoft.com/zh-cn/library/azure/mt204562.aspx)删除锁。

## 示例

以下示例将一个 cannot-delete 锁应用到 Web 应用。

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
                "name": "[variables('siteName')]",
                "type": "Microsoft.Web/sites",
                "location": "[resourceGroup().location]",
                "properties": {
                    "serverFarmId": "[parameters('hostingPlanName')]"
                },
            },
            {
                "type": "Microsoft.Web/sites/providers/locks",
                "apiVersion": "2015-01-01",
                "name": "[concat(variables('siteName'),'/Microsoft.Authorization/MySiteLock')]",
                "dependsOn": [ "[variables('siteName')]" ],
                "properties":
                {
                    "level": "CannotDelete",
                    "notes": "my notes"
                }
             }
        ],
        "outputs": {}
    }

以下示例将一个 cannot-delete 锁应用到资源组。

    {
        "$schema": "https://schema.management.windowsazure.cn/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {},
        "resources": [
            {
                "type": "Microsoft.Authorization/locks",
                "apiVersion": "2015-01-01",
                "name": "MyGroupLock",
                "properties":
                {
                    "level": "CannotDelete",
                    "notes": "my notes"
                }
            }
        ],
        "outputs": {}
    }

## 后续步骤

- 有关模板结构的信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。
- 有关锁的详细信息，请参阅[使用 Azure 资源管理器锁定资源](/documentation/articles/resource-group-lock-resources/)。

<!---HONumber=Mooncake_0425_2016-->