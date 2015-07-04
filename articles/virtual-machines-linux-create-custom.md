<properties pageTitle="在 Azure 中创建运行 Linux 的自定义虚拟机" description="了解如何在 Azure 中创建运行 Linux 的自定义虚拟机。" services="virtual-machines" documentationCenter="" authors="KBDAzure" manager="timlt" editor="tysonn"/>

<tags ms.service="virtual-machines" ms.date="04/23/2015" wacn.date="06/26/2015"/>

#如何在 Azure 中创建运行 Linux 的自定义虚拟机

*自定义*虚拟机只意味着你将使用**“从库中”**选项创建虚拟机，因为它提供的配置选项多于**“快速创建”**选项。这些选项包括：

- 将 VM 连接到虚拟网络
- 安装 VM 代理和扩展，例如反恶意软件 
- 将 VM 添加到现有云服务
- 将 VM 添加到现有存储帐户
- 将 VM 添加到可用性集

> [AZURE.IMPORTANT]如果您希望您的虚拟机使用虚拟网络，以便可以按主机名直接连接到它或者设置跨界连接，则请确保在创建虚拟机时指定虚拟网络。仅当创建虚拟机后，才能将该虚拟机配置为加入虚拟网络。有关虚拟网络的详细信息，请参阅 [Azure 虚拟网络概述](http://msdn.microsoft.com/zh-CN/library/azure/jj156007.aspx)。

[AZURE.INCLUDE [virtual-machines-create-LinuxVM](../includes/virtual-machines-create-LinuxVM)]

<!---HONumber=61-->