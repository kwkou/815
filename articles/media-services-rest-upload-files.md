<properties 
	pageTitle="使用 REST API 将文件上载到媒体服务帐户" 
	description="了解如何通过创建和上载资产将媒体内容加入媒体服务。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016" 
	wacn.date="06/27/2016"/>


#使用 REST API 将文件上载到媒体服务帐户

[AZURE.INCLUDE [media-services-selector-upload-files](../includes/media-services-selector-upload-files.md)]
 

在媒体服务中，可以将数字文件上载到资产中。[资产](https://msdn.microsoft.com/zh-cn/library/azure/hh974277.aspx)实体可以包含视频、音频、图像、缩略图集合、图文轨迹和隐藏式字幕文件（以及有关这些文件的元数据。） 将文件上载到资产后，相关内容即安全地存储在云中供后续处理和流式处理。


>[AZURE.NOTE]生成流式处理内容的 URL（例如 http://{AMSAccount}.origin.mediaservices.chinacloudapi.cn/{GUID}/{IAssetFile.Name}/streamingParameters.）时，媒体服务会使用 IAssetFile.Name 属性的值。出于这个原因，不允许使用百分号编码。**Name** 属性的值不能含有任何以下保留的[百分号编码字符](http://zh.wikipedia.org/wiki/百分号编码#.E4.BF.9D.E7.95.99.E5.AD.97.E7.AC.A6.E7.9A.84.E7.99.BE.E5.88.86.E5.8F.B7.E7.BC.96.E7.A0.81)：!*'();:@&=+$,/?%#"。此外，文件扩展名中只能含有一个“.”。

上载资产的基本工作流分为下列各节：

- 创建资产
- 对资产加密（可选）
- 将文件上载到 blob 存储

AMS 还可用于批量上载资产。有关详细信息，请参阅[此](/documentation/articles/media-services-rest-upload-files/#upload_in_bulk)部分。

##上载资产

###创建资产

>[AZURE.NOTE] 使用媒体服务 REST API 时，需注意以下事项：
>
>访问媒体服务中的实体时，必须在 HTTP 请求中设置特定标头字段和值。有关详细信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use/)。

>在成功连接到 https://media.chinacloudapi.cn 之后，你将接收到指定另一个媒体服务 URI 的 301 重定向。必须按[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect-programmatically/)中所述对新的 URI 执行后续调用。
 
资产是媒体服务中多种类型的对象或多组对象（包括视频、音频、图像、缩略图集合、文本轨道和隐藏的解释性字幕文件）的容器。在 REST API 中，创建资产需要向媒体服务发送 POST 请求，并将任何有关资产的属性信息放入请求正文中。

在创建资产时可以指定的属性之一是 **Options**。**Options** 是一个枚举值，描述可用于创建资产的加密选项。有效值为以下列表中的某个值，而不是这些值的组合。

- **无** = **0**：将不使用加密。这是默认值。请注意，使用此选项时，你的内容在传送过程中或静态存储过程中都不会受到保护。如果计划使用渐进式下载交付 MP4，则使用此选项。 

- **StorageEncrypted** = **1**：如果要使用 AES-256 位加密法对文件加密以方便上载和存储，请指定此值。

	如果你的资产已经过存储加密，则必须配置资产传送策略。有关详细信息，请参阅[配置资产传送策略](/documentation/articles/media-services-rest-configure-asset-delivery-policy/)。

- **CommonEncryptionProtected** = **2**：如果要上载使用常见加密法（例如 PlayReady）保护的文件，请指定此值。

- **EnvelopeEncryptionProtected** = **4**：如果要上载使用 AES 文件加密的 HLS，请指定此值。请注意，Transform Manager 必须已对文件进行编码和加密。

>[AZURE.NOTE]如果资产要使用加密，则你必须按以下主题中所述创建 **ContentKey** 并将其链接到你的资产：[如何创建 ContentKey](/documentation/articles/media-services-rest-create-contentkey/)。请注意，将文件上载到资产后，需要使用加密**资产**期间获取的值更新 **AssetFile** 实体上的加密属性。使用 **MERGE** HTTP 请求完成此操作。


以下示例说明了如何创建资产。

**HTTP 请求**

	POST https://media.chinacloudapi.cn/api/Assets HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-2233-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn
	
	{"Name":"BigBuckBunny.mp4"}
	

**HTTP 响应**

如果成功，将返回以下响应：
	
	HTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 452
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.chinacloudapp.cn/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
	request-id: e98be122-ae09-473a-8072-0ccd234a0657
	x-ms-request-id: e98be122-ae09-473a-8072-0ccd234a0657
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Sun, 18 Jan 2015 22:06:40 GMT
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#Assets/@Element",
	   "Id":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "State":0,
	   "Created":"2015-01-18T22:06:40.6010903Z",
	   "LastModified":"2015-01-18T22:06:40.6010903Z",
	   "AlternateId":null,
	   "Name":"BigBuckBunny.mp4",
	   "Options":0,
	   "Uri":"https://storagetestaccount001.blob.core.chinacloudapi.cn/asset-9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "StorageAccountName":"storagetestaccount001"
	}
	
###创建 AssetFile

[AssetFile](http://msdn.microsoft.com/zh-cn/library/azure/hh974275.aspx) 实体表示 blob 容器中存储的视频或音频文件。一个资产文件始终与一个资产关联，而一个资产则可能包含一个或多个资产文件。如果资产文件对象未与 BLOB 容器中的数字文件关联，则媒体服务 Encoder 任务将失败。

请注意，**AssetFile** 实例和实际媒体文件是两个不同的对象。AssetFile 实例包含有关媒体文件的元数据，而媒体文件包含实际媒体内容。

将数字媒体文件上载到 blob 容器后，需要使用 **MERGE** HTTP 请求来更新 AssetFile 中有关媒体文件的信息（如本主题稍后所述）。

**HTTP 请求**

	POST https://media.chinacloudapi.cn/api/Files HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-4ca2-2233-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn
	Content-Length: 164
	
	{  
	   "IsEncrypted":"false",
	   "IsPrimary":"false",
	   "MimeType":"video/mp4",
	   "Name":"BigBuckBunny.mp4",
	   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
	}


**HTTP 响应**

	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 535
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.chinacloudapp.cn/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5')
	Server: Microsoft-IIS/8.5
	request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
	x-ms-request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Mon, 19 Jan 2015 00:34:07 GMT
	
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#Files/@Element",
	   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
	   "Name":"BigBuckBunny.mp4",
	   "ContentFileSize":"0",
	   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "EncryptionVersion":null,
	   "EncryptionScheme":null,
	   "IsEncrypted":false,
	   "EncryptionKeyId":null,
	   "InitializationVector":null,
	   "IsPrimary":false,
	   "LastModified":"2015-01-19T00:34:08.1934137Z",
	   "Created":"2015-01-19T00:34:08.1934137Z",
	   "MimeType":"video/mp4",
	   "ContentChecksum":null
	}


### 创建具有写入权限的 AccessPolicy。 

将任何文件上载到 BLOB 存储之前，请设置用于对资产执行写入操作的访问策略权限。为此，请向 AccessPolicy 实体集发送一个 HTTP POST 请求。请在执行创建操作时定义 DurationInMinutes 值，否则会在响应中收到 500 内部服务器错误消息。有关 AccessPolicies 的详细信息，请参阅 [AccessPolicy](http://msdn.microsoft.com/zh-cn/library/azure/hh974297.aspx)。

以下示例说明了如何创建 AccessPolicy：
		
**HTTP 请求**

	POST https://media.chinacloudapi.cn/api/AccessPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-2233-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn
	
	{"Name":"NewUploadPolicy", "DurationInMinutes":"440", "Permissions":"2"} 

**HTTP 请求**

	If successful, the following response is returned:
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 312
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.chinacloudapp.cn/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae')
	Server: Microsoft-IIS/8.5
	request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
	x-ms-request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Sun, 18 Jan 2015 22:18:06 GMT

	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#AccessPolicies/@Element",
	   "Id":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
	   "Created":"2015-01-18T22:18:06.6370575Z",
	   "LastModified":"2015-01-18T22:18:06.6370575Z",
	   "Name":"NewUploadPolicy",
	   "DurationInMinutes":440.0,
	   "Permissions":2
	}

###获取上载 URL

若要检索实际上载 URL，请创建一个 SAS 定位符。定位符为希望访问资产中文件的客户端定义连接终结点的开始时间和类型。可以为给定 AccessPolicy 和资产对创建多个定位符实体，以处理不同的客户端请求和需求。这其中的任一定位符都可使用 AccessPolicy 的 StartTime 值和 DurationInMinutes 值来确定可以使用某 URL 的时间长度。有关详细信息，请参阅[定位符](http://msdn.microsoft.com/zh-cn/library/azure/hh974308.aspx)。


SAS URL 采用以下格式：

	{https://myaccount.blob.core.chinacloudapi.cn}/{asset name}/{video file name}?{SAS signature}

请注意以下事项：

- 一项给定的资产一次最多只能与五个唯一的定位符相关联。有关详细信息，请参阅定位符。
- 如果需要立即上载文件，应将 StartTime 值设置为当前时间前五分钟。这是因为你的客户端计算机与媒体服务之间可能存在时钟偏差。此外，StartTime 值必须采用以下 DateTime 格式：YYYY-MM-DDTHH:mm:ssZ（例如，“2014-05-23T17:53:50Z”）。	
- 定位符从创建到可用可能会有 30-40 秒的延迟。SAS URL 和源定位符都会出现这个问题。

以下示例说明了如何创建 SAS URL 定位符，由请求正文中的 Type 属性定义（“1”表示 SAS 定位符，“2”表示按需来源定位符）。返回的 **Path** 属性包含上载文件时必须使用的 URL。
	
**HTTP 请求**
	
	POST https://media.chinacloudapi.cn/api/Locators HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-4ca2-2233-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421640053&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=vlG%2fPYdFDMS1zKc36qcFVWnaNh07UCkhYj3B71%2fk1YA%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn
	{  
	   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
	   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "StartTime":"2015-02-18T16:45:53",
	   "Type":1
	}


**HTTP 响应**

如果成功，将返回以下响应：
		
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 949
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.chinacloudapp.cn/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54')
	Server: Microsoft-IIS/8.5
	request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
	x-ms-request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Mon, 19 Jan 2015 03:01:29 GMT
	
	{  
	   "odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#Locators/@Element",
	   "Id":"nb:lid:UUID:af57bdd8-6751-4e84-b403-f3c140444b54",
	   "ExpirationDateTime":"2015-02-19T00:05:53",
	   "Type":1,
	   "Path":"https://storagetestaccount001.blob.core.chinacloudapi.cn/asset-f438649c-313c-46e2-8d68-7d2550288247?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
	   "BaseUri":"https://storagetestaccount001.blob.core.chinacloudapi.cn/asset-f438649c-313c-46e2-8d68-7d2550288247",
	   "ContentAccessComponent":"?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
	   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
	   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
	   "StartTime":"2015-02-18T16:45:53",
	   "Name":null
	}

### 将文件上载到 Blob 存储容器
	
设置 AccessPolicy 和定位符后，即可使用 Azure 存储 REST API 将具体的文件上载到 Azure BLOB 存储容器。也可以按页或块 BLOB 来上载。

>[AZURE.NOTE] 必须将要上载的文件的文件名添加到在上一节收到的定位符 **Path** 值中。例如，https://storagetestaccount001.blob.core.chinacloudapi.cn/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4? . . .

有关使用 Azure 存储 blob 的详细信息，请参阅 [Blob 服务 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dd135733.aspx)。


### 更新 AssetFile 

上载文件后，请更新 FileAsset 大小（和其他）信息。例如：
	
	MERGE https://media.chinacloudapi.cn/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5') HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-4ca2-2233-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421662918&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=utmoXXbm9Q7j4tW1yJuMVA3egRiQy5FPygwadkmPeaY%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn
	
	{  
	   "ContentFileSize":"1186540",
	   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
	   "MimeType":"video/mp4",
	   "Name":"BigBuckBunny.mp4",
	   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
	}


**HTTP 响应**

如果成功，将返回以下响应：HTTP/1.1 204 无内容

### 删除定位符和 AccessPolicy 

**HTTP 请求**


	DELETE https://media.chinacloudapi.cn/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54') HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-2233-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421662918&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=utmoXXbm9Q7j4tW1yJuMVA3egRiQy5FPygwadkmPeaY%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn

	
**HTTP 响应**

如果成功，将返回以下响应：

	HTTP/1.1 204 No Content 
	...

**HTTP 请求**

	DELETE https://media.chinacloudapi.cn/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae') HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f09258-6753-2233-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421662918&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=utmoXXbm9Q7j4tW1yJuMVA3egRiQy5FPygwadkmPeaY%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn

**HTTP 响应**

如果成功，将返回以下响应：

	HTTP/1.1 204 No Content 
	...

##<a id="upload_in_bulk"></a>批量上载资产

###创建 IngestManifest

IngestManifest 是一个容器，用于放置一组资产、资产文件以及可用于确定该组资产或文件的批量引入进度的统计信息。


**HTTP 请求**

	POST https:// media.chinacloudapi.cn/API/IngestManifests HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.chinacloudapi.cn
	Content-Length: 36
	Expect: 100-continue
	
	{ "Name" : "ExampleManifestREST" }

###创建资产

在创建 IngestManifestAsset 之前，你需要创建将使用批量引入完成的资产。资产是媒体服务中多种类型的对象或多组对象（包括视频、音频、图像、缩略图集合、文本轨道和隐藏的解释性字幕文件）的容器。在 REST API 中，创建资产需要向 Azure 媒体服务发送 HTTP POST 请求，并在请求正文中放置资产属性信息。在本示例中，资产是使用请求正文中包含的 StorageEncrption(1) 选项创建的。

**HTTP 响应**

	POST https://media.chinacloudapi.cn/API/Assets HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.chinacloudapi.cn
	Content-Length: 55
	Expect: 100-continue
	
	{ "Name" : "ExampleManifestREST_Asset", "Options" : 1 }

###创建 IngestManifestAsset

IngestManifestAsset 表示 IngestManifest 内与批量引入一起使用的资产。IngestManifestAsset 主要用于将资产链接到清单。Azure 媒体服务会在内部根据关联到 IngestManifestAsset 的 IngestManifestFile 集合来监视文件上载。上载这些文件后，即完成资产操作。可以使用 HTTP POST 请求创建新的 IngestManifestAsset。在请求正文中包括 IngestManifest ID 和资产 ID，以便 IngestManifestAsset 将其链接到一起进行批量引入。

**HTTP 响应**

	POST https://media.chinacloudapi.cn/API/IngestManifestAssets HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.chinacloudapi.cn
	Content-Length: 152
	Expect: 100-continue
	{ "ParentIngestManifestId" : "nb:mid:UUID:5c77f186-414f-8b48-8231-17f9264e2048", "Asset" : { "Id" : "nb:cid:UUID:b757929a-5a57-430b-b33e-c05c6cbef02e"}}


###为每个资产创建 IngestManifestFile

IngestManifestFile 代表将作为批量引入资产的一部分上载的实际视频或音频 blob 对象。除非资产使用加密选项，否则不需要与加密相关的属性。本部分使用的示例演示了如何创建 IngestManifestFile，以便将 StorageEncryption 用于之前创建的资产。


**HTTP 响应**

	POST https://media.chinacloudapi.cn/API/IngestManifestFiles HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.chinacloudapi.cn
	Content-Length: 367
	Expect: 100-continue
	
	{ "Name" : "REST_Example_File.wmv", "ParentIngestManifestId" : "nb:mid:UUID:5c77f186-414f-8b48-8231-17f9264e2048", "ParentIngestManifestAssetId" : "nb:maid:UUID:beed8531-9a03-9043-b1d8-6a6d1044cdda", "IsEncrypted" : "true", "EncryptionScheme" : "StorageEncryption", "EncryptionVersion" : "1.0", "EncryptionKeyId" : "nb:kid:UUID:32e6efaf-5fba-4538-b115-9d1cefe43510" }
	
###将文件上载到 Blob 存储

可以使用任何能够将资产文件上载到 Blob 存储容器 URI（由 IngestManifest 的 BlobStorageUriForUpload 属性提供）的高速客户端应用程序。一个明显的高速上载服务就是[适用于 Azure 应用程序的点播 Aspera](https://datamarket.azure.com/application/2cdbc511-cb12-4715-9871-c7e7fbbb82a6)。

###监视批量引入进度

可以通过轮询 IngestManifest 的 Statistics 属性来监视 IngestManifest 的批量引入操作的进度。该属性为复杂类型，即 [IngestManifestStatistics](https://msdn.microsoft.com/zh-cn/library/azure/jj853027.aspx)。若要轮询 Statistics 属性，请提交一个传递 IngestManifest ID 的 HTTP GET 请求。
 

##创建用于加密的 ContentKey

如果你的资产将使用加密，则在创建资产文件之前，必须创建用于加密的 ContentKey。对于存储空间加密，应在请求正文中包括以下属性。
 
请求正文属性 | 说明
ID | 我们使用以下格式自行生成的 ContentKey ID：“nb:kid:UUID:<NEW GUID>”。
ContentKeyType | 这是此内容密钥的内容密钥类型（为整数）。我们为存储加密传递了值 1。
EncryptedContentKey | 我们创建一个新的内容密钥值，这是一个 256 位（32 字节）的值。此密钥使用存储加密 X.509 证书进行加密，该证书是我们通过执行 GetProtectionKeyId 和 GetProtectionKey 方法的 HTTP GET 请求从 Azure 媒体服务中检索到的。
ProtectionKeyId | 这是存储加密 X.509 证书的保护密钥 ID，用于加密内容密钥。
ProtectionKeyType | 这是用于加密内容密钥的保护密钥的加密类型。对于我们的示例，此值为 StorageEncryption(1)。
Checksum | 内容密钥的 MD5 计算的校验和。它通过使用内容密钥加密内容 ID 计算得出。此示例代码演示了如何计算校验和。


**HTTP 响应**
	
	POST https://media.chinacloudapi.cn/api/ContentKeys HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.chinacloudapi.cn
	Content-Length: 572
	Expect: 100-continue
	
	{"Id" : "nb:kid:UUID:316d14d4-b603-4d90-b8db-0fede8aa48f8", "ContentKeyType" : 1, "EncryptedContentKey" : "Y4NPej7heOFa2vsd8ZEOcjjpu/qOq3RJ6GRfxa8CCwtAM83d6J2mKOeQFUmMyVXUSsBCCOdufmieTKi+hOUtNAbyNM4lY4AXI537b9GaY8oSeje0NGU8+QCOuf7jGdRac5B9uIk7WwD76RAJnqyep6U/OdvQV4RLvvZ9w7nO4bY8RHaUaLxC2u4aIRRaZtLu5rm8GKBPy87OzQVXNgnLM01I8s3Z4wJ3i7jXqkknDy4VkIyLBSQvIvUzxYHeNdMVWDmS+jPN9ScVmolUwGzH1A23td8UWFHOjTjXHLjNm5Yq+7MIOoaxeMlKPYXRFKofRY8Qh5o5tqvycSAJ9KUqfg==", "ProtectionKeyId" : "7D9BB04D9D0A4A24800CADBFEF232689E048F69C", "ProtectionKeyType" : 1, "Checksum" : "TfXtjCIlq1Y=" }

### 将 ContentKey 链接到资产

ContentKey 通过发送 HTTP POST 请求关联到一个或多个资产。以下请求是一个示例，说明了如何按 ID 将示例 ContentKey 链接到示例资产。

**HTTP 响应**
	
	POST https://media.chinacloudapi.cn/API/Assets('nb:cid:UUID:b3023475-09b4-4647-9d6d-6fc242822e68')/$links/ContentKeys HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.chinacloudapi.cn
	Content-Length: 113
	Expect: 100-continue
	
	{ "uri": "https://media.chinacloudapi.cn/api/ContentKeys('nb%3Akid%3AUUID%3A32e6efaf-5fba-4538-b115-9d1cefe43510')"}

**HTTP 响应**

	POST https://media.chinacloudapi.cn/API/IngestManifests(('nb:mid:UUID:5c77f186-414f-8b48-8231-17f9264e2048') HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=070500D0-F35C-4A5A-9249-485BBF4EC70B&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1334275521&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=GxdBb%2fmEyN7iHdNxbawawHRftLhPFFqxX1JZckuv3hY%3d
	Host: media.chinacloudapi.cn

 
[How to Get a Media Processor]: /documentation/articles/media-services-get-media-processor/
 
<!---HONumber=Mooncake_0620_2016-->