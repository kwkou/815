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
 	ms.date="03/01/2016"  
	wacn.date="04/05/2016"/>


#媒体服务 REST API 概述 

[AZURE.INCLUDE [media-services-selector-setup](../includes/media-services-selector-setup.md)]Azure 媒体服务是一项服务，该服务接受基于 OData 的 HTTP 请求并能够以详细 JSON 或 atom+pub 做出响应。由于媒体服务遵循 Azure 设计准则，因此在连接到媒体服务时，每个客户端必须使用一组必需的 HTTP 标头，还可以使用一组可选标头。以下部分介绍你在创建请求和接收来自媒体服务的响应时可以使用的标头和 HTTP 谓词。

##注意事项 

使用 REST 时需考虑下列事项：

- 查询实体时，一次返回的实体数限制为 1000 个，因为公共 REST v2 将查询结果数限制为 1000 个。你需要使用[此 .NET 示例](/documentation/articles/media-services-dotnet-manage-entities/#enumerating-through-large-collections-of-entities)和[此 REST API 示例](/documentation/articles/media-services-rest-manage-entities/#enumerating-through-large-collections-of-entities)中所述的 Skip 和 Take (.NET)/ top (REST)。 

- 使用 JSON 并指定在请求中使用 **__metadata** 关键字（例如，为了引用某个链接对象）时，必须将 Accept 标头设置为 [JSON 详细格式](http://www.odata.org/documentation/odata-version-3-0/json-verbose-format/)（参阅以下示例）。Odata 并不了解请求中的 **__metadata** 属性，除非你将它设置为 verbose。

		POST https://media.chinacloudapi.cn/API/Jobs HTTP/1.1
		Content-Type: application/json;odata=verbose
		Accept: application/json;odata=verbose
		DataServiceVersion: 3.0
		MaxDataServiceVersion: 3.0
		x-ms-version: 2.11
		Authorization: Bearer <token> 
		Host: media.windows.net
		
		{
			"Name" : "NewTestJob", 
			"InputMediaAssets" : 
				[{"__metadata" : {"uri" : "https://media.chinacloudapi.cn/api/Assets('nb%3Acid%3AUUID%3Aba5356eb-30ff-4dc6-9e5a-41e4223540e7')"}}]
		. . . 
		

## 媒体服务支持的标准 HTTP 请求标头

每次调用媒体服务时，你必须在请求中包括一组必需标头，还可以根据需要包括一组可选标头。下表列出了必需的标头：


标头|类型|值
---|---|---
授权|持有者|持有者是唯一接受的授权机制。该值还必须包括由 ACS 提供的访问令牌。
x-ms-version|小数|2.11
DataServiceVersion|小数|3.0
MaxDataServiceVersion|小数|3.0



>[AZURE.NOTE]由于媒体服务使用 OData 通过 REST API 公布其基础资产元数据存储库，因此任何请求中均应包括 DataServiceVersion 和 MaxDataServiceVersion 标头，但如果未包括这些标头，则当前媒体服务会假定使用的 DataServiceVersion 值为 3.0。

以下是一组可选标头：

标头|类型|值
---|---|---
日期|RFC 1123 日期|请求的时间戳
Accept|内容类型|响应的请求内容类型，如下所示：<p> -application/json;odata=verbose<p> - application/atom+xml<p> 响应可能会有不同的内容类型，比如 blob 提取，成功的响应将在其中包含 blob 流作为有效负载。
Accept-Encoding|Gzip、deflate|GZIP 和 DEFLATE 编码（如果适用）。注意：对于大型资源，媒体服务可能会忽略此标头并返回未经压缩的数据。
Accept-Language|“en”、“es”等。|指定响应的首选语言。
Accept-Charset|字符集类型，如“UTF-8”|默认值为 UTF-8。
X-HTTP-Method|HTTP 方法|允许不支持 HTTP 方法（例如 PUT 或 DELETE）的客户端或防火墙使用这些通过 GET 调用隧道化的方法。
Content-Type|内容类型|PUT 或 POST 请求中请求正文的内容类型。
client-request-id|String|调用方定义的值，用于标识给定请求。如果指定，将在响应消息中包含此值，作为一种映射请求的方法。<p><p>**重要信息**<p>值的上限应为 2096b (2k)。

## 媒体服务支持的标准 HTTP 响应标头

下面是可以根据你请求的资源以及要执行的操作返回给你的一组标头。


标头|类型|值
---|---|---
request-id|String|当前操作的唯一标识符，由服务生成。
client-request-id|String|调用方在原始请求（如果存在）中指定的标识符。
日期|RFC 1123 日期|处理请求的日期。
Content-Type|多种多样|响应正文的内容类型。
Content-Encoding|多种多样|Gzip 或 deflate（视情况而定）。


## 媒体服务支持的标准 HTTP 谓词

下面是在提出 HTTP 请求时可以使用的 HTTP 谓词的完整列表：


Verb|说明
---|---
GET|返回对象的当前值。
POST|根据提供的数据创建对象，或提交命令。
PUT|替换对象，或创建命名对象（如果适用）。
删除|删除对象。
MERGE|使用指定的属性更改更新现有对象。
HEAD|为 GET 响应返回对象的元数据。

##限制

查询实体时，一次返回的实体数限制为 1000 个，因为公共 REST v2 将查询结果数限制为 1000 个。你需要使用[此 .NET 示例](/documentation/articles/media-services-dotnet-manage-entities/#enumerating-through-large-collections-of-entities)和[此 REST API 示例](/documentation/articles/media-services-rest-manage-entities/#enumerating-through-large-collections-of-entities)中所述的 **Skip** 和 **Take** (.NET)/ **top** (REST)。


## 发现媒体服务模型

为了使媒体服务实体易于发现，可使用 $metadata 操作。使用该操作，你可以检索所有有效的实体类型、实体属性、关联、函数、操作等。以下示例说明了如何构建 URI：https://media.chinacloudapi.cn/API/$metadata。

如果希望在浏览器中查看元数据，应在 URI 的末尾追加“?api-version=2.x”，或不要在请求中包括 x-ms-version 标头。



  [Azure Management Portal]: http://manage.windowsazure.cn/



 

<!---HONumber=Mooncake_0328_2016-->