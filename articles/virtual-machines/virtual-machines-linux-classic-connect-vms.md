<properties
	pageTitle="连接云服务中的 Linux 虚拟机 | Azure"
	description="将使用经典部署模型创建的 Linux 虚拟机连接到 Azure 云服务或虚拟网络。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/06/2016"
	wacn.date="08/23/2016"
	ms.author="cynthn"/>



# 如何将 Linux 虚拟机连接到虚拟网络或云服务

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]

使用经典部署模型创建的 Linux 虚拟机一律放置在云服务中。云服务充当容器，并提供唯一的公用 DNS 名称、公用 IP 地址，以及一组通过 Internet 访问虚拟机的终结点。云服务可以位于虚拟网络中，但这不是必要条件。你也可以[将 Windows 虚拟机连接到虚拟网络或云服务](/documentation/articles/virtual-machines-windows-classic-connect-vms/).

如果云服务不在虚拟网络中，就称为*独立* 云服务。独立云服务中的虚拟机只能使用其他虚拟机的公用 DNS 名称与其通信，流量通过 Internet 传送。如果云服务是在虚拟网络中，则该云服务中的虚拟机可与虚拟网络中的其他所有虚拟机通信，而不需要通过 Internet 传送任何流量。

如果你将虚拟机放在相同的独立云服务中，你仍然可以使用负载均衡和可用性集。有关详细信息，请参阅[对虚拟机进行负载均衡](/documentation/articles/load-balancer-overview/)和[管理虚拟机的可用性](/documentation/articles/virtual-machines-linux-manage-availability/)。不过，你无法组织子网上的虚拟机，也无法将独立云服务连接到本地网络。下面是一个示例：

[AZURE.INCLUDE [virtual-machines-common-classic-connect-vms](../../includes/virtual-machines-common-classic-connect-vms.md)]

##下一步

创建虚拟机后，建议[添加数据磁盘](/documentation/articles/virtual-machines-linux-classic-attach-disk/)，你的服务和工作负荷才有地方存储数据。

<!---HONumber=Mooncake_1207_2015-->