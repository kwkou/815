<properties
	pageTitle="存储解决方案准则 | Azure"
	description="了解用于在 Azure 基础结构服务中部署存储解决方案的关键设计和实施准则。"
	documentationCenter=""
	services="virtual-machines-linux"
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="06/22/2016"
	wacn.date="08/08/2016"/>

# 存储基础结构准则

[AZURE.INCLUDE [virtual-machines-linux-infrastructure-guidelines-intro](../../includes/virtual-machines-linux-infrastructure-guidelines-intro.md)]

本文着重于了解实现最佳虚拟机 (VM) 性能的存储需求和设计注意事项。


## 存储的实施准则

决策：

- 需要为工作负荷使用标准存储还是高级存储？
- 是否需要进行磁盘条带化以创建大于 1023 GB 的磁盘？
- 是否需要进行磁盘条带化以获得工作负荷的最佳 I/O 性能？
- 你需要使用哪一组存储帐户来托管你的 IT 工作负荷或基础结构？

任务：

- 查看将要部署的应用程序的 I/O 需求，并计划适当的存储帐户数目和类型。
- 使用命名约定创建存储帐户集。可以使用 Azure CLI 或门户。


## 存储

Azure 存储空间是部署与管理虚拟机 (VM) 和应用程序的重要部分。Azure 存储空间提供的服务可用于存储文件数据、非结构化数据和消息，该存储空间也是为 VM 提供支持的基础结构的一部分。

有两种可为 VM 提供支持的存储帐户：

- 标准存储帐户可以访问 Blob 存储（用于存储 Azure VM 磁盘）、表存储、队列存储和文件存储
- [高级存储](/documentation/articles/storage-premium-storage/)可以为 I/O 密集型工作负荷提供高性能且低延迟的磁盘支持（例如 MongoDB 分片群集），且目前仅支持 Azure VM 磁盘。

Azure 使用一个操作系统磁盘、一个临时磁盘和零个或更多可选数据磁盘创建 VM。操作系统磁盘和数据磁盘是 Azure 页 blob，而临时磁盘则通过本地方式存储在计算机所在的节点上。请注意，在设计应用程序时，务必仅将此临时磁盘用于非持久性数据，因为 VM 可能会在维护事件期间在主机之间迁移。任何存储在临时磁盘上的数据都会丢失。

持久性和高可用性将由基础 Azure 存储空间环境提供，以确保数据在发生非计划维护或硬件故障时受到保护。在设计 Azure 存储空间环境时，可以选择在给定 Azure 数据中心内以本地方式复制 VM 存储、在位于给定区域的 Azure 数据中心之间复制 VM 存储，或者甚至在位于不同区域的 Azure 数据中心之间复制 VM 存储。你可以了解[有关针对高可用性的复制选项的详细信息](/documentation/articles/storage-introduction/#replication-for-durability-and-high-availability)。

由于 blob 的最大大小为 1024 千兆字节 (GB)，并且必须包含 VHD 文件的元数据（页脚），因此操作系统磁盘和数据磁盘的最大大小为 1023 GB（1 GB 为 1024<sup>3</sup> 字节）。你可以使用逻辑卷管理器 (LVM) 来超越此限制，方法是将数据磁盘整合在一起，以向 VM 提供大于 1023GB 的逻辑卷。

设计 Azure 存储空间部署时有几个可伸缩性限制，请参阅 [Azure 订阅和服务限制、配额和约束条件](/documentation/articles/azure-subscription-service-limits/#storage-limits)了解更多详细信息。另请参阅 [Azure 存储空间可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。

针对应用程序存储，你可以使用 Blob 存储来存储非结构化对象数据，例如文档、映像、备份、配置数据、日志等等。应用程序可以直接写入 Azure Blob 存储，而不是写入附加到 VM 的虚拟磁盘。根据你的可用性需求和成本限制，Blob 存储还提供[热存储层和冷存储层](/documentation/articles/storage-blob-storage-tiers/)两种选项。


## 条带化的磁盘
除了提供相关功能来创建大于 1023 GB 的磁盘外，在许多情况下，对数据磁盘使用条带化还可增强性能，因为允许多个 blob 支持单个卷的存储。使用条带化时，将会并行处理针对单个逻辑磁盘写入和读取数据所需的 I/O。

Azure 将对可用的数据磁盘量和带宽加以限制，具体取决于 VM 大小。有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-linux-sizes/)。

如果要对 Azure 数据磁盘使用磁盘条带化，请考虑以下准则：

- 数据磁盘应始终为最大大小 (1023 GB)
- 附加 VM 大小所允许的最大数据磁盘量
- 使用 LVM
- 避免使用 Azure 数据磁盘缓存选项（缓存策略 =“无”）

有关详细信息，请参阅[在 Linux VM 上配置 LVM](/documentation/articles/virtual-machines-linux-configure-lvm/)。


## 多个存储帐户

在设计 Azure 存储空间环境时，随着部署的 VM 数目增加，可以使用多个存储帐户。这有助于将 I/O 分散到基础 Azure 存储空间基础结构上，以维持 VM 和应用程序的最佳性能。在设计将要部署的应用程序时，请考虑每个 VM 的 I/O 需求，并使那些 VM 在 Azure 存储帐户之间取得平衡。请尽量避免将所有高 I/O 需求的 VM 分组在仅仅一个或两个帐户中。

有关不同 Azure 存储空间选项的 I/O 功能及一些建议最大值的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。


## <a name="next-steps"></a> 后续步骤

[AZURE.INCLUDE [virtual-machines-linux-infrastructure-guidelines-next-steps](../../includes/virtual-machines-linux-infrastructure-guidelines-next-steps.md)]

<!---HONumber=Mooncake_0801_2016-->