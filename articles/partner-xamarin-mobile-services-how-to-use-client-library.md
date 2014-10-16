<properties linkid="mobile-services-how-to-xamarin-client" urlDisplayName="Xamarin" pageTitle="How to use the Xamarin Component client - Azure Mobile Services feature guide" metaKeywords="Azure Mobile Services, Xamarin, iOS, Android, .NET client" description="Learn how to use the Xamarin Component client for Azure Mobile Services." metaCanonical="" disqusComments="1" umbracoNaviHide="0" title="How to use the Xamarin Component client for Azure Mobile Services" authors="" />

# 如何使用适用于 Azure 移动服务的 Xamarin 组件客户端

<div class="dev-center-tutorial-selector sublanding"> 
  <a href="/zh-cn/develop/mobile/how-to-guides/work-with-net-client-library/" title=".NET Framework">.NET Framework</a>
  	<a href="/zh-cn/develop/mobile/how-to-guides/work-with-html-js-client/" title="HTML/JavaScript">HTML/JavaScript</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-ios-client-library/" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-android-client-library/" title="Android">Android</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-xamarin-client-library/" title="Xamarin" class="current">Xamarin</a>
</div>

本指南演示如何在用于 iOS 和 Android 的 Xamarin 应用程序中使用针对 Azure 移动服务的 Xamarin 组件客户端来执行常见任务。所述的任务包括：查询数据，插入、更新和删除数据，对用户进行身份验证和处理错误。如果你是第一次使用移动服务，最好先完成“移动服务快速入门”教程 ([Xamarin.iOS][]/[Xamarin.Android][]) 以及“.NET 中的数据处理入门”教程 ([Xamarin.iOS][1]/[Xamarin.Android][2])。快速入门教程要求安装 [Xamarin][3]（即[移动服务 SDK][]），它可帮助你配置帐户并创建第一个移动服务。

## 目录

-   [什么是移动服务][]
-   [概念][]
-   [如何：创建移动服务客户端][]
-   [如何：创建表引用][]
-   [如何：从移动服务查询数据][]

    -   [筛选返回的数据][]
    -   [为返回的数据排序][]
    -   [在页中返回数据][]
    -   [选择特定的列][]
    -   [按 ID 查找数据][]
-   [如何：在移动服务中插入数据][]
-   [如何：在移动服务中修改数据][]
-   [如何：在移动服务中删除数据][]
-   [如何：对用户进行身份验证][]
-   [如何：处理错误][]
-   [如何：处理非类型化数据][]
-   [如何：设计单元测试][]
-   [后续步骤][]

[WACOM.INCLUDE [mobile-services-concepts](../includes/mobile-services-concepts.md)]

<a name="setup"></a>
## 安装安装与先决条件

假设你已创建一个移动服务和一个表。有关详细信息，请参阅[创建表][]。在本主题使用的代码中，表的名称为 `TodoItem`，其中包含以下列：`id`、`Text` 和 `Complete`。

相应的类型化客户端 .NET 类型如下：

    public class TodoItem
    {
    public string id { get; set; }

    [JsonProperty(PropertyName = "text")]
    public string Text { get; set; }

    [JsonProperty(PropertyName = "complete")]
    public bool Complete { get; set; }
    }

启用动态架构后，Azure 移动服务将基于 insert 或 update 请求中的对象自动生成新列。有关详细信息，请参阅[动态架构][]。

<a name="create-client"></a>
## 创建移动服务客户端如何：创建移动服务客户端

以下代码将创建用于访问移动服务的 `MobileServiceClient` 对象。

    MobileServiceClient client = new MobileServiceClient( 
    "AppUrl", 
    "AppKey" 
    ); 

在上面的代码中，请将 `AppUrl` 和 `AppKey` 依次替换为移动服务 URL 和应用程序密钥。在 Azure 管理门户中选择你的移动服务，然后单击“仪表板”即可获取这两个值。

<a name="instantiating"></a>
## 创建表引用如何：创建表引用

访问或修改移动服务表中数据的所有代码都将对 `MobileServiceTable` 对象调用函数。对 `MobileServiceClient` 的实例调用 [GetTable][] 函数可获取对表的引用。

    IMobileServiceTable<TodoItem> todoTable = 
    client.GetTable<TodoItem>();

这是类型化的序列化模型；请参阅下面有关[非类型化序列化模型][如何：处理非类型化数据]的介绍。

<a name="querying"></a>
## 查询数据如何：从移动服务查询数据

本部分介绍如何向移动服务发出查询。其中的小节介绍了排序、筛选和分页等不同操作。

<a name="filtering"></a>
### 如何：筛选返回的数据

以下代码演示了如何通过在查询中包含 `Where` 子句来筛选数据。该代码将返回 `todoTable` 中其 `Complete` 属性等于 `false` 的所有项。`Where` 函数针对该表将一个行筛选谓词应用到查询。

    // This query filters out completed TodoItems and 
    // items without a timestamp. 
    List<TodoItem> items = await todoTable
    .Where(todoItem => todoItem.Complete == false)
    .ToListAsync();

可以使用消息检查软件（例如浏览器开发人员工具或 Fiddler）来查看发送到移动服务的请求的 URI。从下面的请求 URI 中，可以看出我们正在修改查询字符串本身：

    GET /tables/todoitem?$filter=(complete+eq+false) HTTP/1.1                  

在服务器端，此请求通常会粗略地转换成以下 SQL 查询：

    SELECT * 
    FROM TodoItem           
    WHERE ISNULL(complete, 0) = 0
            

传递给 `Where` 方法的函数可以包含任意数目的条件。例如，以下行：

    // This query filters out completed TodoItems where Text isn't null
    List<TodoItem> items = await todoTable
    .Where(todoItem => todoItem.Complete == false
    && todoItem.Text != null)
    .ToListAsync();

将粗略地转换为（针对前面显示的同一请求）

    SELECT * 
    FROM TodoItem 
    WHERE ISNULL(complete, 0) = 0
    AND ISNULL(text, 0) = 0

上述 `where` 语句将查找 `Complete` 状态设置为 false 且 `Text` 不为 null 的项。

我们也可以使用多个行编写该代码：

    List<TodoItem> items = await todoTable
    .Where(todoItem => todoItem.Complete == false)
    .Where(todoItem => todoItem.Text != null)
    .ToListAsync();

这两种方法是等效的，可以换用。前一个选项（在一个查询中连接多个谓词）更为精简，也是我们推荐的方法。

`where` 子句支持可转换成移动服务 OData 子集的操作，其中包括关系运算符（==、!=、\<、\<=、\>、\>=）、数学运算符（+、-、/、\*、%）、数字精度（Math.Floor、Math.Ceiling）、字符串函数（Length、Substring、Replace、IndexOf、StartsWith、EndsWith）、日期属性（Year、Month、Day、Hour、Minute、Second）、对象的访问属性，以及组合了上述所有操作的表达式。

<a name="sorting"></a>
### 如何：为返回的数据排序

以下代码演示了如何通过在查询中包含 `OrderBy` 或 `OrderByDescending` 函数来为数据排序。该代码将返回 `todoTable` 中的项，这些项已按 `Text` 字段的升序排序。默认情况下，服务器只返回前 50 个元素。

"说明"

默认情况下，将使用服务器驱动的页大小来防止返回所有元素。这可以防止对大型数据集发出的默认请求对服务造成负面影响。

你可以通过调用下一节中介绍的 `Take` 增加要返回的项目数。

    // Sort items in ascending order by Text field
    MobileServiceTableQuery<TodoItem> query = todoTable
    .OrderBy(todoItem => todoItem.Text)       
    List<TodoItem> items = await query.ToListAsync();

    // Sort items in descending order by Text field
    MobileServiceTableQuery<TodoItem> query = todoTable
    .OrderByDescending(todoItem => todoItem.Text)       
    List<TodoItem> items = await query.ToListAsync();           

<a name="paging"></a>
### 如何：在页中返回数据

以下代码演示了如何通过在查询中使用 `Take` 和 `Skip` 子句来实现返回数据的分页。执行以下查询后，将返回表中的前三个项。

    // Define a filtered query that returns the top 3 items.
    MobileServiceTableQuery<TodoItem> query = todoTable
    .Take(3);                              
    List<TodoItem> items = await query.ToListAsync();           

以下经过修改的查询将跳过前三个结果，返回其后的三个结果。实际上这是数据的第二“页”，其页大小为三个项。

    // Define a filtered query that skips the top 3 items and returns the next 3 items.
    MobileServiceTableQuery<TodoItem> query = todoTable
    .Skip(3)
    .Take(3);                              
    List<TodoItem> items = await query.ToListAsync();
            

你还可以使用 [IncludeTotalCount][] 方法来确保查询获取应该返回的*所有*记录的总计数，并忽略指定的任何 take 分页/限制子句：

    query = query.IncludeTotalCount();

这是将硬编码分页值传递给 `Take` 和 `Skip` 方法的简化方案。在实际应用程序中，你可以对页导航控件或类似的 UI 使用类似于上面的查询，让用户导航到上一页和下一页。

<a name="selecting"></a>
### 如何：选择特定的列

你可以通过在查询中添加 `Select` 子句来指定要包含在结果中的属性集。例如，以下代码演示了如何做到只选择一个字段，以及如何选择并格式化多个字段：

    // Select one field -- just the Text
    MobileServiceTableQuery<TodoItem> query = todoTable
    .Select(todoItem => todoItem.Text);
    List<string> items = await query.ToListAsync();

    // Select multiple fields -- both Complete and Text info
    MobileServiceTableQuery<TodoItem> query = todoTable
    .Select(todoItem => string.Format("{0} -- {1}", todoItem.Text.PadRight(30), todoItem.Complete ?"Now complete!": "Incomplete!"));
    List<string> items = await query.ToListAsync();
            

到目前为止所述的所有函数都是加性函数，我们可以不断地调用它们，每次调用都能进一步影响查询。再提供一个示例：

    MobileServiceTableQuery<TodoItem> query = todoTable
    .Where(todoItem => todoItem.Complete == false)
    .Select(todoItem => todoItem.Text)
    .Skip(3).
    .Take(3);
    List<string> items = await query.ToListAsync();

<a name="lookingup"></a>
### 如何：按 ID 查找数据

使用 `LookupAsync` 函数可以查找数据库中具有特定 ID 的对象。

    // This query filters out the item with the ID of 25
    TodoItem item25 = await todoTable.LookupAsync(25);

<a name="inserting"></a>
## 插入数据如何：在移动服务中插入数据

<div class="dev-callout"><b>说明</b>

<p>如果你想要对某个类型执行插入、查找、删除或更新操作，则需要创建一个名为 <b>Id</b> 的成员（不考虑大小写）。正因如此，示例类 <b>TodoItem</b> 包含了一个名为 <b>Id</b> 的成员。在插入操作期间，ID 值不能设置为默认值以外的任何值；相比之下，在更新和删除操作中 ID 值应始终设置为非默认值并且应始终存在。</p>
</div>

以下代码演示了如何在表中插入新行。参数包含要作为 .NET 对象插入的数据。

    await todoTable.InsertAsync(todoItem);

在 await `todoTable.InsertAsync` 调用返回后，会将服务器中的对象 ID 填充到客户端中的 `todoItem` 对象中。

若要插入非类型化数据，你可以按如下所示利用 Json.NET。再次请注意，在插入对象时，不能指定 ID。

    JObject jo = new JObject(); 
    jo.Add("Text", "Hello World"); 
    jo.Add("Complete", false);
    var inserted = await table.InsertAsync(jo);

如果你尝试插入已设置“Id”字段的项目，你将收到服务发出的 `MobileServiceInvalidOperationException`。

<a name="modifying"></a>
## 修改数据如何：在移动服务中修改数据

以下代码演示了如何使用新的信息更新具有相同 ID 的现有实例。参数包含要作为 .NET 对象更新的数据。

    await todoTable.UpdateAsync(todoItem);

若要插入非类型化数据，你可以按此方式利用 Json.NET。请注意，在执行更新时，必须指定 ID，移动服务将凭此 ID 来识别要更新的实例。可以从 `InsertAsync` 调用的结果中获取该 ID。

    JObject jo = new JObject(); 
    jo.Add("Id", 52);
    jo.Add("Text", "Hello World"); 
    jo.Add("Complete", false);
    var inserted = await table.UpdateAsync(jo);
            

如果你尝试更新某个项但尚未设置“Id”字段，则服务无法识别要更新的实例，因此你会收到服务发出的 `MobileServiceInvalidOperationException`。同样，如果你尝试更新某个非类型化项但尚未设置“Id”字段，则也会收到服务发出的 `MobileServiceInvalidOperationException`。

<a name="deleting"></a>
## 删除数据如何：在移动服务中删除数据

以下代码演示了如何删除现有实例。该实例由 `todoItem` 中设置的“Id”字段标识。

    await todoTable.DeleteAsync(todoItem);

若要删除非类型化数据，你可以按此方式利用 Json.NET。请注意，在执行删除请求时，必须指定 ID，移动服务将凭此 ID 来识别要删除的实例。删除请求只需要 ID；其他属性将不会传递给服务，如果传递了任何属性，服务会将其忽略。`DeleteAsync` 调用的结果通常也是 `null`。可以从 `InsertAsync` 调用的结果中获取要传入的 ID。

    JObject jo = new JObject(); 
    jo.Add("Id", 52);
    await table.DeleteAsync(jo);
            

如果你尝试删除某个项但尚未设置“Id”字段，则服务无法识别要删除的实例，因此你会收到服务发出的 `MobileServiceInvalidOperationException`。同样，如果你尝试删除某个非类型化项但尚未设置“Id”字段，则也会收到服务发出的 `MobileServiceInvalidOperationException`。

<a name="authentication"></a>
## 身份验证如何：对用户进行身份验证

移动服务支持使用各种外部标识提供者对应用程序用户进行身份验证和授权，这些提供者包括：Facebook、Google、Microsoft 帐户、Twitter 和 Azure Active Directory。你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。你还可以在服务器脚本中使用已经过身份验证的用户的标识来实施授权规则。有关详细信息，请参阅“身份验证入门”教程 ([Xamarin.iOS][4]/[Xamarin.Android][5])。

支持两种身份验证流：*服务器流*和*客户端流*。服务器流依赖于提供者的 Web 身份验证界面，因此可提供最简便的身份验证体验。客户端流依赖于提供者和设备特定的 SDK，因此允许与设备特定的功能进行更深入的集成。

### 服务器流

若要让移动服务管理 Windows 应用商店或 Windows Phone 应用程序中的身份验证过程，
必须将你的应用程序注册到标识提供者。然后，需要在移动服务中配置提供者提供的应用程序 ID 和机密。有关详细信息，请参阅“身份验证入门”教程 ([Xamarin.iOS][4]/[Xamarin.Android][5])。

注册标识提供者后，只需结合提供者的 [MobileServiceAuthenticationProvider][] 值调用 [LoginAsync 方法][]。例如，以下代码将使用 Facebook 启动服务器流登录。

    private MobileServiceUser user;
    private async System.Threading.Tasks.Task Authenticate()
    {
    while (user == null)
        {
    string message;
    try
            {
    user = await client
    .LoginAsync(MobileServiceAuthenticationProvider.Facebook);
    message = 
    string.Format("You are now logged in - {0}", user.UserId);
            }
    catch (InvalidOperationException)
            {
    message = "You must log in. Login Required";
            }

    var dialog = new MessageDialog(message);
    dialog.Commands.Add(new UICommand("OK"));
    await dialog.ShowAsync();
        }
    }

如果使用的标识提供者不是 Facebook，请将上述 [MobileServiceAuthenticationProvider][] 的值更改为提供者的值。

在此情况下，移动服务将通过以下方式管理 OAuth 2.0 身份验证流：显示选定提供者的登录页，并在用户成功使用标识提供者登录后生成移动服务身份验证令牌。[LoginAsync 方法][]将返回 [MobileServiceUser][]，该类将提供已经过身份验证的用户的 [userId][]，以及 JSON Web 令牌 (JWT) 形式的 [MobileServiceAuthenticationToken][]。你可以缓存此令牌，并在它过期之前重复使用。有关详细信息，请参阅[缓存身份验证令牌][]。

### 客户端流

你的应用程序还能够独立联系标识提供者，然后将返回的令牌提供给移动服务以进行身份验证。使用此客户端流可为用户提供单一登录体验，或者从标识提供者中检索其他用户数据。

你可以根据以下代码段中所示，为 Facebook 或 Google 使用这种最简单形式的客户端流。

    var token = new JObject();
    // Replace access_token_value with actual value of your access token obtained 
    // using the Facebook or Google SDK.
    token.Add("access_token", "access_token_value");
            
    private MobileServiceUser user;
    private async System.Threading.Tasks.Task Authenticate()
    {
    while (user == null)
        {
    string message;
    try
            {
    // Change MobileServiceAuthenticationProvider.Facebook 
    // to MobileServiceAuthenticationProvider.Google if using Google auth.
    user = await client
    .LoginAsync(MobileServiceAuthenticationProvider.Facebook, token);
    message = 
    string.Format("You are now logged in - {0}", user.UserId);
            }
    catch (InvalidOperationException)
            {
    message = "You must log in. Login Required";
            }

    var dialog = new MessageDialog(message);
    dialog.Commands.Add(new UICommand("OK"));
    await dialog.ShowAsync();
        }
    }

<a name="caching"></a>
### 缓存身份验证令牌

在某些情况下，完成首次用户身份验证后，可以避免调用 login 方法。你可以使用本地安全存储（如 [Xamarin.Auth][]）缓存当前用户首次登录时使用的标识，以后每次该用户登录时，系统都会检查缓存中是否存在该用户标识。如果缓存为空，则用户仍然需要完成整个登录过程。

    using Xamarin.Auth;
    var accountStore = AccountStore.Create(); // Xamarin.iOS
    //var accountStore = AccountStore.Create(this); // Xamarin.Android

    // After logging in
    var account = new Account (user.UserId, new Dictionary<string,string> {{"token",user.MobileServiceAuthenticationToken}});
    accountStore.Save(account, "Facebook");

    // Log in 
    var accounts = accountStore.FindAccountsForService ("Facebook").ToArray();
    if (accounts.Count != 0)
    {
    user = new MobileServiceUser (accounts[0].Username);
    user.MobileServiceAuthenticationToken = accounts[0].Properties["token"];
    }
    else
    {
    // Regular login flow
    user = new MobileServiceuser( await client
    .LoginAsync(MobileServiceAuthenticationProvider.Facebook, token);
    var token = new JObject();
    // Replace access_token_value with actual value of your access token
    token.Add("access_token", "access_token_value");
    }
            
    // Log out
    client.Logout();
    accountStore.Delete(account, "Facebook");

<a name="errors"></a>
## 错误处理如何：处理错误

在移动服务中，你可能会遇到各种形式的错误，并且可以通过多种方式来验证和解决这些错误。

例如，你可以在移动服务中注册服务器脚本，然后使用这些脚本对所要插入和更新的数据执行各种操作，包括验证和数据修改。你可以按如下所示定义并注册一个用于验证和修改数据的服务器脚本：

    function insert(item, user, request) 
    {
    if (item.text.length > 10) {
    request.respond(statusCodes.BAD_REQUEST, { error:"Text cannot exceed 10 characters" });
    } else {
    request.execute();
       }
    }

此服务器端脚本将验证发送到移动服务的字符串数据长度，并拒绝过长（在本例中为 10 个字符以上）的字符串。

由于移动服务能够在服务器端验证数据和发送错误响应，因此你可以更新你的 .NET 应用程序，使其能够处理验证后生成的错误响应。

    private async void InsertTodoItem(TodoItem todoItem)
    {
    // This code inserts a new TodoItem into the database.When the operation completes
    // and Mobile Services has assigned an Id, the item is added to the CollectionView
    try
        {
    await todoTable.InsertAsync(todoItem);
    items.Add(todoItem);
        }
    catch (MobileServiceInvalidOperationException e)
        {
    // Handle error
        }
    }

<a name="untyped"></a>
## 处理非类型化数据如何：处理非类型化数据

Xamarin 组件客户端在设计上支持强类型化方案。但有时，松散类型化的体验可为用户带来方便；例如，在处理采用开放架构的对象时，可能就需要这种体验。可按如下所示启用这种方案。在查询中，先指定 LINQ 语句并使用有线格式。

    // Get an untyped table reference
    IMobileServiceTable untypedTodoTable = client.GetTable("TodoItem");         

    // Lookup untyped data using OData
    JToken untypedItems = await untypedTodoTable.ReadAsync("$filter=complete eq 0&$orderby=text");

此时，你将获取一些可以像属性包一样使用的 JSON 值。有关 JToken 和 Json.NET 的详细信息，请参阅 [Json.NET][]

<a name="unit-testing"></a>
## 设计测试如何：设计单元测试

`MobileServiceClient.GetTable` 返回的值和查询是接口。你可以针对测试目的轻松“模拟”这些接口，以便创建一个实现测试逻辑的 `MyMockTable :IMobileServiceTable<TodoItem>`。

<a name="nextsteps"></a>
## 后续步骤

完成这篇概念性的操作方法参考主题后，请详细了解如何在移动服务中执行重要任务：

-   移动服务入门 ([Xamarin.iOS][6]/[Xamarin.Android][7])
    了解移动服务使用方面的基础知识。

-   数据处理入门 ([Xamarin.iOS][8]/[Xamarin.Android][9])
    了解有关使用移动服务存储和查询数据的详细信息。

-   身份验证入门 ([Xamarin.iOS][10]/[Xamarin.Android][11])
    了解如何使用标识提供者对应用程序的用户进行身份验证。

-   使用脚本验证和修改数据 ([Xamarin.iOS][12]/[Xamarin.Android][13])
    了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

-   使用分页优化查询 ([Xamarin.iOS][14]/[Xamarin.Android][15])
    了解如何使用查询中的分页控制单个请求中处理的数据量。

-   使用脚本为用户授权 ([Xamarin.iOS][16]/[Xamarin.Android][17])
    了解如何采用移动服务基于已进行身份验证的用户提供的用户 ID 值，并使用该值来筛选移动服务返回的数据。

  [.NET Framework]: /zh-cn/develop/mobile/how-to-guides/work-with-net-client-library/ ".NET Framework"
  [HTML/JavaScript]: /zh-cn/develop/mobile/how-to-guides/work-with-html-js-client/ "HTML/JavaScript"
  [iOS]: /zh-cn/develop/mobile/how-to-guides/work-with-ios-client-library/ "iOS"
  [Android]: /zh-cn/develop/mobile/how-to-guides/work-with-android-client-library/ "Android"
  [Xamarin]: /zh-cn/develop/mobile/how-to-guides/work-with-xamarin-client-library/ "Xamarin"
  [Xamarin.iOS]: /zh-cn/develop/mobile/tutorials/get-started-xamarin-ios/
  [Xamarin.Android]: /zh-cn/develop/mobile/tutorials/get-started-xamarin-android/
  [1]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-ios/
  [2]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-android/
  [3]: http://xamarin.com/download/
  [移动服务 SDK]: http://nuget.org/packages/WindowsAzure.MobileServices/
  [什么是移动服务]: #what-is
  [概念]: #concepts
  [如何：创建移动服务客户端]: #create-client
  [如何：创建表引用]: #instantiating
  [如何：从移动服务查询数据]: #querying
  [筛选返回的数据]: #filtering
  [为返回的数据排序]: #sorting
  [在页中返回数据]: #paging
  [选择特定的列]: #selecting
  [按 ID 查找数据]: #lookingup
  [如何：在移动服务中插入数据]: #inserting
  [如何：在移动服务中修改数据]: #modifying
  [如何：在移动服务中删除数据]: #deleting
  [如何：对用户进行身份验证]: #authentication
  [如何：处理错误]: #errors
  [如何：处理非类型化数据]: #untyped
  [如何：设计单元测试]: #unit-testing
  [后续步骤]: #nextsteps
  [mobile-services-concepts]: ../includes/mobile-services-concepts.md
  [创建表]: http://go.microsoft.com/fwlink/?LinkId=298592
  [动态架构]: http://go.microsoft.com/fwlink/?LinkId=296271
  [GetTable]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554275.aspx
  [IncludeTotalCount]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj730933.aspx
  [4]: /zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-ios/
  [5]: /zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-android/
  [MobileServiceAuthenticationProvider]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceauthenticationprovider.aspx
  [LoginAsync 方法]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceclientextensions.loginasync.aspx
  [MobileServiceUser]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceuser.aspx
  [userId]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceuser.userid.aspx
  [MobileServiceAuthenticationToken]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceuser.mobileserviceauthenticationtoken.aspx
  [缓存身份验证令牌]: #caching
  [Xamarin.Auth]: https://components.xamarin.com/view/xamarin.auth
  [Json.NET]: http://json.codeplex.com/
  [6]: /zh-cn/develop/mobile/tutorials/get-started-xamarin-ios
  [7]: /zh-cn/develop/mobile/tutorials/get-started-xamarin-android
  [8]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-ios
  [9]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-android
  [10]: /zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-ios
  [11]: /zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-android
  [12]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-ios
  [13]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-android
  [14]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-xamarin-ios
  [15]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-xamarin-android
  [16]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-xamarin-ios
  [17]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-xamarin-android
