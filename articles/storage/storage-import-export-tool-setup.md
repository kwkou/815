<properties
    pageTitle="设置 Azure 导入/导出工具 | Azure"
    description="了解如何设置 Azure 导入/导出服务的驱动器准备和修复工具"
    author="muralikk"
    manager="syadav"
    editor="tysonn"
    services="storage"
    documentationcenter="" />  

<tags
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

* 在创建导入作业之前，可以使用此工具将数据复制到要寄送给 Azure 数据中心的硬盘驱动器。
* 完成某个导入作业后，可以使用此工具修复已损坏、丢失或与其他 Blob 冲突的任何 Blob。
* 通过某个已完成的导出作业收到驱动器后，可以使用此工具修复这些驱动器上已损坏或丢失的任何文件。

## 先决条件

若要为导出作业**准备驱动器**，需要满足以下先决条件：

* 必须拥有一个有效的 Azure 订阅。
* 该订阅必须包含一个存储帐户，其中有足够的可用空间可存储所要导入的文件。
* 需要存储帐户的至少一个帐户密钥。
* 需要准备一台装有 Windows 7、Windows Server 2008 R2 或更高版本的 Windows 操作系统的计算机（“复制计算机”）。
* 必须在复制计算机上安装 .NET Framework 4。
* 必须在复制计算机上启用 BitLocker。
* 需要一个或多个已连接到复制计算机的空 3.5 英寸 SATA 硬盘驱动器。
* 打算导入的文件必须可从复制计算机访问，无论这些文件是位于网络共享还是本地硬盘驱动器上。

若要尝试**修复**某个已部分失败的导入，需要：

* 复制日志文件
* 存储帐户密钥

若要尝试**修复**某个已部分失败的导出，需要：

* 复制日志文件
* 清单文件（可选）
* 存储帐户密钥

## 安装 Azure 导入/导出工具

首先，请[下载 Azure 导入/导出工具](http://download.microsoft.com/download/3/6/B/36BFF22A-91C3-4DFC-8717-7567D37D64C5/WAImportExport.zip)，并将其解压缩到计算机上的某个目录，如 `c:\WAImportExport`。

Azure 导入/导出工具由以下文件组成：

* dataset.csv
* driveset.csv
* hddid.dll
* Microsoft.Data.Services.Client.dll
* Microsoft.WindowsAzure.Storage.dll
* WAImportExport.exe
* WAImportExport.exe.config
* WAImportExportCore.dll
* WAImportExportRepair.dll

接下来，以**管理员模式**打开命令提示窗口，并将目录切换到包含解压缩文件的目录。

若要输出命令帮助，请不带参数运行该工具：


	WAImportExport, a client tool for Microsoft Azure Import/Export Service. Microsoft (c) 2013


	Copy directories and/or files with a new copy session:
	    WAImportExport.exe PrepImport
	        /j:<JournalFile>
	        /id:<SessionId> [/logdir:<LogDirectory>]
	        [/sk:<StorageAccountKey>]
	        [/silentmode]
	        [/InitialDriveSet:<driveset.csv>]
	        DataSet:<dataset.csv>

	Add more drives:
	    WAImportExport.exe PrepImport
	        /j:<JournalFile>
	        /id:<SessionId>
	        /AdditionalDriveSet:<driveset.csv>

	Abort an interrupted copy session:
	    WAImportExport.exe PrepImport
	        /j:<JournalFile>
	        /id:<SessionId>
	        /AbortSession

	Resume an interrupted copy session:
	    WAImportExport.exe PrepImport
	        /j:<JournalFile>
	        /id:<SessionId>
	        /ResumeSession

	List drives:
	    WAImportExport.exe PrepImport /j:<JournalFile> /ListDrives

	List copy sessions:
	    WAImportExport.exe PrepImport /j:<JournalFile> /ListCopySessions

	Repair a Drive:
	    WAImportExport.exe RepairImport | RepairExport
	        /r:<RepairFile> [/logdir:<LogDirectory>]
	        [/d:<TargetDirectories>] [/bk:<BitLockerKey>]
	        /sn:<StorageAccountName> /sk:<StorageAccountKey>
	        [/CopyLogFile:<DriveCopyLogFile>] [/ManifestFile:<DriveManifestFile>]
	        [/PathMapFile:<DrivePathMapFile>]

	Preview an Export Job:
	    WAImportExport.exe PreviewExport
	        [/logdir:<LogDirectory>]
	        /sn:<StorageAccountName> /sk:<StorageAccountKey>
	        /ExportBlobListFile:<ExportBlobListFile> /DriveSize:<DriveSize>

	Parameters:

	    /j:<JournalFile>
	        - Required. Path to the journal file. A journal file tracks a set of drives and
	          records the progress in preparing these drives. The journal file must always
	          be specified.
	    /logdir:<LogDirectory>
	        - Optional. The log directory. Verbose log files as well as some temporary
	          files will be written to this directory. If not specified, current directory
	          will be used as the log directory. The log directory can be specified only
	          once for the same journal file.
	    /id:<SessionId>
	        - Optional. The session Id is used to identify a copy session. It is used to
	          ensure accurate recovery of an interrupted copy session.
	    /ResumeSession
	        - Optional. If the last copy session was terminated abnormally, this parameter
	          can be specified to resume the session.
	    /AbortSession
	        - Optional. If the last copy session was terminated abnormally, this parameter
	          can be specified to abort the session.
	    /sn:<StorageAccountName>
	        - Required. Only applicable for RepairImport and RepairExport. The name of
	          the storage account.
	    /sk:<StorageAccountKey>
	        - Required. The key of the storage account.
	    /InitialDriveSet:<driveset.csv>
	        - Required. A .csv file that contains a list of drives to prepare.
	    /AdditionalDriveSet:<driveset.csv>
	        - Required. A .csv file that contains a list of additional drives to be added.
	    /r:<RepairFile>
	        - Required. Only applicable for RepairImport and RepairExport.
	          Path to the file for tracking repair progress. Each drive must have one
	          and only one repair file.
	    /d:<TargetDirectories>
	        - Required. Only applicable for RepairImport and RepairExport.
	          For RepairImport, one or more semicolon-separated directories to repair;
	          For RepairExport, one directory to repair, e.g. root directory of the drive.
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
	    /DataSet:<dataset.csv>
	        - Required. A .csv file that contains a list of directories and/or a list files
	          to be copied to target drives.

	    /silentmode
	        - Optional. If not specified, it will remind you the requirement of drives and
	          need your confirmation to continue.

	Examples:

	    Copy a data set to a drive:
	    WAImportExport.exe PrepImport
	        /j:9WM35C2V.jrn /id:session#1 /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GEL
	        xmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /InitialDriveSet:driveset1.csv
	        /DataSet:data.csv

	    Copy another dataset to the same drive following the above command:
	    WAImportExport.exe PrepImport /j:9WM35C2V.jrn /id:session#2 /DataSet:dataset2.csv

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


## 后续步骤

* [为导入作业准备硬盘驱动器](/documentation/articles/storage-import-export-tool-preparing-hard-drives-import/)
* [预览导出作业的驱动器使用情况](/documentation/articles/storage-import-export-tool-previewing-drive-usage-export-v1/)
* [使用复制日志文件查看作业状态](/documentation/articles/storage-import-export-tool-reviewing-job-status-v1/)
* [修复导入作业](/documentation/articles/storage-import-export-tool-repairing-an-import-job-v1/)
* [修复导出作业](/documentation/articles/storage-import-export-tool-repairing-an-export-job-v1/)
* [排查 Azure 导入/导出工具问题](/documentation/articles/storage-import-export-tool-troubleshooting-v1/)

<!---HONumber=Mooncake_0220_2017-->