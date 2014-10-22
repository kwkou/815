<properties linkid="develop-notificationhubs-tutorials-get-started-kindle" urlDisplayName="Get Started" pageTitle="Get Started with Azure Notification Hubs" metaKeywords="" description="Learn how to use Azure Notification Hubs to send push notifications." metaCanonical="" services="notification-hubs" documentationCenter="Mobile" title="Get started with Notification Hubs" authors="sethm" solutions="" manager="dwrede" editor="" />

# 通知中心入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/" title="Windows 应用商店 C#">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/notification-hubs-windows-phone-get-started/" title="Windows Phone">Windows Phone</a><a href="/zh-cn/documentation/articles/notification-hubs-ios-get-started/" title="iOS">iOS</a><a href="/zh-cn/documentation/articles/notification-hubs-android-get-started/" title="Android">Android</a><a href="/zh-cn/documentation/articles/notification-hubs-kindle-get-started/" title="Kindle" class="current">Kindle</a><a href="/zh-cn/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/" title="Xamarin.iOS" class="current">Xamarin.iOS</a><a href="/zh-cn/documentation/articles/partner-xamarin-notification-hubs-android-get-started/" title="Xamarin.Android" class="current">Xamarin.Android</a></div>

本主题说明如何使用 Azure 通知中心向 Kindle 应用程序发送推送通知。
在本教程中，你将要创建一个使用 Google Cloud Messaging (GCM) 接收推送通知的空白 Kindle 应用程序。

本教程需要的内容如下：

-   Android SDK（假设你要使用 Eclipse），你可以从[此处][此处]下载该 SDK。
-   遵照[此处][1]所述的步骤设置 Kindle 的开发环境。

## 向开发人员门户添加新应用程序

1.  首先，请在[开发人员门户][开发人员门户]中创建一个应用程序。

    ![][]

2.  复制应用程序密钥。

    ![][2]

3.  在门户中单击应用程序的名称，然后单击“设备消息”选项卡。

    ![][3]

4.  单击“创建新的安全配置文件”，然后创建一个新的安全配置文件（例如 **TestAdm security profile**）。然后单击“保存”。

    ![][4]

5.  单击“安全配置文件”以查看你刚刚创建的安全配置文件。复制“客户端 ID”和“客户端机密”值以供稍后使用。

    ![][5]

## 创建 API 密钥

1.  使用管理员特权打开命令提示符。
2.  导航到 Android SDK 文件夹。
3.  输入以下命令：

        keytool -list -v -alias androiddebugkey -keystore ./debug.keystore

    ![][6]

4.  对于“密钥库”密码，请键入 **android**。

5.  复制 **MD5** 指纹。
6.  返回到开发人员门户，在“消息”选项卡中，单击“Android/Kindle”，输入应用程序包的名称（例如 **com.sample.notificationhubtest**）和“MD5”值，然后单击“生成 API 密钥”。

## 将凭据添加到中心

在门户中，将客户端机密和客户端 ID 添加到通知中心的“配置”选项卡。

## 设置应用程序

<div class="dev-callout"><b>说明</b>
<p>创建应用程序时，请至少使用 API 级别 17。</p>
    </div>

 将 ADM 库添加到你的 Eclipse 项目。

1.  若要获取 ADM 库，请[下载 SDK][下载 SDK]。解压缩 SDK zip 文件。
2.  在 Eclipse 中右键单击你的项目，然后单击“属性”。在左侧选择“Java 生成路径”，然后选择顶部的“库”选项卡。单击“添加外部 Jar”，然后从 Amazon SDK 解压缩到的目录中选择`\SDK\Android\DeviceMessaging\lib\amazon-device-messaging-*.jar` 文件。
3.  下载 NotificationHubs Android SDK（链接）。
4.  解压缩该包，然后在 Eclipse 中将文件 `notification-hubs-sdk.jar` 拖放到 `libs`文件夹中。

编辑你的应用程序清单以支持 ADM：

1.  在根清单元素中添加 Amazon 命名空间：

        xmlns:amazon="http://schemas.amazon.com/apk/res/android"

2.  在清单元素下添加权限作为第一个元素。将 **[YOUR PACKAGE NAME]** 替换为用于创建应用程序的包。

        <permission
         android:name="[YOUR PACKAGE NAME].permission.RECEIVE_ADM_MESSAGE"
         android:protectionLevel="signature" />

        <uses-permission android:name="android.permission.INTERNET"/>

        <uses-permission android:name="[YOUR PACKAGE NAME].permission.RECEIVE_ADM_MESSAGE" />

        <!-- This permission allows your app access to receive push notifications
        from ADM. -->
        <uses-permission android:name="com.amazon.device.messaging.permission.RECEIVE" />

        <!-- ADM uses WAKE_LOCK to keep the processor from sleeping when a message is received. -->
        <uses-permission android:name="android.permission.WAKE_LOCK" />

3.  插入以下元素作为应用程序元素的第一个子级。请记得将 **[YOUR SERVICE NAME]** 替换为你在下一部分中创建的 ADM 消息处理程序的名称（包括包），并将 **[YOUR PACKAGE NAME]** 替换为创建应用程序时所用的包名称。

        <amazon:enable-feature
              android:name="com.amazon.device.messaging"
                     android:required="true"/>
        <service
            android:name="[YOUR SERVICE NAME]"
            android:exported="false" />

        <receiver
            android:name="[YOUR SERVICE NAME]$Receiver"

            <!-- This permission ensures that only ADM can send your app registration broadcasts. -->
            android:permission="com.amazon.device.messaging.permission.SEND" >

            <!-- To interact with ADM, your app must listen for the following intents. -->
            <intent-filter>
          <action android:name="com.amazon.device.messaging.intent.REGISTRATION" />
          <action android:name="com.amazon.device.messaging.intent.RECEIVE" />

          <!-- Replace the name in the category tag with your app's package name. -->
          <category android:name="[YOUR PACKAGE NAME]" />
            </intent-filter>
        </receiver>

## 创建 ADM 消息处理程序：

1.  创建继承自 `com.amazon.device.messaging.ADMMessageHandlerBase` 的新类，并将其命名为 `MyADMMessageHandler`，如下图中所示：

    ![][7]

2.  添加以下 `import` 语句：

        import android.app.NotificationManager;
        import android.app.PendingIntent;
        import android.content.Context;
        import android.content.Intent;
        import android.support.v4.app.NotificationCompat;
        import com.amazon.device.messaging.ADMMessageReceiver;
        import com.microsoft.windowsazure.messaging.NotificationHub

3.  在创建的类中添加以下代码。请记得替换中心名称和连接字符串 (listen)：

        public static final int NOTIFICATION_ID = 1;
        private NotificationManager mNotificationManager;
        NotificationCompat.Builder builder;
        private static NotificationHub hub;
        public static NotificationHub getNotificationHub(Context context) {
            Log.v("com.wa.hellokindlefire", "getNotificationHub");
            if (hub == null) {
                hub = new NotificationHub("[hub name]", "[listen connection string]", context);
            }
            return hub;
        }

        public MyADMMessageHandler() {
                super("MyADMMessageHandler");
            }

            public static class Receiver extends ADMMessageReceiver
            {
                public Receiver()
                {
                    super(MyADMMessageHandler.class);
                }
            }

            private void sendNotification(String msg) {
                Context ctx = getApplicationContext();

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

4.  将以下代码添加到`OnMessage()` 方法：

        String nhMessage = intent.getExtras().getString("msg");
        sendNotification(nhMessage);

5.  将以下代码添加到`OnRegistered` 方法：

            try {
        getNotificationHub(getApplicationContext()).register(registrationId);
            } catch (Exception e) {
        Log.e("[your package name]", "Fail onRegister: " + e.getMessage(), e);
            }

6.  将以下代码添加到`OnUnregistered` 方法：

            try {
                getNotificationHub(getApplicationContext()).unregister();
            } catch (Exception e) {
                Log.e("[your package name]", "Fail onUnregister: " + e.getMessage(), e);
            }

7.  然后，在 `MainActivity` 方法中添加以下 import 语句：

        import com.amazon.device.messaging.ADM;             

8.  现在，请在`OnCreate` 方法的末尾添加以下代码：

        final ADM adm = new ADM(this);
        if (adm.getRegistrationId() == null)
        {
           adm.startRegister();
        } else {
            new AsyncTask() {
                  @Override
                  protected Object doInBackground(Object... params) {
                     try {                       MyADMMessageHandler.getNotificationHub(getApplicationContext()).register(adm.getRegistrationId());
                     } catch (Exception e) {
                         Log.e("com.wa.hellokindlefire", "Failed registration with hub", e);
                         return e;
                     }
                     return null;
                 }
               }.execute(null, null, null);
        }

将 ADM 库添加到你的 Eclipse 项目。

1.  若要获取 ADM 库，请[下载 SDK][下载 SDK]。解压缩 SDK zip 文件。
2.  在 Eclipse 中右键单击你的项目，然后单击“属性”。在左侧选择“Java 生成路径”，然后选择顶部的“库”选项卡。单击“添加外部 Jar”，然后从 Amazon SDK 解压缩到的目录中选择`\SDK\Android\DeviceMessaging\lib\amazon-device-messaging-*.jar` 文件。
3.  下载 NotificationHubs Android SDK（链接）。
4.  解压缩该包，然后在 Eclipse 中将文件 `notification-hubs-sdk.jar` 拖放到 `libs`文件夹中。

编辑你的应用程序清单以支持 ADM：

1.  在根清单元素中添加 Amazon 命名空间：

        xmlns:amazon="http://schemas.amazon.com/apk/res/android"

2.  在清单元素下添加权限作为第一个元素。将 **[YOUR PACKAGE NAME]** 替换为用于创建应用程序的包。

        <permission
         android:name="[YOUR PACKAGE NAME].permission.RECEIVE_ADM_MESSAGE"
         android:protectionLevel="signature" />

        <uses-permission android:name="android.permission.INTERNET"/>

        <uses-permission android:name="[YOUR PACKAGE NAME].permission.RECEIVE_ADM_MESSAGE" />

        <!-- This permission allows your app access to receive push notifications
        from ADM. -->
        <uses-permission android:name="com.amazon.device.messaging.permission.RECEIVE" />

        <!-- ADM uses WAKE_LOCK to keep the processor from sleeping when a message is received. -->
        <uses-permission android:name="android.permission.WAKE_LOCK" />

3.  插入以下元素作为应用程序元素的第一个子级。请记得将 **[YOUR SERVICE NAME]** 替换为你在下一部分中创建的 ADM 消息处理程序的名称（包括包），并将 **[YOUR PACKAGE NAME]** 替换为创建应用程序时所用的包名称。

        <amazon:enable-feature
              android:name="com.amazon.device.messaging"
                     android:required="true"/>
        <service
            android:name="[YOUR SERVICE NAME]"
            android:exported="false" />

        <receiver
            android:name="[YOUR SERVICE NAME]$Receiver"

            <!-- This permission ensures that only ADM can send your app registration broadcasts. -->
            android:permission="com.amazon.device.messaging.permission.SEND" >

            <!-- To interact with ADM, your app must listen for the following intents. -->
            <intent-filter>
          <action android:name="com.amazon.device.messaging.intent.REGISTRATION" />
          <action android:name="com.amazon.device.messaging.intent.RECEIVE" />

          <!-- Replace the name in the category tag with your app's package name. -->
          <category android:name="[YOUR PACKAGE NAME]" />
            </intent-filter>
        </receiver>

## 创建 ADM 消息处理程序：

1.  创建继承自 `com.amazon.device.messaging.ADMMessageHandlerBase` 的新类，并将其命名为 `MyADMMessageHandler`，如下图中所示：

    ![][7]

2.  添加以下 `import` 语句：

        import android.app.NotificationManager;
        import android.app.PendingIntent;
        import android.content.Context;
        import android.content.Intent;
        import android.support.v4.app.NotificationCompat;
        import com.amazon.device.messaging.ADMMessageReceiver;
        import com.microsoft.windowsazure.messaging.NotificationHub

3.  在创建的类中添加以下代码。请记得替换中心名称和连接字符串 (listen)：

        public static final int NOTIFICATION_ID = 1;
        private NotificationManager mNotificationManager;
        NotificationCompat.Builder builder;
        private static NotificationHub hub;
        public static NotificationHub getNotificationHub(Context context) {
            Log.v("com.wa.hellokindlefire", "getNotificationHub");
            if (hub == null) {
                hub = new NotificationHub("[hub name]", "[listen connection string]", context);
            }
            return hub;
        }

        public MyADMMessageHandler() {
                super("MyADMMessageHandler");
            }

            public static class Receiver extends ADMMessageReceiver
            {
                public Receiver()
                {
                    super(MyADMMessageHandler.class);
                }
            }

            private void sendNotification(String msg) {
                Context ctx = getApplicationContext();

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

4.  将以下代码添加到`OnMessage()` 方法：

        String nhMessage = intent.getExtras().getString("msg");
        sendNotification(nhMessage);

5.  将以下代码添加到`OnRegistered` 方法：

            try {
        getNotificationHub(getApplicationContext()).register(registrationId);
            } catch (Exception e) {
        Log.e("[your package name]", "Fail onRegister: " + e.getMessage(), e);
            }

6.  将以下代码添加到`OnUnregistered` 方法：

            try {
                getNotificationHub(getApplicationContext()).unregister();
            } catch (Exception e) {
                Log.e("[your package name]", "Fail onUnregister: " + e.getMessage(), e);
            }

7.  然后，在 `MainActivity` 方法中添加以下 import 语句：

        import com.amazon.device.messaging.ADM;             

8.  现在，请在`OnCreate` 方法的末尾添加以下代码：

        final ADM adm = new ADM(this);
        if (adm.getRegistrationId() == null)
        {
           adm.startRegister();
        } else {
            new AsyncTask() {
                  @Override
                  protected Object doInBackground(Object... params) {
                     try {                       MyADMMessageHandler.getNotificationHub(getApplicationContext()).register(adm.getRegistrationId());
                     } catch (Exception e) {
                         Log.e("com.wa.hellokindlefire", "Failed registration with hub", e);
                         return e;
                     }
                     return null;
                 }
               }.execute(null, null, null);
        }


## 发送消息

使用 .NET 发送消息：

        static void Main(string[] args)
        {
            NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("[conn string]", "[hub name]");

            hub.SendAdmNativeNotificationAsync("{\"data\":{\"msg\" : \"Hello from .NET!\"}}").Wait();
        }

![][8]

<!-- URLs. -->

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/ "Windows 应用商店 C#"
  [Windows Phone]: /zh-cn/documentation/articles/notification-hubs-windows-phone-get-started/ "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/notification-hubs-ios-get-started/ "iOS"
  [Android]: /zh-cn/documentation/articles/notification-hubs-android-get-started/ "Android"
  [Kindle]: /zh-cn/documentation/articles/notification-hubs-kindle-get-started/ "Kindle"
  [Xamarin.iOS]: /zh-cn/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/ "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/documentation/articles/partner-xamarin-notification-hubs-android-get-started/ "Xamarin.Android"
  [此处]: http://go.microsoft.com/fwlink/?LinkId=389797
  [1]: https://developer.amazon.com/appsandservices/resources/development-tools/ide-tools/tech-docs/01-setting-up-your-development-environment
  [开发人员门户]: https://developer.amazon.com/home.html
  []: ./media/notification-hubs-kindle-get-started/notification-hub-kindle-portal1.png
  [2]: ./media/notification-hubs-kindle-get-started/notification-hub-kindle-portal2.png
  [3]: ./media/notification-hubs-kindle-get-started/notification-hub-kindle-portal3.png
  [4]: ./media/notification-hubs-kindle-get-started/notification-hub-kindle-portal4.png
  [5]: ./media/notification-hubs-kindle-get-started/notification-hub-kindle-portal5.png
  [6]: ./media/notification-hubs-kindle-get-started/notification-hub-kindle-cmd-window.png
  [下载 SDK]: https://developer.amazon.com/public/resources/development-tools/sdk
  [7]: ./media/notification-hubs-kindle-get-started/notification-hub-kindle-new-java-class.png
  [8]: ./media/notification-hubs-kindle-get-started/notification-hub-kindle-notification.png
