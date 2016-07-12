<properties
	pageTitle="在 Windows VM 上安装 MongoDB | Azure"
	description="了解如何在使用经典部署模型创建的运行 Windows Server 的 Azure VM 上安装 MongoDB。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="iainfoulds"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="06/07/2016"
	wacn.date="07/11/2016"/>

#在 Windows VM 上安装 MongoDB

> [AZURE.IMPORTANT] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。

[MongoDB][MongoDB] 是一个受欢迎的开源、高性能 NoSQL 数据库。本文将指导你完成以下过程：使用 [Azure 经典管理门户][AzurePortal]创建新的 Windows Server 虚拟机 (VM)，创建数据磁盘并将其附加到 VM，然后安装并配置 MongoDB。如果想使用 Azure 中现有的 VM，可以直接跳到[安装并配置 MongoDB](#install-and-run-mongo-on-win2k8-vm)。


## 创建运行 Windows Server 的虚拟机

按照以下说明创建虚拟机。

[AZURE.INCLUDE [virtual-machines-create-WindowsVM](../includes/virtual-machines-create-windowsvm.md)]

> [AZURE.NOTE] 在创建虚拟机时可以为 MongoDB 添加一个终结点并将其配置为如下设置：将其命名为 **Mongo**，使用 **TCP** 作为协议，并将公共端口和专用端口都设为 **27017**。

## 附加数据磁盘
若要为虚拟机提供存储，请附加数据磁盘，然后对其进行初始化，以便 Windows 可以使用它。可以附加现有磁盘（如果你已有要使用的数据），或附加空磁盘。

[AZURE.INCLUDE [howto-attach-disk-windows-linux](../includes/howto-attach-disk-windows-linux.md)]

有关初始化磁盘的说明，请参阅[如何将数据磁盘附加到 Windows 虚拟机](/documentation/articles/virtual-machines-windows-classic-attach-disk/)中的“如何在 Windows Server 中初始化新数据磁盘”。

##<a name="install-and-run-mongo-on-win2k8-vm"></a> 在该虚拟机上安装和运行 MongoDB

[AZURE.INCLUDE [install-and-run-mongo-on-win2k8-vm](../includes/install-and-run-mongo-on-win2k8-vm.md)]

## 摘要
在本教程中，你已了解如何创建运行 Windows Server 的虚拟机、如何远程连接到该虚拟机以及如何附加数据磁盘。你还了解了如何在基于 Windows 的虚拟机上安装和配置 MongoDB。你现在可以访问基于 Windows 的虚拟机上的 MongoDB，按照 [MongoDB 文档][MongoDocs]中的高级主题操作。

[MongoDocs]: http://docs.mongodb.org/manual/
[MongoDB]: http://www.mongodb.org/
[AzurePortal]: http://manage.windowsazure.cn

<!---HONumber=Mooncake_0704_2016-->