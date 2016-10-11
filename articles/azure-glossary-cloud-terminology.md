<properties
    pageTitle="Azure 词汇表 - Azure 字典 | Azure"
    description="使用 Azure 词汇表来理解 Azure 平台上的云术语。这份简短的 Azure 字典提供 Azure 通用云术语的定义。"
    keywords="Azure 字典, 云术语, Azure 词汇表, 术语定义, 云名词"
    services="na"
    documentationCenter="na"
    authors="MonicaRush"
    manager="jhubbard"
    editor=""/>

<tags
    ms.service="multiple"
    ms.date="08/03/2016"
    wacn.date="09/26/2016"/>


# Azure 词汇表：Azure 平台上的云术语字典

Azure 词汇表是适用于 Azure 平台的简短云术语字典。

## 查找服务定义和其他云术语

* 有关 Azure 服务的定义及其 AWS 对等项，请参阅 [Azure and Amazon Web Services（Azure 和 Amazon Web Services）](https://azure.microsoft.com/campaigns/azure-vs-aws/mapping/)。

* 有关一般行业云术语，请参阅 [Cloud computing terms（云计算术语）](https://azure.microsoft.com/zh-cn/overview/cloud-computing-dictionary/)。

Azure 术语以及上述两份参考文档提供了适用于 Azure 和云行业的端到端分类法。

## Azure 词汇表


### <a name="account"></a>帐户  
工作或学校帐户，或者个人 Microsoft 帐户，可用于访问和管理 Azure 订阅。  
另请参阅 [How Azure subscriptions are associated with Azure Active Directory（Azure 订阅与 Azure Active Directory 的关联方式）](/documentation/articles/active-directory-how-subscriptions-associated-directory/)


### <a name="availabilityset"></a>可用性集  
可一起管理的、用于提供应用程序冗余和可靠性的虚拟机集合。使用可用性集可确保在计划内或计划外维护事件期间，至少有一个虚拟机可用。  
另请参阅 [Manage the availability of Windows virtual machines（管理 Windows 虚拟机的可用性）](/documentation/articles/virtual-machines-windows-manage-availability/)或 [Manage the availability of Linux virtual machines（管理 Linux 虚拟机的可用性）](/documentation/articles/virtual-machines-linux-manage-availability/)


### <a name="classicmodel"></a>Azure 经典部署模型  
用于在 Azure 中部署资源的两个[部署模型](/documentation/articles/resource-manager-deployment-model/)中的一个（新模型是 Azure Resource Manager）。可以将一些 Azure 资源部署在其中一个模型中，并将另一些资源部署在这两个模型中。适用于单个 Azure 资源的指导详细说明了可以使用哪些模型来部署资源。


### <a name="cli"></a>Azure 命令行接口 (CLI)  
一个[命令行接口](/documentation/articles/xplat-cli-install/)，可用于从 Windows、OSX 和 Linux 电脑管理 Azure 服务。


### <a name="powershell"></a>Azure PowerShell  
一个[命令行接口](/documentation/articles/powershell-install-configure/)，可用于通过命令行从 Windows 电脑管理 Azure 服务。某些服务或服务功能只能通过 PowerShell 或 CLI 来管理。适用于每个 Azure 资源的指导详细说明了可以使用哪些模型来部署资源。  
另请参阅 [How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure/)。


### <a name="armmodel"></a>Azure Resource Manager 部署模型  
用于在 Azure 中部署资源的两个[部署模型](/documentation/articles/resource-manager-deployment-model/)中的一个（另一个是经典部署模型）。可以将一些 Azure 资源部署在其中一个模型中，并将另一些资源部署在这两个模型中。适用于单个 Azure 资源的指导详细说明了可以使用哪些模型来部署资源。


### <a name="faultdomain"></a>容错域  
可用性集中可能会同时发生故障的虚拟机集合。一个例子是机架中共用一个电源和网络交换机的计算机组。在 Azure 中，可用性集内的虚拟机将自动分布在多个容错域中。  
另请参阅 [Manage the availability of Windows virtual machines（管理 Windows 虚拟机的可用性）](/documentation/articles/virtual-machines-windows-manage-availability/)或 [Manage the availability of Linux virtual machines（管理 Linux 虚拟机的可用性）](/documentation/articles/virtual-machines-linux-manage-availability/)


### <a name="georeplication"></a>异地复制  
在区域对中自动复制内容（例如 Blob、表和队列）的过程。  
另请参阅 [Active Geo-Replication for Azure SQL Database（Azure SQL 数据库的活动异地复制）](/documentation/articles/sql-database-geo-replication-overview/)


### <a name="image"></a>映像  
包含操作系统和应用程序配置的文件，可用于创建任意数目的虚拟机。Azure 中有两种类型的映像：VM 映像和 OS 映像。VM 映像包括在创建该映像时附加到虚拟机的操作系统和所有磁盘。OS 映像仅包含通用化操作系统，而不包含任何数据磁盘配置。  
另请参阅 [Navigate and select Windows virtual machine images in Azure with PowerShell or the CLI（使用 PowerShell 或 CLI 在 Azure 中浏览和选择 Windows 虚拟机映像）](/documentation/articles/virtual-machines-windows-cli-ps-findimage/)


### <a name="limits"></a>限制  
可以创建的资源数目或可实现的性能基准。限制通常与订阅、服务和产品关联。  
另请参阅 [Azure subscription and service limits, quotas, and constraints（Azure 订阅和服务限制、配额与约束）](/documentation/articles/azure-subscription-service-limits/)


### <a name="loadbalancer"></a>负载平衡器  
在网络中的计算机之间分散传入流量的资源。在 Azure 中，负载平衡器将流量分散到负载平衡器集内定义的虚拟机。[负载平衡器](/documentation/articles/load-balancer-overview/)可以面向 Internet，也可以是内部的。


### <a name="portal"></a>经典管理门户  
用于部署和管理 Azure 服务的安全 Web 门户 ：[Azure 经典管理门户](http://manage.windowsazure.cn/)


### <a name="resource"></a>资源  
属于 Azure 解决方案的一部分的项。每个 Azure 服务可让你部署不同类型的资源，例如数据库或虚拟机。  
另请参阅 [Azure Resource Manager overview（Azure Resource Manager 概述）](/documentation/articles/resource-group-overview/)


### <a name="resourcegroup"></a>资源组  
Resource Manager 中的容器，用于保存应用程序的相关资源。资源组可以包含应用程序的所有资源，也可以只包含逻辑分组在一起的资源。你可以根据对组织有利的原则，决定如何将资源分配到资源组。  
另请参阅 [Azure Resource Manager overview（Azure Resource Manager 概述）](/documentation/articles/resource-group-overview/)


### <a name="rmtemplate"></a>Resource Manager 模板  
一个 JSON 文件，它以声明方式定义一个或多个 Azure 资源，并定义所部署资源之间的依赖关系。使用模板能够以一致方式反复部署资源。  
另请参阅 [Authoring Azure Resource Manager templates](/documentation/articles/resource-group-authoring-templates/)（创作 Azure Resource Manager 模板）


### <a name="resourceprovider"></a>资源提供程序  
一种服务，提供可以通过 Resource Manager 进行部署和管理的资源。每个资源提供程序提供用于处理所部署资源的操作。可以通过 Azure 经典管理门户、Azure PowerShell 和多个编程 SDK 来访问资源提供程序。  
另请参阅 [Azure Resource Manager overview（Azure Resource Manager 概述）](/documentation/articles/resource-group-overview/)


### <a name="sla"></a>服务级别协议 (SLA)  
用于描述 Microsoft 在运行时间和连接性方面所做承诺的协议。每个 Azure 服务都有具体的 SLA。  
另请参阅 [Service Level Agreements（服务级别协议）](/support/legal/sla/)


### <a name="storageaccount"></a>存储帐户  
存储帐户授予你访问 Azure 存储空间中的 Azure Blob、队列、表和文件服务的权限。你的存储帐户为你的 Azure 存储空间数据对象提供唯一的命名空间。  
另请参阅 [About Azure storage accounts关于 Azure 存储帐户）](/documentation/articles/storage-create-storage-account/)（


### <a name="subscription"></a>订阅  
客户与 Microsoft 之间的协议，可让客户获取 Azure 服务。订阅定价和相关条款由针对订阅选择的优惠条件控制。另请参阅 [Microsoft Online Subscription Agreement（Microsoft 在线订阅协议）](https://azure.microsoft.com/support/legal/subscription-agreement/)。  
另请参阅 [How Azure subscriptions are associated with Azure Active Directory（Azure 订阅与 Azure Active Directory 的关联方式）](/documentation/articles/active-directory-how-subscriptions-associated-directory/)


### <a name="tag"></a>标记  
一个索引编制术语，让你根据管理或计费要求为资源分类。如果你有一系列复杂的资源组和资源，并想要以最有利的方式可视化这些资产，则可以使用标记。例如，你可以标记组织中充当类似角色或者属于同一部门的资源。  
另请参阅 [Using tags to organize your Azure resources（使用标记来组织 Azure 资源）](/documentation/articles/resource-group-using-tags/)


### <a name="updatedomain"></a>更新域  
可用性集中可同时更新的虚拟机集合。同一个更新域中的虚拟机将在计划维护期间一起重新启动。Azure 永远不会一次性重新启动多个更新域。也称为升级域。  
另请参阅 [Manage the availability of Windows virtual machines（管理 Windows 虚拟机的可用性）](/documentation/articles/virtual-machines-windows-manage-availability/)或 [Manage the availability of Linux virtual machines（管理 Linux 虚拟机的可用性）](/documentation/articles/virtual-machines-linux-manage-availability/)


### <a name="vm"></a>虚拟机  
运行操作系统的物理计算机的软件实现。多个虚拟机可以在同一个硬件上同时运行。在 Azure 中，虚拟机具有不同的大小。  
另请参阅 [Virtual Machines documentation（虚拟机文档）](https://azure.microsoft.com/documentation/services/virtual-machines/)


### <a name="vmextension"></a>虚拟机扩展  
一个实现行为或功能的资源，可帮助其他程序正常工作，或提供与运行中计算机交互的能力。例如，你可以使用 VM 访问扩展来重置或修改 Azure 虚拟机上的远程访问值。  
另请参阅 [About virtual machine extensions and features (Windows)（关于虚拟机的扩展和功能 (Windows)）](/documentation/articles/virtual-machines-windows-extensions-features/)或 [About virtual machine extensions and features (Linux)（关于虚拟机的扩展和功能 (Linux)）](/documentation/articles/virtual-machines-linux-extensions-features/)


### <a name="vnet"></a>虚拟网络  
在 Azure 资源之间提供连接并与其他所有 Azure 租户隔离的网络。它可以通过 [Azure VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)连接到其他 Azure 虚拟网络，以及使用[多个选项](/documentation/articles/vpn-gateway-plan-design/)连接到本地网络。你可以完全控制该网络中的 IP 地址块、DNS 设置、安全策略和路由表。  
另请参阅 [Virtual Network Overview（虚拟网络概述）](/documentation/articles/virtual-networks-overview/)


<!---HONumber=Mooncake_0523_2016-->