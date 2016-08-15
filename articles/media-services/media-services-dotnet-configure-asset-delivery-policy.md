<properties 
	pageTitle="使用 .NET SDK 配置资产传送策略" 
	description="本主题说明如何通过 Azure 媒体服务 .NET SDK 配置不同的资产传送策略。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako,Mingfeiy" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
  	ms.date="03/18/2016"
	wacn.date="04/05/2016"/>

#使用 .NET SDK 配置资产传送策略
[AZURE.INCLUDE [media-services-selector-asset-delivery-policy](../includes/media-services-selector-asset-delivery-policy.md)]

##概述

如果你打算传送加密资产，媒体服务内容传送工作流中的步骤之一是为资产配置传送策略。资产传送策略告知媒体服务你希望如何传送资产：应该将资产动态打包成哪种流式处理协议（例如 MPEG DASH、HLS、平滑流或全部），是否要动态加密资产以及如何加密（信封或常用加密）。

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
- 如果你的资产包含现有的流式处理定位符，则不能将新策略链接到该资产（可以取消现有策略与资产的链接，或者更新与该资产关联的传送策略）。必须先删除流式传输定位符，再调整策略，然后重新创建流式传输定位符。在重新创建流式传输定位符时可以使用同一个 locatorId，但应确保该操作不会导致客户端出现问题，因为内容可能已被来源或下游 CDN 缓存。  


##清除资产传送策略 

以下 **ConfigureClearAssetDeliveryPolicy** 方法会指定不应用动态加密，而是使用以下任一协议传送流：MPEG DASH、HLS 和平滑流。你可能需要对存储加密资产应用此策略。
  
有关创建 AssetDeliveryPolicy 时可以指定哪些值的信息，请参阅[定义 AssetDeliveryPolicy 时使用的类型](#types)部分。

    static public void ConfigureClearAssetDeliveryPolicy(IAsset asset)
    {
        IAssetDeliveryPolicy policy =
            _context.AssetDeliveryPolicies.Create("Clear Policy",
            AssetDeliveryPolicyType.NoDynamicEncryption, 
            AssetDeliveryProtocol.HLS | AssetDeliveryProtocol.SmoothStreaming | AssetDeliveryProtocol.Dash, null);

        asset.DeliveryPolicies.Add(policy);
    }

##DynamicCommonEncryption 资产传送策略 


以下 **CreateAssetDeliveryPolicy** 方法将创建 **AssetDeliveryPolicy**，该策略配置为将动态常用加密 (**DynamicCommonEncryption**) 应用到平滑流式处理协议（将阻止流式处理其他协议）。该方法采用以下两种参数：**Asset**（要将传送策略应用到的资产）和 **IContentKey**（**CommonEncryption** 类型的内容密钥。有关详细信息，请参阅：[创建内容密钥](/documentation/articles/media-services-dotnet-create-contentkey/#common_contentkey)）。

有关创建 AssetDeliveryPolicy 时可以指定哪些值的信息，请参阅[定义 AssetDeliveryPolicy 时使用的类型](#types)部分。


    static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
    {
        Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense);

        Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
        {
            {AssetDeliveryPolicyConfigurationKey.PlayReadyLicenseAcquisitionUrl, acquisitionUrl.ToString()},
        };

        var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                "AssetDeliveryPolicy",
            AssetDeliveryPolicyType.DynamicCommonEncryption,
            AssetDeliveryProtocol.SmoothStreaming,
            assetDeliveryPolicyConfiguration);

        // Add AssetDelivery Policy to the asset
        asset.DeliveryPolicies.Add(assetDeliveryPolicy);

        Console.WriteLine();
        Console.WriteLine("Adding Asset Delivery Policy: " +
            assetDeliveryPolicy.AssetDeliveryPolicyType);
    }

Azure 媒体服务还允许你添加 Widevine 加密。以下示例演示将 PlayReady 和 Widevine 添加到资产传送策略。


    static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
    {
    	// Get the PlayReady license service URL.
        Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense);
        

        // GetKeyDeliveryUrl for Widevine attaches the KID to the URL.
        // For example: https://amsaccount1.keydelivery.mediaservices.chinacloudapi.cn/Widevine/?KID=268a6dcb-18c8-4648-8c95-f46429e4927c.  
        // The WidevineBaseLicenseAcquisitionUrl (used below) also tells Dynamaic Encryption 
        // to append /? KID =< keyId > to the end of the url when creating the manifest.
        // As a result Widevine license aquisition URL will have KID appended twice, 
        // so we need to remove the KID that in the URL when we call GetKeyDeliveryUrl.

        Uri widevineUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.Widevine);
        UriBuilder uriBuilder = new UriBuilder(widevineUrl);
        uriBuilder.Query = String.Empty;
        widevineUrl = uriBuilder.Uri;

        Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
        {
            {AssetDeliveryPolicyConfigurationKey.PlayReadyLicenseAcquisitionUrl, acquisitionUrl.ToString()},
            {AssetDeliveryPolicyConfigurationKey.WidevineLicenseAcquisitionUrl, widevineUrl.ToString()}

        };

        var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                "AssetDeliveryPolicy",
            AssetDeliveryPolicyType.DynamicCommonEncryption,
            AssetDeliveryProtocol.Dash,
            assetDeliveryPolicyConfiguration);


        // Add AssetDelivery Policy to the asset
        asset.DeliveryPolicies.Add(assetDeliveryPolicy);

    }

>[AZURE.NOTE]使用 Widevine 加密时，只能使用 DASH 传送。请确保在资产传送协议中指定 DASH。


##DynamicEnvelopeEncryption 资产传送策略 

以下 **CreateAssetDeliveryPolicy** 方法将创建 **AssetDeliveryPolicy**，该策略配置为将动态信封加密 (**DynamicEnvelopeEncryption**) 应用到 HLS 和 DASH 协议（将阻止流式处理的其他协议）。该方法采用以下两种参数：**Asset**（要将传送策略应用到的资产）和 **IContentKey**（**EnvelopeEncryption** 类型的内容密钥。有关详细信息，请参阅：[创建内容密钥](/documentation/articles/media-services-dotnet-create-contentkey/#envelope_contentkey)）。


有关创建 AssetDeliveryPolicy 时可以指定哪些值的信息，请参阅[定义 AssetDeliveryPolicy 时使用的类型](#types)部分。

    private static void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
    {
        
        //  Get the Key Delivery Base Url by removing the Query parameter.  The Dynamic Encryption service will
        //  automatically add the correct key identifier to the url when it generates the Envelope encrypted content
        //  manifest.  Omitting the IV will also cause the Dynamice Encryption service to generate a deterministic
        //  IV for the content automatically.  By using the EnvelopeBaseKeyAcquisitionUrl and omitting the IV, this
        //  allows the AssetDelivery policy to be reused by more than one asset.
        //
        Uri keyAcquisitionUri = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.BaselineHttp);
        UriBuilder uriBuilder = new UriBuilder(keyAcquisitionUri);
        uriBuilder.Query = String.Empty;
        keyAcquisitionUri = uriBuilder.Uri;

        // The following policy configuration specifies: 
        //   key url that will have KID=<Guid> appended to the envelope and
        //   the Initialization Vector (IV) to use for the envelope encryption.
        Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string> 
        {
            {AssetDeliveryPolicyConfigurationKey.EnvelopeBaseKeyAcquisitionUrl, keyAcquisitionUri.ToString()},
        };

        IAssetDeliveryPolicy assetDeliveryPolicy =
            _context.AssetDeliveryPolicies.Create(
                        "AssetDeliveryPolicy",
                        AssetDeliveryPolicyType.DynamicEnvelopeEncryption,
                        AssetDeliveryProtocol.HLS | AssetDeliveryProtocol.Dash,
                        assetDeliveryPolicyConfiguration);

        // Add AssetDelivery Policy to the asset
        asset.DeliveryPolicies.Add(assetDeliveryPolicy);

        Console.WriteLine();
        Console.WriteLine("Adding Asset Delivery Policy: " + assetDeliveryPolicy.AssetDeliveryPolicyType);
    }


##<a id="types"></a>定义 AssetDeliveryPolicy 时使用的类型

###<a id="AssetDeliveryProtocol"></a>AssetDeliveryProtocol 

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

###<a id="AssetDeliveryPolicyType"></a>AssetDeliveryPolicyType

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

###<a id="ContentKeyDeliveryType"></a>ContentKeyDeliveryType

    /// <summary>
    /// Delivery method of the content key to the client.
    /// </summary>
    public enum ContentKeyDeliveryType
    {
        /// <summary>
        /// None.
        /// </summary>
        None = 0,

        /// <summary>
        /// Use PlayReady License acquistion protocol
        /// </summary>
        PlayReadyLicense = 1,

        /// <summary>
        /// Use MPEG Baseline HTTP key protocol.
        /// </summary>
        BaselineHttp = 2,

        /// <summary>
        /// Use Widevine License acquistion protocol
        ///
        </summary>
        Widevine = 3

    }

###<a id="AssetDeliveryPolicyConfigurationKey"></a>AssetDeliveryPolicyConfigurationKey

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