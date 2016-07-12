<properties
	pageTitle="通过 Azure 通知中心向 iOS 发送推送通知 | Microsoft Azure"
	description="在本教程中，你将了解如何使用 Azure 通知中心将推送通知发送到 iOS 应用程序。"
	services="notification-hubs"
	documentationCenter="ios"
   	keywords="推送通知, 推送通知, ios 推送通知"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="notification-hubs"
	ms.date="03/14/2016"
	wacn.date="05/09/2016"/>

# 通过 Azure 通知中心向 iOS 发送推送通知

[AZURE.INCLUDE [notification-hubs-selector-get-started](../includes/notification-hubs-selector-get-started.md)]

##概述

> [AZURE.NOTE] 若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/free-trial/)。

本教程演示如何使用 Azure 通知中心将推送通知发送到 iOS 应用程序。你将创建一个空白 iOS 应用，它使用 [Apple Push Notification 服务 (APNs)](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html) 接收推送通知。

完成后，你将能够使用通知中心将推送通知广播到运行你的应用的所有设备。

## 开始之前

[AZURE.INCLUDE [notification-hubs-hero-slug](../includes/notification-hubs-hero-slug.md)]

可以在 GitHub 上的[此处](https://github.com/Azure/azure-notificationhubs-samples/tree/master/iOS/GetStartedNH/GetStarted)找到本教程的已完成代码。

##先决条件

本教程需要的内容如下：

+ [移动服务 iOS SDK 版本 1.2.4]
+ 最新版本的 [Xcode]
+ 支持 iOS 8（或更高版本）的设备
+ [Apple 开发人员计划](https://developer.apple.com/programs/)成员身份。

   > [AZURE.NOTE] 由于推送通知配置要求，你必须在支持 iOS 的设备（iPhone 或 iPad）而不是在 iOS 模拟器上部署和测试推送通知。

完成本教程是学习有关 iOS 应用的所有其他通知中心教程的先决条件。

[AZURE.INCLUDE [通知中心：启用 Apple 推送通知](../includes/notification-hubs-enable-apple-push-notifications.md)]


##将你的 iOS 应用连接到通知中心

1. 在 Xcode 中，创建新的 iOS 项目，然后选择“单视图应用程序”模板。

   	![Xcode — 单一视图应用程序][8]

2. 设置新项目的选项时，请务必使用前面在 Apple 开发人员门户上设置捆绑 ID 时使用的同一**产品名称**和**组织标识符**。

	![Xcode — 项目选项][11]

3. 在“目标”下，单击你的项目名称，单击“生成设置”选项卡，展开“代码签名标识”，然后在“调试”下设置你的代码签名标识。将“级别”从“基本”切换为“全部”，然后将“预配配置文件”设为前面创建的预配配置文件。

	如果屏幕未显示在 Xcode 中创建的新预配配置文件，请尝试刷新签名标识的配置文件。单击菜单栏上的“Xcode”，再依次单击“首选项”、“帐户”选项卡、“查看详细信息”按钮、你的签名标识，然后单击右下角的刷新按钮。

   	![Xcode — 预配配置文件][9]

4. 下载[移动服务 iOS SDK 版本 1.2.4]，然后将文件解压缩。在 Xcode 中，右键单击你的项目，然后单击“将文件添加到”选项，将 **WindowsAzureMessaging.framework** 文件夹添加到 Xcode 项目。选择“需要时复制项”，然后单击“添加”。

	>[AZURE.NOTE] 通知中心 SDK 目前不支持 Xcode 7 上的 Bitcode。你必须在项目的“构建选项”中将“启用 Bitcode”设置为“否”。

   	![解压缩 Azure SDK][10]

5. 将新的标头文件添加到名为 `HubInfo.h` 的项目。此文件将保存通知中心的常量。添加以下定义，然后将字符串文本占位符替换为你的中心名称以及前面记下的 *DefaultListenSharedAccessSignature*。

		#ifndef HubInfo_h
		#define HubInfo_h
		
			#define HUBNAME @"<Enter the name of your hub>"
			#define HUBLISTENACCESS @"<Enter your DefaultListenSharedAccess connection string"
		
		#endif /* HubInfo_h */

6. 打开 `AppDelegate.h` 文件并添加以下导入指令：

         #import <WindowsAzureMessaging/WindowsAzureMessaging.h> 
		 #import "HubInfo.h"
		
7. 根据你的 iOS 版本，在 `AppDelegate.m file` 的 `didFinishLaunchingWithOptions` 方法中添加以下代码。此代码将向 APNs 注册设备句柄：

	对于 iOS 8：

	 	UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeSound |
												UIUserNotificationTypeAlert | UIUserNotificationTypeBadge categories:nil];

    	[[UIApplication sharedApplication] registerUserNotificationSettings:settings];
    	[[UIApplication sharedApplication] registerForRemoteNotifications];

	对于 iOS 8 之前的版本：

         [[UIApplication sharedApplication] registerForRemoteNotificationTypes: UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];


8. 在同一文件中，添加以下方法。此代码将使用你在 HubInfo.h 中指定的连接信息连接到通知中心。然后，它向通知中心提供设备令牌，使通知中心能够发送通知：

	    - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *) deviceToken {
		    SBNotificationHub* hub = [[SBNotificationHub alloc] initWithConnectionString:HUBLISTENACCESS
										notificationHubPath:HUBNAME];

		    [hub registerNativeWithDeviceToken:deviceToken tags:nil completion:^(NSError* error) {
		        if (error != nil) {
		            NSLog(@"Error registering for notifications: %@", error);
		        }
				else {
				    [self MessageBox:@"Registration Status" message:@"Registered"];
				}
	    	}];
		}

		-(void)MessageBox:(NSString *)title message:(NSString *)messageText
		{
			UIAlertView *alert = [[UIAlertView alloc] initWithTitle:title message:messageText delegate:self
				cancelButtonTitle:@"OK" otherButtonTitles: nil];
			[alert show];
		}


9. 在同一文件中，添加以下方法以便在应用处于活动状态时收到通知的情况下显示 **UIAlert**：


        - (void)application:(UIApplication *)application didReceiveRemoteNotification: (NSDictionary *)userInfo {
		    NSLog(@"%@", userInfo);
		    [self MessageBox:@"Notification" message:[[userInfo objectForKey:@"aps"] valueForKey:@"alert"]];
		}

10. 在设备上生成并运行应用，以验证是否不会失败。

## 发送测试推送通知


[AZURE.INCLUDE [notification-hubs-sending-notifications-from-the-portal](../includes/notification-hubs-sending-notifications-from-the-portal.md)]


## （可选）从应用发送推送通知

如果你要从应用内发送推送通知，本节提供如何使用 REST 接口执行此操作的示例。

1. 在 Xcode 中，打开 `Main.storyboard` 并从对象库添加以下 UI 组件，以允许用户在应用中发送推送通知：

	- 没有标签文本的标签。该标签用于报告发送通知时发生的错误。**Lines** 属性应该设为 **0**，以便自动根据左右边距和视图顶部的限制来调整大小。
	- 具有**占位符**文本的文本字段设为“输入通知消息”。将字段限制在标签的正下方，如下所示。将视图控制器设为容器委派。
	- 标题为“发送通知”的按钮限制在文本字段的正下方，并在水平居中的位置。

	该视图应如下所示：

	![Xcode 设计器][32]


2. 为与视图连接的标签和文本字段[添加容器](https://developer.apple.com/library/ios/recipes/xcode_help-IB_connections/chapters/CreatingOutlet.html)，并更新 `interface` 定义，以支持 `UITextFieldDelegate` 和 `NSXMLParserDelegate`。添加如下所示的三个属性声明，以帮助调用 REST API 和分析响应。

	ViewController.h 文件应如下所示：

		#import <UIKit/UIKit.h>

		@interface ViewController : UIViewController <UITextFieldDelegate, NSXMLParserDelegate>
		{
			NSXMLParser *xmlParser;
		}

		// Make sure these outlets are connected to your UI by ctrl+dragging
		@property (weak, nonatomic) IBOutlet UITextField *notificationMessage;
		@property (weak, nonatomic) IBOutlet UILabel *sendResults;

		@property (copy, nonatomic) NSString *statusResult;
		@property (copy, nonatomic) NSString *currentElement;

		@end

3. 打开 `HubInfo.h` 并添加以下常量，用于向中心发送通知。将占位符字符串文本替换为实际的 *DefaultFullSharedAccessSignature* 连接字符串。

		#define API_VERSION @"?api-version=2015-01"
		#define HUBFULLACCESS @"<Enter Your DefaultFullSharedAccess Connection string>"

4. 向 `ViewController.h` 文件添加以下 `#import` 语句。

		#import <CommonCrypto/CommonHMAC.h>
		#import "HubInfo.h"

5. 在 `ViewController.m` 中，将以下代码添加到接口实现。此代码将会分析你的 *DefaultFullSharedAccessSignature* 连接字符串。如 [REST API 参考](http://msdn.microsoft.com/library/azure/dn495627.aspx)中所述，此类已分析的信息将用于生成 **Authorization** 请求标头的 SaS 令牌。

		NSString *HubEndpoint;
		NSString *HubSasKeyName;
		NSString *HubSasKeyValue;

		-(void)ParseConnectionString
		{
			NSArray *parts = [HUBFULLACCESS componentsSeparatedByString:@";"];
			NSString *part;

			if ([parts count] != 3)
			{
				NSException* parseException = [NSException exceptionWithName:@"ConnectionStringParseException"
					reason:@"Invalid full shared access connection string" userInfo:nil];

				@throw parseException;
			}

			for (part in parts)
			{
				if ([part hasPrefix:@"Endpoint"])
				{
					HubEndpoint = [NSString stringWithFormat:@"https%@",[part substringFromIndex:11]];
				}
				else if ([part hasPrefix:@"SharedAccessKeyName"])
				{
					HubSasKeyName = [part substringFromIndex:20];
				}
				else if ([part hasPrefix:@"SharedAccessKey"])
				{
					HubSasKeyValue = [part substringFromIndex:16];
				}
			}
		}

6. 在 `ViewController.m` 中更新 `viewDidLoad` 方法，以便在加载视图时分析连接字符串。此外，将如下所示的实用程序方法添加到接口实现。


		- (void)viewDidLoad
		{
			[super viewDidLoad];
			[self ParseConnectionString];
			[_notificationMessage setDelegate:self];
		}

		-(NSString *)CF_URLEncodedString:(NSString *)inputString
		{
		   return (__bridge NSString *)CFURLCreateStringByAddingPercentEscapes(NULL, (CFStringRef)inputString,
				NULL, (CFStringRef)@"!*'();:@&=+$,/?%#[]", kCFStringEncodingUTF8);
		}

		-(void)MessageBox:(NSString *)title message:(NSString *)messageText
		{
			UIAlertView *alert = [[UIAlertView alloc] initWithTitle:title message:messageText delegate:self
				cancelButtonTitle:@"OK" otherButtonTitles: nil];
			[alert show];
		}





7. 如 [REST API 参考](http://msdn.microsoft.com/library/azure/dn495627.aspx)中所述，在 `ViewController.m` 中，将以下代码添加到接口实现，以生成将在 **Authorization** 标头中提供的 SaS 授权令牌。

		-(NSString*) generateSasToken:(NSString*)uri
		{
			NSString *targetUri;
			NSString* utf8LowercasedUri = NULL;
			NSString *signature = NULL;
			NSString *token = NULL;

			@try
			{
				// Add expiration
				uri = [uri lowercaseString];
				utf8LowercasedUri = [self CF_URLEncodedString:uri];
				targetUri = [utf8LowercasedUri lowercaseString];
				NSTimeInterval expiresOnDate = [[NSDate date] timeIntervalSince1970];
				int expiresInMins = 60; // 1 hour
				expiresOnDate += expiresInMins * 60;
				UInt64 expires = trunc(expiresOnDate);
				NSString* toSign = [NSString stringWithFormat:@"%@\n%qu", targetUri, expires];

				// Get an hmac_sha1 Mac instance and initialize with the signing key
				const char *cKey  = [HubSasKeyValue cStringUsingEncoding:NSUTF8StringEncoding];
				const char *cData = [toSign cStringUsingEncoding:NSUTF8StringEncoding];
				unsigned char cHMAC[CC_SHA256_DIGEST_LENGTH];
				CCHmac(kCCHmacAlgSHA256, cKey, strlen(cKey), cData, strlen(cData), cHMAC);
				NSData *rawHmac = [[NSData alloc] initWithBytes:cHMAC length:sizeof(cHMAC)];
				signature = [self CF_URLEncodedString:[rawHmac base64EncodedStringWithOptions:0]];

				// Construct authorization token string
				token = [NSString stringWithFormat:@"SharedAccessSignature sig=%@&se=%qu&skn=%@&sr=%@",
					signature, expires, HubSasKeyName, targetUri];
			}
			@catch (NSException *exception)
			{
				[self MessageBox:@"Exception Generating SaS Token" message:[exception reason]];
			}
			@finally
			{
				if (utf8LowercasedUri != NULL)
					CFRelease((CFStringRef)utf8LowercasedUri);
				if (signature != NULL)
				CFRelease((CFStringRef)signature);
			}

			return token;
		}


8. 按住 Ctrl 并从“发送通知”按钮拖到 `ViewController.m`，为 **Touch Down** 事件添加名为 **SendNotificationMessage** 的操作。使用以下代码更新方法，以使用 REST API 发送通知。

		- (IBAction)SendNotificationMessage:(id)sender
		{
			self.sendResults.text = @"";
			[self SendNotificationRESTAPI];
		}

		- (void)SendNotificationRESTAPI
		{
		    NSURLSession* session = [NSURLSession
                             sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]
                             delegate:nil delegateQueue:nil];

			// Apple Notification format of the notification message
		    NSString *json = [NSString stringWithFormat:@"{"aps":{"alert":"%@"}}",
								self.notificationMessage.text];

			// Construct the message's REST endpoint
			NSURL* url = [NSURL URLWithString:[NSString stringWithFormat:@"%@%@/messages/%@", HubEndpoint,
												HUBNAME, API_VERSION]];

			// Generate the token to be used in the authorization header
			NSString* authorizationToken = [self generateSasToken:[url absoluteString]];

			//Create the request to add the APNs notification message to the hub
			NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
			[request setHTTPMethod:@"POST"];
			[request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];

			// Signify Apple notification format
			[request setValue:@"apple" forHTTPHeaderField:@"ServiceBusNotification-Format"];

			//Authenticate the notification message POST request with the SaS token
			[request setValue:authorizationToken forHTTPHeaderField:@"Authorization"];

			//Add the notification message body
			[request setHTTPBody:[json dataUsingEncoding:NSUTF8StringEncoding]];

			// Send the REST request
		    NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request
				completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
			{
		        NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
		        if (error || (httpResponse.statusCode != 200 && httpResponse.statusCode != 201))
		        {
		            NSLog(@"\nError status: %d\nError: %@", httpResponse.statusCode, error);
		        }
				if (data != NULL)
				{
		        	xmlParser = [[NSXMLParser alloc] initWithData:data];
		        	[xmlParser setDelegate:self];
		       		[xmlParser parse];
		    	}
		    }];
		    [dataTask resume];
		}


9. 在 `ViewController.m` 中，添加以下委派方法，以支持关闭文本字段的键盘。按住 Ctrl 并从文本字段拖到接口设计器中的视图控制器图标，以将视图控制器设为容器委派。

		//===[ Implement UITextFieldDelegate methods ]===

		-(BOOL)textFieldShouldReturn:(UITextField *)textField
		{
			[textField resignFirstResponder];
			return YES;
		}


10. 在 `ViewController.m` 中添加以下委派方法，以支持使用 `NSXMLParser` 分析响应。

		//===[ Implement NSXMLParserDelegate methods ]===

		-(void)parserDidStartDocument:(NSXMLParser *)parser
		{
		    self.statusResult = @"";
		}

		-(void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName
			namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName
			attributes:(NSDictionary *)attributeDict
		{
		    NSString * element = [elementName lowercaseString];
		    NSLog(@"*** New element parsed : %@ ***",element);

		    if ([element isEqualToString:@"code"] | [element isEqualToString:@"detail"])
		    {
		        self.currentElement = element;
		    }
		}

		-(void) parser:(NSXMLParser *)parser foundCharacters:(NSString *)parsedString
		{
		    self.statusResult = [self.statusResult stringByAppendingString:
		        [NSString stringWithFormat:@"%@ : %@\n", self.currentElement, parsedString]];
		}

		-(void)parserDidEndDocument:(NSXMLParser *)parser
		{
			// Set the status label text on the UI thread
			dispatch_async(dispatch_get_main_queue(),
			^{
				[self.sendResults setText:self.statusResult];
			});
		}



11. 生成项目并确认没有错误。


> [AZURE.NOTE] 如果在 Xcode7 中遇到有关 bitcode 支持的生成错误，应该在 Xcode 中将“生成设置”>“启用 Bitcode (ENABLE\_BITCODE)”更改为“否”。通知中心 SDK 目前不支持 Bitcode。

可以在 Apple [本地和推送通知编程指南]中查看所有可能的通知负载。


##检查你的应用能否接收推送通知

若要在 iOS 上测试推送通知，必须将应用部署到物理 iOS 设备。不能使用 iOS 模拟器发送 Apple 推送通知。

1. 运行应用并确认注册成功，然后按“确定”。

	![iOS 应用推送通知注册测试][33]

2. 如上所述，你可以从Azure 门户发送测试推送通知。如果你在应用中添加了发送推送通知的代码，请触摸文本字段以输入通知消息。然后，按键盘上的“发送”按钮或视图中的“发送通知”按钮，以发送通知消息。

	![iOS 应用推送通知发送测试][34]

3. 该推送通知会从特定通知中心发送到所有已注册为接收通知的设备。

	![iOS 应用推送通知接收测试][35]


##后续步骤

在这个简单的示例中，你已将推送通知广播到所有已注册的 iOS 设备。建议你转到 [Azure 通知中心 — 使用 .NET 后端通知 iOS 用户]教程开始下一步的学习，该教程将引导你使用标记创建后端以发送推送通知。

如果要按兴趣组划分用户，可以进一步阅读[使用通知中心发送突发新闻]教程。

有关通知中心的常规信息，请参阅[通知中心指南]。



<!-- Images. -->

[6]: ./media/notification-hubs-ios-get-started/notification-hubs-configure-ios.png
[8]: ./media/notification-hubs-ios-get-started/notification-hubs-create-ios-app.png
[9]: ./media/notification-hubs-ios-get-started/notification-hubs-create-ios-app2.png
[10]: ./media/notification-hubs-ios-get-started/notification-hubs-create-ios-app3.png
[11]: ./media/notification-hubs-ios-get-started/notification-hubs-xcode-product-name.png

[30]: ./media/notification-hubs-ios-get-started/notification-hubs-test-send.png

[31]: ./media/notification-hubs-ios-get-started/notification-hubs-ios-ui.png
[32]: ./media/notification-hubs-ios-get-started/notification-hubs-storyboard-view.png
[33]: ./media/notification-hubs-ios-get-started/notification-hubs-test1.png
[34]: ./media/notification-hubs-ios-get-started/notification-hubs-test2.png
[35]: ./media/notification-hubs-ios-get-started/notification-hubs-test3.png


<!-- URLs. -->
[移动服务 iOS SDK 版本 1.2.4]: http://aka.ms/kymw2g
[移动服务 iOS SDK]: http://go.microsoft.com/fwLink/?LinkID=266533
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253

[Get started with Mobile Services]: /develop/mobile/tutorials/get-started-ios
[Azure 经典管理门户]: https://manage.windowsazure.cn/
[通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
[Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[iOS Provisioning Portal]: http://go.microsoft.com/fwlink/p/?LinkId=272456

[Get started with push notifications in Mobile Services]: /documentation/articles/mobile-services-javascript-backend-ios-get-started-push/
[使用通知中心向用户推送通知]: /documentation/articles/notification-hubs-aspnet-backend-ios-notify-users/
[使用通知中心发送突发新闻]: /documentation/articles/notification-hubs-ios-send-breaking-news/
[本地和推送通知编程指南]: http://developer.apple.com/library/mac/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html#//apple_ref/doc/uid/TP40008194-CH100-SW1
[Azure 门户]: https://portal.azure.cn
<!---HONumber=Mooncake_0405_2016-->