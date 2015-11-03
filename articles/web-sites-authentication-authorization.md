<properties 
	pageTitle="使用 Active Directory 在 Azure 网站中进行身份验证" 
	description="了解部署到 Azure 网站的业务线应用程序的不同身份验证和授权选项" 
	services="app-service\web" 
	documentationCenter="" 
	authors="cephalin" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-web" 
	ms.date="07/02/2015" 
	wacn.date="10/03/2015"/>

# 使用 Active Directory 在 Azure 网站中进行身份验证 #

[Azure 网站](/documentation/services/web-sites)通过支持单一登录 (SSO) 的用户启用企业业务线应用程序方案，允许你在本地环境或公共 Internet 访问应用程序。可以将它与 [Azure Active Directory](/documentation/services/identity) (AAD) 或本地安全令牌服务 (STS)（如 Active Directory 联合身份验证服务 (AD FS)）集成，以便对内部 Active Directory (AD) 用户进行身份验证并正确授权。

## 零摩擦身份验证和授权 ##

单击几下按钮，就可以为 Web 应用启用身份验证和授权。每个 Azure Web 应用中的复选框样式配置都提供了业务线 Web 应用的基本访问控制。它在授予用户对 Web 应用内容的访问权限之前，会强制对所选的 Azure AD 租户进行 HTTPS 通信和身份验证。有关详细信息，请参阅 [Web Apps 身份验证/授权](http://azure.microsoft.com/blog/2014/11/13/azure-websites-authentication-authorization/)。

>[AZURE.NOTE]此功能目前处于预览状态。

## 手动实现身份验证和授权 ##

在许多情况下，你想要自定义应用程序的身份验证和授权行为，例如，登录和注销页、自定义授权逻辑、多租户应用程序行为，等等。在这种情况下，最好是手动配置身份验证和授权以提高功能的灵活性。以下是两个主要选项

-	[Azure AD](/documentation/articles/web-sites-dotnet-lob-application-azure-ad) - 可以使用 Azure AD 为网站实施身份验证和授权。使用 Azure AD 作为标识提供程序具有以下特征：
	-	支持常用的身份验证协议，如 [OAuth 2.0](http://oauth.net/2/)、[OpenID Connect](http://openid.net/connect/) 和 [SAML 2.0](http://en.wikipedia.org/wiki/SAML_2.0)。有关支持的协议的完整列表，请参阅 [Azure Active Directory 身份验证协议](http://msdn.microsoft.com/zh-cn/library/azure/dn151124.aspx)。
	-	可以使用没有任何本地基础结构的仅限 Azure 的标识提供者。
	-	此外可以使用本地 AD（托管在本地）配置目录同步。
	-	当 AD 用户从 Intranet 和 Internet 访问时，Azure AD 将从本地 AD 域进行目录同步，实现顺畅的 SSO 体验。AD 用户可从 Intranet 自动通过集成身份验证来访问网站。AD 用户可使用其 Windows 凭据从 Internet 登录网站。
	-	将 SSO 提供给 Azure AD 支持的所有应用程序，包括 Azure、Office 365、Dynamics CRM Online、Windows InTune 和数千个非 Microsoft 云应用程序。 
	-	Azure AD 将[信赖方](http://en.wikipedia.org/wiki/Relying_party)应用程序管理委派到非管理员角色，但是，对敏感目录数据的应用程序访问仍须由全局管理员配置。
	-	为所有信赖方应用程序发送一组通用声明类型。有关声明类型的列表，请参阅[支持的令牌和声明类型](/documentation/articles/active-directory-token-and-claims)。声明不可自定义。
	-	通过使用 [Azure AD 图形 API](http://msdn.microsoft.com/zh-cn/library/azure/hh974476.aspx)，应用程序能够访问 Azure AD 中的目录数据。
-	[本地安全令牌服务 (STS)，例如 AD FS](/documentation/articles/web-sites-dotnet-lob-application-adfs) - 你可以使用本地 STS（如 AD FS）对 Web 应用实施身份验证和授权。本地 AD FS 的使用具有以下特征：
	-	AD FS 拓扑必须在本地部署，会产生成本和管理开销。
	-	当公司政策要求 AD 数据存储在本地时效果最佳。
	-	只有 AD FS 管理员可以配置[信赖方信任和声明规则](http://technet.microsoft.com/zh-cn/library/dd807108.aspx)。
	-	可以基于按应用程序管理[声明](http://technet.microsoft.com/zh-cn/library/ee913571.aspx)。
	-	必须提供单独的解决方案，用于通过公司防火墙访问本地 AD 数据。



<!---HONumber=71-->