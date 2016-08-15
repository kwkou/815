<properties 
	pageTitle="如何使用本地编码器执行实时流式处理" 
	description="本主题演示如何使用 .NET 通过本地编码器进行实时编码。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako,cenkdin" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="05/05/2016"  
	wacn.date="06/20/2016"/>

#如何使用本地编码器执行实时流式处理

##先决条件

以下是完成本教程所需具备的条件：

- 一个 Azure 帐户。
- 一个媒体服务帐户。若要创建媒体服务帐户，请参阅[如何创建媒体服务帐户](/documentation/articles/media-services-create-account/)。
- 设置开发环境。有关详细信息，请参阅[设置环境](/documentation/articles/media-services-set-up-computer/)。
- 网络摄像机。例如，[Telestream Wirecast 编码器](http://www.telestream.net/wirecast/overview.htm)。 

建议阅读以下文章：

- [Azure 媒体服务 RTMP 支持和实时编码器](https://azure.microsoft.com/blog/2014/09/18/azure-media-services-rtmp-support-and-live-encoders/)
- [使用本地编码器执行实时流式处理以创建多比特率流](/documentation/articles/media-services-live-streaming-with-onprem-encoders/)
 

##示例
	
下面的代码示例演示如何完成以下任务：

- 连接到媒体服务
- 创建通道
- 更新通道
- 检索通道的输入终结点。应将输入终结点提供给本地实时编码器。实时编码器将相机的信号转换为流，以便发送到通道的输入（插入）终结点。
- 检索通道的预览终结点
- 创建并启动节目
- 创建访问该节目所需的定位符
- 创建并启动 StreamingEndpoint
- 更新流式处理终结点
- 获取所有流式处理终结点的定位符
- 关闭资源
	
有关如何配置实时编码器的信息，请参阅 [Azure 媒体服务 RTMP 支持和实时编码器](https://azure.microsoft.com/blog/2014/09/18/azure-media-services-rtmp-support-and-live-encoders/)。
	
		using System;
		using System.Collections.Generic;
		using System.Configuration;
		using System.IO;
		using System.Linq;
		using System.Net;
		using System.Security.Cryptography;
		using System.Text;
		using System.Threading.Tasks;
		using Microsoft.WindowsAzure.MediaServices.Client;
		using Newtonsoft.Json.Linq;
		
		namespace AMSLiveTest
		{
		    class Program
		    {
		        private const string StreamingEndpointName = "streamingendpoint001";
		        private const string ChannelName = "channel001";
		        private const string AssetlName = "asset001";
		        private const string ProgramlName = "program001";
		
				private static readonly String _defaultScope = "urn:WindowsAzureMediaServices";

				// Azure China uses a different API server and a different ACS Base Address from the Global.
				private static readonly String _chinaApiServerUrl = "https://wamsshaclus001rest-hs.chinacloudapp.cn/API/";
				private static readonly String _chinaAcsBaseAddressUrl = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn";

		        // Read values from the App.config file.
		        private static readonly string _mediaServicesAccountName =
		            ConfigurationManager.AppSettings["MediaServicesAccountName"];
		        private static readonly string _mediaServicesAccountKey =
		            ConfigurationManager.AppSettings["MediaServicesAccountKey"];
		
		        // Field for service context.
		        private static CloudMediaContext _context = null;
		        private static MediaServicesCredentials _cachedCredentials = null;
		
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
		
		            IChannel channel = CreateAndStartChannel();
		
		            // Set the Live Encoder to point to the channel's input endpoint:
		            string ingestUrl = channel.Input.Endpoints.FirstOrDefault().Url.ToString();
		
		            // Use the previewEndpoint to preview and verify 
		            // that the input from the encoder is actually reaching the Channel. 
		            string previewEndpoint = channel.Preview.Endpoints.FirstOrDefault().Url.ToString();
		
		            IProgram program = CreateAndStartProgram(channel);
		            ILocator locator = CreateLocatorForAsset(program.Asset, program.ArchiveWindowLength);
		            IStreamingEndpoint streamingEndpoint = CreateAndStartStreamingEndpoint();
		            GetLocatorsInAllStreamingEndpoints(program.Asset);
		
		            // Once you are done streaming, clean up your resources.
		            Cleanup(streamingEndpoint, channel);
		        }
		
		        public static IChannel CreateAndStartChannel()
		        {
					//If you want to change the Smooth fragments to HLS segment ratio, you would set the ChannelCreationOptions’s Output property.
	
		            IChannel channel = _context.Channels.Create(
		                new ChannelCreationOptions
		                {
		                    Name = ChannelName,
		                    Input = CreateChannelInput(),
		                    Preview = CreateChannelPreview()
		                });
		
					//Starting and stopping Channels can take some time to execute. To determine the state of operations after calling Start or Stop, query the IChannel.State .
	
		            channel.Start();
		
		            return channel;
		        }
		
		        private static ChannelInput CreateChannelInput()
		        {
		            return new ChannelInput
		            {
		                StreamingProtocol = StreamingProtocol.RTMP,
		                AccessControl = new ChannelAccessControl
		                {
		                    IPAllowList = new List<IPRange>
		                    {
		                        new IPRange
		                        {
		                            Name = "TestChannelInput001",
		                            // Setting 0.0.0.0 for Address and 0 for SubnetPrefixLength
		                            // will allow access to IP addresses.
		                            Address = IPAddress.Parse("0.0.0.0"),
		                            SubnetPrefixLength = 0
		                        }
		                    }
		                }
		            };
		        }
		
		        private static ChannelPreview CreateChannelPreview()
		        {
		            return new ChannelPreview
		            {
		                AccessControl = new ChannelAccessControl
		                {
		                    IPAllowList = new List<IPRange>
		                    {
		                        new IPRange
		                        {
		                            Name = "TestChannelPreview001",
		                            // Setting 0.0.0.0 for Address and 0 for SubnetPrefixLength
		                            // will allow access to IP addresses.
		                            Address = IPAddress.Parse("0.0.0.0"),
		                            SubnetPrefixLength = 0
		                        }
		                    }
		                }
		            };
		        }
		
		        public static void UpdateCrossSiteAccessPoliciesForChannel(IChannel channel)
		        {
		            var clientPolicy =
		                @"<?xml version=""1.0"" encoding=""utf-8""?>
		            <access-policy>
		                <cross-domain-access>
		                    <policy>
		                        <allow-from http-request-headers=""*"" http-methods=""*"">
		                            <domain uri=""*""/>
		                        </allow-from>
		                        <grant-to>
		                           <resource path=""/"" include-subpaths=""true""/>
		                        </grant-to>
		                    </policy>
		                </cross-domain-access>
		            </access-policy>";
		
		            var xdomainPolicy =
		                @"<?xml version=""1.0"" ?>
		            <cross-domain-policy>
		                <allow-access-from domain=""*"" />
		            </cross-domain-policy>";
		
		            channel.CrossSiteAccessPolicies.ClientAccessPolicy = clientPolicy;
		            channel.CrossSiteAccessPolicies.CrossDomainPolicy = xdomainPolicy;
		
		            channel.Update();
		        }
		
		        public static IProgram CreateAndStartProgram(IChannel channel)
		        {
		            IAsset asset = _context.Assets.Create(AssetlName, AssetCreationOptions.None);
		
		            // Create a Program on the Channel. You can have multiple Programs that overlap or are sequential;
		            // however each Program must have a unique name within your Media Services account.
		            IProgram program = channel.Programs.Create(ProgramlName, TimeSpan.FromHours(3), asset.Id);
		            program.Start();
		
		            return program;
		        }
		
		        public static ILocator CreateLocatorForAsset(IAsset asset, TimeSpan ArchiveWindowLength)
		        {
                	// You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.            

		            var locator = _context.Locators.CreateLocator
		                (
		                    LocatorType.OnDemandOrigin,
		                    asset,
		                    _context.AccessPolicies.Create
		                    (
		                        "Live Stream Policy",
		                        ArchiveWindowLength,
		                        AccessPermissions.Read
		                    )
		                );
		
		            return locator;
		        }
		
		        public static IStreamingEndpoint CreateAndStartStreamingEndpoint()
		        {
		            var options = new StreamingEndpointCreationOptions
		            {
		                Name = StreamingEndpointName,
		                ScaleUnits = 1,
		                AccessControl = GetAccessControl(),
		                CacheControl = GetCacheControl()
		            };
		
		            IStreamingEndpoint streamingEndpoint = _context.StreamingEndpoints.Create(options);
		            streamingEndpoint.Start();
		
		            return streamingEndpoint;
		        }
		
		        private static StreamingEndpointAccessControl GetAccessControl()
		        {
		            return new StreamingEndpointAccessControl
		            {
		                IPAllowList = new List<IPRange>
		                {
		                    new IPRange
		                    {
		                        Name = "Allow all",
		                        Address = IPAddress.Parse("0.0.0.0"),
		                        SubnetPrefixLength = 0
		                    }
		                },
		
		                AkamaiSignatureHeaderAuthenticationKeyList = new List<AkamaiSignatureHeaderAuthenticationKey>
		                {
		                    new AkamaiSignatureHeaderAuthenticationKey
		                    {
		                        Identifier = "My key",
		                        Expiration = DateTime.UtcNow + TimeSpan.FromDays(365),
		                        Base64Key = Convert.ToBase64String(GenerateRandomBytes(16))
		                    }
		                }
		            };
		        }
		
		        private static byte[] GenerateRandomBytes(int length)
		        {
		            var bytes = new byte[length];
		            using (var rng = new RNGCryptoServiceProvider())
		            {
		                rng.GetBytes(bytes);
		            }
		
		            return bytes;
		        }
		
		        private static StreamingEndpointCacheControl GetCacheControl()
		        {
		            return new StreamingEndpointCacheControl
		            {
		                MaxAge = TimeSpan.FromSeconds(1000)
		            };
		        }
		
		        public static void UpdateCrossSiteAccessPoliciesForStreamingEndpoint(IStreamingEndpoint streamingEndpoint)
		        {
		            var clientPolicy =
		                @"<?xml version=""1.0"" encoding=""utf-8""?>
		            <access-policy>
		                <cross-domain-access>
		                    <policy>
		                        <allow-from http-request-headers=""*"" http-methods=""*"">
		                            <domain uri=""*""/>
		                        </allow-from>
		                        <grant-to>
		                           <resource path=""/"" include-subpaths=""true""/>
		                        </grant-to>
		                    </policy>
		                </cross-domain-access>
		            </access-policy>";
		
		            var xdomainPolicy =
		                @"<?xml version=""1.0"" ?>
		            <cross-domain-policy>
		                <allow-access-from domain=""*"" />
		            </cross-domain-policy>";
		
		            streamingEndpoint.CrossSiteAccessPolicies.ClientAccessPolicy = clientPolicy;
		            streamingEndpoint.CrossSiteAccessPolicies.CrossDomainPolicy = xdomainPolicy;
		
		            streamingEndpoint.Update();
		        }
		
		        public static void GetLocatorsInAllStreamingEndpoints(IAsset asset)
		        {
		            var locators = asset.Locators.Where(l => l.Type == LocatorType.OnDemandOrigin);
		            var ismFile = asset.AssetFiles.AsEnumerable().FirstOrDefault(a => a.Name.EndsWith(".ism"));
		            var template = new UriTemplate("{contentAccessComponent}/{ismFileName}/manifest");
		            var urls = locators.SelectMany(l =>
		                        _context
		                            .StreamingEndpoints
		                            .AsEnumerable()
		                            .Where(se => se.State == StreamingEndpointState.Running)
		                            .Select(
		                                se =>
		                                    template.BindByPosition(new Uri("http://" + se.HostName),
		                                    l.ContentAccessComponent,
		                                        ismFile.Name)))
		                        .ToArray();
		
		        }
		
		        public static void Cleanup(IStreamingEndpoint streamingEndpoint,
		                                    IChannel channel)
		        {
		            if (streamingEndpoint != null)
		            {
		                streamingEndpoint.Stop();
		                streamingEndpoint.Delete();
		            }
		
		            IAsset asset;
		            if (channel != null)
		            {
		
		                foreach (var program in channel.Programs)
		                {
		                    asset = _context.Assets.Where(se => se.Id == program.AssetId)
		                                            .FirstOrDefault();
		
		                    program.Stop();
		                    program.Delete();
		
		                    if (asset != null)
		                    {
		                        foreach (var l in asset.Locators)
		                            l.Delete();
		
		                        asset.Delete();
		                    }
		                }
		
		                channel.Stop();
		                channel.Delete();
		            }
		        }
		    }
		}


<!---HONumber=Mooncake_0613_2016-->