<properties
    pageTitle="预览导出作业的驱动器使用情况 | Azure"
    description="了解如何预览针对 Azure 导入/导出服务中的导出作业选择的 Blob 列表"
    author="renashahmsft"
    manager="aungoo"
    editor="tysonn"
    services="storage"
    documentationcenter="" />  

<tags
    ms.assetid="7707d744-7ec7-4de8-ac9b-93a18608dc9a"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/25/2015"
    wacn.date="01/24/2017"
    ms.author="renash" />  


# 预览导出作业的驱动器使用情况
在创建导出作业之前，需要选择一组要导出的 Blob。Azure 导入/导出服务允许使用一系列 Blob 路径或 Blob 前缀来表示选定的 Blob。
  
 接下来，需确定所要发送的驱动器数量。Azure 导入/导出工具提供 `PreviewExport` 命令用于根据所要使用的驱动器大小来预览选定 Blob 的驱动器使用情况。可以指定以下参数：
  
|命令行选项|说明|  
|--------------------------|-----------------|  
|**/logdir:**<LogDirectory>|可选。日志目录。详细日志文件将写入此目录。如果未指定任何日志目录，将使用当前目录作为日志目录。|  
|**/sn:**<StorageAccountName>|必需。导出作业的存储帐户的名称。|  
|**/sk:**<StorageAccountKey>|当且仅当未指定容器 SAS 时才是必需的。导出作业的存储帐户的帐户密钥。|  
|**/csas:**<ContainerSas>|当且仅当未指定存储帐户密钥时才是必需的。用于列出要在导出作业中导出的 Blob 的容器 SAS。|  
|**/ExportBlobListFile:**<ExportBlobListFile>|必需。包含要导出的 Blob 的 Blob 路径列表或 Blob 路径前缀的 XML 文件的路径。导入/导出服务 REST API 的[放置作业](https://docs.microsoft.com/en-us/rest/api/storageimportexport/jobs#Jobs_CreateOrUpdate)操作的 `BlobListBlobPath` 元素中使用的文件格式。|  
|**/DriveSize:**<DriveSize>|必需。用于导出作业的驱动器大小，*例如* 500GB、1.5TB。|  
  
以下示例演示了 `PreviewExport` 命令：
  

	WAImportExport.exe PreviewExport /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /ExportBlobListFile:C:\WAImportExport\mybloblist.xml /DriveSize:500GB    

  
导出 Blob 列表文件可能包含 Blob 名称和 Blob 前缀，如下所示：
  

	<?xml version="1.0" encoding="utf-8"?>  
	<BlobList>  
	<BlobPath>pictures/animals/koala.jpg</BlobPath>  
	<BlobPathPrefix>/vhds/</BlobPathPrefix>  
	<BlobPathPrefix>/movies/</BlobPathPrefix>  
	</BlobList>  


Azure 导入/导出工具可列出要导出的所有 Blob，在考虑所有必要开销的情况下计算如何将其打包到指定大小的驱动器，然后估算保存 Blob 和驱动器使用情况信息所需的驱动器数量。
  
下面是一个省略了信息性日志的输出示例：
  

	Number of unique blob paths/prefixes:   3  
	Number of duplicate blob paths/prefixes:        0  
	Number of nonexistent blob paths/prefixes:      1  
  
	Drive size:     500.00 GB  
	Number of blobs that can be exported:   6  
	Number of blobs that cannot be exported:        2  
	Number of drives needed:        3  
	        Drive #1:       blobs = 1, occupied space = 454.74 GB  
	        Drive #2:       blobs = 3, occupied space = 441.37 GB  
	        Drive #3:       blobs = 2, occupied space = 131.28 GB    
 
  
## 另请参阅  
[Azure 导入/导出工具参考](/documentation/articles/storage-import-export-tool-how-to-v1/)

<!---HONumber=Mooncake_1226_2016-->