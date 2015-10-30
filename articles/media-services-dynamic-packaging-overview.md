<properties 
	pageTitle="动态打包概述" 
	description="主题提供动态打包的概述。" 
	authors="Juliako" 
	manager="dwrede" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>

<tags 
	ms.service="media-services" 
	ms.date="09/07/2015"
	wacn.date="10/22/2015"/>


#动态打包 

##概述

Windows Azure 媒体服务可用于向多种客户端技术（例如，iOS、XBOX、Silverlight、Windows 8）传递多种媒体源文件格式、媒体流格式和内容保护格式。这些客户端可识别不同的协议，例如，iOS 需要 HTTP 实时流 (HLS) V4 格式，Silverlight 和 Xbox 需要平滑流。如果你有一组自适应比特率（多码率）MP4（ISO 基媒体 14496-12）文件或平滑流文件要提供给了解 MPEG DASH、HLS 或平滑流的客户端，则应利用媒体服务动态打包。

使用动态打包，你只需要创建一个包含一组自适应比特率 MP4 文件或自适应比特率平滑流文件的资产。然后，点播流服务器会确保你以选定的协议按清单或分段请求中的指定格式接收流。因此，你只需以单一存储格式存储文件并为其付费，然后媒体服务服务就会基于客户端的请求构建并提供相应响应。

下图显示传统编码和静态打包工作流。

![静态编码](./media/media-services-dynamic-packaging-overview/media-services-static-packaging.png)

下图显示了动态打包工作流。

![动态编码](./media/media-services-dynamic-packaging-overview/media-services-dynamic-packaging.png)


>[AZURE.NOTE]若要利用动态打包，首先必须获取你计划从中传送内容的流式处理终结点的至少一个点播流单元。有关详细信息，请参阅[如何缩放媒体服务](/documentation/articles/media-services-manage-origins#scale_streaming_endpoints)。

##常见方案

1. 上载一个输入文件（称为夹层文件）。例如，H.264、MP4 或 WMV（有关受支持格式的列表，请参阅[媒体服务编码器支持的格式](/documentation/articles/media-services-azure-media-encoder-formats)）。

1. 将夹层文件编码为 H.264 MP4 自适应比特率集。
 
1. 通过创建点播定位符来发布包含自适应比特率 MP4 集的资产。
 
1. 生成用于访问和流式传输内容的流 URL。
 
>[AZURE.NOTE]动态打包并非支持所有 MP4 文件格式，有关详细信息，请参阅[动态打包不支持的格式](/documentation/articles/media-services-dynamic-packaging-overview#unsupported_formats)。

##准备用于动态流式传输的资产

若要准备用于动态流式传输的资产，可以使用两个选项：

- 上载主文件并使用 Azure Media Encoder 生成 H.264 MP4 自适应比特率集。
- 使用 Media Packager 上载现有的自适应比特率集并对其进行验证。

###上载主文件并使用 Azure Media Encoder 生成 H.264 MP4 自适应比特率集

有关如何上载和编码资产的信息，请参阅以下文章：


使用 **Azure 管理门户**、**.NET** 或 **REST API** 上载文件。

[AZURE.INCLUDE [media-services-selector-upload-files](../includes/media-services-selector-upload-files.md)]

使用 **Azure 管理门户**、**.NET** 或 **REST API** 通过 **Azure 媒体编码器**进行编码。
 
[AZURE.INCLUDE [media-services-selector-encode](../includes/media-services-selector-encode.md)]


###使用 Media Packager 上载现有的自适应比特率集并对其进行验证

如果你上载的是一组未使用媒体服务编码器编码的自适应比特率 MP4 文件，则通常需要执行此任务。[验证使用外部编码器编码的自适应比特率 MP4](https://msdn.microsoft.com/zh-CN/library/azure/dn750842.aspx) 主题说明了如何完成此任务。

##将内容流式传输到客户端

在获取自适应比特率集后，可以通过创建 On-Demand 定位符发布资产，并编写用于平滑流、MPEG DASH、HLS 和 HDS（仅适用于 Adobe PrimeTime/Access 许可证持有人）的流 URL。

有关如何创建定位符和使用动态打包来流式传输内容的信息，请参阅以下主题：

[将内容传送到客户概述](/documentation/articles/media-services-deliver-content-overview)。

使用 **.NET** 或 **REST API** 配置资源传送策略。

[AZURE.INCLUDE [media-services-selector-asset-delivery-policy](../includes/media-services-selector-asset-delivery-policy.md)]

使用 **Azure 管理门户**或 **.NET** 发布资源（通过创建定位符）。

[AZURE.INCLUDE [media-services-selector-publish](../includes/media-services-selector-publish.md)]


##<a id="unsupported_formats"></a>动态打包不支持的格式

动态打包不支持以下源文件格式。

- Dolby Digital Plus MP4 文件。
- Dolby Digital Plus 平滑流文件。 

<!---HONumber=74-->