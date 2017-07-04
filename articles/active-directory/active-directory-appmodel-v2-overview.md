<properties
    pageTitle="v2.0 终结点概述 | Azure"
    description="使用 Microsoft 帐户和 Azure Active Directory 登录的构建应用简介。"
    services="active-directory"
    documentationcenter=""
    author="dstrockis"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="2dee579f-fdf6-474b-bc2c-016c931eaa27"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/07/2017"
    ms.author="dastrock" />

# 在单个应用中登录 Microsoft 帐户和 Azure AD 用户

在过去，想要支持 Microsoft 帐户和 Azure Active Directory 的应用开发人员需要集成两个单独的系统。我们现在推出了新的身份验证 API 版本，可让你通过 Azure AD 系统使用这两种帐户类型来登录用户。这种聚合式身份验证系统称为 **v2.0 终结点**。借助 v2.0 终结点，通过一个简单的集成就可以与具有个人和工作/学校帐户的数以百万计的用户受众进行沟通。

使用 v2.0 终结点的应用还可以通过其中一种帐户从 [Microsoft Graph](https://graph.microsoft.io) 和 [Office 365](https://msdn.microsoft.com/office/office365/howto/authenticate-Office-365-APIs-using-v2) 使用 REST API。

## 入门  <a name="getting-started"></a>
从下述列表中选择偏爱的平台，以便使用开源库与框架生成应用。或者，你可以使用我们的 OAuth 2.0 和 OpenID Connect 协议文档来直接发送和接收协议消息，而不必使用身份验证库。

<!-- TODO: Finalize this table  -->
[AZURE.INCLUDE [active-directory-v2-quickstart-table](../../includes/active-directory-v2-quickstart-table.md)]

## 新增功能
此处提供的概念性信息可帮助你了解 v2.0 终结点的定义及其功能。

- 了解[使用 v2.0 终结点可以构建哪种类型的应用](/documentation/articles/active-directory-v2-flows/)。
- 了解 v2.0 终结点的[限制、局限性和约束](/documentation/articles/active-directory-v2-limitations/)。
- 最近添加的支持领域：[管理受限作用域](/documentation/articles/active-directory-v2-scopes/)和 [OAuth2 客户端凭据授予](/documentation/articles/active-directory-v2-protocols-oauth-client-creds/)。何不试验一下！

## 引用
这些链接有助于深入地利用平台：

- 使用 [azure-active-directory](http://stackoverflow.com/questions/tagged/azure-active-directory) 或 [adal](http://stackoverflow.com/questions/tagged/adal) 标记获取有关堆栈溢出的帮助。
- [v2.0 协议参考](/documentation/articles/active-directory-v2-protocols/)
- [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)
- [v2.0 库参考](/documentation/articles/active-directory-v2-libraries/)
- [v2.0 终结点中的范围和许可](/documentation/articles/active-directory-v2-scopes/)
- [Microsoft 应用注册门户](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)
- [Office 365 REST API 参考](https://msdn.microsoft.com/office/office365/howto/authenticate-Office-365-APIs-using-v2)
- [Microsoft Graph](https://graph.microsoft.io)

<!---HONumber=Mooncake_0120_2017-->
<!---Update_Description: update meta properties -->