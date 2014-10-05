<properties linkid="develop-mobile-tutorials-single-sign-on-windows-8-js" urlDisplayName="Authenticate with single sign-on" pageTitle="Authenticate your app with Live Connect (JavaScript)" metaKeywords="Azure Live Connect, Azure SSO, SSO Live Connect, mobile services sso, Windows Store app sso, Azure Javascript SSO" description="Learn how to use Live Connect single sign-on in Azure Mobile Services from a Windows Store application." metaCanonical="http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-dotnet/" services="" documentationCenter="Mobile" title="Authenticate your Windows Store app with Live Connect single sign-on" authors="" solutions="" manager="" editor="" />

# 使用 Live Connect 单一登录对 Windows 应用商店应用程序进行身份验证

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-dotnet" title="Windows Store C#">Windows 应用商店 C\#</a><a href="/zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-js" title="Windows Store JavaScript" class="current">Windows 应用商店 JavaScript</a><a href="/zh-cn/develop/mobile/tutorials/single-sign-on-wp8" title="Windows Phone">Windows Phone</a>
</div>	

本主题说明如何通过 Windows 应用商店应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将使用 Live Connect 向快速入门项目添加身份验证功能。成功通过 Live Connect 进行身份验证后，将使用名称欢迎已登录的用户并显示用户 ID 值。

<div class="dev-callout"><b>说明</b>

<p>本教程演示通过 Live Connect 为 Windows 应用商店应用程序提供单一登录体验的使用好处。这使你可以更轻松地使用移动服务对已登录的用户进行身份验证。有关支持多个身份验证提供程序的更通用的身份验证体验，请参阅<a href="/zh-cn/develop/mobile/tutorials/get-started-with-users-js/">身份验证入门</a>主题。</p>
</div>

本教程将指导你完成以下启用 Live Connect 身份验证的基本步骤：

1.  [注册应用程序以进行身份验证并配置移动服务][]
2.  [将表权限限制给已经过身份验证的用户][]
3.  [向应用程序添加身份验证][]

本教程需要的内容如下：

-   [Live SDK for Windows][]
-   Microsoft Visual Studio 2012 Express for Windows 8 RC 或更高版本

本教程基于移动服务快速入门。因此，你还必须先完成[移动服务入门][]教程。

<a name="register"></a>
## 注册应用程序向 Windows 应用商店注册应用程序

为了能够对用户进行身份验证，你必须将应用程序提交到 Windows 应用商店。然后，你必须注册用于将 Live Connect 与移动服务集成的客户端密钥。

1.  如果尚未注册应用程序，请在开发人员中心内导航到 Windows 应用商店应用程序的[“提交应用程序”页][]，用 Microsoft 帐户登录，然后单击“应用程序名称” 。

    ![][]

2.  在“应用程序名称” 中为应用程序键入一个名称，单击“保留应用程序名称” ，然后单击“保存”。 

    ![][1]

    此操作为应用程序创建一个新的 Windows 应用商店注册。

3.  在 Visual Studio 2012 Express for Windows 8 中，打开你在完成教程[移动服务入门][]时创建的项目。

4.  在解决方案资源管理器中，右键单击项目，单击“应用商店” ，然后单击“将应用程序与应用商店关联...” 。

    ![][2]

    此时将显示“将应用程序与 Windows 应用商店关联” 向导。

5.  在该向导中，单击“登录” ，然后用你的 Microsoft 帐户登录。

6.  选择在第 2 步中注册的应用程序，单击“下一步” ，然后单击“关联” 。

    ![][3]

    这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。

7.  登录到 [Azure 管理门户][]，单击“移动服务”，然后单击你的移动服务 。

    ![][4]

8.  单击“仪表板” 选项卡，记下"站点 URL" 值。

    ![][5]

    你将使用此值来定义重定向域。

9.  在 Live Connect 开发人员中心内导航到[我的应用程序][]页，然后在“我的应用程序”列表中单击你的应用程序 。

    ![][6]

10. 依次单击“编辑设置” 、“API 设置” ，然后记下"客户端 ID" 和"客户端密钥"的值。

    ![][7]

    "安全说明"

    客户端密钥是一个非常重要的安全凭据。请勿与任何人分享客户端密钥或将密钥随应用程序分发。

11. 在“重定向域”中，输入你在执行步骤 8 时获取的移动服务 URL，然后单击“保存” 。

12. 返回管理门户，单击“标识” 选项卡，输入从 Windows 应用商店获得的"客户端密钥"，然后单击“保存” 。

    ![][8]

现在，你的移动服务和应用程序都已配置为使用 Live Connect。

<a name="permissions"></a>
## 限制权限将权限限制给已经过身份验证的用户

1.  在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表 。

    ![][9]

2.  单击“权限” 选项卡，将所有权限设置为“仅经过身份验证的用户” ，然后单击“保存” 。这样可以确保对 "TodoItem" 表的所有操作都要求用户经过身份验证。这样还可简化下一个教程中的脚本，因为它们无需再允许匿名用户。

    ![][10]

3.  在 Visual Studio 2012 Express for Windows 8 中，打开你在完成教程[移动服务入门][]时创建的项目。

4.  按 F5 键以运行这个基于快速入门的应用程序；验证是否会引发状态代码为 401（“未授权”）的异常。

    发生此异常的原因是应用程序以未经身份验证的用户身份访问移动服务，但 *TodoItem* 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前通过 Live Connect 对用户进行身份验证。

<a name="add-authentication"></a>
## 添加身份验证向应用程序添加身份验证

1.  下载并安装[用于 Windows 的 Live SDK][Live SDK for Windows]。

2.  在 Visual Studio 的“项目” 菜单上，单击“添加引用” ，然后展开“Windows” ，单击“扩展” ，选中 "Live SDK"，然后单击“确定” 。

    ![][11]

    这将向项目添加 Live SDK 的引用。

3.  打开 default.html 项目文件，并在 \<head\> 元素中添加以下 \<script\> 元素。

        <script src="///LiveSDKHTML/js/wl.js"></script>

    这将在 default.html 文件中启用 Microsoft IntelliSense。

4.  打开项目文件 default.js，并在该文件的顶部添加以下注释。

        /// <reference path="///LiveSDKHTML/js/wl.js" />

    这将在 default.js 文件中启用 Microsoft IntelliSense。

5.  在 "app.OnActivated" 方法重载中，将对 "refreshTodoItems" 方法的调用替换为以下代码：

        var session = null;   

        var logout = function () {
        return new WinJS.Promise(function (complete) {
        WL.getLoginStatus().then(function () {
        if (WL.canLogout()) {
        WL.logout(complete);                            
                    }
        else {
        complete();
                    }
                });
            });
        };                  

        var login = function () {
        return new WinJS.Promise(function (complete) {                    
        WL.login({ scope:"wl.basic"}).then(function (result) {
        session = result.session;

        WinJS.Promise.join([
        WL.api({ path:"me", method:"GET" }),
        client.login(result.session.authentication_token)
        ]).done(function (results) {
        var profile = results[0];
        var mobileServicesUser = results[1];
        refreshTodoItems();
        var title = "Welcome " + profile.first_name + "!";
        var message = "You are now logged in as:" + mobileServicesUser.userId;
        var dialog = new Windows.UI.Popups.MessageDialog(message, title);
        dialog.showAsync().done(complete);                                
                    });                       
        }, function (error) {                        
        session = null;
        var dialog = new Windows.UI.Popups.MessageDialog("You must log in.", "Login Required");
        dialog.showAsync().done(complete);                        
                });
            });
        }

        var authenticate = function () {
        // Force a logout to make it easier to test with multiple Microsoft Accounts
        logout().then(login).then(function () {
        if (session === null) {
        // Authentication failed, try again.
        authenticate();
                }
            });
        }

        WL.init({
        redirect_uri:"<< INSERT REDIRECT DOMAIN HERE >>"
        });           

        authenticate();

    此代码初始化 Live Connect 客户端，强制注销，向 Live Connect 发送一个新的登录请求，将返回的身份验证令牌发送到移动服务，然后显示有关已登录用户的信息。

    <div class="dev-callout"><b>说明</b>

    <p>在可能情况下，此代码强制注销以确保每次应用程序运行时都提示用户提供凭据。这样便于使用不同 Microsoft 帐户测试应用程序以确保身份验证正常执行。此机制仅在已登录用户没有已连接的 Microsoft 帐户时正常工作。</p>
	</div>

6.  使用在 Live Connect 中设置应用程序时指定的重定向域（采用 "<https://_service-name_.azure-mobile.net/>" 格式）更新上一步中的 *\<\< INSERT REDIRECT DOMAIN HERE \>\>* 字符串。

7.  按 F5 键运行应用程序，并使用 Microsoft 帐户登录 Live Connect。

    当你成功登录时，应用程序应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。

<a name="next-steps"> </a>
## 后续步骤

在下一教程[使用脚本为用户授权][]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。有关如何使用其他标识提供者进行身份验证的信息，请参阅[身份验证入门][12]。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/single-sign-on-wp8 "Windows Phone"
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-js/
  [注册应用程序以进行身份验证并配置移动服务]: #register
  [将表权限限制给已经过身份验证的用户]: #permissions
  [向应用程序添加身份验证]: #add-authentication
  [Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started
  [“提交应用程序”页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
  []: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-services-submit-win8-app.png
  [1]: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-services-win8-app-name.png
  [2]: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-services-store-association.png
  [3]: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-services-select-app-name.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [4]: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-services-selection.png
  [5]: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-service-uri.png
  [我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
  [6]: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-live-connect-apps-list.png
  [7]: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-live-connect-app-api-settings.png
  [8]: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-identity-tab-ma-only.png
  [9]: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-portal-data-tables.png
  [10]: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-portal-change-table-perms.png
  [11]: ./media/mobile-services-windows-store-javascript-single-sign-on/mobile-add-reference-live-js.png
  [使用脚本为用户授权]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-js
  [12]: /zh-cn/develop/mobile/tutorials/get-started-with-users-js
