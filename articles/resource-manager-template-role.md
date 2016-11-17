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
   ms.date="04/05/2016"
   wacn.date="05/05/2016"/>

# 角色分配模板架构

将用户、组或服务主体分配给指定作用域中的角色。

## 架构格式

若要创建角色分配，请将以下架构添加到模板的资源节中。
    
    {
        "type": "Microsoft.Authorization/roleAssignments",
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

| 名称 | 值 |
| ---- | ---- |
| type | 枚举<br />必需<br />**Microsoft.Authorization/roleAssignments**<br /><br />要创建的资源类型。 |
| apiVersion | 枚举<br />必需<br />**2014-10-01-preview**<br /><br />用于创建资源的 API 版本。 |  
| name | 字符串<br />必需<br />**全局唯一标识符**<br /><br />新角色分配的标识符。 |
| dependsOn | 数组<br />可选<br />资源名称或资源唯一标识符的逗号分隔列表。<br /><br />此角色分配依赖的资源的集合。如果要分配的角色的作用域限定为一个资源并且该资源部署在同一模板中，请在此元素中包含该资源名称，以确保该资源先部署。 | 
| properties | 对象<br />必需<br />[properties 对象](#properties)<br /><br />一个对象，用于标识角色定义、主体和范围。 |  

<a id="properties"></a>
### 属性对象

| 名称 | 值 |
| ------- | ---- |
| roleDefinitionId | 字符串<br />必需<br /> **/subscriptions/{subscription-id}/providers/Microsoft.Authorization/roleDefinitions/{role-definition-id}**<br /><br />要在角色分配中使用的现有角色定义的标识符。 |
| principalId | 字符串<br />必需<br />**全局唯一标识符**<br /><br />现有主体的标识符。这将映射到目录中的 ID 并可指向用户、服务主体或安全组。 |
| 作用域 | 字符串<br />必需<br />**/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}**（对于资源组）或<br />**/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{provider-namespace}/{resource-type}/{resource-name}**（对于资源）<br /><br />此角色分配适用的范围。 |


## 如何使用角色分配资源

当你需要在部署期间向角色添加用户、组或服务主体时，就会将角色分配添加到模板。角色分配从更高级别的作用域继承，因此如果你已向订阅级别的角色添加主体，则不需要为资源组或资源重新分配它。

可以使用以下命令为**名称**生成新标识符：

    PS C:\> $name = [System.Guid]::NewGuid().toString()

可以使用以下命令检索角色定义的全局唯一标识符：

    PS C:\> $roledef = (Get-AzureRmRoleDefinition -Name Reader).id

可以使用以下命令之一检索主体的标识符。

对于名为 **Auditors** 的组：

    PS C:\> $principal = (Get-AzureRmADGroup -SearchString Auditors).id

对于名为 **exampleperson** 的用户：

    PS C:\> $principal = (Get-AzureRmADUser -SearchString exampleperson).id

对于名为 **exampleapp** 的服务主体：

    PS C:\> $principal = (Get-AzureRmADServicePrincipal -SearchString exampleapp).id 
 

## 示例

以下示例将一个组分配给资源组的角色。

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

## 快速入门模板

以下模板演示如何使用角色分配资源：

- [将内置角色分配到资源组](https://github.com/Azure/azure-quickstart-templates/tree/master/101-rbac-builtinrole-resourcegroup)
- [将内置角色分配到现有 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-rbac-builtinrole-virtualmachine)
- [将内置角色分配到多个现有 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-rbac-builtinrole-multipleVMs)

## 后续步骤

- 有关模板结构的信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。

<!---HONumber=Mooncake_0425_2016-->