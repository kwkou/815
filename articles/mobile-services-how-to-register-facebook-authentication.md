<properties linkid="develop-mobile-how-to-guides-register-for-facebook-authentication" urlDisplayName="Register for Facebook Authentication" pageTitle="Register for Facebook authentication - Mobile Services" metaKeywords="Azure Facebook, Azure Facebook, Azure authenticate Mobile Services" description="Learn how to use Facebook authentication in your Azure Mobile Services app." metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Register your apps for Facebook authentication with Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 注册应用程序以使用 Facebook 在移动服务中进行身份验证

本主题说明如何注册你的应用程序，以便能够使用 Facebook 在 Azure 移动服务中进行身份验证。

<div class="dev-callout"><b>说明</b>

若要完成本主题中的过程，你必须拥有一个包含已验证电子邮件地址的 Facebook 帐户和一个手机号码。若要创建新的 Facebook 帐户，请转至 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268285" target="_blank">facebook.com</a>.</p>。
</div>

1.  导航到 [Facebook 开发人员][]网站，然后使用你的 Facebook 帐户凭据登录。

2.  （可选）如果你尚未注册，请单击“Apps”（应用程序），单击“Register as a Developer”（以开发人员身份注册），接受用户政策，然后遵照注册步骤操作 。

    ![][]

3.  单击“Apps”（应用程序），然后单击“Create a New App”（创建新应用程序） 。

    ![][1]

4.  为应用程序选择唯一的名称，选择“Apps for Pages”（页面应用程序），单击“Create App”（创建应用程序），然后完成质询问题 。

    ![][2]

    随即会将应用程序注册到 Facebook

5.  单击“Settings”（设置），在“App Domains”（应用程序域）中键入移动服务的域，然后单击“Add Platform”（添加平台）并选择“Website”（网站） 。

    ![][3]

6.  在“Site URL”（站点 URL）中键入移动服务的 URL，然后单击“Save Changes”（保存更改） 。

    ![][4]

7.  单击“Show”（显示），出现请求时请提供你的密码，然后记下“App ID”（应用程序 ID）和“App Secret”（应用程序密钥）的值 。

    ![][5]

    <div class="dev-callout"><b>安全说明</b>

    <p>应用程序密钥是一个非常重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用程序分发。</p>
	</div>

现在，你可以通过向移动服务提供应用程序 ID 和应用程序密钥值，使用 Facebook 登录在应用程序中进行身份验证。

  [facebook.com]: http://go.microsoft.com/fwlink/p/?LinkId=268285
  [Facebook 开发人员]: http://go.microsoft.com/fwlink/p/?LinkId=268286
  []: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-developer-register.png
  [1]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-add-app.png
  [2]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-new-app-dialog.png
  [3]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-configure-app.png
  [4]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-configure-app-2.png
  [5]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-completed.png
