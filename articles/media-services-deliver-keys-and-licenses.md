<properties 
	pageTitle="使用 Azure 媒体服务传送 DRM 许可证或 AES 密钥" 
	description="本文介绍如何使用 Azure 媒体服务 (AMS) 来传送 PlayReady 和/或 Widevine 许可证与 AES 密钥，但余下的操作（编码、加密、流式传输）是使用你的本地服务器完成的。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="02/24/2016" 
	wacn.date="04/05/2016"/>


#使用 Azure 媒体服务传送 DRM 许可证或 AES 密钥

Azure 媒体服务 (AMS) 可让你引入、编码、添加内容保护，以及流式传输内容（有关详细信息，请参阅[此文章](/documentation/articles/media-services-protect-with-drm/)）。但是，有些客户只想使用 AMS 来传送许可证和/或密钥，并使用他们的本地服务器来进行编码、加密和流式传输。本文说明如何使用 AMS 来传送 PlayReady 和/或 Widevine 许可证，但使用本地服务器来完成其余部分。


## 概述

媒体服务提供传送 PlayReady 和 Widevine DRM 许可证及 AES-128 密钥的服务。媒体服务还提供用于配置所需权限和限制的 API，这样当用户播放 DRM 保护的内容时，DRM 运行时便会强制实施这些权限和限制。当用户请求受保护的内容时，播放器应用程序将从 AMS 许可证服务请求许可证。AMS 许可证服务将向播放器颁发许可证（如果播放器已获授权）。PlayReady 和 Widevine 许可证包含客户端播放器用来对内容进行解密和流式传输的解密密钥。

媒体服务支持通过多种方式对发出许可证或密钥请求的用户进行授权。你可以配置内容密钥的授权策略，该策略可以包含一种或多种限制：开放或令牌限制。令牌限制策略必须附带由安全令牌服务 (STS) 颁发的令牌。媒体服务支持采用简单 Web 令牌 (SWT) 格式和 JSON Web 令牌 (JWT) 格式的令牌。


下图显示了使用 AMS 传送 PlayReady 和/或 Widevine 许可证，但使用本地服务器完成其余部分所要执行的主要步骤。

![使用 PlayReady 进行保护](./media/media-services-deliver-keys-and-licenses/media-services-diagram1.png)

##下载示例

可以从[此处](https://github.com/Azure/media-services-dotnet-deliver-drm-licenses)下载本文所述的示例。

##.NET 代码示例

本主题中的代码示例演示如何创建通用内容密钥，并获取 PlayReady 或 Widevine 许可证获取 URL。你需要从 AMS 获取以下信息片段并配置本地服务器： **内容密钥** 、 **密钥 ID** 、 **许可证获取 URL** 。配置本地服务器后，你可以从自己的流服务器进行流式传输。由于加密的流指向 AMS 许可证服务器，播放器将从 AMS 请求许可证。如果你选择令牌身份验证，AMS 许可证服务器将验证通过 HTTPS 发送的令牌，然后（如果有效）将许可证传回给播放器。（代码示例仅演示了如何创建通用内容密钥，并获取 PlayReady 或 Widevine 许可证获取 URL。如果你想要传送 AES-128 密钥，则需要创建信封内容密钥，并获取密钥获取 URL，[此文章](/documentation/articles/media-services-protect-with-aes128/)介绍了具体的操作）。
	
	
	using System;
	using System.Collections.Generic;
	using System.Configuration;
	using System.IO;
	using System.Linq;
	using System.Threading;
	using Microsoft.WindowsAzure.MediaServices.Client;
	using Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization;
	using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;
	using Microsoft.WindowsAzure.MediaServices.Client.Widevine;
	using Newtonsoft.Json;
	
	
	namespace DeliverDRMLicenses
	{
	    class Program
	    {
	        // Read values from the App.config file.
	        private static readonly string _mediaServicesAccountName =
	            ConfigurationManager.AppSettings["MediaServicesAccountName"];
	        private static readonly string _mediaServicesAccountKey =
	            ConfigurationManager.AppSettings["MediaServicesAccountKey"];
	
	        private static readonly Uri _sampleIssuer =
	            new Uri(ConfigurationManager.AppSettings["Issuer"]);
	        private static readonly Uri _sampleAudience =
	            new Uri(ConfigurationManager.AppSettings["Audience"]);
	
	        // Field for service context.
	        private static CloudMediaContext _context = null;
	        private static MediaServicesCredentials _cachedCredentials = null;
	
	        static void Main(string[] args)
	        {
	            // Create and cache the Media Services credentials in a static class variable.
	            _cachedCredentials = new MediaServicesCredentials(
	                            _mediaServicesAccountName,
	                            _mediaServicesAccountKey);
	            // Used the cached credentials to create CloudMediaContext.
	            _context = new CloudMediaContext(_cachedCredentials);
	
	            bool tokenRestriction = true;
	            string tokenTemplateString = null;
	
	
	            IContentKey key = CreateCommonTypeContentKey();
	
	            // Print out the key ID and Key in base64 string format
	            Console.WriteLine("Created key {0} with key value {1} ", 
	                key.Id, System.Convert.ToBase64String(key.GetClearKeyValue()));
	
	            Console.WriteLine("PlayReady License Key delivery URL: {0}", 
	                key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense));
	
	            Console.WriteLine("Widevine License Key delivery URL: {0}",
	                key.GetKeyDeliveryUrl(ContentKeyDeliveryType.Widevine));
	
	            if (tokenRestriction)
	                tokenTemplateString = AddTokenRestrictedAuthorizationPolicy(key);
	            else
	                AddOpenAuthorizationPolicy(key);
	
	            Console.WriteLine("Added authorization policy: {0}", 
	                key.AuthorizationPolicyId);
	            Console.WriteLine();
	            Console.ReadLine();
	        }
	
	        static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
	        {
	
	            // Create ContentKeyAuthorizationPolicy with Open restrictions 
	            // and create authorization policy          
	
	            List<ContentKeyAuthorizationPolicyRestriction> restrictions = 
	                new List<ContentKeyAuthorizationPolicyRestriction>
	            {
	                new ContentKeyAuthorizationPolicyRestriction
	                {
	                    Name = "Open",
	                    KeyRestrictionType = (int)ContentKeyRestrictionType.Open,
	                    Requirements = null
	                }
	            };
	
	            // Configure PlayReady and Widevine license templates.
	            string PlayReadyLicenseTemplate = ConfigurePlayReadyLicenseTemplate();
	
	            string WidevineLicenseTemplate = ConfigureWidevineLicenseTemplate();
	
	            IContentKeyAuthorizationPolicyOption PlayReadyPolicy =
	                _context.ContentKeyAuthorizationPolicyOptions.Create("",
	                    ContentKeyDeliveryType.PlayReadyLicense,
	                        restrictions, PlayReadyLicenseTemplate);
	
	            IContentKeyAuthorizationPolicyOption WidevinePolicy =
	                _context.ContentKeyAuthorizationPolicyOptions.Create("",
	                    ContentKeyDeliveryType.Widevine,
	                    restrictions, WidevineLicenseTemplate);
	
	            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
	                        ContentKeyAuthorizationPolicies.
	                        CreateAsync("Deliver Common Content Key with no restrictions").
	                        Result;
	
	
	            contentKeyAuthorizationPolicy.Options.Add(PlayReadyPolicy);
	            contentKeyAuthorizationPolicy.Options.Add(WidevinePolicy);
	            // Associate the content key authorization policy with the content key.
	            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
	            contentKey = contentKey.UpdateAsync().Result;
	        }
	
	        public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
	        {
	            string tokenTemplateString = GenerateTokenRequirements();
	
	            List<ContentKeyAuthorizationPolicyRestriction> restrictions = 
	                new List<ContentKeyAuthorizationPolicyRestriction>
	            {
	                new ContentKeyAuthorizationPolicyRestriction
	                {
	                    Name = "Token Authorization Policy",
	                    KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
	                    Requirements = tokenTemplateString,
	                }
	            };
	
	            // Configure PlayReady and Widevine license templates.
	            string PlayReadyLicenseTemplate = ConfigurePlayReadyLicenseTemplate();
	
	            string WidevineLicenseTemplate = ConfigureWidevineLicenseTemplate();
	
	            IContentKeyAuthorizationPolicyOption PlayReadyPolicy =
	                _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
	                    ContentKeyDeliveryType.PlayReadyLicense,
	                        restrictions, PlayReadyLicenseTemplate);
	
	            IContentKeyAuthorizationPolicyOption WidevinePolicy =
	                _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
	                    ContentKeyDeliveryType.Widevine,
	                        restrictions, WidevineLicenseTemplate);
	
	            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
	                        ContentKeyAuthorizationPolicies.
	                        CreateAsync("Deliver Common Content Key with token restrictions").
	                        Result;
	
	            contentKeyAuthorizationPolicy.Options.Add(PlayReadyPolicy);
	            contentKeyAuthorizationPolicy.Options.Add(WidevinePolicy);
	
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
	
	            //The PlayReadyLicenseResponseTemplate class represents the template 
	            //for the response sent back to the end user. 
	            //It contains a field for a custom data string between the license server 
	            //and the application (may be useful for custom app logic) 
	            //as well as a list of one or more license templates.
	
	            PlayReadyLicenseResponseTemplate responseTemplate = 
	                new PlayReadyLicenseResponseTemplate();
	
	            // The PlayReadyLicenseTemplate class represents a license template 
	            // for creating PlayReady licenses
	            // to be returned to the end users. 
	            // It contains the data on the content key in the license 
	            // and any rights or restrictions to be 
	            // enforced by the PlayReady DRM runtime when using the content key.
	            PlayReadyLicenseTemplate licenseTemplate = new PlayReadyLicenseTemplate();
	
	            // Configure whether the license is persistent 
	            // (saved in persistent storage on the client) 
	            // or non-persistent (only held in memory while the player is using the license).  
	            licenseTemplate.LicenseType = PlayReadyLicenseType.Nonpersistent;
	
	            // AllowTestDevices controls whether test devices can use the license or not.  
	            // If true, the MinimumSecurityLevel property of the license
	            // is set to 150.  If false (the default), 
	            // the MinimumSecurityLevel property of the license is set to 2000.
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
	
	            // IMPORTANT: These types of restrictions can be very powerful 
	            // but can also affect the consumer experience. 
	            // If the output protections are configured too restrictive, 
	            // the content might be unplayable on some clients. 
	            // For more information, see the PlayReady Compliance Rules document.
	
	            // For example:
	            //licenseTemplate.PlayRight.AgcAndColorStripeRestriction = new AgcAndColorStripeRestriction(1);
	
	            responseTemplate.LicenseTemplates.Add(licenseTemplate);
	
	            return MediaServicesLicenseTemplateSerializer.Serialize(responseTemplate);
	        }
	
	
	        private static string ConfigureWidevineLicenseTemplate()
	        {
	            var template = new WidevineMessage
	            {
	                allowed_track_types = AllowedTrackTypes.SD_HD,
	                content_key_specs = new[]
	                {
	                    new ContentKeySpecs
	                    {
	                        required_output_protection = 
	                            new RequiredOutputProtection { hdcp = Hdcp.HDCP_NONE},
	                        security_level = 1,
	                        track_type = "SD"
	                    }
	                },
	                policy_overrides = new
	                {
	                    can_play = true,
	                    can_persist = true,
	                    can_renew = false
	                }
	            };
	
	            string configuration = JsonConvert.SerializeObject(template);
	            return configuration;
	        }
	
	
	        static public IContentKey CreateCommonTypeContentKey()
	        {
	            // Create envelope encryption content key
	            Guid keyId = Guid.NewGuid();
	            byte[] contentKey = GetRandomBuffer(16);
	
	            IContentKey key = _context.ContentKeys.Create(
	                                    keyId,
	                                    contentKey,
	                                    "ContentKey",
	                                    ContentKeyType.CommonEncryption);
	
	            return key;
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
	




##另请参阅

[使用 PlayReady 和/或 Widevine DRM 动态通用加密](/documentation/articles/media-services-protect-with-drm/)

[使用 AES-128 动态加密和密钥传送服务](/documentation/articles/media-services-protect-with-aes128/)

[使用合作伙伴将 Widevine 许可证传送到 Azure 媒体服务](/documentation/articles/media-services-licenses-partner-integration/)
<!---HONumber=Mooncake_0328_2016-->