<properties
	pageTitle="在 Windows VM 上安装 MongoDB | Azure"
	description="了解如何在使用经典部署模型创建且运行 Windows Server 的 Azure VM 上安装 MongoDB。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="iainfoulds"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>  


<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/10/2017"
	wacn.date="03/01/2017"
	ms.author="iainfou"/>

#在 Windows VM 上安装 MongoDB

> [AZURE.IMPORTANT] Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型的情况。Azure 建议大多数新部署使用 Resource Manager 模型。请参阅[本文](/documentation/articles/virtual-machines-windows-classic-install-mongodb/)，了解如何使用 Resource Manager 部署模型安装和配置 MongoDB。

[MongoDB][MongoDB] 是一个流行的开源、高性能 NoSQL 数据库。本文将引导你使用 [Azure 经典管理门户][AzurePortal]创建 Windows Server 虚拟机 (VM)。然后创建数据磁盘并将其附加到 VM，再安装和配置 MongoDB。如果想使用 Azure 中现有的 VM，可直接跳到[安装并配置 MongoDB](#install-and-run-mongodb-on-the-virtual-machine)。


## 创建运行 Windows Server 的虚拟机

按照以下说明创建虚拟机。

[AZURE.INCLUDE [virtual-machines-create-WindowsVM](../../includes/virtual-machines-create-windowsvm.md)]

> [AZURE.NOTE] 在创建虚拟机时可以为 MongoDB 添加一个终结点并将其配置为如下设置：将其命名为 **Mongo**，使用 **TCP** 作为协议，并将公共端口和专用端口都设为 **27017**。

## 附加数据磁盘
若要为虚拟机提供存储，请附加数据磁盘，然后对其进行初始化，以供 Windows 使用。如果已有数据磁盘，可附加该现有磁盘；此外还可附加空磁盘。

[AZURE.INCLUDE [howto-attach-disk-windows-linux](../../includes/howto-attach-disk-windows-linux.md)]

有关初始化磁盘的说明，请参阅[如何将数据磁盘附加到 Windows 虚拟机](/documentation/articles/virtual-machines-windows-classic-attach-disk/)中的“如何在 Windows Server 中初始化新数据磁盘”。

## <a name="install-and-run-mongodb-on-the-virtual-machine"></a> 在虚拟机上安装和运行 MongoDB

[AZURE.INCLUDE [install-and-run-mongo-on-win2k8-vm](../../includes/install-and-run-mongo-on-win2k8-vm.md)]

## 摘要
在本教程中，你已了解如何创建运行 Windows Server 的虚拟机、如何远程连接到该虚拟机以及如何附加数据磁盘。你还了解了如何在基于 Windows 的虚拟机上安装和配置 MongoDB。你现在可以按照 [MongoDB 文档][MongoDocs]中高级主题中的说明，访问基于 Windows 的虚拟机上的 MongoDB。

[MongoDocs]: http://docs.mongodb.org/manual/
[MongoDB]: http://www.mongodb.org/
[AzurePortal]: http://manage.windowsazure.cn

<!---HONumber=Mooncake_Quality_Review_1215_2016-->