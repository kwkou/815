<properties
	pageTitle="Azure AD iOS 入门 | Azure"
	description="如何生成一个与 Azure AD 集成以方便登录，并使用 OAuth 调用 Azure AD 保护 API 的 iOS 应用程序。"
	services="active-directory"
	documentationCenter="ios"
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="mobile-ios"
	ms.devlang="objective-c"
	ms.topic="article"
	ms.date="09/16/2016"
	wacn.date="10/17/2016"
	ms.author="brandwe"/>

# 将 Azure AD 集成到 iOS 应用程序中

[AZURE.INCLUDE [active-directory-devquickstarts-switcher](../../includes/active-directory-devquickstarts-switcher.md)]

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

对于需要访问受保护资源的 iOS 客户端，Azure AD 提供 Active Directory 身份验证库 (ADAL)。在本质上，ADAL 的唯一用途就是方便应用程序获取访问令牌。为了演示这种简便性，我们生成了一个 Objective C 待办事项列表应用程序，其中包括：

-	使用 [OAuth 2.0 身份验证协议](/documentation/articles/active-directory-protocols-oauth-code/)获取调用 Azure AD Graph API 的访问令牌。
-	在目录中搜索具有给定别名的用户。

若要生成完整的工作应用程序，你需要：

2. 将应用程序注册到 Azure AD。
3. 安装并配置 ADAL。
5. 使用 ADAL 从 Azure AD 获取令牌。

若要开始，请[下载应用程序框架](https://github.com/AzureADQuickStarts/NativeClient-iOS/archive/skeleton.zip)或[下载已完成的示例](https://github.com/AzureADQuickStarts/NativeClient-iOS/archive/complete.zip)。你还需要一个可在其中创建用户和注册应用程序的 Azure AD 租户。如果你还没有租户，请[了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)。

## *1.确定用于 iOS 的重定向 URI*

为了安全地在特定 SSO 方案中启动应用程序，我们需要以特定格式创建**重定向 URI**。重定向 URI 可确保将令牌返回给需要它们的正确应用程序。

重定向 URI 的 iOS 格式为：


	<app-scheme>://<bundle-id>


- 	**aap-scheme** - 已在 XCode 项目中注册。它是其他应用程序与你联系的方式。可以在 Info.plist -> URL types -> URL Identifier 下找到此信息。如果尚未配置一个或多个方案，则你应该创建一个。
- 	**bundle-id** - 这是在 XCode 项目设置中“identity”下可找到的捆绑标识符。

此快速入门代码的示例为：***msquickstart://com.microsoft.azureactivedirectory.samples.graph.QuickStart***

## *2.注册 DirectorySearcher 应用程序*
若要让应用程序获取令牌，首先需要在 Azure AD 租户中注册该应用程序，并授予它访问 Azure AD Graph API 的权限：

-	登录到 Azure 管理门户
-	在左侧的导航栏中单击“Active Directory”
-	选择要在其中注册应用程序的租户。
-	单击“应用程序”选项卡，然后在底部抽屉中单击“添加”。
-	根据提示创建一个新的“本机客户端应用程序”。
    -	应用程序的“名称”向最终用户描述你的应用程序
    -	“重定向 URI”是 Azure AD 要用来返回令牌响应的方案与字符串组合。请根据上面的信息输入特定于你应用程序的值。
-	完成注册后，AAD 将为应用程序分配唯一的客户端标识符。在后面的部分中将会用到此值，因此，请从“配置”选项卡复制此值。
- 另外，请在“配置”选项卡中，找到“针对其他应用程序的权限”部分。对于“Azure Active Directory”应用程序，在“委托的权限”下添加“访问组织的目录”权限。这样，你的应用程序便可以在 Graph API 中查询用户。

## *3.安装并配置 ADAL*
将应用程序注册到 Azure AD 后，可以安装 ADAL 并编写标识相关的代码。为了使 ADAL 能够与 Azure AD 通信，需要为 ADAL 提供一些有关应用程序的注册信息。
-	首先，使用 Cocapods 将 ADAL 添加到 DirectorySearcher 项目。


	$ vi Podfile

将以下内容添加到 podfile：


	source 'https://github.com/CocoaPods/Specs.git'
	link_with ['QuickStart']
	xcodeproj 'QuickStart'

	pod 'ADALiOS'


现在，请使用 cocoapods 加载 podfile。这会创建你要加载的新 XCode 工作区。


	$ pod install
	...
	$ open QuickStart.xcworkspace


-	在快速入门项目中，打开 plist 文件 `settings.plist`。替换节中的元素值，以反映你在 Azure 门户中输入的值。只要使用 ADAL，你的代码就会引用这些值。
    -	`tenant` 是 Azure AD 租户的域，例如 contoso.partner.onmschina.cn
    -	`clientId` 是从门户复制的应用程序 clientId。
    -	`redirectUri` 是在门户中注册的 URL。

## *4.使用 ADAL 从 Azure AD 获取令牌*
ADAL 遵守的基本原理是，每当应用程序需要访问令牌时，它只需调用 completionBlock `+(void) getToken : `，然后 ADAL 就会负责其余的工作。

-	在 `QuickStart` 项目中，打开 `GraphAPICaller.m` 并找到靠近顶部位置的 `// TODO: getToken for generic Web API flows. Returns a token with no additional parameters provided.` 注释。你将在此处通过 CompletionBlock 传递 ADAL 与 Azure AD 通信时所需的坐标，并告诉 ADAL 如何缓存令牌。

ObjC

	+(void) getToken : (BOOL) clearCache
	           parent:(UIViewController*) parent
	completionHandler:(void (^) (NSString*, NSError*))completionBlock;
	{
	    AppData* data = [AppData getInstance];
	    if(data.userItem){
	        completionBlock(data.userItem.accessToken, nil);
	        return;
	    }
	    
	    ADAuthenticationError *error;
	    authContext = [ADAuthenticationContext authenticationContextWithAuthority:data.authority error:&error];
	    authContext.parentController = parent;
	    NSURL *redirectUri = [[NSURL alloc]initWithString:data.redirectUriString];
	    
	    [ADAuthenticationSettings sharedInstance].enableFullScreen = YES;
	    [authContext acquireTokenWithResource:data.resourceId
	                                 clientId:data.clientId
	                              redirectUri:redirectUri
	                           promptBehavior:AD_PROMPT_AUTO
	                                   userId:data.userItem.userInformation.userId
	                     extraQueryParameters: @"nux=1" // if this strikes you as strange it was legacy to display the correct mobile UX. You most likely won't need it in your code.
	                          completionBlock:^(ADAuthenticationResult *result) {
	                              
	                              if (result.status != AD_SUCCEEDED)
	                              {
	                                  completionBlock(nil, result.error);
	                              }
	                              else
	                              {
	                                  data.userItem = result.tokenCacheStoreItem;
	                                  completionBlock(result.tokenCacheStoreItem.accessToken, nil);
	                              }
	                          }];
	}


- 现在，我们需要使用此令牌在图形中搜索用户。查找 `// TODO: implement SearchUsersList` 注释。此方法将向 Azure AD 图形 API 发出 GET 请求，以查询其 UPN 以给定搜索词开头的用户。但是，若要查询 Graph API，你需要在请求的 `Authorization` 标头中包含 access\_token - 这是 ADAL 传入的位置。

ObjC

	+(void) searchUserList:(NSString*)searchString
	                parent:(UIViewController*) parent
	       completionBlock:(void (^) (NSMutableArray* Users, NSError* error)) completionBlock
	{
	    if (!loadedApplicationSettings)
	    {
	        [self readApplicationSettings];
	    }
    
	    AppData* data = [AppData getInstance];
    
	    NSString *graphURL = [NSString stringWithFormat:@"%@%@/users?api-version=%@&$filter=startswith(userPrincipalName, '%@')", data.taskWebApiUrlString, data.tenant, data.apiversion, searchString];

    
	    [self craftRequest:[self.class trimString:graphURL]
	                parent:parent
	     completionHandler:^(NSMutableURLRequest *request, NSError *error) {
         
	         if (error != nil)
	         {
	             completionBlock(nil, error);
	         }
	         else
	         {
             
	             NSOperationQueue *queue = [[NSOperationQueue alloc]init];
             
	             [NSURLConnection sendAsynchronousRequest:request queue:queue completionHandler:^(NSURLResponse *response, NSData *data, NSError *error) {
                 
	                 if (error == nil && data != nil){
                     
	                     NSDictionary *dataReturned = [NSJSONSerialization JSONObjectWithData:data options:0 error:nil];
                     
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
	                         s.name =[keyValuePairs valueForKey:@"givenName"];
                         
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
	     }];
    
	}


- 当应用程序通过调用 `getToken(...)` 请求令牌时，ADAL 将尝试返回一个令牌，而不要求用户输入凭据。如果 ADAL 确定用户需要登录以获取令牌，将显示登录对话框，收集用户的凭据，并在身份验证成功后返回令牌。如果 ADAL 出于任何原因无法返回令牌，则会引发 `AdalException`。
- 请注意，`AuthenticationResult` 对象包含 `tokenCacheStoreItem` 对象，后者可用于收集应用程序可能需要的信息。在快速入门项目中，`tokenCacheStoreItem` 用于确定身份验证是否已发生。


## 步骤 5：生成并运行应用程序



祝贺你！ 现在，你已创建一个有效的 iOS 应用程序，它可以对用户进行身份验证，使用 OAuth 2.0 安全调用 Web API，并获取有关用户的基本信息。如果你尚未这样做，可以在租户中填充一些用户。运行你的快速入门应用，然后使用这些用户之一进行登录。根据用户的 UPN 搜索其他用户。关闭应用程序，然后重新运行它。请注意，用户的会话将保持不变。

使用 ADAL 可以方便地将所有这些常见标识功能合并到应用程序中。它会负责所有的繁琐工作 - 缓存管理、OAuth 协议支持、向用户显示登录名 UI、刷新已过期的令牌，等等。你只需要真正了解一个 API 调用，即 `getToken`。

[此处](https://github.com/AzureADQuickStarts/NativeClient-iOS/archive/complete.zip)提供了已完成示例（无需配置值）供你参考。

## 其他方案
现在，你可以转到其他方案。你可能想要尝试：

- [使用 Azure AD 保护 Node.JS Web API](/documentation/articles/active-directory-devquickstarts-webapi-nodejs/)
- 了解[如何使用 ADAL 在 iOS 上启用跨应用 SSO](/documentation/articles/active-directory-sso-ios/)

[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]

<!---HONumber=Mooncake_1010_2016-->
