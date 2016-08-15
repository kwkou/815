<properties 
	pageTitle="使用 Azure 媒体包装器完成静态打包任务" 
	description="本主题说明了通过 Azure 媒体包装器完成的各种任务。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="05/03/2016"    
	wacn.date="06/27/2016"/>


# 使用 Azure 媒体包装器完成静态打包任务

>[AZURE.NOTE]Azure 媒体包装器和 Azure 媒体加密器的使用期限已延长到 2017 年 3 月 1 日。在此日期之前，这些处理器的功能将添加到媒体编码器标准 (MES) 中。客户将收到有关如何迁移工作流以将作业发送到 MES 的指示。格式转换和加密功能也可通过动态打包和动态加密提供。

## 概述

要通过 Internet 传送数字视频，你必须对媒体进行压缩。数字视频文件相当大，可能因过大而无法通过 Internet 传送或者无法在你客户的设备上正常显示。编码是压缩视频和音频以便你的客户能够查看媒体的过程。视频经过编码后即可放入不同的文件容器中。将编码后的媒体放入容器这一过程称为打包。以 MP4 文件为例，你可以使用 Azure 媒体包装器将其转换为平滑流式处理或 HLS 内容。

媒体服务支持动态和静态打包。使用静态打包时，需要以客户要求的各种格式创建内容副本。使用动态打包，你只需要创建一个包含一组自适应比特率 MP4 或平滑流文件的资产。然后，按需流式处理服务器会确保你的用户以选定的协议按清单或分段请求中的指定格式接收流。因此，你只需以单一存储格式存储文件并为其付费，然后媒体服务服务就会基于客户端的请求构建并提供相应响应。

>[AZURE.NOTE]建议使用[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)。

但是，有的方案需要静态打包：

- 验证使用外部编码器编码的自适应比特率 MP4（例如，使用第三方编码器）。

你还可以使用静态打包执行下列任务。但是，仍建议使用动态加密。

- 通过静态加密使用 PlayReady 来保护平滑流和 MPEG DASH
- 通过静态加密使用 AES-128 来保护 HLSv3
- 通过静态加密使用 PlayReady 来保护 HLSv3


## 验证使用外部编码器编码的自适应比特率 MP4

如果你要使用一组未使用媒体服务编码器编码的自适应比特率（多码率）MP4 文件，则应在进一步处理前验证这些文件。媒体服务包装程序可以验证包含一组 MP4 文件的资产，并可检查该资产是否可以打包成平滑流或 HLS。如果验证任务失败，则处理该任务的作业将完成并显示错误。用于定义验证任务的预设的 XML 可以在 [Azure 媒体包装器的任务预设](http://msdn.microsoft.com/zh-cn/library/azure/hh973635.aspx)主题中找到。

>[AZURE.NOTE]请使用媒体编码器标准版来生成内容，或使用媒体服务包装程序来验证内容，以避免运行时问题。如果按需流式处理服务器在运行时无法解析你的源文件，则你会收到 HTTP 1.1 错误“415 不支持的媒体类型”。服务器多次未能解析你的源文件会影响按需流式处理服务器的性能，并且可能会减少服务于其他请求的可用带宽。Azure 媒体服务在其按需流式处理服务上提供一个服务级别协议 (SLA)；但是，如果以上述方式滥用服务器，则无法遵循此 SLA。

本部分演示如何处理验证任务。本部分还演示如何查看完成时出现 JobStatus.Error 的作业的状态和错误消息。

若要使用媒体服务包装程序验证 MP4 文件，必须创建自己的清单 (.ism) 文件，并将其与源文件一起上载到媒体服务帐户。下面是媒体编码器标准版生成的 .ism 文件的一个示例。文件名区分大小写。另请确保 .ism 文件中的文本采用 UTF-8 编码。

	
	<?xml version="1.0" encoding="utf-8" standalone="yes"?>
	<smil xmlns="http://www.w3.org/2001/SMIL20/Language">
	  <head>
	<!-- Tells the server that these input files are MP4s – specific to Dynamic Packaging -->
	    <meta name="formats" content="mp4" /> 
	  </head>
	  <body>
	    <switch>
	      <video src="BigBuckBunny_1000.mp4" />
	      <video src="BigBuckBunny_1500.mp4" />
	      <video src="BigBuckBunny_2250.mp4" />
	      <video src="BigBuckBunny_3400.mp4" />
	      <video src="BigBuckBunny_400.mp4" />
	      <video src="BigBuckBunny_650.mp4" />
	      <audio src="BigBuckBunny_400.mp4" />
	    </switch>
	  </body>
	</smil>

创建自适应比特率 MP4 集后，便可以利用动态打包功能。动态打包允许你通过指定的协议传送流，而不需要进一步地打包。有关详细信息，请参阅[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)。

以下代码示例使用 Azure 媒体服务 .NET SDK 扩展。请确保更新代码，以指向输入 MP4 文件和 .ism 文件所在的文件夹，并指向 MediaPackager\_ValidateTask.xml 文件所在的位置。此 XML 文件在 [Azure 媒体包装器的任务预设](http://msdn.microsoft.com/zh-cn/library/azure/hh973635.aspx)主题中定义。
	
	using Microsoft.WindowsAzure.MediaServices.Client;
	using System;
	using System.Collections.Generic;
	using System.Configuration;
	using System.IO;
	using System.Linq;
	using System.Text;
	using System.Threading;
	using System.Threading.Tasks;
	using System.Xml.Linq;
	
	namespace MediaServicesStaticPackaging
	{
	    class Program
	    {
	        private static readonly string _mediaFiles =
	            Path.GetFullPath(@"../..\Media");
	
	        // The MultibitrateMP4Files folder should also
	        // contain the .ism manifest file.
	        private static readonly string _multibitrateMP4s =
	            Path.Combine(_mediaFiles, @"MultibitrateMP4Files");
	
	        // XML Configruation files path.
	        private static readonly string _configurationXMLFiles = @"../..\Configurations";
	
	        private static MediaServicesCredentials _cachedCredentials = null;
	        private static CloudMediaContext _context = null;
	
	        // Media Services account information.
	        private static readonly string _mediaServicesAccountName =
	            ConfigurationManager.AppSettings["MediaServicesAccountName"];
	        private static readonly string _mediaServicesAccountKey =
	            ConfigurationManager.AppSettings["MediaServicesAccountKey"];
	
	        static void Main(string[] args)
	        {
	            // Create and cache the Media Services credentials in a static class variable.
	            _cachedCredentials = new MediaServicesCredentials(
	                            _mediaServicesAccountName,
	                            _mediaServicesAccountKey);
	            // Use the cached credentials to create CloudMediaContext.
	            _context = new CloudMediaContext(_cachedCredentials);
	
	            // Ingest a set of multibitrate MP4s.
	            //
	            // Use the SDK extension method to create a new asset by 
	            // uploading files from a local directory.
	            IAsset multibitrateMP4sAsset = _context.Assets.CreateFromFolder(
	                _multibitrateMP4s,
	                AssetCreationOptions.None,
	                (af, p) =>
	                {
	                    Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
	                });
	
	            // Use Azure Media Packager to validate the files.
	            IAsset validatedMP4s =
	                ValidateMultibitrateMP4s(multibitrateMP4sAsset);
	
	            // Publish the asset.
	            _context.Locators.Create(
	                LocatorType.OnDemandOrigin,
	                validatedMP4s,
	                AccessPermissions.Read,
	                TimeSpan.FromDays(30));
	
	                                 // Get the streaming URLs.
	            Console.WriteLine("Smooth Streaming URL:");
	            Console.WriteLine(validatedMP4s.GetSmoothStreamingUri().ToString());
	            Console.WriteLine("MPEG DASH URL:");
	            Console.WriteLine(validatedMP4s.GetMpegDashUri().ToString());
	            Console.WriteLine("HLS URL:");
	            Console.WriteLine(validatedMP4s.GetHlsUri().ToString());
	        }
	
	        public static IAsset ValidateMultibitrateMP4s(IAsset multibitrateMP4sAsset)
	        {
	            // Set .ism as a primary file 
	            // in a multibitrate MP4 set.
	            SetISMFileAsPrimary(multibitrateMP4sAsset);
	
	            // Create a new job.
	            IJob job = _context.Jobs.Create("MP4 validation and converstion to Smooth Stream job.");
	
	            // Read the task configuration data into a string. 
	            string configMp4Validation = File.ReadAllText(Path.Combine(
	                    _configurationXMLFiles,
	                    "MediaPackager_ValidateTask.xml"));
	
	            // Get the SDK extension method to  get a reference to the Azure Media Packager.
	            IMediaProcessor processor = _context.MediaProcessors.GetLatestMediaProcessorByName(
	                MediaProcessorNames.WindowsAzureMediaPackager);
	
	            // Create a task with the conversion details, using the configuration data. 
	            ITask task = job.Tasks.AddNew("Mp4 Validation Task",
	                processor,
	                configMp4Validation,
	                TaskOptions.None);
	
	            // Specify the input asset to be validated.
	            task.InputAssets.Add(multibitrateMP4sAsset);
	
	            // Add an output asset to contain the results of the job. 
	            // This output is specified as AssetCreationOptions.None, which 
	            // means the output asset is in the clear (unencrypted). 
	            task.OutputAssets.AddNew("Validated output asset",
	                    AssetCreationOptions.None);
	
	            // Submit the job and wait until it is completed.
	            job.Submit();
	            job = job.StartExecutionProgressTask(
	                j =>
	                {
	                    Console.WriteLine("Job state: {0}", j.State);
	                    Console.WriteLine("Job progress: {0:0.##}%", j.GetOverallProgress());
	                },
	                CancellationToken.None).Result;
	          
	            // If the validation task fails and job completes with JobState.Error,
	            // display the error message and throw an exception.
	            if (job.State == JobState.Error)
	            {
	                Console.WriteLine("  Job ID: " + job.Id);
	                Console.WriteLine("  Name: " + job.Name);
	                Console.WriteLine("  State: " + job.State);
	
	                foreach (var jobTask in job.Tasks)
	                {
	                    Console.WriteLine("  Task Id: " + jobTask.Id);
	                    Console.WriteLine("  Name: " + jobTask.Name);
	                    Console.WriteLine("  Progress: " + jobTask.Progress);
	                    Console.WriteLine("  Configuration: " + jobTask.Configuration);
	                    Console.WriteLine("  Running time: " + jobTask.RunningDuration);
	                    if (jobTask.ErrorDetails != null)
	                    {
	                        foreach (var errordetail in jobTask.ErrorDetails)
	                        {
	
	                            Console.WriteLine("  Error Message:" + errordetail.Message);
	                            Console.WriteLine("  Error Code:" + errordetail.Code);
	                        }
	                    }
	                }
	                throw new Exception("The specified multi-bitrate MP4 set is not valid.");
	            }
	
	
	            return job.OutputMediaAssets[0];
	        }
	
	        static void SetISMFileAsPrimary(IAsset asset)
	        {
	            var ismAssetFiles = asset.AssetFiles.ToList().
	                Where(f => f.Name.EndsWith(".ism", StringComparison.OrdinalIgnoreCase)).ToArray();
	
	            // The following code assigns the first .ism file as the primary file in the asset.
	            // An asset should have one .ism file.  
	            ismAssetFiles.First().IsPrimary = true;
	            ismAssetFiles.First().Update();
	        }
	    }
	}

## 通过静态加密使用 PlayReady 来保护平滑流和 MPEG DASH

如果你想要通过 PlayReady 来保护你的内容，则可选择使用[动态加密](/documentation/articles/media-services-protect-with-drm/)（推荐选项）或静态加密（如本部分所述）。

本部分的示例将夹层文件（在本例中为 MP4）编码为自适应比特率 MP4 文件。然后，它将 MP4 打包为平滑流，并使用 PlayReady 对平滑流进行加密。因此，你能对平滑流或 MPEG DASH 进行流式处理。

媒体服务现在提供有用于传送 Microsoft PlayReady 许可证的服务。本文中的示例显示如何配置媒体服务 PlayReady 许可证传送服务（请参见以下代码中定义的 ConfigureLicenseDeliveryService 方法）。有关媒体服务 PlayReady 许可证传送服务的详细信息，请参阅[使用 PlayReady 动态加密和许可证传递服务](/documentation/articles/media-services-protect-with-drm/)。

>[AZURE.NOTE]若要传送使用 PlayReady 加密的 MPEG DASH，请确保通过将 useSencBox 和 adjustSubSamples 属性（在 [Azure 媒体加密器的任务预设](http://msdn.microsoft.com/zh-cn/library/azure/hh973610.aspx)主题中说明）设为 true 来使用 CENC 选项。


确保更新以下代码，以便指向输入 MP4 文件所在的文件夹，

并指向 MediaPackager\_MP4ToSmooth.xml 和 MediaEncryptor\_PlayReadyProtection.xml 文件所在的位置。MediaPackager\_MP4ToSmooth.xml 在 [Azure 媒体包装器的任务预设](http://msdn.microsoft.com/zh-cn/library/azure/hh973635.aspx)中定义，MediaEncryptor\_PlayReadyProtection.xml 在 [Azure 媒体加密器的任务预设](http://msdn.microsoft.com/zh-cn/library/azure/hh973610.aspx)主题中定义。

该示例定义的 UpdatePlayReadyConfigurationXMLFile 方法可用于动态更新 MediaEncryptor\_PlayReadyProtection.xml 文件。如果你有可用的密钥种子，则可以使用 CommonEncryption.GeneratePlayReadyContentKey 方法基于 keySeedValue 和 keyId 值生成内容密钥。

	using System;
	using System.Collections.Generic;
	using System.Configuration;
	using System.IO;
	using System.Linq;
	using System.Text;
	using System.Threading;
	using System.Threading.Tasks;
	using Microsoft.WindowsAzure.MediaServices.Client;
	using System.Xml.Linq;
	using Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization;
	using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;
	
	namespace PlayReadyStaticEncryptAndKeyDeliverySvc
	{
	    class Program
	    {
	       
	        private static readonly string _mediaFiles =
	            Path.GetFullPath(@"../..\Media");
	
	        private static readonly string _singleMP4File =
	            Path.Combine(_mediaFiles, @"BigBuckBunny.mp4");
	
	        // XML Configruation files path.
	        private static readonly string _configurationXMLFiles = @"../..\Configurations";
	
	
	        private static MediaServicesCredentials _cachedCredentials = null;
	        private static CloudMediaContext _context = null;
	
	        // Media Services account information.
	        private static readonly string _mediaServicesAccountName =
	            ConfigurationManager.AppSettings["MediaServiceAccountName"];
	        private static readonly string _mediaServicesAccountKey =
	            ConfigurationManager.AppSettings["MediaServiceAccountKey"];
	
	        static void Main(string[] args)
	        {
	            // Create and cache the Media Services credentials in a static class variable.
	            _cachedCredentials = new MediaServicesCredentials(
	                            _mediaServicesAccountName,
	                            _mediaServicesAccountKey);
	            // Use the cached credentials to create CloudMediaContext.
	            _context = new CloudMediaContext(_cachedCredentials);
	
	            // Encoding and encrypting assets //////////////////////
	            // Load a single MP4 file.
	            IAsset asset = IngestSingleMP4File(_singleMP4File, AssetCreationOptions.None);
	
	            // Encode an MP4 file to a set of multibitrate MP4s.
	            // Then, package a set of MP4s to clear Smooth Streaming.
	            IAsset clearSmoothStreamAsset =
	                ConvertMP4ToMultibitrateMP4sToSmoothStreaming(asset);
	
	            // Create a common encryption content key that is used 
	            // a) to set the key values in the MediaEncryptor_PlayReadyProtection.xml file
	            //    that is used for encryption.
	            // b) to configure the license delivery service and 
	            //
	            Guid keyId;
	            byte[] contentKey;
	
	            IContentKey key = CreateCommonEncryptionKey(out keyId, out contentKey);
	
	            // The content key authorization policy must be configured by you 
	            // and met by the client in order for the PlayReady license
	            // to be delivered to the client. 
	            // In this example the Media Services PlayReady license delivery service is used.
	            ConfigureLicenseDeliveryService(key);
	
	            // Get the Media Services PlayReady license delivery URL.
	            // This URL will be assigned to the licenseAcquisitionUrl property 
	            // of the MediaEncryptor_PlayReadyProtection.xml file.
	            Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense);
	
	            // Update the MediaEncryptor_PlayReadyProtection.xml file with the key and URL info.
	            UpdatePlayReadyConfigurationXMLFile(keyId, contentKey, acquisitionUrl);
	
	
	            // Encrypt your clear Smooth Streaming to Smooth Streaming with PlayReady.
	            IAsset outputAsset = CreateSmoothStreamEncryptedWithPlayReady(clearSmoothStreamAsset);
	
	
	            // You can use the http://smf.cloudapp.net/healthmonitor player 
	            // to test the smoothStreamURL URL.
	            string smoothStreamURL = outputAsset.GetSmoothStreamingUri().ToString();
	            Console.WriteLine("Smooth Streaming URL:");
	            Console.WriteLine(smoothStreamURL);
	
	            // You can use the http://dashif.org/reference/players/javascript/ player 
	            // to test the dashURL URL.
	            string dashURL = outputAsset.GetMpegDashUri().ToString();
	            Console.WriteLine("MPEG DASH URL:");
	            Console.WriteLine(dashURL);
	        }
	
	        /// <summary>
	        /// Creates a job with 2 tasks: 
	        /// 1 task - encodes a single MP4 to multibitrate MP4s,
	        /// 2 task - packages MP4s to Smooth Streaming.
	        /// </summary>
	        /// <returns>The output asset.</returns>
	        public static IAsset ConvertMP4ToMultibitrateMP4sToSmoothStreaming(IAsset asset)
	        {
	            // Create a new job.
	            IJob job = _context.Jobs.Create("Convert MP4 to Smooth Streaming.");
	
	            // Add task 1 - Encode single MP4 into multibitrate MP4s.
	            IAsset MP4sAsset = EncodeMP4IntoMultibitrateMP4sTask(job, asset);
	            // Add task 2 - Package a multibitrate MP4 set to Clear Smooth Stream.
	            IAsset packagedAsset = PackageMP4ToSmoothStreamingTask(job, MP4sAsset);
	
	            // Submit the job and wait until it is completed.
	            job.Submit();
	            job = job.StartExecutionProgressTask(
	                j =>
	                {
	                    Console.WriteLine("Job state: {0}", j.State);
	                    Console.WriteLine("Job progress: {0:0.##}%", j.GetOverallProgress());
	                },
	                CancellationToken.None).Result;
	
	            // Get the output asset that contains the Smooth Streaming asset.
	            return job.OutputMediaAssets[1];
	        }
	
	        /// <summary>
	        /// Encrypts Smooth Stream with PlayReady.
	        /// Then creates a Smooth Streaming Url.
	        /// </summary>
	        /// <param name="clearSmoothAsset">Asset that contains clear Smooth Streaming.</param>
	        /// <returns>The output asset.</returns>
	        public static IAsset CreateSmoothStreamEncryptedWithPlayReady(IAsset clearSmoothStreamAsset)
	        {
	            // Create a job.
	            IJob job = _context.Jobs.Create("Encrypt to PlayReady Smooth Streaming.");
	
	            // Add task 1 - Encrypt Smooth Streaming with PlayReady 
	            IAsset encryptedSmoothAsset =
	                EncryptSmoothStreamWithPlayReadyTask(job, clearSmoothStreamAsset);
	
	            // Submit the job and wait until it is completed.
	            job.Submit();
	            job = job.StartExecutionProgressTask(
	                j =>
	                {
	                    Console.WriteLine("Job state: {0}", j.State);
	                    Console.WriteLine("Job progress: {0:0.##}%", j.GetOverallProgress());
	                },
	                CancellationToken.None).Result;
	
	            // The OutputMediaAssets[0] contains the desired asset.
	            _context.Locators.Create(
	                LocatorType.OnDemandOrigin,
	                job.OutputMediaAssets[0],
	                AccessPermissions.Read,
	                TimeSpan.FromDays(30));
	
	            return job.OutputMediaAssets[0];
	        }
	
	        /// <summary>
	        /// Create a common encryption content key that is used 
	        /// to set the key values in the MediaEncryptor_PlayReadyProtection.xml file
	        /// that is used for encryption.
	        /// </summary>
	        /// <param name="keyId"></param>
	        /// <param name="contentKey"></param>
	        /// <returns></returns>
	        public static IContentKey CreateCommonEncryptionKey(out Guid keyId, out byte[] contentKey)
	        {
	            keyId = Guid.NewGuid();
	            contentKey = GetRandomBuffer(16);
	
	            IContentKey key = _context.ContentKeys.Create(
	                                    keyId,
	                                    contentKey,
	                                    "ContentKey",
	                                    ContentKeyType.CommonEncryption);
	
	            return key;
	        }
	
	        /// <summary>
	        /// Update your configuration .xml file dynamically.
	        /// </summary>
	        public static void UpdatePlayReadyConfigurationXMLFile(Guid keyId, byte[] keyValue, Uri licenseAcquisitionUrl)
	        {
	            string xmlFileName = Path.Combine(_configurationXMLFiles,
	                                        @"MediaEncryptor_PlayReadyProtection.xml");
	
	            XNamespace xmlns = "http://schemas.microsoft.com/iis/media/v4/TM/TaskDefinition#";
	
	            // Prepare the encryption task template
	            XDocument doc = XDocument.Load(xmlFileName);
	
	            var licenseAcquisitionUrlEl = doc
	                    .Descendants(xmlns + "property")
	                    .Where(p => p.Attribute("name").Value == "licenseAcquisitionUrl")
	                    .FirstOrDefault();
	            var contentKeyEl = doc
	                    .Descendants(xmlns + "property")
	                    .Where(p => p.Attribute("name").Value == "contentKey")
	                    .FirstOrDefault();
	            var keyIdEl = doc
	                    .Descendants(xmlns + "property")
	                    .Where(p => p.Attribute("name").Value == "keyId")
	                    .FirstOrDefault();
	
	            // Update the "value" property.
	            if (licenseAcquisitionUrlEl != null)
	                licenseAcquisitionUrlEl.Attribute("value").SetValue(licenseAcquisitionUrl.ToString());
	
	            if (contentKeyEl != null)
	                contentKeyEl.Attribute("value").SetValue(Convert.ToBase64String(keyValue));
	
	            if (keyIdEl != null)
	                keyIdEl.Attribute("value").SetValue(keyId);
	
	            doc.Save(xmlFileName);
	        }
	
	        /// <summary>
	        /// Uploads a single file.
	        /// </summary>
	        /// <param name="fileDir">The location of the files.</param>
	        /// <param name="assetCreationOptions">
	        ///  You can specify the following encryption options for the AssetCreationOptions.
	        ///      None:  no encryption.  
	        ///      StorageEncrypted: storage encryption. Encrypts a clear input file 
	        ///        before it is uploaded to Azure storage. 
	        ///      CommonEncryptionProtected: for Common Encryption Protected (CENC) files. 
	        ///        For example, a set of files that are already PlayReady encrypted. 
	        ///      EnvelopeEncryptionProtected: for HLS with AES encryption files.
	        ///        NOTE: The files must have been encoded and encrypted by Transform Manager. 
	        ///     </param>
	        /// <returns>Returns an asset that contains a single file.</returns>
	        /// </summary>
	        /// <returns></returns>
	        private static IAsset IngestSingleMP4File(string fileDir, AssetCreationOptions assetCreationOptions)
	        {
	            // Use the SDK extension method to create a new asset by 
	            // uploading a mezzanine file from a local path.
	            IAsset asset = _context.Assets.CreateFromFile(
	                fileDir,
	                assetCreationOptions,
	                (af, p) =>
	                {
	                    Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
	                });
	
	            return asset;
	        }
	
	        /// <summary>
	        /// Creates a task to encode to Adaptive Bitrate. 
	        /// Adds the new task to a job.
	        /// </summary>
	        /// <param name="job">The job to which to add the new task.</param>
	        /// <param name="asset">The input asset.</param>
	        /// <returns>The output asset.</returns>
	        private static IAsset EncodeMP4IntoMultibitrateMP4sTask(IJob job, IAsset asset)
	        {
	            // Get the SDK extension method to  get a reference to the Media Encoder Standard.
	            IMediaProcessor encoder = _context.MediaProcessors.GetLatestMediaProcessorByName(
	                MediaProcessorNames.MediaEncoderStandard);
	
	            ITask adpativeBitrateTask = job.Tasks.AddNew("MP4 to Adaptive Bitrate Task",
	               encoder,
	               "H264 Multiple Bitrate 720p",
	               TaskOptions.None);
	
	            // Specify the input Asset
	            adpativeBitrateTask.InputAssets.Add(asset);
	
	            // Add an output asset to contain the results of the job. 
	            // This output is specified as AssetCreationOptions.None, which 
	            // means the output asset is in the clear (unencrypted).
	            IAsset abrAsset = adpativeBitrateTask.OutputAssets.AddNew("Multibitrate MP4s",
	                                    AssetCreationOptions.None);
	
	            return abrAsset;
	        }
	
	        /// <summary>
	        /// Creates a task to convert the MP4 file(s) to a Smooth Streaming asset.
	        /// Adds the new task to a job.
	        /// </summary>
	        /// <param name="job">The job to which to add the new task.</param>
	        /// <param name="asset">The input asset.</param>
	        /// <returns>The output asset.</returns>
	        private static IAsset PackageMP4ToSmoothStreamingTask(IJob job, IAsset asset)
	        {
	            // Get the SDK extension method to  get a reference to the Azure Media Packager.
	            IMediaProcessor packager = _context.MediaProcessors.GetLatestMediaProcessorByName(
	                MediaProcessorNames.WindowsAzureMediaPackager);
	
	            // Azure Media Packager does not accept string presets, so load xml configuration
	            string smoothConfig = File.ReadAllText(Path.Combine(
	                        _configurationXMLFiles,
	                        "MediaPackager_MP4toSmooth.xml"));
	
	            // Create a new Task to convert adaptive bitrate to Smooth Streaming.
	            ITask smoothStreamingTask = job.Tasks.AddNew("MP4 to Smooth Task",
	               packager,
	               smoothConfig,
	               TaskOptions.None);
	
	            // Specify the input Asset, which is the output Asset from the first task
	            smoothStreamingTask.InputAssets.Add(asset);
	
	            // Add an output asset to contain the results of the job. 
	            // This output is specified as AssetCreationOptions.None, which 
	            // means the output asset is in the clear (unencrypted).
	            IAsset smoothOutputAsset =
	                smoothStreamingTask.OutputAssets.AddNew("Clear Smooth Stream",
	                    AssetCreationOptions.None);
	
	            return smoothOutputAsset;
	        }
	
	
	        /// <summary>
	        /// Creates a task to encrypt Smooth Streaming with PlayReady.
	        /// Note: To deliver DASH, make sure to set the useSencBox and adjustSubSamples 
	        /// configuration properties to true. 
	        /// In this example, MediaEncryptor_PlayReadyProtection.xml contains configuration.
	        /// </summary>
	        /// <param name="job">The job to which to add the new task.</param>
	        /// <param name="asset">The input asset.</param>
	        /// <returns>The output asset.</returns>
	        private static IAsset EncryptSmoothStreamWithPlayReadyTask(IJob job, IAsset asset)
	        {
	            // Get the SDK extension method to  get a reference to the Azure Media Encryptor.
	            IMediaProcessor playreadyProcessor = _context.MediaProcessors.GetLatestMediaProcessorByName(
	                MediaProcessorNames.WindowsAzureMediaEncryptor);
	
	            // Read the configuration XML.
	            //
	            // Note that the configuration defined in MediaEncryptor_PlayReadyProtection.xml
	            // is using keySeedValue. It is recommended that you do this only for testing 
	            // and not in production. For more information, see 
	            // http://www.windowsazure.cn/documentation/articles/media-services-static-packaging.
	            //
	            string configPlayReady = File.ReadAllText(Path.Combine(_configurationXMLFiles,
	                                        @"MediaEncryptor_PlayReadyProtection.xml"));
	
	            ITask playreadyTask = job.Tasks.AddNew("My PlayReady Task",
	               playreadyProcessor,
	               configPlayReady,
	               TaskOptions.ProtectedConfiguration);
	
	            playreadyTask.InputAssets.Add(asset);
	
	            // Add an output asset to contain the results of the job. 
	            // This output is specified as AssetCreationOptions.CommonEncryptionProtected.
	            IAsset playreadyAsset = playreadyTask.OutputAssets.AddNew(
	                                            "PlayReady Smooth Streaming",
	                                            AssetCreationOptions.CommonEncryptionProtected);
	
	            return playreadyAsset;
	        }
	
	        /// <summary>
	        /// Configures authorization policy for the content key. 
	        /// </summary>
	        /// <param name="contentKey">The content key.</param>
	        static public void ConfigureLicenseDeliveryService(IContentKey contentKey)
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
	
	            // Configure PlayReady license template.
	            string newLicenseTemplate = ConfigurePlayReadyLicenseTemplate();
	
	            IContentKeyAuthorizationPolicyOption policyOption =
	                _context.ContentKeyAuthorizationPolicyOptions.Create("",
	                    ContentKeyDeliveryType.PlayReadyLicense,
	                        restrictions, newLicenseTemplate);
	
	            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
	                        ContentKeyAuthorizationPolicies.
	                        CreateAsync("Deliver Common Content Key with no restrictions").
	                        Result;
	
	
	            contentKeyAuthorizationPolicy.Options.Add(policyOption);
	
	            // Associate the content key authorization policy with the content key.
	            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
	            contentKey = contentKey.UpdateAsync().Result;
	        }
	
	        static private string ConfigurePlayReadyLicenseTemplate()
	        {
	            // The following code configures PlayReady License Template using .NET classes
	            // and returns the XML string.
	
	            PlayReadyLicenseResponseTemplate responseTemplate = new PlayReadyLicenseResponseTemplate();
	            PlayReadyLicenseTemplate licenseTemplate = new PlayReadyLicenseTemplate();
	
	            responseTemplate.LicenseTemplates.Add(licenseTemplate);
	
	            return MediaServicesLicenseTemplateSerializer.Serialize(responseTemplate);
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

## 通过静态加密使用 AES-128 来保护 HLSv3

如果你要使用 AES-128 加密 HLS，可以选择使用动态加密（推荐选项）或静态加密（如本部分所述）。如果你决定使用动态加密，请参阅[使用 AES-128 动态加密和密钥传递服务](/documentation/articles/media-services-protect-with-aes128/)。

>[AZURE.NOTE]若要将内容转换为 HLS，必须先将内容转换/编码为平滑流。此外，对于使用 AES 加密的 HLS，请确保在 MediaPackager\_SmoothToHLS.xml 文件中设置以下属性：将加密属性设置为 true，将密钥值和 keyuri 值设置为指向身份验证\\授权服务器。媒体服务将创建密钥文件，并将其放置在资产容器中。你应该将 /asset-containerguid/*.key 文件复制到服务器（或创建你自己的密钥文件），然后从资产容器中删除 *.key 文件。

本部分的示例将夹层文件（在本例中为 MP4）编码为多比特率 MP4 文件，然后将 MP4 打包为平滑流。然后，它将平滑流打包成使用高级加密标准 (AES) 128 位流加密法加密的 HTTP 实时流 (HLS)。确保更新以下代码，以便指向输入 MP4 文件所在的文件夹，并指向 MediaPackager\_MP4ToSmooth.xml 和 MediaPackager\_SmoothToHLS.xml 配置文件所在的位置。可以在 [Azure 媒体包装器的任务预设](http://msdn.microsoft.com/zh-cn/library/azure/hh973635.aspx)主题中找到这些文件的定义。
	
	using System;
	using System.Collections.Generic;
	using System.Configuration;
	using System.IO;
	using System.Linq;
	using System.Text;
	using System.Threading;
	using System.Threading.Tasks;
	using Microsoft.WindowsAzure.MediaServices.Client;
	using System.Xml.Linq;
	
	namespace MediaServicesContentProtection
	{
	    class Program
	    {
	        // Paths to support files (within the above base path). You can use 
	        // the provided sample media files from the "SupportFiles" folder, or 
	        // provide paths to your own media files below to run these samples.
	
	        private static readonly string _mediaFiles =
	            Path.GetFullPath(@"../..\Media");
	        
	        private static readonly string _singleMP4File =
	            Path.Combine(_mediaFiles, @"SingleMP4\BigBuckBunny.mp4");
	
	        // XML Configruation files path.
	        private static readonly string _configurationXMLFiles = @"../..\Configurations";
	
	        private static MediaServicesCredentials _cachedCredentials = null;
	        private static CloudMediaContext _context = null;
	
	        // Media Services account information.
	        private static readonly string _mediaServicesAccountName = 
	            ConfigurationManager.AppSettings["MediaServiceAccountName"];
	        private static readonly string _mediaServicesAccountKey = 
	            ConfigurationManager.AppSettings["MediaServiceAccountKey"];
	
	        static void Main(string[] args)
	        {
	            // Create and cache the Media Services credentials in a static class variable.
	            _cachedCredentials = new MediaServicesCredentials(
	                            _mediaServicesAccountName, 
	                            _mediaServicesAccountKey);
	            // Use the cached credentials to create CloudMediaContext.
	            _context = new CloudMediaContext(_cachedCredentials);
	
	            // Encoding and encrypting assets //////////////////////
	
	            // Load an MP4 file.
	            IAsset asset = IngestSingleMP4File(_singleMP4File, AssetCreationOptions.None);
	
	            // Encode an MP4 file to a set of multibitrate MP4s.
	            // Then, package a set of MP4s to clear Smooth Streaming.
	            IAsset clearSmoothStreamAsset = ConvertMP4ToMultibitrateMP4sToSmoothStreaming(asset);
	
	            // Create HLS encrypted with AES.
	            IAsset HLSEncryptedWithAESAsset = CreateHLSEncryptedWithAES(clearSmoothStreamAsset);
	
	            // You can use the following player to test the HLS with AES stream.
	            // http://apps.microsoft.com/windows/app/3ivx-hls-player/f79ce7d0-2993-4658-bc4e-83dc182a0614 
	            string hlsWithAESURL = HLSEncryptedWithAESAsset.GetHlsUri().ToString();
	            Console.WriteLine("HLS with AES URL:");
	            Console.WriteLine(hlsWithAESURL);
	        }
	
	
	        /// <summary>
	        /// Creates a job with 2 tasks: 
	        /// 1 task - encodes a single MP4 to multibitrate MP4s,
	        /// 2 task - packages MP4s to Smooth Streaming.
	        /// </summary>
	        /// <returns>The output asset.</returns>
	        public static IAsset ConvertMP4ToMultibitrateMP4sToSmoothStreaming(IAsset asset)
	        {
	            // Create a new job.
	            IJob job = _context.Jobs.Create("Convert MP4 to Smooth Streaming.");
	
	            // Add task 1 - Encode single MP4 into multibitrate MP4s.
	            IAsset MP4sAsset = EncodeSingleMP4IntoMultibitrateMP4sTask(job, asset);
	            // Add task 2 - Package a multibitrate MP4 set to Clear Smooth Streaming.
	            IAsset packagedAsset = PackageMP4ToSmoothStreamingTask(job, MP4sAsset);
	
	            // Submit the job and wait until it is completed.
	            job.Submit();
	            job = job.StartExecutionProgressTask(
	                j =>
	                {
	                    Console.WriteLine("Job state: {0}", j.State);
	                    Console.WriteLine("Job progress: {0:0.##}%", j.GetOverallProgress());
	                },
	                CancellationToken.None).Result;
	
	            // Get the output asset that contains Smooth Streaming.
	            return job.OutputMediaAssets[1];
	        }
	
	        /// <summary>
	        /// Encrypts an HLS with AES-128.
	        /// </summary>
	        /// <param name="clearSmoothAsset">Asset that contains clear Smooth Streaming.</param>
	        /// <returns>The output asset.</returns>
	        public static IAsset CreateHLSEncryptedWithAES(IAsset clearSmoothStreamAsset)
	        {
	            IJob job = _context.Jobs.Create("Encrypt to HLS with AES.");
	
	            // Add task 1 - Package clear Smooth Streaming to HLS with AES.
	            PackageSmoothStreamToHLS(job, clearSmoothStreamAsset);
	
	            // Submit the job and wait until it is completed.
	            job.Submit();
	            job = job.StartExecutionProgressTask(
	                j =>
	                {
	                    Console.WriteLine("Job state: {0}", j.State);
	                    Console.WriteLine("Job progress: {0:0.##}%", j.GetOverallProgress());
	                },
	                CancellationToken.None).Result;
	
	            // The OutputMediaAssets[0] contains the desired asset.
	            _context.Locators.Create(
	                LocatorType.OnDemandOrigin,
	                job.OutputMediaAssets[0],
	                AccessPermissions.Read,
	                TimeSpan.FromDays(30));
	
	            return job.OutputMediaAssets[0];
	        }
	
	        /// <summary>
	        /// Uploads a single file.
	        /// </summary>
	        /// <param name="fileDir">The location of the files.</param>
	        /// <param name="assetCreationOptions">
	        ///  You can specify the following encryption options for the AssetCreationOptions.
	        ///      None:  no encryption.  
	        ///      StorageEncrypted: storage encryption. Encrypts a clear input file 
	        ///        before it is uploaded to Azure storage. 
	        ///      CommonEncryptionProtected: for Common Encryption Protected (CENC) files. 
	        ///        For example, a set of files that are already PlayReady encrypted. 
	        ///      EnvelopeEncryptionProtected: for HLS with AES encryption files.
	        ///        NOTE: The files must have been encoded and encrypted by Transform Manager. 
	        ///     </param>
	        /// <returns>Returns an asset that contains a single file.</returns>
	        /// </summary>
	        /// <returns></returns>
	        private static IAsset IngestSingleMP4File(string fileDir, AssetCreationOptions assetCreationOptions)
	        {
	            // Use the SDK extension method to create a new asset by 
	            // uploading a mezzanine file from a local path.
	            IAsset asset = _context.Assets.CreateFromFile(
	                fileDir,
	                assetCreationOptions,
	                (af, p) =>
	                {
	                    Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
	                });
	 
	            return asset;
	        }
	
	        /// <summary>
	        /// Creates a task to encode to Adaptive Bitrate. 
	        /// Adds the new task to a job.
	        /// </summary>
	        /// <param name="job">The job to which to add the new task.</param>
	        /// <param name="asset">The input asset.</param>
	        /// <returns>The output asset.</returns>
	        private static IAsset EncodeSingleMP4IntoMultibitrateMP4sTask(IJob job, IAsset asset)
	        {
	            // Get the SDK extension method to  get a reference to the Media Encoder Standard.
	            IMediaProcessor encoder = _context.MediaProcessors.GetLatestMediaProcessorByName(
	                MediaProcessorNames.MediaEncoderStandard);
	
	            ITask adpativeBitrateTask = job.Tasks.AddNew("MP4 to Adaptive Bitrate Task",
	               encoder,
	               "H264 Multiple Bitrate 720p",
	               TaskOptions.None);
	
	            // Specify the input Asset
	            adpativeBitrateTask.InputAssets.Add(asset);
	
	            // Add an output asset to contain the results of the job. 
	            // This output is specified as AssetCreationOptions.None, which 
	            // means the output asset is in the clear (unencrypted).
	            IAsset abrAsset = adpativeBitrateTask.OutputAssets.AddNew("Multibitrate MP4s", 
	                                    AssetCreationOptions.None);
	
	            return abrAsset;
	        }
	
	        /// <summary>
	        /// Creates a task to convert the MP4 file(s) to a Smooth Streaming asset.
	        /// Adds the new task to a job.
	        /// </summary>
	        /// <param name="job">The job to which to add the new task.</param>
	        /// <param name="asset">The input asset.</param>
	        /// <returns>The output asset.</returns>
	        private static IAsset PackageMP4ToSmoothStreamingTask(IJob job, IAsset asset)
	        {
	            // Get the SDK extension method to  get a reference to the Azure Media Packager.
	            IMediaProcessor packager = _context.MediaProcessors.GetLatestMediaProcessorByName(
	                MediaProcessorNames.WindowsAzureMediaPackager);
	
	            // Azure Media Packager does not accept string presets, so load xml configuration
	            string smoothConfig = File.ReadAllText(Path.Combine(
	                        _configurationXMLFiles, 
	                        "MediaPackager_MP4toSmooth.xml"));
	
	            // Create a new Task to convert adaptive bitrate to Smooth Streaming.
	            ITask smoothStreamingTask = job.Tasks.AddNew("MP4 to Smooth Task",
	               packager,
	               smoothConfig,
	               TaskOptions.None);
	
	            // Specify the input Asset, which is the output Asset from the first task
	            smoothStreamingTask.InputAssets.Add(asset);
	
	            // Add an output asset to contain the results of the job. 
	            // This output is specified as AssetCreationOptions.None, which 
	            // means the output asset is in the clear (unencrypted).
	            IAsset smoothOutputAsset = 
	                smoothStreamingTask.OutputAssets.AddNew("Clear Smooth Streaming", 
	                    AssetCreationOptions.None);
	
	            return smoothOutputAsset;
	        }
	
	        /// <summary>
	        /// Converts Smooth Streaming to HLS.
	        /// </summary>
	        /// <param name="job">The job to which to add the new task.</param>
	        /// <param name="asset">The Smooth Streaming asset.</param>
	        /// <returns>The asset that was packaged to HLS.</returns>
	        private static IAsset PackageSmoothStreamToHLS(IJob job, IAsset smoothStreamAsset)
	        {
	            // Get the SDK extension method to  get a reference to the Azure Media Packager.
	            IMediaProcessor processor = _context.MediaProcessors.GetLatestMediaProcessorByName(
	                MediaProcessorNames.WindowsAzureMediaPackager);
	
	            // Read the configuration data into a string. 
	            // For the HLS to get encrypted with AES make sure to set the
	            // encrypt configuration property to true.
	            //
	            // In production, it is recommended to do the following:
	            //    Set a Key url for your authn/authz server.
	            //    Copy the /asset-containerguid/*.key file to your server (or craft a key file for yourself).
	            //    Delete *.key from the asset container.
	            //
	            string configuration = File.ReadAllText(Path.Combine(_configurationXMLFiles, @"MediaPackager_SmoothToHLS.xml"));
	
	            // Create a task with the encoding details, using a configuration file.
	            ITask task = job.Tasks.AddNew("My Smooth Streaming to HLS Task",
	               processor,
	               configuration,
	               TaskOptions.ProtectedConfiguration);
	
	            // Specify the input asset to be encoded.
	            task.InputAssets.Add(smoothStreamAsset);
	
	            // Add an output asset to contain the results of the job. 
	            IAsset outputAsset = 
	                task.OutputAssets.AddNew("HLS asset", AssetCreationOptions.None);
	
	
	            return outputAsset;
	        }
	    }
	}

## 通过静态加密使用 PlayReady 来保护 HLSv3

如果你想要通过 PlayReady 来保护你的内容，则可选择使用[动态加密](/documentation/articles/media-services-protect-with-drm/)（推荐选项）或静态加密（如本部分所述）。

>[AZURE.NOTE] 若要使用 PlayReady 保护你的内容，必须先将内容转换/编码为平滑流格式。

本部分的示例将夹层文件（在本例中为 MP4）编码为多比特率 MP4 文件。然后，它将 MP4 打包为平滑流，并使用 PlayReady 对平滑流进行加密。若要生成使用 PlayReady 加密的 HTTP 实时流 (HLS)，需要将 PlayReady 平滑流资产打包成 HLS。本主题演示如何执行所有这些步骤。

媒体服务现在提供有用于传送 Microsoft PlayReady 许可证的服务。本文中的示例显示如何配置媒体服务 PlayReady 许可证传送服务（请参见以下代码中定义的 **ConfigureLicenseDeliveryService** 方法）。

确保更新以下代码，以便指向输入 MP4 文件所在的文件夹，并指向 MediaPackager\_MP4ToSmooth.xml、MediaPackager\_SmoothToHLS.xml 和 MediaEncryptor\_PlayReadyProtection.xml 文件所在的位置。MediaPackager\_MP4ToSmooth.xml 和 MediaPackager\_SmoothToHLS.xml 在 [Azure 媒体包装器的任务预设](http://msdn.microsoft.com/zh-cn/library/azure/hh973635.aspx)中定义，MediaEncryptor\_PlayReadyProtection.xml 在 [Azure 媒体加密器的任务预设](http://msdn.microsoft.com/zh-cn/library/azure/hh973610.aspx)主题中定义。
	
	using System;
	using System.Collections.Generic;
	using System.Configuration;
	using System.IO;
	using System.Linq;
	using System.Text;
	using System.Threading;
	using System.Threading.Tasks;
	using Microsoft.WindowsAzure.MediaServices.Client;
	using System.Xml.Linq;
	using Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization;
	using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;
	
	namespace MediaServicesContentProtection
	{
	    class Program
	    {
	        // Paths to support files (within the above base path). You can use 
	        // the provided sample media files from the "SupportFiles" folder, or 
	        // provide paths to your own media files below to run these samples.
	
	        private static readonly string _mediaFiles =
	            Path.GetFullPath(@"../..\Media");
	
	        private static readonly string _singleMP4File =
	            Path.Combine(_mediaFiles, @"SingleMP4\BigBuckBunny.mp4");
	
	        // XML Configruation files path.
	        private static readonly string _configurationXMLFiles = @"../..\Configurations";
	
	
	        private static MediaServicesCredentials _cachedCredentials = null;
	        private static CloudMediaContext _context = null;
	
	        // Media Services account information.
	        private static readonly string _mediaServicesAccountName =
	            ConfigurationManager.AppSettings["MediaServiceAccountName"];
	        private static readonly string _mediaServicesAccountKey =
	            ConfigurationManager.AppSettings["MediaServiceAccountKey"];
	
	        static void Main(string[] args)
	        {
	            // Create and cache the Media Services credentials in a static class variable.
	            _cachedCredentials = new MediaServicesCredentials(
	                            _mediaServicesAccountName,
	                            _mediaServicesAccountKey);
	            // Used the chached credentials to create CloudMediaContext.
	            _context = new CloudMediaContext(_cachedCredentials);
	
	            // Load an MP4 file.
	            IAsset asset = IngestSingleMP4File(_singleMP4File, AssetCreationOptions.None);
	
	            // Encode an MP4 file to a set of multibitrate MP4s.
	            // Then, package a set of MP4s to clear Smooth Streaming.
	            IAsset clearSmoothStreamAsset = ConvertMP4ToMultibitrateMP4sToSmoothStreaming(asset);
	
	            // Create a common encryption content key that is used 
	            // a) to set the key values in the MediaEncryptor_PlayReadyProtection.xml file
	            //    that is used for encryption.
	            // b) to configure the license delivery service and 
	            //
	            Guid keyId;
	            byte[] contentKey;
	
	            IContentKey key = CreateCommonEncryptionKey(out keyId, out contentKey);
	
	            // The content key authorization policy must be configured by you 
	            // and met by the client in order for the PlayReady license
	            // to be delivered to the client. 
	            // In this example the Media Services PlayReady license delivery service is used.
	            ConfigureLicenseDeliveryService(key);
	
	            // Get the Media Services PlayReady license delivery URL.
	            // This URL will be assigned to the licenseAcquisitionUrl property 
	            // of the MediaEncryptor_PlayReadyProtection.xml file.
	            Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense);
	
	            // Update the MediaEncryptor_PlayReadyProtection.xml file with the key and URL info.
	            UpdatePlayReadyConfigurationXMLFile(keyId, contentKey, acquisitionUrl);
	
	            // Create HLS encrypted with PlayReady.
	            IAsset playReadyHLSAsset = CreateHLSEncryptedWithPlayReady(clearSmoothStreamAsset);
	            //
	            string hlsWithPlayReadyURL = playReadyHLSAsset.GetHlsUri().ToString();
	            Console.WriteLine("HLS with PlayReady URL:");
	            Console.WriteLine(hlsWithPlayReadyURL);
	        }
	
	        /// <summary>
	        /// Creates a job with 2 tasks: 
	        /// 1 task - encodes a single MP4 to multibitrate MP4s,
	        /// 2 task - packages MP4s to Smooth Streaming.
	        /// </summary>
	        /// <returns>The output asset.</returns>
	        public static IAsset ConvertMP4ToMultibitrateMP4sToSmoothStreaming(IAsset asset)
	        {
	            // Create a new job.
	            IJob job = _context.Jobs.Create("Convert MP4 to Smooth Streaming.");
	
	            // Add task 1 - Encode single MP4 into multibitrate MP4s.
	            IAsset MP4sAsset = EncodeSingleMP4IntoMultibitrateMP4sTask(job, asset);
	            // Add task 2 - Package a multibitrate MP4 set to Clear Smooth Streaming.
	            IAsset packagedAsset = PackageMP4ToSmoothStreamingTask(job, MP4sAsset);
	
	            // Submit the job and wait until it is completed.
	            job.Submit();
	            job = job.StartExecutionProgressTask(
	                j =>
	                {
	                    Console.WriteLine("Job state: {0}", j.State);
	                    Console.WriteLine("Job progress: {0:0.##}%", j.GetOverallProgress());
	                },
	                CancellationToken.None).Result;
	
	            // Get the output asset that contains Smooth Streaming.
	            return job.OutputMediaAssets[1];
	        }
	
	        /// <summary>
	        /// Create a common encryption content key that is used 
	        /// to set the key values in the MediaEncryptor_PlayReadyProtection.xml file
	        /// that is used for encryption.
	        /// </summary>
	        /// <param name="keyId"></param>
	        /// <param name="contentKey"></param>
	        /// <returns></returns>
	        public static IContentKey CreateCommonEncryptionKey(out Guid keyId, out byte[] contentKey)
	        {
	            keyId = Guid.NewGuid();
	            contentKey = GetRandomBuffer(16);
	
	            IContentKey key = _context.ContentKeys.Create(
	                                    keyId,
	                                    contentKey,
	                                    "ContentKey",
	                                    ContentKeyType.CommonEncryption);
	
	            return key;
	        }
	
	        /// <summary>
	        /// Update your configuration .xml file dynamically.
	        /// </summary>
	        public static void UpdatePlayReadyConfigurationXMLFile(Guid keyId, byte[] keyValue, Uri licenseAcquisitionUrl)
	        {
	            string xmlFileName = Path.Combine(_configurationXMLFiles,
	                                        @"MediaEncryptor_PlayReadyProtection.xml");
	
	            XNamespace xmlns = "http://schemas.microsoft.com/iis/media/v4/TM/TaskDefinition#";
	
	            // Prepare the encryption task template
	            XDocument doc = XDocument.Load(xmlFileName);
	
	            var licenseAcquisitionUrlEl = doc
	                    .Descendants(xmlns + "property")
	                    .Where(p => p.Attribute("name").Value == "licenseAcquisitionUrl")
	                    .FirstOrDefault();
	            var contentKeyEl = doc
	                    .Descendants(xmlns + "property")
	                    .Where(p => p.Attribute("name").Value == "contentKey")
	                    .FirstOrDefault();
	            var keyIdEl = doc
	                    .Descendants(xmlns + "property")
	                    .Where(p => p.Attribute("name").Value == "keyId")
	                    .FirstOrDefault();
	
	            // Update the "value" property.
	            if (licenseAcquisitionUrlEl != null)
	                licenseAcquisitionUrlEl.Attribute("value").SetValue(licenseAcquisitionUrl.ToString());
	
	            if (contentKeyEl != null)
	                contentKeyEl.Attribute("value").SetValue(Convert.ToBase64String(keyValue));
	
	            if (keyIdEl != null)
	                keyIdEl.Attribute("value").SetValue(keyId);
	
	            doc.Save(xmlFileName);
	        }
	
	        /// <summary>
	        // Encrypts clear Smooth Streaming to Smooth Streaming with PlayReady.
	        // Then, packages the PlayReady Smooth Streaming to HLS with PlayReady.
	        /// </summary>
	        /// <param name="clearSmoothAsset">Asset that contains clear Smooth Streaming.</param>
	        /// <returns>The output asset.</returns>
	        public static IAsset CreateHLSEncryptedWithPlayReady(IAsset clearSmoothStreamAsset)
	        {
	            IJob job = _context.Jobs.Create("Encrypt to HLS with PlayReady.");
	
	            // Add task 1 - Encrypt Smooth Streaming with PlayReady 
	            IAsset encryptedSmoothAsset =
	                EncryptSmoothStreamWithPlayReadyTask(job, clearSmoothStreamAsset);
	
	            // Add task 2 - Package to HLS with PlayReady.
	            PackageSmoothStreamToHLS(job, encryptedSmoothAsset);
	
	            // Submit the job and wait until it is completed.
	            job.Submit();
	            job = job.StartExecutionProgressTask(
	                j =>
	                {
	                    Console.WriteLine("Job state: {0}", j.State);
	                    Console.WriteLine("Job progress: {0:0.##}%", j.GetOverallProgress());
	                },
	                CancellationToken.None).Result;
	
	            // Since we had two tasks, the OutputMediaAssets[1]
	            // contains the desired asset.
	            _context.Locators.Create(
	                LocatorType.OnDemandOrigin,
	                job.OutputMediaAssets[1],
	                AccessPermissions.Read,
	                TimeSpan.FromDays(30));
	
	            return job.OutputMediaAssets[1];
	        }
	
	        /// <summary>
	        /// Uploads a single file.
	        /// </summary>
	        /// <param name="fileDir">The location of the files.</param>
	        /// <param name="assetCreationOptions">
	        ///  You can specify the following encryption options for the AssetCreationOptions.
	        ///      None:  no encryption.  
	        ///      StorageEncrypted: storage encryption. Encrypts a clear input file 
	        ///        before it is uploaded to Azure storage. 
	        ///      CommonEncryptionProtected: for Common Encryption Protected (CENC) files. 
	        ///        For example, a set of files that are already PlayReady encrypted. 
	        ///      EnvelopeEncryptionProtected: for HLS with AES encryption files.
	        ///        NOTE: The files must have been encoded and encrypted by Transform Manager. 
	        ///     </param>
	        /// <returns>Returns an asset that contains a single file.</returns>
	        /// </summary>
	        /// <returns></returns>
	        private static IAsset IngestSingleMP4File(string fileDir, AssetCreationOptions assetCreationOptions)
	        {
	            // Use the SDK extension method to create a new asset by 
	            // uploading a mezzanine file from a local path.
	            IAsset asset = _context.Assets.CreateFromFile(
	                fileDir,
	                assetCreationOptions,
	                (af, p) =>
	                {
	                    Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
	                });
	
	
	            return asset;
	
	        }
	        /// <summary>
	        /// Creates a task to encode to Adaptive Bitrate. 
	        /// Adds the new task to a job.
	        /// </summary>
	        /// <param name="job">The job to which to add the new task.</param>
	        /// <param name="asset">The input asset.</param>
	        /// <returns>The output asset.</returns>
	        private static IAsset EncodeSingleMP4IntoMultibitrateMP4sTask(IJob job, IAsset asset)
	        {
	            // Get the SDK extension method to  get a reference to the Media Encoder Standard.
	            IMediaProcessor encoder = _context.MediaProcessors.GetLatestMediaProcessorByName(
	                MediaProcessorNames.MediaEncoderStandard);
	
	            ITask adpativeBitrateTask = job.Tasks.AddNew("MP4 to Adaptive Bitrate Task",
	               encoder,
	               "H264 Multiple Bitrate 720p",
	               TaskOptions.None);
	
	            // Specify the input Asset
	            adpativeBitrateTask.InputAssets.Add(asset);
	
	            // Add an output asset to contain the results of the job. 
	            // This output is specified as AssetCreationOptions.None, which 
	            // means the output asset is in the clear (unencrypted).
	            IAsset abrAsset = adpativeBitrateTask.OutputAssets.AddNew("Multibitrate MP4s",
	                                    AssetCreationOptions.None);
	
	            return abrAsset;
	        }
	
	        /// <summary>
	        /// Creates a task to convert the MP4 file(s) to a Smooth Streaming asset.
	        /// Adds the new task to a job.
	        /// </summary>
	        /// <param name="job">The job to which to add the new task.</param>
	        /// <param name="asset">The input asset.</param>
	        /// <returns>The output asset.</returns>
	        private static IAsset PackageMP4ToSmoothStreamingTask(IJob job, IAsset asset)
	        {
	            // Get the SDK extension method to  get a reference to the Azure Media Packager.
	            IMediaProcessor packager = _context.MediaProcessors.GetLatestMediaProcessorByName(
	                MediaProcessorNames.WindowsAzureMediaPackager);
	
	            // Azure Media Packager does not accept string presets, so load xml configuration
	            string smoothConfig = File.ReadAllText(Path.Combine(
	                        _configurationXMLFiles,
	                        "MediaPackager_MP4toSmooth.xml"));
	
	            // Create a new Task to convert adaptive bitrate to Smooth Streaming.
	            ITask smoothStreamingTask = job.Tasks.AddNew("MP4 to Smooth Task",
	               packager,
	               smoothConfig,
	               TaskOptions.None);
	
	            // Specify the input Asset, which is the output Asset from the first task
	            smoothStreamingTask.InputAssets.Add(asset);
	
	            // Add an output asset to contain the results of the job. 
	            // This output is specified as AssetCreationOptions.None, which 
	            // means the output asset is in the clear (unencrypted).
	            IAsset smoothOutputAsset =
	                smoothStreamingTask.OutputAssets.AddNew("Clear Smooth Streaming",
	                    AssetCreationOptions.None);
	
	            return smoothOutputAsset;
	        }
	
	
	        /// <summary>
	        /// Converts Smooth Stream to HLS.
	        /// </summary>
	        /// <param name="job">The job to which to add the new task.</param>
	        /// <param name="asset">The Smooth Stream asset.</param>
	        /// <returns>The asset that was packaged to HLS.</returns>
	        private static IAsset PackageSmoothStreamToHLS(IJob job, IAsset smoothStreamAsset)
	        {
	            // Get the SDK extension method to  get a reference to the Azure Media Packager.
	            IMediaProcessor processor = _context.MediaProcessors.GetLatestMediaProcessorByName(
	                MediaProcessorNames.WindowsAzureMediaPackager);
	
	            // Read the configuration data into a string. 
	            //
	            string configuration = File.ReadAllText(
	                        Path.Combine(_configurationXMLFiles,
	                                    @"MediaPackager_SmoothToHLS.xml"));
	
	            // Create a task with the encoding details, using a configuration file.
	            ITask task = job.Tasks.AddNew("My Smooth to HLS Task",
	               processor,
	               configuration,
	               TaskOptions.ProtectedConfiguration);
	
	            // Specify the input asset to be encoded.
	            task.InputAssets.Add(smoothStreamAsset);
	
	            // Add an output asset to contain the results of the job. 
	            IAsset outputAsset =
	                task.OutputAssets.AddNew("HLS asset", AssetCreationOptions.None);
	
	
	            return outputAsset;
	        }
	
	        /// <summary>
	        /// Creates a task to encrypt Smooth Streaming with PlayReady.
	        /// Note: Do deliver DASH, make sure to set the useSencBox and adjustSubSamples 
	        /// configuration properties to true.
	        /// </summary>
	        /// <param name="job">The job to which to add the new task.</param>
	        /// <param name="asset">The input asset.</param>
	        /// <returns>The output asset.</returns>
	        private static IAsset EncryptSmoothStreamWithPlayReadyTask(IJob job, IAsset asset)
	        {
	            // Get the SDK extension method to  get a reference to the Azure Media Encryptor.
	            IMediaProcessor playreadyProcessor = _context.MediaProcessors.GetLatestMediaProcessorByName(
	                MediaProcessorNames.WindowsAzureMediaEncryptor);
	
	            // Read the configuration XML.
	            //
	            // Note that the configuration defined in MediaEncryptor_PlayReadyProtection.xml
	            // is using keySeedValue. It is recommended that you do this only for testing 
	            // and not in production. For more information, see 
	            // http://www.windowsazure.cn/documentation/articles/media-services-static-packaging.
	            //
	            string configPlayReady = File.ReadAllText(Path.Combine(_configurationXMLFiles,
	                                        @"MediaEncryptor_PlayReadyProtection.xml"));
	
	            ITask playreadyTask = job.Tasks.AddNew("My PlayReady Task",
	               playreadyProcessor,
	               configPlayReady,
	               TaskOptions.ProtectedConfiguration);
	
	            playreadyTask.InputAssets.Add(asset);
	
	            // Add an output asset to contain the results of the job. 
	            // This output is specified as AssetCreationOptions.CommonEncryptionProtected.
	            IAsset playreadyAsset = playreadyTask.OutputAssets.AddNew(
	                                            "PlayReady Smooth Streaming",
	                                            AssetCreationOptions.CommonEncryptionProtected);
	
	
	            return playreadyAsset;
	        }
	
	
	        /// <summary>
	        /// Configures authorization policy for the content key. 
	        /// </summary>
	        /// <param name="contentKey">The content key.</param>
	        static public void ConfigureLicenseDeliveryService(IContentKey contentKey)
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
	
	            // Configure PlayReady license template.
	            string newLicenseTemplate = ConfigurePlayReadyLicenseTemplate();
	
	            IContentKeyAuthorizationPolicyOption policyOption =
	                _context.ContentKeyAuthorizationPolicyOptions.Create("",
	                    ContentKeyDeliveryType.PlayReadyLicense,
	                        restrictions, newLicenseTemplate);
	
	            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
	                        ContentKeyAuthorizationPolicies.
	                        CreateAsync("Deliver Common Content Key with no restrictions").
	                        Result;
	
	
	            contentKeyAuthorizationPolicy.Options.Add(policyOption);
	
	            // Associate the content key authorization policy with the content key.
	            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
	            contentKey = contentKey.UpdateAsync().Result;
	        }
	
	        static private string ConfigurePlayReadyLicenseTemplate()
	        {
	            // The following code configures PlayReady License Template using .NET classes
	            // and returns the XML string.
	
	            PlayReadyLicenseResponseTemplate responseTemplate = new PlayReadyLicenseResponseTemplate();
	            PlayReadyLicenseTemplate licenseTemplate = new PlayReadyLicenseTemplate();
	
	            responseTemplate.LicenseTemplates.Add(licenseTemplate);
	
	            return MediaServicesLicenseTemplateSerializer.Serialize(responseTemplate);
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


<!---HONumber=Mooncake_0620_2016-->