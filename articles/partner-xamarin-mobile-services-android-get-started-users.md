<properties 
	pageTitle="身份验证入门 (Xamarin.Android) - 移动服务" 
	description="了解如何在 Xamarin.Android 的 Azure 移动服务应用程序中使用身份验证。" 
	services="mobile-services" 
	documentationCenter="xamarin" 
	manager="dwrede" 
	authors="lindydonna" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/11/2016" 
	wacn.date="05/23/2016"/>

#  向移动服务应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]

<p>本主题说明如何通过 Xamarin.Android 应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。</p>

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1. [注册应用程序以进行身份验证并配置移动服务]
2. [将表权限限制给已经过身份验证的用户]
3. [向应用程序添加身份验证]

本教程基于移动服务快速入门。此外，还必须先完成 [移动服务入门] 教程。

完成此教程要求在 Windows 上安装 Visual Studio with Xamarin，或在 Mac OS X 上安装 Xamarin Studio。[设置和安装 Visual Studio 和 Xamarin](https://msdn.microsoft.com/zh-cn/library/mt613162.aspx) 中提供了完整的安装说明。

## <a name="register"></a>注册应用程序以进行身份验证并配置移动服务

[AZURE.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)]

## <a name="permissions"></a>将权限限制给已经过身份验证的用户


[AZURE.INCLUDE [mobile-services-restrict-permissions-javascript-backend](../includes/mobile-services-restrict-permissions-javascript-backend.md)]


3. 在 Xamarin Studio 中，打开你在完成[移动服务入门]教程后创建的项目。

4. 在“运行”菜单中单击“开始调试”以启动应用；验证启动该应用后，是否会引发状态代码为 401（“未授权”）的未处理异常。

	 发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 _TodoItem_ 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

## <a name="add-authentication"></a>向应用程序添加身份验证

1. 将以下属性添加到 **ToDoActivity** 类：

		private MobileServiceUser user;

2. 将以下方法添加到 **ToDoActivity** 类：

	        private async Task Authenticate()
	        {
	            try
	            {
	                user = await client.LoginAsync(this, MobileServiceAuthenticationProvider.MicrosoftAccount);
	                CreateAndShowDialog(string.Format("you are now logged in - {0}", user.UserId), "Logged in!");
	            }
	            catch (Exception ex)
	            {
	                CreateAndShowDialog(ex, "Authentication failed");
	            }
	        }

    这将会创建一个用于处理身份验证过程的新方法。将使用 Microsoft 帐户登录对用户进行身份验证。此时将出现一个对话框，其中显示了已经过身份验证的用户的 ID。如果未正常完成身份验证，你将无法继续操作。

    > [AZURE.NOTE] 如果使用的标识提供者不是 Microsoft，请将传递给上述 **login** 方法的值更改为下列其中一项：_WindowsAzureActiveDirectory_。

3. 在 **OnCreate** 方法中，在实例化 `MobileServiceClient` 对象的代码后面添加以下代码行。

		await Authenticate();

	此调用启动身份验证过程，并以异步方式等待它。

4. 将 **OnCreate** 方法中 `await Authenticate();` 后面的剩余代码移到新的 **CreateTable** 方法，如下所示：

	        private async Task CreateTable()
	        {
            
            await InitLocalStoreAsync();

            // Get the Mobile Service Table instance to use
            toDoTable = client.GetSyncTable<ToDoItem>();

            textNewToDo = FindViewById<EditText>(Resource.Id.textNewToDo);

	            // Create an adapter to bind the items with the view
            adapter = new ToDoItemAdapter(this, Resource.Layout.Row_List_To_Do);
            var listViewTodo = FindViewById<ListView>(Resource.Id.listViewToDo);
	            listViewTodo.Adapter = adapter;

	            // Load the items from the Mobile Service
	            await RefreshItemsFromTableAsync();
	        }

5. 然后，在 **OnCreate** 中，在完成步骤 2 中添加的 **Authenticate** 调用之后，将调用新的 **CreateTable** 方法：

		await CreateTable();


6. 在“运行”菜单中单击“开始调试”以启动应用，然后使用所选的标识提供者登录。

   	当你成功登录时，应用应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。

##  获取已完成的示例
下载[已完成的示例项目]。请务必使用你自己的 Azure 设置更新 **applicationURL** 和 **applicationKey** 变量。

##  <a name="next-steps"></a>后续步骤

在下一教程[使用脚本为用户授权]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。

<!-- Anchors. -->
[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制给已经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[Next Steps]: #next-steps

<!-- Images. -->
[4]: ./media/partner-xamarin-mobile-services-android-get-started-users/mobile-services-selection.png
[5]: ./media/partner-xamarin-mobile-services-android-get-started-users/mobile-service-uri.png

[13]: ./media/partner-xamarin-mobile-services-android-get-started-users/mobile-identity-tab.png
[14]: ./media/partner-xamarin-mobile-services-android-get-started-users/mobile-portal-data-tables.png
[15]: ./media/partner-xamarin-mobile-services-android-get-started-users/mobile-portal-change-table-perms.png

<!-- URLs. -->
[移动服务入门]: /documentation/articles/partner-xamarin-mobile-services-android-get-started/
[使用脚本为用户授权]: /documentation/articles/mobile-services-javascript-backend-service-side-authorization/

[已完成的示例项目]: http://go.microsoft.com/fwlink/p/?LinkId=331328

<!---HONumber=Mooncake_0118_2016-->