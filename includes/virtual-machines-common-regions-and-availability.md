# 区域和可用性概述
Azure 在中国的 2 个区域运营，使你能够灵活选择在何处构建应用程序。你还可以利用 Azure 平台中的冗余和可用性功能（例如存储复制和可用性集）构建高度可用的应用。

## 什么是 Azure 区域？
Azure 允许你在规定的地理区域（例如“中国北部”或“中国东部”）创建资源（例如 VM）。目前中国有 2 个 Azure 区域：“中国北部”和“中国东部”。为了提供冗余和可用性，每个区域都设有多个数据中心。这样，你便可以在构建应用程序时获得弹性，创建距离用户最近的虚拟机 (VM)，满足任何法律、法规或税务要求。

## 存储可用性
在考虑可用的 Azure 存储空间复制选项时，了解 Azure 区域和地理位置变得相当重要。创建存储帐户时，必须选择以下复制选项之一：

- 本地冗余存储 (LRS)
    - 在你创建存储帐户时所在的区域复制数据三次。
- 区域冗余存储 (ZRS)
    - 在两到三个设施之间复制数据三次（在单个区域内或两个区域之间）。
- 异地冗余存储 (GRS)
    - 将数据复制到距主要区域数百英里以外的次要区域。
- 读取访问异地冗余存储 (RA-GRS)
    - 与 GRS 一样，可将数据复制到次要区域，但此外还提供对次要位置中数据的只读访问权限。

下表提供存储复制类型之间差异的简要概述：

| 复制策略 | LRS | ZRS | GRS | RA-GRS |
|:-----------------------------------------------------------------------------------|:----|:----|:----|:-------|
| 数据在多个设施之间进行复制。 | 否 | 是 | 是 | 是 |
| 可以从辅助位置和主位置读取数据。 | 否 | 否 | 否 | 是 |
| 在单独的节点上维护的数据副本数。 | 3 | 3 | 6 | 6 |

可以[在此处详细了解 Azure 存储空间复制选项](/documentation/articles/storage-redundancy/)。

### 存储费用
价格根据所选存储类型和可用性的不同而异。

- 标准存储以普通的旋转式磁盘为基础，收费方式根据所用容量和所需的存储可用性而定。
    - 对于 RA-GRS，有一个额外的异地复制数据传输费用，它是将该数据复制到另一个 Azure 区域收取的带宽费用。
- 高级存储以固态硬盘 (SSD) 为基础，收费方式根据磁盘容量而定。

有关不同存储类型和可用性选项的定价信息，请参阅 [Azure Storage Pricing](/pricing/details/storage/)（Azure 存储空间定价）。


## Azure 映像
在 Azure 中，VM 是基于映像创建的。通常，这是从 Azure 应用商店创建的映像，合作伙伴可以在 Azure 应用商店提供预配置的完整 OS 映像或应用程序映像。

从 Azure 应用商店中的映像创建 VM 时，实际上是在使用模板。Azure Resource Manager 模板是声明性的 JavaScript 对象表示法 (JSON) 文件，可用于创建包含 VM、存储、虚拟网络等的复杂应用程序环境。详细了解如何使用 [Azure Resource Manager 模板](/documentation/articles/resource-group-overview)，包括如何[构建自己的模板](/documentation/articles/resource-group-authoring-templates/)。

也可以使用 [Azure CLI](/documentation/articles/virtual-machines-linux-upload-vhd/) 或 [Azure PowerShell](/documentation/articles/virtual-machines-windows-upload-image/) 创建自己的自定义映像并将其上载，快速创建符合特定构建要求的自定义 VM。使用自定义映像时，需要将 VM 存储在与映像本身相同的存储帐户中。不能将映像上载到单个区域，然后从该映像跨其他 Azure 区域创建 VM。


## 可用性集
可用性集是 VM 的逻辑分组，可让 Azure 了解应用程序的构建方式，以便提供冗余和可用性。建议在可用性集内创建两个或更多个 VM，提供高度可用的应用程序，以及符合 [99\.95% Azure SLA](/support/sla/virtual-machines/)。可用性集由可防止硬件故障以及允许安全应用更新的两个额外分组构成 - 容错域 (FD) 和更新域 (UD)。

![更新域和容错域配置的概念图](./media/virtual-machines-common-regions-and-availability/ud-fd-configuration.png)  


详细了解如何管理 [Linux VM](/documentation/articles/virtual-machines-linux-manage-availability/) 或 [Windows VM](/documentation/articles/virtual-machines-windows-manage-availability/) 的可用性。

### 容错域
容错域是共享公用电源和网络交换机的基础硬件逻辑组，类似于本地数据中心内的机架。当你在可用性集内创建 VM 时，Azure 平台会自动将 VM 分布到这些容错域，以限制可能的物理硬件故障、网络中断或电源中断造成的影响。

### 更新域
更新域是可以同时维护或重新启动的基础硬件逻辑组。当你在可用性集内创建 VM 时，Azure 平台会自动将 VM 分布到这些更新域，确保在 Azure 平台进行定期维护时，始终至少有一个应用程序实例保持运行。请注意，在计划内维护期间，更新域的重新启动顺序可能不会按序进行，但一次只重新启动一个更新域。


## 后续步骤
阅读有关 [Azure 可用性最佳实践](/documentation/articles/best-practices-availability-checklist/)的具体详细信息。

<!---HONumber=Mooncake_0829_2016-->