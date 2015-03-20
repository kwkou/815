<properties linkid="mobile-services-how-to-ios-client" urlDisplayName="iOS Client Library" pageTitle="如何使用 iOS 客户端库 - Azure 移动服务" metaKeywords="Azure Mobile Services, Mobile Service iOS client library, iOS client library" description="了解如何使用适用于 Azure 移动服务的 iOS 客户端库。" metaCanonical="" services="" documentationCenter="Mobile" title="How to use the iOS client library for Mobile Services" authors="glenga" solutions="" manager="" editor="" />





# 如何使用适用于移动服务的 iOS 客户端库
<div class="dev-center-tutorial-selector sublanding">
  <a href="/zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library/" title=".NET Framework">.NET 框架</a><a href="/zh-cn/documentation/articles/mobile-services-html-how-to-use-client-library/" title="HTML/JavaScript">HTML/JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-ios-how-to-use-client-library/" title="iOS" class="current">iOS</a><a href="/zh-cn/documentation/articles/mobile-services-android-how-to-use-client-library/" title="Android">Android</a><a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-how-to-use-client-library" title="Xamarin">Xamarin</a>
</div>

本指南说明如何使用适用于 Azure 移动服务的 iOS 客户端执行常见任务。这些示例用 Objective-C 编写，需要安装[移动服务 SDK]。本教程还需要安装[iOS SDK]。所述的任务包括：查询数据；插入、更新和删除数据；对用户进行身份验证和处理错误。如果你是第一次使用移动服务，最好先完成[移动服务快速入门][移动服务入门]。快速入门教程可帮助你配置帐户并创建第一个移动服务。

## 目录

- [什么是移动服务][]
- [概念][]
- [安装与先决条件][]
- [如何：创建移动服务客户端][]
- [如何：创建表引用][]
- [如何：从移动服务查询数据][]
	- [筛选返回的数据]
    - [使用 MSQuery 对象][]
	- [选择特定的列]
- [如何：在移动服务中插入数据]
- [如何：在移动服务中修改数据]
- [如何：将数据绑定到用户界面]
- [如何：对用户进行身份验证]
- [如何：处理错误]

<!--- [How to: Design unit tests]
- [How to: Customize the client]
	- [Customize request headers]
	- [Customize data type serialization]
- [Next steps][]-->

[WACOM.INCLUDE [mobile-services-concepts](../includes/mobile-services-concepts.md)]

##<a name="Setup"></a>安装与先决条件

本指南假设你已创建一个移动服务和一个表。有关详细信息，请参阅[创建表]，或再次使用[移动服务入门]教程中创建的 `ToDoItem`表。本主题中的示例使用名为 `ToDoItem`的表，该表包含以下列：

+ `id`
+ `text`
+ `complete`
+ `duration`


如果你是首次创建 iOS 应用程序，请确保在应用程序的[**将二进制文件链接到库**](https://developer.apple.com/library/ios/recipes/xcode_help-project_editor/Articles/AddingaLibrarytoaTarget.html)设置中添加  `WindowsAzureMobileServices.framework`。在此步骤中，单击"添加其他¡"，导航到下载 Windows Azure 移动服务 SDK 的位置，然后选择该位置。

此外，必须在相应的文件或者应用程序的 .pch 文件中添加以下引用。

	#import <WindowsAzureMobileServices/WindowsAzureMobileServices.h>

<h2><a name="create-client"></a>如何：创建移动服务客户端</h2>

以下代码将创建用于访问移动服务的移动服务客户端对象。

	MSClient *client = [MSClient clientWithApplicationURLString:@"MobileServiceUrl" applicationKey:@"AppKey"]

在上面的代码中，使用移动服务 URL 和你的移动服务的应用程序密钥替换 `MobileServiceUrl`和 `AppKey`。若要确定移动服务的这些设置，请在 Azure 管理门户中选择你的移动服务，然后单击**仪表板**。

你还可以按如下所示从**NSURL**对象（服务的 URL）创建客户端：

	MSClient *client = [MSClient clientWithApplicationURL:[NSURL URLWithString:@"MobileServiceUrl"] applicationKey:@"AppKey"];

<h2><a name="table-reference"></a>如何：创建表引用</h2>

在从移动服务访问数据之前，必须先获取对要执行项目查询、更新或删除操作的表的引用。在以下示例中， `ToDoItem`是表名称：

	MSTable *table = [client tableWithName:@"ToDoItem"];


<h2><a name="querying"></a>如何：从移动服务查询数据</h2>

创建 MSTable 对象后，便可以创建查询。下面的简单查询将获取 ToDoItem 表中的所有项。

	[table readWithCompletion:^(NSArray *items, NSInteger totalCount, NSError *error) {
		if(error) {
			NSLog(@"ERROR %@", error);
		} else {
			for(NSDictionary *item in items) {
				NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
			}
		}
	}];

请注意，在此情况下，只会将任务的文本写入日志。

回调中提供了以下参数：

+ _items_:与查询匹配的记录的**NSArray**。
+ _totalCount_:查询的所有页中的总项数，而不仅仅是在当前页中返回的项数。除非你在请求中显式请求了总计数，否则此值将设置为 -1。有关详细信息，请参阅[在页中返回数据]。
+ _error_:发生的任何错误；如果未发生错误，则为 `nil`.

### <a name="filtering"></a>如何：筛选返回的数据

你可以使用许多选项来筛选结果。 

最常用的选项是使用 NSPredicate 来筛选结果。

	[table readWithPredicate:(NSPredicate *)predicate completion:(MSReadQueryBlock)completion];

以下谓词只返回 ToDoItem 表中的不完整项：

	NSPredicate *predicate = [NSPredicate predicateWithFormat:@"complete == NO"];
	[table readWithPredicate:predicate completion:^(NSArray *items, NSInteger totalCount, NSError *error) {
		if(error) {
			NSLog(@"ERROR %@", error);
		} else {
			for(NSDictionary *item in items) {
				NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
			}
		}
	}];

可以使用 ID 来检索单个记录。

	[table readWithId:[@"37BBF396-11F0-4B39-85C8-B319C729AF6D"] completion:^(NSDictionary *item, NSError *error) {
		if(error) {
			NSLog(@"ERROR %@", error);
		} else {
				NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
		}
	}];

请注意，在此情况下，回调参数会略有不同。你不会获得结果数组和可选计数，而只会获得一条记录。

### <a name="query-object"></a>使用 MSQuery 对象

如果你需要创建一个更复杂的查询而不仅仅是使用它来筛选行（例如，你想要更改对结果的排序顺序或者限制获取的数据记录数），则可以使用**MSQuery**对象。下面两个示例演示了如何创建 MSQuery 对象实例：

    MSQuery *query = [table query];

    MSQuery *query = [table queryWithPredicate: [NSPredicate predicateWithFormat:@"complete == NO"]];

使用 MSQuery 对象可以控制以下查询行为：

* 指定返回结果的顺序。
* 限制返回的字段。
* 限制返回记录的个数。
* 指定是否在响应中包含总计数。
* 指定请求中的自定义查询字符串参数。

可以通过应用一个或多个函数来进一步定义查询。定义查询后，可通过调用**readWithCompletion**函数执行该查询。

#### <a name="sorting"></a>为返回的数据排序

使用以下函数来指定用于排序的字段：

	-(void) orderByAscending:(NSString *)field
	-(void) orderByDescending:(NSString *)field

此查询依次按照持续时间和任务的完成状态来为结果排序：

	[query orderByAscending:@"duration"];
	[query orderByAscending:@"complete"];
	[query readWithCompletion:^(NSArray *items, NSInteger totalCount, NSError *error) {
		//code to parse results here
	}];

#### <a name="paging"></a>在页中返回数据 

移动服务可以限制单个响应中返回的记录量。若要控制显示给用户的记录数，你必须实施一个分页系统。可以使用**MSQuery**对象的以下三个属性执行分页：

+	`BOOL includeTotalCount`
+	`NSInteger fetchLimit`
+	`NSInteger fetchOffset`


在以下示例中，一个简单的函数将请求从服务器返回 20 条记录，然后将这些记录追加到以前加载的记录的本地集合：

	- (bool) loadResults() {
		MSQuery *query = [table query];

		query.includeTotalCount = YES;
		query.fetchLimit = 20;
		query.fetchOffset = self.loadedItems.count;

		[query readWithCompletion:^(NSArray *items, NSInteger totalCount, NSError *error) {
			if(!error) {
				// Add the items to our local copy
				[self.loadedItems addObjectsFromArray:items];

				//set a flag to keep track if there are any additional records we need to load
				self.moreResults = (self.loadedItems.count < totalCount);
			}
		}];
	}

#### <a name="selecting"></a>限制返回的字段
 
若要限制查询返回的字段，只需在**selectFields**属性中指定所需字段的名称。以下示例只返回 text 和 completed 字段：

	query.selectFields = @[@"text", @"completed"];

#### <a name="parameters"></a>指定其他查询字符串参数

客户端库允许你在发送到服务器的请求中包含其他查询字符串参数。服务器端脚本可能需要这些参数。以下示例将两个查询字符串参数添加到请求：   

	query.parameters = @{
		@"myKey1" : @"value1",
		@"myKey2" : @"value2",
	};

这些参数追加到查询 URI 作为`myKey1=value1&myKey2=value2`。 
有关详细信息，请参阅[如何： 访问自定义参数]。

<h2><a name="inserting"></a>如何：在移动服务中插入数据</h2>

若要在表中插入新行，可以创建一个新的[NSDictionary 对象]并将该对象传递给 insert 函数。以下代码将在表中插入一个新的 todo 项：

	NSDictionary *newItem = @{@"text": @"my new item", @"complete" : @NO};
	[table insert:newItem completion:^(NSDictionary *result, NSError *error) {
		// The result contains the new item that was inserted,
		// depending on your server scripts it may have additional or modified 
		// data compared to what was passed to the server.
	}];	

移动服务支持为表 ID 使用唯一的自定义字符串值。这样，应用程序便可为移动服务表的 ID 列使用自定义值（如电子邮件地址或用户名）。例如，如果你想要根据电子邮件地址识别每条记录，可以使用以下 JSON 对象。

	NSDictionary *newItem = @{@"id": @"myemail@emaildomain.com", @"text": @"my new item", @"complete" : @NO};
	[table insert:newItem completion:^(NSDictionary *result, NSError *error) {
		// The result contains the new item that was inserted,
		// depending on your server scripts it may have additional or modified 
		// data compared to what was passed to the server.
	}];	

如果将新记录插入到表时未提供字符串 ID 值，移动服务将为 ID 生成唯一值。

支持字符串 ID 为开发人员带来了以下优势

+ 无需往返访问数据库即可生成 ID。
+ 更方便地合并不同表或数据库中的记录。
+ ID 值能够更好地与应用程序的逻辑相集成。

你也可以使用服务器脚本来设置 ID 值。下面的脚本示例将生成一个自定义 GUID 并将其分配给新记录的 ID。此 ID 类似于你未传入记录的 ID 值时，移动服务生成的 ID 值。

	//Example of generating an id. This is not required since Mobile Services
	//will generate an id if one is not passed in.
	item.id = item.id || newGuid();
	request.execute();

	function newGuid() {
		var pad4 = function(str) { return "0000".substring(str.length) + str; };
		var hex4 = function () { return pad4(Math.floor(Math.random() * 0x10000 /* 65536 */ ).toString(16)); };
		return (hex4() + hex4() + "-" + hex4() + "-" + hex4() + "-" + hex4() + "-" + hex4() + hex4() + hex4());
	}


如果应用程序提供了某个 ID 的值，移动服务将按原样存储该值，包括前导和尾随空格。不会从值中裁剪掉空格。

 `id` 的值必须唯一，并且不能包含以下集中的字符：

+ 控制字符：[0x0000-0x001F] 和 [0x007F-0x009F]。有关详细信息，请参阅 [ASCII 控制代码 C0 和 C1]。
+  可打印字符：**"**(0x0022), **\+** (0x002B), **/** (0x002F), **?**(0x003F), **\\** (0x005C), **`** (0x0060)
+  ID"."和".."

也可以为表使用整数 ID。为了使用整数 Id，您必须通过 `mobile table create`命令（使用`--integerId`选项）创建表。应在适用于 Azure 的命令行界面 (CLI) 中使用此命令。有关使用 CLI 的详细信息，请参阅[用于管理移动服务表的 CLI]。

启用动态架构后，移动服务将基于 insert 或 update 请求中对象的字段自动生成新列。有关详细信息，请参阅[动态架构]。

<h2><a name="modifying"></a>如何：在移动服务中修改数据</h2>

通过修改前一查询返回的项，然后调用**update**函数，便可更新现有的对象。

	NSMutableDictionary *item = [self.results.item objectAtIndex:0];
	[item setObject:@YES forKey:@"complete"];
	[table update:item completion:^(NSDictionary *item, NSError *error) {
		//handle errors or any additional logic as needed
	}];

进行更新时，你只需按以下示例中所示提供所要更新的字段以及行 ID：

	[table update:@{@"id" : @"37BBF396-11F0-4B39-85C8-B319C729AF6D", @"Complete": @YES} completion:^(NSDictionary *item, NSError *error) {
		//handle errors or any additional logic as needed
	}];


若要从表中删除一个项，只需按如下所示将该项传递给 delete 方法：

	[table delete:item completion:^(id itemId, NSError *error) {
		//handle errors or any additional logic as needed
	}];

你也可以直接使用记录的 ID 来做到只删除一条记录，如以下示例中所示：

	[table deleteWithId:@"37BBF396-11F0-4B39-85C8-B319C729AF6D" completion:^(id itemId, NSError *error) {
		//handle errors or any additional logic as needed
	}];

请注意，在执行更新和删除时，最起码需要设置 `id`属性。

<h2><a name="authentication"></a>如何：对用户进行身份验证</h2>

移动服务允许你使用以下标识提供者对用户进行身份验证：


- Google
- Microsoft 帐户
- 百度
- Azure Active Directory

有关配置标识提供者的详细信息，请参阅[身份验证入门]。 

移动服务支持以下两种身份验证工作流：

- 在服务器托管登录中，移动服务将代表应用程序管理登录过程。客户端库将显示提供程序特定的登录页，移动服务将使用所选的提供程序执行身份验证工作。

- 在客户端托管登录中，应用程序必须从标识提供者请求令牌，然后将此令牌提供给移动服务以进行身份验证。

如果身份验证成功，则返回包含分配的用户 ID 值和身份验证令牌的用户对象。你可以在服务器脚本中使用此用户 ID 来验证或修改请求。有关详细信息，请参阅[使用脚本为用户授权]。你可以安全缓存该令牌本身，以便在后续登录时使用。 

你还可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。有关详细信息，请参阅[权限]。

### 服务器托管登录

以下示例演示了如何使用 Microsoft 帐户登录。你可以在控制器的 ViewDidLoad 中调用此代码，或者从 UIButton 手动触发此代码。此代码将显示用于登录到标识提供者的标准 UI。

	[client loginWithProvider:@"MicrosoftAccount" controller:self animated:YES
		completion:^(MSUser *user, NSError *error) {
			NSString *msg;
			if(error) {
				msg = [@"An error occured: " stringByAppendingString:error.description];
			} else {
				msg = [@"You are logged in as " stringByAppendingString:user.userId];
			}
		
			UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Login" 
								  message:msg 
								  delegate:nil 
								  cancelButtonTitle:@"OK" 
								  otherButtonTitles: nil];
			[alert show];
	}];	

如果使用的标识提供者不是适用于 Microsoft 帐户的提供者，请将传递给上述 login 方法的值更改为下列其中一项： `facebook`、 `twitter`、 `google` 或  `windowsazureactivedirectory`。

你还可以使用以下代码自行获取并显示对 MSLoginController 的引用：

	-(MSLoginController *)loginViewControllerWithProvider:(NSString *)provider completion:(MSClientLoginBlock)completion;

### 客户端托管登录（单一登录）

在某些情况下，登录过程需要在移动服务客户端外部完成。当你想要启用单一登录功能，或者当你的应用程序必须直接联系标识提供者来获取用户信息时，你可以执行这种登录。在这种情况下，你可以通过提供单独从受支持标识提供者获取的令牌来登录到移动服务。

以下示例使用[Live Connect SDK] 为 iOS 应用程序启用单一登录。 
	
	[client loginWithProvider:@"microsoftaccount" 
		token:@{@"authenticationToken" : self.liveClient.session.authenticationToken}
		completion:^(MSUser *user, NSError *error) {
			self.loggedInUser = user;
			NSString *msg = [@"You are logged in as " stringByAppendingString:user.userId];
			UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Login" 
				message:msg 
				delegate:nil 
				cancelButtonTitle:@"OK" 
				otherButtonTitles: nil];
			[alert show];
	}];

此代码假设你前面在控制器中创建了名为 `liveClient`的**LiveConnectClient**实例，并且用户已登录。

###<a name="caching-tokens"></a>如何：缓存身份验证令牌

为了避免用户每次运行你的应用程序时都要进行身份验证，你可以在当前用户登录后缓存其标识。然后，你可以使用此信息直接创建用户并绕过登录过程。为此，必须在本地存储用户 ID 和身份验证令牌。在以下示例中，令牌将安全缓存在[KeyChain]中：

	- (NSMutableDictionary *) createKeyChainQueryWithClient:(MSClient *)client andIsSearch:(bool)isSearch
	{
		NSMutableDictionary *query = [[NSMutableDictionary alloc] init];
		[query setObject:(__bridge id)kSecClassInternetPassword forKey:(__bridge id)(kSecClass)];
		[query setObject:client.applicationURL.absoluteString forKey:(__bridge id)(kSecAttrServer)];
    	
		if(isSearch) {
			// Use the proper search constants, return only the attributes of the first match.
			[query setObject:(__bridge id)kSecMatchLimitOne forKey:(__bridge id)kSecMatchLimit];
			[query setObject:(id)kCFBooleanTrue forKey:(__bridge id)kSecReturnAttributes];
			[query setObject:(id)kCFBooleanTrue forKey:(__bridge id)kSecReturnData];
		}
    
		return query;
	}

	- (IBAction)loginUser:(id)sender {
		NSMutableDictionary *query = [self createKeyChainQueryWithClient:self.todoService.client andIsSearch:YES];
		CFDictionaryRef cfresult;

		OSStatus status = SecItemCopyMatching((__bridge CFDictionaryRef)query, (CFTypeRef *)&cfresult);
		if (status == noErr) {
			NSDictionary * result = (__bridge_transfer NSDictionary*) cfresult;
			
			//create an MSUser object
			MSUser *user = [[MSUser alloc] initWithUserId:[result objectForKey:(__bridge id)(kSecAttrAccount)]];
			NSData *data = [result objectForKey:(__bridge id)(kSecValueData)];
			user.mobileServiceAuthenticationToken = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
			[self.todoService.client setCurrentUser:user];
			
		} else if (status == errSecItemNotFound) {
			//we need to log the user in
			[self.todoService.client loginWithProvider:@"MicrosoftAccount" controller:self animated:YES
				completion:^(MSUser *user, NSError *error) {
					NSString *msg = [@"You are logged in as " stringByAppendingString:user.userId];
					UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Login" 
											message:msg delegate:nil 
											cancelButtonTitle:@"OK" 
											otherButtonTitles: nil];
					[alert show];
                
					//save the user id and token to the KeyChain
					NSMutableDictionary *newItem = [self createKeyChainQueryWithClient:self.todoService.client 
															andIsSearch:NO];
					[newItem setObject:user.userId forKey:(__bridge id)kSecAttrAccount];
					[newItem setObject:[user.mobileServiceAuthenticationToken dataUsingEncoding:NSUTF8StringEncoding] 
                                                    forKey:(__bridge id)kSecValueData];
                 
					OSStatus status = SecItemAdd((__bridge CFDictionaryRef)newItem, NULL);
					if(status != errSecSuccess) {
						//handle error as needed
						NSAssert(NO, @"Error caching password.");
					}
			}];
		}	

<div class="dev-callout"><strong>注意</strong>
<p>令牌属于敏感数据，必须以加密的形式存储，以防设备遗失或失窃。</p>
</div>

使用缓存的令牌时，在该令牌过期之前，用户不需要重新登录。如果用户尝试使用过期的令牌登录，系统将返回 401 未授权响应。此时，该用户必须重新登录以获取新令牌，而该令牌同样会得到缓存。如果使用筛选器，则不需要在应用程序调用移动服务时编写代码来处理过期的令牌。筛选器可让你截获对移动服务的调用以及来自移动服务的响应。筛选器中的代码将测试 401 响应，如果令牌已过期，则触发登录进程，然后重试执行生成 401 响应的请求。有关详细信息，请参阅[处理过期的令牌]。

<h2><a name="errors"></a>如何：处理错误</h2>

对移动服务执行调用时，完成块将包含 `NSError *error`参数。如果出错，将返回此参数的非 null 值。你应该在代码中检查此参数，并根据需要处理错误。

如果发生了错误，你可以通过在代码中包含 MSError.h 文件来获取更多信息。

    #import <WindowsAzureMobileServices/MSError.h>

此文件定义以下常量，你可以使用这些常量从`[error userInfo]`中访问更多数据：

+ **MSErrorResponseKey**: 与错误关联的 HTTP 响应数据
* **MSErrorRequestKey**: 与错误关联的 HTTP 请求数据

此外，将为每个错误代码定义一个常量。可以在 MSError.h 文件中找到这些代码的说明。

有关执行验证和处理任何错误的示例，请参阅[使用服务器脚本在移动服务中验证和修改数据]。在本主题中，服务器端验证是使用服务器脚本实现的。如果提交了无效的数据，则会返回错误响应，而此响应将由客户端处理。

<!--
<h2><a name="#unit-testing"></a>如何：设计单元测试</h2>

_(Optional) 本部分演示如何在使用客户端库时编写单元测试 (info from Yavor)。_

<h2><a name="#customizing"></a>如何：自定义客户端</h2>

_(Optional) 本部分演示如何发送自定义客户端行为。 _

###<a name="custom-headers"></a>如何：自定义请求标头

_(Optional) 本部分演示如何发送自定义请求标头。 _

有关详细信息，请参阅有关处理服务器端中的标头的新主题。

###<a name="custom-serialization"></a>如何：自定义序列化

_(Optional) 本部分演示如何使用属性自定义数据序列化的方式。 _

有关详细信息，请参阅有关处理服务器端中的标头的新主题。

## <a name="next-steps"></a>后续步骤
-->

<!-- Anchors. -->

[什么是移动服务]: #what-is
[概念]: #concepts
[安装与先决条件]: #Setup
[如何：创建移动服务客户端]: #create-client
[如何：创建表引用]: #table-reference
[如何：从移动服务查询数据]: #querying
[筛选返回的数据]: #filtering
[为返回的数据排序]: #sorting
[在页中返回数据]: #paging
[选择特定的列]: #selecting
[如何：将数据绑定到用户界面]: #binding
[如何：在移动服务中插入数据]: #inserting
[如何：在移动服务中修改数据]: #modifying
[如何：对用户进行身份验证]: #authentication
[缓存身份验证令牌]: #caching-tokens
[如何：将图像和大型文件上载]: #blobs
[如何：处理错误]: #errors
[如何：设计单元测试]: #unit-testing 
[如何：自定义客户端]: #customizing
[自定义请求标头]: #custom-headers
[自定义数据类型的序列化]: #custom-serialization
[后续步骤]: #next-steps
[使用 MSQuery 对象]: #query-object

<!-- Images. -->

<!-- URLs. -->
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-ios
[使用服务器脚本在移动服务中验证和修改数据]: /zh-cn/documentation/articles/mobile-services-ios-validate-modify-data-server-scripts
[移动服务 SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-ios
[iOS SDK]: https://developer.apple.com/xcode

[处理过期的令牌]: http://go.microsoft.com/fwlink/p/?LinkId=301955
[Live Connect SDK]: http://go.microsoft.com/fwlink/p/?LinkId=301960
[权限]: http://msdn.microsoft.com/library/windowsazure/jj193161.aspx
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-ios-authorize-users-in-scripts
[动态架构]: http://go.microsoft.com/fwlink/p/?LinkId=296271
[如何： 访问自定义参数]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts#access-headers
[创建表]: http://msdn.microsoft.com/library/windowsazure/jj193162.aspx
[NSDictionary 对象]: http://go.microsoft.com/fwlink/p/?LinkId=301965
[ASCII 控制代码 C0 和 C1]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set
[用于管理移动服务表的 CLI]: http://www.windowsazure.cn/zh-cn/documentation/articles/command-line-tools/#Mobile_Tables
