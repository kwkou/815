<properties pageTitle="Get started with push notifications (Windows Store) | Mobile Dev Center" metaKeywords="" description="Learn how to use Azure Mobile Services and Notification Hubs to send push notifications to your Windows Store app." metaCanonical="" services="mobile" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 移动服务中的推送通知入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push" title="Windows Store C#">Windows 应用商店 C\#</a><a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push" title="Windows Phone" class="current">Windows Phone</a><a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-push" title="iOS">iOS</a><a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push" title="Android">Android</a></div>

<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push/" title=".NET backend">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/"  title="JavaScript backend" class="current">JavaScript 后端</a></div>

本主题说明如何使用 Azure 移动服务向 Windows Phone Silverlight 应用程序发送推送通知。在本教程中，你将要使用 Azure 通知中心为快速入门项目启用推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会使用通知中心发送一条推送通知。创建的通知中心可在移动服务中任意使用，可独立于移动服务进行管理，并可供其他应用程序和服务使用。

> [WACOM.NOTE] 本教程演示了移动服务与通知中心的集成功能，该功能当前以预览版提供。默认情况下，未在 JavaScript 后端中启用通过通知中心发送推送通知的功能。创建新的通知中心后，集成过程将不可逆。

本教程将指导你完成启用推送通知的以下基本步骤：

1.  [更新应用程序以注册通知][]
2.  [更新服务器脚本以发送推送通知][]
3.  [插入数据以接收推送通知][]

本教程基于移动服务快速入门。在开始学习本教程之前，必须先完成[移动服务入门][]或[数据处理入门][]，以将项目连接到移动服务。未连接移动服务时，“添加推送通知”向导将为你创建此连接。

> [WACOM.NOTE] 若要向 Windows Phone 应用商店应用程序发送推送通知，请遵照本教程的 [Windows 应用商店应用程序][]版本。

<a id="update-app"></a> 
## 更新应用程序以注册通知

只有在你注册通知通道后，你的应用程序才能接收推送通知。

1.  在 Visual Studio 中，打开文件 App.xaml.cs 并添加以下 `using` 语句：

        using Microsoft.Phone.Notification;

2.  将以下代码添加到 App.xaml.cs：

        public static HttpNotificationChannel CurrentChannel { get; private set; }

        private void AcquirePushChannel()
        {
        CurrentChannel = HttpNotificationChannel.Find("MyPushChannel");

        if (CurrentChannel == null)
            {
        CurrentChannel = new HttpNotificationChannel("MyPushChannel");
        CurrentChannel.Open();
        CurrentChannel.BindToShellToast();
            }

        CurrentChannel.ChannelUriUpdated +=
        new EventHandler<NotificationChannelUriEventArgs>(async (o, args) =>
                {

        // Register for notifications using the new channel
        await MobileService.GetPush()
        .RegisterNativeAsync(CurrentChannel.ChannelUri.ToString());
                });
        }

    此代码从 WNS 检索应用程序的 ChannelURI，然后注册该 ChannelURI，以便将其用于推送通知。

    > [WACOM.NOTE] 在本教程中，移动服务将向设备发送一条 toast 通知。而当你发送磁贴通知时，必须在通道上调用 "BindToShellTile" 方法。

3.  在 App.xaml.cs 中 "Application\_Launching" 事件处理程序的顶部，添加对新的 "AcquirePushChannel" 方法的以下调用：

        AcquirePushChannel();

    这可以确保每次加载页时都会请求注册。在应用程序中，你可能只需要定期执行此注册以确保注册是最新的。

4.  按 "F5" 键以运行应用程序。将显示包含注册密钥的弹出式对话框。

5.  在解决方案资源管理器中，展开“属性” ，打开 WMAppManifest.xml 文件，单击“功能” 选项卡并确保选中 "ID\_"CAP"*PUSH*NOTIFICATION" 功能。

    ![][]

    这可以确保你的应用程序能够引发 toast 通知。

<a id="update-scripts"></a>
## 更新服务器脚本以发送推送通知

最后，你必须更新注册到 TodoItem 表上的插入操作的脚本，以便发送通知。

1.  单击“TodoItem” ，再单击“脚本”并选择“插入”。 

    ![][1]

2.  将 insert 函数替换为以下代码，然后单击“保存”： 

            function insert(item, user, request) {
        // Define a payload for the Windows Phone toast notification.
        var payload = '<?xml version="1.0" encoding="utf-8"?>' +
        '<wp:Notification xmlns:wp="WPNotification"><wp:Toast>' +
        '<wp:Text1>New Item</wp:Text1><wp:Text2>' + item.text + 
        '</wp:Text2></wp:Toast></wp:Notification>';

        request.execute({
        success:function() {
        // If the insert succeeds, send a notification.
        push.mpns.send(null, payload, 'toast', 22, {
        success:function(pushResponse) {
        console.log("Sent push:", pushResponse);
        request.respond();
                        },              
        错误：function (pushResponse) {
        console.log("Error Sending push:", pushResponse);
        request.respond(500, { error:pushResponse });
                            }
                        });
                    }
                });      
        }

    插入成功后，此插入脚本会向所有 Windows Phone 应用程序注册发送推送通知（包括插入项的文本）。

3.  单击“推送”选项卡，选中“启用未经身份验证的推送通知”，然后单击“保存” 。

    ![][2]

    这样，移动服务便可以连接到处于未经身份验证模式的 MPNS 以发送推送通知。

    > [WACOM.NOTE] 本教程使用处于未经身份验证模式的 MPNS。在此模式下，MPNS 将限制可发送到某个设备通道的通知数。若要解除此限制，必须生成一个证书，然后通过单击“上载”并选择该证书来上载该证书 。有关生成证书的详细信息，请参阅[设置已经过身份验证的 Web 服务以便为 Windows Phone 发送推送通知][]。

<a id="test"></a>
## 在应用程序中测试推送通知

1.  在 Visual Studio 中，按 F5 键运行应用程序。

2.  在应用程序中的文本框内输入文本“hello push”，然后单击“保存” 。

    ![][3]

    此时会将一个插入请求发送到移动服务，以存储添加的项。可以看到，应用程序收到了一条包含“hello push”字样的 toast 通知 。

<a name="next-steps"> </a>
## 后续步骤

本教程演示了有关如何使 Windows 应用商店应用程序处理移动服务中的数据的基础知识。接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

-   [通知中心入门][]
    了解如何在 Windows 应用商店应用程序中利用通知中心。

-   [向订户发送通知][]
    了解用户如何注册和接收他们感兴趣的类别的推送通知。

-   [向用户发送通知][]
    了解如何从移动服务向任一设备上的特定用户发送推送通知。

-   [向用户发送跨平台通知][]
    了解如何使用模板从移动服务发送推送通知，且不会在后端中产生平台特定的负载。

建议你了解有关以下移动服务主题的详细信息：

-   [数据处理入门][]
    了解有关使用移动服务存储和查询数据的详细信息。

-   [身份验证入门][]
    了解如何使用 Windows 帐户对应用程序用户进行身份验证。

-   [移动服务服务器脚本参考][]
    了解有关注册和使用服务器脚本的详细信息。

-   [移动服务 .NET 操作方法概念性参考][]
    了解有关如何将移动服务与 .NET 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-ios-get-started-push "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push "Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/ "JavaScript 后端"
  [更新应用程序以注册通知]: #update-app
  [更新服务器脚本以发送推送通知]: #update-scripts
  [插入数据以接收推送通知]: #test
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started
  [数据处理入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-data
  [Windows 应用商店应用程序]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push
  [0]: ./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-app-enable-push-wp8.png
  [1]: ./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-insert-script-push2.png
  [2]: ./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-push-tab.png
  [设置已经过身份验证的 Web 服务以便为 Windows Phone 发送推送通知]: http://msdn.microsoft.com/zh-cn/library/windowsphone/develop/ff941099(v=vs.105).aspx
  [3]: ./media/mobile-services-javascript-backend-windows-phone-get-started-push/mobile-quickstart-push3-wp8.png
  [通知中心入门]: /zh-cn/manage/services/notification-hubs/getting-started-windows-dotnet/
  [向订户发送通知]: /zh-cn/manage/services/notification-hubs/breaking-news-dotnet/
  [向用户发送通知]: /zh-cn/manage/services/notification-hubs/notify-users/
  [向用户发送跨平台通知]: /zh-cn/manage/services/notification-hubs/notify-users-xplat-mobile-services/
  [身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-users
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
