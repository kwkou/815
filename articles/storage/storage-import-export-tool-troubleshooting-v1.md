<properties
    pageTitle="排查 Azure 导入/导出工具问题 | Azure"
    description="了解用户在使用导入/导出工具时遇到的常见问题及其解决方法。"
    author="muralikk"
    manager="syadav"
    editor="tysonn"
    services="storage"
    documentationcenter="" />  

<tags
    ms.assetid="b91ca5eb-c557-460a-9afc-0590b38471f9"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/15/2017"
    wacn.date="02/24/2017"
    ms.author="muralikk" />  


# 排查 Azure 导入/导出工具问题
在遇到问题时，Azure 导入/导出工具会返回错误消息。本主题列出用户可能会遇到的一些常见问题。
  
## 复制会话失败该怎么办？  
 如果复制会话失败，可以采用两种做法：
  
 如果该错误可以通过重试来解决，例如，网络共享短期处于脱机状态，而现在已重新联机，则可以恢复复制会话。如果该错误不可以通过重试来解决，例如，在命令行参数中指定了错误的源文件目录，则需要中止复制会话。有关恢复和中止复制会话的详细信息，请参阅[为导入作业准备硬盘驱动器](/documentation/articles/storage-import-export-tool-preparing-hard-drives-import-v1/)。
  
## 无法恢复或中止复制会话。  
 如果该复制会话是驱动器的第一个复制会话，错误消息应会指出：“无法恢复或中止第一个复制会话”。 在此情况下，可以删除旧日记文件，然后重新运行该命令。
  
 如果复制会话不是驱动器的第一个复制会话，则始终可以恢复或中止该会话。
  
## 在丢失日记文件的情况下是否仍可创建作业？  
 驱动器的日记文件包含将数据复制到此驱动器时记录的完整信息，向驱动器添加更多文件以及创建导入作业时需要该日记文件。如果丢失日记文件，需要为驱动器重做所有复制会话。
  
## 另请参阅  
 [设置 Azure 导入/导出工具](/documentation/articles/storage-import-export-tool-setup-v1/)
 [为导入作业准备硬盘驱动器](/documentation/articles/storage-import-export-tool-preparing-hard-drives-import-v1/)
 [使用复制日志文件查看作业状态](/documentation/articles/storage-import-export-tool-reviewing-job-status-v1/)
 [修复导入作业](/documentation/articles/storage-import-export-tool-repairing-an-import-job-v1/)
 [修复导出作业](/documentation/articles/storage-import-export-tool-repairing-an-export-job-v1/)

<!---HONumber=Mooncake_0220_2017-->