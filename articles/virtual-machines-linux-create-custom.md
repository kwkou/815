<properties
	pageTitle="在 Azure 中创建运行 Linux 的自定义虚拟机"
	description="了解如何在 Azure 中创建运行 Linux 的自定义虚拟机。"
	services="virtual-machines"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="07/24/2015"
	wacn.date="09/18/2015"/>

# 如何在 Azure 中创建运行 Linux 的自定义虚拟机

本主题介绍如何通过 Azure CLI 使用经典部署模型创建*自定义*虚拟机。我们将使用 Azure 上可用**映像**中的 Linux 映像。Azure CLI 命令提供以下配置选项以及其他配置选项：

- 将 VM 连接到虚拟网络
- 将 VM 添加到现有云服务
- 将 VM 添加到现有存储帐户
- 将 VM 添加到可用性集或位置

> [AZURE.IMPORTANT]如果您希望您的虚拟机使用虚拟网络，以便可以按主机名直接连接到它或者设置跨界连接，则请确保在创建虚拟机时指定虚拟网络。仅当创建虚拟机后，才能将该虚拟机配置为加入虚拟网络。有关虚拟网络的详细信息，请参阅 [Azure 虚拟网络概述](http://msdn.microsoft.com/zh-CN/library/azure/jj156007.aspx)。

<p/>
[AZURE.INCLUDE [service-management-pointer-to-resource-manager](../includes/service-management-pointer-to-resource-manager.md)]

- [创建运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-tutorial)


## 如何使用经典部署模型创建 Linux 虚拟机

[AZURE.INCLUDE [virtual-machines-create-LinuxVM](../includes/virtual-machines-create-linuxvm.md)]

<!---HONumber=70-->