<properties
	pageTitle="向移动服务应用程序添加推送通知 (Xamarin.iOS) - 移动服务"
	description="了解如何借助 Azure 移动服务在 Xamarin.iOS 应用程序中使用推送通知。"
	documentationCenter="xamarin"
	authors="ysxu"
	manager="dwrede"
	services="mobile-services"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="03/18/2016"
	wacn.date="05/23/2016"/>

#  向移动服务应用程序添加推送通知

[AZURE.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

## 概述

本主题说明如何使用 Azure 移动服务向 Xamarin.iOS 8 应用程序发送推送通知。在本教程中，你将要使用 Apple 推送通知服务 (APNS) 向[移动服务入门]项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

本教程需要的内容如下：

+ IOS 8 设备（无法在 iOS 模拟器中测试推送通知）
+ iOS 开发人员计划成员身份
+ [Xamarin Studio]
+ [Azure 移动服务组件]

>[AZURE.IMPORTANT]根据 APNS 的要求，你必须在支持 iOS 的设备（iPhone 或 iPad）而不是在模拟器上部署和测试推送通知。

APNS 使用证书对你的移动服务进行身份验证。按照以下说明创建必要的证书并将其上载到你的移动服务。有关正式的 APNS 功能文档，请参阅 [Apple 推送通知服务]。

##  <a name="certificates"></a>生成证书签名请求文件

首先，你必须生成证书签名请求 (CSR) 文件，Apple 将使用该文件生成签名证书。

1. 从“Utilities”（实用工具）中，运行 **Keychain Access** 工具。

2. 单击“Keychain Access”，展开“Certificate Assistant”（证书助理），然后单击“Request a Certificate from a Certificate Authority...”（从证书颁发机构请求证书...）。

  	![][5]

3. 输入你的“User Email Address”（用户电子邮件地址），键入“Common Name”（公用名）值，确保已选择“Saved to disk”（保存到磁盘），然后单击“Continue”（继续）。

  	![][6]

4. 在“Save As”（另存为）中为证书签名请求 (CSR) 文件键入一个名称，在“Where”（位置）中选择一个位置，然后单击“Save”（保存）。

  	![][7]
  
    请记住你选择的位置。

接下来，你将要向 Apple 注册你的应用程序、启用推送通知并上载这个导出的 CSR 以创建一个推送证书。

##  <a name="register"></a>为推送通知注册应用程序

若要将推送通知从移动服务发送到 iOS 应用程序，你必须向 Apple 注册应用程序，并注册推送通知。

1. 如果你尚未注册应用程序，请导航到 Apple 开发人员中心的 <a href="http://go.microsoft.com/fwlink/p/?LinkId=272456" target="_blank">iOS 设置门户</a>，使用 Apple ID 登录，单击“Identifiers”（标识符），然后单击“App IDs”（应用程序 ID），最后单击“+”符号创建应用程序的应用程序 ID。

   	![][102]

2. 在“Description”（说明）中为应用程序键入一个名称，记住唯一的“Bundle Identifier”（捆绑标识符），在“App Services”（应用程序服务）部分中选中“Push Notifications”（推送通知）选项，然后单击“Continue”（继续）。此示例将使用 ID **MobileServices.Quickstart**，但你不可以重用这个 ID，因为应用程序 ID 在所有用户之间必须唯一。因此，建议在应用程序名称的后面附加完整名称或首字母。

	![][103]
   
	此时将会生成你的应用程序 ID 并请求你**提交**该信息。单击“提交”。
   
	![][104]
   
   	单击“Submit”（提交）后，你将会看到如下所示的“Registration complete”（注册已完成）屏幕。单击“Done”（完成）。
   
	![][105]

3. 找到你刚刚创建的应用程序 ID，然后单击其行。

   	![][106]
   
   	单击应用程序 ID 会显示有关应用程序和应用程序 ID 的详细信息。单击“设置”按钮。
   
   	![][107]
   
4. 滚动到屏幕底部并单击“Development Push SSL Certificate”（开发推送 SSL 证书 ）部分下的“Create Certificate...”（创建证书...）按钮。

   	![][108]

   	将显示“Add iOS Certificate”（添加 iOS 证书）助手。
   
	注意：本教程使用开发证书。注册生产证书时使用相同的过程。将证书上载至移动服务时，只需确保设置了相同的证书类型即可。

5. 单击“Choose File”（选择文件），浏览到前面保存 CSR 文件的位置，然后单击“Generate”（生成）。

  	![][110]
  
6. 门户创建证书之后，请单击“Download”（下载）按钮，然后单击“Done”（完成）。
 
  	![][111]

   	随后将会下载签名证书并将其保存到计算机上的 Downloads 文件夹。

  	![][9]

    注意：默认情况下，下载的文件（开发证书）名为 <strong>aps\_development.cer</strong>。

7. 双击下载的推送证书 **aps\_development.cer**。

   	将在 Keychain 中安装新证书，如下所示：

   	![][10]

	注意：证书中的名称可能不同，但将以 <strong>Apple Development iOS Push Notification Services:</strong> 作为前缀。

稍后，你将要使用此证书生成一个 .p12 文件，并将其上载到移动服务以使用 APNS 启用身份验证。

##  <a name="profile"></a>为应用程序创建配置文件
 
1. 返回 <a href="http://go.microsoft.com/fwlink/p/?LinkId=272456" target="_blank">iOS 设置门户</a>，选择“Provisioning Profiles”（设置配置文件），选择“All”（全部），然后单击“+”按钮创建一个新的配置文件。此时会启动“Add iOS Provisiong Profile”（添加 iOS 设置配置文件）向导。

   	![][112]

2. 选择“Development”（开发）下的“iOS App Development”（iOS 应用程序开发）作为设置配置文件类型，然后单击“Continue”（继续）。

3. 接下来，从“App ID”（应用程序 ID）下拉列表中选择移动服务快速入门应用程序的应用程序 ID，然后单击“Continue”（继续）。

   	![][113]

4. 在“Select certificates”（选择证书）屏幕中，选择前面创建的证书，然后单击“Continue”（继续）。

   	![][114]

5. 接下来，选择要用于测试的“Devices”（设备），然后单击“Continue”（继续）。
  
   	![][115]

6. 最后，在“Profile Name”（配置文件名称）中为配置文件选取一个名称，单击“Generate”（生成），然后单击“Done”（完成）。

   	![][116]

	此操作可创建新的配置文件。

	![][117]

##  <a name="configure-mobileServices"></a>配置移动服务以发送推送请求

将应用注册到 APNS 并配置项目后，接下来必须配置移动服务以便与 APNS 集成。

1. 在 Keychain Access 中，右键单击新证书，单击“Export”（导出），为文件命名，选择 **.p12** 格式，然后单击“Save”（保存）。

   	![][28]

  	记下文件名和导出的证书的位置。

2. 登录到 [Azure 管理门户]，单击“移动服务”，然后单击你的应用。

   	![][18]

3. 单击“推送”选项卡，单击“Apple 推送通知设置”下的“上载”。

   	![][19]

   	此时将显示“上载证书”对话框。

4. 单击“文件”，选择导出的 .p12 证书文件，输入密码，确保已选择正确的“模式”，单击勾选图标，然后单击“保存”。

   	![][20]

现在，你的移动服务已配置为使用 APNS。

##  <a name="configure-app"></a>配置 Xamarin.iOS 应用程序

1. 在 Xamarin.Studio 中，打开 **Info.plist**，然后使用前面创建的 ID 更新“捆绑标识符”。

    ![][121]

2. 向下滚动到“Background Modes”（后台模式）并选中“Enable Background Modes”（启用后台模式）框和“Remote notifications”（远程通知）框。

    ![][122]

3. 在解决方案面板中双击你的项目以打开“Project Options”（项目选项）。

4.  在“Build”（生成）下面选择“iOS Bundle Signing”（iOS 捆绑签名），并选择你刚刚为此项目设置的“Identity”（标识）和“Provisioning profile”（设置配置文件）。

    ![][120]

    这可以确保 Xamarin 项目使用新配置文件进行代码签名。有关正式的 Xamarin 设备设置文档，请参阅 [Xamarin 设备设置]。

##  <a name="add-push"></a>向应用程序添加推送通知

1. 在 Xamarin.Studio 中，打开 AppDelegate.cs 文件，并添加以下属性：

        public string DeviceToken { get; set; }

2. 打开 **TodoItem** 类，并添加以下属性：

        [JsonProperty(PropertyName = "deviceToken")]
        public string DeviceToken { get; set; }

3. 在 **QSTodoService** 中，将现有的客户端声明重写为：

        public MobileServiceClient client { get; private set; }

4. 然后添加以下方法，以便 **AppDelegate** 稍后可以获取客户端来注册推送通知：

        public MobileServiceClient GetClient {
            get{
                return client;
            }
        }

5. 在 **AppDelegate** 中，重写 **FinishedLaunching** 事件：

        public override bool FinishedLaunching(UIApplication application, NSDictionary launchOptions)
        {
            // registers for push for iOS8
            var settings = UIUserNotificationSettings.GetSettingsForTypes(
                UIUserNotificationType.Alert
                | UIUserNotificationType.Badge
                | UIUserNotificationType.Sound,
                new NSSet());

            UIApplication.SharedApplication.RegisterUserNotificationSettings(settings);
            UIApplication.SharedApplication.RegisterForRemoteNotifications();

            return true;
        }

6. 在 **AppDelegate** 中，重写 **RegisteredForRemoteNotifications** 事件：

        public override void RegisteredForRemoteNotifications(UIApplication application, NSData deviceToken)
        {
            // Modify device token
            DeviceToken = deviceToken.Description;
            DeviceToken = DeviceToken.Trim ('<', '>').Replace (" ", "");

            // Get Mobile Services client
            MobileServiceClient client = QSTodoService.DefaultService.GetClient;

            // Register for push with Mobile Services
            IEnumerable<string> tag = new List<string>() { "uniqueTag" };
            var push = client.GetPush ();
            push.RegisterNativeAsync (DeviceToken, tag);
        }

7. 在 **AppDelegate** 中，重写 **ReceivedRemoteNotification** 事件：

        public override void ReceivedRemoteNotification(UIApplication application, NSDictionary userInfo)
        {
            Debug.WriteLine(userInfo.ToString());
            NSObject inAppMessage;

            bool success = userInfo.TryGetValue(new NSString("inAppMessage"), out inAppMessage);

            if (success)
            {
                var alert = new UIAlertView("Got push notification", inAppMessage.ToString(), null, "OK", null);
                alert.Show();
            }
        }

8. 在 **QSTodoListViewController** 中，修改 **OnAdd** 操作以获取存储在 **AppDelegeate** 中的设备标记，并将它存储到所添加的 **TodoItem** 中。

      string deviceToken = ((AppDelegate)UIApplication.SharedApplication.Delegate).DeviceToken;

			var newItem = new TodoItem() 
			{
				Text = itemText.Text, 
				Complete = false,
                DeviceToken = deviceToken
			};

你的应用现已更新，可支持推送通知。

## <a name="update-scripts"></a>在 Azure 管理门户中更新已注册的插入脚本

1. 在 [Azure 管理门户]中，单击“数据”选项卡，然后单击“TodoItem”表。

   	![][21]

2. 在 **todoitem** 中，单击“脚本”选项卡，然后选择“插入”。
   
  	![][22]

   	将显示当 **TodoItem** 表中发生插入时所调用的函数。

3. 将 insert 函数替换为以下代码，然后单击“保存”：

        function insert(item, user, request) {
            request.execute();
            // Set timeout to delay the notification, to provide time for the 
            // app to be closed on the device to demonstrate toast notifications
            setTimeout(function() {
                push.apns.send("uniqueTag", {
                    alert: "Toast: " + item.text,
                    payload: {
                        inAppMessage: "Hey, a new item arrived: '" + item.text + "'"
                    }
                });
            }, 2500);
        }

   	这将会注册一个新的插入脚本，该脚本使用[apns 对象]将推送通知（插入的文本）发送到插入请求中提供的设备。

   >[AZURE.NOTE]此脚本将延迟发送通知，使你有足够的时间关闭应用程序以接收 toast 通知。

##  <a name="test"></a>在应用程序中测试推送通知

1. 在支持 iOS 的设备中按“运行”按钮以生成项目并启动应用程序，然后单击“确定”接受推送通知

  	![][23]

       >[AZURE.NOTE]你必须显式接受来自应用程序的推送通知。此请求只会在首次运行应用程序时出现。

2. 在应用中键入有意义的文本（例如 _新的移动服务任务_），然后单击加号 (**+**) 图标。

  	![][24]

3. 检查是否已收到通知，然后单击“确定”以取消通知。

  	![][25]

4. 重复步骤 2 并立即关闭应用程序，然后检查是否已显示以下 toast。

  	![][26]

你已成功完成本教程。

<!-- Anchors. -->
[Generate the certificate signing request]: #certificates
[Register your app and enable push notifications]: #register
[Create a provisioning profile for the app]: #profile
[Configure Mobile Services]: #configure-mobileServices
[Configure the Xamarin.iOS App]: #configure-app
[Update scripts to send push notifications]: #update-scripts
[Add push notifications to the app]: #add-push
[Insert data to receive notifications]: #test

<!-- Images. -->

[5]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-step5.png
[6]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-step6.png
[7]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-step7.png

[9]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-step9.png
[10]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-step10.png

[17]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-step17.png
[18]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-selection.png
[19]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-push-tab-ios.png
[20]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-push-tab-ios-upload.png
[21]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-portal-data-tables.png
[22]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-insert-script-push2.png
[23]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-quickstart-push1-ios.png
[24]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-quickstart-push2-ios.png
[25]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-quickstart-push3-ios.png
[26]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-quickstart-push4-ios.png
[28]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-step18.png

[101]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-01.png
[102]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-02.png
[103]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-03.png
[104]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-04.png
[105]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-05.png
[106]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-06.png
[107]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-07.png
[108]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-08.png

[110]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-10.png
[111]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-11.png
[112]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-12.png
[113]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-13.png
[114]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-14.png
[115]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-15.png
[116]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-16.png
[117]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-17.png

[120]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-20.png
[121]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-21.png
[122]: ./media/partner-xamarin-mobile-services-ios-get-started-push/mobile-services-ios-push-22.png

[Install Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[iOS Provisioning Portal]: http://go.microsoft.com/fwlink/p/?LinkId=272456
[Mobile Services iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[Apple 推送通知服务]: http://go.microsoft.com/fwlink/p/?LinkId=272584
[移动服务入门]: /documentation/articles/mobile-services-ios-get-started/

[Xamarin 设备设置]: http://developer.xamarin.com/guides/ios/getting_started/installation/device_provisioning/


[Azure 管理门户]: https://manage.windowsazure.cn/
[apns 对象]: http://go.microsoft.com/fwlink/p/?LinkId=272333
[Azure 移动服务组件]: http://components.xamarin.com/view/azure-mobile-services/
[completed example project]: http://go.microsoft.com/fwlink/p/?LinkId=331303
[Xamarin Studio]: http://xamarin.com/download

<!---HONumber=Mooncake_0118_2016-->