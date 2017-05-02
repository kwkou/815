<properties
    pageTitle="为 Azure 导入/导出服务的导入作业准备硬盘驱动器的示例工作流 | Azure"
    description="请参阅为 Azure 导入/导出服务中的导入作业准备驱动器的完整过程演练。"
    author="muralikk"
    manager="syadav"
    editor="tysonn"
    services="storage"
    documentationcenter=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/07/2017"
    wacn.date="05/02/2017"
    ms.author="muralikk"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="5fc5c2ac955ab3a6b780ce13df618700c04d9837"
    ms.lasthandoff="04/22/2017" />

# <a name="sample-workflow-to-prepare-hard-drives-for-an-import-job"></a>为导入作业准备硬盘驱动器的示例工作流

本文讲解如何完成为导入作业准备驱动器的整个过程。

## <a name="sample-data"></a>样本数据

本示例将以下数据导入到名为 `mystorageaccount`的 Azure 存储帐户：

|位置|说明|数据大小|
|--------------|-----------------|-----|
|H:\\Video|视频集合|12 TB|
|H:\\Photo|照片集合|30 GB|
|K:\\Temp\\FavoriteMovie.ISO|Blu-Ray™ 磁盘映像|25 GB|
|\\\bigshare\\john\\music|网络共享上的音乐文件集合|10 GB|

## <a name="storage-account-destinations"></a>存储帐户目标

导入作业将这些数据导入到存储帐户中的以下目标：

|源|目标虚拟目录或 Blob|
|------------|-------------------------------------------|
|H:\Video\ |video/|
|H:\Photo\ |photo/|
|K:\Temp\FavoriteMovie.ISO|favorite/FavoriteMovies.ISO|
|\\\bigshare\john\music\ |music|

借助此映射，可将文件 `H:\Video\Drama\GreatMovie.mov` 导入 Blob `https://mystorageaccount.blob.core.chinacloudapi.cn/video/Drama/GreatMovie.mov`。

## <a name="determine-hard-drive-requirements"></a>确定硬盘驱动器要求

接下来，为了确定所需的硬盘驱动器数目，应计算数据大小：

`12TB + 30GB + 25GB + 10GB = 12TB + 65GB`

对于本示例，两个 8TB 的硬盘驱动器应该足够。 但是，源目录 `H:\Video` 包含 12TB 数据，而单个硬盘驱动器的容量仅为 8TB。在这种情况下，可按以下方式在 **driveset.csv** 文件中指定设置：

    DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
    X,Format,SilentMode,Encrypt,
    Y,Format,SilentMode,Encrypt,

工具将以优化的方式在两个硬盘驱动器之间分发数据。

## <a name="attach-drives-and-configure-the-job"></a>附加驱动器并配置作业
需要将两个磁盘附加到计算机并创建卷。 然后编写 **dataset.csv** 文件：

    BasePath,DstBlobPathOrPrefix,BlobType,Disposition,MetadataFile,PropertiesFile
    H:\Video\,video/,BlockBlob,rename,None,H:\mydirectory\properties.xml
    H:\Photo\,photo/,BlockBlob,rename,None,H:\mydirectory\properties.xml
    K:\Temp\FavoriteVideo.ISO,favorite/FavoriteVideo.ISO,BlockBlob,rename,None,H:\mydirectory\properties.xml
    \\myshare\john\music\,music/,BlockBlob,rename,None,H:\mydirectory\properties.xml

此外，可为所有文件设置以下元数据：

* **UploadMethod：**Azure 导入/导出服务
* **DataSetName：**SampleData
* **CreationDate：** 10/1/2013

若要为导入的文件设置元数据，请创建包含以下内容的文本文件 `c:\WAImportExport\SampleMetadata.txt`：


	<?xml version="1.0" encoding="UTF-8"?>
	<Metadata>
	    <UploadMethod>Azure Import/Export service</UploadMethod>
	    <DataSetName>SampleData</DataSetName>
	    <CreationDate>10/1/2013</CreationDate>
	</Metadata>

还可为 `FavoriteMovie.ISO` Blob 设置一些属性：

* **Content-Type：**application/octet-stream
* **Content-MD5：**Q2hlY2sgSW50ZWdyaXR5IQ==
* **Cache-Control：**no-cache

若要设置这些属性，请创建文本文件 `c:\WAImportExport\SampleProperties.txt`：


	<?xml version="1.0" encoding="UTF-8"?>
	<Properties>
	    <Content-Type>application/octet-stream</Content-Type>
	    <Content-MD5>Q2hlY2sgSW50ZWdyaXR5IQ==</Content-MD5>
	    <Cache-Control>no-cache</Cache-Control>
	</Properties>

## <a name="run-the-azure-importexport-tool-waimportexportexe"></a>运行 Azure 导入/导出工具 (WAImportExport.exe)

现在，便可以运行 Azure 导入/导出工具来准备两个硬盘驱动器了。

**对于第一个会话：**


	WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1  /sk:************* /InitialDriveSet:driveset-1.csv /DataSet:dataset-1.csv /logdir:F:\logs


如果需要添加更多数据，请创建另一个数据集文件（格式与 Initialdataset 相同）。

**对于第二个会话：**


	WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2  /DataSet:dataset-2.csv


复制会话完成后，可断开两个驱动器与复制计算机的连接，然后将其寄送到相应的 Azure 数据中心。在 Azure 门户中创建导入作业时，将上载两个日记文件：`<FirstDriveSerialNumber>.xml` 和 `<SecondDriveSerialNumber>.xml`。

## <a name="next-steps"></a>后续步骤

* [为导入作业准备硬盘驱动器](/documentation/articles/storage-import-export-tool-preparing-hard-drives-import/)
* [常用命令快速参考](/documentation/articles/storage-import-export-tool-quick-reference/)
<!--Update_Description:update source target mapping table;add anchors to sub titles-->