<properties
	pageTitle="创建自定义 Windows 虚拟机 | Microsoft Azure"
	description="了解如何从 Azure 管理门户使用经典部署模型创建自定义 Windows 虚拟机。"
	services="virtual-machines"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="01/15/2016"
	wacn.date=""/>

	
# 创建运行 Windows 的自定义虚拟机


[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]

*自定义*虚拟机只意味着你将使用**“从库中”**选项创建虚拟机，因为它提供的配置选项多于**“快速创建”**选项。这些选项包括：

- 将虚拟机连接到虚拟网络。
- 安装 Azure 虚拟机代理和 Azure 虚拟机扩展，如反恶意软件。
- 将虚拟机添加到现有云服务。
- 将虚拟机添加到现有存储帐户。
- 将虚拟机添加到可用性集。

> [AZURE.IMPORTANT] 如果你希望你的虚拟机使用虚拟网络，以便可以按主机名直接连接到它或者设置跨界连接，请确保在创建虚拟机时指定虚拟网络。仅当创建虚拟机后，才能将该虚拟机配置为加入虚拟网络。有关虚拟网络的详细信息，请参阅 [Azure 虚拟网络概述](/documentation/articles/virtual-networks-overview)。


## 创建虚拟机


[AZURE.INCLUDE [virtual-machines-create-WindowsVM](../includes/virtual-machines-create-windowsvm.md)]

<!---HONumber=Mooncake_0215_2016-->