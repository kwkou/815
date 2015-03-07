<properties linkid="" urlDisplayName="" pageTitle="" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="Considerations for supporting multiple clients from a single mobile service" authors="krisragh" solutions="" manager="" editor="" />

# 通过单个移动服务支持多个设备平台
 
在移动应用程序开发中使用 Azure 移动服务的主要优势之一在于，能够使用单个后端服务来支持多个客户端平台上的应用程序。移动服务为所有主要设备平台提供了本机客户端库，让你更轻松地使用单个后端服务，通过跨平台开发人员工具开发应用程序。本主题讨论在使用单个移动服务后端时让应用程序运行在多个客户端平台上的以下注意事项：

+ [跨平台推送通知](#push)
+ [跨平台应用程序开发](#xplat-app-dev)
+ [共享 Visual Studio 项目中的代码](#shared-vs)

有关移动服务的更多常规信息，请参阅[移动服务开发人员中心](/zh-cn/documentation/services/mobile-services/)。

## <a id="push"></a>跨平台推送通知

移动服务使用 Azure 通知中心将推送通知发送到你在所有主要设备平台上的客户端应用程序。通知中心提供了一致且统一的基础结构让你创建和管理设备注册，以及发送跨平台的推送通知。通知中心通过使用以下特定于平台的通知服务支持发送推送通知：

+ 适用于 iOS 应用程序的 Apple 推送通知服务 (APNS)
+ 适用于 Android 应用程序的 Google 云消息 (GCM) 服务
+ 适用于 Windows 应用商店、Windows Phone 8.1 应用商店和通用 Windows 应用程序的 Windows 通知服务 (WNS) 
+ 适用于 Windows Phone Silverlight 应用程序的 Microsoft 推送通知服务 (MPNS)

>[WACOM.NOTE]通知中心目前不支持使用 WNS 将推送通知发送到 Windows Phone Silverlight 8.1 应用程序。必须使用 MPNS 将通知发送到 Silverlight 和 Windows Phone 8.0 和 7.0 应用程序。

有关详细信息，请参阅 [Azure 通知中心]。

可使用特定于平台的移动服务客户端库中的注册函数或使用移动服务 REST API 创建客户端注册。通知中心支持两种类型的设备注册：

+ **本机注册**<br/>本机注册专门针对特定于平台的推送通知服务。将通知发送到使用本机注册注册的设备时，你必须在移动服务中调用特定于平台的 API。若要将通知发送到多个平台上的设备，需要多个特定于平台的调用。   
  
+ **模板注册**<br/>通知中心还支持特定于平台的模板注册。通过使用模板注册，你可以使用单个 API 调用将通知发送到任何已注册的平台上运行的应用程序。有关详细信息，请参阅[向用户发送跨平台通知]。

>[WACOM.NOTE]尝试将消息发送到不存在设备注册的本机设备平台时出错。发送模板通知时，不会出现此错误。

链接到特定于客户端的教程的以下各节中的表显示了如何实现从 .NET 和 JavaScript 后端移动服务推送通知。  

### .NET 后端

在 .NET 后端移动服务中，通过调用从 [ApiServices.Push](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobile.service.apiservices.push.aspx) 属性获取的 [PushClient](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobile.service.notifications.pushclient.aspx) 对象的 [SendAsync] 方法发送通知。发送的推送通知（本机或模板）取决于传递给 [SendAsync] 方法的特定 [IPushMessage](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobile.service.notifications.ipushmessage.aspx) 派生的对象，如下表所示： 

|平台 |[APNS](/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-get-started-push)|[GCM](/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-push) |[WNS](/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push) |[MPNS](/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push)|
|-----|-----|----|----|-----|
|本机|[ApplePushMessage](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobile.service.applepushmessage.aspx)   |[GooglePushMessage](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobile.service.googlepushmessage.aspx)     |[WindowsPushMessage](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobile.service.windowspushmessage.aspx) | [MpnsPushMessage](http://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobile.service.mpnspushmessage.aspx) |

下面的代码从 .NET 后端服务将推送通知发送到所有 iOS 和 Windows 应用商店设备注册： 

	// Define a push notification for APNS.
	ApplePushMessage apnsMessage = new ApplePushMessage(item.Text, TimeSpan.FromHours(1));    

	// Define a push notification for WNS.
	WindowsPushMessage wnsMessage = new WindowsPushMessage();
    wnsMessage.XmlPayload = @"<?xml version=""1.0"" encoding=""utf-8""?>" +
                         @"<toast><visual><binding template=""ToastText01"">" +
                         @"<text id=""1"">" + item.Text + @"</text>" +
                         @"</binding></visual></toast>";
    
	// Send push notifications to all registered iOS and Windows Store devices. 
    await Services.Push.SendAsync(apnsMessage);
	await Services.Push.SendAsync(wnsMessage);

有关如何将推送通知发送到其他本机客户端平台的示例，请单击上表标题中的平台链接。 

如果你使用模板客户端注册（而不是本机客户端注册），则只需提供 [TemplatePushMessage] 对象调用 [SendAsync] 一次即可发送同一通知，如下所示： 

	// Create a new template message and add the 'message' parameter.    
	var templatePayload = new TemplatePushMessage();
    templatePayload.Add("message", item.Text);

	// Send a push notification to all template registrations.
    await Services.Push.SendAsync(templatePayload); 
 
### JavaScript 后端

在 JavaScript 后端移动服务中，通过调用从全局 [push 对象]获取的特定于平台的对象的 **send** 方法发送通知，如下表所示： 

|平台 |[APNS](/zh-cn/documentation/articles/mobile-services-javascript-backend-ios-get-started-push)|[GCM](/zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push) |[WNS](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push) |[MPNS](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push)|
|-----|-----|----|----|-----|
|本机|[apns 对象](http://msdn.microsoft.com/library/azure/jj839711.aspx)   |[gcm 对象](http://msdn.microsoft.com/library/azure/dn126137.aspx)     |[wns 对象](http://msdn.microsoft.com/library/azure/jj860484.aspx) | [mpns 对象](http://msdn.microsoft.com/library/azure/jj871025.aspx) |

下面的代码将向所有 Android 和 Windows Phone 注册发送推送通知： 

	// Define a push notification for GCM.
	var gcmPayload = 
    '{"data":{"message" : item.text }}';

	// Define the payload for a Windows Phone toast notification.
	var mpnsPayload = '<?xml version="1.0" encoding="utf-8"?>' +
    '<wp:Notification xmlns:wp="WPNotification"><wp:Toast>' +
    '<wp:Text1>New Item</wp:Text1><wp:Text2>' + item.text + 
    '</wp:Text2></wp:Toast></wp:Notification>';

	// Send push notifications to all registered Android and Windows Phone 8.0 devices. 
	push.mpns.send(null, mpnsPayload, 'toast', 22, {
            success: function(pushResponse) {
                // Push succeeds.
                },              
                error: function (pushResponse) {
                    // Push fails.
                    }
                });
    push.gcm.send(null, gcmPayload, {
            success: function(pushResponse) {
                // Push succeeds.
                },              
                error: function (pushResponse) {
                    // Push fails.
                    }
                });

有关如何将推送通知发送到其他本机客户端平台的示例，请单击上表标题中的平台链接。

如果你使用模板客户端注册（而不是本机客户端注册），则只需提供模板消息负载调用全局 [push 对象]的 **send** 函数一次即可发送同一通知，如下所示： 

	// Create a new template message with the 'message' parameter.    
	var templatePayload = { "message": item.text };

	// Send a push notification to all template registrations.
    push.send(null, templatePayload, {
            success: function(pushResponse) {
                // Push succeeds.
                },              
                error: function (pushResponse) {
                    // Push fails.
                    }
                }); 

## <a id="xplat-app-dev"></a>跨平台应用程序开发
开发所有主要移动设备平台的本机移动设备应用程序要求你（或你的组织）至少具备 Objective-C、Java 和 C# 或 JavaScript 编程语言的专业知识。由于在这些不同平台中开发费用昂贵，一些开发人员为其应用程序选择完全基于 Web 浏览器的体验。但是，此类基于 Web 的体验不能访问大多数本机资源，这些资源提供了用户希望在其移动设备上实现的丰富体验。  

可使用跨平台工具，这些工具在仍共享单一代码库（通常是 JavaScript）的同时，在移动设备上提供了更丰富的本机体验。移动服务通过提供以下开发平台的快速入门教程，可让你轻松创建和管理跨平台应用程序开发平台的后端服务： 

+ [**Appcelerator**](http://go.microsoft.com/fwlink/p/?LinkId=509987)<br/>Appcelerator 允许你使用 JavaScript 开发编译后可在所有移动设备平台上作为本机应用程序运行的单个应用程序。它在 UI 中提供了丰富的用户体验、对所有本机设备资源的访问权限以及本机应用程序性能。有关详细信息，请参阅 [Appcelerator 教程][Appcelerator]。
 
+ [**PhoneGap**](https://go.microsoft.com/fwLink/p/?LinkID=390707)**/**[**Cordova**](http://cordova.apache.org/)<br/>PhoneGap（Apache Cordova 项目的分发产品）是一个免费的开源框架，它允许你使用标准 Web API、HTML 和 JavaScript 开发可在 Android、iOS 和 Windows 设备上运行的单个应用程序。PhoneGap 提供了基于 Web 视图的 UI，但通过允许访问设备上的本机资源增强了用户体验，这些资源包括推送通知、加速计、相机、存储、地理位置和应用程序内浏览器。有关详细信息，请参阅 [PhoneGap 快速入门教程][PhoneGap]。 
	
	现在 Visual Studio 还允许你使用用于 Visual Studio 的多设备混合应用程序扩展（它是预发行软件）构建跨平台的 Cordova 应用程序。有关详细信息，请参阅[使用 HTML 和 JavaScript 的多设备混合应用程序入门](http://msdn.microsoft.com/library/dn771545.aspx)。 

+ [**Sencha Touch**](http://go.microsoft.com/fwlink/p/?LinkId=509988)<br/>Sencha Touch 提供了一组针对触摸屏优化的控件，这些控件使用单个 HTML 和 JavaScript 代码库在各种移动设备上提供类似本机的体验。Sencha Touch 可与 PhoneGap 或 Cordova 库一起使用，为用户提供对本机设备资源的访问权限。有关详细信息，请参阅 [Sencha Touch 快速入门教程][Sencha]。

+ [**Xamarin**](https://go.microsoft.com/fwLink/p/?LinkID=330242)<br/>使用 Xamarin 可以为 iOS 和 Android 设备创建完全本机应用程序，这些应用程序具有完全本机 UI 并可访问所有设备资源。Xamarin 应用程序使用 C#（而不是 Objective-C 和 Java）编码。这使 .NET 开发人员能够将应用程序发布到 iOS 和 Android 并共享 Windows 项目中的代码。Xamarin 通过 C# 代码在 iOS 和 Android 设备上提供完全本机用户体验。这使你能够在 iOS 和 Android 设备上重用 Windows 应用程序中的某些移动服务代码。有关详细信息，请参阅下面的 [Xamarin 开发](#xamarin) 。 

	你可以使用 Xamarin Studio 或 Visual Studio 2013 构建 Xamarin 应用程序。有关详细信息，请参阅 [Visual Studio 中的跨平台开发](http://msdn.microsoft.com/library/dn771552.aspx)。


## <a id="shared-vs"></a>共享和重用 Visual Studio 项目中的代码

移动服务包括 .NET 客户端库，它是支持在所有 Windows 平台上开发的 .NET Framework 可移植类库 (PCL)。有关详细信息，请参阅[如何将 .NET 客户端用于移动服务]。这样，可轻松地在多个 C# 项目中重用相同的移动服务代码，例如用于数据访问或身份验证的代码。

在项目之间共享和重用 C# 代码的一个常规方法是实现模型-视图-视图模型 (MVVM) 模式，并在多个平台之间共享程序集。你可以在 Visual Studio 中的可移植类库项目内实现模型和视图模型类，然后创建针对不同平台自定义的视图。例如，在平台之间共用的模型代码能够以平台未知的方式从移动服务等源中检索数据。MSDN 库提供了<a href="http://msdn.microsoft.com/library/gg597391(v=vs.110)">概述和示例</a>、<a href="http://msdn.microsoft.com/library/gg597392(v=vs.110)">API 差异</a>介绍、有关<a href="http://msdn.microsoft.com/library/hh563947(v=vs.110)">使用可移植类库实现 MVVM 模式</a>的示例、其他<a href="http://msdn.microsoft.com/library/windowsphone/develop/jj714086(v=vs.105).aspx">说明性指导</a>，以及有关在可移植类库项目中<a href="http://msdn.microsoft.com/library/hh871422(v=vs.110)">管理资源</a>的信息。

除了此一般指南外，Visual Studio 还提供了特定工具，用于在多个客户端应用程序项目中重用移动服务代码，这些工具将在以下各节中讨论。有关使用 Visual Studio 2013 构建跨平台应用程序的更多常规信息，请参阅 [Visual Studio 中的跨平台开发](http://msdn.microsoft.com/library/dn771552.aspx)。  

### 通用 Windows 应用程序

Visual Studio 2013 Update 2 添加了对通用 Windows 应用程序项目的支持。通用应用程序是包括 Windows 应用商店 8.1 和 Windows Phone 应用商店 8.1 应用程序项目，以及共享的代码项目的解决方案。在这种项目中，共享的代码将视为是 Windows 应用商店和 Windows Phone 项目的一部分。有关详细信息，请参阅[构建适用于所有 Windows 设备的通用 Windows 应用程序]。可以使用 C#/XAML 和 JavaScript/HTML 编写通用 Windows 应用程序。 

默认情况下，[Azure 管理门户]的"移动服务快速入门"选项卡可生成 TodoList 示例应用程序的通用 Windows 应用程序版本以帮助你入门。你可以选择下载该项目的 C#/XAML 或 JavaScript/HTML 版本。有关详细信息，请参阅[移动服务入门](/zh-cn/documentation/articles/mobile-services-windows-store-get-started/)。 

>[WACOM.NOTE]该门户提供的快速入门应用程序项目的 C# 版本共享 MainPage.xaml.cs 代码隐藏页，但未使用视图模型。有关作为用 C# 编写使用 MVVM 的通用 Windows 应用程序项目的 TodoList 应用程序示例，请参阅[使用 MVVM 的 Azure 移动服务的通用 Windows 应用程序项目]。 

### <a id="xamarin"></a>Xamarin 开发

通过使用 Xamarin 和 Visual Studio 或 Xamarin Studio，你可以利用 Visual Studio 和 C# 开发体验开发适用于 iOS 和 Android 的应用程序。Xamarin 使用 .NET Framework 的跨平台实现，让你使用 C# 代码开发 iOS 和 Android 应用程序。通过使用 Xamarin，你可以利用 Windows 项目中使用移动服务 .NET 客户端库的现有代码来访问移动服务。有关详细信息，请参阅 [Visual Studio 中的跨平台开发](http://msdn.microsoft.com/library/dn771552.aspx)。

若要开始构建使用移动服务的 Xamarin 应用程序，请参阅 Xamarin 快速入门教程 ([iOS](/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started)/[Android](/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started))。


### Windows 应用商店和 Windows Phone Silverlight 应用程序

在 Windows Phone 8.1 中，你可以选择使用以前的基于 Silverlight 的 XAML 开发应用程序，或者使用基于 Windows 运行时的 XAML 实现通用 Windows 应用程序的开发。有关 Windows Phone 8.1 Silverlight 应用程序和 Windows Phone 应用商店 8.1 应用程序的详细信息，请参阅 [Windows Phone 8 开发人员下一步要做什么]。 

移动服务 .NET 客户端库支持 Windows Phone 应用商店 8.1 和 Windows Phone Silverlight 8.1 应用程序。由于无法从同一个项目同时生成 Windows 运行时应用程序和 Windows Phone Silverlight 应用程序，因此应考虑代码重用策略，如上面所述的 PCL 和 MVVM。

>[WACOM.NOTE]若要同时在 Windows 运行时应用程序和 Windows Phone Silverlight 应用程序中使用 Microsoft 帐户进行单一登录客户端身份验证，必须先在 Windows 应用商店仪表板上注册 Windows 运行时应用程序。这是因为，在为 Windows Phone 创建 Live Connect 注册后，将无法为 Windows 应用商店创建这种注册。有关如何执行此操作的详细信息，请阅读主题**"使用 Live Connect 单一登录对 Windows 应用商店应用程序进行身份验证"**（[Windows 应用商店][SSO Windows Store]/[Windows Phone][SSO Windows Phone]）。


<!-- URLs -->
[Azure 管理门户]: https://manage.windowsazure.cn
[Azure 通知中心]: /zh-cn/documentation/articles/notification-hubs-overview/
[SSO Windows 应用商店]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-single-sign-on/
[SSO Windows Phone]: /zh-cn/documentation/articles/mobile-services-windows-phone-single-sign-on/
[教程和资源]: /zh-cn/documentation/services/mobile-services/
[通知中心入门]: /zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started
[向用户发送跨平台通知]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users-xplat-mobile-services/
[推送 Windows dotnet 入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push-vs2012/
[推送 Windows js 入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-js-vs2012/
[推送 Windows Phone 入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-wp8/
[推送 iOS 入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-ios/
[推送 Android 入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-android/
[动态架构]: http://msdn.microsoft.com/library/windowsazure/jj193175.aspx
[如何将 .NET 客户端用于移动服务]: zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library/
[push 对象]: http://msdn.microsoft.com/library/windowsazure/jj554217.aspx
[TemplatePushMessage]:http://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobile.service.templatepushmessage.aspx
[PhoneGap]: /zh-cn/documentation/articles/mobile-services-javascript-backend-phonegap-get-started/
[Sencha]: /zh-cn/documentation/articles/partner-sencha-mobile-services-get-started/
[Appcelerator]: /zh-cn/documentation/articles/partner-appcelerator-mobile-services-javascript-backend-appcelerator-get-started
[SendAsync]: http://msdn.microsoft.com/library/microsoft.windowsazure.mobile.service.notifications.pushclient.sendasync.aspx
[Windows Phone 8 开发人员下一步要做什么]: http://msdn.microsoft.com/library/windows/apps/dn655121(v=vs.105).aspx
[构建适用于所有 Windows 设备的通用 Windows 应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=509905
[使用 MVVM 的 Azure 移动服务的通用 Windows 应用程序项目]: http://code.msdn.microsoft.com/Universal-Windows-app-for-db3564de
