<properties
	pageTitle="Azure AD v2.0 iOS 应用 | Azure"
	description="如何通过第三方库生成一个使用个人 Microsoft 帐户和工作或学校帐户来登录用户的 iOS 应用。"
	services="active-directory"
	documentationCenter=""
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>  

<tags
	ms.service="active-directory"
	ms.date="06/28/2016"
	wacn.date="08/22/2016"/>  


# 使用 v2.0 终结点，通过图形 API 将登录添加到使用第三方库的 iOS 应用

Microsoft 标识平台使用开放式标准，例如 OAuth2 和 OpenID Connect。开发人员可以使用任何想要的库来与我们的服务集成。为了帮助开发人员将我们的平台与其他库结合使用，我们撰写了数篇演练（例如本演练），演示如何配置第三方库以连接到 Microsoft 标识平台。大部分实施 [RFC6749 OAuth2 规范](https://tools.ietf.org/html/rfc6749)的库都能连接到 Microsoft 标识平台。

借助本演练创建的应用程序，用户可以使用图形 API 登录到其组织，然后在组织中搜索其他人。

如果你是 OAuth2 或 OpenID Connect 新手，此示例配置可能不太适合你。建议你阅读 [v2.0 协议 — OAuth 2.0 授权代码流](/documentation/articles/active-directory-v2-protocols-oauth-code/)了解背景信息。


> [AZURE.NOTE]
    我们的平台中有些功能（例如条件性访问和 Intune 策略管理）采用 OAuth2 或 OpenID Connect 标准中的表达式，所以会要求你使用开放源代码 Microsoft Azure 标识库。

v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。

> [AZURE.NOTE]
    若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

## 从 GitHub 下载代码
本教程的代码[在 GitHub 上](https://github.com/Azure-Samples/active-directory-ios-native-nxoauth2-v2)维护。若要遵照该代码，你可以[下载 .zip 格式应用骨架](https://github.com/AzureADQuickStarts/AppModelv2-WebAPI-DotNet/archive/skeleton.zip)，或克隆该骨架：


	git clone --branch skeleton git@github.com:Azure-Samples/active-directory-ios-native-nxoauth2-v2.git


你也可以下载以下示例，并立即开始使用：


	git clone git@github.com:Azure-Samples/active-directory-ios-native-nxoauth2-v2.git


## 注册应用程序
在[应用程序注册门户](https://apps.dev.microsoft.com)创建新的应用，或按照[如何使用 v2.0 终结点注册应用](/documentation/articles/active-directory-v2-app-registration/)中的详细步骤操作。请确保：

- 复制分配给应用的“应用程序 ID”，因为稍后将要用到。
- 为应用添加**移动**平台。
- 从门户复制**重定向 URI**。必须使用默认值 `urn:ietf:wg:oauth:2.0:oob`。


## 下载 NXOAuth2 第三方库并创建工作区

在本演练中，你将使用 GitHub 提供的 OAuth2Client，这是适用于 Mac OS X 和 iOS 的 OAuth2 库（Cocoa 和 Cocoa Touch）。此库以 OAuth2 规范的第 10 版草稿为基础。它将实现本机应用程序配置文件，并支持用户的授权终结点。你需要上述各项，才能与 Microsoft 标识平台集成。

### 使用 CocoaPods 将库添加到项目

CocoaPods 是 Xcode 项目的依赖关系管理器。它会自动管理上述安装步骤。


	$ vi Podfile

1. 将以下内容添加到 podfile：

		
		platform :ios, '8.0'
	
		target 'QuickStart' do
	
		pod 'NXOAuth2Client'
	
		end
		

2. 使用 CocoaPods 加载 podfile。这会创建你要加载的新 Xcode 工作区。

		
		$ pod install
		...
		$ open QuickStart.xcworkspace
		

## 浏览项目结构

在主干中为项目设置以下结构：

- 具有 UPN 搜索功能的主视图
- 所选用户的相关数据的详细信息视图
- 用户可登录到应用来查询图形的登录视图

我们将移至主干中的各种文件，以添加身份验证。代码的其他部分（如可视代码）与标识无关，所以不会提供给你。

## 设置库中的 settings.plst 文件

-	在快速入门项目中，打开 `settings.plist` 文件。替换节中的元素值，以反映你在 Azure 门户中使用的值。每当使用 Active Directory 身份验证库时，你的代码就会参考这些值。
    -	`clientId` 是从门户复制的应用程序的客户端 ID。
    -	`redirectUri` 是门户提供的重定向 URL。

## 在 LoginViewController 中设置 NXOAuth2Client 库

NXOAuth2Client 库要求设置一些值。完成该任务之后，你可以使用所获令牌来调用图形 API。因为每当需要进行身份验证时都会调用 `LoginView`，所以合理的做法是将配置值放入该文件中。

- 让我们在 `LoginViewController.m` 文件中添加一些值，以便针对身份验证和授权来设置上下文。紧跟代码后面的是关于这些值的详细信息。

objc

	NSString *scopes = @"offline_access User.ReadBasic.All";
	NSString *authURL = @"https://login.microsoftonline.com/common/oauth2/v2.0/authorize";
	NSString *loginURL = @"https://login.microsoftonline.com/common/login";
	NSString *bhh = @"urn:ietf:wg:oauth:2.0:oob?code=";
	NSString *tokenURL = @"https://login.microsoftonline.com/common/oauth2/v2.0/token";
	NSString *keychain = @"com.microsoft.azureactivedirectory.samples.graph.QuickStart";
	static NSString * const kIDMOAuth2SuccessPagePrefix = @"session_state=";
	NSURL *myRequestedUrl;
	NSURL *myLoadedUrl;
	bool loginFlow = FALSE;
	bool isRequestBusy;
	NSURL *authcode;
	

让我们看看关于代码的详细信息。

第一个字符串用于 `scopes`。`User.ReadBasic.All` 值可让你读取目录中所有用户的基本个人资料。

你可以在 [Microsoft Graph 权限范围](https://graph.microsoft.io/docs/authorization/permission_scopes)中了解有关所有可用范围的详细信息。

对于 `authURL`、`loginURL`、`bhh` 及 `tokenURL`，你应该使用上面提供的值。如果你使用开放源代码 Microsoft Azure 标识库，我们会使用元数据终结点为你拉取此数据。我们已努力完成为你提取这些值的工作。

`keychain` 值是一个容器，NXOAuth2Client 库将用来创建密钥链以存储你的令牌。如果你想要获取跨多个应用的单一登录 (SSO)，可以在每个应用程序中指定相同的密钥链，并请求在 Xcode 权利中使用该密钥链。这会在 Apple 文档中说明。

这些值的其余部分都必须使用此库并且为你创建位置，才能将这些值送至上下文。

### 创建 URL 缓存

在加载视图之后，始终会调用 `(void)viewDidLoad()`，其内部的以下代码会准备好缓存以供我们使用。

添加以下代码：

objc
	
	- (void)viewDidLoad {
	    [super viewDidLoad];
	    self.loginView.delegate = self;
	    [self setupOAuth2AccountStore];
	    [self requestOAuth2Access];
	    NSURLCache *URLCache = [[NSURLCache alloc] initWithMemoryCapacity:4 * 1024 * 1024
	                                                         diskCapacity:20 * 1024 * 1024
	                                                             diskPath:nil];
	    [NSURLCache setSharedURLCache:URLCache];
	
	}


### 创建用于登录的 Web 视图

Web 视图可提示用户提供短信等附加因素（如果已配置）或向用户返回错误消息。你将在此处设置 Web 视图，然后编写代码，以从标识服务处理将会在 Web 视图中发生的回叫。

objc
	
	-(void)requestOAuth2Access {
	    //to sign in to Microsoft APIs using OAuth2, we must show an embedded browser (UIWebView)
	    [[NXOAuth2AccountStore sharedStore] requestAccessToAccountWithType:@"myGraphService"
	                                   withPreparedAuthorizationURLHandler:^(NSURL *preparedURL) {
	                                       //navigate to the URL returned by NXOAuth2Client
	
	                                       NSURLRequest *r = [NSURLRequest requestWithURL:preparedURL];
	                                       [self.loginView loadRequest:r];
	                                   }];
	}
	

### 重写 Web 视图方法以处理身份验证

如先前所述，当用户需要登录时，若要告诉 Web 视图发生了什么情况，你可以粘贴以下代码。

objc

	- (void)resolveUsingUIWebView:(NSURL *)URL {
	
	    // We get the auth token from a redirect so we need to handle that in the webview.
	
	    if (![NSThread isMainThread]) {
	        [self performSelectorOnMainThread:@selector(resolveUsingUIWebView:) withObject:URL waitUntilDone:YES];
	        return;
	    }
	
	    NSURLRequest *hostnameURLRequest = [NSURLRequest requestWithURL:URL cachePolicy:NSURLRequestUseProtocolCachePolicy timeoutInterval:10.0f];
	    isRequestBusy = YES;
	    [self.loginView loadRequest:hostnameURLRequest];
	
	    NSLog(@"resolveUsingUIWebView ready (status: UNKNOWN, URL: %@)", self.loginView.request.URL);
	}
	
	- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
	
	    NSLog(@"webView:shouldStartLoadWithRequest: %@ (%li)", request.URL, (long)navigationType);
	
	    // The webview is where all the communication happens. Slightly complicated.
	
	    myLoadedUrl = [webView.request mainDocumentURL];
	    NSLog(@"***Loaded url: %@", myLoadedUrl);
	
	    //if the UIWebView is showing our authorization URL or consent URL, show the UIWebView control
	    if ([request.URL.absoluteString rangeOfString:authURL options:NSCaseInsensitiveSearch].location != NSNotFound) {
	        self.loginView.hidden = NO;
	    } else if ([request.URL.absoluteString rangeOfString:loginURL options:NSCaseInsensitiveSearch].location != NSNotFound) {
	        //otherwise hide the UIWebView, we've left the authorization flow
	        self.loginView.hidden = NO;
	    } else if ([request.URL.absoluteString rangeOfString:bhh options:NSCaseInsensitiveSearch].location != NSNotFound) {
	        //otherwise hide the UIWebView, we've left the authorization flow
	        self.loginView.hidden = YES;
	        [[NXOAuth2AccountStore sharedStore] handleRedirectURL:request.URL];
	    }
	    else {
	        self.loginView.hidden = NO;
	        //read the Location from the UIWebView, this is how Microsoft APIs is returning the
	        //authentication code and relation information. This is controlled by the redirect URL we chose to use from Microsoft APIs
	        //continue the OAuth2 flow
	       // [[NXOAuth2AccountStore sharedStore] handleRedirectURL:request.URL];
	    }
	
	    return YES;
	
	}


### 编写代码以处理 OAuth2 请求的结果

以下代码会处理从 Web 视图返回的 redirectURL。如果身份验证未成功，此代码将重试一次。同时，库会提供错误，让你可在控制台中查看或以异步方式处理。

objc

	- (void)handleOAuth2AccessResult:(NSString *)accessResult {
	
	    AppData* data = [AppData getInstance];
	
	    //parse the response for success or failure
	     if (accessResult)
	    //if success, complete the OAuth2 flow by handling the redirect URL and obtaining a token
	     {
	         [[NXOAuth2AccountStore sharedStore] handleRedirectURL:accessResult];
	    } else {
	        //start over
	        [self requestOAuth2Access];
	    }
	}


### 设置 OAuth 上下文（称为帐户存储）

在这里，你可以针对希望应用程序能够访问的每个服务，在共享帐户存储上调用 `-[NXOAuth2AccountStore setClientID:secret:authorizationURL:tokenURL:redirectURL:forAccountType:]`。帐户类型是字符串，可作为特定服务的标识符。由于你正在访问图形 API，因此，代码会将其视为 `"myGraphService"`。接着，你要设置观察器，以在令牌发生任何更改时告诉你。在你获取令牌后，让用户返回到 `masterView`。



objc

		- (void)setupOAuth2AccountStore {
	
	
	        AppData* data = [AppData getInstance];
	
	    [[NXOAuth2AccountStore sharedStore] setClientID:data.clientId
	                                             secret:data.secret
	                                              scope:[NSSet setWithObject:scopes]
	                                   authorizationURL:[NSURL URLWithString:authURL]
	                                           tokenURL:[NSURL URLWithString:tokenURL]
	                                        redirectURL:[NSURL URLWithString:data.redirectUriString]
	                                      keyChainGroup: keychain
	                                     forAccountType:@"myGraphService"];
	
	    [[NSNotificationCenter defaultCenter] addObserverForName:NXOAuth2AccountStoreAccountsDidChangeNotification
	                                                      object:[NXOAuth2AccountStore sharedStore]
	                                                       queue:nil
	                                                  usingBlock:^(NSNotification *aNotification) {
	                                                      if (aNotification.userInfo) {
	                                                          //account added, we have access
	                                                          //we can now request protected data
	                                                          NSLog(@"Success!! We have an access token.");
	                                                          dispatch_async(dispatch_get_main_queue(),^ {
	
	                                                              MasterViewController* masterViewController = [self.storyboard instantiateViewControllerWithIdentifier:@"masterView"];
	                                                              [self.navigationController pushViewController:masterViewController animated:YES];
	                                                          });
	                                                      } else {
	                                                          //account removed, we lost access
	                                                      }
	                                                  }];
	
	    [[NSNotificationCenter defaultCenter] addObserverForName:NXOAuth2AccountStoreDidFailToRequestAccessNotification
	                                                      object:[NXOAuth2AccountStore sharedStore]
	                                                       queue:nil
	                                                  usingBlock:^(NSNotification *aNotification) {
	                                                      NSError *error = [aNotification.userInfo objectForKey:NXOAuth2AccountStoreErrorKey];
	                                                      NSLog(@"Error!! %@", error.localizedDescription);
	                                                  }];
	}


## 设置主视图以从图形 API 搜索和显示用户

在网格中显示所返回数据的主视图控制器 (MVC) 应用超出了本演练的范围，而且有很多在线教程说明了如何生成该应用。此代码全都在主干文件中。不过，必须在此 MVC 应用程序中处理几件事：

* 当用户在搜索字段中键入内容时进行拦截
* 将数据的对象提供给主视图，以便在网格中显示结果

我们会在下面执行这些操作。

### 添加检查以查看你是否已经登录

如果用户未登录，应用程序就没什么作用，因此，检查缓存中是否已有令牌是明智之举。如果没有，则重定向到登录视图以让用户登录。如果你还记得，在视图加载时执行操作的最佳方式，就是使用 Apple 提供的 `viewDidLoad()` 方法。

objc
	
	- (void)viewDidLoad {
	    [super viewDidLoad];
	
	
	    NXOAuth2AccountStore *store = [NXOAuth2AccountStore sharedStore];
	    NSArray *accounts = [store accountsWithAccountType:@"myGraphService"];
	
	        if (accounts.count == 0) {
	
	        dispatch_async(dispatch_get_main_queue(),^ {
	
	            LoginViewController* userSelectController = [self.storyboard instantiateViewControllerWithIdentifier:@"LoginUserView"];
	            [self.navigationController pushViewController:userSelectController animated:YES];
	        });
	        }


### 在收到数据时更新表视图

当图形 API 返回数据时，必须显示该数据。为简单起见，下面提供了用于更新表的全部代码。你可以直接在 MVC 样本代码中粘贴正确的值。

objc

	#pragma mark - Table View
	
	- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
	    return 1;
	}
	
	- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
	
	        return [upnArray count];
	}
	
	- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
	
	    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"TaskPrototypeCell" forIndexPath:indexPath];
	
	    if ( cell == nil ) {
	        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"TaskPrototypeCell"];
	    }
	
	    User *user = nil;
	     user = [upnArray objectAtIndex:indexPath.row];
	
	
	    // Configure the cell
	    cell.textLabel.text = user.name;
	    [cell setAccessoryType:UITableViewCellAccessoryDisclosureIndicator];
	
	    return cell;
	}



### 提供有人在搜索字段中键入内容时调用图形 API 的方法

当用户在框中键入搜索内容时，你需要将该内容塞入图形 API。你将在以下代码中生成的 `GraphAPICaller` 类会将查找功能从演示当中分离出来。现在，让我们编写会将任何搜索字符送入图形 API 的代码。我们的做法是提供称为 `lookupInGraph` 的方法，其采用我们想要搜索的字符串。

objc
	
		-(void)lookupInGraph:(NSString *)searchText {
		if (searchText.length > 0) {
	
	    };
	
	
	
	        [GraphAPICaller searchUserList:searchText completionBlock:^(NSMutableArray* returnedUpns, NSError* error) {
	            if (returnedUpns) {
	
	
	                upnArray = returnedUpns;
	
	
	            }
	            else
	            {
	                UIAlertView *alertView = [[UIAlertView alloc]initWithTitle:nil message:[[NSString alloc]initWithFormat:@"Error : %@", error.localizedDescription] delegate:nil cancelButtonTitle:@"Retry" otherButtonTitles:@"Cancel", nil];
	
	                [alertView setDelegate:self];
	
	                dispatch_async(dispatch_get_main_queue(),^ {
	                    [alertView show];
	                });
	            }
	
	
	        }];
	
	
	}


## 编写帮助程序类以访问图形 API

这是我们应用程序的核心。而其余就是将代码插入 Apple 提供的默认 MVC 模式，你在这里编写的代码会在用户键入内容时查询图形，然后返回该数据。代码如下所示，后面还有其详细说明。

### 创建新的 Objective C 头文件

将文件命名为 `GraphAPICaller.h` 并添加以下代码。

objc

	@interface GraphAPICaller : NSObject<NSURLConnectionDataDelegate>
	
	+(void) searchUserList:(NSString*)searchString
	       completionBlock:(void (^) (NSMutableArray*, NSError* error))completionBlock;
	
	@end


如你所见，指定的方法会获取字符串并返回 completionBlock。此 completionBlock（如你所猜测）提供的对象会在用户搜索时实时填充数据，以此更新表。


### 创建新的 Objective C 文件

将文件命名为 `GraphAPICaller.m` 并添加以下方法。

objc

		+(void) searchUserList:(NSString*)searchString
	       completionBlock:(void (^) (NSMutableArray* Users, NSError* error)) completionBlock
	{
	    if (!loadedApplicationSettings)
	    {
	        [self readApplicationSettings];
	    }
	
	    AppData* data = [AppData getInstance];
	
	    NSString *graphURL = [NSString stringWithFormat:@"%@%@/users", data.graphApiUrlString, data.apiversion];
	
	    NXOAuth2AccountStore *store = [NXOAuth2AccountStore sharedStore];
	    NSDictionary* params = [self convertParamsToDictionary:searchString];
	
	    NSArray *accounts = [store accountsWithAccountType:@"myGraphService"];
	    [NXOAuth2Request performMethod:@"GET"
	                        onResource:[NSURL URLWithString:graphURL]
	                   usingParameters:params
	                       withAccount:accounts[0]
	               sendProgressHandler:^(unsigned long long bytesSend, unsigned long long bytesTotal) {
	                   // e.g., update a progress indicator
	               }
	                   responseHandler:^(NSURLResponse *response, NSData *responseData, NSError *error) {
	                       // Process the response
	                       if (responseData) {
	                           NSError *error;
	                           NSDictionary *dataReturned = [NSJSONSerialization JSONObjectWithData:responseData options:0 error:nil];
	                           NSLog(@"Graph Response was: %@", dataReturned);
	
	                           // We can grab the top most JSON node to get our graph data.
	                           NSArray *graphDataArray = [dataReturned objectForKey:@"value"];
	
	                           // Don't be thrown off by the key name being "value". It really is the name of the
	                           // first node. :-)
	
	                           //each object is a key value pair
	                           NSDictionary *keyValuePairs;
	                           NSMutableArray* Users = [[NSMutableArray alloc]init];
	
	                           for(int i =0; i < graphDataArray.count; i++)
	                           {
	                               keyValuePairs = [graphDataArray objectAtIndex:i];
	
	                               User *s = [[User alloc]init];
	                               s.upn = [keyValuePairs valueForKey:@"userPrincipalName"];
	                               s.name =[keyValuePairs valueForKey:@"displayName"];
	                               s.mail =[keyValuePairs valueForKey:@"mail"];
	                               s.businessPhones =[keyValuePairs valueForKey:@"businessPhones"];
	                               s.mobilePhones =[keyValuePairs valueForKey:@"mobilePhone"];
	
	
	                               [Users addObject:s];
	                           }
	
	                           completionBlock(Users, nil);
	                       }
	                       else
	                       {
	                           completionBlock(nil, error);
	                       }
	
	                   }];
	}



我们会详细解说此方法。

此代码的核心在于 `NXOAuth2Request`，该方法会采用你已经在 settings.plist 文件中定义的参数。

第一步是构造正确的图形 API 调用。由于你正在调用 `/users`，因此，你会将它追加到图形 API 资源和版本来进行指定。因为这些都会随着 API 演进而改变，所以合理的做法是将其放在外部设置文件中。


objc

	NSString *graphURL = [NSString stringWithFormat:@"%@%@/users", data.graphApiUrlString, data.apiversion];


接下来，你需要指定也会提供给图形 API 调用的参数。切记不要将参数放在资源终结点中，因为系统会在运行时针对所有不符合 URI 的字符擦除该终结点。必须在参数中提供所有查询代码。

objc

	NSDictionary* params = [self convertParamsToDictionary:searchString];


你可能发现这会调用你尚未编写的 `convertParamsToDictionary` 方法。让我们立即在文件末尾这样做：

objc

		+(NSDictionary*) convertParamsToDictionary:(NSString*)searchString
	{
	    NSMutableDictionary* dictionary = [[NSMutableDictionary alloc]init];
	
	        NSString *query = [NSString stringWithFormat:@"startswith(givenName, '%@')", searchString];
	
	           [dictionary setValue:query forKey:@"$filter"];
	
	
	
	    return dictionary;
	}


接下来，我们将使用 `NXOAuth2Request` 方法从 API 取回 JSON 格式的数据。

objc

	NSArray *accounts = [store accountsWithAccountType:@"myGraphService"];
    [NXOAuth2Request performMethod:@"GET"
                        onResource:[NSURL URLWithString:graphURL]
                   usingParameters:params
                       withAccount:accounts[0]
               sendProgressHandler:^(unsigned long long bytesSend, unsigned long long bytesTotal) {
                   // e.g., update a progress indicator
               }
                   responseHandler:^(NSURLResponse *response, NSData *responseData, NSError *error) {
                       // Process the response
                       if (responseData) {
                           NSError *error;
                           NSDictionary *dataReturned = [NSJSONSerialization JSONObjectWithData:responseData options:0 error:nil];
                           NSLog(@"Graph Response was: %@", dataReturned);

                           // We can grab the top most JSON node to get our graph data.
                           NSArray *graphDataArray = [dataReturned objectForKey:@"value"];


最后，来看看你要如何将数据返回到 MasterViewController。数据会以序列化方式返回，而且该数据必须反序列化并加载到 MainViewController 可使用的对象中。出于此目的，主干具有的 `User.m/h` 文件可以创建 User 对象。你会使用图形中的信息填充该 User 对象。

objc
	
	// We can grab the top most JSON node to get our graph data.
	NSArray *graphDataArray = [dataReturned objectForKey:@"value"];
	
	// Don't be thrown off by the key name being "value". It really is the name of the
	// first node. :-)
	
	//each object is a key value pair
	NSDictionary *keyValuePairs;
	NSMutableArray* Users = [[NSMutableArray alloc]init];
	
	for(int i =0; i < graphDataArray.count; i++)
	{
	    keyValuePairs = [graphDataArray objectAtIndex:i];
	
	    User *s = [[User alloc]init];
	    s.upn = [keyValuePairs valueForKey:@"userPrincipalName"];
	    s.name =[keyValuePairs valueForKey:@"displayName"];
	    s.mail =[keyValuePairs valueForKey:@"mail"];
	    s.businessPhones =[keyValuePairs valueForKey:@"businessPhones"];
	    s.mobilePhones =[keyValuePairs valueForKey:@"mobilePhone"];
	
	
	    [Users addObject:s];



## 运行示例

如果你使用主干或遵循本演练，那么，你的应用程序现在应该会运行。启动模拟器，然后单击“登录”以使用应用程序。

## 获取产品的安全更新

建议访问[安全技术中心](https://technet.microsoft.com/security/dd252948)并订阅安全公告通知，以便在发生安全事件时获取相关通知。

<!---HONumber=Mooncake_0815_2016-->