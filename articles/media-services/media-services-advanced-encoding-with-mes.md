<properties
    pageTitle="通过自定义 MES 预设执行高级编码 | Azure"
    description="本主题说明如何通过自定义 Media Encoder Standard 任务预设执行高级编码。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="2a4ade25-e600-4bce-a66e-e29cf4a38369"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/05/2017"
    wacn.date="02/24/2017"
    ms.author="juliako" />  


# 通过自定义 MES 预设执行高级编码 

## 概述

本主题演示如何自定义 Media Encoder Standard 预设。[通过使用自定义预设的 Media Encoder Standard 进行编码](/documentation/articles/media-services-custom-mes-presets-with-dotnet/)主题演示如何使用 .NET 创建编码任务和执行此任务的作业。自定义预设后，请将其提供给编码任务。

本主题演示了执行以下编码任务的自定义预设：

- [生成缩略图](#thumbnails)
- [修剪视频（裁剪）](#trim_video)
- [创建覆盖层](#overlay)
- [在输入不包含音频时插入静音曲目](#silent_audio)
- [禁用自动取消隔行扫描](#deinterlacing)
- [仅音频预设](#audio_only)
- [连接两个或更多个视频文件](#concatenate)
- [使用媒体编码器标准版裁剪视频](#crop)
- [在输入不包含视频时插入视频轨迹](#no_video)
- [旋转视频](#rotate_video)

## 支持相对大小

生成缩略图时，不需始终以像素为单位指定输出宽度和高度。你可以以百分比的方式在 \[1%, …, 100%\] 范围内对其进行指定。

###JSON 预设 
	
	"Width": "100%",
	"Height": "100%"

###XML 预设
	
	<Width>100%</Width>
	<Height>100%</Height>
	
##<a id="thumbnails"></a>生成缩略图

本部分说明如何自定义生成缩略图的预设。下面定义的预设包含有关如何将文件编码的信息，以及生成缩略图时所需的信息。可使用[此部分](/documentation/articles/media-services-mes-presets-overview/)所述的任何 MES 预设，并添加生成缩略图的代码。

>[AZURE.NOTE]如果要编码为单比特率视频，以下预设中的 **SceneChangeDetection** 设置只能设置为 true。如果要编码为多比特率视频并将 **SceneChangeDetection** 设置为 true，则编码器将返回错误。



有关架构的信息，请参阅[此主题](/documentation/articles/media-services-mes-schema/)。

请务必仔细阅读[注意事项](#considerations)部分。

###<a id="json"></a>JSON 预设


	{
	  "Version": 1.0,
	  "Codecs": [
	    {
	      "KeyFrameInterval": "00:00:02",
	      "SceneChangeDetection": "true",
	      "H264Layers": [
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 4500,
	          "MaxBitrate": 4500,
	          "BufferWindow": "00:00:05",
	          "Width": 1280,
	          "Height": 720,
	          "ReferenceFrames": 3,
	          "EntropyMode": "Cabac",
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	   
	        }
	      ],
	      "Type": "H264Video"
	    },
	    {
	      "JpgLayers": [
	        {
	          "Quality": 90,
	          "Type": "JpgLayer",
	          "Width": 640,
	          "Height": 360
	        }
	      ],
	      "Start": "{Best}",
	      "Type": "JpgImage"
	    },
	    {
	      "PngLayers": [
	        {
	          "Type": "PngLayer",
	          "Width": 640,
	          "Height": 360,
	        }
	      ],
	      "Start": "00:00:01",
		  "Step": "00:00:10",
	      "Range": "00:00:58",
	      "Type": "PngImage"
	    },
	    {
	      "BmpLayers": [
	        {
	          "Type": "BmpLayer",
	          "Width": 640,
	          "Height": 360
	        }
	      ],
	      "Start": "10%",
		  "Step": "10%",
	      "Range": "90%",
	      "Type": "BmpImage"
	    },
	    {
	      "Channels": 2,
	      "SamplingRate": 48000,
	      "Bitrate": 128,
	      "Type": "AACAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}_{Index}{Extension}",
	      "Format": {
	        "Type": "JpgFormat"
	      }
	    },
	    {
	      "FileName": "{Basename}_{Index}{Extension}",
	      "Format": {
	        "Type": "PngFormat"
	      }
	    },
	    {
	      "FileName": "{Basename}_{Index}{Extension}",
	      "Format": {
	        "Type": "BmpFormat"
	      }
	    },
	    {
	      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	}


###<a id="xml"></a>XML 预设


	<?xml version="1.0" encoding="utf-16"?>
	<Preset xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="http://www.windowsazure.com/media/encoding/Preset/2014/03">
	  <Encoding>
	    <H264Video>
	      <KeyFrameInterval>00:00:02</KeyFrameInterval>
	      <SceneChangeDetection>true</SceneChangeDetection>
	      <H264Layers>
	        <H264Layer>
	          <Bitrate>4500</Bitrate>
	          <Width>1280</Width>
	          <Height>720</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>4500</MaxBitrate>
	        </H264Layer>
	      </H264Layers>
	    </H264Video>
	    <AACAudio>
	      <Profile>AACLC</Profile>
	      <Channels>2</Channels>
	      <SamplingRate>48000</SamplingRate>
	      <Bitrate>128</Bitrate>
	    </AACAudio>
	    <JpgImage Start="{Best}">
	      <JpgLayers>
	        <JpgLayer>
	          <Width>640</Width>
	          <Height>360</Height>
	          <Quality>90</Quality>
	        </JpgLayer>
	      </JpgLayers>
	    </JpgImage>
	    <BmpImage Start="10%" Step="10%" Range="90%">
	      <BmpLayers>
	        <BmpLayer>
	          <Width>640</Width>
	          <Height>360</Height>
	        </BmpLayer>
	      </BmpLayers>
	    </BmpImage>
	    <PngImage Start="00:00:01" Step="00:00:10" Range="00:00:58">
	      <PngLayers>
	        <PngLayer>
	          <Width>640</Width>
	          <Height>360</Height>
	        </PngLayer>
	      </PngLayers>
	    </PngImage>
	  </Encoding>
	  <Outputs>
	    <Output FileName="{Basename}_{Width}x{Height}_{VideoBitrate}.mp4">
	      <MP4Format />
	    </Output>
	    <Output FileName="{Basename}_{Index}{Extension}">
	      <JpgFormat />
	    </Output>
	    <Output FileName="{Basename}_{Index}{Extension}">
	      <BmpFormat />
	    </Output>
	    <Output FileName="{Basename}_{Index}{Extension}">
	      <PngFormat />
	    </Output>
	  </Outputs>
	</Preset>

###<a name="considerations"></a>注意事项

请注意以下事项：

- 为 Start/Step/Range 使用的显式时间戳假设输入源的长度至少为 1 分钟。
- Jpg/Png/BmpImage 元素包含 Start、Step 和 Range 字符串属性 – 这些属性解释如下：

	- 帧数（如果为非负整数），例如："Start": "120"；
	- 相对于源持续时间（如果以 % 为后缀表示），例如："Start": "15%"，或者
	- 时间戳（如果以 HH:MM:SS... 格式表示），例如，"Start" : "00:01:00"

	你可以随意混搭使用表示法。
	
	此外，Start 还支持特殊的宏 {Best}，它会尝试判断第一个“有意义”的内容帧。注意：（Start 设置为 {Best} 时，将忽略 Step 与 Range）
	
	- 默认值：Start:{Best}
- 需要显式提供每个图像格式的输出格式：Jpg/Png/BmpFormat。MES 会将 JpgVideo（如果已指定）与 JpgFormat 进行匹配，依此类推。OutputFormat 引入了新的图像编解码器特定宏 {Index}，需要为图像输出格式提供该宏一次（且只需一次）。

## <a id="trim_video"></a>修剪视频（裁剪）
本部分说明如何修改编码器预设，以裁剪或修剪其输入为所谓的夹层文件或按需文件的输入视频。也可以使用编码器来剪切或剪裁从实时流捕获或存档的资产 – [此博客](https://azure.microsoft.com/blog/sub-clipping-and-live-archive-extraction-with-media-encoder-standard/)提供了详细信息。

若要裁剪视频，可以使用[此部分](/documentation/articles/media-services-mes-presets-overview/)所述的任何 MES 预设，并修改 **Sources** 元素（如下所示）。StartTime 的值需与输入视频的绝对时间戳匹配。例如，如果输入视频第一帧的时间戳为 12:00:10.000，则 StartTime 应大于或等于 12:00:10.000。在以下示例中，假设输入视频的起始时间戳为零。**Sources** 应位于预设的开始处。
 
###<a id="json"></a>JSON 预设
	
	{
	  "Version": 1.0,
	  "Sources": [
	    {
	      "StartTime": "00:00:04",
	      "Duration": "00:00:16"
	    }
	  ],
	  "Codecs": [
	    {
	      "KeyFrameInterval": "00:00:02",
	      "StretchMode": "AutoSize",
	      "H264Layers": [
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 3400,
	          "MaxBitrate": 3400,
	          "BufferWindow": "00:00:05",
	          "Width": 1280,
	          "Height": 720,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        },
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 2250,
	          "MaxBitrate": 2250,
	          "BufferWindow": "00:00:05",
	          "Width": 960,
	          "Height": 540,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        },
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 1500,
	          "MaxBitrate": 1500,
	          "BufferWindow": "00:00:05",
	          "Width": 960,
	          "Height": 540,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        },
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 1000,
	          "MaxBitrate": 1000,
	          "BufferWindow": "00:00:05",
	          "Width": 640,
	          "Height": 360,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        },
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 650,
	          "MaxBitrate": 650,
	          "BufferWindow": "00:00:05",
	          "Width": 640,
	          "Height": 360,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        },
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 400,
	          "MaxBitrate": 400,
	          "BufferWindow": "00:00:05",
	          "Width": 320,
	          "Height": 180,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        }
	      ],
	      "Type": "H264Video"
	    },
	    {
	      "Profile": "AACLC",
	      "Channels": 2,
	      "SamplingRate": 48000,
	      "Bitrate": 128,
	      "Type": "AACAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	} 

###XML 预设
	
若要修剪视频，可以使用[此处](/documentation/articles/media-services-mes-presets-overview/)所述的任何 MES 预设并修改 **Sources** 元素（如下所示）。

	<?xml version="1.0" encoding="utf-16"?>
	<Preset xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="http://www.windowsazure.com/media/encoding/Preset/2014/03">
	  <Sources>
	    <Source StartTime="PT4S" Duration="PT14S"/>
	  </Sources>
	  <Encoding>
	    <H264Video>
	      <KeyFrameInterval>00:00:02</KeyFrameInterval>
	      <H264Layers>
	        <H264Layer>
	          <Bitrate>3400</Bitrate>
	          <Width>1280</Width>
	          <Height>720</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>3400</MaxBitrate>
	        </H264Layer>
	        <H264Layer>
	          <Bitrate>2250</Bitrate>
	          <Width>960</Width>
	          <Height>540</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>2250</MaxBitrate>
	        </H264Layer>
	        <H264Layer>
	          <Bitrate>1500</Bitrate>
	          <Width>960</Width>
	          <Height>540</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>1500</MaxBitrate>
	        </H264Layer>
	        <H264Layer>
	          <Bitrate>1000</Bitrate>
	          <Width>640</Width>
	          <Height>360</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>1000</MaxBitrate>
	        </H264Layer>
	        <H264Layer>
	          <Bitrate>650</Bitrate>
	          <Width>640</Width>
	          <Height>360</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>650</MaxBitrate>
	        </H264Layer>
	        <H264Layer>
	          <Bitrate>400</Bitrate>
	          <Width>320</Width>
	          <Height>180</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>400</MaxBitrate>
	        </H264Layer>
	      </H264Layers>
	    </H264Video>
	    <AACAudio>
	      <Profile>AACLC</Profile>
	      <Channels>2</Channels>
	      <SamplingRate>48000</SamplingRate>
	      <Bitrate>128</Bitrate>
	    </AACAudio>
	  </Encoding>
	  <Outputs>
	    <Output FileName="{Basename}_{Width}x{Height}_{VideoBitrate}.mp4">
	      <MP4Format />
	    </Output>
	  </Outputs>
	</Preset>

##<a id="overlay"></a>创建覆盖层

Media Encoder Standard 允许在现有视频上覆盖图像。目前支持以下格式：png、jpg、gif 和 bmp。下面定义的预设是视频覆盖层的基本示例。

除了定义预设文件外，还必须让媒体服务知道资产中的哪个文件是覆盖层图像，哪个文件是要在其上覆盖图像的源视频。视频文件必须是**主**文件。

如果使用 .NET，请将以下两个函数添加到[此主题](/documentation/articles/media-services-custom-mes-presets-with-dotnet/#encoding_with_dotnet)中定义的 .NET 示例。**UploadMediaFilesFromFolder** 函数从文件夹上传文件（例如 BigBuckBunny.mp4 和 Image001.png），并将 mp4 文件设置为资产中的主文件。**EncodeWithOverlay** 函数使用传递给它的自定义预设文件（例如，下面的预设）来创建编码任务。


	static public IAsset UploadMediaFilesFromFolder(string folderPath)
	{
	    IAsset asset = _context.Assets.CreateFromFolder(folderPath, AssetCreationOptions.None);
	
	    foreach (var af in asset.AssetFiles)
	    {
	        // The following code assumes 
	        // you have an input folder with one MP4 and one overlay image file.
	        if (af.Name.Contains(".mp4"))
	            af.IsPrimary = true;
	        else
	            af.IsPrimary = false;
	
	        af.Update();
	    }
	
	    return asset;
	}

    static public IAsset EncodeWithOverlay(IAsset assetSource, string customPresetFileName)
    {
        // Declare a new job.
        IJob job = _context.Jobs.Create("Media Encoder Standard Job");
        // Get a media processor reference, and pass to it the name of the 
        // processor to use for the specific task.
        IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

        // Load the XML (or JSON) from the local file.
        string configuration = File.ReadAllText(customPresetFileName);

        // Create a task
        ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
            processor,
            configuration,
            TaskOptions.None);

        // Specify the input assets to be encoded.
        // This asset contains a source file and an overlay file.
        task.InputAssets.Add(assetSource);

        // Add an output asset to contain the results of the job. 
        task.OutputAssets.AddNew("Output asset",
            AssetCreationOptions.None);

        job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
        job.Submit();
        job.GetExecutionProgressTask(CancellationToken.None).Wait();

        return job.OutputMediaAssets[0];
    }


> [AZURE.NOTE]
当前限制：
>
>不支持覆盖层不透明度设置。
>
>源视频文件和覆盖层图像文件必须位于相同的资产中，而且视频文件需要设置为此资产中的主文件。

###JSON 预设
	
	{
	  "Version": 1.0,
	  "Sources": [
	    {
	      "Streams": [],
	      "Filters": {
	        "VideoOverlay": {
	          "Position": {
	            "X": 100,
	            "Y": 100,
	            "Width": 100,
	            "Height": 50
	          },
	          "AudioGainLevel": 0.0,
	          "MediaParams": [
	            {
	              "OverlayLoopCount": 1
	            },
	            {
	              "IsOverlay": true,
	              "OverlayLoopCount": 1,
	              "InputLoop": true
	            }
	          ],
	          "Source": "Image001.png",
	          "Clip": {
	            "Duration": "00:00:05"
	          },
	          "FadeInDuration": {
	            "Duration": "00:00:01"
	          },
	          "FadeOutDuration": {
	            "StartTime": "00:00:03",
	            "Duration": "00:00:04"
	          }
	        }
	      },
	      "Pad": true
	    }
	  ],
	  "Codecs": [
	    {
	      "KeyFrameInterval": "00:00:02",
	      "H264Layers": [
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 1045,
	          "MaxBitrate": 1045,
	          "BufferWindow": "00:00:05",
	          "ReferenceFrames": 3,
	          "EntropyMode": "Cavlc",
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "Width": "640",
	          "Height": "360",
	          "FrameRate": "0/1"
	        }
	      ],
	      "Type": "H264Video"
	    },
	    {
	      "Type": "CopyAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}{Extension}",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	}


###XML 预设
	
	<?xml version="1.0" encoding="utf-16"?>
	<Preset xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="http://www.windowsazure.com/media/encoding/Preset/2014/03">
	  <Sources>
	    <Source>
	      <Streams />
	      <Filters>
	        <VideoOverlay>
	          <Source>Image001.png</Source>
	          <Clip Duration="PT5S" />
	          <FadeInDuration Duration="PT1S" />
	          <FadeOutDuration StartTime="PT3S" Duration="PT4S" />
	          <Position X="100" Y="100" Width="100" Height="50" />
	          <Opacity>0</Opacity>
	          <AudioGainLevel>0</AudioGainLevel>
	          <MediaParams>
	            <MediaParam>
	              <IsOverlay>false</IsOverlay>
	              <OverlayLoopCount>1</OverlayLoopCount>
	              <InputLoop>false</InputLoop>
	            </MediaParam>
	            <MediaParam>
	              <IsOverlay>true</IsOverlay>
	              <OverlayLoopCount>1</OverlayLoopCount>
	              <InputLoop>true</InputLoop>
	            </MediaParam>
	          </MediaParams>
	        </VideoOverlay>
	      </Filters>
	      <Pad>true</Pad>
	    </Source>
	  </Sources>
	  <Encoding>
	    <H264Video>
	      <KeyFrameInterval>00:00:02</KeyFrameInterval>
	      <H264Layers>
	        <H264Layer>
	          <Bitrate>1045</Bitrate>
	          <Width>640</Width>
	          <Height>360</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>0</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cavlc</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>1045</MaxBitrate>
	        </H264Layer>
	      </H264Layers>
	    </H264Video>
	    <CopyAudio />
	  </Encoding>
	  <Outputs>
	    <Output FileName="{Basename}{Extension}">
	      <MP4Format />
	    </Output>
	  </Outputs>
	</Preset>


##<a id="silent_audio"></a>在输入不包含音频时插入静音曲目

默认情况下，如果要向编码器发送仅包含视频而不包含音频的输入，则输出资产将包含仅有视频数据的文件。某些播放器可能无法处理此类输出流。对于这种方案，你可以使用此设置来强制编码器将静音曲目添加到输出。

若要强制编码器在输入不包含音频时生成包含静音曲目的资产，请指定“InsertSilenceIfNoAudio”值。

可使用[此部分](/documentation/articles/media-services-mes-presets-overview/)中所述的任何 MES 预设，并进行以下修改：

###JSON 预设

    {
      "Channels": 2,
      "SamplingRate": 44100,
      "Bitrate": 96,
      "Type": "AACAudio",
      "Condition": "InsertSilenceIfNoAudio"
    }

###XML 预设

    <AACAudio Condition="InsertSilenceIfNoAudio">
      <Channels>2</Channels>
      <SamplingRate>44100</SamplingRate>
      <Bitrate>96</Bitrate>
    </AACAudio>

##<a id="deinterlacing"></a>禁用自动取消隔行扫描

如果客户想要将隔行扫描内容自动取消隔行扫描，不需要执行任何操作。当自动取消隔行扫描打开（默认设置）时，MES 将自动检测隔行扫描帧，并且只将标记为隔行扫描的帧取消隔行扫描。

你可以关闭自动取消隔行扫描，但不建议这样做。

###JSON 预设
	
	"Sources": [
	{
	 "Filters": {
	    "Deinterlace": {
	      "Mode": "Off"
	    }
	  },
	}
	]

###XML 预设
	
	<Sources>
	<Source>
	  <Filters>
	    <Deinterlace>
	      <Mode>Off</Mode>
	    </Deinterlace>
	  </Filters>
	</Source>
	</Sources>


##<a id="audio_only"></a>仅音频预设

本部分介绍两个仅音频 MES 预设：AAC 音频和 AAC 优质音频。

###AAC 音频 

	{
	  "Version": 1.0,
	  "Codecs": [
	    {
	      "Profile": "AACLC",
	      "Channels": 2,
	      "SamplingRate": 48000,
	      "Bitrate": 128,
	      "Type": "AACAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}_AAC_{AudioBitrate}.mp4",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	}

###AAC 优质音频

	{
	  "Version": 1.0,
	  "Codecs": [
	    {
	      "Profile": "AACLC",
	      "Channels": 2,
	      "SamplingRate": 48000,
	      "Bitrate": 192,
	      "Type": "AACAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}_AAC_{AudioBitrate}.mp4",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	}

##<a id="concatenate"></a>连接两个或更多视频文件

以下示例演示如何生成预设来连接两个或更多个视频文件。最常见的应用场景：你想在主视频中添加标题或预告片。预期使用场合：当一起编辑的视频文件共享属性（视频分辨率、帧速率、音轨计数等）时。务必注意不要混合使用不同帧速率或不同音轨数的视频。

>[AZURE.NOTE]
串联功能的当前设计要求输入视频剪辑在分辨率、帧率等方面保持一致。

###要求和注意事项

- 输入视频应只有一个音轨。
- 输入视频的帧速率应该都相同。
- 必须将视频上载到不同的资产，并将视频设置为每个资产中的主文件。
- 需要知道视频的持续时间。
- 以下预设示例假设所有输入视频的起始时间戳都为零。如果视频具有不同的起始时间戳（通常是实时存档的情况），则需要修改 StartTime 值。
- JSON 预设会显式引用输入资产的 AssetID 值。
- 示例代码假设 JSON 预设已保存到本地文件（例如“C:\\supportFiles\\preset.json”）。同时假设已通过上载两个视频文件创建了两个资产，并且你知悉生成的 AssetID 值。
- 代码片段和 JSON 预设显示连接两个视频文件的示例。你可以将其扩展至两个以上的视频，方法是：

	1. 重复调用 task. InputAssets.Add() 以便依次添加更多视频。
	2. 通过按相同顺序添加更多条目，对 JSON 中的“Sources”元素进行相应编辑。


###<a name="encoding_with_dotnet"></a>.NET 代码

	
	IAsset asset1 = _context.Assets.Where(asset => asset.Id == "nb:cid:UUID:606db602-efd7-4436-97b4-c0b867ba195b").FirstOrDefault();
	IAsset asset2 = _context.Assets.Where(asset => asset.Id == "nb:cid:UUID:a7e2b90f-0565-4a94-87fe-0a9fa07b9c7e").FirstOrDefault();
	
	// Declare a new job.
	IJob job = _context.Jobs.Create("Media Encoder Standard Job for Concatenating Videos");
	// Get a media processor reference, and pass to it the name of the 
	// processor to use for the specific task.
	IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");
	
	// Load the XML (or JSON) from the local file.
	string configuration = File.ReadAllText(@"c:\supportFiles\preset.json");
	
	// Create a task
	ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
	    processor,
	    configuration,
	    TaskOptions.None);
	
	// Specify the input videos to be concatenated (in order).
	task.InputAssets.Add(asset1);
	task.InputAssets.Add(asset2);
	// Add an output asset to contain the results of the job. 
	// This output is specified as AssetCreationOptions.None, which 
	// means the output asset is not encrypted. 
	task.OutputAssets.AddNew("Output asset",
	    AssetCreationOptions.None);
	
	job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
	job.Submit();
	job.GetExecutionProgressTask(CancellationToken.None).Wait();

###JSON 预设

使用你想连接的资产 ID 以及每个视频的适当时间段，更新你的自定义预设。

	{
	  "Version": 1.0,
	  "Sources": [
	    {
	      "AssetID": "606db602-efd7-4436-97b4-c0b867ba195b",
	      "StartTime": "00:00:01",
	      "Duration": "00:00:15"
	    },
	    {
	      "AssetID": "a7e2b90f-0565-4a94-87fe-0a9fa07b9c7e",
	      "StartTime": "00:00:02",
	      "Duration": "00:00:05"
	    }
	  ],
	  "Codecs": [
	    {
	      "KeyFrameInterval": "00:00:02",
	      "SceneChangeDetection": true,
	      "H264Layers": [
	        {
	          "Level": "auto",
	          "Bitrate": 1800,
	          "MaxBitrate": 1800,
	          "BufferWindow": "00:00:05",
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "Width": "640",
	          "Height": "360",
	          "FrameRate": "0/1"
	        }
	      ],
	      "Type": "H264Video"
	    },
	    {
	      "Channels": 2,
	      "SamplingRate": 48000,
	      "Bitrate": 128,
	      "Type": "AACAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	}

##<a id="crop"></a>使用媒体编码器标准版裁剪视频

请参阅主题 [Crop videos with Media Encoder Standard](/documentation/articles/media-services-crop-video/)（使用媒体编码器标准版裁剪视频）主题。

##<a id="no_video"></a>在输入不包含视频时插入视频轨迹

默认情况下，如果要向编码器发送仅包含音频而不包含视频的输入，则输出资产将包含仅有音频数据的文件。某些播放器（包括 Azure 媒体播放器）（请参阅[此处](https://feedback.azure.com/forums/169396-azure-media-services/suggestions/8082468-audio-only-scenarios)）可能无法处理这样的流。对于这种方案，可使用此设置来强制编码器将单色视频轨迹添加到输出。

>[AZURE.NOTE]强制编码器插入输出视频轨迹会增加输出资产的大小，从而增加编码任务的相关成本。应运行测试来验证此成本增加对每月费用的影响不大。

### 仅以最低比特率插入视频

假设要使用多比特率编码预设（如[“H264 多比特率 720p”](https://msdn.microsoft.com/zh-cn/library/mt269960.aspx)）对整个输入目录进行编码以实现流式处理，且输入目录中混合了视频文件和仅音频文件。在此方案中，如果输入不包含视频，用户可能想要强制编码器仅以最低比特率插入单色视频轨迹，而不是按每个输出比特率插入视频。要实现此目的，需要指定“InsertBlackIfNoVideoBottomLayerOnly”标志。

可使用[此部分](/documentation/articles/media-services-mes-presets-overview/)中所述的任何 MES 预设，并进行以下修改：

#### JSON 预设

	{
	      "KeyFrameInterval": "00:00:02",
	      "StretchMode": "AutoSize",
	      "Condition": "InsertBlackIfNoVideoBottomLayerOnly",
	      "H264Layers": [
	      …
	      ]
	}

#### XML 预设

	<KeyFrameInterval>00:00:02</KeyFrameInterval>
	<StretchMode>AutoSize</StretchMode>
	<Condition>InsertBlackIfNoVideoBottomLayerOnly</Condition>

### 按所有输出比特率插入视频

假设要使用多比特率编码预设（如[“H264 多比特率 720p”](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-720p/)）对整个输入目录进行编码以实现流式处理，且输入目录中混合了视频文件和仅音频文件。在此方案中，如果输入不包含视频，用户可能想要强制编码器按所有输出比特率插入单色视频轨迹。这可确保对于视频轨迹和音频曲目的数目，输出资产都是同源的。要实现此目的，需要指定“InsertBlackIfNoVideo”标志。

可使用[此部分](/documentation/articles/media-services-mes-presets-overview/)中所述的任何 MES 预设，并进行以下修改：

#### JSON 预设

	{
	      "KeyFrameInterval": "00:00:02",
	      "StretchMode": "AutoSize",
	      "Condition": "InsertBlackIfNoVideo",
	      "H264Layers": [
	      …
	      ]
	}

#### XML 预设
	
	<KeyFrameInterval>00:00:02</KeyFrameInterval>
	<StretchMode>AutoSize</StretchMode>
	<Condition>InsertBlackIfNoVideo</Condition>

## <a id="rotate_video"></a>旋转视频
[Media Encoder Standard](/documentation/articles/media-services-dotnet-encode-with-media-encoder-standard/) 支持的旋转角度为 0/90/180/270。默认行为是“自动”，即尝试在传入的视频文件中检测旋转元数据并对其进行补偿。包含[此部分](/documentation/articles/media-services-mes-presets-overview/)中定义的预设之一的以下 **Sources** 元素：

### JSON 预设
    "Sources": [
    {
      "Streams": [],
      "Filters": {
        "Rotation": "90"
      }
    }
    ],
    "Codecs": [

    ...
### XML 预设
    <Sources>
           <Source>
          <Streams />
          <Filters>
            <Rotation>90</Rotation>
          </Filters>
        </Source>
    </Sources>

另请参阅[此](/documentation/articles/media-services-mes-schema/#PreserveResolutionAfterRotation)主题，了解有关编码器如何在触发旋转补偿后解释预设中的宽度和高度设置的详细信息。

可以使用值“0”指示编码器忽略输入视频中的旋转元数据（如果存在）。



## 另请参阅
[媒体服务编码概述](/documentation/articles/media-services-encode-asset/)

<!---HONumber=Mooncake_0220_2017-->
<!--Description_Update: add one note for Concatenate feature-->
