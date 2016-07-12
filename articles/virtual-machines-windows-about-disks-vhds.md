<properties
	pageTitle="关于 Windows 虚拟机的磁盘和 VHD | Azure"
	description="了解 Windows 中虚拟机磁盘和 VHD 的基础知识。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="03/10/2016"
	wacn.date="05/24/2016"/>

# 关于 Azure 虚拟机的磁盘和 VHD

就像其他任何计算机一样，Azure 中的虚拟机将磁盘用作存储操作系统、应用程序和数据的位置。所有 Azure 虚拟机都至少有两个磁盘，即操作系统磁盘和临时磁盘。操作系统磁盘基于映像创建，操作系统磁盘和该映像实际上都存储在 Azure 存储帐户中的虚拟硬盘 (VHD) 内。虚拟机还可以有一个或多个数据磁盘，而这些磁盘也存储为 VHD。这篇文章同时也支持 [Linux 虚拟机](/documentation/articles/virtual-machines-linux-about-disks-vhds/)。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

[AZURE.INCLUDE [virtual-machines-common-about-disks-vhds](../includes/virtual-machines-common-about-disks-vhds.md)]

## 后续步骤

-  [捕获 Windows 虚拟机](/documentation/articles/virtual-machines-windows-classic-capture-image/)
-  [上传 Windows VM 镜像到 Azure](/documentation/articles/virtual-machines-windows-classic-createupload-vhd/)
-  [更改 Windows 临时磁盘的驱动器号](/documentation/articles/virtual-machines-windows-classic-change-drive-letter/)

<!---HONumber=Mooncake_1207_2015-->