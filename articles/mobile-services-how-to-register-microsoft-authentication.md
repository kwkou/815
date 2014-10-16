<properties pageTitle="Register for Microsoft authentication - Mobile Services" metaKeywords="Azure registering application, Azure Microsoft authentication, application authenticate, authenticate mobile services" description="Learn how to register for Microsoft authentication in your Azure Mobile Services application." metaCanonical="" disqusComments="0" umbracoNaviHide="1" title="Register your apps to use a Microsoft Account login" authors="glenga" services="mobile-services" documentationCenter="Mobile" />

# 注册应用程序以使用 Microsoft 帐户登录

本主题说明如何注册你的应用程序，以便能够使用 Live Connect 作为 Azure 移动服务的身份验证提供程序。

> [WACOM.NOTE] 若要为 Windows 8.1 或 Windows Phone 8.1 应用程序配置 Microsoft 帐户身份验证，请参阅[注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证][]。

1.  在 Live Connect 开发人员中心内导航到[我的应用程序][]页，然后根据需要使用你的 Microsoft 帐户登录。

2.  单击“创建应用程序”，然后键入“应用程序名称”并单击“我接受” 。

    ![][]

    随即会将应用程序注册到 Live Connect。

3.  单击“API 设置”，启用“增强的重定向安全”，提供“重定向 URL”中的 `https://<mobile_service>.azure-mobile.net/login/microsoftaccount` 的值，然后单击“保存” 。

    ![][1]

    这样可为你的应用程序启用 Microsoft 帐户身份验证。

4.  单击“应用程序设置”，并记下“客户端 ID”和“客户端密钥”的值 。

    ![][2]

    <div class="dev-callout"><b>安全说明</b>

    <p>客户端密钥是一个非常重要的安全凭据。请勿与任何人分享客户端密钥或将密钥随应用程序分发。</p>
	</div>

现在，你可以通过向移动服务提供客户端 ID 和客户端密钥值，使用 Microsoft 帐户在应用程序中进行身份验证。

  [注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证]: /zh-cn/documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication
  [我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
  [0]: ./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-live-connect-add-app.png
  [1]: ./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-win8-app-push-auth-2.png
  [2]: ./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-win8-app-push-auth.png
