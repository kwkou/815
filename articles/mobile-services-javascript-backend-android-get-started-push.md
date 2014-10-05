<properties linkid="develop-mobile-tutorials-get-started-with-push-js-vs2013" urlDisplayName="Get Started with Push (JS)" pageTitle="Get started with push notifications (Android JavaScript) | Mobile Dev Center" metaKeywords="" description="Learn how to use Windows Azure Mobile Services to send push notifications to your Android JavaScript app." metaCanonical="http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet/" services="" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="ricksal"  solutions="" writer="ricksal" manager="" editor=""  />

<a name="getting-started-with-push"> </a>
# 移动服务中的推送通知入门

<div class="dev-center-tutorial-selector sublanding">
	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push" title="Windows Store C#">Windows 应用商店 C\#</a>
	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push" title="Windows Phone">Windows Phone</a>
	<a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-push" title="iOS">iOS</a>
	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push" title="Android" class="current">Android</a>
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a>
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-android" title="Xamarin.Android">Xamarin.Android</a>
</div>


<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-push/" title=".NET backend" >.NET 后端</a> | 
	<a href="/zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push/" title="JavaScript backend" class="current">JavaScript 后端</a>
</div>

本主题说明如何使用 Azure 移动服务向 Android 应用程序发送推送通知。在本教程中，你将要使用 Google Cloud Messaging (GCM) 向快速入门项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

[WACOM.NOTE] 本教程演示了移动服务与通知中心的集成功能，该功能当前以预览版提供。默认情况下，未在 JavaScript 后端中启用通过通知中心发送推送通知的功能。创建新的通知中心后，集成过程将不可逆。

本教程将指导你完成启用推送通知的以下基本步骤：

1.  [启用 Google Cloud Messaging][]
2.  [配置移动服务][]
3.  [向应用程序添加推送通知][]
4.  [更新脚本以发送推送通知][]
5.  [插入数据以接收通知][]

本教程基于移动服务快速入门。在开始学习本教程之前，必须先完成[移动服务入门][]或[数据处理入门][]，以将项目连接到移动服务。

<a id="register"></a>
## 启用 Google Cloud Messaging

[WACOM.INCLUDE [启用 GCM][]]

接下来，你将使用此 API 密钥值，让移动服务向 GCM 进行身份验证并代表你的应用程序发送推送通知。

<a id="configure"></a>
## 配置移动服务以发送推送请求

1.  登录到 [Windows Azure 管理门户][]，单击“移动服务”，然后单击你的应用程序 。

    ![][]

2.  单击“推送”选项卡，再 单击“启用增强的推送” ，然后单击“是” 以接受配置更改。

    ![][1]

    这样可以更新你的移动服务的配置，以使用通知中心提供的增强的推送功能。对于你的付费移动服务，有些通知中心使用是免费的。有关详细信息，请参阅[移动服务定价详细信息][]。

    <div class="dev-callout"><b>重要说明</b>

    <p>此操作可重置你的推送凭据，并更改脚本中的推送方法行为。这些更改无法恢复。不要使用此方法向生产移动服务添加通知中心。有关如何在生产移动服务中启用增强的推送通知的指导，请参阅<a href="http://go.microsoft.com/fwlink/p/?LinkId=391951">本指南</a>。</p>
	</div>

3.  输入执行前一过程时从 GCM 获取的“API 密钥”值，然后单击“保存” 。

    ![][2]

    <div class="dev-callout"><b>重要说明</b>

    <p>在门户的“推送”选项卡中为增强型推送通知设置 GCM 凭据后，这些凭据将与通知中心共享，以使用你的应用程序配置通知中心。</p>
	</div>

现在，你的移动服务和应用程序都已配置为使用 GCM 和通知中心。

<a id="add-push"></a>
## 向应用程序添加推送通知

### 验证 Android SDK 版本

[WACOM.INCLUDE [验证 SDK][]]

下一步就是安装 Google Play Services。Google Cloud Messaging 对开发和测试提出了一些最低的 API 级别要求，清单中的 "minSdkVersion" 属性必须符合这些要求。

如果你要对某台较旧的设备进行测试，请查阅[设置 Google Play Services SDK][]，以确定此值可设置到的最小值，并相应地进行设置。

### 将 Google Play Services 添加到项目

[WACOM.INCLUDE [添加 Play Services][]]

### 添加代码

[WACOM.INCLUDE [mobile-services-android-getting-started-with-push][]]

<a id="update-scripts"></a>
## 在管理门户中更新已注册的插入脚本

1.  在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表 。

    ![][3]

2.  在“todoitem” 中，单击“脚本” 选项卡，然后选择“插入” 。

    ![][4]

    将显示当 "TodoItem" 表中发生插入时所调用的函数。

3.  将 insert 函数替换为以下代码，然后单击“保存”： 

        function insert(item, user, request) {
        // Define a payload for the Google Cloud Messaging toast notification.
        var payload = 
        '{"data":{"message" :"Hello from Mobile Services!"}}';

        request.execute({
        success:function() {
        // If the insert succeeds, send a notification.
        push.gcm.send(null, payload, {
        success:function(pushResponse) {
        console.log("Sent push:", pushResponse, payload);
        request.respond();
                        },              
        错误：function (pushResponse) {
        console.log("Error Sending push:", pushResponse);
        request.respond(500, { error:pushResponse });
                        }
                    });
                },
        错误：function(err) {
        console.log("request.execute error", err)
        request.respond();
            }
          });
        }

    这将会注册一个新的插入脚本，插入成功后，该脚本将使用 [gcm 对象][]向所有已注册的设备发送推送通知。

<a id="test"></a>
## 在应用程序中测试推送通知

你可以通过以下方式测试应用程序：使用 USB 电缆直接连接 Android 手机，或者在模拟器中使用虚拟设备。

### 设置模拟器以进行测试

当你在模拟器中运行此应用程序时，请确保使用支持 Google API 的 Android 虚拟设备 (AVD)。

1.  重新启动 Eclipse，在包资源管理器中，右键单击项目，单击“属性”，单击“Android”，选中“Google API”，然后单击“确定” 。

    ![][5]

    这样便指定了项目要使用 Google API。

2.  从“窗口”中选择“Android 虚拟设备管理器”，选择你的设备，然后单击“编辑” 。

    ![][6]

3.  在“目标” 中选择“Google API” ，然后单击“确定”。

    ![][7]

    这样便指定了 AVD 要使用 Google API。

### 运行测试

1.  在 Eclipse 中打开“运行” 菜单，然后单击“运行”以启动应用程序 。

2.  在应用程序中键入有意义的文本（例如 *A new Mobile Services task*），然后单击“添加” 按钮。

    ![][8]

3.  从屏幕顶部向下轻扫，打开设备的通知中心以查看通知。

你已成功完成本教程。

<a name="next-steps"> </a>
## 后续步骤

本教程演示了移动服务提供的基本推送通知功能。如果你的应用程序需要更高级功能，例如发送跨平台通知、基于订阅的路由或极大量的通知，请考虑为移动服务使用 Windows Azure 通知中心。有关详细信息，请参阅下列通知中心主题之一：

-   [通知中心入门][]
    了解如何在 Android 应用程序中利用通知中心。

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

-   [如何使用适用于移动服务的 Android 客户端库][]
    了解有关如何将移动服务与 Android 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started-push "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-ios-get-started-push "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push "Android"
  [Xamarin.iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-android "Xamarin.Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-get-started-push/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push/ "JavaScript 后端"
  [启用 Google Cloud Messaging]: #register
  [配置移动服务]: #configure
  [向应用程序添加推送通知]: #add-push
  [更新脚本以发送推送通知]: #update-scripts
  [插入数据以接收通知]: #test
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-android-get-started/
  [数据处理入门]: /zh-cn/documentation/articles/mobile-services-android-get-started-data/
  [启用 GCM]: ../includes/mobile-services-enable-Google-cloud-messaging.md
  [Windows Azure 管理门户]: https://manage.windowsazure.cn/
  []: ./media/mobile-services-android-get-started-push/mobile-services-selection.png
  [1]: ./media/mobile-services-android-get-started-push/mobile-enable-enhanced-push.png
  [移动服务定价详细信息]: http://go.microsoft.com/fwlink/p/?LinkID=311786
  [本指南]: http://go.microsoft.com/fwlink/p/?LinkId=391951
  [2]: ./media/mobile-services-android-get-started-push/mobile-push-tab-android.png
  [验证 SDK]: ../includes/mobile-services-verify-android-sdk-version.md
  [设置 Google Play Services SDK]: http://go.microsoft.com/fwlink/?LinkId=389801
  [添加 Play Services]: ../includes/mobile-services-add-Google-play-services.md
  [mobile-services-android-getting-started-with-push]: ../includes/mobile-services-android-getting-started-with-push.md
  [3]: ./media/mobile-services-android-get-started-push/mobile-portal-data-tables.png
  [4]: ./media/mobile-services-android-get-started-push/mobile-insert-script-push2.png
  [gcm 对象]: http://go.microsoft.com/fwlink/p/?LinkId=282645
  [5]: ./media/mobile-services-android-get-started-push/mobile-services-import-android-properties.png
  [6]: ./media/mobile-services-android-get-started-push/mobile-services-android-virtual-device-manager.png
  [7]: ./media/mobile-services-android-get-started-push/mobile-services-android-virtual-device-manager-edit.png
  [8]: ./media/mobile-services-android-get-started-push/mobile-quickstart-push1-android.png
  [通知中心入门]: /zh-cn/manage/services/notification-hubs/getting-started-windows-dotnet/
  [向订户发送通知]: /zh-cn/manage/services/notification-hubs/breaking-news-dotnet/
  [向用户发送通知]: /zh-cn/manage/services/notification-hubs/notify-users/
  [向用户发送跨平台通知]: /zh-cn/manage/services/notification-hubs/notify-users-xplat-mobile-services/
  [身份验证入门]: /zh-cn/documentation/articles/mobile-services-android-get-started-users
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
  [如何使用适用于移动服务的 Android 客户端库]: /zh-cn/documentation/articles/mobile-services-android-how-to-use-client-library
