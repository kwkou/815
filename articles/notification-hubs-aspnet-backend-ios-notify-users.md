<properties title="Azure Notification Hubs Notify Users" pageTitle="Azure 通知中心通知用户" metaKeywords="Azure push notifications, Azure notification hubs" description="了解如何在 Azure 中发送安全推送通知。使用 .NET API 通过 Objective-C 编写的代码示例。" documentationCenter="Mobile" metaCanonical="" disqusComments="1" umbracoNaviHide="0" authors="yuaxu" manager="dwrede" />

<tags ms.service="notification-hubs" ms.date="11/22/2014" wacn.date="08/29/2015" />

# Azure 通知中心通知用户

[AZURE.INCLUDE [notification-hubs-selector-aspnet-backend-notify-users](../includes/notification-hubs-selector-aspnet-backend-notify-users.md)]

## 概述

利用 Azure 中的推送通知支持，您可以访问易于使用且向外扩展的多平台推送基础结构，这大大简化了为移动平台的使用者应用程序和企业应用程序实现推送通知的过程。本教程说明如何使用 Azure 通知中心将推送通知发送到特定设备上的特定应用用户。ASP.NET WebAPI 后端用于对客户端进行身份验证并生成通知，如指南主题[从应用后端注册](http://msdn.microsoft.com/zh-cn/library/dn743807.aspx)中所述。

> [AZURE.NOTE]本教程假设您已根据[通知中心入门 (iOS)](/documentation/articles/notification-hubs-ios-get-started) 中所述创建并配置了通知中心。此外，只有在学习本教程后，才可以学习[安全推送 (iOS)](/documentation/articles/notification-hubs-aspnet-backend-ios-secure-push) 教程。如果您使用移动服务作为后端服务，请参阅本教程的[移动服务版本](/documentation/articles/mobile-services-javascript-backend-ios-push-notifications-app-users)。



[AZURE.INCLUDE [notification-hubs-aspnet-backend-notifyusers](../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## 修改您的 iOS 应用

1. 打开您在[通知中心入门 (iOS)](/documentation/articles/notification-hubs-ios-get-started) 教程中创建的单页视图应用。

> [AZURE.NOTE]在本部分中，我们假设您的项目是使用空组织名称配置的。如果不是，您需要在所有类名称前面预置组织名称。

2. 在您的 Main.storyboard 中，添加对象库中以下屏幕截图中所示的组件。 

    ![][1]
 
	+ **用户名**：紧跟在发送结果标记下方、包含占位符文本*输入用户名*的 UITextField，受发送结果标记下方左右两侧的边距限制。 
	+ **密码**：紧跟在用户名文本字段下方、包含占位符文本*输入密码*的 UITextField，受用户名文本字段下方左右两侧的边距限制。选中“返回密钥”下属性检查器中的“安全文本输入”选项。
	+ **登录**：紧跟在密码文本字段下方标记的 UIButton，取消选中“控件内容”下属性检查器中的“已启用”选项
	+ **WNS**：如果已在中心设置了 WNS，则它就是启用向 Windows 通知服务发送通知的标记和开关。请参阅 [Windows 入门](/documentation/articles/notification-hubs-windows-store-dotnet-get-started)教程。
	+ **GCM**：如果已在中心设置了 GCM，则它就是启用向 Google Cloud Messaging 发送通知的标记和开关。请参阅 [Android 入门](/documentation/articles/notification-hubs-android-get-started)教程。
	+ **APNS**：启用向 Apple 平台通知服务发送通知的标记和开关。
	+ **收件人用户名**：紧跟在 GCM 标记下方、包含占位符文本*收件人用户名标记*的 UITextField，受 GCM 标记下方左右两侧的边距限制。
	

	一些组件已添加到[通知中心入门 (iOS)](/documentation/articles/notification-hubs-ios-get-started) 教程中。

3. 按住 **Ctrl** 键将视图中的组件拖动至 ViewController.h 并添加这些新的出口。

	    @property (weak, nonatomic) IBOutlet UITextField *UsernameField;
		@property (weak, nonatomic) IBOutlet UITextField *PasswordField;
		@property (weak, nonatomic) IBOutlet UITextField *RecipientField;
		@property (weak, nonatomic) IBOutlet UITextField *NotificationField;

		// Used to enable the buttons on the UI
		@property (weak, nonatomic) IBOutlet UIButton *LogInButton;
		@property (weak, nonatomic) IBOutlet UIButton *SendNotificationButton;

		// Used to enabled sending notifications across platforms
		@property (weak, nonatomic) IBOutlet UISwitch *WNSSwitch;
		@property (weak, nonatomic) IBOutlet UISwitch *GCMSwitch;
		@property (weak, nonatomic) IBOutlet UISwitch *APNSSwitch;

		- (IBAction)LogInAction:(id)sender;		

4. 在 ViewController.h 中，在导入语句下方添加以下 `#define`。将 *<输入您的后端终结点>* 占位符替换为您在上一节中用于部署应用后端的目标 URL。例如，**http://you_backend.chinacloudsites.cn*。

		#define BACKEND_ENDPOINT @"<Enter Your Backend Endpoint>"

4. 在您的项目中，新建名为 **RegisterClient** 的 **Cocoa Touch 类**来与您创建的 ASP.NET 后端交互。创建继承自 `NSObject` 的类。然后在 RegisterClient.h 中添加以下代码。

		@interface RegisterClient : NSObject

		@property (strong, nonatomic) NSString* authenticationHeader;
		
		-(void) registerWithDeviceToken:(NSData*)token tags:(NSSet*)tags
			andCompletion:(void(^)(NSError*))completion;

		-(instancetype) initWithEndpoint:(NSString*)Endpoint;

		@end

5. 在 `@interface` 部分的 RegisterClient.m 更新中：

		@interface RegisterClient ()

		@property (strong, nonatomic) NSURLSession* session;
		@property (strong, nonatomic) NSURLSession* endpoint;

		-(void) tryToRegisterWithDeviceToken:(NSData*)token tags:(NSSet*)tags retry:(BOOL)retry 
					andCompletion:(void(^)(NSError*))completion;
		-(void) retrieveOrRequestRegistrationIdWithDeviceToken:(NSString*)token 
					completion:(void(^)(NSString*, NSError*))completion;
		-(void) upsertRegistrationWithRegistrationId:(NSString*)registrationId deviceToken:(NSString*)token 
					tags:(NSSet*)tags andCompletion:(void(^)(NSURLResponse*, NSError*))completion;

		@end

6. 将 RegisterClient.m 中的 `@implementation` 部分替换为以下代码。


		@implementation RegisterClient

		// Globals used by RegisterClient
		NSString *const RegistrationIdLocalStorageKey = @"RegistrationId";		

		-(instancetype) initWithEndpoint:(NSString*)Endpoint
		{
		    self = [super init];
		    if (self) {
		        NSURLSessionConfiguration* config = [NSURLSessionConfiguration defaultSessionConfiguration];
		        _session = [NSURLSession sessionWithConfiguration:config delegate:nil delegateQueue:nil];
				_endpoint = Endpoint;
		    }
		    return self;
		}

		-(void) registerWithDeviceToken:(NSData*)token tags:(NSSet*)tags 
					andCompletion:(void(^)(NSError*))completion
		{
		    [self tryToRegisterWithDeviceToken:token tags:tags retry:YES andCompletion:completion];
		}

		-(void) tryToRegisterWithDeviceToken:(NSData*)token tags:(NSSet*)tags retry:(BOOL)retry 
					andCompletion:(void(^)(NSError*))completion
		{
		    NSSet* tagsSet = tags?tags:[[NSSet alloc] init];

		    NSString *deviceTokenString = [[token description] 
				stringByTrimmingCharactersInSet:[NSCharacterSet characterSetWithCharactersInString:@"<>"]];
		    deviceTokenString = [[deviceTokenString stringByReplacingOccurrencesOfString:@" " withString:@""] 
									uppercaseString];

		    [self retrieveOrRequestRegistrationIdWithDeviceToken: deviceTokenString 
				completion:^(NSString* registrationId, NSError *error) {
		        NSLog(@"regId: %@", registrationId);
		        if (error) {
		            completion(error);
		            return;
		        }

		        [self upsertRegistrationWithRegistrationId:registrationId deviceToken:deviceTokenString 
					tags:tagsSet andCompletion:^(NSURLResponse * response, NSError *error) {
		            if (error) {
		                completion(error);
		                return;
		            }

		            NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*)response;
		            if (httpResponse.statusCode == 200) {
		                completion(nil);
		            } else if (httpResponse.statusCode == 410 && retry) {
		                [self tryToRegisterWithDeviceToken:token tags:tags retry:NO andCompletion:completion];
		            } else {
		                NSLog(@"Registration error with response status: %ld", (long) httpResponse.statusCode);

		                completion([NSError errorWithDomain:@"Registration" code:httpResponse.statusCode 
									userInfo:nil]);
		            }

		        }];
		    }];
		}

		-(void) upsertRegistrationWithRegistrationId:(NSString*)registrationId deviceToken:(NSData*)token 
					tags:(NSSet*)tags andCompletion:(void(^)(NSURLResponse*, NSError*))completion
		{
		    NSDictionary* deviceRegistration = @{@"Platform" : @"apns", @"Handle": token, 
													@"Tags": [tags allObjects]};
		    NSData* jsonData = [NSJSONSerialization dataWithJSONObject:deviceRegistration 
								options:NSJSONWritingPrettyPrinted error:nil];

		    NSLog(@"JSON registration: %@", [[NSString alloc] initWithData:jsonData 
												encoding:NSUTF8StringEncoding]);

		    NSString* endpoint = [NSString stringWithFormat:@"%@/api/register/%@", _endpoint, 
									registrationId];
		    NSURL* requestURL = [NSURL URLWithString:endpoint];
		    NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
		    [request setHTTPMethod:@"PUT"];
		    [request setHTTPBody:jsonData];
		    NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@", 
													self.authenticationHeader];
		    [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];
		    [request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];

		    NSURLSessionDataTask* dataTask = [self.session dataTaskWithRequest:request 
				completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) 
			{
		        if (!error)
		        {
		            completion(response, error);
		        }
		        else
		        {
		            NSLog(@"Error request: %@", error);
		            completion(nil, error);
		        }
		    }];
		    [dataTask resume];
		}

		-(void) retrieveOrRequestRegistrationIdWithDeviceToken:(NSString*)token 
					completion:(void(^)(NSString*, NSError*))completion
		{
		    NSString* registrationId = [[NSUserDefaults standardUserDefaults] 
										objectForKey:RegistrationIdLocalStorageKey];

		    if (registrationId)
		    {
		        completion(registrationId, nil);
		        return;
		    }

		    // request new one & save
		    NSURL* requestURL = [NSURL URLWithString:[NSString stringWithFormat:@"%@/api/register?handle=%@", 
									_endpoint, token]];
		    NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
		    [request setHTTPMethod:@"POST"];
		    NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@", 
													self.authenticationHeader];
		    [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];

		    NSURLSessionDataTask* dataTask = [self.session dataTaskWithRequest:request 
				completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) 
			{
		        NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
		        if (!error && httpResponse.statusCode == 200)
		        {
		            NSString* registrationId = [[NSString alloc] initWithData:data 
						encoding:NSUTF8StringEncoding];

		            // remove quotes
		            registrationId = [registrationId substringWithRange:NSMakeRange(1, 
										[registrationId length]-2)];

		            [[NSUserDefaults standardUserDefaults] setObject:registrationId 
						forKey:RegistrationIdLocalStorageKey];
		            [[NSUserDefaults standardUserDefaults] synchronize];

		            completion(registrationId, nil);
		        }
		        else
		        {
		            NSLog(@"Error status: %ld, request: %@", (long)httpResponse.statusCode, error);
		            if (error)
		                completion(nil, error);
		            else {
		                completion(nil, [NSError errorWithDomain:@"Registration" code:httpResponse.statusCode 
									userInfo:nil]);
		            }
		        }
		    }];
		    [dataTask resume];
		}

		@end

	上面的代码通过使用 NSURLSession 对应用后端执行 REST 调用，并使用 NSUserDefaults 在本地存储通知中心返回的 registrationId，来实现指南文章[从应用后端进行注册](http://msdn.microsoft.com/zh-cn/library/dn743807.aspx)中所述的逻辑。

	请注意，此类需要设置其属性 **authorizationHeader** 才能正常运行。登录后，由 **ViewController** 类设置此属性。

7. 在 ViewController.h 中，为 RegisterClient.h 添加一个 `#import` 语句。然后，添加设备令牌的声明，并引用 `@interface` 部分 中的 `RegisterClient` 实例：

		#import "RegisterClient.h"

		@property (strong, nonatomic) NSData* deviceToken;
		@property (strong, nonatomic) RegisterClient* registerClient;

8. 在 ViewController.m 中，在 `@interface` 部分添加私有方法声明：

		@interface ViewController () <UITextFieldDelegate, NSURLConnectionDataDelegate, NSXMLParserDelegate>

		// create the Authorization header to perform Basic authentication with your app back-end
		-(void) createAndSetAuthenticationHeaderWithUsername:(NSString*)username 
						AndPassword:(NSString*)password;

		@end

> [AZURE.NOTE]下面的代码段不是安全的身份验证方案，您应该将 **createAndSetAuthenticationHeaderWithUsername:AndPassword:** 的实现替换为生成供注册客户端类（例如 OAuth、Active Directory）使用的身份验证令牌的特定身份验证机制。

9. 然后在 ViewController.m 的 `@implementation` 部分中，添加以下代码，此代码将添加设置设备令牌和身份验证标头的实现。 

		-(void) setDeviceToken: (NSData*) deviceToken
		{
		    _deviceToken = deviceToken;
		    self.LogInButton.enabled = YES;
		}

		-(void) createAndSetAuthenticationHeaderWithUsername:(NSString*)username 
						AndPassword:(NSString*)password;
		{
		    NSString* headerValue = [NSString stringWithFormat:@"%@:%@", username, password];

		    NSData* encodedData = [[headerValue dataUsingEncoding:NSUTF8StringEncoding] base64EncodedDataWithOptions:NSDataBase64EncodingEndLineWithCarriageReturn];

		    self.registerClient.authenticationHeader = [[NSString alloc] initWithData:encodedData 
														encoding:NSUTF8StringEncoding];
		}

		-(BOOL)textFieldShouldReturn:(UITextField *)textField
		{
		    [textField resignFirstResponder];
		    return YES;
		}

	请注意设置设备令牌如何能够启用登录按钮。这是因为作为登录操作的一部分，视图控制器通过应用后端注册推送通知。因此，我们希望正确设置了设备令牌后，再访问登录操作。如果登录早于设置设备令牌，您可以将登录与推送注册分离。

10. 在 ViewController.m 中，使用下面的代码段来实现“登录”按钮的操作方法和一个使用 ASP.NET 后端发送通知消息的方法。

		- (IBAction)LogInAction:(id)sender {
		    // create authentication header and set it in register client
		    NSString* username = self.UsernameField.text;
		    NSString* password = self.PasswordField.text;

		    [self createAndSetAuthenticationHeaderWithUsername:username AndPassword:password];

		    __weak ViewController* selfie = self;
		    [self.registerClient registerWithDeviceToken:self.deviceToken tags:nil 
				andCompletion:^(NSError* error) {
		        if (!error) {
		            dispatch_async(dispatch_get_main_queue(), 
					^{
		                selfie.SendNotificationButton.enabled = YES;
		                [self MessageBox:@"Success" message:@"Registered successfully!"];
		            });
		        }
		    }];
		}


		- (void)SendNotificationASPNETBackend:(NSString*)pns UsernameTag:(NSString*)usernameTag 
					Message:(NSString*)message
		{
		    NSURLSession* session = [NSURLSession
		    	sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:nil
		        delegateQueue:nil];
		    
			// Pass the pns and username tag as parameters with the REST URL to the ASP.NET backend
		    NSURL* requestURL = [NSURL URLWithString:[NSString 
				stringWithFormat:@"%@/api/notifications?pns=%@&to_tag=%@", BACKEND_ENDPOINT, pns, 
				usernameTag]];

		    NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
		    [request setHTTPMethod:@"POST"];

			// Get the mock authenticationheader from the register client
		    NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@", 
				self.registerClient.authenticationHeader];
		    [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];
		    
		    //Add the notification message body
		    [request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];
		    [request setHTTPBody:[message dataUsingEncoding:NSUTF8StringEncoding]];
		    
			// Execute the send notification REST API on the ASP.NET Backend
		    NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request 
				completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) 
			{
		        NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
		        if (error || httpResponse.statusCode != 200)
		        {
		            NSString* status = [NSString stringWithFormat:@"Error Status for %@: %d\nError: %@\n", 
										pns, httpResponse.statusCode, error];
		            dispatch_async(dispatch_get_main_queue(),
		            ^{
						// Append text because all 3 PNS calls may also have information to view
		                [self.sendResults setText:[self.sendResults.text stringByAppendingString:status]];
		            });
		            NSLog(status);
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


11. 更新“发送通知”按钮的操作以使用 ASP.NET 后端，并发送到由开关启用的任何 PNS。


		- (IBAction)SendNotificationMessage:(id)sender
		{
		    //[self SendNotificationRESTAPI];
		    [self SendToEnabledPlatforms];
		}


		-(void)SendToEnabledPlatforms
		{
		    NSString* json = [NSString stringWithFormat:@"\"%@\"",self.notificationMessage.text];
		    
			[self.sendResults setText:@""];

		    if ([self.WNSSwitch isOn])
		        [self SendNotificationASPNETBackend:@"wns" UsernameTag:self.RecipientField.text Message:json];
		
		    if ([self.GCMSwitch isOn])
		        [self SendNotificationASPNETBackend:@"gcm" UsernameTag:self.RecipientField.text Message:json];
		
		    if ([self.APNSSwitch isOn])
		        [self SendNotificationASPNETBackend:@"apns" UsernameTag:self.RecipientField.text Message:json];
		}
		
		

11. 在函数 **ViewDidLoad** 中，添加以下内容来实例化 RegisterClient 实例并设置文本字段的委托。

		self.UsernameField.delegate = self;
		self.PasswordField.delegate = self;
		self.RecipientField.delegate = self;
		self.registerClient = [[RegisterClient alloc] initWithEndpoint:BACKEND_ENDPOINT];

12. 现在，在 **AppDelegate.m** 中，删除方法 **application:didRegisterForPushNotificationWithDeviceToken:** 的所有内容，并将其替换为以下内容，以确保视图控制器包含从 APN 中检索到的最新设备令牌:

		// Add import to the top of the file
		#import "ViewController.h"

	    - (void)application:(UIApplication *)application 
	    			didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken 
	    {
		    ViewController* rvc = (ViewController*) self.window.rootViewController;
		    rvc.deviceToken = deviceToken;
		}

13. 最后，在 **AppDelegate.m** 中，请确保您具有以下方法：

		- (void)application:(UIApplication *)application didReceiveRemoteNotification: (NSDictionary *)userInfo {
		    NSLog(@"%@", userInfo);
		    [self MessageBox:@"Notification" message:[[userInfo objectForKey:@"aps"] valueForKey:@"alert"]];
		}

## 测试应用程序

1. 在 XCode 中，在物理 iOS 设备上运行此应用（推送通知将无法在模拟器中正常工作）。

2. 在 iOS 应用 UI 中，输入用户名和密码。这些信息可以是任意字符串，但它们必须都是相同的字符串值。然后，单击“登录”。

	![][2]


3. 您应该可以看到弹出窗口，通知您注册成功。单击“确定”。

	![][3]

4. 在**收件人用户名标记*文本字段中，输入用于从另一台设备进行注册的用户名标记。
5. 输入一条通知消息，然后单击“发送通知”。只有使用此收件人用户名标记进行注册的设备才能接收此通知消息。此通知消息将只发送给上述用户。

	![][4]


[1]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-interface.png
[2]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-enter-user-pwd.png
[3]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-registered.png
[4]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-enter-msg.png
 

<!---HONumber=67-->