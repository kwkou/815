<properties 
    pageTitle="如何在 Azure Active Directory 中调试对应用程序进行基于 SAML 的单一登录 | Microsoft Azure" 
    description="了解如何在 Azure Active Directory 中调试对应用程序进行基于 SAML 的单一登录" 
    services="active-directory" 
    authors="asmalser-msft"  
    documentationCenter="na" manager="stevenpo"/>
<tags 
    ms.service="active-directory" 
    ms.date="02/09/2016" 
    wacn.date="06/27/2016" />

#如何在 Azure Active Directory 中调试对应用程序进行基于 SAML 的单一登录

在调试基于 SAML 的应用程序集成时，使用 [Fiddler](http://www.telerik.com/fiddler) 之类的工具查看 SAML 请求、SAML 响应和颁发给应用程序的实际 SAML 令牌通常很有帮助。通过检查 SAML 令牌，可以确保按预期传递所有所需的属性、SAML 主题中的用户名和颁发者 URI。

![][1]

包含 SAML 令牌的 Azure AD 响应通常是在从 [https://login.chinacloudapi.cn](https://login.chinacloudapi.cn) 发出 HTTP 302 重定向之后发生的响应，它将发送到应用程序的已配置**回复 URL**。
 
你可以通过选择此行，然后在右窗格中选择“检查器”>“WebForms”，来查看 SAML 令牌。右键单击“SAMLResponse”值并选择“发送到 TextWizard”。然后在“转换”菜单中选择“从 Base64”以解码令牌并查看其内容。
 
**注意**：当你查看此 HTTP 请求的内容时，Fiddler 可能会提示你配置 HTTPS 流量解密，此时你需要执行此操作。

<!--## 相关文章

- [有关 Azure Active Directory 中应用程序管理的文章索引](/documentation/articles/active-directory-apps-index/)
- [如何为预先集成的应用程序自定义 SAML 令牌中颁发的声明](/documentation/articles/active-directory-saml-claims-customization/)
-->
<!--Image references-->
[1]: ./media/active-directory-saml-debugging/fiddler.png
<!---HONumber=Mooncake_0620_2016-->