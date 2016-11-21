<properties
   pageTitle="用于角色分配的资源管理器模板 | Azure"
   description="介绍用于通过模板部署角色分配的资源管理器架构。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="10/03/2016"
   wacn.date="11/21/2016"
   ms.author="tomfitz"/>

# 角色分配模板架构

将用户、组或服务主体分配给指定作用域中的角色。

## 资源格式

若要创建角色分配，请将以下架构添加到模板的资源节中。
    
    {
        "type": string,
        "apiVersion": "2015-07-01",
        "name": string,
        "dependsOn": [ array values ],
        "properties":
        {
            "roleDefinitionId": string,
            "principalId": string,
            "scope": string
        }
    }

## 值

下表描述了需要在架构中设置的值。

| 名称 | 必选 | 说明 |
| ---- | -------- | ----------- |
| type | 是 | 要创建的资源类型。<br /><br /> 对于资源组：<br />**Microsoft.Authorization/roleAssignments**<br /><br />对于资源：<br />**{provider-namespace}/{resource-type}/providers/roleAssignments** |
| apiVersion |是 | 要用于创建资源的 API 版本。<br /><br /> 使用 **2015-07-01**。 | 
| 名称 | 是 | 新角色分配的全局唯一标识符。 |
| dependsOn | 否 | 资源名称或资源唯一标识符的逗号分隔数组。<br /><br />此角色分配所依赖的资源的集合。如果要分配的角色的作用域限定为一个资源并且该资源部署在同一模板中，请在此元素中包含该资源名称，以确保该资源先部署。 |
| properties | 是 | 属性对象，用于标识角色定义、主体和作用域。 |

### 属性对象

| 名称 | 必选 | 说明 |
| ---- | -------- | ----------- |
| roleDefinitionId | 是 | 要在角色分配中使用的现有角色定义的标识符。<br /><br /> 使用以下格式：<br />**/subscriptions/{subscription-id}/providers/Microsoft.Authorization/roleDefinitions/{role-definition-id}** |
| principalId | 是 | 现有主体的全局唯一标识符。此值会映射到目录中的 ID 并可指向用户、服务主体或安全组。 |
| 作用域 | 否 | 此角色分配适用的作用域。<br /><br />对于资源组，请使用：<br />**/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}**<br /><br />对于资源，请使用：<br />**/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{provider-namespace}/{resource-type}/{resource-name}** | |


## 如何使用角色分配资源

当你需要在部署期间向角色添加用户、组或服务主体时，就会将角色分配添加到模板。角色分配从更高级别的作用域继承，因此，如果已向订阅级别的角色添加主体，则不需要为资源组或资源重新分配它。

使用角色分配时，需要提供多个标识符值。可以通过 PowerShell 或 Azure CLI 检索这些值。

### PowerShell

角色分配的名称需要全局唯一标识符。可以使用以下命令为**名称**生成新标识符：

    $name = [System.Guid]::NewGuid().toString()

可以使用以下命令检索角色定义的标识符：

    $roledef = (Get-AzureRmRoleDefinition -Name Reader).id

可以使用以下命令之一检索主体的标识符。

对于名为 **Auditors** 的组：

    $principal = (Get-AzureRmADGroup -SearchString Auditors).id

对于名为 **exampleperson** 的用户：

    $principal = (Get-AzureRmADUser -SearchString exampleperson).id

对于名为 **exampleapp** 的服务主体：

    $principal = (Get-AzureRmADServicePrincipal -SearchString exampleapp).id 

### Azure CLI

可以使用以下命令检索角色定义的标识符：

    azure role show Reader --json | jq .[].Id -r

可以使用以下命令之一检索主体的标识符。

对于名为 **Auditors** 的组：

    azure ad group show --search Auditors --json | jq .[].objectId -r

对于名为 **exampleperson** 的用户：

    azure ad user show --search exampleperson --json | jq .[].objectId -r

对于名为 **exampleapp** 的服务主体：

    azure ad sp show --search exampleapp --json | jq .[].objectId -r

## 示例

下面的模板接收角色的标识符以及用户、组或服务主体的标识符。它在资源组级别分配角色。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "roleDefinitionId": {
                "type": "string"
            },
            "roleAssignmentId": {
                "type": "string"
            },
            "principalId": {
                "type": "string"
            }
        },
        "resources": [
            {
              "type": "Microsoft.Authorization/roleAssignments",
              "apiVersion": "2015-07-01",
              "name": "[parameters('roleAssignmentId')]",
              "properties":
              {
                "roleDefinitionId": "[concat(subscription().id, '/providers/Microsoft.Authorization/roleDefinitions/', parameters('roleDefinitionId'))]",
                "principalId": "[parameters('principalId')]",
                "scope": "[concat(subscription().id, '/resourceGroups/', resourceGroup().name)]"
              }
            }
        ],
        "outputs": {}
    }



下一个模板创建存储帐户，并将读取者角色分配到存储帐户。模板中已包含两个组和读取者角色的标识符，可简化部署。这些值可在部署时通过脚本检索，并作为参数传递。

    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "roleName": {
          "type": "string"
        },
        "groupToAssign": {
          "type": "string",
          "allowedValues": [
            "Auditors",
            "Limited"
          ]
        }
      },
      "variables": {
        "readerRole": "[concat('/subscriptions/',subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7')]",
        "storageName": "[concat('storage', uniqueString(resourceGroup().id))]",
        "scope": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Storage/storageAccounts/', variables('storageName'))]",
        "Auditors": "1c272299-9729-462a-8d52-7efe5ece0c5c",
        "Limited": "7c7250f0-7952-441c-99ce-40de5e3e30b5",
        "fullRoleName": "[concat(variables('storageName'), '/Microsoft.Authorization/', parameters('roleName'))]"
      },
      "resources": [
        {
          "apiVersion": "2016-01-01",
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[variables('storageName')]",
          "location": "[resourceGroup().location]",
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "Storage",
          "tags": {
            "displayName": "MyStorageAccount"
          },
          "properties": {}
        },
        {
          "type": "Microsoft.Storage/storageAccounts/providers/roleAssignments",
          "apiVersion": "2015-07-01",
          "name": "[variables('fullRoleName')]",
          "dependsOn": [
            "[variables('storageName')]"
          ],
          "properties": {
            "roleDefinitionId": "[variables('readerRole')]",
            "principalId": "[variables(parameters('groupToAssign'))]"
          }
        }
      ]
    }

## 快速入门模板

以下模板演示如何使用角色分配资源：

- [将内置角色分配到资源组](https://github.com/Azure/azure-quickstart-templates/tree/master/101-rbac-builtinrole-resourcegroup)
- [将内置角色分配到现有 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-rbac-builtinrole-virtualmachine)
- [将内置角色分配到多个现有 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-rbac-builtinrole-multipleVMs)

## 后续步骤

- 有关模板结构的信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。
- 有关基于角色的访问控制的详细信息，请参阅 [Azure Active Directory 基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)。

<!---HONumber=Mooncake_1114_2016-->