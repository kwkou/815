<properties
    pageTitle="如何使用 Azure 媒体服务执行实时流式处理以通过 .NET 创建多比特率流 | Azure"
    description="本教程将指导你使用 .NET SDK 完成创建通道的步骤，该通道接收单比特率实时流，并将其编码为多比特率流。"
    services="media-services"
    documentationcenter=""
    author="anilmur"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="4df5e690-ff63-47cc-879b-9c57cb8ec240"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="01/05/2017"
    wacn.date="02/24/2017"
    ms.author="juliako;anilmur" />  


#如何使用 Azure 媒体服务执行实时流式处理以通过 .NET 创建多比特率流

> [AZURE.SELECTOR]
- [门户](/documentation/articles/media-services-portal-creating-live-encoder-enabled-channel/)
- [.NET](/documentation/articles/media-services-dotnet-creating-live-encoder-enabled-channel/)
- [REST API](https://docs.microsoft.com/zh-cn/rest/api/media/operations/channel)

>[AZURE.NOTE]
若要完成本教程，你需要一个 Azure 帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

##概述

本教程将指导你完成创建**通道**的步骤，该通道接收单比特率实时流，并将其编码为多比特率流。

有关为实时编码启用的通道的更多相关概念信息，请参阅[使用 Azure 媒体服务执行实时流式处理以创建多比特率流](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)。


##常见的实时流方案

以下步骤介绍创建常见的实时流式处理应用程序时涉及的任务。

>[AZURE.NOTE] 目前，实时事件的最大建议持续时间为 8 小时。如果你需要运行一个需要更长时间的通道，请通过 Azure.cn 联系 amslived。

1. 将视频摄像机连接到计算机。启动并配置可以通过以下协议之一输出单比特率流的本地实时编码器：RTMP、平滑流式处理或 RTP (MPEG-TS)。有关详细信息，请参阅 [Azure 媒体服务 RTMP 支持和实时编码器](https://azure.microsoft.com/zh-cn/blog/azure-media-services-rtmp-support-and-live-encoders/)。

	此步骤也可以在创建通道后执行。

2. 创建并启动通道。
3. 检索通道引入 URL。

	实时编码器使用引入 URL 将流发送到频道。

4. 检索频道预览 URL。

	使用此 URL 来验证通道是否正常接收实时流。

5. 创建资源。
6. 如果你想让资源在播放期间进行动态加密，请执行以下操作：
7. 创建内容密钥。
8. 配置内容密钥授权策略。
9. 配置资产传送策略（由动态打包和动态加密使用）。
10. 创建节目并指定使用你创建的资产。
11. 通过创建按需定位器发布与节目关联的资产。

	>[AZURE.NOTE]
	创建 AMS 帐户时，系统会将**默认**流式处理终结点以“已停止”状态添加到用户的帐户。流式处理终结点（用于内容流式处理）必须处于“正在运行”状态。

12. 准备好开始流式传输和存档后，启动节目。
13. （可选）可以向实时编码器发信号，以启动广告。将广告插入到输出流中。
14. 在要停止对事件进行流式传输和存档时，停止节目。
15. 删除节目（并选择性地删除资产）。

## 学习内容
本主题演示如何使用适用于 .NET 的媒体服务 SDK 对通道和节目执行不同操作。由于许多操作都长时间运行，因此将使用管理长时间运行的操作的 .NET API。

本主题显示如何执行以下任务：

1. 创建并启动通道。将使用长时间运行的 API。
2. 获取通道引入（输入）终结点。应将此终结点提供给可以发送单比特率实时流的编码器。
3. 获取预览终结点。此终结点用于预览流。
4. 创建将用于存储你的内容的资源。还应配置资源传送策略，如此示例中所示。
5. 创建节目并指定使用你先前创建的资源。启动该节目。将使用长时间运行的 API。
6. 为资源创建定位器，以便发布内容，并可以将内容流式传输到客户端。
7. 显示和隐藏清单。启动和停止广告。将使用长时间运行的 API。
8. 清理通道及所有关联的资源。


##先决条件

以下是完成本教程所需具备的条件。

- 若要完成本教程，你需要一个 Azure 帐户。
	
	如果没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。获取可用来尝试付费版 Azure 服务的信用额度。即使在信用额度用完后，也可保留帐户并使用免费的 Azure 服务和功能，例如 Azure App Service 中的 Web 应用功能。
- 一个媒体服务帐户。若要创建媒体服务帐户，请参阅[创建帐户](/documentation/articles/media-services-create-account/)。
- Visual Studio 2010 SP1（Professional、Premium、Ultimate 或 Express）或更高版本。
- 必须使用适用于 .NET 的媒体服务 SDK 版本 3.2.0.0 或更高版本。
- 可以发送单比特率实时流的摄像头和编码器。

##注意事项

- 目前，实时事件的最大建议持续时间为 8 小时。如果你需要运行一个需要更长时间的通道，请通过 Azure.cn 联系 amslived。



##使用用于 .NET 的媒体服务 SDK 进行开发设置

1. 使用 Visual Studio 创建控制台应用程序。
2. 使用媒体服务 NuGet 包将适用于 .NET 的媒体服务 SDK 添加到控制台应用程序。

##连接到媒体服务
最佳做法是，应使用 app.config 文件来存储媒体服务名称和帐户密钥。

>[AZURE.NOTE]若要查找 Name 和 Key 值，请转到 Azure 经典管理门户，选择媒体服务帐户，然后单击门户窗口底部的“管理密钥”图标。单击每个文本框旁边的图标将值复制到系统剪贴板中。

在 app.config 文件中添加 appSettings 部分，并设置媒体服务帐户名称和帐户密钥的值。


	<?xml version="1.0"?>
	<configuration>
	  <appSettings>
	      <add key="MediaServicesAccountName" value="YouMediaServicesAccountName" />
	      <add key="MediaServicesAccountKey" value="YouMediaServicesAccountKey" />
	  </appSettings>
	</configuration>
	 
	
##代码示例

	using System;
	using System.Collections.Generic;
	using System.Configuration;
	using System.IO;
	using System.Linq;
	using System.Net;
	using Microsoft.WindowsAzure.MediaServices.Client;
	using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;
	
	namespace EncodeLiveStreamWithAmsClear
	{
	    class Program
	    {
	        private const string ChannelName = "channel001";
	        private const string AssetlName = "asset001";
	        private const string ProgramlName = "program001";
	
	        // Read values from the App.config file.
	        private static readonly string _mediaServicesAccountName =
	            ConfigurationManager.AppSettings["MediaServicesAccountName"];
	        private static readonly string _mediaServicesAccountKey =
	            ConfigurationManager.AppSettings["MediaServicesAccountKey"];

			private static readonly String _defaultScope = "urn:WindowsAzureMediaServices";

			// Azure China uses a different API server and a different ACS Base Address from the Global.
			private static readonly String _chinaApiServerUrl = "https://wamsshaclus001rest-hs.chinacloudapp.cn/API/";
			private static readonly String _chinaAcsBaseAddressUrl = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn";
	
	        // Field for service context.
	        private static CloudMediaContext _context = null;
	        private static MediaServicesCredentials _cachedCredentials = null;

			private static Uri _apiServer = null;
	
	
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

                // Used the chached credentials to create CloudMediaContext.
                _context = new CloudMediaContext(_apiServer, _cachedCredentials);
	
	            IChannel channel = CreateAndStartChannel();
	
	            // The channel's input endpoint:
	            string ingestUrl = channel.Input.Endpoints.FirstOrDefault().Url.ToString();
	
	            Console.WriteLine("Intest URL: {0}", ingestUrl);
	
	
	            // Use the previewEndpoint to preview and verify 
	            // that the input from the encoder is actually reaching the Channel. 
	            string previewEndpoint = channel.Preview.Endpoints.FirstOrDefault().Url.ToString();
	
	            Console.WriteLine("Preview URL: {0}", previewEndpoint);
	
	            // When Live Encoding is enabled, you can now get a preview of the live feed as it reaches the Channel. 
	            // This can be a valuable tool to check whether your live feed is actually reaching the Channel. 
	            // The thumbnail is exposed via the same end-point as the Channel Preview URL.
	            string thumbnailUri = new UriBuilder
	            {
	                Scheme = Uri.UriSchemeHttps,
	                Host = channel.Preview.Endpoints.FirstOrDefault().Url.Host,
	                Path = "thumbnails/input.jpg"
	            }.Uri.ToString();
	
	            Console.WriteLine("Thumbain URL: {0}", thumbnailUri);
	
	            // Once you previewed your stream and verified that it is flowing into your Channel, 
	            // you can create an event by creating an Asset, Program, and Streaming Locator. 
	            IAsset asset = CreateAndConfigureAsset();
	
	            IProgram program = CreateAndStartProgram(channel, asset);
	
	            ILocator locator = CreateLocatorForAsset(program.Asset, program.ArchiveWindowLength);
	
	            // You can use slates and ads only if the channel type is Standard.  
	            StartStopAdsSlates(channel);
	
	            // Once you are done streaming, clean up your resources.
	            Cleanup(channel);
	
	        }
	
	        public static IChannel CreateAndStartChannel()
	        {
	            var channelInput = CreateChannelInput();
	            var channePreview = CreateChannelPreview();
	            var channelEncoding = CreateChannelEncoding();
	
	
	            ChannelCreationOptions options = new ChannelCreationOptions
	            {
	                EncodingType = ChannelEncodingType.Standard,
	                Name = ChannelName,
	                Input = channelInput,
	                Preview = channePreview,
	                Encoding = channelEncoding
	            };
	
	            Log("Creating channel");
	            IOperation channelCreateOperation = _context.Channels.SendCreateOperation(options);
	            string channelId = TrackOperation(channelCreateOperation, "Channel create");
	
	            IChannel channel = _context.Channels.Where(c => c.Id == channelId).FirstOrDefault();
	
	            Log("Starting channel");
	            var channelStartOperation = channel.SendStartOperation();
	            TrackOperation(channelStartOperation, "Channel start");
	
	            return channel;
	        }
	
	        /// <summary>
	        /// Create channel input, used in channel creation options. 
	        /// </summary>
	        /// <returns></returns>
	        private static ChannelInput CreateChannelInput()
	        {
	            return new ChannelInput
	            {
	                StreamingProtocol = StreamingProtocol.RTPMPEG2TS,
	                AccessControl = new ChannelAccessControl
	                {
	                    IPAllowList = new List<IPRange>
	                    {
	                        new IPRange
	                        {
	                            Name = "TestChannelInput001",
	                            Address = IPAddress.Parse("0.0.0.0"),
	                            SubnetPrefixLength = 0
	                        }
	                    }
	                }
	            };
	        }
	
	        /// <summary>
	        /// Create channel preview, used in channel creation options. 
	        /// </summary>
	        /// <returns></returns>
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
	                            Address = IPAddress.Parse("0.0.0.0"),
	                            SubnetPrefixLength = 0
	                        }
	                    }
	                }
	            };
	        }
	
	        /// <summary>
	        /// Create channel encoding, used in channel creation options. 
	        /// </summary>
	        /// <returns></returns>
	        private static ChannelEncoding CreateChannelEncoding()
	        {
	            return new ChannelEncoding
	            {
	                SystemPreset = "Default720p",
	                IgnoreCea708ClosedCaptions = false,
	                AdMarkerSource = AdMarkerSource.Api,
	                // You can only set audio if streaming protocol is set to StreamingProtocol.RTPMPEG2TS.
	                AudioStreams = new List<AudioStream> { new AudioStream { Index = 103, Language = "eng" } }.AsReadOnly()
	            };
	        }
	
	        /// <summary>
	        /// Create an asset and configure asset delivery policies.
	        /// </summary>
	        /// <returns></returns>
	        public static IAsset CreateAndConfigureAsset()
	        {
	            IAsset asset = _context.Assets.Create(AssetlName, AssetCreationOptions.None);
	
	            IAssetDeliveryPolicy policy =
	                _context.AssetDeliveryPolicies.Create("Clear Policy",
	                AssetDeliveryPolicyType.NoDynamicEncryption,
	                AssetDeliveryProtocol.HLS | AssetDeliveryProtocol.SmoothStreaming | AssetDeliveryProtocol.Dash, null);
	
	            asset.DeliveryPolicies.Add(policy);
	
	            return asset;
	        }
	
	        /// <summary>
	        /// Create a Program on the Channel. You can have multiple Programs that overlap or are sequential;
	        /// however each Program must have a unique name within your Media Services account.
	        /// </summary>
	        /// <param name="channel"></param>
	        /// <param name="asset"></param>
	        /// <returns></returns>
	        public static IProgram CreateAndStartProgram(IChannel channel, IAsset asset)
	        {
	            IProgram program = channel.Programs.Create(ProgramlName, TimeSpan.FromHours(3), asset.Id);
	            Log("Program created", program.Id);
	
	            Log("Starting program");
	            var programStartOperation = program.SendStartOperation();
	            TrackOperation(programStartOperation, "Program start");
	
	            return program;
	        }
	
	        /// <summary>
	        /// Create locators in order to be able to publish and stream the video.
	        /// </summary>
	        /// <param name="asset"></param>
	        /// <param name="ArchiveWindowLength"></param>
	        /// <returns></returns>
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
	
	        /// <summary>
	        /// Perform operations on slates.
	        /// </summary>
	        /// <param name="channel"></param>
	        public static void StartStopAdsSlates(IChannel channel)
	        {
	            int cueId = new Random().Next(int.MaxValue);
	            var path = Path.GetFullPath(Path.Combine(AppDomain.CurrentDomain.BaseDirectory, @"..\\..\\SlateJPG\\DefaultAzurePortalSlate.jpg"));
	
	            Log("Creating asset");
	            var slateAsset = _context.Assets.Create("Slate test asset " + DateTime.Now.ToString("yyyy-MM-dd HH-mm"), AssetCreationOptions.None);
	            Log("Slate asset created", slateAsset.Id);
	
	            Log("Uploading file");
	            var assetFile = slateAsset.AssetFiles.Create("DefaultAzurePortalSlate.jpg");
	            assetFile.Upload(path);
	            assetFile.IsPrimary = true;
	            assetFile.Update();
	
	            Log("Showing slate");
	            var showSlateOpeartion = channel.SendShowSlateOperation(TimeSpan.FromMinutes(1), slateAsset.Id);
	            TrackOperation(showSlateOpeartion, "Show slate");
	
	            Log("Hiding slate");
	            var hideSlateOperation = channel.SendHideSlateOperation();
	            TrackOperation(hideSlateOperation, "Hide slate");
	
	            Log("Starting ad");
	            var startAdOperation = channel.SendStartAdvertisementOperation(TimeSpan.FromMinutes(1), cueId, false);
	            TrackOperation(startAdOperation, "Start ad");
	
	            Log("Ending ad");
	            var endAdOperation = channel.SendEndAdvertisementOperation(cueId);
	            TrackOperation(endAdOperation, "End ad");
	
	            Log("Deleting slate asset");
	            slateAsset.Delete();
	        }
	
	        /// <summary>
	        /// Clean up resources associated with the channel.
	        /// </summary>
	        /// <param name="channel"></param>
	        public static void Cleanup(IChannel channel)
	        {
	            IAsset asset;
	            if (channel != null)
	            {
	                foreach (var program in channel.Programs)
	                {
	                    asset = _context.Assets.Where(se => se.Id == program.AssetId)
	                                            .FirstOrDefault();
	
	                    Log("Stopping program");
	                    var programStopOperation = program.SendStopOperation();
	                    TrackOperation(programStopOperation, "Program stop");
	
	                    program.Delete();
	
	                    if (asset != null)
	                    {
	                        Log("Deleting locators");
	                        foreach (var l in asset.Locators)
	                            l.Delete();
	
	                        Log("Deleting asset");
	                        asset.Delete();
	                    }
	                }
	
	                Log("Stopping channel");
	                var channelStopOperation = channel.SendStopOperation();
	                TrackOperation(channelStopOperation, "Channel stop");
	
	                Log("Deleting channel");
	                var channelDeleteOperation = channel.SendDeleteOperation();
	                TrackOperation(channelDeleteOperation, "Channel delete");
	            }
	        }
	
	
	        /// <summary>
	        /// Track long running operations.
	        /// </summary>
	        /// <param name="operation"></param>
	        /// <param name="description"></param>
	        /// <returns></returns>
	        public static string TrackOperation(IOperation operation, string description)
	        {
	            string entityId = null;
	            bool isCompleted = false;
	
	            Log("starting to track ", null, operation.Id);
	            while (isCompleted == false)
	            {
	                operation = _context.Operations.GetOperation(operation.Id);
	                isCompleted = IsCompleted(operation, out entityId);
	                System.Threading.Thread.Sleep(TimeSpan.FromSeconds(30));
	            }
	            // If we got here, the operation succeeded.
	            Log(description + " in completed", operation.TargetEntityId, operation.Id);
	
	            return entityId;
	        }
	
	        /// <summary> 
	        /// Checks if the operation has been completed. 
	        /// If the operation succeeded, the created entity Id is returned in the out parameter.
	        /// </summary> 
	        /// <param name="operationId">The operation Id.</param> 
	        /// <param name="channel">
	        /// If the operation succeeded, 
	        /// the entity Id associated with the sucessful operation is returned in the out parameter.</param>
	        /// <returns>Returns false if the operation is still in progress; otherwise, true.</returns> 
	        private static bool IsCompleted(IOperation operation, out string entityId)
	        {
	
	            bool completed = false;
	
	            entityId = null;
	
	            switch (operation.State)
	            {
	                case OperationState.Failed:
	                    // Handle the failure. 
	                    // For example, throw an exception. 
	                    // Use the following information in the exception: operationId, operation.ErrorMessage.
	                    Log("operation failed", operation.TargetEntityId, operation.Id);
	                    break;
	                case OperationState.Succeeded:
	                    completed = true;
	                    entityId = operation.TargetEntityId;
	                    break;
	                case OperationState.InProgress:
	                    completed = false;
	                    Log("operation in progress", operation.TargetEntityId, operation.Id);
	                    break;
	            }
	            return completed;
	        }
	
	
	        private static void Log(string action, string entityId = null, string operationId = null)
	        {
	            Console.WriteLine(
	                "{0,-21}{1,-51}{2,-51}{3,-51}",
	                DateTime.Now.ToString("yyyy'-'MM'-'dd HH':'mm':'ss"),
	                action,
	                entityId ?? string.Empty,
	                operationId ?? string.Empty);
	        }
	    }
	}	

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: update "通道" to "频道"; add note for creating AMS account-->