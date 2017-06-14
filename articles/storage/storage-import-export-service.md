<properties
    pageTitle="使用 Azure 导入/导出将数据传入/传出 Blob 存储 | Azure"
    description="了解如何在 Azure 门户中创建导入和导出作业，以便将数据传入/传出到 Blob 存储。"
    author="muralikk"
    manager="syadav"
    editor="tysonn"
    services="storage"
    documentationcenter="" />
<tags
    ms.assetid="668f53f2-f5a4-48b5-9369-88ec5ea05eb5"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/17/2017"
    wacn.date="05/31/2017"
    ms.author="muralikk"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="3677f8b953cd519e5b1fe765118d17e291d79a8f"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="use-the-azure-importexport-service-to-transfer-data-to-blob-storage"></a>使用 Azure 导入/导出服务可将数据传输到 Blob 存储中
使用 Azure 导入/导出服务，可以将硬盘驱动器寄送到 Azure 数据中心，从而安全地将大量数据传输到 Azure Blob 存储。 你还可以使用此服务将数据从 Azure Blob 存储传输到硬盘驱动器，然后再寄送到你的本地站点。 如果需要在本地站点和 Azure 之间传输数 TB 的数据，而由于带宽限制或网络成本过高，通过网络上载或下载数据不可行，在这种情况下，则可以使用此服务。

此服务要求对硬盘驱动器进行 BitLocker 加密以确保数据的安全性。此服务支持中国北部和中国东部的经典和 Azure Resource Manager 存储帐户（标准层和冷层）。必须将硬盘驱动器寄送到本文后面指定的受支持的位置。

本文将详细介绍 Azure 导入/导出服务，以及如何通过寄送驱动器将数据复制到 Azure Blob 存储以及从该存储复制数据。

## <a name="when-should-i-use-the-azure-importexport-service"></a>应该在什么时候使用 Azure 导入/导出服务？
如果通过网络上载或下载数据速度太慢，或者获取额外的网络带宽因成本过高而受到限制，则可考虑使用 Azure 导入/导出服务。

下面这样的场景可以使用此服务：

* 将数据迁移到云：将大量数据快速且经济高效地转移到 Azure。
* 内容分发：将数据快速发送到客户站点。
* 备份：将本地数据备份后存储在 Azure Blob 存储中。
* 数据恢复：恢复存储在 Blob 存储中的大量数据，然后将其递送到本地位置。

## <a name="prerequisites"></a><a name="pre-requisites"></a>先决条件
本部分列出了使用此服务需要满足的先决条件。 请在寄送驱动器之前仔细查看这些先决条件。

### <a name="storage-account"></a>存储帐户
必须拥有 Azure 订阅以及一个或多个存储帐户才能使用导入/导出服务。 每个作业只能用于将数据传输到一个存储帐户或者从一个存储帐户传输数据。 换言之，一个导入/导出作业不能跨多个存储帐户。 有关创建新存储帐户的信息，请参阅[如何创建存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)。

### <a name="blob-types"></a>Blob 类型
可使用 Azure 导入/导出服务将数据复制到块 blob 或页 blob。 与之相反，只能使用此服务从 Azure 存储中导出块 blob、页 blob 或追加 blob。

### <a name="job"></a>作业
若要开始从 Blob 存储进行导入或导出的过程，你首先要创建一个“作业”。 作业可以是“导入作业”或“导出作业”：

* 需要将本地数据传输到 Azure 存储帐户中的 Blob 时，可创建导入作业。
* 在你想要将当前作为 Blob 存储于你的存储帐户中的数据传输到运送给你的硬盘驱动器时创建导出作业。

创建作业时，需通知导入/导出服务：你要将一个或多个硬盘驱动器运送到 Azure 数据中心。

* 对于导入作业，你需要寄送包含数据的硬盘驱动器。
* 对于导出作业，你需要寄送空硬盘驱动器。
* 每个作业最多可以寄送 10 个硬盘驱动器。

可以使用[Azure 门户](https://portal.azure.cn/)或 [Azure 存储导入/导出 REST API](https://docs.microsoft.com/zh-cn/rest/api/storageimportexport/) 创建导入或导出作业。

### <a name="waimportexport-tool"></a>WAImportExport 工具
创建 **导入** 作业的第一步是准备需要通过寄送进行数据导入的驱动器。 若要准备驱动器，必须将其连接到本地服务器，然后在本地服务器上运行 WAImportExport 工具。 使用此 WAImportExport 工具可以方便地将数据复制到驱动器、使用 BitLocker 加密驱动器上的数据，以及生成驱动器日志文件。

日志文件存储有关作业和驱动器的基本信息，例如驱动器序列号和存储帐户名称。此日志文件不存储在该驱动器上。它在导入作业创建期间使用。本文稍后将提供有关创建作业的分步详细信息。

WAImportExport 工具仅兼容 64 位 Windows 操作系统。请参阅[操作系统](#operating-system)部分以了解受支持的特定 OS 版本。

下载最新版本的 [WAImportExport 工具](http://download.microsoft.com/download/3/6/B/36BFF22A-91C3-4DFC-8717-7567D37D64C5/WAImportExport.zip)。有关使用 WAImportExport 工具的详细信息，请参阅 [使用 WAImportExport 工具](/documentation/articles/storage-import-export-tool-how-to/)。

>[AZURE.NOTE]
>以前的版本：可以[下载 WAImportExpot V1](http://download.microsoft.com/download/0/C/D/0CD6ABA7-024F-4202-91A0-CE2656DCE413/WaImportExportV1.zip) 版本的工具，并参考 [WAImportExpot V1 使用指南](/documentation/articles/storage-import-export-tool-how-to-v1/)。 WAImportExpot V1 版本的工具支持 **在已将数据预先写入磁盘的情况下准备磁盘**。 如果唯一可用的密钥是 SAS 密钥，则也需要 WAImportExpot V1 工具。



### <a name="hard-disk-drives"></a>硬盘驱动器
只支持将 2.5 英寸 SSD, 2.5/3.5 英寸 SATA II/III 或 SAS 内部硬盘驱动器用于导入/导出服务。可以使用容量最高为 10TB 的硬盘驱动器。对于导入作业，将处理驱动器上的第一个数据卷。该数据卷必须使用 NTFS 进行格式化。将数据复制到硬盘驱动器时，可以使用 2.5 英寸 SSD 或者 2.5/3.5 英寸 SATA II/III 或 SAS 连接器直接连接该驱动器，也可以使用外部 2.5 英寸 SSD 或者 2.5/3.5 英寸 SATA II/III 或 SAS USB 适配器进行外部连接。不支持单硬盘阵列。


> [AZURE.IMPORTANT] 此服务不支持内置 USB 适配器附带的外部硬盘驱动器。此外，不能使用外部 HDD 包装内的磁盘；请勿寄送外部 HDD。


### <a name="encryption"></a>加密
必须使用 BitLocker 驱动器加密对驱动器上的数据进行加密。这将在运送过程中保护你的数据。

对于导入作业，可以通过两种方式进行加密。第一种方式是在运行 WAImportExport 工具准备驱动器时，在 dataset CSV 文件中指定。第二种方式是在准备驱动器期间，在驱动器上手动启用 BitLocker 加密并在运行 WAImportExport 工具命令行时，在 driveset CSV 文件中指定加密密钥。

对于导出作业，在将你的数据复制到驱动器以后，此服务会使用 BitLocker 加密驱动器，然后再将驱动器寄回给你。加密密钥将通过 Azure 门户提供。

### <a name="operating-system"></a>操作系统
在将驱动器寄送到 Azure 之前，可以使用下述 64 位操作系统之一通过 WAImportExport 工具准备硬盘驱动器：

Windows 7 Enterprise、Windows 7 Ultimate、Windows 8 Pro、Windows 8 Enterprise、Windows 8.1 Pro、Windows 8.1 Enterprise、Windows 10<sup>1</sup>、Windows Server 2008 R2、Windows Server 2012、Windows Server 2012 R2。所有这些操作系统都支持 BitLocker 驱动器加密。

### <a name="locations"></a>位置
Azure 导入/导出服务支持将数据复制到 Azure 存储帐户，以及从后者进行复制。可以将硬盘驱动器寄送到以下位置。如果存储帐户所在的 Azure 位置未在此处指定，则使用 Azure 门户或导入/导出 REST API 创建作业时，系统会提供备用的寄送位置。

支持的寄送位置：

- 中国东部
- 中国北部

### <a name="shipping"></a>装运
**将驱动器寄送到数据中心：**

创建导入或导出作业时，将会向你提供某个受支持位置的寄送地址，以便你寄送自己的驱动器。提供的寄送地址将取决于你存储帐户的位置，但可能不同于存储帐户位置。

你可以通过 FedEx、DHL、UPS 或中国邮政总局等快递商将你的驱动器寄送到寄送地址。

**从数据中心寄送驱动器：**

创建导入或导出作业时，必须提供回寄地址，以便在你的作业完成后使用该地址将驱动器寄回给你。请确保提供有效的回寄地址，以免延误对你的驱动器的处理。

此外还必须提供有效的中国邮政服务帐号，以便寄回驱动器。如果你已经有了一个快递商帐户号码，请确保其有效。

寄送包裹时，必须遵循 [Azure 服务条款](/support/legal/services-terms/)中的条款。

> [AZURE.IMPORTANT] 请注意，你发运的物理介质可能需要穿越国界。你应当负责确保你的物理介质和数据是遵照适用的法律导入和/或导出的。在寄送物理介质之前，请咨询你的顾问以验证你的介质和数据是否可以合法地寄送到所确定的数据中心。这将有助于确保它可以及时到达。

## <a name="how-does-the-azure-importexport-service-work"></a>Azure 导入/导出服务是如何工作的？
你可以使用 Azure 导入/导出服务在本地站点和 Azure Blob 存储之间传输数据，只需创建作业，然后将硬盘驱动器寄送到 Azure 数据中心即可。你寄送的每个硬盘驱动器都与单个作业相关联。每个作业都与单个存储帐户相关联。请仔细查看[“先决条件”部分](#pre-requisites)，以了解此服务的具体情况，例如支持的 Blob 类型、磁盘类型、位置和寄送方式。

此部分将深入介绍导入和导出作业所涉及的步骤。 随后，会在[“快速启动”部分](#quick-start)提供分步说明，指导创建导入和导出作业。

### <a name="inside-an-import-job"></a>关于导入作业
概括而言，导入作业包括以下步骤：

- 确定要导入的数据，以及所需驱动器数目。
- 确定 Blob 存储中用于存储数据的目标 Blob 位置。
- 使用 WAImportExport 工具将数据复制到一个或多个硬盘驱动器，并使用 BitLocker 进行加密。
- 使用 Azure 门户或导入/导出 REST API 在目标存储帐户中创建导入作业。如果使用 Azure 门户，请上载驱动器日志文件。
- 请提供回寄地址以及快递商帐户号码，以便我们将驱动器寄回给你。
- 将硬盘驱动器寄送到在创建作业时获得的寄送地址。
- 在导入作业详细信息中更新快递跟踪号码，然后提交导入作业。
- Azure 数据中心在收到驱动器后会对其进行处理。
- 该中心会使用你的快递商帐户将驱动器寄送到你在导入作业中提供的回寄地址。

	![图 1：导入作业流](./media/storage-import-export-service/importjob.png)

### <a name="inside-an-export-job"></a>关于导出作业
概括而言，导出作业包括以下步骤：

- 确定要导出的数据，以及所需驱动器数目。
- 确定你的数据在 Blob 存储中的源 Blob 或容器路径。
- 使用 Azure 门户或导入/导出 REST API 在源存储帐户中创建导出作业。
- 指定你的数据在导出作业中的源 Blob 或容器路径。
- 请提供回寄地址以及快递商帐户号码，以便我们将驱动器寄回给你。
- 将硬盘驱动器寄送到在创建作业时获得的寄送地址。
- 在导出作业详细信息中更新快递跟踪号码，然后提交导出作业。
- Azure 数据中心在收到驱动器后会对其进行处理。
- 驱动器使用 BitLocker 加密；密钥通过 Azure 门户提供。
- 该中心会使用你的快递商帐户将驱动器寄送到你在导入作业中提供的回寄地址。

	![图 2：导出作业流](./media/storage-import-export-service/exportjob.png)

### <a name="viewing-your-job-and-drive-status"></a><a name="viewing-your-job-status"></a>查看作业和驱动器状态

可以从 Azure 门户跟踪导入或导出作业的状态。单击“导入/导出”选项卡。作业的列表将出现在该页上。

![View Job State](./media/storage-import-export-service/jobstate.png)


你会看到以下作业状态之一，具体取决于你的驱动器处于哪个处理阶段。

| 作业状态 | 说明 |
|:--- |:--- |
| Creating | 创建一个作业后，其状态设置为 Creating。当作业处于 Creating 状态时，导入/导出服务假设驱动器尚未寄送到数据中心。作业可保持 Creating 状态最多两周，之后服务会自动将其删除。 |
| 装运 | 寄出包裹后，应在 Azure 门户中更新跟踪信息。这会将作业转为“Shipping”状态。作业将保持 Shipping 状态最多两周。 
| Received | 数据中心中收到所有驱动器后，作业状态将设置为 Received。 |
| 转移 | 至少已开始处理一个驱动器后，作业状态将设置为 Transferring。有关详细信息，请参阅下面的“驱动器状态”部分。 |
| 打包 | 处理完所有驱动器后，作业将进入 Packaging 状态，直到将驱动器寄回给客户。 |
| 已完成 | 将所有驱动器寄回客户后，如果完成作业时未出现错误，该作业将设置为 Completed 状态。作业在保持 Completed 状态 90 天后将自动删除。 |
| 已关闭 | 将所有驱动器寄回客户后，如果在处理作业期间出现过错误，该作业将设置为 Closed 状态。作业在保持 Closed 状态 90 天后将自动删除。 |

下表描述了单个驱动器在导入或导出作业中转换时的生命周期。 现在，作业中每个驱动器的当前状态都会在 Azure 门户中显示。下表描述了每个驱动器在作业中可能经历的每种状态。

![查看驱动器状态](./media/storage-import-export-service/drivestate.png)  


| 驱动器状态 | 说明 |
|:--- |:--- |
| Specified | 对于导入作业，在 Azure 门户中创建作业时，驱动器的初始状态为 Specified。对于导出作业，由于在创建该作业时未指定驱动器，因此驱动器的初始状态为 Received。 |
| Received | 导入/导出服务操作员为导入作业处理从货运公司收到的驱动器后，驱动器将转换为 Received 状态。对于导出作业，初始驱动器状态为 Received。 |
| NeverReceived | 当作业的包裹已送达但包裹不包含驱动器时，驱动器将转换为 NeverReceived 状态。如果在服务收到发货信息后的两周内包裹未送达数据中心，驱动器也会转换为此状态。 |
| 转移 | 服务开始将数据从驱动器传输到 Azure 存储时，驱动器将转换为 Transferring 状态。 |
| 已完成 | 服务成功传输所有数据且未出错时，驱动器将转换为 Completed 状态。
| CompletedMoreInfo | 如果服务在从驱动器中复制数据或将数据复制到驱动器时遇到一些问题，驱动器将转换为 CompletedMoreInfo 状态。信息可以包含有关覆盖 Blob 的错误、警告或信息性消息。
| ShippedBack | 驱动器从数据中心寄送到回邮地址后，驱动器将转换为 ShippedBack 状态。 |

Azure 门户中的此映像会显示示例作业的驱动器状态：

![查看驱动器状态](./media/storage-import-export-service/drivestate.png)

下表描述了驱动器故障状态以及针对每种状态采取的措施。

| 驱动器状态 | 事件 | 解决方法/后续步骤 |
|:--- |:--- |:--- |
| NeverReceived | 标记为 NeverReceived 的驱动器（因为在作业寄送过程中未收到）通过另一次寄送送达。 | 运营团队将驱动器状态转换为 Received。 |
| 不适用 | 不属于任何作业的驱动器将作为其他作业的一部分送至数据中心。 | 完成与原始包裹关联的作业后，驱动器将标记为额外驱动器并寄回给客户。 |

### <a name="time-to-process-job"></a>处理作业的时间
处理导入/导出作业的时间各不相同，取决于不同的因素，例如寄送时间、作业类型、要复制的数据的类型和大小，以及所提供磁盘的大小。 导入/导出服务没有 SLA。 你可以通过 REST API 更密切地跟踪作业进度。 在“列出作业”操作中有一个完成百分比参数，该参数指示复制进度。 如果你需要估算何时才能完成时间要求紧的导入/导出作业，请联系我们。

### <a name="pricing"></a>定价
**驱动器处理费用**

在导入或导出作业的过程中，处理每个驱动器都需要支付驱动器处理费用。请参阅有关 [Azure 导入/导出定价](/pricing/details/storage-import-export/)的详细信息。

**寄送费用**

将驱动器寄送到 Azure 时，你需要向快递商支付寄送费用。当驱动器寄回给你时，寄送费用由你在创建作业时提供的快递商帐户支付。

**事务费用**

将数据导入 Blob 存储没有事务费用。将数据从 Blob 存储导出时，需支付标准的传出费用。有关事务费用的更多详细信息，请参阅[数据传输定价](/pricing/details/data-transfer/)。

## <a name="quick-start"></a>快速启动
此部分提供创建导入和导出作业的分步说明。 请确保在执行下一步操作之前符合所有 [先决条件](#pre-requisites) 。

## <a name="create-an-import-job"></a>创建导入作业
创建导入作业时，需将数据从硬盘驱动器复制到 Azure 存储帐户，其方法是将一个或多个包含数据的驱动器寄送到指定的数据中心。 导入作业会将有关硬盘驱动器、要复制的数据、目标存储帐户和寄送信息的详细信息传至 Azure 导入/导出服务。创建导入作业是一个三步过程。首先，使用 WAImportExport 工具准备驱动器。其次，使用 Azure 门户提交导入作业。最后，将驱动器寄送到你在创建作业时获得的寄送地址，并在作业详细信息中更新寄送信息。

> [AZURE.IMPORTANT]
> 一个存储帐户只能提交一个作业。 寄送驱动器时，一个驱动器只能导入一个存储帐户中。 例如，假设希望将数据导入两个存储帐户中。 则必须对每个存储帐户使用不同的硬盘驱动器，并针对存储帐户创建不同的作业。
> 
> 

### <a name="prepare-your-drives"></a>准备驱动器
使用 Azure 导入/导出服务导入数据时，第一步是通过 WAImportExport 工具准备驱动器。按照以下步骤准备你的驱动器。


1. 确定要导入的数据。导入的数据可以是本地服务器或网络共享中的目录和独立文件。
2. 根据数据总大小确定所需驱动器数目。采购所需数目的 2.5 英寸 SSD 或者 2.5/3.5 英寸 SATA II/III 或 SAS 硬盘驱动器。
3. 确定目标存储帐户、容器、虚拟目录和 Blob。
4. 确定要复制到每个磁盘驱动器的目录和/或独立文件。
5. 为 dataset 和 driveset 创建 CSV 文件。
    
    **Dataset CSV 文件**
    
    下面是 dataset CSV 文件的示例：
    

	    BasePath,DstBlobPathOrPrefix,BlobType,Disposition,MetadataFile,PropertiesFile
	    "F:\50M_original\100M_1.csv.txt","containername/100M_1.csv.txt",BlockBlob,rename,"None",None
	    "F:\50M_original\","containername/",BlockBlob,rename,"None",None 

   
    在上面的示例中，100M_1.csv.txt 将复制到名为“containername”的根容器中。 如果名为“containername”的容器不存在，系统会创建一个。 50M_original 下的所有文件和文件夹将以递归方式复制到 containername。 文件夹结构保持不变。

    详细了解如何[准备 dataset CSV 文件](/documentation/articles/storage-import-export-tool-preparing-hard-drives-import/#prepare-the-dataset-csv-file)。
    
    **请记住**：默认情况下，数据将作为块 Blob 导入。可以使用 BlobType 字段值以页 Blob 的方式导入数据。例如，如果你要要导入 VHD 文件，而这些文件需要以磁盘方式装载到 Azure VM 上，则必须以页 Blob 方式导入。
    
    **Driveset CSV 文件**

    driveset 标志的值是 一个CSV 文件，其中包含要使工具可以正确选择要准备的磁盘列表，要将驱动器号映射到的磁盘的列表。 

    下面是 driveset CSV 文件的示例：
    

	    DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
	    G,AlreadyFormatted,SilentMode,AlreadyEncrypted,060456-014509-132033-080300-252615-584177-672089-411631 |
	    H,Format,SilentMode,Encrypt,


    在上面的示例中，假设附加了两个磁盘，并创建了盘符为 G:\\ 和 H:\\ 的基本 NFTS 卷。工具将格式化并加密托管 H:\\ 的磁盘，但不会格式化并加密托管卷 G: 的磁盘。

    详细了解如何[准备 driveset CSV 文件](/documentation/articles/storage-import-export-tool-preparing-hard-drives-import/#prepare-initialdriveset-or-additionaldriveset-csv-file)。

6. 使用 [WAImportExport 工具](http://download.microsoft.com/download/3/6/B/36BFF22A-91C3-4DFC-8717-7567D37D64C5/WAImportExport.zip) 将数据复制到一个或多个硬盘驱动器。
7. 可以通过在 drivset CSV 中的 Encryption 字段内指定 “Encrypt”，在硬盘驱动器上启用 BitLocker 加密。也可以手动在硬盘驱动器上启用 BitLocker 加密，然后在运行工具时，在 driveset CSV 文件中指定 “AlreadyEncrypted”并提供密钥。

8. 完成磁盘准备操作以后，请勿修改硬盘驱动器上的数据，也勿修改日志文件。

> [AZURE.IMPORTANT]准备的每个硬盘驱动器都会生成一个日志文件。使用 Azure 门户创建导入作业时，必须上传属于该导入作业的驱动器的所有日志文件。不带日志文件的驱动器将得不到处理。


下面是使用 WAImportExport 工具进行硬盘驱动器准备的命令和示例。

在第一个复制会话中使用新复制会话复制目录和/或文件的 WAImportExport 工具 PrepImport 命令：


	WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] [/sk:<StorageAccountKey>] [/silentmode] [/InitialDriveSet:<driveset.csv>] DataSet:<dataset.csv>

导入示例 1


	WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1  /sk:************* /InitialDriveSet:driveset-1.csv /DataSet:dataset-1.csv /logdir:F:\logs


若要**添加更多驱动器**，可以创建一个新的 driveset 文件并运行以下命令。如果后续复制会话中的磁盘驱动器与 InitialDriveset .csv 中指定的不同，可以指定一个新的 driveset CSV 文件并将其作为值提供给 “AdditionalDriveSet” 参数。 使用 同一日记文件名称并提供新的会话 ID。 AdditionalDriveset CSV 文件的格式与 InitialDriveSet 的格式相同。


	WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> /AdditionalDriveSet:<driveset.csv>

导入示例 2

	WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#3  /AdditionalDriveSet:driveset-2.csv


若要向同一个的 driveset 添加更多数据，后续复制会话可以调用 WAImportExport 工具的 PrepImport 命令来复制其他文件/目录：
对于 InitialDriveset .csv 文件中为相同硬盘驱动器指定的后续复制会话，请指定**同一日志文件**的名称并提供**新的会话 ID**；不需要提供存储帐户密钥。


	WAImportExport PrepImport /j:<JournalFile> /id:<SessionId> /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] DataSet:<dataset.csv>

导入示例 3


	WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2  /DataSet:dataset-2.csv

若要更详细了解如何使用 WAImportExport 工具，请参阅[为导入准备硬盘驱动器](/documentation/articles/storage-import-export-tool-preparing-hard-drives-import/)。

另请参阅[为导入作业准备硬盘驱动器的示例工作流](/documentation/articles/storage-import-export-tool-sample-preparing-hard-drives-import-job-workflow/)，以获取更详细的分步说明。  

### <a name="create-the-import-job"></a>创建导入作业
1. 准备好驱动器后，在[ Azure 门户](https://portal.azure.cn)中导航到你的存储帐户，然后查看“仪表板”。在“速览”下，单击“创建导入作业”。查看相关步骤，然后选择指示你已准备好驱动器并提供了驱动器日志文件的复选框。

2. 在步骤 1 中，提供负责该导入作业的人员的联系信息，以及有效的回寄地址。如果你希望保存导入作业的详细日志数据，则选中“将详细日志保存在我的‘waimportexport’Blob 容器中”选项。

3. 在步骤 2 中，上载你在驱动器准备步骤中获取的驱动器日志文件。你需要为已准备好的每个驱动器上载一个文件。

    ![创建导入作业 - 步骤 3](./media/storage-import-export-service/import-job-03.png)

4. 在步骤 3 中，输入导入作业的描述性名称。请注意，你输入的名称只能包含小写字母、数字、连字符和下划线，必须以字母开头并且不得包含空格。在作业进行中以及作业完成后，你将使用所选名称来跟踪作业。

    接下来，从列表中选择你的数据中心区域。数据中心区域会指示必须将你的包裹运送到的数据中心和地址。有关更多信息，请参见下面的“常见问题”。

5. 在步骤 4 中，从列表中选择回程快递商，并输入你的快递商帐户号码。当你的导入作业完成后，使用此帐户寄回驱动器。

    如果你有跟踪号码，则从列表中选择你的快递商，并输入你的跟踪号码。

    如果你还没有跟踪号码，请选择“我将在发运我的包裹后提供此导入作业的发运信息”，然后完成导入过程。

6. 若要在寄送包裹后输入跟踪号，请在 Azure 门户中返回到存储帐户的“导入/导出”页面，从列表中选择作业并选择“寄送信息”。在向导中导航并在步骤 2 中输入跟踪号码。

    如果在创建作业后的 2 周内未更新跟踪号，该作业将会过期。

    如果作业处于“正在创建”、“正在发运”或“正在传送”状态，则你还可以在向导的第 2 步中更新你的承运人帐号。一旦作业处于“正在打包”状态，你将无法更新该作业的承运人帐号。

7. 你可以在门户仪表板上跟踪作业进度。若要了解上一部分中每个作业状态的含义，请[查看作业状态](#viewing-your-job-status)。

## <a name="create-an-export-job"></a>创建导出作业
创建导出作业的目的是通知导入/导出服务：你要将一个或多个空驱动器寄送到数据中心；这样数据中心就可以将数据从你的存储帐户导出到驱动器，然后将驱动器寄送给你。

### <a name="prepare-your-drives"></a>准备驱动器
对驱动器进行准备以便完成导出作业时，建议你执行以下预检查：

1. 使用 WAImportExport 工具的 PreviewExport 命令检查所需的磁盘数。有关详细信息，请参阅 [预览导出作业的驱动器使用情况](https://msdn.microsoft.com/zh-cn/library/azure/dn722414.aspx)。它可以根据你要使用的驱动器大小，帮助你预览所选 Blob 的驱动器使用情况。
2. 请检查你是否可以读取/写入需要寄送的用于导出作业的硬盘驱动器。

### <a name="create-the-export-job"></a>创建导出作业
1. 若要创建导出作业，请导航到[ Azure 门户](https://portal.azure.cn)中的存储帐户，然后查看“仪表板”。在“速览”下，单击“创建导出作业”，然后继续完成向导。
2. 在步骤 2 中，提供负责该导出作业的人员的联系信息。如果你希望保存导出作业的详细日志数据，则选中“将详细日志保存在我的‘waimportexport’Blob 容器中”选项。
3. 在步骤 3 中，指定要从你的存储帐户导出到空驱动器中的 Blob 数据。你可以选择导出该存储帐户中的所有 Blob 数据，也可以指定要导出的 Blob 或 Blob 组。

    若要指定要导出的 Blob，请使用“等于”选择器，并指定该 Blob 的相对路径，以容器名称开头。使用 *$root* 指定根容器。
    
    若要指定以某一前缀开头的所有 Blob，请使用“开头为”选择器，并指定前缀，以正斜杠“/”开头。该前缀可以是容器名称的前缀、完整容器名称或者后跟 Blob 名称前缀的完整容器名称。
    
    下表显示有效 Blob 路径的示例：
    
    | 选择器 | Blob 路径 | 说明 |
    | --- | --- | --- |
    | 开头为 |/ |导出存储帐户中的所有 Blob |
    | 开头为 |/$root/ |导出根容器中的所有 Blob |
    | 开头为 |/book | 导出任何容器中以前缀 book 开头的所有 Blob |
    | 开头为 |/music/ | 导出容器 music 中的所有 Blob |
    | 开头为 |/music/love | 导出容器 music 中以前缀 love 开头的所有 Blob |
    | 等于 |$root/logo.bmp | 导出根容器中的 Blob logo.bmp |
    | 等于 |videos/story.mp4 |导出容器 videos 中的 Blob story.mp4 |
   
    必须以有效格式提供 Blob 路径，以免在处理过程中出现错误，如以下屏幕快照所示。
   
    ![创建导出作业 - 步骤 3](./media/storage-import-export-service/export-job-03.png)
   
4. 在步骤 4 中，为导出作业输入一个描述性名称。你输入的名称只能包含小写字母、数字、连字符和下划线，必须以字母开头并且不得包含空格。
    
    数据中心区域将指示必须将你的包装运送到的数据中心。有关更多信息，请参见下面的“常见问题”。

5. 在步骤 5 中，从列表中选择回程承运人，并输入你的承运人帐号。当你的导出作业完成后，将使用此帐户寄回你的驱动器。
   
    如果你有跟踪号码，则从列表中选择你的快递商，并输入你的跟踪号码。
   
    如果你还没有跟踪号码，请选择“我将在发运我的包裹后提供此导出作业的发运信息”，然后完成导出过程。

6. 若要在寄送包裹后输入跟踪号，请在 Azure 门户中返回到存储帐户的“导入/导出”页面，从列表中选择作业并选择“寄送信息”。在向导中导航并在步骤 2 中输入跟踪号码。
   
    如果在创建作业后的 2 周内未更新跟踪号，该作业将会过期。
   
    如果作业处于“正在创建”、“正在发运”或“正在传送”状态，则你还可以在向导的第 2 步中更新你的承运人帐号。一旦作业处于“正在打包”状态，你将无法更新该作业的承运人帐号。
   
	> [AZURE.NOTE] 如果在复制到硬盘时要导出的 Blob 正在使用中，Azure 导入/导出服务将生成该 Blob 的快照，然后复制快照。

7. 可以在 Azure 门户中的仪表板上跟踪作业进度。通过“查看作业状态”，了解上一部分中每个作业状态的含义。

8. 收到包含已导出数据的驱动器以后，即可查看和复制该服务为你的驱动器生成的 BitLocker 密钥。在 Azure 门户中导航到你的存储帐户，然后单击“导入/导出”选项卡。从列表中选择你的导出作业，然后单击“查看密钥”按钮。BitLocker 密钥随即出现，如下所示：

	![查看导出作业的 BitLocker 密钥](./media/storage-import-export-service/export-job-bitlocker-keys.png)

请查看下面的“常见问题”部分，因为该部分介绍了客户在使用此服务时遇到的最常见问题。

## <a name="frequently-asked-questions"></a>常见问题

**能否使用 Azure 导入/导出服务复制 Azure 文件？**

否。Azure 导入/导出服务仅支持块 Blob 和页 Blob。所有其他存储类型（包括 Azure 文件、表和队列）均不受支持。

**Azure 导入/导出服务是否适用于 CSP 订阅？**

否。Azure 导入/导出服务不支持 CSP 订阅。将来会增加此方面的支持。

**能否跳过导入作业的驱动器准备步骤？能否在不复制的情况下准备驱动器？**

任何需要寄送到数据中心进行数据导入的驱动器都必须使用 WAImportExport 工具进行准备。必须使用 WAImportExport 工具将数据复制到驱动器。

**在创建导出作业时我是否需要执行任何磁盘准备操作？**

不需要，但建议执行一些预先检查。使用 WAImportExport 工具的 PreviewExport 命令检查所需的磁盘数。有关详细信息，请参阅 [预览导出作业的驱动器使用情况](https://msdn.microsoft.com/zh-cn/library/azure/dn722414.aspx)。它可以根据你要使用的驱动器大小，帮助你预览所选 Blob 的驱动器使用情况。此外，请检查你是否可以读取和写入需要寄送的用于导出作业的硬盘驱动器。

**如果我无意中发送了不符合支持的要求的 HDD，会发生什么情况？**

Azure 数据中心会将不符合支持要求的驱动器返还给你。如果包裹中只有某些驱动器满足支持要求，将处理这些驱动器，并且不符合支持的要求的驱动器将返还给你。

**是否可以取消我的作业？**

你可以取消状态为“创建”或“装运”的作业。

**在 Azure 门户中可以查看多长时间的已完成作业的状态？**

你可以查看最长 90 天的已完成作业的状态。已完成作业将在 90 天后被删除。

**如果我想要导入或导出超过 10 个驱动器，我应该做什么？**

对于导入/导出服务，一个导入或导出作业在单个作业中只能引用 10 个驱动器。如果你想要装运超过 10 个驱动器，可以创建多个作业。与同一作业关联的驱动器必须放在同一个包裹中一起寄送。

**该服务是否会在返还驱动器之前将其格式化？**

否。所有驱动器都使用 BitLocker 加密。

**我是否可为导入/导出作业从 21 世纪互联购买驱动器？**

不可以。对于导入和导出作业，你将需要装运你自己的驱动器。

**导入作业完成后，我的数据在存储帐户中看起来是什么样的？ 是否会保留我的目录层次结构？**

为导入作业准备硬盘驱动器时，目标由 dataset CSV 文件中的 DstBlobPathOrPrefix 字段指定。该目标是存储帐户中的目标容器，可以将硬盘驱动器中的数据复制到其中。在该目标容器中，将为硬盘驱动器中的文件夹创建虚拟目录，为文件创建 Blob。

**如果驱动器的文件已存在于我的存储帐户中，该服务是否会覆盖我的存储帐户中的现有 Blob？**

准备驱动器时，可以使用 dataset CSV 文件中名为 Disposition:<rename|no-overwrite|overwrite> 的字段指定是否应覆盖或忽略目标文件。默认情况下，该服务会将新文件重命名，而不是覆盖现有 Blob。


**WAImportExport 工具是否与 32 位操作系统兼容？** 
否。WAImportExport 工具仅与 64 位 Windows 操作系统兼容。有关受支持的 OS 版本的完整列表，请参阅[先决条件](#pre-requisites)中的“操作系统”部分。

**是否应该在包裹中添加除硬盘驱动器之外的其他东西？**

请仅发运你的硬盘驱动器。不要包括电源线或 USB 电缆之类的物品。

**是否必须通过 FedEx 或 DHL 寄送我的驱动器？**

你可以通过任何知名的快递商（例如 FedEx、DHL、UPS 或中国邮政总局）将驱动器寄送到数据中心。

**跨国寄送驱动器是否存在限制？**

请注意，你发运的物理介质可能需要穿越国界。你应当负责确保你的物理介质和数据是遵照适用的法律导入和/或导出的。在发运物理介质之前，请咨询你的顾问以验证你的介质和数据是否可以合法地发运到所确定的数据中心。这将有助于确保它可以及时到达。

**创建作业时，寄送地址是一个不同于存储帐户位置的位置。我该怎样做？**

某些存储帐户位置映射到备用寄送位置。此前可用的寄送位置也可临时映射到备用位置。在寄送驱动器之前，请始终查看你在创建作业的过程中提供的寄送地址。

**寄送我的驱动器时，快递商要求我提供数据中心的联系人姓名和电话号码。我该如何提供？**

电话号码在创建作业时提供给你。

**能否使用 Azure 导入/导出服务将 PST 邮箱和 SharePoint 数据复制到 O365？**

请参阅[将 PST 文件或 SharePoint 数据导入到 Office 365](https://technet.microsoft.com/zh-cn/library/ms.o365.cc.ingestionhelp.aspx)。

**能否使用 Azure 导入/导出服务将我的备份脱机复制到 Azure 备份服务？**

请参阅 [Azure 备份中的脱机备份工作流](/documentation/articles/backup-azure-backup-import-export/)。

**一次寄送中 HDD 的最大数量是多少？

一次寄送中可以有任何数量 HDD，并且如果磁盘属于多个作业，建议：a) 使用对应的作业名称标记磁盘。 b) 使用后缀为 -1、-2 等的跟踪号码更新作业。
  
**磁盘导入/导出支持的块 Blob 和页 Blob 的最大大小是多少？

最大块 Blob 大小约为 4.768 TB 或 5,000,000 MB。
最大页 Blob 大小为 1 TB。
## <a name="next-steps"></a>后续步骤

* [设置 WAImportExport 工具](/documentation/articles/storage-import-export-tool-how-to/)
* [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)

<!--Update_Description:add two commom questions;wording update;add anchors to subtitles-->