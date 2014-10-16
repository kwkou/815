<properties linkid="develop-mobile-tutorials-get-started-with-users-html" urlDisplayName="Get Started with Authentication (HTML5)" pageTitle="Get started with authentication (HTML 5) | Mobile Dev Center" metaKeywords="" description="Learn how to use Mobile Services to authenticate users of your HTML app through a variety of identity providers, including Google, Facebook, Twitter, and Microsoft." metaCanonical="" services="" documentationCenter="Mobile" title="Get started with authentication in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 移动服务中的身份验证入门

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet" title="Windows Store C#">Windows 应用商店 C\#</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-js" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-ios" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-android" title="Android">Android</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-html" title="HTML" class="current">HTML</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-android" title="Xamarin.Android">Xamarin.Android</a>
</div>

本主题说明如何通过 HTML 应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供者向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

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

- [Microsoft 帐户][]
- [Facebook 登录][]
- [Twitter 登录][]
- [Google 登录][]
- [Azure Active Directory][]

    请记住，要记下标识提供者生成的客户端标识和密钥值。

    <div class="dev-callout"><b>安全说明</b>

    <p>标识提供者生成的密钥是一个重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用程序分发。</p>
	</div>

4.  回到管理门户中，单击“标识”选项卡， 输入从标识提供者获取的应用程序标识符和共享密钥值，然后单击“保存”。 

    ![][2]

你的移动服务和应用程序现已配置为使用你选择的身份验证提供者。

<a name="permissions"></a>
## 限制权限将权限限制给已经过身份验证的用户

1.  在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表 。

    ![][3]

2.  单击“权限” 选项卡，将所有权限设置为“仅经过身份验证的用户” ，然后单击“保存” 。这样可以确保对 "TodoItem" 表的所有操作都要求用户经过身份验证。这样还可简化下一个教程中的脚本，因为它们无需再允许匿名用户。

    ![][4]

3.  在 app 目录中，启动 "server" 子文件夹中的下列命令文件之一。

    -   "launch-windows"（Windows 计算机）
    -   "launch-mac.command"（Mac OS X 计算机）
    -   "launch-linux.sh"（Linux 计算机）

	<div class="dev-callout"><b>说明</b>

    <p>在 Windows 计算机上，当 PowerShell 要求你确认是否要运行脚本时，请键入“R”。你的 Web 浏览器可能会警告你不要运行该脚本，因为它是从 Internet 下载的。如果出现此警告，你必须请求浏览器继续加载该脚本。</p>
	</div>

    随后将在本地计算机上启动用于托管新应用程序的 Web 服务器。

2.  在 Web 浏览器中打开 URL <http://localhost:8000/> 以启动该应用程序。

    无法加载数据。发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 *TodoItem* 表现在要求身份验证。

3.  （可选）打开 Web 浏览器的脚本调试程序，并重新加载页。检查是否发生了访问被拒绝错误。

接下来，你需要更新应用程序，以允许在从移动服务请求资源之前进行身份验证。

<a name="add-authentication"></a>
## 添加身份验证向应用程序添加身份验证

<div class="dev-callout"><b>说明</b>

<p>由于登录是在弹出窗口中执行的，因此你应该从按钮的 click 事件调用 <b>login</b> 方法。否则，许多浏览器都会隐藏登录窗口。</p>
</div>

1.  打开项目文件 index.html，找到 H1 元素，并在该元素的下面添加以下代码段：

        <div id="logged-in">
        You are logged in as <span id="login-name"></span>.
        <button id="log-out">Log out</button>
        </div>
        <div id="logged-out">
        You are not logged in.
        <button>Log in</button>
        </div>

    这样，你便可以从该页登录到移动服务。

2.  在 app.js 文件中，找到位于文件最底部的、调用 refreshTodoItems 函数的代码行，并将它替换为以下代码：

        function refreshAuthDisplay() {
        var isLoggedIn = client.currentUser !== null;
        $("#logged-in").toggle(isLoggedIn);
        $("#logged-out").toggle(!isLoggedIn);

        if (isLoggedIn) {
        $("#login-name").text(client.currentUser.userId);
        refreshTodoItems();
            }
        }

        function logIn() {
        client.login("facebook").then(refreshAuthDisplay, function(error){
        alert(error);
            });
        }

        function logOut() {
        client.logout();
        refreshAuthDisplay();
        $('#summary').html('<strong>You must login to access data.</strong>');
        }

        // On page init, fetch the data and set up event handlers
        $(function () {
        refreshAuthDisplay();
        $('#summary').html('<strong>You must login to access data.</strong>');          
        $("#logged-out button").click(logIn);
        $("#logged-in button").click(logOut);
        });

    这将会创建一组用于处理身份验证过程的函数。将使用 Facebook 登录对用户进行身份验证。

    <div class="dev-callout"><b>说明</b>

    <p>如果使用的标识提供者不是 Facebook，请将传递给上述 <b>login</b> 方法的值更改为下列其中一项：<em>microsoftaccount</em>、<em>facebook</em>、<em>twitter</em>、<em>google</em> 或 <em>windowsazureactivedirectory</em>。</p>
	</div>

3.  返回到运行应用程序的浏览器并刷新页。

当你成功登录时，应用程序应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。

    <div class="dev-callout"><b>Note</b>
    <p>When you use Internet Explorer, you may receive the error after login:<code>Cannot reach window opener.It may be on a different Internet Explorer zone</code>.This occurs because the pop-up runs in a different security zone (internet) from localhost (intranet).This only affects apps during development using localhost.As a workaround, open the <strong>Security</strong> tab of <strong>Internet Options</strong>, click <strong>Local Intranet</strong>, click <strong>Sites</strong>, and disable <strong>Automatically detect intranet network</strong>.Remember to change this setting back when you are done testing.</p>
    </div>

<a name="next-steps"> </a>
## 后续步骤

在下一教程[使用脚本为用户授权][]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。请在[移动服务 HTML/JavaScript 操作方法概念性参考][]中了解有关如何将移动服务与 HTML/JavaScript 一起使用的详细信息

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
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started-html
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [0]: ./media/mobile-services-html-get-started-users/mobile-services-selection.png
  [1]: ./media/mobile-services-html-get-started-users/mobile-service-uri.png
  [Microsoft 帐户]: /zh-cn/develop/mobile/how-to-guides/register-for-microsoft-authentication/
  [Facebook 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-facebook-authentication/
  [Twitter 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-twitter-authentication/
  [Google 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-google-authentication/
  [Azure Active Directory]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
  [2]: ./media/mobile-services-html-get-started-users/mobile-identity-tab.png
  [3]: ./media/mobile-services-html-get-started-users/mobile-portal-data-tables.png
  [4]: ./media/mobile-services-html-get-started-users/mobile-portal-change-table-perms.png
  [使用脚本为用户授权]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-html
  [移动服务 HTML/JavaScript 操作方法概念性参考]: /zh-cn/develop/mobile/how-to-guides/work-with-html-js-client
