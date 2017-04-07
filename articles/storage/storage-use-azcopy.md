<properties
    pageTitle="使用 AzCopy 将数据复制或移动到存储 | Azure"
    description="使用 AzCopy 实用程序将数据移动或复制到 blob、表和文件内容或从 blob、表和文件内容移动或复制数据。从本地文件将数据复制到 Azure 存储，或者在存储帐户中或存储帐户之间复制数据。轻松地将数据迁移到 Azure 存储。"
    services="storage"
    documentationcenter=""
    author="seguler"
    manager="jahogg"
    editor="tysonn" />
<tags
    ms.assetid="aa155738-7c69-4a83-94f8-b97af4461274"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/30/2017"
    wacn.date="03/20/2017"
    ms.author="seguler" />

# 使用 AzCopy 命令行实用程序传输数据
## 概述
AzCopy 是一个 Windows 命令行实用程序，专用于使用具有优化性能的简单命令将数据复制到 Azure Blob、文件和表存储以及从这些位置复制数据。可在存储帐户中将数据从一个对象复制到另一个对象，或者在存储帐户之间复制。


> [AZURE.NOTE] 本指南假定用户已熟悉了 [Azure 存储](/home/features/storage/)。如果不熟悉，阅读 [Azure 存储简介](/documentation/articles/storage-introduction/)文档将有所帮助。最重要的是，需要[创建一个存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)以开始使用 AzCopy。

## 下载并安装 AzCopy
### Windows
下载[最新版本的 AzCopy](http://aka.ms/downloadazcopy)。

### Mac/Linux
AzCopy 不可用于 Mac/Linux 操作系统。但是，Azure CLI 可用作将数据复制 Azure 存储以及从其中复制数据的替代方法。请参阅[配合使用 Azure CLI 与 Azure 存储](/documentation/articles/storage-azure-cli/)了解详细信息。

## 编写第一条 AzCopy 命令
AzCopy 命令的基本语法是：

    AzCopy /Source:<source> /Dest:<destination> [Options]

打开一个命令窗口，然后导航到计算机上的 AzCopy 安装目录，该位置存放着可执行的 `AzCopy.exe`。如果需要，可以将 AzCopy 安装位置添加到系统路径。默认情况下，AzCopy 安装到 `%ProgramFiles(x86)%\Microsoft SDKs\Azure\AzCopy` 或 `%ProgramFiles%\Microsoft SDKs\Azure\AzCopy`。

以下示例演示了将数据复制到 Microsoft Azure Blob、文件和表以及从这些位置复制数据的各种情况。请参阅 [AzCopy 参数](#azcopy-parameters)部分，了解每个示例中使用的参数的详细说明。

## Blob：下载
### 下载单个 blob
	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:"abc.txt"

请注意，如果文件夹 `C:\myfolder` 不存在，AzCopy 会创建该文件夹并将 `abc.txt ` 下载到新文件夹中。

### 从次要区域下载单个 blob
	AzCopy /Source:https://myaccount-secondary.blob.core.chinacloudapi.cn/mynewcontainer /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt

请注意，必须已启用读取访问异地冗余存储。

### 下载所有 blob
	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /S

假定指定的容器中存在以下 blob：

	abc.txt
	abc1.txt
	abc2.txt
	vd1\a.txt
	vd1\abcd.txt

在下载操作完成后，目录 `C:\myfolder` 中将包括以下文件：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt
	C:\myfolder\vd1\a.txt
	C:\myfolder\vd1\abcd.txt

如果未指定选项 `/S`，则不会下载任何 blob。

### 下载具有指定前缀的 blob

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:a /S

假定指定的容器中存在以下 blob。将下载所有以前缀 `a` 开头的 blob：

	abc.txt
	abc1.txt
	abc2.txt
	xyz.txt
	vd1\a.txt
	vd1\abcd.txt

在下载操作完成后，文件夹 `C:\myfolder` 中将包括以下文件：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt

前缀适用于虚拟目录，后者构成了 blob 名称的第一个部分。在上面所示的示例中，虚拟目录与指定的前缀不匹配，因此不会下载它。此外，如果未指定选项 `\S`，则 AzCopy 将不会下载任何 blob。

### 将已导出文件的上次修改时间设置为与源 blob 相同

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /MT

还可以根据 blob 的上次修改时间将其从下载操作中排除例如，如果想要排除其上次修改时间与目标文件相同或晚于目标文件的 blob，则添加 `/XN` 选项：

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /MT /XN

或者，如果想要排除其上次修改时间与目标文件相同或早于目标文件的 blob，则添加 `/XO` 选项：

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /MT /XO

##<a name="blob-upload"></a> Blob：上传

### 上传单个文件

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Pattern:"abc.txt"

如果指定的目标容器不存在，AzCopy 则将创建它并将文件上传到其中。

### 将单个文件上传到虚拟目录

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer/vd /DestKey:key /Pattern:abc.txt

如果指定的虚拟目录不存在，AzCopy 则将上传文件以在其名称中包括虚拟目录（ *例如* 上例中的 `vd/abc.txt`）。

### 上传全部文件

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /S

指定选项 `/S` 会以递归方式将指定目录的内容上传到 Blob 存储，这意味着也会上传所有的子文件夹及其文件。例如，假定以下文件存在于文件夹 `C:\myfolder` 中：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt
	C:\myfolder\subfolder\a.txt
	C:\myfolder\subfolder\abcd.txt

在上传操作完成后，容器中将包括以下文件：

  	abc.txt
	abc1.txt
	abc2.txt
	subfolder\a.txt
	subfolder\abcd.txt

如果未指定选项 `/S`，AzCopy 将不会以递归方式上传。在上传操作完成后，容器中将包括以下文件：

	abc.txt
	abc1.txt
	abc2.txt

### 上传与指定模式相匹配的文件

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Pattern:a* /S

假定以下文件存在于文件夹 `C:\myfolder` 中：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt
	C:\myfolder\xyz.txt
	C:\myfolder\subfolder\a.txt
	C:\myfolder\subfolder\abcd.txt

在上传操作完成后，容器中将包括以下文件：

	abc.txt
	abc1.txt
	abc2.txt
	subfolder\a.txt
	subfolder\abcd.txt

如果未指定选项 `/S`，AzCopy 将仅上传不会在虚拟目录中驻留的 blob：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt

### 指定目标 blob 的 MIME 内容类型

默认情况下，AzCopy 将目标 blob 的内容类型设置为 `application/octet-stream`。从 3.1.0 版开始，可以通过选项 `/SetContentType:[content-type]` 显示指定内容类型。此语法将在上传操作中设置所有 blob 的内容类型。

	AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.blob.core.chinacloudapi.cn/myContainer/ /DestKey:key /Pattern:ab /SetContentType:video/mp4

如果指定不带任何值的 `/SetContentType` ，AzCopy 将根据文件扩展名设置每个 Blob 或文件的内容类型。

	AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.blob.core.chinacloudapi.cn/myContainer/ /DestKey:key /Pattern:ab /SetContentType

## Blob：复制

### 在存储帐户内复制单个 blob

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer1 /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer2 /SourceKey:key /DestKey:key /Pattern:abc.txt

在存储帐户内复制某个 blob 时，将执行[服务器端复制](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)操作。

### 跨存储帐户复制单个 blob

	AzCopy /Source:https://sourceaccount.blob.core.chinacloudapi.cn/mycontainer1 /Dest:https://destaccount.blob.core.chinacloudapi.cn/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt

在跨存储帐户复制某个 blob 时，将执行[服务器端复制](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)操作。

### 将单个 blob 从次要区域复制到主要区域

	AzCopy /Source:https://myaccount1-secondary.blob.core.chinacloudapi.cn/mynewcontainer1 /Dest:https://myaccount2.blob.core.chinacloudapi.cn/mynewcontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt

请注意，必须已启用读取访问异地冗余存储。

### 跨存储帐户复制单个 blob 及其快照

	AzCopy /Source:https://sourceaccount.blob.core.chinacloudapi.cn/mycontainer1 /Dest:https://destaccount.blob.core.chinacloudapi.cn/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt /Snapshot

在复制操作完成后，目标容器中将包括 blob 及其快照。假定上面的示例中的 blob 具有两个快照，则容器将包括以下 blob 和快照：

	abc.txt
	abc (2013-02-25 080757).txt
	abc (2014-02-21 150331).txt

### 跨存储帐户同步复制 blob
默认情况下，AzCopy 以异步方式复制两个存储终结点之间的数据。因此，复制操作使用空闲的带宽容量在后台运行，没有规定 blob 复制速率的 SLA，AzCopy 将定期检查复制状态直到复制完成或失败。

`/SyncCopy` 选项确保复制操作的速度一致。AzCopy 通过下载 blob，将 blob 从指定的源复制到本地内存，然后将上传到 Blob 存储目标，以实现同步复制。

	AzCopy /Source:https://myaccount1.blob.core.chinacloudapi.cn/myContainer/ /Dest:https://myaccount2.blob.core.chinacloudapi.cn/myContainer/ /SourceKey:key1 /DestKey:key2 /Pattern:ab /SyncCopy

与异步复制相比，`/SyncCopy` 可能会产生额外的对外费用，建议在与源存储帐户所在的同一区域的 Azure VM 中使用该选项，以避免对外费用。

## 文件：下载

### 下载单个文件

	AzCopy /Source:https://myaccount.file.core.chinacloudapi.cn/myfileshare/myfolder1/ /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt

如果指定的源是 Azure 文件共享，则必须指定确切的文件名（ *例如* `abc.txt`）以下载单个文件，或者指定选项 `/S` 来以递归方式下载该共享中的所有文件。尝试同时指定文件模式和选项 `/S` 将导致错误。

### 下载所有文件

	AzCopy /Source:https://myaccount.file.core.chinacloudapi.cn/myfileshare/ /Dest:C:\myfolder /SourceKey:key /S

请注意，不会下载任何空文件夹。

## 文件：上传

### 上传单个文件

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.chinacloudapi.cn/myfileshare/ /DestKey:key /Pattern:abc.txt

### 上传全部文件

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.chinacloudapi.cn/myfileshare/ /DestKey:key /S

请注意，不会上传任何空文件夹。

### 上传与指定模式相匹配的文件

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.chinacloudapi.cn/myfileshare/ /DestKey:key /Pattern:ab* /S

##<a name="file-copy"></a> 文件：复制

### 跨文件共享复制

	AzCopy /Source:https://myaccount1.file.core.chinacloudapi.cn/myfileshare1/ /Dest:https://myaccount2.file.core.chinacloudapi.cn/myfileshare2/ /SourceKey:key1 /DestKey:key2 /S

### 从文件共享复制到 blob

	AzCopy /Source:https://myaccount1.file.core.chinacloudapi.cn/myfileshare/ /Dest:https://myaccount2.blob.core.chinacloudapi.cn/mycontainer/ /SourceKey:key1 /DestKey:key2 /S

请注意，不支持从文件存储到页 Blob 的异步复制。

### 从 blob 复制到文件共享

	AzCopy /Source:https://myaccount1.blob.core.chinacloudapi.cn/mycontainer/ /Dest:https://myaccount2.file.core.chinacloudapi.cn/myfileshare/ /SourceKey:key1 /DestKey:key2 /S

### 同步复制文件

可以指定选项 `/SyncCopy`，以从文件存储到文件存储、从文件存储到 Blob 存储以及从 Blob 存储到文件存储同步复制数据，AzCopy 通过将源数据下载到本地内存并再将其上传到目标以实现此同步操作。

	AzCopy /Source:https://myaccount1.file.core.chinacloudapi.cn/myfileshare1/ /Dest:https://myaccount2.file.core.chinacloudapi.cn/myfileshare2/ /SourceKey:key1 /DestKey:key2 /S /SyncCopy

当从文件存储复制到 Blob 存储时，默认的 blob 类型是块 blob，用户可以指定选项 `/BlobType:page` 以更改目标 blob 类型。

请注意，与异步复制相比，`/SyncCopy` 可能会产生额外的对外费用，建议在与源存储帐户所在的同一区域的 Azure VM 中使用该选项，以避免对外费用。

## 表：导出

### 导出表

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:C:\myfolder\ /SourceKey:key

AzCopy 将一个清单文件写入到指定的目标文件夹。在导入过程中使用该清单文件查找必要的数据文件并执行数据验证。默认情况下，清单文件使用以下命名约定：

	<account name>_<table name>_<timestamp>.manifest

用户还可以指定选项 `/Manifest:<manifest file name>` 以设置清单文件名。

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:C:\myfolder\ /SourceKey:key /Manifest:abc.manifest

### 将导出拆分为多个文件

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/mytable/ /Dest:C:\myfolder /SourceKey:key /S /SplitSize:100

AzCopy 在已拆分的数据文件名称中使用 *卷索引* 来区分多个文件。卷索引由两个部分组成： *分区键范围索引* 和 *拆分文件索引* 。两个索引都是从零开始。

如果用户未指定选项 `/PKRS`，则分区键范围索引将是 0。

例如，假设 AzCopy 在用户指定选项 `/SplitSize` 后生成了两个数据文件。生成的数据文件名称可能是：

	myaccount_mytable_20140903T051850.8128447Z_0_0_C3040FE8.json
	myaccount_mytable_20140903T051850.8128447Z_0_1_0AB9AC20.json

请注意，选项 `/SplitSize` 的最小可能值为 32 MB。如果指定的目标是 Blob 存储，无论用户是否指定了选项 `/SplitSize`，AzCopy 将在数据文件的大小达到 blob 的大小限制 (200 GB) 时拆分数据文件。

### 将表导出为 JSON 或 CSV 数据文件格式

默认情况下，AzCopy 会将表导出为 JSON 数据文件。可以指定选项 `/PayloadFormat:JSON|CSV` 以将表导出为 JSON 或 CSV。

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:C:\myfolder\ /SourceKey:key /PayloadFormat:CSV

指定 CSV 负载格式时，AzCopy 还将为每个数据文件生成文件扩展名为 `.schema.csv` 的架构文件。

### 并发导出表实体

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:C:\myfolder\ /SourceKey:key /PKRS:"aa#bb"

当用户指定了选项 `/PKRS` 时，AzCopy 将启动并发操作来导出实体。每个操作导出一个分区键范围。

请注意，并发操作的数量还受选项 `/NC` 的控制。当复制表实体时，AzCopy 使用核心处理器的数量作为 `/NC` 的默认值，即使未指定 `/NC` 也是如此。当用户指定了选项 `/PKRS` 时，AzCopy 将使用以下两个值中的较小者（分区键范围和隐式或显式指定的并发操作数量）来确定要启动的并发操作的数量。有关详细信息，请在命令行中键入 `AzCopy /?:NC`。

### 将表导出到 blob

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer/ /SourceKey:key1 /Destkey:key2

AzCopy 将使用以下命名约定在 blob 容器中生成一个 JSON 数据文件：

	<account name>_<table name>_<timestamp>_<volume index>_<CRC>.json

生成的 JSON 数据文件遵循最少元数据负载格式。有关此负载格式的详细信息，请参阅[表服务操作的有效负载格式](http://msdn.microsoft.com/zh-cn/library/azure/dn535600.aspx)。

请注意，当将表导出到 blob 中时，AzCopy 会将表实体下载到本地临时数据文件，然后再将这些实体上传到 blob。这些临时数据文件将放入默认路径为“<code>%LocalAppData%\\Microsoft\\Azure\\AzCopy</code>”的日志文件文件夹中，可以指定选项 /Z:[journal-file-folder] 以更改日志文件文件夹的位置，从而更改临时数据文件的位置。临时数据文件的大小由表实体的大小和使用选项 /SplitSize 指定的大小所决定，尽管本地磁盘中的临时数据文件在上传到 blob 后将被立即删除，但请确保在删除之前，拥有足够的本地磁盘空间来存储这些临时数据文件。

## 表：导入

### 导入表

	AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.table.core.chinacloudapi.cn/mytable1/ /DestKey:key /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:InsertOrReplace

选项 `/EntityOperation` 指示如何将实体插入到表中。可能的值包括：

- `InsertOrSkip`：跳过现有实体，或者插入新实体（如果它不存在于表中）。
- `InsertOrMerge`：合并现有实体，或者插入新实体（如果它不存在于表中）。
- `InsertOrReplace`：替换现有实体，或者插入新实体（如果它不存在于表中）。

请注意，在导入方案中不能指定选项 `/PKRS`。与导出方案不同（在导出方案中必须指定 `/PKRS` 选项才会启动并发操作），在导入表时，AzCopy 将默认启动并发操作。启动的并发操作的默认数量与核心处理器的数量相等；但是，可以通过选项 `/NC` 指定一个不同的并发数量。有关详细信息，请在命令行中键入 `AzCopy /?:NC`。

请注意，AzCopy 仅支持导入 JSON 文件，不支持导入 CSV 文件。AzCopy 不支持来自用户创建的 JSON 和清单文件的表导入。这两类文件必须来自 AzCopy 表导出。若要避免错误，请不要修改导出的 JSON 或清单文件。

### 使用 blob 将实体导入表中

假定 Blob 容器包含如下内容：表示 Azure 表的 JSON 文件及其随附的清单文件。

	myaccount_mytable_20140103T112020.manifest
	myaccount_mytable_20140103T112020_0_0_0AF395F1DC42E952.json

可以使用 blob 容器中的清单文件运行以下命令将实体导入表中：

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:https://myaccount.table.core.chinacloudapi.cn/mytable /SourceKey:key1 /DestKey:key2 /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:"InsertOrReplace"

## AzCopy 的其他功能

### 仅复制目标中不存在的数据

`/XO` 和 `/XN` 参数分别用于阻止复制较早或较新的源资源。如果只想复制目标中不存在的源资源，可以在 AzCopy 命令中指定这两个参数：

	/Source:http://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /XO /XN

	/Source:C:\myfolder /Dest:http://myaccount.file.core.chinacloudapi.cn/myfileshare /DestKey:<destkey> /S /XO /XN

	/Source:http://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:http://myaccount.blob.core.chinacloudapi.cn/mycontainer1 /SourceKey:<sourcekey> /DestKey:<destkey> /S /XO /XN

注意：当源或目标为表时，则不支持此操作。

### 使用响应文件指定命令行参数

	AzCopy /@:"C:\responsefiles\copyoperation.txt"

可以在响应文件中包括任何 AzCopy 命令行参数。AzCopy 会像处理在命令行上指定的参数一样处理该文件中的参数，使用该文件的内容执行直接替换。

假定有一个名为 `copyoperation.txt` 的响应文件，其中包含以下行：可以在同一行或多行中指定

	/Source:http://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /Y

每个 AzCopy 参数：

	/Source:http://myaccount.blob.core.chinacloudapi.cn/mycontainer
	/Dest:C:\myfolder
	/SourceKey:<sourcekey>
	/S
	/Y

如果将参数拆分到两行（如此处所示的 `/sourcekey` 参数），AzCopy 将会失败：

	http://myaccount.blob.core.chinacloudapi.cn/mycontainer
 	C:\myfolder
	/sourcekey:
	<sourcekey>
	/S
	/Y

### 使用多个响应文件指定命令行参数

假定有一个名为 `source.txt` 的响应文件，该文件指定了一个源容器：

	/Source:http://myaccount.blob.core.chinacloudapi.cn/mycontainer

有一个名为 `dest.txt` 的响应文件，该文件在文件系统中指定了一个目标文件夹：

	/Dest:C:\myfolder

并且有一个名为 `options.txt` 的响应文件，该文件指定了 AzCopy 的选项：

	/S /Y

要使用这些响应文件（都位于目录 `C:\responsefiles` 中）调用 AzCopy，请使用以下命令：

	AzCopy /@:"C:\responsefiles\source.txt" /@:"C:\responsefiles\dest.txt" /SourceKey:<sourcekey> /@:"C:\responsefiles\options.txt"   

AzCopy 会像在命令行上包括了所有单个参数一样来处理此命令：

	AzCopy /Source:http://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /Y

### 指定共享访问签名 (SAS)

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer1 /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer2 /SourceSAS:SAS1 /DestSAS:SAS2 /Pattern:abc.txt

此外，还可以在容器 URI 上指定一个 SAS：

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer1/?SourceSASToken /Dest:C:\myfolder /S

### 日志文件文件夹

每次向 AzCopy 发出命令时，它都会检查默认文件夹中是否存在日志文件，或者通过此选项指定的文件夹中是否存在日志文件。如果这两个位置中都不存在日志文件，AzCopy 则会将操作视为新操作并生成一个新的日志文件。

如果存在日志文件，AzCopy 则将检查输入的命令行是否与该日志文件中的命令行相匹配。如果两个命令行相匹配，AzCopy 则将恢复未完成的操作。如果它们不匹配，系统将提示用户是选择覆盖该日志文件以启动新操作，还是取消当前操作。

如果想要为日志文件使用默认位置：

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Z

如果省略了选项 `/Z`，或者指定了选项 `/Z` 但未指定文件夹路径（如上所示），AzCopy 则会在默认位置中创建恢复日志，默认位置为 `%SystemDrive%\Users\%username%\AppData\Local\Microsoft\Azure\AzCopy`。如果恢复日志已存在，AzCopy 则将继续根据恢复日志恢复操作。

如果想要为恢复日志指定自定义位置：

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Z:C:\journalfolder\

如果恢复日志尚不存在，此示例将创建恢复日志。如果它已存在，AzCopy 则将根据该恢复日志恢复操作。

如果想要恢复 AzCopy 操作：

	AzCopy /Z:C:\journalfolder\

此示例将恢复可能没有完成的上一操作。

### 生成日志文件

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /V

如果指定了选项 `/V` 但未提供详细日志的文件路径，AzCopy 则将在默认位置中创建日志文件，默认位置为 `%SystemDrive%\Users\%username%\AppData\Local\Microsoft\Azure\AzCopy`。

或者，可以在自定义位置中创建日志文件：

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /V:C:\myfolder\azcopy1.log

请注意，如果在选项 `/V` 后指定了相对路径，例如 `/V:test/azcopy1.log`，则会在当前工作目录中名为 `test` 的子文件夹内创建详细日志。

### 指定要启动的并发操作的数量

选项 `/NC` 指定并发复制操作的数量。默认情况下，AzCopy 会启动一定数量的并发操作以提高数据传输吞吐量。对于表操作，并发操作的数量与所拥有的处理器数相等。对于 Blob 和文件操作，并发操作数等于所拥有的处理器数的 8 倍。如果正在低带宽网络中运行 AzCopy，则可为 /NC 指定较低的数量以避免由于资源争用所导致的故障。

### 针对 Azure 存储模拟器运行 AzCopy

可以针对 Blob 的 [Azure 存储模拟器](/documentation/articles/storage-use-emulator/)运行 AzCopy：

	AzCopy /Source:https://127.0.0.1:10000/myaccount/mycontainer/ /Dest:C:\myfolder /SourceKey:key /SourceType:Blob /S

和针对表：

	AzCopy /Source:https://127.0.0.1:10002/myaccount/mytable/ /Dest:C:\myfolder /SourceKey:key /SourceType:Table

##<a name="azcopy-parameters"></a> AzCopy 参数

以下描述了 AzCopy 的参数。还可以从命令行键入下列命令之一以获取如何使用 AzCopy 的帮助信息：

- 若要获取 AzCopy 的详细命令行帮助信息，请键入：`AzCopy /?`
- 若要获取任何 AzCopy 参数的详细帮助信息，请键入：`AzCopy /?:SourceKey`
- 若要获取命令行示例，请键入：`AzCopy /?:Samples`

### /Source:"source"

指定要从中复制数据的源。源可以是文件系统目录、blob 容器、blob 虚拟目录、存储文件共享、存储文件目录或 Azure 表。

**适用对象：**Blob、文件、表

### /Dest:"destination"

指定要复制到的目标。目标可以是文件系统目录、blob 容器、blob 虚拟目录、存储文件共享、存储文件目录或 Azure 表。

**适用对象：**Blob、文件、表

### /Pattern:"file-pattern"

指定文件模式，它指示要复制哪些文件。/Pattern 参数的行为是由源数据的位置以及是否存在递归模式选项决定的。递归模式是通过选项 /S 指定的。

如果指定的源是文件系统中的一个目录，则标准通配符将生效，并且会将该目录中的文件与提供的文件模式进行匹配。如果指定了选项 /S，则 AzCopy 还会将该目录下的任何子文件夹中的所有文件与指定模式进行匹配。

如果指定的源是一个 blob 容器或虚拟目录，则不会应用通配符。如果指定了选项 /S，则 AzCopy 会将指定的文件模式解释为 blob 前缀。如果未指定选项 /S，则 AzCopy 会将确切的 blob 名称与文件模式进行匹配。

如果指定的源是 Azure 文件共享，则必须指定确切的文件名（如 abc.txt）以复制单个文件，或指定选项 /S 以递归方式复制该共享中的所有文件。尝试同时指定文件模式和选项 /S 将导致错误。

当 /Source 是 blob 容器或 blob 虚拟目录时，AzCopy 使用区分大小写匹配，而在所有其他情况下则使用不区分大小写匹配。

未指定文件模式时使用的默认文件模式为 *.* （对于文件系统位置）或空前缀（对于 Azure 存储位置）。不支持指定多个文件模式。

**适用对象：**Blob、文件

### /DestKey:"storage-key"

指定目标资源的存储帐户密钥。

**适用对象：**Blob、文件、表

### /DestSAS:"sas-token"

指定对目标具有读写权限的共享访问签名 (SAS)（如果适用）。请将 SAS 用双引号括起来，因为它可能包含特殊的命令行字符。

如果目标资源是 blob 容器、文件共享或表，则可以指定此选项，后跟 SAS 令牌，或者可以将 SAS 指定为目标 blob 容器、文件共享或表 URI 的一部分，而不使用此选项。

如果源和目标都是 blob，则目标 blob 必须与源 blob 位于同一个存储帐户中。

**适用对象：**Blob、文件、表

### /SourceKey:"storage-key"

指定源资源的存储帐户密钥。

**适用对象：**Blob、文件、表

### /SourceSAS:"sas-token"

指定对源具有读取和列表权限的共享访问签名（如果适用）。请将 SAS 用双引号括起来，因为它可能包含特殊的命令行字符。

如果源资源是 blob 容器，并且既未提供密钥又未提供 SAS，则可以通过匿名访问读取 blob 容器。

如果源是文件共享或表，必须提供一个键或某一 SAS。

**适用对象：**Blob、文件、表

### /S

指定复制操作的递归模式。在递归模式下，AzCopy 将复制与指定的文件模式相匹配的所有 blob 或文件，包括子文件夹中的对象。

**适用对象：**Blob、文件

### /BlobType:"block" | "page" | "append"

指定目标 Blob 是块 Blob、页 Blob 还是追加 Blob。仅在要上传 Blob 时，此选项才适用。否则将会发生错误。如果目标是一个 Blob 并且未指定此选项，则默认情况下 AzCopy 将创建块 Blob。

**适用对象：**Blob

### /CheckMD5

计算已下载的数据的 MD5 哈希，并验证存储在 blob 或文件的 Content-MD5 属性中的 MD5 哈希是否与计算得到的哈希匹配。默认情况下，MD5 检查处于关闭状态，因此，必须指定此选项以在下载数据时执行 MD5 检查。

请注意，Azure 存储不保证为 blob 或文件所存储的 MD5 哈希是最新的。每当 blob 或文件被修改时，客户端都需要负责对 MD5 进行更新。

在将 Azure blob 或文件上传到服务后，AzCopy 始终会为其设置 Content-MD5 属性。

**适用对象：**Blob、文件

### /Snapshot

指示是否传输快照。只有当源是 blob 时，此选项才有效。

传输的 Blob 快照会按以下格式重命名：blob-name (snapshot-time).extension

默认情况下，不会复制快照。

**适用对象：**Blob

### /V:[verbose-log-file]

将详细的状态消息输出到日志文件中。

默认情况下，详细日志文件在 `%LocalAppData%\Microsoft\Azure\AzCopy` 中将被命名为 AzCopyVerbose.log。如果为此选项指定了现有的文件位置，详细日志则将追加到该文件中。

**适用对象：**Blob、文件、表

### /Z:[journal-file-folder]

指定用于恢复某一操作的日志文件文件夹。

AzCopy 始终支持对被中断的操作进行恢复。

如果未指定此选项，或者未指定其文件夹路径，AzCopy 则会在默认位置中创建日志文件，默认位置为 %LocalAppData%\\Microsoft\\Azure\\AzCopy。

每次向 AzCopy 发出命令时，它都会检查默认文件夹中是否存在日志文件，或者通过此选项指定的文件夹中是否存在日志文件。如果这两个位置中都不存在日志文件，AzCopy 则会将操作视为新操作并生成一个新的日志文件。

如果存在日志文件，AzCopy 则将检查输入的命令行是否与该日志文件中的命令行相匹配。如果两个命令行相匹配，AzCopy 则将恢复未完成的操作。如果它们不匹配，系统将提示用户是选择覆盖该日志文件以启动新操作，还是取消当前操作。

成功完成操作后，将删除该日志文件。

请注意，不支持通过由以前版本的 AzCopy 创建的日志文件来恢复操作。

**适用对象：**Blob、文件、表

### /@:"parameter-file"

指定包含参数的文件。AzCopy 会像处理在命令行上指定参数一样处理文件中的参数。

在响应文件中，可以在单个行上指定多个参数，也可以将每个参数指定在其单独的行上。请注意，单个参数不能跨多个行。

响应文件可包括以 # 符号开头的命令行。

可以指定多个响应文件。但请注意，AzCopy 不支持嵌套的响应文件。

**适用对象：**Blob、文件、表

### /Y

取消所有的 AzCopy 确认提示。

**适用对象：**Blob、文件、表

### /L

仅指定列出操作；不复制任何数据。

AzCopy 将使用此选项解释为在没有此选项 /L 的情况下，模拟运行命令行，并对复制的对象数量进行计数，可以同时指定选项 /V 以检查哪些对象将复制到详细日志中。

此选项的行为也由源数据的位置、是否存在递归模式选项 /S 以及文件模式选项 /Pattern 所决定。

使用此选项时，AzCopy 需要此源位置的列出和读取权限。

**适用对象：**Blob、文件

### /MT

将下载的文件的上次修改时间设置为与源 blob 或文件的上次修改时间相同。

**适用对象：**Blob、文件

### /XN

排除较新的源资源。如果源的上次修改时间同于或晚于目标，将不会复制该资源。

**适用对象：**Blob、文件

### /XO

排除较旧的源资源。如果源的上次修改时间同于或早于目标，将不会复制该资源。

**适用对象：**Blob、文件

### /A

仅上传设置了存档属性的文件。

**适用对象：**Blob、文件

### /IA:[RASHCNETOI]

仅上传设置了任何指定属性的文件。

可用的属性包括：

- R 表示只读文件
- A 表示用于存档的文件
- S 表示系统文件
- H 表示隐藏文件
- C 表示压缩文件
- N 表示普通文件
- E 表示加密文件
- T 表示临时文件
- O 表示脱机文件
- I 表示未编制索引的文件

**适用对象：**Blob、文件

### /XA:[RASHCNETOI]

排除设置了任何指定属性的文件。

可用的属性包括：

- R 表示只读文件
- A 表示用于存档的文件
- S 表示系统文件
- H 表示隐藏文件
- C 表示压缩文件
- N 表示普通文件
- E 表示加密文件
- T 表示临时文件
- O 表示脱机文件
- I 表示未编制索引的文件

**适用对象：**Blob、文件

### /Delimiter:"delimiter"

指示用于分隔 blob 名称中的虚拟目录的分隔符字符。

默认情况下，AzCopy 使用 / 作为分隔符字符。不过，AzCopy 支持使用任何常见字符（例如 @、# 或 %）作为分隔符。如果需要在命令行上包括这些特殊字符之一，请将文件名用双引号引起来。

此选项仅适用于下载 blob。

**适用对象：**Blob

### /NC:"number-of-concurrent-operations"

指定并发操作的数量。

默认情况下，AzCopy 会启动一定数量的并发操作以提高数据传输吞吐量。请注意，在低带宽环境中，大量的并发操作可能会压垮网络连接，并且会阻碍操作彻底完成。请根据实际可用的网络带宽限制并发操作。

并发操作的上限为 512。

**适用对象：**Blob、文件、表

### /SourceType:"Blob" | "Table"

指定 `source` 资源是本地开发环境中可用的一个 Blob，在存储模拟器中运行。

**适用对象：**Blob、表

### /DestType:"Blob" | "Table"

指定 `destination` 资源是本地开发环境中可用的一个 Blob，在存储模拟器中运行。

**适用对象：**Blob、表

### /PKRS:"key1#key2#key3#..."

对分区键范围进行拆分以便并行导出表数据，这可以提高导出操作的速度。

如果未指定此选项，AzCopy 将使用单个线程来导出表实体。例如，如果用户指定了 /PKRS:"aa#bb"，AzCopy 则将启动三个并发操作。

每个操作将导出三个分区键范围中的一个，如下所示：

  [first-partition-key, aa)

  [aa, bb)

  [bb, last-partition-key]

**适用对象：**表

### /SplitSize:"file-size"

指定已导出文件的拆分大小（单位为 MB），允许的最小值为 32。

如果未指定此选项，AzCopy 则会将表数据导出到单个文件。

如果将表数据导出到一个 blob，并且已导出文件的大小达到了 200 GB 的 blob 大小限制，AzCopy 则将拆分导出的文件，即使未指定此选项也是如此。

**适用对象：**表

### /EntityOperation:"InsertOrSkip" | "InsertOrMerge" | "InsertOrReplace"

指定表数据导入行为。

- InsertOrSkip - 跳过现有实体，或者插入新实体（如果它不存在于表中）。

- InsertOrMerge - 合并现有实体，或者插入新实体（如果它不存在于表中）。

- InsertOrReplace - 替换现有实体，或者插入新实体（如果它不存在于表中）。

**适用对象：**表

### /Manifest:"manifest-file"

指定表导出和导入操作的清单文件。

此选项在导出操作过程中是可选的，如果未指定此选项，AzCopy 将生成具有预定义名称的清单文件。

在导入操作期间用于定位数据文件，此选项是必要的。

**适用对象：**表

### /SyncCopy

指示是否要以同步方式在两个 Azure 存储终结点之间复制 blob 或文件。

AzCopy 默认情况下使用服务器端的异步复制。指定此选项以执行同步复制，可将 blob 或文件下载到本地内存，然后将其上传到 Azure 存储。

在 Blob 存储内复制文件、文件存储内复制文件或者从 Bolb 存储将文件复制到文件存储时，可使用此选项，反之亦然。

**适用对象：**Blob、文件

### /SetContentType:"content-type"

指定目标 blob 或文件的 MIME 内容类型。

默认情况下，AzCopy 将 blob 或文件的内容类型设置为 application/octet-stream。通过显式指定此选项的值，可设置所有 blob 或文件的内容类型。

如果指定此选项不带值，AzCopy 将根据 blob 或文件扩展名设置每个 blob 或文件的内容类型。

**适用对象：**Blob、文件

### /PayloadFormat:"JSON" | "CSV"

指定表导出数据文件的格式。

如果未指定此选项，则默认情况下，AzCopy 以 JSON 格式导出表数据文件。

**适用对象：**表

## 已知问题和最佳实践

### 限制复制数据时的并发写入

在使用 AzCopy 复制 blob 或文件时，请记住，在复制数据时其他应用程序可能正在修改该数据。如果可能，请确保要复制的数据在复制操作期间不会被修改。例如，当复制与 Azure 虚拟机关联的 VHD 时，请确保当前没有其他应用程序正在向该 VHD 进行写入。执行此操作的一个好方法是租用要复制的资源。另外，还可以先创建 VHD 的快照，然后复制该快照。

如果在复制 blob 或文件时无法阻止其他应用程序向其进行写入，请记住，在作业完成时，复制的资源可能不再与源资源完全相同。

### 在一台计算机上运行一个 AzCopy 实例。
AzCopy 旨在最大程度上利用计算机资源来加快数据传输，如果需要更多的并发操作，我们建议在一台计算机上只运行一个 AzCopy 实例并指定选项 `/NC`。有关详细信息，请在命令行中键入 `AzCopy /?:NC`。

### 当进行“使用适用于加密、哈希和签名的 FIPS 兼容算法”时，请启用适用于 AzCopy、与 FIPS 兼容的 MD5 算法。
默认情况下，在复制对象时，如有需要 AzCopy 启动 FIPS 兼容的 MD5 设置的某些安全需求时，AzCopy 则会使用 .NET MD5 实现来计算 MD5。

可以创建属性为 `AzureStorageUseV1MD5` 的 app.config 文件 `AzCopy.exe.config`，并将其与 AzCopy.exe 分开放。

	<?xml version="1.0" encoding="utf-8" ?>
	<configuration>
	  <appSettings>
	    <add key="AzureStorageUseV1MD5" value="false"/>
	  </appSettings>
	</configuration>

当属性“AzureStorageUseV1MD5”为 True（默认值）时，AzCopy 将使用 .NET MD5 实现；为 False 时，AzCopy 将使用兼容 FIPS 的 MD5 算法。

请注意，默认情况下，Windows 计算机上禁用 FIPS 兼容的算法，可以在运行的窗口中键入 secpol.msc 并在“安全设置”->“本地策略”->“安全选项”->“系统加密”处检查此开关：使用 FIPS 兼容算法来加密、哈希和签名。

## 后续步骤

有关 Azure 存储和 AzCopy 的更多信息，请参阅以下资源。

### Azure 存储文档：

- [Azure 存储简介](/documentation/articles/storage-introduction/)
- [如何通过 .NET 使用 Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)
- [如何通过 .NET 使用文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)
- [如何通过 .NET 使用表存储](/documentation/articles/storage-dotnet-how-to-use-tables/)
- [如何创建、管理或删除存储帐户](/documentation/articles/storage-create-storage-account/)

### Azure 存储博客文章：
- [Azure 存储数据移动库预览版简介](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/)
- [AzCopy：引入了同步复制和自定义内容类型](http://blogs.msdn.com/b/windowsazurestorage/archive/2015/01/13/azcopy-introducing-synchronous-copy-and-customized-content-type.aspx)
- [AzCopy：宣布公开发行支持表和文件的 AzCopy 3.0 增强预览版本 AzCopy 4.0](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/10/29/azcopy-announcing-general-availability-of-azcopy-3-0-plus-preview-release-of-azcopy-4-0-with-table-and-file-support.aspx)
- [AzCopy：针对大规模复制方案进行了优化](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/08/08/azcopy-2-5-release.aspx)
- [AzCopy：支持读取访问异地冗余存储](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/04/07/azcopy-support-for-read-access-geo-redundant-account.aspx)
- [AzCopy：使用可重新启动的模式和 SAS 令牌传输数据](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/09/07/azcopy-transfer-data-with-re-startable-mode-and-sas-token.aspx)
- [AzCopy：使用跨帐户复制 Blob](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/04/01/azcopy-using-cross-account-copy-blob.aspx)
- [AzCopy：为 Azure Blob 上传/下载文件](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/12/03/azcopy-uploading-downloading-files-for-windows-azure-blobs.aspx)

<!---HONumber=Mooncake_0313_2017-->