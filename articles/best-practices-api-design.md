<properties
   pageTitle="API 设计指南 | Microsoft Azure"
   description="有关如何创建设计良好的 API 的指南。"
   services=""
   documentationCenter="na"
   authors="dragon119"
   manager="masimms"
   editor=""
   tags=""/>

<tags
   ms.service="best-practice"
   ms.date="04/28/2015"
   wacn.date="10/3/2015"/>

# API 设计指南

![](./media/best-practices-api-design/pnp-logo.png)

本指南中的一些主题正在讨论中，将来可能会更改。我们欢迎你的反馈！


## 概述

许多现代的基于 Web 的解决方案都利用 Web 服务器托管的 Web 服务，来为远程客户端应用程序提供相关功能。Web 服务公开的操作构成 Web API。设计良好的 Web API 应旨在支持：

- **平台独立性**。客户端应用程序应能够利用 Web 服务提供的 API，而不需要了解该 API 公开的数据或操作如何物理实现。这就要求该 API 遵守通用标准，使客户端应用程序和 Web 服务可以就以下项达成一致：要使用哪些数据格式以及客户端应用程序和 Web 服务之间交换的数据的结构。

- **服务演变**。Web 服务应能够改进和添加（或删除）功能，而不会影响客户端应用程序。当 Web 服务所提供的功能发生更改时，现有客户端应用程序应该能够继续不变地运行。所有功能还应是可检测到的，以便客户端应用程序能够充分利用它。

本指南旨在说明设计 Web API 时应考虑的问题。

## 表述性状态传递 (REST) 简介

Roy Fielding 在他 2000 年的论文中提出了一种替代架构方法，用于组织 Web 服务公开的操作；REST。REST 是基于超媒体构建分布式系统的架构样式。REST 模型的主要优势在于它基于开放标准，并且不将模型的实现或访问它的客户端应用程序绑定到任何特定实现。例如，REST Web 服务可以使用 Microsoft ASP.NET Web API 加以实现，而客户端应用程序则可通过使用任何可生成 HTTP 请求并分析 HTTP 响应的语言和工具集进行开发。

> [AZURE.NOTE]\: REST 实际上独立于任何基础协议，并且不一定绑定到 HTTP。但是，最常见的基于 REST 的系统实现利用 HTTP 作为发送和接收请求的应用程序协议。本文档重点介绍如何将 REST 原则映射到设计为使用 HTTP 运行的系统。

REST 模型使用导航方案来表示网络上的对象和服务（称为_资源_）。实现 REST 的许多系统通常使用 HTTP 协议来传输要访问这些资源的请求。在这些系统中，客户端应用程序以 URI 的形式提交请求，其中标识资源和 HTTP 方法（最常见的有 GET、POST、PUT 或 DELETE，用于指示要对该资源执行的操作）。HTTP 请求的正文包含执行该操作所需的数据。要了解的要点是 REST 定义无状态的请求模型。HTTP 请求应是独立的，并可以任何顺序发出，因此尝试保留请求之间的瞬时状态信息并不可行。存储信息的唯一位置是在资源本身中，并且每个请求应是原子操作。实际上，REST 模型可实现有限状态机，其中请求将资源从一种明确定义的非瞬时状态转换为另一种状态。

> [AZURE.NOTE]REST 模型中单个请求的无状态性质使得按照这些原则构造的系统高度可缩放。发出一系列请求的客户端应用程序与处理这些请求的特定 Web 服务器的之间无需保留任何关联。

实现有效的 REST 模型的另一个要点是了解该模型提供访问的各种资源之间的关系。这些资源通常被组织为集合和关系。例如，假设一个电子商务系统的快速分析显示有两个集合客户端应用程序可能感兴趣：订单和客户。为了进行标识，每个订单和客户都应有自己的唯一键。用于访问订单集合的 URI 可以是如下简单的形式：_/orders_，同样，用于检索所有客户的 URI 可以是 _/customers_。向 _/orders_URI 发出 HTTP GET 请求应返回表示编码为 HTTP 响应的集合中的所有订单的列表：

```HTTP
GET http://adventure-works.com/orders HTTP/1.1
...
```

下面所示的响应将订单编码为 XML 列表结构。该列表包含 7 个订单：

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
<OrderList xmlns:i="..." xmlns="..."><Order><OrderID>1</OrderID><OrderValue>99.90</OrderValue><ProductID>1</ProductID><Quantity>1</Quantity></Order><Order><OrderID>2</OrderID><OrderValue>10.00</OrderValue><ProductID>4</ProductID><Quantity>2</Quantity></Order><Order><OrderID>3</OrderID><OrderValue>16.60</OrderValue><ProductID>2</ProductID><Quantity>4</Quantity></Order><Order><OrderID>4</OrderID><OrderValue>25.90</OrderValue><ProductID>3</ProductID><Quantity>1</Quantity></Order><Order><OrderID>7</OrderID><OrderValue>99.90</OrderValue><ProductID>1</ProductID><Quantity>1</Quantity></Order></OrderList>
```
若要提取单个订单，需要指定_订单_资源中的订单标识符，例如 _/orders/2_：

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
```

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
<Order xmlns:i="..." xmlns="...">
<OrderID>2</OrderID><OrderValue>10.00</OrderValue><ProductID>4</ProductID><Quantity>2</Quantity></Order>
```

> [AZURE.NOTE]为简单起见，这些示例显示响应中作为 XML 文本数据返回的信息。但是，没有理由资源不应包含 HTTP 支持的任何其他类型的数据（如二进制或加密信息）；HTTP 响应中的 content-type 应指定类型。此外，REST 模型还能够以不同格式返回相同数据，如 XML 或 JSON。在这种情况下，Web 服务应能够与发出请求的客户端执行内容协商。请求可包含 _Accept_ 标头以指定客户端想要接收的首选格式，Web 服务应尝试尽可能采用这种格式。

请注意，REST 请求的响应需使用标准 HTTP 状态代码。例如，返回有效数据的请求应包括 HTTP 响应代码 200（正常），而无法找到或删除指定资源的请求则应返回包含 HTTP 状态代码 404（未找到）的响应。

## 设计和组织 RESTful Web API

设计成功的 Web API 的关键是简单和一致性。使用表现出这两个因素的 Web API，可更轻松地生成需要使用该 API 的客户端应用程序。

RESTful Web API 着重于公开一组连接资源，并提供核心操作使应用程序可以处理这些资源并在这些资源之间轻松导航。为此，构成典型 RESTful Web API 的 URI 应面向它所公开的数据，并使用 HTTP 提供的工具来操作此数据。此方法的思维方式需要与在面向对象的 API 中设计一组类时通常采用的思维方式不同，后者往往更多由对象和类的行为驱动。此外，RESTful Web API 应是无状态的，并且不依赖于按特定顺序调用的操作。以下部分总结了在设计 RESTful Web API 时应考虑的点。

### 围绕资源组织 Web API

> [AZURE.TIP]REST Web 服务公开的 URI 应基于名词（Web API 提供访问的数据），而不是基于动词（应用程序可以对数据执行什么操作）。

侧重于 Web API 公开的业务实体。例如，在前面所述的设计为支持电子商务系统的 Web API 中，主要实体为客户和订单。下订单操作等过程可以通过提供 HTTP POST 操作完成，该操作接受订单信息并将其添加到客户的订单列表中。在内部，此 POST 操作可以执行检查库存级别和为客户计费等任务。HTTP 响应可以指示下订单是否成功。另请注意，资源无需基于单个物理数据项目。例如，订单资源可能会通过使用从分布在关系数据库中多个表的多个行聚合的信息在内部实现，但以单个实体的形式提供给客户端。

> [AZURE.TIP]避免设计这样的 REST 接口：镜像或依赖于它公开的数据的内部结构。REST 不只是关于对关系数据库中的单独表实现简单 CRUD（创建、检索、更新、删除）操作。REST 旨在将业务实体和应用程序可以对这些实体上执行的操作映射到这些实体的物理实现，但不应向客户端公开这些物理详细信息。

单个业务实体很少会孤立存在（尽管可能存在某些单一实例对象），而是往往会组合在一起成为集合。在 REST 术语中，每个实体和每个集合都是资源。在 RESTful Web API 中，每个集合都在 Web 服务中有自己的 URI，对集合的 URI 执行 HTTP GET 请求将检索该集合中的项目的列表。每个单个项目也具有自己的 URI，应用程序可以使用该 URI 再提交一个 HTTP GET 请求以检索该项目的详细信息。应以分层方式组织集合和项目的 URI。在电子商务系统中，URI _/customers_ 表示客户的集合，_/customers/5_ 检索此集合中 ID 为 5 的单个客户的详细信息。这种方法有助于使 Web API 保持直观。

> [AZURE.TIP]在 URI 中采用一致的命名约定；通常，这有助于对引用集合的 URI 中使用复数名词。

你还需要考虑不同类型的资源之间的关系，以及你可能会如何公开这些关联。例如，客户可能会下零个或更多订单。表示此关系的自然方式可以是通过 URI（如 _/customers/5/orders_）查找客户 5 的所有订单。你还可能会考虑通过 URI 从订单追溯到特定客户来表示此关联，例如 _/orders/99/customer_ 可查找订单 99 的客户，但扩展此模型太长可能会变得难以实现。较好的解决方案是在查询订单时返回的 HTTP 响应消息的正文中提供指向关联资源（例如，客户）的可导航链接。此机制将稍后在本指南的“使用 HATEOAS 方法启用相关资源的导航”一节中详细介绍。

在更复杂的系统中可能会有更多类型的实体，这容易让人想到提供 URI 使客户端应用程序可以在多级关系中导航，例如 _/customers/1/orders/99/products_ 用于获取客户 1 下的订单 99 中的产品列表。但是，如果资源之间的关系在将来更改，此级别的复杂性可能很难维护并且不够灵活。更准确地说，你应尽量保持 URI 相对简单。请记住，应用程序具有对资源的引用后，它应能够使用此引用查找与该资源相关的项目。前面的查询可以替换为 URI _/customers/1/orders_ 以查找客户 1 的所有订单，然后查询 URI _/orders/99/products_ 以查找此订单中的产品（假定订单 99 是客户 1 下的）。

> [AZURE.TIP]避免需要复杂度超过_集合/项目/集合_的资源 URI。

要考虑的另一点是：所有 Web 请求都在 Web 服务器上施加负载，请求数越多，负载越大。你应尝试定义资源以避免使用公开大量小型资源的“细粒度”Web API。此类 API 可能需要客户端应用程序提交多个请求才能查找它所需的所有数据。最好使数据非规范化，将相关信息组合在一起成为可通过发出单个请求检索的较大资源。但是，你需要根据客户端可能并不经常需要执行的提取数据的开销来权衡此方法。如果附加的数据不经常使用，检索大型对象可能增加请求的延迟时间并产生额外的带宽成本，相对而言优势则较小。

避免将 Web API 之间的依赖关系引入到基础数据源的结构、类型或位置。例如，如果你的数据位于关系数据库中，则 Web API 不需要将每个表公开为资源的集合。将 Web API 视为数据库的抽象，如有必要引入数据库和 Web API 之间的映射层。这样，如果数据库的设计或实现发生更改（例如，你从包含规范化表集合的关系数据库移到文档数据库等非规范化 NoSQL 存储系统），客户端应用程序将不会受这些更改影响。
> [AZURE.TIP]支持 Web API 的数据源不必是数据存储；它可以是另一个服务或业务线应用程序，甚至可以是在组织内本地运行的旧版应用程序。

最后，可能无法将 Web API 实现的每个操作都映射到特定资源。你可以通过 HTTP GET 请求处理此类_非资源_方案，这些请求调用一项功能并将结果作为 HTTP 响应消息返回。实现简单的计算器样式操作（例如，加法和减法）的 Web API 可以提供公开这些操作作为伪资源的 URI，并利用查询字符串来指定所需的参数。例如，向 URI _/add?operand1=99&operand2=1_ 发出 GET 请求会返回正文包含值 100 的响应消息，而向 URI _/subtract?operand1=50&operand2=20_ 发出 GET 请求则会返回正文包含值 30 的响应消息。但是，请尽量少使用这些形式的 URI。

### 根据 HTTP 方法定义操作

HTTP 协议定义了大量为请求赋于语义含义的方法。大多数 RESTful Web API 使用的常见 HTTP 方法是：

- **GET**，用于检索指定 URI 处的资源的副本。响应消息的正文包含所请求资源的详细信息。

- **POST**，用于在指定 URI 处创建新资源。请求消息的正文将提供新资源的详细信息。请注意，POST 还用于触发不实际创建资源的操作。

- **PUT**，用于替换或更新指定 URI 处的资源。请求消息的正文指定要修改的资源和要应用的值。

- **DELETE**，用于删除指定 URI 处的资源。

> [AZURE.NOTE]HTTP 协议还定义了其他不太常用的方法，如 PATCH（用于请求对资源进行选择性更新）、HEAD（用于请求资源的说明）、OPTIONS（使客户端可以获取有关服务器支持的通信选项的信息）和 TRACE（允许客户端请求可用于测试和诊断目的的信息）。

特定请求的作用应取决于它所应用于的资源是集合还是单个项目。下表汇总了使用电子商务示例的大多数 RESTful 实现所采用的常见约定。请注意，可能并非实现所有这些请求；这取决于特定方案。

| **资源** | **POST** | **GET** | **PUT** | **删除** |
|--------------|----------|---------|---------|------------|
| /customers | 创建新客户 | 检索所有客户 | 批量更新客户（_如果已实现_） | 删除所有客户 |
| /customers/1 | 错误 | 检索客户 1 的详细信息 | 如果客户 1 存在，更新客户 1 的详细信息，否则返回错误 | 删除客户 1 |
| /customers/1/orders | 创建客户 1 的新订单 | 检索客户 1 的所有订单 | 批量更新客户 1 的订单（_如果已实现_） | 删除客户 1 的所有订单（_如果已实现_） |

GET 和 DELETE 请求的目的相对简单，但就 POST 和 PUT 请求的的目的和作用而言，存在混淆区域。

POST 请求应使用请求正文中提供的数据创建新资源。在 REST 模型中，你会经常将 POST 请求应用于作为集合的资源；将新资源添加到集合中。

> [AZURE.NOTE]你还可以定义触发某种功能（以及不必返回数据）的 POST 请求，这些类型的请求可应用于集合。例如，可以使用 POST 请求将工时单传递给工资单处理服务并返回计算的税款作为响应。

PUT 请求用于修改现有资源。如果指定的资源不存在，PUT 请求可能返回错误（在某些情况下，它可能会实际创建该资源）。PUT 请求最常应用于作为单个项目的资源（例如，特定客户或订单），虽然它们可以应用于集合，但这不太常实现。请注意，PUT 请求是幂等的，而 POST 请求则不是；如果应用程序多次提交同一 PUT 请求，则结果应始终是相同的（将使用相同的值修改同一资源），但如果应用程序重复同一 POST 请求，则结果将是创建多个资源。

> [AZURE.NOTE]严格地说，HTTP PUT 请求会将现有资源替换为请求正文中指定的资源。如果目的是要修改资源的所选属性但保留其他属性保持不变，则应使用 HTTP PATCH 请求实现此目的。但是，许多 RESTful 实现放宽了此规则并对这两种情况都使用 PUT。

### 处理 HTTP 请求
许多 HTTP 请求中客户端应用程序提供的数据和 Web 服务器返回的相应响应消息可以各种格式（或媒体类型）提供。例如，用于指定客户或订单的详细信息的数据可以 XML、JSON 或其他其种编码和压缩格式提供。RESTful Web API 应支持提交请求的客户端应用程序请求的多种不同的媒体类型。

当客户端应用程序发送需要在消息正文中返回数据的请求时，它可以在请求的 Accept 标头中指定它可以处理的媒体类型。下面的代码说明了一个 HTTP GET 请求，该请求检索客户 1 的详细信息并请求以 JSON 格式返回结果（客户端仍应检查响应中的数据的媒体类型以验证返回的数据格式）：

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
Accept: application/json
...
```

如果 Web 服务器支持此媒体类型，则可以使用包含 Content-Type 标头（指定消息正文中的数据的格式）的响应来应答：

> [AZURE.NOTE]为了最大限度地提高互操作性，Accept 和 Content-type 标头中引用的媒体类型应为公认的 MIME 类型而不是某种自定义媒体类型。

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"OrderID":2,"ProductID":4,"Quantity":2,"OrderValue":10.00}
```

如果 Web 服务器不支持所请求的媒体类型，它可以使用其他格式发送数据。在所有情况下，它都必须在 Content-Type 标头中指定媒体类型（如 _text/xml_）。客户端应用程序负责分析响应消息并相应地解释消息正文中的结果。

请注意，在此示例中，Web 服务器成功检索所请求的数据，并通过在响应标头中传递回状态代码 200 来指示成功。如果未找到任何匹配的数据，它应改为返回状态代码 404（找不到），响应消息的正文可以包含附加信息。此信息的格式由 Content-Type 标头指定，如下面的示例中所示：

```HTTP
GET http://adventure-works.com/orders/222 HTTP/1.1
...
Accept: application/json
...
```

订单 222 不存在，因此响应消息如下所示：

```HTTP
HTTP/1.1 404 Not Found
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"Message":"No such order"}
```

当应用程序发送 HTTP PUT 请求以更新资源时，它将指定资源的 URI 并在请求消息的正文中提供要修改的数据。它还应使用 Content-Type 标头指定此数据的格式。用于基于文本的信息的常见格式是 _application/x-www-form-urlencoded_，它由用 & 字符分隔的一组名称/值对组成。下一个示例演示了用于修改订单 1 中的信息的 HTTP PUT 请求：

```HTTP
PUT http://adventure-works.com/orders/1 HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
ProductID=3&Quantity=5&OrderValue=250
```

如果修改成功，在理想情况下它应以 HTTP 204 状态代码响应，以指示该过程已成功处理，但响应正文未包含任何进一步的信息。响应中的 Location 标头包含新更新的资源的 URI：

```HTTP
HTTP/1.1 204 No Content
...
Location: http://adventure-works.com/orders/1
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

> [AZURE.TIP]如果 HTTP PUT 请求消息中的数据包括日期和时间信息，请确保你的 Web 服务接受按 ISO 8601 标准设置格式的日期和时间。

如果要更新的资源不存在，如前所述，Web 服务器可以使用“未找到”响应进行响应。或者，如果服务器实际创建了该对象本身，则可以返回状态代码 HTTP 200（正常）或 HTTP 201（已创建），响应正文可以包含新资源的数据。如果请求的 Content-Type 标头指定了 Web 服务器无法处理的数据格式，则它应以 HTTP 状态代码 415（不支持的媒体类型）进行响应。

> [AZURE.TIP]请考虑实现可批量更新集合中的多个资源的批量 HTTP PUT 操作。PUT 请求应指定集合的 URI，而请求正文则应指定要修改的资源的详细信息。此方法可帮助减少交互成本并提高性能。

用于创建新资源的 HTTP POST 请求的格式与 PUT 请求的格式类似；消息正文包含要添加的新资源的详细信息。但是，URI 通常指定资源应添加到的集合。下面的示例将创建新订单并将其添加到订单集合：

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
ProductID=5&Quantity=15&OrderValue=400
```

如果该请求成功，Web 服务器应以包含 HTTP 状态代码 201（已创建）的消息代码进行响应。Location 标头应包含新创建的资源的 URI，响应正文应包含新资源的副本；Content-Type 标头指定此数据的格式：

```HTTP
HTTP/1.1 201 Created
...
Content-Type: application/json; charset=utf-8
Location: http://adventure-works.com/orders/99
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"OrderID":99,"ProductID":5,"Quantity":15,"OrderValue":400}
```

> [AZURE.TIP]如果 PUT 或 POST 请求提供的数据无效，Web 服务器应以包含 HTTP 状态代码 400（错误请求）的消息进行响应。此消息的正文可以包含有关请求存在的问题的其他信息以及所需的格式，也可以包含指向提供更多详细信息的 URL 的链接。

若要删除资源，HTTP DELETE 请求只需提供要删除的资源的 URI。下面的示例将尝试删除订单 99：

```HTTP
DELETE http://adventure-works.com/orders/99 HTTP/1.1
...
```

如果删除操作成功，Web 服务器应以 HTTP 状态代码 204 进行响应，以指示该过程已成功处理，但响应正文未包含任何进一步的信息（这与成功的 PUT 操作返回的响应相同，但没有 Location 标头，因为资源不再存在）。 如果异步执行删除操作，DELETE 请求还有可能返回 HTTP 状态代码 200（正常）或 202（已接受）。

```HTTP
HTTP/1.1 204 No Content
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

如果未找到资源，Web 服务器应改为返回 404（未找到）消息。

> [AZURE.TIP]如果需要删除集合中的所有资源，请为 HTTP DELETE 请求指定集合的 URI，而不是强制应用程序从集合中依次删除每个资源。

### 对数据进行筛选和分页

你应尽力使 URI 保持简单和直观。通过单个 URI 公开资源的集合可在此方面有帮助，但这可能会导致应用程序在只需信息的子集时提取大量数据。生成大量流量不仅会影响 Web 服务器的性能和可伸缩性，而且会反过来影响请求数据的客户端应用程序的响应能力。

例如，如果订单包含为订单支付的价格，需要检索成本高于特定值的所有订单的客户端应用程序可能需要从 _/orders_ URI 检索所有订单，然后在本地筛选出这些订单。此过程显然效率非常低；它浪费了托管 Web API 的服务器的网络带宽和处理能力。

一种解决方案是提供 URI 方案，例如 _/orders/ordervalue\_greater\_than\_n_，其中 _n_ 是订单价格，但对于所有订单而言只是有限数量的价格，这种方法不切实际。此外，如果你需要基于其他条件查询订单，你可能最终面临着需要提供使用可能非直观名称的 URI 的长列表。

筛选数据的较好策略是在传递到 Web API 的查询字符串中提供筛选条件，例如 _/orders?ordervaluethreshold=n_。在此示例中，Web API 中的相应操作负责分析和处理查询字符串中的 `ordervaluethreshold` 参数并在 HTTP 响应中返回筛选结果。

对集合资源执行的一些简单 HTTP GET 请求可能会返回大量项目。若要防止发生这种现象，应将 Web API 设计为限制任何单个请求返回的数据量。可以通过支持这样的查询字符串来达到此目的：允许用户指定要检索的最大项目数（这样它本身就可以受到上限约束有助于防止拒绝服务攻击）和集合中的起始偏移量。例如，URI _/orders?limit=25&offset=50_ 中的查询字符串应检索从订单集合中找到的第 50 个订单开始的 25 个订单。与筛选数据一样，在 Web API 中实现 GET 请求的操作负责分析和处理查询字符串中的 `limit` 和 `offset` 参数。若要帮助客户端应用程序，返回分页数据的 GET 请求还应包含某种形式的元数据，以指示集合中可用的资源总数。你可能还会考虑其他智能分页策略；有关详细信息，请参阅 [API 设计说明：智能分页](http://bizcoder.com/api-design-notes-smart-paging)

在提取数据时可以执行类似的策略对数据进行排序；你可以提供一个接受字段名称作为值的排序参数，例如 _/orders?sort=ProductID_。但请注意，这种方法会对缓存产生负面影响（查询字符串参数构成许多缓存实现用作缓存数据的键的资源标识符的一部分）。

如果单个资源项目包含大量数据，你可以扩展此方法来限制（投影）返回的字段。例如，可以使用接受以逗号分隔的字段列表的查询字符串参数，例如 _/orders?fields=ProductID,Quantity_。

> [AZURE.TIP]为查询字符串中的所有可选参数提供有意义的默认值。例如，如果实现分页，将 `limit` 参数设为 10，将 `offset` 参数设为 0；如果实现排序，将排序参数设为资源的键；如果支持投影，将 `fields` 参数设为资源中的所有字段。

### 处理大型二进制资源

单个资源可能包含大型二进制字段，例如文件或图像。若要解决由不可靠和间歇性连接导致的传输问题并缩短响应时间，请考虑提供使客户端应用程序能够分块检索此类资源的操作。为此，Web API 对于针对大型资源的 GET 请求应支持 Accept-Ranges 标头，并完美地实现对这些资源的 HTTP HEAD 请求。Accept-Ranges 标头指示 GET 操作支持部分结果并且客户端应用程序可以提交返回指定为字节范围的资源子集的 GET 请求。HEAD 请求与 GET 请求类似，不同的是它仅返回描述资源的标头和空的消息正文。客户端应用程序可以发出 HEAD 请求以确定是否要通过使用部分 GET 请求获取某个资源。下面的示例演示获取有关产品图像的信息的 HEAD 请求：

```HTTP
HEAD http://adventure-works.com/products/10?fields=ProductImage HTTP/1.1
...
```

响应消息包含一个提供资源大小（4580 字节）的标头和 Accept-Ranges 标头以指示相应的 GET 操作支持部分结果：

```HTTP
HTTP/1.1 200 OK
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
...
```

客户端应用程序可以使用此信息构造一系列 GET 操作以检索较小区块中的图像。第一个请求通过使用 Range 标头提取前 2500 个字节：

```HTTP
GET http://adventure-works.com/products/10?fields=ProductImage HTTP/1.1
Range: bytes=0-2499
...
```

响应消息通过返回 HTTP 状态代码 206 指示这是部分响应。Content-Length 标头指定消息正文中返回的实际字节数（不是资源的大小），Content-Range 标头指示这是该资源的哪一部分（第 0 到 2499 字节，总共 4580 个字节）:

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580
...
_{binary data not shown}_
```

客户端应用程序的后续请求可以通过使用相应的 Range 标头检索资源的其余部分：

```HTTP
GET http://adventure-works.com/products/10?fields=ProductImage HTTP/1.1
Range: bytes=2500-
...
```

相应的结果消息应如下所示：

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2080
Content-Range: bytes 2500-4580/4580
...
```

## 使用 HATEOAS 方法实现导航到相关资源

REST 背后的主要动机之一是它应能够导航整个资源集，而无需事先了解 URI 方案。每个 HTTP GET 请求应通过响应中包含的超链接返回查找与所请求的对象直接相关的资源所需的信息，还应为它提供描述其中每个资源提供的操作的信息。此原则称为 HATEOAS 或作为应用程序状态引擎的超文本。该系统实际上是有限状态机，每个请求的响应包含从一种状态转为另一种状态所需的信息；任何其他信息都不应是必需的。

> [AZURE.NOTE]当前没有任何标准或规范定义如何为 HATEOAS 原则建模。此节中所示的示例说明了一个可能的解决方案。

例如，若要处理客户和订单之间的关系，在特定订单的响应中返回的数据应包含超链接形式的 URI，以标识下订单的客户以及可以对该客户执行的操作。

```HTTP
GET http://adventure-works.com/orders/3 HTTP/1.1
Accept: application/json
...
```

响应消息的正文包含一个 `Links` 数组（在代码示例中突出显示），用于指定关系的性质（_客户_）、客户的 URI (\__http://adventure-works.com/customers/3_))、检索此客户的详细信息的方法 (_GET_)，以及 Web 服务器支持检索此信息的 MIME 类型（_text/xml_ 和 _application/json_）。这是客户端应用程序要能够提取客户的详细信息所需的所有信息。此外，链接数组还包括其他可以执行的操作的链接，例如 PUT（用于修改客户以及 Web 服务器需要客户端提供的格式）和 DELETE。

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"OrderID":3,"ProductID":2,"Quantity":4,"OrderValue":16.60,"Links":[(some links omitted){"Relationship":"customer","HRef":" http://adventure-works.com/customers/3", "Action":"GET","LinkedResourceMIMETypes":["text/xml","application/json"]},{"Relationship":"
customer","HRef":" http://adventure-works.com /customers/3", "Action":"PUT","LinkedResourceMIMETypes":["application/x-www-form-urlencoded"]},{"Relationship":"customer","HRef":" http://adventure-works.com /customers/3","Action":"DELETE","LinkedResourceMIMETypes":[]}]}
```

出于完整性考虑，链接数组还应包括有关已检索到的资源的自引用信息。这些链接在前一示例中省略了，但在下面的代码中突出显示。请注意，在这些链接中，关系 _self_ 已用于指示这是对该操作所返回的资源的引用：

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"OrderID":3,"ProductID":2,"Quantity":4,"OrderValue":16.60,"Links":[{"Relationship":"self","HRef":" http://adventure-works.com/orders/3", "Action":"GET","LinkedResourceMIMETypes":["text/xml","application/json"]},{"Relationship":" self","HRef":" http://adventure-works.com /orders/3", "Action":"PUT","LinkedResourceMIMETypes":["application/x-www-form-urlencoded"]},{"Relationship":"self","HRef":" http://adventure-works.com /orders/3", "Action":"DELETE","LinkedResourceMIMETypes":[]},{"Relationship":"customer",
"HRef":" http://adventure-works.com /customers/3", "Action":"GET","LinkedResourceMIMETypes":["text/xml","application/json"]},{"Relationship":" customer" (customer links omitted)}]}
```

要使此方法有效，必须客户端应用程序准备好检索和分析此附加信息。

## 对 RESTful Web API 进行版本控制

在除最简单的情况以外的所有情况下，Web API 将不大可能保持不变。随着业务需求变化，可能会添加新的资源集合，资源之间的关系可能会更改，并且可能会修改资源中的数据结构。虽然更新 Web API 以处理新的或不同的需求是一个相对简单的过程，但你必须考虑此类更改将对使用该 Web API 的客户端应用程序造成的影响。问题在于尽管设计和实现 Web API 的开发人员可以完全控制该 API，但开发人员对客户端应用程序不具有相同程度的控制，因为这些客户端应用程序可能是由远程运营的第三方组织生成的。主要规则是要让现有客户端应用程序能够继续不变地正常运行，同时允许新客户端应用程序利用新功能和新资源。

版本控制使 Web API 可以指定它所公开的功能和资源，并且客户端应用程序可以提交定向到特定版本的功能或资源的请求。以下各节介绍几种不同的方法，其中每一种方法都有其自己的优势和不足。

### 无版本控制

这是最简单的方法，它对于一些内部 API 来说可能是可以接受的。较大的更改可以表示为新资源或新链接。向现有资源添加内容可能未呈现重大更改，因为不应查看此内容的客户端应用程序将直接忽略它。

例如，向 URI \__http://adventure-works.com/customers/3_ 发出请求应返回包含客户端应用程序所需的 `Id`、`Name` 和 `Address` 字段的单个客户的详细信息：

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
[{"Id":3,"Name":"Contoso LLC","Address":"1 Microsoft Way Redmond WA 98053"}]
```

> [AZURE.NOTE]为简单清晰起见，本节中所示的示例响应将不包括 HATEOAS 链接。

如果 `DateCreated` 字段已添加到客户资源的架构中，则响应将如下所示：

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
[{"Id":3,"Name":"Contoso LLC","DateCreated":"2014-09-04T12:11:38.0376089Z","Address":"1 Microsoft Way Redmond WA 98053"}]
```

现有的客户端应用程序可能会继续正常工作（如果能够忽略无法识别的字段），而新的客户端应用程序则可以设计为处理该新字段。但是，如果对资源的架构进行了更根本的更改（如删除或重命名字段）或资源之间的关系发生更改，则这些更改可能构成重大更改，从而阻止现有客户端应用程序正常工作。在这些情况下应考虑以下方法之一。

### URI 版本控制

每次修改 Web API 或更改资源的架构时，向每个资源的 URI 添加版本号。以前存在的 URI 应像以前一样继续运行，并返回符合原始架构的资源。

继续前面的示例，如果将 `Address` 字段重构为包含地址的每个构成部分的子字段（例如 `StreetAddress`、`City`、`State` 和 `ZipCode`），则此版本的资源可通过包含版本号的 URI（如 http://adventure-works.com/v2/customers/3）公开：

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
[{"Id":3,"Name":"Contoso LLC","DateCreated":"2014-09-04T12:11:38.0376089Z","Address":{"StreetAddress":"1 Microsoft Way","City":"Redmond","State":"WA","ZipCode":98053}}]
```

此版本控制机制非常简单，但依赖于将请求路由到相应终结点的服务器。但是，随着 Web API 经过多次迭代而变得成熟，服务器必须支持多个不同版本，它可能变得难以处理。此外，从单纯的角度来看，在所有情况下客户端应用程序都要提取相同数据（客户 3），因此 URI 实在不应该因版本而有所不同。此方案也增加了 HATEOAS 实现的复杂性，因为所有链接都需要在其 URI 中包括版本号。

### 查询字符串版本控制

不是提供多个 URI，而是可以通过在追加到 HTTP 请求后面的查询字符串中使用参数来指定资源的版本，例如 \__http://adventure-works.com/customers/3?version=2_。如果 version 参数被较旧的客户端应用程序省略，则应默认为有意义的值（例如 1）。

此方法具有语义优势（即，同一资源始终从同一 URI 进行检索），但它依赖于代码处理请求以分析查询字符串并发送回相应的 HTTP 响应。此方法也与 URI 版本控制机制一样，增加了实现 HATEOAS 的复杂性。

> [AZURE.NOTE]某些较旧的 Web 浏览器和 Web 代理将不会缓存在 URL 中包括查询字符串的请求的响应。这可能会对使用 Web API 的 Web 应用程序以及从此类 Web 浏览器运行的 Web 应用程序的性能产生不利影响。

### 标头版本控制

不是追加版本号作为查询字符串参数，而是可以实现指示资源的版本的自定义标头。此方法需要客户端应用程序将相应标头添加到所有请求，虽然如果省略了版本标头，处理客户端请求的代码可以使用默认值（版本 1）。下面的示例利用了名为 _Custom-Header_ 的自定义标头。此标头的值指示 Web API 的版本。

版本 1：

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=1
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
[{"Id":3,"Name":"Contoso LLC","Address":"1 Microsoft Way Redmond WA 98053"}]
```

版本 2：

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=2
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
[{"Id":3,"Name":"Contoso LLC","DateCreated":"2014-09-04T12:11:38.0376089Z","Address":{"StreetAddress":"1 Microsoft Way","City":"Redmond","State":"WA","ZipCode":98053}}]
```

请注意，与前面两个方法一样，实现 HATEOAS 需要在任何链接中包括相应的自定义标头。

### 媒体类型版本控制

如本指南前面所述，当客户端应用程序向 Web 服务器发送 HTTP GET 请求时，它应使用 Accept 标头规定它可以处理的内容的格式。通常，_Accept_ 标头的用途是允许客户端应用程序指定响应的正文应是 XML、JSON 还是客户端可以分析的其他某种常见格式。但是，可以定义包括以下信息的自定义媒体类型：该信息使客户端应用程序可以指示它所需的资源版本。下面的示例演示了将 _Accept_ 标头指定为值 _application/vnd.adventure-works.v1+json_ 的请求。_vnd.adventure-works.v1_ 元素向 Web 服务器指示它应返回资源的版本 1，而 _json_ 元素则指定响应正文的格式应为 JSON：

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Accept: application/vnd.adventure-works.v1+json
...
```

处理请求的代码负责处理 _Accept_ 标头并尽可能采用该值（客户端应用程序可以在 _Accept_ 标头中指定多种格式，在这种情况下，Web 服务器可以在其中选择最适合的格式用于响应正文）。Web 服务器使用 Content-Type 标头确认响应正文中的数据格式：

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8
...
Content-Length: ...
[{"Id":3,"Name":"Contoso LLC","Address":"1 Microsoft Way Redmond WA 98053"}]
```

如果 Accept 标头未指定任何已知的媒体类型，则 Web 服务器可以生成 HTTP 406（不可接受）响应消息或返回使用默认媒体类型的消息。

此方法可以说是最纯粹的版本控制机制并自然地适用于 HATEOAS，后者可以在资源链接中包含相关数据的 MIME 类型。

> [AZURE.NOTE]当你选择版本控制策略时，还应考虑对性能的影响，尤其是在 Web 服务器上缓存时。URI 版本控制和查询字符串版本控制方案都是缓存友好的，因为同一 URI/查询字符串组合每次都指向相同的数据。

> 标头版本控制和媒体类型版本控制机制通常需要其他逻辑来检查自定义标头或 Accept 标头中的值。在大型环境中，使用不同版本的 Web API 的多个客户端可能会在服务器端缓存中生成大量重复数据。如果客户端应用程序通过实现缓存的代理与 Web 服务器进行通信，并且该代理在当前未在其缓存中保留所请求数据的副本时，仅将请求转发到 Web 服务器，则此问题可能会变得很严重。

## 详细信息

- [RESTful Cookbook](http://restcookbook.com/) 包含构建 RESTful API 的简介。
- Web [API 清单](https://mathieu.fenniak.net/the-api-checklist/)包含在设计和实现 Web API 时要考虑的有用的项目列表。

<!---HONumber=71-->