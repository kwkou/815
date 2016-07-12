<properties
	pageTitle="使用 Azure Media Indexer 为媒体文件编制索引"
	description="使用 Azure Media Indexer，可以使媒体文件内容可供搜索，并为隐藏的字幕和关键字生成全文本脚本。本主题说明如何使用 Media Indexer。"
	services="media-services"
	documentationCenter=""
	authors="Juliako,Asolanki,johndeu"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="04/08/2016"   
	wacn.date="05/16/2016"/>


# 使用 Azure Media Indexer 为媒体文件编制索引

> [AZURE.SELECTOR]
- [门户](/documentation/articles/media-services-manage-content/#index)
- [.NET](/documentation/articles/media-services-index-content/)


使用 Azure Media Indexer，可以使媒体文件内容可供搜索，并为隐藏的字幕和关键字生成全文本脚本。你可以只处理一个媒体文件，也可以一次处理多个媒体文件。

>[AZURE.IMPORTANT] 在编制内容的索引时，请确保使用语音极其清晰的媒体文件（没有背景音乐、噪音、特效音或麦克风电流嘶嘶声）。适当内容的某些示例包括：录制的会议、讲座或演示内容。以下内容可能不适合用于编制索引：电影、电视剧、混合了音频和声音特效的任何内容、带有背景噪音（电流嘶嘶声）的不当录制内容。


索引作业可生成以下输出：

- 以下格式的隐藏字幕文件：**SAMI**、**TTML** 和 **WebVTT**。

	隐藏字幕文件包含名为 Recognizability 的标记，该标记可以根据源视频中的语音辨别度对索引作业评分。你可以使用 Recognizability 的值筛选可用的输出文件。如果分数较低，则表示索引结果由于音频质量问题而不佳。
- 关键字文件 (XML)。
- 用于 SQL Server 的音频索引 Blob (AIB) 文件。

	有关详细信息，请参阅[在 Azure 媒体索引器和 SQL Server 中使用 AIB 文件](https://azure.microsoft.com/blog/2014/11/03/using-aib-files-with-azure-media-indexer-and-sql-server/)。


本主题介绍如何创建索引作业，以便“为资产编制索引”以及“为多个文件编制索引”。

有关最新的 Azure 媒体索引器更新，请参阅[媒体服务博客](#preset)。

## 使用配置和清单文件执行索引编制任务

你可以使用任务配置为你的索引编制任务指定更多详细信息。例如，你可以指定要用于媒体文件的元数据。此元数据可供语言引擎用于扩充其词汇，并大幅提高语言识别的准确性。你还可以指定所需的输出文件。

你还可以使用清单文件一次处理多个媒体文件。

有关详细信息，请参阅 [Azure Media Indexer 的任务预设](#)。

## 为资产编制索引

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
	    string MediaProcessorName = "Azure Media Indexer";
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

### <a id="output_files"></a>输出文件

默认情况下，索引作业生成以下输出文件。这些文件将存储在第一个输出资产中。

如果有多个输入媒体文件，则索引器将为名为“JobResult.txt”的作业输出生成清单文件。对于每个输入媒体文件，生成的 AIB、SAMI、TTML、WebVTT 和关键字文件将依序编号，并使用“别名”来命名。

文件名 | 说明
----------|------------
__InputFileName.aib__ | 音频索引 Blob 文件。<br /><br />音频索引 Blob (AIB) 文件是二进制文件，可使用全文本搜索在 Microsoft SQL Server 中对其进行搜索。AIB 文件的功能比简单的字幕文件要强大，因为它包含每个单词的替代项，使你可以获得更丰富的搜索体验。<br/><br/>它要求在运行 Microsoft SQL Server 2008 或更高版本的计算机上安装 Indexer SQL 外接程序。使用 Microsoft SQL Server 全文搜索对 AIB 搜索时，搜索结果比搜索由 WAMI 生成的隐藏字幕文件要准确。这是因为，AIB 包含发音类似单词的替代项，而隐藏字幕文件包含每个音频段的最高信任度单词。如果搜索说过的话很重要，则建议将 AIB 与 Microsoft SQL Server 结合使用。<br/><br/> 若要下载外接程序，请单击 <a href="http://manlingtest-samplescdn.streaming.mediaservices.windows.net/be5e4df9-6ec6-44e4-9e6e-40c82dc95d1c/Azure%20Media%20Indexer%20SQL%20Add-on.zip">Azure Media Indexer SQL 外接程序</a>。<br/><br/>此外，还可以利用其他搜索引擎，如 Apache Lucene/Solr，仅根据隐藏字幕文件和关键字 XML 文件为视频编制索引，但这将导致搜索结果不太准确。
__InputFileName.smi__<br />__InputFileName.ttml__<br />__InputFileName.vtt__ |采用 SAMI、TTML 和 WebVTT 格式的隐藏字幕 (CC) 文件。<br/><br/>这些文件可用于使听力障碍用户能够访问音频和视频文件。<br/><br/>隐藏字幕文件包含名为 <b>Recognizability</b> 的标记，该标记可以根据源视频中的语音辨别度对索引作业评分。你可以使用 <b>Recognizability</b> 的值筛选可用的输出文件。如果分数较低，则表示索引结果由于音频质量问题而不佳。
__InputFileName.kw.xml<br />InputFileName.info__ |关键字和信息文件。<br/><br/>关键字文件是 XML 文件，其中包含从语音内容中提取的关键字，以及频率和偏移量信息。<br/><br/>信息文件是一种纯文本文件，其中包含有关每个已识别术语的详细信息。第一行很特别，保护 Recognizability 评分。后续每一行是使用制表符分隔的以下数据的列表：开始时间、结束时间、单词/短语、置信度。时间以秒为单位，置信度为数字 0-1。<br/><br/>示例行："1.20 1.45 word 0.67"<br/><br/>这些文件可用于各种目的，如执行语音分析，公开给必应、Google 或 Microsoft SharePoint 等搜索引擎以使媒体文件更容易被发现，甚至用于传送更具相关性的广告。
__JobResult.txt__ |输出清单，仅当为多个文件编制索引时才提供，包含以下信息：<br/><br/><table border="1"><tr><th>InputFile</th><th>Alias</th><th>MediaLength</th><th>Error</th></tr><tr><td>a.mp4</td><td>Media\_1</td><td>300</td><td>0</td></tr><tr><td>b.mp4</td><td>Media\_2</td><td>0</td><td>3000</td></tr><tr><td>c.mp4</td><td>Media\_3</td><td>600</td><td>0</td></tr></table><br/>



如果没有为所有输入媒体文件成功编制索引，则索引编制作业将失败，出现错误代码 4000。有关详细信息，请参阅[错误代码](#error_codes)。

## 为多个文件编制索引

以下方法将多个媒体文件上载为资产，并创建一次性为所有这些文件编制索引的作业。

使用 .lst 扩展名的清单文件在创建后将上载到资产中。该清单文件包含所有资产文件的列表。有关详细信息，请参阅 [Azure 媒体索引器的任务预设](https://msdn.microsoft.com/zh-cn/library/azure/dn783454.aspx)。

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

### 部分成功的作业

如果没有为所有输入媒体文件成功编制索引，则索引编制作业将失败，出现错误代码 4000。有关详细信息，请参阅[错误代码](#error_codes)。


生成相同的输出（与成功的作业一样）。你可以参考输出清单文件，以根据“错误”列的值查明哪些输入文件失败。对于失败的输入文件，将不生成相应的 AIB、SAMI、TTML、WebVTT 和关键字文件。

### <a id="preset"></a>Azure Media Indexer 的任务预设

可以通过连同任务一起提供可选任务预设，来自定义 Azure Media Indexer 的处理。下面描述了此配置 xml 文件的格式。

Name | 必需 | 说明
----|----|---
__input__ | false | 要编制索引的资产文件。</p><p>Azure Media Indexer 支持以下媒体文件格式：MP4、WMV、MP3、M4A、WMA、AAC、WAV。</p><p>你可以在 **input** 元素的 **name** 或 **list** 属性中指定文件名（如下所示）。如果未指定要编制索引的资产文件，将会选择主文件。如果未设置主资产文件，将为输入资产中的第一个文件编制索引。</p><p>若要显式指定资产文件名，请执行以下操作：<br />`<input name="TestFile.wmv">`<br /><br />还可以一次性为多个资产文件编制索引（最多 10 个文件）。为此，请执行以下操作：<br /><br /><ol class="ordered"><li><p>创建一个文本文件（清单文件），并为其指定扩展名 .lst。</p></li><li><p>将输入资产中所有资产文件名的列表添加到此清单文件。</p></li><li><p>将该清单文件添加（上载）到资源。</p></li><li><p>在输入的列表属性中指定清单文件的名称。<br />`<input list="input.lst">`</li></ol><br /><br />注意：如果添加到清单文件的文件超过 10 个，索引作业将会失败，并显示 2006 错误代码。
__metadata__ | false | 用于词汇自适应的指定资产文件的元数据。在预备索引器以使其能够识别非标准词汇（例如专有名词）时，该元素非常有用。<br />`<metadata key="..." value="..."/>` <br /><br />可以提供预定义 __keys__ 的 __values__。当前支持以下键：<br /><br />“title”和“description”- 用于词汇自适应，以微调作业的语言模型及改进语音辨识准确度。值将植入 Internet 搜索以供用户查找上下文相关的文本文档，并在执行索引任务期间使用内容来补充内部字典。<br />`<metadata key="title" value="[Title of the media file]" />`<br />`<metadata key="description" value="[Description of the media file] />"`
__features__ <br /><br />在版本 1.2 中添加。目前，唯一支持的功能是语音识别（“ASR”）。| false | “语音识别”功能具有以下设置项：<table><tr><th><p>键</p></th> <th><p>说明</p></th><th><p>示例值</p></th></tr><tr><td><p>Language</p></td><td><p>要在多媒体文件中识别的自然语言。</p></td><td><p>English、Spanish</p></td></tr><tr><td><p>CaptionFormats</p></td><td><p>所需输出字幕格式的分号分隔列表（如果有）</p></td><td><p>ttml;sami;webvtt</p></td></tr><tr><td><p>GenerateAIB</p></td><td><p>一个布尔值标志，用于指定是否需要 AIB 文件（适用于 SQL Server 和客户索引器 IFilter）。有关详细信息，请参阅<a href="http://azure.microsoft.com/blog/2014/11/03/using-aib-files-with-azure-media-indexer-and-sql-server/">在 Azure Media Indexer 和 SQL Server 中使用 AIB 文件</a>。</p></td><td><p>True；False</p></td></tr><tr><td><p>GenerateKeywords</p></td><td><p>一个布尔值标志，用于指定是否需要关键字 XML 文件。</p></td><td><p>True；False。</p></td></tr><tr><td><p>ForceFullCaption</p></td><td><p>一个布尔值标志，用于指定是否强制完整字幕（不管置信度如何）。</p><p>默认值为 false，在此情况下，将忽略最终字幕输出中置信度小于 50% 的单词和短语并将其替换为省略号（“...”）。省略号可用于字幕质量控制和审核。</p></td><td><p>True；False。</p></td></tr></table>

### <a id="error_codes"></a>错误代码

如果发生错误，Azure Media Indexer 应返回以下错误代码之一：

代码 | Name | 可能的原因
-----|------|------------------
2000 | 配置无效 | 配置无效
2001 | 输入资产无效 | 输入资产缺失或资产为空。
2002 | 清单无效 | 清单为空或清单包含无效项目。
2003 | 无法下载媒体文件 | 清单文件中的 URL 无效。
2004 | 协议不受支持 | 不支持媒体 URL 的协议。
2005 | 文件类型不受支持 | 不支持输入媒体文件类型。
2006 | 输入文件太多 | 输入清单中的文件超过 10 个。
3000 | 无法解码媒体文件 | 媒体编解码器不受支持，<br/>或者<br/>媒体文件已损坏，<br/>或者<br/>输入媒体中没有音频流。
4000 | 分批编制索引部分成功 | 一些输入媒体文件无法编制索引。有关详细信息，请参阅<a href="output_files">输出文件</a>。



## <a id="supported_languages"></a>支持的语言

当前支持英语和西班牙语。有关详细信息，请参阅[版本 v1.2 博客文章](https://azure.microsoft.com/blog/2015/04/13/azure-media-indexer-spanish-v1-2/)。



## 相关链接

[Azure 媒体服务分析概述](/documentation/articles/media-services-analytics-overview/)

[在 Azure Media Indexer 和 SQL Server 中使用 AIB 文件](https://azure.microsoft.com/blog/2014/11/03/using-aib-files-with-azure-media-indexer-and-sql-server/)

[使用 Azure Media Indexer 2 预览版为媒体文件编制索引](/documentation/articles/media-services-process-content-with-indexer2/)
<!---HONumber=Mooncake_0509_2016-->