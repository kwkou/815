<properties 
	pageTitle="使用移动服务注册推送通知的当前用户 | Azure" 
	description="了解在 Azure 移动服务执行注册时如何在 iOS 应用程序中请求向 Azure 通知中心注册推送通知。" 
	services="notification-hubs" 
	documentationCenter="ios" 
	authors="ysxu" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="11/01/2015" 
	wacn.date="05/31/2016"/>

# 通过使用移动服务注册推送通知的当前用户

> [AZURE.SELECTOR]
[Windows Store C#](/documentation/articles/notification-hubs-ios-mobile-services-register-user-push-notifications/)
[iOS](/documentation/articles/notification-hubs-ios-aspnet-register-user-push-notifications/)

本主题说明在 Azure 移动服务执行注册时如何请求向 Azure 通知中心注册推送通知。它是对教程[使用通知中心通知用户][使用通知中心通知用户]的扩展。你必须在该教程中已完成创建经过身份验证的移动服务所需的步骤。有关通知用户方案的详细信息，请参阅[使用通知中心通知用户][使用通知中心通知用户]。

1.  在 Xcode 的项目中打开 QSTodoService.h 文件（该项目是你在完成基础教程[身份验证入门][身份验证入门]时创建的）并添加以下 **deviceToken** 属性：

		@property (nonatomic) NSData* deviceToken;

 	此属性存储设备标记。

2. 在 QSTodoService.m 文件中，添加以下 **getDeviceTokenInHex** 方法：

			- (NSString*)getDeviceTokenInHex {
			    const unsigned *tokenBytes = [[self deviceToken] bytes];
			    NSString *hexToken = [NSString stringWithFormat:@"%08X%08X%08X%08X%08X%08X%08X%08X",
			                          ntohl(tokenBytes[0]), ntohl(tokenBytes[1]), ntohl(tokenBytes[2]),
			                          ntohl(tokenBytes[3]), ntohl(tokenBytes[4]), ntohl(tokenBytes[5]),
			                          ntohl(tokenBytes[6]), ntohl(tokenBytes[7])];
			    return hexToken;
			}

	此方法将设备标记转换为十六进制字符串值。

3. 在 QSAppDelegate.m 文件中，将以下代码行添加到 **didFinishLaunchingWithOptions** 方法：

			[[UIApplication sharedApplication] registerForRemoteNotificationTypes: UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];

	这在你的应用程序中启用推送通知。

4. 	在 QSAppDelegate.m 文件中，添加以下方法：

			- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
			    [QSTodoService defaultService].deviceToken = deviceToken;
			}

	这会更新 **deviceToken** 属性。

	> [AZURE.NOTE]此时，此方法中不应有任何其他代码。如果你已调用在完成[通知中心入门](/documentation/articles/notification-hubs-ios-get-started/)教程的学习时添加的 **registerNativeWithDeviceToken** 方法，必须注释掉或删除该调用。

5.  （可选）在 QSAppDelegate.m 文件中，添加以下处理程序方法：

			- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
			    NSLog(@"%@", userInfo);
			    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
			                          [[userInfo objectForKey:@"aps"] valueForKey:@"alert"] delegate:nil cancelButtonTitle:
			                          @"OK" otherButtonTitles:nil, nil];
			    [alert show];
			}

    当你的应用程序接收到它正在运行的通知时，此方法将在 UI 中显示一个警报。

6.  在 QSTodoListViewController.m 文件中，添加 **registerForNotificationsWithBackEnd** 方法：

			- (void)registerForNotificationsWithBackEnd {    
			    NSString* json = [NSString  stringWithFormat:@"{"platform":"ios", "deviceToken":"%@"}", [self.todoService getDeviceTokenInHex] ];
			    
			    [self.todoService.client invokeAPI:@"register_notifications" data:[json dataUsingEncoding:NSUTF8StringEncoding] HTTPMethod:@"POST" parameters:nil headers:nil completion:^(id result, NSHTTPURLResponse *response, NSError *error) {
			        if (error != nil) {
			            NSLog(@"Registration failed: %@", error);
			        } else {
			            // display UIAlert with successful login
			            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Back-end registration" message:@"Registration successful" delegate:nil cancelButtonTitle: @"OK" otherButtonTitles:nil, nil];
			            [alert show];
			        }
			    }];
			}

    此方法构造包含设备标记的 json 负载。它然后在你的移动服务中调用自定义 API 以注册通知。此方法为推送通知创建一个设备标记并将它与设备类型一起发送到在通知中心创建注册的自定义 API 方法。此自定义 API 已在[使用通知中心通知用户][使用通知中心通知用户]中定义。

7.	最后，在 **viewDidAppear** 方法中，在用户成功进行身份验证后添加对这个新 **registerForNotificationsWithBackEnd** 方法的调用，如下例中所示：
	
			- (void)viewDidAppear:(BOOL)animated
			{
			    MSClient *client = self.todoService.client;
			
			    if (client.currentUser != nil) {
			        return;
			    }
			    
			    [client loginWithProvider:@"microsoftaccount" controller:self animated:YES completion:^(MSUser *user, NSError *error) {
			        [self refresh];
			        [self registerForNotificationsWithBackEnd];
			    }];
			}

	> [AZURE.NOTE]这可以确保每次加载页时都会请求注册。在应用程序中，你可能只需要定期执行此注册以确保注册是最新的。
	
现在客户端应用程序已更新，请返回到[使用通知中心通知用户]并更新移动服务以使用通知中心发送通知。

<!-- Anchors. --> 

<!-- Images. --> 

<!-- URLs. -->
[使用通知中心通知用户]: /documentation/articles/notification-hubs-aspnet-backend-ios-notify-users/
[身份验证入门]:/documentation/articles/mobile-services-dotnet-backend-ios-get-started-users/

[Azure Management Portal]: https://manage.windowsazure.cn/
[Get Started with Notification Hubs]: /documentation/articles/notification-hubs-ios-get-started/

<!---HONumber=Mooncake_0405_2016-->