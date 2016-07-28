<properties
	pageTitle="Azure Active Directory 常见问题 | Azure"
	description="Azure Active Directory 常见问题，其中提供了有关访问 Azure 和 Azure Active Directory、密码管理和应用程序访问的问题的解答。"
	services="active-directory"
	documentationCenter=""
	authors="markusvi"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/20/2016"
	wacn.date="07/26/2016"/>

# Azure Active Directory 常见问题

Azure Active Directory 是综合性的标识即服务 (IDaaS) 解决方案，其涉及到标识、访问管理和安全的方方面面。


有关详细信息，请参阅 [What is Azure Active Directory?（什么是 Azure Active Directory？）](/documentation/articles/active-directory-whatis/)。

## 访问 Azure 和 Azure Active Directory


**问：当我尝试在 Azure 经典门户中访问 Azure AD 时，为何收到“找不到订阅”(https://manage.windowsazure.cn)？**

**答：**若要访问 Azure 经典门户，每位用户必须拥有 Azure 订阅的权限。请导航到 [我要购买](/pricing/pia/) 完成一次性的激活步骤，或者注册 [Azure 试用版](/pricing/1rmb-trial/)。

有关详细信息，请参阅：

- [Azure 订阅与 Azure Active Directory 的关联方式](/documentation/articles/active-directory-how-subscriptions-associated-directory/)

- [在 Azure 中管理 Office 365 订阅的目录](/documentation/articles/active-directory-manage-o365-subscription/)

---

**问：Azure AD、Office 365 与 Azure 之间有哪种关系？**

**答：**Azure Active Directory 为所有 Microsoft 在线服务提供通用的标识和访问功能。无论你使用 Office 365、Microsoft Azure、Intune 还是其他服务，都可以使用 Azure AD 来为上述所有服务启用登录和访问管理。

事实上，为 Microsoft 在线服务启用的所有用户都定义为一个或多个 Azure AD 实例中的用户帐户。你可以为这些帐户启用免费的 Azure AD 功能，例如云应用程序访问。
 
此外，Azure AD 付费版服务（例如 Azure AD Basic、Premium、EMS 等）可通过综合性的企业级管理和安全解决方案来弥补其他在线服务（例如 Office 365 和 Microsoft Azure）的不足。


---

## 混合 Azure AD 入门


**问：如何将我的本地目录连接到 Azure AD？**

**答：**可以使用 **Azure AD Connect** 将本地目录连接到 Azure AD。

有关详细信息，请参阅 [Integrating your on-premises identities with Azure Active Directory（将本地标识与 Azure Active Directory 集成）](/documentation/articles/active-directory-aadconnect/)。


---

**问：如何设置本地目录与云应用程序之间的 SSO？**

**答：**只需在本地目录与 Azure AD 之间设置 SSO。只要通过 Azure AD 访问云应用程序，该服务就会自动让用户使用其本地凭据正确进行身份验证。

通过联合身份验证解决方案（例如 ADFS）或通过配置密码哈希同步，就能轻松地从本地实现 SSO。可以使用 Azure AD Connect 配置向导轻松部署这两个选项。
  

有关详细信息，请参阅 [Integrating your on-premises identities with Azure Active Directory（将本地标识与 Azure Active Directory 集成）](/documentation/articles/active-directory-aadconnect/)。
  

---

**问：Azure Active Directory 是否为组织中的用户提供自助服务门户？**

**答：**是的，Azure Active Directory 提供 [Azure AD 访问面板](http://myapps.microsoft.com)，方便用户进行自助服务和应用程序访问。如果你是 Office 365 客户，可以在 Office 365 门户中找到许多相同的功能。

有关详细信息，请参阅 [Introduction to the Access Panel（访问面板简介）](/documentation/articles/active-directory-saas-access-panel-introduction/)。



---

**问：Azure AD 是否可以帮助我管理本地基础结构？**

**答：**可以。Azure AD Premium Edition 提供 **Connect Health**。Azure AD Connect Health 可帮助你监视和深入了解本地标识基础结构和同步服务。

有关详细信息，请参阅 [Monitor your on-premises identity infrastructure and synchronization services in the cloud（在云中监视本地标识基础结构和同步服务）](/documentation/articles/active-directory-aadconnect-health)。

---

## 密码管理

**问：是否可以使用 Azure AD 密码写回但不使用密码同步？ （也就是说，我想要使用 Azure AD SSPR 和密码写回，但不想将密码存储在云中。）**

**答：**无需将 AD 密码同步到 Azure AD 即可启用写回。在联合环境中，Azure AD SSO 依赖本地目录来对用户进行身份验证。在这种情况下，并不需要在 Azure AD 中跟踪本地密码。

---

**问：需要多长时间才能将密码写回到 AD 本地？**

**答：**密码写回实时运行。

有关详细信息，请参阅 [Getting started with Password Management（密码管理入门）](/documentation/articles/active-directory-passwords-getting-started/)。


---

**问：是否可以对管理员管理的密码使用密码写回？**

**答：**可以。如果已启用密码写回，管理员执行的密码操作将写回到你的本地环境。

有关密码相关问题的详细解答，请参阅 [Password Management Frequently Asked Questions（密码管理常见问题）](/documentation/articles/active-directory-passwords-faq/)。

---


**问：用户如何使用 Azure Active Directory 来登录应用程序？**
 
**答：**Azure Active Directory 提供多种方式供用户查看和访问其应用程序，例如：

- Azure AD 访问面板

- Office 365 应用程序启动器

- 直接登录联合应用

- 联合、基于密码或现有的应用的深层链接

有关详细信息，请参阅 [Deploying Azure AD integrated applications to users（为用户部署 Azure AD 集成的应用程序）](/documentation/articles/active-directory-appssoaccess-whatis/#deploying-azure-ad-integrated-applications-to-users)。


---

**问：Azure Active Directory 可通过哪些不同的方式来启用对应用程序的身份验证和单一登录？**
 
**答：**Azure Active Directory 支持使用多种标准化协议（例如 SAML 2.0、OpenID Connect、OAuth 2.0 和 WS-Federation）进行身份验证和授权。对于仅支持基于窗体的身份验证的应用程序，Azure AD 还支持密码保管和自动化登录功能。

有关详细信息，请参阅：

- [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)

- [Active Directory 身份验证协议](https://msdn.microsoft.com/library/azure/dn151124.aspx)

- [Azure Active Directory 中单一登录的工作原理是什么？](/documentation/articles/active-directory-appssoaccess-whatis/#how-does-single-sign-on-with-azure-active-directory-work)



<!---HONumber=AcomDC_0718_2016-->