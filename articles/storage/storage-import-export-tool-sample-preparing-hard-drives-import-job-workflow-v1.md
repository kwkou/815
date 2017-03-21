<properties
    pageTitle="为 Azure 导入作业准备硬盘驱动器的示例工作流 | Azure"
    description="参阅为 Azure 导入/导出服务中的导入作业准备驱动器的整个过程演练"
    author="muralikk"
    manager="syadav"
    editor="tysonn"
    services="storage"
    documentationcenter="" />
<tags
    ms.assetid="6eb1b1b7-c69f-4365-b5ef-3cd5e05eb72a"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/23/2017"
    wacn.date="03/20/2017"
    ms.author="muralikk" />  


# 为导入作业准备硬盘驱动器的示例工作流
本主题讲解如何完成为导入作业准备驱动器的整个过程。
  
本示例将以下数据导入到名为 `mystorageaccount` 的 Azure 存储帐户：
  
|位置|说明|  
|--------------|-----------------|  
|H:\\Video|视频集合，总共 5 TB。|  
|H:\\Photo|照片集合，总共 30 GB。|  
|K:\\Temp\\FavoriteMovie.ISO|Blu-Ray™ 磁盘映像，25 GB。|  
|\\\bigshare\\john\\music|网络共享上的音乐文件集合，总共 10 GB。|  
  
导入作业将这些数据导入到存储帐户中的以下目标：
  
|源|目标虚拟目录或 Blob|  
|------------|-------------------------------------------|  
|H:\\Video|https://mystorageaccount.blob.core.chinacloudapi.cn/video|
|H:\\Photo|https://mystorageaccount.blob.core.chinacloudapi.cn/photo|
|K:\\Temp\\FavoriteMovie.ISO|https://mystorageaccount.blob.core.chinacloudapi.cn/favorite/FavoriteMovies.ISO|
|\\\bigshare\\john\\music|https://mystorageaccount.blob.core.chinacloudapi.cn/music|  
  
  
借助此映射，可将文件 `H:\Video\Drama\GreatMovie.mov` 导入 Blob `https://mystorageaccount.blob.core.chinacloudapi.cn/video/Drama/GreatMovie.mov`。
  
接下来，为了确定所需的硬盘驱动器数目，应计算数据大小：
  
`5TB + 30GB + 25GB + 10GB = 5TB + 65GB`  

  
对于本示例，两个 3TB 的硬盘驱动器应该足够。但是，由于源目录 `H:\Video` 包含 5TB 数据，而单个硬盘驱动器的容量仅为 3TB，因此在运行 Microsoft Azure 导入/导出工具之前，需要将 `H:\Video` 分解为两个小目录：`H:\Video1` 和 `H:\Video2`。此步骤将生成以下源目录：
  
|位置|大小|目标虚拟目录或 Blob|  
|--------------|----------|-------------------------------------------|  
|H:\\Video1|2\.5TB|https://mystorageaccount.blob.core.chinacloudapi.cn/video|
|H:\\Video2|2\.5TB|https://mystorageaccount.blob.core.chinacloudapi.cn/video|
|H:\\Photo|30GB|https://mystorageaccount.blob.core.chinacloudapi.cn/photo|
|K:\\Temp\\FavoriteMovies.ISO|25GB|https://mystorageaccount.blob.core.chinacloudapi.cn/favorite/FavoriteMovies.ISO|
|\\\bigshare\\john\\music|10GB|https://mystorageaccount.blob.core.chinacloudapi.cn/music|  
  
  
 请注意，即使已将 `H:\Video` 目录分解为两个目录，它们也会指向存储帐户中的同一目标虚拟目录。这样，所有视频文件将保留在存储帐户中的单个 `video` 容器下。
  
 接下来，将上面的源目录均匀分配到两个硬盘驱动器中：
  
||||  
|-|-|-|  
|硬盘驱动器|源目录|总大小|  
|第一个驱动器|H:\\Video1|2\.5TB + 30GB|  
||H:\\Photo||  
|第二个驱动器|H:\\Video2|2\.5TB + 35GB|  
||K:\\Temp\\BlueRay.ISO||  
||\\\bigshare\\john\\music||  
  
此外，可为所有文件设置以下元数据：
  
-   **UploadMethod：**Microsoft Azure 导入/导出服务
  
-   **DataSetName：**SampleData
  
-   **CreationDate：**10/1/2013
  
若要为导入的文件设置元数据，请创建包含以下内容的文本文件 `c:\WAImportExport\SampleMetadata.txt`：
  

	<?xml version="1.0" encoding="UTF-8"?>  
	<Metadata>  
	    <UploadMethod>Microsoft Azure Import/Export Service</UploadMethod>  
	    <DataSetName>SampleData</DataSetName>  
	    <CreationDate>10/1/2013</CreationDate>  
	</Metadata>  

  
还可为 `FavoriteMovie.ISO` Blob 设置一些属性：
  
-   **Content-Type：**application/octet-stream
  
-   **Content-MD5：**Q2hlY2sgSW50ZWdyaXR5IQ==
  
-   **Cache-Control：**no-cache
  
若要设置这些属性，请创建文本文件 `c:\WAImportExport\SampleProperties.txt`：
  

	<?xml version="1.0" encoding="UTF-8"?>  
	<Properties>  
	    <Content-Type>application/octet-stream</Content-Type>  
	    <Content-MD5>Q2hlY2sgSW50ZWdyaXR5IQ==</Content-MD5>  
	    <Cache-Control>no-cache</Cache-Control>  
	</Properties>  

  
现在，便可以运行 Azure 导入/导出工具来准备两个硬盘驱动器了。请注意：
  
-   第一个驱动器将作为驱动器 X 装入。
  
-   第二个驱动器将作为驱动器 Y 装入。
  
-   存储帐户 `mystorageaccount` 的密钥为 `8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg==`。

## 在预先加载了数据的情况下为导入准备磁盘
 
 如果要导入的数据已在磁盘上，可以使用标志 /skipwrite。/t 和 /srcdir 的值都应该指向要为导入准备的磁盘。如果磁盘上并非所有数据都需要传送到同一个目标虚拟目录或存储帐户根目录，则可针对每个目录单独运行相同的命令，但每次运行需要保持使用相同的 /id 值。

>[AZURE.NOTE] 请不要指定 /format，否则会擦除磁盘上的数据。可以根据磁盘是否已加密，指定 /encrypt 或 /bk。


如果数据已在磁盘上，请针对每个驱动器运行以下命令。
    
    WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:Video1 /logdir:c:\logs /sk:8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg== /t:x /format /encrypt /srcdir:x:\Video1 /dstdir:video/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt /skipwrite


对于第一个驱动器，请运行 Azure 导入/导出工具两次来复制两个源目录：
  

## 第一个驱动器的第一个复制会话  
	
	WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:Video1 /logdir:c:\logs /sk:8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg== /t:x /format /encrypt /srcdir:H:\Video1 /dstdir:video/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt  



## 第一个驱动器的第二个复制会话  

	WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:Photo /srcdir:H:\Photo /dstdir:photo/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt

  
对于第二个驱动器，请运行 Azure 导入/导出工具三次（针对源目录运行两次，针对独立的 Blu-Ray™ 映像文件运行一次）：
  

## 第一个复制会话  
	WAImportExport.exe PrepImport /j:SecondDrive.jrn /id:Video2 /logdir:c:\logs /sk:8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg== /t:y /format /encrypt /srcdir:H:\Video2 /dstdir:video/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt  

  

## 第二个复制会话  
	WAImportExport.exe PrepImport /j:SecondDrive.jrn /id:Music /srcdir:\\bigshare\john\music /dstdir:music/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt  

  

## 第三个复制会话  
	WAImportExport.exe PrepImport /j:SecondDrive.jrn /id:BlueRayIso /srcfile:K:\Temp\BlueRay.ISO /dstblob:favorite/BlueRay.ISO /MetadataFile:c:\WAImportExport\SampleMetadata.txt /PropertyFile:c:\WAImportExport\SampleProperties.txt  

  
复制会话完成后，可断开两个驱动器与复制计算机的连接，然后将其寄送到相应的 Microsoft Azure 数据中心。在 [Azure 管理门户](https://manage.windowsazure.CN/)中创建导入作业时，将上载两个日记文件：`FirstDrive.jrn` 和 `SecondDrive.jrn`。
  
## 另请参阅  

- [为导入作业准备硬盘驱动器](/documentation/articles/storage-import-export-tool-preparing-hard-drives-import-v1/)  
- [常用命令快速参考](/documentation/articles/storage-import-export-tool-quick-reference-v1/)

<!---HONumber=Mooncake_0313_2017-->