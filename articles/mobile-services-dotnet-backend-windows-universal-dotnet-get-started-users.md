<properties pageTitle="身份验证入门（Windows 应用商店） | 移动开发人员中心" metaKeywords="authentication, FAcebook, GOogle, Twitter, Microsoft Account, login" description="了解如何使用移动服务通过各种标识提供程序（包括 Google、Facebook、Twitter 和 Microsoft）对 Windows 应用商店应用程序的用户进行身份验证。" metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Get started with authentication in Mobile Services" authors="Glenn Gailey" solutions="Mobile" manager="dwrede" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-store" ms.devlang="dotnet" ms.topic="article" ms.date="09/23/2014" ms.author="glenga" />

# 向移动服务应用程序添加身份验证 

[WACOM.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]

本主题说明如何通过通用 Windows 应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1. [注册应用程序以进行身份验证并配置移动服务]
2. [将表权限限制为提供给经过身份验证的用户]
3. [向应用程序添加身份验证]
4. [在客户端上存储身份验证令牌]

本教程基于移动服务快速入门。因此，你还必须先完成[移动服务入门]教程。 

>[AZURE.NOTE]本教程演示了如何对 Windows 应用商店和 Windows Phone 应用商店 8.1 应用程序中的用户进行身份验证。对于 Windows Phone 8.0 或 Windows Phone Silverlight 8.1 应用程序，请参阅此版本的[移动服务中的身份验证入门](/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-users)。

##<a name="register"></a>注册应用程序以进行身份验证并配置移动服务

[WACOM.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)] 

[WACOM.INCLUDE [mobile-services-dotnet-backend-aad-server-extension](../includes/mobile-services-dotnet-backend-aad-server-extension.md)] 

##<a name="permissions"></a>将权限限制为提供给经过身份验证的用户

[WACOM.INCLUDE [mobile-services-restrict-permissions-dotnet-backend](../includes/mobile-services-restrict-permissions-dotnet-backend.md)] 

<ol start="6">
<li><p>在 Visual Studio 中，右键单击 TodoList 应用程序的 Windows 应用商店项目，然后单击<strong>"设为启动项目"</strong>。</p></li>
<li><p>在共享的项目中，打开 App.xaml.cs 项目文件，找到 <a href="http://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobileservices.mobileserviceclient.aspx">MobileServiceClient</a> 的定义，并确保它已配置为连接到 Azure 中运行的移动服务。</p>
<p>请注意，使用 Visual Studio 工具将应用程序连接到移动服务时，该工具生成两组 <strong>MobileServiceClient</strong> 定义，每个客户端平台一组。这是简化生成的代码的好时机，你可以通过将 <code>#if...#endif</code> 包装的 <strong>MobileServiceClient</strong> 定义统一为单个解包的定义供这两个版本的应用程序使用来简化生成的代码。从 Azure 管理门户下载快速入门应用程序时无需执行此操作。</p>
</li> 
<li><p>按 F5 键运行这个 Windows 应用商店应用程序，验证启动该应用程序后，是否会引发状态代码为 401（"未授权"）的未处理异常。</p>
   
   	<p>发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 <em>TodoItem</em> 表现在要求身份验证。</p></li>
</ol>

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

##<a name="add-authentication"></a>向应用程序添加身份验证

[WACOM.INCLUDE [mobile-services-windows-universal-dotnet-authenticate-app](../includes/mobile-services-windows-universal-dotnet-authenticate-app.md)] 

[AZURE.NOTE]如果已将 Windows 应用商店应用程序包信息注册到移动服务，则应该为 <em>useSingleSignOn</em> 参数提供 <strong>true</strong> 值以调用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=311594" target="_blank">LoginAsync</a> 方法。如果你不执行此操作，则每次调用 login 方法时，系统将继续向你的用户显示登录提示。

##<a name="tokens"></a>在客户端上存储授权令牌

[WACOM.INCLUDE [mobile-services-windows-store-dotnet-authenticate-app-with-token](../includes/mobile-services-windows-store-dotnet-authenticate-app-with-token.md)] 


## <a name="next-steps"> </a>后续步骤

在下一教程[移动服务用户的服务端授权][使用脚本为用户授权]中，你将获取移动服务基于经过身份验证的用户提供的用户 ID 值并使用该值来筛选移动服务返回的数据。请在[移动服务 .NET 操作方法概念性参考]中了解有关如何将移动服务与 .NET 一起使用的详细信息。

<!-- Anchors. -->
[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制为提供给经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[在客户端上存储身份验证令牌]: #tokens
[后续步骤]:#next-steps


<!-- URLs. -->
["提交应用"页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[使用 Live Connect 实现对 Windows 应用商店应用程序的单一登录]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-single-sign-on
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-users/
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push/
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-authorize-users-in-scripts
[JavaScript 和 HTML]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-users/

[Azure 管理门户]: https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library/
[注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证]: /zh-cn/documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication/
