<properties
    pageTitle="查看 Azure 导入/导出作业状态 | Azure"
    description="了解如何使用运行导入或导出作业时创建的日志文件查看导入/导出作业的状态。"
    author="muralikk"
    manager="syadav"
    editor="tysonn"
    services="storage"
    documentationcenter="" />
<tags
    ms.assetid="c69d1d69-6403-4eee-9949-0185faeecfa1"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/26/2017"
    wacn.date="03/20/2017"
    ms.author="muralikk" />  


# 使用复制日志文件查看 Azure 导入/导出作业状态
Azure 导入/导出服务在处理与某个导入或导出作业关联的驱动器时，会将日志文件写入到要在其中导入或导出 Blob 的存储帐户。该日志文件包含与已导入或导出的每个文件相关的详细状态。查询某个已完成作业的状态时，将返回每个复制日志文件的 URL；有关详细信息，请参阅 [Get Job](https://docs.microsoft.com/en-us/rest/api/storageimportexport/jobs#Jobs_Get)（获取作业）。
  
 下面是包含两个驱动器的导入作业的复制日志文件的示例 URL：
  
 `http://myaccount.blob.core.chinacloudapi.cn/ImportExportStatesPath/waies/myjob_9WM35C2V_20130921-034307-902_error.xml`  

  
 `http://myaccount.blob.core.chinacloudapi.cn/ImportExportStatesPath /waies/myjob_9WM45A6Q_20130921-042122-021_error.xml`  

  
 有关复制日志的格式以及状态代码的完整列表，请参阅[导入/导出服务日志文件格式](/documentation/articles/storage-import-export-file-format-log/)。
  
## 另请参阅  
 [设置 Azure 导入/导出工具](/documentation/articles/storage-import-export-tool-setup-v1/)
 [为导入作业准备硬盘驱动器](/documentation/articles/storage-import-export-tool-preparing-hard-drives-import-v1/)
 [修复导入作业](/documentation/articles/storage-import-export-tool-repairing-an-import-job-v1/)
 [修复导出作业](/documentation/articles/storage-import-export-tool-repairing-an-export-job-v1/)
 [排查 Azure 导入/导出工具问题](/documentation/articles/storage-import-export-tool-troubleshooting-v1/)

<!---HONumber=Mooncake_0313_2017-->