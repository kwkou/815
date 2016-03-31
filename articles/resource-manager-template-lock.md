<properties
   pageTitle="资源锁的资源管理器模板 | Azure"
   description="显示资源锁的资源管理器架构。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="01/21/2016"
   wacn.date="03/21/2016"/>

# 资源锁 - 模板架构

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

| Name | 类型 | 必选 | 允许的值 | 说明 |
| ---- | ---- | -------- | ---------------- | ----------- |
| type | 枚举 | 是 | 对于资源：<br />**{namespace}/{type}/providers/locks**<br /><br />对于资源组：<br />**Microsoft.Authorization/locks** | 要创建的资源类型。 |
| apiVersion | 枚举 | 是 | **2015-01-01** | 要用于创建该资源的 API 版本。 |  
| name | 字符串 | 是 | 对于资源：<br />**{resouce}/Microsoft.Authorization/{lockname}**<br /><br />对于资源组：<br />**{lockname}**<br /><br />最多为 64 个字符<br />不能包含 <、>、%、&、? 或任何控制字符。| 一个值，同时指定要锁定的资源和锁的名称。|
| dependsOn | 数组 | 否 | 资源名称或资源唯一标识符的逗号分隔列表。| 此锁所依赖的资源的集合。如果要锁定的资源部署在同一模板中，请在此元素中包含该资源名称以确保该资源先部署。|
| 属性 | 对象 | 是 |（下面显示）| 一个对象，用于标识锁的类型，以及有关锁的说明。| 

### 属性对象

| Name | 类型 | 必选 | 允许的值 | 说明 |
| ------- | ---- | ---------------- | -------- | ----------- |
| 级别 | 枚举 | 是 | **CannotDelete** | 要应用于作用域的锁的类型。CanNotDelete 允许修改，但阻止删除。 |
| 说明 | 字符串 | 否 | 512 个字符 | 该锁的说明。 |


## 如何使用锁资源

将此资源添加到你的模板可防止对资源执行指定的操作。该锁将应用于所有用户和组。通常情况下，你只在有限的持续时间内应用锁，例如，当某个进程正在运行时，你想要确保组织中的某人不会无意中修改或删除某个资源。

若要创建或删除管理锁，你必须有权访问 **Microsoft.Authorization/*** 或 **Microsoft.Authorization/locks/*** 操作。在内置角色中，只有**所有者**和**用户访问管理员**有权执行这些操作。有关基于角色的访问控制的信息，请参阅[管理对资源的访问权限](/documentation/articles/resource-group-rbac)。

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
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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

- 有关模板结构的信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)。
- 有关锁的详细信息，请参阅[使用 Azure 资源管理器锁定资源](/documentation/articles/resource-group-lock-resources)。

<!---HONumber=Mooncake_0314_2016-->