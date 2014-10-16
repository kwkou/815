<properties linkid="" urlDisplayName="" pageTitle="Get started with Notification Hubs for Xamarin.Android apps" metaKeywords="" description="Learn how to use Azure Notification Hubs to send push notifications to a Xamarin Android application." metaCanonical="" authors="elioda" solutions="" manager="" editor="" services="mobile-services,notification-hubs" documentationCenter="" title="Get started with Notification Hubs" />

# 通知中心入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/" title="Windows 应用商店 C#">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/notification-hubs-windows-phone-get-started/" title="Windows Phone">Windows Phone</a><a href="/zh-cn/documentation/articles/notification-hubs-ios-get-started/" title="iOS">iOS</a><a href="/zh-cn/documentation/articles/notification-hubs-android-get-started/" title="Android">Android</a><a href="/zh-cn/documentation/articles/notification-hubs-kindle-get-started/" title="Kindle">Kindle</a><a href="/zh-cn/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zh-cn/documentation/articles/partner-xamarin-notification-hubs-android-get-started/" title="Xamarin.Android" class="current">Xamarin.Android</a></div>

本主题说明如何使用 Azure 通知中心向 Xamarin.Android 应用程序发送推送通知。
在本教程中，你将要创建一个使用 Google Cloud Messaging (GCM) 接收推送通知的空白 Xamarin.Android 应用程序。完成后，你将能使用通知中心将推送通知广播到运行你的应用程序的所有设备。[NotificationHubs 应用程序][NotificationHubs 应用程序]示例中提供了完成的代码。

本教程将指导你完成启用推送通知的以下基本步骤：

1.  [启用 Google Cloud Messaging][启用 Google Cloud Messaging]
2.  [配置通知中心][配置通知中心]
3.  [将你的应用程序连接到通知中心][将你的应用程序连接到通知中心]
4.  [使用模拟器运行你的应用程序][使用模拟器运行你的应用程序]
5.  [从后端发送通知][从后端发送通知]

本教程演示使用通知中心的简单广播方案。本教程需要的内容如下：

-   [Xamarin.Android][1]
-   有效的 Google 帐户
-   [Azure 移动服务组件][Azure 移动服务组件]
-   [Google Cloud Messaging 组件][Google Cloud Messaging 组件]

只有在完成本教程后，才能完成有关 Xamarin.Android 应用程序通知中心的其他所有教程。

<div class="dev-callout"><strong>说明</strong> <p>若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/pricing/free-trial/" target="_blank">Azure 免费试用</a>。</p></div>

## <a name="register"></a><span class="short-header">启用 Google Cloud Messaging</span>启用 Google Cloud Messaging

<div class="dev-callout"><b>说明</b>
<p>若要完成本主题中的过程，你必须拥有一个包含已验证电子邮件地址的 Google 帐户。若要新建一个 Google 帐户，请转至 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268302" target="_blank">accounts.google.com</a>。</p>
</div>

1.  导航到 [Google API][Google API] 网站，使用你的 Google 帐户凭据登录，然后单击“Create project...”（创建项目...）。

    ![][]

    <div class="dev-callout"><b>说明</b>
<p>如果你已拥有现成项目，则在登录后你将定向到&ldquo;仪表板&rdquo;页。若要从仪表板新建一个项目，请展开&ldquo;API 项目&rdquo;，单击&ldquo;其他项目&rdquo;下面的&ldquo;创建...&rdquo;，然后输入项目名称并单击&ldquo;创建项目&rdquo;。</p>
</div>

2.  单击左侧列中的“概述”，记下“仪表板”部分中的项目编号。

    在教程的稍后部分中，你要将此值设置为客户端中的 PROJECT\_ID 变量。

3.  在 [Google API][Google API] 页面上，单击“服务”，然后单击开关以启用 **Google Cloud Messaging for Android** 并接受服务条款。

4.  单击“API Access”（API 访问），然后单击“Create new Server key...”（新建服务器密钥...）

    ![][2]

5.  在“为 API 项目配置服务器密钥”中，单击“创建”。

    ![][3]

6.  记下“API 密钥”值。

    ![][4]

接下来，你将使用此 API 密钥值让通知中心向 GCM 进行身份验证并代表你的应用程序发送推送通知。

## <a name="configure-hub"></a><span class="short-header">配置通知中心</span>配置通知中心

1.  登录到 [Azure 管理门户][Azure 管理门户]，然后单击屏幕底部的“+新建”。

2.  依次单击**“应用程序服务”**、**“Service Bus”**、**“通知中心”**和**“快速创建”**。

    ![][5]

3.  键入通知中心的名称，选择所需的区域，然后单击“创建新的通知中心”。

    ![][6]

4.  单击刚刚创建的命名空间（通常为***通知中心名称*-ns**），然后单击顶部的**“配置”**。

    ![][7]

5.  单击顶部的**“通知中心”**选项卡，然后单击你刚刚创建的通知中心。

    ![][8]

6.  单击顶部的**“配置”**选项卡，输入在上一部分中获得的“API 密钥”值，然后单击**“保存”**。

    ![][9]

7.  选择顶部的**“仪表板”**选项卡，然后单击**“连接信息”**。记下两个连接字符串。

    ![][10]

你的通知中心现在已配置为使用 GCM，并且你有连接字符串用于注册你的应用程序和发送推送通知。

## <a name="connecting-app"></a><span class="short-header">连接应用程序</span>将应用程序连接到通知中心

### 创建新项目

1.  在 Xamarin Studio（或 Visual Studio）中，创建新的 Android 项目（依次单击“文件”、“新建”、“解决方案”、“Android 应用程序”。）

    ![][11]

    ![][12]

2.  通过在“解决方案”视图中右键单击你的新项目，然后选择“选项”，打开项目属性。选择“生成”部分中的“Android 应用程序”项。

    ![][13]

3.  将“最低 Android 版本”设置为“API 级别 8”。

4.  将“目标 Android 版本”设置为要针对的 API 版本（必须是 API 级别 8 或更高版本）。

5.  确保你的“包名称”的第一个字母为小写形式。

    <div class="dev-callout"><b>说明</b>
<p>包名称的第一个字母必须为小写形式。否则，在为以下的推送通知注册 **BroadcastReceiver** 和 **IntentFilter** 时，将收到应用程序清单错误。</p>
</div>

### 将 Google Cloud Messaging Client 添加到项目

Xamarin 组件应用商店中提供的 Google Cloud Messaging Client 可以简化在 Xamarin.Android 中支持推送通知的过程。

1.  在 Xamarin.Android 应用程序中右键单击“Components”文件夹，然后选择“Get More Components...”（获取更多组件...）

2.  搜索 **Google Cloud Messaging Client** 组件。

3.  将该组件添加到 Xamarin.Android 应用程序。系统会自动添加必要的程序集引用。

### 向你的项目添加 Xamarin.NotificationHub

使用此程序集可以方便地注册到 Azure 通知中心。可以参考以下说明下载该程序集，也可以在[示例下载][NotificationHubs 应用程序]中找到其下载链接。

1.  访问 [Xamarin.NotificationHub Github 页][Xamarin.NotificationHub Github 页]，下载并生成源文件夹。

2.  在 Xamarin.Android 项目文件夹中创建 \*\*\_external\*\* 文件夹，然后将编译的 **ByteSmith.WindowsAzure.Messaging.Android.dll** 复制到该文件夹。

3.  在 Xamarin Studio（或者 Visual Studio）中打开你的 Xamarin.Android 项目。

4.  右键单击项目 **References** 文件夹，然后选择“编辑引用…”

5.  转到“.Net 程序集”选项卡，浏览到你的项目的 \*\*\_external\*\* 文件夹，选择我们前面生成的 **ByteSmith.WindowsAzure.Messaging.Android.dll**，然后单击“添加”。单击“确定”以关闭该对话框。

### 在你的项目中设置通知中心

1.  创建 **Constants.cs** 类并定义以下常量值（使用它们替换占位符）：

        public const string SenderID = "<GoogleProjectNumber>"; // Google API Project Number

        // Azure app specific connection string and hub path
        public const string ConnectionString = "<Azure connection string>";
        public const string NotificationHubPath = "<hub path>";

2.  将以下 using 语句添加到 **MainActivity.cs**：

        using ByteSmith.WindowsAzure.Messaging;
        using Gcm.Client;

3.  在 **MainActivity** 类中创建以下方法：

        private void RegisterWithGCM()
        {
            // Check to ensure everything's setup right
            GcmClient.CheckDevice(this);
            GcmClient.CheckManifest(this);

            // Register for push notifications
            System.Diagnostics.Debug.WriteLine("Registering...");
            GcmClient.Register(this, Constants.SenderID);
        }

4.  创建新类 **MyBroadcastReceiver**。

    <div class="dev-callout"><b>说明</b>
<p>下面，我们将演练如何从头开始创建 **BroadcastReceiver**。但是，可以使用一种快捷方法来替代手动创建 **MyBroadcastReceiver.cs**：引用 GitHub 上的示例 Xamarin.Android 项目中提供的 **GcmService.cs** 文件。也可以从复制 **GcmService.cs** 并更改类名称着手。</p>
</div>

5.  将以下 using 语句添加到 **MyBroadcastReceiver.cs**（引用前面添加的组件和程序集）：

        using ByteSmith.WindowsAzure.Messaging;
        using Gcm.Client;

6.  在 **using** 语句和 **namespace** 声明之间添加以下权限请求：

        [assembly: Permission(Name = "@PACKAGE_NAME@.permission.C2D_MESSAGE")]
        [assembly: UsesPermission(Name = "@PACKAGE_NAME@.permission.C2D_MESSAGE")]
        [assembly: UsesPermission(Name = "com.google.android.c2dm.permission.RECEIVE")]

        //GET_ACCOUNTS is only needed for android versions 4.0.3 and below
        [assembly: UsesPermission(Name = "android.permission.GET_ACCOUNTS")]
        [assembly: UsesPermission(Name = "android.permission.INTERNET")]
        [assembly: UsesPermission(Name = "android.permission.WAKE_LOCK")]

7.  在 **MyBroadcastReceiver.cs** 中，更改 **MyBroadcastReceiver** 类以匹配以下内容：

        [BroadcastReceiver(Permission=Gcm.Client.Constants.PERMISSION_GCM_INTENTS)]
        [IntentFilter(new string[] { Gcm.Client.Constants.INTENT_FROM_GCM_MESSAGE }, Categories = new string[] { "@PACKAGE_NAME@" })]
        [IntentFilter(new string[] { Gcm.Client.Constants.INTENT_FROM_GCM_REGISTRATION_CALLBACK }, Categories = new string[] { "@PACKAGE_NAME@" })]
        [IntentFilter(new string[] { Gcm.Client.Constants.INTENT_FROM_GCM_LIBRARY_RETRY }, Categories = new string[] { "@PACKAGE_NAME@" })]
        public class MyBroadcastReceiver : GcmBroadcastReceiverBase<GcmService>
        {
            public static string[] SENDER_IDS = new string[] { Constants.SenderID };

            public const string TAG = "MyBroadcastReceiver-GCM";
        }

8.  在 **MyBroadcastReceiver.cs** 中添加名为 **PushHandlerService** 的一个类，该类从 **PushHandlerServiceBase** 派生。确保对该类使用 **Service** 指令：

        [Service] //Must use the service tag
        public class GcmService : GcmServiceBase
        {
            public static string RegistrationID { get; private set; }
            private NotificationHub Hub { get; set; }

            public GcmService() : base(Constants.SenderID) 
            {
                Log.Info(MyBroadcastReceiver.TAG, "GcmService() constructor"); 
            }
        }

9.  **GcmServiceBase** 实现方法 **OnRegistered()**、**OnUnRegistered()**、**OnMessage()**、**OnRecoverableError()** 和 **OnError()**。我们的实现类 **GcmService** 必须重写这些方法，在响应与通知中心的交互将触发这些方法。

10. 在 **PushHandlerService** 中使用以下代码重写 **OnRegistered()** 方法：

        protected override async void OnRegistered(Context context, string registrationId)
        {
            Log.Verbose(MyBroadcastReceiver.TAG, "GCM Registered: " + registrationId);
            RegistrationID = registrationId;

            createNotification("GcmService-GCM Registered...", "The device has been Registered, Tap to View!");

            Hub = new NotificationHub(Constants.NotificationHubPath, Constants.ConnectionString);
            try
            {
                await Hub.UnregisterAllAsync(registrationId);
            }
            catch (Exception ex)
            {
                Debug.WriteLine(ex.Message);
                Debugger.Break();
            }

            var tags = new List<string>() { "falcons" }; // create tags if you want

            try
            {
                var hubRegistration = await Hub.RegisterNativeAsync(registrationId, tags);
            }
            catch (Exception ex)
            {
                Debug.WriteLine(ex.Message); 
                Debugger.Break();
            }
        }

    <div class="dev-callout"><b>说明</b>
<p>在上述 **OnRegistered()** 代码中，你应注意可指定标签来注册特定消息传送通道。</p>
</div>

11. 在 **PushHandlerService** 中使用以下代码重写 **OnMessage** 方法：

        protected override void OnMessage(Context context, Intent intent)
        {
            Log.Info(MyBroadcastReceiver.TAG, "GCM Message Received!");

            var msg = new StringBuilder();

            if (intent != null && intent.Extras != null)
            {
                foreach (var key in intent.Extras.KeySet())
                    msg.AppendLine(key + "=" + intent.Extras.Get(key).ToString());
            }

            string messageText = intent.Extras.GetString("msg");
            if (!string.IsNullOrEmpty(messageText))
            {
                createNotification("New hub message!", messageText);
                return;
            }

            createNotification("Unknown message details", msg.ToString());
        }

12. 将以下 **createNotification** 方法添加到 **PushHandlerService** 以通知上面使用的用户：

        void createNotification(string title, string desc)
        {
            //Create notification
            var notificationManager = GetSystemService(Context.NotificationService) as NotificationManager;

            //Create an intent to show ui
            var uiIntent = new Intent(this, typeof(MainActivity));

            //Create the notification
            var notification = new Notification(Android.Resource.Drawable.SymActionEmail, title);

            //Auto cancel will remove the notification once the user touches it
            notification.Flags = NotificationFlags.AutoCancel;

            //Set the notification info
            //we use the pending intent, passing our ui intent over which will get called
            //when the notification is tapped.
            notification.SetLatestEventInfo(this, title, desc, PendingIntent.GetActivity(this, 0, uiIntent, 0));

            //Show the notification
            notificationManager.Notify(1, notification);
        }

13. 重写抽象成员 **OnUnRegistered()**、**OnRecoverableError()** 和 **OnError()**，以便能够编译你的代码。

## <a name="run-app"></a><span class="short-header">运行应用程序</span>在模拟器中运行应用程序

当你在模拟器中运行此应用程序时，请确保使用支持 Google API 的 Android 虚拟设备 (AVD)。

1.  从“工具”中，单击“打开 Android 模拟器管理器”，选择你的设备，然后单击“编辑”。

    ![][14]

2.  在“目标”中选择“Google API”，然后单击“确定”。

    ![][15]

3.  在顶部工具栏中，单击“运行”，然后选择你的应用程序。这将启动模拟器并运行该应用程序。

4.  应用程序从 GCM 检索 *registrationId* 并注册到通知中心。

    <div class="dev-callout"><b>说明</b>
<p>为了接收推送通知，你必须在 Android 虚拟设备上设置 Google 帐户（方法如下：在模拟器中，导航到&ldquo;设置&rdquo;，然后单击&ldquo;添加帐户&rdquo;）。此外，请确保模拟器已连接到 Internet。</p>
</div>

## <a name="send"></a><span class="short-header">发送通知</span>从后端发送通知

你可以使用通知中心通过 [REST 接口][REST 接口]从任意后端发送通知。在本教程中，我们将使用 .NET 控制台应用程序和移动服务来发送通知，通过节点脚本来执行这些操作。

使用 .NET 应用程序发送通知：

1.  创建新的 Visual C# 控制台应用程序：

    ![][16]

2.  通过使用 [WindowsAzure.ServiceBus NuGet 包][WindowsAzure.ServiceBus NuGet 包]添加对 Azure Service Bus SDK 的引用。在 Visual Studio 主菜单中，依次单击“工具”、“库程序包管理器”和“程序包管理器控制台”。然后，在控制台窗口中键入：

        Install-Package WindowsAzure.ServiceBus

    然后按 Enter。

3.  打开文件 Program.cs 并添加以下 using 语句：

        using Microsoft.ServiceBus.Notifications;

4.  在 `Program` 类中添加以下方法：

        private static async void SendNotificationAsync()
        {
            NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");
            await hub.SendGcmNativeNotificationAsync("{ \"data\" : {\"msg\":\"Hello from Azure!\"}}");
        }

5.  然后在 Main 方法中添加下列行：

         SendNotificationAsync();
         Console.ReadLine();

6.  按 F5 键以运行应用程序。你应接收 toast 通知。

    ![][17]

若要使用移动服务发送通知，请按[移动服务入门][移动服务入门]中的说明操作，然后：

1.  登录到 [Azure 管理门户][Azure 管理门户]并选择你的移动服务。

2.  选择顶部的“计划程序”选项卡。

    ![][18]

3.  创建新的计划作业，插入名称，然后选择“按需”。

    ![][19]

4.  创建作业时，单击该作业名称。然后单击顶部栏上的“脚本”选项卡。

5.  在你的计划程序函数中插入以下脚本。确保将占位符替换为你的通知中心名称和你以前获取的 *DefaultFullSharedAccessSignature* 的连接字符串。单击**“保存”**。

        var azure = require('azure');
        var notificationHubService = azure.createNotificationHubService('<hub name>', '<connection string>');
        notificationHubService.gcm.send(null,'{"data":{"msg" : "Hello from Mobile Services!"}}',
          function (error)
          {
            if (!error) {
               console.warn("Notification successful");
            }
            else
            {
              console.warn("Notification failed" + error);
            }
          }
        );

6.  单击底部栏上的“运行一次”。你应接收 toast 通知。

## <a name="next-steps"> </a>后续步骤

在这个简单的示例中，你将通知广播到所有 Android 设备。为了针对特定用户广播，请参考教程[使用通知中心将通知推送到用户][使用通知中心将通知推送到用户]，如果你要按兴趣细分用户组，请参考[使用通知中心发送突发新闻][使用通知中心发送突发新闻]。在[通知中心指南][通知中心指南]和[针对 Android 的通知中心操作指南][针对 Android 的通知中心操作指南]中了解有关如何使用通知中心的详细信息。

<!-- Anchors. -->  

<!-- URLs. -->

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/ "Windows 应用商店 C#"
  [Windows Phone]: /zh-cn/documentation/articles/notification-hubs-windows-phone-get-started/ "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/notification-hubs-ios-get-started/ "iOS"
  [Android]: /zh-cn/documentation/articles/notification-hubs-android-get-started/ "Android"
  [Kindle]: /zh-cn/documentation/articles/notification-hubs-kindle-get-started/ "Kindle"
  [Xamarin.iOS]: /zh-cn/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/ "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/documentation/articles/partner-xamarin-notification-hubs-android-get-started/ "Xamarin.Android"
  [NotificationHubs 应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=331329
  [启用 Google Cloud Messaging]: #register
  [配置通知中心]: #configure-hub
  [将你的应用程序连接到通知中心]: #connecting-app
  [使用模拟器运行你的应用程序]: #run-app
  [从后端发送通知]: #send
  [1]: http://xamarin.com/download/
  [Azure 移动服务组件]: http://components.xamarin.com/view/azure-mobile-services/
  [Google Cloud Messaging 组件]: http://components.xamarin.com/view/GCMClient/
  [Azure 免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [accounts.google.com]: http://go.microsoft.com/fwlink/p/?LinkId=268302
  [Google API]: http://go.microsoft.com/fwlink/p/?LinkId=268303

  [使用通知中心将通知推送到用户]: /zh-cn/manage/services/notification-hubs/notify-users-aspnet
  [使用通知中心发送突发新闻]: /zh-cn/manage/services/notification-hubs/breaking-news-dotnet
  [通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
  [针对 Android 的通知中心操作指南]: http://msdn.microsoft.com/zh-cn/library/dn282661.aspx

<!-- Images. -->
  []: ./media/partner-xamarin-notification-hubs-android-get-started/mobile-services-google-developers.png
  [2]: ./media/partner-xamarin-notification-hubs-android-get-started/mobile-services-google-create-server.png
  [3]: ./media/partner-xamarin-notification-hubs-android-get-started/mobile-services-google-create-server2.png
  [4]: ./media/partner-xamarin-notification-hubs-android-get-started/mobile-services-google-create-server3.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [5]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-from-portal.png
  [6]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-from-portal2.png
  [7]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-select-from-portal.png
  [8]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-select-from-portal2.png
  [9]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-configure-android.png
  [10]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-connection-strings.png
  [11]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-app1.png
  [12]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-app2.png
  [13]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-app3.png
  [Xamarin.NotificationHub Github 页]: https://github.com/SaschaDittmann/Xamarin.NotificationHub
  [14]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-android-app7.png
  [15]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-android-app8.png
  [REST 接口]: http://msdn.microsoft.com/zh-cn/library/azure/dn223264.aspx
  [16]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-console-app.png
  [WindowsAzure.ServiceBus NuGet 包]: http://nuget.org/packages/WindowsAzure.ServiceBus/
  [17]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-android-toast.png
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started-xamarin-android/#create-new-service
  [18]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-scheduler1.png
  [19]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-scheduler2.png



