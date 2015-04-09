<properties linkid="develop-mobile-tutorials-single-sign-on-windows-8-wp8" urlDisplayName="使用单一登录进行身份验证" pageTitle="使用 Live Connect 进行应用程序身份验证 (Windows Phone) | 移动开发人员中心" metaKeywords="" description="了解如何通过 Windows Phone 应用程序在 Azure 移动服务中使用 Live Connect 单一登录。" metaCanonical="" services="" documentationCenter="Mobile" title="Authenticate your Windows Phone 8 app with Live Connect single sign-on" authors="glenga" solutions="" manager="" editor="" />




# 使用 Live Connect 单一登录对 Windows Phone 8 应用程序进行身份验证

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-single-sign-on/" title="Windows Store C#">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-single-sign-on/" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-windows-phone-single-sign-on/" title="Windows Phone" class="current">Windows Phone</a>
</div>	

本主题说明如何使用 Live Connect 单一登录从 Windows Phone 8 应用程序对 Azure 移动服务的用户进行身份验证。在本教程中，你将使用 Live Connect 向快速入门项目添加身份验证功能。成功通过 Live Connect 进行身份验证后，将使用名称欢迎已登录的用户并显示用户 ID 值。  

>[WACOM.NOTE]本教程演示通过 Live Connect 为 Windows Phone 应用程序提供单一登录体验的使用好处。这使你可以更轻松地使用移动服务对已登录的用户进行身份验证。有关支持多个身份验证提供程序的更通用的身份验证体验，请参阅主题 <a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-users">向应用程序添加身份验证</a>。 

本教程将指导你完成以下启用 Live Connect 身份验证的基本步骤：

1. [注册应用程序以进行身份验证并配置移动服务]
2. [将表权限限制给已经过身份验证的用户]
3. [向应用程序添加身份验证]

本教程需要的内容如下：

+ [用于 Windows 和 Windows Phone 的 Live SDK]
+ Microsoft Visual Studio 2012 Express for Windows Phone
+ 此外，还必须先完成教程[将移动服务添加到现有应用程序]。

<h2><a name="register"></a>使用 Live Connect 注册应用程序</h2>

若要对用户进行身份验证，必须在 Live Connect 开发人员中心注册你的应用程序。然后，你必须注册用于将 Live Connect 与移动服务集成的客户端密钥。

1. 登录到 [Azure 管理门户]、单击"移动服务"，然后单击您的移动服务。

   	![][4]

2. 单击"仪表板"选项卡，记下"站点 URL"值。

   	![][5]

    你将使用此值来定义重定向域。

3. 导航到你的本地 Git 存储库的 <a href="http://go.microsoft.com/fwlink/p/?LinkId=262039" target="_blank">我的应用程序</a> 页（在 Live Connect 开发人员中心中），然后使用您的 Microsoft 帐户登录（如果需要）。 

4. 单击"创建应用程序"，然后键入"应用程序名称"并单击"我接受"。

   	![][1] 

   	随即会将应用程序注册到 Live Connect。

5. 依次单击"应用程序设置页"、"API 设置"，然后记下"客户端 ID"和"客户端密钥"的值。 

   	![][2]

 > [AZURE.NOTE] **安全说明** 客户端密钥是一个重要的安全凭据。请勿与任何人分享客户端密钥或将密钥随应用程序分发。

6. 在"重定向域"中，输入在步骤 2 中提供的移动服务 URL，在"移动客户端应用程序"下单击"是"，然后单击"保存"。

7. 返回管理门户，单击"标识"选项卡，输入从 Live Connect 获得的客户端密钥，然后单击"保存"。

   	![][13]

现在，你的移动服务和应用程序都已配置为使用 Live Connect。

<h2><a name="permissions"></a>将权限限制给已经过身份验证的用户</h2>

1. 在管理门户中，单击"数据"选项卡，然后单击"TodoItem"表。 

   	![][14]

2. 单击"权限"选项卡，将所有权限设置为"仅经过身份验证的用户"，然后单击"保存"。这样可以确保对 **TodoItem** 表的所有操作都要求用户经过身份验证。这样还可简化下一个教程中的脚本，因为它们无需再允许匿名用户。

   	![][15]

3. 在 Visual Studio 2012 Express for Windows Phone 中，打开在完成教程[将移动服务添加到现有应用程序]后创建的项目。 

4. 按 F5 键以运行这个基于快速入门的应用程序；验证是否会引发状态代码为 401（"未授权"）的异常。 
   
   	发生此异常的原因是应用程序以未经身份验证的用户身份访问移动服务，但 _TodoItem_ 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前通过 Live Connect 对用户进行身份验证。

<h2><a name="add-authentication"></a>向应用程序添加身份验证</h2>

1. 下载并安装用于 [Windows 和 Windows Phone 的 Live SDK](http://go.microsoft.com/fwlink/p/?LinkId=262253)。

2. 在 Visual Studio 的"项目"菜单上，单击"添加引用"，然后展开"程序集"，单击"扩展"，选中"Microsoft.Live"，然后单击"确定"。 

   	![][16]

  	这将向项目添加 Live SDK 的引用。

5. 打开项目文件 mainpage.xaml.cs 并添加以下 using 语句：

        using Microsoft.Live;      

6. 将以下代码段添加到 MainPage 类：
	
        private LiveConnectSession session;
        private async System.Threading.Tasks.Task Authenticate()
        {
            LiveAuthClient liveIdClient = new LiveAuthClient("<< INSERT CLIENT ID HERE >>");

            while (session == null)
            {
                LiveLoginResult result = await liveIdClient.LoginAsync(new[] { "wl.basic" });
                if (result.Status == LiveConnectSessionStatus.Connected)
                {
                    session = result.Session;
                    LiveConnectClient client = new LiveConnectClient(result.Session);
                    LiveOperationResult meResult = await client.GetAsync("me");
                    MobileServiceUser loginResult = await App.MobileService
                        .LoginWithMicrosoftAccountAsync(result.Session.AuthenticationToken);

                    string title = string.Format("Welcome {0}!", meResult.Result["first_name"]);
                    var message = string.Format("You are now logged in - {0}", loginResult.UserId);
                    MessageBox.Show(message, title, MessageBoxButton.OK);                    
                }
                else
                {
                    session = null;
                    MessageBox.Show("You must log in.", "Login Required", MessageBoxButton.OK);                    
                }
            }
         }

    这样可以创建用于存储当前 Live Connect 会话的成员变量，以及用于处理身份验证过程的方法。

7. 使用向 Live Connect 注册应用程序时生成的客户端 ID 值更新上一步中的 _<< INSERT CLIENT ID HERE >>_ 字符串。

    > [AZURE.NOTE] 在 Windows Phone 8 应用程序中，通过将客户端 ID 值传递给类构造函数创建 **LiveAuthClient** 类的一个实例。在 [Windows 应用商店应用程序](/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-single-sign-on/)中，通过传递重定向域 URI 实例化同一个类。

8. 删除或注释掉现有的 **OnNavigatedTo** 方法覆盖，并将其替换为以下方法，用于处理页的 **Loaded** 事件。 

        async void MainPage_Loaded(object sender, RoutedEventArgs e)
        {
            await Authenticate();
            RefreshTodoItems();
        }

   	此方法调用新的 **Authenticate** 方法。 

9. 将 MainPage 构造函数替换为以下代码：

        // Constructor
        public MainPage()
        {
            InitializeComponent();
            this.Loaded += MainPage_Loaded;
        }

   	此构造函数还注册 Loaded 事件的处理程序。
		
9. 按 F5 键运行应用程序，并使用 Microsoft 帐户登录 Live Connect。 

   	成功登录后，该应用程序运行时将不会出错，并且你将能够查询移动服务，并对数据进行更新。

## <a name="next-steps"> </a>后续步骤

在下一教程[使用脚本为用户授权]中，将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。有关如何使用其他标识提供者进行身份验证的信息，请参阅[身份验证入门]。 

<!-- Anchors. -->
[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制给已经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[后续步骤]:#next-steps

<!-- Images. -->
[1]: ./media/mobile-services-windows-phone-single-sign-on/mobile-services-live-connect-add-app.png
[2]: ./media/mobile-services-windows-phone-single-sign-on/mobile-live-connect-app-api-settings-mobile.png
[4]: ./media/mobile-services-windows-phone-single-sign-on/mobile-services-selection.png
[5]: ./media/mobile-services-windows-phone-single-sign-on/mobile-service-uri.png

[13]: ./media/mobile-services-windows-phone-single-sign-on/mobile-identity-tab-ma-only.png
[14]: ./media/mobile-services-windows-phone-single-sign-on/mobile-portal-data-tables.png
[15]: ./media/mobile-services-windows-phone-single-sign-on/mobile-portal-change-table-perms.png
[16]: ./media/mobile-services-windows-phone-single-sign-on/mobile-add-reference-live-wp8.png

<!-- URLs. -->
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[用于 Windows 和 Windows Phone 的 Live SDK]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[将移动服务添加到现有应用程序]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-data/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-users
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-windows-phone-authorize-users-in-scripts/

[Azure 管理门户]: https://manage.windowsazure.cn/
