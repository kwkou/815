<properties
	pageTitle="身份验证入门 (Xamarin.iOS) - 移动服务"
	description="了解如何在 Xamarin.iOS 的 Azure 移动服务应用程序中使用身份验证。"
	documentationCenter="xamarin"
	services="mobile-services"
	manager="dwrede"
	authors="lindydonna"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="02/11/2016"
	wacn.date="05/23/2016"/>

# 向移动服务应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]


本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1. [注册应用程序以进行身份验证并配置移动服务]
2. [将表权限限制给已经过身份验证的用户]
3. [向应用程序添加身份验证]

本教程基于移动服务快速入门。此外，还必须先完成[移动服务入门]教程。

完成本教程需要安装 [Xamarin Studio]、XCode 6.0 和 iOS 7.0 或更高版本。

## <a name="register"></a>注册应用程序以进行身份验证并配置移动服务

[AZURE.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)]

## <a name="permissions"></a>将权限限制给已经过身份验证的用户


[AZURE.INCLUDE [mobile-services-restrict-permissions-javascript-backend](../includes/mobile-services-restrict-permissions-javascript-backend.md)]


3. 在 Xcode 中，打开你在完成[移动服务入门]教程时创建的项目。 

2. 在 iPhone 模拟器中按“运行”按钮以生成项目并启动应用程序；验证启动该应用程序后，是否会引发状态代码为 401（“未授权”）的未处理异常。
   
   	发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 _TodoItem_ 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

## <a name="add-authentication"></a>向应用程序添加身份验证

1. 打开 **QSToDoService** 项目文件并添加以下变量

		// Mobile Service logged in user
		private MobileServiceUser user; 
		public MobileServiceUser User { get { return user; } }

2. 然后，向 **TodoService** 添加一个名为 **Authenticate** 的新方法，其定义为：

        private async Task Authenticate(MonoTouch.UIKit.UIViewController view)
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

	> [AZURE.NOTE]如果使用的标识提供程序不是 Microsoft 帐户，请将传递给上述 **LoginAsync** 方法的值更改为下列其中一项：_WindowsAzureActiveDirectory_。

3. 从 **ToDoService** 构造函数将对 **ToDoItem** 表的请求移到名为 **CreateTable** 的新方法中：

        private async Task CreateTable()
        {
            // Create an MSTable instance to allow us to work with the ToDoItem table
            todoTable = client.GetSyncTable<ToDoItem>();
        }
	
4. 创建一个名为 **LoginAndGetData** 的新异步公共方法，其定义为：

        public async Task LoginAndGetData(MonoTouch.UIKit.UIViewController view)
        {
            await Authenticate(view);
            await CreateTable();
        }

5. 在 **TodoListViewController** 中，重写 **ViewDidAppear** 方法，并按如下所示定义它。如果 **TodoService** 还没有用户的句柄，则此代码让用户登录：

        public override async void ViewDidAppear(bool animated)
        {
            base.ViewDidAppear(animated);

            if (QSTodoService.DefaultService.User == null)
            {
                await QSTodoService.DefaultService.LoginAndGetData(this);
            }

            if (QSTodoService.DefaultService.User == null)
            {
                // TODO:: show error
                return;
            }


            await RefreshAsync();
        }
6. 从 **TodoListViewController.ViewDidLoad** 中删除对 **RefreshAsync** 的原始调用。
		
7. 按“运行”按钮以生成项目，在 iPhone 模拟器中启动应用，然后使用你选择的标识提供者登录。

   	当你成功登录时，应用应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。

## 获取已完成的示例
下载[已完成的示例项目]。请务必使用你自己的 Azure 设置更新 **applicationURL** 和 **applicationKey** 变量。

## <a name="next-steps"></a>后续步骤

在下一教程[使用脚本为用户授权]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。

<!-- Anchors. -->
[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制给已经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[Next Steps]:#next-steps

<!-- Images. -->
[4]: ./media/partner-xamarin-mobile-services-ios-get-started-users/mobile-services-selection.png
[5]: ./media/partner-xamarin-mobile-services-ios-get-started-users/mobile-service-uri.png
[13]: ./media/partner-xamarin-mobile-services-ios-get-started-users/mobile-identity-tab.png
[14]: ./media/partner-xamarin-mobile-services-ios-get-started-users/mobile-portal-data-tables.png
[15]: ./media/partner-xamarin-mobile-services-ios-get-started-users/mobile-portal-change-table-perms.png

<!-- URLs. TODO:: update completed example project link with project download -->
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253

[移动服务入门]: /documentation/articles/partner-xamarin-mobile-services-ios-get-started/
[Get started with data]: /documentation/articles/partner-xamarin-mobile-services-ios-get-started-data/
[Get started with authentication]: /documentation/articles/partner-xamarin-mobile-services-ios-get-started-users/
[Get started with push notifications]: /documentation/articles/partner-xamarin-mobile-services-ios-get-started-push/
[使用脚本为用户授权]: /documentation/articles/mobile-services-javascript-backend-service-side-authorization/

[已完成的示例项目]: http://go.microsoft.com/fwlink/p/?LinkId=331328
[Xamarin Studio]: http://xamarin.com/download

<!---HONumber=Mooncake_0118_2016-->