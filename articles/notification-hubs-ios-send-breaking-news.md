<properties linkid="develop-notificationhubs-tutorials-send-breaking-news-ios" urlDisplayName="Breaking News" pageTitle="Notification Hubs Breaking News Tutorial - iOS" metaKeywords="" description="Learn how to use Azure Service Bus Notification Hubs to send breaking news notifications to iOS devices." metaCanonical="" services="mobile-services,notification-hubs" documentationCenter="" title="Use Notification Hubs to send breaking news" authors="ricksal" solutions="" manager="" editor="" />

# 使用通知中心发送突发新闻

<div class="dev-center-tutorial-selector sublanding">       
<a href="/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/" title="Windows 应用商店 C#" >Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/notification-hubs-windows-phone-send-breaking-news/" title="Windows Phone">Windows Phone</a><a href="/zh-cn/documentation/articles/notification-hubs-ios-send-breaking-news/" title="iOS" class="current">iOS</a>
</div>

本主题说明如何使用 Azure 通知中心将突发新闻通知广播到 iOS 应用程序。完成时，你可以注册感兴趣的突发新闻类别并仅接收这些类别的推送通知。此方案对于很多应用程序来说是常见模式，在其中必须将通知发送到以前声明过对它们感兴趣的一组用户，这样的应用程序有 RSS 阅读器、针对音乐迷的应用程序等。

在通知中心创建注册时通过包括一个或多个*标签*来启用广播方案。将通知发送到标签时，已注册该标签的所有设备将接收通知。因为标签是简单的字符串，它们不必提前设置。有关标签的更多信息，请参见[通知中心指南][通知中心指南]。

本教程将指导你完成启用此方案的以下基本步骤：

1.  [向应用程序中添加类别选择][向应用程序中添加类别选择]
2.  [注册通知][注册通知]
3.  [从后端发送通知][从后端发送通知]
4.  [运行应用程序并生成通知][运行应用程序并生成通知]

本主题以你在[通知中心入门][通知中心入门]中创建的应用程序为基础。在开始本教程之前，必须先阅读[通知中心入门][通知中心入门]。

## <a name="adding-categories"></a>向应用程序中添加类别选择

第一步是向现有 Storyboard 添加 UI 元素，这些元素允许用户选择要注册的类别。用户选择的类别存储在设备上。应用程序启动时，使用所选类别作为标签在你的通知中心创建设备注册。

1.  在 MainStoryboard\_iPhone.storyboard 中，从对象库添加以下组件：

    -   具有“Breaking News”文本的标签
    -   具有“World”、“Politics”、“Business”、“Technology”、“Science”、“Sports”类别文本的标签
    -   六个开关，每个类别一个
    -   “Subscribe”按钮

    Storyboard 应类似于：

    ![][]

2.  在助手编辑器中，为所有开关创建插座并称它们为“WorldSwitch”、“PoliticsSwitch”、“BusinessSwitch”、“TechnologySwitch”、“ScienceSwitch”、“SportsSwitch”

    ![][1]

3.  为名为“subscribe”的按钮创建一个操作。BreakingNewsViewController.h 应包含以下内容：

        @property (weak, nonatomic) IBOutlet UISwitch *WorldSwitch;
        @property (weak, nonatomic) IBOutlet UISwitch *PoliticsSwitch;
        @property (weak, nonatomic) IBOutlet UISwitch *BusinessSwitch;
        @property (weak, nonatomic) IBOutlet UISwitch *TechnologySwitch;
        @property (weak, nonatomic) IBOutlet UISwitch *ScienceSwitch;
        @property (weak, nonatomic) IBOutlet UISwitch *SportsSwitch;

        - (IBAction)subscribe:(id)sender;

4.  创建新类 `Notifications`. 在文件 Notifications.h 的接口部分中复制以下代码：

        - (void)storeCategoriesAndSubscribeWithCategories:(NSArray*) categories completion: (void (^)(NSError* error))completion;
        - (void)subscribeWithCategories:(NSSet*) categories completion:(void (^)(NSError *))completion;

5.  将以下导入指令添加到 Notifications.m：

        #import <WindowsAzureMessaging/WindowsAzureMessaging.h>

6.  在文件 Notifications.m 的实现部分中复制以下代码：

        - (void)storeCategoriesAndSubscribeWithCategories:(NSSet *)categories completion:(void (^)(NSError *))completion {
            NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
            [defaults setValue:[categories allObjects] forKey:@"BreakingNewsCategories"];

            [self subscribeWithCategories:categories completion:completion];
        }

        - (void)subscribeWithCategories:(NSSet *)categories completion:(void (^)(NSError *))completion{
            SBNotificationHub* hub = [[SBNotificationHub alloc] initWithConnectionString:@"<connection string with listen access>" notificationHubPath:@"<hub name>"];

            [hub registerNativeWithDeviceToken:self.deviceToken tags:categories completion: completion];
        }

    此类使用本地存储区存储此设备必须接收的新闻类别。它还包含注册这些类别所用的方法。

7.  在上述代码中，将`<hub name>` 和 `<connection string with listen access>` 占位符替换为通知中心名称和前面获取的 *DefaultListenSharedAccessSignature* 的连接字符串。

    <div class="dev-callout"><strong>说明</strong> 
<p>由于使用客户端应用程序分发的凭据通常是不安全的，你只应使用客户端应用程序分发具有侦听访问权限的密钥。侦听访问权限允许应用程序注册通知，但是无法修改现有注册，也无法发送通知。在受保护的后端服务中使用完全访问权限密钥，以便发送通知和更改现有注册。</p>
</div>

8.  在 BreakingNewsAppDelegate.h 文件中，添加以下属性：

        @property (nonatomic) Notifications* notifications;

    它在 AppDelegate 中创建 Notification 类的单一实例。

9.  在 BreakingNewsAppDelegate.m 的 **didFinishLaunchingWithOptions** 方法中，在 **registerForRemoteNotificationTypes** 的前面添加以下代码：

        self.notifications = [[Notifications alloc] init];

    初始化 Notification 单一实例。

10. 在 BreakingNewsAppDelegate.m 的 **didRegisterForRemoteNotificationsWithDeviceToken** 方法中，删除对 **registerNativeWithDeviceToken** 的调用并添加以下代码：

        self.notifications.deviceToken = deviceToken;

    请注意，此时 **didRegisterForRemoteNotificationsWithDeviceToken** 方法中应没有其他代码。

11. 在 BreakingNewsAppDelegate.m 中添加以下方法：

        - (void)application:(UIApplication *)application didReceiveRemoteNotification:
            (NSDictionary *)userInfo {
            NSLog(@"%@", userInfo);
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
            [[userInfo objectForKey:@"aps"] valueForKey:@"alert"] delegate:nil cancelButtonTitle: @"OK" otherButtonTitles:nil, nil];
            [alert show];
        }

    此方法通过显示简单的 **UIAlert** 处理运行应用程序时收到的通知。

12. 在 BreakingNewsViewController.m 中，将以下代码复制到 XCode-generated **subscribe** 方法：

        NSMutableArray* categories = [[NSMutableArray alloc] init];

        if (self.WorldSwitch.isOn) [categories addObject:@"World"];
        if (self.PoliticsSwitch.isOn) [categories addObject:@"Politics"];
        if (self.BusinessSwitch.isOn) [categories addObject:@"Business"];
        if (self.TechnologySwitch.isOn) [categories addObject:@"Technology"];
        if (self.ScienceSwitch.isOn) [categories addObject:@"Science"];
        if (self.SportsSwitch.isOn) [categories addObject:@"Sports"];

        Notifications* notifications = [(BreakingNewsAppDelegate*)[[UIApplication sharedApplication]delegate] notifications];

        [notifications storeCategoriesAndSubscribeWithCategories:categories completion: ^(NSError* error) {
            if (!error) {
                UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
                                      @"Subscribed!" delegate:nil cancelButtonTitle:
                                      @"OK" otherButtonTitles:nil, nil];
                [alert show];
            } else {
                NSLog(@"Error subscribing: %@", error);
            }
        }];

    此方法创建一个类别的 **NSMutableArray** 并使用 **Notifications** 类将该列表存储在本地存储区中，将相应的标记注册到你的通知中心。更改类别时，使用新类别重新创建注册。

你的应用程序现在可以将一组类别存储在设备的本地存储区中了，每当用户更改所选类别时，会将这些类别注册到通知中心。

## <a name="register"></a>注册通知

这些步骤用于在启动时将在本地存储区中存储的类别注册到通知中心。

<div class="dev-callout"><strong>说明</strong> 
<p>由于 Apple 推送通知服务 (APNS) 分配的设备标记随时可能更改，因此你应该经常注册通知以避免通知失败。此示例在每次应用程序启动时注册通知。对于经常运行（一天一次以上）的应用程序，如果每次注册间隔时间不到一天，你可以跳过注册来节省带宽。</p>
</div>

1.  在文件 Notifications.h 的接口部分中添加以下方法：

        - (NSSet*)retrieveCategories;

    此代码检索 Notifications 类中的类别。

2.  在文件 Notifications.m 中添加相应的实现：

        - (NSSet*)retrieveCategories {
            NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

            NSArray* categories = [defaults stringArrayForKey:@"BreakingNewsCategories"];

            if (!categories) return [[NSSet alloc] init];
            return [[NSSet alloc] initWithArray:categories];
        }

3.  在 **didRegisterForRemoteNotificationsWithDeviceToken** 方法中添加以下代码：

        Notifications* notifications = [(BreakingNewsAppDelegate*)[[UIApplication sharedApplication]delegate] notifications];

        NSSet* categories = [notifications retrieveCategories];
        [notifications subscribeWithCategories:categories completion:^(NSError* error) {
            if (error != nil) {
                NSLog(@"Error registering for notifications: %@", error);
            }
        }];

    这确保每次应用程序启动时，它从本地存储区检索类别并请求注册这些类别。

4.  在 BreakingNewsViewController.h 的 **viewDidLoad** 方法中添加以下代码：

        Notifications* notifications = [(BreakingNewsAppDelegate*)[[UIApplication sharedApplication]delegate] notifications];

        NSSet* categories = [notifications retrieveCategories];

        if ([categories containsObject:@"World"]) self.WorldSwitch.on = true;
        if ([categories containsObject:@"Politics"]) self.PoliticsSwitch.on = true;
        if ([categories containsObject:@"Business"]) self.BusinessSwitch.on = true;
        if ([categories containsObject:@"Technology"]) self.TechnologySwitch.on = true;
        if ([categories containsObject:@"Science"]) self.ScienceSwitch.on = true;
        if ([categories containsObject:@"Sports"]) self.SportsSwitch.on = true;

    这基于以前保存的类别状态更新启动时的 UI。

应用程序现在已完成，可以在设备的本地存储区中存储一组类别了，每当用户更改所选类别时将使用这些类别注册到通知中心。接下来，你要定义一个后端，它可将类别通知发送到此应用程序。

## <a name="send"></a><span class="short-header">发送通知</span>从后端发送通知

[WACOM.INCLUDE [notification-hubs-back-end][notification-hubs-back-end]]

## <a name="test-app"></a>运行应用程序并生成通知

1.  按“运行”按钮生成项目并且启动应用程序。

    ![][2]

    请注意，应用程序 UI 提供了一组开关，你可以使用它们选择要订阅的类别。

2.  启用一个或多个类别开关，然后单击“订阅”。

    选择“订阅”时，应用程序将所选类别转换为标记并针对所选标签从通知中心请求注册新设备。

3.  使用以下方式之一从后端发送新通知：

    -   **控制台应用程序：**启动控制台应用程序。

    -   **移动服务：**单击“计划程序”选项卡，单击该作业，然后单击“运行一次”。

4.  所选类别的通知作为 toast 通知显示。

## <a name="next-steps"> </a>后续步骤

在本教程中，我们了解了如何按类别广播突发消息。请考虑学习侧重说明其他高级通知中心方案的以下教程之一：

-   **[使用通知中心广播本地化的突发新闻][使用通知中心广播本地化的突发新闻]**

    了解如何扩展突发新闻应用程序以允许发送本地化的通知。

-   **[使用通知中心通知用户][使用通知中心通知用户]**

    了解如何将通知推送到经过身份验证的特定用户。这是仅将通知发送到特定用户的好的解决方案。

<!-- Anchors. -->  

<!-- URLs. -->

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/ "Windows 应用商店 C#"
  [Windows Phone]: /zh-cn/documentation/articles/notification-hubs-windows-phone-send-breaking-news/ "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/notification-hubs-ios-send-breaking-news/ "iOS"
  [通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
  [向应用程序中添加类别选择]: #adding-categories
  [注册通知]: #register
  [从后端发送通知]: #send
  [运行应用程序并生成通知]: #test-app
  [通知中心入门]: /en-us/manage/services/notification-hubs/get-started-notification-hubs-ios/

<!-- Images. -->
  []: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-ios2.png
  [1]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-ios3.png
  [notification-hubs-back-end]: ../includes/notification-hubs-back-end.md
  [2]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-ios1.png
  [使用通知中心广播本地化的突发新闻]: /en-us/manage/services/notification-hubs/breaking-news-localized-dotnet/
  [使用通知中心通知用户]: /en-us/manage/services/notification-hubs/notify-users/
