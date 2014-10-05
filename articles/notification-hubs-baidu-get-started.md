<properties linkid="develop-notificationhubs-tutorials-get-started-baidu" urlDisplayName="Get Started" pageTitle="Get Started with Azure Notification Hubs" metaKeywords="" description="Learn how to use Azure Notification Hubs to push notifications." metaCanonical="" services="notification-hubs" documentationCenter="Mobile" title="Get started with Notification Hubs" authors="piyushjo" solutions="" manager="kirillg" editor="" />

# 通知中心入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/" title="Windows 应用商店 C#">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/notification-hubs-windows-phone-get-started/" title="Windows Phone">Windows Phone</a><a href="/zh-cn/documentation/articles/notification-hubs-ios-get-started/" title="iOS">iOS</a><a href="/zh-cn/documentation/articles/notification-hubs-android-get-started/" title="Android">Android</a><a href="/zh-cn/documentation/articles/notification-hubs-kindle-get-started/" title="Kindle">Kindle</a><a href="/zh-cn/documentation/articles/notification-hubs-baidu-get-started/" title="百度" class="current">百度</a><a href="/zn-cn/documentation/articles/partner-xamarin-notification-hubs-ios-get-started/" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zn-cn/documentation/articles/partner-xamarin-notification-hubs-android-get-started/" title="Xamarin.Android">Xamarin.Android</a></div>

百度云推送是一种中国云服务，可用于将推送通知发送到移动设备。在中国，将推送通知传递到 Android 的过程很复杂，因为存在不同的应用程序商店和推送服务，还有通常未连接到 GCM（Google 云消息传送）的 Android 设备，所以，此服务在该国特别有用。

本教程需要的内容如下：

+ Android SDK（假定你将使用 Eclipse），你可以从<a href="http://go.microsoft.com/fwlink/?LinkId=389797">此处</a>下载该 SDK
+ [移动服务 Android SDK][Mobile Services Android SDK]
+ [百度推送 Android SDK][Baidu Push Android SDK]

> [WACOM.NOTE] 若要完成本教程，你必须具有有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.com/en-us/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started%2F" target="_blank">Azure 免费试用</a>。

本教程将指导你完成启用推送通知的以下基本步骤：

* [创建百度帐户](#createBaiduAccount)
* [注册为百度开发者](#registerBaiduDeveloper)
* [创建百度云推送项目](#createBaiduPushProject)
* [配置通知中心](#configure-hub)
* [将你的应用程序连接到通知中心](#connecting-app)
*  [向应用程序发送通知](#send)

## <a id="createBaiduAccount"></a>创建百度帐户

若要使用百度，必须创建帐户。如果你已有帐户，请使用百度帐户登录到[百度门户][Baidu portal]，并跳到下一个步骤；否则，请参阅下面有关如何创建新百度帐号的说明。

1.  转到[百度门户][Baidu portal]并单击“登录”链接。单击“立即注册”以启动新帐户注册过程。

   	![][1]

2.  输入所需的详细信息（电话/电子邮件地址、密码和验证码），然后单击“注册”。

   	![][2]

3.  系统会将一封电子邮件发送到你输入的电子邮件地址，该邮件包含一个用于激活你的百度帐号的链接。

   	![][3]

4.  登录到你的电子邮件帐户，打开百度激活邮件，然后单击激活链接以激活你的百度帐号。

   	![][4]

当你有已激活的百度帐户后，请使用你的帐户登录到[百度门户][Baidu portal]。

## <span id="registerBaiduDeveloper"></span></a>注册为百度开发者

1.  登录到[百度门户][Baidu portal]后，请单击“更多\>\>”。

  	![][5]

2.  向下滚动“站长与开发者服务”部分并单击“百度开放云平台”。

  	![][6]

3.  在下一页上，单击右上角的“开发者服务”。

  	![][7]

4.  在下一页上，单击右上角菜单中的“注册开发者”。

  	![][8]

5.  输入你的名称、说明以及用于接收验证短信的手机号码，然后单击“发送验证码”。请注意，对于国际电话号码，你需要将国家/地区代码括在括号内，例如对于美国号码，电话号码将为 **(1)1234567890**。

  	![][9]

6.  然后，你应收到一条包含验证码的短信，如以下示例所示：

  	![][10] 

7.  在“验证码”中输入消息中的验证码。

8.  最后，接受百度协议并单击“提交”以完成开发者注册。成功完成注册时，你会看到以下页：

  	![][11] 

## <a id="createBaiduPushProject"></a>创建百度云推送项目

在创建百度云推送项目时，你将收到应用程序 ID、API 密钥和密钥。

1.  登录到[百度门户][Baidu portal]后，请单击“更多\>\>”。

  	![][5]

2.  向下滚动“站长与开发者服务”部分并单击“百度开放云平台”。

  	![][6]

3.  在下一页上，单击右上角的“开发者服务”。

  	![][7]

4.  在下一页上，单击“云服务”部分中的“云推送”。

  	![][12]

5.  注册开发者后，你将在顶部菜单上看到“管理控制台”。单击“开发者服务管理”。

  	![][13]

6.  在下一页上，单击“创建工程”。

  	![][14]

7.  输入应用程序名称，并单击“创建”。

  	![][15]

8.  创建成功后，将显示一个页面，其中包含 **AppID**、“API 密钥”和“密钥”。请记下 **API 密钥**和**密钥**，我们将在以后使用它们。

  	![][16]

## <a id="configure-hub"></a>配置通知中心

1.  登录到 [Azure 管理门户][Azure Management Portal]，然后单击屏幕底部的“+新建”。

2.  依次单击“应用服务”、“Service Bus”、“通知中心”和“快速创建”。

3.  为你的**通知中心**提供名称，选择要在其中创建此通知中心的**区域**和**命名空间**，然后单击“创建新的通知中心”

  	![][17]

4.  单击已在其中创建通知中心的命名空间，然后单击顶部的“通知中心”。

  	![][18]

5.  选择所创建的通知中心，然后单击顶部菜单中的“配置”。

  	![][19]

6.  向下滚动到“百度通知设置”部分，然后输入先前从百度控制台获得的百度云推送项目的 **API 密钥**和**密钥**。在输入这些值后，单击“保存”。

  	![][20]

7.  单击通知中心顶部的“仪表板”选项卡，然后单击“查看连接字符串”。

  	![][21]

8.  记下“访问连接信息”窗口中的 **DefaultListenSharedAccessSignature** 和 **DefaultFullSharedAccessSignature**。

    ![][22]

## <a id="connecting-app"></a>将你的应用程序连接到通知中心

1.  在 Eclipse ADT 中，创建新的 Android 项目（“文件”-\>“新建”-\>“Android 应用程序”）。

    ![][23]

2.  输入**应用程序名称**，并确保将**要求的最低 SDK** 版本设为 **API 16:Android 4.1**。

    ![][24]

3.  单击“下一步”，然后继续执行向导，直到显示“创建活动”窗口。确保选中了“空白活动”，最后选择“完成”以创建新的 Android 应用程序。

    ![][25]

4.  确保“项目生成目标”已正确设置。

    ![][26]

5.  下载并解压缩[移动服务 Android SDK][Mobile Services Android SDK]，打开 **notificationhubs** 文件夹，将 **notification-hubs-x.y.jar** 文件复制到 Eclipse 项目的 *libs* 文件夹，并刷新 *libs* 文件夹。

6.  下载并解压缩[百度推送 Android SDK][Baidu Push Android SDK]，打开 **libs** 文件夹，将 *pushservice-x.y.z* jar 文件以及 *armeabi* 和 *mips* 文件夹复制到 Android 应用程序的 **libs** 文件夹。

    ![][27]

7.  打开 Android 项目的 **AndroidManifest.xml**，并添加百度 SDK 所需的权限。

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

8.  向 **AndroidManifest.xml** 中的 *application* 元素添加 *android:name* 属性，并替换 *yourprojectname*（例如 **com.example.BaiduTest**）：

        <application android:name="yourprojectname.DemoApplication"

9.  在 .MainActivity 活动元素后的 application 元素内添加以下配置，并替换 *yourprojectname*（例如 **com.example.BaiduTest**）：

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

10. 将一个名为 **ConfigurationSettings.java** 的新类添加到项目中。

    ![][28]

    ![][29]

11. 将以下代码添加到该类中：

        public class ConfigurationSettings {
                public static String API_KEY = "...";
                public static String NotificationHubName = "...";
                public static String NotificationHubConnectionString = "...";
            }

    使用先前从百度云项目中检索到的内容设置 *API\_KEY* 的值，使用 Azure 门户中的通知中心名称设置 *NotificationHubName*，并使用 Azure 门户中的 DefaultListenSharedAccessSignature 设置 *NotificationHubConnectionString*。

12. 添加一个名为 **DemoApplication.java** 的新类，并向此类中添加以下代码：

        import com.baidu.frontia.FrontiaApplication;

        public class DemoApplication extends FrontiaApplication {
            @Override
            public void onCreate() {
                super.onCreate();
            }
        }

13. 添加另一个名为 **MyPushMessageReceiver.java** 的新类，并向此类中添加以下代码。此类用于处理从百度推送服务器收到的推送通知：

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
                String notifyString = "title=\"" + title + "\" description=\""
                        + description + "\" customContent=" + customContentString;
                Log.d(TAG, notifyString);
            }

            @Override
            public void onMessage(Context context, String message,
                    String customContentString) {
                String messageString = "message=\"" + message + "\" customContentString=" + customContentString;
                Log.d(TAG, messageString);
            }
        }

14. 打开 **MainActivity.java**，并将以下内容添加到 **onCreate** 方法中：

        PushManager.startWork(getApplicationContext(),
                PushConstants.LOGIN_TYPE_API_KEY, ConfigurationSettings.API_KEY);

并在顶部添加以下 import 语句：
import com.baidu.android.pushservice.PushConstants;
 import com.baidu.android.pushservice.PushManager;

##<a id="send"></a>向应用程序发送通知


你可以使用通知中心从任何使用 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/dn223264.aspx">REST 接口</a>的后端发送通知。在本教程中，我们使用 .NET 控制台应用程序来对其进行说明。

1.  创建新的 Visual C# 控制台应用程序：

	![][30]

2.  使用 <a href="http://nuget.org/packages/WindowsAzure.ServiceBus/">WindowsAzure.ServiceBus NuGet 包</a>添加对 Azure Service Bus SDK 的引用。在 Visual Studio 主菜单中，依次单击“工具”、“库程序包管理器”和“程序包管理器控制台”。然后，在控制台窗口中键入以下内容并按 Enter：

        Install-Package WindowsAzure.ServiceBus

3.  打开文件 Program.cs 并添加以下 using 语句：

        using Microsoft.ServiceBus.Notifications;

4.  在 `Program` 类中添加以下方法，并将 *DefaultFullSharedAccessSignatureSASConnectionString* 和 *NotificationHubName* 替换为你的值。

        private static async void SendNotificationAsync()
        {
            NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("DefaultFullSharedAccessSignatureSASConnectionString", "NotificationHubName");
            await hub.SendBaiduNativeNotificationAsync("{\"data\" : {\"payload\":\"" + "Hello from Azure" + "\"}}");
        }

5.  然后在 Main 方法中添加下列行：

         SendNotificationAsync();
         Console.ReadLine();

## <a name="test-app"></a>测试应用程序

若要使用实际的手机测试此应用程序，只需使用 USB 电缆将该手机连接到计算机。

若要使用模拟器测试此应用程序，请执行以下操作：

1.  在 Eclipse 顶部工具栏中，单击“运行”，然后选择你的应用程序。

2.  这将在连接的手机上加载你的应用程序，或者它启动模拟器，加载并运行该应用程序。

3.  此应用程序从百度推送通知服务检索“userId”和“channelId”，并注册到通知中心。

4.  若要在使用 .Net 控制台应用程序时发送测试通知，只需在 Visual Studio 中按 F5 键以运行此应用程序，此应用程序将发送一则通知，该通知将显示在你的设备或模拟器的顶部通知区域中。

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

<!-- URLs. -->
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[Baidu Push Android SDK]: http://developer.baidu.com/wiki/index.php?title=docs/cplat/push/sdk/clientsdk
[Azure Management Portal]: https://manage.windowsazure.com/
[Baidu portal]: http://www.baidu.com/



