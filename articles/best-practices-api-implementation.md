<properties
   pageTitle="API 实现指南 | Windows Azure"
   description="有关如何实现 API 的指南。"
   services=""
   documentationCenter="na"
   authors="dragon119"
   manager="masimms"
   editor=""
   tags=""/>

<tags
   ms.service="best-practice"
   ms.date="05/13/2015"
   wacn.date="08/29/2015"/>

# API 实现指南

本指南中的一些主题正在讨论中，将来可能会更改。我们欢迎你的反馈！

## 概述
精心设计的 REST 样式 Web API 定义了客户端应用程序可以访问的资源、关系和导航方案。当你实现和部署 Web API 时，应考虑托管该 Web API 的环境的物理要求以及构造该 Web API 的方式，而不是考虑数据的逻辑结构。本指南重点介绍实现 Web API 并发布它，使其可供客户端应用程序使用的最佳实践。安全问题将在 API 安全指南文档中单独说明。你可以在 API 设计指南文档中找到有关 Web API 设计的详细信息。

## 有关实现 REST 样式 Web API 的注意事项
以下各节说明了使用 ASP.NET Web API 模板构建 REST 样式 Web API 的最佳做法。有关使用 Web API 模板的详细信息，请访问 Microsoft 网站上的[了解 ASP.NET Web API](http://www.asp.net/web-api) 页。

## 有关实现请求路由的注意事项

在使用 ASP.NET Web API 实现的服务中，每个请求都将路由到_控制器_类中的某个方法。Web API 框架提供了用于实现路由的两个主要选项：_基于约定的_路由和_基于属性的_路由。在你确定使用 Web API 路由请求的最佳方式时，请考虑以下几点：

- **了解基于约定的路由的限制和要求**。

	默认情况下，Web API 框架使用基于约定的路由。Web API 框架将创建包含以下条目的初始路由表：

	```C#
	config.Routes.MapHttpRoute(
  		name: "DefaultApi",
	  	routeTemplate: "api/{controller}/{id}",
	  	defaults: new { id = RouteParameter.Optional }
	);
	```

	路由可以是包含文本的泛型，例如 _api_ 和 变量（如 _{controller}_ 和 _{id}_）。基于约定的路由允许路由的某些元素是可选的。Web API 框架通过将请求中的 HTTP 方法与 API 中方法名称的开头部分进行匹配，然后匹配任何可选参数来确定在控制器中调用哪个方法。例如，如果名为 _orders_ 的控制器包含方法 _GetAllOrders()_ 或 _GetOrderByInt(int id)_，则 GET 请求 __http://www.adventure-works.com/api/orders/_ 将定向到方法 _GetAlllOrders()_，而 GET 请求 __http://www.adventure-works.com/api/orders/99_ 则将路由到方法 _GetOrderByInt(int id)_。如果控制器中没有以前缀 Get 开头的匹配方法可用，则 Web API 框架将以“HTTP 405 (不允许的方法)”消息进行答复。此外，路由表中指定的参数 (id) 的名称还必须与 _GetOrderById_ 方法的参数名称相同，否则 Web API 框架将以“HTTP 404 (未找到)”响应进行答复。

	相同的规则也适用于 POST、PUT 和 DELETE HTTP 请求；更新订单 101 的详细信息的 PUT 请求将定向到 URI __http://www.adventure-works.com/api/orders/101_，消息正文将包含订单的新详细信息，并且此信息将作为参数传递给 orders 控制器中名称以前缀 _Put_ 开头的方法（如 _PutOrder_）。

	默认路由表不会与引用 REST 样式 Web API 中的子资源的请求匹配，如 __http://www.adventure-works.com/api/customers/1/orders_（查找客户 1 所下的所有订单的详细信息）。若要处理这些情况，可以向路由表添加自定义路由：

	```C#
	config.Routes.MapHttpRoute(
	    name: "CustomerOrdersRoute",
	    routeTemplate: "api/customers/{custId}/orders",
	    defaults: new { controller="Customers", action="GetOrdersForCustomer" })
	);
	```

	此路由将与 URI 匹配的请求定向到 _Customers_ 控制器中的 _GetOrdersForCustomer_ 方法。此方法必须采用名为 _custI_ 的单个参数：

	```C#
	public class CustomersController : ApiController
	{
	    ...
	    public IEnumerable<Order> GetOrdersForCustomer(int custId)
	    {
	        // Find orders for the specified customer
	        var orders = ...
	        return orders;
	    }
	    ...
	}
	```

	> [AZURE.TIP]尽可能利用默认路由并避免定义很多复杂的自定义路由，因为这可能会导致易受攻击（很容易向控制器添加导致多义性路由的方法）和性能降低（路由表越大，Web API 框架越不得不执行更多操作来找出哪个路由与给定的 URI 匹配）。简化 API 和路由。有关详细信息，请参阅 API 设计指南中的“围绕资源组织 Web API”一节。如果你必须定义自定义路由，则更可取的方法是使用本节稍后所述的基于属性的路由。

	有关基于约定的路由的详细信息，请参阅 Microsoft 网站上的 [ASP.NET Web API 中的路由](http://www.asp.net/web-api/overview/web-api-routing-and-actions/routing-in-aspnet-web-api)页。

- **避免路由中的多义性**。

	如果控制器中的多个方法匹配同一路由，则基于约定的路由可能会导致多义性路径。在这些情况下，Web API 框架将以“HTTP 500 (内部服务器错误)”响应消息（包含文本“找到与请求匹配的多个操作”）进行响应。

- **首选基于属性的路由**。

	基于属性的路由提供了一种替代方法，用于将路由连接到控制器中的方法。它不依赖于基于约定的路由的模式匹配功能，而是可以显式使用控制器中的方法应关联到的路由的详细信息标注这些方法。此方法可帮助消除可能的多义性。此外，由于在设计时定义了显式路由，此方法在运行时比基于约定的路由更高效。下面的代码演示如何将 _Route_ 属性应用于 Customers 控制器中的方法。这些方法还使用 HttpGet 属性以指示它们应响应 _HTTP GET_ 请求。此属性允许你使用任何方便的命名方案（而不是基于约定的路由所需的命名方案）来命名方法。你还可以使用 _HttpPost_、_HttpPut_ 和 _HttpDelete_ 属性来标注方法以定义响应其他类型的 HTTP 请求的方法。

	```C#
	public class CustomersController : ApiController
	{
	    ...
	    [Route("api/customers/{id}")]
	    [HttpGet]
	    public Customer FindCustomerByID(int id)
	    {
	        // Find the matching customer
	        var customer = ...
	        return customer;
	    }
	    ...
	    [Route("api/customers/{id}/orders")]
	    [HttpGet]
	    public IEnumerable<Order> FindOrdersForCustomer(int id)
	    {
	        // Find orders for the specified customer
	        var orders = ...
	        return orders;
	    }
	    ...
	}
	```

	基于属性的路由还具有有用的副效用，即充当需要在将来维护代码的开发人员的文档；它让你一目了然地了解哪个方法属于哪个路由，_HttpGet_ 属性阐明了该方法响应的 HTTP 请求的类型。

	基于属性的路由允许你定义用于限制匹配参数的方式的约束。约束可以指定参数的类型，并且在某些情况下它们还可以指示参数值的可接受范围。在下面的示例中，_FindCustomerByID_ 方法的 id 参数必须为非负整数。如果应用程序提交了包含负客户编号的 HTTP GET 请求，Web API 框架将以“HTTP 405 (不允许的方法)”消息进行响应：

	```C#
	public class CustomersController : ApiController
	{
	    ...
	    [Route("api/customers/{id:int:min(0)}")]
	    [HttpGet]
	    public Customer FindCustomerByID(int id)
	    {
	        // Find the matching customer
	        var customer = ...
	        return customer;
	    }
	    ...
	}
	```

	有关基于属性的路由的详细信息，请参阅 Microsoft 网站上的 [Web API 2 中的属性路由](http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2)页。

- **在路由中支持 Unicode 字符**。

	用于在 GET 请求中标识资源的键可以是字符串。因此，在全局应用程序中，你可能需要支持包含非英语字符的 URI。

- **区分不应路由的方法**。

	如果你使用的是基于约定的路由，请通过使用 _NonAction_ 属性修饰不对应于 HTTP 操作的方法来指示这些方法。这通常适用于设计为供控制器中的其他方法使用的帮助器方法，此属性将阻止错误的 HTTP 请求匹配和调用这些方法。

- **考虑在子域中放置 API 的优点和缺点**。

	默认情况下，ASP.NET Web API 将 API 组织到域中的 /api 目录，如 http://www.adventure-works.com/api/orders。 此目录与同一主机公开的任何其他服务位于同一域中。最好将 Web API 拆分到具有 URI（如 http://api.adventure-works.com/orders） 的单独主机上运行的其自己的子域中。这种分离使你可以更有效地对 Web API 进行分隔和缩放，而不会影响 www.adventure-works.com 域中运行的任何其他 Web 应用程序或服务。

	但是，将 Web API 放在不同的子域中也可能会带来安全问题。在 _www.adventure-works.com_ 上托管的任何 Web 应用程序或服务调用在其他位置运行的 Web API 可能会违反很多 Web 浏览器的同源策略。在这种情况下，将需要启用主机之间的跨域资源共享 (CORS)。有关详细信息，请参阅 API 安全指南文档。

## 有关处理请求的注意事项

在来自客户端应用程序的请求已成功路由到 Web API 中的方法后，必须以尽可能高效的方式处理该请求。在实现处理请求的代码时，请考虑以下几点：

- **GET、PUT、DELETE、HEAD 和 PATCH 操作应当是幂等的**。

	实现这些请求的代码不应施加任何副作用。对同一资源重复进行同一请求应导致同一状态。例如，向同一 URI 发送多个 DELETE 请求应具有相同的效果，尽管响应消息中的 HTTP 状态代码可能会有所不同（第一个 DELETE 请求可能会返回状态代码 204（无内容），而后续的 DELETE 请求可能会返回状态代码 404（未找到））。

> [AZURE.NOTE]Jonathan Oliver 博客上的[幂等性模式](http://blog.jonathanoliver.com/idempotency-patterns/)一文概述了幂等性以及它如何与数据管理操作相关。

- **创建新资源的 POST 操作应这样做才不会有不相关的副作用**。

	如果 POST 请求用于创建新资源，则请求的作用应限制为新资源（以及任何可能直接相关的资源，如果涉及某种关联）。例如，在电子商务系统中，创建新客户订单的 POST 请求可能还修改库存水平并生成计费信息，但它不应修改与订单不直接相关的信息，也不应对系统的整体状态有任何其他副作用。

- **避免实现琐碎的 POST、PUT 和 DELETE 操作**。

	支持对资源集合发出 POST、PUT 和 DELETE 请求。POST 请求可以包含多个新资源的详细信息并将这些资源全都添加到同一个集合，PUT 请求可以替换集合中的整个资源集，DELETE 请求可以删除整个集合。

	请注意，ASP.NET Web API 2 中包含的 OData 支持提供了成批处理请求的功能。客户端应用程序可以打包多个 Web API 请求并通过单个 HTTP 请求将其发送给服务器，然后接收包含每个请求的答复的单个 HTTP 响应。有关详细信息，请参阅 Microsoft 网站上的 [Web API 和 Web API OData 中的批处理支持简介](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx)。

- **将响应发送回客户端应用程序时遵守 HTTP 协议**。

	Web API 必须返回包含正确 HTTP 状态代码的消息（使客户端可以确定如何处理结果）、相应的 HTTP 标头（以便客户端了解结果的性质），以及适当格式化的正文（使客户端可以分析结果）。如果你使用的是 ASP.NET Web API 模板，则实现响应 HTTP POST 请求的方法的默认策略只是返回新创建的资源的副本，如下面的示例所示：

	```C#
	public class CustomersController : ApiController
	{
	    ...
	    [Route("api/customers")]
	    [HttpPost]
	    public Customer CreateNewCustomer(Customer customerDetails)
	    {
	        // Add the new customer to the repository
	        // This method returns a customer with a unique ID allocated
	        // by the repository
	        var newCust = repository.Add(customerDetails);
	        // Return the newly added customer
	        return newCust;
	    }
	    ...
	}
	```

	如果 POST 操作成功，则 Web API 框架将创建具有状态代码 200（正常）的 HTTP 响应并将客户的详细信息作为消息正文。但是，在本示例中，根据 HTTP 协议，POST 操作应返回状态代码 201（已创建），并且响应消息应在其 Location 标头中包含新创建的资源的 URI。

	若要提供这些功能，请通过使用 `IHttpActionResult` 接口返回你自己的 HTTP 响应消息。这种方法使你能够很好地控制 HTTP 状态代码、响应消息中的标头，甚至响应消息正文中的数据格式，如下面的代码示例中所示。此版本的 `CreateNewCustomer` 方法更紧密地符合人们对执行 HTTP 协议的客户端的期望。`ApiController` 类的 `Created` 方法基于指定的数据构造响应消息，并将 Location 标头添加到结果：

	```C#
	public class CustomersController : ApiController
	{
	    ...
	    [Route("api/customers")]
	    [HttpPost]
	    public IHttpActionResult CreateNewCustomer(Customer customerDetails)
	    {
	        // Add the new customer to the repository
	        var newCust = repository.Add(customerDetails);

	        // Create a value for the Location header to be returned in the response
	        // The URI should be the location of the customer including its ID,
	        // such as http://adventure-works.com/api/customers/99
	        var location = new Uri(...);

            // Return the HTTP 201 response,
            // including the location and the newly added customer
	        return Created(location, newCust);
	    }
	    ...
	}
	```

- **支持内容协商**。

	响应消息的正文可能包含不同格式的数据。例如，HTTP GET 请求可以返回 JSON 或 XML 格式的数据。客户端提交请求时，它可以包括指定它可以处理的数据格式的 Accept 标头。这些格式将指定为媒体类型。例如，发出用于检索图像的 GET 请求的客户端可以指定 Accept 标头，其中列出客户端可以处理的媒体类型，如“image/jpeg, image/gif, image/png”。当 Web API 返回结果时，它应使用其中一种媒体类型设置数据格式，并在响应的 Content-Type 标头中指定该格式。

	如果客户端未指定 Accept 标头，则对响应正文使用有意义的默认格式。例如，ASP.NET Web API 框架对基于文本的数据默认使用 JSON 格式。

	> [AZURE.NOTE]ASP.NET Web API 框架执行 Accept 标头的某种自动检测，并根据响应消息正文中的数据类型自行处理它们。例如，如果响应消息正文中包含 CLR（公共语言运行时）对象，则 ASP.NET Web API 会将响应的格式自动设置为 JSON，并将响应的 Content-Type 标头设置为“application/json”，除非客户端指示它需要 XML 格式的结果，在这种情况下，ASP.NET Web API 框架会将响应的格式设置为 XML，并将响应的 Content-Type 标头设置为“text/xml”。但是，可能需要处理在操作的实现代码中显式指定多种媒体类型的 Accept 标头。

- **提供支持 HATEOAS 样式导航和资源发现的链接**。

	API 设计指南介绍以下 HATEOAS 方法如何使客户端能够从初始的起点导航和发现资源。这通过使用包含 URI 的链接来实现；当客户端发出 HTTP GET 请求以获取资源时，响应应包含使客户端应用程序能够快速找到任何直接相关的资源的 URI。例如，在支持电子商务解决方案的 Web API 中，客户可能下了多个订单。当客户端应用程序检索客户的详细信息时，响应中应该包含使客户端应用程序能够发送可检索这些订单的 HTTP GET 请求的链接。此外，HATEOAS 样式的链接应描述每个链接的资源支持的其他操作（POST、PUT、DELETE 等）以及用于执行每个请求的相应 URI。此方法将在 API 设计指南文档中更详细地介绍。

	当前没有控制 HATEOAS 的实现的标准，但下面的示例说明了一种可行方法。在此示例中，用于查找客户的详细信息的 HTTP GET 请求返回一个响应，其中包括引用该客户的订单的 HATEOAS 链接：

	```HTTP
	GET http://adventure-works.com/customers/2 HTTP/1.1
	Accept: text/json
	...
	```

	```HTTP
	HTTP/1.1 200 OK
	...
	Content-Type: application/json; charset=utf-8
	...
	Content-Length: ...
	{"CustomerID":2,"CustomerName":"Bert","Links":[
	  {"Relationship":"self",
	   "HRef":"http://adventure-works.com/customers/2",
	   "Action":"GET",
	   "LinkedResourceMIMETypes":["text/xml","application/json"]},
	  {"Relationship":"self",
	   "HRef":"http://adventure-works.com/customers/2",
	   "Action":"PUT",
	   "LinkedResourceMIMETypes":["application/x-www-form-urlencoded"]},
	  {"Relationship":"self",
	   "HRef":"http://adventure-works.com/customers/2",
	   "Action":"DELETE",
	   "LinkedResourceMIMETypes":[]},
	  {"Relationship":"orders",
	   "HRef":"http://adventure-works.com/customers/2/orders",
	   "Action":"GET",
	   "LinkedResourceMIMETypes":["text/xml","application/json"]},
	  {"Relationship":"orders",
	   "HRef":"http://adventure-works.com/customers/2/orders",
	   "Action":"POST",
	   "LinkedResourceMIMETypes":["application/x-www-form-urlencoded"]}
	]}
	```

	在此示例中，客户数据由以下代码片段中所示的 `Customer` 类表示。HATEOAS 链接在 `Links` 集合属性中保存：

	```C#
	public class Customer
	{
    	public int CustomerID { get; set; }
    	public string CustomerName { get; set; }
    	public List<Link> Links { get; set; }
    	...
	}

	public class Link
	{
    	public string Relationship { get; set; }
    	public string HRef { get; set; }
    	public string Action { get; set; }
    	public string [] LinkedResourceMIMETypes { get; set; }
	}
	```

	HTTP GET 操作从存储中检索客户数据并构造 `Customer` 对象，然后填充 `Links` 集合。结果将格式化为 JSON 响应消息。每个链接包含以下字段：

	- 返回的对象与链接所描述的对象之间的关系。在此示例中，“self”指示该链接是返回到对象本身的引用（类似于许多面向对象语言中的 `this` 指针），而“orders”则是包含相关订单信息的集合的名称。

	- URI 形式的链接所描述的对象的超链接 (`HRef`)。

	- 可发送到此 URI 的 HTTP 请求的类型 (`Action`)。

	- 应在 HTTP 请求中提供或可在响应中返回的任何数据的格式 (`LinkedResourceMIMETypes`)，具体取决于请求的类型。

	HTTP 响应的示例中所示的 HATEOAS 链接指示客户端应用程序可以执行以下操作：

	- 向 URI *http://adventure-works.com/customers/2* 发出 HTTP GET 请求以提取客户的详细信息（再次）。数据可以 XML 或 JSON 格式返回。

	- 向 URI *http://adventure-works.com/customers/2* 发出 HTTP PUT 请求以修改客户的详细信息。必须在请求消息中以 x-www-form-urlencoded 格式提供新数据。

	- 向 URI *http://adventure-works.com/customers/2* 发出 HTTP DELETE 请求以删除客户。该请求不需要任何其他信息，也不需要在响应消息正文中返回数据。

	- 向 URI *http://adventure-works.com/customers/2/orders* 发出 HTTP GET 请求以查找客户的所有订单。数据可以 XML 或 JSON 格式返回。

	- 向 URI *http://adventure-works.com/customers/2/orders* 发出 HTTP PUT 请求以创建此客户的新订单。必须在请求消息中以 x-www-form-urlencoded 格式提供数据。

## 有关处理异常的注意事项
默认情况下，在 ASP.NET Web API 框架中，如果操作引发未捕获的异常，则框架将返回具有 HTTP 状态代码 500（内部服务器错误）的响应消息。在许多情况下，此简单方法单独使用并不十分有用，并且会导致很难确定异常的原因。因此，你应采用更全面的方法来处理异常，请考虑以下几点：

- **捕获异常并向客户端返回有意义的响应**。

	实现 HTTP 操作的代码应提供全面的异常处理，而不是让未捕获的异常传播到 Web API 框架。如果异常会导致无法成功完成此操作，则可以在响应消息中传递回此异常，但它应包括导致异常的错误的有意义描述。该异常还应包括相应的 HTTP 状态代码，而不是对于每种情况都只返回状态代码 500。例如，如果用户请求导致了违反约束的数据库更新（例如，尝试删除具有未完成订单的客户），则应返回状态代码 409（冲突）和指示冲突原因的消息正文。如果某种其他情况显示请求无法完成，则可以返回状态代码 400（错误请求）。你可以在 W3C 网站上的[状态代码定义](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)页中找到 HTTP 状态代码的完整列表。

	下面的代码演示了捕获不同条件并返回相应响应的示例。

	```C#
	[HttpDelete]
	[Route("customers/{id:int}")]
	public IHttpActionResult DeleteCustomer(int id)
	{
		try
		{
			// Find the customer to be deleted in the repository
			var customerToDelete = repository.GetCustomer(id);

			// If there is no such customer, return an error response
			// with status code 404 (Not Found)
			if (customerToDelete == null)
			{
					return NotFound();
			}

			// Remove the customer from the repository
			// The DeleteCustomer method returns true if the customer
			// was successfully deleted
			if (repository.DeleteCustomer(id))
			{
				// Return a response message with status code 204 (No Content)
				// To indicate that the operation was successful
				return StatusCode(HttpStatusCode.NoContent);
			}
			else
			{
				// Otherwise return a 400 (Bad Request) error response
				return BadRequest(Strings.CustomerNotDeleted);
			}
		}
		catch
		{
			// If an uncaught exception occurs, return an error response
			// with status code 500 (Internal Server Error)
			return InternalServerError();
		}
	}
	```

	> [AZURE.TIP]不要包括可能对尝试入侵你的 Web API 的攻击者有用的信息。有关详细信息，请访问 Microsoft 网站上的 [ASP.NET Web API 中的异常处理](http://www.asp.net/web-api/overview/error-handling/exception-handling)页。

	> [AZURE.NOTE]许多 Web 服务器在错误条件到达 Web API 之前，自行捕获错误条件。例如，如果你为网站配置了身份验证，但用户无法提供正确的身份验证信息，则 Web 服务器应以状态代码 401（未经授权）进行响应。客户端经过身份验证后，你的代码可以执行自己的检查来验证客户端是否应能够访问所请求的资源。如果此授权失败，则应返回状态代码 403（禁止访问）。

- **以一致方式处理异常，并记录有关错误的信息**。

	若要以一致方式处理异常，请考虑在整个 Web API 中实现全局错误处理策略。可以通过创建一个异常筛选器来实现部分此功能，该异常筛选器在每次控制器引发非 `HttpResponseException` 异常的任何未处理异常时运行。此方法在 Microsoft 网站上的 [ASP.NET Web API 中的异常处理](http://www.asp.net/web-api/overview/error-handling/exception-handling)页中进行了介绍。

	但是，在几种情况下，异常筛选器将不会捕获异常，这些情况包括：

	- 从控制器构造函数引发的异常。

	- 从消息处理程序引发的异常。

	- 在路由过程中引发的异常。

	- 序列化响应消息的内容时引发的异常。

	若要处理这种情况，可能需要实现更具自定义性的方法。你还应将整合捕获每个异常的完整详细信息的错误日志记录；只要客户端无法通过 Web 访问它，此错误日志就会包含详细的信息。Microsoft 网站上的 [Web API 全局错误处理](http://www.asp.net/web-api/overview/error-handling/web-api-global-error-handling)一文显示了执行此任务的一种方式。

- **区分客户端错误和服务器端错误**。

	HTTP 协议可区分因客户端应用程序发生的错误（HTTP 4xx 状态代码）和因服务器上的事故导致的错误（HTTP 5xx 状态代码）。请确保在任何错误响应消息中遵守此约定。

<a name="considerations-for-optimizing"></a>
## 有关优化客户端数据访问的注意事项

例如，在分布式环境（例如，涉及 Web 服务器和客户端应用程序）中，主要问题源之一是网络。这可能表现为值得注意的瓶颈问题，尤其当客户端应用程序频繁地将发送请求或接收数据时。因此，你的目标应是将网络间流动的通信量降到最低。在实现检索和维护数据的代码时，请考虑以下几点：

- **支持客户端缓存**。

	HTTP 1.1 协议支持在客户端和中间服务器中缓存，请求通过这些客户端和服务器使用 Cache-Control 标头进行路由。当客户端应用程序向 Web API 发送 HTTP GET 请求时，响应可以包含 Cache-Control 标头，以指示响应的正文中的数据是否可以由通过其路由该请求的客户端或中间服务器安全地缓存，以及多长时间之后应失效并视为过期。下面的示例演示了 HTTP GET 请求和包含 Cache-Control 标头的相应响应：

	```HTTP
	GET http://adventure-works.com/orders/2 HTTP/1.1
	...
	```

	```HTTP
	HTTP/1.1 200 OK
	...
	Cache-Control: max-age=600, private
	Content-Type: text/json; charset=utf-8
	Content-Length: ...
	{"OrderID":2,"ProductID":4,"Quantity":2,"OrderValue":10.00}
	```

	在此示例中，Cache-Control 标头指定返回的数据应在 600 秒后过期并且只适用于单个客户端，不能在其他客户端使用的共享缓存中存储（它是_私有_的）。Cache-Control 可以指定 _public_（而不是 _private_），在这种情况下，数据可以存储在共享缓存中，或者它可以指定 _no-store_，在这种情况下，数据**不**能由客户端缓存。下面的代码示例显示如何在响应消息中构造 Cache-Control 标头：

	```C#
	public class OrdersController : ApiController
	{
    	...
    	[Route("api/orders/{id:int:min(0)}")]
    	[HttpGet]
    	public IHttpActionResult FindOrderByID(int id)
    	{
    		// Find the matching order
    		Order order = ...;
    		...
    		// Create a Cache-Control header for the response
    		var cacheControlHeader = new CacheControlHeaderValue();
    		cacheControlHeader.Private = true;
    		cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
    		...

    		// Return a response message containing the order and the cache control header
    		OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
    		{
    			CacheControlHeader = cacheControlHeader
    		};
    		return response;
    	}
    	...
	}
	```

	此代码使用名为 `OkResultWithCaching` 的自定义 `IHttpActionResult` 类。此类使控制器可设置缓存标头内容：

	```C#
	public class OkResultWithCaching<T> : OkNegotiatedContentResult<T>
    {
        public OkResultWithCaching(T content, ApiController controller)
            : base(content, controller) { }

        public OkResultWithCaching(T content, IContentNegotiator contentNegotiator, HttpRequestMessage request, IEnumerable<MediaTypeFormatter> formatters)
            : base(content, contentNegotiator, request, formatters) { }

        public CacheControlHeaderValue CacheControlHeader { get; set; }
        public EntityTagHeaderValue ETag { get; set; }

        public override async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
        {
            HttpResponseMessage response = await base.ExecuteAsync(cancellationToken);

            response.Headers.CacheControl = this.CacheControlHeader;
            response.Headers.ETag = ETag;

            return response;
        }
    }
	```

	> [AZURE.NOTE]HTTP 协议还为 Cache-Control 标头定义了 _no-cache_ 指令。令人困惑的是，此指令并不意味着“不缓存”而是指示“在返回信息之前使用服务器重新验证缓存的信息”；数据仍可以缓存，但在每次使用它时检查以确保它仍是最新的。

	缓存管理是客户端应用程序或中间服务器的职责，但如果正确实现它，可以节省带宽和提高性能，因为它可以消除提取最近已检索的数据的需要。

	Cache-Control 标头中的 _max-age_ 值只是指南，并不保证相应的数据在指定的时间期间内不会更改。Web API 应将 max-age 设置为适当的值，具体取决于数据的预期波动性。此期间到期后，客户端应放弃缓存中的对象。

	> [AZURE.NOTE]如前所述，大多数现代 Web 浏览器通过向请求添加相应的 Cache-Control 标头并检查结果的标头来支持客户端缓存。但是，某些较旧的浏览器将不缓存从包含查询字符串的 URL 返回的值。这通常不是基于此处讨论的协议实现自己的缓存管理策略的自定义客户端应用程序的问题。
	>
	> 某些较旧的代理显示相同的行为并可能不会基于包含查询字符串的 URL 缓存请求。这可能是通过此类代理连接到 Web 服务器的自定义客户端应用程序的问题。

- **提供 Etag 以优化查询处理**。

	当客户端应用程序检索对象时，响应消息还可以包括 _ETag_（实体标记）。ETag 是不透明的字符串，它指示资源的版本，每次资源发生更改时，也会修改 Etag。客户端应用程序应将此 ETag 作为数据的一部分缓存。下面的代码示例演示如何添加 ETag 作为 HTTP GET 请求的响应的一部分。此代码使用对象的 `GetHashCode` 方法生成用于标识对象的数字值（如有必要，可以使用 MD5 等算法重写此方法并生成你自己的哈希值）：

	```C#
	public class OrdersController : ApiController
	{
    	...
    	public IHttpActionResult FindOrderByID(int id)
    	{
    		// Find the matching order
    		Order order = ...;
    		...

    		var hashedOrder = order.GetHashCode();
    		string hashedOrderEtag = String.Format("\"{0}\"", hashedOrder);
    		var eTag = new EntityTagHeaderValue(hashedOrderEtag);

    		// Return a response message containing the order and the cache control header
    		OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
    		{
    			...,
    			ETag = eTag
    		};
    		return response;
    	}
    	...
    }
	```

	Web API 发布的响应消息如下所示：

	```HTTP
	HTTP/1.1 200 OK
	...
	Cache-Control: max-age=600, private
	Content-Type: text/json; charset=utf-8
	ETag: "2147483648"
	Content-Length: ...
	{"OrderID":2,"ProductID":4,"Quantity":2,"OrderValue":10.00}
	```

	> [AZURE.TIP]出于安全原因，不允许缓存敏感数据或通过经过身份验证 (HTTPS) 的连接返回的数据。

	客户端应用程序随时可以发出后续 GET 请求以检索同一资源，并且如果资源已更改（它具有不同的 ETag），则应放弃缓存的版本，并将新版本添加到缓存中。如果资源很大并且需要大量的带宽才能传输回客户端，则重复执行提取相同数据的请求可能会效率低下。为了应对这种情况，HTTP 协议定义了以下过程来优化应在 Web API 中支持的 GET 请求：

	- 客户端构造 GET 请求，该请求包含 If-None-Match HTTP 标头中引用的资源的当前缓存版本的 ETag：

	```HTTP
	GET http://adventure-works.com/orders/2 HTTP/1.1
	If-None-Match: "2147483648"
	...
	```

	- Web API 中的 GET 操作获取所请求数据的当前 ETag（上面示例中的订单 2），并将其与 If-None-Match 标头中的值进行比较。

	- 如果所请求数据的当前 ETag 与请求提供的 ETag 匹配，则资源尚未更改，Web API 应返回 HTTP 响应，其中包含空的消息正文和状态代码 304（未修改）。

	- 如果所请求数据的当前 ETag 与请求提供的 ETag 不匹配，则数据已更改，Web API 应返回 HTTP 响应，其中包含带有新数据的消息正文和状态代码 200（正常）。

	- 如果所请求的数据不再存在，则 Web API 应返回状态代码为 404（未找到）的 HTTP 响应。

	- 客户端使用状态代码来维护缓存。如果数据尚未更改（状态代码 304），则对象可保持缓存，并且客户端应用程序应继续使用此版本的对象。如果数据已更改（状态代码 200），则应放弃缓存的对象，并插入新对象。如果数据不再可用（状态代码 404），则应从缓存中删除该对象。

	> [AZURE.NOTE]如果响应标头包含 Cache-Control 标头 no-store，则应始终从缓存中删除对象，而不考虑 HTTP 状态代码。

	下面的代码显示了已扩展为支持 If-None-Match 标头的 `FindOrderByID` 方法。请注意，如果省略 If-None-Match 标头，则始终检索指定的订单：

	```C#
	public class OrdersController : ApiController
	{
   		...
    	[Route("api/orders/{id:int:min(0)}")]
    	[HttpGet]
    	public IHttpActionResult FindOrderById(int id)
        {
            try
            {
                // Find the matching order
    		    Order order = ...;

                // If there is no such order then return NotFound
                if (order == null)
                {
                    return NotFound();
                }

                // Generate the ETag for the order
                var hashedOrder = order.GetHashCode();
                string hashedOrderEtag = String.Format("\"{0}\"", hashedOrder);

                // Create the Cache-Control and ETag headers for the response
                IHttpActionResult response = null;
                var cacheControlHeader = new CacheControlHeaderValue();
                cacheControlHeader.Public = true;
                cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
                var eTag = new EntityTagHeaderValue(hashedOrderEtag);

                // Retrieve the If-None-Match header from the request (if it exists)
                var nonMatchEtags = Request.Headers.IfNoneMatch;

                // If there is an ETag in the If-None-Match header and
                // this ETag matches that of the order just retrieved,
                // then create a Not Modified response message
                if (nonMatchEtags.Count > 0 &&
                    String.Compare(nonMatchEtags.First().Tag, hashedOrderEtag) == 0)
                {
                    response = new EmptyResultWithCaching()
                    {
                        StatusCode = HttpStatusCode.NotModified,
                        CacheControlHeader = cacheControlHeader,
                        ETag = eTag
                    };
                }
                // Otherwise create a response message that contains the order details
                else
                {
                    response = new OkResultWithCaching<Order>(order, this)
                    {
                        CacheControlHeader = cacheControlHeader,
                        ETag = eTag
                    };
                }

                return response;
            }
            catch
            {
                return InternalServerError();
            }
        }
    ...
    }
	```

	此示例引入了名为 `EmptyResultWithCaching` 的附加自定义 `IHttpActionResult` 类。此类只充当不包含响应正文的 `HttpResponseMessage` 对象的包装：

	```C#
    public class EmptyResultWithCaching : IHttpActionResult
    {
        public CacheControlHeaderValue CacheControlHeader { get; set; }
        public EntityTagHeaderValue ETag { get; set; }
        public HttpStatusCode StatusCode { get; set; }
		public Uri Location { get; set; }

        public async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
        {
            HttpResponseMessage response = new HttpResponseMessage(StatusCode);
            response.Headers.CacheControl = this.CacheControlHeader;
            response.Headers.ETag = this.ETag;
            response.Headers.Location = this.Location;
            return response;
        }
    }
	```

	> [AZURE.TIP]在此示例中，通过对从基础数据源检索到的数据进行哈希处理生成数据的 ETag。如果 ETag 可以某种其他方式计算，则可以进一步优化此过程，并且仅在数据已更改时，才需要从数据源中提取数据。如果数据很大或访问数据源可能导致显著延迟（例如，如果数据源是远程数据库），则此方法特别有用。

- **使用 Etag 支持乐观并发**。

	为了启用对以前缓存的数据更新，HTTP 协议支持乐观并发策略。如果在提取和缓存资源后，客户端应用程序随后发送 PUT 或 DELETE 请求以更改或删除资源，则应包含引用 ETag 的 If-match 标头。然后，Web API 可以使用此信息来确定该资源在检索到后是否已被其他用户更改，并将相应响应发送回客户端应用程序，如下所示：

	- 客户端构造 PUT 请求，该请求包含资源的新详细信息，以及 If-Match HTTP 标头中引用的资源的当前缓存版本的 ETag。下面的示例演示了用于更新订单的 PUT 请求：

	```HTTP
	PUT http://adventure-works.com/orders/1 HTTP/1.1
	If-None-Match: "2282343857"
	Content-Type: application/x-www-form-urlencoded
	...
	Date: Fri, 12 Sep 2014 09:18:37 GMT
	Content-Length: ...
	ProductID=3&Quantity=5&OrderValue=250
	```

	- Web API 中的 PUT 操作获取所请求数据的当前 ETag（上面示例中的订单 1），并将其与 If-Match 标头中的值进行比较。

	- 如果所请求数据的当前 ETag 与请求提供的 ETag 匹配，则资源尚未未更改并且 Web API 应执行更新，并在成功的情况下返回包含 HTTP 状态代码 204（无内容）的消息。响应可以包括资源的更新后版本的 Cache-Control 和 ETag 标头。响应中应始终包含引用新更新资源的 URI 的 Location 标头。

	- 如果所请求数据的当前 ETag 与请求提供的 ETag 不匹配，则数据在提取后已被其他用户更改，Web API 应返回包含空的消息正文和状态代码 412（不满足前提条件）的 HTTP 响应。

	- 如果要更新的资源不再存在，则 Web API 应返回状态代码为 404（未找到）的 HTTP 响应。

	- 客户端使用状态代码和响应标头来维护缓存。如果数据已更新（状态代码 204），则对象可保持缓存（只要 Cache-Control 标头未指定 no-store），但应更新 ETag。如果数据已被其他用户更改（状态代码 412）或未找到（状态代码 404），则应放弃缓存的对象。

	下一个代码示例显示了 Orders 控制器的 PUT 操作的实现：

	```C#
	public class OrdersController : ApiController
	{
   		...
    	[HttpPut]
    	[Route("api/orders/{id:int}")]
    	        public IHttpActionResult UpdateExistingOrder(int id, DTOOrder order)
        {
            try
            {
                var baseUri = Constants.GetUriFromConfig();
                var orderToUpdate = this.ordersRepository.GetOrder(id);
                if (orderToUpdate == null)
                {
                    return NotFound();
                }

                var hashedOrder = orderToUpdate.GetHashCode();
                string hashedOrderEtag = String.Format("\"{0}\"", hashedOrder);

                // Retrieve the If-Match header from the request (if it exists)
                var matchEtags = Request.Headers.IfMatch;

                // If there is an Etag in the If-Match header and
                // this etag matches that of the order just retrieved,
                // or if there is no etag, then update the Order
                if (((matchEtags.Count > 0 &&
                     String.Compare(matchEtags.First().Tag, hashedOrderEtag) == 0)) ||
                     matchEtags.Count == 0)
                {
                    // Modify the order
                    orderToUpdate.OrderValue = order.OrderValue;
                    orderToUpdate.ProductID = order.ProductID;
                    orderToUpdate.Quantity = order.Quantity;

                    // Save the order back to the data store
                    // ...

                    // Create the No Content response with Cache-Control, ETag, and Location headers
                    var cacheControlHeader = new CacheControlHeaderValue();
                    cacheControlHeader.Private = true;
                    cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);

                    hashedOrder = order.GetHashCode();
                    hashedOrderEtag = String.Format("\"{0}\"", hashedOrder);
                    var eTag = new EntityTagHeaderValue(hashedOrderEtag);

                    var location = new Uri(string.Format("{0}/{1}/{2}", baseUri, Constants.ORDERS, id));
                    var response = new EmptyResultWithCaching()
                    {
                        StatusCode = HttpStatusCode.NoContent,
                        CacheControlHeader = cacheControlHeader,
                        ETag = eTag,
                        Location = location
                    };

                    return response;
                }

                // Otherwise return a Precondition Failed response
                return StatusCode(HttpStatusCode.PreconditionFailed);
            }
            catch
            {
                return InternalServerError();
            }
        }
        ...
    }
	```

	> [AZURE.TIP]是否使用 If-match 标头完全是可选的，如果省略它，Web API 将始终尝试更新指定的订单，可能会盲目地覆盖其他用户所做的更新。若要避免由于丢失更新出现的问题，请始终提供 If-Match 标头。

<a name="considerations-for-handling-large"></a>
## 有关处理大型请求和响应的注意事项

可能会有这样的情况：客户端应用程序需要发出用于发送或接收大小可能为几兆字节（或更大）的数据的请求。等待传输如此大量的数据可能会导致客户端应用程序停止响应。在需要处理包含大量数据的请求时，请考虑以下几点：

- **优化涉及大型对象的请求和响应**。

	一些资源可能是大型对象或包含大量字段，例如图形图像或其他类型的二进制数据。Web API 应支持流式处理以实现优化这些资源的上载和下载。

	HTTP 协议提供了分块传输编码机制，用于将大型数据对象流式传输回客户端。当客户端为大型对象发送 HTTP GET 请求时，Web API 可以通过 HTTP 连接在段落_区块_中发送回答复。最初答复中数据的长度可能未知（可能会生成它），因此托管 Web API 的服务器应发送包含每个区块的响应消息，这些区块指定 Transfer-Encoding: Chunked 标头而不是 Content-Length 标头。客户端应用程序可以依次接收每个区块以组合成完整响应。在数据传输完成后，服务器将发送回最后一个区块（大小为零）。可以使用 `PushStreamContent` 类在 ASP.NET Web API 中实现分块。

	下面的示例演示用于响应针对产品图像的 HTTP GET 请求的操作：

	```C#
	public class ProductImagesController : ApiController
	{
    	...
    	[HttpGet]
        [Route("productimages/{id:int}")]
        public IHttpActionResult Get(int id)
        {
            try
            {
                var container = ConnectToBlobContainer(Constants.PRODUCTIMAGESCONTAINERNAME);

                if (!BlobExists(container, string.Format("image{0}.jpg", id)))
                {
                    return NotFound();
                }
                else
                {
                    return new FileDownloadResult()
                    {
                        Container = container,
                        ImageId = id
                    };
                }
            }
            catch
            {
                return InternalServerError();
            }
        }
    	...
	}
	```

	在此示例中，`ConnectBlobToContainer` 是用于连接到 Azure Blob 存储中指定的容器（未显示名称）的帮助器方法。`BlobExists` 是另一个帮助器方法，它返回一个布尔值，以指示 blob 存储容器中是否存在具有指定名称的 blob。

	每个产品都将其自己的图像保存在 blob 存储中。`FileDownloadResult` 类是一个自定义 `IHttpActionResult` 类，它使用 `PushStreamContent` 对象从相应的 blob 中读取图像数据，并将以异步方式传输这些数据作为响应消息的内容：

	```C#
	public class FileDownloadResult : IHttpActionResult
    {
        public CloudBlobContainer Container { get; set; }
        public int ImageId { get; set; }

        public async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
        {
            var response = new HttpResponseMessage();
            response.Content = new PushStreamContent(async (outputStream, _, __) =>
            {
                try
                {
                    CloudBlockBlob blockBlob = Container.GetBlockBlobReference(String.Format("image{0}.jpg", ImageId));
                    await blockBlob.DownloadToStreamAsync(outputStream);
                }
                finally
                {
                    outputStream.Close();
                }
            });

            response.StatusCode = HttpStatusCode.OK;
            response.Content.Headers.ContentType = new MediaTypeHeaderValue("image/jpeg");
            return response;
        }
    }
	```

	如果客户端需要 POST 包含大型对象的新资源，你还可以对上载操作应用流式处理。下一个示例演示了 `ProductImages` 控制器的 Post 方法。此方法允许客户端上载新的产品图像：

	```C#
	public class ProductImagesController : ApiController
	{
    	...
        [HttpPost]
        [Route("productimages")]
        public async Task<IHttpActionResult> Post()
        {
            try
            {
                if (!Request.Content.Headers.ContentType.MediaType.Equals("image/jpeg"))
                {
                    return StatusCode(HttpStatusCode.UnsupportedMediaType);
                }
                else
                {
                    var id = new Random().Next(); // Use a random int as the key for the new resource. Should probably check that this key has not already been used
                    var container = ConnectToBlobContainer(Constants.PRODUCTIMAGESCONTAINERNAME);
                    return new FileUploadResult()
                    {
                        Container = container,
                        ImageId = id,
                        Request = Request
                    };
                }
            }
            catch
            {
                return InternalServerError();
            }
        }
    	...
	}
	```

	此代码使用名为 `FileUploadResult` 的另一个自定义 `IHttpActionResult` 类。此类包含以异步方式上载数据的逻辑：

	```C#
    public class FileUploadResult : IHttpActionResult
    {
        public CloudBlobContainer Container { get; set; }
        public int ImageId { get; set; }
        public HttpRequestMessage Request { get; set; }

        public async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
        {
            var response = new HttpResponseMessage();
            CloudBlockBlob blockBlob = Container.GetBlockBlobReference(String.Format("image{0}.jpg", ImageId));
            await blockBlob.UploadFromStreamAsync(await Request.Content.ReadAsStreamAsync());
            var baseUri = string.Format("{0}://{1}:{2}", Request.RequestUri.Scheme, Request.RequestUri.Host, Request.RequestUri.Port);
            response.Headers.Location = new Uri(string.Format("{0}/productimages/{1}", baseUri, ImageId));
            response.StatusCode = HttpStatusCode.OK;
            return response;
        }
    }
	```

	> [AZURE.TIP]可以上载到 Web 服务的数据量不受流式处理约束，并且单个请求可以想象生成使用大量资源的大型对象。如果在流式处理过程中，Web API 确定请求中的数据量超过一些可接受的范围，它可以中止操作并返回具有状态代码 413（请求实体太大）的响应消息。

	可以使用 HTTP 压缩将通过网络传输的大型对象的大小降到最低。此方法可帮助减少网络流量和关联的网络延迟，但代价是需要在客户端和托管 Web API 的服务器上进行额外的处理。例如，需要接收压缩数据的客户端应用程序可以提供 Accept-Encoding: gzip 请求标头（还可以指定其他数据压缩算法）。如果服务器支持压缩，则应以消息正文中以 gzip 格式存储的内容以及 Content-Encoding: gzip 响应标头进行响应。

	> [AZURE.TIP]你可以将编码压缩和流式处理结合使用；在流式处理数据之前先压缩它，并在消息标头中指定 gzip 内容编码和 chunked 传输编码。另请注意，某些 Web 服务器（如 Internet Information Server）可以配置为自动压缩 HTTP 响应，而不管 Web API 是否压缩数据。

- **为不支持异步操作的客户端实现部分响应**。

	作为异步流式处理的替代方法，客户端应用程序可以区块方式显式请求大型对象的数据，称为部分响应。客户端应用程序发送 HTTP HEAD 请求以获取有关对象的信息。如果 Web API 支持部分响应，则应以包含 Accept-Ranges 标头和 Content-Length 标头（指示该对象的总大小），但消息正文应为空的响应消息响应 HEAD 请求。客户端应用程序可以使用此信息来构造一系列指定要接收的字节范围的 GET 请求。Web API 应返回包含以下内容的响应消息：HTTP 状态 206（部分内容）、指定响应消息正文中包含的实际数据量的 Content-Length 标头，以及指示此数据表示对象的哪一部分（例如 4000 到 8000 个字节）的 Content-Range 标头。

	HTTP HEAD 请求和部分响应将在 API 设计指南文档中详细介绍。

- **避免在客户端应用程序中发送不必要的“继续”状态消息**。

	将要发送大量数据到服务器的客户端应用程序可能会先确定服务器是否实际可以接受该请求。在发送数据之前，客户端应用程序可以提交一个 HTTP 请求，其中包含 Expect: 100-Continue 标头、Content-Length 标头（指示数据的大小），但消息正文为空。如果服务器可以处理该请求，则应以指定 HTTP 状态 100（继续）的消息进行响应。然后，客户端应用程序可以继续操作并发送在消息正文中包含数据的完整请求。

	如果你使用 IIS 来托管服务，则 HTTP.sys 驱动程序会自动检测并处理 Expect: 100-Continue 标头，然后再将请求传递到你的 Web 应用程序。这意味着你很可能在应用程序代码中看不到这些标头，你可以假设 IIS 已筛选掉任何它认为不适合或太大的消息。

	如果你使用.NET Framework 生成客户端应用程序，则默认情况下所有 POST 和 PUT 消息都将先发送包含 Expect: 100-Continue 标头的消息。与服务器端一样，由 .NET Framework 透明地处理该过程。但是，此过程的结果是，每个 POST 和 PUT 请求会导致对服务器进行 2 次往返，即使是小请求，也是如此。如果你的应用程序不发送包含大量数据的请求，则可以通过在客户端应用程序中使用 `ServicePointManager` 类创建 `ServicePoint` 对象来禁用此功能。`ServicePoint` 对象将基于标识服务器上的资源的 URI 的方案和主机片段处理客户端建立的与服务器的连接。然后，你可以将 `ServicePoint` 对象的 `Expect100Continue` 属性设置为 false。客户端通过与 `ServicePoint` 对象的方案和主机片段匹配的 URI 发出的所有后续 POST 和 PUT 请求在发送时将不包含 Expect: 100-Continue 标头。下面的代码演示如何配置 `ServicePoint` 对象，以便将所有请求都发送到方案为 `http` 且主机为 `www.contoso.com` 的 URI。

	```C#
	Uri uri = new Uri("http://www.contoso.com/");
	ServicePoint sp = ServicePointManager.FindServicePoint(uri);
	sp.Expect100Continue = false;
	```

	你还可以设置 `ServicePointManager` 类的静态 `Expect100Continue` 属性，以便为所有后续创建的 `ServicePoint` 对象指定此属性的默认值。有关详细信息，请参阅 Microsoft 网站上的 [ServicePoint 类](https://msdn.microsoft.com/zh-cn/library/system.net.servicepoint.aspx)页。

- **对可能返回大量对象的请求支持分页**。

	如果集合包含大量资源，则向相应的 URI 发出 GET 请求可能会导致在托管 Web API 的服务器上进行大量处理而影响性能，并生成大量网络流量导致延迟时间增加。

	若要处理这些情况，Web API 应支持这样的查询字符串：使客户端应用程序可以细化请求或在更易于管理的离散块（或页）中提取数据。ASP.NET Web API 框架分析查询字符串并将它们拆分为一系列参数/值对，然后按照前面所述的路由规则将这些参数/值对传递给相应的方法。应实现该方法，以使用查询字符串中指定的相同名称接受这些参数。此外，这些参数应是可选的（以防客户端忽略请求中的查询字符串）并具有有意义的默认值。下面的代码显示 `Orders` 控制器中的 `GetAllOrders` 方法。此方法检索订单的详细信息。如果此方法不受约束，可以想像它可能返回大量数据。`limit` 和 `offset` 参数旨在将数据量减少到较小子集，在本示例中默认情况下为前 10 个订单：

	```C#
	public class OrdersController : ApiController
	{
    	...
    	[Route("api/orders")]
    	[HttpGet]
    	public IEnumerable<Order> GetAllOrders(int limit=10, int offset=0)
    	{
    	    // Find the number of orders specified by the limit parameter
    	    // starting with the order specified by the offset parameter
    	    var orders = ...
    	    return orders;
    	}
    	...
	}
	```

	客户端应用程序可以使用 URI *http://www.adventure-works.com/api/orders?limit=30&offset=50* 发出请求以检索从偏移量 50 开始的 30 个订单。

	> [AZURE.TIP]请避免让客户端应用程序指定的查询字符串导致 URI 超过 2000 个字符。许多 Web 客户端和服务器无法处理这么长的 URI。

<a name="considerations-for-maintaining-responsiveness"></a>
## 有关保持响应能力、可伸缩性和可用性的注意事项

同一 Web API 可能由世界任何地方运行的许多客户端应用程序利用。请务必确保将 Web API 实现为在重负载下保持响应能力、可扩展以支持高度变化的工作负荷，并保证执行关键业务操作的客户端的可用性。确定如何满足这些要求时，请考虑以下几点：

- **为长时间运行的请求提供异步支持**。

	可能需要很长的时间来处理的请求应在执行时确保不会阻塞提交请求的客户端。Web API 可以执行一些初始检查来验证请求，启动单独的任务来执行工作，然后返回包含 HTTP 代码 202（已接受）的响应消息。此任务可以在 Web API 处理期间异步运行，也可以卸载到 Azure Web 作业（如果 Web API 由 Azure 网站托管）或辅助角色（如果将 Web API 作为 Azure 云服务实现）。

	> [AZURE.NOTE]有关在 Azure 网站上使用 Web 作业的详细信息，请访问 Microsoft 网站上的[使用 Web 作业在 Microsoft Azure 网站中运行后台任务](/documentation/articles/web-sites-create-web-jobs)页。

	Web API 还应提供一种向客户端应用程序返回处理结果的机制。可以通过以下两种方案实现此目的：为客户端应用程序提供轮询机制以定期查询处理是否已完成并获取结果，或者使 Web API 可以在操作完成时发送通知。

	可以通过使用以下方法提供作为虚拟资源的_轮询_ URI 来实现简单的轮询机制：

	1. 客户端应用程序将初始请求发送到 Web API。

	2. Web API 将有关该请求的信息存储在表存储或 Microsoft Azure 缓存中保存的表中，并可能以 GUID 形式为此条目生成唯一键。

	3. Web API 启动处理作为单独的任务。Web API 在表中将该任务的状态记录为_“正在运行”_。

	4. Web API 返回包含 HTTP 状态代码 202（已接受）的响应消息，消息正文中包含该表条目的 GUID。

	5. 完成该任务后，Web API 将结果存储在表中，并将该任务的状态设置为_“完成”_。请注意，如果该任务失败，Web API 还可能存储有关失败的信息并将状态设置为_“失败”_。

	6. 该任务运行时，客户端可以继续执行其自己的处理。它可以定期将请求发送到 URI _/polling/{guid}_，其中 _{guid}_ 是 Web API 在 202 响应消息中返回的 GUID。

	7. _/polling{guid}_ URI 中的 Web API 在表中查询相应任务的状态并返回包含 HTTP 状态代码 200（正常）的响应消息，该代码包含以下状态（_正在运行_、_完成_或_失败_）。如果该任务已完成或失败，则响应消息还可以包括处理的结果或获得的有关失败原因的任何信息。

	如果你想要实现通知，可用选项包括：

	- 使用 Azure 通知中心将异步响应推送到客户端应用程序。Microsoft 网站上的 [Azure 通知中心通知用户](/documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-notify-users)页提供了进一步的详细信息。

	- 使用 Comet 模型来保持客户端与托管 Web API 的服务器之间的永久网络连接，并使用此连接将消息从服务器推送回客户端。MSDN 杂志文章[在 Microsoft .NET Framework 中构建简单的 Comet 应用程序](https://msdn.microsoft.com/magazine/jj891053.aspx)介绍了一个示例解决方案。

	- 使用 SignalR 通过永久网络连接将 Web 服务器中的实时数据推送到客户端。SignalR 可作为 NuGet 程序包用于 ASP.NET Web 应用程序。可以在 [ASP.NET SignalR](http://signalr.net/) 网站上找到详细信息。

	> [AZURE.NOTE]Comet 和 SignalR 均利用 Web 服务器和客户端应用程序之间的永久网络连接。这会影响可伸缩性，因为大量客户端可能需要同样多的并发连接数。

- **确保每个请求都是无状态的**。

	每个请求应视为原子的。客户端应用程序发出的一个请求应与同一客户端提交的任何后续请求之间没有任何依赖关系。此方法可帮助你实现可伸缩性；可以在多个服务器上部署 Web 服务的实例。客户端请求可以定向到其中任一实例，并且结果应始终相同。由于类似的原因，它还提高了可用性；如果某个 Web 服务器失败，可以将请求路由到其他实例（通过使用 Azure 流量管理器），同时重新启动该服务器，对客户端应用程序没有任何不良影响。

- **跟踪客户端并实施限制以减少 DOS 攻击的可能性**。

	如果特定客户端在给定的时间段内发出了大量请求，则它可能会独占服务并影响其他客户端的性能。若要缓解此问题，Web API 可以通过跟踪所有传入请求的 IP 地址或通过记录每次经过身份验证的访问来监视客户端应用程序的调用。你可以使用此信息来限制资源访问。如果客户端超出定义的限制，Web API 可以返回包含状态 503（服务不可用）的响应消息并在其中包括 Retry-After 标头以指定客户端可以发送下一个请求而不会被拒绝的时间。此策略可帮助降低停止系统的一组客户端发起拒绝服务 (DOS) 攻击的可能性。

- **小心管理永久 HTTP 连接**。

	HTTP 协议支持永久 HTTP 连接（在这些连接可用时）。HTTP 1.0 规范已添加 Connection:Keep-Alive 标头以允许客户端应用程序向服务器表明它可能使用同一连接发送后续请求而不是打开新连接。如果客户端在主机定义的时间段内未重用连接，则该连接将自动关闭。此行为在 Azure 服务使用的 HTTP 1.1 中是默认设置，因此无需在消息中包括 Keep-Alive 标头。

	让连接保持打开状态可以减少延迟和网络拥塞，从而有助于提高响应能力，但让不必要的连接保持打开状态的时间长于所需时间可能会不利于可扩展性，从而限制了其他并发客户端进行连接的能力。如果客户端应用程序运行在移动设备上，则它还会影响电池使用寿命；如果应用程序只偶尔向服务器发出请求，则维护打开的连接可能会导致电池更快耗尽。若要确保不使用 HTTP 1.1 建立永久连接，客户端可以在消息中包括 Connection:Close 标头以覆盖默认行为。同样，如果服务器正在处理大量客户端，它可以在响应消息中包括 Connection:Close 标头，这将关闭连接并节省服务器资源。

	> [AZURE.NOTE]永久 HTTP 连接是纯粹的可选功能，用于减少与反复建立通信通道关联的网络开销。Web API 和客户端应用程序都不应依赖于可用的永久 HTTP 连接。不要使用永久 HTTP 连接实现 Comet 样式通知系统，而是应利用 TCP 层的套接字（或 websocket，如果可用）。最后，请注意：如果客户端应用程序通过代理与服务器通信，则 Keep-Alive 标头的作用有限；只有与客户端和代理的连接将是持久的。

## 有关发布和管理 Web API 的注意事项

要使 Web API 可供客户端应用程序使用，Web API 必须部署到主机环境中。此环境通常是 Web 服务器，尽管它可能是某种其他类型的主机进程。发布 Web API 时，应考虑以下几点：

- 所有请求都必须经过身份验证和授权，必须强制实施相应的访问控制级别。
- 商业 Web API 可能会受到与响应时间有关的各种质量保证约束。如果负载随着时间的推移会发生显著变化，请务必确保主机环境是可缩放的。
- 出于盈利目的，可能有必要计量请求。
- 可能需要使用 Web API 调控通信流量，并对已用完其配额的特定客户端实施限制。
- 法规要求可能会强制执行所有请求和响应的记录和审核。
- 为了确保可用性，可能有必要监视托管 Web API 的服务器的运行状况并在必要时重新启动它。

如果能够将这些问题从有关 Web API 的实现的技术问题中分离出来，会很有用。为此，请考虑创建 [façade](http://en.wikipedia.org/wiki/Facade_pattern)，它作为单独的进程运行，并将请求路由到 Web API。façade 可以提供管理操作并将已验证的请求转发到 Web API。使用 façade 还可以带来很多功能优势，包括：

- 充当多个 Web API 的集成点。
- 转换消息并转换使用不同技术生成的客户端的通信协议。
- 缓存请求和响应以减少托管 Web API 的服务器上的负载。

## 有关测试 Web API 的注意事项
Web API 应和软件的任何其他部分一样进行全面测试。你应考虑创建单元测试来验证每个操作的功能，就像对任何其他类型的应用程序所做的那样。有关详细信息，请参阅 Microsoft 网站上的[使用单元测试验证代码](https://msdn.microsoft.com/zh-cn/library/dd264975.aspx)页。

> [AZURE.NOTE]本指南提供的示例 Web API 包括一个演示如何对所选操作执行单元测试的测试项目。

Web API 的性质带来了其自己的验证是否正常运行的附加要求。你应特别注意以下几个方面：

- 测试所有路由以验证它们是否调用正确的操作。特别要注意意外返回的 HTTP 状态代码 405（不允许的方法），因为这可能指示路由与可分派给该路由的 HTTP 方法（GET、POST、PUT、DELETE）不匹配。

	将 HTTP 请求发送到不支持这些请求的路由，例如，将 POST 请求提交到特定资源（POST 请求只应发送到资源集合）。在这些情况下，唯一有效的响应_应_为状态代码 405（不允许）。

- 验证所有路由是否都得到正确保护并受相应身份验证和授权检查的制约。

	> [AZURE.NOTE]安全性的某些方面（如用户身份验证）最有可能是主机环境（而不是 Web API）的职责，但仍有必要在部署过程中进行安全测试。

- 测试每个操作执行的异常处理，并验证是否将相应的有意义的 HTTP 响应传递回客户端应用程序。
- 验证请求和响应消息的格式是否正确。例如，如果 HTTP POST 请求包含 x-www-form-urlencoded 格式的新资源数据，请确认相应的操作正确分析数据、创建该资源，并返回包含新资源的详细信息的响应，包括正确的 Location 标头。
- 验证响应消息中的所有链接和 URI。例如，HTTP POST 消息应返回新创建的资源的 URI。所有 HATEOAS 链接都应有效。

	> [AZURE.IMPORTANT]如果通过 API 管理服务发布 Web API，则这些 URI 应反映管理服务的 URL，而不是托管 Web API 的 Web 服务器的 URL。

- 确保每个操作针对不同输入组合返回正确的状态代码。例如：
	- 如果查询成功，则应返回状态代码 200（正常）
	- 如果未找到资源，则操作应返回 HTTP 状态代码 404（未找到）。
	- 如果客户端发送的请求成功删除资源，则状态代码应为 204（无内容）。
	- 如果客户端发送的请求创建了新资源，则状态代码应为 201（已创建）

密切注意 5xx 范围内的异常响应状态代码。这些消息通常由主机服务器报告，以指示无法完成有效请求。

- 测试客户端应用程序可以指定的不同请求标头组合并确保 Web API 在响应消息中返回预期的信息。

- 测试查询字符串。如果操作可以接受可选参数（例如分页请求），则测试参数的不同组合和顺序。

- 验证异步操作是否成功完成。如果 Web API 支持对返回大型二进制对象（如视频或音频）的请求进行流式处理，请确保在流式传输数据时不会阻止客户端请求。如果 Web API 实现了轮询长时间运行的数据修改操作，请验证这些操作在执行时正确报告其状态。

你还应创建并运行性能测试以检查 Web API 在压力下令人满意地运行。你可以使用 Visual Studio Ultimate 构建一个 Web 性能和负载测试项目。有关详细信息，请参阅 Microsoft 网站上的[在发布前对应用程序运行性能测试](https://msdn.microsoft.com/zh-cn/library/dn250793.aspx)页。

<!--## 通过使用 Azure API 管理服务发布和管理 Web API

Azure 提供了 [API 管理服务](http://azure.microsoft.com/documentation/services/api-management/)，你可以使用它来发布和管理 Web API。使用此工具，你可以生成一个充当一个或多个 Web API 的外观的服务。该服务本身是一个可缩放的 Web 服务，你可以使用 Azure 管理门户创建和配置它。可以使用此服务发布和管理 Web API，如下所示：

1. 将 Web API 部署到网站、Azure 云服务或 Azure 虚拟机。

2. 将 API 管理服务连接到 Web API。发送到管理 API 的 URL 的请求将映射到 Web API 中的 URI。同一 API 管理服务可以将请求路由到多个 Web API。这使你可以将多个 Web API 聚合为单个管理服务。同样，如果你需要限制或分隔可用于不同应用程序的功能，则可以从多个 API 管理服务引用同一 Web API。

	> [AZURE.NOTE]作为 HTTP GET 请求的响应的一部分生成的 HATEOAS 链接中的 URI 应引用 API 管理服务（而不是托管 Web API 的 Web 服务器）的 URL。

3. 对于每个 Web API，指定该 Web API 公开的 HTTP 操作以及操作可以获取为输入的任何可选参数。还可以配置 API 管理服务是否应缓存从 Web API 接收的响应以优化对相同数据的重复请求。记录每个操作可以生成的 HTTP 响应的详细信息。此信息用于为开发人员生成文档，因此它应准确且完整。

	你可以使用 Azure 管理门户提供的向导手动定义操作，也可以从包含 WADL 或 Swagger 格式的定义的文件中导入操作。

4. 为 API 管理服务与托管 Web API 的 Web 服务器之间的通信配置安全设置。API 管理服务目前支持使用证书和 OAuth 2.0 用户授权的基本身份验证和相互身份验证。

5. 创建产品。产品是发布的单元；可将先前连接到管理服务的 Web API 添加到产品。发布产品后，该 Web API 便可供开发人员使用了。

	> [AZURE.NOTE]在发布产品之前，还可以定义可以访问该产品的用户组，并将用户添加到这些组。这让你可以控制可以使用该 Web API 的开发人员和应用程序。如果 Web API 需要批准，则在能够访问它之前，开发人员必须向产品管理员发送请求。管理员可以授予或拒绝开发人员的访问权限。如果情况发生变化，也可以阻止现有开发人员。

6.	为每个 Web API 配置策略。策略可以控制以下方面：是否应允许跨域调用、如何对客户端进行身份验证、是否要在 XML 和 JSON 数据格式之间透明地进行转换、是否要限制从给定 IP 范围发起的调用、使用配额，以及是否要限制调用率等。策略可以对整个产品全局应用、对产品中的单个 Web API 应用，或者对 Web API 中的单个操作应用。

你可以在 Microsoft 网站上的 [API 管理](http://azure.microsoft.com/services/api-management/)页中找到描述如何执行这些任务的完整详细信息。Azure API 管理服务还提供其自己的 REST 接口，使你可以构建自定义界面来简化配置 Web API 的过程。有关详细信息，请访问 Microsoft 网站上的 [Azure API 管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn776326.aspx)页。

> [AZURE.TIP]Azure 提供了使用 Azure 流量管理器，使用它可以实现故障转移和负载平衡，并可以减少在不同地理位置托管的多个网站实例之间的延迟。可以将 Azure 流量管理器与 API 管理服务结合使用；API 管理服务可以通过 Azure 流量管理器将请求路由到网站实例。有关详细信息，请访问 Microsoft 网站上的[关于流量管理器负载平衡方法](https://msdn.microsoft.com/zh-cn/library/azure/dn339010.aspx)页。

> 在此结构中，如果你要对网站使用自定义 DNS 名称，则应将每个网站的相应 CNAME 记录配置为指向 Azure 流量管理器网站的 DNS 名称。-->

## 为构建客户端应用程序的开发人员提供支持
构造客户端应用程序的开发人员通常需要了解有关如何访问 Web API 和与参数、数据类型、返回类型和返回代码（描述 Web 服务和客户端应用程序之间的不同请求和响应）相关的文档的信息。

### 记录 Web API 的 REST 操作
Azure API 管理服务包括一个开发人员门户，其中描述了由 Web API 公开的 REST 操作。产品发布后，便会显示在此门户上。开发人员可以使用此门户注册访问；然后，管理员可以批准或拒绝该请求。如果开发人员获得批准，将会为他们分配订阅密钥，用于对来自他们开发的客户端应用程序的调用进行身份验证。此密钥必须与每个 Web API 调用一起提供，否则将会被拒绝。

此门户还提供了：

- 产品的文档，列出它公开的操作、所需参数和可以返回的不同响应。请注意，此信息从[通过使用 Windows Azure API 管理服务发布 Web API](#publishing-a-web-API) 部分的列表中的步骤 3 中提供的详细信息生成。

- 演示如何通过多种语言（包括 JavaScript、C#、Java、Ruby、Python 和 PHP）调用操作的代码片段。

- 开发人员使用开发人员控制台，能够发送 HTTP 请求以测试产品中的每个操作并查看结果。

- 在此页中开发人员可以报告发现的任何问题。

使用 Azure 管理门户可以自定义开发人员门户，以便更改样式和布局以匹配你组织的品牌。

### 实现客户端 SDK
构建调用 REST 请求以访问 Web API 的客户端应用程序需要编写大量代码来构造每个请求并相应地设置其格式，将请求发送到托管 Web 服务的服务器，分析响应以确定请求是成功还是失败并提取返回的任何数据。要使客户端应用程序免除这些问题，可以提供这样一个 SDK：包装 REST 接口并在一组功能更强的方法内抽象这些低级别的详细信息。客户端应用程序使用这些方法，这些方法透明地将调用转换为 REST 请求，然后将响应转换回方法返回值。这是许多服务（包括 Azure SDK）已实现的一种常用技术。

创建客户端 SDK 是一项要求相当高的任务，因为它必须一致地实现，并经过严格测试。但是，此过程的大部分操作可以机械地进行，并且许多供应商提供了可自动执行上述许多任务的工具。

## 监视 Web API

根据你发布和部署 Web API 的方式，可以直接监视 Web API，也可以通过分析通过 API 管理服务传递的流量收集使用情况和运行状况信息。

### 直接监视 Web API
如果已通过使用 ASP.NET Web API 模板（不管作为 Web API 项目还是作为 Azure 云服务中的 Web 角色）和 Visual Studio 2013 实现你的 Web API，则可以通过使用 ASP.NET Application Insights 收集可用性、性能和使用情况数据。Application Insights 是在 Web API 部署到云中后透明地跟踪和记录有关请求和响应的信息的程序包；安装并配置该包后，无需修改你的 Web API 中的任何代码即可使用它。将 Web API 部署到 Azure 网站时，将检查所有通信并收集以下统计信息：

- 服务器响应时间。

- 服务器请求数和每个请求的详细信息。

- 就平均响应时间而言，速度最慢的前几个请求。

- 任何失败的请求的详细信息。

- 由不同浏览器和用户代理启动的会话数。

- 最经常查看的网页（主要适用于 Web 应用程序而不是 Web API）。

- 访问 Web API 的不同用户角色。

可以从 Azure 管理门户实时查看此数据。此外，还可以创建监视 Web API 的运行状况的 webtest。Webtest 将定期请求发送到 Web API 中指定的 URI，并捕获响应。你可以指定成功响应的定义（如 HTTP 状态代码 200），如果请求未返回此响应，可以安排向管理员发送警报。如有必要，管理员可以重新启动托管 Web API 的服务器（如果它出现故障）。

<!--Microsoft 网站上的 [Application Insights - 开始监视你的应用的运行状况和使用情况](/documentation/articles/app-insights-start-monitoring-app-health-usage)页提供了详细信息。-->

### 通过 API 管理服务监视 Web API

如果已通过使用 API 管理服务发布你的 Web API，则 Azure 管理门户上的 API 管理页包含一个可用于查看该服务的整体性能的仪表板。使用“分析”页，可向下钻取到该产品的使用方式的详细信息。此页包含以下选项卡：

- **使用情况**。此选项卡提供有关随着时间推移已进行的 API 调用数和用于处理这些调用的带宽的信息。你可以按产品、API 和操作筛选使用情况详细信息。

- **运行状况**。使用此选项卡可查看 API 请求的结果（返回的 HTTP 状态代码）、缓存策略的有效性、API 响应时间和服务响应时间。同样，你可以按产品、API 和操作筛选运行状况数据。

- **活动**。此选项卡提供了以下信息的文本摘要：成功的调用数、失败的调用数、阻止的调用数、平均响应时间以及每个产品、Web API 和操作的响应时间。此页还列出了每个开发人员进行的调用数。

- **速览**。此选项卡显示性能数据的摘要，包括负责进行大多数 API 调用的开发人员，以及接收这些调用的产品、Web API 和操作。

可以使用此信息来确定是否是特定 Web API 或操作导致了瓶颈问题，如有必要，扩展主机环境并添加更多服务器。你还可以确定一个或多个应用程序是否正在使用不相称的资源量，从而应用适当的策略以设置配额并限制调用率。

> [AZURE.NOTE]你可以更改已发布产品的详细信息，所做的更改将立即应用。例如，你可以在 Web API 中添加或删除操作，而无需重新发布包含该 Web API 的产品。

## 相关模式
- [外观](http://en.wikipedia.org/wiki/Facade_pattern)模式描述了如何为 Web API 提供接口。

## 详细信息
- Microsoft 网站上的[了解 ASP.NET Web API](http://www.asp.net/web-api) 页提供了通过使用 Web API 构建 REST 样式 Web 服务的详细介绍。
- Microsoft 网站上的 [ASP.NET Web API 中的路由](http://www.asp.net/web-api/overview/web-api-routing-and-actions/routing-in-aspnet-web-api)页介绍了基于约定的路由在 ASP.NET Web API 框架中的工作方式。
- 有关基于属性的路由的详细信息，请参阅 Microsoft 网站上的 [Web API 2 中的属性路由](http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2)页。
- OData 网站上的[基础教程](http://www.odata.org/getting-started/basic-tutorial/)页提供了 OData 协议功能的简介。
- Microsoft 网站上的 [ASP.NET Web API OData](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) 页包含有关使用使用 ASP.NET 实现 OData Web API 的示例和更多信息。
- Microsoft 网站上的 [Web API 和 Web API OData 中的批处理支持简介](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx)介绍了如何使用 OData 在 Web API 中实现批处理操作。
- Jonathan Oliver 博客上的[幂等性模式](http://blog.jonathanoliver.com/idempotency-patterns/)一文概述了幂等性以及它如何与数据管理操作相关。
- W3C 网站上的[状态代码定义](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)页包含 HTTP 状态代码及其说明的完整列表。
- 有关使用 ASP.NET Web API 处理 HTTP 异常的详细信息，请访问 Microsoft 网站上的 [ASP.NET Web API 中的异常处理](http://www.asp.net/web-api/overview/error-handling/exception-handling)页。
- Microsoft 网站上的 [Web API 全局错误处理](http://www.asp.net/web-api/overview/error-handling/web-api-global-error-handling)一文介绍了如何为 Web API 实现全局错误处理和日志记录策略。
- Microsoft 网站上的[使用 Web 作业在 Microsoft Azure 网站中运行后台任务](/documentation/articles/web-sites-create-web-jobs)页提供了有关使用 Web 作业在 Azure 网站上执行后台操作的信息和示例。
- Microsoft 网站上的 [API 管理](http://azure.microsoft.com/services/api-management/)页介绍了如何发布可提供对 Web API 的受控且安全访问的产品。
- Microsoft 网站上的 [Azure API 管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn776326.aspx)页介绍了如何使用 API 管理 REST API 构建自定义管理应用程序。
- Microsoft 网站上的[使用单元测试验证代码](https://msdn.microsoft.com/zh-cn/library/dd264975.aspx)页提供了有关使用 Visual Studio 创建和管理单元测试的详细信息。
- Microsoft 网站上的[在发布前对应用程序运行性能测试](https://msdn.microsoft.com/zh-cn/library/dn250793.aspx)页介绍了如何使用 Visual Studio Ultimate 来创建 Web 性能和负载测试项目。

<!---HONumber=67-->