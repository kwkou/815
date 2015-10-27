
在每次启动该应用程序时，前面的示例既联系标识提供程序又联系移动服务。相反，你可以缓存授权令牌并首先尝试使用它。

* 加密并将身份验证令牌存储在 iOS 客户端上的推荐方式是使用 iOS 密钥链。我们将使用 [SSKeychain](https://github.com/soffes/sskeychain)，这是一种围绕 iOS 密钥链的简单包装器。按照 SSKeychain 页上的说明操作并将其添加到你的项目。验证在项目的**生成设置**中已启用**启用模块**设置（**Apple LLVM - 语言 - 模块**部分。）

* 打开 **QSTodoListViewController.m** 并添加以下代码：

```
		- (void) saveAuthInfo {
				[SSKeychain setPassword:self.todoService.client.currentUser.mobileServiceAuthenticationToken forService:@"AzureMobileServiceTutorial" account:self.todoService.client.currentUser.userId]
		}


		- (void)loadAuthInfo {
				NSString *userid = [[SSKeychain accountsForService:@"AzureMobileServiceTutorial"][0] valueForKey:@"acct"];
		    if (userid) {
		        NSLog(@"userid: %@", userid);
		        self.todoService.client.currentUser = [[MSUser alloc] initWithUserId:userid];
		         self.todoService.client.currentUser.mobileServiceAuthenticationToken = [SSKeychain passwordForService:@"AzureMobileServiceTutorial" account:userid];

		    }
		}
```

* 在 `loginAndGetData` 中，修改 `loginWithProvider:controller:animated:completion:` 的 completion 块。就在 `[self refresh]` 的前面添加以下行来存储用户 ID 和标记属性：

```
				[self saveAuthInfo];
```

* 在该应用程序启动时加载用户 ID 和令牌。在 **QSTodoListViewController.m** 中的 `viewDidLoad` 中，就在 `self.todoService` 初始化后添加此项。

```
				[self loadAuthInfo];
```

<!---HONumber=74-->