<properties
	pageTitle="Xamarin.Forms 应用中的移动应用身份验证入门 | Azure"
	description="了解如何使用移动应用通过各种标识提供者（包括 AAD 和 Microsoft）对 Xamarin Forms 应用的用户进行身份验证。"
	services="app-service\mobile"
	documentationCenter="xamarin"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-xamarin"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="06/16/2016"
	wacn.date="09/26/2016"
	ms.author="adrianha"/>

# 向 Xamarin.Forms 应用添加身份验证

[AZURE.INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

##概述

本主题演示如何从客户端应用程序对应用服务移动应用的用户进行身份验证。在本教程中，使用应用服务支持的标识提供者向 Xamarin.Forms 快速入门项目添加身份验证。移动应用成功进行身份验证和授权后，将显示用户 ID 值，该用户将能够访问受限制的表数据。

##先决条件

为了使本教程达到最佳效果，建议用户先完成[创建 Xamarin.Forms 应用](/documentation/articles/app-service-mobile-xamarin-forms-get-started/)教程。完成此教程后，用户将获得一个 Xamarin.Forms 项目，它是一个多平台 TodoList 应用。

如果不使用下载的快速入门服务器项目，必须将身份验证扩展包添加到项目。有关服务器扩展包的详细信息，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

##注册应用以进行身份验证并配置应用服务

[AZURE.INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

##将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]


##向可移植类库添加身份验证

移动应用使用 [MobileServiceClient] 的 [LoginAsync] 扩展方法通过应用服务身份验证登录用户。此示例使用服务器托管的身份验证流，在应用中显示提供程序的登录界面。有关详细信息，请参阅[服务器托管的身份验证](/documentation/articles/app-service-mobile-dotnet-how-to-use-client-library/#serverflow)。若要在生产应用中提供更好的用户体验，可以考虑改用[客户端托管的身份验证](/documentation/articles/app-service-mobile-dotnet-how-to-use-client-library/#clientflow)。

若要使用 Xamarin.Forms 项目进行身份验证，请在可移植类库中为应用定义 **IAuthenticate** 接口。还更新可移植类库中定义的用户界面，添加“登录”按钮，用户单击该按钮即可开始进行身份验证。身份验证成功后，将从移动应用后端加载数据。

必须为应用支持的每个平台实现 **IAuthenticate** 接口。


1. 在 Visual Studio 或 Xamarin Studio 中，从名称中包含 **Portable** 的项目（该项目是可移植类库项目）中打开 App.cs，然后添加以下 `using` 语句：

		using System.Threading.Tasks;

2. 在 App.cs 中，在 `App` 类定义前添加以下 `IAuthenticate` 接口定义。

	    public interface IAuthenticate
	    {
	        Task<bool> Authenticate();
	    }

3. 向 **App** 类添加以下静态成员，以使用平台特定的实现初始化接口。

	    public static IAuthenticate Authenticator { get; private set; }

        public static void Init(IAuthenticate authenticator)
        {
            Authenticator = authenticator;
        }

4. 从可移植类库项目中打开 TodoList.xaml，在 *buttonsPanel* 布局元素中现有按钮之后添加以下 **Button** 元素：

      	<Button x:Name="loginButton" Text="Sign-in" MinimumHeightRequest="30" 
			Clicked="loginButton_Clicked"/>

	此按钮将通过移动应用后端触发服务器托管的身份验证。

5. 从可移植类库项目中打开 TodoList.xaml.cs，然后将以下字段添加到 `TodoList` 类：

		// Track whether the user has authenticated. 
        bool authenticated = false;


6. 将 **OnAppearing** 方法替换为以下代码：

	    protected override async void OnAppearing()
	    {
	        base.OnAppearing();
	
	        // Refresh items only when authenticated.
	        if (authenticated == true)
	        {
	            // Set syncItems to true in order to synchronize the data 
	            // on startup when running in offline mode.
	            await RefreshItems(true, syncItems: false);

				// Hide the Sign-in button.
                this.loginButton.IsVisible = false;
	        }
	    }

	这可确保仅在用户经过身份验证后，才从服务刷新数据。

7. 在 **TodoList** 类中为 **Clicked** 事件添加以下处理程序：

        async void loginButton_Clicked(object sender, EventArgs e)
        {
            if (App.Authenticator != null)
                authenticated = await App.Authenticator.Authenticate();

            // Set syncItems to true to synchronize the data on startup when offline is enabled.
            if (authenticated == true)
                await RefreshItems(true, syncItems: false);
        }

8. 保存更改，重新生成可移植类库项目，并验证没有错误。


##向 Android 应用添加身份验证

本部分演示如何在 Android 应用项目中实现 **IAuthenticate** 接口。如果不要支持 Android 设备，请跳过本部分。

1. 在 Visual Studio 或 Xamarin Studio 中，右键单击 **droid** 项目，然后单击“设为启动项目”。

2. 按 F5 在调试器中启动项目，然后验证启动该应用后，是否会引发状态代码为 401（“未授权”）的未处理异常。之所以会发生此异常是因为对后端的访问仅限于授权用户。

3. 在 Android 项目中打开 MainActivity.cs，并添加以下 `using` 语句：

		using Microsoft.WindowsAzure.MobileServices;
		using System.Threading.Tasks;

4. 更新 **MainActivity** 类，以实现 **IAuthenticate** 接口，如下所示：

		public class MainActivity : global::Xamarin.Forms.Platform.Android.FormsApplicationActivity, IAuthenticate


5. 通过添加 **MobileServiceUser** 字段和 **IAuthenticate** 接口所需的 **Authenticate** 方法，更新 **MainActivity** 类，如下所示：

        // Define a authenticated user.
        private MobileServiceUser user;

        public async Task<bool> Authenticate()
        {
            var success = false;
            var message = string.Empty;
            try
            {
                // Sign in with Microsoft login using a server-managed flow.
                user = await TodoItemManager.DefaultManager.CurrentClient.LoginAsync(this,
                    MobileServiceAuthenticationProvider.Microsoft);
                if (user != null)
                {
                    message = string.Format("you are now signed-in as {0}.",
                        user.UserId);
                    success = true;
                }
            }
            catch (Exception ex)
            {
                message = ex.Message;
            }

            // Display the success or failure message.
            AlertDialog.Builder builder = new AlertDialog.Builder(this);
            builder.SetMessage(message);
            builder.SetTitle("Sign-in result");
            builder.Create().Show();

            return success;
        }


	如果使用的是 Microsoft 以外的其他标识提供者，请为 [MobileServiceAuthenticationProvider] 选择不同的值。

6. 在 **MainActivity** 类的 **OnCreate** 方法中调用 `LoadApplication()` 之前添加以下代码：

        // Initialize the authenticator before loading the app.
        App.Init((IAuthenticate)this);

	这可确保验证器在应用加载前进行初始化。

7. 重新生成应用，运行它，然后使用所选的身份验证提供者登录，并验证是否能够以经过身份验证的用户身份访问数据。

##向 iOS 应用添加身份验证

本部分演示如何在 iOS 应用项目中实现 **IAuthenticate** 接口。如果不要支持 iOS 设备，请跳过本部分。

1. 在 Visual Studio 或 Xamarin Studio 中，右键单击 **iOS** 项目，然后单击“设为启动项目”。

2. 按 F5 在调试器中启动项目，然后验证启动该应用后，是否会引发状态代码为 401（“未授权”）的未处理异常。之所以会发生此异常是因为对后端的访问仅限于授权用户。

4. 打开 iOS 项目中的 AppDelegate.cs，并添加以下 `using` 语句：

		using Microsoft.WindowsAzure.MobileServices;
		using System.Threading.Tasks;

4. 更新 **AppDelegate** 类，以实现 **IAuthenticate** 接口，如下所示：

		public partial class AppDelegate : global::Xamarin.Forms.Platform.iOS.FormsApplicationDelegate, IAuthenticate

5. 通过添加 **MobileServiceUser** 字段和 **IAuthenticate** 接口所需的 **Authenticate** 方法，更新 **AppDelegate** 类，如下所示：

        // Define a authenticated user.
        private MobileServiceUser user;

        public async Task<bool> Authenticate()
        {
            var success = false;
            var message = string.Empty;
            try
            {
                // Sign in with Microsoft login using a server-managed flow.
                if (user == null)
                {
                    user = await TodoItemManager.DefaultManager.CurrentClient
                        .LoginAsync(UIApplication.SharedApplication.KeyWindow.RootViewController,
                        MobileServiceAuthenticationProvider.Microsoft);
                    if (user != null)
                    {
                        message = string.Format("You are now signed-in as {0}.", user.UserId);
                        success = true;                        
                    }
                }        
            }
            catch (Exception ex)
            {
               message = ex.Message;
            }

            // Display the success or failure message.
            UIAlertView avAlert = new UIAlertView("Sign-in result", message, null, "OK", null);
            avAlert.Show();         

            return success;
        }

	如果使用的是 Microsoft 以外的其他标识提供者，请为 [MobileServiceAuthenticationProvider] 选择不同的值。

6. 在 **FinishedLaunching** 方法中调用 `LoadApplication()` 之前添加以下代码行：

        App.Init(this);

	这可确保验证器在加载应用前进行初始化。

7. 重新生成应用，运行它，然后使用所选的身份验证提供者登录，并验证是否能够以经过身份验证的用户身份访问数据。


##向 Windows 应用项目添加身份验证

本部分演示如何在 Windows 8.1 和 Windows Phone 8.1 应用项目中实现 **IAuthenticate** 接口。这些步骤同样适用于通用 Windows 平台 (UWP) 项目。如果不要支持 Windows 设备，请跳过本部分。

1. 在 Visual Studio 中，右键单击 **WinApp** 或 **WinPhone81** 项目，然后单击“设为启动项目”。

2. 按 F5 在调试器中启动项目，然后验证启动该应用后，是否会引发状态代码为 401（“未授权”）的未处理异常。之所以会发生此异常是因为对后端的访问仅限于授权用户。

3. 打开 Windows 应用项目的 MainPage.xaml.cs，并添加以下 `using` 语句：

		using Microsoft.WindowsAzure.MobileServices;
		using System.Threading.Tasks;
		using Windows.UI.Popups;
		using <your_Portable_Class_Library_namespace>;

	将 `<your_Portable_Class_Library_namespace>` 替换为可移植类库的命名空间。

4. 更新 **MainPage** 类，以实现 **IAuthenticate** 接口，如下所示：

	    public sealed partial class MainPage : IAuthenticate


5. 通过添加 **MobileServiceUser** 字段和 **IAuthenticate** 接口所需的 **Authenticate** 方法，更新 **MainPage** 类，如下所示：

        // Define a authenticated user.
        private MobileServiceUser user;

        public async Task<bool> Authenticate()
        {
            string message = string.Empty;
            var success = false;

            try
            {
                // Sign in with Microsoft login using a server-managed flow.
                if (user == null)
                {
                    user = await TodoItemManager.DefaultManager.CurrentClient
                        .LoginAsync(MobileServiceAuthenticationProvider.Microsoft);
                    if (user != null)
                    {
						success = true;
                        message = string.Format("You are now signed-in as {0}.", user.UserId);
                    }
                }
                
            }
            catch (Exception ex)
            {
                message = string.Format("Authentication Failed: {0}", ex.Message);
            }

			// Display the success or failure message.
            await new MessageDialog(message, "Sign-in result").ShowAsync();

            return success;
        }


	如果使用的是 Microsoft 以外的其他标识提供者，请为 [MobileServiceAuthenticationProvider] 选择不同的值。

6. 在 **MainPage** 类的构造函数中调用 `LoadApplication()` 之前添加以下代码行：

        // Initialize the authenticator before loading the app.
        <your_Portable_Class_Library_namespace>.App.Init(this);
 
    将 `<your_Portable_Class_Library_namespace>` 替换为可移植类库的命名空间。如果这是 WinApp 项目，可以跳到步骤 8。下一步仅适用于 WinPhone81 项目，其中需要完成登录回调。

7. （可选）在 **WinPhone81** 应用项目中，打开 App.xaml.cs 并添加以下 `using` 语句：

		using Microsoft.WindowsAzure.MobileServices;
		using <your_Portable_Class_Library_namespace>;

	将 `<your_Portable_Class_Library_namespace>` 替换为可移植类库的命名空间。

8.  在 **App** 类中添加以下 **OnActivated** 方法重写：

		protected override void OnActivated(IActivatedEventArgs args)
		{
		    base.OnActivated(args);

			// We just need to handle activation that occurs after web authentication. 
		    if (args.Kind == ActivationKind.WebAuthenticationBrokerContinuation)
		    {
				// Get the client and call the LoginComplete method to complete authentication.
		        var client = TodoItemManager.DefaultManager.CurrentClient as MobileServiceClient;
		        client.LoginComplete(args as WebAuthenticationBrokerContinuationEventArgs);
		    }
		}

	如果该方法重写已存在，只需添加上面的代码段中的条件代码。

7. 重新生成应用，运行它，然后使用所选的身份验证提供者登录，并验证是否能够以经过身份验证的用户身份访问数据。

##后续步骤

完成此基本身份验证教程后，请考虑继续学习以下教程之一：

+ [向应用添加推送通知](/documentation/articles/app-service-mobile-xamarin-forms-get-started-push/) 了解如何为应用添加推送通知支持，以及如何将移动应用后端配置为使用 Azure 通知中心发送推送通知。

+ [为应用启用脱机同步](/documentation/articles/app-service-mobile-xamarin-forms-get-started-offline-data/) 了解如何使用移动应用后端向应用添加脱机支持。脱机同步允许最终用户与移动应用交互（查看、添加或修改数据），即使在没有网络连接时也是如此。

<!-- Images. -->

<!-- URLs. -->
[apns object]: http://go.microsoft.com/fwlink/p/?LinkId=272333
[LoginAsync]: https://msdn.microsoft.com/zh-cn/library/azure/dn268341(v=azure.10).aspx
[MobileServiceClient]: https://msdn.microsoft.com/zh-cn/library/azure/JJ553674(v=azure.10).aspx
[MobileServiceAuthenticationProvider]: https://msdn.microsoft.com/zh-cn/library/azure/jj730936(v=azure.10).aspx

<!---HONumber=Mooncake_0919_2016-->