<properties
	pageTitle="使用 Azure 命令行界面管理基于角色的访问控制"
	description="使用 Azure 命令行界面管理基于角色的访问控制"
	services="azure-portal"
	documentationCenter="na"
	authors="IHenkel"
	manager="stevenpo"
	editor=""/>

<tags 
	ms.service="azure-portal" 
	ms.date="08/14/2015"
	wacn.date="10/03/2015"/>

# 使用 Azure 命令行界面 (Azure CLI) 管理基于角色的访问控制 #

> [AZURE.SELECTOR]
- [Windows PowerShell](/documentation/articles/role-based-access-control-powershell/)
- [Azure CLI](/documentation/articles/role-based-access-control-xplat-cli/)

使用 Azure 门户预览中基于角色的访问控制 (RBAC) 和 Azure 资源管理器 API 可以精细地管理对订阅和资源的访问。使用此功能，可以通过在特定范围内为 Active Directory 用户、组或服务主体分配某些角色来向其授予访问权限。

在本教程中，你将学习如何使用 Azure CLI 来管理基于角色的访问控制。它将指导你完成创建和检查角色分配的过程。

**估计完成时间：**15 分钟

## 先决条件 ##

在使用 Azure CLI 管理 RBAC 之前，必须具备以下条件：

- Azure CLI 0.8.8 版或更高。若要安装最新版本并将其与 Azure 订阅相关联，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)。


## 本教程的内容 ##

* [连接到订阅](#connect)
* [检查现有的角色分配](#check)
* [创建角色分配](#create)
* [验证权限](#verify)
* [后续步骤](#next)

## <a id="connect"></a>连接到订阅 ##

由于 RBAC 仅适用于 Azure 资源管理器，因此首先要做的是切换到 Azure 资源管理器模式。键入：

    azure config mode arm

若要连接到 Azure 订阅，请键入：

    azure login -e AzureChinaCloud -u <username> 

在命令行提示符下，输入 Azure 帐户密码（仅使用组织帐户）。Azure CLI 将使用此帐户获取你拥有的所有订阅并将自己配置为使用第一个订阅作为默认订阅。请注意，使用基于角色的访问控制，你只有通过成为共同管理员或具有某种角色分配才能获取你有某些权限的订阅。

如果你有多个订阅并且想要切换到另一个订阅，请键入：

    # This will show you the subscriptions under the account.
    azure account list
    # Use the subscription name to select the one you want to work on.
    azure account set <subscription name>

有关详细信息，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)。

## <a id="check"></a>检查现有的角色分配 ##

现在让我们来看一下订阅中已存在哪些角色分配。键入：

    azure role assignment list

这将返回订阅中的所有角色分配。有两点需要注意：

1. 你需要在订阅级别具有读取访问权限。
2. 如果订阅包含大量角色分配，则可能需要一段时间才能获取所有这些角色分配。

你还可以在特定范围内针对特定角色定义检查特定用户的现有角色分配。键入：

    azure role assignment list -g group1 --upn <user email> -o Owner

这将返回你的 Azure AD 目录中对资源组“group1”具有“所有者”角色分配的特定用户的所有角色分配。角色分配可以来自两个位置：

1. 为用户分配资源组的“所有者”角色。
2. 为用户分配资源组的父级（在本示例中为订阅）的“所有者”角色，因为如果你对某种父资源具有任何权限，就会对其所有子资源具有相同的权限。

此 cmdlet 的所有参数都是可选的。你可以将这些参数与不同筛选器组合使用来检查角色分配。

## <a id="create"></a>创建角色分配 ##

若要创建角色分配，需要考虑：

- 要将角色分配给谁：可以使用以下 Azure Active Directory cmdlet 查看你在目录中拥有哪些用户、组和服务主体。

	    
	    azure ad user list  
	    azure ad user show  
	    azure ad group list  
	    azure ad group show  
	    azure ad group member list  
	    azure ad sp list  
	    azure ad sp show  
    

- 你要分配哪些角色：可以使用以下 cmdlet 查看支持的角色定义。

    `azure role list`

- 要在哪个范围内分配：有三个级别的范围：

    - 当前订阅
    - 资源组。若要获取资源组的列表，请键入 `azure group list`
    - 资源。若要获取资源的列表，请键入 `azure resource list`

然后，使用 `azure role assignment create` 创建角色分配。例如：

 - 这将在当前订阅级别，为用户创建作为“读者”的角色分配：

    `azure role assignment create --upn <user's email> -o Reader`

- 这将在资源组级别创建角色分配：

    `PS C:\> azure role assignment create --upn <user's email> -o Contributor -g group1`

- 这将在资源级别创建角色分配：

    `azure role assignment create --upn <user's email> -o Owner -g group1 -r Microsoft.Web/sites -u site1`

## <a id="verify"></a>验证权限 ##

检查你的帐户具有一些角色分配后，你可以通过运行以下命令实际查看这些角色分配授予你的权限：
	
	    PS C:\> azure group list
	    PS C:\> azure resource list

这两个 cmdlet 将仅返回你具有读取权限的资源组或资源。并且它还会显示你具有的权限。

然后，当你尝试运行其他 cmdlet（如 `azure group create`）时，如果你没有相应权限，将收到拒绝访问错误。

## <a id="next"></a>后续步骤 ##

若要了解有关使用 Azure CLI 管理基于角色的访问控制及相关主题的详细信息，请执行以下操作：


- [安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)


- [Azure 博客](http://blogs.msdn.com/windowsazure)：了解 Azure 中的新功能。
- [使用 Windows PowerShell 配置基于角色的访问控制](/documentation/articles/role-based-access-control-powershell/)
- [故障排除基于角色的访问控制](/documentation/articles/role-based-access-control-troubleshooting/)

<!---HONumber=71-->