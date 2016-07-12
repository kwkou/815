<properties
	pageTitle="应用模型 v2.0 概述 | Azure"
	description="使用 Microsoft 帐户和 Azure Active Directory 登录的构建应用简介。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="04/26/2016"
	wacn.date="06/23/2016"/>

# 在单个应用中登录 Microsoft 帐户和 Azure AD 用户

在过去，想要支持 Microsoft 帐户和 Azure Active Directory 的应用开发人员需要集成两个单独的系统。我们现在推出了新的身份验证 API 版本，可让你通过 Azure AD 系统使用这两种帐户类型来登录用户。这种聚合式身份验证系统称为 **v2.0 终结点**。借助 v2.0 终结点，通过一个简单的集成就可以与具有个人和工作/学校帐户的数以百万计的用户受众进行沟通。

使用 v2.0 终结点的应用还可以通过其中一种帐户从 [Microsoft Graph](https://graph.microsoft.io) 和 [Office 365](https://msdn.microsoft.com/office/office365/howto/authenticate-Office-365-APIs-using-v2) 使用 REST API。

## 入门
从下面选择你偏爱的平台，以使用我们的开源库与框架来生成应用。或者，你可以使用我们的 OAuth 2.0 和 OpenID Connect 协议文档来直接发送和接收协议消息，而不必使用身份验证库。
<!-- TODO: Finalize this table  -->

[AZURE.INCLUDE [active-directory-v2-quickstart-table](../includes/active-directory-v2-quickstart-table.md)]

## 新增功能
此处提供的概念性信息可帮助你了解 v2.0 终结点的定义及其功能。

- 如果你在 v2.0 终结点 2015 预览期生成应用，请务必[阅读我们最近所做的重大协议更改](/documentation/articles/active-directory-v2-preview-oidc-changes/)。
- 了解[使用 v2.0 终结点可以生成哪种类型的应用](/documentation/articles/active-directory-v2-flows/)。
- 对于熟悉 Azure Active Directory 的开发人员，则应查看[协议更新和 v2.0 终结点的差异](/documentation/articles/active-directory-v2-compare/)。
- 了解 v2.0 终结点的[限制、局限性和约束](/documentation/articles/active-directory-v2-limitations/)。

## 引用
这些链接有助于深入地利用平台：

- Build 2016：[Getting Started with Microsoft Identities: Enterprise Grade Sign In For Your Apps（开始使用 Microsoft 标识：应用的企业级登录）](https://azure.microsoft.com/documentation/videos/build-2016-getting-started-with-microsoft-identities-enterprise-grade-sign-in-for-your-apps/)
- 使用 [azure-active-directory](http://stackoverflow.com/questions/tagged/azure-active-directory) 或 [adal](http://stackoverflow.com/questions/tagged/adal) 标记获取有关堆栈溢出的帮助。
- [v2.0 协议参考](/documentation/articles/active-directory-v2-protocols/)
- [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)
- [v2.0 终结点中的范围和许可](/documentation/articles/active-directory-v2-scopes/)
- [Microsoft 应用注册门户](https://apps.dev.microsoft.com)
- [Office 365 REST API 参考](https://msdn.microsoft.com/office/office365/howto/authenticate-Office-365-APIs-using-v2)
- [Microsoft Graph](https://graph.microsoft.io)
- 下面是已通过 v2.0 终结点测试的开源客户端库和示例。

  - [Java WSO2 Identity Server](https://docs.wso2.com/display/IS500/Introducing+the+Identity+Server)
  - [Java Gluu Federation](https://github.com/GluuFederation/oxAuth)
  - [Node.Js passport-openidconnect](https://www.npmjs.com/package/passport-openidconnect)
  - [PHP OpenID Connect Basic Client](https://github.com/jumbojett/OpenID-Connect-PHP)
  - [iOS OAuth2 Client](https://github.com/nxtbgthng/OAuth2Client)
  - [Android OAuth2 Client](https://github.com/wuman/android-oauth-client)
  - [Android OpenID Connect Client](https://github.com/kalemontes/OIDCAndroidLib)

<!---HONumber=Mooncake_0613_2016-->