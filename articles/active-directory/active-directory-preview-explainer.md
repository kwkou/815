<properties
	pageTitle="Azure Active Directory 预览版说明 | Azure"
	description="本主题介绍经典管理门户中的 Azure Active Directory 与 Azure 门户中的 Azure Active Directory 预览版之间的差异。"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/12/2016"
	ms.author="curtand"
   wacn.date="10/11/2016"/>
   	wacn.date="10/11/2016"/>


# Azure 门户中的 Azure Active Directory 管理体验预览

Azure 门户中的 Azure Active Directory (Azure AD) 管理体验目前处于预览状态。以目录的全局管理员身份登录 [Azure 门户](https://portal.azure.cn)即可试用它。然后，在服务列表中选择"Azure Active Directory"（如果可见），或者选择"更多服务"以查看所有服务的列表。无需 Azure 订阅即可使用 Azure 门户中的 Azure AD 管理体验。


## 预览版体验的功能

预览版体验可让用户在 Azure 门户中管理多个目录资源（例如用户、组和应用程序），以及目录设置。我们正在改进此体验，使其包括 [Azure 经典管理门户](https://manage.windowsazure.cn)的 Azure AD 管理体验中存在的所有功能。在此之前，有一些目录管理任务仍必须在经典管理门户中完成。

## 管理相同的 Azure AD 租户

预览版体验与经典管理门户和 Office 365 管理中心读取和写入到相同的 Azure Active Directory 租户。在这些门户中的任一门户所做的更改都将反映到所有其他门户。

## 使用相同的授权逻辑

预览版体验与现有的 Active Directory 客户端使用相同的授权逻辑。用户根据其目录角色（例如，全局管理员、用户管理员、密码管理员）有权对目录资源进行更改。具有 Azure 资源或 Azure 订阅中的角色并未授取用户管理目录资源。有关 Azure AD 管理角色的详细信息，请参阅[在 Azure Active Directory 中分配管理员角色](/documentation/articles/active-directory-assign-admin-roles/)。

预览版体验已针对全局管理员进行了优化。如果使用预览版体验时未以全局管理员用户身份登录，可能具有降级的体验。例如，可能能够在目录中选择用于开始某个任务的按钮，但却无法完成该任务。我们不久将改进此体验。
 
## 请将想法告诉我们

可以在 [Azure AD 反馈论坛](https://social.msdn.microsoft.com/Forums/home?forum=WindowsAzureAD&filter=alltypes&sort=lastpostdesc)的管理门户部分提供有关预览版体验的反馈。

<!---HONumber=Mooncake_0926_2016-->
