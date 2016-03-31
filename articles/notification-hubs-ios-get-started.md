<properties
	pageTitle="Azure 通知中心入门（iOS 应用）| Azure"
	description="在本教程中，你将了解如何使用 Azure 通知中心将通知推送到 iOS 应用程序。"
	services="notification-hubs"
	documentationCenter="ios"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="notification-hubs"
	ms.date="12/15/2015"
	wacn.date="01/14/2016"/>

# 通知中心入门（iOS 应用）

[AZURE.INCLUDE [notification-hubs-selector-get-started](../includes/notification-hubs-selector-get-started.md)]

##概述

本教程演示如何使用 Azure 通知中心将推送通知发送到 iOS 应用程序。你将创建一个空白 iOS 应用，它使用 Apple 推送通知服务 (APNS) 接收推送通知。完成后，你将能够使用通知中心将推送通知广播到运行你的应用的所有设备。

本教程演示使用通知中心的简单广播方案。


## 开始之前

[AZURE.INCLUDE [notification-hubs-hero-slug](../includes/notification-hubs-hero-slug.md)]

可以在 GitHub 上的[此处](https://github.com/Azure/azure-notificationhubs-samples/tree/master/iOS/GetStartedNH/GetStarted)找到本教程的已完成代码。

##先决条件

本教程需要的内容如下：

+ [移动服务 iOS SDK]
+ [XCode 6][Install Xcode]
+ 支持 iOS 8（或更高版本）的设备
+ iOS 开发人员计划成员身份

   >[AZURE.NOTE]由于推送通知配置要求，你必须在支持 iOS 的设备（iPhone 或 iPad）而不是在模拟器上部署和测试推送通知。

完成本教程是学习有关 iOS 应用的所有其他通知中心教程的先决条件。

> [AZURE.NOTE]若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用](/pricing/free-trial)。

[AZURE.INCLUDE [通知中心：启用 Apple 推送通知](../includes/enable-apple-push-notifications.md)]

##<a name="configure-hub"></a>配置通知中心

本部分将引导你使用创建的 **.p12** 推送证书创建新的通知中心并配置 APNS 身份验证。如果你想要使用已创建的通知中心，可以跳到步骤 5。

[AZURE.INCLUDE [notification-hubs-portal-create-new-hub](../includes/notification-hubs-portal-create-new-hub.md)]

<ol start="7">
<li>
<p>单击顶部的“配置”选项卡，然后单击 Apple 通知设置中的“上载”按钮，以上载证书指纹。<b></b><b></b>然后选择前面导出的 <b>.p12</b> 证书以及证书的密码。</p>
<p>由于此项目用于开发，因此请务必选择“沙箱”模式。<b></b>仅当你想要将推送通知发送给从应用商店购买你应用的用户时，才使用“生产”模式。<b></b></p>
</li>
</ol>
&emsp;&emsp;![](./media/notification-hubs-ios-get-started/notification-hubs-upload-cert.png)

&emsp;&emsp;![](./media/notification-hubs-ios-get-started/notification-hubs-configure-ios.png)



你的通知中心现在已配置为使用 APNS，并且你有连接字符串用于注册你的应用程序和发送推送通知。

##将你的应用程序连接到通知中心

1. 在 XCode 中，创建新的 iOS 项目，然后选择“单视图应用程序”模板。

   	![][8]

2. 设置新项目的选项时，请务必使用前面在 Apple 开发人员门户上设置捆绑 ID 时使用的相同**产品名称**和**组织标识符**。

	![][11]

3. 在“目标”下，单击你的项目名称，单击“生成设置”选项卡，展开“代码签名标识”，然后在“调试”下设置你的**代码签名标识**。将“级别”从“基本”切换为“全部”，然后将“预配配置文件”设为前面创建的预配配置文件。

	如果屏幕未显示在 Xcode 中创建的新预配配置文件，请尝试刷新签名标识的配置文件，方法是单击菜单栏上的“Xcode”，再依次单击“首选项”、“帐户”选项卡、“查看详细信息”按钮、你的签名标识，然后单击右下角的刷新按钮。

   	![][9]

4. 下载[移动服务 iOS SDK 版本 1.2.4]，然后将文件解压缩。在 Xcode 中，右键单击你的项目，然后单击“将文件添加到”选项，将 **WindowsAzureMessaging.framework** 文件夹添加到 Xcode 项目。选择“需要时复制项”，然后单击“添加”。

	>[AZURE.NOTE]通知中心 SDK 目前不支持 Xcode 7 上的 Bitcode。你必须在项目的“构建选项”中将“启用 Bitcode”设置为“否”。

   	![][10]

5. 将新的标头文件添加到名为 **HubInfo.h** 的项目。此文件将保存通知中心的常量。添加以下定义，然后将字符串文本占位符替换为你的*中心名称*以及前面记下的 *DefaultListenSharedAccessSignature*。

		#ifndef HubInfo_h
		#define HubInfo_h
		
			#define HUBNAME @"<Enter the name of your hub>"
			#define HUBLISTENACCESS @"<Enter your DefaultListenSharedAccess connection string"
		
		#endif /* HubInfo_h */

6. 打开 AppDelegate.h 文件并添加以下导入指令：

         #import <WindowsAzureMessaging/WindowsAzureMessaging.h> 
		 #import "HubInfo.h"
		
7. 根据你的 iOS 版本，在 AppDelegate.m 文件的 `didFinishLaunchingWithOptions` 方法中添加以下代码。此代码将向 APNs 注册设备句柄：

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

## 发送通知


在 [Azure 管理门户]中通过通知中心上的调试选项卡（如以下屏幕中所示）来发送通知，可以在应用中测试通知的接收情况。

![][30]

[AZURE.INCLUDE [notification-hubs-sending-notifications-from-the-portal](../includes/notification-hubs-sending-notifications-from-the-portal.md)]

![][31]

## （可选）从应用发送通知

假设你要在应用内发送通知。本部分提供一个示例来演示如何使用 REST 接口执行此操作。

1. 在 Xcode 中，打开 Main.storyboard 并从对象库添加以下 UI 组件，以允许用户在应用中发送推送通知。

	- 没有标签文本的标签。该标签用于报告发送通知时发生的错误。**Lines** 属性应该设为 **0**，以便自动根据左右边距和视图顶部的限制来调整大小。
	- 具有**占位符**文本的文本字段设为“输入通知消息”。将字段限制在标签的正下方，如下所示。将视图控制器设为容器委派。
	- 标题为“发送通知”的按钮限制在文本字段的正下方，并在水平居中的位置。

	该视图应如下所示：

	![][32]


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

3. 打开 HubInfo.h 并添加以下常量，用于向中心发送通知。将占位符字符串文本替换为实际的 *DefaultFullSharedAccessSignature* 连接字符串。

		#define API_VERSION @"?api-version=2015-01"
		#define HUBFULLACCESS @"<Enter Your DefaultFullSharedAccess Connection string>"

4. 将以下 `#import` 语句添加到 ViewController.h 文件。

		#import <CommonCrypto/CommonHMAC.h>
		#import "HubInfo.h"

5. 在 ViewController.m 中，将以下代码添加到接口实现。此代码将会分析你的 *DefaultFullSharedAccessSignature* 连接字符串。如 [REST API 参考](http://msdn.microsoft.com/library/azure/dn495627.aspx)中所述，此类已分析的信息将用于生成 **Authorization** 请求标头的 SaS 令牌。

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

6. 在 ViewController.m 中更新 `viewDidLoad` 方法，以便在加载视图时分析连接字符串。此外，将如下所示的实用程序方法添加到接口实现。


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





7. 如 [REST API 参考](http://msdn.microsoft.com/library/azure/dn495627.aspx)中所述，在 ViewController.m 中，将以下代码添加到接口实现，以生成将在 **Authorization** 标头中提供的 SaS 授权令牌。

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


8. 按住 Ctrl 并从“发送通知”按钮拖到 ViewController.m，为 **Touch Down** 添加名为 **SendNotificationMessage** 的操作。使用以下代码更新方法，以使用 REST API 发送通知。

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

			//Create the request to add the APNS notification message to the hub
			NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
			[request setHTTPMethod:@"POST"];
			[request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];

			// Signify apple notification format
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


9. 在 ViewController.m 中，添加以下委派方法，以支持关闭文本字段的键盘。按住 Ctrl 并从文本字段拖到接口设计器中的视图控制器图标，以将视图控制器设为容器委派。

		//===[ Implement UITextFieldDelegate methods ]===

		-(BOOL)textFieldShouldReturn:(UITextField *)textField
		{
			[textField resignFirstResponder];
			return YES;
		}


10. 在 ViewController.m 中添加以下委派方法，以支持使用 `NSXMLParser` 分析响应。

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


> [AZURE.NOTE]如果在 Xcode7 中遇到有关 Bitcode 支持的生成错误，应该在 Xcode 中将“生成设置”->“启用 Bitcode”(ENABLE\_BITCODE) 更改为“否”。通知中心 SDK 目前不支持 Bitcode。

可以在 Apple [本地和推送通知编程指南]中查看所有可能的通知负载。



##测试应用程序

若要在 iOS 上测试推送通知，必须将应用部署到设备。不能使用 iOS 模拟器发送 Apple 推送通知。

1. 运行应用并确认注册成功，然后按“确定”。

	![][33]

2. 你可以从 [Azure 管理门户]发送测试通知。如果你在应用中添加了发送通知的代码，请触摸文本字段以输入通知消息。然后，按键盘上的“发送”按钮或视图中的“发送通知”按钮，以发送通知消息。

	![][34]

3. 该通知将发送到已注册接收通知的所有设备。

	![][35]

如果你有任何疑问或者想要提出建议以便为所有读者改进本教程，请在下面的 Disqus 部分为我们留言。


##后续步骤

在这个简单的示例中，你已将通知广播到所有 iOS 设备。我们建议你继续学习[使用通知中心向用户推送通知]教程。该教程将逐步引导你创建一个后端，以使用标记发送通知。

如果要按兴趣组划分用户，可以进一步阅读[使用通知中心发送突发新闻]。

有关通知中心的更多常规信息，请参阅[通知中心指南]。



<!-- Images. -->

[6]: ./media/notification-hubs-ios-get-started/notification-hubs-configure-ios.png
[8]: ./media/notification-hubs-ios-get-started/notification-hubs-create-ios-app.png
[9]: ./media/notification-hubs-ios-get-started/notification-hubs-create-ios-app2.png
[10]: ./media/notification-hubs-ios-get-started/notification-hubs-create-ios-app3.png
[11]: ./media/notification-hubs-ios-get-started/notification-hubs-xcode-product-name.png

[30]: ./media/notification-hubs-ios-get-started/notification-hubs-debug-hub-ios.png

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
[Live SDK for Windows]: https://msdn.microsoft.com/zh-cn/onedrive/dn632140

[Get started with Mobile Services]: /develop/mobile/tutorials/get-started-ios
[Azure 管理门户]: https://manage.windowsazure.cn/
[通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
[Install Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[iOS Provisioning Portal]: http://go.microsoft.com/fwlink/p/?LinkId=272456

[Get started with push notifications in Mobile Services]: /documentation/articles/mobile-services-javascript-backend-ios-get-started-push
[使用通知中心向用户推送通知]: /documentation/articles/notification-hubs-aspnet-backend-ios-notify-users
[使用通知中心发送突发新闻]: /documentation/articles/notification-hubs-ios-send-breaking-news
[本地和推送通知编程指南]: http://developer.apple.com/library/mac/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html#//apple_ref/doc/uid/TP40008194-CH100-SW1

<!---HONumber=Mooncake_0104_2016-->