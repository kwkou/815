<properties 
	pageTitle="如何在媒体服务帐户中管理流式处理终结点" 
	description="本主题说明如何使用 Azure 管理门户管理流式处理终结点。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	writer="juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="04/18/2016"
	wacn.date="06/27/2016"/>


#<a id="managemediaservicesorigins"></a>如何在媒体服务帐户中管理流式处理终结点

> [AZURE.SELECTOR]
- [门户](/documentation/articles/media-services-manage-origins/)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)


在 Azure 媒体服务中，**流式处理终结点**表示一个流服务，该服务可以直接将内容传递给客户端播放器应用程序，也可以传递给内容传送网络 (CDN) 以进一步分发。媒体服务还提供无缝 Azure CDN 集成。StreamingEndpoint 服务的出站流可以是实时流，也可以是媒体服务帐户中的视频点播资产。

此外，还可以通过调整扩展单元（也称为流单元）来控制流式处理终结点服务处理不断增长的带宽需求的能力。建议为生产环境中的应用程序分配一个或多个扩展单元。缩放单位为你提供了可按照 200 Mbps 的增量购买的专用出口容量和包括[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)、CDN 集成和高级配置在内的其他功能。

请注意，仅当 StreamingEndpoint 处于运行状态时才进行计费。

本主题概述了流式处理终结点提供的主要功能。本主题还展示如何使用 Azure 管理门户管理流式处理终结点。


##添加和删除流式处理终结点

可以使用 .NET SDK、REST API 或 Azure 管理门户添加或删除流式处理终结点。

若要使用 Azure 管理门户添加/删除流式处理终结点，请执行以下操作：

1. 在 [Azure 管理门户](https://manage.windowsazure.cn/)中单击“媒体服务”。然后，单击媒体服务的名称。
2. 选择“流式处理终结点”页。
3. 单击页底部的“添加”或“删除”按钮。请注意，不能删除默认的流式处理终结点。
4. 单击“启动”按钮以启动流式处理终结点。
5. 单击要配置的流式处理终结点的名称。

![“流式处理终结点”页][streaming-endpoint]


默认情况下可以具有最多两个流式处理终结点。如果需要请求详细信息，请参阅[配额和限制](/documentation/articles/media-services-quotas-and-limitations/)。

##<a name="scale_streaming_endpoints"></a>缩放流式处理终结点

流式处理单元为你提供了可按照 200 Mbps 的增量购买的专用出口容量和其他功能（当前包括[动态打包功能](/documentation/articles/media-services-dynamic-packaging-overview/)）默认情况下，流式处理在共享实例模型中配置，该模型的服务器资源（例如计算机、出口容量等）与所有其他用户共享。若要增加流式处理吞吐量，建议购买流式处理单元。

可以使用 .NET SDK、REST API 或 Azure 管理门户进行缩放。

若要使用门户更改流式处理单元数，请执行以下操作：

1. 若要指定流式处理单元数，请选择“缩放”选项卡并移动**“保留容量”**滑块。

	![“缩放”页](./media/media-services-manage-origins/media-services-origin-scale.png)

4. 按“保存”按钮保存更改。

	分配所有新的流式处理单位大约需要 20 分钟才能完成。

	 
	>[AZURE.NOTE] 当前，将流式处理单位的任何正值设置回“无”可将按需流式处理功能禁用最多 1 小时。

	>[AZURE.NOTE] 为 24 小时期间指定的最大单位数将用于计算成本。有关定价详细信息，请参阅 [媒体服务定价详细信息](/home/features/media-services/pricing/)。
	
##<a name="configure_streaming_endpoints"></a>配置流式处理终结点

当你拥有至少 1 个缩放单位时，通过流式处理终结点可以配置以下属性：

- 访问控制
- 自定义主机名
- 缓存控制
- 跨网站访问策略

有关这些属性的详细信息，请参阅 [StreamingEndpoint](https://msdn.microsoft.com/zh-cn/library/azure/dn783468.aspx)。

可以使用 .NET SDK、REST API 或 Azure 管理门户配置这些属性。

若要使用门户更改流式处理单元数，请执行以下操作：

1. 选择要配置的流式处理终结点。
1. 选择“配置”选项卡。
  
后面提供了字段的简要说明。

![配置来源][configure-origin]
  

1. 设置在 HTTP 响应的缓存控制标头中指定的最长缓存期。此值将不覆盖在 blob 内容上显式设置的最大缓存值。

2. 指定将允许连接到发布的流式处理终结点的 IP 地址。如果未指定 IP 地址，则任何 IP 地址都可以连接。

3. 指定 Akamai 签名标头身份验证的配置。

4. 你可以为 Adobe Flash 客户端指定跨域访问策略（有关详细信息，请参阅[跨域策略文件规范](http://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html)），以及为 Microsoft Silverlight 客户端指定客户端访问策略（有关详细信息，请参阅 [跨域边界提供服务](https://msdn.microsoft.com/zh-cn/library/cc197955(v=vs.95).aspx)。

5. 还可以通过单击“配置”按钮配置自定义主机名。有关详细信息，请参阅 [StreamingEndpont](https://msdn.microsoft.com/zh-cn/library/dn783468.aspx) 主题中的 **CustomHostNames** 属性。


##<a id="enable_cdn"></a>启用 Azure CDN 集成

可以指定为流式处理终结点启用 Azure CDN 集成（默认禁用）。

若要将 Azure CDN 集成设置为 true：

- 流式处理终结点必须具有至少一个流式处理（缩放）单元。如果稍后想要将缩放单位设置为 0，必须首先禁用 CDN 集成。默认情况下，创建新的流式处理终结点时会自动设置一个流式处理单元。

- 流式处理终结点必须处于停止状态。一旦启用 CDN，即可启动流式处理终结点。

启用 Azure CDN 集成可能需要花费最多 90 分钟。在所有 CDN POP 中激活更改将花费最多两个小时。


在所有 Azure 数据中心启用 CDN 集成：中国东部、中国北部。

一旦启用，则将禁用下列配置：**自定义主机名**和**访问控制**。

![流式处理终结点启用 CDN][streaming-endpoint-enable-cdn]


###其他注意事项

- 为流式处理终结点启用 CDN 时，客户端不能从原点直接请求内容。如果需要能够分别使用或不使用 CDN 测试内容，则可以创建另一个不启用 CDN 的流式处理终结点。
- 流式处理终结点主机名在启用 CDN 后仍保持不变。启用 CDN 后，不需要对媒体服务工作流进行任何更改。例如，如果流式处理终结点主机名是 strasbourg.streaming.mediaservices.chinacloudapi.cn，则启用 CDN 后使用完全相同的主机名。
- 对于新的流式处理终结点，只需通过创建新的终结点即可启用 CDN；对于现有流式处理终结点，则需要首先停止该终结点，然后再启用 CDN。
 

有关详细信息，请参阅[通过 Azure CDN（内容传送网络）宣布 Azure 媒体服务集成](http://azure.microsoft.com/zh-cn/blog/2015/03/17/announcing-azure-media-services-integration-with-azure-cdn-content-delivery-network/)。


[streaming-endpoint-enable-cdn]: ./media/media-services-manage-origins/media-services-origins-enable-cdn.png
[streaming-endpoint]: ./media/media-services-manage-origins/media-services-origins-page.png
[configure-origin]: ./media/media-services-manage-origins/media-services-origins-configure.png
[configure-origin-configure-custom-host-names]: ./media/media-services-manage-origins/media-services-configure-custom-host-names.png
 
<!---HONumber=Mooncake_0620_2016-->