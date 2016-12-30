<properties
    pageTitle="枚举 Azure 导入/导出服务中的作业 | Azure"
    description="了解如何枚举订阅中的所有 Azure 导入/导出服务作业。"
    author="renashahmsft"
    manager="aungoo"
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
    ms.date="12/16/2016"
    wacn.date="12/29/2016"
    ms.author="renash" />  


# 枚举作业
若要枚举某个订阅中的所有作业，请调用[列出作业](https://docs.microsoft.com/zh-CN/rest/api/storageimportexport/jobs#Jobs_List)操作。“`List Jobs`”将返回作业列表及以下属性：

-   作业的类型（导入或导出）

-   当前作业状态

-   作业的关联存储帐户

## 另请参阅
 [使用导入/导出服务 REST API](/documentation/articles/storage-import-export-using-the-rest-api/)

<!---HONumber=Mooncake_1226_2016-->