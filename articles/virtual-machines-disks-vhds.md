<properties
	pageTitle="关于 Linux 虚拟机的磁盘和 VHD | Azure"
	description="了解 Linux 中虚拟机磁盘和 VHD 的基础知识。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-linux""
	ms.date="03/10/2016"
	wacn.date="05/24/2016"/>

# 关于 Azure 虚拟机的磁盘和 VHD

就像其他任何计算机一样，Azure 中的虚拟机将磁盘用作存储操作系统、应用程序和数据的位置。所有 Azure 虚拟机都至少有两个磁盘，即操作系统磁盘和临时磁盘。操作系统磁盘基于映像创建，操作系统磁盘和该映像实际上都存储在 Azure 存储帐户中的虚拟硬盘 (VHD) 内。虚拟机还可以有一个或多个数据磁盘，而这些磁盘也存储为 VHD。这篇文章同时也支持 [Windows 虚拟机](/documentation/articles/virtual-machines-windows-about-disks-vhds/)。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

[AZURE.INCLUDE [virtual-machines-common-about-disks-vhds](../includes/virtual-machines-common-about-disks-vhds.md)]

## 后续步骤

-  [附加磁盘](/documentation/articles/virtual-machines-linux-classic-attach-disk/)，为你的 VM 增加额外的存储。
-  [配置软件 RAID](/documentation/articles/virtual-machines-linux-configure-raid/)。
-  [捕获 Linux 虚拟机](/documentation/articles/virtual-machines-linux-classic-capture-image/)，然后你就可以快速发布额外的 VM。


<!---HONumber=Mooncake_1207_2015-->