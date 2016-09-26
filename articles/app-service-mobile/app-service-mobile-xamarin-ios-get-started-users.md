<properties
	pageTitle="Xamarin iOS 应用中的移动应用身份验证入门"
	description="了解如何使用移动应用通过各种标识提供者（包括 AAD 和 Microsoft）对 Xamarin iOS 应用的用户进行身份验证。"
	services="app-service\mobile"
	documentationCenter="xamarin"
	authors="mattchenderson"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.date="06/28/2016"
	wacn.date="09/26/2016"/>

# 向 Xamarin.iOS 应用添加身份验证

[AZURE.INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

本主题演示如何从客户端应用程序对应用服务移动应用的用户进行身份验证。在本教程中，使用应用服务支持的标识提供者向 Xamarin.iOS 快速入门项目添加身份验证。移动应用成功进行身份验证和授权后，将显示用户 ID 值，该用户将能够访问受限制的表数据。

必须先完成教程[创建 Xamarin.iOS 应用]。如果不使用下载的快速入门服务器项目，必须将身份验证扩展包添加到项目。有关服务器扩展包的详细信息，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

##注册应用以进行身份验证并配置应用服务

[AZURE.INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

##将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

&nbsp;&nbsp;4.在 Visual Studio 或 Xamarin Studio 中，运行设备或模拟器中的客户端项目。验证在应用程序启动后是否引发状态代码为 401（“未授权”）的未处理异常。失败将记录到调试器的控制台中。因此，在 Visual Studio 中，应在输出窗口中看到失败。

发生此未授权失败的原因是应用尝试以未经身份验证的用户身份访问移动应用后端。*TodoItem* 表现在要求身份验证。

接下来，更新客户端应用，以使用经过身份验证的用户从移动应用后端请求资源。

##向应用程序添加身份验证

在本部分中，你将修改应用程序，以便在显示数据之前显示登录屏幕。应用启动时，它不会连接到应用服务，并且不会显示任何数据。用户首次执行刷新笔势后，将显示登录屏幕；成功登录后，将显示 Todo 项列表。

1. 在客户端项目中，打开文件 **QSTodoService.cs**，在 QSTodoService 类中添加以下 using 语句和带访问器的 `MobileServiceUser`：

	```
		using UIKit;
	```

		// Logged in user
		private MobileServiceUser user;
		public MobileServiceUser User { get { return user; } }

2. 使用以下定义向 **QSTodoService** 添加名为 **Authenticate** 的新方法：


        public async Task Authenticate(UIViewController view)
        {
            try
            {
                user = await client.LoginAsync(view, MobileServiceAuthenticationProvider.Microsoft);
            }
            catch (Exception ex)
            {
                Console.Error.WriteLine (@"ERROR - AUTHENTICATION FAILED {0}", ex.Message);
            }
        }

	>[AZURE.NOTE] 如果使用的标识提供者不是 Microsoft，请将传递给上述 **LoginAsync** 的值更改为下列其中一项：_MicrosoftAccount_ 或 _WindowsAzureActiveDirectory_。

3. 打开 **QSTodoListViewController.cs**。修改 **ViewDidLoad** 的方法定义，删除接近结尾处对 **RefreshAsync()** 的调用：

		public override async void ViewDidLoad ()
		{
			base.ViewDidLoad ();

			todoService = QSTodoService.DefaultService;
           await todoService.InitializeStoreAsync ();

           RefreshControl.ValueChanged += async (sender, e) => {
                await RefreshAsync ();
           }

			// Comment out the call to RefreshAsync
			// await RefreshAsync ();
		}


4. 修改方法 **RefreshAsync**，以便在 **User** 属性为 null 时进行身份验证。将以下代码添加到方法定义顶部：

		// start of RefreshAsync method
		if (todoService.User == null) {
			await QSTodoService.DefaultService.Authenticate (this);
			if (todoService.User == null) {
				Console.WriteLine ("couldn't login!!");
				return;
			}
		}
		// rest of RefreshAsync method

5. 在已连接到 Mac 上的 Xamarin 生成主机的 Visual Studio 或 Xamarin Studio 中，针对设备或模拟器运行客户端项目。验证应用程序是否未显示任何数据。

	通过向下拉动项列表来执行刷新笔势，这将导致显示登录屏幕。成功输入有效的凭据后，应用将显示待办事项列表，用户可以对数据进行更新。


<!-- URLs. -->
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[创建 Xamarin.iOS 应用]: /documentation/articles/app-service-mobile-xamarin-ios-get-started/

<!---HONumber=Mooncake_0919_2016-->