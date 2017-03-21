<properties
    pageTitle="取消/删除 Azure 导入/导出作业 | Azure"
    description="了解如何取消和删除 Microsoft Azure 导入/导出服务的作业"
    author="muralikk"
    manager="syadav"
    editor="tysonn"
    services="storage"
    documentationcenter="" />
<tags
    ms.assetid="fd3d66f0-1dbb-4c75-9223-307d5abaeefc"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/23/2017"
    wacn.date="03/20/2017"
    ms.author="muralikk" />  


# 取消和删除 Azure 导入/导出作业
在作业进入 `Packaging` 状态之前，可以通过调用[更新作业属性](https://docs.microsoft.com/zh-cn/rest/api/storageimportexport/jobs#Jobs_Update)操作并将 `CancelRequested` 元素设置为 `true` 来请求取消该作业。系统会尽最大努力取消该作业。如果驱动器正在传输数据，则即使在请求取消后，数据也仍可能继续传输。

 已取消的作业将转换为 `Completed` 状态并保留 90 天，之后将被删除。

 若要删除某个作业，请在传送该作业之前（*即*，在该作业处于 `Creating` 状态时）调用[删除作业](https://docs.microsoft.com/zh-cn/rest/api/storageimportexport/jobs#Jobs_Delete)操作。也可以在作业处于 `Completed` 状态时将其删除。删除某个作业后，不再能够通过 REST API 或 Azure 门户预览访问其信息和状态。

## 另请参阅
 [使用导入/导出服务 REST API](/documentation/articles/storage-import-export-using-the-rest-api/)

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description: update page title-->