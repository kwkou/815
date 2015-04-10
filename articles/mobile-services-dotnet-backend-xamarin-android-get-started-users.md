<properties urlDisplayName="Get Started with authentication in Mobile Services for Xamarin Android apps" pageTitle="用于 Xamarin Android 应用程序的移动服务中的身份验证入门 - Azure 移动服务" metaKeywords="" description="了解如何使用移动服务通过各种标识提供程序（包括 Google、Facebook、Twitter 和 Microsoft）对 Xamarin Android 应用程序的用户进行身份验证。" metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Get Started with authentication in Mobile Services" authors="donnam" solutions="" manager="dwrede" editor="mollybos" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-xamarin-android" ms.devlang="dotnet" ms.topic="article" ms.date="09/23/2014" ms.author="donnam" />

# 移动服务中的身份验证入门

[WACOM.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]

本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1. [注册应用程序以进行身份验证并配置移动服务]
2. [将表权限限制为提供给经过身份验证的用户]
3. [向应用程序添加身份验证]

本教程基于移动服务快速入门。因此，你还必须先完成[移动服务入门]教程。 

## <a name="register"></a>注册应用程序以进行身份验证并配置移动服务

[WACOM.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)] 

[WACOM.INCLUDE [mobile-services-dotnet-backend-aad-server-extension](../includes/mobile-services-dotnet-backend-aad-server-extension.md)] 

## <a name="permissions"></a>将权限限制为提供给经过身份验证的用户

[WACOM.INCLUDE [mobile-services-restrict-permissions-dotnet-backend](../includes/mobile-services-restrict-permissions-dotnet-backend.md)] 

<ol start="6">
<li><p>在 Visual Studio 或 Xamarin Studio 中，运行设备或模拟器中的客户端项目。验证在应用程序启动后是否引发状态代码为 401（"未授权"）的未处理异常。</p>
   
   	<p>发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 <em>TodoItem</em> 表现在要求身份验证。</p></li>
</ol>

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

## <a name="add-authentication"></a>向应用程序添加身份验证

1. 将以下属性添加到 **TodoActivity** 类：

			private MobileServiceUser user;

2. 将以下方法添加到 **TodoActivity** 类： 

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

    这将会创建一个用于处理身份验证过程的新方法。将使用 Facebook 登录对用户进行身份验证。此时将出现一个对话框，其中显示了经过身份验证的用户的 ID。 

    > [AZURE.NOTE] 如果使用的标识提供程序不是 Facebook，请将传递给上述 **LoginAsync** 的值更改为以下项之一：_MicrosoftAccount__Twitter_、_Google_ 或_WindowsAzureActiveDirectory。

3. 在 **OnCreate** 方法中，在实例化  `MobileServiceClient` 对象的代码后面添加以下代码行。

		// Get the Mobile Service Table instance to use
        toDoTable = client.GetTable <ToDoItem> ();

        await Authenticate(); // add this line

	此调用启动身份验证过程，并以异步方式等待它。


4. 然后，从**"运行"**菜单中单击**"运行"**以启动应用程序，并使用所选的标识提供程序登录。 

   	当您成功登录时，应用应该运行而不出现错误，您应该能够查询移动服务，并对数据进行更新。


<!-- ## <a name="next-steps"> </a>后续步骤

在下一教程[移动服务用户的服务端授权][使用脚本为用户授权]中，你将获取移动服务基于经过身份验证的用户提供的用户 ID 值并使用该值来筛选移动服务返回的数据。 
 -->
 
<!-- Anchors. -->
[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制为提供给经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[后续步骤]:#next-steps


<!-- URLs. -->
["提交应用"页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-xamarin-android-get-started/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-xamarin-android-get-started-users/
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-xamarin-android-get-started-push/
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-authorize-users-in-scripts
[JavaScript 和 HTML]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-users/

[Azure 管理门户]: https://manage.windowsazure.cn/
