<properties
	pageTitle="在 Azure Active Directory 中管理密码 | Azure"
	description="如何在 Azure Active Directory 中管理密码。"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="stevenpo"
	editor=""/>

<tags 
	ms.service="active-directory" 
	ms.date="05/16/2016"
	wacn.date="06/14/2016"/>

# 在 Azure Active Directory 中管理密码

如果你是管理员，可以通过 Azure 经典管理门户中的 Azure Active Directory (Azure AD) 重置用户密码。单击你的目录名称，在“用户”页上单击相应的用户名，然后在门户底部单击“重置密码”。

本主题的余下部分将介绍 Azure AD 支持的整套密码管理功能，包括：

- **自助密码更改**：最终用户或管理员可以更改过期的或未过期的密码，而无需请求管理员或支持人员提供支持。
- **自助密码重置**：最终用户或管理员可以自行重置密码，而无需请求管理员或支持人员提供支持。自助密码重置功能需要 Azure AD 高级或基本版。有关详细信息，请参阅 [Azure Active Directory 版本](/documentation/articles/active-directory-editions/)。
- **管理员启动的密码重置**：管理员可以通过 Azure 经典管理门户重置某个最终用户的密码或其他管理员的密码。
- **密码管理活动报告**：管理员可以深入了解发生在其组织中的密码重置和注册活动。
- **密码写回**：从云管理本地密码，因此，上述所有方案都可以由联合用户或密码同步用户本人或其代表来执行。密码写回功能需要 Azure AD 高级版。有关详细信息，请参阅 [Azure Active Directory Premium 入门](/documentation/articles/active-directory-get-started-premium/)。

> [AZURE.NOTE]
Azure AD 高级版适用于使用世界范围的 Azure AD 实例的中国客户。由中国的 21Vianet 运营的 Microsoft Azure 服务目前不支持 Azure AD 高级版。有关详细信息，请在 [Azure Active Directory 论坛](https://feedback.azure.com/forums/169401-azure-active-directory/)与我们联系。

使用以下链接可跳转至你最感兴趣的文档：

- [概述：Azure AD 中的密码管理](/documentation/articles/active-directory-passwords-how-it-works/)
- [Azure AD 中的自助密码重置：如何启用、配置和测试自助密码重置](/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-their-azure-ad-passwords)
- [Azure AD 中的自助密码重置：如何根据你的需要自定义密码重置](/documentation/articles/active-directory-passwords-customize/)
- [Azure AD 中的自助密码重置：部署和管理最佳实践](/documentation/articles/active-directory-passwords-best-practices/)
- [Azure AD 中的密码管理报告：如何查看租户中的密码管理活动](/documentation/articles/active-directory-passwords-get-insights/)
- [密码写回：如何配置 Azure AD 以管理本地密码](/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-or-change-their-ad-passwords)
- [Azure AD 密码管理故障排除](/documentation/articles/active-directory-passwords-troubleshoot/)
- [Azure AD 密码管理常见问题](/documentation/articles/active-directory-passwords-faq/)

## 后续步骤

- [管理 Azure AD](/documentation/articles/active-directory-administer/)
- [在 Azure AD 中创建或编辑用户](/documentation/articles/active-directory-create-users/)
- [在 Azure AD 中管理组](/documentation/articles/active-directory-manage-groups/)


<!---HONumber=Mooncake_0620_2016-->