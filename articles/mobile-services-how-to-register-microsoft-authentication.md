<properties pageTitle="注册以进行 Microsoft 身份验证 - 移动服务" metaKeywords="Azure registering application, Azure Microsoft authentication, application authenticate, authenticate mobile services" description="了解如何在 Azure 移动服务应用程序中注册以进行 Microsoft 身份验证。" metaCanonical="" disqusComments="0" umbracoNaviHide="1" title="Register your apps to use a Microsoft Account login" authors="glenga" services="mobile-services" documentationCenter="Mobile" />

# 注册应用程序以使用 Microsoft 帐户登录

本主题说明如何注册你的应用程序，以便能够使用 Live Connect 作为 Azure 移动服务的身份验证提供程序。 

>[WACOM.NOTE]若要为通用 Windows 应用程序配置 Microsoft 帐户身份验证或者为 Windows 应用商店应用程序提供单一登录体验，请参阅[注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证](/zh-cn/documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication)。

1. 在 Live Connect 开发人员中心内导航到<a href="http://go.microsoft.com/fwlink/p/?LinkId=262039" target="_blank">我的应用程序</a>页，然后根据需要使用你的 Microsoft 帐户登录。 

2. 单击**"创建应用程序"**，然后键入**"应用程序名称"**并单击**"我接受"**。

   	![][1] 

   	随即会将应用程序注册到 Live Connect。

3. 单击**"API 设置"**，在**"重定向 URL"**中提供  `https://<mobile_service>.azure-mobile.cn/login/microsoftaccount` 值，然后单击**"保存"**。

	>[WACOM.NOTE]对于使用 Visual Studio 发布到 Azure 的 .NET 后端移动服务，重定向 URL 是移动服务的 URL 后接用作 .NET 服务的移动服务的路径 _signin-microsoft_，例如 <code>https://todolist.azure-mobile.cn/signin-microsoft</code>。  

	![][3]

	这样可为您的应用启用 Microsoft 帐户身份验证。

	>[WACOM.NOTE]对于现有的 Live Connect 应用程序注册，你可能必须先启用**"增强型重定向安全"**。

4. 单击**"应用程序设置"**，并记下**"客户端 ID"**和**"客户端密钥"**的值。 

   	![][2]

    > [AZURE.NOTE] 客户端密钥是一个非常重要的安全凭据。请勿与任何人分享客户端密钥或将密钥随应用程序分发。

现在，你可以通过向移动服务提供客户端 ID 和客户端密钥值，使用 Microsoft 帐户在应用程序中进行身份验证。

<!-- Anchors. -->

<!-- Images. -->
[1]: ./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-live-connect-add-app.png
[2]: ./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-win8-app-push-auth.png
[3]: ./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-win8-app-push-auth-2.png

<!-- URLs. -->

["提交应用程序"页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039

[Azure 管理门户]: https://manage.windowsazure.cn/
