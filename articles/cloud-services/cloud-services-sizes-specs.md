<properties
 pageTitle="云服务的大小"
 description="列出 Azure 云服务 Web 角色和辅助角色的不同虚拟机大小。"
 services="cloud-services"
 documentationCenter=""
 authors="Thraka"
 manager="timlt"
 editor=""/>
<tags
 ms.service="cloud-services"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="tbd"
 ms.date="01/12/2017"
 ms.author="adegeo"
 wacn.date="01/25/2017"/>

# 云服务的大小

本主题介绍云服务角色实例（Web 角色和辅助角色）的可用大小和选项。此外，还提供了在计划使用这些资源时要考虑的部署注意事项。每个大小都有一个在[服务定义文件](/documentation/articles/cloud-services-model-and-package/#csdef)中定义的 ID。[云服务定价](/pricing/details/cloud-services/)页提供每种大小的价格。


> [AZURE.NOTE]若要查看相关的 Azure 限制，请参阅 [Azure 订阅和服务限制、配额与约束](/documentation/articles/azure-subscription-service-limits/)

## Web 角色实例和辅助角色实例的大小
Azure 上有多个标准大小可供选择。其中某些大小的注意事项包括：

* D 系列的 VM 旨在运行需要更高计算能力和临时磁盘性能的应用程序。D 系列 VM 可为临时磁盘提供更快的处理器、更高的内存内核比和固态驱动器 (SSD)。
* Dv2 系列是原 D 系列的后续系列，其特点是 CPU 功能更强大。Dv2 系列 CPU 比 D 系列 CPU 快大约 35%。该系列基于最新一代的 2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) 处理器，通过 Intel Turbo Boost Technology 2.0 可以达到 3.1 GHz。Dv2 系列的内存和磁盘配置与 D 系列相同。
* G 系列的 VM 可提供最大的内存，并在配备 Intel Xeon E5 V3 系列处理器的主机上运行。
* A 系列 VM 可以部署在各种不同的硬件类型和处理器上。根据硬件限制大小，为运行中的实例提供一致的处理器性能，而不考虑硬件部署的位置。若要判断此大小部署所在的物理硬件，请从虚拟机中查询虚拟硬件。
* A0 大小在物理硬件上过度订阅。仅针对此特定大小，其他客户部署可能影响正在运行的工作负荷的性能。以下概述的相对性能为预期的基准，受限于近似变化性的 15%。

虚拟机的大小会影响定价。此外，大小还会影响虚拟机的处理能力、内存和存储容量。存储成本根据存储帐户中的已使用页数进行单独计算。有关详细信息，请参阅[云服务定价详细信息](/pricing/details/cloud-services/)和 [Azure 存储定价](/pricing/details/storage/)。

以下注意事项可能有助于确定大小：

* A8-A11 和 H 系列大小也称为*计算密集型实例*。运行这些大小的硬件专为计算密集型和网络密集型应用程序而设计和优化，包括高性能计算 (HPC) 群集应用程序、建模和模拟。A8-A11 系列使用 Intel Xeon E5-2670 @ 2.6 GHZ，H 系列使用 Intel Xeon E5-2667 v3 @ 3.2 GHz。
* Dv2 系列、D 系列和 G 系列是要求有更快速的 CPU、更好的本地磁盘性能，或更高内存的应用程序的理想选择。它们为许多企业级应用程序提供强大的组合。
* Azure 数据中心内的一些物理主机可能不支持更大的虚拟机大小，例如 A5 – A11。因此，在以下情况中，可能会显示错误消息“未能配置虚拟机 {machine name}”或“未能创建虚拟机 {machine name}”：将现有虚拟机的大小调整为新的大小时；在 2013 年 4 月 16 日之前创建的虚拟网络中创建新的虚拟机时；或者向现有的云服务中添加新的虚拟机时。
* 订阅也可能会限制某些大小系列中可部署的核心数。若要增加配额，请联系 Azure 支持。

## 性能注意事项
我们已创建 Azure 计算单位 (ACU) 概念，提供一种方法来比较各 Azure SKU 的计算 (CPU) 性能并识别哪个 SKU 最能满足用户的性能需求。ACU 目前在小型 (Standard\_A1) VM 上标准为 100，而所有其他 SKU 表示 SKU 在运行标准基准测试时大约可以有多快。

> [AZURE.IMPORTANT]
ACU 只是一种规则。工作负荷的结果可能会有所不同。
> 
> 

<br>  


| SKU 系列 | ACU/核心 |
| --- | --- |
| [特小型](#a-series) |50 |
| [Small-ExtraLarge](#a-series) |100 |
| [A5-7](#a-series) |100 |
| [Standard\_A1-8v2](#av2-series) |100 |
| [Standard\_A2m-8mv2](#av2-series) |100 |
| [A8-A11](#a-series) |225* |
| [D1-14](#d-series) |160 |
| [D1-15v2](#dv2-series) |210 - 250* |
| [G1-5](#g-series) |180 - 240* |
| [H](#h-series) |290 - 300* |

ACU 标有 *使用 Intel® Turbo 技术来增加 CPU 频率，并提升性能。提升量可能因 VM 大小、工作负荷和同一主机上运行的其他工作负荷而有所不同。

## 大小表
下表显示这些虚拟机提供的大小和容量。

* 存储容量的单位为 GiB 或 1024^3 字节。比较以 GB（1000^3 字节）为单位的磁盘和以 GiB（1024^3 字节）为单位的磁盘时，请记住以 GiB 为单位的容量数显得更小。例如，1023 GiB = 1098.4 GB
* 磁盘吞吐量的单位为每秒输入/输出操作数 (IOPS) 和 MBps，其中 MBps = 10^6 字节/秒。
* 数据磁盘可以在缓存或非缓存模式下运行。对于缓存数据磁盘操作，主机缓存模式设置为**只读**或**读写**。对于非缓存数据磁盘操作，主机缓存模式设置为**无**。
* 最大网络带宽是每个 VM 类型分配并分配的最大聚合带宽。最大带宽提供选择正确 VM 类型的指导原则，以确保有适当的网络容量可用。在低、中、高和极高之间切换时，吞吐量将随之增加。实际的网络性能将取决于许多因素，包括网络和应用程序负载，以及应用程序的网络设置。

## <a name="a-series"></a> A 系列
| 大小 | CPU 核心数 | 内存：GiB | 本地 HDD：GiB | 最大网卡数/网络带宽等级 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| 特小型 | 1 | 0\.768 | 20 | 1/低 |
| 小型 | 1 | 1\.75 | 225 | 1/中 |
| 中型 | 2 | 3\.5 GB | 490 | 1/中 |
| 大型 | 4 | 7 | 1000 | 2/高 |
| 超大型 | 8 | 14 | 2040 | 4/高 |
| A5 | 2 | 14 | 490 | 1/中 |
| A6 | 4 | 28 | 1000 | 2/高 |
| A7 | 8 | 56 | 2040 | 4/高 |

## A 系列 - 计算密集型实例

| 大小 | CPU 核心数 | 内存：GiB | 本地 HDD：GiB | 最大网卡数/网络带宽等级 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| A8* |8 | 56 | 1817 | 2/高 |
| A9* |16 | 112 | 1817 | 4/非常高 |
| A10 |8 | 56 | 1817 | 2/高 |
| A11 |16 | 112 | 1817 | 4/非常高 |

*具有 RDMA 功能

## <a name="av2-series"></a> Av2 系列

| 大小 | CPU 核心数 | 内存：GiB | 本地 SSD：GiB | 最大网卡数/网络带宽等级 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Standard\_A1\_v2 | 1 | 2 | 10 | 1/中 |
| Standard\_A2\_v2 | 2 | 4 | 20 | 2/中 |
| Standard\_A4\_v2 | 4 | 8 | 40 | 4/高 |
| Standard\_A8\_v2 | 8 | 16 | 80 | 8/高 |
| Standard\_A2m\_v2 | 2 | 16 | 20 | 2/中 |
| Standard\_A4m\_v2 | 4 | 32 | 40 | 4/高 |
| Standard\_A8m\_v2 | 8 | 64 | 80 | 8/高 |


## <a name="d-series"></a> D 系列
| 大小 | CPU 核心数 | 内存：GiB | 本地 SSD：GiB | 最大网卡数/网络带宽等级 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Standard\_D1 | 1 | 3\.5 | 50 | 1/中 |
| Standard\_D2 | 2 | 7 | 100 | 2/高 |
| Standard\_D3 | 4 | 14 | 200 | 4/高 |
| Standard\_D4 | 8 | 28 | 400 | 8/高 |
| Standard\_D11 | 2 | 14 | 100 | 2/高 |
| Standard\_D12 | 4 | 28 | 200 | 4/高 |
| Standard\_D13 | 8 | 56 | 400 | 8/高 |
| Standard\_D14 | 16 | 112 | 800 | 8/非常高 |

## <a name="dv2-series"></a> Dv2 系列
| 大小 | CPU 核心数 | 内存：GiB | 本地 SSD：GiB | 最大网卡数/网络带宽等级 |
|---------------- | --------- | ------------ | -------------------- | ---------------------------- |
| Standard\_D1\_v2 | 1 | 3\.5 | 50 | 1/中 |
| Standard\_D2\_v2 | 2 | 7 | 100 | 2/高 |
| Standard\_D3\_v2 | 4 | 14 | 200 | 4/高 |
| Standard\_D4\_v2 | 8 | 28 | 400 | 8/高 |
| Standard\_D5\_v2 | 16 | 56 | 800 | 8/极高 |
| Standard\_D11\_v2 | 2 | 14 | 100 | 2/高 |
| Standard\_D12\_v2 | 4 | 28 | 200 | 4/高 |
| Standard\_D13\_v2 | 8 | 56 | 400 | 8/高 |
| Standard\_D14\_v2 | 16 | 112 | 800 | 8/极高 |
| Standard\_D15\_v2 | 20 | 140 | 1,000 | 8/极高 |

## 配置云服务的大小

你可以指定角色实例的虚拟机大小作为[服务定义文件](/documentation/articles/cloud-services-model-and-package/#csdef)描述的服务模型的一部分。角色大小确定了 CPU 核心数目、内存容量，以及分配给正在运行的实例的本地文件系统大小。根据应用程序的资源要求选择角色大小。

下面是一个将 Web 角色实例的角色大小设置为 Standard_D2 的示例：


	<WorkerRole name="Worker1" vmsize="Standard_D2">
	...
	</WorkerRole>

## 更改现有角色的大小

工作负荷性质更改或有新的 VM 大小可用时，可能需要更改角色的大小。为此，必须在服务定义文件中更改 VM 大小（如上所示），然后重新打包云服务并进行部署。不能直接从门户或 PowerShell 更改 VM 大小。

>[AZURE.TIP]
用户可能希望在不同的环境（例如测试和生产）中为角色使用不同的 VM 大小。一种办法是在项目中创建多个服务定义文件 (.csdef)，然后使用 CSPack 工具在自动生成期间为每个环境创建不同的云服务包。若要了解云服务包的元素以及如何创建云服务包，请参阅[什么是云服务模型以及如何打包？](/documentation/articles/cloud-services-model-and-package/)
>
>

## 获取大小列表
可以使用 PowerShell 或 REST API 获取大小列表。[此处](https://msdn.microsoft.com/zh-cn/library/azure/dn469422.aspx)介绍了 REST API。以下代码是一个 PowerShell 命令，用于列出当前可用于云服务的所有大小。

	Get-AzureRoleSize | where SupportedByWebWorkerRoles -eq $true | select InstanceSize

## 后续步骤
* 了解 [Azure 订阅和服务的限制、配额和约束](/documentation/articles/azure-subscription-service-limits/)。

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description:update wording-->