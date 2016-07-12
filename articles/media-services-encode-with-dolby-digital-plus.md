<properties 
	pageTitle="使用 Dolby Digital Plus 编码媒体" 
	description="本主题介绍如何使用 Dolby Digital Plus 对媒体进行编码。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="09/07/2015"   
	wacn.date="10/22/2015"/>

#使用 Dolby Digital Plus 编码媒体

Azure 媒体编码器支持 **Dolby® Digital Plus** 编码。Dolby® Digital Plus 或高级 AC-3 (E-AC-3) 是一种专为不断进化的媒体设计的高级环绕声音频编解码器。从家庭影院和 PC 到手机和在线流式处理，都可使用 Dolby Digital Plus 来定义高保真音频。你将获得著名的涵盖所有娱乐的杜比影院体验。Dolby Digital Plus 基于核心 Dolby Digital 技术，该技术是影院、广播和家庭影院环绕声的公认标准。随着移动设备数量的不断增加，Dolby Digital Plus 也逐渐成为移动娱乐的标准。它推出的具有音频增强功能的高级新技术可提供更优良的音质，并可进一步节省带宽。它的中断会更少（即使在带宽有限的情况下），为你提供优质的内容。


##将 Azure Media Encoder 设置为使用 Dolby Digital Plus 进行编码

###获取 Azure Media Encoder 处理器 

Azure Media Encoder 支持 Dolby Digital Plus。若要获取对 **Azure 媒体编码器**的引用，请参阅[获取媒体处理器](/documentation/articles/media-services-get-media-processor/)主题。

###<a id="configure_preset"></a>配置 Azure 媒体编码器设置

当编码设置配置为与 Azure Media Encoder 搭配使用时，可以使用各种由易记字符串表示的预定义预设。Dolby Digital Plus 编码器提供了丰富的控件，有关详细信息，请参阅[<DolbyDigitalPlusAudioProfile>](https://msdn.microsoft.com/zh-cn/library/azure/dn296500.aspx)。因此，并没有任何使用此编解码器的预建字符串预设。你必须在 XML 文件中指定所需的编码器设置，然后将此数据与你的任务一起提交，如以下代码示例所示：
	
	string configuration = File.ReadAllText(pathToXMLConfigFile));

    ITask task = job.Tasks.AddNew("My Dolby Digital Plus Encode Task",
        processor,
        configuration,
        _clearConfig);

本主题介绍几个用于配置编码器设置的示例 XML 预设。用于配置 Dolby Digital Plus 编码的元素为 [<DolbyDigitalPlusAudioProfile>](https://msdn.microsoft.com/zh-cn/library/azure/dn296500.aspx)，在 Azure 媒体编码器 XML 预设中，该元素显示为 <AudioProfile> 元素的子节点。此 XML 元素包含大量用于控制各种编码元素的属性。

##编码成 Dolby Digital Plus 5.1 多声道

若要编码为 Dolby Digital Plus 5.1 多声道，请将 Codec 和 EncoderMode 属性设置为“DolbyDigitalPlus”。所编码的通道数使用 <DolbyDigitalPlusAudioProfile> 元素的 AudioCodingMode 属性指定。对于 5.1 多声道编码，请将 AudioCodingMode 设置为“Mode32”。

以下 XML 预设包含一个完整的 Azure Media Encoder XML 预设，该预设可使用 H264 宽带 1080p 视频和 Dolby Digital Plus 5.1 多声道音频生成 MP4 文件。此预设还指定编码低频效果 (LFE) 通道，这是通过将 LFEOn 属性设置为 true 来指定的。任何未指定的属性将使用其默认值。

此 XML 预设应传递到 **Azure 媒体编码器**，以创建[本](/documentation/articles/media-services-dotnet-encode-asset/)主题中所述的编码作业（只不过，你应该传递整个 XML 预设，而不是预定义的预设字符串，如[此处](#configure_preset)所述）。


	<?xml version="1.0" encoding="utf-16"?>
	<!--Created for Azure Media Encoder, May 26 2013 -->
	  <Preset
	    Version="5.0">
	    <Job />
	    <MediaFile
	      DeinterlaceMode="AutoPixelAdaptive"
	      ResizeQuality="Super"
	      AudioGainLevel="1"
	      VideoResizeMode="Stretch">
	      <OutputFormat>
	        <MP4OutputFormat
	          StreamCompatibility="Standard">
	        <AudioProfile Condition="SourceContainsAudio">
	            <DolbyDigitalPlusAudioProfile
	              Codec="DolbyDigitalPlus"
	              EncoderMode="DolbyDigitalPlus"
	              AudioCodingMode="Mode32"
	              LFEOn="True"
	              SamplesPerSecond="48000"
	              BandwidthLimitingLowpassFilter="True"
	              DialogNormalization="-31">
	              <Bitrate>
	                <ConstantBitrate
	                  Bitrate="512"
	                  IsTwoPass="False"
	                  BufferWindow="00:00:00" />
	              </Bitrate>
	            </DolbyDigitalPlusAudioProfile>
	        </AudioProfile>
	          <VideoProfile Condition="SourceContainsVideo">
	            <HighH264VideoProfile
	              BFrameCount="3"
	              EntropyMode="Cabac"
	              RDOptimizationMode="Speed"
	              HadamardTransform="False"
	              SubBlockMotionSearchMode="Speed"
	              MultiReferenceMotionSearchMode="Balanced"
	              ReferenceBFrames="False"
	              AdaptiveBFrames="True"
	              SceneChangeDetector="True"
	              FastIntraDecisions="False"
	              FastInterDecisions="False"
	              SubPixelMode="Quarter"
	              SliceCount="0"
	              KeyFrameDistance="00:00:05"
	              InLoopFilter="True"
	              MEPartitionLevel="EightByEight"
	              ReferenceFrames="4"
	              SearchRange="64"
	              AutoFit="True"
	              Force16Pixels="False"
	              FrameRate="0"
	              SeparateFilesPerStream="True"
	              SmoothStreaming="False"
	              NumberOfEncoderThreads="0">
	              <Streams
	                AutoSize="False"
	                FreezeSort="False">
	                <StreamInfo
	                  Size="1920, 1080">
	                  <Bitrate>
	                    <ConstantBitrate
	                      Bitrate="6000"
	                      IsTwoPass="False"
	                      BufferWindow="00:00:05" />
	                  </Bitrate>
	                </StreamInfo>
	              </Streams>
	            </HighH264VideoProfile>
	          </VideoProfile>
	        </MP4OutputFormat>
	      </OutputFormat>
	    </MediaFile>
	  </Preset>

##编码成 Dolby Digital Plus 立体声

若要编码为 Dolby Digital Plus 立体声，请将 Codec 和 EncoderMode 属性设置为“DolbyDigitalPlus”。所编码的通道数使用 AudioCodingMode 属性指定。对于立体声编码，请将 AudioCodingMode 设置为“Mode20”。以下 XML 预设示例显示了用于编码为 5.1 音频的 <DolbyDigitalPlusAudioProfile>。任何未指定的属性将使用其默认值。

此 XML 预设应传递到 **Azure 媒体编码器**，以创建[本](/documentation/articles/media-services-dotnet-encode-asset/)主题中所述的编码作业（只不过，你应该传递整个 XML 预设，而不是预定义的预设字符串，如[此处](#configure_preset)所述）。

	<?xml version="1.0" encoding="utf-16"?>
	<!--Created for Azure Media Encoder, May 26 2013 -->
	  <Preset
	    Version="5.0">
	    <Job />
	    <MediaFile
	      DeinterlaceMode="AutoPixelAdaptive"
	      ResizeQuality="Super"
	      AudioGainLevel="1"
	      VideoResizeMode="Stretch">
	      <OutputFormat>
	        <MP4OutputFormat
	          StreamCompatibility="Standard">
	        <AudioProfile Condition="SourceContainsAudio">
	            <DolbyDigitalPlusAudioProfile
	              Codec="DolbyDigitalPlus"
	              EncoderMode="DolbyDigitalPlus"
	              AudioCodingMode="Mode20"
	              LFEOn="False"
	              SamplesPerSecond="48000"
	              DialogNormalization="-31">
	              <Bitrate>
	                <ConstantBitrate
	                  Bitrate="128"
	                  IsTwoPass="False"
	                  BufferWindow="00:00:00" />
	              </Bitrate>
	            </DolbyDigitalPlusAudioProfile>
	        </AudioProfile>
	          <VideoProfile Condition="SourceContainsVideo">
	            <HighH264VideoProfile
	              BFrameCount="1"
	              EntropyMode="Cabac"
	              RDOptimizationMode="Speed"
	              HadamardTransform="False"
	              SubBlockMotionSearchMode="Speed"
	              MultiReferenceMotionSearchMode="Speed"
	              ReferenceBFrames="False"
	              AdaptiveBFrames="True"
	              SceneChangeDetector="True"
	              FastIntraDecisions="False"
	              FastInterDecisions="False"
	              SubPixelMode="Quarter"
	              SliceCount="0"
	              KeyFrameDistance="00:00:05"
	              InLoopFilter="True"
	              MEPartitionLevel="EightByEight"
	              ReferenceFrames="4"
	              SearchRange="32"
	              AutoFit="True"
	              Force16Pixels="False"
	              FrameRate="0"
	              SeparateFilesPerStream="True"
	              SmoothStreaming="False"
	              NumberOfEncoderThreads="0">
	              <Streams
	                AutoSize="False"
	                FreezeSort="False">
	              <StreamInfo
	                Size="852, 480">
	                <Bitrate>
	                  <ConstantBitrate
	                    Bitrate="2200"
	                    IsTwoPass="False"
	                    BufferWindow="00:00:05" />
	                </Bitrate>
	              </StreamInfo>
	              </Streams>
	            </HighH264VideoProfile>
	          </VideoProfile>
	        </MP4OutputFormat>
	      </OutputFormat>
	    </MediaFile>
	  </Preset>

##编码为多个 MP4 文件 

你可以在单个 XML 预设中编码为多个 MP4。对于你想要生成的每个 MP4，请将 <Preset> 元素添加到配置中。每个 <Preset> 元素可包含视频和/或音频的配置信息。

###配置

以下配置将生成以下输出：

- 8 个仅视频 MP4 文件
	- 1080p 视频 @ 6000 kbps
	- 1080p 视频 @ 4700 kbps
	- 720p 视频 @ 3400 kbps
	- 960 x 540 视频 @ 2250 kbps
	- 960 x 540 视频 @ 1500 kbps
	- 640 x 380 视频 @ 1000 kbps
	- 640 x 380 视频 @ 650 kbps
	- 320 x 180 视频 @ 400 kbps

- 5 个仅音频 MP4 文件
	- AAC 立体声音频 @ 128 kbp
	- AAC 音频 5.1 @ 512 kbps
	- Dolby Digital Plus 立体声 @ 128 kbps
	- Dolby Digital Plus 5.1 多声道 @ 512 kbps
	- AAC 立体声 @ 56 kbps
- .ism 清单
- 列出所生成的 MP4 文件的属性的 XML 文件。
		
		<?xml version="1.0" encoding="utf-16"?>
		<!--Created for Azure Media Encoder, May 16 2013 -->
		<Presets>
		  <Preset
		    Version="5.0">
		    <Job />
		    <MediaFile
		      DeinterlaceMode="AutoPixelAdaptive"
		      ResizeQuality="Super"   
		      AudioGainLevel="1"
		      VideoResizeMode="Stretch">
		      <OutputFormat>
		        <MP4OutputFormat
		          StreamCompatibility="Standard">
		          <VideoProfile Condition="SourceContainsVideo">
		            <HighH264VideoProfile
		              BFrameCount="3"
		              EntropyMode="Cabac"
		              RDOptimizationMode="Speed"
		              HadamardTransform="False"
		              SubBlockMotionSearchMode="Speed"
		              MultiReferenceMotionSearchMode="Balanced"
		              ReferenceBFrames="False"
		              AdaptiveBFrames="True"
		              SceneChangeDetector="True"
		              FastIntraDecisions="False"
		              FastInterDecisions="False"
		              SubPixelMode="Quarter"
		              SliceCount="0"
		              KeyFrameDistance="00:00:05"
		              InLoopFilter="True"
		              MEPartitionLevel="EightByEight"
		              ReferenceFrames="4"
		              SearchRange="64"
		              AutoFit="True"
		              Force16Pixels="False"
		              FrameRate="0"
		              SeparateFilesPerStream="True"
		              SmoothStreaming="False"
		              NumberOfEncoderThreads="0">
		              <Streams
		                AutoSize="False"
		                FreezeSort="False">
		                <StreamInfo
		                  Size="1920, 1080">
		                  <Bitrate>
		                    <ConstantBitrate
		                      Bitrate="6000"
		                      IsTwoPass="False"
		                      BufferWindow="00:00:05" />
		                  </Bitrate>
		                </StreamInfo>
		                <StreamInfo
		                  Size="1920, 1080">
		                  <Bitrate>
		                    <ConstantBitrate
		                      Bitrate="4700"
		                      IsTwoPass="False"
		                      BufferWindow="00:00:05" />
		                  </Bitrate>
		                </StreamInfo>
		                <StreamInfo
		                  Size="1280, 720">
		                  <Bitrate>
		                    <ConstantBitrate
		                      Bitrate="3400"
		                      IsTwoPass="False"
		                      BufferWindow="00:00:05" />
		                  </Bitrate>
		                </StreamInfo>
		                <StreamInfo
		                  Size="960, 540">
		                  <Bitrate>
		                    <ConstantBitrate
		                      Bitrate="2250"
		                      IsTwoPass="False"
		                      BufferWindow="00:00:05" />
		                  </Bitrate>
		                </StreamInfo>
		                <StreamInfo
		                  Size="960, 540">
		                  <Bitrate>
		                    <ConstantBitrate
		                      Bitrate="1500"
		                      IsTwoPass="False"
		                      BufferWindow="00:00:05" />
		                  </Bitrate>
		                </StreamInfo>
		                <StreamInfo
		                  Size="640, 360">
		                  <Bitrate>
		                    <ConstantBitrate
		                      Bitrate="1000"
		                      IsTwoPass="False"
		                      BufferWindow="00:00:05" />
		                  </Bitrate>
		                </StreamInfo>
		                <StreamInfo
		                  Size="640, 360">
		                  <Bitrate>
		                    <ConstantBitrate
		                      Bitrate="650"
		                      IsTwoPass="False"
		                      BufferWindow="00:00:05" />
		                  </Bitrate>
		                </StreamInfo>
		                <StreamInfo
		                  Size="320, 180">
		                  <Bitrate>
		                    <ConstantBitrate
		                      Bitrate="400"
		                      IsTwoPass="False"
		                      BufferWindow="00:00:05" />
		                  </Bitrate>
		                </StreamInfo>
		              </Streams>
		            </HighH264VideoProfile>
		          </VideoProfile>
		        </MP4OutputFormat>
		      </OutputFormat>
		    </MediaFile>
		  </Preset>
		  <Preset
		    Version="5.0">
		    <Job />
		    <MediaFile
		      DeinterlaceMode="AutoPixelAdaptive"
		      ResizeQuality="Super"
		      NormalizeAudio="True"
		      AudioGainLevel="1"
		      VideoResizeMode="Stretch">
		      <OutputFormat>
		        <MP4OutputFormat
		          StreamCompatibility="PropagateSourceTimeStamps">
		          <AudioProfile>
		            <AacAudioProfile
		              Codec="AAC"
		              Channels="2"
		              BitsPerSample="16"
		              SamplesPerSecond="48000">
		              <Bitrate>
		                <ConstantBitrate
		                  Bitrate="128"
		                  IsTwoPass="False"
		                  BufferWindow="00:00:00" />
		              </Bitrate>
		            </AacAudioProfile>
		          </AudioProfile>
		        </MP4OutputFormat>
		      </OutputFormat>
		    </MediaFile>
		  </Preset>
		  <Preset
		    Version="5.0">
		    <Job />
		    <MediaFile
		      DeinterlaceMode="AutoPixelAdaptive"
		      ResizeQuality="Super"
		      NormalizeAudio="True"
		      AudioGainLevel="1"
		      VideoResizeMode="Stretch">
		      <OutputFormat>
		        <MP4OutputFormat
		          StreamCompatibility="Standard">
		          <AudioProfile>
		            <AacAudioProfile
		              Codec="AAC"
		              Channels="6"
		              BitsPerSample="16"
		              SamplesPerSecond="48000">
		              <Bitrate>
		                <ConstantBitrate
		                  Bitrate="512"
		                  IsTwoPass="False"
		                  BufferWindow="00:00:00" />
		              </Bitrate>
		            </AacAudioProfile>
		          </AudioProfile>
		        </MP4OutputFormat>
		      </OutputFormat>
		    </MediaFile>
		  </Preset>
		  <Preset
		    Version="5.0">
		    <Job />
		    <MediaFile
		      DeinterlaceMode="AutoPixelAdaptive"
		      ResizeQuality="Super"
		      NormalizeAudio="True"
		      AudioGainLevel="1"
		      VideoResizeMode="Stretch">
		      <OutputFormat>
		        <MP4OutputFormat
		          StreamCompatibility="Standard">
		          <AudioProfile>
		            <DolbyDigitalPlusAudioProfile
		              Codec="DolbyDigitalPlus"
		              EncoderMode="DolbyDigitalPlus"
		              Channels="2"
		              AudioCodingMode="Mode20"
		              LFEOn="False"
		              NinetyDegreePhaseShiftSurrounds="False"
		              ThreeDBAttenuationSurrounds="False"
		              DolbySurroundMode="NotIndicated"
		              StereoDownmixPreference="NotIndicated"
		              LtRtCenterMixLevel="-3"
		              LoRoCenterMixLevel="-3"
		              LtRtSurroundMixLevel="-3"
		              LoRoSurroundMixLevel="-3"
		              LFELowpassFilter="False"
		              SamplesPerSecond="48000"
		              BandwidthLimitingLowpassFilter="True"
		              DCHighpassFilter="True"
		              LineModeDynamicRangeControl="FilmStandard"
		              RFModeDynamicRangeControl="FilmStandard"
		              DialogNormalization="-31">
		              <Bitrate>
		                <ConstantBitrate
		                  Bitrate="128"
		                  IsTwoPass="False"
		                  BufferWindow="00:00:00" />
		              </Bitrate>
		            </DolbyDigitalPlusAudioProfile>
		          </AudioProfile>
		        </MP4OutputFormat>
		      </OutputFormat>
		    </MediaFile>
		  </Preset>
		  <Preset
		    Version="5.0">
		    <Job />
		    <MediaFile
		      DeinterlaceMode="AutoPixelAdaptive"
		      ResizeQuality="Super"
		      NormalizeAudio="True"
		      AudioGainLevel="1"
		      VideoResizeMode="Stretch">
		      <OutputFormat>
		        <MP4OutputFormat
		          StreamCompatibility="Standard">
		          <AudioProfile Condition="SourceContainsAudio">
		            <DolbyDigitalPlusAudioProfile
		              Codec="DolbyDigitalPlus"
		              EncoderMode="DolbyDigitalPlus"
		              Channels="6"
		              AudioCodingMode="Mode32"
		              LFEOn="True"
		              NinetyDegreePhaseShiftSurrounds="True"
		              ThreeDBAttenuationSurrounds="False"
		              DolbySurroundMode="NotIndicated"
		              StereoDownmixPreference="NotIndicated"
		              LtRtCenterMixLevel="-3"
		              LoRoCenterMixLevel="-3"
		              LtRtSurroundMixLevel="-3"
		              LoRoSurroundMixLevel="-3"
		              LFELowpassFilter="True"
		              SamplesPerSecond="48000"
		              BandwidthLimitingLowpassFilter="True"
		              DCHighpassFilter="True"
		              LineModeDynamicRangeControl="FilmStandard"
		              RFModeDynamicRangeControl="FilmStandard"
		              DialogNormalization="-31">
		              <Bitrate>
		                <ConstantBitrate
		                  Bitrate="512"
		                  IsTwoPass="False"
		                  BufferWindow="00:00:00" />
		              </Bitrate>
		            </DolbyDigitalPlusAudioProfile>
		          </AudioProfile>
		        </MP4OutputFormat>
		      </OutputFormat>
		    </MediaFile>
		  </Preset>
		  <Preset
		    Version="5.0">
		    <Job />
		    <MediaFile
		      DeinterlaceMode="AutoPixelAdaptive"
		      ResizeQuality="Super"
		      NormalizeAudio="True"
		      AudioGainLevel="1"
		      VideoResizeMode="Stretch">
		      <OutputFormat>
		        <MP4OutputFormat
		          StreamCompatibility="Standard">
		          <AudioProfile>
		            <AacAudioProfile
		              Codec="AAC"
		              Channels="2"
		              BitsPerSample="16"
		              SamplesPerSecond="48000">
		              <Bitrate>
		                <ConstantBitrate
		                  Bitrate="56"
		                  IsTwoPass="False"
		                  BufferWindow="00:00:00" />
		              </Bitrate>
		            </AacAudioProfile>
		          </AudioProfile>
		        </MP4OutputFormat>
		      </OutputFormat>
		    </MediaFile>
		  </Preset>
		</Presets>

##创建商用编码服务

一些客户可能希望在 Azure 媒体服务的基础上构建商用编码服务。如果要创建此类“拓展”服务，必须确保所有 Dolby Digital Plus 编码参数均可用。请确保公开了 <DolbyDigitalPlusAudioProfile> 标记中的所有参数，并且最终用户可以配置这些参数。若要使这些参数可用，请联系 prolicensingsupport@dolby.com 以获取相关指南。

##使用 Dolby Professional Loudness Metering (DPLM) 支持

Azure 媒体编码器可以使用 DPLM SDK 来测量输入音频中对话的响度，并设置正确的 DialogNormalization 值。只有在音频编码为 Dolby Digital Plus 时才会启用此功能。在预设配置文件中使用 <LoudnessMetering> 元素（即 <DolbyDigitalPlusAudioProfile> 元素的子项）来配置 DPLM。以下示例预设显示了如何配置 DPLM：
	
	<?xml version="1.0" encoding="utf-16"?>
	<Preset
	  Version="5.0">
	  <Job />
	  <MediaFile>
	    <OutputFormat>
	      <MP4OutputFormat
	        StreamCompatibility="Standard">
	    <AudioProfile>
	             <DolbyDigitalPlusAudioProfile
	               Codec="DolbyDigitalPlus"
	               EncoderMode="DolbyDigitalPlus"
	               DolbySurroundMode="NotIndicated"
	               StereoDownmixPreference="NotIndicated"
	               SamplesPerSecond="48000"
	               AudioCodingMode="Mode20"
	               Channels="2"
	               BitsPerSample="24">
	               <LoudnessMetering
	                 Mode= "ITU_R_BS_1770_2_DI"
	                 SpeechThreshold="20"
	                 TruePeakEmphasis="false"
	                 TruePeakDCBlock="false" />
	              <Bitrate>
	                <ConstantBitrate
	                  Bitrate="320"
	                  IsTwoPass="False"
	                  BufferWindow="00:00:00" />
	              </Bitrate>
	     </DolbyDigitalPlusAudioProfile>
	    </AudioProfile>
	      </MP4OutputFormat>
	    </OutputFormat>
	  </MediaFile>
	</Preset>

<LoudnessMetering> 元素只能在 <DolbyDigitalPlusAudioProfile> 元素中指定。如果使用了 <LoudnessMetering> 元素，则不得使用 DialogNormalization 属性。如果同时使用 <LoudnessMetering> 元素和 DialogNormalization 属性，编码器会生成错误。LoudnessMetering 的所有属性都是可选的，编码器将默认使用 Dolby Laboratories, Inc. 建议的值。

以下部分介绍了各个属性。

###Mode 属性

该属性确定响度计量模式。允许值包括：

 
**ITU\_R\_BS\_1770\_2\_DI**（默认值）- 表示 ITU-R BS.1770-2 plus 对话智能

**ITU\_R\_BS\_1770\_1\_DI** - 表示 ITU-R BS.1770-1 plus 对话智能

**ITU\_R\_BS\_1770\_2** - 表示 ITU-R BS.1770-2

**LEQA\_DI** - 表示 Leq(A) plus 对话智能

**注意：**

**EBU R128** 模式可以通过 **ITU\_R\_BS\_1770\_2\_DI** 实现

包括 **Leq(A)** 完全是出于历史原因的考虑，它只能用于特定的旧工作流

**ITU** 最近发布了一条标题为 BS.1770-3 的更新，等同于 BS.1770-2，其中 TruePeakDCBlock 和 TruePeakEmphasis 均设置为 false

###SpeechThreshold 属性

指定 DPLM 用来生成集成响度结果的语音阈值（例如，在语音选通、电平选通和无选通之间进行选择）。语音阈值设置范围是 0% 至 100%（以 1% 为增量）。此参数仅适用于 DPLM 配置为利用对话智能的模式的情况，也就是说，仅当模式设置为 ITU\_R\_BS\_1770\_2\_DI 或 ITU\_R\_BS\_1770\_1\_DI 时才能指定此参数。当模式为 ITU\_R\_BS\_1770\_2\_DI 或 ITU\_R\_BS\_1770\_1\_DI 时，默认值为 20%。此属性的值必须在范围 0、1 – 100 内设置。

###TruePeakDCBlock 属性

此输入参数指定是启用 (true) 还是禁用 (false) 真实峰值计量中的 DC 块。有关 DC 块的详细信息，请参阅 ITU‐R BS.1770‐2。默认值为 false。

###TruePeakEmphasis 属性

指定是启用 (true) 还是禁用 (false) 真实峰值计量中的加重滤波器。有关加重滤波器的详细信息，请参阅 ITU‐R BS.1770‐2。默认值为 false。


###DPLM 结果

如果编码任务指定使用 DPLM，则响度测量结果将包含在输出资产的元数据 XML 中。下面是一个示例。
	
	<LoudnessMeteringResultParameters 
	     DPLMVersionInformation="Dolby Professional Loudness Metering Development Kit Version 1.0"
	     DialogNormalization="-15" 
	     IntegratedLoudness="-14.8487606" 
	     IntegratedLoudnessGatingMethod="2" 
	     IntegratedLoudnessSpeechPercentage="11.673481" 
	     SamplePeak="-0.7028221" 
	     TruePeak="0.705999851" />

下面介绍各个属性。

**DPLMVersionInformation** - 表示所用 DPLM SDK 版本的字符串。

**DialogNormalization** - 从输入音频中测得的 DialNorm 值（单位为分贝），将嵌入到输出 DD+ 流中，范围为 {-31, -30, …, -1} dB。

**IntegratedLoudness** - 由 DPLM 测得的集成响度，范围是 -70 到 +10 LKFS/dBFS（仅当模式设置为 LEQA\_DI 时才使用 dBFS）。

**IntegratedLoudnessGatingMethod** - 有效值为：0 – 无/无选通；1 – 语音选通；2 – 电平选通。

**IntegratedLoudnessSpeechPercentage** - 结果包含检测到语音的输入媒体时间线百分比。值范围为 0% 至 100%。

**SamplePeak** - 此结果包含自重置计量以来任何通道内的最大绝对采样值，范围是 -70 至 +10 dBFS。

**TruePeak** - 此结果包含自重置计量以来任何通道内的最大绝对真实峰值。有关真实峰值的说明，请参阅 ITU‐R BS.1770‐2。值范围可以是 -70 至 12.04 dBTP。

<!---HONumber=74-->