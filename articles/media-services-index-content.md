<properties 
	pageTitle="使用 Azure Media Indexer 为媒体文件编制索引" 
	description="使用 Azure Media Indexer，可以使媒体文件内容可供搜索，并为隐藏的字幕和关键字生成全文本脚本。本主题说明如何使用 Media Indexer。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="09/07/2015"   
	wacn.date="10/22/2015"/>


# 使用 Azure Media Indexer 为媒体文件编制索引

> [AZURE.SELECTOR]
- [门户](/documentation/articles/media-services-manage-content#index)
- [.NET](/documentation/articles/media-services-index-content)


使用 Azure Media Indexer，可以使媒体文件内容可供搜索，并为隐藏的字幕和关键字生成全文本脚本。你可以只处理一个媒体文件，也可以一次处理多个媒体文件。

>[AZURE.NOTE]在编制内容的索引时，请确保使用语音极其清晰的媒体文件（没有背景音乐、噪音、特效音或麦克风电流嘶嘶声）。适当内容的某些示例包括：录制的会议、讲座或演示内容。以下内容可能不适合用于编制索引：电影、电视剧、混合了音频和声音特效的任何内容、带有背景噪音（电流嘶嘶声）的不当录制内容。


索引作业将在每个索引文件中生成四个输出：

- 采用 SAMI 格式的隐藏字幕文件。
- 采用时序文本标记语言 (TTML) 格式的隐藏字幕文件。

	SAMI 和 TTML 包含名为 Recognizability 的标记，该标记可以根据源视频中的语音辨别度对索引作业评分。你可以使用 Recognizability 的值筛选可用的输出文件。如果分数较低，则表示索引结果由于音频质量问题而不佳。
- 关键字文件 (XML)。
- 用于 SQL Server 的音频索引 Blob (AIB) 文件。
	
	有关详细信息，请参阅[在 Azure Media Indexer 和 SQL Server 中使用 AIB 文件](http://azure.microsoft.com/blog/2014/11/03/using-aib-files-with-azure-media-indexer-and-sql-server/)。


本主题介绍如何创建索引作业，以便“为资产编制索引”以及“为多个文件编制索引”。

有关最新的 Azure Media Indexer 更新，请参阅[媒体服务博客](http://azure.microsoft.com/blog/topics/media-services/)。

##使用配置和清单文件执行索引编制任务

你可以使用任务配置为你的索引编制任务指定更多详细信息。例如，你可以指定要用于媒体文件的元数据。此元数据可供语言引擎用于扩充其词汇，并大幅提高语言识别的准确性。

你还可以使用清单文件一次处理多个媒体文件。

有关详细信息，请参阅 [Azure Media Indexer 的任务预设](https://msdn.microsoft.com/zh-cn/library/azure/dn783454.aspx)。

##为资产编制索引

以下方法将媒体文件上载为资产，并创建为资产编制索引的作业。

请注意，如果未指定配置文件，则将使用默认设置为媒体文件编制索引。
	
	static bool RunIndexingJob(string inputMediaFilePath, string outputFolder, string configurationFile = "")
	{
	    // Create an asset and upload the input media file to storage.
	    IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
	        "My Indexing Input Asset",
	        AssetCreationOptions.None);
	
	    // Declare a new job.
	    IJob job = _context.Jobs.Create("My Indexing Job");
	
	    // Get a reference to the Azure Media Indexer.
	    string MediaProcessorName = "Azure Media Indexer",
	    IMediaProcessor processor = GetLatestMediaProcessorByName(MediaProcessorName);
	
	    // Read configuration from file if specified.
	    string configuration = string.IsNullOrEmpty(configurationFile) ? "" : File.ReadAllText(configurationFile);
	
	    // Create a task with the encoding details, using a string preset.
	    ITask task = job.Tasks.AddNew("My Indexing Task",
	        processor,
	        configuration,
	        TaskOptions.None);
	
	    // Specify the input asset to be indexed.
	    task.InputAssets.Add(asset);
	
	    // Add an output asset to contain the results of the job. 
	    task.OutputAssets.AddNew("My Indexing Output Asset", AssetCreationOptions.None);
	
	    // Use the following event handler to check job progress.  
	    job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);
	
	    // Launch the job.
	    job.Submit();
	
	    // Check job execution and wait for job to finish. 
	    Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
	    progressJobTask.Wait();
	
	    // If job state is Error, the event handling 
	    // method for job progress should log errors.  Here we check 
	    // for error state and exit if needed.
	    if (job.State == JobState.Error)
	    {
	        Console.WriteLine("Exiting method due to job error.");
	        return false;
	    }
	
	    // Download the job outputs.
	    DownloadAsset(task.OutputAssets.First(), outputFolder);
	
	    return true;
	}
	
	static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
	{
	    IAsset asset = _context.Assets.Create(assetName, options);
	
	    var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
	    assetFile.Upload(filePath);
	
	    return asset;
	}
	        
	static void DownloadAsset(IAsset asset, string outputDirectory)
	{
	    foreach (IAssetFile file in asset.AssetFiles)
	    {
	        file.Download(Path.Combine(outputDirectory, file.Name));
	    }
	}
	
	static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
	{
	    var processor = _context.MediaProcessors
	    .Where(p => p.Name == mediaProcessorName)
	    .ToList()
	    .OrderBy(p => new Version(p.Version))
	    .LastOrDefault();
	
	    if (processor == null)
	        throw new ArgumentException(string.Format("Unknown media processor",
	                                                   mediaProcessorName));
	
	    return processor;
	} 
	
###<a id="output_files"></a>输出文件

索引作业生成以下输出文件。这些文件将存储在第一个输出资产中。


<table border="1">
<tr><th>文件名</th><th>说明</th></tr>
<tr><td>InputFileName.aib </td>
<td>音频索引 Blob 文件。<br/><br/>
音频索引 Blob (AIB) 文件是二进制文件，可使用全文本搜索在 Microsoft SQL Server 中对其进行搜索。AIB 文件的功能比简单的字幕文件要强大，因为它包含每个单词的替代项，使你可以获得更丰富的搜索体验。
<br/>
<br/>
它要求在运行 Microsoft SQL Server 2008 或更高版本的计算机上安装 Indexer SQL 外接程序。使用 Microsoft SQL Server 全文搜索对 AIB 搜索时，搜索结果比搜索由 WAMI 生成的隐藏字幕文件要准确。这是因为，AIB 包含发音类似单词的替代项，而隐藏字幕文件包含每个音频段的最高信任度单词。如果搜索说过的话很重要，则建议将 AIB 与 Microsoft SQL Server 结合使用。
<br/><br/>
若要下载外接程序，请单击 <a href="http://manlingtest-samplescdn.streaming.mediaservices.windows.net/be5e4df9-6ec6-44e4-9e6e-40c82dc95d1c/Azure%20Media%20Indexer%20SQL%20Add-on.zip">Azure Media Indexer SQL 外接程序</a>。
<br/><br/>
此外，还可以利用其他搜索引擎，如 Apache Lucene/Solr，仅根据隐藏字幕文件和关键字 XML 文件为视频编制索引，但这将导致搜索结果不太准确。</td></tr>
<tr><td>InputFileName.smi<br/>InputFileName.ttml</td>
<td>采用 SAMI 和 TTML 格式的隐藏字幕 (CC) 文件。
<br/><br/>
这些文件可用于使听力障碍用户能够访问音频和视频文件。
<br/><br/>
SAMI 和 TTML 包含名为 <b>Recognizability</b> 的标记，该标记可以根据源视频中的语音辨别度对索引作业评分。你可以使用 <b>Recognizability</b> 的值筛选可用的输出文件。如果分数较低，则表示索引结果由于音频质量问题而不佳。</td></tr>
<tr><td>InputFileName.kw.xml</td>
<td>关键字文件。
<br/><br/>
关键字文件是 XML 文件，其中包含从语音内容中提取的关键字，以及频率和偏移量信息。
<br/><br/>
该文件可用于各种目的，如执行语音分析，公开给必应、Google&#160;或 Microsoft SharePoint 等搜索引擎以使媒体文件更容易被发现，或用于传送更具相关性的广告。</td></tr>
</table>

如果没有为所有输入媒体文件成功编制索引，则索引编制作业将失败，出现错误代码 4000。有关详细信息，请参阅[错误代码](#error_codes)。

##为多个文件编制索引

以下方法将多个媒体文件上载为资产，并创建一次性为所有这些文件编制索引的作业。

使用 .lst 扩展名的清单文件在创建后将上载到资产中。该清单文件包含所有资产文件的列表。有关详细信息，请参阅 [Azure Media Indexer 的任务预设](https://msdn.microsoft.com/zh-cn/library/azure/dn783454.aspx)。
	
	static bool RunBatchIndexingJob(string[] inputMediaFiles, string outputFolder)
	{
	    // Create an asset and upload to storage.
	    IAsset asset = CreateAssetAndUploadMultipleFiles(inputMediaFiles,
	        "My Indexing Input Asset - Batch Mode",
	        AssetCreationOptions.None);
	
	    // Create a manifest file that contains all the asset file names and upload to storage.
	    string manifestFile = "input.lst";            
	    File.WriteAllLines(manifestFile, asset.AssetFiles.Select(f => f.Name).ToArray());
	    var assetFile = asset.AssetFiles.Create(Path.GetFileName(manifestFile));
	    assetFile.Upload(manifestFile);
	
	    // Declare a new job.
	    IJob job = _context.Jobs.Create("My Indexing Job - Batch Mode");
	
	    // Get a reference to the Azure Media Indexer.
	    string MediaProcessorName = "Azure Media Indexer";
	    IMediaProcessor processor = GetLatestMediaProcessorByName(MediaProcessorName);
	
	    // Read configuration.
	    string configuration = File.ReadAllText("batch.config");
	
	    // Create a task with the encoding details, using a string preset.
	    ITask task = job.Tasks.AddNew("My Indexing Task - Batch Mode",
	        processor,
	        configuration,
	        TaskOptions.None);
	
	    // Specify the input asset to be indexed.
	    task.InputAssets.Add(asset);
	
	    // Add an output asset to contain the results of the job.
	    task.OutputAssets.AddNew("My Indexing Output Asset - Batch Mode", AssetCreationOptions.None);
	
	    // Use the following event handler to check job progress.  
	    job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);
	
	    // Launch the job.
	    job.Submit();
	
	    // Check job execution and wait for job to finish. 
	    Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
	    progressJobTask.Wait();
	
	    // If job state is Error, the event handling 
	    // method for job progress should log errors.  Here we check 
	    // for error state and exit if needed.
	    if (job.State == JobState.Error)
	    {
	        Console.WriteLine("Exiting method due to job error.");
	        return false;
	    }
	
	    // Download the job outputs.
	    DownloadAsset(task.OutputAssets.First(), outputFolder);
	
	    return true;
	}
	
	private static IAsset CreateAssetAndUploadMultipleFiles(string[] filePaths, string assetName, AssetCreationOptions options)
	{
	    IAsset asset = _context.Assets.Create(assetName, options);
	
	    foreach (string filePath in filePaths)
	    {
	        var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
	        assetFile.Upload(filePath);
	    }
	
	    return asset;
	}


###输出文件

如果有多个输入媒体文件，则 WAMI 将为名为“JobResult.txt”的作业输出生成清单文件。对于每个输入媒体文件，生成的 AIB、SAMI、TTML 和关键字文件依次进行编号（已在下面列出）。

有关输出文件的说明，请参阅[输出文件](#output_files)。


<table border="1">
<tr><th>文件名</th><th>说明</th></tr>
<tr><td>JobResult.txt</td>
<td>输出清单
<br/><br/>下面是输出清单文件 (JobResult.txt) 的格式。
<br/><br/>

<table border="1">
<tr><th>InputFile</th><th>别名</th><th>MediaLength</th><th>错误</th></tr>
<tr><td>a.mp4</td><td>Media_1</td><td>300</td><td>0</td></tr>
<tr><td>b.mp4</td><td>Media_2</td><td>0</td><td>3000</td></tr>
<tr><td>c.mp4</td><td>Media_3</td><td>600</td><td>0</td></tr>
</table><br/>
每行都表示一个输入媒体文件：
<br/><br/>
InputFile：输入媒体文件的资产文件名或 URL。
<br/><br/>
Alias：相应的输出文件名。
<br/><br/>
MediaLength：输入媒体文件的长度（以秒为单位）。Can be 0 是出现在此输入中的错误。
<br/><br/>
Error：指示是否为此媒体文件成功编制索引。0 表示成功，其他表示失败。有关具体错误，请参考<a href="#error_codes">错误代码</a>。
</td></tr>
<tr><td>Media_1.aib </td>
<td>文件 #0 - 音频索引 Blob 文件。</td></tr>
<tr><td>Media_1.smi<br/>Media_1.ttml</td>
<td>文件 #0 - 采用 SAMI 和 TTML 格式的隐藏字幕 (CC) 文件。</td></tr>
<tr><td>Media_1.kw.xml</td>
<td>文件 #0 - 关键字文件。</td></tr>
<tr><td>Media_2.aib </td>
<td>文件 #1 - 音频索引 Blob 文件。</td></tr>
</table>

如果没有为所有输入媒体文件成功编制索引，则索引编制作业将失败，出现错误代码 4000。有关详细信息，请参阅[错误代码](#error_codes)。

###部分成功的作业

如果没有为所有输入媒体文件成功编制索引，则索引编制作业将失败，出现错误代码 4000。有关详细信息，请参阅[错误代码](#error_codes)。


生成相同的输出（与成功的作业一样）。你可以参考输出清单文件，以根据“错误”列的值查明哪些输入文件失败。对于失败的输入文件，将不生成相应的 AIB、SAMI、TTML 和关键字文件。


### <a id="error_codes"></a>错误代码


<table border="1">
<tr><th>代码</th><th>名称</th><th>可能的原因</th></tr>
<tr><td>2000</td><td>配置无效</td><td>配置无效</td></tr>
<tr><td>2001</td><td>输入资产无效</td><td>输入资产缺失或资产为空。</td></tr>
<tr><td>2002</td><td>清单无效</td><td>清单为空或清单包含无效项目。</td></tr>
<tr><td>2003</td><td>无法下载媒体文件</td><td>清单文件中的 URL 无效。</td></tr>
<tr><td>2004</td><td>协议不受支持</td><td>不支持媒体 URL 的协议。</td></tr>
<tr><td>2005</td><td>文件类型不受支持</td><td>不支持输入媒体文件类型。</td></tr>
<tr><td>2006</td><td>输入文件太多</td><td>输入清单中的文件超过 10 个。</td></tr>
<tr><td>3000</td><td>无法解码媒体文件</td>
<td>媒体编解码器不受支持。
<br/>或<br/>
媒体文件受损。
<br/>或<br/>
输入媒体中没有音频流。</td></tr>
<tr><td>4000</td><td>分批编制索引部分成功</td><td>一些输入媒体文件无法编制索引。有关详细信息，请参阅<a href="#output_files">输出文件</a>。</td></tr>
<tr><td>其他</td><td>内部错误</td><td>请联系支持团队。</td></tr>
</table>


##<a id="supported_languages"></a>支持的语言

当前支持英语和西班牙语。有关详细信息，请参阅 [Azure Media Indexer 西班牙语版](http://azure.microsoft.com/blog/2015/04/13/azure-media-indexer-spanish-v1-2/)。

##相关链接

[在 Azure Media Indexer 和 SQL Server 中使用 AIB 文件](http://azure.microsoft.com/blog/2014/11/03/using-aib-files-with-azure-media-indexer-and-sql-server/)

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->

<!---HONumber=74-->