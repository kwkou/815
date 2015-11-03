<properties 
	pageTitle="媒体服务 REST API 概述 - Azure" 
	description="媒体服务 REST API 概述" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="09/07/2015"
	wacn.date="11/02/2015"/>


#媒体服务 REST API 概述 

[AZURE.INCLUDE [media-services-selector-setup](../includes/media-services-selector-setup.md)]Windows Azure 媒体服务是一项服务，该服务接受基于 OData 的 HTTP 请求并能够以详细 JSON 或 atom+pub 做出响应。由于媒体服务遵循 Azure 设计准则，因此在连接到媒体服务时，每个客户端必须使用一组必需的 HTTP 标头，还可以使用一组可选标头。以下部分介绍你在创建请求和接收来自媒体服务的响应时可以使用的标头和 HTTP 谓词。


##媒体服务支持的标准 HTTP 请求标头

每次调用媒体服务时，你必须在请求中包括一组必需标头，还可以根据需要包括一组可选标头。下表列出了必需的标头：


<table border="1"> <tr><th>标头</th><th>类型</th><th>值</th></tr> <tr><td>Authorization</td><td>Bearer</td><td>Bearer 是唯一接受的授权机制。该值还必须包含由 ACS 提供的访问令牌。</td></tr> <tr><td>x-ms-version</td><td>十进制</td><td>2.11</td></tr> <tr><td>DataServiceVersion</td><td>十进制</td><td>3.0</td></tr> <tr><td>MaxDataServiceVersion</td><td>十进制</td><td>3.0</td></tr> </table><br/>


>[AZURE.NOTE]由于媒体服务使用 OData 通过 REST API 公布其基础资产元数据存储库，因此任何请求中均应包括 DataServiceVersion 和 MaxDataServiceVersion 标头，但如果未包括这些标头，则当前媒体服务会假定使用的 DataServiceVersion 值为 3.0。

以下是一组可选标头：

<table border="1"> <tr><th>标头</th><th>类型</th><th>值</th></tr> <tr><td>Date</td><td>RFC 1123 日期</td><td>请求的时间戳</td></tr> <tr><td>Accept</td><td>内容类型</td><td>响应的请求内容类型，如下所示：<ul><li>application/json;odata=verbose</li><li>application/atom+xml</li></ul></br> 响应可能会有不同的内容类型，比如 blob 提取，成功的响应将在其中包含 blob 流作为有效负载。</td></tr> <tr><td>Accept-Encoding</td><td>Gzip、deflate</td><td>GZIP 和 DEFLATE 编码（如果适用）。注意：对于大型资源，媒体服务可能会忽略此标头并返回未经压缩的数据。</td></tr> <tr><td>Accept-Language</td><td>“en”、“es”等等。</td><td>指定响应的首选语言。</td></tr> <tr><td>Accept-Charset</td><td>字符集类型，如“UTF-8”</td><td>默认值为 UTF-8。</td></tr> <tr><td>X-HTTP-Method</td><td>HTTP 方法</td><td>允许不支持 HTTP 方法（如 PUT 或 DELETE）的客户端或防火墙使用这些通过 GET 调用隧道化的方法。</td></tr> <tr><td>Content-Type</td><td>内容类型</td><td>PUT 或 POST 请求中请求正文的内容类型。</td></tr> <tr><td>client-request-id</td><td>字符串</td><td>调用方定义的值，用于标识给定请求。如果指定，将在响应消息中包括此值，作为一种映射请求的方法。<br/><br/> <b>重要信息</b><br/> 值的上限应为 2096b (2k)。</td></tr> </table><br/>


##媒体服务支持的标准 HTTP 响应标头

下面是可以根据你请求的资源以及要执行的操作返回给你的一组标头。


<table border="1"> <tr><th>标头</th><th>类型</th><th>值</th></tr> <tr><td>request-id</td><td>字符串</td><td>当前操作的唯一标识符，由服务生成。</td></tr> <tr><td>client-request-id</td><td>字符串</td><td>调用方在原始请求（如果存在）中指定的标识符。</td></tr> <tr><td>Date</td><td>RFC 1123 日期</td><td>处理请求的日期。</td></tr> <tr><td>Content-Type</td><td>视情况而异</td><td>响应正文的内容类型。</td></tr> <tr><td>Content-Encoding</td><td>视情况而异</td><td>Gzip 或 deflate（视情况而定）。</td></tr> </table><br/>

##媒体服务支持的标准 HTTP 谓词

下面是在提出 HTTP 请求时可以使用的 HTTP 谓词的完整列表：


<table border="1"> <tr><th>谓词</th><th>说明</th></tr> <tr><td>GET</td><td>返回对象的当前值。</td></tr> <tr><td>POST</td><td>根据提供的数据创建对象，或提交命令。</td></tr> <tr><td>PUT</td><td>替换对象，或创建命名对象（如果适用）。</td></tr> <tr><td>DELETE</td><td>删除对象。</td></tr> <tr><td>MERGE</td><td>使用指定的属性更改更新现有对象。</td></tr> <tr><td>HEAD</td><td>为 GET 响应返回对象的元数据。</td></tr> </table><br/>

## 发现媒体服务模型

为了使媒体服务实体易于发现，可使用 $metadata 操作。使用该操作，你可以检索所有有效的实体类型、实体属性、关联、函数、操作等。以下示例说明了如何构建 URI：https://media.chinacloudapi.cn/API/$metadata。

如果希望在浏览器中查看元数据，应在 URI 的末尾追加“?api-version=2.x”，或不要在请求中包括 x-ms-version 标头。


<!-- Anchors. -->


<!-- URLs. -->
  [Management Portal]: http://manage.windowsazure.cn/

<!---HONumber=76-->