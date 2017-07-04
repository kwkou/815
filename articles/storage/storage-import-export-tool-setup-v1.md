<properties
    pageTitle="设置 Azure 导入/导出工具 | Azure"
    description="了解如何设置 Azure 导入导出工具的驱动器准备和修复工具"
    author="muralikk"
    manager="syadav"
    editor="tysonn"
    services="storage"
    documentationcenter="" />  

<tags
    ms.assetid="c312b1ab-5b9e-4d24-becd-790a88b3ba8d"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/15/2017"
    wacn.date="02/24/2017"
    ms.author="muralikk" />  


# 设置 Azure 导入/导出工具
Azure 导入/导出工具是可与 Azure 导入/导出服务一起使用的驱动器准备和修复工具。可以使用该工具实现以下功能：
  
-   在创建导入作业之前，可以使用此工具将数据复制到要寄送给 Azure 数据中心的硬盘驱动器。
  
-   完成某个导入作业后，可以使用此工具修复已损坏、丢失或与其他 Blob 冲突的任何 Blob。
  
-   通过某个已完成的导出作业收到驱动器后，可以使用此工具修复这些驱动器上已损坏或丢失的任何文件。
  
## 先决条件  
若要为导出作业准备驱动器，需要满足以下先决条件：
  
-   必须拥有一个有效的 Azure 订阅。
  
-   该订阅必须包含一个存储帐户，其中有足够的可用空间可存储所要导入的文件。
  
-   需要存储帐户的至少一个帐户密钥。
  
-   需要准备一台装有 Windows 7、Windows Server 2008 R2 或更高版本的 Windows 操作系统的计算机（“复制计算机”）。
  
-   必须在复制计算机上安装 .NET Framework 4。
  
-   必须在复制计算机上启用 BitLocker。
  
-   需要一个或多个包含所要导入数据的驱动器，或者与复制计算机连接的空 3.5 英寸 SATA 硬盘驱动器。
  
-   打算导入的文件必须可从复制计算机访问，无论这些文件是位于网络共享还是本地硬盘驱动器上。
  
若要尝试修复某个已部分失败的导入，需要：
  
-   复制日志文件
  
-   存储帐户密钥
  
  若要尝试修复某个已部分失败的导出，需要：
  
-   复制日志文件
  
-   清单文件（可选）
  
-   存储帐户密钥
  
## 安装 Azure 导入/导出工具  
 Azure 导入/导出工具由以下文件组成：
  
-   WAImportExport.exe
  
-   WAImportExport.exe.config
  
-   WAImportExportCore.dll
  
-   WAImportExportRepair.dll
  
-   Microsoft.WindowsAzure.Storage.dll
  
-   Hddid.dll
  
 请将这些文件复制到某个工作目录，如 `c:\WAImportExport`。接下来，在管理员模式下打开命令行窗口，将上述目录设为当前目录。
  
 若要输出命令帮助，请不带参数运行该工具：
  

	WAImportExport, a client tool for Azure Import/Export Service. Microsoft (c) 2013, 2014  
  
	Copy a Directory:  
	    WAImportExport.exe PrepImport  
	        /j:<JournalFile> [/logdir:<LogDirectory>] [/id:<SessionId>] [/resumesession]  
	        [/abortsession] [/sk:<StorageAccountKey>] [/csas:<ContainerSas>]  
	        [/t:<TargetDriveLetter>] [/format] [/silentmode] [/encrypt]  
	        [/bk:<BitLockerKey>] [/Disposition:<Disposition>] [/BlobType:<BlobType>]  
	        [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>]  
	        /srcdir:<SourceDirectory> /dstdir:<DestinationBlobVirtualDirectory>  
  
	Copy a File:  
	    WAImportExport.exe PrepImport  
	        /j:<JournalFile> [/logdir:<LogDirectory>] [/id:<SessionId>] [/resumesession]  
	        [/abortsession] [/sk:<StorageAccountKey>] [/csas:<ContainerSas>]  
	        [/t:<TargetDriveLetter>] [/format] [/silentmode] [/encrypt]  
	        [/bk:<BitLockerKey>] [/Disposition:<Disposition>] [/BlobType:<BlobType>]  
	        [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>]  
	        /srcfile:<SourceFilePath> /dstblob:<DestinationBlobPath>  
  
	Repair a Drive:  
	    WAImportExport.exe RepairImport | RepairExport  
	        /r:<RepairFile> [/logdir:<LogDirectory>]  
	        [/d:<TargetDirectories>] [/bk:<BitLockerKey>]  
	        /sn:<StorageAccountName> [/sk:<StorageAccountKey> | /csas:<ContainerSas>]  
	        [/CopyLogFile:<DriveCopyLogFile>] [/ManifestFile:<DriveManifestFile>]  
	        [/PathMapFile:<DrivePathMapFile>]  
  
	Preview an Export Job:  
	    WAImportExport.exe PreviewExport  
	        [/logdir:<LogDirectory>]  
	        /sn:<StorageAccountName> [/sk:<StorageAccountKey> | /csas:<ContainerSas>]  
	        /ExportBlobListFile:<ExportBlobListFile> /DriveSize:<DriveSize>  
  
	Parameters:  
  
	    /j:<JournalFile>  
	        - Required. Path to the journal file. Each drive must have one and only one  
	          journal file. The journal file corresponding to the target drive must always  
	          be specified.  
	    /logdir:<LogDirectory>  
	        - Optional. The log directory. Verbose log files as well as some temporary  
	          files will be written to this directory. If not specified, current directory  
	          will be used as the log directory.  
	    /id:<SessionId>  
	        - Required. The session Id is used to identify a copy session. It is used to  
	          ensure accurate recovery of an interrupted copy session. In addition, files  
	          that are copied in a copy session are stored in a directory named after the  
	          session Id on the target drive.  
	    /resumesession  
	        - Optional. If the last copy session was terminated abnormally, this parameter  
	          can be specified to resume the session.  
	    /abortsession  
	        - Optional. If the last copy session was terminated abnormally, this parameter  
	          can be specified to abort the session.  
	    /sn:<StorageAccountName>  
	        - Required. Only applicable for RepairImport and RepairExport. The name of  
	          the storage account.  
	    /sk:<StorageAccountKey>  
	        - Optional. The key of the storage account. One of /sk: and /csas: must be  
	          specified.  
	    /csas:<ContainerSas>  
	        - Optional. A container SAS, in format of <ContainerName>?<SasString>, to be  
	          used for import the data. One of /sk: and /csas: must be specified.  
	    /t:<TargetDriveLetter>  
	        - Required. Drive letter of the target drive.  
	    /r:<RepairFile>  
	        - Required. Only applicable for RepairImport and RepairExport.  
	          Path to the file for tracking repair progress. Each drive must have one  
	          and only one repair file.  
	    /d:<TargetDirectories>  
	        - Required. Only applicable for RepairImport and RepairExport.  
	          For RepairImport, one or more semicolon-separated directories to repair;  
	          For RepairExport, one directory to repair, e.g. root directory of the drive.  
	    /format  
	        - Optional. If specified, the target drive will be formatted. DO NOT specify  
	          this parameter if you do not want to format the drive.  
	    /silentmode  
	        - Optional. If not specified, the /format parameter will require a confirmation  
	          from console before the tool formats the drive. If this parameter is specified,  
	          not confirmation will be given for formatting the drive.  
	    /encrypt  
	        - Optional. If specified, the target drive will be encrypted with BitLocker.  
	          If the drive has already been encrypted with BitLocker, do not specify this  
	          parameter and instead specify the BitLocker key using the "/k" parameter.  
	    /bk:<BitLockerKey>  
	        - Optional. The current BitLocker key if the drive has already been encrypted  
	          with BitLocker.  
	    /Disposition:<Disposition>  
	        - Optional. Specifies the behavior when a blob with the same path as the one  
	          being imported already exists. Valid values are: rename, no-overwrite and  
	          overwrite (case-sensitive). If not specified, "rename" will be used as the  
	          default value.  
	    /BlobType:<BlobType>  
	        - Optional. The blob type for the imported blob(s). Valid values are BlockBlob  
	          and PageBlob. If not specified, BlockBlob will be used as the default value.  
	    /PropertyFile:<PropertyFile>  
	        - Optional. Path to the property file for the file(s) to be imported.  
	    /MetadataFile:<MetadataFile>  
	        - Optional. Path to the metadata file for the file(s) to be imported.  
	    /CopyLogFile:<DriveCopyLogFile>  
	        - Required. Only applicable for RepairImport and RepairExport. Path to the  
	          drive copy log file (verbose or error).  
	    /ManifestFile:<DriveManifestFile>  
	        - Required. Only applicable for RepairExport. Path to the drive manifest file.  
	    /PathMapFile:<DrivePathMapFile>  
	        - Optional. Only applicable for RepairImport. Path to the file containing  
	          mappings of file paths relative to the drive root to locations of actual files  
	          (tab-delimited). When first specified, it will be populated with file paths  
	          with empty targets, which means either they are not found in TargetDirectories,  
	          access denied, with invalid name, or they exist in multiple directories. The  
	          path map file can be manually edited to include the correct target paths and  
	          specified again for the tool to resolve the file paths correctly.  
	    /ExportBlobListFile:<ExportBlobListFile>  
	        - Required. Path to the XML file containing list of blob paths or blob path  
	          prefixes for the blobs to be exported. The file format is the same as the  
	          blob list blob format in the Put Job operation of the Import/Export Service  
	          REST API.  
	    /DriveSize:<DriveSize>  
	        - Required. Size of drives to be used for export. For example, 500GB, 1.5TB.  
	          Note: 1 GB = 1,000,000,000 bytes  
	                1 TB = 1,000,000,000,000 bytes  
	    /srcdir:<SourceDirectory>  
	        - Required. Source directory that contains files to be copied to the  
	          target drives.  
	    /dstdir:<DestinationBlobVirtualDirectory>  
	        - Required. Destination blob virtual directory to which the files will  
	          be imported.  
	    /srcfile:<SourceFilePath>  
	        - Required. Path to the source file to be imported.  
	    /dstblob:<DestinationBlobPath>  
	        - Required. Destination blob path for the file to be imported.  
	    /skipwrite
	        - Optional. To skip write process. Used for inplace data drive preparation.
	          Be sure to reserve enough space (3 GB per 7TB) for drive manifest file!
	Examples:  
  
	    Copy a source directory to a drive:  
	    WAImportExport.exe PrepImport  
	        /j:9WM35C2V.jrn /id:session#1 /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GEL  
	        xmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /t:x /format /encrypt /srcdir:d:\movi  
	        es\drama /dstdir:movies/drama/  
  
	    Copy another directory to the same drive following the above command:  
	    WAImportExport.exe PrepImport  
	        /j:9WM35C2V.jrn /id:session#2 /srcdir:d:\movies\action /dstdir:movies/action/  
  
	    Copy another file to the same drive following the above commands:  
	    WAImportExport.exe PrepImport  
	        /j:9WM35C2V.jrn /id:session#3 /srcfile:d:\movies\dvd.vhd /dstblob:movies/dvd.vhd /BlobType:PageBlob  
  
	    Preview how many 1.5 TB drives are needed for an export job:  
	    WAImportExport.exe PreviewExport  
	        /sn:mytestaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7K  
	        ysbbeKLDksg7VoN1W/a5UuM2zNgQ== /ExportBlobListFile:C:\temp\myexportbloblist.xml  
	        /DriveSize:1.5TB  
  
	    Repair an finished import job:  
	    WAImportExport.exe RepairImport  
	        /r:9WM35C2V.rep /d:X:\ /bk:442926-020713-108086-436744-137335-435358-242242-2795  
	        98 /sn:mytestaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94  
	        f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\temp\9WM35C2V_error.log 

	    Skip write process, inplace data drive preparation:
	    WAImportExport.exe PrepImport
	        /j:9WM35C2V.jrn /id:session#1 /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GEL
	        xmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /t:d /encrypt /srcdir:d:\movi
	        es\drama /dstdir:movies/drama/ /skipwrite

  
## 另请参阅  
 [为导入作业准备硬盘驱动器](/documentation/articles/storage-import-export-tool-preparing-hard-drives-import-v1/)
 [预览导出作业的驱动器使用情况](/documentation/articles/storage-import-export-tool-previewing-drive-usage-export-v1/)
 [使用复制日志文件查看作业状态](/documentation/articles/storage-import-export-tool-reviewing-job-status-v1/)
 [修复导入作业](/documentation/articles/storage-import-export-tool-repairing-an-import-job-v1/)
 [修复导出作业](/documentation/articles/storage-import-export-tool-repairing-an-export-job-v1/)
 [排查 Azure 导入/导出工具问题](/documentation/articles/storage-import-export-tool-troubleshooting-v1/)

<!---HONumber=Mooncake_0220_2017-->