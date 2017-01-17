<properties
    pageTitle="H264 单比特率 4K | Azure"
    description="本主题概述了 **H264 单比特率 4K** 任务预设。"
    author="Juliako"
    manager="erikre"
    editor=""
    services="media-services"
    documentationcenter="" />
<tags
    ms.assetid="8e437aea-8193-49a0-9ff2-4fd391c80972"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/23/2016"
    wacn.date="01/13/2017"
    ms.author="juliako" />  


# H264 单比特率 4K
`Media Encoder Standard` 定义一组可在创建编码作业时使用的编码预设。可使用 `preset name` 指定要将媒体文件编码为哪种格式。或者，可创建自己的基于 JSON 或 XML 的预设（使用 UTF-8 或 UTF-16 编码）。然后，将自定义预设传递到编码器。有关此 `Media Encoder Standard` 编码器支持的所有预设名称的列表，请参阅 [Media Encoder Standard 的任务预设](/documentation/articles/media-services-mes-presets-overview/)。
  
 本主题演示 XML 和 JSON 格式的 `H264 Single Bitrate 4K` 预设。
  
 此预设将生成单个 MP4 文件，其比特率为 18000 kbps，并且带有立体声 AAC 音频。若要深入了解此预设的配置文件、比特率、采样率等，请检查下面定义的 XML 或 JSON。有关这些预设中每个元素含义的说明以及每个元素的有效值，请参阅 [Media Encoder Standard 架构](/documentation/articles/media-services-mes-schema/)主题。
  
> [AZURE.NOTE]
应利用 4K 编码获取“高级版”保留单位类型。有关详细信息，请参阅[如何缩放编码](/documentation/articles/media-services-portal-encoding-units/)。
  
 XML
  

	<?xml version="1.0" encoding="utf-16"?>  
	<Preset xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="http://www.windowsazure.com/media/encoding/Preset/2014/03">  
	  <Encoding>  
	    <H264Video>  
	      <KeyFrameInterval>00:00:02</KeyFrameInterval>  
	      <SceneChangeDetection>true</SceneChangeDetection>  
	      <H264Layers>  
	        <H264Layer>  
	          <Bitrate>18000</Bitrate>  
	          <Width>3840</Width>  
	          <Height>2160</Height>  
	          <FrameRate>0/1</FrameRate>  
	          <Profile>Auto</Profile>  
	          <Level>auto</Level>  
	          <BFrames>3</BFrames>  
	          <ReferenceFrames>3</ReferenceFrames>  
	          <Slices>0</Slices>  
	          <AdaptiveBFrame>true</AdaptiveBFrame>  
	          <EntropyMode>Cabac</EntropyMode>  
	          <BufferWindow>00:00:05</BufferWindow>  
	          <MaxBitrate>18000</MaxBitrate>  
	        </H264Layer>  
	      </H264Layers>  
	      <Chapters />  
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
 
  
 JSON
  

	{  
	  "Version": 1.0,  
	  "Codecs": [  
	    {  
	      "KeyFrameInterval": "00:00:02",  
	      "SceneChangeDetection": true,  
	      "H264Layers": [  
	        {  
	          "Profile": "Auto",  
	          "Level": "auto",  
	          "Bitrate": 18000,  
	          "MaxBitrate": 18000,  
	          "BufferWindow": "00:00:05",  
	          "Width": 3840,  
	          "Height": 2160,  
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

<!---HONumber=Mooncake_0109_2017-->