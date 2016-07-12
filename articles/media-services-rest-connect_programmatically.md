<properties 
	pageTitle="使用 REST API 连接到媒体服务帐户" 
	description="本主题演示如何使用 REST API 连接到媒体服务。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="02/03/2016"  
	wacn.date="07/01/2016"/>

# 使用媒体服务 REST API 连接到媒体服务帐户

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-dotnet-connect_programmatically/)
- [REST](/documentation/articles/media-services-rest-connect_programmatically/)

本主题介绍如何在使用媒体服务 REST API 编程时获取与 Azure 媒体服务的编程连接。

访问 Azure 媒体服务时需要以下两项内容：由 Azure 访问控制服务 (ACS) 提供的访问令牌和媒体服务本身的 URI。在创建这些请求时，可以使用任何想要的方法，前提是在调用媒体服务时指定了正确的标头值，并且正确地传入了访问令牌。

以下步骤描述了在使用媒体服务 REST API 连接到媒体服务时运用的最常见工作流：

1. 获取访问令牌 
2. 连接到媒体服务 URI 

	>[AZURE.NOTE]上海 DC URI：https://wamsshaclus001rest-hs.chinacloudapp.cn/API/ ； 北京 DC URI： https://wamsbjbclus001rest-hs.chinacloudapp.cn/API/

3. 将后续 API 调用发布到相应的 URL。

##获取访问令牌

若要直接通过 REST API 访问媒体服务，请从 ACS 检索一个访问令牌，并在向该服务提出每个 HTTP 请求期间使用该令牌。此令牌类似于 ACS 基于 HTTP 请求标头中的访问声明提供的其他令牌，并且使用 OAuth v2 协议。在直接连接到媒体服务之前，你不需要满足任何其他先决条件。

以下示例显示了用于检索令牌的 HTTP 请求标头和正文。

**标头**：

	POST https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn/v2/OAuth2-13 HTTP/1.1
	Content-Type: application/x-www-form-urlencoded
	Host: wamsprodglobal001acs.accesscontrol.chinacloudapi.cn
	Content-Length: 120
	Expect: 100-continue
	Connection: Keep-Alive
	Accept: application/json

	
**正文**：

需要在此请求的正文中提供 client_id 和 client_secret 值；client_id 和 client_secret 分别对应于 AccountName 和 AccountKey 值。在你设置帐户时，媒体服务将提供这些值。

请注意，当用作访问令牌请求中的 client_secret 值时，你的媒体服务帐户的 AccountKey 必须进行 URL 编码（请参阅[百分号编码](http://tools.ietf.org/html/rfc3986#section-2.1)）。

	grant_type=client_credentials&client_id=ams_account_name&client_secret=URL_encoded_ams_account_key&scope=urn%3aWindowsAzureMediaServices


例如：

	grant_type=client_credentials&client_id=amstestaccount001&client_secret=wUNbKhNj07oqjqU3Ah9R9f4kqTJ9avPpfe6Pk3YZ7ng%3d&scope=urn%3aWindowsAzureMediaServices


以下示例显示了在响应正文中包含访问令牌的 HTTP 响应。

	HTTP/1.1 200 OK
	Cache-Control: no-cache, no-store
	Pragma: no-cache
	Content-Type: application/json; charset=utf-8
	Expires: -1
	request-id: a65150f5-2784-4a01-a4b7-33456187ad83
	X-Content-Type-Options: nosniff
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Thu, 15 Jan 2015 08:07:20 GMT
	Content-Length: 670
	
	{  
	   "token_type":"http://schemas.xmlsoap.org/ws/2009/11/swt-token-profile-1.0",
	   "access_token":"http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f19258-2233-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421330840&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=uf69n82KlqZmkJDNxhJkOxpyIpA2HDyeGUTtSnq1vlE%3d",
	   "expires_in":"21600",
	   "scope":"urn:WindowsAzureMediaServices"
	}
	

>[AZURE.NOTE]建议在外部存储中缓存“access_token”和“expires_in”值。以后可以从存储中检索令牌数据，并在你的媒体服务 REST API 调用中重新使用。这对于令牌可以在多个进程或多台计算机之间安全共享的方案尤其有用。

确保监视访问令牌的“expires_in”值，并在必要时使用新令牌更新你的 REST API 调用。

###连接到媒体服务 URI

请注意，请勿在请求中使用任何自动重定向/跟踪逻辑。HTTP 谓词和请求正文将不会转发到新 URI。用于上载和下载资产文件的根 URI 为 https://yourstorageaccount.blob.core.chinacloudapi.cn/ ，其中的存储帐户名为你在媒体服务帐户设置期间使用的同一帐户名。

以下示例演示了对上海 DC 媒体服务 URI 发出的 HTTP 请求 (https://wamsshaclus001rest-hs.chinacloudapp.cn/API/)。     

**HTTP 请求**：
			
	GET https://wamsshaclus001rest-hs.chinacloudapp.cn/API/ HTTP/1.1
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstestaccount001&urn%3aSubscriptionId=z7f19258-2233-4ca2-b1ae-193798e2c9d8&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1421500579&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=ElVWXOnMVggFQl%2ft9vhdcv1qH1n%2fE8l3hRef4zPmrzg%3d
	x-ms-version: 2.11
	Accept: application/json
	Host: wamsshaclus001rest-hs.chinacloudapp.cn


**HTTP 响应**：
	
	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Content-Length: 1250
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Server: Microsoft-IIS/8.5
	request-id: 5f52ea9d-177e-48fe-9709-24953d19f84a
	x-ms-request-id: 5f52ea9d-177e-48fe-9709-24953d19f84a
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Sat, 17 Jan 2015 07:44:52 GMT
	
	{"odata.metadata":"https://wamsshaclus001rest-hs.chinacloudapp.cn/api/$metadata","value":[{"name":"AccessPolicies","url":"AccessPolicies"},{"name":"Locators","url":"Locators"},{"name":"ContentKeys","url":"ContentKeys"},{"name":"ContentKeyAuthorizationPolicyOptions","url":"ContentKeyAuthorizationPolicyOptions"},{"name":"ContentKeyAuthorizationPolicies","url":"ContentKeyAuthorizationPolicies"},{"name":"Files","url":"Files"},{"name":"Assets","url":"Assets"},{"name":"AssetDeliveryPolicies","url":"AssetDeliveryPolicies"},{"name":"IngestManifestFiles","url":"IngestManifestFiles"},{"name":"IngestManifestAssets","url":"IngestManifestAssets"},{"name":"IngestManifests","url":"IngestManifests"},{"name":"StorageAccounts","url":"StorageAccounts"},{"name":"Tasks","url":"Tasks"},{"name":"NotificationEndPoints","url":"NotificationEndPoints"},{"name":"Jobs","url":"Jobs"},{"name":"TaskTemplates","url":"TaskTemplates"},{"name":"JobTemplates","url":"JobTemplates"},{"name":"MediaProcessors","url":"MediaProcessors"},{"name":"EncodingReservedUnitTypes","url":"EncodingReservedUnitTypes"},{"name":"Operations","url":"Operations"},{"name":"StreamingEndpoints","url":"StreamingEndpoints"},{"name":"Channels","url":"Channels"},{"name":"Programs","url":"Programs"}]}
	 



<!---HONumber=Mooncake_0321_2016-->