<properties linkid="develop-mobile-tutorials-get-started-with-push-xamarin-android" urlDisplayName="Get Started with Push Notifications" pageTitle="Get started with push notifications - Mobile Services" metaKeywords="" description="Learn how to use push notifications in Xamarin.Android apps with Azure Mobile Services." metaCanonical="" disqusComments="0" umbracoNaviHide="1" title="Get started with push notifications in Mobile Services" documentationCenter="Mobile" authors="" />

# 移动服务中的推送通知入门

<div class="dev-center-tutorial-selector sublanding">
<a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet" title="Windows Store C#">Windows 应用商店 C\#</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-js" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-ios" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-android" title="Android">Android</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-android" title="Xamarin.Android" class="current">Xamarin.Android</a></div>

本主题说明如何使用 Azure 移动服务向 Xamarin.Android 应用程序发送推送通知。在本教程中，你将要使用 Google Cloud Messaging (GCM) 服务向快速入门项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

本教程将指导你完成启用推送通知的以下基本步骤：

1.  [为推送通知注册应用程序][]
2.  [配置移动服务][]
3.  [向应用程序添加推送通知][]
4.  [更新脚本以发送推送通知][]
5.  [插入数据以接收通知][]

本教程需要的内容如下：

-   有效的 Google 帐户

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门][]。

<a name="register"></a>
## 注册应用程序为推送通知注册应用程序

<div class="dev-callout"><b>说明</b>

<p>若要完成本主题中的过程，你必须拥有一个包含已验证电子邮件地址的 Google 帐户。若要新建一个 Google 帐户，请转至 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268302" target="_blank">accounts.google.com</a>。</p>
</div>

1.  导航到 [Google API][] 网站，使用你的 Google 帐户凭据登录，然后单击“Create project...”（创建项目...） 。

    ![][]

    > [WACOM.NOTE]
    > 如果你已拥有现成项目，则在登录后你将定向到“仪表板” 页。若要从仪表板新建一个项目，请展开“API 项目” ，单击“其他项目” 下面的“创建...” ，然后输入项目名称并单击“创建项目” 。

2.  单击左栏中的“概述”按钮，记下“仪表板”部分中的项目编号。

    在教程的稍后部分中，你要将此值设置为客户端中的 "PROJECT\_ID" 变量。

3.  在[Google API][] 页面上，单击“服务” ，然后单击开关以启用 "Google Cloud Messaging for Android" 并接受服务条款。

4.  单击"“API 访问”"，然后单击“新建服务器密钥...” 

    ![][1]

5.  在“为 API 项目配置服务器密钥” 中，单击“创建” 。

    ![][2]

6.  记下“API 密钥” 值。

    ![][3]

接下来，你将使用此 API 密钥值，让移动服务向 GCM 进行身份验证并代表你的应用程序发送推送通知。

<a name="configure"></a>
## 配置服务配置移动服务以发送推送请求

1.  登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][4]

2.  单击“推送” 选项卡，输入你在执行前一过程时从 GCM 获取的“API 密钥” 值，然后单击“保存” 。

    ![][5]

    你的移动服务现已配置为使用 GCM 发送推送通知。

<a name="add-push"></a>
## 添加推送通知向应用程序添加推送通知

1.  首先，我们要将 "PushSharp" 添加为项目中的引用。为此，我们必须编译 PushSharp 的最新版本，并且将已编译的 DLL 作为对 Xamarin.Android 项目的引用添加。

2.  访问 [PushSharp Github 页][]，并下载最新版本。在提取了文件集合后，导航到以下示例项目文件夹：

    "/Client.Samples/PushSharp.ClientSample.MonoForAndroid/PushSharp.ClientSample.MonoForAndroid.Gcm/"

    .. 然后，打开项目文件：

    \*\* PushSharp.ClientSample.MonoForAndroid.Gcm.csproj \*\*

3.  在“版本” 模式下生成 MonoForAndroid PushSharp 客户端示例。

4.  在 Xamarin.Android 项目文件夹中创建 \*\*\_external\*\* 文件夹

5.  将 MonoForAndroid PushSharp 客户端示例中的以下文件复制到 Xamarin.Android 项目文件夹中新创建的 \*\*\_external\*\* 文件夹：

    "\\bin\\Release\\PushSharp.Client.MonoForAndroid.dll"

6.  在 Xamarin Studio（或者 Visual Studio）中打开你的 Xamarin.Android 项目。

7.  右键单击项目 "References" 文件夹，然后选择“编辑引用…” 

8.  转到“.Net 程序集” 选项卡，浏览到你的项目的 \*\*\_external\*\* 文件夹，选择我们之前生成的 "PushSharp.Client.MonoForAndroid.dll"，然后单击“添加” 。单击“确定”以关闭该对话框。

9.  打开 "Constants.cs" 并添加以下行，将 "PROJECT\_ID" 替换为你之前记下的 Google Project\_ID：

        public const string SenderID = "PROJECT_ID"; // Google API Project Number

10. 从 MonoForAndroid PushSharp 客户端示例将文件 "PushService.cs" 复制到 Xamarin.Android 项目文件夹，并将其添加到你的项目。

11. 更改 "PushService.cs" 中使用的命名空间，使之匹配你的项目的命名空间（例如：XamarinTodoQuickStart）。

12. 更改 "PushService.cs" 中的 "SENDER\_IDS" 数组，以引用我们上面创建的 "SenderID" 常量：

        public static string[] SENDER_IDS = new string[] { Constants.SenderID };

13. 将一个新的静态属性添加到 "PushService.cs" 中的 "PushHandlerService"，以跟踪设备注册 ID：

        public static string RegistrationID { get; private set; }

14. 更新 "PushService.cs" 中的 "OnRegistered" 方法，以将收到的注册 ID 存储到本地的静态变量：

        protected override void OnRegistered(Context context, string registrationId)
        {
        Log.Verbose(PushHandlerBroadcastReceiver.TAG, "GCM Registered:" + registrationId);
        RegistrationID = registrationId;
        }

15. 更新 "PushService.cs" 中的 "OnMessage" 方法，以显示作为通知的一部分收到的推送消息（替换现有的 "createNotification" 调用）：

        string message = intent.Extras.GetString("message");
        createNotification("New todo item!", "Todo item:" + message);

16. 请注意，"OnMessage" 方法在默认情况下具有以下代码，以保存收到的最后一条推送消息：

        //Store the message
        var prefs = GetSharedPreferences(context.PackageName, FileCreationMode.Private);
        var edit = prefs.Edit();
        edit.PutString("last_msg", msg.ToString());
        edit.Commit();

17. 更新 "PushService.cs" 中的 "createNotification" 方法，以引用 "TodoActivity" 而非 "DefaultActivity"。

18. 打开 "TodoActivity.cs" 并添加以下 using 语句：

        using PushSharp.Client;

19. 在 "TodoActivity.cs" 中，在创建 "MobileServiceClient" 的位置上面插入以下行：

        // Check to ensure everything's setup right
        PushClient.CheckDevice(this);
        PushClient.CheckManifest(this);

        // Register for push notifications
        System.Diagnostics.Debug.WriteLine("Registering...");
        PushClient.Register(this, PushHandlerBroadcastReceiver.SENDER_IDS);

20. 打开 "TodoItem.cs" 并添加一个新字段以跟踪添加 TodoItem 的人员的已注册设备 ID：

        [DataMember(Name = "channel")]
        public string RegistrationId { get; set; }

21. 在 "TodoActivity.cs" 中更新 "AddItem" 方法，以将新添加的 "TodoItem" 的 "RegistrationID" 设为在注册过程中收到的设备注册 ID：

        // Create a new item
        var item = new TodoItem() {
        Text = textNewTodo.Text,
        Complete = false,
        RegistrationId = PushHandlerService.RegistrationID
        };

你的应用程序现已更新，可支持推送通知。

<a name="update-scripts"></a>
## 更新插入脚本在管理门户中更新注册的插入脚本

1.  在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表 。

    ![][6]

2.  在“TodoItem” 中，单击“脚本” 选项卡，然后选择“插入” 。

    ![][7]

    将显示当 "TodoItem" 表中发生插入时所调用的函数。

3.  将 insert 函数替换为以下代码，然后单击“保存”： 

        function insert(item, user, request) {
        request.execute({
        success:function() {
        // Write to the response and then send the notification in the background
        request.respond();
        push.gcm.send(item.channel, item.text, {
        success:function(response) {
        console.log('Push notification sent:', response);
        }, error:function(error) {
        console.log('Error sending push notification:', error);
                        }
                    });
                }
            });
        }

这将会注册一个新的插入脚本，该脚本使用 [GCM 对象][]将推送通知（插入的文本）发送到插入请求中提供的设备。

<a name="test"></a>
## 测试应用程序在应用程序中测试推送通知

1.  运行应用程序并添加一个新的 Todo 项目。确保你收到有关所添加的新 Todo 项目的推送通知。

2.  在 Azure 管理门户中复查你的移动应用程序的“日志” 选项卡，以查看我们添加到 "Insert" 方法的日志记录消息是否在上面的 "TodoItem" 表中。

3.  查看 Azure 管理门户中的 "TodoItem" 表，以了解新的“通道” 列是否已添加并包含唯一的设备注册标识符。

你已成功完成本教程。

## 获取已完成的示例

下载[已完成的示例项目][]。请务必使用你自己的 Azure 设置更新 "ApplicationURL"、"ApplicationKey" 和 "SenderID" 变量。

<a name="next-steps"> </a>
## 后续步骤

在这个简单的示例中，用户将会收到包含刚刚插入的数据的推送通知。在下一教程[向应用程序用户推送通知][]中，你将要创建一个单独的 Devices 表，该表用于存储设备标记，以及在发生插入操作时向所有存储的通道发出推送通知。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/get-started-with-push-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/get-started-with-push-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-push-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/get-started-with-push-android "Android"
  [Xamarin.iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-android "Xamarin.Android"
  [为推送通知注册应用程序]: #register
  [配置移动服务]: #configure
  [向应用程序添加推送通知]: #add-push
  [更新脚本以发送推送通知]: #update-scripts
  [插入数据以接收通知]: #test
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started-xamarin-android
  [accounts.google.com]: http://go.microsoft.com/fwlink/p/?LinkId=268302
  [Google API]: http://go.microsoft.com/fwlink/p/?LinkId=268303
  [0]: ./media/partner-xamarin-mobile-services-android-get-started-push/mobile-services-google-developers.png
  [1]: ./media/partner-xamarin-mobile-services-android-get-started-push/mobile-services-google-create-server.png
  [2]: ./media/partner-xamarin-mobile-services-android-get-started-push/mobile-services-google-create-server2.png
  [3]: ./media/partner-xamarin-mobile-services-android-get-started-push/mobile-services-google-create-server3.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [4]: ./media/partner-xamarin-mobile-services-android-get-started-push/mobile-services-selection.png
  [5]: ./media/partner-xamarin-mobile-services-android-get-started-push/mobile-push-tab-android.png
  [PushSharp Github 页]: https://github.com/Redth/PushSharp
  [6]: ./media/partner-xamarin-mobile-services-android-get-started-push/mobile-portal-data-tables.png
  [7]: ./media/partner-xamarin-mobile-services-android-get-started-push/mobile-insert-script-push2.png
  [GCM 对象]: http://go.microsoft.com/fwlink/p/?LinkId=282645
  [已完成的示例项目]: http://go.microsoft.com/fwlink/p/?LinkId=331303
  [向应用程序用户推送通知]: /zh-cn/develop/mobile/tutorials/push-notifications-to-users-android
