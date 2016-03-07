<properties
   pageTitle="用于角色分配的资源管理器模板 | Azure"
   description="介绍用于通过模板部署角色分配的资源管理器架构。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="01/04/2016"
   wacn.date="02/26/2016"/>

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

| Name | 类型 | 必选 | 允许的值 | 说明 |
| ---- | ---- | -------- | ---------------- | ----------- |
| type | 枚举 | 是 | **Microsoft.Authorization/roleAssignments** | 要创建的资源类型。 |
| apiVersion | 枚举 | 是 | **2015-07-01** | 要用于创建该资源的 API 版本。 |  
| name | 字符串 | 是 | 全局唯一标识符 | 新角色分配的标识符。 |
| dependsOn | 数组 | 否 | 资源名称或资源唯一标识符的逗号分隔列表。 | 此角色分配依赖的资源的集合。如果要分配的角色的作用域限定为一个资源并且该资源部署在同一模板中，请在此元素中包含该资源名称，以确保该资源先部署。 | 
| properties | 对象 | 是 | （下面显示） | 一个对象，用于标识角色定义、主体和作用域。 |  

### 属性对象

| Name | 类型 | 必选 | 允许的值 | 说明 |
| ------- | ---- | ---------------- | -------- | ----------- |
| roleDefinitionId | 字符串 | 是 | **/subscriptions/{subscription-id}/providers/Microsoft.Authorization/roleDefinitions/{role-definition-id}** | 要在角色分配中使用的现有角色定义的标识符。 |
| principalId | 字符串 | 是 | 全局唯一标识符 | 现有主体的标识符。这将映射到目录中的 ID 并可指向用户、服务主体或安全组。 |
| 作用域 | 字符串 | 是 | 对于资源组：<br />**/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}**<br /><br />对于资源：<br />**/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{provider-namespace}/{resource-type}/{resource-name}** | 此角色分配适用的作用域。 |


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


## 后续步骤

- 有关模板结构的信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)。
- 有关基于角色的访问控制的详细信息，请参阅 [Azure Active Directory 基于角色的访问控制](/documentation/articles/role-based-access-control-configure)。

<!---HONumber=Mooncake_0215_2016-->