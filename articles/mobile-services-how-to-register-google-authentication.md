<properties linkid="develop-mobile-how-to-guides-register-for-google-authentication" urlDisplayName="Register for Google Authentication" pageTitle="注册以进行 Google 身份验证 - 移动服务" metaKeywords="Azure registering application, Azure authentication, Google application authenticate, authenticate mobile services" description="了解如何注册你的应用程序，以便使用 Google 在 Azure 移动服务中进行身份验证。" metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Register your apps for Google login with Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 注册应用程序以便在移动服务中进行 Google 登录

本主题说明如何注册你的应用程序，以便能够使用 Google 在 Azure 移动服务中进行身份验证。

>[WACOM.NOTE] 本教程有关 [Azure 移动服务](/zh-cn/services/mobile-services/)，该解决方案可帮助你构建任意平台的可缩放移动应用程序。使用移动服务可以轻松地同步数据、对用户进行身份验证和发送推送通知。此页支持<a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-users/">身份验证入门</a>教程，该教程演示如何将用户登录到你的应用程序。如果这是你第一次体验移动服务，请先完成<a href="/zh-cn/documentation/articles/mobile-services-ios-get-started/">移动服务入门教程</a>。

若要完成本主题中的过程，你必须拥有一个包含已验证电子邮件地址的 Google 帐户。若要新建一个 Google 帐户，请转至 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268302" target="_blank">accounts.google.com</a>。

1. 导航到 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268303" target="_blank">Google API</a> 网站，使用 Google 帐户凭据登录，单击**"创建项目"**，提供**"项目名称"**，然后单击**"创建"**。

   	![][1]

2. 单击**"API 和身份验证"**，单击**"凭据"**，然后单击**"创建新的客户端 ID"**。

   	![][2]

3. 选择**"Web 应用程序"**，在**"授权的 JavaScript 源"**中键入移动服务 URL，将**"授权的重定向 URI"**中生成的 URL 替换为你的移动服务 URL 后接路径 _/login/google_，然后单击**"创建客户端 ID"**。

	>[WACOM.NOTE]对于使用 Visual Studio 发布到 Azure 的 .NET 后端移动服务，重定向 URL 是移动服务的 URL 后接用作 .NET 服务的移动服务的路径 _signin-google_，例如 <code>https://todolist.azure-mobile.cn/signin-google</code>。 

   	![][3]

6. 在**"Web 应用程序的客户端 ID"**下面，记下**"客户端 ID"**和**"客户端密钥"**的值。 

   	![][4]

    > [AZURE.NOTE] 客户端密钥是一个非常重要的安全凭据。请勿与任何人分享此密钥或将此密钥随应用程序分发。

你现在可以通过向移动服务提供客户端 ID 和客户端密钥值，使用 Google 登录在应用程序中进行身份验证。

<!-- Anchors. -->

<!-- Images. -->
[1]: ./media/mobile-services-how-to-register-google-authentication/mobile-services-google-new-project.png
[2]: ./media/mobile-services-how-to-register-google-authentication/mobile-services-google-create-client.png
[3]: ./media/mobile-services-how-to-register-google-authentication/mobile-services-google-create-client2.png
[4]: ./media/mobile-services-how-to-register-google-authentication/mobile-services-google-create-client3.png
[5]: ./media/mobile-services-how-to-register-google-authentication/mobile-services-google-app-details.png

<!-- URLs. -->

[Google API]: http://go.microsoft.com/fwlink/p/?LinkId=268303
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/

[Azure 管理门户]: https://manage.windowsazure.cn/
