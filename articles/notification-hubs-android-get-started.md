<properties linkid="develop-notificationhubs-tutorials-get-started-android" urlDisplayName="Get Started" pageTitle="Get Started with Azure Notification Hubs" metaKeywords="" description="Learn how to use Azure Notification Hubs to push notifications." metaCanonical="" services="notification-hubs" documentationCenter="Mobile" title="Get started with Notification Hubs" authors="ricksal" solutions="" manager="dwrede" editor="" />

# 通知中心入门

<div class="dev-center-tutorial-selector sublanding"><a href="/en-us/documentation/articles/notification-hubs-windows-store-dotnet-get-started/" title="Windows 应用商店 C#">Windows 应用商店 C#</a><a href="/en-us/documentation/articles/notification-hubs-windows-phone-get-started/" title="Windows Phone">Windows Phone</a><a href="/en-us/documentation/articles/notification-hubs-ios-get-started/" title="iOS">iOS</a><a href="/en-us/documentation/articles/notification-hubs-android-get-started/" title="Android" class="current">Android</a><a href="/en-us/documentation/articles/notification-hubs-kindle-get-started/" title="Kindle" class="current">Kindle</a><a href="/en-us/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/" title="Xamarin.iOS" class="current">Xamarin.iOS</a><a href="/en-us/documentation/articles/partner-xamarin-notification-hubs-android-get-started/" title="Xamarin.Android" class="current">Xamarin.Android</a></div>

本主题说明如何使用 Azure 通知中心向 Android 应用程序发送推送通知。
在本教程中，你将要创建一个使用 Google Cloud Messaging (GCM) 接收推送通知的空白 Android 应用程序。完成后，你将能使用通知中心将推送通知广播到运行你的应用程序的所有设备。

本教程将指导你完成启用推送通知的以下基本步骤：

-   [启用 Google Cloud Messaging][启用 Google Cloud Messaging]
-   [配置通知中心][配置通知中心]
-   [将你的应用程序连接到通知中心][将你的应用程序连接到通知中心]
-   [如何向应用程序发送通知][如何向应用程序发送通知]
-   [测试应用程序][测试应用程序]

本教程演示使用通知中心的简单广播方案。请确保随后学习下一教程以了解如何使用通知中心来发送到特定用户和设备组。

本教程需要的内容如下：

-   Android SDK（假设你要使用 Eclipse），你可以从[此处][此处]下载该 SDK。
-   [移动服务 Android SDK][移动服务 Android SDK]

完成本教程是学习有关 Android 应用程序的所有其他通知中心教程的先决条件。

<div class="dev-callout"><strong>说明</strong> <p>若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/pricing/free-trial/" target="_blank">Azure 免费试用</a>。</p></div>

## <span id="register"></span></a>启用 Google Cloud Messaging

[WACOM.INCLUDE [启用 GCM][启用 GCM]]

接下来，你将使用此 API 密钥值让通知中心向 GCM 进行身份验证并代表你的应用程序发送推送通知。

## <span id="configure-hub"></span></a>配置通知中心

1.  登录到 [Azure 管理门户][Azure 管理门户]，然后单击屏幕底部的“+新建”。

2.  依次单击“应用程序服务”、“Service Bus”、“通知中心”和“快速创建”。

    ![][]

3.  键入通知中心的名称，选择所需的区域，然后单击“创建新的通知中心”。

    ![][1]

4.  单击刚刚创建的命名空间（通常为***通知中心名称*-ns**），然后单击顶部的“配置”。

    ![][2]

5.  单击顶部的“通知中心”选项卡，然后单击你刚刚创建的通知中心。

    ![][3]

6.  单击顶部的“配置”选项卡，输入在上一部分中获得的“API 密钥”值，然后单击“保存”。

    ![][4]

7.  选择顶部的“仪表板”选项卡，然后单击“查看连接字符串”。记下两个连接字符串。

你的通知中心现在已配置为使用 GCM，并且你有连接字符串用于注册你的应用程序和发送推送通知。

## <span id="connecting-app"></span></a>将你的应用程序连接到通知中心

### 创建新的 Android 项目

1.  在 Eclipse ADT 中，创建新的 Android 项目（依次单击“文件”、“新建”、“Android 应用程序”）。

    ![][5]

2.  确保“要求的最低 SDK”已设置为“API 8:Android 2.2 (Froyo)”，并且后面的两个 SDK 条目已设置为最新的可用版本。选择“下一步”，然后按向导说明操作，确保选择“创建活动”以创建空白活动。在下一个框中接受默认的启动器图标，然后在最后一个框中单击“完成”。

    ![][6]

### 将 Google Play Services 添加到项目

[WACOM.INCLUDE [添加 Play Services][添加 Play Services]]

### 添加代码

1.  从[此处][移动服务 Android SDK]下载通知中心 Android SDK。解压缩 .zip 文件并在包资源管理器中将 notificationhubs\\notification-hubs-0.1.jar 文件复制到你项目的 \\libs 目录。

2.  下载和解压缩[移动服务 Android SDK][移动服务 Android SDK]，打开 **notifications** 文件夹，将 **notifications-1.0.1.jar** 文件复制到 Eclipse 项目的 *libs* 文件夹，并刷新 *libs* 文件夹。

    <div class="dev-callout"><b>说明</b>
<p>在后续的 SDK 版本中，文件名末尾的数字可能更改。</p>
</div>

    现在，设置该应用程序以从 GCM 获取 *registrationId*，并使用它将应用程序实例注册到通知中心。

3.  在 AndroidManifest.xml 文件中，在紧挨 <uses-sdk></uses-sdk> 元素下添加以下行。确保将 `<your package>` 替换为你在步骤 1 中为应用程序选择的包（在本示例中为`com.yourCompany.wams_notificationhubs` ）。

        <uses-permission android:name="android.permission.INTERNET"/>
        <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
        <uses-permission android:name="android.permission.WAKE_LOCK"/>
        <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

        <permission android:name="<your package>.permission.C2D_MESSAGE" android:protectionLevel="signature" />
        <uses-permission android:name="<your package>.permission.C2D_MESSAGE"/>

4.  在 **MainActivity** 类中添加以下语句。

        import android.os.AsyncTask;    
        import com.google.android.gms.gcm.*;
        import com.microsoft.windowsazure.messaging.*;
        import com.microsoft.windowsazure.notifications.NotificationsManager;

5.  将以下私有成员添加到类的顶部。

    <div class="dev-callout"><b>说明</b>
<p>确保将 SENDER_ID 设置为你前面获取的项目编号。</p>
</div>

        private String SENDER_ID = "<your project number>";
        private GoogleCloudMessaging gcm;
        private NotificationHub hub;

6.  在 **OnCreate** 方法中添加以下代码，确保将占位符替换为你的连接字符串（它具有在前一部分中获取的侦听访问权限）以及 Azure 中与你的中心对应的页面顶部显示的通知中心名称（**不是**完整 URL）。

        NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);

        gcm = GoogleCloudMessaging.getInstance(this);

        String connectionString = "<your listen access connection string>";
        hub = new NotificationHub("<your notification hub name>", connectionString, this);

        registerWithNotificationHubs();

7.  在 MainActivity.java 中，创建以下方法：

        @SuppressWarnings("unchecked")
        private void registerWithNotificationHubs() {
           new AsyncTask() {
              @Override
              protected Object doInBackground(Object... params) {
                 try {
                    String regid = gcm.register(SENDER_ID);
                    hub.register(regid);
                 } catch (Exception e) {
                    return e;
                 }
                 return null;
             }
           }.execute(null, null, null);
        }

8.  因为 Android 不显示通知，你必须编写自己的接收器。在 **AndroidManifest.xml** 中，在`<application/>` 元素内添加以下元素。

    <div class="dev-callout"><b>说明</b>
<p>将占位符替换为你的包名称。</p>
</div>

        <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
            android:permission="com.google.android.c2dm.permission.SEND">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="**my_app_package**" />
            </intent-filter>
        </receiver>

9.  在包资源管理器中，右键单击包（在 `src` 节点下面），单击“新建”，然后单击“类”。

10. 在“名称”中，键入`MyHandler`，在“超类”中，键入`com.microsoft.windowsazure.notifications.NotificationsHandler`，然后单击“完成”。

    ![][7]

    这样可创建新的 MyHandler 类。

11. 添加以下 import 语句：

        import android.app.NotificationManager;
        import android.app.PendingIntent;
        import android.content.Context;
        import android.content.Intent;
        import android.os.Bundle;
        import android.support.v4.app.NotificationCompat;

12. 将以下代码添加到类：

        public static final int NOTIFICATION_ID = 1;
        private NotificationManager mNotificationManager;
        NotificationCompat.Builder builder;
        Context ctx;


        @Override
        public void onReceive(Context context, Bundle bundle) {
            ctx = context;
            String nhMessage = bundle.getString("msg");

            sendNotification(nhMessage);
        }

        private void sendNotification(String msg) {
            mNotificationManager = (NotificationManager)
                      ctx.getSystemService(Context.NOTIFICATION_SERVICE);

            PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
                  new Intent(ctx, MainActivity.class), 0);

            NotificationCompat.Builder mBuilder =
                  new NotificationCompat.Builder(ctx)
                  .setSmallIcon(R.drawable.ic_launcher)
                  .setContentTitle("Notification Hub Demo")
                  .setStyle(new NotificationCompat.BigTextStyle()
                             .bigText(msg))
                  .setContentText(msg);

             mBuilder.setContentIntent(contentIntent);
             mNotificationManager.notify(NOTIFICATION_ID, mBuilder.build());
        }

## <a name="send"></a>如何向应用程序发送通知

你可以使用通知中心通过 [REST 接口][REST 接口]从任意后端发送通知。本教程演示了发送通知的两种方法：使用 .NET 控制台应用程序，以及结合使用移动服务和节点脚本。

### 使用 .NET 控制台应用程序发送通知：

1.  创建新的 Visual C# 控制台应用程序：

    ![][8]

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

### 使用移动服务发送通知

1.  登录到 [Azure 管理门户][Azure 管理门户]并选择你的移动服务。如果你还没有移动服务，请按[移动服务入门][移动服务入门]中的说明操作。

2.  选择顶部的“计划程序”选项卡。

    ![][9]

3.  创建新的计划作业，插入名称，然后选择“按需”。

    ![][10]

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

## <a name="run-app"></a>测试应用程序

若要使用真正的电话来测试此应用程序，你只需使用 USB 电缆将它连接到计算机即可。

使用模拟器测试此应用程序：

1.  确保使用支持 Google API 的 Android 虚拟设备 (AVD)。

2.  从“窗口”中，单击“Android 虚拟设备管理器”，选择你的设备，然后单击“编辑”。

    ![][11]

3.  在“目标”中选择“Google API”，然后单击“确定”。

    ![][12]

4.  为了接收推送通知，你必须在 Android 虚拟设备上设置 Google 帐户（方法如下：在模拟器中，导航到“设置”，然后单击“添加帐户”）。此外，请确保模拟器已连接到 Internet。

无论选择了哪种设备，接下来都请执行以下操作：

1.  在 Eclipse 顶部工具栏中，单击“运行”，然后选择你的应用程序。这会将你的应用程序加载到连接的电话，或者启动模拟器，然后加载并运行此应用程序。

2.  应用程序从 GCM 检索 *registrationId* 并注册到通知中心。

3.  现在，请使用上一部分介绍的方法之一将通知发送到应用程序：

    -   如果你使用的是 .Net 控制台应用程序，请在 Visual Studio 中按 F5 键以运行将要发送通知的应用程序。
    -   如果使用的是移动服务脚本，请单击移动服务屏幕底部栏上的“运行一次”，随后脚本将会发送通知。

4.  通知区域中（左上角）会出现一个图标。下拉通知抽屉可以查看该通知。

    ![][13]

## <a name="next-steps"> </a>后续步骤

在这个简单的示例中，你将通知广播到所有 Android 设备。为了针对特定用户广播，请参考教程[使用通知中心将通知推送到用户][使用通知中心将通知推送到用户]，如果你要按兴趣细分用户组，请参考[使用通知中心发送突发新闻][使用通知中心发送突发新闻]。在[通知中心指南][通知中心指南]中了解通知中心的详细使用信息。


<!-- URLs. -->

  [Windows 应用商店 C\#]: /en-us/documentation/articles/notification-hubs-windows-store-dotnet-get-started/ "Windows 应用商店 C#"
  [Windows Phone]: /en-us/documentation/articles/notification-hubs-windows-phone-get-started/ "Windows Phone"
  [iOS]: /en-us/documentation/articles/notification-hubs-ios-get-started/ "iOS"
  [Android]: /en-us/documentation/articles/notification-hubs-android-get-started/ "Android"
  [Kindle]: /en-us/documentation/articles/notification-hubs-kindle-get-started/ "Kindle"
  [Xamarin.iOS]: /en-us/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/ "Xamarin.iOS"
  [Xamarin.Android]: /en-us/documentation/articles/partner-xamarin-notification-hubs-android-get-started/ "Xamarin.Android"
  [启用 Google Cloud Messaging]: #register
  [配置通知中心]: #configure-hub
  [将你的应用程序连接到通知中心]: #connecting-app
  [如何向应用程序发送通知]: #send
  [测试应用程序]: #run-app
  [此处]: http://go.microsoft.com/fwlink/?LinkId=389797
  [移动服务 Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
  [Azure 免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [启用 GCM]: ../includes/mobile-services-enable-Google-cloud-messaging.md
  [Azure 管理门户]: https://manage.windowsazure.cn/

<!-- Images. --> 

  []: ./media/notification-hubs-android-get-started/notification-hub-create-from-portal.png
  [1]: ./media/notification-hubs-android-get-started/notification-hub-create-from-portal2.png
  [2]: ./media/notification-hubs-android-get-started/notification-hub-select-from-portal.png
  [3]: ./media/notification-hubs-android-get-started/notification-hub-select-from-portal2.png
  [4]: ./media/notification-hubs-android-get-started/notification-hub-configure-android.png
  [5]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app.png
  [6]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app2.png
  [添加 Play Services]: ../includes/mobile-services-add-Google-play-services.md
  [7]: ./media/notification-hubs-android-get-started/notification-hub-android-new-class.png
  [REST 接口]: http://msdn.microsoft.com/zh-cn/library/azure/dn223264.aspx
  [8]: ./media/notification-hubs-android-get-started/notification-hub-create-console-app.png
  [WindowsAzure.ServiceBus NuGet 包]: http://nuget.org/packages/WindowsAzure.ServiceBus/
  [移动服务入门]: /en-us/develop/mobile/tutorials/get-started/#create-new-service
  [9]: ./media/notification-hubs-android-get-started/notification-hub-scheduler1.png
  [10]: ./media/notification-hubs-android-get-started/notification-hub-scheduler2.png
  [11]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app7.png
  [12]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app8.png
  [13]: ./media/notification-hubs-android-get-started/notification-hub-android-toast.png
  [使用通知中心将通知推送到用户]: /en-us/manage/services/notification-hubs/notify-users-aspnet
  [使用通知中心发送突发新闻]: /en-us/manage/services/notification-hubs/breaking-news-dotnet
  [通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
