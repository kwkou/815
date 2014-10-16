<properties pageTitle="Get started with push notifications (Windows Phone) | Mobile Dev Center" metaKeywords="" description="Learn how to use Azure Mobile Services to send push notifications to your Windows Phone app." metaCanonical="" services="" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 移动服务中的推送通知入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-push" title="Windows Store C#" >Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-push" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-push" title="Windows Phone" class="current">Windows Phone</a><a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-push" title="iOS">iOS</a><a href="/zh-cn/documentation/articles/mobile-services-android-get-started-push" title="Android">Android</a><a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started-push" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started-push" title="Xamarin.Android">Xamarin.Android</a></div>

<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push/" title=".NET backend">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-push/"  title="JavaScript backend" class="current">JavaScript 后端</a>
</div>

本主题说明如何使用 Azure 移动服务向 Windows Phone 8 应用程序发送推送通知。
在本教程中，你将向快速入门项目添加使用 Microsoft 推送通知服务 (MPNS) 的推送通知功能。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

> [WACOM.NOTE] 移动服务现在将与 Azure 通知中心集成，以支持附加的推送通知功能，如模板、多个平台和规模。此集成的功能目前处于预览状态。有关详细信息，请参阅此版本的[推送通知入门][]。

本教程将指导你完成启用推送通知的以下基本步骤：

1.  [创建 Registrations 表][]
2.  [向应用程序添加推送通知][]
3.  [更新脚本以发送推送通知][]
4.  [插入数据以接收通知][]

本教程需要安装 [Visual Studio 2012 Express for Windows Phone][] 或更高版本。

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门][]。

> [WACOM.NOTE] 如果你每天向每个用户发送超过 500 条的消息，则必须改用通知中心。有关详细信息，请参阅[通知中心入门][]。

<a name="create-table"></a>
## 创建新表

[WACOM.INCLUDE [mobile-services-create-new-push-table](../includes/mobile-services-create-new-push-table.md)]

<a name="add-push"></a>
## 添加推送通知向应用程序添加推送通知

1.  在 Visual Studio 中，打开项目文件 MainPage.xaml.cs 并添加以下代码，以创建新的 "Registrations" 类：

        public class Registrations
        {
        public string Id { get; set; }

        [JsonProperty(PropertyName = "handle")]
        public string Handle { get; set; }
        }

    Handle 属性用于存储通道 URI。

2.  打开文件 App.xaml.cs 并添加以下 using 语句：

        using Microsoft.Phone.Notification;

3.  将以下代码添加到 App.xaml.cs：

        public static HttpNotificationChannel CurrentChannel { get; private set; }

        private void AcquirePushChannel()
        {
        CurrentChannel = HttpNotificationChannel.Find("MyPushChannel");

        if (CurrentChannel == null)
            {
        CurrentChannel = new HttpNotificationChannel("MyPushChannel");
        CurrentChannel.Open();
        CurrentChannel.BindToShellTile();
            }

        IMobileServiceTable<Registrations> registrationsTable = App.MobileService.GetTable<Registrations>();
        var registration = new Registrations { Handle = CurrentChannel.Uri };
        await registrationsTable.InsertAsync(registration);
        }

    此代码获取并存储推送通知订阅的通道，并将其绑定到应用程序的默认磁贴上。

	<div class="dev-callout"><b>说明</b>

    <p>在本教程中，移动服务向设备发送翻转磁贴通知。当你发送 toast 通知时，必须改为对通道调用 <b>BindToShellToast</b> 方法。若要同时支持 toast 通知和磁贴通知，请同时调用 <b>BindToShellTile</b> 和 <b>BindToShellToast</b></p>
	</div>

4.  在 App.xaml.cs 中 "Application\_Launching" 事件处理程序的顶部，添加对新的 "AcquirePushChannel" 方法的以下调用：

        AcquirePushChannel();

    这确保每次启动应用程序时都初始化 "CurrentChannel" 属性。

5.  在解决方案资源管理器中，展开“属性” ，打开 WMAppManifest.xml 文件，单击“功能” 选项卡并确保选中 "ID\_"CAP"*PUSH*NOTIFICATION" 功能。

    ![][0]

    这样可确保你的应用程序可收到推送通知。

<a name="update-scripts"></a>
## 更新插入脚本在管理门户中更新注册的插入脚本

[WACOM.INCLUDE [mobile-services-update-registrations-script](../includes/mobile-services-update-registrations-script.md)]

1.  单击“TodoItem” ，再单击“脚本”并选择“插入”。 

    ![][1]

2.  将 insert 函数替换为以下代码，然后单击“保存”： 

        function insert(item, user, request) {
        request.execute({
        success:function() {
        request.respond();
        sendNotifications();
                }
            });

        function sendNotifications() {
        var registrationsTable = tables.getTable('Registrations');
        registrationsTable.read({
        success:function(registrations) {
        registrations.forEach(function(registration) {
        push.mpns.sendFlipTile(registration.uri, {
        title:item.text
                            }, {
        success:function(pushResponse) {
        console.log("Sent push:", pushResponse);
                                }
                            });
                        });
                    }
                });
            }
        }

    此插入脚本会向 "Registrations" 表中存储的所有通道发送推送通知（包括已插入项的文本）。

<a name="test"></a>
## 测试应用程序在应用程序中测试推送通知

1.  在 Visual Studio 中，选择“生成” 菜单上的“部署解决方案” 。

2.  在模拟器中，向左轻扫以显示已安装应用程序的列表，并找到新的 "TodoList" 应用程序。

3.  点击并按住该应用程序图标，然后从上下文菜单中选择“固定到‘开始’屏幕” 。

    ![][2]

    这会将名为 "TodoList" 的磁贴固定到到“开始”菜单。

4.  点击名为 "TodoList" 的磁贴以启动应用程序。

    ![][3]

5.  在应用程序中的文本框内输入文本“hello push”，然后单击“保存” 。

    ![][4]

    此时会将一个插入请求发送到移动服务，以存储添加的项。

6.  按“开始”按钮 以返回到“开始”菜单。

    ![][5]

    请注意，应用程序已收到该推送通知，并且磁贴的标题现在是 "hello push"。

<a name="next-steps"> </a>
## 后续步骤

本教程演示了移动服务提供的基本推送通知功能。如果你的应用程序需要更高级功能，例如发送跨平台通知、基于订阅的路由或极大量的通知，请考虑为移动服务使用 Azure 通知中心。有关详细信息，请参阅下列通知中心主题之一：

-   [通知中心入门][6]
    了解如何在 Windows 应用商店应用程序中利用通知中心。

-   [什么是通知中心？][]
    了解如何创建通知并将通知推送到多个平台上的用户。

-   [向订户发送通知][]
    了解用户如何注册和接收他们感兴趣的类别的推送通知。

建议你了解有关以下移动服务主题的详细信息：

-   [数据处理入门][]
    了解有关使用移动服务存储和查询数据的详细信息。

-   [身份验证入门][]
    了解如何使用 Windows 帐户对应用程序用户进行身份验证。

-   [移动服务服务器脚本参考][]
    了解有关注册和使用服务器脚本的详细信息。

-   [移动服务 .NET 操作方法概念性参考][]
    了解有关如何将移动服务与 .NET 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-push "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-push "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-push "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-ios-get-started-push "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-android-get-started-push "Android"
  [Xamarin.iOS]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started-push "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started-push "Xamarin.Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-push/ "JavaScript 后端"
  [推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/
  [创建 Registrations 表]: #create-table
  [向应用程序添加推送通知]: #add-push
  [更新脚本以发送推送通知]: #update-scripts
  [插入数据以接收通知]: #test
  [Visual Studio 2012 Express for Windows Phone]: https://go.microsoft.com/fwLink/p/?LinkID=268374
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started-wp8
  [通知中心入门]: /zh-cn/manage/services/notification-hubs/getting-started-windows-dotnet/
  [mobile-services-create-new-push-table]: ../includes/mobile-services-create-new-push-table.md
  [0]: ./media/mobile-services-windows-phone-get-started-push/mobile-app-enable-push-wp8.png
  [mobile-services-update-registrations-script]: ../includes/mobile-services-update-registrations-script.md
  [1]: ./media/mobile-services-windows-phone-get-started-push/mobile-insert-script-push2.png
  [2]: ./media/mobile-services-windows-phone-get-started-push/mobile-quickstart-push1-wp8.png
  [3]: ./media/mobile-services-windows-phone-get-started-push/mobile-quickstart-push2-wp8.png
  [4]: ./media/mobile-services-windows-phone-get-started-push/mobile-quickstart-push3-wp8.png
  [5]: ./media/mobile-services-windows-phone-get-started-push/mobile-quickstart-push4-wp8.png
  [6]: /zh-cn/manage/services/notification-hubs/get-started-notification-hubs-wp8/
  [什么是通知中心？]: /zh-cn/develop/net/how-to-guides/service-bus-notification-hubs/
  [向订户发送通知]: /zh-cn/manage/services/notification-hubs/breaking-news-wp8/
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-wp8
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-wp8
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/develop/mobile/how-to-guides/work-with-net-client-library/
