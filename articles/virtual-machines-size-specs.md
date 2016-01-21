<properties
	pageTitle="虚拟机大小"
	description="列出虚拟机的不同大小及其容量。"
	services="virtual-machines"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="09/02/2015"
	wacn.date="01/21/2016"/>

# 虚拟机的大小

## 概述

本文介绍了你可用于运行应用程序和工作负荷的基于虚拟机的计算资源的可用大小和选项。此外，还提供了你在计划使用这些资源时要考虑的部署注意事项。

Azure 提供了多种类型的计算资源，其中的两种类型是 Azure 虚拟机和云服务。有关说明，请参阅 [Azure 提供的计算托管选项](/documentation/articles/fundamentals-application-models/)。

## 规划注意事项

虚拟机有两个可用层级：基本和标准。这两种类型都提供大小选择，但基本层级不提供标准层级中可用的一些功能，例如负载平衡和自动缩放。标准层级的大小由不同系列组成：A 和 D。其中某些大小的注意事项包括：

*   D 系列的 VM 旨在运行需要更高计算能力和临时磁盘性能的应用程序。D 系列 VM 为临时磁盘提供更快的处理器、更高的内存内核比和固态驱动器 (SSD)。有关详细信息，请参阅 Azure 博客[新的 D 系列虚拟机大小](http://azure.microsoft.com/blog/2014/09/22/new-d-series-virtual-machine-sizes/)上的公告。  

虚拟机的大小会影响定价。大小还会影响虚拟机的处理、内存和存储容量。存储成本是基于存储帐户中的已使用页数进行单独计算。有关详细信息，请参阅[虚拟机定价详细信息](/home/features/virtual-machines/#price)和 [Azure 存储定价](/home/features/storage/#price)。有关 VM 的存储的更多详细信息，请参阅[关于虚拟机的磁盘和 VHD](/documentation/articles/virtual-machines-disks-vhds)。

以下注意事项可能会帮助你决定大小：

*   只有使用 Azure SDK 版本 1.3 或更高版本，A0\\Basic\_A0 大小才可用。  

*   A1\\Basic\_A1 是建议用于生产工作负荷的最小大小。

*   在使用 SQL Server Enterprise 版时，选择具有 4 个或 8 个 CPU 内核的虚拟机。

*   Azure 数据中心内的一些物理主机可能不支持更大的虚拟机大小，例如 A5 – A11。因此，在以下情况下，可能会显示错误消息**<machine name>**“未能配置虚拟机”或**<machine name>**“未能创建虚拟机”：将现有虚拟机的大小调整为新的大小时；在 2013 年 4 月 16 日之前创建的虚拟网络中创建新的虚拟机时；或者向现有的云服务中添加新的虚拟机时。有关每个部署方案的解决方法，请参阅支持论坛上的主题[错误：“未能配置虚拟机”](https://social.msdn.microsoft.com/Forums/zh-CN/9693f56c-fcd3-4d42-850e-5e3b56c7d6be/error-failed-to-configure-virtual-machine-with-a5-a6-or-a7-vm-size?forum=WAVirtualMachinesforWindows)。

## 常规限制

下表显示适用于使用服务管理部署模型创建的虚拟机的限制，而与虚拟机的大小无关。

[AZURE.INCLUDE [azure-virtual-machines-limits](../includes/azure-virtual-machines-limits.md)]

下表显示适用于使用资源管理器部署模型创建的虚拟机的限制，而与虚拟机的大小无关。

[AZURE.INCLUDE [azure-virtual-machines-limits-azure-resource-manager](../includes/azure-virtual-machines-limits-azure-resource-manager.md)]

## 大小表

下表显示这些虚拟机提供的大小和容量。

>[AZURE.NOTE]存储容器使用 1024^3 字节数（GB 的计量单位）表示。这有时也称为 GiB，即二进制定义。在比较使用不同进制系统的大小时，请记住二进制大小可能看上去小于十进制大小，但对于任何具体大小（例如 1 GB），二进制系统提供的容量多于十进制系统，因为 1024^3 大于 1000^3。

## 基本层

|大小 – Azure 门户\\cmdlet 和 API|CPU 核心数|内存|最大磁盘大小 – 虚拟机|最大数据磁盘每个 1023 GB）|最大IOPS（每个磁盘 300 次）|
|---|---|---|---|---|---|
|A0\\Basic\_A0|1|768 MB|20 GB|1|1x300|
|A1\\Basic\_A1|1|1\.75 GB|70 GB|2|2x300|
|A2\\Basic\_A2|2|3\.5 GB|135 GB|4|4x300|
|A3\\Basic\_A3|4|7 GB|285 GB|8|8x300|
|A4\\Basic\_A4|8|14 GB|605 GB|16|16x300|

## 标准层
### A 系列和 D 系列

|大小 – Azure 门户\\cmdlet 和 API|CPU 核心数|内存|最大磁盘大小 – 虚拟机|最大数据磁盘（每个 1023 GB）|最大IOPS（每个磁盘 500 次）|
|---|---|---|---|---|---|
|A0\\特小型|1|768 MB|20 GB|1|1x500|
|A1\\小型|1|1\.75 GB|40 GB|2|2x500|
|A2\\中型|2|3\.5 GB|60 GB|4|4x500|
|A3\\大型|4|7 GB|120 GB|8|8x500|
|A4\\特大型|8|14 GB|240 GB|16|16x500|
|A5（相同）|2|14 GB|135 GB|4|4X500|
|A6（相同）|4|28 GB|285 GB|8|8x500|
|A7（相同）|8|56 GB|605 GB|16|16x500|
|Standard\_D1（相同）|1|3\.5 GB|(SSD) = 50 GB|2|2x500|
|Standard\_D2（相同）|2|7 GB|(SSD) = 100 GB|4|4x500|
|Standard\_D3（相同）|4|14 GB|(SSD) = 200 GB|8|8x500|
|Standard\_D4（相同）|8|28 GB|(SSD) = 400 GB|16|16x500|
|Standard\_D11（相同）|2|14 GB|(SSD) = 100 GB|4|4x500|
|Standard\_D12（相同）|4|28 GB|(SSD) = 200 GB|8|8x500|
|Standard\_D13（相同）|8|56 GB|(SSD) = 400 GB|16|16x500|
|Standard\_D14（相同）|16|112 GB|(SSD) = 800 GB|32|32x500|


### 另请参阅

[Azure 订阅和服务限制、配额和约束](/documentation/articles/azure-subscription-service-limits)

<!---HONumber=76-->