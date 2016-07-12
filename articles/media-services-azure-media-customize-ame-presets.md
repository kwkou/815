<properties 
	pageTitle="通过自定义任务预设操作编码任务" 
	description="Azure 媒体服务编码器使你能够将自定义预设文件传递给 Azure 媒体编码器。本主题演示如何自定义预设文件以完成以下任务：将图像叠加到现有视频、控制编码器生成的输出文件名、拼结视频。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="12/05/2015"
	wacn.date="01/14/2016"/>

#通过自定义任务预设操作编码任务 

Azure 媒体服务编码器使你能够将自定义预设文件传递给 Azure 媒体编码器。本主题演示如何自定义预设文件以完成以下任务：

- 将图像叠加到现有视频、 
- 控制编码器生成的输出文件名、 
- 拼结视频、
- 对主要包含语音的演示文稿进行编码。

##控制 Azure 媒体编码器输出文件名 
<a name="controlling-azure-media-encoder-output-file-names"></a>

默认情况下，Azure 媒体编码器通过将输入资产和编码过程的各种特性组合在一起来创建输出文件名。如下文所述，使用宏来标识每种特性。

以下是可用于输出文件命名的宏的完整列表：
音频比特率 - 对音频进行编码时使用的比特率，单位为 kbps

- 音频编解码器 - 用于对音频进行编码的编解码器，有效值为 AAC、WMA 和 DDP
- 通道计数 - 编码的音频通道数，有效值为：1、2 或 6
- 默认扩展名 – 默认的文件扩展名 
- 语言 - BCP-47 语言代码，表示音频中使用的语言。当前默认为“und”。 
- 原始文件名 - 上载到 Azure 存储空间的文件的名称
- StreamId – 由预设文件中 <StreamInfo> 元素的 streamID 特性定义的流 ID 
- 视频编解码器 - 用于编码的编解码器，有效值为 H264 和 VC1
- 视频比特率 - 对视频进行编码时使用的比特率，单位为 kbps

这些宏可按任意排列组合，从而控制媒体服务编码器生成的文件名。例如，默认命名约定为：

	{Original File Name}_{Video Codec}{Video Bitrate}{Audio Codec}{Language}{Channel Count}{Audio Bitrate}.{Default Extension}

使用 [Preset](https://msdn.microsoft.com/zh-cn/library/azure/dn554334.aspx) 元素的 DefaultMediaOutputFileName 属性来指定文件命名约定。例如：

	<Preset DefaultMediaOutputFileName="{Original file name}{StreamId}_LongOutputFileName{Bit Rate}{Video Codec}{Video Bitrate}{Audio Codec}{Audio Bitrate}{Language}{Channel Count}.{Default extension}"
	  Version="5.0">
	<MediaFile …>
	   <OutputFormat>
	      <MP4OutputFormat StreamCompatibility="Standard">
	         <VideoProfile>
	            <MainH264VideoProfile … >
	               <Streams  AutoSize="False"
	                         FreezeSort="False">
	                  <StreamInfo StreamId="1"
	                              Size="1280, 720">
	                     <Bitrate>
	                       <ConstantBitrate Bitrate="3400"
	                                        IsTwoPass="False"
	                                        BufferWindow="00:00:05" />
	                     </Bitrate>
	                   </StreamInfo>
	                  </Streams>
	               </MainH264VideoProfile>
	            </VideoProfile>
	         </MP4OutputFormat>
	   </OutputFormat>
	</MediaFile>

编码器将在每个宏之间插入下划线，例如，上面的配置将会产生类似于以下形式的文件名：MyVideo\_H264\_4500kpbs\_AAC\_und\_ch2\_128kbps.mp4。


##创建叠加
<a name="creating-overlays"></a>

Azure 媒体服务编码器可让你将图像（jpg、bmp、gif、tif）、视频或音轨（*.wma、*.mp3、*.wav）叠加到现有视频。此功能类似于 Expression Encoder 4 (Service Pack 2) 的功能。

###使用媒体服务编码器进行叠加

可以指定叠加出现的时间、呈现叠加的持续时间、叠加出现在屏幕上的位置（对于图像/视频叠加）。还可使叠加层淡入和/或淡出。要叠加的音频/视频文件可包含在多个资产或单个资产中。可通过传递到编码器的预设 XML 对叠加进行控制。有关预设架构的完整说明，请参阅“Azure 媒体编码器架构”。如下面的预设代码片段所示，将在 <MediaFile> 元素中指定叠加：

	<MediaFile
	    ...
	    OverlayFileName="%1%"
	    OverlayRect="257, 144, 255, 144"
	    OverlayOpacity="0.9"
	    OverlayFadeInDuration="00:00:02"
	    OverlayFadeOutDuration="00:00:02"
	    OverlayLayoutMode="Custom"
	    OverlayStartTime="00:00:05"
	    OverlayEndTime="00:00:10.2120000"
	
	    AudioOverlayFileName="%2%"
	    AudioOverlayLoop="True"
	    AudioOverlayLoopingGap="00:00:00"
	    AudioOverlayLayoutMode="WholeSequence"
	    AudioOverlayGainLevel="2.2"
	    AudioOverlayFadeInDuration="00:00:00"
	    AudioOverlayFadeOutDuration="00:00:00">
	    ...
	</MediaFile>

###视频或图像叠加的预设

叠加可源自单个或多个资产。当使用多个资产创建视频叠加时，通过使用 %n% 语法（其中 n 是编码任务输入资产从零开始的索引）在 OverlayFileName 特性中指定叠加文件名。当使用单个资产创建视频叠加时，直接在 OverlayFileName 特性中指定叠加文件名，如下面的预设代码片段所示：

示例 1：

	<!-- Multiple Assets -->
	<MediaFile
		...
		OverlayFileName="%1%"
		OverlayRect="257, 144, 255, 144"
		OverlayOpacity="0.9"
		OverlayFadeInDuration="00:00:02"
		OverlayFadeOutDuration="00:00:02"
		OverlayLayoutMode="Custom"
		OverlayStartTime="00:00:05"
		OverlayEndTime="00:00:10.2120000">

示例 2：

	<!-- Single Asset -->
	<MediaFile
		...
		OverlayFileName="videoOverlay.mp4"
		OverlayRect="257, 144, 255, 144"
		OverlayOpacity="0.9"
		OverlayFadeInDuration="00:00:02"
		OverlayFadeOutDuration="00:00:02"
		OverlayLayoutMode="Custom"
		OverlayStartTime="00:00:05"
		OverlayEndTime="00:00:10.2120000">

视频叠加的位置和大小由 OverlayRect 特性控制。将调整要叠加的内容的大小以适应此矩形。不透明度由 OverlayOpacity 特性控制。有效值为 0.0 – 1.0，其中 1.0 表示 100% 不透明。叠加将于 OverlayStartTime 特性所指定的时间显示，并将于 OverlayEndTime 特性所指定的时间结束。OverlayStartTime 和 OverlayEndTime 都应在源视频的时间线内。有关每个特定于叠加的特性的详细信息，请参阅“Azure 媒体编码器架构”。

###音频叠加的预设

音频叠加可以是任何支持的音频文件格式。有关受支持音频文件格式的完整列表，请参阅“媒体服务编码器支持的格式”。如下面的预设代码片段所示，也会在 <MediaFile> 元素中指定音频叠加：

	<MediaFile
		...
		AudioOverlayFileName="%1%" <!-- or AudioOverlayFileName=”audioOverlay.mp3” for overlays from a single asset -->
		AudioOverlayLoop="True"
		AudioOverlayLoopingGap="00:00:00"
		AudioOverlayLayoutMode="Custom"
		AudioOverlayStartTime="00:05:00"
		AudioOverlayEndTime="00:10:00"
		AudioOverlayGainLevel="2.2"
		AudioOverlayFadeInDuration="00:00:00"
		AudioOverlayFadeOutDuration="00:00:00">

当音频叠加内容存储在多个资产时，通过使用 %n% 语法（其中 n 是编码任务的输入资产集合的从零开始的索引）在 AudioOverlayFileName 特性中指定音频叠加文件名。对于存储在单个资产中的音频叠加内容，直接在 AudioOverlayFileName 特性中指定叠加文件名。AudioOverlayLayoutMode 确定呈现音频叠加的时间。当设置为“WholeSequence”时，音轨将在整个视频持续期间呈现。当设置为“自定义”时，由 AudioOverlayStartTime 和 AudioOverlayEndTime 特性确定音频叠加的开始和结束时间。OverlayStartTime 和 OverlayEndTime 都应在源视频的时间线内。有关所有音频叠加特性的详细信息，请参阅“Azure 媒体编码器架构”。如下面的预设代码片段所示，音频叠加可与视频叠加合并：
	
	<MediaFile
	    DeinterlaceMode="AutoPixelAdaptive"
	    ResizeQuality="Super"
	    AudioGainLevel="1"
	    VideoResizeMode="Stretch"
	    OverlayFileName="%1%"
	    OverlayRect="257, 144, 255, 144"
	    OverlayOpacity="0.9"
	    OverlayFadeInDuration="00:00:02"
	    OverlayFadeOutDuration="00:00:02"
	    OverlayLayoutMode="Custom"
	    OverlayStartTime="00:00:05"
	    OverlayEndTime="00:00:10.2120000"
	    AudioOverlayFileName="%2%"
	    AudioOverlayLoop="True"
	    AudioOverlayLoopingGap="00:00:00"
	    AudioOverlayLayoutMode="Custom"
	    AudioOverlayStartTime="00:05:00"
	    AudioOverlayEndTime="00:10:00"
	    AudioOverlayGainLevel="2.2"
	    AudioOverlayFadeInDuration="00:00:00"
	    AudioOverlayFadeOutDuration="00:00:00">


###提交具有叠加的任务

创建预设文件后必须执行以下操作：

- 上载资产
- 加载预设配置（请注意：以下代码假定使用上述预设。）
- 创建作业
- 获取对媒体服务编码器的引用
- 创建一个任务，该任务指定输入资产、预设配置、媒体编码器和输出资产的集合
- 提交作业

以下代码片段演示如何执行这些步骤：

	static public void CreateOverlayJob()
	{
        // Create and cache the Media Services credentials in a static class variable.
        _cachedCredentials = new MediaServicesCredentials(
                        MediaServicesAccountName,
                        MediaServicesAccountKey,
						_defaultScope,
						_chinaAcsBaseAddressUrl);

		// Create the API server Uri
		_apiServer = new Uri(_chinaApiServerUrl);

        // Used the cached credentials to create CloudMediaContext.
        _context = new CloudMediaContext(_apiServer, _cachedCredentials);


		// Upload assets to overlay
		IAsset inputAsset1 = CreateAssetAndUploadSingleFile(AssetCreationOptions.None, video1.mp4); // this is the input mezzanine
		IAsset inputAsset2 = CreateAssetAndUploadSingleFile(AssetCreationOptions.None, video2.wmv);// this will be used as a video overlay
		IAsset inputAsset3 = CreateAssetAndUploadSingleFile(AssetCreationOptions.None, video3.wmv); // this will be used as an audio overlay
		
		// Load the preset configuration
		string presetFileName = "OverlayPreset.xml";
		string configuration = File.ReadAllText(presetFileName);
		
		// Create a job
		IJob job = _context.Jobs.Create("A AME overlay job, using " + presetFileName);
		         
		// Get a reference to the media services encoder   
		IMediaProcessor processor = GetLatestMediaProcessorByName("Azure Media Encoder");
		    
		// Create a task    
		ITask task = job.Tasks.AddNew("Encode Task for overlay, using " + presetFileName, processor, configuration, TaskOptions.None);
		
		// Add the input assets
		task.InputAssets.Add(inputAsset1);
		task.InputAssets.Add(inputAsset2);
		task.InputAssets.Add(inputAsset3);
		
		// Add the output asset
		task.OutputAssets.AddNew("Result of an overlay job, using " + presetFileName, AssetCreationOptions.None);
		
		// Submit the job
		job.Submit();
	}



>[AZURE.NOTE]为简单起见，此代码段按顺序加载每个资产。在生产环境中将批量加载资产。有关批量上载多个资产的详细信息，请参阅[使用用于 .NET 的媒体服务 SDK 批量引入资产](/documentation/articles/media-services-dotnet-upload-files/#ingest_in_bulk)。

有关完整的示例代码，请参阅[使用媒体服务编码器创建叠加](https://code.msdn.microsoft.com/Creating-Audio-and-Video-c2942c47)。

###错误条件

当编辑预设字符串时，必须确保以下各项：

- 对于视频/图像叠加，叠加矩形必须完全适应源视频的尺寸
- 叠加的开始和结束时间应在源视频的时间线内
- 如果预设 XML 包含对 ?OverlayFileName=”%n%” 的引用，那么任务的 InputAssets 集合应包含至少 n+1 个资产



##拼结视频段

媒体服务编码器支持将一组视频拼结在一起。视频可首尾相连地拼结在一起，也可以指定一个或两个视频要拼结到一起的部分。拼结生成的单个输出资产，其中包含输入资产种的指定视频。要拼结的视频可包含在多个资产或单个资产中。可通过传递到编码器的预设 XML 对拼结进行控制。有关预设架构的完整说明，请参阅 [Azure 媒体编码器架构](https://msdn.microsoft.com/zh-cn/library/azure/dn584702.aspx)。

###使用媒体服务编码器进行拼结

在<MediaFile>元素内对拼结进行控制，如下面的预设所示：
	
	<MediaFile
	    DeinterlaceMode="AutoPixelAdaptive"
	    ResizeQuality="Super"
	    AudioGainLevel="1"
	    VideoResizeMode="Stretch">
	    <Sources>
	      <Source
	        AudioStreamIndex="0">
	        <Clips>
	          <Clip
	            StartTime="00:00:00"
	            EndTime="00:00:05" />
	        </Clips>
	      </Source>
	      <Source
	        ResizeMode="Letterbox"
	        ApplyCrop="False"
	        AudioStreamIndex="0"
	        MediaFile="%1%">
	        <Clips>
	          <Clip
	            StartTime="00:00:00"
	            EndTime="00:00:05" />
	        </Clips>
	      </Source>
	      <Source
	        ResizeMode="Letterbox"
	        ApplyCrop="False"
	        AudioStreamIndex="0"
	        MediaFile="%2%">
	        <Clips>
	          <Clip
	            StartTime="00:00:00"
	            EndTime="00:00:05" />
	        </Clips>
	      </Source>
	     </Sources>
	</MediaFile>

对于要拼结的每个视频，将向<Sources>元素添加<Source>元素。每个<Source>元素包含一个<Clips>元素。每个<Clips>元素包含一个或多个<Clip>元素，后者通过指定开始时间和结束时间来指定拼结到输出资产中的视频数目。<Source>元素引用其作用于的资产。引用的格式取决于要拼结的视频是位于不同资产中还是位于单个资产中。如果想要拼结整个视频，只需省略<Clips>元素。

###拼结多个资产中的视频

拼结多个资产中的视频时，将从零开始的索引用于<Source>元素的 MediaFile 特性，以标识<Source>元素对应的资产。当未指定零索引时，未指定 MediaFile 特性的<Source>元素将引用首个输入资产。所有其他<Source>元素都必须通过使用 %n% 语法（其中 n 是输入资产从零开始的索引）来指定所引用输入资产从零开始的索引。在前面的示例中，第一个<Source>元素指定第一个输入资产，第二个<Source>元素指定第二个输入资产，依此类推。不要求按顺序引用输入资产，例如：
	
	<MediaFile
	    DeinterlaceMode="AutoPixelAdaptive"
	    ResizeQuality="Super"
	    AudioGainLevel="1"
	    VideoResizeMode="Stretch">
	    <Sources>
	      <Source
	        AudioStreamIndex="0"
	        MediaFile="%1%">
	        <Clips>
	          <Clip EndTime="00:00:05" 
	                StartTime="00:00:00" />
	        </Clips>
	                  
	        </Source>
	      <Source
	       AudioStreamIndex="0">
	        <Clips>
	          <Clip
	            StartTime="00:00:00"
	            EndTime="00:00:05" />
	        </Clips>
	      </Source>
	      <Source
	          AudioStreamIndex="0"
	         MediaFile="%2%">
	        <Clips>
	          <Clip
	            StartTime="00:00:00"
	            EndTime="00:00:05" />
	        </Clips>
	      </Source>
	     </Sources>
	</MediaFile>

此示例将第二个、第一个和第三个输入资产的某些部分拼结在一起。请注意，由于每个视频由从零开始的索引引用，因此可能会将两个具有相同名称的视频拼结在一起。创建预设文件后必须执行以下操作：
 
- 上载资产
- 加载预设配置
- 创建作业
- 获取对媒体服务编码器的引用
- 创建一个任务，该任务指定输入资产、预设配置、媒体编码器和输出资产
- 提交作业

以下代码片段演示如何执行这些步骤：
	
	static public void StitchWithMultipleAssets()
	{
        // Create and cache the Media Services credentials in a static class variable.
        _cachedCredentials = new MediaServicesCredentials(
                        MediaServicesAccountName,
                        MediaServicesAccountKey,
						_defaultScope,
						_chinaAcsBaseAddressUrl);

		// Create the API server Uri
		_apiServer = new Uri(_chinaApiServerUrl);

        // Used the cached credentials to create CloudMediaContext.
        _context = new CloudMediaContext(_apiServer, _cachedCredentials);
		
		// Upload assets to stitch
		IAsset inputAsset1 = CreateAssetAndUploadSingleFile(AssetCreationOptions.None, video1.mp4);
		IAsset inputAsset2 = CreateAssetAndUploadSingleFile(AssetCreationOptions.None, video2.wmv);
		IAsset inputAsset3 = CreateAssetAndUploadSingleFile(AssetCreationOptions.None, video3.wmv);
		
		// Load the preset configuration
		string presetFileName = "StitchingWithMultipleAssets.xml";
		string configuration = File.ReadAllText(presetFileName);
		
		// Create a job
		IJob job = _context.Jobs.Create("A AME stitching job, using " + presetFileName);
		         
		// Get a reference to the media services encoder   
		IMediaProcessor processor = GetLatestMediaProcessorByName("Azure Media Encoder");
		    
		// Create a task    
		ITask task = job.Tasks.AddNew("Encode Task for stitching, using " + presetFileName, processor, configuration, TaskOptions.None);
		
		// Add the input assets
		task.InputAssets.Add(inputAsset1);
		task.InputAssets.Add(inputAsset2);
		task.InputAssets.Add(inputAsset3);
		
		// Add the output asset
		task.OutputAssets.AddNew("Result of a stitching job, using " + presetFileName, AssetCreationOptions.None);
		
		// Submit the job
		job.Submit();
		} 


为简单起见，此代码段按顺序加载每个资产。在生产环境中将批量加载资产。有关批量上载多个资产的详细信息，请参阅[使用用于 .NET 的媒体服务 SDK 批量引入资产](/documentation/articles/media-services-dotnet-upload-files/#ingest_in_bulk)。有关完整的示例代码，请参阅[使用媒体服务编码器进行拼结](https://code.msdn.microsoft.com/Stitching-with-Media-8fd5f203)。

###拼结单个资产中的视频

拼结单个资产中的视频时，每个视频必须具有唯一的名称。视频通过使用文件名作为特性值的 MediaFile 特性指定。例如：
	
	<MediaFile
	    DeinterlaceMode="AutoPixelAdaptive"
	    ResizeQuality="Super"
	    AudioGainLevel="1"
	    VideoResizeMode="Stretch">
	    <Sources>
	      <Source
	        AudioStreamIndex="0"
	        MediaFile="video1.mp4">
	        <Clips>
	          <Clip
	            StartTime="00:00:00"
	            EndTime="00:00:05" />
	        </Clips>
	      </Source>
	      <Source
	       AudioStreamIndex="0"
	       MediaFile="video2.wmv">
	
	        <Clips>
	          <Clip
	            StartTime="00:00:00"
	            EndTime="00:00:05" />
	        </Clips>
	      </Source>
	      <Source
	          AudioStreamIndex="0"
	         MediaFile="video3.wmv">
	        <Clips>
	          <Clip
	            StartTime="00:00:00"
	            EndTime="00:00:05" />
	        </Clips>
	      </Source>
	     </Sources>
	</MediaFile>

此预设将 video1.mp4、video2.wmv 和 video3.wmv 的某些部分拼结为输出资产。
创建作业和任务与拼结多个资产中的视频相同，只需上载单个资产，如以下代码所示：

	// Creates a stitching job that uses a single asset 
    static public void StitchWithASingleAsset()
    {
        string presetFileName = "StitchingWithASingleAsset.xml";
        string configuration = File.ReadAllText(presetFileName);

        // Create and cache the Media Services credentials in a static class variable.
        _cachedCredentials = new MediaServicesCredentials(
                        MediaServicesAccountName,
                        MediaServicesAccountKey,
						_defaultScope,
						_chinaAcsBaseAddressUrl);

		// Create the API server Uri
		_apiServer = new Uri(_chinaApiServerUrl);

        // Used the cached credentials to create CloudMediaContext.
        _context = new CloudMediaContext(_apiServer, _cachedCredentials);

        IMediaProcessor processor = GetLatestMediaProcessorByName("Azure Media Encoder");
        IJob job = _context.Jobs.Create("A AME stitching job, using " + presetFileName);
        IAsset asset = CreateAssetAndUploadMultipleFiles(AssetCreationOptions.None, _stitchingFiles);

        ITask task = job.Tasks.AddNew("Encode Task for stitching, using " + presetFileName, processor, configuration, TaskOptions.None);
        task.InputAssets.Add(asset);
        task.OutputAssets.AddNew("Result of a stitching job, using " + presetFileName, AssetCreationOptions.None);

        job.Submit();
    }

##对主要包含语音的演示文稿或音频流进行编码
 
当对其音轨主要包含语音的视频进行编码时，默认编码器预设可能会导致背景噪音在已编码资产中放大。此行为是由于将 NormalizeAudio 特性设置为 true 所致。

###对主要包含语音的演示文稿进行编码

若要防止背景噪音被放大，请执行以下操作：

1. 将正在使用的编码器预设复制到 XML 文件中。可在 Azure 媒体编码器架构中找到编码器预设
1. 删除 NormalizeAudio 特性，它位于 <MediaFile> 元素下的预设文件顶部附近：
	
	<MediaFile
	     DeinterlaceMode="AutoPixelAdaptive"
	     ResizeQuality="Super"
	     NormalizeAudio="True"
	     AudioGainLevel="1"
	     VideoResizeMode="Stretch">

1. 将已修改的预设文件保存到本地硬盘驱动器，然后通过诸如以下形式的代码使用自定义预设进行编码：
		
		// Upload file and create asset
		IAsset asset = CreateAssetAndUploadSingleFile(AssetCreationOptions.None, @"C:\TEMP\Original.mp4");
		 
		string inputPresetFile = @"C:\TEMP\H264 Broadband 720p NoAudioNorm.xml";
		string presetName = Path.GetFileNameWithoutExtension(inputPresetFile);
		 
		IJob job = _context.Jobs.Create("Encode Job for " + asset.Name + ", encoded using " +  presetName);
		
		Console.WriteLine("Encode Job for " + asset.Name + ", encoded using " + presetName);
		
		// Get a media processor reference, and pass to it the name of the processor to use for the specific task.
		IMediaProcessor processor = GetLatestMediaProcessorByName("Azure Media Encoder");
		Console.WriteLine("Got MP " + processor.Name + ", ID : " + processor.Id + ", version: " + processor.Version);
		 
		// Read the configuration data into a string. 
		string configuration = File.ReadAllText(inputPresetFile);
		 
		// Create a task with the encoding details, using a string preset.
		ITask task = job.Tasks.AddNew("Encode Task for " + asset.Name + ", encoded using " + presetName, processor, configuration,
		                Microsoft.WindowsAzure.MediaServices.Client.TaskOptions.None);
		 
		// Specify the input asset to be encoded.
		task.InputAssets.Add(asset);
		 
		// Add an output asset to contain the results of the job.
		task.OutputAssets.AddNew("Output asset for " + asset.Name + ", encoded using " + presetName, AssetCreationOptions.None);
		 
		// Launch the job. 
		job.Submit();

##另请参阅

[Azure 媒体编码器 XML 架构](https://msdn.microsoft.com/zh-cn/library/azure/dn584702.aspx)

<!---HONumber=74-->