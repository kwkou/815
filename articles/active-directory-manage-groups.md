<properties 
	pageTitle="在 Azure AD 中管理组" 
	description="本主题介绍如何在 Azure AD 中管理组。" 
	services="active-directory" 
	documentationCenter="" 
	authors="Justinha" 
	manager="TerryLan" 
	editor="LisaToft"
	tags="azure-classic-portal"/>

<tags 
	ms.service="active-directory" 
	ms.workload="infrastructure-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/27/2015" 
	wacn.date="05/26/2015"
	ms.author="Justinha"/>

# 在 Azure AD 中管理组

组是可作为单个单元管理的用户和组的集合。属于特定组的用户和组称为组成员。使用组可以一次性向许多帐户分配一组通用的权限，而不必单独向每个帐户分配权限，从而简化了管理。

目前，在 Azure AD 中，你只能创建安全组。 

如果需要将许多用户分配到同一个应用程序，使用组可以方便地为用户分配对应用程序的访问权限。在配置用于控制资源访问权限的其他联机服务（例如 SharePoint Online）的访问管理时，也可以使用组。

如果已配置目录同步，可以查看已从本地 Windows Server Active Directory 同步的组，这些组的"源自"属性中的值为"本地 Active Directory"。必须继续在本地 Active Directory 中管理这些组；不能在 Azure 管理门户中管理或删除这些组。 

如果你安装了 Office 365，则可以看到在 Office 365 中的 Exchange 管理中心内创建和管理的分发组及支持邮件的安全组。这些组的"源自"属性中的值为"Office 365"，你必须继续在 Exchange 管理中心管理这些组。 

你还可以通过访问面板创建统一组。在"配置"选项卡的"组管理"下，将"用户可以创建 O365 组"小组件设为"是"。如果你有在访问面板或 Office 365 中创建的 Office 365 统一组，则这些组的"源自"属性将设为"Azure Active Directory"，并且可以通过访问面板管理这些组。

如果已经为用户启用了自助组管理（有关详细信息，请参阅"针对 Azure AD 中的用户进行自助组管理"），作为租户管理员，你还可以通过 Azure 管理门户管理这些组。你可以添加和删除组成员、添加和删除组所有者、编辑组属性以及查看组的历史记录活动报告，其中显示了在组中执行的操作，以及何时执行了该操作。

> [AZURE.NOTE]
> 若要将组分配到应用程序，必须使用 Azure AD 高级版。如果你安装了 Azure AD 高级版，则还可以使用组来分配对集成到 Azure AD 的 SaaS 应用程序的访问权限。有关详细信息，请参阅"在 Azure AD 中为组分配对 SaaS 应用程序的访问权限"。

## 后续步骤

- [管理 Azure AD](active-directory-administer)
- [在 Azure AD 中创建或编辑用户](active-directory-create-users)
- [在 Azure AD 中管理密码](active-directory-manage-passwords)

<!--HONumber=57-->