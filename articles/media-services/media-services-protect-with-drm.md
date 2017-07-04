<properties
    pageTitle="使用 PlayReady 动态通用加密 | Azure"
    description="Azure 媒体服务允许你传送受 Microsoft PlayReady DRM 保护的 MPEG-DASH 流、平滑流式处理流和 HTTP 实时流式处理 (HLS) 流。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="548d1a12-e2cb-45fe-9307-4ec0320567a2"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="03/16/2017"
    wacn.date="04/24/2017"
    ms.author="juliako"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="0ed5680a59ac884c112a9dfbce5ca561b7967deb"
    ms.lasthandoff="04/14/2017" />

# <a name="using-playready-dynamic-common-encryption"></a>使用 PlayReady 动态通用加密
> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-protect-with-drm/)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
- [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)

Azure 媒体服务允许你传送受 [Microsoft PlayReady DRM](https://www.microsoft.com/playready/overview/) 保护的 MPEG-DASH 流、平滑流式处理流和 HTTP 实时流式处理 (HLS) 流。 PlayReady 是按通用加密 (ISO/IEC 23001-7 CENC) 规范加密的。

媒体服务提供了用于传送 PlayReady DRM 许可证的服务。 媒体服务还提供用于配置所需权限和限制的 API，这样当用户播放受保护的内容时，PlayReady DRM 运行时便会强制实施这些权限和限制。 当用户请求受 DRM 保护的内容时，播放器应用程序将从 AMS 许可证服务请求许可证。 如果播放器已获授权，AMS 许可证服务将向播放器颁发许可证。 PlayReady 许可证包含客户端播放器用来对内容进行解密和流式传输的解密密钥。

媒体服务支持通过多种方式对发出密钥请求的用户进行授权。 内容密钥授权策略可能受到一种或多种授权限制：开放或令牌限制。 令牌限制策略必须附带由安全令牌服务 (STS) 颁发的令牌。 媒体服务支持采用[简单 Web 令牌](https://msdn.microsoft.com/zh-cn/library/gg185950.aspx#BKMK_2) (SWT) 格式和 [JSON Web 令牌](https://msdn.microsoft.com/zh-cn/library/gg185950.aspx#BKMK_3) (JWT) 格式的令牌。 有关详细信息，请参阅“配置内容密钥授权策略”。

为了充分利用动态加密，你的资产必须包含一组多码率 MP4 文件或多码率平滑流源文件。 你还需要为资产配置传送策略（在本主题后面部分介绍）。 然后，根据你在流 URL 中指定的格式，按需流式处理服务器将确保使用你选定的协议来传送流。 因此，你只需以单一存储格式存储文件并为其付费，然后媒体服务就会基于客户端的每个请求构建并提供相应的 HTTP 响应。

开发应用程序以传送受多个 DRM（例如 PlayReady）保护的媒体的开发人员可以参考本主题。 本主题介绍如何使用授权策略来配置 PlayReady 许可证传送服务，确保只有经过授权的客户端才能接收 PlayReady。 此外，还介绍如何通过 DASH 使用 PlayReady 进行动态加密。

>[AZURE.NOTE]
>创建 AMS 帐户后，会将一个处于“已停止”状态的**默认**流式处理终结点添加到帐户。 若要开始流式传输内容并利用动态打包和动态加密，要从中流式传输内容的流式处理终结点必须处于“正在运行”状态。 

## <a name="download-sample"></a>下载示例
可以从 [此处](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-drm)下载本文所述的示例。

## <a name="configuring-dynamic-common-encryption-and-drm-license-delivery-services"></a>配置动态通用加密和 DRM 许可证传送服务

下面是使用 PlayReady 保护资产时需要执行的常规步骤，这些步骤使用媒体服务许可证传送服务，也使用动态加密。

1. 创建资产并将文件上传到资产。
2. 将包含文件的资产编码为自适应比特率 MP4 集。
3. 创建内容密钥并将其与编码资产相关联。 在媒体服务中，内容密钥包含资产的加密密钥。
4. 配置内容密钥授权策略。 你必须配置内容密钥授权策略，客户端必须遵守该策略，才能将内容密钥传送到客户端。

    在创建内容密钥授权策略时，需要指定以下信息：传送方法 (PlayReady)、限制（开放或令牌），以及用于定义如何将密钥传送到客户端的密钥传送类型的具体信息（[PlayReady](/documentation/articles/media-services-playready-license-template-overview/) 许可证模板）。

5. 为资产配置传送策略。 传送策略配置包括：传送协议（例如 MPEG DASH、HLS、平滑流式处理或所有这些协议）、动态加密类型（例如常用加密）、PlayReady 许可证获取 URL。

    你可以将不同的策略应用到同一资产上的每个协议。 例如，可以将 PlayReady 加密应用到平滑流/DASH，将 AES 信封应用到 HLS。 将阻止流式处理传送策略中未定义的任何协议（例如，添加仅将 HLS 指定为协议的单个策略）。 如果你根本没有定义任何传送策略，则情况不是这样。 此时，将允许所有明文形式的协议。

6. 创建 OnDemand 定位符以获取流 URL。

你可以在主题末尾找到完整的 .NET 示例。

下图演示了上述工作流。 在图中，使用令牌进行了身份验证。

![使用 PlayReady 进行保护](./media/media-services-content-protection-overview/media-services-content-protection-with-drm.png)

本主题的余下部分提供了详细说明、代码示例和主题链接，向你演示如何完成上述任务。

## <a name="current-limitations"></a>当前限制
如果你添加或更新资产的传送策略，则必须删除关联的定位符（如果有）并创建新定位符。


## <a name="create-an-asset-and-upload-files-into-the-asset"></a>创建资产并将文件上载到资产
为了对视频进行管理、编码和流式处理，必须首先将内容上传到 Azure 媒体服务中。上传完成后，相关内容即安全地存储在云中供后续处理和流式处理。

有关详细信息，请参阅 [将文件上载到媒体服务帐户](/documentation/articles/media-services-dotnet-upload-files/)。

## <a name="encode-the-asset-containing-the-file-to-the-adaptive-bitrate-mp4-set"></a>将包含文件的资产编码为自适应比特率 MP4 集。
使用动态加密时，你只需创建包含一组多码率 MP4 文件或多码率平滑流源文件的资产。 然后，按需流式处理服务器会确保你以选定的协议按清单和分段请求中的指定格式接收流。 因此，你只需以单一存储格式存储文件并为其付费，然后媒体服务服务就会基于客户端的请求构建并提供相应响应。 有关详细信息，请参阅 [动态打包概述](/documentation/articles/media-services-dynamic-packaging-overview/) 主题。

有关如何编码的说明，请参阅[如何使用 Media Encoder Standard 对资产进行编码](/documentation/articles/media-services-dotnet-encode-with-media-encoder-standard/)。

## <a id="create_contentkey"></a>创建内容密钥并将其与编码资产相关联
在媒体服务中，内容密钥包含用于加密资产的密钥。

有关详细信息，请参阅 [创建内容密钥](/documentation/articles/media-services-dotnet-create-contentkey/)。

## <a id="configure_key_auth_policy"></a>配置内容密钥授权策略
媒体服务支持通过多种方式对发出密钥请求的用户进行身份验证。 你必须配置内容密钥授权策略，客户端（播放器）必须遵守该策略，才能将密钥传送到客户端。 内容密钥授权策略可能受到一种或多种授权限制：开放或令牌限制。

有关详细信息，请参阅 [配置内容密钥授权策略](/documentation/articles/media-services-dotnet-configure-content-key-auth-policy/#playready-dynamic-encryption)。

## <a id="configure_asset_delivery_policy"></a>配置资产传送策略
为资产配置传送策略。 资产传送策略配置包括：

* DRM 许可证获取 URL。
* 资产传送协议（例如 MPEG DASH、HLS、平滑流式处理或全部）。
* 动态加密类型（在本示例中为“常用加密”）。

有关详细信息，请参阅 [配置资产传送策略 ](/documentation/articles/media-services-rest-configure-asset-delivery-policy/)。

## <a id="create_locator"></a>创建 OnDemand 流定位符以获取流 URL
需要为用户提供平滑流、DASH 或 HLS 的流 URL。

> [AZURE.NOTE]
> 如果你添加或更新资产的传送策略，则必须删除现有定位符（如果有）并创建新定位符。
>
>

有关如何发布资产和生成流 URL 的说明，请参阅 [生成流 URL](/documentation/articles/media-services-deliver-streaming-content/)。

## <a name="get-a-test-token"></a>获取测试令牌
获取用于密钥授权策略的基于令牌限制的测试令牌。

    // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
    // back into a TokenRestrictionTemplate class instance.
    TokenRestrictionTemplate tokenTemplate =
        TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

    // Generate a test token based on the data in the given TokenRestrictionTemplate.
    //The GenerateTestToken method returns the token without the word “Bearer” in front
    //so you have to add it in front of the token string.
    string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate);
    Console.WriteLine("The authorization token is:\nBearer {0}", testToken);


你可以使用 [AMS Player](http://amsplayer.azurewebsites.net/azuremediaplayer.html) 来测试你的流。

## <a id="example"></a>示例
以下示例演示的功能已引入用于 .Net 的 Azure 媒体服务 SDK - 版本 3.5.2。 以下 Nuget 包命令用于安装该包：

    PM> Install-Package windowsazure.mediaservices -Version 3.5.2


1. 创建新的控制台项目。
2. 使用 NuGet 安装和添加 Azure 媒体服务 .NET SDK。
3. 添加附加引用：System.Configuration。
4. 添加包含帐户名称和密钥信息的配置文件：
	
		<?xml version="1.0" encoding="utf-8"?>
		<configuration>
		    <startup> 
		        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
		    </startup>
			  <appSettings>
			
			    <add key="MediaServicesAccountName" value="AccountName"/>
			    <add key="MediaServicesAccountKey" value="AccountKey"/>
			
			    <add key="Issuer" value="http://testacs.com"/>
			    <add key="Audience" value="urn:test"/>
			  </appSettings>
		</configuration>

7. 使用本部分中所示的代码覆盖 Program.cs 文件中的代码。

    >[AZURE.NOTE]
    >不同 AMS 策略的策略限制为 1,000,000 个（例如，对于定位器策略或 ContentKeyAuthorizationPolicy）。 如果始终使用相同的日期/访问权限，则应使用相同的策略 ID，例如，用于要长期就地保留的定位符的策略（非上传策略）。 有关详细信息，请参阅[此](/documentation/articles/media-services-dotnet-manage-entities/#limit-access-policies)主题。
	请务必将变量更新为指向输入文件所在的文件夹。
		
		using System;
		using System.Collections.Generic;
		using System.Configuration;
		using System.IO;
		using System.Linq;
		using System.Threading;
		using Microsoft.WindowsAzure.MediaServices.Client;
		using Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization;
		using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;
		using Newtonsoft.Json;
		
		namespace DynamicEncryptionWithDRM
		{
		    class Program
		    {
		        // Read values from the App.config file.
		        private static readonly string _mediaServicesAccountName =
		            ConfigurationManager.AppSettings["MediaServicesAccountName"];
		        private static readonly string _mediaServicesAccountKey =
		            ConfigurationManager.AppSettings["MediaServicesAccountKey"];
		
				private static readonly String _defaultScope = "urn:WindowsAzureMediaServices";

				// Azure China uses a different API server and a different ACS Base Address from the Global.
				private static readonly String _chinaApiServerUrl = "https://wamsshaclus001rest-hs.chinacloudapp.cn/API/";
				private static readonly String _chinaAcsBaseAddressUrl = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn";

		        private static readonly Uri _sampleIssuer =
		            new Uri(ConfigurationManager.AppSettings["Issuer"]);
		        private static readonly Uri _sampleAudience =
		            new Uri(ConfigurationManager.AppSettings["Audience"]);
		
		        // Field for service context.
		        private static CloudMediaContext _context = null;
		        private static MediaServicesCredentials _cachedCredentials = null;
				private static Uri _apiServer = null;
		
		        private static readonly string _mediaFiles =
		            Path.GetFullPath(@"../..\Media");
		
		        private static readonly string _singleMP4File =
		            Path.Combine(_mediaFiles, @"BigBuckBunny.mp4");
		
		        static void Main(string[] args)
		        {
		            // Create and cache the Media Services credentials in a static class variable.
		            _cachedCredentials = new MediaServicesCredentials(
		                            _mediaServicesAccountName,
		                            _mediaServicesAccountKey,
									_defaultScope,
									_chinaAcsBaseAddressUrl);

					// Create the API server Uri
					_apiServer = new Uri(_chinaApiServerUrl);

		            // Used the cached credentials to create CloudMediaContext.
		            _context = new CloudMediaContext(_apiServer, _cachedCredentials);
		
		            bool tokenRestriction = false;
		            string tokenTemplateString = null;
		
		            IAsset asset = UploadFileAndCreateAsset(_singleMP4File);
		            Console.WriteLine("Uploaded asset: {0}", asset.Id);
		
		            IAsset encodedAsset = EncodeToAdaptiveBitrateMP4Set(asset);
		            Console.WriteLine("Encoded asset: {0}", encodedAsset.Id);
		
		            IContentKey key = CreateCommonTypeContentKey(encodedAsset);
		            Console.WriteLine("Created key {0} for the asset {1} ", key.Id, encodedAsset.Id);
		            Console.WriteLine("PlayReady License Key delivery URL: {0}", key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense));
		            Console.WriteLine();
		
		            if (tokenRestriction)
		                tokenTemplateString = AddTokenRestrictedAuthorizationPolicy(key);
		            else
		                AddOpenAuthorizationPolicy(key);
		
		            Console.WriteLine("Added authorization policy: {0}", key.AuthorizationPolicyId);
		            Console.WriteLine();
		
		            CreateAssetDeliveryPolicy(encodedAsset, key);
		            Console.WriteLine("Created asset delivery policy. \n");
		            Console.WriteLine();
		
		            if (tokenRestriction && !String.IsNullOrEmpty(tokenTemplateString))
		            {
		                // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
		                // back into a TokenRestrictionTemplate class instance.
		                TokenRestrictionTemplate tokenTemplate =
		                    TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);
		
		                // Generate a test token based on the the data in the given TokenRestrictionTemplate.
		                // Note, you need to pass the key id Guid because we specified 
		                // TokenClaim.ContentKeyIdentifierClaim in during the creation of TokenRestrictionTemplate.
		                Guid rawkey = EncryptionUtils.GetKeyIdAsGuid(key.Id);
		                string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate, null, rawkey, 
																				DateTime.UtcNow.AddDays(365));
		                Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
		                Console.WriteLine();
		            }
		
		            // You can use the http://amsplayer.azurewebsites.net/azuremediaplayer.html player to test streams.
		            // Note that DASH works on IE 11 (via PlayReady), Edge (via PlayReady).
		             
		            string url = GetStreamingOriginLocator(encodedAsset);
		            Console.WriteLine("Encrypted DASH URL: {0}/manifest(format=mpd-time-csf)", url);
		
		            Console.ReadLine();
		        }
		
		
		        static public IAsset UploadFileAndCreateAsset(string singleFilePath)
		        {
		            if (!File.Exists(singleFilePath))
		            {
		                Console.WriteLine("File does not exist.");
		                return null;
		            }
		
		            var assetName = Path.GetFileNameWithoutExtension(singleFilePath);
		            IAsset inputAsset = _context.Assets.Create(assetName, AssetCreationOptions.None);
		
		            var assetFile = inputAsset.AssetFiles.Create(Path.GetFileName(singleFilePath));
		
		            Console.WriteLine("Created assetFile {0}", assetFile.Name);
		
		            var policy = _context.AccessPolicies.Create(
		                                    assetName,
		                                    TimeSpan.FromDays(30),
		                                    AccessPermissions.Write | AccessPermissions.List);
		
		            var locator = _context.Locators.CreateLocator(LocatorType.Sas, inputAsset, policy);
		
		            Console.WriteLine("Upload {0}", assetFile.Name);
		
		            assetFile.Upload(singleFilePath);
		            Console.WriteLine("Done uploading {0}", assetFile.Name);
		
		            locator.Delete();
		            policy.Delete();
		
		            return inputAsset;
		        }
		
			    static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset inputAsset)
			    {
			        var encodingPreset = "H264 Multiple Bitrate 720p";
			
			        IJob job = _context.Jobs.Create(String.Format("Encoding into Mp4 {0} to {1}",
			                                inputAsset.Name,
			                                encodingPreset));
			
			        var mediaProcessors =
			            _context.MediaProcessors.Where(p => p.Name.Contains("Media Encoder Standard")).ToList();
			
			        var latestMediaProcessor =
			            mediaProcessors.OrderBy(mp => new Version(mp.Version)).LastOrDefault();
			
			        ITask encodeTask = job.Tasks.AddNew("Encoding", latestMediaProcessor, encodingPreset, TaskOptions.None);
			        encodeTask.InputAssets.Add(inputAsset);
			        encodeTask.OutputAssets.AddNew(String.Format("{0} as {1}", inputAsset.Name, encodingPreset), 	AssetCreationOptions.StorageEncrypted);
			
			        job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
			        job.Submit();
			        job.GetExecutionProgressTask(CancellationToken.None).Wait();
			
			        return job.OutputMediaAssets[0];
			    }
		
		
		        static public IContentKey CreateCommonTypeContentKey(IAsset asset)
		        {
		            
		            Guid keyId = Guid.NewGuid();
		            byte[] contentKey = GetRandomBuffer(16);
		
		            IContentKey key = _context.ContentKeys.Create(
		                                    keyId,
		                                    contentKey,
		                                    "ContentKey",
		                                    ContentKeyType.CommonEncryption);
		
		            // Associate the key with the asset.
		            asset.ContentKeys.Add(key);
		
		            return key;
		        }
		
		        static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
		        {
		
		            // Create ContentKeyAuthorizationPolicy with Open restrictions 
		            // and create authorization policy          
		
		            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
		            {
		                new ContentKeyAuthorizationPolicyRestriction
		                {
		                    Name = "Open",
		                    KeyRestrictionType = (int)ContentKeyRestrictionType.Open,
		                    Requirements = null
		                }
		            };
		
		            // Configure PlayReady license templates.
		            string PlayReadyLicenseTemplate = ConfigurePlayReadyLicenseTemplate();
		
		            IContentKeyAuthorizationPolicyOption PlayReadyPolicy =
		                _context.ContentKeyAuthorizationPolicyOptions.Create("",
		                    ContentKeyDeliveryType.PlayReadyLicense,
		                        restrictions, PlayReadyLicenseTemplate);
		
		            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
		                        ContentKeyAuthorizationPolicies.
		                        CreateAsync("Deliver Common Content Key with no restrictions").
		                        Result;
		
		
		            contentKeyAuthorizationPolicy.Options.Add(PlayReadyPolicy);
		            // Associate the content key authorization policy with the content key.
		            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
		            contentKey = contentKey.UpdateAsync().Result;
		        }
		
		        public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
		        {
		            string tokenTemplateString = GenerateTokenRequirements();
		
		            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
		            {
		                new ContentKeyAuthorizationPolicyRestriction
		                {
		                    Name = "Token Authorization Policy",
		                    KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
		                    Requirements = tokenTemplateString,
		                }
		            };
		
		            // Configure PlayReady license templates.
		            string PlayReadyLicenseTemplate = ConfigurePlayReadyLicenseTemplate();
		
		            IContentKeyAuthorizationPolicyOption PlayReadyPolicy =
		                _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
		                    ContentKeyDeliveryType.PlayReadyLicense,
		                        restrictions, PlayReadyLicenseTemplate);
		
		            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
		                        ContentKeyAuthorizationPolicies.
		                        CreateAsync("Deliver Common Content Key with token restrictions").
		                        Result;
		
		            contentKeyAuthorizationPolicy.Options.Add(PlayReadyPolicy);
		
		            // Associate the content key authorization policy with the content key
		            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
		            contentKey = contentKey.UpdateAsync().Result;
		
		            return tokenTemplateString;
		        }
		
		        static private string GenerateTokenRequirements()
		        {
		            TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);
		
		            template.PrimaryVerificationKey = new SymmetricVerificationKey();
		            template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
		            template.Audience = _sampleAudience.ToString();
		            template.Issuer = _sampleIssuer.ToString();
		            template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);
		
		            return TokenRestrictionTemplateSerializer.Serialize(template);
		        }
		
		        static private string ConfigurePlayReadyLicenseTemplate()
		        {
		            // The following code configures PlayReady License Template using .NET classes
		            // and returns the XML string.
		
		            //The PlayReadyLicenseResponseTemplate class represents the template for the response sent back to the end user. 
		            //It contains a field for a custom data string between the license server and the application 
		            //(may be useful for custom app logic) as well as a list of one or more license templates.
		            PlayReadyLicenseResponseTemplate responseTemplate = new PlayReadyLicenseResponseTemplate();
		
		            // The PlayReadyLicenseTemplate class represents a license template for creating PlayReady licenses
		            // to be returned to the end users. 
		            //It contains the data on the content key in the license and any rights or restrictions to be 
		            //enforced by the PlayReady DRM runtime when using the content key.
		            PlayReadyLicenseTemplate licenseTemplate = new PlayReadyLicenseTemplate();
		            //Configure whether the license is persistent (saved in persistent storage on the client) 
		            //or non-persistent (only held in memory while the player is using the license).  
		            licenseTemplate.LicenseType = PlayReadyLicenseType.Nonpersistent;
		
		            // AllowTestDevices controls whether test devices can use the license or not.  
		            // If true, the MinimumSecurityLevel property of the license
		            // is set to 150.  If false (the default), the MinimumSecurityLevel property of the license is set to 2000.
		            licenseTemplate.AllowTestDevices = true;
		
		            // You can also configure the Play Right in the PlayReady license by using the PlayReadyPlayRight class. 
		            // It grants the user the ability to playback the content subject to the zero or more restrictions 
		            // configured in the license and on the PlayRight itself (for playback specific policy). 
		            // Much of the policy on the PlayRight has to do with output restrictions 
		            // which control the types of outputs that the content can be played over and 
		            // any restrictions that must be put in place when using a given output.
		            // For example, if the DigitalVideoOnlyContentRestriction is enabled, 
		            //then the DRM runtime will only allow the video to be displayed over digital outputs 
		            //(analog video outputs won’t be allowed to pass the content).
		
		            //IMPORTANT: These types of restrictions can be very powerful but can also affect the consumer experience. 
		            // If the output protections are configured too restrictive, 
		            // the content might be unplayable on some clients. For more information, see the PlayReady Compliance Rules document.
		
		            // For example:
		            //licenseTemplate.PlayRight.AgcAndColorStripeRestriction = new AgcAndColorStripeRestriction(1);
		
		            responseTemplate.LicenseTemplates.Add(licenseTemplate);
		
		            return MediaServicesLicenseTemplateSerializer.Serialize(responseTemplate);
		        }
		
		        static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
		        {
		            // Get the PlayReady license service URL.
		            Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense);
		
		            Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
		                new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
		                {
		                    {AssetDeliveryPolicyConfigurationKey.PlayReadyLicenseAcquisitionUrl, acquisitionUrl.ToString()}
		
		                };
		
					// In this case we only specify Dash streaming protocol in the delivery policy,
					// All other protocols will be blocked from streaming.
		            var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
		                    "AssetDeliveryPolicy",
		                AssetDeliveryPolicyType.DynamicCommonEncryption,
		                AssetDeliveryProtocol.Dash,
		                assetDeliveryPolicyConfiguration);
		
		
		            // Add AssetDelivery Policy to the asset
		            asset.DeliveryPolicies.Add(assetDeliveryPolicy);
		
		        }
		
		        /// <summary>
		        /// Gets the streaming origin locator.
		        /// </summary>
		        /// <param name="assets"></param>
		        /// <returns></returns>
		        static public string GetStreamingOriginLocator(IAsset asset)
		        {
		
		            // Get a reference to the streaming manifest file from the  
		            // collection of files in the asset. 
		
		            var assetFile = asset.AssetFiles.Where(f => f.Name.ToLower().
		                                         EndsWith(".ism")).
		                                         FirstOrDefault();
		
		            // Create a 30-day readonly access policy. 
		            IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
		                TimeSpan.FromDays(30),
		                AccessPermissions.Read);
		
		            // Create a locator to the streaming content on an origin. 
		            ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
		                policy,
		                DateTime.UtcNow.AddMinutes(-5));
		
		            // Create a URL to the manifest file. 
		            return originLocator.Path + assetFile.Name;
		        }
		
		        static private void JobStateChanged(object sender, JobStateChangedEventArgs e)
		        {
		            Console.WriteLine(string.Format("{0}\n  State: {1}\n  Time: {2}\n\n",
		                ((IJob)sender).Name,
		                e.CurrentState,
		                DateTime.UtcNow.ToString(@"yyyy_M_d__hh_mm_ss")));
		        }
		
		        static private byte[] GetRandomBuffer(int length)
		        {
		            var returnValue = new byte[length];
		
		            using (var rng =
		                new System.Security.Cryptography.RNGCryptoServiceProvider())
		            {
		                rng.GetBytes(returnValue);
		            }
		
		            return returnValue;
		        }
		    }
		}




<!--Update_Description: add notes for limits access policy; wording update; add anchors to H2 titles-->