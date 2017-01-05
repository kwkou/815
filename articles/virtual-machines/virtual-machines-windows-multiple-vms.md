<properties
	pageTitle="创建多个虚拟机 | Azure"
	description="用于在 Windows 上创建多个虚拟机的选项"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="gbowerman"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>  


<tags
	ms.service="virtual-machines-windows"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/25/2016"
	wacn.date="12/30/2016"
	ms.author="guybo"/>  


# 创建多个 Azure 虚拟机

在很多情况下，需要创建大量类似的虚拟机 (VM)。具体的示例包括高性能计算 (HPC)、大规模数据分析、可缩放且通常无状态的中间层或后端服务器（例如 Web 服务器），以及分布式数据库。

本文介绍用于在 Azure 中创建多个 VM 的可用选项。当手动创建一系列 VM 的简单方案无法满足需求时，可以使用这些选项。在创建数量较多的 VM 时，如果按照平时使用的过程将无法适当缩放。

若要创建许多类似的 VM，一种方法是使用 Azure Resource Manager 的_资源循环_构造。

## 资源循环

资源循环是 Azure Resource Manager 模板中的速记语法。资源循环可在循环中创建一组设置相似的资源。用户可以使用资源循环来创建多个存储帐户、网络接口或虚拟机。有关资源循环的详细信息，请参阅[使用资源循环在可用性集中创建 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-copy-index-loops/)。

## 规模的挑战

尽管资源循环可以使大规模构建云基础结构变得更轻松，同时也可以生成更精简的模板，但仍有一些难题有待克服。例如，如果使用资源循环创建 100 个虚拟机，则需要将网络接口控制器 (NIC) 关联到相应的 VM 和存储帐户。由于 VM 的数目和存储帐户的数目可能不同，因此还必须应对不同的资源循环大小。这些问题是可以解决的，不过复杂性将随着规模扩大而大幅增长。

另一项挑战发生在需要弹性缩放的基础结构时。例如，用户可能想要自动缩放基础结构，以根据工作负荷自动增加或减少 VM 数目。VM 不提供改变数目（扩展和缩减）的集成机制。如果要通过删除 VM 来达到缩减目的，即使 VM 在更新域和容错域之间保持平衡，也依然很难保证高可用性。

最后，使用资源循环时，将造成许多涉及到底层结构的资源创建诉求。当有许多创建类似资源的诉求出现后，Azure 将有潜力可以改善设计和实施优化，以此提升部署可靠性和性能。这是引入_虚拟机规模集_的原因。

## 虚拟机规模集

虚拟机规模集是一种 Azure 计算资源，用于部署和管理一组相同的 VM。由于所有 VM 配置相同，VM 规模集很容易进行缩放。只需更改集中 VM 的数目即可。也可将 VM 规模集配置为按工作负荷需求自动缩放。

对于需要扩大和缩小计算资源的应用程序，缩放操作在容错域和更新域之间进行隐式平衡。

VM 规模集可以集中配置网络、存储、虚拟机和扩展属性，不需将多个资源（例如 NIC 和 VM）关联在一起。

如需 VM 规模集的简介，请参阅[虚拟机规模集产品页](/home/features/virtual-machine-scale-sets/)。有关详细信息，请访问[虚拟机规模集文档](/documentation/services/virtual-machine-scale-sets/)。

<!---HONumber=Mooncake_Quality_Review_1202_2016-->