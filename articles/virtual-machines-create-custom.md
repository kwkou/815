<properties linkid="manage-linux-howto-custom-create-vm" urlDisplayName="Custom create a VM" pageTitle="在 Azure 中自定义创建运行 Linux 的虚拟机" metaKeywords="Azure custom vm, creating custom vm" description="了解如何在 Azure 中创建自定义虚拟机。" metaCanonical="/zh-cn/manage/windows/how-to-guides/custom-create-a-vm/" services="virtual-machines" documentationCenter="" title="" authors="kathydav" solutions="" manager="dongill" editor="tysonn" />


#如何创建自定义虚拟机

*custom* 虚拟机仅指您使用"从库中"****选项创建的虚拟机，因为与"快速创建"****选项相比，它为您提供了更多的配置选项。这些选项包括：

- 将虚拟机连接到虚拟网络
- 为反恶意软件等安装虚拟机代理和扩展 
- 将虚拟机添加到现有的云服务 
- 将虚拟机添加到可用性集中

**重要说明**：如果您希望您的虚拟机使用虚拟网络，以便可以按主机名直接连接到它或者设置跨界连接，则请确保在创建虚拟机时指定虚拟网络。仅当创建虚拟机后，才能将该虚拟机配置为加入虚拟网络。有关虚拟网络的详细信息，请参阅 [Azure 虚拟网络概述](http://msdn.microsoft.com/library/azure/jj156007.aspx)。

[WACOM.INCLUDE [virtual-machines-create-WindowsVM](../includes/virtual-machines-create-WindowsVM.md)]


