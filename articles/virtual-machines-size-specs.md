<properties
 pageTitle="虚拟机大小 | Microsoft Azure"
 description="列出虚拟机的不同大小及其容量。"
 services="virtual-machines"
 documentationCenter=""
 authors="cynthn"
 manager="timlt"
 editor=""
 tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="01/05/2016"
	wacn.date=""/>

# 虚拟机的大小

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

## 概述

本文介绍了你可用于运行应用程序和工作负荷的基于虚拟机的计算资源的可用大小和选项。此外，还提供了你在计划使用这些资源时要考虑的部署注意事项。有关不同大小的定价信息，请参阅[虚拟机定价](/home/features/virtual-machines/#price)。

若要查看 Azure VM 的一般限制，请参阅 [Azure 订阅和服务限制、配额和约束](/documentation/articles/azure-subscription-service-limits)。

标准大小由几个系列组成：A 和 D。其中某些大小的注意事项包括：

*   D 系列的 VM 旨在运行需要更高计算能力和临时磁盘性能的应用程序。D 系列 VM 为临时磁盘提供更快的处理器、更高的内存内核比和固态驱动器 (SSD)。有关详细信息，请参阅 Azure 博客[新的 D 系列虚拟机大小](http://azure.microsoft.com/blog/2014/09/22/new-d-series-virtual-machine-sizes/)上的公告。

*   DS 系列的 VM 可使用高级存储，从而为 I/O 密集型工作负荷提供高性能、低延迟的存储。这些 VM 使用固态硬盘 (SSD) 托管虚拟机的磁盘，而且还提供本地 SSD 磁盘高速缓存。高级存储只在某些区域可用。有关详细信息，请参阅[高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage-preview-portal)。

虚拟机的大小会影响定价。大小还会影响虚拟机的处理、内存和存储容量。存储成本是基于存储帐户中的已使用页数进行单独计算。有关详细信息，请参阅[虚拟机定价详细信息](/home/features/virtual-machines/#price)和 [Azure 存储定价](/home/features/storage/#price)。有关 VM 的存储的更多详细信息，请参阅[关于虚拟机的磁盘和 VHD](/documentation/articles/virtual-machines-disks-vhds)。

以下注意事项可能会帮助你决定大小：


*   Azure 数据中心内的一些物理主机可能不支持更大的虚拟机大小，例如 A5 - A11。因此，在以下情况下，可能会显示错误消息**<machine name>**“未能配置虚拟机”或**<machine name>**“未能创建虚拟机”：将现有虚拟机的大小调整为新的大小时；在 2013 年 4 月 16 日之前创建的虚拟网络中创建新的虚拟机时；或者向现有的云服务中添加新的虚拟机时。有关每个部署方案的解决方法，请参阅支持论坛上的[错误：“未能配置虚拟机”](https://social.msdn.microsoft.com/Forums/9693f56c-fcd3-4d42-850e-5e3b56c7d6be/error-failed-to-configure-virtual-machine-with-a5-a6-or-a7-vm-size?forum=WAVirtualMachinesforWindows)。  


## 性能注意事项

我们已创建 Azure 计算单元 (ACU) 的概念，以提供一种比较 Azure SKU 中的计算 (CPU) 性能的方法。这可帮助你轻松确定哪个 SKU 最有可能满足你的性能需要。ACU 当前已在小型 (Standard\_A1) VM 上标准化为 100，而所有其他 SKU 则表示 SKU 在运行标准基准测试时大约可以快多少。

>[AZURE.IMPORTANT] ACU 仅是准则。你的工作负荷的结果可能会有所不同。

<br>

|SKU 系列 |ACU/核心 |
|---|---|
|[Standard\_A0（特小型）](#standard-tier-a-series) |50 |
|[Standard\_A1-4（小型 - 大型）](#standard-tier-a-series) |100 |
|[Standard\_A5-7](#standard-tier-a-series) |100 |
|[D1-14](#standard-tier-d-series) |160 |
|[DS1-14](#standard-tier-ds-series) |160 |

ACU 标有 *使用 Intel® Turbo 技术来增加 CPU 频率，并提升性能。提升量可能因 VM 大小、工作负荷和同一主机上运行的其他工作负荷而有所不同。



## 大小表

下表显示这些虚拟机提供的大小和容量。

>[AZURE.NOTE] 存储容器使用 1024^3 字节数（GB 的计量单位）表示。这有时也称为 GiB，即二进制定义。在比较使用不同进制系统的大小时，请记住二进制大小可能看上去小于十进制大小，但对于任何具体大小（例如 1 GB），二进制系统提供的容量多于十进制系统，因为 1024^3 大于 1000^3。

<br>

## 标准层：A 系列

在经典部署模型中，某些 VM 大小在 Powershell 和 CLI 中稍有不同。

* Standard\_A0 是特小型 
* Standard\_A1 是小型
* Standard\_A2 是中型
* Standard\_A3 是大型
* Standard\_A4 是超大型

<br>

|大小 |CPU 核心数|内存|NIC 数（最大值）|最大磁盘大小|最大数据磁盘（每个 1023 GB）|最大IOPS（每个磁盘 500 次）|
|---|---|---|---|---|---|---|
|Standard\_A0\\特小型 |1|768 MB|1| 临时磁盘 = 20 GB |1|1x500|
|Standard\_A1\\小型|1|1\.75 GB|1|临时磁盘 = 70 GB |2|2x500|
|Standard\_A2\\中型|2|3\.5 GB|1|临时磁盘 = 135 GB |4|4x500|
|Standard\_A3\\大型|4|7 GB|2|临时磁盘 = 285 GB |8|8x500|
|Standard\_A4\\超大型|8|14 GB|4|临时磁盘 = 605 GB |16|16x500|
|Standard\_A5|2|14 GB|1|临时磁盘 = 135 GB |4|4X500|
|Standard\_A6|4|28 GB|2|临时磁盘 = 285 GB |8|8x500|
|Standard\_A7|8|56 GB|4|临时磁盘 = 605 GB |16|16x500|

## 标准层：D 系列

|大小 |CPU 核心数|内存|NIC 数（最大值）|最大磁盘大小|最大数据磁盘（每个 1023 GB）|最大IOPS（每个磁盘 500 次）|
|---|---|---|---|---|---|---|
|Standard\_D1 |1|3\.5 GB|1|临时磁盘 (SSD) = 50 GB |2|2x500|
|Standard\_D2 |2|7 GB|2|临时磁盘 (SSD) = 100 GB |4|4x500|
|Standard\_D3 |4|14 GB|4|临时磁盘 (SSD) = 200 GB |8|8x500|
|Standard\_D4 |8|28 GB|8|临时磁盘 (SSD) = 400 GB |16|16x500|
|Standard\_D11 |2|14 GB|2|临时磁盘 (SSD) = 100 GB |4|4x500|
|Standard\_D12 |4|28 GB|4|临时磁盘 (SSD) = 200 GB |8|8x500|
|Standard\_D13 |8|56 GB|8|临时磁盘 (SSD) = 400 GB |16|16x500|
|Standard\_D14 |16|112 GB|8|临时磁盘 (SSD) = 800 GB |32|32x500|

## 标准层：DS 系列*

|大小 |CPU 核心数|内存|NIC 数（最大值）|最大磁盘大小|最大数据磁盘（每个 1023 GB）|高速缓存大小 (GB)|最大磁盘 IOPS 和带宽|
|---|---|---|---|---|---|---|---|
|Standard\_DS1 |1|3\.5|1|本地 SSD 磁盘 = 7 GB |2|43| 3,200 每秒 32 MB |
|Standard\_DS2 |2|7|2|本地 SSD 磁盘 = 14 GB |4|86| 6,400 每秒 64 MB |
|Standard\_DS3 |4|14|4|本地 SSD 磁盘 = 28 GB |8|172| 12,800 每秒 128 MB |
|Standard\_DS4 |8|28|8|本地 SSD 磁盘 = 56 GB |16|344| 25,600 每秒 256 MB |
|Standard\_DS11 |2|14|2|本地 SSD 磁盘 = 28 GB |4|72| 6,400 每秒 64 MB |
|Standard\_DS12 |4|28|4|本地 SSD 磁盘 = 56 GB |8|144| 12,800 每秒 128 MB |
|Standard\_DS13 |8|56|8|本地 SSD 磁盘 = 112 GB |16|288| 25,600 每秒 256 MB |
|Standard\_DS14 |16|112|8|本地 SSD 磁盘 = 224 GB |32|576| 50,000 每秒 512 MB |

*DS 系列 VM 可能的最大每秒输入/输出操作次数 (IOPS) 和吞吐量（带宽）受磁盘大小影响。有关详细信息，请参阅[高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage-preview-portal)。

### 另请参阅

[Azure 订阅和服务限制、配额和约束](/documentation/articles/azure-subscription-service-limits)

<!---HONumber=Mooncake_0215_2016-->