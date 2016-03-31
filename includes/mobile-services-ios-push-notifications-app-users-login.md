
接下来，需要更改注册推送通知的方式，以便在尝试注册之前对用户进行身份验证。

1. 在 **QSAppDelegate.m** 中，将 **didFinishLaunchingWithOptions** 的实现一起删除。

2. 打开 **QSTodoListViewController.m**，在 **viewDidLoad** 方法的末尾添加以下代码：
	
	```
	// Register for remote notifications
	[[UIApplication sharedApplication] registerForRemoteNotificationTypes:
	UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];
	```

<!---HONumber=71-->