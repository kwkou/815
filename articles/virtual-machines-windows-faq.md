<properties
	pageTitle="Windows VM 的常见问题 | Azure"
	description="回答了通过 Resource Manager 模型创建的 Windows 虚拟机的一些常见问题。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/16/2016"
	wacn.date="07/11/2016"/>

# 有关 Windows 虚拟机的常见问题 


本文讨论了在 Azure 中使用 Resource Manager 部署模型创建的 Windows 虚拟机的一些用户常见问题。有关本主题的 Linux 版本，请参阅[有关 Linux 虚拟机的常见问题](/documentation/articles/virtual-machines-linux-faq/)

## 我可以在 Azure VM 上运行什么程序？

所有订户都可以在 Azure 虚拟机上运行服务器软件。有关在 Azure 中运行 Microsoft 服务器软件的支持策略的信息，请参阅 [Microsoft server software support for Azure Virtual Machines](https://support.microsoft.com/kb/2721672)（对 Azure 虚拟机中的 Microsoft 服务器软件的支持）

## 使用虚拟机时，我可以使用多少存储？

每个数据磁盘的容量高达 1 TB。你可以使用的数据磁盘的数目取决于虚拟机的大小。有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)。

Azure 存储帐户提供可用于操作系统磁盘和任意数据磁盘的存储。每个磁盘都是一个 .vhd 文件，以页 blob 形式存储。有关定价详细信息，请参阅[存储定价详细信息](/home/features/storage/pricing/)。


## 如何访问我的虚拟机？

需要使用适用于 Windows VM 的远程桌面连接 (RDP) 建立远程连接。有关说明，请参阅[如何连接并登录到运行 Windows 的 Azure 虚拟机](/documentation/articles/virtual-machines-windows-connect-logon/)。除非将服务器配置为远程桌面服务会话主机，否则最多支持 2 个并发连接。


如果在使用远程桌面时遇到问题，请参阅[对与基于 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除](/documentation/articles/virtual-machines-windows-troubleshoot-rdp-connection/)。

如果你熟悉 Hyper-V，可以寻找类似于 VMConnect 的工具。Azure 不提供类似的工具，因为不支持通过控制台来访问虚拟机。

## 我是否可以使用临时磁盘（默认为 D: 驱动器）存储数据？

不应使用临时磁盘存储数据。它只是临时存储空间，因此存在丢失数据且数据不能恢复的风险。将虚拟机移到另一主机时，可能会发生这种情况。调整虚拟机大小、更新主机、主机硬件故障等都是需要移动虚拟机的原因。

如果你有应用程序需要使用 D: 驱动器号，可以重新分配驱动器号以便临时磁盘使用除 D: 以外的位置。有关说明，请参阅[更改 Windows 临时磁盘的驱动器号](/documentation/articles/virtual-machines-windows-classic-change-drive-letter/)。

## 如何更改临时磁盘的驱动器号？

在 Windows 虚拟机中，你可以通过移动页面文件和重新分配驱动器号来更改驱动器号，但需确保按特定顺序执行这些步骤。有关说明，请参阅[更改 Windows 临时磁盘的驱动器号](/documentation/articles/virtual-machines-windows-classic-change-drive-letter/)。

## 我是否可以将现有 VM 添加到可用性集？

不可以。如果你希望你的 VM 成为可用性集的一部分，需要在该集内创建 VM。目前不支持在创建 VM 之后再将其添加到可用性集。

## 我是否可以将虚拟机上传到 Azure？

是的。有关说明，请参阅[将 Windows VM 映像上传到 Azure](/documentation/articles/virtual-machines-windows-upload-image/)

## 我是否可以调整 OS 磁盘的大小？

是的。有关说明，请参阅[如何扩展 Azure 资源组中虚拟机的 OS 驱动器](/documentation/articles/virtual-machines-windows-expand-os-disk/)。

## 我是否可以复制或克隆现有的 Azure VM？

是的。有关说明，请参阅[如何在 Resource Manager 部署模型中创建 Windows 虚拟机的副本](/documentation/articles/virtual-machines-windows-specialized-image/)。

## Azure 是否支持 Linux VM？

是的。若要快速创建 Linux VM 进行试用，请参阅[使用门户在 Azure 上创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/)。


<!---HONumber=Mooncake_0704_2016-->