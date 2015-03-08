
先前示例显示标准登录，这要求在此应用每次启动时客户端同时联系标识提供商和移动服务。此方法不仅效率低下，而且如果很多客户尝试同时启动您的应用，您会遇到关于使用率的问题。更好的方法是缓存移动服务返回的授权令牌，然后在使用基于提供商的登录之前首先尝试使用此令牌。 


>[WACOM.NOTE] 无论您使用客户端管理的还是服务管理的身份验证，都可以缓存由移动服务颁发的令牌。本教程使用服务管理的身份验证。

1. 加密并将身份验证令牌存储在 iOS 客户端上的推荐方式是使用密钥链。要执行此操作，请创建类 KeychainWrapper，从 [LensRocket 样本](https://github.com/WindowsAzure-Samples/iOS-LensRocket)复制 [KeychainWrapper.m](https://github.com/WindowsAzure-Samples/iOS-LensRocket/blob/master/source/client/LensRocket/Misc/KeychainWrapper.m) 和 [KeychainWrapper.h](https://github.com/WindowsAzure-Samples/iOS-LensRocket/blob/master/source/client/LensRocket/Misc/KeychainWrapper.h)。我们将此 KeychainWrapper 用作 KeychainWrapper，如 Apple 的文档中定义，不考虑自动引用计数 (ARC)。


2. 打开项目文件 **QSTodoListViewController.m** 并添加以下代码：

		
		- (void) saveAuthInfo{
		    [KeychainWrapper createKeychainValue:self.todoService.client.currentUser.userId
				 forIdentifier:@"userid"];
		    [KeychainWrapper createKeychainValue:self.todoService.client.currentUser.mobileServiceAuthenticationToken
				 forIdentifier:@"token"];
		}
		
		
		- (void)loadAuthInfo {
		    NSString *userid = [KeychainWrapper keychainStringFromMatchingIdentifier:@"userid"];
		    if (userid) {
		        NSLog(@"userid: %@", userid);
		        self.todoService.client.currentUser = [[MSUser alloc] initWithUserId:userid];
		        self.todoService.client.currentUser.mobileServiceAuthenticationToken = [KeychainWrapper keychainStringFromMatchingIdentifier:@"token"];
		    }
		}
		


3. 在 **QSTodoListViewController.m** 中 **viewDidAppear** 方法的末尾，添加对 saveAuthInfo 的调用。凭借此调用，我们只需存储用户 ID 和令牌属性。  



		- (void)viewDidAppear:(BOOL)animated
		{
		    MSClient *client = self.todoService.client;
		
		    if (client.currentUser != nil) {
		        return;
		    }
		
		    [client loginWithProvider:@"facebook" controller:self animated:YES completion:^(MSUser *user, NSError *error) {
		
		        [self saveAuthInfo];
		        [self refresh];
		    }];
		}

  
4. 既然我们已发现可以如何缓存用户令牌和 ID，请了解该应用启动时我们可以如何将其加载。使用 **QSTodoListViewController.m** 中的 **viewDidLoad** 方法，在 **self.todoService** 初始化之后，添加对 loadAuthInfo 的调用。 
		
		- (void)viewDidLoad
		{
		    [super viewDidLoad];
		    
		    // Create the todoService - this creates the Mobile Service client inside the wrapped service
		    self.todoService = [QSTodoService defaultService];

			[self loadAuthInfo];
		    
		    // Set the busy method
		    UIActivityIndicatorView *indicator = self.activityIndicator;
		    self.todoService.busyUpdate = ^(BOOL busy)
		    {
		        if (busy)
		        {
		            [indicator startAnimating];
		        } else
		        {
		            [indicator stopAnimating];
		        }
		    };
		    
		    // have refresh control reload all data from server
		    [self.refreshControl addTarget:self
		                            action:@selector(onRefresh:)
		                  forControlEvents:UIControlEventValueChanged];
		
		    // load the data
		    [self refresh];
		}

5. 如果该应用向移动服务提出请求，该请求应该完成，因为该用户经过身份验证且您收到 401 响应（未授权错误），这意味着您传递所用的用户令牌已到期。我们拥有的与移动服务交互的每种方法的完成处理程序中，可以检查 401 响应，或者我们在一个位置处理操作：MSFilter 的 handleRequest 方法。要了解如何处理此场景，请参阅[这篇博客文章](http://www.thejoyofcode.com/Handling_expired_tokens_in_your_application_Day_11_.aspx)

