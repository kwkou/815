<properties
	pageTitle="通过百度开始使用 Azure 通知中心 | Azure"
	description="在本教程中，你将了解如何通过百度使用 Azure 通知中心将通知推送到 Android 设备。"
	services="notification-hubs"
	documentationCenter="android"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="notification-hubs"
	ms.date="05/05/2016"
	wacn.date="05/31/2016"/>
	
# 通过百度开始使用通知中心

[AZURE.INCLUDE [notification-hubs-selector-get-started](../includes/notification-hubs-selector-get-started.md)]

##概述

百度云推送是一种中国云服务，可用于将推送通知发送到移动设备。在中国，将推送通知传递到 Android 的过程很复杂，因为存在不同的应用程序商店和推送服务，还有通常未连接到 GCM (Google Cloud Messaging) 的 Android 设备，所以，此服务在该国特别有用。

##先决条件

本教程需要的内容如下：

+ Android SDK（我们假设你要使用 Eclipse），可从 <a href="http://go.microsoft.com/fwlink/?LinkId=389797">Android 站点</a>下载该 SDK
+ [移动服务 Android SDK]
+ [百度推送 Android SDK]

>[AZURE.NOTE]若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 免费试用](/pricing/free-trial/)。


##创建百度帐户

若要使用百度，你必须有一个百度帐户。如果你已有帐户，请登录[百度门户]，并跳到下一步。否则请参阅以下说明创建百度帐户。

1. 转到[百度门户]并单击“登录”链接。单击“立即注册”以启动帐户注册过程。

   	![][1]

2. 输入所需的详细信息（电话/电子邮件地址、密码和验证码），然后单击“注册”。

   	![][2]

3. 系统会将一封电子邮件发送到你输入的电子邮件地址，该邮件包含一个用于激活你的百度帐户的链接。

    ![][3]

4. 登录到你的电子邮件帐户，打开百度激活邮件，然后单击激活链接以激活你的百度帐号。

   	![][4]

当你有已激活的百度帐户后，请登录[百度门户]。

##<a id="registerBaiduDeveloper"></a>注册为百度开发者

1. 登录到[百度门户]后，请单击“更多>>”。

  	![][5]

2. 向下滚动“站长与开发者服务”部分并单击“百度开放云平台”。

  	![][6]

3. 在下一页上，单击右上角的“开发者服务”。

  	![][7]

4. 在下一页上，单击右上角菜单中的“注册开发者”。

  	![][8]

5. 输入你的名称、说明以及用于接收验证短信的手机号码，然后单击“发送验证码”。请注意，对于国际电话号码，你需要将国家/地区代码括在括号内。例如，对于美国号码，电话号码类似于 **(1)1234567890**。

  	![][9]

6. 然后，你应收到一条包含验证码的短信，如以下示例所示：

  	![][10]

7. 在“验证码”中输入消息中的验证码。

8. 最后，接受百度协议并单击“提交”以完成开发者注册。成功完成注册时，你会看到以下页：

  	![][11]

##<a id="createBaiduPushProject"></a>创建百度云推送项目

在创建百度云推送项目时，你将收到应用程序 ID、API 密钥和密钥。

1. 登录到[百度门户]后，请单击“更多>>”。

  	![][5]

2. 向下滚动“站长与开发者服务”部分并单击“百度开放云平台”。

  	![][6]

3. 在下一页上，单击右上角的“开发者服务”。

  	![][7]

4. 在下一页上，单击“云服务”部分中的“云推送”。

  	![][12]

5. 注册开发者后，你将在顶部菜单上看到“管理控制台”。单击“开发者服务管理”。

  	![][13]

6. 在下一页上，单击“创建工程”。

  	![][14]

7. 输入应用程序名称，并单击“创建”。

  	![][15]

8. 成功创建百度云推送项目后，将显示一个页面，其中包含“AppID”、“API 密钥”和“密钥”。请记下 API 密钥和密钥，因为稍后将要用到。

  	![][16]

9. 通过单击左侧窗格中的“云推送”来配置推送通知项目。

  	![][31]

10. 在下一页上，单击“推送设置”按钮。

	![][32]

11. 在配置页的“应用包名”字段中添加将在 Android 项目中使用的包名，然后单击“保存设置”

	![][33]

你将会看到**“保存成功!”**消息。

##<a id="configure-hub"></a>配置通知中心

1. 登录到 [Azure 经典管理门户]，然后单击屏幕底部的“+新建”。

2. 依次单击“应用程序服务”、“服务总线”、“通知中心”和“快速创建”。

3. 为你的**通知中心**提供名称，选择要在其中创建此通知中心的**区域**和**命名空间**，然后单击“创建新的通知中心”。

  	![][17]

4. 单击已在其中创建通知中心的命名空间，然后单击顶部的“通知中心”。

  	![][18]

5. 选择所创建的通知中心，然后单击顶部菜单中的“配置”。

  	![][19]

6. 向下滚动到“百度通知设置”部分，然后输入先前从百度控制台获得的百度云推送项目的 API 密钥和密钥。单击“保存”。

  	![][20]

7. 单击通知中心顶部的“仪表板”选项卡，然后单击“查看连接字符串”。

  	![][21]

8. 记下“访问连接信息”窗口中的 **DefaultListenSharedAccessSignature** 和 **DefaultFullSharedAccessSignature**。

    ![][22]

##<a id="connecting-app"></a>将你的应用连接到通知中心

1. 在 Eclipse ADT 中，创建新的 Android 项目（“文件”>“新建”>“Android 应用程序项目”）。

    ![][23]

2. 输入**应用程序名称**，并确保将**要求的最低 SDK 版本**设为 **API 16: Android 4.1**。

    ![][24]

3. 单击“下一步”，然后继续执行向导，直到显示“创建活动”窗口。确保选中了“空白活动”，最后选择“完成”以创建新的 Android 应用程序。

    ![][25]

4. 确保“项目生成目标”已正确设置。

    ![][26]

5. 从 [Notification-Hubs-Android-SDK on Bintray](https://bintray.com/microsoftazuremobile/SDK/Notification-Hubs-Android-SDK/0.4) 的“文件”选项卡下载 notification-hubs-0.4.jar 文件。将该文件添加到 Eclipse 项目的 **libs** 文件夹，然后刷新 *libs* 文件夹。

6. 下载并解压缩[百度推送 Android SDK]，打开 **libs** 文件夹，然后将 **pushservice-x.y.z** jar 文件以及 **armeabi** 和 **mips** 文件夹复制到 Android 应用程序的 **libs** 文件夹。

7. 打开 Android 项目的 **AndroidManifest.xml** 文件，并添加百度 SDK 所需的权限。

	    <uses-permission android:name="android.permission.INTERNET" />
	    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
	    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
	    <uses-permission android:name="android.permission.WRITE_SETTINGS" />
	    <uses-permission android:name="android.permission.VIBRATE" />
	    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	    <uses-permission android:name="android.permission.DISABLE_KEYGUARD" />
	    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
	    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
	    <uses-permission android:name="android.permission.ACCESS_DOWNLOAD_MANAGER" />
	    <uses-permission android:name="android.permission.DOWNLOAD_WITHOUT_NOTIFICATION" />

8. 向 **AndroidManifest.xml** 中的 **application** 元素添加 **android:name** 属性，并替换 *yourprojectname*（例如 **com.example.BaiduTest**）。确保此项目名称与你在百度控制台中配置的项目名称匹配。

		<application android:name="yourprojectname.DemoApplication"

9. 在 **.MainActivity** 活动元素后的 application 元素内添加以下配置，并替换 *yourprojectname*（例如 **com.example.BaiduTest**）：

		<receiver android:name="yourprojectname.MyPushMessageReceiver">
		    <intent-filter>
		        <action android:name="com.baidu.android.pushservice.action.MESSAGE" />
		        <action android:name="com.baidu.android.pushservice.action.RECEIVE" />
		        <action android:name="com.baidu.android.pushservice.action.notification.CLICK" />
		    </intent-filter>
		</receiver>

		<receiver android:name="com.baidu.android.pushservice.PushServiceReceiver"
		    android:process=":bdservice_v1">
		    <intent-filter>
		        <action android:name="android.intent.action.BOOT_COMPLETED" />
		        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
				<action android:name="com.baidu.android.pushservice.action.notification.SHOW" />
		    </intent-filter>
		</receiver>

        <receiver android:name="com.baidu.android.pushservice.RegistrationReceiver"
            android:process=":bdservice_v1">
            <intent-filter>
                <action android:name="com.baidu.android.pushservice.action.METHOD" />
                <action android:name="com.baidu.android.pushservice.action.BIND_SYNC" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.PACKAGE_REMOVED"/>
                <data android:scheme="package" />
            </intent-filter>                   
        </receiver>
        
        <service
            android:name="com.baidu.android.pushservice.PushService"
            android:exported="true"
            android:process=":bdservice_v1"  >
            <intent-filter>
                <action android:name="com.baidu.android.pushservice.action.PUSH_SERVICE" />
            </intent-filter>
        </service>

9. 将一个名为 **ConfigurationSettings.java** 的新类添加到项目中。

    ![][28]

    ![][29]

10. 将以下代码添加到该类中：

		public class ConfigurationSettings {
		        public static String API_KEY = "...";
				public static String NotificationHubName = "...";
				public static String NotificationHubConnectionString = "...";
			}

	使用前面从百度云项目中检索到的内容设置 **API\_KEY** 的值，使用 Azure 经典管理门户中的通知中心名称设置 **NotificationHubName**，并使用 Azure 经典管理门户中的 DefaultListenSharedAccessSignature 设置 **NotificationHubConnectionString**。

11. 添加一个名为 **DemoApplication.java** 的新类，并向此类中添加以下代码：

		import com.baidu.frontia.FrontiaApplication;
		
		public class DemoApplication extends FrontiaApplication {
		    @Override
		    public void onCreate() {
		        super.onCreate();
		    }
		}

12. 添加另一个名为 **MyPushMessageReceiver.java** 的新类，并向此类中添加以下代码。此类用于处理从百度推送服务器收到的推送通知：

		import java.util.List;
		import android.content.Context;
		import android.os.AsyncTask;
		import android.util.Log;
		import com.baidu.frontia.api.FrontiaPushMessageReceiver;
		import com.microsoft.windowsazure.messaging.NotificationHub;
		
		public class MyPushMessageReceiver extends FrontiaPushMessageReceiver {
		    /** TAG to Log */
			public static NotificationHub hub = null;
			public static String mChannelId, mUserId;
		    public static final String TAG = MyPushMessageReceiver.class
		            .getSimpleName();
		    
			@Override
		    public void onBind(Context context, int errorCode, String appid,
		            String userId, String channelId, String requestId) {
		        String responseString = "onBind errorCode=" + errorCode + " appid="
		                + appid + " userId=" + userId + " channelId=" + channelId
		                + " requestId=" + requestId;
		        Log.d(TAG, responseString);
		        mChannelId = channelId;
		        mUserId = userId;
		        
		        try {
		       	 if (hub == null) {
		                hub = new NotificationHub(
		                		ConfigurationSettings.NotificationHubName, 
		                		ConfigurationSettings.NotificationHubConnectionString, 
		                		context);
		                Log.i(TAG, "Notification hub initialized");
		            }
		        } catch (Exception e) {
		           Log.e(TAG, e.getMessage());
		        }
		        
		        registerWithNotificationHubs();
			}
		    
		    private void registerWithNotificationHubs() {
		       new AsyncTask<Void, Void, Void>() {
		          @Override
		          protected Void doInBackground(Void... params) {
		             try {
		            	 hub.registerBaidu(mUserId, mChannelId);
		            	 Log.i(TAG, "Registered with Notification Hub - '" 
		     	    			+ ConfigurationSettings.NotificationHubName + "'" 
		     	    			+ " with UserId - '"
		     	    			+ mUserId + "' and Channel Id - '"
		     	    			+ mChannelId + "'");
		             } catch (Exception e) {
		            	 Log.e(TAG, e.getMessage());
		             }
		             return null;
		         }
		       }.execute(null, null, null);
		    }
		    
		    @Override
		    public void onSetTags(Context context, int errorCode,
		            List<String> sucessTags, List<String> failTags, String requestId) {
		        String responseString = "onSetTags errorCode=" + errorCode
		                + " sucessTags=" + sucessTags + " failTags=" + failTags
		                + " requestId=" + requestId;
		        Log.d(TAG, responseString);
		    }
		
		    @Override
		    public void onDelTags(Context context, int errorCode,
		            List<String> sucessTags, List<String> failTags, String requestId) {
		        String responseString = "onDelTags errorCode=" + errorCode
		                + " sucessTags=" + sucessTags + " failTags=" + failTags
		                + " requestId=" + requestId;
		        Log.d(TAG, responseString);
		    }
		
		    @Override
		    public void onListTags(Context context, int errorCode, List<String> tags,
		            String requestId) {
		        String responseString = "onListTags errorCode=" + errorCode + " tags="
		                + tags;
		        Log.d(TAG, responseString);
		    }
		
		    @Override
		    public void onUnbind(Context context, int errorCode, String requestId) {
		        String responseString = "onUnbind errorCode=" + errorCode
		                + " requestId = " + requestId;
		        Log.d(TAG, responseString);
		    }
		
		    @Override
		    public void onNotificationClicked(Context context, String title,
		            String description, String customContentString) {
		        String notifyString = "title="" + title + "" description=""
		                + description + "" customContent=" + customContentString;
		        Log.d(TAG, notifyString);
		    }
		
		    @Override
		    public void onMessage(Context context, String message,
		            String customContentString) {
		        String messageString = "message="" + message + "" customContentString=" + customContentString;
		        Log.d(TAG, messageString);
		    }
		}

13. 打开 **MainActivity.java**，并将以下内容添加到 **onCreate** 方法中：

	        PushManager.startWork(getApplicationContext(),
	                PushConstants.LOGIN_TYPE_API_KEY, ConfigurationSettings.API_KEY);

14. 打开顶部的以下 import 语句：

			import com.baidu.android.pushservice.PushConstants;
			import com.baidu.android.pushservice.PushManager;

##<a id="send"></a>向应用程序发送通知


在 Azure 经典管理门户中通过通知中心上的调试选项卡（如以下屏幕中所示）来发送通知，可以在应用中测试通知的接收情况。

![](./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-debug.png)

通常，推送通知是在后端服务（例如，移动服务，或者使用兼容库的 ASP.NET）中发送的。如果你的后端没有可用的库，则你也可以使用 REST API 直接发送通知消息。

在本教程中，为了保持内容的简单性，我们只会演示如何在控制台应用程序（而不是后端服务）中，使用通知中心的 .NET SDK 发送通知，以此测试你的客户端应用。建议你接下来学习[使用通知中心向用户推送通知](/documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-notify-users/)教程，以了解如何从 ASP.NET 后端发送通知。不过，可以使用以下方法来发送通知：

* **REST 接口**：可以使用 [REST 接口](http://msdn.microsoft.com/library/windowsazure/dn223264.aspx)在任何后端平台上支持通知。

* **Azure 通知中心 .NET SDK**：在 Visual Studio 的 Nuget 包管理器中，运行 [Install-Package Microsoft.Azure.NotificationHubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)。

* **Node.js**：[如何通过 Node.js 使用通知中心](/documentation/articles/notification-hubs-nodejs-how-to-use-notification-hubs/)。

* **Azure 移动服务**：有关如何从通知中心集成的 Azure 移动服务后端发送通知的示例，请参阅 [Add push notifications to your Mobile Services app](/documentation/articles/mobile-services-javascript-backend-windows-universal-dotnet-get-started-push/)（将推送通知添加到移动服务应用）。

* **Java/PHP**：有关如何使用 REST API 发送通知的示例，请参阅“如何通过 Java/PHP 使用通知中心”([Java](/documentation/articles/notification-hubs-java-backend-how-to/) | [PHP](/documentation/articles/notification-hubs-php-backend-how-to/))。

##（可选）通过 .NET 控制台应用发送通知。

在本部分，我们将演示如何使用 .NET 控制台应用发送通知。

1. 创建新的 Visual C# 控制台应用程序：

	![][30]

2. 在“包管理器控制台”窗口中，将“默认项目”设置为新的控制台应用程序项目，然后在控制台窗口中执行以下命令：

        Install-Package Microsoft.Azure.NotificationHubs

	这将使用 <a href="http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/">Microsoft.Azure.Notification Hubs NuGet 包</a>添加对 Azure 通知中心 SDK 的引用。

	![](./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-package-manager.png)

3. 打开文件 **Program.cs** 并添加以下 using 语句：

        using Microsoft.Azure.NotificationHubs;

4. 在 `Program` 类中添加以下方法，并将 *DefaultFullSharedAccessSignatureSASConnectionString* 和 *NotificationHubName* 替换为你的值。

		private static async void SendNotificationAsync()
		{
			NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("DefaultFullSharedAccessSignatureSASConnectionString", "NotificationHubName");
			string message = "{"title":"((Notification title))","description":"Hello from Azure"}";
			var result = await hub.SendBaiduNativeNotificationAsync(message);
		}

5. 在 **Main** 方法中添加下列行：

         SendNotificationAsync();
		 Console.ReadLine();

##<a name="test-app"></a>测试应用程序


若要使用实际的手机测试此应用，只需使用 USB 电缆将该手机连接到你的计算机。这会将你的应用加载到连接的手机中。

若要使用模拟器测试此应用，请在 Eclipse 顶部工具栏中，单击“运行”，然后选择你的应用。这将启动模拟器，然后加载并运行该应用。

该应用将从百度推送通知服务检索“userId”和“channelId”，并注册到通知中心。

若要发送测试通知，可以使用 Azure 经典管理门户的调试选项卡。如果你为 Visual Studio 生成了 .NET 控制台应用程序，只需在 Visual Studio 中按 F5 键以运行该应用程序。该应用程序将发送一条通知，该通知显示在设备或模拟器的顶部通知区域。


<!-- Images. -->
[1]: ./media/notification-hubs-baidu-get-started/BaiduRegistration.png
[2]: ./media/notification-hubs-baidu-get-started/BaiduRegistrationInput.png
[3]: ./media/notification-hubs-baidu-get-started/BaiduConfirmation.png
[4]: ./media/notification-hubs-baidu-get-started/BaiduActivationEmail.png
[5]: ./media/notification-hubs-baidu-get-started/BaiduRegistrationMore.png
[6]: ./media/notification-hubs-baidu-get-started/BaiduOpenCloudPlatform.png
[7]: ./media/notification-hubs-baidu-get-started/BaiduDeveloperServices.png
[8]: ./media/notification-hubs-baidu-get-started/BaiduDeveloperRegistration.png
[9]: ./media/notification-hubs-baidu-get-started/BaiduDevRegistrationInput.png
[10]: ./media/notification-hubs-baidu-get-started/BaiduDevRegistrationConfirmation.png
[11]: ./media/notification-hubs-baidu-get-started/BaiduDevConfirmationFinal.png
[12]: ./media/notification-hubs-baidu-get-started/BaiduCloudPush.png
[13]: ./media/notification-hubs-baidu-get-started/BaiduDevSvcMgmt.png
[14]: ./media/notification-hubs-baidu-get-started/BaiduCreateProject.png
[15]: ./media/notification-hubs-baidu-get-started/BaiduCreateProjectInput.png
[16]: ./media/notification-hubs-baidu-get-started/BaiduProjectKeys.png
[17]: ./media/notification-hubs-baidu-get-started/AzureNHCreation.png
[18]: ./media/notification-hubs-baidu-get-started/NotificationHubs.png
[19]: ./media/notification-hubs-baidu-get-started/NotificationHubsConfigure.png
[20]: ./media/notification-hubs-baidu-get-started/NotificationHubBaiduConfigure.png
[21]: ./media/notification-hubs-baidu-get-started/NotificationHubsConnectionStringView.png
[22]: ./media/notification-hubs-baidu-get-started/NotificationHubsConnectionString.png
[23]: ./media/notification-hubs-baidu-get-started/EclipseNewProject.png
[24]: ./media/notification-hubs-baidu-get-started/EclipseProjectCreation.png
[25]: ./media/notification-hubs-baidu-get-started/EclipseBlankActivity.png
[26]: ./media/notification-hubs-baidu-get-started/EclipseProjectBuildProperty.png
[27]: ./media/notification-hubs-baidu-get-started/EclipseBaiduReferences.png
[28]: ./media/notification-hubs-baidu-get-started/EclipseNewClass.png
[29]: ./media/notification-hubs-baidu-get-started/EclipseConfigSettingsClass.png
[30]: ./media/notification-hubs-baidu-get-started/ConsoleProject.png
[31]: ./media/notification-hubs-baidu-get-started/BaiduPushConfig1.png
[32]: ./media/notification-hubs-baidu-get-started/BaiduPushConfig2.png
[33]: ./media/notification-hubs-baidu-get-started/BaiduPushConfig3.png

<!-- URLs. -->
[移动服务 Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[百度推送 Android SDK]: http://developer.baidu.com/wiki/index.php?title=docs/cplat/push/sdk/clientsdk
[Azure 经典管理门户]: https://manage.windowsazure.cn/
[百度门户]: http://www.baidu.com/

<!---HONumber=Mooncake_0523_2016-->