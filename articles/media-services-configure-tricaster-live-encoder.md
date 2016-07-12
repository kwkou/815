<properties 
	pageTitle="将 NewTek TriCaster 编码器配置为发送单比特率实时流" 
	description="本主题说明了如何配置 Tricaster 实时编码器，以便将单比特率流发送到 AMS 频道进行实时编码。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako,cenkdin,anilmur" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="05/03/2016" 
	wacn.date="06/20/2016"/>

#使用 NewTek TriCaster 编码器发送单比特率实时流

> [AZURE.SELECTOR]
- [Tricaster](/documentation/articles/media-services-configure-tricaster-live-encoder/)
- [Elemental Live](/documentation/articles/media-services-configure-elemental-live-encoder/)
- [Wirecast](/documentation/articles/media-services-configure-wirecast-live-encoder/)
- [FMLE](/documentation/articles/media-services-configure-fmle-live-encoder/)

本主题说明了如何配置 [NewTek TriCaster](http://newtek.com/products/tricaster-40.html) 实时编码器，以便将单比特率流发送到 AMS 频道进行实时编码。有关详细信息，请参阅[使用能够通过 Azure 媒体服务执行实时编码的频道](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)。

本教程演示了如何通过 Azure 媒体服务浏览器 (AMSE) 工具管理 Azure 媒体服务 (AMS)。此工具仅在 Windows 电脑上运行。如果你使用的是 Mac 或 Linux，则可使用 Azure 管理门户创建[频道](/documentation/articles/media-services-portal-creating-live-encoder-enabled-channel/#create-a-channel)和[节目](/documentation/articles/media-services-portal-creating-live-encoder-enabled-channel/#create-and-manage-a-program)。

>[AZURE.NOTE]使用 Tricaster 将贡献源发送到已启用实时编码的 AMS 通道时，如果你使用了 Tricaster 的某些功能（例如，在源之间快速剪切，或者切入/切出静态图像），则你的实时事件可能会出现视频/音频抖动。AMS 团队正在努力解决这些问题，在此之前，不建议你使用这些功能。


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

![tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster1.png)

2. 指定频道名称，说明字段为可选字段。在“频道设置”下针对“实时编码”选项选择“标准”，将“输入协议”设置为“RTMP”。所有其他设置可保留原样。


确保选中“立即启动新频道”。

3. 单击“创建频道”。
![tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster2.png)

>[AZURE.NOTE] 启动频道可能需要长达 20 分钟的时间。


在启动频道时，你可以[配置编码器](/documentation/articles/media-services-configure-tricaster-live-encoder/#configure_tricaster_rtmp)。

>[AZURE.IMPORTANT] 请注意，只要频道进入就绪状态，就会开始计费。有关详细信息，请参阅[频道的状态](/documentation/articles/media-services-manage-live-encoder-enabled-channels/#states)。

##<a id=configure_tricaster_rtmp></a>配置 NewTek TriCaster 编码器

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

1. 根据所用的视频输入源创建一个新的 **NewTek TriCaster** 项目。 
2. 进入该项目以后，找到“流”按钮，单击该按钮旁边的齿轮图标，以便访问流配置菜单。

	![tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster3.png)
3. 菜单打开以后，单击“连接”标题下的“新建”。当系统提示你输入连接类型时，请选择“Adobe Flash”。

	![tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster4.png)

4. 单击**“确定”**。

5. 现在，你可以单击“流式处理配置文件”下的下拉箭头并导航到“浏览”，以便导入 FMLE 配置文件。

	![tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster5.png)

6. 导航到保存已配置的 FMLE 配置文件的位置。
7. 选择该文件，然后按“确定”。

	上载该配置文件以后，即可继续执行下一步。

6. 获取频道的输入 URL，以便将其分配给 Tricaster 的“RTMP 终结点”。
	
	导航回 AMSE 工具，查看频道完成状态。一旦状态从“正在启动”变为“正在运行”，即可获取输入 URL。
	  
	当频道正在运行时，右键单击频道名称，向下导航，将鼠标悬停在“将输入 URL 复制到剪贴板”上方，然后选择“主要输入 URL”。
	
	![tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster6.png)

7. 在 Tricaster 项目中，将此信息粘贴到“闪存服务器”下的“位置”字段。另请在“流 ID”字段中指定一个流名称。

	如果流信息已添加到 FMLE 配置文件中，则也可通过以下方式将其导入此部分：单击“导入设置”，导航到已保存的 FMLE 配置文件，然后单击“确定”。相关的“闪存服务器”字段应使用 FMLE 中的信息进行填充。

	![tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster7.png)

9. 完成后，单击屏幕底部的“确定”。当输入到 Tricaster 中的视频和音频已就绪时，则可单击“流”按钮开始将其流式传输到 AMS。

	![tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster11.png)

>[AZURE.IMPORTANT] 在单击“流”之前，**必须**确保频道已就绪。 
>另外，请确保不要让频道在没有一个输入/贡献源的情况下处于就绪状态的时间超出 15 分钟。

##测试播放
  
1. 导航回 AMSE 工具，然后右键单击要测试的频道。在菜单中，将鼠标悬停在“播放预览”上方，然后选择“使用 Azure Media Player”。  

	![tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster8.png)

如果流出现在播放器中，则编码器已正确配置，可以连接到 AMS。

如果收到错误，则需重置频道并调整编码器设置。请参阅[故障排除](/documentation/articles/media-services-troubleshooting-live-streaming/)主题以获取相关指导。

##创建节目

1. 一旦确认频道可以播放，则可创建节目。在 AMSE 工具的“实时”选项卡下，右键单击节目区域，然后选择“创建新节目”。  

	![tricaster](./media/media-services-tricaster-live-encoder/media-services-tricaster9.png)

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