<properties linkid="develop-mobile-tutorials-get-started-with-push-android" urlDisplayName="Get Started with Push (Android)" pageTitle="Get started with push notifications (Android) | Mobile Dev Center" metaKeywords="" description="Learn how to use Azure Mobile Services to send push notifications to your Android app." metaCanonical="" services="" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="ricksal" solutions="" manager="dwrede" editor="" />

# 移动服务中的推送通知入门

<div class="dev-center-tutorial-selector sublanding">
<a href="/en-us/develop/mobile/tutorials/get-started-with-push-dotnet" title="Windows 应用商店 C#">Windows 应用商店 C#</a><a href="/en-us/develop/mobile/tutorials/get-started-with-push-js" title="Windows 应用商店 JavaScript">Windows 应用商店 JavaScript</a><a href="/en-us/develop/mobile/tutorials/get-started-with-push-wp8" title="Windows Phone">Windows Phone</a><a href="/en-us/develop/mobile/tutorials/get-started-with-push-ios" title="iOS">iOS</a><a href="/en-us/develop/mobile/tutorials/get-started-with-push-android" title="Android" class="current">Android</a><a href="/en-us/develop/mobile/tutorials/get-started-with-push-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a><a href="/en-us/develop/mobile/tutorials/get-started-with-push-xamarin-android" title="Xamarin.Android" class="current">Xamarin.Android</a>
</div>

本主题说明如何使用 Azure 移动服务向 Android 应用程序发送推送通知。在本教程中，你将要使用 Google Cloud Messaging (GCM) 服务向快速入门项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

> [WACOM.NOTE] 移动服务现在将与 Azure 通知中心集成，以支持附加的推送通知功能，如模板、多个平台和规模。此集成的功能目前处于预览状态。有关详细信息，请参阅此版本的[推送通知入门][推送通知入门]。

本教程将指导你完成启用推送通知的以下基本步骤：

-   [启用 Google Cloud Messaging][启用 Google Cloud Messaging]
-   [配置移动服务][配置移动服务]
-   [向应用程序添加推送通知][向应用程序添加推送通知]
-   [更新脚本以发送推送通知][更新脚本以发送推送通知]
-   [插入数据以接收通知][插入数据以接收通知]

本教程需要的内容如下：

-   [移动服务 Android SDK][移动服务 Android SDK]
-   有效的 Google 帐户

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门][移动服务入门]。

## <span id="register"></span></a>启用 Google Cloud Messaging

[WACOM.INCLUDE [启用 GCM][启用 GCM]]

接下来，你将使用此 API 密钥值，让移动服务向 GCM 进行身份验证并代表你的应用程序发送推送通知。

## <span id="configure"></span></a>配置移动服务以发送推送请求

1.  登录到 [Azure 管理门户][Azure 管理门户]，单击“移动服务”，然后单击你的应用程序。

    ![][]

2.  单击“推送”选项卡，输入你在执行前一过程时从 GCM 获取的“API 密钥”值，然后单击“保存”。

    ![][1]

你的移动服务现已配置为使用 GCM 发送推送通知。

## <span id="add-push"></span></a>向应用程序添加推送通知

### 将 Google Play Services 添加到项目

[WACOM.INCLUDE [添加 Play Services][添加 Play Services]]

### 添加代码

1.  打开项目文件 **AndroidManifest.xml**。Google Cloud Messaging 对开发和测试提出了一些最低的 API 级别要求，清单中的 **minSdkVersion** 属性必须符合这些要求。如果由于使用的设备较旧而需要将此属性设置为小于 16，请参阅[设置 Google Play Services SDK][设置 Google Play Services SDK] 以确定它可以设置到的最小值，并相应地进行设置。

2.  确保在 `uses-sdk` 元素中，**targetSdkVersion** 已设置为安装的 SDK 平台的版本号（与步骤 1 中所述的版本相同）。最好是将它设置为可用的最新版本。

3.  **uses-sdk** 标记可能如下所示，具体取决于你在前面步骤中所做的选择：

        <uses-sdk
            android:minSdkVersion="17"
            android:targetSdkVersion="19" />

4.  在随后两个步骤中，请将代码中的 *`**my_app_package**`* 替换为你的项目的应用程序包名称，即 manifest 标记的 package 属性值。

5.  在现有 `uses-permission` 元素之后添加以下新权限：

        <permission android:name="**my_app_package**.permission.C2D_MESSAGE" 
            android:protectionLevel="signature" />
        <uses-permission android:name="**my_app_package**.permission.C2D_MESSAGE" /> 
        <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
        <uses-permission android:name="android.permission.GET_ACCOUNTS" />
        <uses-permission android:name="android.permission.WAKE_LOCK" />

6.  将以下代码添加到 `application` 开始标记的后面：

        <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
            android:permission="com.google.android.c2dm.permission.SEND">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="**my_app_package**" />
            </intent-filter>
        </receiver>

7.  打开 ToDoItem.java 文件，并将以下代码添加到 **TodoItem** 类：

            @com.google.gson.annotations.SerializedName("handle")
            private String mHandle;

            public String getHandle() {
                return mHandle;
            }

            public final void setHandle(String handle) {
                mHandle = handle;
            }

    此代码将创建一个用于保存注册 ID 的新属性。

    <div class="dev-callout"><b>说明</b>
<p>如果在移动服务中启用了动态架构，则插入包含此属性的新项时，将在 <strong>TodoItem</strong> 表中自动添加新的 <strong>handle</strong> 列。</p>
</div>

8.  下载和解压缩[移动服务 Android SDK][移动服务 Android SDK]，打开 **notifications** 文件夹，将 **notifications-1.0.1.jar** 文件复制到 Eclipse 项目的 *libs* 文件夹，并刷新 *libs* 文件夹。

    <div class="dev-callout"><b>说明</b>
<p>在后续的 SDK 版本中，文件名末尾的数字可能更改。</p>
</div>

9.  打开文件 *ToDoItemActivity.java*，并添加以下 import 语句：

        import com.microsoft.windowsazure.notifications.NotificationsManager;

10. 将以下私有变量添加到类，其中，*`<PROJECT_NUMBER>`* 是 Google 在前一个过程中为你的应用程序分配的项目编号：

        public static final String SENDER_ID = "<PROJECT_NUMBER>";

11. 在 **onCreate** 方法中，在 MobileServiceClient 实例化之前，添加以下代码以注册设备的通知处理程序：

        NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);

12. 在 **addItem** 方法中要插入到表内的项之前添加以下代码行：

        item.setHandle(MyHandler.getHandle());

    此代码会将该项的 `handle` 属性设置为设备的注册 ID。

13. 在包资源管理器中，右键单击包（在 `src` 节点下面），单击“新建”，然后单击“类”。

14. 在“名称”中，键入`MyHandler`，在“超类”中，键入`com.microsoft.windowsazure.notifications.NotificationsHandler`，然后单击“完成”。

    ![][2]

    这样可创建新的 MyHandler 类。

15. 添加以下 import 语句：

        import android.content.Context;

        import com.microsoft.windowsazure.notifications.NotificationsHandler;

16. 添加以下代码：

        @com.google.gson.annotations.SerializedName("handle")
        private static String mHandle;

        public static String getHandle() {
            return mHandle;
        }

        public static final void setHandle(String handle) {
            mHandle = handle;
        }

17. 将现有的 **onRegistered** 方法重写替换为以下代码：

        @Override
        public void onRegistered(Context context, String gcmRegistrationId) {
            super.onRegistered(context, gcmRegistrationId);

            setHandle(gcmRegistrationId);
        }

你的应用程序现已更新，可支持推送通知。

## <span id="update-scripts"></span></a>在管理门户中更新已注册的插入脚本

1.  在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表。

    ![][3]

2.  在“todoitem”中，单击“脚本”选项卡，然后选择“插入”。

    ![][4]

    将显示当 **TodoItem** 表中发生插入时所调用的函数。

3.  将 insert 函数替换为以下代码，然后单击“保存”：

        function insert(item, user, request) {
            request.execute({
                success: function() {
                    // Write to the response and then send the notification in the background
                    request.respond();
                    push.gcm.send(item.handle, item.text, {
                        success: function(response) {
                            console.log('Push notification sent: ', response);
                        }, error: function(error) {
                            console.log('Error sending push notification: ', error);
                        }
                    });
                }
            });
        }

    这将会注册一个新的插入脚本，该脚本使用 [GCM 对象][GCM 对象]将推送通知（插入的文本）发送到插入请求中提供的设备。

## <span id="test"></span></a>在应用程序中测试推送通知

<div class="dev-callout"><b>说明</b>
<p>当你在模拟器中运行此应用程序时，请确保使用支持 Google API 的 Android 虚拟设备 (AVD)。</p>
</div>

1.  重新启动 Eclipse，在包资源管理器中，右键单击项目，单击“属性”，单击“Android”，选中“Google API”，然后单击“确定”。

    ![][5]

    这样便指定了项目要使用 Google API。

2.  从“窗口”中选择“Android 虚拟设备管理器”，选择你的设备，然后单击“编辑”。

    ![][6]

3.  在“目标”中选择“Google API”，然后单击“确定”。

    ![][7]

    这样便指定了 AVD 要使用 Google API。

4.  在“运行”菜单中，单击“运行”以启动应用程序。

5.  在应用程序中键入有意义的文本（例如 *A new Mobile Services task*），然后单击“添加”按钮。

    ![][8]

6.  屏幕下方将短暂出现一个黑色的通知框。

    ![][9]

<!--7. Tap on the icon and swipe down to display the notification, which appears in the graphic below.    ![][27]-->

你已成功完成本教程。

## <a name="next-steps"></a>后续步骤

在这个简单的示例中，用户将会收到包含刚刚插入的数据的推送通知。请求中的客户端会将 GCM 使用的设备标记提供给移动服务。在下一教程[向应用程序用户推送通知][向应用程序用户推送通知]中，你将要创建一个单独的 Devices 表，该表用于存储设备标记，以及在发生插入操作时向所有存储的通道发出推送通知。

<!-- URLs. -->

  [Windows 应用商店 C\#]: /en-us/develop/mobile/tutorials/get-started-with-push-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /en-us/develop/mobile/tutorials/get-started-with-push-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /en-us/develop/mobile/tutorials/get-started-with-push-wp8 "Windows Phone"
  [iOS]: /en-us/develop/mobile/tutorials/get-started-with-push-ios "iOS"
  [Android]: /en-us/develop/mobile/tutorials/get-started-with-push-android "Android"
  [Xamarin.iOS]: /en-us/develop/mobile/tutorials/get-started-with-push-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /en-us/develop/mobile/tutorials/get-started-with-push-xamarin-android "Xamarin.Android"
  [推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-android-get-started-push/
  [启用 Google Cloud Messaging]: #register
  [配置移动服务]: #configure
  [向应用程序添加推送通知]: #add-push
  [更新脚本以发送推送通知]: #update-scripts
  [插入数据以接收通知]: #test
  [移动服务 Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started-android
  [启用 GCM]: ../includes/mobile-services-enable-Google-cloud-messaging.md
  [Azure 管理门户]: https://manage.windowsazure.cn/

<!-- Images. --> 
  []: ./media/mobile-services-android-get-started-push/mobile-services-selection.png
  [1]: ./media/mobile-services-android-get-started-push/mobile-push-tab-android.png
  [添加 Play Services]: ../includes/mobile-services-add-Google-play-services.md
  [设置 Google Play Services SDK]: http://go.microsoft.com/fwlink/?LinkId=389801
  [2]: ./media/mobile-services-android-get-started-push/mobile-services-android-create-class.png
  [3]: ./media/mobile-services-android-get-started-push/mobile-portal-data-tables.png
  [4]: ./media/mobile-services-android-get-started-push/mobile-insert-script-push2.png
  [GCM 对象]: http://go.microsoft.com/fwlink/p/?LinkId=282645
  [5]: ./media/mobile-services-android-get-started-push/mobile-services-import-android-properties.png
  [6]: ./media/mobile-services-android-get-started-push/mobile-services-android-virtual-device-manager.png
  [7]: ./media/mobile-services-android-get-started-push/mobile-services-android-virtual-device-manager-edit.png
  [8]: ./media/mobile-services-android-get-started-push/mobile-quickstart-push1-android.png
  [9]: ./media/mobile-services-android-get-started-push/mobile-push-icon.png
  [向应用程序用户推送通知]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-android
