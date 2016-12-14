<properties
    pageTitle="使用 .NET 配置 Azure 媒体服务遥测 | Azure"
    description="本文说明如何通过 .NET SDK 使用 Azure 媒体服务遥测。"
    services="media-services"
    documentationcenter=""
    author="Juliako"
    manager="erikre"
    editor="" />  

<tags
    ms.assetid="4e4a9ec3-8ddb-4938-aec1-d7172d3db858"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/17/2016"
    wacn.date="12/12/2016"
    ms.author="juliako" />  


# 使用 .NET 配置 Azure 媒体服务遥测

本主题介绍使用 .NET SDK 配置 Azure 媒体服务 (AMS) 遥测可采取的常规步骤。

>[AZURE.NOTE]有关 AMS 遥测定义和使用方法的详细介绍，请参阅[概述](/documentation/articles/media-services-telemetry-overview/)主题。

可通过以下方式之一使用遥测数据：

- 直接从 Azure 表存储中读取数据，例如使用存储 SDK。有关遥测存储表的说明，请参阅[此主题](https://msdn.microsoft.com/zh-cn/library/mt742089.aspx)中的**使用遥测信息**。

或

- 使用媒体服务 .NET SDK 中支持的内容来读取存储数据。本主题说明如何为指定的 AMS 帐户启用遥测，以及如何使用 Azure 媒体服务 .NET SDK 查询度量值。

## 为媒体服务帐户配置遥测

启用遥测需要执行以下步骤：

- 获取已附加到媒体服务帐户的存储帐户的凭据。
- 创建一个通知终结点，其 **EndPointType** 设置为 **AzureTable**，endPontAddress 指向存储表。

	    INotificationEndPoint notificationEndPoint = 
	                  _context.NotificationEndPoints.Create("monitoring", 
	                  NotificationEndPointType.AzureTable,
	                  "https://" + _mediaServicesStorageAccountName + ".table.core.chinacloudapi.cn/");

- 为要监视的服务创建监视配置设置。最多允许一个监视配置设置。
  
        IMonitoringConfiguration monitoringConfiguration = _context.MonitoringConfigurations.Create(notificationEndPoint.Id,
            new List<ComponentMonitoringSetting>()
            {
                new ComponentMonitoringSetting(MonitoringComponent.Channel, MonitoringLevel.Normal),
                new ComponentMonitoringSetting(MonitoringComponent.StreamingEndpoint, MonitoringLevel.Normal)
            });

## 使用遥测信息

有关使用遥测信息的信息，请参阅[此主题](/documentation/articles/media-services-telemetry-overview/)。
 
## 示例  
	
本部分所示的示例假设 App.config 文件中包含以下部分。
	
	<appSettings>
	  <add key="MediaServicesAccountName" value="ams_acct_name" />
	  <add key="MediaServicesAccountKey" value="ams_acct_key" />
	  <add key="MediaServicesAccountID" value="ams_acct_id" />
	  <add key="StorageAccountName" value="storage_name" />
	  <add key="StorageAccountKey" value="storage_key" />
	</appSettings>

以下示例说明如何为指定的 AMS 帐户启用遥测，以及如何使用 Azure 媒体服务 .NET SDK 查询度量值。

	using System;
	using System.Collections.Generic;
	using System.Configuration;
	using System.Linq;
	using Microsoft.WindowsAzure.MediaServices.Client;
	
	namespace AMSMetrics
	{
	    class Program
	    {
	        private static readonly string _mediaServicesAccountName =
	            ConfigurationManager.AppSettings["MediaServicesAccountName"];
	        private static readonly string _mediaServicesAccountKey =
	            ConfigurationManager.AppSettings["MediaServicesAccountKey"];
	        private static readonly string _mediaServicesAccountID =
	            ConfigurationManager.AppSettings["MediaServicesAccountID"];
	        private static readonly string _mediaServicesStorageAccountName =
	            ConfigurationManager.AppSettings["StorageAccountName"];
	        private static readonly string _mediaServicesStorageAccountKey =
	            ConfigurationManager.AppSettings["StorageAccountKey"];
		    
		private static readonly String _defaultScope = "urn:WindowsAzureMediaServices";
		
		// Azure China uses a different API server and a different ACS Base Address from the Global.
		private static readonly String _chinaApiServerUrl = "https://wamsshaclus001rest-hs.chinacloudapp.cn/API/";
		private static readonly String _chinaAcsBaseAddressUrl = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn";
	
	        // Field for service context.
	        private static CloudMediaContext _context = null;
	        private static MediaServicesCredentials _cachedCredentials = null;
		private static Uri _apiServer = null;
		
		
	        private static IStreamingEndpoint _streamingEndpoint = null;
	        private static IChannel _channel = null;
	
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
	
	            _streamingEndpoint = _context.StreamingEndpoints.FirstOrDefault();
	            _channel = _context.Channels.FirstOrDefault();
	
	            var monitoringConfigurations = _context.MonitoringConfigurations;
	            IMonitoringConfiguration monitoringConfiguration = null;
	
	            // No more than one monitoring configuration settings is allowed.
	            if (monitoringConfigurations.ToArray().Length != 0)
	            {
	                monitoringConfiguration = _context.MonitoringConfigurations.FirstOrDefault();
	            }
	            else
	            {
	                INotificationEndPoint notificationEndPoint =
	                              _context.NotificationEndPoints.Create("monitoring",
	                              NotificationEndPointType.AzureTable, GetTableEndPoint());
	
	                monitoringConfiguration = _context.MonitoringConfigurations.Create(notificationEndPoint.Id,
	                    new List<ComponentMonitoringSetting>()
	                    {
	                        new ComponentMonitoringSetting(MonitoringComponent.Channel, MonitoringLevel.Normal),
	                        new ComponentMonitoringSetting(MonitoringComponent.StreamingEndpoint, MonitoringLevel.Normal)
	
	                    });
	            }
	
	            //Print metrics for a Streaming Endpoint.
	            PrintStreamingEndpointMetrics();
	
	            Console.ReadLine();
	        }
	
	        private static string GetTableEndPoint()
	        {
	            return "https://" + _mediaServicesStorageAccountName + ".table.core.chinacloudapi.cn/";
	        }
	
	        private static void PrintStreamingEndpointMetrics()
	        {
	            Console.WriteLine(string.Format("Telemetry for streaming endpoint '{0}'", _streamingEndpoint.Name));
	
	            var end = DateTime.UtcNow;
	            var start = DateTime.UtcNow.AddHours(-5);
	
	            // Get some streaming endpoint metrics.
	            var res = _context.StreamingEndPointRequestLogs.GetStreamingEndPointMetrics(
	                    GetTableEndPoint(),
	                    _mediaServicesStorageAccountKey,
	                    _mediaServicesAccountID,
	                    _streamingEndpoint.Id,
	                    start,
	                    end);
	
	
	            Console.Title = "Streaming endpoint metrics:";
	
	            foreach (var log in res)
	            {
	                Console.WriteLine("AccountId: {0}", log.AccountId);
	                Console.WriteLine("BytesSent: {0}", log.BytesSent);
	                Console.WriteLine("EndToEndLatency: {0}", log.EndToEndLatency);
	                Console.WriteLine("HostName: {0}", log.HostName);
	                Console.WriteLine("ObservedTime: {0}", log.ObservedTime);
	                Console.WriteLine("PartitionKey: {0}", log.PartitionKey);
	                Console.WriteLine("RequestCount: {0}", log.RequestCount);
	                Console.WriteLine("ResultCode: {0}", log.ResultCode);
	                Console.WriteLine("RowKey: {0}", log.RowKey);
	                Console.WriteLine("ServerLatency: {0}", log.ServerLatency);
	                Console.WriteLine("StatusCode: {0}", log.StatusCode);
	                Console.WriteLine("StreamingEndpointId: {0}", log.StreamingEndpointId);
	                Console.WriteLine();
	            }
	
	            Console.WriteLine();
	        }
	
	        private static void PrintChannelMetrics()
	        {
	            if (_channel == null)
	            {
	                Console.WriteLine("There are no channels in this AMS account");
	                return;
	            }
	
	            Console.WriteLine(string.Format("Telemetry for channel '{0}'", _channel.Name));
	
	            var end = DateTime.UtcNow;
	            var start = DateTime.UtcNow.AddHours(-5);
	
	            // Get some channel metrics.
	            var channelMetrics = _context.ChannelMetrics.GetChannelMetrics(
	                GetTableEndPoint(),
	                _mediaServicesStorageAccountKey,
	                _mediaServicesAccountID,
	               _channel.Id,
	                start,
	                end);
	
	            // Print the channel metrics.
	            Console.WriteLine("Channel metrics:");
	
	            foreach (var channelHeartbeat in channelMetrics.OrderBy(x => x.ObservedTime))
	            {
	                Console.WriteLine(
	                    "    Observed time: {0}, Last timestamp: {1}, Incoming bitrate: {2}",
	                    channelHeartbeat.ObservedTime,
	                    channelHeartbeat.LastTimestamp,
	                    channelHeartbeat.IncomingBitrate);
	            }
	
	            Console.WriteLine();
	        }
	    }
	}

<!---HONumber=Mooncake_1205_2016-->