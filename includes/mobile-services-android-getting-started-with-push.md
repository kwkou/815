1. 在您的应用项目中，打开文件  `AndroidManifest.xml`。在随后两个步骤中，请将代码中的 _`**my_app_package**`_ 替换为您的项目的应用程序包名称，即  `manifest` 标签的  `package` 属性的值。 

2. 在现有  `uses-permission` 元素之后添加以下新权限：

        <permission android:name="**my_app_package**.permission.C2D_MESSAGE" 
            android:protectionLevel="signature" />
        <uses-permission android:name="**my_app_package**.permission.C2D_MESSAGE" /> 
        <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
        <uses-permission android:name="android.permission.GET_ACCOUNTS" />
        <uses-permission android:name="android.permission.WAKE_LOCK" />

3. 在  `application` 开始标记之后添加以下代码： 

        <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
            						 	android:permission="com.google.android.c2dm.permission.SEND">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="**my_app_package**" />
            </intent-filter>
        </receiver>


4. 下载和解压缩[移动服务 Android SDK]，打开 **notifications** 文件夹，将 **notifications-1.0.1.jar** 文件复制到 Eclipse 项目的  *libs* 文件夹，并刷新  *libs* 文件夹。

    <div class="dev-callout"><b>注意</b>
	<p>在后续的 SDK 版本中，文件名末尾的数字可能更改。</p>
    </div>

5.  打开文件  *ToDoItemActivity.java*，并添加以下 import 语句：

		import com.microsoft.windowsazure.notifications.NotificationsManager;
		import com.microsoft.windowsazure.mobileservices.notifications.Registration;


6. 将以下私有变量添加到类：将 _`<PROJECT_NUMBER>`_ 替换为 Google 在前一个过程中为您的应用分配的项目编号：

		public static final String SENDER_ID = "<PROJECT_NUMBER>";

7. 将 MobileServiceClient 的定义从私有静态更改为公共静态，以便它现在如下所示：

		public static MobileServiceClient mClient;


8. 在 ToDoActivity.java 中，向 ToDoActivity 类添加以下方法，以允许注册通知。

        /**
		 * Registers mobile services client to receive GCM push notifications
		 * @param gcmRegistrationId The Google Cloud Messaging session Id returned 
		 * by the call to GoogleCloudMessaging.register in NotificationsManager.handleNotifications
		 */
		public void registerForPush(String gcmRegistrationId)
		{
			String [] tags = {null};
			ListenableFuture<Registration> reg = mClient.getPush().register(gcmRegistrationId, tags);
			
	    	Futures.addCallback(reg, new FutureCallback<Registration>() {
	    		@Override
	    		public void onFailure(Throwable exc) {
	    			createAndShowDialog((Exception) exc, "Error");
	    		}
	    		
	    		@Override
	    		public void onSuccess(Registration reg) {
	    			createAndShowDialog(reg.getRegistrationId() + " resistered", "Registration");
	    		}
	    	});
		}



9. 接下来，我们需要添加一个新的类以处理通知。在"程序包资源管理器"中，右键单击程序包（在  `src` 节点下）、单击"新建"，然后单击"类"。

10. 在"名称"中键入  `MyHandler`、在"超类"中键入  `com.microsoft.windowsazure.notifications.NotificationsHandler`，然后单击"完成"

	![](./media/mobile-services-android-get-started-push/mobile-services-android-create-class.png)

	这样可创建新的 MyHandler 类。

11. 为  `MyHandler` 类添加以下 import 语句：

		import android.app.NotificationManager;
		import android.app.PendingIntent;
		import android.content.Context;
		import android.content.Intent;
		import android.os.AsyncTask;
		import android.os.Bundle;
		import android.support.v4.app.NotificationCompat;

	
12. 接下来，为  `MyHandler` 类添加以下成员：

		public static final int NOTIFICATION_ID = 1;
		private NotificationManager mNotificationManager;
		NotificationCompat.Builder builder;
		Context ctx;


13. 在  `MyHandler` 类中，添加以下代码以覆盖 **onRegistered** 方法：用于在移动服务通知中心中注册您的设备。

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



14. 在  `MyHandler` 类中，添加以下代码以覆盖 **onReceive** 方法，这样可在收到通知时显示通知。

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


15. 回到 TodoActivity.java 文件中，更新 ToDoActivity 类的 **onCreate** 方法，以注册通知处理程序类。请确保在 MobileServiceClient 实例化之后再添加此代码。


		NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);

    您的应用现已更新，可支持推送通知。

<!-- URLs. -->
[移动服务 Android SDK]: http://aka.ms/Iajk6q
<!--HONumber=41-->
