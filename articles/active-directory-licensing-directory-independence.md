<properties
   pageTitle="添加和管理多个 Azure AD 目录 | Windows Azure"
   description="添加和管理 Azure AD 目录的说明和最佳实践，将目录解释为完全独立的资源"
   services="active-directory"
   documentationCenter=""
   authors="curtand"
   manager="swadhwa"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="07/16/2015"
   wacn.date="08/29/2015"/>

# 添加和管理多个 Azure AD 目录

在 Azure AD 目录中，每个目录都是完全独立的资源，即功能齐全且在逻辑上独立于你所管理的其他目录的对等资源。目录之间没有任何父子关系。目录之间的这种独立性包括资源独立性、管理独立性和同步独立性。

##资源独立性

如果你创建或删除某个目录中的资源，这将不会影响其他目录中的任何资源，但对外部用户而言具有部分例外，如下所述。如果你在某个目录中使用自定义域“contoso.com”，则无法在任何其他目录中使用它。

##管理独立性

如果目录“Contoso”的一个非管理用户创建了测试目录“Test”，则在默认情况下，创建目录的用户将作为外部用户添加到新的目录中，并在此目录中被分配全局管理员角色。目录“Contoso”的管理员对目录“Test”没有直接管理权限，除非“Test”的管理员专门向其授予这些权限。如果“Contoso”的管理员控制创建了“Test”的用户帐户，则可控制目录“Test”的访问权限。如果你更改（添加或删除）某个目录中用户的管理员角色，则此更改不会影响此用户可能在其他目录中拥有的任何管理员角色。

##同步独立性

你可以独立配置每个 Azure AD 目录以从以下两个渠道的单个实例实现数据同步：通过目录同步 (DirSync) 工具使用单个 AD 林同步数据，或者通过适用于 Forefront Identity Manager 的 Azure Active Directory Connector 使用一个或多个本地林和/或非 Azure AD 数据源来同步数据。

##添加 Azure AD 目录

若要在 Azure 管理门户中添加 Azure AD 目录，请选择左侧的 Active Directory 扩展，然后点击“添加”。

> [AZURE.NOTE]与其他 Azure 资源不同，你的目录不是 Azure 订阅的子资源。如果你取消 Azure 订阅或任由其过期，则仍可使用 Azure PowerShell、Azure 图形 API 或 Office 365 管理中心等其他界面来访问你的目录数据。你还可以将其他订阅与此目录相关联。

有关 Azure AD 许可问题的概述和最佳实践，请参阅[什么是 Azure Active Directory 许可？](/documentation/articles/active-directory-licensing-what-is)。

<!---HONumber=67-->