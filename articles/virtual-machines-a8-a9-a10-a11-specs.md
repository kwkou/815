<properties
	 pageTitle="关于 A8、A9、A10 和 A11 实例 | Windows Azure"
	 description="了解有关使用 Azure A8、A9、A10 和 A11 计算密集型实例的背景信息和注意事项。"
	 services="virtual-machines, cloud-services"
	 documentationCenter=""
	 authors="dlepow"
	 manager="timlt"
	 editor=""/>
<tags 
	ms.service="virtual-machines"
	ms.date="07/22/2015" 
	wacn.date="09/15/2015"/>

# 关于 A8、A9、A10 和 A11 计算密集型实例

本文提供了有关使用 Azure A8、A9、A10 和 A11 实例（亦称*计算密集型*实例）的背景信息和注意事项。这些实例的主要功能包括：

* **高性能硬件** – 运行这些实例的 Azure 数据中心硬件专为计算密集型和网络密集型应用程序而设计和优化，包括高性能计算 (HPC) 群集应用程序、建模和模拟。

* **MPI 应用程序的 RDMA 网络连接** – 当配置有必要的网络驱动程序时，A8 和 A9 实例可以通过 Azure 中基于远程直接内存访问 (RDMA) 技术的低延迟、高吞吐量网络与其他 A8 和 A9 实例通信。此功能可以提高使用受支持的 Linux 或 Windows 消息传递接口 (MPI) 实现的应用程序的性能。

* **支持 Linux 和 Windows HPC 群集** – 在 Azure 中的 A8、A9、A10 和 A11 实例上部署群集管理和作业计划软件，以创建一个独立的 HPC 群集或将容量添加到本地群集。与其他 Azure VM 大小一样，A8、A9、A10 和 A11 实例支持标准或自定义 Windows Server 和 Linux 操作系统映像或 Azure VM (IaaS) 中的 Azure 资源管理器模板，或云服务中的 Azure 来宾 OS 版本（PaaS，仅限于 Windows Server）。

>[AZURE.NOTE]A10 和 A11 实例与 A8 和 A9 实例具有相同的性能优化和规范。但是，它们不包括对 Azure 中 RDMA 网络的访问权限。它们为节点间不需要保持恒定和低延迟通信的 HPC 应用程序（也称为参数化或易并行应用程序）而设计。对于运行 MPI 应用程序以外的工作负荷，A8 和 A9 实例不会访问 RDMA 网络，并且功能上等效于 A10 和 A11 实例。


## 规范

### CPU 和内存

Azure A8、A9、A10 和 A11 计算密集型实例的特点在于高速、多核 CPU 和大量内存，如下表所示。

大小 | CPU | 内存
------------- | ----------- | ----------------
A8 和 A10 | Intel Xeon E5-2670<br/>8 核 @ 2.6 GHz | DDR3-1600 MHz<br/>56 GB
A9 和 A11 | Intel Xeon E5-2670<br/>16 核 @ 2.6 GHz | DDR3-1600 MHz<br/>112 GB


>[AZURE.NOTE]其他处理器详细信息（包括支持的指令集扩展）都可在 Intel.com 网站上找到。有关 VM 存储容量和磁盘详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-size-specs)。

### 网络适配器

A8 和 A9 实例具有两个网络适配器，可连接到以下两个后端 Azure 网络。


网络 | 说明
-------- | -----------
10-Gbps 以太网 | 连接到 Azure 服务（如 Azure 存储空间和 Azure 虚拟网络）和 Internet。
32-Gbps 后端，支持 RDMA | 支持在单个云服务或可用性集内实例之间的低延迟、高吞吐量应用程序通信。仅为 MPI 流量保留。


>[AZURE.IMPORTANT]在 IaaS 中运行 Linux 的 A8 和 A9 VM 上，目前只能通过使用 SUSE Linux Enterprise Server 12 (SLES 12) 上的 Azure Linux RDMA 和 Intel MPI Library 5.0 的应用程序访问 RDMA 网络。在 IaaS 或 PaaS 中运行 Windows Server 的 A8 和 A9 实例中，目前只能通过使用 Microsoft Network Direct 接口的应用程序访问 RDMA 网络。有关其他要求，请参阅本文中的 [RDMA 网络访问权限](#access-the-rdma-network)。

A10 和 A11 实例具有连接到 Azure 服务和 Internet 的单个 10-Gbps 以太网网络适配器。

## 有关 Azure 订阅的注意事项

* **Azure 帐户** – 如果要部署非少量的计算密集型实例，请考虑使用即付即用订阅或其他购买选项。<!--还可以使用你的 MSDN 订阅。请参阅 [MSDN 订户的 Azure 权益](/pricing/member-offers/msdn-benefits-details/)。-->如果你使用的是 [Azure 免费试用版](http://www.windowsazure.cn/pricing/1rmb-trial/)，则仅可以使用有限数量的 Azure 计算核心。

* **内核配额** – 你可能需要在 20 个的默认内核配额的基础上增加 Azure 订阅中的内核配额，对于许多 8 核心或 16 核心实例的方案，20 个的默认内核是不够的。对于初始测试，可考虑请求将配额增加到 100 个内核。若要执行此操作，请打开免费支持票证，如[了解 Azure 限制与增加](http://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)中所示。

>[AZURE.NOTE]Azure 配额为信用额度，而不是容量保障。仅收取你使用的内核的费用。

* **地缘组** – 当前，对于大多数新部署不建议使用地缘组。但是请注意，如果正在使用包含 A8 – A11 以外大小的实例的地缘组，则无法对 A8 – A11 实例使用该地缘组，反之亦然。

* **虚拟网络** – Azure 虚拟网络不需要使用计算密集型实例。但是，如果需要访问本地资源（例如，应用程序许可证服务器），则对于许多 IaaS 方案或者一个站点到站点连接，可能至少需要一个基于云的 Azure 虚拟网络。需要先创建一个新的（区域）虚拟网络，再部署实例。不支持将 A8、A9、A10 或 A11 VM 添加到地缘组中的虚拟网络。有关详细信息，请参阅[如何创建虚拟网络 (VNet)](/documentation/articles/virtual-networks-create-vnet) 和[为虚拟网络配置站点到站点 VPN 连接](/documentation/articles/vpn-gateway-site-to-site-create)。

* **云服务或可用性集** – 若要通过 RDMA 网络进行连接，A8 和 A9 实例必须部署在同一云服务（用于在 Azure 资源管理器中具有基于 Linux 的 VM 或基于 Windows 的 VM 的 IaaS 方案；或者具有 Windows Server 的 PaaS 方案）或同一可用性集（用于 Azure 资源管理器中基于 Linux 的 VM 或基于 Windows 的 VM）中。

## 使用 HPC Pack 时的注意事项

### HPC Pack 和 Linux 的注意事项

[HPC Pack](https://technet.microsoft.com/zh-cn/library/jj899572.aspx) 是用于 Windows 的 Microsoft 免费 HPC 群集和作业管理解决方案。从 HPC Pack 2012 R2 Update 2 开始，HPC Pack 支持多个 Linux 分发在 Windows Server 头节点管理的 Azure VM 中部署的计算节点上运行。使用 HPC Pack 的最新版本，可以部署基于 Linux 的群集，该群集可运行访问 Azure 中的 RDMA 网络的 MPI 应用程序。有关详细信息，请参阅 [HPC Pack 文档](/documentation/articles/virtual-machines-linux-cluster-hpcpack)。

### HPC Pack 和 Windows 的注意事项

通过 Windows Server 使用 A8、A9、A10 和 A11 实例时，并不需要使用 HPC Pack，但在 Azure 中创建基于 Windows HPC Server 的群集时，它是推荐使用的工具。在 A8 和 A9 实例的情况下，HPC Pack 是运行可访问 Azure 中的 RDMA 网络的基于 Windows 的 MPI 应用程序的最有效方法。HPC Pack 中包括适用于 Windows 消息传递接口的 Microsoft 实现的运行时环境。

有关通过 Windows Server 上 的 HPC Pack，在 IaaS 和 PaaS 方案中部署和使用计算密集型实例的详细信息和清单，请参阅 [A8 和 A9 计算密集型实例：使用 HPC Pack 快速启动](https://msdn.microsoft.com/zh-cn/library/azure/dn594431.aspx)。

## RDMA 网络访问权限

### Linux A8 和 A9 VM 的访问权限

在单个云服务或可用性集中，A8 和 A9 实例可访问 Azure 中的 RDMA 网络，以运行使用 Linux RDMA 驱动程序在实例间通信的 MPI 应用程序。目前，仅 [Intel MPI Library 5.0](https://software.intel.com/zh-cn/intel-mpi-library/) 支持 Azure Linux RDMA。

>[AZURE.NOTE]目前，Azure Linux RDMA 驱动程序不可通过驱动程序扩展安装。仅当使用来自 Azure 应用商店的已启用 RDMA 的 SLES 12 映像时，它们才可用。

请参阅下表，了解 Linux MPI 应用程序要能够访问计算节点 (IaaS) 群集中的 RDMA 网络所需具备的先决条件。有关部署选项和配置步骤，请参阅[设置 Linux RDMA 群集以运行 MPI 应用程序](/documentation/articles/virtual-machines-linux-cluster-rdma)。

先决条件 | 虚拟机 (IaaS)
------------ | -------------
操作系统 | 来自 Azure 应用商店的 SLES 12 HPC 映像
MPI | Intel MPI Library 5.0

### Windows A8 和 A9 实例的访问权限

在单个云服务或可用性集中，A8 和 A9 实例可访问 Azure 中的 RDMA 网络，以运行使用 Microsoft Network Direct 接口在实例间通信的 MPI 应用程序。A10 和 A11 实例不包括对 RDMA 网络的访问权限。

请参阅下表，了解 MPI 应用程序要能够访问虚拟机 (IaaS) 中的 RDMA 网络和 A8 或 A9 实例的云服务 (PaaS) 部署所需具备的先决条件。对于典型的部署方案，请参阅 [A8 和 A9 计算密集型实例：使用 HPC Pack 快速启动](https://msdn.microsoft.com/zh-cn/library/azure/dn594431.aspx)。


先决条件 | 虚拟机 (IaaS) | 云服务 (PaaS)
---------- | ------------ | -------------
操作系统 | Windows Server 2012 R2 或 Windows Server 2012 | Windows Server 2012 R2、Windows Server 2012 或 Windows Server 2008 R2 来宾 OS 系列
MPI | MS-MPI 2012 R2 或更高版本，独立安装或通过 HPC Pack 2012 R2 或更高版本安装<br/><br/>Intel MPI Library 5.0 | MS-MPI 2012 R2 或更高版本，通过 HPC Pack 2012 R2 或更高版本安装<br/><br/>Intel MPI Library 5.0


>[AZURE.NOTE]对于 IaaS 方案，[HpcVmDrivers 扩展](https://msdn.microsoft.com/zh-cn/library/azure/dn690126.aspx)必须添加到 VM 以安装 RDMA 连接所需的 Windows 驱动程序。


## 其他须知项

* A8 – A11 VM 大小仅在标准定价层中可用。

* 不能将现有的 Azure VM 的大小调整为 A8、A9、A10 或 A11 大小。

* A8、A9、A10 和 A11 实例当前不能通过使用属于现有地缘组的云服务进行部署。同样，具有包含 A8、 A9、 A10 和 A11 实例的云服务的地缘组不能用于其他实例大小的部署。如果尝试这些部署，则将看到类似于 `Azure deployment failure (Compute.OverconstrainedAllocationRequest): The VM size (or combination of VM sizes) required by this deployment cannot be provisioned due to deployment request constraints.` 的错误消息

* Azure 中的 RDMA 网络保留地址空间 172.16.0.0/12。如果计划在部署于 Azure 虚拟网络中的 A8 和 A9 实例上运行 MPI 应用程序，请确保虚拟网络地址空间不与 RDMA 网络重叠。

## 后续步骤

* 有关 A8、A9、A10 和 A11 实例的可用性和定价的详细信息，请参阅[虚拟机定价](/home/features/virtual-machines/#price)和[云服务定价](/home/features/cloud-services/#price)。
* 若要使用 A8 和 A9 实例部署和配置基于 Linux 的群集以访问 Azure RDMA 网络，请参阅[设置 Linux RDMA 群集以运行 MPI 应用程序](/documentation/articles/virtual-machines-linux-cluster-rdma)。
* 若要通过 Windows 上的 HPC Pack 开始部署和使用 A8 和 A9 实例，请参阅 [A8 和 A9 计算密集型实例：使用 HPC Pack 快速启动](https://msdn.microsoft.com/zh-cn/library/azure/dn594431.aspx)和[在 A8 和 A9 实例上运行 MPI 应用程序](https://msdn.microsoft.com/zh-cn/library/azure/dn592104.aspx)。

<!---HONumber=69-->