<properties 
	pageTitle="将 Telestream Wirecast 编码器配置为发送单比特率实时流" 
	description="本主题说明了如何配置 Wirecast 实时编码器，以便将单比特率流发送到 AMS 频道进行实时编码。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako,cenkdin,anilmur" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="05/03/2016" 
	wacn.date="06/20/2016"/>

#使用 Wirecast 编码器发送单比特率实时流

> [AZURE.SELECTOR]
- [Wirecast](/documentation/articles/media-services-configure-wirecast-live-encoder/)
- [Elemental Live](/documentation/articles/media-services-configure-elemental-live-encoder/)
- [Tricaster](/documentation/articles/media-services-configure-tricaster-live-encoder/)
- [FMLE](/documentation/articles/media-services-configure-fmle-live-encoder/)

本主题说明了如何配置 [Telestream Wirecast](http://www.telestream.net/wirecast/overview.htm) 实时编码器，以便将单比特率流发送到 AMS 频道进行实时编码。有关详细信息，请参阅[使用能够通过 Azure 媒体服务执行实时编码的频道](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)。

本教程演示了如何通过 Azure 媒体服务浏览器 (AMSE) 工具管理 Azure 媒体服务 (AMS)。此工具仅在 Windows 电脑上运行。如果你使用的是 Mac 或 Linux，则可使用 Azure 管理门户创建[频道](/documentation/articles/media-services-portal-creating-live-encoder-enabled-channel/#create-a-channel)和[节目](/documentation/articles/media-services-portal-creating-live-encoder-enabled-channel/#create-and-manage-a-program)。


##先决条件

- [创建 Azure 媒体服务帐户](/documentation/articles/media-services-create-account/)
- 确保在运行流式处理终结点时，至少为其分配了一个流式处理单元。有关详细信息，请参阅[在媒体服务帐户中管理流式处理终结点](/documentation/articles/media-services-manage-origins/)
- 安装最新版本的 [AMSE](https://github.com/Azure/Azure-Media-Services-Explorer) 工具。
- 启动该工具并连接到你的 AMS 帐户。

##提示

- 尽可能使用硬编码的 Internet 连接。
- 在确定带宽要求时，可以认为它就是将流式处理比特率翻倍。虽然此要求不是强制性要求，但它可以减轻网络拥塞的影响。
- 使用基于软件的编码器时，请关闭任何不需要的程序。


## 创建通道

1.  在 AMSE 工具中，导航到“实时”选项卡，然后右键单击频道区域。从菜单中选择“创建频道…”。

![wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast1.png)

2. 指定频道名称，说明字段为可选字段。在“频道设置”下针对“实时编码”选项选择“标准”，将“输入协议”设置为“RTMP”。所有其他设置可保留原样。


确保选中“立即启动新频道”。

3. 单击“创建频道”。
![wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast2.png)

>[AZURE.NOTE] 启动频道可能需要长达 20 分钟的时间。

在启动频道时，你可以[配置编码器](/documentation/articles/media-services-configure-wirecast-live-encoder/#configure_wirecast_rtmp)。

>[AZURE.IMPORTANT] 请注意，只要频道进入就绪状态，就会开始计费。有关详细信息，请参阅[频道的状态](/documentation/articles/media-services-manage-live-encoder-enabled-channels/#states)。

##<a id=configure_wirecast_rtmp></a>配置 Telestream Wirecast 编码器

在本教程中，将使用以下输出设置。本部分的其余内容介绍更详细的配置步骤。

**视频**：
 
- 编解码器：H.264 
- 配置文件：高（等级 4.0） 
- 比特率：5000 kbps 
- 关键帧：2 秒（60 秒） 
- 帧速率：30
 
**音频**：

- 编码解码器：AAC (LC) 
- 比特率：192 kbps 
- 采样速率：44.1 kHz


###配置步骤

1. 在所使用的计算机上打开 Telestream Wirecast 应用程序，然后针对 RTMP 流式处理进行设置。
2. 导航到“输出”选项卡并选择“输出设置…”，以便对输出进行配置。
	
	确保将“输出目标”设置为“RTMP 服务器”。
3. 单击**“确定”**。
4. 在设置页上，将“目标”字段设置为“Azure 媒体服务”。
 
	编码配置文件已预先选择为 **Azure H.264 720p 16:9 (1280x720)**。若要自定义这些设置，请选择下拉菜单右侧的齿轮图标，然后选择“新建预设”。

	![wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast3.png)

5. 配置编码器预设。

	为预设命名，并查看是否存在下列建议的设置：

	**视频**
	
	- 编码器：MainConcept H.264
	- 每秒帧数：30
	- 平均比特率：5000 千位/秒（可根据网络限制进行调整）
	- 配置文件：主
	- 关键帧间隔：60 帧

	**音频：**

	- 目标比特率：192 千位/秒
	- 采样速率：44.100 kHz
	 
	![wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast4.png)

6. 按“保存”。

	“编码”字段现在有新建的配置文件可供选择。

	请确保选择新的配置文件。

7. 获取频道的输入 URL，以便将其分配给 Wirecast 的“RTMP 终结点”。
	
	导航回 AMSE 工具，查看频道完成状态。一旦状态从“正在启动”变为“正在运行”，即可获取输入 URL。
	  
	当频道正在运行时，右键单击频道名称，向下导航，将鼠标悬停在“将输入 URL 复制到剪贴板”上方，然后选择“主要输入 URL”。
	
	![wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast6.png)

8. 在 Wirecast 的“输出设置”窗口中，将此信息粘贴到输出部分的“地址”字段，然后指定一个流名称。


	![wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast5.png)

9. 选择“确定”。

10. 在“Wirecast”主屏幕上，确认视频和音频的输入源已就绪，然后单击左上角的“流”。

	![wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast7.png)

>[AZURE.IMPORTANT] 在单击“流”之前，**必须**确保频道已就绪。
>另外，请确保不要让频道在没有一个输入/贡献源的情况下处于就绪状态的时间超出 15 分钟。

##测试播放
  
1. 导航回 AMSE 工具，然后右键单击要测试的频道。在菜单中，将鼠标悬停在“播放预览”上方，然后选择“使用 Azure Media Player”。  

	![wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast8.png)

如果流出现在播放器中，则编码器已正确配置，可以连接到 AMS。

如果收到错误，则需重置频道并调整编码器设置。请参阅[故障排除](/documentation/articles/media-services-troubleshooting-live-streaming/)主题以获取相关指导。

##创建节目

1. 一旦确认频道可以播放，则可创建节目。在 AMSE 工具的“实时”选项卡下，右键单击节目区域，然后选择“创建新节目”。  

	![wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast9.png)

2. 为节目命名，然后根据需要调整“存档时段长度”（默认为 4 小时）。你还可以指定存储位置，也可以将其保留为默认值。
3. 选中“立即启动节目”框。
4. 单击“创建节目”。  
  
	注意：创建节目需要的时间比创建频道需要的时间少。
 
5. 可以运行节目以后，可通过下述方式来确认其是否能够播放：右键单击该节目，导航到“播放节目”，然后选择“使用 Azure Media Player”。
6. 确认以后，再次右键单击该节目，然后选择“将输出 URL 复制到剪贴板”（也可通过菜单从“节目信息和设置”选项检索此信息）。 

现在可以将流嵌入到播放器中，也可将其分发给受众进行实时观看。


## 故障排除
 
请参阅[故障排除](/documentation/articles/media-services-troubleshooting-live-streaming/)主题以获取相关指导。


<!---HONumber=Mooncake_0613_2016-->