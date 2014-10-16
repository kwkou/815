<properties linkid="mobile-services-how-to-html-client" urlDisplayName="HTML Client" pageTitle="How to use an HTML client - Azure Mobile Services" metaKeywords="Azure Mobile Services, Mobile Service HTML client, HTML client" description="Learn how to use an HTML client for Azure Mobile Services." metaCanonical="" services="" documentationCenter="Mobile" title="How to use an HTML/JavaScript client for Azure Mobile Services" authors="krisragh" solutions="" manager="" editor="" />

# 如何使用适用于 Azure 移动服务的 HTML/JavaScript 客户端

<div class="dev-center-tutorial-selector sublanding"> 
  <a href="/zh-cn/develop/mobile/how-to-guides/work-with-net-client-library/" title=".NET Framework">.NET Framework</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-html-js-client/" title="HTML/JavaScript" class="current">HTML/JavaScript</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-ios-client-library/" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-android-client-library/" title="Android">Android</a><a href="/zh-cn/develop/mobile/how-to-guides/work-with-xamarin-client-library/" title="Xamarin">Xamarin</a>
</div>

本指南说明如何使用适用于 Azure 移动服务的 HTML/JavaScript 客户端执行常见任务。所述的任务包括：查询数据，插入、更新和删除数据，对用户进行身份验证和处理错误。如果你是第一次使用移动服务，最好先完成移动服务 [Windows 应用商店 JavaScript 快速入门][]或 [HTML 快速入门][]。快速入门教程可帮助你配置帐户并创建第一个移动服务。

## 目录

-   [什么是移动服务][]
-   [概念][]
-   [如何：创建移动服务客户端][]
-   [如何：从移动服务查询数据][]

    -   [筛选返回的数据][]
    -   [为返回的数据排序][]
    -   [在页中返回数据][]
    -   [选择特定的列][]
    -   [按 ID 查找数据][]
-   [如何：在移动服务中插入数据][]
-   [如何：在移动服务中修改数据][]
-   [如何：在移动服务中删除数据][]
-   [如何：在用户界面中显示数据][]
-   [如何：对用户进行身份验证][]
-   [如何：处理错误][]
-   [如何：使用约定][]
-   [如何：自定义请求标头][]
-   [如何：使用跨域资源共享][]
-   [后续步骤][]

[WACOM.INCLUDE [mobile-services-concepts](../includes/mobile-services-concepts.md)]

<a name="create-client"></a>
## 创建移动服务客户端如何：创建移动服务客户端

以下代码将实例化移动服务客户端对象。

在 Web 编辑器中打开 HTML 文件，然后将以下代码添加到页的脚本引用中：

            <script src='http://ajax.aspnetcdn.com/ajax/mobileservices/MobileServices.Web-1.1.2.min.js'></script>

在编辑器中，先打开或创建一个 JavaScript 文件，再添加以下代码以定义 `MobileServiceClient` 变量，然后在 `MobileServiceClient` 构造函数中提供来自移动服务的应用程序 URL 和应用程序密钥。必须将占位符 `AppUrl` 替换为移动服务的应用程序 URL，将 `AppKey` 替换为应用程序密钥。若要了解如何获取移动服务的应用程序 URL 和应用程序密钥，请查阅教程 [Windows 应用商店 JavaScript 中的数据处理入门][]或 [HTML/JavaScript 中的数据处理入门][]。

            var MobileServiceClient = WindowsAzure.MobileServiceClient;
    var client = new MobileServiceClient('AppUrl', 'AppKey');

<a name="querying"></a>
## 查询数据如何：从移动服务查询数据

访问或修改 SQL Database 表中数据的所有代码都将对 `MobileServiceTable` 对象调用函数。对 `MobileServiceClient` 的实例调用 `getTable()` 函数可获取对表的引用。

            var todoItemTable = client.getTable('todoitem');

<a name="filtering"></a>
### 如何：筛选返回的数据

以下代码演示了如何通过在查询中包含 `where` 子句来筛选数据。该代码将返回 complete 字段等于 `false` 的 `todoItemTable` 中的所有项。`todoItemTable` 是对前面创建的移动服务表的引用。where 函数针对该表将一个行筛选谓词应用到查询。该函数接受使用 JSON 对象或者使用定义行筛选器的函数作为参数，并返回可进一步撰写的查询。

            var query = todoItemTable.where({
    complete:false
    }).read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

通过在 Query 对象中添加调用 `where` 并传递一个对象作为参数，我们可以指示移动服务仅返回 `complete` 列包含 `false` 值的行。另外，请查看以下请求 URI，可以看出，我们正在修改查询字符串本身：

            GET /tables/todoitem?$filter=(complete+eq+false) HTTP/1.1           

可以使用消息检查软件（例如浏览器开发人员工具或 Fiddler）来查看发送到移动服务的请求的 URI。

在服务器端，此请求通常会粗略地转换成以下 SQL 查询：

            SELECT * 
    FROM TodoItem           
    WHERE ISNULL(complete, 0) = 0

传递给 `where` 方法的对象可以包含任意数目的参数，所有这些参数都将解释为查询的 AND 子句。例如，以下行：

            query.where({
    complete:false,
    assignee:"david",
    difficulty:"medium"
    }).read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

将粗略地转换为（针对前面显示的同一请求）

            SELECT * 
    FROM TodoItem 
    WHERE ISNULL(complete, 0) = 0
    AND assignee = 'david'
    AND difficulty = 'medium'

上述 `where` 语句和上述 SQL 查询将查找分配给“david”、难度为“medium”的不完整项。

不过，还可以通过另一种方法来编写相同的查询。对 Query 对象的 `.where` 调用将在 `WHERE` 子句中添加一个 `AND` 表达式，因此，我们也可以在三个行中编写该查询：

            query.where({
    complete:false
            });
    query.where({
    assignee:"david"
            });
    query.where({
    difficulty:"medium"
            });

或者使用 Fluent API：

            query.where({
    complete:false
            })
    .where({
    assignee:"david"
            })
    .where({
    difficulty:"medium"
            });

这两种方法是等效的，可以换用。到目前为止，所有 `where` 调用都使用了带有某些参数的对象，并且会根据数据库中的数据比较相等性。但是，查询方法的另一个重载使用函数而不是对象。在这种情况下，我们可以在此函数中使用“不等于”等运算符和其他关系运算来编写更复杂的表达式。在这些函数中，关键字 `this` 将绑定到服务器对象。

函数的正文转换为 OData 布尔表达式，该表达式将传递给查询字符串参数。可以传入不带参数的函数，例如：

            query.where(function () {
    return this.assignee == "david" && (this.difficulty == "medium" || this.difficulty == "low");
    }).read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

如果传入带参数的函数，则 `where` 子句后面的所有参数都将按顺序绑定到函数参数。来自函数范围以外的任何对象都必须作为参数传递 - 函数无法捕获任何外部变量。在接下来的两个示例中，参数“david”将绑定到第一个示例中的参数 `name`，参数“medium”也绑定到参数 `level`。另外，函数必须包括带有受支持表达式的单个 `return` 语句，例如：

             query.where(function (name, level) {
    return this.assignee == name && this.difficulty == level;
    }, "david", "medium").read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
             });

因此，只要我们遵守规则，就能将更复杂的筛选器添加到数据库查询，例如：

            query.where(function (name) {
    return this.assignee == name &&

    (this.difficulty == "medium" || this.difficulty == "low");
    }, "david").read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

可以将 `where` 与 `orderBy`、`take` 和 `skip` 相组合。有关详细信息，请参阅下一部分。

<a name="sorting"></a>
### 如何：为返回的数据排序

以下代码演示了如何通过在查询中包含 `orderBy` 或 `orderByDescending` 函数来为数据排序。该代码将返回 `todoItemTable` 中的项，这些项已按 `text` 字段的升序排序。默认情况下，服务器只返回前 50 个元素。

<div class="dev-callout"><b>说明</b>

<p>默认情况下，将使用服务器驱动的页大小来防止返回所有元素。这可以防止对大型数据集发出的默认请求对服务造成负面影响。</p>
</div>

> 你可以根据下一部分中所述，通过调用 `take` 来增加返回的项数。`todoItemTable` 是对前面创建的移动服务表的引用。

            var ascendingSortedTable = todoItemTable.orderBy("text").read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

    var descendingSortedTable = todoItemTable.orderByDescending("text").read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

    var descendingSortedTable = todoItemTable.orderBy("text").orderByDescending("text").read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

<a name="paging"></a>
### 如何：在页中返回数据

以下代码演示了如何通过在查询中使用 `take` 和 `skip` 子句来实现返回数据的分页。执行以下查询后，将返回表中的前三个项。

            var query = todoItemTable.take(3).read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

请注意，`take(3)` 方法已转换成查询 URI 中的查询选项 `$top=3`。

以下经过修改的查询将跳过前三个结果，返回其后的三个结果。实际上这是数据的第二“页”，其页大小为三个项。

            var query = todoItemTable.skip(3).take(3).read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

同样，你可以查看发送到移动服务的请求的 URI。请注意，`skip(3)` 方法已转换成查询 URI 中的查询选项 `$skip=3`。

这是将硬编码分页值传递给 `take` 和 `skip` 函数的简化方案。在实际应用程序中，你可以对页导航控件或类似的 UI 使用类似于上面的查询，让用户导航到上一页和下一页。

<a name="selecting"></a>
### 如何：选择特定的列

你可以通过在查询中添加 `select` 子句来指定要包含在结果中的属性集。例如，以下代码将返回 `todoItemTable` 的每一行中的 `id`、`complete` 和 `text` 属性：

            var query = todoItemTable.select("id", "complete", "text").read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            })

此处，select 函数的参数是要返回的表列的名称。

到目前为止所述的所有函数都是加性函数，我们可以不断地调用它们，每次调用都能进一步影响查询。再提供一个示例：

            query.where({
    complete:false
            })
    .select('id', 'assignee')
    .orderBy('assignee')
    .take(10)
    .read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);

<a name="lookingup"></a>
### 如何：按 ID 查找数据

`lookup` 函数只取 `id` 值，它可以从数据库中返回具有该 ID 的对象。数据库表是使用整数或字符串 `id` 列创建的。默认使用字符串 `id` 列。

            todoItemTable.lookup("37BBF396-11F0-4B39-85C8-B319C729AF6D").done(function (result) {
    alert(JSON.stringify(result));
    }, function (err) {
    alert("Error:" + err);
            })

<a name="inserting"></a>
## 插入数据如何：在移动服务中插入数据

以下代码演示了如何在表中插入新行。客户端通过将 POST 请求发送到移动服务来请求插入数据行。请求正文包含要插入的、JSON 对象形式的数据。

            todoItemTable.insert({
    text:"New Item",
    complete:false
            })

以上代码会将提供的 JSON 对象中的数据插入到表中。你还可以指定完成插入时要调用的回调函数：

            todoItemTable.insert({
    text:"New Item",
    complete:false
    }).done(function (result) {
    alert(JSON.stringify(result));
    }, function (err) {
    alert("Error:" + err);
            });

移动服务支持为表 ID 使用唯一的自定义字符串值。这样，应用程序便可为移动服务表的 ID 列使用自定义值（如电子邮件地址或用户名）。例如，如果你想要根据电子邮件地址识别每条记录，可以使用以下 JSON 对象。

            todoItemTable.insert({
    id:"myemail@domain.com",                
    text:"New Item",
    complete:false
            })

如果将新记录插入到表时未提供字符串 ID 值，移动服务将为 ID 生成唯一值。

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

<a name="modifying"></a>
## 修改数据如何：在移动服务中修改数据

以下代码演示了如何更新表中的数据。客户端通过将 PATCH 请求发送到移动服务来请求更新数据行。请求正文包含要更新的、JSON 对象形式的特定字段。该代码将更新表 `todoItemTable` 中的某个现有项。

            todoItemTable.update({
    id:idToUpdate,
    text:newText
            })

第一个参数使用 ID 指定了表中要更新的实例。

你还可以指定完成更新时要调用的回调函数：

            todoItemTable.update({
    id:idToUpdate,
    text:newText
    }).done(function (result) {
    alert(JSON.stringify(result));
    }, function (err) {
    alert("Error:" + err);
            });

<a name="deleting"></a>
## 删除数据如何：在移动服务中删除数据

以下代码演示了如何删除表中的数据。客户端通过将 DELETE 请求发送到移动服务来请求删除数据行。该代码将删除表 todoItemTable 中的某个现有项。

            todoItemTable.del({
    id:idToDelete
            })

第一个参数使用 ID 指定了表中要删除的实例。

你还可以指定完成删除时要调用的回调函数：

            todoItemTable.del({
    id:idToDelete
    }).done(function () {
    /* Do something */
    }, function (err) {
    alert("Error:" + err);
            }); 

<a name="binding"></a>
## 显示数据如何：在用户界面中显示数据

本部分说明如何使用 UI 元素显示返回的数据对象。若要查询 `todoItemTable` 中的项并在极简单的列表中显示这些项，可以运行以下示例代码。这里未执行任何形式的选择、筛选或排序操作。

            var query = todoItemTable;
        
    query.read().then(function (todoItems) {
    // The space specified by 'placeToInsert' is an unordered list element <ul> ...</ul>
    var listOfItems = document.getElementById('placeToInsert');
    for (var i = 0; i < todoItems.length; i++) {
    var li = document.createElement('li');
    var div = document.createElement('div');
    div.innerText = todoItems[i].text;
    li.appendChild(div);
    listOfItems.appendChild(li);
               }
    }).read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

在 Windows 应用商店应用程序中，可以使用查询的结果来创建 [WinJS.Binding.List] 对象，该对象可绑定为 [ListView][] 对象的数据源。有关详细信息，请参阅[数据绑定（使用 JavaScript 和 HTML 的 Windows 应用商店应用程序）][]。

<a name="caching"></a>
## 身份验证如何：对用户进行身份验证

移动服务支持使用各种外部标识提供者对应用程序用户进行身份验证和授权，这些提供者包括：Facebook、Google、Microsoft 帐户和 Twitter。你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。你还可以在服务器脚本中使用已经过身份验证的用户的标识来实施授权规则。有关详细信息，请参阅[身份验证入门][]教程。

支持两种身份验证流：*服务器流*和*客户端流*。服务器流依赖于提供者的 Web 身份验证界面，因此可提供最简便的身份验证体验。客户端流依赖于提供者和设备特定的 SDK，因此允许与设备特定的功能（例如单一登录）进行更深入的集成。

### 服务器流

若要让移动服务管理 Windows 应用商店或 HTML5 应用程序中的身份验证过程，
必须将你的应用程序注册到标识提供者。然后，需要在移动服务中配置提供者提供的应用程序 ID 和机密。有关详细信息，请参阅“身份验证入门”教程（[Windows 应用商店][]/[HTML][身份验证入门]）。

注册标识提供者后，只需结合提供者的 [MobileServiceAuthenticationProvider] 值调用 [LoginAsync 方法]。例如，若要使用 Facebook 登录，请使用以下代码。

        client.login("facebook").done(function (results) {
    alert("You are now logged in as:" + results.userId);
    }, function (err) {
    alert("Error:" + err);
        });

如果使用的标识提供者不是 Facebook，请将传递给上述 `login` 方法的值更改为下列其中一项：`microsoftaccount`、`facebook`、`twitter`、`google` 或 `windowsazureactivedirectory`。

在此情况下，移动服务将通过以下方式管理 OAuth 2.0 身份验证流：显示选定提供者的登录页，并在用户成功使用标识提供者登录后生成移动服务身份验证令牌。[login][] 函数在完成后将返回一个 JSON 对象 ("user")，该对象分别在 "userId" 和 "authenticationToken" 字段中公开用户 ID 和移动服务身份验证令牌。你可以缓存此令牌，并在它过期之前重复使用。有关详细信息，请参阅“缓存身份验证令牌”。

<div class="dev-callout"><b>Windows 应用商店应用程序</b>

<p>当你使用 Microsoft 帐户登录提供程序对 Windows 应用商店应用程序的用户进行身份验证时，还应该将应用程序包注册到移动服务。将 Windows 应用商店应用程序包信息注册到移动服务后，客户端可以重复使用 Microsoft 帐户登录凭据来提供单一登录体验。如果你不执行此操作，则每次调用 login 方法时，系统都会向 Microsoft 帐户登录用户显示登录提示。若要了解如何注册 Windows 应用商店应用程序包，请参阅<a href="/zh-cn/develop/mobile/how-to-guides/register-windows-store-app-package/" target="_blank">注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证</a>。将程序包信息注册到移动服务后，请为 <em>useSingleSignOn</em> 参数提供 <b>true</b> 值以重复使用凭据，方便调用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=322050" target="_blank">login</a> 方法。</p>
</div>

### 客户端流

你的应用程序还能够独立联系标识提供者，然后将返回的令牌提供给移动服务以进行身份验证。使用此客户端流可为用户提供单一登录体验，或者从标识提供者中检索其他用户数据。

以下示例使用 Live SDK，该 SDK 使用 Microsoft 帐户来支持 Windows 应用商店应用程序的单一登录：

        WL.login({ scope:"wl.basic"}).then(function (result) {
    client.login(
    "microsoftaccount", 
    {"authenticationToken":result.session.authentication_token})
    .done(function(results){
    alert("You are now logged in as:" + results.userId);
              },
    function(error){
    alert("Error:" + err);
              });
        });

这个简化的示例将从 Live Connect 获取一个令牌，并通过调用 [login][] 函数将该令牌提供给移动服务。有关如何使用 Microsoft 帐户提供单一登录体验的更完整示例，请参阅[使用单一登录对应用程序进行身份验证][]。

如果使用 Facebook 或 Google API 进行客户端身份验证，则需要对该示例略做更改。

        client.login(
    "facebook", 
    {"access_token":token})
    .done(function(results){
    alert("You are now logged in as:" + results.userId);
    }, function (err) {
    alert("Error:" + err);
        });

此示例假设相应提供程序 SDK 提供的令牌存储在 `token` 变量中。
目前无法将 Twitter 用于客户端身份验证。

<a name="errors"></a>
### 缓存身份验证令牌

在某些情况下，完成首次用户身份验证后，可以避免调用 login 方法。我们可以使用 [sessionStorage][] 或 [localStorage][sessionStorage] 来缓存当前用户首次登录时使用的标识，以后每次该用户登录时，系统都会检查缓存中是否存在该用户标识。如果缓存为空或者调用失败（意味着当前登录会话已过期），则用户仍然需要完成整个登录过程。

        // After logging in
    sessionStorage.loggedInUser = JSON.stringify(client.currentUser);

    // Log in 
    if (sessionStorage.loggedInUser) {
    client.currentUser = JSON.parse(sessionStorage.loggedInUser);
    } else {
    // Regular login flow
       }

    // Log out
    client.logout();
    sessionStorage.loggedInUser = null;

<a name="promises"></a>
## 错误处理如何：处理错误

在移动服务中，你可能会遇到各种形式的错误，并且可以通过多种方式来验证和解决这些错误。

例如，你可以在移动服务中注册服务器脚本，然后使用这些脚本对所要插入和更新的数据执行各种操作，包括验证和数据修改。你可以按如下所示定义并注册一个用于验证和修改数据的服务器脚本：

            function insert(item, user, request) {
    if (item.text.length > 10) {
    request.respond(statusCodes.BAD_REQUEST, { error:"Text cannot exceed 10 characters" });
    } else {
    request.execute();
               }
            }

此服务器端脚本将验证发送到移动服务的字符串数据长度，并拒绝过长（在本例中为 10 个字符以上）的字符串。

由于移动服务能够在服务器端验证数据和发送错误响应，因此你可以更新你的 HTML 应用程序，使其能够处理验证后生成的错误响应。

            todoItemTable.insert({
    text:itemText,
    complete:false
        })
    .then(function (results) {
    alert(JSON.stringify(results));
    }, function (error) {
    alert(JSON.parse(error.request.responseText).error);
        });

更明白地说，每次执行数据访问时，你都可以传入错误处理程序作为第二个参数：

            function handleError(message) {
    if (window.console && window.console.error) {
    window.console.error(message);
               }
            }

    client.getTable("tablename").read().then(function (data) { /* do something */ }, handleError);

<a name="customizing"></a>
## 约定如何：使用约定

约定提供了一种机制，让你基于尚未计算的值安排有待完成的工作。它是用于管理与异步 API 的交互的抽象。

每当提供给 `done` 约定的函数已成功完成或者收到错误时，就会立即执行该约定。与 `then` 约定不同，它肯定会引发无法在函数内部处理的错误；当处理程序完成执行后，此函数将引发当某个约定处于错误状态时返回的错误。有关详细信息，请参阅 [done][]。

            promise.done(onComplete, onError);

例如：

            var query = todoItemTable;
    query.read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

`then` 约定与 `done` 约定类似，而与 `then` 约定的不同之处在于，`done` 肯定会引发无法在函数内部处理的错误。如果你不向 `then` 提供错误处理程序，当操作出错时，then 不会引发异常，而是返回一个处于错误状态的约定。有关详细信息，请参阅 [then][]。

            promise.then(onComplete, onError).done( /* Your success and error handlers */ );

例如：

            var query = todoItemTable;
    query.read().done(function (results) {
    alert(JSON.stringify(results));
    }, function (err) {
    alert("Error:" + err);
            });

你可以通过许多不同的方式使用约定。你可以通过对前一个 `then` 函数返回的约定调用 `then` 或 `done` 来链接约定操作。对操作的中间阶段使用 `then`（例如 `.then().then()`），对操作的最终阶段使用 `done`（例如 `.then().then().done()`）。你可以链接多个 `then` 函数，因为 `then` 返回约定。无法链接多个 `done` 方法，因为该方法返回 undefined。[详细了解 then 和 done 之间的差别][]。

            todoItemTable.insert({
    text:"foo"
    }).then(function (inserted) {
    inserted.newField = 123;
    return todoItemTable.update(inserted);
    }).done(function (insertedAndUpdated) {
    alert(JSON.stringify(insertedAndUpdated));
            })

<a name="hostnames"></a>
## 自定义请求标头如何：自定义客户端请求标头

你可以使用 `withFilter` 函数发送自定义请求标头，以便读取和写入筛选器中即将发送的请求的任意属性。如果服务器端脚本需要自定义的 HTTP 标头或者可以使用这种标头增强自身，则你可能需要添加这样的标头。

            var client = new WindowsAzure.MobileServiceClient('https://your-app-url', 'your-key')
    .withFilter(function (request, next, callback) {
    request.headers.MyCustomHttpHeader = "Some value";
    next(request, callback);
            });

筛选器的作用远远不只是自定义请求标头，它们还可用于检查或更改请求、检查或更改响应、绕过网络调用、发送多个调用，等等。

<a name="hostnames"></a>
## 使用 CORS如何：使用跨域资源共享

若要控制允许与移动服务交互以及向其发送请求的网站，请确保使用“配置”选项卡将用于托管移动服务的网站主机名添加到跨域资源共享 (CORS) 允许列表。你可以根据需要使用通配符。默认情况下，新的移动服务将指示浏览器只能允许来自 `localhost` 的访问，跨域资源共享 (CORS) 允许外部主机名上的浏览器中运行的 JavaScript 代码与移动服务交互。对于 WinJS 应用程序，不需要使用此配置。

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
  [Windows 应用商店 JavaScript 快速入门]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started
  [HTML 快速入门]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-html
  [什么是移动服务]: #what-is
  [概念]: #concepts
  [如何：创建移动服务客户端]: #create-client
  [如何：从移动服务查询数据]: #querying
  [筛选返回的数据]: #filtering
  [为返回的数据排序]: #sorting
  [在页中返回数据]: #paging
  [选择特定的列]: #selecting
  [按 ID 查找数据]: #lookingup
  [如何：在移动服务中插入数据]: #inserting
  [如何：在移动服务中修改数据]: #modifying
  [如何：在移动服务中删除数据]: #deleting
  [如何：在用户界面中显示数据]: #binding
  [如何：对用户进行身份验证]: #caching
  [如何：处理错误]: #errors
  [如何：使用约定]: #promises
  [如何：自定义请求标头]: #customizing
  [如何：使用跨域资源共享]: #hostnames
  [后续步骤]: #nextsteps
  [mobile-services-concepts]: ../includes/mobile-services-concepts.md
  [Windows 应用商店 JavaScript 中的数据处理入门]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-with-data-js
  [HTML/JavaScript 中的数据处理入门]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started-with-data-html/
  [ASCII 控制代码 C0 和 C1]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set
  [用于管理移动服务表的 CLI]: http://www.windowsazure.com/zh-cn/manage/linux/other-resources/command-line-tools/#Mobile_Tables
  [ListView]: http://msdn.microsoft.com/zh-cn/library/windows/apps/br211837.aspx
  [数据绑定（使用 JavaScript 和 HTML 的 Windows 应用商店应用程序）]: http://msdn.microsoft.com/zh-cn/library/windows/apps/hh758311.aspx
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-html
  [Windows 应用商店]: /zh-cn/develop/mobile/tutorials/get-started-with-users-js
  [login]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554236.aspx
  [注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证]: /zh-cn/develop/mobile/how-to-guides/register-windows-store-app-package/
  [1]: http://go.microsoft.com/fwlink/p/?LinkId=322050
  [使用单一登录对应用程序进行身份验证]: /zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-dotnet/
  [sessionStorage]: http://msdn.microsoft.com/zh-cn/library/cc197062(v=vs.85).aspx
  [done]: http://msdn.microsoft.com/zh-cn/library/windows/apps/hh701079.aspx
  [then]: http://msdn.microsoft.com/zh-cn/library/windows/apps/br229728.aspx
  [详细了解 then 和 done 之间的差别]: http://msdn.microsoft.com/zh-cn/library/windows/apps/hh700334.aspx
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started-html
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-html
  [使用脚本验证和修改数据]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-html
  [使用分页优化查询]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-html
  [使用脚本为用户授权]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-html
