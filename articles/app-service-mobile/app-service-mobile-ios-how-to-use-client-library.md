<properties
	pageTitle="如何使用适用于 Azure 移动应用的 iOS SDK"
	description="如何使用适用于 Azure 移动应用的 iOS SDK"
	services="app-service\mobile"
	documentationCenter="ios"
	authors="krisragh"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.date="06/30/2016"
	wacn.date="10/17/2016"/>

# 如何使用适用于 Azure 移动应用的 iOS 客户端库

[AZURE.INCLUDE [app-service-mobile-selector-client-library](../../includes/app-service-mobile-selector-client-library.md)]

本指南介绍如何使用最新的 [Azure 移动应用 iOS SDK](https://github.com/Azure/azure-mobile-apps-ios-client/blob/master/README.md#ios-client-sdk) 执行常见任务。对于 Azure 移动应用的新手，请先完成 [Azure Mobile Apps Quick Start]（Azure 移动应用快速入门），创建后端、创建表并下载预先生成的 iOS Xcode 项目。本指南侧重于客户端 iOS SDK。有关后端 .NET 服务器端 SDK 的详细信息，请参阅 [Work with .NET Backend](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/)（使用 .NET 后端）

## 参考文档

iOS 客户端 SDK 的参考文档位于此处：[Azure Mobile Apps iOS Client Reference](http://azure.github.io/azure-mobile-apps-ios-client/)（Azure 移动应用 iOS 客户端参考）。

##<a name="Setup"></a>安装与先决条件

本指南假设已创建了包含表的后端。本指南假设该表的架构与这些教程中的表相同。本指南还假设在代码中引用了 `MicrosoftAzureMobile.framework` 并导入了 `MicrosoftAzureMobile/MicrosoftAzureMobile.h`。

##<a name="create-client"></a>如何创建客户端

若要在项目中访问 Azure 移动应用后端，请创建 `MSClient`。将 `AppUrl` 替换为应用 URL。可以将 `gatewayURLString` 和 `applicationKey` 留空。如果设置了用于身份验证的网关，请使用网关 URL 填充 `gatewayURLString`。

**Objective-C**：

```
MSClient *client = [MSClient clientWithApplicationURLString:@"AppUrl"];
```

**Swift**：

```
let client = MSClient(applicationURLString: "AppUrl")
```


##<a name="table-reference"></a>如何：创建表引用

若要访问或更新数据，请创建到后端表的引用。将 `TodoItem` 替换为表名称

**Objective-C**：

```
MSTable *table = [client tableWithName:@"TodoItem"];
```

**Swift**：

```
let table = client.tableWithName("TodoItem")
```


##<a name="querying"></a>如何：查询数据

若要创建数据库查询，请查询 `MSTable` 对象。以下查询将获取 `TodoItem` 中的所有项，并记录每个项的文本。

**Objective-C**：

```
[table readWithCompletion:^(MSQueryResult *result, NSError *error) {
		if(error) { // error is nil if no error occured
				NSLog(@"ERROR %@", error);
		} else {
				for(NSDictionary *item in result.items) { // items is NSArray of records that match query
						NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
				}
		}
}];
```

**Swift**：

```
table.readWithCompletion { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let items = result?.items {
        for item in items {
            print("Todo Item: ", item["text"])
        }
    }
}
```

##<a name="filtering"></a>如何：筛选器返回的数据

可以使用许多可用选项来筛选结果。

若要使用谓词进行筛选，请使用 `NSPredicate` 和 `readWithPredicate`。以下筛选器返回的数据只用于查找未完成的待办事项。

**Objective-C**：

```
// Create a predicate that finds items where complete is false
NSPredicate * predicate = [NSPredicate predicateWithFormat:@"complete == NO"];
// Query the TodoItem table
[table readWithPredicate:predicate completion:^(MSQueryResult *result, NSError *error) {
		if(error) {
				NSLog(@"ERROR %@", error);
		} else {
				for(NSDictionary *item in result.items) {
						NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
				}
		}
}];
```

**Swift**：

```
// Create a predicate that finds items where complete is false
let predicate =  NSPredicate(format: "complete == NO")
// Query the TodoItem table
table.readWithPredicate(predicate) { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let items = result?.items {
        for item in items {
            print("Todo Item: ", item["text"])
        }
    }
}
```

##<a name="query-object"></a>如何：使用 MSQuery

若要执行复杂查询（包括排序和分页），请使用谓词直接创建 `MSQuery` 对象：

**Objective-C**：

```
MSQuery *query = [table query];
MSQuery *query = [table queryWithPredicate: [NSPredicate predicateWithFormat:@"complete == NO"]];
```

**Swift**：

```
let query = table.query()
let query = table.queryWithPredicate(NSPredicate(format: "complete == NO"))
```

`MSQuery` 可让你控制以下几种查询行为。通过对 `MSQuery` 查询调用 `readWithCompletion` 来执行该查询，如以下例所示。
* 指定结果顺序
* 限制要返回的字段
* 限制要返回的记录数
* 指定响应中的总计数
* 指定请求中的自定义查询字符串参数
* 应用其他函数


## <a name="sorting"></a>如何：使用 MSQuery 对数据排序

让我们先看一个示例，来了解如何对结果排序。若要先按 `text` 字段升序排序，然后按 `completion` 字段降序排序，请按如下所示调用 `MSQuery`：

**Objective-C**：

```
[query orderByAscending:@"text"];
[query orderByDescending:@"complete"];
[query readWithCompletion:^(MSQueryResult *result, NSError *error) {
		if(error) {
				NSLog(@"ERROR %@", error);
		} else {
				for(NSDictionary *item in result.items) {
						NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
				}
		}
}];
```

**Swift**：

```
query.orderByAscending("text")
query.orderByDescending("complete")
query.readWithCompletion { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let items = result?.items {
        for item in items {
            print("Todo Item: ", item["text"])
        }
    }
}
```


## <a name="selecting"></a><a name="parameters"></a>如何：使用 MSQuery 限制字段和展开查询字符串参数

若要限制在查询中返回的字段，请在 **selectFields** 属性中指定字段的名称。这样，便只会返回文本和已完成的字段：

**Objective-C**：

```
query.selectFields = @[@"text", @"complete"];
```

**Swift**：

```
query.selectFields = ["text", "complete"]
```

若要在服务器请求中包含其他查询字符串参数（例如，某个自定义服务器端脚本要使用这些参数），请按如下所示填充 `query.parameters`：

**Objective-C**：

```
query.parameters = @{
	@"myKey1" : @"value1",
	@"myKey2" : @"value2",
};
```

**Swift**：

```
query.parameters = ["myKey1": "value1", "myKey2": "value2"]
```

##<a name="inserting"></a>如何：插入数据

若要插入新的表行，请创建新的 `NSDictionary` 并调用 `table insert`。移动服务将会根据 `NSDictionary` 自动生成新列（如果未启用[动态架构]）。

如果未提供 `id`，后端会自动生成新的唯一 ID。提供你自己的 `id`，以使用电子邮件地址、用户名或你自己的自定义值作为 ID。提供自己的 ID 可以让联接和业务导向型数据库逻辑变得更容易。

`result` 包含插入的新项；根据服务器逻辑，与传递给服务器的项相比，它可能包含其他或已修改的数据。

**Objective-C**：

```
NSDictionary *newItem = @{@"id": @"custom-id", @"text": @"my new item", @"complete" : @NO};
[table insert:newItem completion:^(NSDictionary *result, NSError *error) {
	if(error) {
		NSLog(@"ERROR %@", error);
	} else {
		NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
	}
}];
```

**Swift**：

```
let newItem = ["id": "custom-id", "text": "my new item", "complete": false]
table.insert(newItem) { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let item = result {
        print("Todo Item: ", item["text"])
    }
}
```

##<a name="modifying"></a>如何：修改数据

若要更新现有的行，请修改项并调用 `update`：

**Objective-C**：

```
NSMutableDictionary *newItem = [oldItem mutableCopy]; // oldItem is NSDictionary
[newItem setValue:@"Updated text" forKey:@"text"];
[table update:newItem completion:^(NSDictionary *result, NSError *error) {
	if(error) {
		NSLog(@"ERROR %@", error);
	} else {
		NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
	}
}];
```

**Swift**：

```
if let newItem = oldItem.mutableCopy() as? NSMutableDictionary {
    newItem["text"] = "Updated text"
    table2.update(newItem as [NSObject: AnyObject], completion: { (result, error) -> Void in
        if let err = error {
            print("ERROR ", err)
        } else if let item = result {
            print("Todo Item: ", item["text"])
        }
    })
}
```

或者，提供行 ID 和更新的字段：

**Objective-C**：

```
[table update:@{@"id":@"custom-id", @"text":"my EDITED item"} completion:^(NSDictionary *result, NSError *error) {
	if(error) {
		NSLog(@"ERROR %@", error);
	} else {
		NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
	}
}];
```

**Swift**：

```
table.update(["id": "custom-id", "text": "my EDITED item"]) { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let item = result {
        print("Todo Item: ", item["text"])
    }
}
```

进行更新时，至少必须设置 `id` 属性。

##<a name="deleting"></a>如何：删除数据

若要删除某个项，请对该项调用 `delete`：

**Objective-C**：

```
[table delete:item completion:^(id itemId, NSError *error) {
	if(error) {
		NSLog(@"ERROR %@", error);
	} else {
		NSLog(@"Todo Item ID: %@", itemId);
	}
}];
```

**Swift**：

```
table.delete(newItem as [NSObject: AnyObject]) { (itemId, error) in
    if let err = error {
        print("ERROR ", err)
    } else {
        print("Todo Item ID: ", itemId)
    }
}
```

或者，提供行 ID 来进行删除：

**Objective-C**：

```
[table deleteWithId:@"37BBF396-11F0-4B39-85C8-B319C729AF6D" completion:^(id itemId, NSError *error) {
	if(error) {
		NSLog(@"ERROR %@", error);
	} else {
		NSLog(@"Todo Item ID: %@", itemId);
	}
}];
```

**Swift**：

```
table.deleteWithId("37BBF396-11F0-4B39-85C8-B319C729AF6D") { (itemId, error) in
    if let err = error {
        print("ERROR ", err)
    } else {
        print("Todo Item ID: ", itemId)
    }
}
```

进行删除时，至少必须设置 `id` 属性。

##<a name="customapi"></a>如何调用自定义 API

使用自定义 API 可以公开任何后端功能。无需映射到表操作。不仅能进一步控制消息，甚至还可以读取或设置标头，并更改响应正文格式。若要了解如何在后端上创建自定义 API，请阅读 [Custom APIs](/documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/#work-easy-apis)（自定义 API）

若要调用自定义 API，请按如下所示调用 `MSClient.invokeAPI`。请求和响应内容被视为 JSON。若要使用其他媒体类型，请[使用 `invokeAPI` 的其他重载](http://azure.github.io/azure-mobile-services/iOS/v3/Classes/MSClient.html#//api/name/invokeAPI:data:HTTPMethod:parameters:headers:completion:)

若要发出 `GET` 请求而不是 `POST` 请求，请将参数 `HTTPMethod` 设置为 `"GET"`，将参数 `body` 设置为 `nil`（因为 GET 请求没有消息正文）。 如果自定义 API 支持其他 HTTP 谓词，请相应地更改 `HTTPMethod`。

**Objective-C**：
```
[self.client invokeAPI:@"sendEmail"
                  body:@{ @"contents": @"Hello world!" }
            HTTPMethod:@"POST"
            parameters:@{ @"to": @"bill@contoso.com", @"subject" : @"Hi!" }
               headers:nil
            completion: ^(NSData *result, NSHTTPURLResponse *response, NSError *error) {
                if(error) {
                    NSLog(@"ERROR %@", error);
                } else {
                    // Do something with result
                }
            }];
```

**Swift**：

```
client.invokeAPI("sendEmail",
            body: [ "contents": "Hello World" ],
            HTTPMethod: "POST",
            parameters: [ "to": "bill@contoso.com", "subject" : "Hi!" ],
            headers: nil)
            {
                (result, response, error) -> Void in
                if let err = error {
                    print("ERROR ", err)
                } else if let res = result {
                          // Do something with result
                }
        }
```

##<a name="templates"></a>如何注册推送模板以发送跨平台通知

若要注册模板，只需在客户端应用中连同 **client.push registerDeviceToken** 方法一起传递模板即可。

**Objective-C**：

```
[client.push registerDeviceToken:deviceToken template:iOSTemplate completion:^(NSError *error) {
	if(error) {
		NSLog(@"ERROR %@", error);
	}
}];
```

**Swift**：

```
    client.push?.registerDeviceToken(NSData(), template: iOSTemplate, completion: { (error) in
        if let err = error {
            print("ERROR ", err)
        }
    })
```

模板类型是 NSDictionary，可以包含采用以下格式的多个模板：

**Objective-C**：

```
NSDictionary *iOSTemplate = @{ @"templateName": @{ @"body": @{ @"aps": @{ @"alert": @"$(message)" } } } };
```

**Swift**：

```
let iOSTemplate = ["templateName": ["body": ["aps": ["alert": "$(message)"]]]]
```

请注意，所有标记因安全考虑而删除。若要将标记添加到安装或安装中的模板，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/#tags)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

若要使用这些注册的模板发送通知，请参阅 [Notification Hubs APIs](https://msdn.microsoft.com/zh-cn/library/azure/dn495101.aspx)（通知中心 API）。

##<a name="errors"></a>如何：处理错误

调用移动服务时，完成块将包含 `NSError` 参数。如果出错，此参数为非 nil 值。应该在代码中检查此参数，并根据需要处理错误，如上面的代码片段中所示。

[`<WindowsAzureMobileServices/MSError.h>`](https://github.com/Azure/azure-mobile-services/blob/master/sdk/iOS/src/MSError.h) 文件定义 `MSErrorResponseKey`、`MSErrorRequestKey` 和 `MSErrorServerItemKey` 常量以获取有关错误的更多数据，可按如下所示获取：

**Objective-C**：

```
NSDictionary *serverItem = [error.userInfo objectForKey:MSErrorServerItemKey];
```

**Swift**：

```
let serverItem = error.userInfo[MSErrorServerItemKey]
```

此外，该文件还定义每个错误代码的常量，使用方式如下：

**Objective-C**：

```
if (error.code == MSErrorPreconditionFailed) {
```

**Swift**：

```
if (error.code == MSErrorPreconditionFailed) {
```

## <a name="adal"></a>如何使用 Active Directory 身份验证库对用户进行身份验证

可以借助 Active Directory 身份验证库 (ADAL) 使用 Azure Active Directory 将用户登录到应用程序。这通常比使用 `loginAsync()` 方法更有利，因为它提供更直观的 UX 风格，并允许其他自定义。

1. 根据 [How to configure App Service for Active Directory login](/documentation/articles/app-service-mobile-how-to-configure-active-directory-authentication/)（如何为 Active Directory 登录配置应用服务）教程的说明，为 AAD 登录配置移动应用。请务必完成注册本机客户端应用程序的可选步骤。对于 iOS，建议为重定向 URI 使用 `<app-scheme>://<bundle-id>` 格式（但不一定要这样做）。

2. 使用 Cocoapods 安装 ADAL。编辑 Podfile 以包含以下内容，将 **YOUR-PROJECT** 替换为 Xcode 项目的名称：

		source 'https://github.com/CocoaPods/Specs.git'
		link_with ['YOUR-PROJECT']
		xcodeproj 'YOUR-PROJECT'
Pod：

		pod 'ADALiOS'

3. 使用终端，从包含项目的目录运行 `pod install`，然后打开生成的 Xcode 工作区（而不是项目）。

4. 根据使用的语言，将以下代码添加到应用程序。在每条代码中进行以下替换：

* 将 **INSERT-AUTHORITY-HERE** 替换为在其中预配了应用程序的租户的名称。格式应为 https://login.chinacloudapi.cn/contoso.partner.onmschina.cn。可以在 [Azure 经典管理门户] 中 Azure Active Directory 的“域”选项卡复制此值。

* 将 **INSERT-RESOURCE-ID-HERE** 替换移动应用后端的客户端 ID。可以在门户中“Azure Active Directory 设置”下面的“高级”选项卡获取此值。

* 将 **INSERT-CLIENT-ID-HERE** 替换为从本机客户端应用程序复制的客户端 ID。

* 将 **INSERT-REDIRECT-URI-HERE** 替换为站点的 _/.auth/login/done_ 终结点（使用 HTTPS 方案）。此值应类似于 \_https://contoso.chinacloudsites.cn/.auth/login/done_。

**Objective-C**：

	#import <ADALiOS/ADAuthenticationContext.h>
	#import <ADALiOS/ADAuthenticationSettings.h>
	// ...
	- (void) authenticate:(UIViewController*) parent
	           completion:(void (^) (MSUser*, NSError*))completionBlock;
	{
	    NSString *authority = @"INSERT-AUTHORITY-HERE";
	    NSString *resourceId = @"INSERT-RESOURCE-ID-HERE";
	    NSString *clientId = @"INSERT-CLIENT-ID-HERE";
	    NSURL *redirectUri = [[NSURL alloc]initWithString:@"INSERT-REDIRECT-URI-HERE"];
	    ADAuthenticationError *error;
	    ADAuthenticationContext *authContext = [ADAuthenticationContext authenticationContextWithAuthority:authority error:&error];
	    authContext.parentController = parent;
	    [ADAuthenticationSettings sharedInstance].enableFullScreen = YES;
	    [authContext acquireTokenWithResource:resourceId
	                                 clientId:clientId
	                              redirectUri:redirectUri
	                          completionBlock:^(ADAuthenticationResult *result) {
	                              if (result.status != AD_SUCCEEDED)
	                              {
	                                  completionBlock(nil, result.error);;
	                              }
	                              else
	                              {
	                                  NSDictionary *payload = @{
	                                                            @"access_token" : result.tokenCacheStoreItem.accessToken
	                                                            };
	                                  [client loginWithProvider:@"aad" token:payload completion:completionBlock];
	                              }
	                          }];
	}


**Swift**：

	// add the following imports to your bridging header:
	//		#import <ADALiOS/ADAuthenticationContext.h>
	//		#import <ADALiOS/ADAuthenticationSettings.h>

	func authenticate(parent: UIViewController, completion: (MSUser?, NSError?) -> Void) {
		let authority = "INSERT-AUTHORITY-HERE"
		let resourceId = "INSERT-RESOURCE-ID-HERE"
		let clientId = "INSERT-CLIENT-ID-HERE"
		let redirectUri = NSURL(string: "INSERT-REDIRECT-URI-HERE")
		var error: AutoreleasingUnsafeMutablePointer<ADAuthenticationError?> = nil
		let authContext = ADAuthenticationContext(authority: authority, error: error)
		authContext.parentController = parent
		ADAuthenticationSettings.sharedInstance().enableFullScreen = true
		authContext.acquireTokenWithResource(resourceId, clientId: clientId, redirectUri: redirectUri) { (result) in
		        if result.status != AD_SUCCEEDED {
		            completion(nil, result.error)
		        }
		        else {
		            let payload: [String: String] = ["access_token": result.tokenCacheStoreItem.accessToken]
		            client.loginWithProvider("aad", token: payload, completion: completion)
		        }
    		}
	}

<!-- Anchors. -->

[What is Mobile Services]: #what-is
[Concepts]: #concepts
[Setup and Prerequisites]: #Setup
[How to: Create the Mobile Services client]: #create-client
[How to: Create a table reference]: #table-reference
[How to: Query data from a mobile service]: #querying
[Filter returned data]: #filtering
[Sort returned data]: #sorting
[Return data in pages]: #paging
[Select specific columns]: #selecting
[How to: Bind data to the user interface]: #binding
[How to: Insert data into a mobile service]: #inserting
[How to: Modify data in a mobile service]: #modifying
[How to: Authenticate users]: #authentication
[Cache authentication tokens]: #caching-tokens
[How to: Upload images and large files]: #blobs
[How to: Handle errors]: #errors
[How to: Design unit tests]: #unit-testing
[How to: Customize the client]: #customizing
[Customize request headers]: #custom-headers
[Customize data type serialization]: #custom-serialization
[Next Steps]: #next-steps
[How to: Use MSQuery]: #query-object

<!-- Images. -->

<!-- URLs. -->
[Azure Mobile Apps Quick Start]: /documentation/articles/app-service-mobile-ios-get-started/

[Add Mobile Services to Existing App]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/-data
[Get started with Mobile Services]: /documentation/articles/mobile-services-ios-get-started/
[Validate and modify data in Mobile Services by using server scripts]: /develop/mobile/tutorials/validate-modify-and-augment-data-ios
[Mobile Services SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[Authentication]: /documentation/articles/mobile-services-ios-get-started-users/
[iOS SDK]: https://developer.apple.com/xcode

[Handling Expired Tokens]: http://go.microsoft.com/fwlink/p/?LinkId=301955
[Live Connect SDK]: http://go.microsoft.com/fwlink/p/?LinkId=301960
[Permissions]: http://msdn.microsoft.com/zh-cn/library/azure/jj193161.aspx
[Service-side Authorization]: /documentation/articles/mobile-services-javascript-backend-service-side-authorization/
[Use scripts to authorize users]: /documentation/articles/mobile-services-javascript-backend-service-side-authorization/
[动态架构]: https://msdn.microsoft.com/zh-cn/library/azure/jj193175.aspx
[How to: access custom parameters]: /develop/mobile/how-to-guides/work-with-server-scripts#access-headers
[Create a table]: http://msdn.microsoft.com/zh-cn/library/azure/jj193162.aspx
[NSDictionary object]: http://go.microsoft.com/fwlink/p/?LinkId=301965
[ASCII control codes C0 and C1]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set
[CLI to manage Mobile Services tables]: /documentation/articles/virtual-machines-command-line-tools/#Mobile_Tables
[Conflict-Handler]: /documentation/articles/mobile-services-ios-handling-conflicts-offline-data/#add-conflict-handling

<!---HONumber=Mooncake_0919_2016-->