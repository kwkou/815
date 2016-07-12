<properties
	pageTitle="通知中心突发新闻教程 - iOS"
	description="了解如何使用 Azure 服务总线通知中心向 iOS 设备发送突发新闻通知。"
	services="notification-hubs"
	documentationCenter="ios"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="notification-hubs"
	ms.date="03/28/2016"
	wacn.date="05/18/2016"/>

# 使用通知中心发送突发新闻

[AZURE.INCLUDE [notification-hubs-selector-breaking-news](../includes/notification-hubs-selector-breaking-news.md)]


##概述

本主题说明如何使用 Azure 通知中心将突发新闻通知广播到 iOS 应用程序。完成时，你可以注册感兴趣的突发新闻类别并仅接收这些类别的推送通知。此方案对于很多应用程序来说是常见模式，在其中必须将通知发送到以前声明过对它们感兴趣的一组用户，这样的应用程序有 RSS 阅读器、针对音乐迷的应用程序等。

在创建通知中心的注册时，通过加入一个或多个_标记_来启用广播方案。将通知发送到标签时，已注册该标签的所有设备将接收通知。因为标签是简单的字符串，它们不必提前设置。有关标记的详细信息，请参阅[通知中心指南]。


##先决条件

本主题以你在[通知中心入门][get-started]中创建的应用程序为基础。在开始本教程之前，必须先阅读[通知中心入门][get-started]。

##向应用程序中添加类别选择

第一步是向现有 Storyboard 添加 UI 元素，这些元素允许用户选择要注册的类别。用户选择的类别存储在设备上。应用程序启动时，使用所选类别作为标签在你的通知中心创建设备注册。

1. 在 MainStoryboard\_iPhone.storyboard 中，从对象库添加以下组件：
	+ 具有“Breaking News”文本的标签
	+ 具有“World”、“Politics”、“Business”、“Technology”、“Science”、“Sports”类别文本的标签
	+ 六个开关，每个类别一个。默认情况下，每个开关的 **State** 设置为 **Off**。
	+ 一个标有“Subscribe”的按钮

	Storyboard 应类似于：

	![][3]

2. 在助手编辑器中，为所有开关创建插座并称它们为“WorldSwitch”、“PoliticsSwitch”、“BusinessSwitch”、“TechnologySwitch”、“ScienceSwitch”、“SportsSwitch”


3. 为名为“subscribe”的按钮创建一个操作。ViewController.h 应包含以下内容：

		@property (weak, nonatomic) IBOutlet UISwitch *WorldSwitch;
		@property (weak, nonatomic) IBOutlet UISwitch *PoliticsSwitch;
		@property (weak, nonatomic) IBOutlet UISwitch *BusinessSwitch;
		@property (weak, nonatomic) IBOutlet UISwitch *TechnologySwitch;
		@property (weak, nonatomic) IBOutlet UISwitch *ScienceSwitch;
		@property (weak, nonatomic) IBOutlet UISwitch *SportsSwitch;

		- (IBAction)subscribe:(id)sender;

4. 创建名为 `Notifications` 的新 **Cocoa Touch 类**。在文件 Notifications.h 的接口部分中复制以下代码：

		@property NSData* deviceToken;

		- (id)initWithConnectionString:(NSString*)listenConnectionString HubName:(NSString*)hubName;

		- (void)storeCategoriesAndSubscribeWithCategories:(NSArray*)categories
					completion:(void (^)(NSError* error))completion;

		- (NSSet*)retrieveCategories;

		- (void)subscribeWithCategories:(NSSet*)categories completion:(void (^)(NSError *))completion;

5. 将以下导入指令添加到 Notifications.m：

		#import <WindowsAzureMessaging/WindowsAzureMessaging.h>

6. 在文件 Notifications.m 的实现部分中复制以下代码。

	    SBNotificationHub* hub;

		- (id)initWithConnectionString:(NSString*)listenConnectionString HubName:(NSString*)hubName{

		    hub = [[SBNotificationHub alloc] initWithConnectionString:listenConnectionString
										notificationHubPath:hubName];

			return self;
		}

		- (void)storeCategoriesAndSubscribeWithCategories:(NSSet *)categories completion:(void (^)(NSError *))completion {
		    NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
		    [defaults setValue:[categories allObjects] forKey:@"BreakingNewsCategories"];

		    [self subscribeWithCategories:categories completion:completion];
		}


		- (NSSet*)retrieveCategories {
		    NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

		    NSArray* categories = [defaults stringArrayForKey:@"BreakingNewsCategories"];

		    if (!categories) return [[NSSet alloc] init];
		    return [[NSSet alloc] initWithArray:categories];
		}


		- (void)subscribeWithCategories:(NSSet *)categories completion:(void (^)(NSError *))completion
		{
		   //[hub registerNativeWithDeviceToken:self.deviceToken tags:categories completion: completion];

			NSString* templateBodyAPNS = @"{"aps":{"alert":"$(messageParam)"}}";

			[hub registerTemplateWithDeviceToken:self.deviceToken name:@"simpleAPNSTemplate" 
				jsonBodyTemplate:templateBodyAPNS expiryTemplate:@"0" tags:categories completion:completion];
		}



	此类使用本地存储区存储和检索此设备将要接收的新闻类别。此外，它还包含了一个方法用于通过[模板](/documentation/articles/notification-hubs-templates/)注册来注册这些类别。

7. 在 AppDelegate.h 文件中，添加 Notifications.h 的导入语句，并添加 Notifications 类实例的属性：

		#import "Notifications.h"

		@property (nonatomic) Notifications* notifications;
	

8. 在 AppDelegate.m 的 **didFinishLaunchingWithOptions** 方法中，于方法开头添加代码来初始化 notifications 实例：
 
	在 hubinfo.h 中定义的 `HUBNAME` 和 `HUBLISTENACCESS` 内，`<hub name>` 和 `<connection string with listen access>` 占位符应已替换为你的通知中心的名称和你之前获取的 DefaultListenSharedAccessSignature 的连接字符串。

		self.notifications = [[Notifications alloc] initWithConnectionString:HUBLISTENACCESS HubName:HUBNAME];

	> [AZURE.NOTE]由于使用客户端应用程序分发的凭据通常是不安全的，你只应使用客户端应用程序分发具有侦听访问权限的密钥。侦听访问权限允许应用程序注册通知，但是无法修改现有注册，也无法发送通知。在受保护的后端服务中使用完全访问权限密钥，以便发送通知和更改现有注册。


9. 在 AppDelegate.m 的 **didRegisterForRemoteNotificationsWithDeviceToken** 方法中，使用以下代码来替换方法中的代码，以将设备令牌传递给 notifications 类。notifications 类将通知注册到类别。如果用户更改类别选择，我们将调用 `subscribeWithCategories` 方法以响应“订阅”按钮来进行更新。

	> [AZURE.NOTE]由于 Apple 推送通知服务 (APNS) 分配的设备标记随时可能更改，因此你应该经常注册通知以避免通知失败。此示例在每次应用程序启动时注册通知。对于经常运行（一天一次以上）的应用程序，如果每次注册间隔时间不到一天，你可以跳过注册来节省带宽。

		self.notifications.deviceToken = deviceToken;

		// Retrieves the categories from local storage and requests a registration for these categories
		// each time the app starts and performs a registration.

	    NSSet* categories = [self.notifications retrieveCategories];
	    [self.notifications subscribeWithCategories:categories completion:^(NSError* error) {
	        if (error != nil) {
	            NSLog(@"Error registering for notifications: %@", error);
	        }
	    }];


	请注意，此时 **didRegisterForRemoteNotificationsWithDeviceToken** 方法中应没有其他代码。

10.	完成[通知中心入门][get-started]教程时，以下方法应已经出现在 AppDelegate.m 中。否则，请添加这些方法。

		-(void)MessageBox:(NSString *)title message:(NSString *)messageText
		{
			UIAlertView *alert = [[UIAlertView alloc] initWithTitle:title message:messageText delegate:self 
				cancelButtonTitle:@"OK" otherButtonTitles: nil];
			[alert show];
		}

		- (void)application:(UIApplication *)application didReceiveRemoteNotification:
			(NSDictionary *)userInfo {
		    NSLog(@"%@", userInfo);
		    [self MessageBox:@"Notification" message:[[userInfo objectForKey:@"aps"] valueForKey:@"alert"]];
	    }

	此方法通过显示简单的 **UIAlert** 处理运行应用程序时收到的通知。

11. 在 ViewController.m 中，添加 AppDelegate.h 的导入语句，并将以下代码复制到 XCode 生成的 **subscribe** 方法中。此代码将更新通知注册，以使用用户在用户界面中选择的新类别标记。

		```
		#import "Notifications.h"
		```

		NSMutableArray* categories = [[NSMutableArray alloc] init];

	    if (self.WorldSwitch.isOn) [categories addObject:@"World"];
	    if (self.PoliticsSwitch.isOn) [categories addObject:@"Politics"];
	    if (self.BusinessSwitch.isOn) [categories addObject:@"Business"];
	    if (self.TechnologySwitch.isOn) [categories addObject:@"Technology"];
	    if (self.ScienceSwitch.isOn) [categories addObject:@"Science"];
	    if (self.SportsSwitch.isOn) [categories addObject:@"Sports"];

	    Notifications* notifications = [(AppDelegate*)[[UIApplication sharedApplication]delegate] notifications];

	    [notifications storeCategoriesAndSubscribeWithCategories:categories completion: ^(NSError* error) {
	        if (!error) {
	            [(AppDelegate*)[[UIApplication sharedApplication]delegate] MessageBox:@"Notification" message:@"Subscribed!"];
	        } else {
	            NSLog(@"Error subscribing: %@", error);
	        }
	    }];

	此方法创建一个类别的 **NSMutableArray** 并使用 **Notifications** 类将该列表存储在本地存储区中，将相应的标记注册到你的通知中心。更改类别时，使用新类别重新创建注册。

12. 在 ViewController.m 中，于 **viewDidLoad** 方法中添加以下代码，以根据前面保存的类别来设置用户界面。


		// This updates the UI on startup based on the status of previously saved categories.

		Notifications* notifications = [(AppDelegate*)[[UIApplication sharedApplication]delegate] notifications];

	    NSSet* categories = [notifications retrieveCategories];

	    if ([categories containsObject:@"World"]) self.WorldSwitch.on = true;
	    if ([categories containsObject:@"Politics"]) self.PoliticsSwitch.on = true;
	    if ([categories containsObject:@"Business"]) self.BusinessSwitch.on = true;
	    if ([categories containsObject:@"Technology"]) self.TechnologySwitch.on = true;
	    if ([categories containsObject:@"Science"]) self.ScienceSwitch.on = true;
	    if ([categories containsObject:@"Sports"]) self.SportsSwitch.on = true;



应用程序现在可以在设备的本地存储区中存储一组类别，每当应用程序启动时，将使用这些类别注册到通知中心。用户可以在运行时更改选择的类别，然后单击 **subscribe** 方法来更新设备注册。接下来，你将更新应用程序，以直接从应用本身发送突发新闻通知。


##（可选）发送带标记的通知

如果你无权访问 Visual Studio，可以跳到下一部分，并从应用内部发送通知。你还可以在 [Azure 经典管理门户] 中使用通知中心的调试选项卡发送适当的模板通知。

[AZURE.INCLUDE [notification-hubs-send-categories-template](../includes/notification-hubs-send-categories-template.md)]


##（可选）从设备发送通知

通常，通知将由后端服务发送，但你也可以直接从应用发送突发新闻通知。为此，我们需要更新[通知中心入门][get-started]教程中所定义的 `SendNotificationRESTAPI` 方法。


1. 在 ViewController.m 中，按如下所示更新 `SendNotificationRESTAPI` 方法，使其接受类别标记的参数并发送适当的[模板](/documentation/articles/notification-hubs-templates/)通知。

		- (void)SendNotificationRESTAPI:(NSString*)categoryTag
		{
		    NSURLSession* session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration
									 defaultSessionConfiguration] delegate:nil delegateQueue:nil];
		    
		    NSString *json;
		    
		    // Construct the messages REST endpoint
		    NSURL* url = [NSURL URLWithString:[NSString stringWithFormat:@"%@%@/messages/%@", HubEndpoint,
		                                       HUBNAME, API_VERSION]];
		    
		    // Generated the token to be used in the authorization header.
		    NSString* authorizationToken = [self generateSasToken:[url absoluteString]];

		    //Create the request to add the template notification message to the hub
		    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
		    [request setHTTPMethod:@"POST"];
		    
		    // Add the category as a tag
		    [request setValue:categoryTag forHTTPHeaderField:@"ServiceBusNotification-Tags"];

			// Template notification
	        json = [NSString stringWithFormat:@"{"messageParam":"Breaking %@ News : %@"}",
	                categoryTag, self.notificationMessage.text];

	        // Signify template notification format
	        [request setValue:@"template" forHTTPHeaderField:@"ServiceBusNotification-Format"];

			// JSON Content-Type
			[request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];

		    //Authenticate the notification message POST request with the SaS token
		    [request setValue:authorizationToken forHTTPHeaderField:@"Authorization"];
		    
		    //Add the notification message body
		    [request setHTTPBody:[json dataUsingEncoding:NSUTF8StringEncoding]];
		    
		    // Send the REST request
		    NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request
		               completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
		               {
		               NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
		                   if (error || httpResponse.statusCode != 200)
		                   {
		                       NSLog(@"\nError status: %d\nError: %@", httpResponse.statusCode, error);
		                   }
		                   if (data != NULL)
		                   {
		                       //xmlParser = [[NSXMLParser alloc] initWithData:data];
		                       //[xmlParser setDelegate:self];
		                       //[xmlParser parse];
		                   }
		               }];
		    [dataTask resume];
		}
		


2. 在 ViewController.m 中，更新“发送通知”操作（如以下代码所示）。因此，它将使用每个标记分别发送通知，并发送到多个平台。



		- (IBAction)SendNotificationMessage:(id)sender
		{
		    self.sendResults.text = @"";
		    
		    NSArray* categories = [NSArray arrayWithObjects: @"World", @"Politics", @"Business", 
									@"Technology", @"Science", @"Sports", nil];
		
		    // Lets send the message as breaking news for each category to WNS, GCM, and APNS
			// using a template.
		    for(NSString* category in categories)
		    {
		        [self SendNotificationRESTAPI:category];
		    }
		}



3. 重新生成项目，并确定没有生成错误。


##运行应用并生成通知

1. 按“运行”按钮生成项目并启动应用程序。选择要订阅的一些突发新闻选项，然后按“订阅”按钮。你应会看到一个对话框，表示已订阅通知。

	![][1]

	选择“订阅”时，应用程序将所选类别转换为标记并针对所选标签从通知中心请求注册新设备。

2. 输入要以突发新闻形式发送的消息，然后按“发送通知”按钮。或者，运行 .NET 控制台应用以生成通知。

	![][2]


3. 每个订阅突发新闻的设备都会收到刚刚发送的突发新闻通知。



## 后续步骤

在本教程中，我们了解了如何按类别广播突发消息。请考虑学习侧重说明其他高级通知中心方案的以下教程之一：

+ **[使用通知中心广播本地化的突发新闻]**

	了解如何扩展突发新闻应用程序以允许发送本地化的通知。





<!-- Images. -->
[1]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-subscribed.png
[2]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-ios1.png
[3]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-ios2.png








<!-- URLs. -->
[How To: Service Bus Notification Hubs (iOS Apps)]: http://msdn.microsoft.com/zh-cn/library/jj927168.aspx
[使用通知中心广播本地化的突发新闻]: /documentation/articles/notification-hubs-ios-send-localized-breaking-news/
[Mobile Service]: /documentation/articles/notification-hubs-ios-mobile-services-register-user-push-notifications/
[使用通知中心通知用户]: /documentation/articles/notification-hubs-ios-aspnet-register-user-push-notifications/
[Azure Management Portal]: https://manage.windowsazure.cn/
[通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
[Notification Hubs How-To for iOS]: http://msdn.microsoft.com/zh-cn/library/jj927168.aspx
[get-started]: /documentation/articles/notification-hubs-ios-get-started/
[Azure 经典管理门户]: https://manage.windowsazure.cn
<!---HONumber=Mooncake_0503_2016-->