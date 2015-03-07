<properties linkid="manage-linux-tutorial-vm-from-galllery" urlDisplayName="Create a virtual machine" pageTitle="创建在 Azure 中运行 Linux 的虚拟机" metaKeywords="Azure Linux vm, Linux vm" description="了解如何捕获运行 Linux Azure 虚拟机 (VM) 的映像。 " metaCanonical="" services="virtual-machines" documentationCenter="" title="" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />


#创建运行 Linux 的虚拟机 

当您使用 Azure 管理门户中的映像库时，创建运行 Linux 的虚拟机很容易。本指南演示如何执行此操作，并假定您没有使用过 Azure。

> [WACOM.NOTE] 即使您不需要任何熟悉 Azure 虚拟机，若要完成本教程中，您需要一个 Azure 帐户。只需几分钟即可创建一个免费的试用帐户。有关详细信息，请参阅[创建 Azure 帐户](/zh-cn/develop/php/tutorials/create-a-windows-azure-account/)。 

本教程介绍了：

- [有关在 Azure 中的虚拟机][]
- [如何创建虚拟机][]
- [如何登录到虚拟机创建表后][]
- [如何将数据磁盘附加到新的虚拟机][]

**重要**：本教程创建的是不连接到虚拟网络的虚拟机。如果您希望虚拟机使用虚拟网络，必须指定虚拟网络，当您创建虚拟机。有关虚拟网络的详细信息，请参阅[Azure 虚拟网络概述](http://msdn.microsoft.com/library/azure/jj156007.aspx)。

## <a id="virtualmachine"> </a>有关在 Azure 中的虚拟机 # #

Azure 中的虚拟机是云中，您可以控制和管理的服务器。在 Azure 中创建虚拟机后，可以删除并重新创建它，只要需要，并且可以访问虚拟机，就像您办公室中执行与服务器。可使用虚拟硬盘 (VHD) 文件创建虚拟机。以下类型的 Vhd 用于虚拟机：

- **图像**-作为模板用于创建新的虚拟机的 VHD。映像是一个模板，因为它没有设置运行的虚拟机，例如计算机名称和用户帐户设置等特定设置。如果您创建虚拟机使用的映像，操作系统磁盘是自动创建新的虚拟机。
- **磁盘**-磁盘是您可以启动和安装作为操作系统的运行版本的 VHD。映像在进行设置后，即成为磁盘。使用映像创建虚拟机时始终会创建一个磁盘。附加到的任何 VHD 虚拟化硬件和程序运行时使用服务的一部分是磁盘

使用映像创建虚拟机时可采用以下方法：

- 通过使用 Azure 管理门户的映像库中提供的映像创建虚拟机。
- 创建和上载包含到 Azure，映像的.vhd 文件，然后创建虚拟机使用的映像。有关创建和上载自定义映像的详细信息，请参阅[创建和上载包含 Linux 操作系统的虚拟硬盘](/zh-cn/documentation/articles/virtual-machines-linux-create-upload-vhd/).

每个虚拟机驻留在云服务中，通过本身，或与其他虚拟机组合在一起。可以将虚拟机放在相同的云服务，以使虚拟机到虚拟机之间的负载平衡网络流量与彼此、 通信和维护高可用性的计算机中。有关云服务和虚拟机的详细信息，请参阅中的"执行模型"一节[Azure 简介](http://azure.microsoft.com/zh-cn/documentation/articles/fundamentals-introduction-to-azure/?fb=zh-cn#models)。

## <a id="custommachine"> </a>如何创建虚拟机 # #

本教程使用**从库**方法来创建虚拟机，因为它使您更多选项比**快速创建**方法。如果需要，您可以选择连接的资源、 DNS 名称和网络连接。

[WACOM.INCLUDE [virtual-machines-create-LinuxVM](../includes/virtual-machines-create-LinuxVM.md)]

[WACOM.INCLUDE [virtual-machines-Linux-tutorial-log-on-attach-disk](../includes/virtual-machines-Linux-tutorial-log-on-attach-disk.md)]


##后续步骤 

若要了解有关在 Azure 上 Linux 的详细信息，请参阅以下文章：

- [在 Azure 上的 Linux 简介](/zh-cn/documentation/articles/introduction-linux/)

- [如何使用针对 Mac 和 Linux 的 Azure 命令行工具](/zh-cn/documentation/articles/xplat-cli/)

- [有关 Azure 虚拟机配置设置](http://msdn.microsoft.com/library/azure/dn763935.aspx)


[下一步行动]: #next
[有关在 Azure 中的虚拟机]: #virtualmachine
[如何创建虚拟机]: #custommachine
[如何登录到虚拟机创建表后]: #logon
[如何将数据磁盘附加到新的虚拟机]: #attachdisk
