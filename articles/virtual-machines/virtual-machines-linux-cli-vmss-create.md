<properties
	pageTitle="什么是 VM 规模集？| Azure"
	description="了解 VM 规模集。"
	keywords="linux 虚拟机, 虚拟机规模集" 
	services="virtual-machines-linux"
	documentationCenter=""
	authors="gatneil"
	manager="madhana"
	editor="tysonn"
	tags="azure-resource-manager" />

<tags
	ms.service="virtual-machine-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/24/2016"
	wacn.date="03/06/2017"
	ms.author="gatneil"/>

# 什么是虚拟机规模集？

虚拟机规模集可让你以集的形式管理多个 VM。概括而言，规模集具有以下优点和缺点：

优点：

1. 高可用性。每个规模集将它的 VM 放入具有 5 个容错域 (FD) 和 5 个更新域 (UD) 的可用性集，以确保可用性（有关 FD 和 UD 的详细信息，请参阅 [VM availability（VM 可用性）](/documentation/articles/virtual-machines-linux-manage-availability/)）。 
2. 与 Azure 负载均衡器和应用网关轻松集成。
4. 简化 VM 的部署、管理和清理。
5. 支持常用的 Windows 和 Linux 版本以及自定义映像。

缺点：

1. 无法将数据磁盘连接到规模集中的 VM 实例。你必须使用 Blob 存储、Azure 文件、Azure 表或其他存储解决方案。

## 使用 Azure CLI 快速创建

[AZURE.INCLUDE [cli-vmss-quick-create](../../includes/virtual-machines-linux-cli-vmss-quick-create-include.md)]

## 后续步骤

有关一般信息，请参阅 [main landing page for scale sets（规模集的主要登录页）](/home/features/virtual-machine-scale-sets/)。

有关其他文档，请参阅 [main documentation page for scale sets（规模集的主要文档页）](/documentation/articles/virtual-machine-scale-sets-overview/)。

有关使用规模集的示例 Resource Manager 模板，请在 [Azure 快速入门模板 github 存储库](https://github.com/Azure/azure-quickstart-templates)中搜索“vmss”。


<!---HONumber=Mooncake_0425_2016-->