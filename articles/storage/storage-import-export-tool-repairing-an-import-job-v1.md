<properties
    pageTitle="修复导入作业 | Azure"
    description="了解如何使用导入/导出服务修复已创建和运行的导入作业。"
    author="muralikk"
    manager="syadav"
    editor="tysonn"
    services="storage"
    documentationcenter="" />  

<tags
    ms.assetid="38cc16bd-ad55-4625-9a85-e1726c35fd1b"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/15/2017"
    wacn.date="02/24/2017"
    ms.author="muralikk" />  


# 修复导入作业
Azure 导入/导出服务可能无法将某些文件或某个文件的部分内容复制到 Azure Blob 服务。失败的部分原因包括：
  
-   文件已损坏
  
-   驱动器受损
  
-   存储帐户密钥在传输文件时已更改。
  
可以使用导入作业的复制日志文件来运行 Azure 导入/导出工具，将缺失的文件（或某个文件的部分内容）上载到 Azure 存储帐户，从而完成导入作业。
  
用于修复导入作业的命令是 **RepairImport**。可以指定以下参数：
  
|||  
|-|-|  
|**/r:**<RepairFile>|**必需。** 修复文件的路径。该文件用于跟踪修复进度，以及恢复已中断的修复。每个驱动器都必须有且仅有一个修复文件。在开始对某个给定驱动器进行修复时，会传入尚不存在的某个修复文件的路径。若要恢复已中断的修复，应该传入现有修复文件的名称。始终必须指定与目标驱动器对应的修复文件。|  
|**/logdir:**<LogDirectory>|**可选。** 日志目录。详细日志文件将写入此目录。如果未指定任何日志目录，将使用当前目录作为日志目录。|  
|**/d:**<TargetDirectories>|**必需。** 包含已导入的原始文件的一个或多个目录（以分号分隔）。也可以使用导入驱动器，但如果原始文件的备用位置可用，则无需使用该驱动器。|  
|**/bk:**<BitLockerKey>|**可选。** 如果希望工具将原始文件所在的已加密驱动器解锁，应指定 BitLocker 密钥。|  
|**/sn:**<StorageAccountName>|**必需。** 导入作业的存储帐户的名称。|  
|**/sk:**<StorageAccountKey>|当且仅当未指定容器 SAS 时才是**必需**的。导入作业的存储帐户的帐户密钥。|  
|**/csas:**<ContainerSas>|当且仅当未指定存储帐户密钥时才是**必需**的。用于访问与导入作业关联的 Blob 的容器 SAS。|  
|**/CopyLogFile:**<DriveCopyLogFile>|**必需。** 驱动器复制日志文件（详细日志或错误日志）的路径。该文件由 Azure 导入/导出服务生成，可以从与该作业关联的 Blob 存储下载。复制日志文件包含有关要修复的已失败 Blob 或文件的信息。|  
|**/PathMapFile:**<DrivePathMapFile>|**可选。** 可用于解决歧义（如果在同一作业中导入多个同名文件）的文本文件的路径。工具首次运行时，将在此文件中填充所有不明确的名称。工具后续运行时将使用此文件来解决歧义。|  
  
## 使用 RepairImport 命令  
若要通过网络流式传输导入数据来修复这些数据，必须使用 `/d` 参数指定包含导入的原始文件的目录。此外，必须指定从存储帐户下载的复制日志文件。下面显示了一个用于修复部分失败的导入作业的典型命令行：
  

	WAImportExport.exe RepairImport /r:C:\WAImportExport\9WM35C2V.rep /d:C:\Users\bob\Pictures;X:\BobBackup\photos /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C2V.log  

  
下面是复制日志文件的示例。在此示例中，为导入作业寄送的驱动器上某个文件的 64K 内容已损坏。由于这只是指示失败，作业中剩余 Blob 已成功导入。
  

	<?xml version="1.0" encoding="utf-8"?>  
	<DriveLog>  
	 <DriveId>9WM35C2V</DriveId>  
	 <Blob Status=”CompletedWithErrors”>  
	 <BlobPath>pictures/animals/koala.jpg</BlobPath>  
	 <FilePath>\animals\koala.jpg</FilePath>  
	 <Length>163840</Length>  
	 <ImportDisposition Status=”Overwritten”>overwrite</ImportDisposition>  
	 <PageRangeList>  
	  <PageRange Offset=”65536” Length=”65536” Hash=”AA2585F6F6FD01C4AD4256E018240CD4” Status=”Corrupted” />  
	 </PageRangeList>  
	 </Blob>  
	 <Status>CompletedWithErrors</Status>  
	</DriveLog>  

  
将此复制日志传递给 Azure 导入/导出工具后，该工具将尝试通过网络复制缺失的内容来完成此文件的导入。根据上面的示例，该工具将在 `C:\Users\bob\Pictures` 和 `X:\BobBackup\photos` 目录中查找原始文件 `\animals\koala.jpg`。如果文件 `C:\Users\bob\Pictures\animals\koala.jpg` 存在，Azure 导入/导出工具会将缺失的数据部分复制到对应的 Blob `http://bobmediaaccount.blob.core.chinacloudapi.cn/pictures/animals/koala.jpg`。
  
## 使用 RepairImport 时解决冲突  
在某些情况下，可能会出于以下原因之一，工具无法找到或打开所需的文件：找不到该文件、文件不可访问、文件名不明确，或文件内容不再正确。
  
如果工具尝试查找 `\animals\koala.jpg` 并且 `C:\Users\bob\pictures` 和 `X:\BobBackup\photos` 中存在同名的文件，可能会发生歧义错误。也就是说，`C:\Users\bob\pictures\animals\koala.jpg` 和 `X:\BobBackup\photos\animals\koala.jpg` 都位于导入作业驱动器上。
  
使用 `/PathMapFile` 选项可以解决这些错误。可以指定包含工具无法正确识别的文件列表的文件名称。下面是可填充 `9WM35C2V_pathmap.txt` 的示例命令行：
  

	WAImportExport.exe RepairImport /r:C:\WAImportExport\9WM35C2V.rep /d:C:\Users\bob\Pictures;X:\BobBackup\photos /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C2V.log /PathMapFile:C:\WAImportExport\9WM35C2V_pathmap.txt  

  
然后，工具会将有问题的文件路径写入 `9WM35C2V_pathmap.txt`（每行一个路径）。例如，在运行该命令后，该文件可能包含以下条目：
 

	\animals\koala.jpg  
	\animals\kangaroo.jpg  

  
 对于列表中的每个文件，应尝试找到并打开该文件，确保工具可对其进行处理。若要明确告知工具可找到文件的位置，可以修改路径映射文件，并在同一行上添加每个文件的路径（制表符分隔）：
  

	\animals\koala.jpg           C:\Users\bob\Pictures\animals\koala.jpg  
	\animals\kangaroo.jpg        X:\BobBackup\photos\animals\kangaroo.jpg  

  
使工具可以处理所需的文件或者更新路径映射文件后，可返回此工具完成导入过程。
  
## 另请参阅  

[设置 Azure 导入/导出工具](/documentation/articles/storage-import-export-tool-setup-v1/)  
[为导入作业准备硬盘驱动器](/documentation/articles/storage-import-export-tool-preparing-hard-drives-import-v1/)  
[使用复制日志文件查看作业状态](/documentation/articles/storage-import-export-tool-reviewing-job-status-v1/)  
[修复导出作业](/documentation/articles/storage-import-export-tool-repairing-an-export-job-v1/)  
[排查 Azure 导入/导出工具问题](/documentation/articles/storage-import-export-tool-troubleshooting-v1/)

<!---HONumber=Mooncake_0220_2017-->