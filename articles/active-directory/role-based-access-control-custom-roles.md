<properties
    pageTitle="为 Azure RBAC 创建自定义角色 | Azure"
    description="了解如何通过 Azure 基于角色的访问控制来定义自定义角色，以便在 Azure 订阅中进行更精确的身份管理。"
    services="active-directory"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="e4206ea9-52c3-47ee-af29-f6eef7566fa5"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="03/21/2017"
    wacn.date="05/08/2017"
    ms.author="kgremban"
    ms.custom="H1Hack27Feb2017"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="41047be4bef215cb57e08bc50ce0be291ad6b7d5"
    ms.lasthandoff="04/28/2017" />

# <a name="create-custom-roles-for-azure-role-based-access-control"></a>创建用于 Azure 基于角色的访问控制的自定义角色
如果没有符合你特定访问需求的内置角色，可在 Azure 基于角色的访问控制 (RBAC) 中创建自定义角色。 可以使用 [Azure PowerShell](/documentation/articles/role-based-access-control-manage-access-powershell/)、[Azure 命令行接口](/documentation/articles/role-based-access-control-manage-access-azure-cli/) (CLI) 和 [REST API](/documentation/articles/role-based-access-control-manage-access-rest/) 创建自定义角色。 与内置角色一样，可以将自定义角色分配到订阅、资源组和资源范围内的用户、组和应用程序。 自定义角色存储在 Azure AD 租户中，并且可以在将该租户用作其 Azure AD 目录的所有订阅之间共享。

每个租户可以创建最多 2000 个自定义角色。 

下面是用于监视和重新启动虚拟机的自定义角色的一个示例：

    {
      "Name": "Virtual Machine Operator",
      "Id": "cadb4a5a-4e7a-47be-84db-05cad13b6769",
      "IsCustom": true,
      "Description": "Can monitor and restart virtual machines.",
      "Actions": [
        "Microsoft.Storage/*/read",
        "Microsoft.Network/*/read",
        "Microsoft.Compute/*/read",
        "Microsoft.Compute/virtualMachines/start/action",
        "Microsoft.Compute/virtualMachines/restart/action",
        "Microsoft.Authorization/*/read",
        "Microsoft.Resources/subscriptions/resourceGroups/read",
        "Microsoft.Insights/alertRules/*",
        "Microsoft.Insights/diagnosticSettings/*",
        "Microsoft.Support/*"
      ],
      "NotActions": [

      ],
      "AssignableScopes": [
        "/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e",
        "/subscriptions/e91d47c4-76f3-4271-a796-21b4ecfe3624",
        "/subscriptions/34370e90-ac4a-4bf9-821f-85eeedeae1a2"
      ]
    }

## <a name="actions"></a>操作
自定义角色的 **Actions** 属性指定该角色向其授予访问权限的 Azure 操作。 它是操作字符串的集合，可标识 Azure 资源提供程序的安全对象操作。 操作字符串遵循格式 `Microsoft.<ProviderName>/<ChildResourceType>/<action>`。 包含通配符 (\*) 的操作字符串可以授权访问与该操作字符串相匹配的所有操作。 例如：

- `*/read` 向所有 Azure 资源提供程序的所有资源类型的读取操作授予访问权限。
- `Microsoft.Compute/*` 授予对 Microsoft.Compute 资源提供程序中的所有资源类型的所有操作的访问权限。
- `Microsoft.Network/*/read` 向 Azure 的 Microsoft.Network 资源提供程序中的所有资源类型的读取操作授予访问权限。
- `Microsoft.Compute/virtualMachines/*` 向虚拟机及其子资源类型的所有操作授予访问权限。
- `Microsoft.Web/sites/restart/Action` 授予重新启动网站的访问权限。

使用 `Get-AzureRmProviderOperation`（在 PowerShell 中）或 `azure provider operations show`（在 Azure CLI 中）列出 Azure 资源提供程序的操作。 还可以使用这些命令来验证操作字符串是否有效，并展开通配符操作字符串。

    Get-AzureRMProviderOperation Microsoft.Compute/virtualMachines/*/action | FT Operation, OperationName

    Get-AzureRMProviderOperation Microsoft.Network/*

![PowerShell 屏幕截图 - Get-AzureRMProviderOperation](./media/role-based-access-control-configure/1-get-azurermprovideroperation-1.png)

    azure provider operations show "Microsoft.Compute/virtualMachines/*/action" --js on | jq '.[] | .operation'

    azure provider operations show "Microsoft.Network/*"

![Azure CLI 屏幕截图 - Azure 提供程序操作显示“Microsoft.Compute/virtualMachines/\*/action” ](./media/role-based-access-control-configure/1-azure-provider-operations-show.png)

## <a name="notactions"></a>NotActions
如果排除受限制的操作可以更方便地定义希望允许的操作集，则使用 **NotActions** 属性。 通过用 **Actions** 操作减去 **NotActions** 操作可以计算出自定义角色授予的访问权限。

> [AZURE.NOTE]
> 如果用户分配到的角色排除 **NotActions**中的一个操作，并且分配到向同一操作授予访问权限的第二个角色，则用户可以执行该操作。 **NotActions** 不是拒绝规则 - 它只是一个简便方法，可在需要排除特定操作时创建一组允许的操作。
>
>

## <a name="assignablescopes"></a>AssignableScopes
自定义角色的 **AssignableScopes** 属性指定可以分配该自定义角色的范围（订阅、资源组或资源）。 可以让自定义角色只在需要它的订阅或资源组中进行分配，而不影响其他订阅或资源组的用户体验。

有效的可分配范围的示例包括：

- “/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e”、“/subscriptions/e91d47c4-76f3-4271-a796-21b4ecfe3624”- 可让角色在两个订阅中进行分配。
- “/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e”- 可让角色在单个订阅中进行分配。
- “/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e/resourceGroups/Network”- 可让角色只在网络资源组中进行分配。

> [AZURE.NOTE]
> 必须使用至少一个订阅、资源组或资源 ID。
>
>

## <a name="custom-roles-access-control"></a>自定义角色的访问控制
自定义角色的 **AssignableScopes** 属性还能控制谁可以查看、修改和删除角色。

- 谁可以创建自定义角色？
    订阅、资源组和资源的所有者（和用户访问管理员）可以创建能在这些范围中使用的自定义角色。
    创建角色的用户需要能够执行角色的所有 **AssignableScopes** 上的 `Microsoft.Authorization/roleDefinition/write` 操作。
- 谁可以修改自定义角色？
    订阅、资源组和资源的所有者（和用户访问管理员）可以修改这些范围中的自定义角色。 用户需要能够执行自定义角色的所有 **AssignableScopes** 上的 `Microsoft.Authorization/roleDefinition/write` 操作。
- 谁可以查看自定义角色？
    Azure RBAC 中的所有内置角色都允许查看可以进行分配的角色。 范围中能够执行 `Microsoft.Authorization/roleDefinition/read` 操作的用户可以查看能在该范围中进行分配的 RBAC 角色。

## <a name="see-also"></a>另请参阅
- [基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)：Azure 门户中的 RBAC 入门。
- 了解如何通过以下方式管理访问权限：
  - [PowerShell](/documentation/articles/role-based-access-control-manage-access-powershell/)
  - [Azure CLI](/documentation/articles/role-based-access-control-manage-access-azure-cli/)
  - [REST API](/documentation/articles/role-based-access-control-manage-access-rest/)
- [内置角色](/documentation/articles/role-based-access-built-in-roles/)：获取有关 RBAC 中标配角色的详细信息。

<!---Update_Description: wording update -->