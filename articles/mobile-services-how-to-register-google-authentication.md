<properties linkid="develop-mobile-how-to-guides-register-for-google-authentication" urlDisplayName="Register for Google Authentication" pageTitle="Register for Google authentication - Mobile Services" metaKeywords="Azure registering application, Azure authentication, Google application authenticate, authenticate mobile services" description="Learn how to register your apps to use Google to authenticate with Azure Mobile Services." metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Register your apps for Google login with Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 注册应用程序以便在移动服务中进行 Google 登录

本主题说明如何注册你的应用程序，以便能够使用 Google 在 Azure 移动服务中进行身份验证。

<div class="dev-callout"><b>说明</b>

<p>若要完成本主题中的过程，你必须拥有一个包含已验证电子邮件地址的 Google 帐户。若要新建一个 Google 帐户，请转至 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268302" target="_blank">accounts.google.com</a>。</p>
</div>

1.  导航到 [Google API][] 网站，使用你的 Google 帐户凭据登录，然后单击“Create project...”（创建项目...） 。

    ![][]

2.  单击“API Access”（API 访问） ，然后单击“Create an OAuth 2.0 client ID...”（创建 OAuth 2.0 客户端 ID...） 。

    ![][1]

3.  在“Branding Information”（品牌信息）下面键入你的“Product name”（产品名称），然后单击“Next”（下一步） 。

    ![][2]

4.  在“Client ID Settings”（客户端 ID 设置）下面选择“Web application”（Web 应用程序），在“Your site or hostname”（你的站点或主机名）中键入移动服务 URL，单击“more options”（更多选项），将“Authorized Redirect URIs”（已授权的重定向 URL）中的已生成 URL 替换为移动服务 URL（后接路径 */login/google*），然后单击“Create client ID”（创建客户端 ID） 。

    ![][3]

    > [WACOM.NOTE] 对于使用 Visual Studio 发布到 Azure 的 .NET 后端移动服务，重定向 URL 是移动服务的 URL，后接用作 .NET 服务的移动服务的路径 *signin-google*，例如 `https://todolist.azure-mobile.net/signin-google`。

5.  在“Client ID for web applications”（Web 应用程序的客户端 ID）下面，记下“Client ID”（客户端 ID）和“Client secret”（客户端密钥）的值 。

    ![][4]

    <div class="dev-callout"><b>安全说明</b>

    <p>客户端密钥是一个非常重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用程序分发。</p>
	</div>

你现在可以通过向移动服务提供客户端 ID 和客户端密钥值，使用 Google 登录在应用程序中进行身份验证。

  [accounts.google.com]: http://go.microsoft.com/fwlink/p/?LinkId=268302
  [Google API]: http://go.microsoft.com/fwlink/p/?LinkId=268303
  [0]: ./media/mobile-services-how-to-register-google-authentication/mobile-services-google-developers.png
  [1]: ./media/mobile-services-how-to-register-google-authentication/mobile-services-google-create-client.png
  [2]: ./media/mobile-services-how-to-register-google-authentication/mobile-services-google-create-client2.png
  [3]: ./media/mobile-services-how-to-register-google-authentication/mobile-services-google-create-client3.png
  [4]: ./media/mobile-services-how-to-register-google-authentication/mobile-services-google-app-details.png
