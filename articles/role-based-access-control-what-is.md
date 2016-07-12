<properties
	pageTitle="Azure Active Directory 基于角色的访问控制| Azure"
	description="Azure 门户中具有 Azure 基于角色的访问控制的访问管理入门。在目录中使用角色分配来分配权限。"
	services="active-directory"
	documentationCenter=""
	authors="kgremban"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/03/2016"
	wacn.date="07/05/2016"/>

# Azure 门户中的访问管理入门

Azure 基于角色的访问控制 (RBAC) 可用于对 Azure 进行细致的访问管理。使用 RBAC，你可以在开发运营团队中对职责进行分配，仅向用户授予执行作业所需的访问权限。本文介绍了访问管理的基础知识，并且帮助你在 Azure 门户启动和运行 RBAC。

## Azure 中的访问管理的基础知识
每个 Azure 订阅都与一个 Azure Active Directory (AD) 目录相关联。该目录中的用户、组和应用程序可以管理 Azure 订阅中的资源。可以通过使用 Azure 门户、Azure 命令行工具或 Azure 管理 API 授予这些访问权限。

访问权限是通过将相应的 RBAC 角色分配给特定范围内的用户、组和应用程序来授予的。角色分配的范围可以是订阅、资源组或单个资源。分配在父范围内的角色也会将访问权限授予给其中所含的子范围。例如，具有对资源组访问权限的用户可以管理其包含的所有资源，如网站、虚拟机和子网。

![Azure Active Directory 元素之间的关系 - 关系图](./media/role-based-access-control-what-is/rbac_aad.png)

你分配的 RBAC 角色决定了用户、组或应用程序可以在该范围内所管理的资源。

## 内置角色
Azure RBAC 有三种适用于所有资源类型的基本角色：

- **所有者**具有对所有资源的完全访问权限，包括委派对其他用户的访问权限。
- **参与者**可以创建和管理所有类型的 Azure 资源，但不能将访问权限授予其他用户。
- **读者**可以查看现有的 Azure 资源。

Azure 中的其他 RBAC 角色允许对特定的 Azure 资源进行管理。例如，虚拟机参与者角色允许用户创建和管理虚拟机。它并不授予其访问虚拟机连接的虚拟网络或子网的权限。

[RBAC 内置角色](/documentation/articles/role-based-access-built-in-roles/)列出了 Azure 中可用的角色。它指定每个内置角色向用户授予的操作和范围。若要定义自己的角色以便进一步控制，请参阅如何生成 [Azure RBAC 中的自定义角色](/documentation/articles/role-based-access-control-custom-roles/)。

## 资源层次结构和访问权限继承
- Azure 中的每个**订阅**仅属于一个目录。
- 每个**资源组**仅属于一个订阅。
- 每个**资源**仅属于一个资源组。

你在父范围授予的访问权限在子范围被继承。例如：

- 将读者角色分配给订阅范围内的 Azure AD 组。该组的成员可以查看订阅中的每个资源组和资源。
- 将参与者角色分配给资源组范围内的应用程序。它可以管理该资源组中所有类型的资源，但不能管理订阅中的其他资源组。

## Azure RBAC 与经典订阅管理员
经典订阅管理员和共同管理员具有对 Azure 订阅的完全访问权限。他们可以将 [Azure 门户](https://portal.azure.cn)和 Azure Resource Manager API 配合使用或使用 [Azure 经典门户](https://manage.windowsazure.cn) 和 Azure 服务管理 API 来管理资源。在 RBAC 模型中，经典管理员具有订阅范围内的所有者角色。

仅 Azure 门户和新的 Azure Resource Manager API 支持 Azure RBAC。分配了 RBAC 角色的用户和应用程序不能使用经典管理门户和 Azure 服务管理 API。

## 管理授权与数据操作
Azure RBAC 仅支持 Azure 门户和 Azure Resource Manager API 中的 Azure 资源的管理操作。并不是 Azure 资源的所有数据级别操作都可通过 RBAC 授权。例如，可以使用 RBAC 对存储帐户进行管理，但是不能使用 RBAC 管理存储帐户中的 blob 或表。同样，可以管理SQL 数据库，但是不能管理其中的表。

## 后续步骤
- [Azure 门户中基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)入门。
- 请参阅 [RBAC 内置角色](/documentation/articles/role-based-access-built-in-roles/)
- 定义你在 [Azure RBAC 中的自定义角色](/documentation/articles/role-based-access-control-custom-roles/)

<!---HONumber=Mooncake_0627_2016-->