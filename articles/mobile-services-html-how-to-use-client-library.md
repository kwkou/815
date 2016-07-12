<properties
	pageTitle="如何使用 HTML 客户端 | Azure"
	description="了解如何使用适用于 Azure 移动服务的 HTML 客户端。"
	services="mobile-services"
	documentationCenter=""
	authors="ggailey777"
	manager="dwrede"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="01/26/2016"
	wacn.date="03/28/2016"/>

#  如何使用适用于 Azure 移动服务的 HTML/JavaScript 客户端

[AZURE.INCLUDE [mobile-services-selector-client-library](../includes/mobile-services-selector-client-library.md)]

## 概述

本指南说明如何使用适用于 Azure 移动服务的 HTML/JavaScript 客户端（包括 Windows 应用商店 JavaScript 和 PhoneGap/Cordova 应用程序）执行常见任务。所述的任务包括查询数据、插入、更新和删除数据、对用户进行身份验证和处理错误。如果你是第一次使用移动服务，最好先完成[移动服务快速入门](/documentation/articles/mobile-services-html-get-started/)。快速入门教程可帮助你配置帐户并创建第一个移动服务。

[AZURE.INCLUDE [mobile-services-concepts](../includes/mobile-services-concepts.md)]

## <a name="create-client"></a>如何创建移动服务客户端

添加移动服务客户端引用的方式取决于应用程序平台，其中包括：

- 对于 Web 的应用程序，请打开 HTML 文件，然后将以下代码添加到页的脚本引用中：

        <script src="http://ajax.aspnetcdn.com/ajax/mobileservices/MobileServices.Web-1.2.7.min.js"></script>

- 对于使用 JavaScript/HTML 编写的 Windows 应用商店应用程序，请将 **WindowsAzure.MobileServices.WinJS NuGet** 包添加到你的项目。

- 对于 PhoneGap 或 Cordova 应用程序，请在项目中添加[移动服务插件](https://github.com/Azure/azure-mobile-services-cordova)。此插件支持[推送通知](#push-notifications)。

在编辑器中，打开或创建一个 JavaScript 文件，添加以下代码以定义 `MobileServiceClient` 变量，然后在 `MobileServiceClient` 构造函数中按顺序提供移动服务的应用程序 URL 和应用程序密钥。

	var MobileServiceClient = WindowsAzure.MobileServiceClient;
    var client = new MobileServiceClient('AppUrl', 'AppKey');

必须将占位符 `AppUrl` 替换为移动服务的应用程序 URL，将 `AppKey` 替换为你从 [Azure 管理门户](http://manage.windowsazure.cn/)获取的应用程序密钥。

>[AZURE.IMPORTANT]应用程序密钥用于针对移动服务筛选出随机请求，将随应用程序一起分发。由于此密钥未加密，因此不能被认为是安全的。为确保安全访问你的移动服务数据，你必须改为在允许用户访问前对用户进行身份验证。有关详细信息，请参阅[如何：对用户进行身份验证](#authentication)。

## <a name="querying"></a>如何从移动服务查询数据

访问或修改 SQL 数据库表中数据的所有代码均将调用 `MobileServiceTable` 对象的函数。可通过对 `MobileServiceClient` 实例调用 `getTable()` 函数来获取对表的引用。

    var todoItemTable = client.getTable('todoitem');


###  <a name="filtering"></a>如何筛选返回的数据

以下代码演示了如何通过在查询中包含 `where` 子句来筛选数据。该代码将返回 complete 字段等于 `false` 的 `todoItemTable` 中的所有项。`todoItemTable` 是对前面创建的移动服务表的引用。where 函数针对该表将一个行筛选谓词应用到查询。该函数接受 JSON 对象或定义行筛选器的函数作为其参数，并返回可进一步编写的查询。

	var query = todoItemTable.where({
	    complete: false
	}).read().done(function (results) {
	    alert(JSON.stringify(results));
	}, function (err) {
	    alert("Error: " + err);
	});

通过在 Query 对象中调用 `where` 并传递一个对象作为参数，我们可以指示移动服务仅返回 `complete` 列包含 `false` 值的行。另外，请查看以下请求 URI，可以看出，我们正在修改查询字符串本身：

	GET /tables/todoitem?$filter=(complete+eq+false) HTTP/1.1

可以使用消息检查软件（例如浏览器开发人员工具或 Fiddler）来查看发送到移动服务的请求的 URI。

在服务器端，此请求通常会粗略地转换成以下 SQL 查询：

	SELECT *
	FROM TodoItem
	WHERE ISNULL(complete, 0) = 0

传递给 `where` 方法的对象可以包含任意数目的参数，所有这些参数都将解释为查询的 AND 子句。例如，以下行：

	query.where({
	   complete: false,
	   assignee: "david",
	   difficulty: "medium"
	}).read().done(function (results) {
	   alert(JSON.stringify(results));
	}, function (err) {
	   alert("Error: " + err);
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
	   complete: false
	});
	query.where({
	   assignee: "david"
	});
	query.where({
	   difficulty: "medium"
	});

或者使用 Fluent API：

	query.where({
	   complete: false
	})
	   .where({
	   assignee: "david"
	})
	   .where({
	   difficulty: "medium"
	});

这两种方法是等效的，可以换用。到目前为止，所有 `where` 调用都使用了带有某些参数的对象，并且会根据数据库中的数据比较相等性。但是，查询方法的另一个重载使用函数而不是对象。在这种情况下，我们可以在此函数中使用“不等于”等运算符和其他关系运算来编写更复杂的表达式。在这些函数中，关键字 `this` 将绑定到服务器对象。

函数的正文将转换为开放数据协议 (OData) 布尔表达式，该表达式将传递给查询字符串参数。可以传入不带参数的函数，例如：

    query.where(function () {
       return this.assignee == "david" && (this.difficulty == "medium" || this.difficulty == "low");
    }).read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });


如果传入带参数的函数，则 `where` 子句后面的所有参数都将按顺序绑定到函数参数。来自函数范围以外的任何对象都必须作为参数传递 - 函数无法捕获任何外部变量。在接下来的两个示例中，自变量“david”将绑定到参数 `name`，在第一个示例中，自变量“medium”也绑定到参数 `level`。另外，函数必须包含带受支持表达式的单个 `return` 语句，例如：

	 query.where(function (name, level) {
	    return this.assignee == name && this.difficulty == level;
	 }, "david", "medium").read().done(function (results) {
	    alert(JSON.stringify(results));
	 }, function (err) {
	    alert("Error: " + err);
	 });

因此，只要我们遵守规则，就能将更复杂的筛选器添加到数据库查询，例如：

    query.where(function (name) {
       return this.assignee == name &&
          (this.difficulty == "medium" || this.difficulty == "low");
    }, "david").read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);
    });

可以将 `where` 与 `orderBy`、`take` 和 `skip` 组合使用。有关详细信息，请参阅下一节。

###  <a name="sorting"></a>如何为返回的数据排序

以下代码演示了如何通过在查询中包含 `orderBy` 或 `orderByDescending` 函数来为数据排序。该代码将返回 `todoItemTable` 中的项，这些项已按 `text` 字段的升序排序。默认情况下，服务器只返回前 50 个元素。

> [AZURE.NOTE]默认情况下，将使用服务器驱动的页大小来防止返回所有元素。这可以防止对大型数据集发出的默认请求对服务造成负面影响。 
你可以根据下一节中所述，通过调用 `take` 来增加返回的项数。`todoItemTable` 是对前面创建的移动服务表的引用。

	var ascendingSortedTable = todoItemTable.orderBy("text").read().done(function (results) {
	   alert(JSON.stringify(results));
	}, function (err) {
	   alert("Error: " + err);
	});

	var descendingSortedTable = todoItemTable.orderByDescending("text").read().done(function (results) {
	   alert(JSON.stringify(results));
	}, function (err) {
	   alert("Error: " + err);
	});

	var descendingSortedTable = todoItemTable.orderBy("text").orderByDescending("text").read().done(function (results) {
	   alert(JSON.stringify(results));
	}, function (err) {
	   alert("Error: " + err);
	});

###  <a name="paging"></a>如何在页中返回数据

默认情况下，移动服务只在给定的请求中返回 50 行，除非客户端显式要求在响应中返回更多的数据。以下代码演示了如何通过在查询中使用 `take` 和 `skip` 子句来实现返回数据的分页。执行以下查询后，将返回表中的前三个项。

	var query = todoItemTable.take(3).read().done(function (results) {
	   alert(JSON.stringify(results));
	}, function (err) {
	   alert("Error: " + err);
	});

请注意，`take(3)` 方法已转换成查询 URI 中的查询选项 `$top=3`。

以下经过修改的查询将跳过前三个结果，返回其后的三个结果。实际上这是数据的第二“页”，其页大小为三个项。

	var query = todoItemTable.skip(3).take(3).read().done(function (results) {
	   alert(JSON.stringify(results));
	}, function (err) {
	   alert("Error: " + err);
	});

同样，你可以查看发送到移动服务的请求的 URI。请注意，`skip(3)` 方法已转换成查询 URI 中的查询选项 `$skip=3`。

这是将硬编码分页值传递给 `take` 和 `skip` 函数的简化方案。在实际应用中，你可以对页导航控件或类似的 UI 使用类似于上面的查询，让用户导航到上一页和下一页。

###  <a name="selecting"></a>如何选择特定的列

你可以通过在查询中添加 `select` 子句来指定要包含在结果中的属性集。例如，以下代码将从 `todoItemTable` 的每一行中返回 `id`、`complete` 和 `text` 属性：

	var query = todoItemTable.select("id", "complete", "text").read().done(function (results) {
	   alert(JSON.stringify(results));
	}, function (err) {
	   alert("Error: " + err);
	})

此处，select 函数的参数是要返回的表列的名称。


到目前为止所述的所有函数都是加性函数，我们可以不断地调用它们，每次调用都能进一步影响查询。再提供一个示例：

    query.where({
       complete: false
    })
       .select('id', 'assignee')
       .orderBy('assignee')
       .take(10)
       .read().done(function (results) {
       alert(JSON.stringify(results));
    }, function (err) {
       alert("Error: " + err);

###  <a name="lookingup"></a>如何：按 ID 查找数据

`lookup` 函数只使用 `id` 值，并从数据库返回具有该 ID 的对象。创建数据库表时使用了整数或字符串 `id` 列。默认使用字符串 `id` 列。

	todoItemTable.lookup("37BBF396-11F0-4B39-85C8-B319C729AF6D").done(function (result) {
	   alert(JSON.stringify(result));
	}, function (err) {
	   alert("Error: " + err);
	})

## <a name="odata-query"></a>执行 OData 查询操作

移动服务使用 OData 查询 URI 约定编写和执行 REST 查询。并非所有 OData 查询都可使用内置查询函数编写，尤其是复杂的筛选器操作，如搜索属性中的子字符串。对于这些类型的复杂查询，可以将任何有效的 OData 查询选项字符串传递给 `read` 函数，如下所示：

	function refreshTodoItems() {
	    todoItemTable.read("$filter=substringof('search_text',text)").then(function(items) {
	        var itemElements = $.map(items, createUiForTodoItem);
	        $("#todo-items").empty().append(itemElements);
	        $("#no-items").toggle(items.length === 0);
	    }, handleError);
	}

>[AZURE.NOTE]在 `read` 函数中提供原始 OData 查询选项字符串时，不能在同一查询中使用查询生成器方法。在这种情况下，必须将整个查询编写为 OData 查询字符串。有关 OData 系统查询选项的详细信息，请参阅 [OData 系统查询选项参考]。

## <a name="inserting"></a>如何在移动服务中插入数据

以下代码演示了如何在表中插入新行。客户端通过将 POST 请求发送到移动服务来请求插入数据行。请求正文包含要插入的、JSON 对象形式的数据。

	todoItemTable.insert({
	   text: "New Item",
	   complete: false
	})

以上代码会将提供的 JSON 对象中的数据插入到表中。你还可以指定完成插入时要调用的回调函数：

	todoItemTable.insert({
	   text: "New Item",
	   complete: false
	}).done(function (result) {
	   alert(JSON.stringify(result));
	}, function (err) {
	   alert("Error: " + err);
	});

### 使用 ID 值

移动服务支持为表的 **ID** 列使用唯一的自定义字符串值。这样，应用程序便可为 ID 使用自定义值（如电子邮件地址或用户名）。例如，以下代码将插入 JSON 对象形式的新项，其中，唯一 ID 是电子邮件地址：

			todoItemTable.insert({
			   id: "myemail@domain.com",
			   text: "New Item",
			   complete: false
	});

字符串 ID 可提供以下优势：

+ 无需往返访问数据库即可生成 ID。
+ 更方便地合并不同表或数据库中的记录。
+ ID 值能够更好地与应用程序的逻辑相集成。

如果插入的记录中尚未设置字符串 ID 值，移动服务将为 ID 生成唯一值。有关如何在客户端上或 .NET 后端中生成自己的 ID 值的详细信息，请参阅[如何：生成唯一 ID 值](/documentation/articles/mobile-services-how-to-use-server-scripts/#generate-guids)。

也可以为表使用整数 ID。若要使用整数 ID，必须结合 `--integerId` 选项使用 `mobile table create` 命令创建表。应在适用于 Azure 的命令行界面 (CLI) 中使用此命令。有关使用 CLI 的详细信息，请参阅[用于管理移动服务表的 CLI](/documentation/articles/virtual-machines-command-line-tools/#Mobile_Tables)。

## <a name="modifying"></a>如何：在移动服务中修改数据

以下代码演示了如何更新表中的数据。客户端通过将 PATCH 请求发送到移动服务来请求更新数据行。请求正文包含要更新的、JSON 对象形式的特定字段。该代码将更新表 `todoItemTable` 中的某个现有项。

			todoItemTable.update({
			   id: idToUpdate,
			   text: newText
			})

第一个参数使用 ID 指定了表中要更新的实例。

你还可以指定在完成更新时要调用的回调函数：

			todoItemTable.update({
			   id: idToUpdate,
			   text: newText
			}).done(function (result) {
			   alert(JSON.stringify(result));
			}, function (err) {
			   alert("Error: " + err);
			});

## <a name="deleting"></a>如何在移动服务中删除数据

以下代码演示了如何删除表中的数据。客户端通过将 DELETE 请求发送到移动服务来请求删除数据行。该代码将删除表 todoItemTable 中的某个现有项。

			todoItemTable.del({
			   id: idToDelete
			})

第一个参数使用 ID 指定了表中要删除的实例。

你还可以指定在完成删除时要调用的回调函数：

			todoItemTable.del({
			   id: idToDelete
			}).done(function () {
			   /* Do something */
			}, function (err) {
			   alert("Error: " + err);
			});

## <a name="binding"></a>如何：在用户界面中显示数据

本部分说明如何使用 UI 元素显示返回的数据对象。若要查询 `todoItemTable` 中的项并在极简单的列表中显示这些项，可以运行以下示例代码。这里未执行任何形式的选择、筛选或排序操作。

			var query = todoItemTable;

			query.read().then(function (todoItems) {
			   // The space specified by 'placeToInsert' is an unordered list element <ul> ... </ul>
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
			   alert("Error: " + err);
			});

在 Windows 应用商店应用程序中，可以使用查询的结果来创建 [WinJS.Binding.List] 对象，该对象可绑定为 [ListView] 对象的数据源。有关详细信息，请参阅[数据绑定（使用 JavaScript 和 HTML 的 Windows 应用商店应用程序）]。

##<a name="custom-api"></a>如何：调用自定义 API

自定义 API 可让你定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。有关如何在移动服务中创建自定义 API 的示例，请参阅[如何：定义自定义 API 终结点](/documentation/articles/mobile-services-dotnet-backend-define-custom-api/)。

通过对 **MobileServiceClient** 调用 [invokeApi](https://github.com/Azure/azure-mobile-services/blob/master/sdk/Javascript/src/MobileServiceClient.js#L337) 方法，从客户端调用自定义 API。例如，以下代码行向移动服务上的 **completeAll** API 发送 POST 请求：

    client.invokeApi("completeall", {
        body: null,
        method: "post"
    }).done(function (results) {
        var message = results.result.count + " item(s) marked as complete.";
        alert(message);
        refreshTodoItems();
    }, function(error) {
        alert(error.message);
    });

 
有关更现实可行的示例和对 **invokeApi** 更完整的介绍，请参阅[Azure 移动服务客户端 SDK 中的自定义 API](http://blogs.msdn.com/b/carlosfigueira/archive/2013/06/19/custom-api-in-azure-mobile-services-client-sdks.aspx)。

##<a name="authentication"></a>如何对用户进行身份验证

移动服务支持使用 Microsoft 帐户和 Azure Active Direcotry 对应用程序用户进行身份验证和授权。你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。你还可以在服务器脚本中使用已经过身份验证的用户的标识来实施授权规则。有关详细信息，请参阅 [身份验证入门] 教程。

>[AZURE.NOTE]在 PhoneGap 或 Cordova 应用程序中使用身份验证时，还必须向项目中添加以下插件：
>
>+ https://git-wip-us.apache.org/repos/asf/cordova-plugin-device.git
>+ https://git-wip-us.apache.org/repos/asf/cordova-plugin-inappbrowser.git


支持两种身份验证流：_服务器流_和_客户端流_。服务器流依赖于提供者的 Web 身份验证界面，因此可提供最简便的身份验证体验。客户端流依赖于提供者和设备特定的 SDK，因此允许与设备特定的功能（例如单一登录）进行更深入的集成。

###服务器流
若要让移动服务管理 Windows 应用商店或 HTML5 应用程序中的身份验证过程，必须将你的应用程序注册到标识提供者。然后，需要在移动服务中配置提供者提供的应用程序 ID 和机密。有关详细信息，请参阅[向应用程序添加身份验证](/documentation/articles/mobile-services-html-get-started-users/)教程。

注册标识提供者后，只需结合提供者的 [MobileServiceAuthenticationProvider] 值调用 [LoginAsync 方法]。例如，若要使用 microsoftaccount 登录，请使用以下代码。

		client.login("microsoftaccount").done(function (results) {
		     alert("You are now logged in as: " + results.userId);
		}, function (err) {
		     alert("Error: " + err);
		});

如果你使用的标识提供者不是 microsoftaccount，请将传递给上述 `login` 方法的值更改为 `windowsazureactivedirectory`。

在此情况下，移动服务将通过以下方式管理 OAuth 2.0 身份验证流：显示选定提供者的登录页，并在用户成功使用标识提供者登录后生成移动服务身份验证令牌。[login] 函数在完成时将返回一个 JSON 对象 (**user**)，该对象分别在 **userId** 和 **authenticationToken** 字段中公开用户 ID 和移动服务身份验证令牌。你可以缓存此令牌，并在它过期之前重复使用。有关详细信息，请参阅“缓存身份验证令牌”。

###客户端流
你的应用程序还能够独立联系标识提供者，然后将返回的令牌提供给移动服务以进行身份验证。使用此客户端流可为用户提供单一登录体验，或者从标识提供者中检索其他用户数据。


####Microsoft 帐户基本示例
以下示例使用 Live SDK，该 SDK 使用 Microsoft 帐户来支持 Windows 应用商店应用程序的单一登录：

		WL.login({ scope: "wl.basic"}).then(function (result) {
		      client.login(
		            "microsoftaccount",
		            {"authenticationToken": result.session.authentication_token})
		      .done(function(results){
		            alert("You are now logged in as: " + results.userId);
		      },
		      function(error){
		            alert("Error: " + err);
		      });
		});

这个简化的示例将从 Live Connect 获取一个令牌，并通过调用 [login] 函数将该令牌提供给移动服务。


####Microsoft 帐户完整示例

以下示例演示如何使用 Live SDK 和 WinJS API 来提供增强的单一登录体验：

	// Set the mobileClient variable to client variable generated by the tooling.
	var mobileClient = <yourClient>;

	var session = null;
	var login = function () {
		return new WinJS.Promise(function (complete) {
			WL.login({ scope: "wl.basic" }).then(function (result) {
				session = result.session;

				WinJS.Promise.join([
					WL.api({ path: "me", method: "GET" }),
					mobileClient.login(result.session.authentication_token)
				]).done(function (results) {
					// Build the welcome message from the Microsoft account info.
					var profile = results[0];
					var title = "Welcome " + profile.first_name + "!";
					var message = "You are now logged in as: "
						+ mobileClient.currentUser.userId;
					var dialog = new Windows.UI.Popups.MessageDialog(message, title);
					dialog.showAsync().then(function () {
						// Reload items from the mobile service.
						refreshTodoItems();
					}).done(complete);

				}, function (error) {

				});
			}, function (error) {
				session = null;
				var dialog = new Windows.UI.Popups.MessageDialog("You must log in.", "Login Required");
				dialog.showAsync().done(complete);
			});
		});
	}

	var authenticate = function () {
		// Block until sign-in is successful.
		login().then(function () {
			if (session === null) {
				// Authentication failed, try again.
				authenticate();
			}
		});
	}

	// Initialize the Live client.
	WL.init({
		redirect_uri: mobileClient.applicationUrl
	});

	// Start the sign-in process.
	authenticate();

此代码初始化 Live Connect 客户端，向 Microsoft 帐户发送一个新的登录请求，将返回的身份验证令牌发送到移动服务，然后显示有关已登录用户的信息。在身份验证成功之前，该应用不会启动。
<!--- //this guidance may be bad from an XSS vulnerability standpoint. We need to find better guidance for this
###Caching the authentication token
In some cases, the call to the login method can be avoided after the first time the user authenticates. We can use [sessionStorage] or [localStorage] to cache the current user identity the first time they log in and every subsequent time we check whether we already have the user identity in our cache. If the cache is empty or calls fail (meaning the current login session has expired), we still need to go through the login process.

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
-->

## <a name="push-notifications"></a>如何：注册推送通知

如果你的应用程序是 PhoneGap 或 Apache Cordova HTML/JavaScript 应用程序，则你可以使用本机移动平台在设备上接收推送通知。[Azure 移动服务的 Apache Cordova 插件](https://github.com/Azure/azure-mobile-services-cordova)可让你向 Azure 通知中心注册推送通知。使用的具体通知服务取决于执行代码的本机设备平台。有关如何执行此操作的示例，请参阅[使用 Microsoft Azure 将通知推送到 Cordova 应用程序](https://github.com/Azure/mobile-services-samples/tree/master/CordovaNotificationsArticle)。

>[AZURE.NOTE]此插件目前仅支持 iOS 和 Android 设备。有关也包含 Windows 设备的解决方案，请参阅文章[使用通知中心集成将通知推送到 PhoneGap 应用程序](http://blogs.msdn.com/b/azuremobile/archive/2014/06/17/push-notifications-to-phonegap-apps-using-notification-hubs-integration.aspx)。

## <a name="errors"></a>如何：处理错误

在移动服务中，你可能会遇到各种形式的错误，并且可以通过多种方式来验证和解决这些错误。

例如，你可以在移动服务中注册服务器脚本，然后使用这些脚本对所要插入和更新的数据执行各种操作，包括验证和数据修改。你可以按如下所示定义并注册一个用于验证和修改数据的服务器脚本：

			function insert(item, user, request) {
			   if (item.text.length > 10) {
				  request.respond(statusCodes.BAD_REQUEST, { error: "Text cannot exceed 10 characters" });
			   } else {
			      request.execute();
			   }
			}

此服务器端脚本将验证发送到移动服务的字符串数据长度，并拒绝过长（在本例中为 10 个字符以上）的字符串。

由于移动服务能够在服务器端验证数据和发送错误响应，因此你可以更新你的 HTML 应用程序，使其能够处理验证后生成的错误响应。

		todoItemTable.insert({
		   text: itemText,
		   complete: false
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

	client.getTable("tablename").read()
		.then(function (data) { /* do something */ }, handleError);

## <a name="promises"></a>如何：使用约定

约定提供了一种机制，让你基于尚未计算的值安排有待完成的工作。它是用于管理与异步 API 的交互的抽象。

每当提供给 `done` 约定的函数已成功完成或者收到错误时，就会立即执行该约定。与 `then` 约定不同，它肯定会引发无法在函数内部处理的错误；当处理程序完成执行后，此函数将引发当某个约定处于错误状态时返回的错误。有关详细信息，请参阅 [done]。

			promise.done(onComplete, onError);

例如：

			var query = todoItemTable;
			query.read().done(function (results) {
			   alert(JSON.stringify(results));
			}, function (err) {
			   alert("Error: " + err);
			});

`then` 约定与 `done` 约定类似，而与 `then` 约定的不同之处在于，`done` 肯定会引发无法在函数内部处理的错误。如果你未向 `then` 提供错误处理程序，当操作出错时，它不会引发异常，而是返回一个处于错误状态的约定。有关详细信息，请参阅 [then]。

			promise.then(onComplete, onError).done( /* Your success and error handlers */ );

例如：

			var query = todoItemTable;
			query.read().done(function (results) {
			   alert(JSON.stringify(results));
			}, function (err) {
			   alert("Error: " + err);
			});

你可以通过许多不同的方式使用约定。你可以通过对前一个 `then` 函数返回的约定调用 `then` 或 `done` 来链接约定操作。对于操作的中间阶段使用 `then`（例如 `.then().then()`），对于操作的最后阶段使用 `done`（例如 `.then().then().done()`）。你可以链接多个 `then` 函数，因为 `then` 返回约定。无法链接多个 `done` 方法，因为该方法返回 undefined。[详细了解 then 和 done 之间的差别]。

 			todoItemTable.insert({
 			   text: "foo"
 			}).then(function (inserted) {
 			   inserted.newField = 123;
 			   return todoItemTable.update(inserted);
 			}).done(function (insertedAndUpdated) {
 			   alert(JSON.stringify(insertedAndUpdated));
 			})

##<a name="customizing"></a>如何：自定义客户端请求标头

你可以使用 `withFilter` 函数发送自定义请求标头，以便读取和写入筛选器中即将发送的请求的任意属性。如果服务器端脚本需要自定义的 HTTP 标头或者可以使用这种标头增强自身，则你可能需要添加这样的标头。

			var client = new WindowsAzure.MobileServiceClient('https://your-app-url', 'your-key')
			   .withFilter(function (request, next, callback) {
			   request.headers.MyCustomHttpHeader = "Some value";
			   next(request, callback);
			});

筛选器的作用远远不只是自定义请求标头，它们还可用于检查或更改请求、检查或更改响应、绕过网络调用、发送多个调用，等等。

## <a name="hostnames"></a>如何：使用跨域资源共享

若要控制允许与移动服务交互以及向其发送请求的网站，请确保将用于托管移动服务的网站主机名添加到跨域资源共享 (CORS) 允许列表。对于 JavaScript 后端移动服务，可以在 [Azure 经典门户](https://manage.windowsazure.cn)的“配置”选项卡中配置允许列表。你可以根据需要使用通配符。默认情况下，新的移动服务将指示浏览器只能允许来自 `localhost` 的访问，跨域资源共享 (CORS) 允许外部主机名上的浏览器中运行的 JavaScript 代码与移动服务交互。对于 WinJS 应用程序，不需要使用此配置。

<!-- Anchors. -->

[What is Mobile Services]: #what-is
[Concepts]: #concepts
[How to: Create the Mobile Services client]: #create-client
[How to: Query data from a mobile service]: #querying
[Filter returned data]: #filtering
[Sort returned data]: #sorting
[Return data in pages]: #paging
[Select specific columns]: #selecting
[Look up data by ID]: #lookingup
[How to: Display data in the user interface]: #binding
[How to: Insert data into a mobile service]: #inserting
[How to: Modify data in a mobile service]: #modifying
[How to: Delete data in a mobile service]: #deleting
[How to: Authenticate users]: #authentication
[How to: Handle errors]: #errors
[How to: Use promises]: #promises
[How to: Customize request headers]: #customizing
[How to: Use cross-origin resource sharing]: #hostnames
[Next steps]: #nextsteps
[Execute an OData query operation]: #odata-query



<!-- URLs. -->

[then]: http://msdn.microsoft.com/zh-cn/library/windows/apps/br229728.aspx
[done]: http://msdn.microsoft.com/zh-cn/library/windows/apps/hh701079.aspx
[详细了解 then 和 done 之间的差别]: http://msdn.microsoft.com/zh-cn/library/windows/apps/hh700334.aspx
[how to handle errors in promises]: http://msdn.microsoft.com/zh-cn/library/windows/apps/hh700337.aspx

[sessionStorage]: http://msdn.microsoft.com/zh-cn/library/cc197062(v=vs.85).aspx
[localStorage]: http://msdn.microsoft.com/zh-cn/library/cc197062(v=vs.85).aspx

[ListView]: http://msdn.microsoft.com/zh-cn/library/windows/apps/br211837.aspx
[数据绑定（使用 JavaScript 和 HTML 的 Windows 应用商店应用程序）]: http://msdn.microsoft.com/zh-cn/library/windows/apps/hh758311.aspx
[login]: https://github.com/Azure/azure-mobile-services/blob/master/sdk/Javascript/src/MobileServiceClient.js#L301
[ASCII control codes C0 and C1]: http://zh.wikipedia.org/wiki/Data_link_escape_character#C1_set
[OData 系统查询选项参考]: http://go.microsoft.com/fwlink/p/?LinkId=444502

<!---HONumber=Mooncake_0118_2016-->