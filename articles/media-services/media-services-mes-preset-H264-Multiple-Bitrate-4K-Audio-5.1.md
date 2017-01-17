<properties
    pageTitle="264 多比特率 4K 音频 5.1 | Azure"
    description="本主题概述了 **264 多比特率 4K 音频 5.1** 任务预设。"
    author="Juliako"
    manager="erikre"
    editor=""
    services="media-services"
    documentationcenter="" />
<tags
    ms.assetid="c8e0bd6a-86ef-481f-83fa-453bdb042df8"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/23/2016"
    wacn.date="01/13/2017"
    ms.author="juliako" />  


# H264 多比特率 4K 音频 5.1
`Media Encoder Standard` 定义一组可在创建编码作业时使用的编码预设。可使用 `preset name` 指定要将媒体文件编码为哪种格式。或者，可创建自己的基于 JSON 或 XML 的预设（使用 UTF-8 或 UTF-16 编码）。然后，将自定义预设传递到编码器。有关此 `Media Encoder Standard` 编码器支持的所有预设名称的列表，请参阅 [Media Encoder Standard 的任务预设](/documentation/articles/media-services-mes-presets-overview/)。
  
 本主题演示 XML 和 JSON 格式的 `H264 Multiple Bitrate 4K Audio 5.1` 预设。
  
 此预设将生成一组 12 个 GOP 对齐的 MP4 文件，范围为 20000 kbps - 1000 kbps，以及 AAC 5.1 音频。若要深入了解此预设的配置文件、比特率、采样率等，请检查下面定义的 XML 或 JSON。有关每个元素含义的说明以及每个元素的有效值，请参阅 [Media Encoder Standard 架构](/documentation/articles/media-services-mes-schema/)。
  
> [AZURE.NOTE]
应利用 4K 编码获取“高级版”保留单位类型。有关详细信息，请参阅[如何缩放编码](/documentation/articles/media-services-portal-encoding-units/)。
  
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
	          <Bitrate>20000</Bitrate>  
	          <Width>4096</Width>  
	          <Height>2304</Height>  
	          <FrameRate>0/1</FrameRate>  
	          <Profile>Auto</Profile>  
	          <Level>auto</Level>  
	          <BFrames>3</BFrames>  
	          <ReferenceFrames>3</ReferenceFrames>  
	          <Slices>0</Slices>  
	          <AdaptiveBFrame>true</AdaptiveBFrame>  
	          <EntropyMode>Cabac</EntropyMode>  
	          <BufferWindow>00:00:05</BufferWindow>  
	          <MaxBitrate>20000</MaxBitrate>  
	        </H264Layer>  
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
	        <H264Layer>  
	          <Bitrate>16000</Bitrate>  
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
	          <MaxBitrate>16000</MaxBitrate>  
	        </H264Layer>  
	        <H264Layer>  
	          <Bitrate>14000</Bitrate>  
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
	          <MaxBitrate>14000</MaxBitrate>  
	        </H264Layer>  
	        <H264Layer>  
	          <Bitrate>12000</Bitrate>  
	          <Width>2560</Width>  
	          <Height>1440</Height>  
	          <FrameRate>0/1</FrameRate>  
	          <Profile>Auto</Profile>  
	          <Level>auto</Level>  
	          <BFrames>3</BFrames>  
	          <ReferenceFrames>3</ReferenceFrames>  
	          <Slices>0</Slices>  
	          <AdaptiveBFrame>true</AdaptiveBFrame>  
	          <EntropyMode>Cabac</EntropyMode>  
	          <BufferWindow>00:00:05</BufferWindow>  
	          <MaxBitrate>12000</MaxBitrate>  
	        </H264Layer>  
	        <H264Layer>  
	          <Bitrate>10000</Bitrate>  
	          <Width>2560</Width>  
	          <Height>1440</Height>  
	          <FrameRate>0/1</FrameRate>  
	          <Profile>Auto</Profile>  
	          <Level>auto</Level>  
	          <BFrames>3</BFrames>  
	          <ReferenceFrames>3</ReferenceFrames>  
	          <Slices>0</Slices>  
	          <AdaptiveBFrame>true</AdaptiveBFrame>  
	          <EntropyMode>Cabac</EntropyMode>  
	          <BufferWindow>00:00:05</BufferWindow>  
	          <MaxBitrate>10000</MaxBitrate>  
	        </H264Layer>  
	        <H264Layer>  
	          <Bitrate>8000</Bitrate>  
	          <Width>2560</Width>  
	          <Height>1440</Height>  
	          <FrameRate>0/1</FrameRate>  
	          <Profile>Auto</Profile>  
	          <Level>auto</Level>  
	          <BFrames>3</BFrames>  
	          <ReferenceFrames>3</ReferenceFrames>  
	          <Slices>0</Slices>  
	          <AdaptiveBFrame>true</AdaptiveBFrame>  
	          <EntropyMode>Cabac</EntropyMode>  
	          <BufferWindow>00:00:05</BufferWindow>  
	          <MaxBitrate>8000</MaxBitrate>  
	        </H264Layer>  
	        <H264Layer>  
	          <Bitrate>6000</Bitrate>  
	          <Width>1920</Width>  
	          <Height>1080</Height>  
	          <FrameRate>0/1</FrameRate>  
	          <Profile>Auto</Profile>  
	          <Level>auto</Level>  
	          <BFrames>3</BFrames>  
	          <ReferenceFrames>3</ReferenceFrames>  
	          <Slices>0</Slices>  
	          <AdaptiveBFrame>true</AdaptiveBFrame>  
	          <EntropyMode>Cabac</EntropyMode>  
	          <BufferWindow>00:00:05</BufferWindow>  
	          <MaxBitrate>6000</MaxBitrate>  
	        </H264Layer>  
	        <H264Layer>  
	          <Bitrate>4700</Bitrate>  
	          <Width>1920</Width>  
	          <Height>1080</Height>  
	          <FrameRate>0/1</FrameRate>  
	          <Profile>Auto</Profile>  
	          <Level>auto</Level>  
	          <BFrames>3</BFrames>  
	          <ReferenceFrames>3</ReferenceFrames>  
	          <Slices>0</Slices>  
	          <AdaptiveBFrame>true</AdaptiveBFrame>  
	          <EntropyMode>Cabac</EntropyMode>  
	          <BufferWindow>00:00:05</BufferWindow>  
	          <MaxBitrate>4700</MaxBitrate>  
	        </H264Layer>  
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
	          "Bitrate": 20000,  
	          "MaxBitrate": 20000,  
	          "BufferWindow": "00:00:05",  
	          "Width": 4096,  
	          "Height": 2304,  
	          "BFrames": 3,  
	          "ReferenceFrames": 3,  
	          "AdaptiveBFrame": true,  
	          "Type": "H264Layer",  
	          "FrameRate": "0/1"  
	        },  
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
	        },  
	        {  
	          "Profile": "Auto",  
	          "Level": "auto",  
	          "Bitrate": 16000,  
	          "MaxBitrate": 16000,  
	          "BufferWindow": "00:00:05",  
	          "Width": 3840,  
	          "Height": 2160,  
	          "BFrames": 3,  
	          "ReferenceFrames": 3,  
	          "AdaptiveBFrame": true,  
	          "Type": "H264Layer",  
	          "FrameRate": "0/1"  
	        },  
	        {  
	          "Profile": "Auto",  
	          "Level": "auto",  
	          "Bitrate": 14000,  
	          "MaxBitrate": 14000,  
	          "BufferWindow": "00:00:05",  
	          "Width": 3840,  
	          "Height": 2160,  
	          "BFrames": 3,  
	          "ReferenceFrames": 3,  
	          "AdaptiveBFrame": true,  
	          "Type": "H264Layer",  
	          "FrameRate": "0/1"  
	        },  
	        {  
	          "Profile": "Auto",  
	          "Level": "auto",  
	          "Bitrate": 12000,  
	          "MaxBitrate": 12000,  
	          "BufferWindow": "00:00:05",  
	          "Width": 2560,  
	          "Height": 1440,  
	          "BFrames": 3,  
	          "ReferenceFrames": 3,  
	          "AdaptiveBFrame": true,  
	          "Type": "H264Layer",  
	          "FrameRate": "0/1"  
	        },  
	        {  
	          "Profile": "Auto",  
	          "Level": "auto",  
	          "Bitrate": 10000,  
	          "MaxBitrate": 10000,  
	          "BufferWindow": "00:00:05",  
	          "Width": 2560,  
	          "Height": 1440,  
	          "BFrames": 3,  
	          "ReferenceFrames": 3,  
	          "AdaptiveBFrame": true,  
	          "Type": "H264Layer",  
	          "FrameRate": "0/1"  
	        },  
	        {  
	          "Profile": "Auto",  
	          "Level": "auto",  
	          "Bitrate": 8000,  
	          "MaxBitrate": 8000,  
	          "BufferWindow": "00:00:05",  
	          "Width": 2560,  
	          "Height": 1440,  
	          "BFrames": 3,  
	          "ReferenceFrames": 3,  
	          "AdaptiveBFrame": true,  
	          "Type": "H264Layer",  
	          "FrameRate": "0/1"  
	        },  
	        {  
	          "Profile": "Auto",  
	          "Level": "auto",  
	          "Bitrate": 6000,  
	          "MaxBitrate": 6000,  
	          "BufferWindow": "00:00:05",  
	          "Width": 1920,  
	          "Height": 1080,  
	          "BFrames": 3,  
	          "ReferenceFrames": 3,  
	          "AdaptiveBFrame": true,  
	          "Type": "H264Layer",  
	          "FrameRate": "0/1"  
	        },  
	        {  
	          "Profile": "Auto",  
	          "Level": "auto",  
	          "Bitrate": 4700,  
	          "MaxBitrate": 4700,  
	          "BufferWindow": "00:00:05",  
	          "Width": 1920,  
	          "Height": 1080,  
	          "BFrames": 3,  
	          "ReferenceFrames": 3,  
	          "AdaptiveBFrame": true,  
	          "Type": "H264Layer",  
	          "FrameRate": "0/1"  
	        },  
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