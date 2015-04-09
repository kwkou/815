<properties linkid="develop-mobile-tutorials-get-started-with-users-html" urlDisplayName="身份验证入门 (HTML5)" pageTitle="身份验证入门 (HTML5) | 移动开发人员中心" metaKeywords="" description="了解如何使用移动服务通过各种标识提供程序（包括 Google、Facebook、Twitter 和 Microsoft）对 HTML 应用程序的用户进行身份验证。" metaCanonical="" services="" documentationCenter="Mobile" title="Get started with authentication in Mobile Services" authors="glenga" solutions="" manager="" editor="" />



# 向移动服务应用程序添加身份验证 

[WACOM.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]

本主题说明如何通过 HTML 或 PhoneGap 应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。  

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1. [注册应用程序以进行身份验证并配置移动服务]
2. [将表权限限制为提供给经过身份验证的用户]
3. [向应用程序添加身份验证]

本教程基于移动服务快速入门。因此，你还必须先完成[移动服务入门]教程。 

## <a name="register"></a>注册应用程序以进行身份验证并配置移动服务

[WACOM.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)] 

## <a name="permissions"></a>将权限限制给已经过身份验证的用户

[WACOM.INCLUDE [mobile-services-restrict-permissions-javascript-backend](../includes/mobile-services-restrict-permissions-javascript-backend.md)] 


3. 在 app 目录中，启动 **server** 子文件夹中的下列命令文件之一。

	+ **launch-windows**（Windows 计算机） 
	+ **launch-mac.command**（Mac OS X 计算机）
	+ **launch-linux.sh**（Linux 计算机）

	>[WACOM.NOTE]在 Windows 计算机上，当 PowerShell 要求你确认是否要运行脚本时，请键入"R"。你的 Web 浏览器可能会警告你不要运行该脚本，因为它是从 Internet 下载的。如果出现此警告，你必须请求浏览器继续加载该脚本。

	随后将在本地计算机上启动用于托管新应用程序的 Web 服务器。

2. 在 Web 浏览器中打开 URL <a href="http://localhost:8000/" target="_blank">http://localhost:8000/</a> 以启动应用程序。 

	无法加载数据。发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 _TodoItem_ 表现在要求身份验证。

3. （可选）打开 Web 浏览器的脚本调试程序，并重新加载页。检查是否发生了访问被拒绝错误。 

接下来，你需要更新应用程序，以允许在从移动服务请求资源之前进行身份验证。

## <a name="add-authentication"></a>向应用程序添加身份验证

>[WACOM.NOTE]因为在弹出窗口中执行登录，应通过按钮的单击事件调用 <strong>login</strong> 方法。否则，许多浏览器都会隐藏登录窗口。

1. 打开项目文件 index.html，找到 H1 元素，并在该元素的下面添加以下代码段：

	    <div id="logged-in">
            You are logged in as <span id="login-name"></span>.
            <button id="log-out">Log out</button>
        </div>
        <div id="logged-out">
            You are not logged in.
            <button>Log in</button>
        </div>

	这样，你便可以从该页登录到移动服务。

2. 在 app.js 文件中，找到位于文件最底部的、调用 refreshTodoItems 函数的代码行，并将它替换为以下代码： 
	
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

    这将会创建一组用于处理身份验证过程的函数。将使用 Facebook 登录对用户进行身份验证。如果你使用的标识提供程序不是 Facebook，请将传递给 <strong>login</strong> 方法的值更改为以下值之一： <em>microsoftaccount</em>、 <em>facebook</em>、 <em>twitter</em>、 <em>google</em>或 <em>aad</em>。

	>[WACOM.NOTE]在 PhoneGap 应用程序中，还必须向项目中添加以下插件：
	><ul><li><code>phonegap 插件添加 https://git-wip-us.apache.org/repos/asf/cordova-plugin-device.git</code></li>
	><li><code>phonegap 插件添加 https://git-wip-us.apache.org/repos/asf/cordova-plugin-inappbrowser.git</code></li></ul>

9. 返回到运行应用程序的浏览器并刷新页。 

	   当您成功登录时，应用应该运行而不出现错误，您应该能够查询移动服务，并对数据进行更新。

	>[WACOM.NOTE]如果使用 Internet Explorer，可能会在登录后收到错误： <code>无法访问窗口打开程序。它可能位于不同的 Internet Explorer 区域</code>。发生此错误的原因是因为弹出窗口在与本地主机 (Intranet) 不同的安全区域 (Internet) 上运行。这只在使用本地主机开发期间影响应用程序。一种解决方法是打开 Internet 选项的 <strong>"安全"</strong> 选项卡 <strong>，</strong>单击 <strong>"本地 Intranet"</strong>，单击 <strong>"站点"</strong>，然后禁用 <strong>"自动检测 Intranet 网络"</strong>。完成测试后，请记得将此设置更改回来。

## <a name="next-steps"> </a>后续步骤

在下一教程[使用脚本为用户授权]中，你将获取移动服务基于经过身份验证的用户提供的用户 ID 值并使用该值来筛选移动服务返回的数据。请在[移动服务 HTML/JavaScript 操作方法概念性参考]中了解有关如何将移动服务与 HTML/JavaScript 一起使用的详细信息

<!-- Anchors. -->
[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制为提供给经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[后续步骤]:#next-steps

<!-- Images. -->

[4]: ./media/mobile-services-html-get-started-users/mobile-services-selection.png
[5]: ./media/mobile-services-html-get-started-users/mobile-service-uri.png
[13]: ./media/mobile-services-html-get-started-users/mobile-identity-tab.png
[14]: ./media/mobile-services-html-get-started-users/mobile-portal-data-tables.png
[15]: ./media/mobile-services-html-get-started-users/mobile-portal-change-table-perms.png

<!-- URLs. -->
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-html-get-started
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-html-get-started-data
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-html-authorize-users-in-scripts

[Azure 管理门户]: https://manage.windowsazure.cn/
[移动服务 HTML/JavaScript 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-html-how-to-use-client-library
