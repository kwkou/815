<properties
    pageTitle="为导入作业准备硬盘驱动器 | Azure"
    description="了解如何为 Azure 导入-导出服务的导入作业准备一个或多个硬盘驱动器"
    author="muralikk"
    manager="syadav"
    editor="tysonn"
    services="storage"
    documentationcenter="" />
<tags
    ms.assetid="3d818245-8b1b-4435-a41f-8e5ec1f194b2"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/15/2017"
    wacn.date="02/24/2017"
    ms.author="muralikk" />  


# 为导入作业准备硬盘驱动器
若要为导入作业准备一个或多个硬盘驱动器，请执行以下步骤：

-   确定要导入 Blob 服务中的数据

-   在 Blob 服务中确定目标虚拟目录和 Blob

-   确定所需的驱动器数

-   将数据复制到每个硬盘驱动器

 有关示例工作流，请参阅 [为导入作业准备硬盘驱动器的示例工作流](/documentation/articles/storage-import-export-tool-sample-preparing-hard-drives-import-job-workflow-v1/)。

##  <a name="PrepareHardDrives_IdentifytheDatatoBeImported"></a>确定要导入的数据
 创建导入作业的第一个步骤是确定要导入哪些目录和文件，这可以是目录列表、唯一文件列表或两者的组合。包含某个目录时，该目录及其子目录中的所有文件都将成为导入作业的一部分。

> [AZURE.NOTE]<p>由于在包含父目录时将以递归方式包含子目录，因此请仅指定父目录。请勿同时指定其任何子目录。<p>目前，Azure 导入/导出工具存在以下限制：如果目录包含的数据超过硬盘驱动器可包含的数据，该目录需要分解为小目录。例如，如果目录包含 2.5TB 数据，而硬盘驱动器的容量仅为 2TB，则需要将此 2.5TB 目录分解为多个小目录。该工具的更高版本中将解除该限制。

##  <a name="PrepareHardDrives_IdentifytheDestinationLocationsintheBlobService"></a>在 Blob 服务中确定目标位置
 对于要导入的每个目录或文件，需要在 Azure Blob 服务中指定一个目标虚拟目录或 Blob。这些目标将用作 Azure 导入/导出工具的输入。请注意，目录应使用正斜杠字符“/”来分隔。

 下表显示了 Blob 目标的一些示例：

|源文件或目录|目标 Blob 或虚拟目录|
|------------------------------|-------------------------------------------|
|H:\Video|https://mystorageaccount.blob.core.chinacloudapi.cn/video|
|H:\Photo|https://mystorageaccount.blob.core.chinacloudapi.cn/photo|
|K:\Temp\FavoriteVideo.ISO|https://mystorageaccount.blob.core.chinacloudapi.cn/favorite/FavoriteVideo.ISO|
|\\\myshare\john\music|https://mystorageaccount.blob.core.chinacloudapi.cn/music|


##  <a name="PrepareHardDrives_DetermineHowManyDrivesAreNeeded"></a>确定所需的驱动器数
 接下来，需要确定：

-   存储数据所需的硬盘驱动器数。

-   将复制到每个硬盘驱动器中的目录和/或独立文件。

 确保准备好所需数量的硬盘驱动器，以便能够存储所要传输的数据。

##  <a name="PrepareHardDrives_CopyDatatoaSingleHardDrive"></a>将数据复制到硬盘驱动器
 本部分介绍如何调用 Azure 导入/导出工具，将数据复制到一个或多个硬盘驱动器。每次调用 Azure 导入/导出工具都会创建一个新的*复制会话*。需要为数据复制到的每个驱动器至少创建一个复制会话；在某些情况下，可能需要使用多个复制会话将所有数据复制到单个驱动器。下面是可能需要多个复制会话的一些原因：

-   必须为将数据复制到的每个驱动器创建一个单独的复制会话。

-   复制会话可将单个目录或单个 Blob 复制到驱动器。若要复制多个目录、多个 Blob 或两者的组合，需要创建多个复制会话。

-   可以指定要在执行导入作业过程中导入的 Blob 上设置的属性和元数据。为某个复制会话指定的属性或元数据将应用到该复制会话指定的所有 Blob。若要为某些 Blob 指定不同的属性或元数据，需要创建单独的复制会话。有关详细信息，请参阅 [Setting Properties and Metadata during the Import Process](/documentation/articles/storage-import-export-tool-setting-properties-metadata-import-v1/)（在导入过程中设置属性和元数据）。

> [AZURE.NOTE]如果有多台计算机满足 [Setting Up the Azure Import-Export Tool](/documentation/articles/storage-import-export-tool-setup-v1/)（设置 Azure 导入/导出工具）中所列的要求，则可以通过在每台计算机上运行此工具的实例，将数据并行复制到多个硬盘驱动器。

 对于使用 Azure 导入/导出工具准备的每个硬盘驱动器，该工具将创建单个日记文件。需要使用所有驱动器中的日记文件创建导入作业。在工具中断的情况下，还可使用日记文件来恢复驱动器准备操作。

### 导入作业的 Azure 导入/导出工具语法
 若要为导入作业准备驱动器，请使用 **PrepImport** 命令调用 Azure 导入/导出工具。要包含的参数取决于这是第一个复制会话还是后续复制会话。

 驱动器的第一个复制会话需要一些附加参数来指定：存储帐户密钥；目标驱动器号；是否必须格式化驱动器；是否必须对驱动器加密，如果是，则指定 BitLocker 密钥；以及日志目录。下面是用于复制目录或单个文件的初始复制会话的语法：

 **用于复制单个目录的第一个复制会话**

 `WAImportExport PrepImport /sk:<StorageAccountKey> /csas:<ContainerSas> /t: <TargetDriveLetter> [/format] [/silentmode] [/encrypt] [/bk:<BitLockerKey>] [/logdir:<LogDirectory>] /j:<JournalFile> /id:<SessionId> /srcdir:<SourceDirectory> /dstdir:<DestinationBlobVirtualDirectory> [/Disposition:<Disposition>] [/BlobType:<BlockBlob|PageBlob>] [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>]`  


 **用于复制单个文件的第一个复制会话**

 `WAImportExport PrepImport /sk:<StorageAccountKey> /csas:<ContainerSas> /t: <TargetDriveLetter> [/format] [/silentmode] [/encrypt] [/bk:<BitLockerKey>] [/logdir:<LogDirectory>] /j:<JournalFile> /id:<SessionId> /srcfile:<SourceFile> /dstblob:<DestinationBlobPath> [/Disposition:<Disposition>] [/BlobType:<BlockBlob|PageBlob>] [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>]`  


 在后续复制会话中，无需指定初始参数。下面是用于复制目录或单个文件的后续复制会话的语法：

 **用于复制单个目录的后续复制会话**

 `WAImportExport PrepImport /j:<JournalFile> /id:<SessionId> /srcdir:<SourceDirectory> /dstdir:<DestinationBlobVirtualDirectory> [/Disposition:<Disposition>] [/BlobType:<BlockBlob|PageBlob>] [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>]`  


 **用于复制单个文件的后续复制会话**

 `WAImportExport PrepImport /j:<JournalFile> /id:<SessionId> /srcfile:<SourceFile> /dstblob:<DestinationBlobPath> [/Disposition:<Disposition>] [/BlobType:<BlockBlob|PageBlob>] [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>]`  


### 硬盘驱动器的第一个复制会话的参数
 每次运行 Azure 导入/导出工具将文件复制到硬盘驱动器时，该工具会创建一个复制会话。每个复制会话会将单个目录或单个文件复制到硬盘驱动器。复制会话的状态将写入日记文件。如果某个复制会话中断（例如，由于系统电源中断），可以通过重新运行该工具并在命令行上指定日记文件来恢复该复制会话。

> [AZURE.WARNING] 如果为第一个复制会话指定 **/format** 参数，将格式化驱动器并清除该驱动器上的所有数据。建议仅对复制会话使用空白驱动器。

 用于每个驱动器的第一个复制会话的命令需要的参数不同于后续复制会话的命令所需的参数。下表列出了可用于第一个复制会话的附加参数：

|命令行参数|说明|
|-----------------------------|-----------------|
|**/sk:**<StorageAccountKey>|`Optional.` 将数据导入到的存储帐户的存储帐户密钥。必须在命令中包含 **/sk:**<StorageAccountKey> 或 **/csas:**<ContainerSas>。|
|**/csas:**<ContainerSas>|<p>`Optional`。用于将数据导入存储帐户的容器 SAS。必须在命令中包含 **/sk:**<StorageAccountKey> 或 **/csas:**<ContainerSas>。</p><p> 此参数的值必须以容器名称开头，后接问号 (?) 和 SAS 令牌。例如：</p><p> `mycontainer?sv=2014-02-14&sr=c&si=abcde&sig=LiqEmV%2Fs1LF4loC%2FJs9ZM91%2FkqfqHKhnz0JM6bqIqN0%3D&se=2014-11-20T23%3A54%3A14Z&sp=rwdl`</p><p>无论是在 URL 上还是存储的访问策略中指定，权限都必须包括“读取”、“写入”和“删除”导入作业，以及“读取”、“写入”和“列出”导出作业。</p><p> 如果指定此参数，则要导入或导出的所有 Blob 都必须位于共享访问签名中指定的容器内。</p>|
|**/t:**<TargetDriveLetter>|`Required.` 当前复制会话的目标硬盘驱动器的驱动器号（不带尾随冒号）。|
|**/format**|`Optional.` 在需要格式化驱动器时指定此参数；否则，请将其忽略。在对驱动器进行格式化之前，该工具将提示你通过控制台进行确认。若不希望显示该确认，请指定 /silentmode 参数。|
|**/silentmode**|`Optional.` 指定此参数将不显示对目标驱动器进行格式化的确认。|
|**/encrypt**|<p>`Optional.` 在尚未使用 BitLocker 对驱动器进行加密但需要使用此工具进行加密时，指定此参数。如果已使用 BitLocker 对驱动器进行加密，则忽略此参数并指定 `/bk` 参数，同时还提供现有 BitLocker 密钥。</p><p> 如果指定 `/format` 参数，则还必须指定 `/encrypt` 参数。</p>|
|**/bk:**<BitLockerKey>|`Optional.` 如果指定 `/encrypt`，请省略此参数。如果省略 `/encrypt`，则需要事先使用 BitLocker 为驱动器加密。使用此参数可指定 BitLocker 密钥。导入作业的所有硬盘驱动器都需要 BitLocker 加密。|
|**/logdir:**<LogDirectory>|`Optional.` 日志目录指定用于存储详细日志和临时清单文件的目录。如果未指定，则使用当前目录作为日志目录。|

### 所有复制会话所需的参数
 日记文件包含硬盘驱动器的所有复制会话的状态。它还包含创建导入作业所需的信息。运行 Azure 导入/导出工具时，始终必须指定一个日志文件，以及一个复制会话 ID。


|命令行参数|说明|
|---|---|
|**/j:**<JournalFile>|`Required.` 日记文件的路径。每个驱动器必须正好有一个日志文件。请注意，日志文件不得驻留在目标驱动器上。日记文件扩展名为 `.jrn`。|
|**/id:**<SessionId>|`Required.` 会话 ID 标识复制会话。它用于确保准确恢复中断的复制会话。复制会话中复制的文件将存储在以目标驱动器上的会话 ID 命名的目录中。|

### 用于复制单个目录的参数
 复制单个目录时，可应用以下必需和可选参数：

|命令行参数|说明|
|----------------------------|-----------------|
|**/srcdir:**<SourceDirectory>|`Required.` 包含要复制到目标驱动器中的文件的源目录。目录路径必须是绝对路径（而非相对路径）。|
|**/dstdir:**<DestinationBlobVirtualDirectory>|<p>`Required.` Azure 存储帐户中的目标虚拟目录的路径。虚拟目录可能存在，也可能不存在。</p><p> 可以指定容器或 Blob 前缀（如 `music/70s/`）。目标目录必须以容器名称开头，后接正斜杠“/”，并且可以选择包含以“/”结尾的虚拟 Blob 目录。</p><p> 当目标容器为根容器时，必须显式指定包含正斜杠的根容器（如 `$root/`）。由于根容器下的 Blob 的名称中不能包含“/”，因此当目标目录为根容器时，将不会复制源目录中的任何子目录。</p><p> 在指定目标虚拟目录或 Blob 时，请确保使用有效的容器名称。请记住，容器名称必须是小写的。有关容器命名规则，请参阅 [Naming and Referencing Containers, Blobs, and Metadata](https://docs.microsoft.com/rest/api/storageservices/fileservices/naming-and-referencing-containers--blobs--and-metadata)（命名和引用容器、Blob 与元数据）。</p>|
|**/Disposition:**<rename\|no-overwrite\|overwrite>|<p>`Optional.` 指定存在带指定地址的 Blob 时的行为。此参数的有效值为：`rename`、`no-overwrite` 和 `overwrite`。请注意，这些值区分大小写。如果未指定任何值，将使用默认值 `rename`。</p><p> 为此参数指定的值会影响 `/srcdir` 参数指定的目录中的所有文件。</p>|
|**/BlobType:**<BlockBlob\|PageBlob>|<p>`Optional.` 指定目标 Blob 的 Blob 类型。有效值为：`BlockBlob` 和 `PageBlob`。请注意，这些值区分大小写。如果未指定任何值，将使用默认值 `BlockBlob`。</p><p> 在大多数情况下，建议使用 `BlockBlob`。如果指定 `PageBlob`，目录中每个文件的长度必须是 512 的倍数（即页 Blob 的页面大小）。</p>|
|**/PropertyFile:**<PropertyFile>|`Optional.` 目标 Blob 的属性文件的路径。有关详细信息，请参阅[导入/导出服务元数据和属性文件格式](/documentation/articles/storage-import-export-file-format-metadata-and-properties/)。|
|**/MetadataFile:**<MetadataFile>|`Optional.` 目标 Blob 的元数据文件的路径。有关详细信息，请参阅[导入/导出服务元数据和属性文件格式](/documentation/articles/storage-import-export-file-format-metadata-and-properties/)。|

### 用于复制单个文件的参数
 复制单个文件时，可应用以下必需和可选参数：

|命令行参数|说明|
|----------------------------|-----------------|
|**/srcfile:**<SourceFile>|`Required.` 要复制的文件的完整路径。目录路径必须是绝对路径（而非相对路径）。|
|**/dstblob:**<DestinationBlobPath>|<p>`Required.` Azure 存储帐户中的目标 Blob 的路径。该 Blob 可能存在，也可能不存在。<p/><p> 指定以容器名称开头的 Blob 名称。Blob 名称不能以“/”或存储帐户名称开头。有关 Blob 命名规则，请参阅 [Naming and Referencing Containers, Blobs, and Metadata](https://docs.microsoft.com/rest/api/storageservices/fileservices/naming-and-referencing-containers--blobs--and-metadata)（命名和引用容器、Blob 与元数据）。<p/><p> 当目标容器为根容器时，必须显式将 `$root` 指定为容器，如 `$root/sample.txt`。请注意，根容器下的 Blob 的名称不能包含“/”。</p>|
|**/Disposition:**<rename\|no-overwrite\|overwrite>|`Optional.` 指定存在带指定地址的 Blob 时的行为。此参数的有效值为：`rename`、`no-overwrite` 和 `overwrite`。请注意，这些值区分大小写。如果未指定任何值，将使用默认值 `rename`。|
|**/BlobType:**<BlockBlob\|PageBlob>|<p>`Optional.` 指定目标 Blob 的 Blob 类型。有效值为：`BlockBlob` 和 `PageBlob`。请注意，这些值区分大小写。如果未指定任何值，将使用默认值 `BlockBlob`。</p><p> 在大多数情况下，建议使用 `BlockBlob`。如果指定 `PageBlob`，目录中每个文件的长度必须是 512 的倍数（即页 Blob 的页面大小）。</p>|
|**/PropertyFile:**<PropertyFile>|`Optional.` 目标 Blob 的属性文件的路径。有关详细信息，请参阅[导入/导出服务元数据和属性文件格式](/documentation/articles/storage-import-export-file-format-metadata-and-properties/)。|
|**/MetadataFile:**<MetadataFile>|`Optional.` 目标 Blob 的元数据文件的路径。有关详细信息，请参阅[导入/导出服务元数据和属性文件格式](/documentation/articles/storage-import-export-file-format-metadata-and-properties/)。|

### 恢复已中断的复制会话
 如果复制会话因任何原因导致中断，可通过在仅指定日记文件的情况下运行该工具来恢复会话：


	WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> /ResumeSession


 只能恢复最近异常终止的复制会话。

> [AZURE.IMPORTANT]恢复复制会话时，请不要通过添加或删除文件来修改源数据文件和目录。

### 中止已中断的复制会话
 如果复制会话已中断且无法恢复（例如，如果源目录被证实不可访问），则必须中止当前会话，以便可将其回滚并启动新的复制会话：


	WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> /AbortSession


 如果异常终止，则只能中止最后的复制会话。请注意，无法中止驱动器的第一个复制会话。在此情况下，必须重新启动包含新日记文件的复制会话。

## 另请参阅
 [设置 Azure 导入/导出工具](/documentation/articles/storage-import-export-tool-setup-v1/)
 [在导入过程中设置属性和元数据](/documentation/articles/storage-import-export-tool-setting-properties-metadata-import-v1/)
 [为导入作业准备硬盘驱动器的示例工作流](/documentation/articles/storage-import-export-tool-sample-preparing-hard-drives-import-job-workflow-v1/)
 [常用命令快速参考](/documentation/articles/storage-import-export-tool-quick-reference-v1/)
 [使用复制日志文件查看作业状态](/documentation/articles/storage-import-export-tool-reviewing-job-status-v1/)
 [修复导入作业](/documentation/articles/storage-import-export-tool-repairing-an-import-job-v1/)
 [修复导出作业](/documentation/articles/storage-import-export-tool-repairing-an-export-job-v1/)
 [排查 Azure 导入/导出工具问题](/documentation/articles/storage-import-export-tool-troubleshooting-v1/)

<!---HONumber=Mooncake_0220_2017-->