<properties 
	pageTitle="用于 Xamarin iOS 应用的移动服务中的身份验证入门 | Azure"
	description="了解如何使用移动服务通过各种标识提供程序（包括 Microsoft 和 Azure Active Directory 对 Xamarin iOS 应用程序的用户进行身份验证。" 
	services="mobile-services" 
	documentationCenter="xamarin" 
	authors="lindydonna" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="05/11/2016" 
	wacn.date="06/13/2016"/>

# 向移动服务应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]


本主题说明如何通过应用对移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1. [注册应用程序以进行身份验证并配置移动服务]
2. [将表权限限制给已经过身份验证的用户]
3. [向应用程序添加身份验证]

本教程基于移动服务快速入门。此外，还必须先完成[移动服务入门]教程。

##<a name="register"></a>注册应用程序以进行身份验证并配置移动服务

[AZURE.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)]

[AZURE.INCLUDE [mobile-services-dotnet-backend-aad-server-extension](../includes/mobile-services-dotnet-backend-aad-server-extension.md)]

##<a name="permissions"></a>将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [mobile-services-restrict-permissions-dotnet-backend](../includes/mobile-services-restrict-permissions-dotnet-backend.md)]

&nbsp;&nbsp;&nbsp;6.在 Visual Studio 或 Xamarin Studio 中，运行设备或模拟器中的客户端项目。验证在应用程序启动后是否引发状态代码为 401（“未授权”）的未处理异常。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;发生此异常的原因是应用尝试以未经身份验证的用户身份访问移动服务，但 *TodoItem* 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

##<a name="add-authentication"></a>向应用程序添加身份验证

在本部分中，你将修改应用程序，以便在显示数据之前显示登录屏幕。当应用启动时，它将不会连接到移动服务，并且不会显示任何数据。用户首次执行刷新笔势后，将显示登录屏幕；成功登录后，将显示 Todo 项列表。

1. 在客户端项目中，打开文件 **QSTodoService.cs** 并将以下声明添加到 QSTodoService：

		// Mobile Service logged in user
		private MobileServiceUser user; 
		public MobileServiceUser User { get { return user; } }

2. 使用以下定义向 **QSTodoService** 添加新方法 **Authenticate**：

        private async Task Authenticate(UIViewController view)
        {
            try
            {
                user = await client.LoginAsync(view, MobileServiceAuthenticationProvider.Facebook);
            }
            catch (Exception ex)
            {
                Console.Error.WriteLine (@"ERROR - AUTHENTICATION FAILED {0}", ex.Message);
            }
        }

	> [AZURE.NOTE] 如果使用的标识提供者不是 Facebook，请将传递给上述 **LoginAsync** 的值更改为下列其中一项：_MicrosoftAccount_ 或 _WindowsAzureActiveDirectory_。

3. 打开 **QSTodoListViewController.cs**，并修改 **ViewDidLoad** 的方法定义以删除或注释禁止接近结尾处对 **RefreshAsync()** 的调用。

4. 在 **RefreshAsync** 方法定义的顶部添加以下代码：

		// Add at the start of the RefreshAsync method.
		if (todoService.User == null) {
			await QSTodoService.DefaultService.Authenticate (this);
			if (todoService.User == null) {
				Console.WriteLine ("You must sign in.");
				return;
			}
		}
		
	这会在“User”属性为 null 时显示登录屏幕来尝试进行身份验证。登录成功时，“User”即设置完毕。

5. 按“运行”按钮以生成项目，并在 iPhone 模拟器中启动应用程序。验证应用程序是否未显示任何数据。此时尚未调用 **RefreshAsync()**。

6. 通过下拉项列表来执行刷新手势（将调用 **RefreshAsync()**）。这将调用 **Authenticate()** 来启动身份验证，并显示登录屏幕。当你验证成功之后，应用会显示待办事项列表，你可以对数据进行更新。

## <a name="next-steps"></a>后续步骤

在下一教程[移动服务用户的服务端授权][Authorize users with scripts]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。

<!-- Anchors. -->

[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制给已经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[Next Steps]: #next-steps


<!-- URLs. -->

[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-xamarin-ios-get-started/
[Get started with authentication]: /documentation/articles/mobile-services-dotnet-backend-xamarin-ios-get-started-users/
[Get started with push notifications]: /documentation/articles/mobile-services-dotnet-backend-xamarin-ios-get-started-push/
[Authorize users with scripts]: /documentation/articles/mobile-services-dotnet-backend-service-side-authorization/
[JavaScript and HTML]: /documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-users/
[Azure Management Portal]: https://manage.windowsazure.cn/

<!---HONumber=Mooncake_0118_2016-->