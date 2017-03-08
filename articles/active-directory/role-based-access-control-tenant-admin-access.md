<properties
    pageTitle="租户管理员提升访问权限 - Azure AD | Azure"
    description="本主题介绍适用于基于角色的访问控制 (RBAC) 的内置角色。"
    services="active-directory"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="rqureshi" />
<tags
    ms.assetid="b547c5a5-2da2-4372-9938-481cb962d2d6"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="02/09/2017"
    wacn.date="03/07/2017"
    ms.author="kgremban" />  


# 使用基于角色的访问控制提升作为租户管理员的访问权限

基于角色的访问控制可帮助租户管理员临时提升访问权限，以便他们可以授予高于标准的权限。需要时，租户管理员可以将自己提升到“用户访问管理员”角色。该角色为租户管理员提供权限，使其能够在“/”范围内向自己或其他人授予角色。

此功能非常重要，因为它允许租户管理员查看组织中存在的所有订阅。它还允许自动化应用（例如开票和审核）访问所有订阅，并从计费或资产管理的角度提供组织状态的准确视图。

## 如何使用 elevateAccess 为租户提供访问权限

基本过程使用以下步骤：

1. 使用 REST 调用 *elevateAccess*，以便在“/”范围内向你授予“用户访问管理员”角色。

    ```
    POST https://management.azure.com/providers/Microsoft.Authorization/elevateAccess?api-version=2016-07-01
    ```

2. 创建[角色分配](https://docs.microsoft.com/zh-cn/rest/api/authorization/roleassignments/)，以便在任何范围内分配任何角色。以下示例显示用于在“/”范围内分配“读者”角色的属性：

    ```
    { "properties":{
    "roleDefinitionId": "providers/Microsoft.Authorization/roleDefinitions/acdd72a7338548efbd42f606fba81ae7",
    "principalId": "cbc5e050-d7cd-4310-813b-4870be8ef5bb",
    "scope": "/"
    },
    "id": "providers/Microsoft.Authorization/roleAssignments/64736CA0-56D7-4A94-A551-973C2FE7888B",
    "type": "Microsoft.Authorization/roleAssignments",
    "name": "64736CA0-56D7-4A94-A551-973C2FE7888B"
    }
    ```

3. 同时作为用户访问管理员，你还可以在“/”范围内删除角色分配。

4. 撤销用户访问管理员权限，直到再次需要这些权限。


## 如何撤消 elevateAccess 操作

当用户调用 *elevateAccess* 时，将为用户创建角色分配，因此，若要撤销这些权限，需要删除该分配。

1.  调用 [GET roleDefinitions](https://docs.microsoft.com/zh-cn/rest/api/authorization/roledefinitions#RoleDefinitions_Get/)（其中 roleName = 用户访问管理员）以确定“用户访问管理员”角色的名称 GUID。响应应如下所示：

    ```
    {"value":[{"properties":{
    "roleName":"User Access Administrator",
    "type":"BuiltInRole",
    "description":"Lets you manage user access to Azure resources.",
    "assignableScopes":["/"],
    "permissions":[{"actions":["*/read","Microsoft.Authorization/*","Microsoft.Support/*"],"notActions":[]}],
    "createdOn":"0001-01-01T08:00:00.0000000Z",
    "updatedOn":"2016-05-31T23:14:04.6964687Z",
    "createdBy":null,
    "updatedBy":null},
    "id":"/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
    "type":"Microsoft.Authorization/roleDefinitions",
    "name":"18d7d88d-d35e-4fb5-a5c3-7773c20a72d9"}],
    "nextLink":null}
    ```

    保存 *name* 参数中的 GUID，在本例中为 **18d7d88d-d35e-4fb5-a5c3-7773c20a72d9**。

2. 调用 [GET roleAssignments](https://docs.microsoft.com/zh-cn/rest/api/authorization/roleassignments#RoleAssignments_Get/)，其中 principalId = 你自己的 ObjectId。这将列出租户中的所有分配。查找这样的分配：其中，范围是“/”，RoleDefinitionId 以在步骤 1 中找到的角色名称 GUID 结尾。此角色分配应如下所示：

    ```
    {"value":[{"properties":{
    "roleDefinitionId":"/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
    "principalId":"{objectID}",
    "scope":"/",
    "createdOn":"2016-08-17T19:21:16.3422480Z",
    "updatedOn":"2016-08-17T19:21:16.3422480Z",
    "createdBy":"93ce6722-3638-4222-b582-78b75c5c6d65",
    "updatedBy":"93ce6722-3638-4222-b582-78b75c5c6d65"},
    "id":"/providers/Microsoft.Authorization/roleAssignments/e7dd75bc-06f6-4e71-9014-ee96a929d099",
    "type":"Microsoft.Authorization/roleAssignments",
    "name":"e7dd75bc-06f6-4e71-9014-ee96a929d099"}],
    "nextLink":null}
    ```

    同样，保存 *name* 参数中的 GUID，在本例中为 **e7dd75bc-06f6-4e71-9014-ee96a929d099**。

3. 最后，调用 [DELETE roleAssignments](/rest/api/authorization/roleassignments#RoleAssignments_DeleteById/)，其中 roleAssignmentId = 在步骤 2 中找到的名称 GUID。

## 后续步骤

- 详细了解[使用 REST 管理基于角色的访问控制](/documentation/articles/role-based-access-control-manage-access-rest/)

<!---HONumber=Mooncake_0227_2017-->
