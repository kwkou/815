<properties
    pageTitle="使用 Azure 导入/导出服务 REST API | Azure"
    description="了解如何使用 Azure 导入/导出服务 REST API"
    author="renashahmsft"
    manager="aungoo"
    editor="tysonn"
    services="storage"
    documentationcenter="" />  

<tags
    ms.assetid="233f80e9-2e7f-48e0-9639-5c7785e7d743"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/16/2016"
    wacn.date="12/29/2016"
    ms.author="renash" />  


# 使用 Azure 导入/导出服务 REST API

Azure 导入/导出服务公开一个 REST API 来实现对导入/导出作业的编程控制。可通过 [Azure 门户预览](https://portal.azure.cn/)执行的所有导入/导出操作也都可以使用 REST API 来完成。此外，可以使用 REST API 来执行某些精确的操作，例如查询作业完成百分比，而这些操作目前在经典门户中是无法完成的。

有关导入/导出服务的概述以及演示如何使用经典门户创建和管理导入和导出作业的教程，请参阅 [使用 Azure 导入/导出服务将数据传输到 Blob 存储](/documentation/articles/storage-import-export-service/)。

## 服务终结点

Azure 导入/导出服务是 Azure Resource Manager 的资源提供程序，它在以下 HTTPS 终结点上提供一组 REST API 用于管理导入/导出作业：


	https://management.chinacloudapi.cn/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.ImportExport/jobs/<job-name>


## 版本控制

对导入/导出服务发出的请求必须指定 `api-version` 参数并将其值设置为 `2016-11-01`。

## 本节内容

[创建导入作业](/documentation/articles/storage-import-export-creating-an-import-job/)

[创建导出作业](/documentation/articles/storage-import-export-creating-an-export-job/)

[检索作业的状态信息](/documentation/articles/storage-import-export-retrieving-state-info-for-a-job/)

[枚举作业](/documentation/articles/storage-import-export-enumerating-jobs/)

[取消和删除作业](/documentation/articles/storage-import-export-cancelling-and-deleting-jobs/)

[备份驱动器清单](/documentation/articles/storage-import-export-backing-up-drive-manifests/)

[导入/导出作业的诊断和错误恢复](/documentation/articles/storage-import-export-diagnostics-and-error-recovery/)

## 另请参阅
 [存储导入/导出 REST](https://docs.microsoft.com/rest/api/storageimportexport)

<!---HONumber=Mooncake_1226_2016-->