<properties 
	pageTitle="使用媒体服务 REST API 配置资产传送策略" 
	description="本主题说明如何使用媒体服务 REST API 配置不同的资产传送策略。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="03/14/2016"  
	wacn.date="04/05/2016"/>

#如何：配置资产传送策略

[AZURE.INCLUDE [media-services-selector-asset-delivery-policy](../includes/media-services-selector-asset-delivery-policy.md)]

如果你打算传送动态加密的资产，媒体服务内容传送工作流中的步骤之一是为资产配置传送策略。资产传送策略告知媒体服务你希望如何传送资产：应该将资产动态打包成哪种流式处理协议（例如 MPEG DASH、HLS、平滑流或全部），是否要动态加密资产以及如何加密（信封或常用加密）。

本主题介绍为何以及如何创建和配置资产传送策略。

>[AZURE.NOTE]若要使用动态打包和动态加密，必须确保至少有一个缩放单位（也称为流式处理单位）。有关详细信息，请参阅[如何缩放媒体服务](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints)。
>
>此外，你的资产必须包含一组自适应比特率 MP4 或自适应比特率平滑流文件。

可以将不同的策略应用到同一个资产。例如，可以将 PlayReady 加密应用到平滑流，将 AES 信封应用到 MPEG DASH 和 HLS。将阻止流式处理传送策略中未定义的任何协议（例如，添加仅将 HLS 指定为协议的单个策略）。如果你根本没有定义任何传送策略，则情况不是这样。此时，将允许所有明文形式的协议。

如果要传送存储加密资产，则必须配置资产的传送策略。在流式传输资产之前，流式处理服务器会删除存储加密，然后再使用指定的传送策略流式传输你的内容。例如，若要传送使用高级加密标准 (AES) 信封加密密钥加密的资产，请将策略类型设为 **DynamicEnvelopeEncryption**。若要删除存储加密并以明文的形式流式传输资产，请将策略类型设为 **NoDynamicEncryption**。下面是演示如何配置这些策略类型的示例。

根据配置资产传送策略的方式，你可以动态打包、动态加密和流式传输以下流式处理协议：平滑流式处理、HLS、MPEG DASH 和 HDS 流。

以下列表显示了用于流式传输平滑流、HLS DASH 和 HDS 的格式。

平滑流：

	{streaming endpoint name-media services account name}.streaming.mediaservices.chinacloudapi.cn/{locator ID}/{filename}.ism/Manifest

HLS：

	{streaming endpoint name-media services account name}.streaming.mediaservices.chinacloudapi.cn/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl)

MPEG DASH

	{streaming endpoint name-media services account name}.streaming.mediaservices.chinacloudapi.cn/{locator ID}/{filename}.ism/Manifest(format=mpd-time-csf) 

HDS

	{streaming endpoint name-media services account name}.streaming.mediaservices.chinacloudapi.cn/{locator ID}/{filename}.ism/Manifest(format=f4m-f4f)

有关如何发布资产和生成流 URL 的说明，请参阅[生成流 URL](/documentation/articles/media-services-deliver-streaming-content/)。


##注意事项

- 如果某个资产存在 OnDemand（流式处理）定位符，则不能与该资产关联的 AssetDeliveryPolicy。在删除策略之前，建议先从资产中删除该策略。
- 如果未设置资产传送策略，则无法在存储加密的资产上创建流式处理定位符。如果资产未经过存储加密，则即使未设置资产传送策略，系统也可让你顺利地创建定位符和流式处理资产。
- 可将多个资产传送策略关联到单个资产，但只能指定一种方法来处理给定的 AssetDeliveryProtocol。也就是说，如果你尝试链接两个指定 AssetDeliveryProtocol.SmoothStreaming 协议的传送策略，则会导致出错，因为当客户端发出平滑流请求时，系统不知道你要应用哪个策略。  
- 如果你的资产包含现有的流式处理定位符，则不能将新策略链接到该资产、取消现有策略与资产的链接，或者更新与该资产关联的传送策略。必须先删除流式传输定位符，再调整策略，然后重新创建流式传输定位符。在重新创建流式处理定位符时可以使用同一个 locatorId，但应确保该操作不会导致客户端出现问题，因为内容可能已被来源或下游 CDN 缓存。  
 
>[AZURE.NOTE] 使用媒体服务 REST API 时，需注意以下事项：
>
>访问媒体服务中的实体时，必须在 HTTP 请求中设置特定标头字段和值。有关详细信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use/)。

>在成功连接到 https://media.chinacloudapi.cn 之后，你将接收到指定另一个媒体服务 URI 的 301 重定向。必须按[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect_programmatically/)中所述对新的 URI 执行后续调用。


##清除资产传送策略 

###<a id="create_asset_delivery_policy"></a>创建资产传送策略
以下 HTTP 请求将创建一个资产传送策略，该策略指定不要应用动态加密，而使用以下任何协议传送流：MPEG DASH、HLS 和平滑流式处理协议。

有关创建 AssetDeliveryPolicy 时可以指定哪些值的信息，请参阅[定义 AssetDeliveryPolicy 时使用的类型](#types)部分。


请求：
	  
	POST https://media.chinacloudapi.cn/api/AssetDeliveryPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amsaccount1&urn%3aSubscriptionId=zbbef702-e769-2233-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423397827&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=Szo6lbJAvL3dyecAeVmyAnzv3mGzfUNClR5shk9Ivbk%3d
	x-ms-version: 2.11
	x-ms-client-request-id: 4651882c-d7ad-4d5e-86ab-f07f47dcb41e
	Host: media.chinacloudapi.cn
	
	{"Name":"Clear Policy",
	"AssetDeliveryProtocol":7,
	"AssetDeliveryPolicyType":2,
	"AssetDeliveryConfiguration":null}

响应：
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 363
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://media.chinacloudapi.cn/api/AssetDeliveryPolicies('nb%3Aadpid%3AUUID%3A92b0f6ba-3c9f-49b6-a5fa-2a8703b04ecd')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: 4651882c-d7ad-4d5e-86ab-f07f47dcb41e
	request-id: 6aedbf93-4bc2-4586-8845-fd45590136af
	x-ms-request-id: 6aedbf93-4bc2-4586-8845-fd45590136af
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Sun, 08 Feb 2015 06:21:27 GMT
	
	{"odata.metadata":"https://media.chinacloudapi.cn/api/$metadata#AssetDeliveryPolicies/@Element",
	"Id":"nb:adpid:UUID:92b0f6ba-3c9f-49b6-a5fa-2a8703b04ecd",
	"Name":"Clear Policy",
	"AssetDeliveryProtocol":7,
	"AssetDeliveryPolicyType":2,
	"AssetDeliveryConfiguration":null,
	"Created":"2015-02-08T06:21:27.6908329Z",
	"LastModified":"2015-02-08T06:21:27.6908329Z"}
	
###<a id="link_asset_with_asset_delivery_policy"></a>将资产与资产传送策略相链接

以下 HTTP 请求将指定的资产链接到资产传送策略。

请求：

	POST https://media.chinacloudapi.cn/api/Assets('nb%3Acid%3AUUID%3A86933344-9539-4d0c-be7d-f842458693e0')/$links/DeliveryPolicies HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Content-Type: application/json
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amsaccount1&urn%3aSubscriptionId=zbbef702-e769-3344-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423397827&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=Szo6lbJAvL3dyecAeVmyAnzv3mGzfUNClR5shk9Ivbk%3d
	x-ms-version: 2.11
	x-ms-client-request-id: 56d2763f-6e72-419d-ba3c-685f6db97e81
	Host: media.chinacloudapi.cn
	
	{"uri":"https://media.chinacloudapi.cn/api/AssetDeliveryPolicies('nb%3Aadpid%3AUUID%3A92b0f6ba-3c9f-49b6-a5fa-2a8703b04ecd')"}

响应：

	HTTP/1.1 204 No Content


##DynamicEnvelopeEncryption 资产传送策略 

###创建 EnvelopeEncryption 类型的内容密钥，并将其链接到资产

在指定 DynamicEnvelopeEncryption 传送策略时，需要确保将你的资产链接到 EnvelopeEncryption 类型的内容密钥。有关详细信息，请参阅：[创建内容密钥](/documentation/articles/media-services-rest-create-contentkey/)。


###<a id="get_delivery_url"></a>获取传送 URL

获取上一步创建的内容密钥的指定传送方法的传送 URL。客户端使用返回的 URL 来请求 AES 密钥或 PlayReady 许可证，以播放受保护内容。

指定要在 HTTP 请求正文中获取的 URL 类型。如果要使用 PlayReady 保护内容，请通过用 1 表示 keyDeliveryType 来请求媒体服务 PlayReady 许可证：{"keyDeliveryType":1}。如果要使用信封加密来保护内容，请为 keyDeliveryType 指定 2，以请求密钥获取 URL：{"keyDeliveryType":2}。

请求：
	
	POST https://media.chinacloudapi.cn/api/ContentKeys('nb:kid:UUID:dc88f996-2859-4cf7-a279-c52a9d6b2f04')/GetKeyDeliveryUrl HTTP/1.1
	Content-Type: application/json
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amsaccount1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423452029&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=IEXV06e3drSIN5naFRBdhJZCbfEqQbFZsGSIGmawhEo%3d
	x-ms-version: 2.11
	x-ms-client-request-id: 569d4b7c-a446-4edc-b77c-9fb686083dd8
	Host: media.chinacloudapi.cn
	Content-Length: 21
	
	{"keyDeliveryType":2}

响应：
	
	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Content-Length: 198
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: 569d4b7c-a446-4edc-b77c-9fb686083dd8
	request-id: d26f65d2-fe65-4136-8fcf-31545be68377
	x-ms-request-id: d26f65d2-fe65-4136-8fcf-31545be68377
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Sun, 08 Feb 2015 21:42:30 GMT
	
	{"odata.metadata":"media.chinacloudapi.cn/api/$metadata#Edm.String","value":"https://amsaccount1.keydelivery.mediaservices.chinacloudapi.cn/?KID=dc88f996-2859-4cf7-a279-c52a9d6b2f04"}


###创建资产传送策略

以下 HTTP 请求将创建 **AssetDeliveryPolicy**，该策略配置为将动态信封加密 (**DynamicEnvelopeEncryption**) 应用到 **HLS** 协议（在本示例中，已阻止流式处理其他协议）。


有关创建 AssetDeliveryPolicy 时可以指定哪些值的信息，请参阅[定义 AssetDeliveryPolicy 时使用的类型](#types)部分。

请求：

	POST https://media.chinacloudapi.cn/api/AssetDeliveryPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	User-Agent: Microsoft ADO.NET Data Services
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amsaccount1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423480651&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=T2FG3tIV0e2ETzxQ6RDWxWAsAzuy3ez2ruXPhrBe62Y%3d
	x-ms-version: 2.11
	x-ms-client-request-id: fff319f6-71dd-4f6c-af27-b675c0066fa7
	Host: media.chinacloudapi.cn
	
	{"Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":4,"AssetDeliveryPolicyType":3,"AssetDeliveryConfiguration":"[{"Key":2,"Value":"https:\\/\\/amsaccount1.keydelivery.mediaservices.chinacloudapi.cn\\/"}]"}

	
响应：
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 460
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: media.chinacloudapi.cn/api/AssetDeliveryPolicies('nb%3Aadpid%3AUUID%3Aec9b994e-672c-4a5b-8490-a464eeb7964b')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: fff319f6-71dd-4f6c-af27-b675c0066fa7
	request-id: c2a1ac0e-9644-474f-b38f-b9541c3a7c5f
	x-ms-request-id: c2a1ac0e-9644-474f-b38f-b9541c3a7c5f
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Mon, 09 Feb 2015 05:24:38 GMT
	
	{"odata.metadata":"media.chinacloudapi.cn/api/$metadata#AssetDeliveryPolicies/@Element","Id":"nb:adpid:UUID:ec9b994e-672c-4a5b-8490-a464eeb7964b","Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":4,"AssetDeliveryPolicyType":3,"AssetDeliveryConfiguration":"[{"Key":2,"Value":"https:\\/\\/amsaccount1.keydelivery.mediaservices.chinacloudapi.cn\\/"}]","Created":"2015-02-09T05:24:38.9167436Z","LastModified":"2015-02-09T05:24:38.9167436Z"}


###将资产与资产传送策略相链接

请参阅[将资产与资产传送策略相链接](#link_asset_with_asset_delivery_policy)

##DynamicCommonEncryption 资产传送策略 

###创建 CommonEncryption 类型的内容密钥，并将其链接到资产

在指定 DynamicCommonEncryption 传送策略时，需要确保将你的资产链接到 CommonEncryption 类型的内容密钥。有关详细信息，请参阅：[创建内容密钥](/documentation/articles/media-services-rest-create-contentkey/)。


###获取传送 URL

获取上一步创建的内容密钥的 PlayReady 传送方法的传送 URL。客户端使用返回的 URL 来请求 PlayReady 许可证，以播放受保护内容。有关详细信息，请参阅[获取传送 URL](#get_delivery_url)。

###创建资产传送策略

以下 HTTP 请求将创建 **AssetDeliveryPolicy**，该策略配置为将动态通用加密 (**DynamicCommonEncryption**) 应用到**平滑流式处理**协议（在本示例中，已阻止流式处理其他协议）。

有关创建 AssetDeliveryPolicy 时可以指定哪些值的信息，请参阅[定义 AssetDeliveryPolicy 时使用的类型](#types)部分。


请求：

	POST https://media.chinacloudapi.cn/api/AssetDeliveryPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	User-Agent: Microsoft ADO.NET Data Services
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=amsaccount1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423480651&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=T2FG3tIV0e2ETzxQ6RDWxWAsAzuy3ez2ruXPhrBe62Y%3d
	x-ms-version: 2.11
	x-ms-client-request-id: fff319f6-71dd-4f6c-af27-b675c0066fa7
	Host: media.chinacloudapi.cn
	
	{"Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":1,"AssetDeliveryPolicyType":4,"AssetDeliveryConfiguration":"[{"Key":2,"Value":"https:\\/\\/amsaccount1.keydelivery.mediaservices.chinacloudapi.cn\/PlayReady\/"}]"}


若要使用 Widevine DRM 保护你的内容，请更新 AssetDeliveryConfiguration 值以使用 WidevineLicenseAcquisitionUrl（其值为 7），并指定许可证交付服务的 URL。你可以通过以下 AMS 合作伙伴来交付 Widevine 许可证：[EZDRM](http://ezdrm.com/)、[castLabs](http://castlabs.com/company/partners/azure/)

例如：
 
	
	{"Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":2,"AssetDeliveryPolicyType":4,"AssetDeliveryConfiguration":"[{"Key":7,"Value":"https:\\/\\/example.net\/WidevineLicenseAcquisition\/"}]"}

>[AZURE.NOTE]使用 Widevine 加密时，只能使用 DASH 传送。请确保在资产传送协议中指定 DASH (2)。
  
###将资产与资产传送策略相链接

请参阅[将资产与资产传送策略相链接](#link_asset_with_asset_delivery_policy)


##<a id="types"></a>定义 AssetDeliveryPolicy 时使用的类型

###AssetDeliveryProtocol 

    /// <summary>
    /// Delivery protocol for an asset delivery policy.
    /// </summary>
    [Flags]
    public enum AssetDeliveryProtocol
    {
        /// <summary>
        /// No protocols.
        /// </summary>
        None = 0x0,

        /// <summary>
        /// Smooth streaming protocol.
        /// </summary>
        SmoothStreaming = 0x1,

        /// <summary>
        /// MPEG Dynamic Adaptive Streaming over HTTP (DASH)
        /// </summary>
        Dash = 0x2,

        /// <summary>
        /// Apple HTTP Live Streaming protocol.
        /// </summary>
        HLS = 0x4,

        /// <summary>
        /// Adobe HTTP Dynamic Streaming (HDS)
        /// </summary>
        Hds = 0x8,

        /// <summary>
        /// Include all protocols.
        /// </summary>
        All = 0xFFFF
    }

###AssetDeliveryPolicyType

    /// <summary>
    /// Policy type for dynamic encryption of assets.
    /// </summary>
    public enum AssetDeliveryPolicyType
    {
        /// <summary>
        /// Delivery Policy Type not set.  An invalid value.
        /// </summary>
        None,

        /// <summary>
        /// The Asset should not be delivered via this AssetDeliveryProtocol. 
        /// </summary>
        Blocked, 

        /// <summary>
        /// Do not apply dynamic encryption to the asset.
        /// </summary>
        /// 
        NoDynamicEncryption,  

        /// <summary>
        /// Apply Dynamic Envelope encryption.
        /// </summary>
        DynamicEnvelopeEncryption,

        /// <summary>
        /// Apply Dynamic Common encryption.
        /// </summary>
        DynamicCommonEncryption
        }

###ContentKeyDeliveryType


    /// <summary>
    /// Delivery method of the content key to the client.
    ///
    </summary>
    public enum ContentKeyDeliveryType
    {
        /// <summary>
        /// None.
        ///
        </summary>
        None = 0,

        /// <summary>
        /// Use PlayReady License acquistion protocol
        ///
        </summary>
        PlayReadyLicense = 1,

        /// <summary>
        /// Use MPEG Baseline HTTP key protocol.
        ///
        </summary>
        BaselineHttp = 2,

        /// <summary>
        /// Use Widevine License acquistion protocol
        ///
        </summary>
        Widevine = 3

    }


###AssetDeliveryPolicyConfigurationKey

    /// <summary>
    /// Keys used to get specific configuration for an asset delivery policy.
    /// </summary>

    public enum AssetDeliveryPolicyConfigurationKey
    {
        /// <summary>
        /// No policies.
        /// </summary>
        None,

        /// <summary>
        /// Exact Envelope key URL.
        /// </summary>
        EnvelopeKeyAcquisitionUrl,

        /// <summary>
        /// Base key url that will have KID=<Guid> appended for Envelope.
        /// </summary>
        EnvelopeBaseKeyAcquisitionUrl,
        
        /// <summary>
        /// The initialization vector to use for envelope encryption in Base64 format.
        /// </summary>
        EnvelopeEncryptionIVAsBase64,

        /// <summary>
        /// The PlayReady License Acquisition Url to use for common encryption.
        /// </summary>
        PlayReadyLicenseAcquisitionUrl,

        /// <summary>
        /// The PlayReady Custom Attributes to add to the PlayReady Content Header
        /// </summary>
        PlayReadyCustomAttributes,

        /// <summary>
        /// The initialization vector to use for envelope encryption.
        /// </summary>
        EnvelopeEncryptionIV,

        /// <summary>
        /// Widevine DRM acquisition url
        /// </summary>
        WidevineLicenseAcquisitionUrl
    }



<!---HONumber=Mooncake_0328_2016-->