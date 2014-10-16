<properties linkid="obile-services-how-to-dotnet-client" urlDisplayName=".NET Client Library" pageTitle="Working with the Mobile Services .NET Client Library" metaKeywords="Azure Mobile Services, Mobile Service .NET client, .NET client" description="Learn how to use an .NET client for Azure Mobile Services." metaCanonical="" services="" documentationCenter="Mobile" title="How to use a .NET client for Azure Mobile Services" authors="krisragh" solutions="" manager="" editor="" />

# 如何使用适用于 Azure 移动服务的 .NET 客户端

<div class="dev-center-tutorial-selector sublanding"> 
  <a href="/zh-cn/develop/mobile/how-to-guides/work-with-net-client-library/" title=".NET Framework" class="current">.NET Framework</a>
  	<a href="/zh-cn/develop/mobile/how-to-guides/work-with-html-js-client/" title="HTML/JavaScript">HTML/JavaScript</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-ios-client-library/" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-android-client-library/" title="Android">Android</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-xamarin-client-library/" title="Xamarin">Xamarin</a>
</div>

本指南说明如何在 Windows 应用商店应用程序和 Windows Phone 应用程序中使用适用于 Azure 移动服务的 .NET 客户端执行常见任务。所述的任务包括：查询数据，插入、更新和删除数据，对用户进行身份验证和处理错误。如果你是第一次使用移动服务，最好先完成“移动服务快速入门”教程（[Windows 应用商店快速入门教程][]/[Windows Phone 快速入门教程][]）和“.NET 中的数据处理入门”教程（[Windows 应用商店数据教程][]/[Windows Phone 数据教程][]）。快速入门教程要求安装[移动服务 SDK][]，它可帮助你配置帐户并创建第一个移动服务。

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
-   [如何：使用乐观并发][]
-   [如何：在移动服务中将数据绑定到用户界面][]
-   [如何：对用户进行身份验证][]
-   [如何：处理错误][]
-   [如何：处理非类型化数据][]
-   [如何：设计单元测试][]
-   [如何：自定义客户端][]

    -   [自定义请求标头][]
    -   [自定义序列化][]
-   [后续步骤][]

[WACOM.INCLUDE [mobile-services-concepts](../includes/mobile-services-concepts.md)]

<a name="setup"></a>
## 安装安装与先决条件

假设你已创建一个移动服务和一个表。有关详细信息，请参阅[创建表][]。在本主题使用的代码中，表的名称为 `TodoItem`，其中包含以下列：`Id`、`Text` 和 `Complete`。

相应的类型化客户端 .NET 类型如下：

    public class TodoItem
    {
    public string Id { get; set; }

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

> [WACOM.NOTE] 默认情况下，将使用服务器驱动的页大小来防止返回所有行。这可以防止对大型数据集发出的默认请求对服务造成负面影响。若要返回 50 个以上的行，请根据[在页中返回数据][]中所述使用 `Take` 方法。

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

以下代码演示了如何通过在查询中包含 `OrderBy` 或 `OrderByDescending` 函数来为数据排序。该代码将返回 `todoTable` 中的项，这些项已按 `Text` 字段的升序排序。

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

默认情况下，服务器只返回前 50 行。你可以通过调用 [Take][] 方法来增加返回的行数。将 `Take` 与 [Skip][] 方法一起使用可以请求查询返回的总数据集的特定“页”。执行以下查询后，将返回表中的前三个项。

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

    // This query filters out the item with the ID of 37BBF396-11F0-4B39-85C8-B319C729AF6D
    TodoItem item = await todoTable.LookupAsync("37BBF396-11F0-4B39-85C8-B319C729AF6D");

<a name="inserting"></a>
## 插入数据如何：在移动服务中插入数据

<div class="dev-callout"><b>说明</b>

<p>如果你想要对某个类型执行插入、查找、删除或更新操作，则需要创建一个名为 <b>Id</b> 的成员。正因如此，示例类 <b>TodoItem</b> 包含了一个名为 <b>Id</b> 的成员。更新和删除操作中始终必须存在一个有效的 ID 值。</p>
</div>

以下代码演示了如何在表中插入新行。参数包含要作为 .NET 对象插入的数据。

    await todoTable.InsertAsync(todoItem);

如果在传递给 `todoTable.InsertAsync` 调用的 `todoItem` 中未包含唯一的自定义 ID 值，则返回到客户端的 `todoItem` 对象中设置的服务器将会生成一个 ID 值。

移动服务支持为表 ID 使用唯一的自定义字符串值。这样，应用程序便可为移动服务表的 ID 列使用自定义值（如电子邮件地址或用户名）。如果将新记录插入到表时未提供字符串 ID 值，移动服务将为 ID 生成唯一值。

支持字符串 ID 为开发人员带来了以下优势

-   无需往返访问数据库即可生成 ID。
-   更方便地合并不同表或数据库中的记录。
-   ID 值能够更好地与应用程序的逻辑相集成。

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

-   控制字符：[0x0000-0x001F] 和 [0x007F-0x009F]。有关详细信息，请参阅 [ASCII 控制代码 C0 和 C1][]。
-   可打印字符："""(0x0022)、"+" (0x002B)、"/" (0x002F)、"?"(0x003F)、"\\" (0x005C)、"\`" (0x0060)
-   ID“.”和“..”

也可以为表使用整数 ID。若要使用整数 ID，必须使用 `mobile table create` 命令并结合 `--integerId` 选项创建表。应在适用于 Azure 的命令行界面 (CLI) 中使用此命令。有关使用 CLI 的详细信息，请参阅[用于管理移动服务表的 CLI][]。

若要插入非类型化数据，你可以按如下所示利用 Json.NET。

    JObject jo = new JObject(); 
    jo.Add("Text", "Hello World"); 
    jo.Add("Complete", false);
    var inserted = await table.InsertAsync(jo);

以下示例使用电子邮件地址作为唯一的字符串 ID。

    JObject jo = new JObject(); 
    jo.Add("id", "myemail@emaildomain.com"); 
    jo.Add("Text", "Hello World"); 
    jo.Add("Complete", false);
    var inserted = await table.InsertAsync(jo);

<a name="modifying"></a>
## 修改数据如何：在移动服务中修改数据

以下代码演示了如何使用新的信息更新具有相同 ID 的现有实例。参数包含要作为 .NET 对象更新的数据。

    await todoTable.UpdateAsync(todoItem);

若要插入非类型化数据，你可以按此方式利用 Json.NET。请注意，在执行更新时，必须指定 ID，移动服务将凭此 ID 来识别要更新的实例。可以从 `InsertAsync` 调用的结果中获取该 ID。

    JObject jo = new JObject(); 
    jo.Add("Id", "37BBF396-11F0-4B39-85C8-B319C729AF6D");
    jo.Add("Text", "Hello World"); 
    jo.Add("Complete", false);
    var inserted = await table.UpdateAsync(jo);
            

如果你尝试更新某个项但未提供“ID”值，则服务无法识别要更新的实例，从而导致移动服务 SDK 引发 `ArgumentException`。

<a name="deleting"></a>
## 删除数据如何：在移动服务中删除数据

以下代码演示了如何删除现有实例。该实例由 `todoItem` 中设置的“Id”字段标识。

    await todoTable.DeleteAsync(todoItem);

若要删除非类型化数据，你可以按此方式利用 Json.NET。请注意，在执行删除请求时，必须指定 ID，移动服务将凭此 ID 来识别要删除的实例。删除请求只需要 ID；其他属性将不会传递给服务，如果传递了任何属性，服务会将其忽略。`DeleteAsync` 调用的结果通常也是 `null`。可以从 `InsertAsync` 调用的结果中获取要传入的 ID。

    JObject jo = new JObject(); 
    jo.Add("Id", "37BBF396-11F0-4B39-85C8-B319C729AF6D");
    await table.DeleteAsync(jo);
            

如果你尝试删除某个项但尚未设置“Id”字段，则服务无法识别要删除的实例，因此你会收到服务发出的 `MobileServiceInvalidOperationException`。同样，如果你尝试删除某个非类型化项但尚未设置“Id”字段，则也会收到服务发出的 `MobileServiceInvalidOperationException`。

<a name="optimisticconcurrency"></a>
## 乐观并发如何：使用乐观并发

在某些情况下，两个或两个以上客户端可能会同时将更改写入同一项目。如果没有任何冲突检测，则最后一次写入会覆盖任何以前的更新，即使这并不是所需要的结果。乐观并发控制假定每个事务均可以提交，因此不使用任何资源锁定。提交事务之前，乐观并发控制将验证是否没有其他事务修改了数据。如果数据已修改，则将回滚正在提交的事务。

移动服务通过使用 `__version` 系统属性列（该列是为移动服务创建的每个表定义的）跟踪对每个项的更改来支持乐观并发控制。每次更新某个记录时，移动服务都将该记录的 `__version` 属性设置为新值。在每次执行更新请求期间，会将该请求包含的记录的 `__version` 属性与服务器上的记录的同一属性进行比较。如果随请求传递的版本与服务器不匹配，则移动服务 .NET 客户端库将引发 `MobileServicePreconditionFailedException<T>`。该异常中提供的类型就是包含记录服务器版本的服务器中的记录。然后，应用程序可以借助此信息来确定是否要使用服务器中正确的 `__version` 值再次执行更新请求以提交更改。

为了启用乐观并发，应用程序将在表类中为 `__version` 系统属性定义一个列。以下定义就是一个示例。

    public class TodoItem
    {
    public string Id { get; set; }

    [JsonProperty(PropertyName = "text")]
    public string Text { get; set; }

    [JsonProperty(PropertyName = "complete")]
    public bool Complete { get; set; }

    // "* Enable Optimistic Concurrency "* //
    [JsonProperty(PropertyName = "__version")]
    public byte[] Version { set; get; }
    }

使用非类型化表的应用程序通过在表的 `SystemProperties` 中设置 `Version` 标志来启用乐观并发，如下所示。

    //Enable optimistic concurrency by retrieving __version 
    todoTable.SystemProperties |= MobileServiceSystemProperties.Version;

以下代码演示了如何解决检测到的写入冲突。若要提交解决方法，必须在 `UpdateAsync()` 调用中包含正确的 `__version` 值。

    private async void UpdateToDoItem(TodoItem item)
    {
    MobileServicePreconditionFailedException<TodoItem> exception = null;         
        
    try
        {
    //update at the remote table
    await todoTable.UpdateAsync(item);
        }
    catch (MobileServicePreconditionFailedException<TodoItem> writeException)
        {
    exception = writeException;
        }
        
    if (exception != null)
        {
    // Conflict detected, the item has changed since the last query
    // Resolve the conflict between the local and server item
    await ResolveConflict(item, exception.Item);
        }
    }


    private async Task ResolveConflict(TodoItem localItem, TodoItem serverItem)
    {
    //Ask user to choose the resoltion between versions
    MessageDialog msgDialog = new MessageDialog(String.Format("Server Text:\"{0}\" \nLocal Text:\"{1}\"\n", 
    serverItem.Text, localItem.Text), 
    "CONFLICT DETECTED - Select a resolution:");

    UICommand localBtn = new UICommand("Commit Local Text");
    UICommand ServerBtn = new UICommand("Leave Server Text");
    msgDialog.Commands.Add(localBtn);
    msgDialog.Commands.Add(ServerBtn);          

    localBtn.Invoked = async (IUICommand command) =>
        {
    // To resolve the conflict, update the version of the 
    // item being committed.Otherwise, you will keep
    // catching a MobileServicePreConditionFailedException.
    localItem.Version = serverItem.Version;             

    // Updating recursively here just in case another 
    // change happened while the user was making a decision
    UpdateToDoItem(localItem);
        };          
        
    ServerBtn.Invoked = async (IUICommand command) =>
        {
    RefreshTodoItems();
        };          
        
    await msgDialog.ShowAsync();
    }

有关使用移动服务乐观并发的更完整示例，请参阅[乐观并发教程][]。

<a name="binding"></a>
## 显示数据如何：在移动服务中将数据绑定到用户界面

本部分说明如何使用 UI 元素显示返回的数据对象。若要查询 `todoTable` 中的不完整项并在极简单的列表中显示这些项，可以运行以下示例代码，以使用查询绑定列表源。使用 `MobileServiceCollection` 可以创建移动服务感知型绑定集合。

    // This query filters out completed TodoItems. 
    MobileServiceCollection<TodoItem, TodoItem> items = await todoTable
    .Where(todoItem => todoItem.Complete == false)
    .ToCollectionAsync();

    // itemsControl is an IEnumerable that could be bound to a UI list control
    IEnumerable itemsControl  = items;            
            
    // Bind this to a ListBox
    ListBox lb = new ListBox();
    lb.ItemsSource = items;

Windows 运行时中的某些控件支持名为 [ISupportIncrementalLoading][] 的接口。当用户滚动浏览时，此接口允许控件请求更多的数据。系统通过 `MobileServiceIncrementalLoadingCollection`（可自动处理来自控件的调用）为这个适用于 Windows 应用商店应用程序的接口提供内置支持。若要在 Windows 应用商店应用程序中使用 `MobileServiceIncrementalLoadingCollection`，请执行以下代码：

            MobileServiceIncrementalLoadingCollection<TodoItem,TodoItem> items;
    items =  todoTable.Where(todoItem => todoItem.Complete == false)
    .ToIncrementalLoadingCollection();

    ListBox lb = new ListBox();
    lb.ItemsSource = items;

若要在 Windows Phone 上使用新集合，请在 `IMobileServiceTableQuery<T>` 和 `IMobileServiceTable<T>` 中使用 `ToCollection` 扩展方法。若要实际加载数据，请调用 `LoadMoreItemsAsync()`。

    MobileServiceCollection<TodoItem, TodoItem> items = todoTable.Where(todoItem => todoItem.Complete==false).ToCollection(); 
    await items.LoadMoreItemsAsync();         

当你使用通过调用 `ToCollectionAsync` 或 `ToCollection` 创建的集合时，将获取一个可以绑定到 UI 控件的集合。此集合支持分页，也就是说，控件可以要求该集合“加载更多项”，而该集合也确实会这样做。这样看来，无需执行任何用户代码，控件就能启动工作流。但是，由于集合要从网络加载数据，因此可以预料到这种加载有时会失败。若要处理这种失败，你可以重写 `MobileServiceIncrementalLoadingCollection` 中的 `OnException` 方法，以处理调用控件执行的 `LoadMoreItemsAsync` 后发生的异常。

最后，假设你的表包含许多字段，但你只想在控件中显示其中的某些字段。在这种情况下，你可以参考上面[选择特定的列][]部分中的指导，选择要在 UI 中显示的特定列。

<a name="authentication"></a>
## 身份验证如何：对用户进行身份验证

移动服务支持使用各种外部标识提供者对应用程序用户进行身份验证和授权，这些提供者包括：Facebook、Google、Microsoft 帐户、Twitter 和 Azure Active Directory。你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。你还可以在服务器脚本中使用已经过身份验证的用户的标识来实施授权规则。有关详细信息，请参阅“身份验证入门”教程（[Windows 应用商店][]/[Windows Phone][]）

支持两种身份验证流：*服务器流*和*客户端流*。服务器流依赖于提供者的 Web 身份验证界面，因此可提供最简便的身份验证体验。客户端流依赖于提供者和设备特定的 SDK，因此允许与设备特定的功能进行更深入的集成。

### 服务器流

若要让移动服务管理 Windows 应用商店或 Windows Phone 应用程序中的身份验证过程，
必须将你的应用程序注册到标识提供者。然后，需要在移动服务中配置提供者提供的应用程序 ID 和机密。有关详细信息，请参阅“身份验证入门”教程（[Windows 应用商店][]/[Windows Phone][]）。

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

<div class="dev-callout"><b>Windows 应用商店应用程序</b>

<p>当你使用 Microsoft 帐户登录提供程序对 Windows 应用商店应用程序的用户进行身份验证时，还应该将应用程序包注册到移动服务。将 Windows 应用商店应用程序包信息注册到移动服务后，客户端可以重复使用 Microsoft 帐户登录凭据来提供单一登录体验。如果你不执行此操作，则每次调用 login 方法时，系统都会向 Microsoft 帐户登录用户显示登录提示。若要了解如何注册 Windows 应用商店应用程序包，请参阅<a href="/zh-cn/develop/mobile/how-to-guides/register-windows-store-app-package/" target="_blank">注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证</a>。将程序包信息注册到移动服务后，请为 *useSingleSignOn* 参数提供 <b>true</b> 值以重复使用凭据，方便调用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=311594" target="_blank">LoginAsync</a> 方法。</p>
</div>

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

如果使用 Microsoft 帐户，可按如下所示登录：

    // Replace authentication_token_value with actual value of your Microsoft authentication token obtained through the Live SDK
    user = await client
    .LoginWithMicrosoftAccountAsync(authentication_token_value);

有关如何使用 Microsoft 帐户提供单一登录体验的示例，请参阅“使用单一登录对应用程序进行身份验证”教程（[Windows 应用商店][1]/[Windows Phone][2]）。

<a name="caching"></a>
### 缓存身份验证令牌

在某些情况下，完成首次用户身份验证后，可以避免调用 login 方法。你可以使用适用于 Windows 应用商店应用程序的 [PasswordVault][] 来缓存当前用户首次登录时使用的标识，以后每次该用户登录时，系统都会检查缓存中是否存在该用户标识。如果缓存为空，则用户仍然需要完成整个登录过程。

        // After logging in
    PasswordVault vault = new PasswordVault();
    vault.Add(new PasswordCredential("Facebook", user.UserId, user.MobileServiceAuthenticationToken));

    // Log in 
    var creds = vault.FindAllByResource("Facebook").FirstOrDefault();
    if (creds != null)
    {
    user = new MobileServiceUser(creds.UserName);
    user.MobileServiceAuthenticationToken = vault.Retrieve("Facebook", creds.UserName).Password;
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
    vault.Remove(vault.Retrieve("Facebook", user.UserId));

对于 Windows Phone 应用程序，可以使用 [ProtectedData][] 类加密和缓存数据，并在隔离的存储中存储敏感信息。

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

.NET 客户端在设计上支持强类型化方案。但有时，松散类型化的体验可为用户带来方便；例如，在处理采用开放架构的对象时，可能就需要这种体验。可按如下所示启用这种方案。在查询中，先指定 LINQ 语句并使用有线格式。

    // Get an untyped table reference
    IMobileServiceTable untypedTodoTable = client.GetTable("TodoItem");         

    // Lookup untyped data using OData
    JToken untypedItems = await untypedTodoTable.ReadAsync("$filter=complete eq 0&$orderby=text");

此时，你将获取一些可以像属性包一样使用的 JSON 值。有关 JToken 和 Json.NET 的详细信息，请参阅 [Json.NET][]

<a name="unit-testing"></a>
## 设计测试如何：设计单元测试

`MobileServiceClient.GetTable` 返回的值和查询是接口。你可以针对测试目的轻松“模拟”这些接口，以便创建一个实现测试逻辑的 `MyMockTable :IMobileServiceTable<TodoItem>`。

<a name="customizing"></a>
## 自定义客户端如何：自定义客户端

<a name="headers"></a>
### 如何：自定义请求标头

你可能需要将一个自定义标头附加到每个传出请求，或者更改响应状态代码。按如下所示配置 DelegatingHandler 便可实现此目的：

    public async Task CallClientWithHandler()
    {
    MobileServiceClient client = new MobileServiceClient( 
    "AppUrl", 
    "AppKey" ,
    new MyHandler()
            ); 
    IMobileServiceTable<TodoItem> todoTable = client.GetTable<TodoItem>();
    var newItem = new TodoItem { Text = "Hello world", Complete = false };
    await table.InsertAsync(newItem);
    }
             
    public class MyHandler :DelegatingHandler
    {
    protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
    request.Headers.Add("x-my-header", "my value");
    var response = awaitbase.SendAsync(request, cancellationToken);
    response.StatusCode = HttpStatusCode.ServiceUnavailable;
    return response;
        }
    }
            
            
<a name="serialization"></a>
### 如何：自定义序列化

[MobileServiceClient][] 类公开类型 [JsonSerializerSettings][] 的 `SerializerSettings` 属性

使用此属性可以设置 Json.NET 属性（这样的属性有很多），其中包括一个用于将所有属性转换为小写的属性，例如：

    var settings = new JsonSerializerSettings();
    settings.ContractResolver = new CamelCasePropertyNamesContractResolver();
    client.SerializerSettings = settings;

<a name="nextsteps"></a>
## 后续步骤

完成这篇概念性的操作方法参考主题后，请详细了解如何在移动服务中执行重要任务：

-   [移动服务入门][]
    了解有关移动服务用法的基本知识。

-   [数据处理入门][]
    了解有关使用移动服务存储和查询数据的详细信息。

-   [身份验证入门][]
    了解如何使用标识提供者对应用程序的用户进行身份验证。

-   [使用脚本验证和修改数据][]
    了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

-   [使用分页优化查询][]
    了解如何使用查询中的分页控制单个请求中处理的数据量。

-   [使用脚本为用户授权][]
    了解如何采用移动服务基于已进行身份验证的用户提供的用户 ID 值，并使用该值来筛选移动服务返回的数据。

  [.NET Framework]: /zh-cn/develop/mobile/how-to-guides/work-with-net-client-library/ ".NET Framework"
  [HTML/JavaScript]: /zh-cn/develop/mobile/how-to-guides/work-with-html-js-client/ "HTML/JavaScript"
  [iOS]: /zh-cn/develop/mobile/how-to-guides/work-with-ios-client-library/ "iOS"
  [Android]: /zh-cn/develop/mobile/how-to-guides/work-with-android-client-library/ "Android"
  [Xamarin]: /zh-cn/develop/mobile/how-to-guides/work-with-xamarin-client-library/ "Xamarin"
  [Windows 应用商店快速入门教程]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started/
  [Windows Phone 快速入门教程]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-wp8/
  [Windows 应用商店数据教程]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet/
  [Windows Phone 数据教程]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-with-data-wp8/
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
  [如何：使用乐观并发]: #optimisticconcurrency
  [如何：在移动服务中将数据绑定到用户界面]: #binding
  [如何：对用户进行身份验证]: #authentication
  [如何：处理错误]: #errors
  [如何：处理非类型化数据]: #untyped
  [如何：设计单元测试]: #unit-testing
  [如何：自定义客户端]: #customizing
  [自定义请求标头]: #headers
  [自定义序列化]: #serialization
  [后续步骤]: #nextsteps
  [mobile-services-concepts]: ../includes/mobile-services-concepts.md
  [创建表]: http://go.microsoft.com/fwlink/?LinkId=298592
  [动态架构]: http://go.microsoft.com/fwlink/?LinkId=296271
  [GetTable]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554275.aspx
  [Take]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn250574.aspx
  [Skip]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn250573.aspx
  [IncludeTotalCount]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn250560.aspx
  [ASCII 控制代码 C0 和 C1]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set
  [用于管理移动服务表的 CLI]: http://www.windowsazure.com/zh-cn/manage/linux/other-resources/command-line-tools/#Mobile_Tables
  [乐观并发教程]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/handle-database-write-conflicts-dotnet/
  [ISupportIncrementalLoading]: http://msdn.microsoft.com/zh-cn/library/windows/apps/Hh701916
  [Windows 应用商店]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet/
  [Windows Phone]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-with-users-wp8/
  [MobileServiceAuthenticationProvider]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceauthenticationprovider.aspx
  [LoginAsync 方法]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceclientextensions.loginasync.aspx
  [MobileServiceUser]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceuser.aspx
  [userId]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceuser.userid.aspx
  [MobileServiceAuthenticationToken]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceuser.mobileserviceauthenticationtoken.aspx
  [缓存身份验证令牌]: #caching
  [注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证]: /zh-cn/develop/mobile/how-to-guides/register-windows-store-app-package/
  [LoginAsync]: http://go.microsoft.com/fwlink/p/?LinkId=311594
  [1]: /zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-dotnet/
  [2]: /zh-cn/develop/mobile/tutorials/single-sign-on-wp8/
  [PasswordVault]: http://msdn.microsoft.com/zh-cn/library/windows/apps/windows.security.credentials.passwordvault.aspx
  [ProtectedData]: http://msdn.microsoft.com/zh-cn/library/system.security.cryptography.protecteddata%28VS.95%29.aspx
  [Json.NET]: http://json.codeplex.com/
  [MobileServiceClient]: http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.mobileservices.mobileserviceclient.aspx
  [JsonSerializerSettings]: http://james.newtonking.com/projects/json/help/?topic=html/T_Newtonsoft_Json_JsonSerializerSettings.htm
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet/
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet
  [使用脚本验证和修改数据]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-dotnet
  [使用分页优化查询]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-dotnet
  [使用脚本为用户授权]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-dotnet
