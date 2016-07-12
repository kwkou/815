<properties 
	pageTitle="如何从媒体服务传送流内容" 
	description="了解如何创建用于生成流 URL 的定位符。代码使用 REST API。" 
	authors="Juliako" 
	manager="dwrede" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016"  
	wacn.date="06/27/2016"/>


#如何：传送流内容

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-deliver-streaming-content/)
- [REST](/documentation/articles/media-services-rest-deliver-streaming-content/)
- [门户](/documentation/articles/media-services-manage-content/#publish)

##概述


你可以通过创建 OnDemand 流式处理定位符并生成流 URL 来流式传输自适应比特率 MP4 集。[对资产进行编码](/documentation/articles/media-services-rest-encode-asset/)主题说明了如何编码成自适应比特率 MP4 集。如果内容已加密，则在创建定位符之前配置资产传送策略（如[本](/documentation/articles/media-services-rest-configure-asset-delivery-policy/)主题中所述）。

你也可以使用 OnDemand 流式处理定位符生成指向可渐进式下载的 MP4 文件的 URL。

本主题说明如何创建 OnDemand 流式处理定位符，以发布资产及生成平滑流、MPEG DASH 和 HLS 流 URL。此外，还将示范如何生成渐进式下载 URL。

[以下](#types)部分显示了其值将在 REST 调用中使用的枚举类型。
  
##创建 OnDemand 流式处理定位符

若要创建 OnDemand 流式处理定位符并获取 URL，你需要执行以下操作：


   1. 如果内容已加密，则定义访问策略。
   2. 创建 OnDemand 流式处理定位符。
   3. 如果你想要流式处理，请获取资产中的流式处理清单文件 (.ism)。 
   		
	如果你想要渐进式下载，请获取资产中的 MP4 文件名。 
   4. 生成清单文件或 MP4 文件的 URL。 
   5. 请注意，你无法使用包含写入或删除权限的 AccessPolicy 创建流式处理定位符。


###创建访问策略

请求：
		
	POST https://media.chinacloudapi.cn/api/AccessPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstest1&urn%3aSubscriptionId=zbbef702-e769-2233-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1424263184&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=NWE%2f986Hr5lZTzVGKtC%2ftzHm9n6U%2fxpTFULItxKUGC4%3d
	x-ms-version: 2.11
	x-ms-client-request-id: 6bcfd511-a561-448d-a022-a319a89ecffa
	Host: media.chinacloudapi.cn
	Content-Length: 68
	
	{"Name":"access policy","DurationInMinutes":43200.0,"Permissions":1}
	
响应：
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 311
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https:/media.chinacloudapi.cn/api/AccessPolicies('nb%3Apid%3AUUID%3A69c80d98-7830-407f-a9af-e25f4b0d3e5f')
	Server: Microsoft-IIS/8.5
	request-id: a877528a-bdb4-4414-9862-273f8e64f882
	x-ms-request-id: a877528a-bdb4-4414-9862-273f8e64f882
	x-ms-client-request-id: 6bcfd511-a561-448d-a022-a319a89ecffa
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Wed, 18 Feb 2015 06:52:09 GMT
	
	{"odata.metadata":"https://media.chinacloudapi.cn/api/$metadata#AccessPolicies/@Element","Id":"nb:pid:UUID:69c80d98-7830-407f-a9af-e25f4b0d3e5f","Created":"2015-02-18T06:52:09.8862191Z","LastModified":"2015-02-18T06:52:09.8862191Z","Name":"access policy","DurationInMinutes":43200.0,"Permissions":1}

###创建 OnDemand 流式处理定位符

创建指定资产和资产策略的定位符。

请求：
	
	POST https://media.chinacloudapi.cn/api/Locators HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amstest1&urn%3aSubscriptionId=zbbef702-e769-2233-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1424263184&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=NWE%2f986Hr5lZTzVGKtC%2ftzHm9n6U%2fxpTFULItxKUGC4%3d
	x-ms-version: 2.11
	x-ms-client-request-id: ac159492-9a0c-40c3-aacc-551b1b4c5f62
	Host: media.chinacloudapi.cn
	Content-Length: 181
	
	{"AccessPolicyId":"nb:pid:UUID:1480030d-c481-430a-9687-535c6a5cb272","AssetId":"nb:cid:UUID:cc1e445d-1500-80bd-538e-f1e4b71b465e","StartTime":"2015-02-18T06:34:47.267872Z","Type":2}

响应：
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 637
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://media.chinacloudapi.cn/api/Locators('nb%3Alid%3AUUID%3Abe245661-2bbd-4fc6-b14f-9cf9a1492e5e')
	Server: Microsoft-IIS/8.5
	request-id: 5bd5864a-0afd-44c0-a67a-4044a2c9043b
	x-ms-request-id: 5bd5864a-0afd-44c0-a67a-4044a2c9043b
	x-ms-client-request-id: ac159492-9a0c-40c3-aacc-551b1b4c5f62
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Wed, 18 Feb 2015 06:58:37 GMT
	
	{"odata.metadata":"https://media.chinacloudapi.cn/api/$metadata#Locators/@Element","Id":"nb:lid:UUID:be245661-2bbd-4fc6-b14f-9cf9a1492e5e","ExpirationDateTime":"2015-03-20T06:34:47.267872+00:00","Type":2,"Path":"http://amstest1.streaming.mediaservices.chinacloudapi.cn/be245661-2bbd-4fc6-b14f-9cf9a1492e5e/","BaseUri":"http://amstest1.streaming.mediaservices.chinacloudapi.cn","ContentAccessComponent":"be245661-2bbd-4fc6-b14f-9cf9a1492e5e","AccessPolicyId":"nb:pid:UUID:1480030d-c481-430a-9687-535c6a5cb272","AssetId":"nb:cid:UUID:cc1e445d-1500-80bd-538e-f1e4b71b465e","StartTime":"2015-02-18T06:34:47.267872+00:00","Name":null}

###生成流 URL

使用创建定位符后返回的**路径**值生成平滑流式处理、HLS 和 MPEG DASH URL。

平滑流式处理：**路径** + 清单文件名 +“/manifest”

示例：

	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest

HLS：**路径** + 清单文件名 +“/manifest(format=m3u8-aapl)”

示例：

	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=m3u8-aapl)


DASH：**路径** + 清单文件名 +“/manifest(format=mpd-time-csf)”


示例：

	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=mpd-time-csf)


###生成渐进式下载 URL

使用创建定位符后返回的**路径**值生成渐进式下载 URL。

URL：**路径** + 资产文件 mp4 名称

示例：

	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4

##<a id="types"></a>枚举类型

    [Flags]
    public enum AccessPermissions
    {
        None = 0,
        Read = 1,
        Write = 2,
        Delete = 4,
        List = 8,
    }

    public enum LocatorType
    {
        None = 0,
        Sas = 1,
        OnDemandOrigin = 2,
    }


##另请参阅

[配置资产传送策略](/documentation/articles/media-services-rest-configure-asset-delivery-policy/)
<!---HONumber=Mooncake_0620_2016-->