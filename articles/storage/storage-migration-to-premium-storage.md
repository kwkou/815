<properties
    pageTitle="迁移到 Azure 高级存储 | Azure"
    description="将现有的虚拟机迁移到 Azure 高级存储。高级存储为 Azure 虚拟机上运行的 I/O 密集型工作负荷提供高性能、低延迟的磁盘支持。"
    services="storage"
    documentationcenter="na"
    author="aungoo-msft"
    manager="tadb"
    editor="tysonn" />  

<tags
    ms.assetid="272250b3-fd4e-41d2-8e34-fd8cc341ec87"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/21/2016"
    wacn.date="01/24/2017"
    ms.author="yuemlu" />

# 迁移到 Azure 高级存储
## 概述
Azure 高级存储为运行 I/O 密集型工作负荷的虚拟机提供高性能、低延迟的磁盘支持。可以将应用程序的 VM 磁盘迁移到 Azure 高级存储，以充分利用这些磁盘的速度和性能。

本指南旨在帮助 Azure 高级存储的新用户更好地准备从当前系统到高级存储的平稳转换。本指南涉及此过程中的 3 个关键部分：

* [迁移到高级存储的计划](#plan-the-migration-to-premium-storage)
* [准备并将虚拟硬盘 (VHD) 复制到高级存储](#prepare-and-copy-virtual-hard-disks-VHDs-to-premium-storage)
* [使用高级存储创建 Azure 虚拟机](#create-azure-virtual-machine-using-premium-storage)

可将其他平台中的 VM 迁移到 Azure 高级存储，或将现有 Azure VM 从标准存储迁移到高级存储。本指南介绍了这两种方案的相关步骤。根据具体方案，执行相关部分中指定的步骤。

> [AZURE.NOTE]有关高级存储的功能概述和定价信息，可参阅高级存储：[适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)。建议将任何需要高 IOPS 的虚拟机磁盘迁移到 Azure 高级存储，以便应用程序实现最佳性能。如果磁盘不需要高 IOPS，可以通过在标准存储（将虚拟机磁盘数据存储在硬盘驱动器 (HDD) 上而不是 SSD 上）中对其进行维护来限制成本。


完成整个迁移过程可能需要在执行本指南中提供的步骤前后执行其他操作。示例包括配置虚拟网络/终结点，或在应用程序内部更改代码，后者可能需要停止应用程序一段时间。这些操作特定于每个应用程序，应根据本指南中的步骤完成操作，尽可能无缝地完全转换到高级存储。

## <a name="plan-the-migration-to-premium-storage"></a>迁移到高级存储的计划
本部分确保你已准备好执行本文中的迁移步骤，并帮助你选择最适合的 VM 和磁盘类型。

### 先决条件
* 需要 Azure 订阅。如果没有，可创建一个月的[试用](/pricing/1rmb-trial/)订阅，或者访问 [Azure 定价](/pricing/)了解更多选项。
* 需要 Azure PowerShell 模块才能执行 PowerShell cmdlet。有关安装点和安装说明，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。
* 如果计划使用在高级存储上运行的 Azure VM，则需使用支持高级存储的 VM。支持高级存储的 VM 上可同时使用标准和高级存储磁盘。在将来，会有更多的 VM 类型提供高级存储磁盘。有关所有可用 Azure VM 磁盘类型和大小的详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)和[云服务大小](/documentation/articles/cloud-services-sizes-specs/)。

### 注意事项
Azure VM 可附加多个高级存储磁盘，使应用程序可具有每个 VM 多达 64 TB 的存储空间。借助高级存储，应用程序对于每个 VM 可以实现 80,000 IOPS（每秒输入/输出操作数）和每秒 2000 MB 的磁盘吞吐量，并且读取操作的延迟非常低。多种 VM 和磁盘可供选择。本部分旨在帮助用户确定最适合工作负荷的选项。

#### VM 大小
[虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)中列出了 Azure VM 大小规范。查看适用于高级存储的虚拟机的性能特征并选择最适合工作负荷的 VM 大小。确保 VM 上有足够的带宽来驱动磁盘通信。

#### 磁盘大小
有 3 种类型的磁盘可用于 VM，每种磁盘都具有特定的 IOP 和吞吐率限制。根据应用程序在容量、性能、可伸缩性和峰值负载方面的需要为 VM 选择磁盘类型时，需要考虑这些限制。

| 高级存储磁盘类型 | P10 | P20 | P30 |
|:---:|:---:|:---:|:---:|
| 磁盘大小 |128 GB |512 GB |1024 GB (1 TB) |
| 每个磁盘的 IOPS |500 |2300 |5000 |
| 每个磁盘的吞吐量 |每秒 100 MB |每秒 150 MB |每秒 200 MB |

根据工作负荷，确定 VM 是否需要附加数据磁盘。可以将多个持久性数据磁盘附加到 VM。如有需要，可以跨磁盘条带化，以增加卷的容量与性能。（请参阅[此处](/documentation/articles/storage-premium-storage-performance/#disk-striping)，了解什么是磁盘条带化。） 如果你使用[存储空间][4]来条带化高级存储数据磁盘，应该以使用的每个磁盘一个列的方式来配置它。否则，条带化卷的整体性能可能会低于预期，因为磁盘之间的通信分配不平均。对于 Linux VM，可使用 *mdadm* 实用工具实现该目的。有关详细信息，请参阅文章[在 Linux 上配置软件 RAID](/documentation/articles/virtual-machines-linux-configure-raid/)。

#### 存储帐户的可伸缩性目标
除了 [Azure 存储可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)外，高级存储帐户还具有以下可伸缩性目标。如果应用程序需求超过了单个存储帐户的可伸缩性目标，则在生成应用程序时请让它使用多个存储帐户，并将数据分布在这些存储帐户中。

| 总帐户容量 | 本地冗余存储帐户的总带宽 |
|:--- |:--- |
| 磁盘容量：35 TB<br />快照容量：10 TB |入站 + 出站最高每秒 50 Gbps |

有关高级存储规范的详细信息，请查看[使用高级存储时的可伸缩性和性能目标](/documentation/articles/storage-premium-storage/#premium-storage-scalability-and-performance-targets)。

#### 磁盘缓存策略
默认情况下，所有高级数据磁盘的磁盘缓存策略都是“只读的”，所有附加到 VM 的高级操作系统都是“读写的”。为使应用程序的 IO 达到最佳性能，建议使用此配置设置。对于频繁写入或只写的磁盘（例如 SQL Server 日志文件），禁用磁盘缓存可获得更佳的应用程序性能。可以使用 [Azure 门户预览](https://portal.azure.cn)或 *Set-AzureDataDisk* cmdlet 的 *-HostCaching* 参数更新现有数据磁盘的缓存设置。

#### 位置
选择 Azure 高级存储可用的位置。如果 VM 与存储 VM 磁盘的存储帐户位于同一区域，则该 VM 比位于单独区域时的性能更优异。

#### 其他 Azure VM 配置设置
创建 Azure VM 时，需配置一些 VM 设置。请记住，在 VM 的生存期内，少数设置固定不变，而其他设置可在稍后修改或添加。请查看这些 Azure VM 配置设置，并确保这些设置都正确配置以满足工作负荷要求。

### 优化
若要了解使用 Azure 高级存储构建高性能应用程序的准则，请参阅[Azure 高级存储：旨在实现高性能](/documentation/articles/storage-premium-storage-performance/)。可将本指南与适用于应用程序所用技术的性能最佳实践结合使用。

## <a name="prepare-and-copy-virtual-hard-disks-VHDs-to-premium-storage"></a>准备并将虚拟硬盘 (VHD) 复制到高级存储
以下部分介绍了相关准则，帮助你通过 VM 准备 VHD 并将 VHD 复制到 Azure 存储空间。

* [方案 1：“我要将现有 Azure VM 迁移到 Azure 高级存储。”](#scenario1)
* [方案 2：“我要将其他平台中的 VM 迁移到 Azure 高级存储。”](#scenario2)

### 先决条件
若要准备迁移 VHD，用户需要：

* Azure 订阅和存储帐户，外加要将 VHD 复制到的存储帐户中的容器。请注意，目标存储帐户可以是标准或高级存储帐户，具体取决于具体需求。
* 用于通用化 VHD 的工具（如果计划从中创建多个 VM 实例）。例如，sysprep for Windows 或 virt-sysprep for Ubuntu。
* 用于将 VHD 文件上传到存储帐户的工具。请参阅[使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)或者使用 [Azure 存储资源管理器](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx)。本指南介绍使用 AzCopy 工具复制 VHD 的步骤。

> [AZURE.NOTE] 如果选择基于 AzCopy 的同步复制选项，为了获得最佳性能，请通过目标存储帐户所在区域中的 Azure VM 运行上述某个工具，进而复制 VHD。如果从其他区域中的 Azure VM 复制 VHD，性能可能会下降。
>
> 若要在带宽有限的情况下复制大量数据，可以考虑将硬盘驱动器传送到 Azure 数据中心，以便使用 Azure 导入/导出服务来传输数据。使用 Azure 导入/导出服务可以仅将数据复制到标准存储帐户。当数据传入标准存储帐户后，可以使用[复制 Blob API](https://msdn.microsoft.com/zh-cn/library/azure/dd894037.aspx) 或 AzCopy 将数据传输到高级存储帐户。
>
> 请注意，Azure 仅支持固定大小的 VHD 文件。不支持 VHDX 文件或动态 VHD。如果有动态 VHD，可以使用 [Convert-VHD](http://technet.microsoft.com/zh-cn/library/hh848454.aspx) cmdlet 将其转换为固定大小。

### <a name="scenario1"></a>方案 1：“我要将现有 Azure VM 迁移到 Azure 高级存储。”
如果要迁移现有的 Azure VM，请停止运行 VM，根据所需的 VHD 类型准备 VHD，然后使用 AzCopy 或 PowerShell 复制 VHD。

VM 必须完全关闭才能干净迁移。在迁移完成之前将会存在停机时间。

#### 步骤 1。准备 VHD 进行迁移
若要将现有 Azure Vm 迁移到高级存储，VHD 可能为：

* 通用的操作系统映像
* 唯一的操作系统磁盘
* 数据磁盘

下面将演练这三种 VHD 准备方案。

##### 使用通用的操作系统 VHD 创建多个 VM 实例
如果要上传将用于创建多个泛型 Azure 虚拟机实例的 VHD，必须先使用 sysprep 实用工具通用化 VHD。这适用于本地或云中的 VHD。Sysprep 将从 VHD 中删除计算机特定的所有信息。

>[AZURE.IMPORTANT] 通用化 VM 之前，请先创建其快照或进行备份。运行 sysprep 将停止并释放 VM 实例。按照以下步骤对 Windows OS VHD 运行 sysprep 命令。请注意，运行 Sysprep 命令时将要求关闭虚拟机。有关 Sysprep 的详细信息，请参阅 [Sysprep 概述](http://technet.microsoft.com/zh-cn/library/hh825209.aspx)或 [Sysprep 技术参考](http://technet.microsoft.com/zh-cn/library/cc766049.aspx)。

1. 以管理员身份打开“命令提示符”窗口。
2. 输入以下命令打开 Sysprep：

		%windir%\system32\sysprep\sysprep.exe

4. 在系统准备工具中，选择“进入系统全新体验(OOBE)”，选中“通用化”复选框，选中“关闭”，然后单击“确定”，如下图所示。这将通用化操作系统，并关闭系统。

	![][1]

对于 Ubuntu VM，使用 virt-sysprep 实现同一目的。有关更多详细信息，请参阅 [virt-sysprep](http://manpages.ubuntu.com/manpages/precise/man1/virt-sysprep.1.html)。有关其他 Linux 操作系统，另请参阅一些开放源代码 [Linux 服务器设置软件](http://www.cyberciti.biz/tips/server-provisioning-software.html)。

##### 使用唯一的操作系统 VHD 创建单个 VM 实例
如果拥有在 VM 上运行的需要计算机特定的数据的应用程序，请不要通用化 VHD。非通用化 VHD 可用于创建唯一的 Azure VM 实例。例如，如果 VHD 上有域控制器，则执行 sysprep 会使它像域控制器一样没有效率。检查 VM 上运行的应用程序，查看当前运行的 sysprep 对这些程序的影响，然后再通用化 VHD。

##### 注册数据磁盘 VHD
如果在 Azure 中有要迁移的数据磁盘，必须确保关闭使用这些数据磁盘的 VM。

请按照下述步骤将 VHD 复制到 Azure 高级存储，然后将其注册为预配的数据磁盘。

#### 步骤 2.创建 VHD 的目标
创建用于维护 VHD 的存储帐户。规划 VHD 存储位置时，应注意以下几点：

* 目标高级存储帐户。
* 存储帐户位置必须与最后阶段创建的支持高级存储的 Azure VM 相同。可以复制到新的存储帐户，也可以根据需求计划使用同一存储帐户。
* 为下一阶段复制并保存目标存储帐户的存储帐户密钥。

对于数据磁盘，可选择在标准存储帐户中保留一些数据磁盘（例如具有冷却存储功能的磁盘），但强烈建议用户迁移生产工作负荷的所有数据以使用高级存储。

#### <a name="copy-vhd-with-azcopy-or-powershell"></a>步骤 3.使用 AzCopy 或 PowerShell 复制 VHD
需查找容器路径和存储帐户密钥，处理其中某个选项。可依次访问 **Azure 门户** 和“存储”，查找容器路径和存储帐户密钥。容器 URL 将类似于“https://myaccount.blob.core.chinacloudapi.cn/mycontainer/”。

##### 选项 1：使用 AzCopy 复制 VHD（异步复制）
可使用 AzCopy 通过 Internet 轻松上传 VHD。根据 VHD 的大小，这可能需要时间。请记住，在使用此选项时，检查存储帐户传入/传出限制。有关详细信息，请参阅 [Azure 存储可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。

1. 从此处下载并安装 AzCopy：[最新版本的 AzCopy](http://aka.ms/downloadazcopy)
2. 打开 Azure PowerShell，并转到安装 AzCopy 的文件夹。
3. 使用以下命令从“Source”将 VHD 文件复制到“Destination”。

	
	AzCopy /Source: <source> /SourceKey: <source-account-key> /Dest: <destination> /DestKey: <dest-account-key> /BlobType:page /Pattern: <file-name>
	

    示例：

	
	AzCopy /Source:https://sourceaccount.blob.core.chinacloudapi.cn/mycontainer1 /SourceKey:key1 /Dest:https://destaccount.blob.core.chinacloudapi.cn/mycontainer2 /DestKey:key2 /Pattern:abc.vhd
	

    下面是 AzCopy 命令中使用的参数的说明：

   * **/Source: *&lt;source&gt;***： 包含 VHD 的文件夹或存储容器 URL 的位置。
   * **/SourceKey: *&lt;source-account-key&gt;***：源存储帐户的存储帐户密钥。
   * **/Dest: *&lt;destination&gt;***：要将 VHD 复制到的存储容器 URL。
   * **/DestKey: *&lt;dest-account-key&gt;***：目标存储帐户的存储帐户密钥。
   * **/Pattern: *&lt;file-name&gt;***：指定要复制的 VHD 文件名。

有关使用 AzCopy 工具的详细信息，请参阅[使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)。

##### 选项 2：使用 PowerShell 复制 VHD（同步复制）
还可以使用 PowerShell cmdlet Start-AzureStorageBlobCopy 复制 VHD 文件。在 Azure PowerShell 上使用以下命令复制 VHD。将 <> 中的值替换为源和目标存储帐户中的相应值。若要使用此命令，必须在目标存储帐户中有名为 vhds 的容器。如果该容器不存在，则应在运行此命令之前创建一个。


	$sourceBlobUri = <source-vhd-uri>

	$sourceContext = New-AzureStorageContext  –StorageAccountName <source-account> -StorageAccountKey <source-account-key>

	$destinationContext = New-AzureStorageContext  –StorageAccountName <dest-account> -StorageAccountKey <dest-account-key>

	Start-AzureStorageBlobCopy -srcUri $sourceBlobUri -SrcContext $sourceContext -DestContainer <dest-container> -DestBlob <dest-disk-name> -DestContext $destinationContext


示例：


	C:\PS> $sourceBlobUri = "https://sourceaccount.blob.core.chinacloudapi.cn/vhds/myvhd.vhd"

	C:\PS> $sourceContext = New-AzureStorageContext  –StorageAccountName "sourceaccount" -StorageAccountKey "J4zUI9T5b8gvHohkiRg"

	C:\PS> $destinationContext = New-AzureStorageContext  –StorageAccountName "destaccount" -StorageAccountKey "XZTmqSGKUYFSh7zB5"

	C:\PS> Start-AzureStorageBlobCopy -srcUri $sourceBlobUri -SrcContext $sourceContext -DestContainer "vhds" -DestBlob "myvhd.vhd" -DestContext $destinationContext


### <a name="scenario2"></a>方案 2：“我要将其他平台中的 VM 迁移到 Azure 高级存储。”
如果要将 VHD 从非 Azure 云存储迁移到 Azure，必须首先将 VHD 导出到本地目录中。获取就地存储 VHD 的本地目录的完整源路径，然后使用 AzCopy 将其上传到 Azure 存储。

#### 步骤 1。将 VHD 导出到本地目录
##### 从 AWS 复制 VHD
1. 如果要使用 AWS，请将 EC2 实例导出到 Amazon S3 存储桶中的 VHD。按照 Amazon 文档“导出 Amazon EC2 实例”中所述的步骤安装 Amazon EC2 命令行接口 (CLI) 工具，并运行 create-instance-export-task 命令将 EC2 实例导出到 VHD 文件。运行 **create-instance-export-task** 命令时，请务必对 DISK&#95;IMAGE&#95;FORMAT 变量使用 **VHD**。导出的 VHD 文件将保存在该过程中指定的 Amazon S3 存储桶中。


	aws ec2 create-instance-export-task --instance-id ID --target-environment TARGET\_ENVIRONMENT \\ --export-to-s3-task DiskImageFormat=DISK\_IMAGE\_FORMAT,ContainerFormat=ova,S3Bucket=BUCKET,S3Prefix=PREFIX


2. 从 S3 存储桶中下载 VHD 文件。选择 VHD 文件，然后单击“操作”>“下载”。

    ![][3]  


##### 从其他非 Azure 云复制 VHD
如果要将 VHD 从非 Azure 云存储迁移到 Azure，必须首先将 VHD 导出到本地目录中。复制存储 VHD 的本地目录的完整源路径。

##### 从本地复制 VHD
如果要从本地环境迁移 VHD，将需要 VHD 存储位置的完整源路径。该源路径可以是服务器位置或文件共享。

#### 步骤 2.创建 VHD 的目标
创建用于维护 VHD 的存储帐户。规划 VHD 存储位置时，应注意以下几点：

* 目标存储帐户可以是标准或高级存储，具体取决于应用程序需求。
* 存储帐户区域必须与最后阶段创建的支持高级存储的 Azure VM 相同。可以复制到新的存储帐户，也可以根据需求计划使用同一存储帐户。
* 为下一阶段复制并保存目标存储帐户的存储帐户密钥。

强烈建议用户迁移生产工作负荷的所有数据以使用高级存储。

#### 步骤 3。将 VHD 上传到 Azure 存储
现在，本地目录中已有 VHD，可使用 AzCopy 或 AzurePowerShell 将 .vhd 文件上传到 Azure 存储。可两个选项可用：

##### 选项 1：使用 Azure PowerShell Add-azurevhd 上传 .vhd 文件


	Add-AzureVhd [-Destination] <Uri> [-LocalFilePath] <FileInfo>


示例 < Uri > 可能是 ***“https://storagesample.blob.core.chinacloudapi.cn/mycontainer/blob1.vhd”*** 。示例 < FileInfo > 可能是 ***“C:\\path\\to\\upload.vhd”*** 。

##### 选项 2：使用 AzCopy 上传 .vhd 文件
可使用 AzCopy 通过 Internet 轻松上传 VHD。根据 VHD 的大小，这可能需要时间。请记住，在使用此选项时，检查存储帐户传入/传出限制。有关详细信息，请参阅 [Azure 存储可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。

1. 从此处下载并安装 AzCopy：[最新版本的 AzCopy](http://aka.ms/downloadazcopy)
2. 打开 Azure PowerShell，并转到安装 AzCopy 的文件夹。
3. 使用以下命令从“Source”将 VHD 文件复制到“Destination”。

		AzCopy /Source: <source> /SourceKey: <source-account-key> /Dest: <destination> /DestKey: <dest-account-key> /BlobType:page /Pattern: <file-name>
    示例：


    		AzCopy /Source:https://sourceaccount.blob.core.chinacloudapi.cn/mycontainer1 /SourceKey:key1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /DestKey:key2 /BlobType:page /Pattern:abc.vhd


    下面是 AzCopy 命令中使用的参数的说明：

   * **/Source: *&lt;source&gt;***：包含 VHD 的文件夹或存储容器 URL 的位置。
   * **/SourceKey: *&lt;source-account-key&gt;***：源存储帐户的存储帐户密钥。
   * **/Dest: *&lt;destination&gt;***：要将 VHD 复制到的存储容器 URL。
   * **/DestKey: *&lt;dest-account-key&gt;***：目标存储帐户的存储帐户密钥。
   * **/BlobType: page**：指定目标是页 Blob。
   * **/Pattern: *&lt;file-name&gt;***：指定要复制的 VHD 文件名。

有关使用 AzCopy 工具的详细信息，请参阅[使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)。

##### 用于上传 VHD 的其他选项
也可以使用以下方法之一将 VHD 上传到存储帐户：

- [Azure 存储复制 Blob API](https://msdn.microsoft.com/zh-CN/library/azure/dd894037.aspx)
* [Azure 存储资源管理器上传 Blob](https://azurestorageexplorer.codeplex.com/)
- [存储导入/导出服务 REST API 参考](https://docs.microsoft.com/en-us/rest/api/storageimportexport/)

>[AZURE.NOTE]如果预估上传时间大于 7 天，建议使用导入/导出服务。可根据数据大小和传输单位，利用 [DataTransferSpeedCalculator](https://github.com/Azure-Samples/storage-dotnet-import-export-job-management/blob/master/DataTransferSpeedCalculator.html) 预估时间。导入/导出可用于复制到标准存储帐户。需要使用 AzCopy 等工具从标准存储复制到高级存储帐户。


## <a name="create-azure-virtual-machine-using-premium-storage"></a>使用高级存储创建 Azure VM
将 VHD 上传或复制到所需的存储帐户后，请按照本部分中的说明将 VHD 注册为 OS 映像或 OS 磁盘（具体取决于方案），然后从中创建 VM 实例。在创建后，可以将数据磁盘 VHD 附加到 VM。本部分末尾提供了一个示例迁移脚本。此简单脚本不符合任意方案。可能需要更新该脚本以与特定方案相匹配。若要查看此脚本是否适用于用户方案，请参阅下面的[示例迁移脚本](#a-sample-migration-script)。

### 清单
1. 请在完成所有 VHD 磁盘复制前等待。
2. 确保高级存储在要迁移到的区域中可用。
3. 决定要使用的新 VM 系列。它应支持高级存储，大小应取决于区域中的可用性和用户需求。
4. 决定要使用的确切 VM 大小。VM 大小需要足够大以支持所拥有的数据磁盘数。例如，如果有 4 个数据磁盘，则 VM 必须具有 2 个或更多核心。此外，还应考虑处理能力、内存和网络带宽需求。
5. 在目标区域中创建高级存储帐户。这是将用于新 VM 的帐户。
6. 手边具备当前 VM 详细信息，包括磁盘和对应的 VHD blob 的列表。

让应用程序做好停机准备。为了执行干净的迁移，必须停止当前系统中的所有处理。只有这样才能使其处于一致状态，可以将该状态迁移到新的平台。停机持续时间将取决于要迁移的磁盘中的数据量。

> [AZURE.NOTE]如果要通过专用 VHD 磁盘创建 Azure Resource Manager VM，请参阅 [此模板](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd)，了解如何使用现有磁盘部署 Resource Manager VM。


### 注册 VHD
若要从 OS VHD 创建 VM 或将数据磁盘附加到新的 VM，必须先注册它们。请按以下步骤操作，具体取决于 VHD 方案。

#### 用于创建多个 Azure VM 实例的通用操作系统 VHD
将通用 OS 映像 VHD 上传到存储帐户后，将其注册为 **Azure VM 映像**，以便可从中创建一个或多个 VM 实例。使用以下 PowerShell cmdlet 将 VHD 注册为 Azure VM OS 映像。提供 VHD 已复制到的完整容器 URL。

	Add-AzureVMImage -ImageName "OSImageName" -MediaLocation "https://storageaccount.blob.core.chinacloudapi.cn/vhdcontainer/osimage.vhd" -OS Windows

复制并保存这个新的 Azure VM 映像的名称。在上面的示例中，它是 *OSImageName* 。

#### 用于创建单个 Azure VM 实例的唯一操作系统 VHD
将唯一的 OS VHD 上传到存储帐户后，将其注册为 **Azure OS 磁盘**，以便可从中创建 VM 实例。使用这些 PowerShell cmdlet 将 VHD 注册为 Azure OS 磁盘。提供 VHD 已复制到的完整容器 URL。

	Add-AzureDisk -DiskName "OSDisk" -MediaLocation "https://storageaccount.blob.core.chinacloudapi.cn/vhdcontainer/osdisk.vhd" -Label "My OS Disk" -OS "Windows"

复制并保存这个新的 Azure OS 磁盘的名称。在上面的示例中，它是 *OSDisk* 。

#### 要附加到 Azure VM 新实例的数据磁盘 VHD
将数据磁盘 VHD 上传到存储帐户后，将其注册为 Azure 数据磁盘，以便可以将它附加到新的 DS 系列 Azure VM 实例。

使用这些 PowerShell cmdlet 将 VHD 注册为 Azure 数据磁盘。提供 VHD 已复制到的完整容器 URL。

	Add-AzureDisk -DiskName "DataDisk" -MediaLocation "https://storageaccount.blob.core.chinacloudapi.cn/vhdcontainer/datadisk.vhd" -Label "My Data Disk"

复制并保存这个新的 Azure 数据磁盘的名称。在上面的示例中，它是 *DataDisk* 。

### 创建支持高级存储的 VM
注册 OS 映像或 OS 磁盘后，请创建新的 DS 系列 VM。将使用注册的操作系统映像或操作系统磁盘名称。从高级存储层选择 VM 类型。在以下示例中，我们将使用 *Standard\_DS2* VM 大小。

>[AZURE.NOTE] 更新磁盘大小，以确保它满足容量、性能要求和可用的 Azure 磁盘大小。

逐步执行以下 PowerShell cmdlet 创建新的 VM。首先，设置公共参数：

	$serviceName = "yourVM"
	$location = "location-name" (e.g., China East) 
	$vmSize ="Standard_DS2"
	$adminUser = "youradmin"
	$adminPassword = "yourpassword"
	$vmName ="yourVM"
	$vmSize = "Standard_DS2"

首先，创建要在其中托管新 VM 的云服务。

	New-AzureService -ServiceName $serviceName -Location $location

接下来，根据具体方案，从注册的 OS 映像或 OS 磁盘创建 Azure VM 实例。

#### 用于创建多个 Azure VM 实例的通用操作系统 VHD
使用注册的 **Azure OS 映像**创建一个或多个新的 DS 系列 Azure VM 实例。创建新 VM 时，在 VM 配置中指定此 OS 映像名称，如下所示。

	$OSImage = Get-AzureVMImage –ImageName "OSImageName"

	$vm = New-AzureVMConfig -Name $vmName –InstanceSize $vmSize -ImageName $OSImage.ImageName

	Add-AzureProvisioningConfig -Windows –AdminUserName $adminUser -Password $adminPassword –VM $vm

	New-AzureVM -ServiceName $serviceName -VM $vm

#### 用于创建单个 Azure VM 实例的唯一操作系统 VHD
使用注册的 **Azure OS 磁盘**创建新的 DS 系列 Azure VM 实例。创建新的 VM 时，在 VM 配置中指定此 OS 磁盘名称，如下所示。

	$OSDisk = Get-AzureDisk –DiskName "OSDisk"

	$vm = New-AzureVMConfig -Name $vmName -InstanceSize $vmSize -DiskName $OSDisk.DiskName

	New-AzureVM -ServiceName $serviceName –VM $vm

指定其他 Azure VM 信息，例如云服务、区域、存储帐户、可用性集和缓存策略。请注意，VM 实例必须与关联的操作系统或数据磁盘同位，因此，所选的云服务、区域和存储帐户必须都与这些磁盘的基础 VHD 位于同一位置中。

### 附加数据磁盘
最后，如果你已注册数据磁盘 VHD，请将它们附加到支持高级存储的新 Azure VM。

使用以下 PowerShell cmdlet 将数据磁盘附加到新的 VM，并指定缓存策略。在以下示例中，缓存策略设为 *ReadOnly*。

	$vm = Get-AzureVM -ServiceName $serviceName -Name $vmName

	Add-AzureDataDisk -ImportFrom -DiskName "DataDisk" -LUN 0 –HostCaching ReadOnly –VM $vm

	Update-AzureVM  -VM $vm



> [AZURE.NOTE]本指南可能未涵盖支持应用程序所要执行的其他步骤。

### 检查和计划备份
启动并运行新的 VM 后，使用与原始 VM 相同的登录 ID 和密码访问它，并验证所有功能是否按预期正常工作。所有设置（包括带区卷）都会出现在新的 VM 中。

最后一步是根据应用程序的需求规划新 VM 的备份和维护计划。

### <a name="a-sample-migration-script"></a>示例迁移脚本
如果有多个 VM 要迁移，通过 PowerShell 脚本自动执行会很有帮助。下面是自动执行 VM 迁移的示例脚本。请注意，以下脚本仅为示例并对当前 VM 磁盘做了几点假设。可能需要更新该脚本以与特定方案相匹配。

假设是：

* 用户正在创建经典 Azure VM。
* 用户的源 OS 磁盘和源数据磁盘位于同一存储帐户和同一容器中。如果 OS 磁盘和数据磁盘不在同一位置，可使用 AzCopy 或 Azure PowerShell 对存储帐户和容器复制 VHD。请参阅上一步：[使用 AzCopy 或 PowerShell 复制 VHD](#copy-vhd-with-azcopy-or-powershell)。还可以编辑此脚本来满足方案，但建议使用 AzCopy 或 PowerShell，因为后两种更便捷、速度更快。

下面提供的是自动化脚本。将文本替换为用户信息，并更新脚本以匹配特定方案。

> [AZURE.NOTE]使用现有脚本不会保留源 VM 的网络配置。将需要在迁移后的 VM 上重新配置网络设置。



	    <#
	    .Synopsis
	    This script is provided as an EXAMPLE to show how to migrate a VM from a standard storage account to a premium storage account. You can customize it according to your specific requirements.

	    .Description
	    The script will copy the vhds (page blobs) of the source VM to the new storage account.
	    And then it will create a new VM from these copied vhds based on the inputs that you specified for the new VM.
	    You can modify the script to satisfy your specific requirement, but please be aware of the items specified
	    in the Terms of Use section.

	    .Terms of Use
	    Copyright © 2015 Microsoft Corporation.  All rights reserved.

	    THIS CODE AND ANY ASSOCIATED INFORMATION ARE PROVIDED “AS IS” WITHOUT WARRANTY OF ANY KIND,
	    EITHER EXPRESSED OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE IMPLIED WARRANTIES OF MERCHANTABILITY
	    AND/OR FITNESS FOR A PARTICULAR PURPOSE. THE ENTIRE RISK OF USE, INABILITY TO USE, OR
	    RESULTS FROM THE USE OF THIS CODE REMAINS WITH THE USER.

	    .Example (Save this script as Migrate-AzureVM.ps1)

	    .\Migrate-AzureVM.ps1 -SourceServiceName CurrentServiceName -SourceVMName CurrentVMName –DestStorageAccount newpremiumstorageaccount -DestServiceName NewServiceName -DestVMName NewDSVMName -DestVMSize "Standard_DS2" –Location “China East”

	    .Link
	    To find more information about how to set up Azure PowerShell, refer to the following links.
	    https://www.azure.cn/documentation/articles/powershell-install-configure/
	    https://www.azure.cn/documentation/articles/storage-powershell-guide-full/
	    http://azure.microsoft.com/blog/2014/10/22/migrate-azure-virtual-machines-between-storage-accounts/

	    #>

	    Param(
	    # the cloud service name of the VM.
	    [Parameter(Mandatory = $true)]
	    [string] $SourceServiceName,

	    # The VM name to copy.
	    [Parameter(Mandatory = $true)]
	    [String] $SourceVMName,

	    # The destination storage account name.
	    [Parameter(Mandatory = $true)]
	    [String] $DestStorageAccount,

	    # The destination cloud service name
	    [Parameter(Mandatory = $true)]
	    [String] $DestServiceName,

	    # the destination vm name
	    [Parameter(Mandatory = $true)]
	    [String] $DestVMName,

	    # the destination vm size
	    [Parameter(Mandatory = $true)]
	    [String] $DestVMSize,

	    # the location of destination VM.
	    [Parameter(Mandatory = $true)]
	    [string] $Location,

	    # whether or not to copy the os disk, the default is only copy data disks
	    [Parameter(Mandatory = $false)]
	    [Bool] $DataDiskOnly = $true,

	    # how frequently to report the copy status in sceconds
	    [Parameter(Mandatory = $false)]
	    [Int32] $CopyStatusReportInterval = 15,

	    # the name suffix to add to new created disks to avoid conflict with source disk names
	    [Parameter(Mandatory = $false)]
	    [String]$DiskNameSuffix = "-prem"

	    ) #end param

	    #######################################################################
	    #  Verify Azure PowerShell module and version
	    #######################################################################

	    #import the Azure PowerShell module
	    Write-Host "`n[WORKITEM] - Importing Azure PowerShell module" -ForegroundColor Yellow
	    $azureModule = Import-Module Azure -PassThru

	    if ($azureModule -ne $null)
	    {
	        Write-Host "`tSuccess" -ForegroundColor Green
	    }
	    else
	    {
	        #show module not found interaction and bail out
	        Write-Host "[ERROR] - PowerShell module not found. Exiting." -ForegroundColor Red
	        Exit
	    }


	    #Check the Azure PowerShell module version
	    Write-Host "`n[WORKITEM] - Checking Azure PowerShell module verion" -ForegroundColor Yellow
	    If ($azureModule.Version -ge (New-Object System.Version -ArgumentList "0.8.14"))
	    {
	        Write-Host "`tSuccess" -ForegroundColor Green
	    }
	    Else
	    {
	        Write-Host "[ERROR] - Azure PowerShell module must be version 0.8.14 or higher. Exiting." -ForegroundColor Red
	        Exit
	    }

	    #Check if there is an azure subscription set up in PowerShell
	    Write-Host "`n[WORKITEM] - Checking Azure Subscription" -ForegroundColor Yellow
	    $currentSubs = Get-AzureSubscription -Current
	    if ($currentSubs -ne $null)
	    {
	        Write-Host "`tSuccess" -ForegroundColor Green
	        Write-Host "`tYour current azure subscription in PowerShell is $($currentSubs.SubscriptionName)." -ForegroundColor Green
	    }
	    else
	    {
	        Write-Host "[ERROR] - There is no valid azure subscription found in PowerShell. Please refer to this article https://www.azure.cn/documentation/articles/powershell-install-configure/ to connect an azure subscription. Exiting." -ForegroundColor Red
	        Exit
	    }


	    #######################################################################
	    #  Check if the VM is shut down
	    #  Stopping the VM is a required step so that the file system is consistent when you do the copy operation.
	    #  Azure does not support live migration at this time..
	    #######################################################################

	    if (($sourceVM = Get-AzureVM –ServiceName $SourceServiceName –Name $SourceVMName) -eq $null)
	    {
	        Write-Host "[ERROR] - The source VM doesn't exist in the current subscription. Exiting." -ForegroundColor Red
	        Exit
	    }

	    # check if VM is shut down
	    if ( $sourceVM.Status -notmatch "Stopped" )
	    {
	        Write-Host "[Warning] - Stopping the VM is a required step so that the file system is consistent when you do the copy operation. Azure does not support live migration at this time. If you’d like to create a VM from a generalized image, sys-prep the Virtual Machine before stopping it." -ForegroundColor Yellow
	        $ContinueAnswer = Read-Host "`n`tDo you wish to stop $SourceVMName now? Input 'N' if you want to shut down the vm mannually and come back later.(Y/N)"
	        If ($ContinueAnswer -ne "Y") { Write-Host "`n Exiting." -ForegroundColor Red;Exit }
	        $sourceVM | Stop-AzureVM

	        # wait until the VM is shut down
	        $VMStatus = (Get-AzureVM –ServiceName $SourceServiceName –Name $vmName).Status
	        while ($VMStatus -notmatch "Stopped")
	        {
	            Write-Host "`n[Status] - Waiting VM $vmName to shut down" -ForegroundColor Green
	            Sleep -Seconds 5
	            $VMStatus = (Get-AzureVM –ServiceName $SourceServiceName –Name $vmName).Status
	        }
	    }

	    # exporting the sourve vm to a configuration file, you can restore the original VM by importing this config file
	    # see more information for Import-AzureVM
	    $workingDir = (Get-Location).Path
	    $vmConfigurationPath = $env:HOMEPATH + "\VM-" + $SourceVMName + ".xml"
	    Write-Host "`n[WORKITEM] - Exporting VM configuration to $vmConfigurationPath" -ForegroundColor Yellow
	    $exportRe = $sourceVM | Export-AzureVM -Path $vmConfigurationPath


	    #######################################################################
	    #  Copy the vhds of the source vm
	    #  You can choose to copy all disks including os and data disks by specifying the
	    #  parameter -DataDiskOnly to be $false. The default is to copy only data disk vhds
	    #  and the new VM will boot from the original os disk.
	    #######################################################################

	    $sourceOSDisk = $sourceVM.VM.OSVirtualHardDisk
	    $sourceDataDisks = $sourceVM.VM.DataVirtualHardDisks

	    # Get source storage account information, not considering the data disks and os disks are in different accounts
	    $sourceStorageAccountName = $sourceOSDisk.MediaLink.Host -split "\." | select -First 1
	    $sourceStorageKey = (Get-AzureStorageKey -StorageAccountName $sourceStorageAccountName).Primary
	    $sourceContext = New-AzureStorageContext –StorageAccountName $sourceStorageAccountName -StorageAccountKey $sourceStorageKey

	    # Create destination context
	    $destStorageKey = (Get-AzureStorageKey -StorageAccountName $DestStorageAccount).Primary
	    $destContext = New-AzureStorageContext –StorageAccountName $DestStorageAccount -StorageAccountKey $destStorageKey

	    # Create a container of vhds if it doesn't exist
	    if ((Get-AzureStorageContainer -Context $destContext -Name vhds -ErrorAction SilentlyContinue) -eq $null)
	    {
	        Write-Host "`n[WORKITEM] - Creating a container vhds in the destination storage account." -ForegroundColor Yellow
	        New-AzureStorageContainer -Context $destContext -Name vhds
	    }


	    $allDisksToCopy = $sourceDataDisks
	    # check if need to copy os disk
	    $sourceOSVHD = $sourceOSDisk.MediaLink.Segments[2]
	    if ($DataDiskOnly)
	    {
	        # copy data disks only, this option requires deleting the source VM so that dest VM can boot
	        # from the same vhd blob.
	        $ContinueAnswer = Read-Host "`n`t[Warning] You chose to copy data disks only. Moving VM requires removing the original VM (the disks and backing vhd files will NOT be deleted) so that the new VM can boot from the same vhd. This is an irreversible action. Do you wish to proceed right now? (Y/N)"
	        If ($ContinueAnswer -ne "Y") { Write-Host "`n Exiting." -ForegroundColor Red;Exit }
	        $destOSVHD = Get-AzureStorageBlob -Blob $sourceOSVHD -Container vhds -Context $sourceContext
	        Write-Host "`n[WORKITEM] - Removing the original VM (the vhd files are NOT deleted)." -ForegroundColor Yellow
	        Remove-AzureVM -Name $SourceVMName -ServiceName $SourceServiceName

	        Write-Host "`n[WORKITEM] - Waiting utill the OS disk is released by source VM. This may take up to several minutes."
	        $diskAttachedTo = (Get-AzureDisk -DiskName $sourceOSDisk.DiskName).AttachedTo
	        while ($diskAttachedTo -ne $null)
	        {
	            Start-Sleep -Seconds 10
	            $diskAttachedTo = (Get-AzureDisk -DiskName $sourceOSDisk.DiskName).AttachedTo
	        }

	    }
	    else
	    {
	        # copy the os disk vhd
	        Write-Host "`n[WORKITEM] - Starting copying os disk $($disk.DiskName) at $(get-date)." -ForegroundColor Yellow
	        $allDisksToCopy += @($sourceOSDisk)
	        $targetBlob = Start-AzureStorageBlobCopy -SrcContainer vhds -SrcBlob $sourceOSVHD -DestContainer vhds -DestBlob $sourceOSVHD -Context $sourceContext -DestContext $destContext -Force
	        $destOSVHD = $targetBlob
	    }


	    # Copy all data disk vhds
	    # Start all async copy requests in parallel.
	    foreach($disk in $sourceDataDisks)
	    {
	        $blobName = $disk.MediaLink.Segments[2]
	        # copy all data disks
	        Write-Host "`n[WORKITEM] - Starting copying data disk $($disk.DiskName) at $(get-date)." -ForegroundColor Yellow
	        $targetBlob = Start-AzureStorageBlobCopy -SrcContainer vhds -SrcBlob $blobName -DestContainer vhds -DestBlob $blobName -Context $sourceContext -DestContext $destContext -Force
	        # update the media link to point to the target blob link
	        $disk.MediaLink = $targetBlob.ICloudBlob.Uri.AbsoluteUri
	    }

	    # Wait until all vhd files are copied.
	    $diskComplete = @()
	    do
	    {
	        Write-Host "`n[WORKITEM] - Waiting for all disk copy to complete. Checking status every $CopyStatusReportInterval seconds." -ForegroundColor Yellow
	        # check status every 30 seconds
	        Sleep -Seconds $CopyStatusReportInterval
	        foreach ( $disk in $allDisksToCopy)
	        {
	            if ($diskComplete -contains $disk)
	            {
	                Continue
	            }
	            $blobName = $disk.MediaLink.Segments[2]
	            $copyState = Get-AzureStorageBlobCopyState -Blob $blobName -Container vhds -Context $destContext
	            if ($copyState.Status -eq "Success")
	            {
	                Write-Host "`n[Status] - Success for disk copy $($disk.DiskName) at $($copyState.CompletionTime)" -ForegroundColor Green
	                $diskComplete += $disk
	            }
	            else
	            {
	                if ($copyState.TotalBytes -gt 0)
	                {
	                    $percent = ($copyState.BytesCopied / $copyState.TotalBytes) * 100
	                    Write-Host "`n[Status] - $('{0:N2}' -f $percent)% Complete for disk copy $($disk.DiskName)" -ForegroundColor Green
	                }
	            }
	        }
	    }
	    while($diskComplete.Count -lt $allDisksToCopy.Count)

	    #######################################################################
	    #  Create a new vm
	    #  the new VM can be created from the copied disks or the original os disk.
	    #  You can ddd your own logic here to satisfy your specific requirements of the vm.
	    #######################################################################

	    # Create a VM from the existing os disk
	    if ($DataDiskOnly)
	    {
	        $vm = New-AzureVMConfig -Name $DestVMName -InstanceSize $DestVMSize -DiskName $sourceOSDisk.DiskName
	    }
	    else
	    {
	        $newOSDisk = Add-AzureDisk -OS $sourceOSDisk.OS -DiskName ($sourceOSDisk.DiskName + $DiskNameSuffix) -MediaLocation $destOSVHD.ICloudBlob.Uri.AbsoluteUri
	        $vm = New-AzureVMConfig -Name $DestVMName -InstanceSize $DestVMSize -DiskName $newOSDisk.DiskName
	    }
	    # Attached the copied data disks to the new VM
	    foreach ($dataDisk in $sourceDataDisks)
	    {
	        # add -DiskLabel $dataDisk.DiskLabel if there are labels for disks of the source vm
	        $diskLabel = "drive" + $dataDisk.Lun
	        $vm | Add-AzureDataDisk -ImportFrom -DiskLabel $diskLabel -LUN $dataDisk.Lun -MediaLocation $dataDisk.MediaLink
	    }

	    # Edit this if you want to add more custimization to the new VM
	    # $vm | Add-AzureEndpoint -Protocol tcp -LocalPort 443 -PublicPort 443 -Name 'HTTPs'
	    # $vm | Set-AzureSubnet "PubSubnet","PrivSubnet"

	    New-AzureVM -ServiceName $DestServiceName -VMs $vm -Location $Location


#### <a name="optimization"></a>优化
可以对当前 VM 配置进行专门自定义，使其很好地适用于标准磁盘。例如，通过使用带区卷中的多个磁盘来提高性能。例如，无需在高级存储上单独使用 4 个磁盘，可通过单个磁盘优化成本。此类优化需要根据具体情况进行处理，并且需要在迁移后执行自定义步骤。另请注意，此过程可能并不适用于依赖设置中定义的磁盘布局的数据库和应用程序。

##### 准备工作
1. 按前面部分中所述完成简单迁移。在迁移后将在新的 VM 上执行优化。
2. 定义优化的配置所需的新磁盘大小。
3. 确定当前磁盘/卷到新磁盘规范的映射。

##### 执行步骤
1. 在高级存储 VM 上创建具有合适大小的新磁盘。
2. 登录到该 VM 并将当前卷中的数据复制到映射到该卷的新磁盘。对需要映射到新磁盘的所有当前卷执行此操作。
3. 接下来，更改应用程序设置以切换到新磁盘，并分离旧卷。

如果希望调整应用程序以提升磁盘性能，请参阅 [优化应用程序性能](/documentation/articles/storage-premium-storage-performance/#optimizing-application-performance)。

### 应用程序迁移
数据库和其他复杂的应用程序可能需要执行应用程序提供程序定义的特殊步骤以进行迁移。请参阅各自的应用程序文档。例如，通常情况下，可以通过备份和还原来迁移数据库。

## 后续步骤
有关虚拟机迁移的特定方案，请参阅以下资源：

- [在存储帐户之间迁移 Azure 虚拟机](https://azure.microsoft.com/blog/2014/10/22/migrate-azure-virtual-machines-between-storage-accounts/)
- [创建 Windows Server VHD 并将其上传到 Azure。](/documentation/articles/virtual-machines-windows-classic-createupload-vhd/)
- [创建并上传包含 Linux 操作系统的虚拟硬盘](/documentation/articles/virtual-machines-linux-classic-create-upload-vhd/)
- [将虚拟机从 Amazon AWS 迁移到 Azure](http://channel9.msdn.com/Series/Migrating-Virtual-Machines-from-Amazon-AWS-to-Microsoft-Azure)  

另请参阅以下资源，深入了解 Azure 存储和 Azure 虚拟机：

- [Azure 存储](/documentation/services/storage/)
- [Azure 虚拟机](/documentation/services/virtual-machines/)
- [高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)

[1]: ./media/storage-migration-to-premium-storage/migration-to-premium-storage-1.png
[2]: ./media/storage-migration-to-premium-storage/migration-to-premium-storage-2.png
[3]: ./media/storage-migration-to-premium-storage/migration-to-premium-storage-3.png
[4]: http://technet.microsoft.com/zh-cn/library/hh831739.aspx

<!---HONumber=Mooncake_1128_2016-->