1. 在你的应用项目中，打开文件 `AndroidManifest.xml`。在随后两个步骤中，请将代码中的 _`**my_app_package**`_ 替换为你的项目的应用程序包名称，即 `manifest` 标记的 `package` 属性的值。 

2. 在现有 `uses-permission` 元素之后添加以下新权限：

        <permission android:name="**my_app_package**.permission.C2D_MESSAGE" 
            android:protectionLevel="signature" />
        <uses-permission android:name="**my_app_package**.permission.C2D_MESSAGE" /> 
        <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
        <uses-permission android:name="android.permission.GET_ACCOUNTS" />
        <uses-permission android:name="android.permission.WAKE_LOCK" />

3. 在 `application` 开始标记之后添加以下代码：

        <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
            						 	android:permission="com.google.android.c2dm.permission.SEND">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="**my_app_package**" />
            </intent-filter>
        </receiver>


4. 下载和解压缩[移动服务 Android SDK]，打开 **notifications** 文件夹，将 **notifications-1.0.1.jar** 文件复制到 Eclipse 项目的 *libs* 文件夹，并刷新 *libs* 文件夹。

    > [AZURE.NOTE]在后续的 SDK 版本中，文件名末尾的数字可能更改。

5.  打开文件 *ToDoItemActivity.java*，并添加以下 import 语句：

		import com.microsoft.windowsazure.notifications.NotificationsManager;


6. 将以下私有变量添加到类：将 _`<PROJECT_NUMBER>`_ 替换为 Google 在前一个过程中为你的应用分配的项目编号：

		public static final String SENDER_ID = "<PROJECT_NUMBER>";

7. 将 *MobileServiceClient* 的定义从 **private** 更改为 **public static**，以便它现在如下所示：

		public static MobileServiceClient mClient;



9. 接下来，我们需要添加一个新的类以处理通知。在程序包资源管理器中，右键单击程序包（在 `src` 节点下）、单击“新建”，然后单击“类”。

10. 在“名称”中键入、在“超类”中键入，然后单击“完成”`MyHandler``com.microsoft.windowsazure.notifications.NotificationsHandler`

	![](./media/mobile-services-android-get-started-push/mobile-services-android-create-class.png)

	这样可创建新的 MyHandler 类。

11. 为 `MyHandler` 类添加以下 import 语句：

		import android.app.NotificationManager;
		import android.app.PendingIntent;
		import android.content.Context;
		import android.content.Intent;
		import android.os.AsyncTask;
		import android.os.Bundle;
		import android.support.v4.app.NotificationCompat;

	
12. 接下来，为 `MyHandler` 类添加以下成员：

		public static final int NOTIFICATION_ID = 1;
		private NotificationManager mNotificationManager;
		NotificationCompat.Builder builder;
		Context ctx;


13. 在 `MyHandler` 类中，添加以下代码以覆盖 **onRegistered** 方法，用于在移动服务通知中心中注册你的设备。

		@Override
		public void onRegistered(Context context,  final String gcmRegistrationId) {
		    super.onRegistered(context, gcmRegistrationId);
	
		    new AsyncTask<Void, Void, Void>() {
		    		    	
		    	protected Void doInBackground(Void... params) {
		    		try {
		    		    ToDoActivity.mClient.getPush().register(gcmRegistrationId, null);
		    		    return null;
	    		    }
	    		    catch(Exception e) { 
			    		// handle error    		    
	    		    }
					return null;  		    
	    		}
		    }.execute();
		}



14. 在 `MyHandler` 类中，添加以下代码以覆盖 **onReceive** 方法，这样可在收到通知时显示通知。

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


15. 回到 TodoActivity.java 文件中，更新 *ToDoActivity* 类的 **onCreate** 方法，以注册通知处理程序类。请确保在 *MobileServiceClient* 实例化之后再添加此代码。


		NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);

    你的应用现已更新，可支持推送通知。

<!-- URLs. -->
[移动服务 Android SDK]: https://zumo.blob.core.windows.net/sdk/azuresdk-android-2.0.2-beta2.zip

<!---HONumber=74-->