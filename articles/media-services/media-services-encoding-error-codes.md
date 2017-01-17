<properties
    pageTitle="编码错误代码 | Azure"
    description="本主题列出了在执行编码任务期间发生错误时可能返回的错误代码。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="ce4e939f-5aee-41f9-859d-e4429815e9f2"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="09/19/2016"
    wacn.date="01/13/2017"
    ms.author="juliako" />  


## 编码错误代码

下表列出了在执行编码任务期间发生错误的情况下可能返回的错误代码。若要获取 .NET 代码中的错误详细信息，请使用 [ErrorDetails](http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.mediaservices.client.errordetail.aspx) 类。若要获取 REST 代码中的错误详细信息，请使用 [ErrorDetail](https://docs.microsoft.com/zh-cn/rest/api/media/operations/errordetail) REST API。

| ErrorDetail.Code | 出错的可能原因 |
| --- | --- |
| Unknown |执行任务期间发生未知的错误 |
| ErrorDownloadingInputAssetMalformedContent |涵盖下载输入资产时出错（例如，错误的文件名称、文件长度为零、格式不正确，等等）的错误类别。 |
| ErrorDownloadingInputAssetServiceFailure |涵盖服务端问题（例如，下载时发生网络或存储错误）的错误类别。 |
| ErrorParsingConfiguration |任务 \<see cref="MediaTask.PrivateData"/\>（配置）无效的错误类别，例如，配置不是有效的系统预设或包含无效的 XML。 |
| ErrorExecutingTaskMalformedContent |在执行任务期间因输入媒体文件内部问题导致失败的错误类别。 |
| ErrorExecutingTaskUnsupportedFormat |媒体处理器无法处理提供的文件（不支持的媒体格式或与配置不匹配）的错误类别。例如，尝试从只包含视频的资产生成只包含音频的输出 |
| ErrorProcessingTask |媒体处理器在处理与内容无关的任务时发生的其他错误类别。 |
| ErrorUploadingOutputAsset |上载输出资产时的错误类别 |
| ErrorCancelingTask |涵盖尝试取消任务时失败的错误类别 |
| TransientError |涵盖暂时性问题（例如 Azure 存储空间发生暂时性网络问题）的错误类别 |



## 相关文章
* [通过自定义媒体编码器标准预设执行高级编码任务](/documentation/articles/media-services-custom-mes-presets-with-dotnet/)
* [配额和限制](/documentation/articles/media-services-quotas-and-limitations/)

<!--Reference links in article-->

[1]: /pricing/details/media-services/

<!---HONumber=Mooncake_0109_2017-->