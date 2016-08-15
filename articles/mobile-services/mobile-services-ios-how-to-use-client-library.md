<properties
	pageTitle="如何使用适用于 Azure 移动服务的 iOS 客户端库"
	description="如何使用适用于移动服务的 iOS 客户端库"
	services="mobile-services"
	documentationCenter="ios"
	authors="krisragh"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="03/09/2016"
	wacn.date="02/26/2016"/>

#  如何使用适用于 Azure 移动服务的 iOS 客户端库

[AZURE.INCLUDE [mobile-services-selector-client-library](../includes/mobile-services-selector-client-library.md)]

本指南介绍如何使用 Azure 移动服务 [iOS SDK] 执行常见任务。如果你不熟悉移动服务，请先完成[移动服务快速入门]，以配置你的帐户、创建表，并创建移动服务。

> [AZURE.NOTE]本指南使用最新的 [iOS 移动服务 SDK](https://go.microsoft.com/fwLink/?LinkID=266533&clcid=0x409)。如果你的项目使用旧版 SDK，请先升级 Xcode 中的框架。

[AZURE.INCLUDE [mobile-services-concepts](../includes/mobile-services-concepts.md)]

## <a name="Setup"></a>安装与先决条件

本指南假设你已创建一个移动服务和一个表。有关详细信息，请参阅[创建表]，或再次使用[移动服务快速入门]中创建的 `TodoItem` 表。本指南假设该表的架构与这些教程中的表相同。本指南还假设你的 Xcode 将引用 `WindowsAzureMobileServices.framework` 并导入 `WindowsAzureMobileServices/WindowsAzureMobileServices.h`。

## <a name="create-client"></a>如何：创建移动服务客户端

若要在项目中访问 Azure 移动服务，请创建 `MSClient` 客户端对象。将 `AppUrl` 和 `AppKey` 分别替换为“仪表板”中的移动服务 URL 和应用程序密钥值。

```
MSClient *client = [MSClient clientWithApplicationURLString:@"AppUrl" applicationKey:@"AppKey"];
```

## <a name="table-reference"></a>如何：创建表引用

若要访问或更新 Azure 移动服务的数据，请创建对表的引用。将 `TodoItem` 替换为你的表名称。

```
	MSTable *table = [client tableWithName:@"TodoItem"];
```

## <a name="querying"></a>如何：查询数据

若要创建数据库查询，请查询 `MSTable` 对象。以下查询将获取 `TodoItem` 中的所有项，并记录每个项的文本。

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

## <a name="filtering"></a>如何：筛选器返回的数据

可以使用许多可用选项来筛选结果。

若要使用谓词进行筛选，请使用 `NSPredicate` 和 `readWithPredicate`。以下筛选器返回的数据只用于查找未完成的待办事项。

```
// Create a predicate that finds items where complete is false
	NSPredicate *predicate = [NSPredicate predicateWithFormat:@"complete == NO"];
// Query the TodoItem table and update the items property with the results from the service
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

## <a name="query-object"></a>如何：使用 MSQuery

若要执行复杂查询（包括排序和分页），请使用谓词直接创建 `MSQuery` 对象：

```
    MSQuery *query = [table query];
    MSQuery *query = [table queryWithPredicate: [NSPredicate predicateWithFormat:@"complete == NO"]];
```

`MSQuery` 可让你控制以下几种查询行为。通过调用 `readWithCompletion` 执行 `MSQuery` 查询，如下一示例中所示。
* 指定结果的顺序 
* 限制要返回的字段 
* 限制要返回的记录数 
* 指定响应中的总计数 
* 指定请求中的自定义查询字符串参数 
* 应用其他函数


##  <a name="sorting"></a>如何：使用 MSQuery 对数据排序

让我们先看一个示例，来了解如何对结果排序。若要先按 **text** 字段升序排序，然后按 `completion` 字段降序排序，请按如下所示调用 `MSQuery`：

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

##  <a name="paging"></a>如何：使用 MSQuery 在页中返回数据

移动服务可以限制单个响应中返回的记录量。若要控制显示给用户的记录数，你必须实施一个分页系统。你可以使用 **MSQuery** 对象的以下三个属性来执行分页：

```
+	`BOOL includeTotalCount`
+	`NSInteger fetchLimit`
+	`NSInteger fetchOffset`
```

在以下示例中，一个简单函数会向服务器请求 5 条记录，然后将这些记录附加到先前加载记录的本地集合：

```
// Create and initialize these properties
@property (nonatomic, strong)   NSMutableArray *loadedItems; // Init via [[NSMutableArray alloc] init]
@property (nonatomic)   				BOOL moreResults;
```

```
-(void)loadResults
{
    MSQuery *query = [self.table query];

		query.includeTotalCount = YES;
    query.fetchLimit = 5;
		query.fetchOffset = self.loadedItems.count;


    [query readWithCompletion:^(MSQueryResult *result, NSError *error) {
			if(!error) {
				// Add the items to our local copy
            [self.loadedItems addObjectsFromArray:result.items];

            // Set a flag to keep track if there are any additional records we need to load
            self.moreResults = (self.loadedItems.count <= result.totalCount);
			}
		}];
	}

```
 
##  <a name="selecting"></a><a name="parameters"></a>如何：使用 MSQuery 限制字段和展开查询字符串参数

若要限制在查询中返回的字段，请在 **selectFields** 属性中指定字段的名称。这样，便只会返回文本和已完成的字段：

```
	query.selectFields = @[@"text", @"completed"];
```

若要在服务器请求中包含其他查询字符串参数（例如，某个自定义服务器端脚本要使用这些参数），请按如下所示填充 `query.parameters`：

```
	query.parameters = @{
		@"myKey1" : @"value1",
		@"myKey2" : @"value2",
	};
```

## <a name="inserting"></a>如何：插入数据

若要插入新的表行，请创建新的 `NSDictionary` 并调用 `table insert`。移动服务将会根据 `NSDictionary` 自动生成新列（如果未启用[动态架构]）。

如果未提供 `id`，后端会自动生成新的唯一 ID。提供你自己的 `id`，以使用电子邮件地址、用户名或你自己的自定义值作为 ID。提供自己的 ID 可以让联接和业务导向型数据库逻辑变得更容易。

```
	NSDictionary *newItem = @{@"id": @"custom-id", @"text": @"my new item", @"complete" : @NO};
	[self.table insert:newItem completion:^(NSDictionary *result, NSError *error) {
		// The result contains the new item that was inserted,
		// depending on your server scripts it may have additional or modified 
		// data compared to what was passed to the server.
		if(error) {
				NSLog(@"ERROR %@", error);
		} else {
						NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
	}
	}];
```

## <a name="modifying"></a>如何：修改数据

若要更新现有的行，请修改项并调用 `update`：

```
	NSMutableDictionary *newItem = [oldItem mutableCopy]; // oldItem is NSDictionary
	[newItem setValue:@"Updated text" forKey:@"text"];
	[self.table update:newItem completion:^(NSDictionary *item, NSError *error) {
		// Handle error or perform additional logic as needed
	}];
```

或者，提供行 ID 和更新的字段：

```
	[self.table update:@{@"id":@"37BBF396-11F0-4B39-85C8-B319C729AF6D", @"Complete":@YES} completion:^(NSDictionary *item, NSError *error) {
		// Handle error or perform additional logic as needed
	}];
```

进行更新时，至少必须设置 `id` 属性。

## <a name="deleting"></a>如何：删除数据

若要删除某个项，请对该项调用 `delete`：

```
	[self.table delete:item completion:^(id itemId, NSError *error) {
		// Handle error or perform additional logic as needed
	}];
```

或者，提供行 ID 来进行删除：

```
	[self.table deleteWithId:@"37BBF396-11F0-4B39-85C8-B319C729AF6D" completion:^(id itemId, NSError *error) {
		// Handle error or perform additional logic as needed
	}];
```

进行删除时，至少必须设置 `id` 属性。

##<a name="#custom-api"></a>如何：调用自定义 API

自定义 API 可让你定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。有关如何在移动服务中创建自定义 API 的示例，请参阅[如何：定义自定义 API 终结点](/documentation/articles/mobile-services-dotnet-backend-define-custom-api/)。

[AZURE.INCLUDE [mobile-services-ios-call-custom-api](../includes/mobile-services-ios-call-custom-api.md)]


## <a name="authentication"></a>如何：对用户进行身份验证

Azure 移动服务支持各种标识提供者。有关基本教程，请参阅 [身份验证]。

Azure 移动服务支持两种身份验证工作流：

- **服务器托管登录**：Azure 移动服务将代表应用程序管理登录过程。它会显示提供者特定的登录页，并使用选择的提供者进行身份验证。

- **客户端托管登录**：_应用程序_必须从标识提供者请求令牌，然后将此令牌提供给 Azure 移动服务以进行身份验证。
		
当身份验证成功时，你将获得具有用户 ID 值和身份验证令牌的用户对象。若要使用此用户 ID 来授权用户，请参阅[服务器端授权]。若要将表访问权限限制给已经过身份验证的用户，请参阅[权限]。

###  服务器托管登录

以下是可以将服务器托管登录添加到[移动服务快速入门]项目的方式；你可以为其他项目使用类似的代码。

[AZURE.INCLUDE [mobile-services-ios-authenticate-app](../includes/mobile-services-ios-authenticate-app.md)]

###  客户端托管登录（单一登录）

你可以在移动服务客户端外部执行登录过程来启用单一登录，或者使应用程序能够直接联系标识提供者。在这种情况下，你可以通过提供单独从受支持标识提供者获取的令牌来登录到移动服务。

以下示例使用 [Live Connect SDK] 为 iOS 应用程序启用单一登录。该示例假设你前面在控制器中创建了名为 `liveClient` 的 **LiveConnectClient** 实例，并且用户已登录。
	
```
	[client loginWithProvider:@"microsoftaccount" 
		token:@{@"authenticationToken" : self.liveClient.session.authenticationToken}
		completion:^(MSUser *user, NSError *error) {
				// Handle success and errors
	}];
```

## <a name="caching-tokens"></a>如何：缓存身份验证令牌

了解如何在[移动服务快速入门]项目中缓存令牌；你可以对任何项目应用类似的步骤。[AZURE.INCLUDE [mobile-services-ios-authenticate-app-with-token](../includes/mobile-services-ios-authenticate-app-with-token.md)]

## <a name="errors"></a>如何：处理错误

调用移动服务时，完成块将包含 `NSError *error` 参数。如果出错，此参数为非 nil 值。你应该在代码中检查此参数，并根据需要处理错误。

[`<WindowsAzureMobileServices/MSError.h>`](https://github.com/Azure/azure-mobile-services/blob/master/sdk/iOS/src/MSError.h) 文件定义 `MSErrorResponseKey`、`MSErrorRequestKey` 和 `MSErrorServerItemKey` 常量以获取有关错误的更多数据。此外，该文件还定义每个错误代码的常量。有关如何使用这些常量的示例，请参阅[冲突处理程序]，以了解 `MSErrorServerItemKey` 和 `MSErrorPreconditionFailed` 的用法。

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
[移动服务快速入门]: /documentation/articles/mobile-services-ios-get-started/
[Get started with Mobile Services]: /documentation/articles/mobile-services-ios-get-started/
[Mobile Services SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533

[iOS SDK]: https://developer.apple.com/xcode

[Handling Expired Tokens]: http://go.microsoft.com/fwlink/p/?LinkId=301955
[Live Connect SDK]: http://go.microsoft.com/fwlink/p/?LinkId=301960
[权限]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj193161.aspx
[服务器端授权]: /documentation/articles/mobile-services-javascript-backend-service-side-authorization/
[动态架构]: http://go.microsoft.com/fwlink/p/?LinkId=296271
[创建表]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj193162.aspx
[NSDictionary object]: http://go.microsoft.com/fwlink/p/?LinkId=301965
[ASCII control codes C0 and C1]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set
[CLI to manage Mobile Services tables]: /documentation/articles/virtual-machines-command-line-tools/#Mobile_Tables
[冲突处理程序]: /documentation/articles/mobile-services-ios-handling-conflicts-offline-data/#add-conflict-handling
 

<!---HONumber=Mooncake_0215_2016-->