<properties
   pageTitle="Azure 资源管理器体系结构"
   description="了解资源管理器的体系结构以及计算、网络和存储资源提供程序之间的关系。"
   services="virtual-machines"
   documentationCenter=""
   authors="davidmu1"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"/>

<tags 
	ms.service="virtual-machines"
	ms.date="07/07/2015"
	wacn.date="08/29/2015"/>

# Azure 资源管理器体系结构

本文概述了用于创建基于基础结构的应用程序和工作负荷的服务管理和资源管理器体系结构。

## 服务管理的体系结构

在我们讨论 Azure 资源管理器的体系结构和各种资源提供程序之前，先来查看一下 Azure 服务管理当前存在的体系结构。在 Azure 服务管理中，计算、存储或用于承载虚拟机的网络资源的提供方是：

- 一种必需的云服务，充当承载虚拟机（计算）的容器。虚拟机自动随附一个网络接口卡 (NIC)，并由 Azure 分配一个 IP 地址。此外，云服务包含一个外部负载平衡器实例、一个公共 IP 地址和多个默认终结点，以允许基于 Windows 的虚拟机使用远程桌面和远程 PowerShell 流量，以及基于 Linux 的虚拟机使用安全外壳 (SSH) 流量。
- 一个存储虚拟机 VHD 所需的存储帐户，它包括操作系统、临时及附加数据磁盘（存储）。
- 一个可选的虚拟网络，它充当附加容器，可以在其中创建子网的结构并指定虚拟机所在的子网（网络）。

以下是 Azure 服务管理的组件及其相互关系。

![](./media/virtual-machines-azure-resource-manager-architecture/arm_arch1.png)

## 资源管理器的体系结构

资源提供程序支持 Azure 资源管理器使用单个资源，这些资源用于在所需配置中创建正常运行的虚拟机。对于虚拟机，有三个主要的资源提供程序：

- 计算资源提供程序 (CRP)：支持虚拟机和可选可用性集的实例。
- 存储资源提供程序 (SRP)：支持存储虚拟机 VHD 所需的存储帐户，包括其操作系统和附加数据磁盘。
- 网络资源提供程序 (NRP)：支持必需的 NIC、虚拟机 IP 地址、虚拟网络和可选负载平衡器内的子网、负载平衡器 IP 地址和网络安全组。

此外，资源提供程序内各资源间也彼此相关：

- 虚拟机依赖 SRP 中定义的特定存储帐户，以将其磁盘存入 Blob 存储中（必需）。
- 虚拟机引用 NRP 中定义的特定 NIC（必需）以及 CRP 中定义的可用性集（可选）。
- NIC 引用分配给虚拟机的 IP 地址（必需）、虚拟机虚拟网络的子网（必需）和一个网络安全组（可选）。
- 虚拟网络内的子网引用了网络安全组（可选）。
- 负载平衡器实例引用含有虚拟机 NIC（可选）的 IP 地址的后端池，并引用一个负载平衡器公用或专用 IP 地址（可选）。

![](./media/virtual-machines-azure-resource-manager-architecture/arm_arch2.png)

通过将资源组件化，用户在配置 Azure 中托管的 IT 工作服在的基础结构时拥有更大的灵活性。Azure 资源管理器模板利用这种灵活性来创建特定配置所需的从属资源集。执行模板时，资源管理器会确保配置资源的创建顺序正确无误，以保留依赖项和引用。例如，资源管理器会先创建具有子网和 IP 地址（网络安全组为可选）的虚拟网络，然后才会创建虚拟机的 NIC。

资源组是一个逻辑容器，它包含应用程序的相关资源，这可以由多个虚拟机、NIC、IP 地址、负载平衡器、子网和网络安全组构成。例如，你可以将应用程序的所有资源视为单个管理单元加以管理。你可以一并创建、更新和删除所有资源。下面是在单一资源组中部署的示例应用程序。

![](./media/virtual-machines-azure-resource-manager-architecture/arm_arch3.png)

此应用程序由以下部分组成：

- 两个虚拟机，它们使用同一存储帐户，属于同一可用性集，且位于虚拟网络的同一子网上。
- 每个虚拟机的一个 NIC 和 VM IP 地址。
- 一个外部负载平衡器，它将 Internet 流量分配到两个虚拟机的 NIC。

此应用程序的所有这些资源均由其所在的单个资源组进行管理。

使用 Azure PowerShell 或 Azure CLI 创建基于资源管理器的虚拟机时，还可以查看资源之间的组件化和依赖关系。必须先创建创建资源组、存储帐户、带子网的虚拟网络和具有 IP 地址的 NIC，才可以运行创建虚拟机的命令。有关详细信息，请参阅[使用资源管理器和 Azure PowerShell 创建并预配置 Windows 虚拟机](/documentation/articles/virtual-machines-ps-create-preconfigure-windows-resource-manager-vms)。

## 后续步骤

[使用 Azure 资源管理器模板和 Azure CLI 部署和管理虚拟机](/documentation/articles/virtual-machines-deploy-rmtemplates-azure-cli)

[使用资源管理器模板与 PowerShell 来部署和管理 Azure 虚拟机](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell)

## 其他资源

[Azure 资源管理器下的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-azurerm-versus-azuresm)

<!--[-->Azure 资源管理器概述<!--](/documentation/articles/resource-group-overview)-->

<!---HONumber=67-->