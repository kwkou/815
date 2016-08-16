<properties
	pageTitle="Linux 虚拟机准则 |Azure"
	description="了解将 Linux 虚拟机部署到 Azure 中的关键设计和实施准则"
	documentationCenter=""
	services="virtual-machines-linux"
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="06/22/2016"
	wacn.date="08/08/2016"/>

# 虚拟机准则

[AZURE.INCLUDE [virtual-machines-linux-infrastructure-guidelines-intro](../../includes/virtual-machines-linux-infrastructure-guidelines-intro.md)]

本文重点介绍在 Azure 环境中创建和管理虚拟机 (VM) 所需的规划步骤。

## VM 的实施准则
决策：

- 你的基础结构的各应用程序层和组件需要多少 VM？
- 每个 VM 需要哪些 CPU 和内存资源，各有什么存储空间要求？

任务：

- 定义应用程序的工作负荷和 VM 需要的资源。
- 为每台 VM 配置适当的 VM 大小和存储类型以满足资源需求。
- 为基础结构的不同层和组件定义资源组。
- 定义 VM 命名约定。
- 使用 Azure CLI、Web 门户或 Resource Manager 模板创建虚拟机。

## 虚拟机

Azure 环境中的主要组件之一很可能是 VM。你将在其中运行应用程序、数据库、身份验证服务等。

务必了解[不同 VM 大小](/documentation/articles/virtual-machines-linux-sizes/)以便从性能和成本的角度确定环境的适当大小。如果 VM 的 CPU 核心数或内存不足，则无论应用程序的设计和开发如何完善，其性能都将受到影响。在确定要用于基础结构中每个组件的 VM 大小之前，请首先检查每个 VM 系列的建议工作负荷。完成部署后，你可以[更改 VM 大小](/documentation/articles/virtual-machines-linux-change-vm-size/)。

存储在 VM 性能中扮演着重要角色。你可以使用应用常规旋转磁盘的标准存储，也可以使用应用 SSD 磁盘、可处理高 I/O 工作负荷、性能最佳的高级存储。置于 VM 大小，选择存储介质时应考虑成本。可参阅[“存储基础结构准则”文章](/documentation/articles/virtual-machines-linux-infrastructure-storage-solutions-guidelines/)，以了解如何设计适当存储以使 VM 发挥出最佳性能。


## 资源组
VM 等组件在逻辑上组合在一起，以便使用 [Azure 资源组](/documentation/articles/resource-group-overview/)进行管理和维护。使用资源组，可以创建、管理和监视组成给定应用程序的所有资源。你还可以实现[基于角色的访问控制](/documentation/articles/role-based-access-control-what-is/)，对团队中的其他人授予仅其所需资源的访问权限。花时间规划出资源组和角色分配。实际设计和实现资源组还有其他方法，因此请务必阅读[“资源组准则”文章](/documentation/articles/virtual-machines-linux-infrastructure-resource-groups-guidelines/)以了解如何以最佳方式构建你的 VM。


## 模板 
你可以构建由声明性 JSON 文件定义的模板以创建 VM。模板通常还将随 VM 本身生成所需存储、网络、网络接口、IP 寻址等。你可以使用模板来创建一致的可重现开发和测试环境以轻松复制生产环境，反之亦然。可参阅有关[构建和使用模板](/documentation/articles/resource-group-overview/#template-deployment)的详细信息，以了解如何用其创建和部署 VM。


## <a name="next-steps"></a> 后续步骤
[AZURE.INCLUDE [virtual-machines-linux-infrastructure-guidelines-next-steps](../../includes/virtual-machines-linux-infrastructure-guidelines-next-steps.md)]

<!---HONumber=Mooncake_0801_2016-->