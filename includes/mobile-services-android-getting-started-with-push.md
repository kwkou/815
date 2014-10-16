1.  打开文件 `AndroidManifest.xml`。在随后两个步骤中，请将代码中的 *`"my_app_package"`* 替换为你的项目的应用程序包名称，即 `manifest` 标签的 `package` 属性的值。

2.  在现有 `uses-permission` 元素之后添加以下新权限：

        <permission android:name=""my_app_package".permission.C2D_MESSAGE" 
        android:protectionLevel="signature" />
        <uses-permission android:name=""my_app_package".permission.C2D_MESSAGE" /> 
        <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
        <uses-permission android:name="android.permission.GET_ACCOUNTS" />
        <uses-permission android:name="android.permission.WAKE_LOCK" />

3.  在 `application` 开始标记之后添加以下代码：

        <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
        android:permission="com.google.android.c2dm.permission.SEND">
        <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <category android:name=""my_app_package"" />
        </intent-filter>
        </receiver>

4.  下载和解压缩 [移动服务 Android SDK]，打开 "notifications" 文件夹，将 "notifications-1.0.1.jar" 文件复制到 Eclipse 项目的 *libs* 文件夹，并刷新 *libs* 文件夹。

    <div class="dev-callout"><b>说明</b>

    <p>在后续的 SDK 版本中，文件名末尾的数字可能更改。</p>
	</div>

5.  打开文件 *ToDoItemActivity.java*，并添加以下 import 语句：

        import com.microsoft.windowsazure.notifications.NotificationsManager;

6.  将以下私有变量添加到类，其中 *`<PROJECT_NUMBER>`* 是 Google 在前一个过程中为你的应用程序分配的项目编号：

        public static final String SENDER_ID = "<PROJECT_NUMBER>";

7.  在 "onCreate" 方法中，在 MobileServiceClient 实例化之前，添加以下代码以注册设备的通知处理程序：

        NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);

    随后，我们要定义在这段代码中引用的 MyHandler.class。

8.  在“包资源管理器”中，右键单击包（在 `src` 节点下），单击“新建” ，再单击“类” 。

9.  在“名称”中， 键入 `MyHandler`，在“超类”中 ，键入 `com.microsoft.windowsazure.notifications.NotificationsHandler`，然后单击“完成” 

    ![][0]

    这样可创建新的 MyHandler 类。

10. 添加以下 import 语句：

        import android.content.Context;
        import android.content.Context;
        import android.content.Intent;

        import com.microsoft.windowsazure.mobileservices.RegistrationCallback;

11. 接下来，将这段代码添加到类：

        public static final int NOTIFICATION_ID = 1;
        private NotificationManager mNotificationManager;
        NotificationCompat.Builder builder;
        Context ctx;

12. 添加以下代码以覆盖 "onRegistered" 方法：它向移动服务通知中心注册你的设备。

        @Override
        public void onRegistered(Context context, String gcmRegistrationId) {
        super.onRegistered(context, gcmRegistrationId);

        ToDoActivity.mClient.getPush().register(gcmRegistrationId, null, new RegistrationCallback() {
        @Override
        public void onRegister(Registration registration, Exception exception) {
        if (exception != null) {
        // handle error
                      }
                }
            });
        }

13. 添加以下代码以覆盖 "onReceive" 方法，这样可在收到通知时显示通知。

        @Override
        public void onReceive(Context context, Bundle bundle) {
        ctx = context;
        String nhMessage = bundle.getString("message");

        sendNotification(nhMessage);
        }

        private void sendNotification(String msg) {
        mNotificationManager = (NotificationManager)
        ctx.getSystemService(Context.NOTIFICATION_SERVICE);

        PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
        new Intent(ctx, ToDoActivity.class), 0);

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

你的应用程序现已更新，可支持推送通知。

  [0]: ./media/mobile-services-android-get-started-push/mobile-services-android-create-class.png
