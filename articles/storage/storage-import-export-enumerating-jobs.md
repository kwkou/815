<properties
    pageTitle="列出所有 Azure 导入/导出作业 | Microsoft 文档"
    description="了解如何列出订阅中的所有 Azure 导入/导出服务作业。"
    author="muralikk"
    manager="syadav"
    editor="tysonn"
    services="storage"
    documentationcenter="" />
<tags
    ms.assetid="f2e619be-1bbd-4a54-9472-9e2f70a83b64"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/23/2017"
    wacn.date="03/20/2017"
    ms.author="muralikk" />  



# 枚举 Azure 导入/导出服务中的作业
若要枚举某个订阅中的所有作业，请调用[列出作业](https://docs.microsoft.com/zh-cn/rest/api/storageimportexport/jobs#Jobs_List)操作。“`List Jobs`”将返回作业列表及以下属性：

-   作业的类型（导入或导出）

-   当前作业状态

-   作业的关联存储帐户

## 另请参阅
 [使用导入/导出服务 REST API](/documentation/articles/storage-import-export-using-the-rest-api/)

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description: update page title-->