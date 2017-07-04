<properties
    pageTitle="H264 多比特率 720p 音频 5.1 | Azure"
    description="本主题概述了 **H264 多比特率 720p 音频 5.1** 任务预设。"
    author="Juliako"
    manager="erikre"
    editor=""
    services="media-services"
    documentationcenter="" />
<tags
    ms.assetid="92da14ad-7b04-4bd9-8eb5-9103f38441be"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/23/2016"
    wacn.date="01/13/2017"
    ms.author="juliako" />  


# H264 多比特率 720p 音频 5.1
`Media Encoder Standard` 定义一组可在创建编码作业时使用的编码预设。可使用 `preset name` 指定要将媒体文件编码为哪种格式。或者，可创建自己的基于 JSON 或 XML 的预设（使用 UTF-8 或 UTF-16 编码）。然后，将自定义预设传递到编码器。有关此 `Media Encoder Standard` 编码器支持的所有预设名称的列表，请参阅 [Media Encoder Standard 的任务预设](/documentation/articles/media-services-mes-presets-overview/)。
  
 本主题演示 XML 和 JSON 格式的 `H264 Multiple Bitrate 720p Audio 5.1` 预设。
  
 此预设将生成一组 6 个 GOP 对齐的 MP4 文件，范围为 3400 kbps - 400 kbps，以及 AAC 5.1 音频。若要深入了解此预设的配置文件、比特率、采样率等，请检查下面定义的 XML 或 JSON。有关每个元素含义的说明以及每个元素的有效值，请参阅 [Media Encoder Standard 架构](/documentation/articles/media-services-mes-schema/)。
  
> [AZURE.NOTE]
修改各层的 `Width` 和 `Height` 值时，请确保纵横比保持一致。例如：1920x1080、1280x720、1080x576、640x360。不应使用混合纵横比，如 1280x720、720x480、640x360。
  
 XML
  

	<?xml version="1.0" encoding="utf-16"?>  
	<Preset xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="http://www.windowsazure.com/media/encoding/Preset/2014/03">  
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
	      <Chapters />  
	    </H264Video>  
	    <AACAudio>  
	      <Profile>AACLC</Profile>  
	      <Channels>6</Channels>  
	      <SamplingRate>48000</SamplingRate>  
	      <Bitrate>384</Bitrate>  
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
	      "Channels": 6,  
	      "SamplingRate": 48000,  
	      "Bitrate": 384,  
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