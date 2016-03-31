<properties 
	pageTitle="处理 Azure 媒体服务作业" 
	description="本主题概述如何管理 Azure 媒体服务作业。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="02/03/2016"  
	wacn.date="03/21/2016"/>

#处理 Azure 媒体服务作业

**作业**包含有关将要执行的处理的元数据。每个**作业**包含一个或多个**任务**，这些任务指定一个原子处理任务、该任务的输入资产和输出资产、一个媒体处理器及其关联的设置。有关编码器设置的详细信息，请参阅“编码器指南”和“编码器架构”。

编码作业通常与其他处理相结合，例如打包或加密编码器生成的资产。作业中的各个任务可连接在一起，其中一个任务的输出资产指定为下一任务的输入资产。因此，一个作业可以包含播放媒体所必需的全部处理过程。

本部分提供了当你处理作业\\任务时要执行的常见任务的链接。

>[AZURE.NOTE]每个作业当前有 30 个任务的限制。如果需要链接超过 30 个的任务，请创建多个作业以包含任务。


##获取媒体处理器

使用 **.NET** 或 **REST API** 获取媒体处理器。

<div class="technical-azure-selector">
<a href="/documentation/articles/media-services-get-media-processor">.NET</a>
<a href="/documentation/articles/media-services-rest-get-media-processor">REST API</a>
</div>
<!---HONumber=67-->

##创建作业

作业是包含有关一组任务（例如，编码或索引编制）的元数据的实体。每个任务对输入资产执行原子操作。有关如何创建编码作业的示例，请参阅：

<div class="technical-azure-selector">
<a href="/documentation/articles/media-services-manage-content#encode">Portal</a>
<a href="/documentation/articles/media-services-dotnet-encode-asset">.NET</a>
<a href="/documentation/articles/media-services-rest-encode-asset">REST API</a>
</div>
<!---HONumber=67-->

##索引

<div class="technical-azure-selector">
<a href="/documentation/articles/media-services-manage-content">Portal</a>
<a href="/documentation/articles/media-services-index-content">.NET</a>
</div>
<!---HONumber=67-->

##编码

使用 **Azure 管理门户**、**.NET** 或 **REST API** 通过 **Azure 媒体编码器**进行编码。

<div class="technical-azure-selector">
<a href="/documentation/articles/media-services-manage-content#encode">Portal</a>
<a href="/documentation/articles/media-services-dotnet-encode-asset">.NET</a>
<a href="/documentation/articles/media-services-rest-encode-asset">REST API</a>
</div>
<!---HONumber=67-->

##监视作业进度

使用 **Azure 管理门户**、**.NET** 或 **REST API** 监视作业进度。

<div class="technical-azure-selector">
<a href="/documentation/articles/media-services-portal-check-job-progress">Portal</a>
<a href="/documentation/articles/media-services-check-job-progress">.NET</a>
<a href="/documentation/articles/media-services-rest-check-job-progress">REST API</a>
</div>
<!---HONumber=67-->

##列出 

<div class="technical-azure-selector">
<a href="/documentation/articles/media-services-dotnet-manage-entities#list-jobs-and-assets">.NET</a>
<a href="/documentation/articles/media-services-rest-manage-entities#querying-entities">REST</a>
</div>
##删除作业

<div class="technical-azure-selector">
<a href="/documentation/articles/media-services-dotnet-manage-entities#delete-a-job">.NET</a>
<a href="/documentation/articles/media-services-rest-manage-entities##deleting-entities">REST</a>
</div>
##相关链接

[配额与限制](/documentation/articles/media-services-quotas-and-limitations) - 介绍媒体服务编码器的使用配额和限制

<!---HONumber=Mooncake_0314_2016-->