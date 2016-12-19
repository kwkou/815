
<properties 
	pageTitle="使用 REST API 管理媒体服务实体 | Azure" 
	description="了解如何使用 REST API 管理媒体服务实体。" 
	authors="juliako" 
	manager="dwrede" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>


<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/26/2016" 
	wacn.date="12/16/2016" 
	ms.author="juliako"/>


#使用 REST API 管理媒体服务实体

> [AZURE.SELECTOR]
- [REST](/documentation/articles/media-services-rest-manage-entities/)
- [.NET](/documentation/articles/media-services-dotnet-manage-entities/)

Azure 媒体服务是一项以 OData v3 为基础的基于 REST 的服务。因此，你可以像在任何其他 OData 服务上一样添加、查询、更新和删除实体。适用时，将标注例外情况。有关 OData 的详细信息，请参阅[开放数据协议文档](http://www.odata.org/documentation/)。

- 添加实体
- 查询实体
- 枚举实体的大型集合
- 更新实体
- 删除实体

>[AZURE.NOTE] 使用媒体服务 REST API 时，需注意以下事项：
>
>访问媒体服务中的实体时，必须在 HTTP 请求中设置特定标头字段和值。有关详细信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use/)。

>请按照[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect-programmatically/)中所述对媒体服务 URI 执行后续调用。


##添加实体

媒体服务中的每个实体都通过 POST HTTP 请求添加到实体集（如资产）中。

以下示例说明了如何创建 AccessPolicy。

	POST https://wamsshaclus001rest-hs.chinacloudapp.cn/API/AccessPolicies HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1337067658&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=dithjGvlXR9HlyAf5DE99N5OCYkPAxsHIcsTSjm9%2fVE%3d
	Host: wamsshaclus001rest-hs.chinacloudapp.cn
	Content-Length: 74
	Expect: 100-continue
	
	{"Name": "DownloadPolicy", "DurationInMinutes" : "300", "Permissions" : 1}

 
##查询实体

查询和列出实体非常简单，仅涉及 GET HTTP 请求和可选的 OData 操作。以下示例会检索包含所有 MediaProcessor 实体的列表。

	GET https://wamsshaclus001rest-hs.chinacloudapp.cn/API/MediaProcessors HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1337078831&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=suFkxhvPWxQVMjOYelOJfYEWkyTWJCBc02pF0N7NghI%3d
	Host: wamsshaclus001rest-hs.chinacloudapp.cn

也可检索特定实体或与特定实体关联的所有实体集，如以下示例所示：

	GET https://wamsshaclus001rest-hs.chinacloudapp.cn/API/JobTemplates('nb:jtid:UUID:e81192f5-576f-b247-b781-70a790c20e7c') HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1336907474&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=OpuY0CeTylqFFcFaP4pKUVGesT4PGx4CP55zDf2zXnc%3d
	Host: wamsshaclus001rest-hs.chinacloudapp.cn

	GET https://wamsshaclus001rest-hs.chinacloudapp.cn/API/JobTemplates('nb:jtid:UUID:e81192f5-576f-b247-b781-70a790c20e7c')/TaskTemplates HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1336907474&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=OpuY0CeTylqFFcFaP4pKUVGesT4PGx4CP55zDf2zXnc%3d
	Host: wamsshaclus001rest-hs.chinacloudapp.cn

以下示例仅返回所有作业的 State 属性。

	GET https://wamsshaclus001rest-hs.chinacloudapp.cn/API/Jobs?$select=State HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1337078831&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=suFkxhvPWxQVMjOYelOJfYEWkyTWJCBc02pF0N7NghI%3d
	Host: wamsshaclus001rest-hs.chinacloudapp.cn

以下示例返回名为“SampleTemplate”的所有 JobTemplate。

	GET https://wamsshaclus001rest-hs.chinacloudapp.cn/API/JobTemplates?$filter=startswith(Name,%20'SampleTemplate') HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1337078831&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=suFkxhvPWxQVMjOYelOJfYEWkyTWJCBc02pF0N7NghI%3d
	Host: wamsshaclus001rest-hs.chinacloudapp.cn

>[AZURE.NOTE]媒体服务不支持 $expand 操作以及“LINQ 注意事项（WCF 数据服务）”中所述的不受支持的 LINQ 方法。

##<a name="enumerating-through-large-collections-of-entities"></a>枚举实体的大型集合

查询实体时，一次返回的实体数限制为 1000 个，因为公共 REST v2 将查询结果数限制为 1000 个。使用 **skip** 和 **top** 来枚举实体的大型集合。

以下示例说明如何使用 **skip** 和 **top** 来跳过前 2000 个作业并获取其后的 1000 个作业。

	GET https://wamsshaclus001rest-hs.chinacloudapp.cn/api/Jobs()?$skip=2000&$top=1000 HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1337078831&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=suFkxhvPWxQVMjOYelOJfYEWkyTWJCBc02pF0N7NghI%3d
	Host: wamsshaclus001rest-hs.chinacloudapp.cn

##更新实体

根据实体类型及其所处的状态，可以通过 PATCH、PUT 或 MERGE HTTP 请求更新该实体上的属性。有关这些操作的详细信息，请参阅 [PATCH/PUT/MERGE](https://msdn.microsoft.com/zh-cn/library/dd541276.aspx)。

以下代码示例演示如何更新资产实体上的 Name 属性。

	MERGE https://wamsshaclus001rest-hs.chinacloudapp.cn/API/Assets('nb:cid:UUID:80782407-3f87-4e60-a43e-5e4454232f60') HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1337083279&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=DMLQXWah4jO0icpfwyws5k%2b1aCDfz9KDGIGao20xk6g%3d
	Host: wamsshaclus001rest-hs.chinacloudapp.cn
	Content-Length: 21
	Expect: 100-continue
	
	{"Name" : "NewName" }

##删除实体

可以使用 DELETE HTTP 请求在媒体服务中删除实体。删除实体的顺序可能很重要，具体视实体而定。例如，资产等实体要求先撤消（或删除）引用该特定资产的所有定位符，然后再删除资产。

以下示例演示如何删除用于将文件上载到 blob 存储的定位符。

	DELETE https://wamsshaclus001rest-hs.chinacloudapp.cn/API/Locators('nb:lid:UUID:76dcc8e8-4230-463d-97b0-ce25c41b5c8d') HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=youraccountname&urn%3aSubscriptionId=2f84471d-b1ae-4e75-aa09-010f0fc0cf5b&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1337067658&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=dithjGvlXR9HlyAf5DE99N5OCYkPAxsHIcsTSjm9%2fVE%3d
	Host: wamsshaclus001rest-hs.chinacloudapp.cn
	Content-Length: 0

<!---HONumber=Mooncake_Quality_Review_1202_2016-->