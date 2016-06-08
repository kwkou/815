<properties 
	pageTitle="使用 JavaScript 后端移动服务" 
	description="提供有关如何在 Azure 移动服务中定义、注册以及使用服务器脚本的示例。" 
	services="mobile-services" 
	documentationCenter="" 
	authors="RickSaling" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/23/2016" 
	wacn.date="01/29/2016"/>


#  使用 JavaScript 后端移动服务

 
本文提供有关如何在 Azure 移动服务中使用 JavaScript 后端的详细信息和示例。

## <a name="intro"></a>介绍

在 JavaScript 后端移动服务中，你可以定义 JavaScript 代码形式的自定义业务逻辑，该代码将在服务器中存储和执行。此服务器脚本代码将分配到下列服务器功能之一：

+ [对给定表执行的插入、读取、更新或删除操作][Table operations]。
+ [计划的作业][Job Scheduler]。
+ [自定义 API 中定义的 HTTP 方法][Custom API anchor]。 

服务器脚本中 main 函数的签名取决于脚本的具体使用位置。你还可以将公用脚本代码定义为可在脚本之间共享的 nodes.js 模块。有关详细信息，请参阅[源代码管理和共享代码][Source control, shared code, and helper functions]。

有关各个服务器脚本对象和函数的说明，请参阅[移动服务服务器脚本参考]。


## <a name="table-scripts"></a>表操作

表操作脚本是一种服务器脚本，它将注册到对表执行的操作 &mdash; 插入、读取、更新或删除 (*del*)。本部分介绍如何在 JavaScript 后端使用表操作，具体包括以下小节：

+ [表操作概述][Basic table operations]
+ [如何：注册表操作]
+ [如何：重写默认响应]
+ [如何：重写 execute success]
+ [如何：重写默认错误处理]
+ [如何：生成唯一 ID 值](#generate-guids)
+ [如何：添加自定义参数]
+ [如何：处理表用户][How to: Work with users]

### <a name="basic-table-ops"></a>表操作概述

该脚本的名称必须与注册的操作类型相匹配。对于一个给定的表操作，只能注册一个脚本。每当 REST 请求调用给定的操作时（例如，当收到要在表中插入项的 POST 请求时），就会执行该脚本。移动服务不会保存每次执行脚本后的状态。由于每次运行脚本时都会创建一个新的全局上下文，因此脚本中定义的所有状态变量都会重新初始化。如果你想要存储执行不同请求后的状态，请在移动服务中创建一个表，然后读取状态并将状态写入该表。有关详细信息，请参阅[如何：从脚本访问表]。

如果需要在执行操作时强制实施自定义的业务逻辑，请编写表操作脚本。例如，当 `text` 字段的字符串长度大于 10 个字符时，以下脚本将拒绝插入操作：

	function insert(item, user, request) {
	    if (item.text.length > 10) {
	        request.respond(statusCodes.BAD_REQUEST, 
				'Text length must be less than 10 characters');
	    } else {
	        request.execute();
	    }
	}

表脚本函数始终采用三个参数。

- 第一个参数根据表操作的不同而异。 

	- 对于插入和更新，它是一个 **item** 对象，即操作所影响的行的 JSON 表示形式。这样，你便可以按名称（例如 *item.Owner*，其中 *Owner* 是 JSON 表示形式的名称之一）访问列值。
	- 对于删除，它是要删除的记录的 ID。 
	- 对于读取，它是用于指定要返回的行集的 [query 对象]。

- 第二个参数始终是 [user 对象][User object]，表示提交了请求的用户。

- 第三个参数始终是 [request 对象][request object]，你可以凭此控制所请求的操作的执行，以及发送到客户端的响应。

以下是表操作的规范主函数签名：

+ [Insert][insert function]：`function insert (item, user, request) { ... }`
+ [Update][update function]：`function update (item, user, request) { ... }`
+ [Delete][delete function]：`function del (id, user, request) { ... }`
+ [Read][read function]：`function read (query, user, request) { ... }`

>[AZURE.NOTE]注册到删除操作的函数必须命名为 _del_，因为 delete 是 JavaScript 中的保留关键字。

每个服务器脚本都有一个主函数，并包含可选的 Helper 函数。即使服务器脚本是为特定表创建的，它也可以引用同一数据库中的其他表。你还可以将公用函数定义为可在脚本之间共享的模块。有关详细信息，请参阅[源代码管理和共享代码][Source control, shared code, and helper functions]。

### <a name="register-table-scripts"></a>如何：注册表脚本

你可以使用下列方式之一定义可注册到表操作的服务器脚本：

+ 通过 [Azure 管理门户]。在给定表的“脚本”选项卡中访问表操作的脚本。下面显示了已注册到 `TodoItem` 表的插入脚本的默认代码。你可以使用自己的自定义业务逻辑重写此代码。

	![1][1]
	
	若要了解如何执行此操作，请参阅[使用服务器脚本在移动服务中验证和修改数据]。

+ 使用源代码管理。启用源代码管理后，只需在 git 存储库中的 .\\service\\table 子文件夹内创建一个名为 <em>`<table>`</em>.<em>`<operation>`</em>.js 的文件，其中，<em>`<table>`</em> 是表的名称，<em>`<operation>`</em> 是要注册的表操作。有关详细信息，请参阅[源代码管理和共享代码][Source control, shared code, and helper functions]。

+ 使用 Azure 命令行工具中的命令提示符。有关详细信息，请参阅[使用命令行工具]。


表操作脚本必须至少调用 [request 对象]的下列函数之一，以确保客户端收到响应。
 
+ **execute 函数**：已按请求完成操作，并已返回标准响应。
 
+ **respond 函数**：已返回自定义响应。

> [AZURE.IMPORTANT]如果在脚本的某个代码路径中 **execute** 和 **respond** 均未调用，则该操作可能不返回响应。

以下脚本将调用 **execute** 函数来完成客户端请求的数据操作：

	function insert(item, user, request) { 
	    request.execute(); 
	}

在此示例中，将向数据库中插入项目，并且将相应的状态代码返回给用户。

调用 **execute** 函数时，作为第一个参数传入脚本函数的 `item`、[query][query object] 或 `id` 值用于执行该操作。对于插入、更新或查询操作，你可以在调用 **execute** 之前修改 item 或 query：

	function insert(item, user, request) { 
	    item.scriptComment =
			'this was added by a script and will be saved to the database'; 
	    request.execute(); 
	} 
 
	function update(item, user, request) { 
	    item.scriptComment = 
			'this was added by a script and will be saved to the database'; 
	    request.execute(); 
	} 

	function read(query, user, request) { 
		// Only return records for the current user 	    
		query.where({ userid: user.userId}); 
	    request.execute(); 
	}
 
>[AZURE.NOTE]在删除脚本中，更改所提供的 userId 变量不会影响所删除的记录。

有关更多示例，请参阅[读取和写入数据]、[修改请求]和[验证数据]。


### <a name="override-response"></a>如何：重写默认响应

你还可以使用脚本来实现能够重写默认响应行为的验证逻辑。如果验证失败，则只需调用 **respond** 函数而不是 **execute** 函数，然后将响应写入客户端：

	function insert(item, user, request) {
	    if (item.userId !== user.userId) {
	        request.respond(statusCodes.FORBIDDEN, 
	        'You may only insert records with your userId.');
	    } else {
	        request.execute();
	    }
	}

在此示例中，当所插入项的 `userId` 属性与为经过身份验证的客户端提供提供的 [user 对象]的 `userId` 不匹配时，该请求将被拒绝。在这种情况下，数据库操作 (*insert*) 将不会发生，并且会将 HTTP 状态代码为 403 的响应以及自定义的错误消息返回到客户端。有关更多示例，请参阅[修改响应]。

### <a name="override-success"></a>如何：重写 execute success

默认情况下，在表操作中，**execute** 函数会自动写入响应。但是，你可以向 execute 函数传递两个可选参数，用于重写该函数在成功和/或出错时的行为。

通过在调用 execute 时传入 **success** 处理程序，你可以先修改查询的结果，然后再将结果写入到响应中。以下示例调用 `execute({ success: function(results) { ... })`，以便在从数据库读取数据后但在写入响应之前执行附加操作：

	function read(query, user, request) {
	    request.execute({
	        success: function(results) {
	            results.forEach(function(r) {
	                r.scriptComment = 
	                'this was added by a script after querying the database';
	            });
	            request.respond();
	        }
	    });
	}

如果为 **execute** 函数提供了 **success** 处理程序，则还必须在 **success** 处理程序中调用 **respond** 函数，使运行时知道脚本已完成，并且可写入响应。如果在调用 **respond** 时未传递任何参数，则移动服务将生成默认响应。

>[AZURE.NOTE]只有在先调用 **execute** 函数之后，才能调用不带参数的 **respond** 来调用默认响应。
 
### <a name="override-error"></a>如何：重写默认错误处理

如果与数据库的连接断开、对象无效或者查询不正确，**execute** 函数可能会失败。默认情况下，当发生错误时，服务器脚本会记录错误，并将错误结果写入到响应中。由于移动服务提供了默认的错误处理，因此你不需要处理服务中可能会发生的错误。

如果你想要采取特定的补救措施，或者想要使用全局控制台对象向日志写入更详细信息，则可以通过实施显式错误处理来重写默认的错误处理。向 **execute** 函数提供一个 **error** 处理程序即可实现此目的：

	function update(item, user, request) { 
	  request.execute({ 
	    error: function(err) { 
	      // Do some custom logging, then call respond. 
	      request.respond(); 
	    } 
	  }); 
	}
 

提供 error 处理程序后，调用 **respond** 时，移动服务会向客户端返回错误结果。

如果需要，你也可以同时提供 **success** 和 **error** 处理程序。

### <a name="generate-guids"></a>如何：生成唯一 ID 值

移动服务支持为表的 **ID** 列使用唯一的自定义字符串值。这样，应用程序便可为 ID 使用自定义值（如电子邮件地址或用户名）。

字符串 ID 可提供以下优势：

+ 无需往返访问数据库即可生成 ID。
+ 更方便地合并不同表或数据库中的记录。
+ ID 值能够更好地与应用程序的逻辑相集成。

如果插入的记录中未设置字符串 ID 值，移动服务将为 ID 生成唯一值。你可以在服务器脚本中生成自己的唯一 ID 值。下面的脚本示例将生成一个自定义 GUID 并将其分配给新记录的 ID。此 ID 类似于你未传入记录的 ID 值时，移动服务生成的 ID 值。

	// Example of generating an id. This is not required since Mobile Services
	// will generate an id if one is not passed in.
	item.id = item.id || newGuid();
	request.execute();

	function newGuid() {
		var pad4 = function(str) { return "0000".substring(str.length) + str; };
		var hex4 = function () { return pad4(Math.floor(Math.random() * 0x10000 /* 65536 */ ).toString(16)); };
		return (hex4() + hex4() + "-" + hex4() + "-" + hex4() + "-" + hex4() + "-" + hex4() + hex4() + hex4());
	}


如果应用程序提供了某个 ID 的值，移动服务将按原样存储该值，包括前导和尾随空格。不会从值中裁剪掉空格。

`id` 的值必须唯一，并且不能包含以下集中的字符：

+ 控制字符：[0x0000-0x001F] 和 [0x007F-0x009F]。有关详细信息，请参阅 [ASCII 控制代码 C0 和 C1](http://zh.wikipedia.org/wiki/Data_link_escape_character#C1_set)。
+  可打印字符：**"**(0x0022), **+** (0x002B), **/** (0x002F), **?** (0x003F), **\** (0x005C), **`** (0x0060)
+  ID“.”和“..”

也可以为表使用整数 ID。若要使用整数 ID，必须结合 `--integerId` 选项使用 `mobile table create` 命令创建表。应在适用于 Azure 的命令行界面 (CLI) 中使用此命令。有关使用 CLI 的详细信息，请参阅[用于管理移动服务表的 CLI](/documentation/articles/virtual-machines-command-line-tools/#Mobile_Tables)。


### <a name="access-headers"></a>如何：访问自定义参数

向移动服务发送请求时，你可以在请求 URI 中包含自定义参数，以指示表操作脚本如何处理给定的请求。然后，你可以修改脚本，通过检查参数的方式来确定处理路径。

例如，以下 POST 请求 URI 将指示服务不要允许插入具有相同 text 值的新 *TodoItem*：

		https://todolist.azure-mobile.net/tables/TodoItem?duplicateText=false

可从 [request 对象]的 **parameters** 属性访问这些以 JSON 值提供的自定义查询参数。移动服务向已注册到表操作的任何函数提供 **request** 对象。以下用于插入操作的服务器脚本将在运行插入操作之前检查 `duplicateText` 参数的值：

		function insert(item, user, request) {
		    var todoItemTable = tables.getTable('TodoItem');
		    // Check the supplied custom parameter to see if
		    // we should allow duplicate text items to be inserted.		   
		    if (request.parameters.duplicateText === 'false') {
		        // Find all existing items with the same text
		        // and that are not marked 'complete'. 
		        todoItemTable.where({
		            text: item.text,
		            complete: false
		        }).read({
		            success: insertItemIfNotComplete
		        });
		    } else {
		        request.execute();
		    }

		    function insertItemIfNotComplete(existingItems) {
		        if (existingItems.length > 0) {
		            request.respond(statusCodes.CONFLICT, 
                        "Duplicate items are not allowed.");
		        } else {
		            // Insert the item as normal. 
		            request.execute();
		        }
		    }
		}

请注意，在 **insertItemIfNotComplete** 中，如果不存在重复文本，则会调用 [request 对象]的 **execute** 函数来插入项；否则，会调用 **respond** 函数来通知客户端存在重复文本。

请注意上述代码中 **success** 函数的调用语法：

 		        }).read({
		            success: insertItemIfNotComplete
		        });

上述代码较为精简，JavaScript 中更冗长的等效代码为：

		success: function(results) 
		{ 
			insertItemIfNotComplete(results); 
		}


### <a name="work-with-users"></a>如何：处理用户

在 Azure 移动服务中，你可以使用标识提供程序对用户进行身份验证。有关详细信息，请参阅[身份验证入门]。当经过身份验证的用户调用表操作时，移动服务将使用 [user 对象]向已注册的脚本函数提供有关该用户的信息。可以使用 **userId** 属性来存储和检索用户特定的信息。以下示例将基于某个经过身份验证的用户的 **userId** 来设置项的 owner 属性：

	function insert(item, user, request) {
	    item.owner = user.userId;
	    request.execute();
	}

以下示例将基于某个经过身份验证的用户的 **userId** 向查询添加一个附加的筛选器。此筛选器会将结果限制为属于当前用户的项：

	function read(query, user, request) {
	    query.where({
	        owner: user.userId
	    });
	    request.execute();
	}

## <a name="custom-api"></a>自定义 API

本部分介绍如何创建和使用自定义 API 终结点，具体包括以下小节：
	
+ [自定义 API 概述](#custom-api-overview)
+ [如何：定义自定义 API]
+ [如何：实现 HTTP 方法]
+ [如何：发送和接收 XML 格式的数据]
+ [如何：处理用户和自定义 API 中的标头]
+ [如何：在一个自定义 API 中定义多个路由]

### <a name="custom-api-overview"></a>自定义 API 概述

自定义 API 是移动服务中可通过一个或多个标准 HTTP 方法访问的终结点，这些方法包括：GET、POST、PUT、PATCH 和 DELETE。可以在单个脚本文件中为自定义 API 支持的每个 HTTP 方法单独定义一个函数导出。收到使用给定方法向自定义 API 发出的请求后，将调用注册的脚本。有关详细信息，请参阅[自定义 API]。

当移动服务运行时调用自定义 API 函数时，将同时提供 [request][request object] 和 [response][response object] 对象。这些对象将公开 [express.js 库]的功能，而你的脚本可以利用这些功能。以下名为 **hello** 的自定义 API 是一个极简单的示例，它会返回 _Hello, world!_ 以响应 POST 请求：

		exports.post = function(request, response) {
		    response.send(200, "{ message: 'Hello, world!' }");
		} 

[response 对象]的 **send** 函数向客户端返回所需的响应。可以通过向以下 URL 发送 POST 请求来调用此代码：

		https://todolist.azure-mobile.net/api/hello  

每次执行后都会保留全局状态。

### <a name="define-custom-api"></a>如何：定义自定义 API

你可以使用下列方式之一定义可注册到自定义 API 终结点中 HTTP 方法的服务器脚本：

+ 通过 [Azure 管理门户]。可以在“API”选项卡中创建和修改自定义 API 脚本。服务器脚本代码位于给定自定义 API 的“脚本”选项卡中。下面显示了向 `CompleteAll` 自定义 API 终结点发出的 POST 请求调用的脚本。 

	![2][2]
	
	对自定义 API 方法的访问权限在“权限”选项卡中分配。若要了解此自定义 API 的创建方式，请参阅[从客户端调用自定义 API]。

+ 使用源代码管理。如果已启用源代码管理，只需在 git 存储库的 .\\service\\api 子文件夹中创建一个名为 <em>`<custom_api>`</em>.js 的文件，其中 <em>`<custom_api>`</em> 是要注册的自定义 API 的名称。此脚本文件包含自定义 API 公开的每个 HTTP 方法的 _exported_ 函数。权限在随附的 .json 文件中定义。有关详细信息，请参阅[源代码管理和共享代码][Source control, shared code, and helper functions]。

+ 使用 Azure 命令行工具中的命令提示符。有关详细信息，请参阅[使用命令行工具]。

### <a name="handle-methods"></a>如何：实现 HTTP 方法

一个自定义 API 可以处理一个或多个 HTTP 方法：GET、POST、PUT、PATCH 和 DELETE。将为自定义 API 处理的每个 HTTP 方法定义一个导出函数。单个自定义 API 代码文件可以导出下列一个或所有函数：

		exports.get = function(request, response) { ... };
		exports.post = function(request, response) { ... };
		exports.patch = function(request, response) { ... };
		exports.put = function(request, response) { ... };
		exports.delete = function(request, response) { ... };

不能使用服务器脚本中尚未实现的 HTTP 方法调用自定义 API 终结点，该调用会返回 405（“不允许的方法”）错误响应。可向每个支持 HTTP 方法单独分配权限级别。

### <a name="api-return-xml"></a>如何：发送和接收 XML 格式的数据

当客户端存储和检索数据时，移动服务将使用 JavaScript 对象表示法 (JSON) 来表示消息正文中的数据。但是，在某些情况下，你可能希望使用 XML 负载。例如，Windows 应用商店应用程序具有内置的定期通知功能，这就需要服务发出 XML 数据。有关详细信息，请参阅[定义支持定期通知的自定义 API]。

以下 **OrderPizza** 自定义 API 函数将返回一个简单的 XML 文档作为响应负载：

		exports.get = function(request, response) {
		  response.set('content-type', 'application/xml');
		  var xml = '<?xml version="1.0"?><PizzaOrderForm><PizzaOrderForm/>';
		  response.send(200, xml);
		};

可以通过向以下终结点发出 HTTP GET 请求来调用此自定义 API 函数：

		https://todolist.azure-mobile.net/api/orderpizza

### <a name="get-api-user"></a>如何：处理用户和自定义 API 中的标头

在 Azure 移动服务中，你可以使用标识提供程序对用户进行身份验证。有关详细信息，请参阅[身份验证入门]。当经过身份验证的用户请求自定义 API 时，移动服务将使用[用户对象]向自定义 API 代码提供有关该用户的信息。可从 [request 对象]的 user 属性访问 [user 对象]。可以使用 **userId** 属性来存储和检索用户特定的信息。

以下 **OrderPizza** 自定义 API 函数将基于某个经过身份验证的用户的 **userId** 来设置项的 owner 属性：

		exports.post = function(request, response) {
			var userTable = request.service.tables.getTable('user');
			userTable.lookup(request.user.userId, {
				success: function(userRecord) {
					callPizzaAPI(userRecord, request.body, function(orderResult) {
						response.send(201, orderResult);
					});
				}
			});
		
		};

可以通过向以下终结点发出 HTTP POST 请求来调用此自定义 API 函数：

		https://<service>.azure-mobile.net/api/orderpizza

你还可以通过 [request 对象]访问特定的 HTTP 标头，如以下代码中所示：

		exports.get = function(request, response) {    
    		var header = request.header('my-custom-header');
    		response.send(200, "You sent: " + header);
		};

这个简单示例将读取名为 `my-custom-header` 的自定义标头，然后在响应中返回值。

### <a name="api-routes"></a>如何：在一个自定义 API 中定义多个路由

移动服务允许你在一个自定义 API 中定义多个路径或路由。例如，向 **calculator** 自定义 API 中的以下 URL 发出的 HTTP GET 请求将分别调用 **add** 或 **subtract** 函数：

+ `https://<service>.azure-mobile.net/api/calculator/add`
+ `https://<service>.azure-mobile.net/api/calculator/sub`

可以通过导出一个传递了 **api** 对象（类似于 [express.js 中的 express 对象]）的 **register** 函数来定义多个路由，该对象用于在自定义 API 终结点下注册路由。以下示例将在 **calculator** 自定义 API 中实现 **add** 和 **sub** 方法：

		exports.register = function (api) {
		    api.get('add', add);
		    api.get('sub', subtract);
		}
		
		function add(req, res) {
		    var result = parseInt(req.query.a) + parseInt(req.query.b);
		    res.send(200, { result: result });
		}
		
		function subtract(req, res) {
		    var result = parseInt(req.query.a) - parseInt(req.query.b);
		    res.send(200, { result: result });
		}

传递给 **register** 函数的 **api** 对象将为每个 HTTP 方法（**get**、**post**、**put**、**patch** 和 **delete**）公开一个函数。这些函数会将一个路由注册到特定 HTTP 方法的已定义函数。每个函数均采用两个参数，第一个参数是路由名称，第二个参数是注册到路由的函数。

HTTP GET 请求可按如下所示调用上述自定义 API 示例中的两个路由（随响应一起显示）：

+ `https://<service>.azure-mobile.net/api/calculator/add?a=1&b=2`

		{"result":3}

+ `https://<service>.azure-mobile.net/api/calculator/sub?a=3&b=5`

		{"result":-2}

## <a name="scheduler-scripts"></a>作业计划程序

移动服务允许你定义按固定计划以作业形式执行或通过 Azure 管理门户按需执行的服务器脚本。计划的作业可用于执行周期性任务，例如，清理表数据和批处理。有关详细信息，请参阅[计划作业]。

注册到计划作业的脚本具有一个与计划作业同名的主函数。由于 HTTP 请求不调用计划的脚本，没有可由服务器运行时传递的上下文，因此该函数不采用任何参数。与其他类型的脚本一样，你可以使用子例程函数并需要使用共享模块。有关详细信息，请参阅[源代码管理、共享代码和 Helper 函数]。

### <a name="scheduler-scripts"></a>如何：定义计划的作业脚本

可将一个服务器脚本分配到移动服务计划程序中定义的作业。这些脚本属于该作业，并根据作业计划执行。（你也可以使用 [Azure 管理门户]按需运行作业。） 定义计划作业的脚本不带参数，因为移动服务不会向它传递任何数据；该脚本作为常规 JavaScript 函数执行，不直接与移动服务交互。

可通过下列方式之一定义计划作业：

+ 通过 [Azure 管理门户]中的计划程序的“脚本”选项卡：

	![3][3]

+ 使用 Azure 命令行工具中的命令提示符。有关详细信息，请参阅[使用命令行工具]。

>[AZURE.NOTE]启用源代码管理后，你可以直接在 git 存储库的 .\\service\\scheduler 子文件夹中编辑计划的作业脚本文件。有关详细信息，请参阅 [如何：使用源代码管理来共享代码]。

## <a name="shared-code"></a>源代码管理、共享代码和 Helper 函数

本部分说明如何利用源代码管理来添加你自己的自定义 node.js 模块、共享的代码和其他代码重用策略，具体包括以下小节：

+ [利用共享代码概述](#leverage-source-control)
+ [如何：加载 Node.js 模块]
+ [如何：使用 Helper 函数]
+ [如何：使用源代码管理来共享代码]
+ [如何：使用应用程序设置] 

### <a name="leverage-source-control"></a>利用共享代码概述

由于移动服务使用服务器上的 Node.js，因此你的脚本已经获取了对内置 Node.js 模块的访问权限。你也可以使用源代码管理定义自己的模块，或者将其他 Node.js 模块添加到你的服务。

下面列出了你可以通过全局 **require** 函数在脚本中利用的一些较为有用的模块：

+ **azure**：公开 Azure SDK for Node.js 的功能。有关详细信息，请参阅 [Azure SDK for Node.js]。 
+ **crypto**：提供 OpenSSL 的加密功能。有关详细信息，请参阅 [Node.js 文档][crypto API]。
+ **path**：包含用于处理文件路径的实用工具。有关详细信息，请参阅 [Node.js 文档][path API]。
+ **querystring**：包含用于处理查询字符串的实用工具。有关详细信息，请参阅 [Node.js 文档][querystring API]。
+ **request**：向 Twitter 和 Facebook 等外部 REST 服务发送 HTTP 请求。有关详细信息，请参阅[发送 HTTP 请求]。
+ **sendgrid**：使用 Azure 中的 Sendgrid 电子邮件服务发送电子邮件。有关详细信息，请参阅[使用 SendGrid 从移动服务发送电子邮件]。
+ **url**：包含用于分析和解析 URL 的实用工具。有关详细信息，请参阅 [Node.js 文档][url API]。
+ **util**：包含各种实用工具，例如字符串格式设置和对象类型检查。有关详细信息，请参阅 [Node.js 文档][util API]。 
+ **zlib**：公开压缩功能，例如 gzip 和 deflate。有关详细信息，请参阅 [Node.js 文档][zlib API]。 

### <a name="modules-helper-functions"></a>如何：利用模块

移动服务公开了脚本可以使用全局 **require** 函数加载的一组模块。例如，脚本可以要求使用 **request** 发出 HTTP 请求：

	function update(item, user, request) { 
	    var httpRequest = require('request'); 
	    httpRequest('http://www.google.com', function(err, response, body) { 
	    	... 
	    }); 
	} 


### <a name="shared-code-source-control"></a>如何：使用源代码管理来共享代码

你可以将源代码管理与 Node.js 程序包管理器 (npm) 结合使用，以控制可供移动服务使用的模块。可通过两种方式实现此目的：

+ 对于已发布到 npm 的模块以及 npm 安装的模块，可以使用 package.json 文件来声明你希望通过移动服务进行安装的程序包。这样，你的服务始终都能访问所需程序包的最新版本。package.json 文件驻留在 `.\service` 目录中。有关详细信息，请参阅 [Azure 移动服务中对 package.json 的支持]。

+ 对于专用或自定义模块，你可以使用 npm 手动将模块安装到源代码管理的 `.\service\node_modules` 目录中。有关如何手动上载模块的示例，请参阅[在服务器脚本中利用共享代码和 Node.js 模块]。

	>[AZURE.NOTE]如果 `node_modules` 已在目录层次结构中存在，NPM 将在该目录中创建 `\node-uuid` 子目录，而不是在存储库中创建一个新的 `node_modules`。在此情况下，你只需删除现有的 `node_modules` 目录即可。

将 package.json 文件或自定义模块提交到移动服务的存储库后，请使用 **require** 来按名称引用这些模块。

>[AZURE.NOTE]在 package.json 中指定的模块或者上载到移动服务的模块只会在服务器脚本代码中使用。移动服务运行时不使用这些模块。

### <a name="helper-functions"></a>如何：使用 Helper 函数

除了要求使用模块外，单个服务器脚本还可以包含 Helper 函数。这些函数与主函数不同，后者可用于分离脚本中的代码。

在以下示例中，表脚本已注册到包含 Helper 函数 **handleUnapprovedItem** 的插入操作：


	function insert(item, user, request) {
	    if (!item.approved) {
	        handleUnapprovedItem(item, user, request);
	    } else {
	        request.execute();
	    }
	}
	
	function handleUnapprovedItem(item, user, request) {
	    // Do something with the supplied item, user, or request objects.
	}
 
在脚本中，Helper 函数必须在主函数之后声明。必须声明脚本中的所有变量。未声明的变量会导致出错。

Helper 函数也可以只定义一次，然后在服务器脚本之间共享。若要在脚本之间共享某个函数，必须导出该函数，并且脚本文件必须在 `.\service\shared` 目录中存在。以下模板演示了如何在文件 `.\services\shared\helpers.js` 中导出共享函数：

		exports.handleUnapprovedItem = function (tables, user, callback) {
		    
		    // Do something with the supplied tables or user objects and 
			// return a value to the callback function.
		};
 
然后，你可以在表操作脚本中使用类似于下面的函数：

		function insert(item, user, request) {
		    var helper = require('../shared/helper');
		    helper.handleUnapprovedItem(tables, user, function(result) {
		        	
					// Do something based on the result.
		            request.execute();
		        }
		    }
		}

在此示例中，必须将 [tables 对象]和 [user 对象]都传递给共享函数。这是因为共享脚本不能访问全局 [tables 对象]，而 [user 对象]只在请求的上下文中存在。

可以使用[源代码管理][How to: Share code by using source control]或[命令行工具][Using the command line tool]将脚本文件上载到共享目录。

### <a name="app-settings"></a>如何：使用应用程序设置

移动服务允许你将值安全地存储为应用程序设置，服务器脚本在运行时可以访问这些设置。将数据添加到移动服务的应用程序设置时，名称/值对将以加密的形式存储，你可以在服务器脚本中访问这些数据，而无需在脚本文件中对其进行硬编码。有关详细信息，请参阅[应用设置]。

以下自定义 API 示例使用提供的 [service 对象]来检索某个应用程序设置值。

		exports.get = function(request, response) {
		
			// Get the MY_CUSTOM_SETTING value from app settings.
		    var customSetting = 
		        request.service.config.appSettings.my_custom_setting;
				
			// Do something and then send a response.

		}

以下代码使用配置模块来检索计划作业脚本使用的应用程序设置中存储的 Twitter 访问令牌值：

		// Get the service configuration module.
		var config = require('mobileservice-config');

		// Get the stored Twitter consumer key and secret. 
		var consumerKey = config.twitterConsumerKey,
		    consumerSecret = config.twitterConsumerSecret
		// Get the Twitter access token from app settings.    
		var accessToken= config.appSettings.TWITTER_ACCESS_TOKEN,
		    accessTokenSecret = config.appSettings.TWITTER_ACCESS_TOKEN_SECRET;

请注意，此代码还会检索门户的“标识”选项卡中存储的 Twitter 使用者密钥值。由于 **config 对象**在表操作和计划的作业脚本中不可用，因此你必须要求配置模块访问应用程序设置。
<h2><a name="command-prompt"></a>使用命令行工具</h2>

在移动服务中，你可以使用 Azure 命令行工具创建、修改和删除服务器脚本。上载脚本之前，请确保使用的是以下目录结构：

![4][4]

请注意，此目录结构与使用源代码管理时的 git 存储库相同。

从命令行工具上载脚本文件时，必须先导航到 `.\services` 目录。以下命令从 `table` 子目录上载名为 `todoitem.insert.js` 的脚本：

		~$azure mobile script upload todolist table/todoitem.insert.js
		info:    Executing command mobile script upload
		info:    mobile script upload command OK

以下命令返回移动服务中维护的每个脚本文件的相关信息：

		~$ azure mobile script list todolist
		info:    Executing command mobile script list
		+ Retrieving script information
		info:    Table scripts
		data:    Name                       Size
		data:    -------------------------  ----
		data:    table/channels.insert      1980
		data:    table/TodoItem.insert      5504
		data:    table/TodoItem.read        64
		info:    Shared scripts
		data:    Name              Size
		data:    ----------------  ----
		data:    shared/helper.js  62
		data:    shared/uuid.js    7452
		info:    Scheduled job scripts
		data:    Job name    Script name           Status    Interval     Last run  Next run
		data:    ----------  --------------------  --------  -----------  --------  --------
		data:    getUpdates  scheduler/getUpdates  disabled  15 [minute]  N/A       N/A
		info:    Custom API scripts
		data:    Name                    Get          Put          Post         Patch        Delete
		data:    ----------------------  -----------  -----------  -----------  -----------  -----------
		data:    completeall             application  application  application  application  application
		data:    register_notifications  application  application  user         application  application
		info:    mobile script list command OK

有关详细信息，请参阅[用于管理 Azure 移动服务的命令]。

## <a name="working-with-tables"></a>使用表

本部分详细介绍了用于直接处理 SQL 数据库表数据的策略，具体包括以下小节：

+ [使用表的概述](#overview-tables)
+ [如何：从脚本访问表]
+ [如何：执行批量插入]
+ [如何：将 JSON 类型映射到数据库类型]
+ [使用 Transact-SQL 访问表]

### <a name="overview-tables"></a>使用表的概述

移动服务中的许多情况都要求服务器脚本访问数据库中的表。例如，由于每次执行脚本后移动服务不保存状态，因此，必须在表中存储每次执行脚本后需要持久保留的数据。你还可能想要检查权限表中的条目，或者要存储审核数据而不仅仅是写入日志，因为日志中的数据保留期有限，并且无法以编程方式访问。

移动服务提供两种用于访问表的方法：使用 [table 对象]代理，或者通过使用 [mssql 对象]撰写 Transact-SQL 查询。使用 [table 对象]可以轻松访问服务器脚本代码中的表数据，不过，[mssql 对象]支持更复杂的数据操作，并提供最大的灵活性。

### <a name="access-tables"></a>如何：从脚本访问表

从脚本访问表的最简单方法就是使用 [tables 对象]。**getTable** 函数将返回一个 [table 对象]实例，即用于访问所请求表的代理。然后，你可以调用该代理的函数来访问和更改数据。

已同时注册到表操作和计划作业的脚本可以访问全局对象形式的 [tables 对象]。以下代码行将从全局 [tables 对象]中获取 *TodoItems* 表的代理：

		var todoItemsTable = tables.getTable('TodoItems');

自定义 API 脚本可从提供的 [request 对象]的 <strong>service</strong> 属性访问 [tables 对象]。以下代码行将从请求中获取 [tables 对象]：

		var todoItemsTable = request.service.tables.getTable('TodoItem');

> [AZURE.NOTE]共享函数不能直接访问 **tables** 对象。在共享函数中，必须将表对象传递给该函数。

获得 [table 对象]后，可以调用一个或多个表操作函数：insert、update、delete 或 read。以下示例将从 permissions 表中读取用户权限：

	function insert(item, user, request) {
		var permissionsTable = tables.getTable('permissions');
	
		permissionsTable
			.where({ userId: user.userId, permission: 'submit order'})
			.read({ success: checkPermissions });
			
		function checkPermissions(results) {
			if(results.length > 0) {
				// Permission record was found. Continue normal execution.
				request.execute();
			} else {
				console.log('User %s attempted to submit an order without permissions.', user.userId);
				request.respond(statusCodes.FORBIDDEN, 'You do not have permission to submit orders.');
			}
		}
	}

下一个示例将审核信息写入 **audit** 表：

	function update(item, user, request) {
		request.execute({ success: insertAuditEntry });
		
		function insertAuditEntry() {
			var auditTable = tables.getTable('audit');
			var audit = {
				record: 'checkins',
				recordId: item.id,
				timestamp: new Date(),
				values: JSON.stringify(item)
			};
			auditTable.insert(audit, {
				success: function() {
					// Write to the response now that all data operations are complete
					request.respond();
				}
			});
		}
	}

以下部分的代码示例中提供了最后一个示例：[如何：访问自定义参数][How to: Add custom parameters]。

### <a name="bulk-inserts"></a>如何：执行批量插入

如果你使用 **for** 或 **while** 循环直接在表中插入大量的项（例如 1000 个），可能会遇到 SQL 连接限制，导致某些插入操作失败。你的请求可能永远无法完成，或者返回 HTTP 500 内部服务器错误。若要避免此问题，可以按照大约每 10 个一批的形式插入项。插入第一批后，再提交下一批，直至完成。

使用以下脚本可以设置要同时插入的记录批的大小。建议保持使用较小的记录数。完成异步插入批时，**insertItems** 函数将以递归方式调用自身。末尾的 for 循环一次插入一条记录，并在成功时调用 **insertComplete**，在出错时调用 **errorHandler**。**insertComplete** 控制是以递归方式为下一批调用 **insertItems**，还是在作业已完成的情况下退出脚本。

		var todoTable = tables.getTable('TodoItem');
		var recordsToInsert = 1000;
		var batchSize = 10; 
		var totalCount = 0;
		var errorCount = 0; 
		
		function insertItems() {        
		    var batchCompletedCount = 0;  
		
		    var insertComplete = function() { 
		        batchCompletedCount++; 
		        totalCount++; 
		        if(batchCompletedCount === batchSize || totalCount === recordsToInsert) {                        
		            if(totalCount < recordsToInsert) {
		                // kick off the next batch 
		                insertItems(); 
		            } else { 
		                // or we are done, report the status of the job 
		                // to the log and don't do any more processing 
		                console.log("Insert complete. %d Records processed. There were %d errors.", totalCount, errorCount); 
		            } 
		        } 
		    }; 
		
		    var errorHandler = function(err) { 
		        errorCount++; 
		        console.warn("Ignoring insert failure as part of batch.", err); 
		        insertComplete(); 
		    };
		
		    for(var i = 0; i < batchSize; i++) { 
		        var item = { text: "This is item number: " + totalCount + i }; 
		        todoTable.insert(item, { 
		            success: insertComplete, 
		            error: errorHandler 
		        }); 
		    } 
		} 
		
		insertItems(); 


可以在此[博客文章](http://blogs.msdn.com/b/jpsanders/archive/2013/03/20/server-script-to-insert-table-items-in-windows-azure-mobile-services.aspx)中找到整个代码示例和相关的讨论。如果使用此代码，你可以根据你的具体情况对它进行改写，并全面进行测试。

### <a name="JSON-types"></a>如何：将 JSON 类型映射到数据库类型

客户端上的数据类型集合不同于移动服务数据库表中的数据类型集合。有时它们可以轻松地映射到另一种类型，而其他时候不会映射。移动服务执行映射中的多种类型转换：

- 客户端语言特定的类型将序列化为 JSON。
- JSON 表示形式在出现于服务器脚本中之前将转换成 JavaScript。
- 使用 [tables 对象]保存 JavaScript 数据类型时，这些类型将转换成 SQL 数据库类型。

从客户端架构到 JSON 的转换根据平台的不同而异。Windows 应用商店和 Windows Phone 客户端使用 JSON.NET。Android 客户端使用 gson 库。iOS 客户端使用 NSJSONSerialization 类。将使用其中每个库的默认序列化行为，不过，日期对象将转换成 JSON 字符串，这些字符串包含使用 ISO 8601 编码的日期。

当你编写使用 [insert]、[update]、[read] 或 [delete] 函数的服务器脚本时，可以访问数据的 JavaScript 表示形式。移动服务使用 Node.js 的反序列化函数 ([JSON.parse](http://es5.github.io/#x15.12)) 将 JSON 在线转换为 JavaScript 对象。但是，移动服务将执行转换以提取 ISO 8601 字符串中的 **Date** 对象。

当你使用 [tables 对象]或 [mssql 对象]时，或只是执行表脚本时，将在 SQL 数据库中插入反序列化的 JavaScript 对象。在此过程中，对象属性将映射到 T-SQL 类型：

JavaScript 属性|T-SQL 类型
---|---
Number|Float(53)
Boolean|Bit
Date|DateTimeOffset(3)|
String|Nvarchar(max)
Buffer|不支持
对象|不支持
Array|不支持
Stream|不支持

### <a name="TSQL"></a>使用 Transact-SQL 访问表

从服务器脚本处理表数据的最简单方法就是使用 [table 对象]代理。但是，[table 对象]并不支持一些较为高级的方案，例如，联接查询和其他一些复杂查询，以及存储过程的调用。在这些情况下，你必须使用 [mssql 对象]针对关系表直接执行 Transact-SQL 语句。此对象提供以下函数：

- **query**：执行 TSQL 字符串指定的查询；结果将返回到 **options** 对象中的 **success** 回调。如果存在 *params* 参数，则该查询可以包含参数。
- **queryRaw**：与 *query* 类似，不过，从查询返回的结果集采用“原始”格式（请参阅以下示例）。
- **open**：用于获取与移动服务数据库建立的连接，获取该连接后，你可以使用连接对象来调用 transactions 等数据库操作。

这些方法可以进一步让你对查询处理进行低级别的控制。

+ [如何：运行静态查询]
+ [如何：运行动态查询]
+ [如何：联接关系表]
+ [如何：运行返回 *raw* 结果的查询]
+ [如何：获取对数据库连接的访问权限]	

#### <a name="static-query"></a>如何：运行静态查询

以下查询不带参数，将返回 `statusupdate` 表中的三条记录。行集采用标准的 JSON 格式。

		mssql.query('select top 3 * from statusupdates', {
		    success: function(results) {
		        console.log(results);
		    },
            error: function(err) {
                console.log("error is: " + err);
			}
		});


#### <a name="dynamic-query"></a>如何：运行动态参数化查询

以下示例通过从权限表中读取每个用户的权限来实现自定义授权。执行该查询时，占位符 (?) 将被替换为提供的参数。

		    var sql = "SELECT _id FROM permissions WHERE userId = ? AND permission = 'submit order'";
		    mssql.query(sql, [user.userId], {
		        success: function(results) {
		            if (results.length > 0) {
		                // Permission record was found. Continue normal execution. 
		                request.execute();
		            } else {
		                console.log('User %s attempted to submit an order without permissions.', user.userId);
		                request.respond(statusCodes.FORBIDDEN, 'You do not have permission to submit orders.');
		            }
		        },
            	error: function(err) {
                	console.log("error is: " + err);
				}	
		    });


#### <a name="joins"></a>如何：联接关系表

你可以使用 [mssql 对象]的 **query** 方法联接两个表，以传入实现联接的 TSQL 代码。假设 **ToDoItem** 表中有一些项，其中每个项都有一个对应于表中的列的 **priority** 属性。其中一个项类似于：

		{ text: 'Take out the trash', complete: false, priority: 1}

另外，我们假设还有一个名为 **Priority** 的表，它的行包含优先级 **number** 和文本 **description**。如果优先级编号 (number) 1 的描述 (description) 为“Critical”，则相应的对象类似于：

		{ number: 1, description: 'Critical'}

现在，我们可以将项中的 **priority** 编号替换为优先级编号的文本描述。将两个表进行关系联接即可实现此目的。

		mssql.query('SELECT t.text, t.complete, p.description FROM ToDoItem as t INNER JOIN Priority as p ON t.priority = p.number', {
			success: function(results) {
				console.log(results);
			},
            error: function(err) {
                console.log("error is: " + err);
		});
	
该脚本将联接两个表，并将结果写入日志。最终的对象可能类似于：

		{ text: 'Take out the trash', complete: false, description: 'Critical'}


#### <a name="raw"></a>如何：运行返回 *raw* 结果的查询

此示例将像前面一样执行查询，不过，这次会逐行逐列地返回需要你予以分析的“原始”格式结果集。用到此查询的可能情况是你需要访问移动服务不支持的数据类型。此代码会直接将输出写入控制台日志，使你能够检查原始格式。

		mssql.queryRaw('SELECT * FROM ToDoItem', {
		    success: function(results) {
		        console.log(results);
		    },
            error: function(err) {
                console.log("error is: " + err);
			}
		});

下面显示了运行此查询后的输出。其中包含有关表中每个列的元数据，后接行和列的表示形式。

		{ meta: 
		   [ { name: 'id',
		       size: 19,
		       nullable: false,
		       type: 'number',
		       sqlType: 'bigint identity' },
		     { name: 'text',
		       size: 0,
		       nullable: true,
		       type: 'text',
		       sqlType: 'nvarchar' },
		     { name: 'complete',
		       size: 1,
		       nullable: true,
		       type: 'boolean',
		       sqlType: 'bit' },
		     { name: 'priority',
		       size: 53,
		       nullable: true,
		       type: 'number',
		       sqlType: 'float' } ],
		  rows: 
		   [ [ 1, 'good idea for the future', null, 3 ],
		     [ 2, 'this is important but not so much', null, 2 ],
		     [ 3, 'fix this bug now', null, 0 ],
		     [ 4, 'we need to fix this one real soon now', null, 1 ],
		   ] }

#### <a name="connection"></a>如何：获取对数据库连接的访问权限

可以使用 **open** 方法获取对数据库连接的访问权限。执行此操作的原因之一是你需要使用数据库事务。

成功执行 **open** 后，将在 **success** 函数中以参数形式传入数据库连接。你可以对 **connection** 对象调用以下任何函数：*close*、*queryRaw*、*query*、*beginTransaction*、*commit* 和 *rollback*。

		    mssql.open({
		        success: function(connection) {
		            connection.query(//query to execute);
		        },
	            error: function(err) {
	                console.log("error is: " + err);
				}
		    });

## <a name="debugging"></a>调试和故障排除

调试服务器脚本及排查其错误的主要方法是写入服务日志。默认情况下，移动服务会将执行服务脚本期间发生的错误写入服务日志。你的脚本也可以对日志执行写入操作。写入日志是调试脚本及验证其行为是否符合预期的良好方法。

### <a name="write-to-logs"></a>如何：将输出写入移动服务日志

若要写入日志，请使用全局 [console 对象]。使用 **log** 或 **info** 函数记录信息级警告。**warning** 和 **error** 函数将记录其对应级别，这些级别已在日志中予以标注。

> [AZURE.NOTE]若要查看移动服务的日志，请登录到 [Azure 管理门户](https://manage.windowsazure.cn/)，选择你的移动服务，然后选择“日志”选项卡。

你还可以使用 [console 对象]的日志记录功能通过参数来设置消息格式。以下示例向消息字符串提供了一个参数形式的 JSON 对象：

	function insert(item, user, request) {
	    console.log("Inserting item '%j' for user '%j'.", item, user);  
	    request.execute();
	}

请注意，字符串 `%j` 用作 JSON 对象的占位符，并且参数是按顺序提供的。

为了避免在日志中记录过多的信息，你应该删除或禁用生产环境不需要使用的 console.log() 调用。

<!-- Anchors. -->

[Introduction]: #intro
[Table operations]: #table-scripts
[Basic table operations]: #basic-table-ops
[如何：注册表操作]: #register-table-scripts
[How to: Define table scripts]: #execute-operation
[如何：重写默认响应]: #override-response
[How to: Modify an operation]: #modify-operation
[How to: Override success and error]: #override-success-error
[如何：重写 execute success]: #override-success
[如何：重写默认错误处理]: #override-error
[如何：从脚本访问表]: #access-tables
[How to: Add custom parameters]: #access-headers
[如何：添加自定义参数]: #access-headers
[How to: Work with users]: #work-with-users
[How to: Define scheduled job scripts]: #scheduler-scripts
[How to: Refine access to tables]: #authorize-tables
[使用 Transact-SQL 访问表]: #TSQL
[如何：运行静态查询]: #static-query
[如何：运行动态查询]: #dynamic-query
[如何：运行返回 *raw* 结果的查询]: #raw
[如何：获取对数据库连接的访问权限]: #connection
[如何：联接关系表]: #joins
[如何：执行批量插入]: #bulk-inserts
[如何：将 JSON 类型映射到数据库类型]: #JSON-types
[如何：加载 Node.js 模块]: #modules-helper-functions
[How to: Write output to the mobile service logs]: #write-to-logs
[Source control, shared code, and helper functions]: #shared-code
[源代码管理、共享代码和 Helper 函数]: #shared-code
[Using the command line tool]: #command-prompt
[使用命令行工具]: #command-prompt
[Working with tables]: #working-with-tables
[Custom API anchor]: #custom-api
[如何：定义自定义 API]: #define-custom-api
[How to: Share code by using source control]: #shared-code-source-control
[如何：使用源代码管理来共享代码]: #shared-code-source-control
[如何：使用 Helper 函数]: #helper-functions
[Debugging and troubleshooting]: #debugging
[如何：实现 HTTP 方法]: #handle-methods
[如何：处理用户和自定义 API 中的标头]: #get-api-user
[How to: Access custom API request headers]: #get-api-headers
[Job Scheduler]: #scheduler-scripts
[如何：在一个自定义 API 中定义多个路由]: #api-routes
[如何：发送和接收 XML 格式的数据]: #api-return-xml
[如何：使用应用程序设置]: #app-settings

[1]: ./media/mobile-services-how-to-use-server-scripts/1-mobile-insert-script-users.png
[2]: ./media/mobile-services-how-to-use-server-scripts/2-mobile-custom-api-script.png
[3]: ./media/mobile-services-how-to-use-server-scripts/3-mobile-schedule-job-script.png
[4]: ./media/mobile-services-how-to-use-server-scripts/4-mobile-source-local-cli.png

<!-- URLs. -->
[移动服务服务器脚本参考]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554226.aspx

[request object]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554218.aspx
[request 对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554218.aspx
[response object]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn303373.aspx
[response 对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn303373.aspx
[User object]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554220.aspx
[user 对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554220.aspx
[用户对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554220.aspx
[push object]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554217.aspx
[insert function]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554229.aspx
[insert]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554229.aspx
[update function]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554214.aspx
[delete function]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554215.aspx
[read function]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554224.aspx
[update]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554214.aspx
[delete]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554215.aspx
[read]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554224.aspx
[query object]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj613353.aspx
[query 对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj613353.aspx
[apns object]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj839711.aspx
[mpns object]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj871025.aspx
[wns object]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj860484.aspx
[table 对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554210.aspx
[tables 对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj614364.aspx
[mssql 对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554212.aspx
[console 对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554209.aspx
[读取和写入数据]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj631640.aspx
[验证数据]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj631638.aspx
[修改请求]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj631635.aspx
[修改响应]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj631631.aspx
[Azure 管理门户]: https://manage.windowsazure.cn/
[计划作业]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj860528.aspx
[使用服务器脚本在移动服务中验证和修改数据]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-validate-modify-data-server-scripts/
[用于管理 Azure 移动服务的命令]: /zh-cn/documentation/articles/command-line-tools/#Commands_to_manage_mobile_services/#Mobile_Scripts
[Windows Store Push]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push/
[Windows Phone Push]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-wp8/
[iOS Push]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-ios/
[Android Push]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-android/
[Azure SDK for Node.js]: http://go.microsoft.com/fwlink/p/?LinkId=275539
[发送 HTTP 请求]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj631641.aspx
[使用 SendGrid 从移动服务发送电子邮件]: /zh-cn/documentation/articles/store-sendgrid-mobile-services-send-email-scripts/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users
[crypto API]: http://go.microsoft.com/fwlink/p/?LinkId=288802
[path API]: http://go.microsoft.com/fwlink/p/?LinkId=288803
[querystring API]: http://go.microsoft.com/fwlink/p/?LinkId=288804
[url API]: http://go.microsoft.com/fwlink/p/?LinkId=288805
[util API]: http://go.microsoft.com/fwlink/p/?LinkId=288806
[zlib API]: http://go.microsoft.com/fwlink/p/?LinkId=288807
[自定义 API]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn280974.aspx
[从客户端调用自定义 API]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-call-custom-api/#define-custom-api
[express.js 库]: http://go.microsoft.com/fwlink/p/?LinkId=309046
[定义支持定期通知的自定义 API]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-create-pull-notifications/
[express.js 中的 express 对象]: http://expressjs.com/api.html#express
[Store server scripts in source control]: /zh-cn/documentation/articles/mobile-services-store-scripts-source-control/
[在服务器脚本中利用共享代码和 Node.js 模块]: /zh-cn/documentation/articles/mobile-services-store-scripts-source-control/#use-npm
[service 对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn303371.aspx
[应用设置]: http://msdn.microsoft.com/zh-cn/library/dn529070.aspx
[config module]: http://msdn.microsoft.com/zh-cn/library/dn508125.aspx
[Azure 移动服务中对 package.json 的支持]: http://go.microsoft.com/fwlink/p/?LinkId=391036

<!---HONumber=Mooncake_0118_2016-->