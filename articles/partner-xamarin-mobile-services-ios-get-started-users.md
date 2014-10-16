<properties linkid="develop-mobile-tutorials-get-started-with-users-xamarin-ios" urlDisplayName="Get Started with Authentication (Xamarin.iOS)" pageTitle="Get started with authentication (Xamarin.iOS) - Mobile Services" metaKeywords="Azure registering application, Azure authentication, application authenticate, authenticate mobile services, Mobile Services Xamarin.iOS" description="Learn how to use authentication in your Azure Mobile Services app for Xamarin.iOS." metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile"  title="Get started with authentication in Mobile Services" />

# 移动服务中的身份验证入门

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet" title="Windows Store C#">Windows 应用商店 C\#</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-js" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-ios" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-android" title="Android">Android</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-html" title="HTML">HTML</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-ios" title="Xamarin.iOS" class="current">Xamarin.iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-android" title="Xamarin.Android">Xamarin.Android</a>
</div>

本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供者向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1.  [注册应用程序以进行身份验证并配置移动服务][]
2.  [将表权限限制给已经过身份验证的用户][]
3.  [向应用程序添加身份验证][]

本教程基于移动服务快速入门。因此，你还必须先完成[移动服务入门][]教程。

完成本教程需要安装 [Xamarin.iOS]、XCode 5.0 和 iOS 5.0 或更高版本。

<a name="register"></a>
## 注册应用程序注册应用程序以进行身份验证并配置移动服务

为了能够对用户进行身份验证，你必须通过标识提供者注册你的应用程序。然后，你需要向移动服务注册标识提供者生成的客户端密钥。

1.  登录到 [Azure 管理门户][]，单击“移动服务”，然后单击你的移动服务 。

    ![][]

2.  单击“仪表板” 选项卡，记下"站点 URL" 值。

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

你的移动服务和应用程序现已配置为使用你选择的身份验证提供者。

<a name="permissions"></a>
## 限制权限将权限限制给已经过身份验证的用户

1.  在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表 。

    ![][3]

2.  单击“权限” 选项卡，将所有权限设置为“仅经过身份验证的用户” ，然后单击“保存” 。这样可以确保对 "TodoItem" 表的所有操作都要求用户经过身份验证。这样还可简化下一个教程中的脚本，因为它们无需再允许匿名用户。

    ![][4]

3.  在 Xcode 中，打开你在完成[移动服务入门][]教程后创建的项目。

4.  在 iPhone 模拟器中按“运行”按钮以生成项目并启动应用程序；验证启动该应用程序后，是否会引发状态代码为 401（“未授权”）的未处理异常 。

    发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 *TodoItem* 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

<a name="add-authentication"></a>
## 添加身份验证向应用程序添加身份验证

1.  打开 "TodoService" 项目文件，并添加以下变量

        // Mobile Service logged in user
        private MobileServiceUser user; 
        public MobileServiceUser User { get { return user; } }

2.  然后，向 "TodoService" 添加一个名为 "Authenticate" 的新方法，其定义为：

        private async Task Authenticate(UIViewController view)
        {
        try
            {
        user = await client.LoginAsync(view, MobileServiceAuthenticationProvider.MicrosoftAccount);
            }
        catch (Exception ex)
            {
        Console.Error.WriteLine (@"ERROR - AUTHENTICATION FAILED {0}", ex.Message);
            }
        }

    <div class="dev-callout"><b>说明</b>

    <p>如果使用的标识提供者不是 Microsoft 帐户，请将传递给上述 <b>LoginAsync</b> 的值更改为下列其中一项：<em>Facebook</em>、<em>Twitter</em>、<em>Google</em> 或 <em>WindowsAzureActiveDirectory</em>。</p>
	</div>

3.  从 "TodoService" 构造函数将对 "TodoItem" 表的请求移到名为 "CreateTable" 的新方法中：

        private async Task CreateTable()
        {
        // Create an MSTable instance to allow us to work with the TodoItem table
        todoTable = client.GetTable<TodoItem>();
        }

4.  创建一个名为 "LoginAndGetData" 的新异步公共方法，其定义为：

        public async Task LoginAndGetData(UIViewController view)
        {
        await Authenticate(view);
        await CreateTable();
        }

5.  在 "TodoListViewController" 中，重写 "ViewDidAppear" 方法，并按如下所示定义它。如果 "TodoService" 还没有用户的句柄，则此代码让用户登录：

        public override async void ViewDidAppear(bool animated)
        {
        base.ViewDidAppear(animated);

        if (TodoService.DefaultService.User == null)
            {
        await TodoService.DefaultService.LoginAndGetData(this);
            }

        if (TodoService.DefaultService.User == null)
            {
        // TODO::show error
        return;
            } 

        RefreshAsync();
        }

6.  从 "TodoListViewController.ViewDidLoad" 中删除对 "RefreshAsync" 的原始调用。

7.  按“运行”按钮 以生成项目，在 iPhone 模拟器中启动应用程序，然后使用你选择的标识提供者登录。

    当你成功登录时，应用程序应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。

## 获取已完成的示例

下载[已完成的示例项目][]。请务必使用你自己的 Azure 设置更新 "applicationURL" 和 "applicationKey" 变量。

<a name="next-steps"></a>
## 后续步骤

在下一教程[使用脚本为用户授权][]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/get-started-with-users-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/get-started-with-users-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-users-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/get-started-with-users-android "Android"
  [HTML]: /zh-cn/develop/mobile/tutorials/get-started-with-users-html "HTML"
  [Xamarin.iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-android "Xamarin.Android"
  [注册应用程序以进行身份验证并配置移动服务]: #register
  [将表权限限制给已经过身份验证的用户]: #permissions
  [向应用程序添加身份验证]: #add-authentication
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started-xamarin-ios
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [0]: ./media/partner-xamarin-mobile-services-ios-get-started-users/mobile-services-selection.png
  [1]: ./media/partner-xamarin-mobile-services-ios-get-started-users/mobile-service-uri.png
  [Microsoft 帐户]: /zh-cn/develop/mobile/how-to-guides/register-for-microsoft-authentication/
  [Facebook 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-facebook-authentication/
  [Twitter 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-twitter-authentication/
  [Google 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-google-authentication/
  [Azure Active Directory]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
  [2]: ./media/partner-xamarin-mobile-services-ios-get-started-users/mobile-identity-tab.png
  [3]: ./media/partner-xamarin-mobile-services-ios-get-started-users/mobile-portal-data-tables.png
  [4]: ./media/partner-xamarin-mobile-services-ios-get-started-users/mobile-portal-change-table-perms.png
  [已完成的示例项目]: http://go.microsoft.com/fwlink/p/?LinkId=331328
  [使用脚本为用户授权]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-xamarin-ios
