<properties 
	pageTitle="用于 Xamarin Android 应用的移动服务中的身份验证入门 | Azure"
	description="了解如何使用移动服务通过各种标识提供程序（包括 Microsoft 和 Azure Active Directory 对 Xamarin Android 应用程序的用户进行身份验证。" 
	services="mobile-services" 
	documentationCenter="xamarin" 
	authors="lindydonna" 
	manager="dwrede" 
	editor="mollybos"/>

<tags 
	ms.service="mobile-services" 
	ms.date="05/11/2015" 
	wacn.date="06/13/2016"/>

# 移动服务中的身份验证入门

[AZURE.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]


本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

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

<ol start="6">
<li><p>在 Visual Studio 或 Xamarin Studio 中，运行设备或模拟器中的客户端项目。验证在应用程序启动后是否引发状态代码为 401（“未授权”）的未处理异常。</p>
   
   	<p>发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 <em>TodoItem</em> 表现在要求身份验证。</p></li>
</ol>

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

##<a name="add-authentication"></a>向应用程序添加身份验证

1. 将以下属性添加到 **TodoActivity** 类：

			private MobileServiceUser user;

2. 将以下方法添加到 **ToDoActivity** 类：

	        private async Task Authenticate()
	        {
	            try
	            {
	                user = await client.LoginAsync(this, MobileServiceAuthenticationProvider.Facebook);
	                CreateAndShowDialog(string.Format("you are now logged in - {0}", user.UserId), "Logged in!");
	            }
	            catch (Exception ex)
	            {
	                CreateAndShowDialog(ex, "Authentication failed");
	            }
	        }

    这将会创建一个用于处理身份验证过程的新方法。将使用 Facebook 登录对用户进行身份验证。此时将出现一个对话框，其中显示了已经过身份验证的用户的 ID。

    > [AZURE.NOTE]如果使用的标识提供程序不是 Facebook，请将传递给上述 **LoginAsync** 方法的值更改为下列其中一项：_MicrosoftAccount_、_Twitter_、_Google_ 或 _WindowsAzureActiveDirectory_。

3. 在 **OnCreate** 方法中，在实例化 `MobileServiceClient` 对象的代码后面添加以下代码行。

		await Authenticate(); // add this line

	此调用启动身份验证过程，并以异步方式等待它。


4. 在“运行”菜单中单击“开始调试”以启动应用，然后使用所选的标识提供者登录。

   	当你成功登录时，应用应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。


<!-- ## <a name="next-steps"> </a>Next steps

In the next tutorial, [Service-side authorization of Mobile Services users][Authorize users with scripts], you will take the user ID value provided by Mobile Services based on an authenticated user and use it to filter the data returned by Mobile Services. 
 -->
<!-- Anchors. -->

[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制给已经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[Next Steps]: #next-steps


<!-- URLs. -->

[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-xamarin-android-get-started/
[Get started with authentication]: /documentation/articles/mobile-services-dotnet-backend-xamarin-android-get-started-users/

[Authorize users with scripts]: /documentation/articles/mobile-services-dotnet-backend-service-side-authorization/
[JavaScript and HTML]: /documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-users/
[Azure Management Portal]: https://manage.windowsazure.cn/

<!---HONumber=Mooncake_0118_2016-->