<properties linkid="develop-mobile-tutorials-get-started-with-users-wp8" urlDisplayName="Get Started with Authentication" pageTitle="Get started with authentication (Windows Phone) | Mobile Dev Center" metaKeywords="" description="Learn how to use Mobile Services to authenticate users of your Windows Phone app through a variety of identity providers, including Google, Facebook, Twitter, and Microsoft." metaCanonical="" services="" documentationCenter="Mobile" title="Get started with authentication in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 移动服务中的身份验证入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users" title="Windows Store C#">Windows 应用商店 C\#</a><a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-users" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-users" title="Windows Phone" class="current">Windows Phone</a><a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-users" title="iOS">iOS</a><a href="/zh-cn/documentation/articles/mobile-services-android-get-started-users" title="Android">Android</a><a href="/zh-cn/documentation/articles/mobile-services-html-get-started-users" title="HTML">HTML</a><a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started-users" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started-users" title="Xamarin.Android">Xamarin.Android</a></div>
<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-users/" title=".NET backend">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-users/"  title="JavaScript backend" class="current">JavaScript 后端</a></div>

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">

<p>本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供者向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。</p>

<div class="dev-onpage-video-wrapper"><a href="http://go.microsoft.com/fwlink/?LinkId=298631" target="_blank" class="label">播放视频</a> <a style="background-image: url('/media/devcenter/mobile/videos/mobile-wp8-get-started-authentication-180x120.png') !important;" href="http://go.microsoft.com/fwlink/?LinkId=298631" target="_blank" class="dev-onpage-video"><span class="icon">观看教程</span></a> <span class="time">10:50</span></div>
</div>  

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1.  [注册应用程序以进行身份验证并配置移动服务][]
2.  [将表权限限制给已经过身份验证的用户][]
3.  [向应用程序添加身份验证][]

本教程基于移动服务快速入门。因此，你还必须先完成[移动服务入门][]教程。

<a name="register"></a>
## 注册应用程序注册应用程序以进行身份验证并配置移动服务

为了能够对用户进行身份验证，你必须通过标识提供者注册你的应用程序。然后，你需要向移动服务注册标识提供者生成的客户端密钥。

1.  登录到 [Azure 管理门户][]，单击“移动服务”，然后单击你的移动服务 。

    ![][]

2.  单击“仪表板”选项卡， 并记下“移动服务 URL”值。 

    ![][1]

    注册你的应用程序时，可能需要向标识提供者提供此值。

3.  从以下列表中选择支持的标识提供者，并按步骤向该标识提供者注册你的应用程序：

-   [Microsoft 帐户][]
-   [Facebook 登录][]
-   [Twitter 登录][]
-   [Google 登录][]
-   [Azure Active Directory][]

    请记住，要记下标识提供者生成的客户端标识和密钥值。

    "安全说明"

    标识提供者生成的密钥是一个重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用程序分发。

1.  回到管理门户中，单击“标识”选项卡， 输入从标识提供者获取的应用程序标识符和共享密钥值，然后单击“保存”。 

    ![][2]

你的移动服务和应用程序现已配置为使用你选择的身份验证提供者。

<a name="permissions"></a>
## 将权限限制给已经过身份验证的用户

1.  在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表 。

    ![][3]

2.  单击“权限” 选项卡，将所有权限设置为“仅经过身份验证的用户” ，然后单击“保存” 。这样可以确保对 "TodoItem" 表的所有操作都要求用户经过身份验证。这样还可简化下一个教程中的脚本，因为它们无需再允许匿名用户。

    ![][4]

3.  在 Visual Studio 2012 Express for Windows Phone 中，打开你在完成教程[移动服务入门][]后创建的项目。

4.  按 F5 键运行这个基于快速入门的应用程序；验证启动该应用程序后，是否会引发状态代码为 401（“未授权”）的未处理异常。

    发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 *TodoItem* 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

<a name="add-authentication"></a>
## 向应用程序添加身份验证

[WACOM.INCLUDE [mobile-services-windows-phone-authenticate-app][]]

<a name="next-steps"> </a>
## 后续步骤

在下一教程[移动服务用户的服务端授权][]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-users "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-users "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-ios-get-started-users "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-android-get-started-users "Android"
  [HTML]: /zh-cn/documentation/articles/mobile-services-html-get-started-users "HTML"
  [Xamarin.iOS]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started-users "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started-users "Xamarin.Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-users/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-users/ "JavaScript 后端"
  [观看教程]: http://go.microsoft.com/fwlink/?LinkId=298631
  [注册应用程序以进行身份验证并配置移动服务]: #register
  [将表权限限制给已经过身份验证的用户]: #permissions
  [向应用程序添加身份验证]: #add-authentication
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [0]: ./media/mobile-services-wp8-get-started-users/mobile-services-selection.png
  [1]: ./media/mobile-services-wp8-get-started-users/mobile-service-uri.png
  [Microsoft 帐户]: /zh-cn/develop/mobile/how-to-guides/register-for-microsoft-authentication/
  [Facebook 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-facebook-authentication/
  [Twitter 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-twitter-authentication/
  [Google 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-google-authentication/
  [Azure Active Directory]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
  [2]: ./media/mobile-services-wp8-get-started-users/mobile-identity-tab.png
  [3]: ./media/mobile-services-wp8-get-started-users/mobile-portal-data-tables.png
  [4]: ./media/mobile-services-wp8-get-started-users/mobile-portal-change-table-perms.png
  [mobile-services-windows-phone-authenticate-app]: ../includes/mobile-services-windows-phone-authenticate-app.md
  [移动服务用户的服务端授权]: /zh-cn/documentation/articles/mobile-services-windows-phone-authorize-users-in-scripts
