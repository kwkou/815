<properties
	pageTitle="创建 Linux VM | Azure"
	description="了解如何使用运行 Linux 操作系统的经典部署模型创建自定义虚拟机。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="iainfoulds"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="03/28/2016"
	wacn.date="05/24/2016"/>


# 如何创建自定义 Linux VM

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用[资源管理器模型](/documentation/articles/virtual-machines-linux-quick-create-cli/)。

本主题介绍如何通过 Azure CLI 使用经典部署模型创建*自定义*虚拟机（VM）。我们将使用 Azure 上可用**映像**中的 Linux 映像。Azure CLI 命令提供以下配置选项以及其他配置选项：

- 将 VM 连接到虚拟网络
- 将 VM 添加到现有云服务
- 将 VM 添加到现有存储帐户
- 将 VM 添加到可用性集或位置

> [AZURE.IMPORTANT]如果您希望您的虚拟机使用虚拟网络，以便可以按主机名直接连接到它或者设置跨界连接，则请确保在创建虚拟机时指定虚拟网络。仅当创建虚拟机后，才能将该虚拟机配置为加入虚拟网络。有关虚拟网络的详细信息，请参阅 [Azure 虚拟网络概述](/documentation/articles/virtual-networks-overview/)。


## 如何使用经典部署模型创建 Linux 虚拟机

[AZURE.INCLUDE [virtual-machines-create-LinuxVM](../includes/virtual-machines-create-linuxvm.md)]

<!---HONumber=Mooncake_1207_2015-->