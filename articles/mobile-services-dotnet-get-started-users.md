<properties linkid="develop-mobile-tutorials-get-started-with-users-dotnet" urlDisplayName="Get Started with Users" pageTitle="Get started with authentication (Windows Store) | Mobile Dev Center" metaKeywords="" description="Learn how to use Mobile Services to authenticate users of your Windows Store app through a variety of identity providers, including Google, Facebook, Twitter, and Microsoft." metaCanonical="" services="" documentationCenter="Mobile" title="Get started with authentication in Mobile Services" authors="" solutions="" manager="" editor="" />

# 移动服务中的身份验证入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet" title="Windows Store C#" class="current">Windows 应用商店 C\#</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-js" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-ios" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-android" title="Android">Android</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-html" title="HTML">HTML</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-android" title="Xamarin.Android">Xamarin.Android</a></div>

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">
<p>本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供者向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。</p>


<p>单击右侧的剪辑可观看本教程的视频版本。</p>
</div>

<div class="dev-onpage-video-wrapper"><a href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Introduction-to-Windows-Azure-Mobile-Services" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/get-started-with-users-windows-store-180x120.png') !important;" href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Windows-Store-app-Getting-Started-with-Authentication-in-Windows-Azure-Mobile-Services" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a> <span class="time">10:04</span></div>
</div> 

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1.  [注册应用程序以进行身份验证并配置移动服务][]
2.  [将表权限限制给已经过身份验证的用户][]
3.  [向应用程序添加身份验证][]

本教程基于移动服务快速入门。因此，你还必须先完成[移动服务入门][]教程。

<div class="dev-callout"><b>说明</b>

<p>本教程演示了移动服务为了让你使用各种标识提供者对用户进行身份验证而提供的基本方法。此方法易于配置，并支持多个提供者。但是，此方法还要求用户在每次启动你的应用程序时登录。若要改用 Live Connect 在 Windows 应用商店应用程序中提供单一登录体验，请参阅主题<a href="/zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-dotnet">使用 Live Connect 实现对 Windows 应用商店应用程序的单一登录</a>。</p>
</div>

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

    <div class="dev-callout"><b>安全说明</b>

    <p>标识提供者生成的密钥是一个重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用程序分发。</p>
	</div>

1.  回到管理门户中，单击“标识”选项卡， 输入从标识提供者获取的应用程序标识符和共享密钥值，然后单击“保存”。 

    ![][2]

2.  （可选）完成[注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证][]中的步骤。

    <div class="dev-callout"><b>说明</b>

    <p>由于此步骤只适用于 Microsoft 帐户登录提供程序，因此是可选的。将 Windows 应用商店应用程序包信息注册到移动服务后，客户端可以重复使用 Microsoft 帐户登录凭据来提供单一登录体验。如果你不执行此操作，则每次调用 login 方法时，系统都会向 Microsoft 帐户登录用户显示登录提示。如果你打算使用 Microsoft 帐户标识提供者，请完成此步骤。</p>
	</div>

你的移动服务和应用程序现已配置为使用你选择的身份验证提供者。

<a name="permissions"></a>
## 限制权限将权限限制给已经过身份验证的用户

1.  在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表 。

    ![][3]

2.  单击“权限” 选项卡，将所有权限设置为“仅经过身份验证的用户” ，然后单击“保存” 。这样可以确保对 "TodoItem" 表的所有操作都要求用户经过身份验证。这样还可简化下一个教程中的脚本，因为它们无需再允许匿名用户。

    ![][4]

3.  在 Visual Studio 2012 Express for Windows 8 中，打开你在完成教程[移动服务入门][]时创建的项目。

4.  按 F5 键运行这个基于快速入门的应用程序；验证启动该应用程序后，是否会引发状态代码为 401（“未授权”）的未处理异常。

    发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 *TodoItem* 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

<a name="add-authentication"></a>
## 添加身份验证向应用程序添加身份验证

1.  打开项目文件 mainpage.xaml.cs 并添加以下 using 语句：

        using Windows.UI.Popups;

2.  将以下代码段添加到 MainPage 类：

        private MobileServiceUser user;
        private async System.Threading.Tasks.Task Authenticate()
        {
        while (user == null)
            {
        string message;
        try
                {
        user = await App.MobileService
        .LoginAsync(MobileServiceAuthenticationProvider.Facebook);
        message = 
        string.Format("You are now logged in - {0}", user.UserId);
                }
        catch (InvalidOperationException)
                {
        message = "You must log in. Login Required";
                }

        var dialog = new MessageDialog(message);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
            }
        }

    这样可以创建用于存储当前用户的成员变量，以及用于处理身份验证过程的方法。将使用 Facebook 登录对用户进行身份验证。如果使用的标识提供者不是 Facebook，请将上述 "MobileServiceAuthenticationProvider" 的值更改为提供者的值。

    <div class="dev-callout"><b>说明</b>

    <p>如果已将 Windows 应用商店应用程序包信息注册到移动服务，则应该为 <em>useSingleSignOn</em> 参数提供 <b>true</b> 值以调用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=311594" target="_blank">LoginAsync</a> 方法。如果你不执行此操作，则每次调用 login 方法时，系统仍会向你的用户提供登录提示。</p>
	</div>

3.  将现有的 "OnNavigatedTo" 方法覆盖替换为以下方法，以便调用新的 "Authenticate" 方法：

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
        await Authenticate();
        RefreshTodoItems();
        }

4.  按 F5 键运行应用程序，并使用你选择的标识提供者登录应用程序。

    当你成功登录时，应用程序应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。

<a name="next-steps"> </a>
## 后续步骤

在下一教程[使用脚本为用户授权][]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。请在[移动服务 .NET 操作方法概念性参考][]中了解有关如何将移动服务与 .NET 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/get-started-with-users-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/get-started-with-users-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-users-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/get-started-with-users-android "Android"
  [HTML]: /zh-cn/develop/mobile/tutorials/get-started-with-users-html "HTML"
  [Xamarin.iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-android "Xamarin.Android"
  [观看教程]: http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Introduction-to-Windows-Azure-Mobile-Services
  [播放视频]: http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Windows-Store-app-Getting-Started-with-Authentication-in-Windows-Azure-Mobile-Services
  [注册应用程序以进行身份验证并配置移动服务]: #register
  [将表权限限制给已经过身份验证的用户]: #permissions
  [向应用程序添加身份验证]: #add-authentication
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started
  [使用 Live Connect 实现对 Windows 应用商店应用程序的单一登录]: /zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-dotnet
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [0]: ./media/mobile-services-dotnet-get-started-users/mobile-services-selection.png
  [1]: ./media/mobile-services-dotnet-get-started-users/mobile-service-uri.png
  [Microsoft 帐户]: /zh-cn/develop/mobile/how-to-guides/register-for-microsoft-authentication/
  [Facebook 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-facebook-authentication/
  [Twitter 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-twitter-authentication/
  [Google 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-google-authentication/
  [Azure Active Directory]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
  [2]: ./media/mobile-services-dotnet-get-started-users/mobile-identity-tab.png
  [注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证]: /zh-cn/develop/mobile/how-to-guides/register-windows-store-app-package
  [3]: ./media/mobile-services-dotnet-get-started-users/mobile-portal-data-tables.png
  [4]: ./media/mobile-services-dotnet-get-started-users/mobile-portal-change-table-perms.png
  [LoginAsync]: http://go.microsoft.com/fwlink/p/?LinkId=311594
  [使用脚本为用户授权]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-dotnet
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/develop/mobile/how-to-guides/work-with-net-client-library
