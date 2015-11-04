<properties 
	pageTitle="使用条件性访问管理风险" 
	description="本主题介绍如何从符合策略的已知设备对特定资源进行随处访问，并禁止从丢失、被盗、不合规设备进行访问。" 
	services="active-directory, virtual-network" 
	documentationCenter="" 
	authors="Justinha" 
	manager="TerryLan" 
	editor="LisaToft"/>

<tags 
	ms.service="active-directory"  
	ms.date="05/05/2015"
	wacn.date="06/16/2015"/>


# 使用条件性访问管理风险

现在，员工们在工作方式、提高工作效率的方式以及工作手段等方面正经历快速改变。员工们将其个人设备带到了工作中。在这些个人设备上，安装了个人数字化生活所用的应用程序。他们习惯了这样的自由，以及由此而来的各种便利。他们想要在工作中使用这些相同的应用程序，他们希望在工作时能够像平时生活一样灵活地处理各种数字化任务。如今的企业员工希望工作场所不受限制，希望自己选择工作设备，并且希望无缝地连接和访问业务应用程序。

允许用户选择工作地点、工作时间和工作方式虽然提高了灵活性，但与这种灵活性相伴随的，是风险的增加。大量指向这些设备的数据点被盗用、错放或丢失。许多这样的智能手机和平板电脑上存放着大量敏感性的和机密的客户内容和企业内容。这就是 IT 架构师、安全专家和 IT 管理员尝试维持的一种微妙的平衡。这种平衡就是，一方面，需要让用户在自己喜欢的设备上（在所有喜欢的设备上）工作以提高工作效率，另一方面，需要为驻留在这些设备上的公司内容提供适当水平的安全性和保护措施。

使用通过 Azure Active Directory、Office 365 和 Microsoft Intune 提供的多个条件性访问功能，IT 管理员可以解决以下问题：

- 只允许员工进行随处访问，而不是为 Internet 上的每个人都打开大门。
- 允许对已知合规设备上的特定资源进行随处访问。
- 禁止从丢失、被盗、不合规设备进行访问。

![][1]

## 后续步骤

以下主题讨论在设置组织中的条件性访问策略时可以使用的每个不同的机制。

- [Azure Active Directory Device Registration 概述](/documentation/articles/active-directory-conditional-access-device-registration-overview)
- [使用 Azure Active Directory Device Registration 设置本地条件性访问](/documentation/articles/active-directory-conditional-access-on-premises-setup)
- [Office 365 服务的条件性访问设备策略](/documentation/articles/active-directory-conditional-access-device-policies)
- [SaaS 应用程序的 Azure 条件性访问预览](/documentation/articles/active-directory-conditional-access-azuread-connected-apps)


<!--Image references--> 

[1]: ./media/active-directory-conditional-access/condaccoverviewvsdx1.png

<!---HONumber=60-->