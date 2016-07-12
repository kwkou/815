<properties 
	pageTitle="使用 Windows PowerShell 管理基于角色的访问控制" 
	description="使用 Windows PowerShell 管理基于角色的访问控制" 
	services="azure-portal" 
	documentationCenter="na" 
	authors="IHenkel"
	manager="stevenpo"
	editor="mollybos"/>

<tags 
	ms.service="azure-portal" 
	ms.date="08/14/2015"
	wacn.date="10/03/2015"/>

# 使用 Windows PowerShell 管理基于角色的访问控制 #

> [AZURE.SELECTOR]
- [Windows PowerShell](/documentation/articles/role-based-access-control-powershell/)
- [Azure CLI](/documentation/articles/role-based-access-control-xplat-cli/)


使用 Azure 门户中基于角色的访问控制 (RBAC) 和 Azure 资源管理器 API 可以精细地管理对订阅的访问。使用此功能，可以通过在特定范围内为 Active Directory 用户、组或服务主体分配某些角色来向其授予访问权限。

在本教程中，你将学习如何使用 Windows PowerShell 来管理 RBAC。它将指导你完成创建和检查角色分配的过程。

**估计完成时间：**15 分钟

## 先决条件

在使用 Windows PowerShell 管理 RBAC 之前，必须具备以下条件：

- Windows PowerShell 3.0 版或 4.0 版。若要查找 Windows PowerShell 版本，请键入：`$PSVersionTable` 并验证 `PSVersion` 的值是 3.0 或 4.0。若要安装兼容版本，请参阅 [Windows Management Framework 3.0 ](http://www.microsoft.com/download/details.aspx?id=34595) 或 [Windows Management Framework 4.0](https://www.microsoft.com/zh-CN/download/details.aspx?id=40855)。

- Azure PowerShell 0.8.8 版或更高版本。若要安装最新版本并将其与 Azure 订阅相关联，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell/)。

本教程专为 Windows PowerShell 新手设计，但它假定您了解基本概念，如模块、cmdlet 和会话。有关 Windows PowerShell 的详细信息，请参阅 [Windows PowerShell 入门](http://technet.microsoft.com/zh-cn/library/hh857337.aspx)。

若要获得您在此教程中看到的任何 cmdlet 的详细帮助，请使用 Get-Help cmdlet。

	Get-Help <cmdlet-name> -Detailed

例如，若要获得有关 Add-AzureAccount cmdlet 的帮助，请键入：

	Get-Help Add-AzureAccount -Detailed

另请阅读以下教程以熟悉如何在 Windows PowerShell 中设置和使用 Azure 资源管理器：

- [如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell/)
- [将 Windows PowerShell 与资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager/)


## 连接到订阅

由于 RBAC 仅适用于 Azure 资源管理器，因此首先要做的是切换到 Azure 资源管理器模式，请键入：

    PS C:\> Switch-AzureMode -Name AzureResourceManager

有关详细信息，请参阅[将 Windows PowerShell 与资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager/)。

若要连接到 Azure 订阅，请键入：

    PS C:\> Add-AzureAccount

在弹出的浏览器控件中，输入你的 Azure 帐户用户名和密码。PowerShell 将使用此帐户获取你拥有的所有订阅并自己判断使用第一个订阅作为默认订阅。请注意，使用 RBAC，你只有通过成为共同管理员或具有某种角色分配才能获取你有某些权限的订阅。

如果你有多个订阅并且想要切换到另一个订阅，请键入：

    # This will show you the subscriptions under the account.
    PS C:\> Get-AzureSubscription
    # Use the subscription name to select the one you want to work on.
    PS C:\> Select-AzureSubscription -SubscriptionName <subscription name>

有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell/)。

## 检查现有的角色分配

现在让我们来看一下订阅中已存在哪些角色分配。键入：

    PS C:\> Get-AzureRoleAssignment

这将返回订阅中的所有角色分配。有两点需要注意：

1. 你需要在订阅级别具有读取访问权限。
2. 如果订阅包含大量角色分配，则可能需要一段时间才能获取所有这些角色分配。

你还可以在特定范围内针对特定角色定义检查特定用户的现有角色分配。键入：

    PS C:\> Get-AzureRoleAssignment -ResourceGroupName group1 -Mail <user email> -RoleDefinitionName Owner

这将返回你的 AD 租户中对资源组“group1”具有“所有者”角色分配的特定用户的所有角色分配。角色分配可以来自两个位置：

1. 为用户分配资源组的“所有者”角色。
2. 为用户分配资源组的父级（在本示例中为订阅）的“所有者”角色，因为如果具有某个级别的任何权限，就会对其所有子级具有相同的权限。

此 cmdlet 的所有参数都是可选的。你可以将这些参数与不同筛选器组合使用来检查角色分配。

## 创建角色分配

若要创建角色分配，需要考虑：

要将角色分配给谁：可以使用以下 Azure Active Directory cmdlet 查看你在 AD 租户中拥有哪些用户、组和服务主体。

    PS C:\> Get-AzureADUser
	PS C:\> Get-AzureADGroup
	PS C:\> Get-AzureADGroupMember
	PS C:\> Get-AzureADServicePrincipal

你要分配哪些角色：可以使用以下 cmdlet 查看支持的角色定义。

    PS C:\> Get-AzureRoleDefinition

要在哪个范围内分配：有三个级别的范围

    - The current subscription
    - A resource group, to get a list of resource groups, type `PS C:\> Get-AzureResourceGroup`
    - A resource, to get a list of resources, type `PS C:\> Get-AzureResource`

然后，使用 `New-AzureRoleAssignment` 创建角色分配。例如：


这将在当前订阅级别，为用户创建作为“读者”的角色分配。

	 PS C:\> New-AzureRoleAssignment -Mail <user email> -RoleDefinitionName Reader

这将在资源组级别创建角色分配。

	PS C:\> New-AzureRoleAssignment -Mail <user email> -RoleDefinitionName Contributor -ResourceGroupName group1

这将在资源组级别为组创建角色分配。

	PS C:\> New-AzureRoleAssignment -ObjectID <group object ID> -RoleDefinitionName Reader -ResourceGroupName group1

这将在资源级别创建角色分配。

	PS C:\> $resources = Get-AzureResource
    PS C:\> New-AzureRoleAssignment -Mail <user email> -RoleDefinitionName Owner -Scope $resources[0].ResourceId


## 验证权限

检查你的帐户具有一些角色分配后，你可以通过运行以下命令实际查看这些角色分配授予你的权限

    PS C:\> Get-AzureResourceGroup
    PS C:\> Get-AzureResource

这两个 cmdlet 将仅返回你具有读取权限的资源组或资源。并且它还会显示你具有的权限。

然后，当你尝试运行其他 cmdlet（如 `New-AzureResourceGroup`）时，如果你没有相应权限，将收到拒绝访问错误。

## 后续步骤

若要了解有关使用 Windows PowerShell 管理基于角色的访问控制及相关主题的详细信息，请执行以下操作：
 

- [Azure 资源管理器 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn708504.aspx)：了解如何在 AzureResourceManager 模块中使用这些 cmdlet。

- [Azure 博客](http://blogs.msdn.com/windowsazure)：了解 Azure 中的新功能。
- [Windows PowerShell 博客](http://blogs.msdn.com/powershell)：了解 Windows PowerShell 中的新功能。
- [“你好，脚本编写专家！” 博客](http://blogs.technet.com/b/heyscriptingguy/)：从 Windows PowerShell 社区获取实用提示和技巧。
- [使用 Azure CLI 配置基于角色的访问控制](/documentation/articles/role-based-access-control-xplat-cli/)
- [故障排除基于角色的访问控制](/documentation/articles/role-based-access-control-troubleshooting/)

<!---HONumber=71-->