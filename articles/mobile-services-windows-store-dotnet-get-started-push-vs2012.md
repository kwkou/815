<properties linkid="develop-mobile-tutorials-get-started-with-push-dotnet" urlDisplayName="Get Started with Push Notifications" pageTitle="Get started with push notifications - Mobile Services" metaKeywords="push notifications c#" description="Learn how to use push notifications with Azure Mobile Services." metaCanonical="http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet/" disqusComments="0" umbracoNaviHide="1" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services using Visual Studio 2012" authors="" />

# 使用 Visual Studio 2012 的移动服务中的推送通知入门

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet-vs2012" title="Windows Store C#" class="current">Windows 应用商店 C#</a>
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-js-vs2012" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-wp8" title="Windows Phone">Windows Phone</a>
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-ios" title="iOS">iOS</a>
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-android" title="Android">Android</a>
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a>
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-android" title="Xamarin.Android">Xamarin.Android</a>
</div>	

本主题说明如何使用 Azure 移动服务向 Windows 应用商店应用程序发送推送通知。
在本教程中，你将向快速入门项目添加使用 Windows 推送通知服务 (WNS) 的推送通知功能。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

<div class="dev-callout"><b>说明</b>
	<p>本教程将向在 Visual Studio 2012 中创建的 Windows 应用商店应用程序添加推送通知功能。使用 Visual Studio 2013 包含的新功能，可以轻松地在使用移动服务的 Windows 应用商店应用程序中设置推送通知。有关 Visual Studio 2013 版本的信息，请参阅<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet">推送通知入门</a>。</p>
</div>

本教程将指导你完成启用推送通知的以下基本步骤：

1.  [注册用于推送通知的应用程序并配置移动服务][]
2.  [创建 Registrations 表][]
3.  [向应用程序添加推送通知][]
4.  [更新脚本以发送推送通知][]
5.  [插入数据以接收通知][]

本教程需要的内容如下：

-   Microsoft Visual Studio 2012 Express for Windows 8
-   有效的 Windows 应用商店帐户

本教程基于[数据处理入门][]教程。在开始本教程之前，必须先完成[此教程][数据处理入门]。

<a name="register"></a>
## 注册应用程序向 Windows 应用商店注册应用程序

若要从移动服务将推送通知发送到 Windows 应用商店应用程序，必须将你的应用程序提交到 Windows 应用商店。然后必须将移动服务配置为与 WNS 集成。

[WACOM.INCLUDE [mobile-services-register-windows-store-app](../includes/mobile-services-register-windows-store-app.md)]

现在，你的移动服务和应用程序都已配置为使用 WNS。接下来，你将创建一个新表来存储注册信息。

<a name="create-table"></a>
## 创建新表

[WACOM.INCLUDE [mobile-services-create-new-push-table](../includes/mobile-services-create-new-push-table.md)]

<a name="add-push"></a>
## 添加推送通知向应用程序添加推送通知

1.  打开项目文件 MainPage.xaml.cs 并添加以下代码，以创建新的 "Registrations" 类：

        public class Registrations
        {
        public string Id { get; set; }

        [JsonProperty(PropertyName = "handle")]
        public string Handle { get; set; }
        }

    Handle 属性用于存储通道 URI。

2.  打开文件 App.xaml.cs 并添加以下 using 语句：

        using Windows.Networking.PushNotifications;

3.  将以下代码添加到 App.xaml.cs：

        private async void AcquirePushChannel()
        {
        CurrentChannel = 
        await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        IMobileServiceTable<Registrations> registrationsTable = App.MobileService.GetTable<Registrations>();
        var registration = new Registrations { Handle = CurrentChannel.Uri };
        await registrationsTable.InsertAsync(registration);
        }

    此代码将当前通道插入 Registrations 表。

4.  在 App.xaml.cs 中 "OnLaunched" 事件处理程序的顶部，添加对新的 "AcquirePushChannel" 方法的以下调用：

        AcquirePushChannel();

这确保每次启动应用程序时都初始化 "CurrentChannel" 属性。

1.  打开 Package.appxmanifest 文件，并确保“应用程序 UI”选项卡上的“支持 Toast 通知”已设置为“是” 。

    ![][0]

    这可以确保你的应用程序能够引发 toast 通知。

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
        push.wns.sendToastText04(registration.handle, {
        text1:item.text
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

1.  在 Visual Studio 中，按 F5 键运行应用程序。

2.  在应用程序中的“插入 TodoItem”内键入文本，然后单击“保存” 。

    ![][2]

    请注意，完成插入后，应用程序将会接收来自 WNS 的推送通知。

    ![][3]

<a name="next-steps"> </a>
## 后续步骤

本教程演示了移动服务提供的基本推送通知功能。如果你的应用程序需要更高级功能，例如发送跨平台通知、基于订阅的路由或极大量的通知，请考虑为移动服务使用 Azure 通知中心。有关详细信息，请参阅下列通知中心主题之一：

-   [通知中心入门][]
    了解如何在 Windows 应用商店应用程序中利用通知中心。

-   [什么是通知中心？][]
    了解如何创建通知并将通知推送到多个平台上的用户。

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

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet-vs2012 "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/get-started-with-push-js-vs2012 "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/get-started-with-push-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-push-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/get-started-with-push-android "Android"
  [Xamarin.iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-android "Xamarin.Android"
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet
  [注册用于推送通知的应用程序并配置移动服务]: #register
  [创建 Registrations 表]: #create-table
  [向应用程序添加推送通知]: #add-push
  [更新脚本以发送推送通知]: #update-scripts
  [插入数据以接收通知]: #test
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet
  [mobile-services-register-windows-store-app]: ../includes/mobile-services-register-windows-store-app.md
  [mobile-services-create-new-push-table]: ../includes/mobile-services-create-new-push-table.md
  [0]: ./media/mobile-services-windows-store-dotnet-get-started-push-vs2012/mobile-app-enable-toast-win8.png
  [mobile-services-update-registrations-script]: ../includes/mobile-services-update-registrations-script.md
  [1]: ./media/mobile-services-windows-store-dotnet-get-started-push-vs2012/mobile-insert-script-push2.png
  [2]: ./media/mobile-services-windows-store-dotnet-get-started-push-vs2012/mobile-quickstart-push1.png
  [3]: ./media/mobile-services-windows-store-dotnet-get-started-push-vs2012/mobile-quickstart-push2.png
  [通知中心入门]: /zh-cn/manage/services/notification-hubs/getting-started-windows-dotnet/
  [什么是通知中心？]: /zh-cn/develop/net/how-to-guides/service-bus-notification-hubs/
  [向订户发送通知]: /zh-cn/manage/services/notification-hubs/breaking-news-dotnet/
  [向用户发送通知]: /zh-cn/manage/services/notification-hubs/notify-users/
  [向用户发送跨平台通知]: /zh-cn/manage/services/notification-hubs/notify-users-xplat-mobile-services/
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/develop/mobile/how-to-guides/work-with-net-client-library/
