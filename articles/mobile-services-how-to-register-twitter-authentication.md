<properties linkid="develop-mobile-how-to-guides-register-for-twitter-authentication" urlDisplayName="Register for Twitter Authentication" pageTitle="Register for Twitter authentication - Mobile Services" metaKeywords="Azure registering application, Azure Twitter authentication, application authenticate, authenticate mobile services, Mobile Services Twitter" description="Learn how to use Twitter authentication with your Azure Mobile Services application." metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Register your apps for Twitter login with Mobile Services" authors="" solutions="" manager="" editor="" />

# 注册应用程序以便在移动服务中进行 Twitter 登录

本主题说明如何注册你的应用程序，以便能够使用 Twitter 在 Azure 移动服务中进行身份验证。

<div class="dev-callout"><b>说明</b>

若要完成本主题中的过程，你必须拥有一个包含已验证电子邮件地址的 Twitter 帐户。若要创建新的 Twitter 帐户，请转至 <a href="http://go.microsoft.com/fwlink/p/?LinkID=268287" target="_blank">twitter.com</a></p>。
</div>

1.  导航到 [Twitter 开发人员][]网站，使用你的 Twitter 帐户凭据登录，然后单击“Create a new application”（创建新应用程序） 。

    ![][]

2.  键入应用程序的“Name”（名称）、“Description”（说明）和“Web Site”（网站）值，然后在“Callback URL”（回调 URL）中键入移动服务的 URL 。

    ![][1]

    > [WACOM.NOTE] 对于使用 Visual Studio 发布到 Azure 的 .NET 后端移动服务，重定向 URL 是移动服务的 URL，后接用作 .NET 服务的移动服务的路径 *signin-twitter*，例如 `https://todolist.azure-mobile.net/signin-twitter`。
    > “Web Site”（网站）值是必填的，但并不会使用该值 。

3.  在页的底部，阅读并接受条款，键入正确的 CAPTCHA 字，然后单击“Create your Twitter application”（创建 Twitter 应用程序） 。

    ![][2]

    随即将会注册应用程序并显示应用程序详细信息。

4.  记下“Consumer key”（使用者密钥）和“Consumer secret”（使用者机密）的值 。

    ![][3]

    "安全说明"

    使用者机密是一个非常重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用程序分发。

5.  单击“Settings”（设置）选项卡，向下滚动并选中“Allow this application to be used to sign in with Twitter”（允许使用此应用程序进行 Twitter 登录），然后单击“Update this Twitter application's settings”（更新此 Twitter 应用程序的设置）。 

    ![][4]

现在，你可以通过向移动服务提供使用者密钥和使用者机密值，使用 Twitter 登录在应用程序中进行身份验证。

  [twitter.com]: http://go.microsoft.com/fwlink/p/?LinkID=268287
  [Twitter 开发人员]: http://go.microsoft.com/fwlink/p/?LinkId=268300
  [0]: ./media/mobile-services-how-to-register-twitter-authentication/mobile-services-twitter-developers.png
  [1]: ./media/mobile-services-how-to-register-twitter-authentication/mobile-services-twitter-register-app1.png
  [2]: ./media/mobile-services-how-to-register-twitter-authentication/mobile-services-twitter-register-app2.png
  [3]: ./media/mobile-services-how-to-register-twitter-authentication/mobile-services-twitter-app-details.png
  [4]: ./media/mobile-services-how-to-register-twitter-authentication/mobile-services-twitter-register-settings.png
