<properties
    pageTitle="Azure 媒体服务输出元数据架构 | Azure"
    description="本主题概述 Azure 媒体服务输出元数据架构。"
    author="Juliako"
    manager="erikre"
    editor=""
    services="media-services"
    documentationcenter="" />  

<tags
    ms.assetid="1ed84c88-eea5-4a24-9c4f-f2428157d08a"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/26/2016"
    wacn.date="12/12/2016"
    ms.author="juliako" />  


# 输出元数据
## 概述
编码作业与要对其执行编码任务的输入资产关联。例如，将 MP4 文件编码为 H.264 MP4 自适应比特率集；创建缩略图；创建覆盖。完成任务后，会生成输出资产。输出资产包括视频、音频、缩略图等。输出资产还包含提供输出资产相关元数据的文件。元数据 XML 文件的名称采用下列格式：&lt;source\_file\_name&gt;\_manifest.xml（例如，BigBuckBunny\_manifest.xml）。

若要检查元数据文件，可以创建 **SAS** 定位器并将文件下载到本地计算机。

本主题介绍输出元数据 (&lt;source\_file\_name&gt;\_manifest.xml) 以其为基础的 XML 架构的元素和类型。有关包含输入资产的元数据的文件的信息，请参阅[输入元数据](/documentation/articles/media-services-input-metadata-schema/)。

> [AZURE.NOTE]可以在本主题末尾找到完整架构代码 XML 示例。


## <a name="AssetFiles "></a> AssetFiles 根元素
用于编码作业的 AssetFile 条目的集合。

### 子元素
| 名称 | 说明 |
| --- | --- |
| <p>**AssetFile**</p><p> minOccurs="0"</p><p> maxOccurs="1"</p> |AssetFiles 集合中的 [AssetFile e元素](/documentation/articles/media-services-output-metadata-schema/)。 |

## <a name="AssetFile "></a> AssetFile 元素
可以找到 XML 示例 [XML 示例](/documentation/articles/media-services-output-metadata-schema/#xml)。

### 属性
| 名称 | 类型 | 说明 |
| --- | --- | --- |
| <p>**Name**</p><p> 必需</p> |**xs:string** |媒体资产文件名称。 |
| <p>**Size**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:long** |以字节为单位的资产文件大小。 |
| <p>**Duration**</p><p> 必需</p> |**xs:duration** |内容播放持续时间。 |

### 子元素
| 名称 | 说明 |
| --- | --- |
| **源** |输入/源媒体文件的集合，处理该集合以生成此 AssetFile。有关详细信息，请参阅[源元素](/documentation/articles/media-services-output-metadata-schema/)。 |
| <p>**VideoTracks**</p><p> minOccurs="0"</p><p> maxOccurs="1"</p> |每个物理 AssetFile 可以包含零个或以上交错到相应容器格式的视频轨道。这是所有这些视频轨道的集合。有关详细信息，请参阅 [VideoTracks 元素](/documentation/articles/media-services-output-metadata-schema/)。 |
| <p>**AudioTracks**</p><p> minOccurs="0"</p><p> maxOccurs="1"</p> |每个物理 AssetFile 可以包含零个或以上交错到相应容器格式的音频轨道。这是所有这些音频轨道的集合。有关详细信息，请参阅 [AudioTracks 元素](/documentation/articles/media-services-output-metadata-schema/)。 |

## <a name="Sources "></a> Sources 元素
输入/源媒体文件的集合，处理该集合以生成此 AssetFile。

可以找到 XML 示例 [XML 示例](/documentation/articles/media-services-output-metadata-schema/#xml)。

### 子元素
| 名称 | 说明 |
| --- | --- |
| <p>**Source**</p><p> minOccurs="1"</p><p> maxOccurs="unbounded"</p> |生成此资产时所使用的输入/源文件。有关详细信息，请参阅[ 元素](/documentation/articles/media-services-output-metadata-schema/)。 |

## <a name="Source "></a> Source 元素
生成此资产时所使用的输入/源文件。

可以找到 XML 示例 [XML 示例](/documentation/articles/media-services-output-metadata-schema/#xml)。

### 属性
| 名称 | 类型 | 说明 |
| --- | --- | --- |
| <p>**Name**</p><p> 必需</p> |**xs:string** |输入源文件名称。 |

## <a name="VideoTracks "></a> VideoTracks 元素
每个物理 AssetFile 可以包含零个或以上交错到相应容器格式的视频轨道。这是所有这些视频轨道的集合。

可以找到 XML 示例 [XML 示例](/documentation/articles/media-services-output-metadata-schema/#xml)。

### 子元素
| 名称 | 说明 |
| --- | --- |
| <p>**VideoTrack**</p><p> minOccurs="1"</p><p> maxOccurs="unbounded"</p> |父 AssetFile 中的特定视频轨道。有关详细信息，请参阅 [VideoTrack 元素](/documentation/articles/media-services-output-metadata-schema/#VideoTrack)。 |

## <a name="VideoTrack"></a> VideoTrack 元素
父 AssetFile 中的特定视频轨道。

可以找到 XML 示例 [XML 示例](/documentation/articles/media-services-output-metadata-schema/#xml)。

### 属性
| 名称 | 类型 | 说明 |
| --- | --- | --- |
| <p>**Id**</p><p> minInclusive ="0"</p><p> Required</p> |**xs:int** |此视频轨道从零开始的索引。**注意：**这并不一定是 MP4 文件中使用的 TrackID。 |
| <p>**FourCC**</p><p> 必需</p> |**xs:string** |视频编解码器 FourCC 代码。 |
| **配置文件** |**xs:string** |H264 配置文件（仅适用于 H264 编解码器）。 |
| **级别** |**xs:string** |H264 级别（仅适用于 H264 编解码器）。 |
| <p>**Width**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:int** |以像素为单位的编码视频宽度。 |
| <p>**Height**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:int** |以像素为单位的编码视频高度。 |
| <p>**DisplayAspectRatioNumerator**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:double** |视频显示纵横比分子。 |
| <p>**DisplayAspectRatioDenominator**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:double** |视频显示纵横比分母。 |
| <p>**Framerate**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:decimal** |采用 .3f 格式的测量的视频帧速率。 |
| <p>**TargetFramerate**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:decimal** |预设 .3f 格式的目标视频帧速率。 |
| <p>**Bitrate**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:int** |由 AssetFile 计算所得的平均视频比特率（千比特/秒）。仅针对基本流有效负载计数，不包含打包开销。 |
| <p>**TargetBitrate**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:int** |通过编码预设请求的此视频轨道的目标平均比特率（千比特/秒）。 |
| <p>**MaxGOPBitrate**</p><p> minInclusive ="0"</p> |**xs:int** |此视频轨道的最大 GOP 平均比特率（千比特/秒）。 |

## <a name="AudioTracks "></a> AudioTracks 元素
每个物理 AssetFile 可以包含零个或以上交错到相应容器格式的音频轨道。这是所有这些音频轨道的集合。

可以找到 XML 示例 [XML 示例](/documentation/articles/media-services-output-metadata-schema/#xml)。

### 子元素
| 名称 | 说明 |
| --- | --- |
| <p>**AudioTrack**</p><p> minOccurs="1"</p><p> maxOccurs="unbounded"</p> |父 AssetFile 中的特定音频轨道。有关详细信息，请参阅 [AudioTrack 元素](/documentation/articles/media-services-output-metadata-schema/)。 |

## <a name="AudioTrack "></a> AudioTrack 元素
父 AssetFile 中的特定音频轨道。

可以找到 XML 示例 [XML 示例](/documentation/articles/media-services-output-metadata-schema/#xml)。

### 属性
| 名称 | 类型 | 说明 |
| --- | --- | --- |
| <p>**Id**</p><p> minInclusive ="0"</p><p> Required</p> |**xs:int** |此音频轨道从零开始的索引。**注意：**这并不一定是 MP4 文件中使用的 TrackID。 |
| **Codec** |**xs:string** |音频轨道编解码器字符串。 |
| **EncoderVersion** |**xs:string** |可选的编码器版本字符串，对于 EAC3 是必需的。 |
| <p>**Channels**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:int** |音频通道数。 |
| <p>**SamplingRate**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:int** |音频采样率（样本数/秒或 Hz）。 |
| <p>**Bitrate**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:int** |由 AssetFile 计算所得的平均音频比特率（比特/秒）。仅针对基本流有效负载计数，不包含打包开销。 |
| <p>**BitsPerSample**</p><p> minInclusive ="0"</p><p> 必需</p> |**xs:int** |wFormatTag 格式类型的每个样本位数。 |

### 子元素
| 名称 | 说明 |
| --- | --- |
| <p>**LoudnessMeteringResultParameters**</p><p> minOccurs="0"</p><p> maxOccurs="1"</p> |响度测量结果参数。有关详细信息，请参阅 [LoudnessMeteringResultParameters 元素](/documentation/articles/media-services-output-metadata-schema/)。 |

## <a name="LoudnessMeteringResultParameters "></a> LoudnessMeteringResultParameters 元素
响度测量结果参数。

可以找到 XML 示例 [XML 示例](/documentation/articles/media-services-output-metadata-schema/#xml)。

### 属性
| 名称 | 类型 | 说明 |
| --- | --- | --- |
| **DPLMVersionInformation** |**xs:string** |**Dolby** 专业响度测量开发套件版本。 |
| <p>**DialogNormalization**</p><p> minInclusive="-31" maxInclusive="-1"</p><p> 必需</p> |**xs:int** |通过 DPLM 收集的 DialogNormalization，设置 LoudnessMetering 时为必需 |
| <p>**IntegratedLoudness**</p><p> minInclusive="-70" maxInclusive="10"</p><p> 必需</p> |**xs:float** |集成的响度 |
| <p>**IntegratedLoudnessUnit**</p><p> 必需</p> |**xs:string** |集成的响度单位。 |
| <p>**IntegratedLoudnessGatingMethod**</p><p> 必需</p> |**xs:string** |选通标识符 |
| <p>**IntegratedLoudnessSpeechPercentage**</p><p> minInclusive ="0" maxInclusive="100"</p> |**xs:float** |节目的语音内容，以百分比表示。 |
| <p>**SamplePeak**</p><p> 必需</p> |**xs:float** |从重置或上一次清除开始每通道绝对采样峰值。单位为 dBFS。 |
| <p>**SamplePeakUnit**</p><p> fixed="dBFS"</p><p> 必需</p> |**xs:anySimpleType** |采样峰值单位。 |
| <p>**TruePeak**</p><p> 必需</p> |**xs:float** |根据 ITU-R BS.1770-2，从重置或上一次清除开始每通道最大实际峰值。单位为 dBTP。 |
| <p>**TruePeakUnit**</p><p> fixed="dBTP"</p><p> 必需</p> |**xs:anySimpleType** |实际峰值单位。 |

## 架构代码
    <?xml version="1.0" encoding="utf-8"?>  
    <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:msdata="urn:schemas-microsoft-com:xml-msdata" version="1.2"  
               xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata"  
               targetNamespace="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata"  
               elementFormDefault="qualified">  
      <xs:element name="AssetFiles">  
        <xs:annotation>  
          <xs:documentation>Collection of AssetFile entries for the encoding job</xs:documentation>  
        </xs:annotation>  
        <xs:complexType>  
          <xs:sequence>  
            <xs:element name="AssetFile" minOccurs="1" maxOccurs="unbounded">  
              <xs:annotation>  
                <xs:documentation>asset file</xs:documentation>  
              </xs:annotation>  
              <xs:complexType>  
                <xs:sequence>  
                  <xs:element name="Sources">  
                    <xs:annotation>  
                      <xs:documentation>Collection of input/source media files, that was processed in order to produce this AssetFile</xs:documentation>  
                    </xs:annotation>  
                    <xs:complexType>  
                      <xs:sequence>  
                        <xs:element name="Source" minOccurs="1" maxOccurs="unbounded">  
                          <xs:annotation>  
                            <xs:documentation>An input/source file used when generating this asset</xs:documentation>  
                          </xs:annotation>  
                          <xs:complexType>  
                            <xs:attribute name="Name" type="xs:string" use="required">  
                              <xs:annotation>  
                                <xs:documentation>input source file name</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                          </xs:complexType>  
                        </xs:element>  
                      </xs:sequence>  
                    </xs:complexType>  
                  </xs:element>  
                  <xs:element name="VideoTracks" minOccurs="0">  
                    <xs:annotation>  
                      <xs:documentation>Each physical AssetFile can contain in it zero or more video tracks interleaved into an appropriate container format. This is the collection of all those video tracks</xs:documentation>  
                    </xs:annotation>  
                    <xs:complexType>  
                      <xs:sequence>  
                        <xs:element name="VideoTrack" maxOccurs="unbounded">  
                          <xs:annotation>  
                            <xs:documentation>A specific video track in the parent AssetFile</xs:documentation>  
                          </xs:annotation>  
                          <xs:complexType>  
                            <xs:attribute name="Id" use="required">  
                              <xs:annotation>  
                                <xs:documentation>zero-based index of this video track. Note: this is not necessarily the TrackID as used in an MP4 file</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="FourCC" type="xs:string" use="required">  
                              <xs:annotation>  
                                <xs:documentation>video codec FourCC code</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                            <xs:attribute name="Profile" type="xs:string">  
                              <xs:annotation>  
                                <xs:documentation>H264 profile (only appliable for H264 codec)</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                            <xs:attribute name="Level" type="xs:string">  
                              <xs:annotation>  
                                <xs:documentation>H264 level (only appliable for H264 codec)</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                            <xs:attribute name="Width" use="required">  
                              <xs:annotation>  
                                <xs:documentation>encoded video width in pixels</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="Height" use="required">  
                              <xs:annotation>  
                                <xs:documentation>encoded video height in pixels</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="DisplayAspectRatioNumerator" use="required">  
                              <xs:annotation>  
                                <xs:documentation>video display aspect ratio numerator</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:double">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="DisplayAspectRatioDenominator" use="required">  
                              <xs:annotation>  
                                <xs:documentation>video display aspect ratio denominator</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:double">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="Framerate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>measured video frame rate in .3f format</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:decimal">  
                                  <xs:minInclusive value="0"/>  
                                  <xs:fractionDigits value="3"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="TargetFramerate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>preset target video frame rate in .3f format</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:decimal">  
                                  <xs:minInclusive value="0"/>  
                                  <xs:fractionDigits value="3"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="Bitrate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>average video bit rate in kilobits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="TargetBitrate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>target average bitrate for this video track, as requested via the encoding preset, in kilobits per second</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="MaxGOPBitrate">  
                              <xs:annotation>  
                                <xs:documentation>Max GOP average bitrate for this video track, in kilobits per second</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                          </xs:complexType>  
                        </xs:element>  
                      </xs:sequence>  
                    </xs:complexType>  
                  </xs:element>  
                  <xs:element name="AudioTracks" minOccurs="0">  
                    <xs:annotation>  
                      <xs:documentation>each physical AssetFile can contain in it zero or more audio tracks interleaved into an appropriate container format. This is the collection of all those audio tracks</xs:documentation>  
                    </xs:annotation>  
                    <xs:complexType>  
                      <xs:sequence>  
                        <xs:element name="AudioTrack" maxOccurs="unbounded">  
                          <xs:annotation>  
                            <xs:documentation>a specific audio track in the parent AssetFile</xs:documentation>  
                          </xs:annotation>  
                          <xs:complexType>  
                            <xs:sequence>  
                              <xs:element name="LoudnessMeteringResultParameters" minOccurs="0" maxOccurs="1">  
                                <xs:annotation>  
                                  <xs:documentation>Loudness Metering Result Parameters</xs:documentation>  
                                </xs:annotation>  
                                <xs:complexType>  
                                  <xs:attribute name="DPLMVersionInformation" type="xs:string">  
                                    <xs:annotation>  
                                      <xs:documentation>Dolby Professional Loudness Metering Development Kit Version</xs:documentation>  
                                    </xs:annotation>  
                                  </xs:attribute>  
                                  <xs:attribute name="DialogNormalization" use="required">  
                                    <xs:annotation>  
                                      <xs:documentation> DialogNormalization generated through DPLM, required when LoudnessMetering is set</xs:documentation>  
                                    </xs:annotation>  
                                    <xs:simpleType>  
                                      <xs:restriction base="xs:int">  
                                        <xs:minInclusive value="-31"/>  
                                        <xs:maxInclusive value="-1"/>  
                                      </xs:restriction>  
                                    </xs:simpleType>  
                                  </xs:attribute>  
                                  <xs:attribute name="IntegratedLoudness" use="required">  
                                    <xs:annotation>  
                                      <xs:documentation>Integrated loudness</xs:documentation>  
                                    </xs:annotation>  
                                    <xs:simpleType>  
                                      <xs:restriction base="xs:float">  
                                        <xs:minInclusive value="-70"/>  
                                        <xs:maxInclusive value="10"/>  
                                      </xs:restriction>  
                                    </xs:simpleType>  
                                  </xs:attribute>  
                                  <xs:attribute name="IntegratedLoudnessUnit" use="required" type="xs:string">  
                                  </xs:attribute>  
                                  <xs:attribute name="IntegratedLoudnessGatingMethod" use="required" type="xs:string">  
                                    <xs:annotation>  
                                      <xs:documentation>Gating identifier</xs:documentation>  
                                    </xs:annotation>  
                                  </xs:attribute>  
                                  <xs:attribute name="IntegratedLoudnessSpeechPercentage">  
                                    <xs:annotation>  
                                      <xs:documentation>Speech content over the program, as a percentage.</xs:documentation>  
                                    </xs:annotation>  
                                    <xs:simpleType>  
                                      <xs:restriction base="xs:float">  
                                        <xs:minInclusive value="0"/>  
                                        <xs:maxInclusive value="100"/>  
                                      </xs:restriction>  
                                    </xs:simpleType>  
                                  </xs:attribute>  
                                  <xs:attribute name="SamplePeak" use="required" type="xs:float">  
                                    <xs:annotation>  
                                      <xs:documentation>Peak absolute sample value, since reset or since it was last cleared, per channel.  Units are dBFS.</xs:documentation>  
                                    </xs:annotation>  
                                  </xs:attribute>  
                                  <xs:attribute name="SamplePeakUnit" use="required" fixed="dBFS">  
                                  </xs:attribute>  
                                  <xs:attribute name="TruePeak" use="required" type="xs:float">  
                                    <xs:annotation>  
                                      <xs:documentation>Maximum True Peak value, as per ITU-R BS.1770-2, since reset or since it was last cleared, per channel.  Units are dBTP.</xs:documentation>  
                                    </xs:annotation>  
                                  </xs:attribute>  
                                  <xs:attribute name="TruePeakUnit" use="required" fixed="dBTP">  
                                  </xs:attribute>  
                                </xs:complexType>  
                              </xs:element>  
                            </xs:sequence>  
                            <xs:attribute name="Id" use="required">  
                              <xs:annotation>  
                                <xs:documentation>zero-based index of this audio track. Note: this is not necessarily the TrackID as used in an MP4 file</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="Codec" type="xs:string">  
                              <xs:annotation>  
                                <xs:documentation>audio track codec string</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                            <xs:attribute name="EncoderVersion" type="xs:string">  
                              <xs:annotation>  
                                <xs:documentation>optional encoder version string, required for EAC3</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                            <xs:attribute name="Channels" use="required">  
                              <xs:annotation>  
                                <xs:documentation>number of audio channels</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="SamplingRate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>audio sampling rate in samples/sec or Hz</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="Bitrate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>average audio bit rate in bits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="BitsPerSample" use="required">  
                              <xs:annotation>  
                                <xs:documentation>Bits per sample for the wFormatTag format type</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                          </xs:complexType>  
                        </xs:element>  
                      </xs:sequence>  
                    </xs:complexType>  
                  </xs:element>  
                </xs:sequence>  
                <xs:attribute name="Name" type="xs:string" use="required">  
                  <xs:annotation>  
                    <xs:documentation>the media asset file name</xs:documentation>  
                  </xs:annotation>  
                </xs:attribute>  
                <xs:attribute name="Size" use="required">  
                  <xs:annotation>  
                    <xs:documentation>size of file in bytes</xs:documentation>  
                  </xs:annotation>  
                  <xs:simpleType>  
                    <xs:restriction base="xs:long">  
                      <xs:minInclusive value="0"/>  
                    </xs:restriction>  
                  </xs:simpleType>  
                </xs:attribute>  
                <xs:attribute name="Duration" use="required">  
                  <xs:annotation>  
                    <xs:documentation>content play back duration</xs:documentation>  
                  </xs:annotation>  
                  <xs:simpleType>  
                    <xs:restriction base="xs:duration"/>  
                  </xs:simpleType>  
                </xs:attribute>  
              </xs:complexType>  
            </xs:element>  
          </xs:sequence>  
        </xs:complexType>  
      </xs:element>  
    </xs:schema>  



## <a name="xml"></a> XML 示例
 以下是输出元数据文件示例。

    <AssetFiles xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"   
                xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata">  
      <AssetFile Name="BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4" Size="4646283" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.2" Width="1280" Height="720" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="4250" TargetBitrate="3400" MaxGOPBitrate="5514"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4" Size="3166728" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.1" Width="960" Height="540" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="2846" TargetBitrate="2250" MaxGOPBitrate="3630"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4" Size="2205095" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.1" Width="960" Height="540" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="1932" TargetBitrate="1500" MaxGOPBitrate="2513"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4" Size="1508567" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.0" Width="640" Height="360" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="1271" TargetBitrate="1000" MaxGOPBitrate="1527"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4" Size="1057155" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.0" Width="640" Height="360" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="843" TargetBitrate="650" MaxGOPBitrate="1086"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4" Size="699262" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="1.3" Width="320" Height="180" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="503" TargetBitrate="400" MaxGOPBitrate="661"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_AAC_und_ch2_96kbps.mp4" Size="166780" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_AAC_und_ch2_56kbps.mp4" Size="124576" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="53" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
    </AssetFiles>  

<!---HONumber=Mooncake_1205_2016-->