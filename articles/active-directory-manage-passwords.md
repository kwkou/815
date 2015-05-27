<properties 
	pageTitle="在 Azure AD 中管理密码" 
	description="本主题介绍如何在 Azure AD 中管理密码。" 
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

# 在 Azure AD 中管理密码

如果你是管理员，可以通过 Azure 经典门户重置用户在 Azure 中的密码。单击你的目录名称，在"用户"页上单击相应的用户名，然后在门户底部单击"重置密码"。

本主题的余下部分将介绍 Azure Active Directory 支持的整套密码管理功能，包括：

- **自助密码更改**：可让最终用户或管理员就可以更改过期的或未过期的密码，而无需请求管理员或帮助台提供支持。
- **自助密码重置**：最终用户或管理员可以自行重置密码，而无需请求管理员或帮助台提供支持。自助密码重置功能需要 Azure AD 高级或基本版。有关详细信息，请参阅 [Azure Active Directory 版本](active-directory-editions)。
- **管理员启动的密码重置**：管理员可以通过 Azure 管理门户重置某个最终用户的或其他管理员的密码。
- **密码管理活动报告**：管理员可以深入了解发生在其组织中的密码重置和注册活动。
- **密码写回**：从云管理本地密码，因此，所有上述方案都可以由经过联合身份验证的或密码同步的用户本人或其代表来执行。密码写回功能需要 Azure AD 高级版。 

<!--For more information, see [Getting started with Azure Active Directory Premium](active-directory-get-started-premium).-->

> [AZURE.NOTE] 
> Azure AD 高级版适用于使用世界范围的 Azure AD 实例的中国客户。由中国的 21Vianet 运营的 Microsoft Azure 服务目前不支持 Azure AD 高级版。有关详细信息，请在 [Azure Active Directory 论坛](http://feedback.azure.com/forums/169401-azure-active-directory)与我们联系。 

使用以下链接可跳转至你最感兴趣的文档：

- [概述：Azure AD 中的密码管理](https://msdn.microsoft.com/zh-cn/library/azure/dn683880.aspx)
- [Azure AD 中的自助密码重置：如何启用、配置和测试自助密码重置](https://msdn.microsoft.com/zh-cn/library/azure/dn683881.aspx)
- [Azure AD 中的自助密码重置：如何根据你的需要自定义密码重置](https://msdn.microsoft.com/zh-cn/library/azure/dn688249.aspx)
- [Azure AD 中的自助密码重置：部署和管理最佳实践](https://msdn.microsoft.com/zh-cn/library/azure/dn903643.aspx)
- [Azure AD 中的密码管理报告：如何查看租户中的密码管理活动](https://msdn.microsoft.com/zh-cn/library/azure/dn903641.aspx)
- [密码写回：如何配置 Azure AD 以管理本地密码](https://msdn.microsoft.com/zh-cn/library/azure/dn903642.aspx)
- [Azure AD 密码管理常见问题/疑难解答](https://msdn.microsoft.com/zh-cn/library/azure/dn683878.aspx)

## 后续步骤

- [管理 Azure AD](active-directory-administer)
- [在 Azure AD 中创建或编辑用户](active-directory-create-users)
- [在 Azure AD 中管理组](active-directory-manage-groups)

<!--HONumber=57-->