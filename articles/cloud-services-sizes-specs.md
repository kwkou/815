<properties
 pageTitle="云服务的大小"
 description="列出 Azure 云服务 Web 角色和辅助角色的不同大小。"
 services="cloud-services"
 documentationCenter=""
 authors="Thraka"
 manager="timlt"
 editor=""/>
<tags
 ms.service="cloud-services"
 ms.date="03/25/2016"
 wacn.date="05/12/2016"/>

# 云服务的大小

本主题介绍云服务角色实例（Web 角色和辅助角色）的可用大小与选项。此外，还提供了在计划使用这些资源时要考虑的部署注意事项。

云服务是 Azure 提供的多种类型的计算资源之一。有关云服务的详细信息，请单击[此处](/documentation/articles/cloud-services-choose-me/)。

> [AZURE.NOTE]若要查看相关的 Azure 限制，请参阅 [Azure 订阅和服务限制、配额与约束](/documentation/articles/azure-subscription-service-limits/)

## Web 角色实例和辅助角色实例的大小

以下注意事项可能会帮助你决定大小：

* D 系列的 VM 实例旨在运行需要更高计算能力和临时磁盘性能的应用程序。D 系列 VM 为临时磁盘提供更快的处理器、更高的内存内核比和固态驱动器 (SSD)。有关详细信息，请参阅 Azure 博客[新的 D 系列虚拟机大小](https://azure.microsoft.com/blog/2014/09/22/new-d-series-virtual-machine-sizes)上的公告。  

* Dv2 系列是原 D 系列的后续系列，其特点是 CPU 功能更强大。Dv2 系列 CPU 比 D 系列 CPU 快大约 35%。该系列基于最新一代的 2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) 处理器，通过 Intel Turbo Boost Technology 2.0 可以达到 3.2 GHz。Dv2 系列的内存和磁盘配置与 D 系列相同。

* 由于系统要求，Web 角色和辅助角色需要比 Azure 虚拟机更多的临时磁盘空间。系统文件为 Windows 页面文件保留 4 GB 的空间，并为 Windows 转储文件保留 2 GB 的空间。

* 操作系统磁盘包含 Windows 来宾操作系统并包含 Program Files 文件夹（除非你指定另一个磁盘，否则包括通过启动任务完成的安装）、注册表更改、System32 文件夹和 .NET Framework。

* **临时存储磁盘**包含 Azure 日志和配置文件、Azure 诊断（包括 IIS 日志）和你定义的任何本地存储资源。

* **应用程序磁盘**是提取你的 .cspkg 的位置，它包含你的网站、二进制文件、角色主机进程、启动任务、web.config，等等。

* A8/A10 和 A9/A11 虚拟机大小具有相同的容量。A8 和 A9 虚拟机实例包括一个连接到远程直接内存访问 (RDMA) 网络的附加网络适配器，从而实现两台虚拟机之间的快速通信。A8 和 A9 实例专门用于在执行期间需要在两个节点之间保持恒定和低延迟通信的高性能计算应用程序，例如使用消息传递接口 (MPI) 的应用程序。A10 和 A11 虚拟机实例不包含其他网络适配器。A10 和 A11 实例专门用于不需要在两个节点之间保持恒定和低延迟通信的高性能计算应用程序，也称为参数化或高度并行应用程序。

    >[AZURE.NOTE] 如果你正在考虑 A8 到 A11 大小，请阅读[此](/documentation/articles/virtual-machines-windows-a8-a9-a10-a11-specs/)信息。

>[AZURE.NOTE] 所有计算机大小都提供一个**应用程序磁盘**用于存储云服务包中的所有文件；其大小约为 1.5 GB。

请务必查看每个云服务大小的[定价](/home/features/cloud-services/pricing/)。

## 常规用途

用于网站、中小型数据库，以及其他日常应用程序。

>[AZURE.NOTE] 存储容器使用 1024^3 字节数（GB 的计量单位）表示。这有时也称为 GiB，即二进制定义。在比较使用不同进制系统的大小时，请记住二进制大小可能看上去小于十进制大小，但对于任何具体大小（例如 1 GB），二进制系统提供的容量多于十进制系统，因为 1024^3 大于 1000^3。

| 大小 (id) | 核心数 | RAM | 总磁盘大小 |
| --------------- | :-------: | ------: | ------: |
| 特小型 | 1 | 0\.75 GB | 19 GB |
| 小型 | 1 | 1\.75 GB | 224 GB |
| 中型 | 2 | 3\.5 GB | 489 GB |
| 大型 | 4 | 7 GB | 999 GB |
| 超大型 | 8 | 14 GB | 2,039 GB |

>[AZURE.NOTE] **特小**到**特大**型也可分别称为 **A0-A4**。

## 内存密集型

用于大型数据库、SharePoint 服务器场，以及高吞吐量应用程序。

| 大小 (id) | 核心数 | RAM | 总磁盘大小 |
| --------------- | :-------: | ------: | ------: |
| A5 | 2 | 14 GB | 489 GB |
| A6 | 4 | 28 GB | 999 GB |
| A7 | 8 | 56 GB | 2,039 GB |

## 网络优化并提供 InfiniBand 支持

在选定的数据中心可用。A8 和 A9 虚拟机配有 [Intel® Xeon® E5 处理器](http://www.intel.com/content/www/us/en/processors/xeon/xeon-processor-e5-family.html)。增加一个采用远程直接内存访问 (RDMA) 技术的 32 Gbit/s 的 **InfiniBand** 网络。适用于消息传递接口 (MPI) 应用程序、高性能群集、建模和模拟、视频编码，以及其他计算密集型或网络密集型方案。

| 大小 (id) | 核心数 | RAM | 总磁盘大小 |
| --------------- | :-------: | ------: | ------: |
| A8 | 8 | 56 GB | 382 GB |
| A9 | 16 | 112 GB | 382 GB |

## 计算密集型

在选定的数据中心可用。A10 和 A11 虚拟机配有 [Intel® Xeon® E5 处理器](http://www.intel.com/content/www/us/en/processors/xeon/xeon-processor-e5-family.html)。适用于高性能群集、建模和模拟、视频编码以及其他计算密集型或网络密集型方案。与 A8 和 A9 实例配置类似，无需 InfiniBand 网络和 RDMA 技术。

| 大小 (id) | 核心数 | RAM | 总磁盘大小 |
| --------------- | :-------: | ------: | ------: |
| A10 | 8 | 56 GB | 382 GB |
| A11 | 16 | 112 GB | 382 GB |

## D 系列：优化计算

D 系列虚拟机的特色是固态硬盘 (SSD) 和比 A 系列更快的处理器（快 60%），而且也适用于 Azure 云服务中的 Web 角色或辅助角色。此系列适用于需要更快的 CPU、更好的本地磁盘性能或更大的内存的应用程序。

## 常规用途 (D)

用于网站、中小型数据库，以及其他日常应用程序。

| 大小 (id) | 核心数 | RAM | 总磁盘大小 |
| --------------- | :-------: | ------: | ------: |
| Standard\_D1 | 1 | 3\.5 GB | 50 GB |
| Standard\_D2 | 2 | 7 GB | 100 GB |
| Standard\_D3 | 4 | 14 GB | 200 GB |
| Standard\_D4 | 8 | 28 GB | 400 GB |

## 内存密集型 (D)

用于大型数据库、SharePoint 服务器场，以及高吞吐量应用程序。

| 大小 (id) | 核心数 | RAM | 总磁盘大小 |
| --------------- | :-------: | ------: | ------: |
| Standard\_D11 | 2 | 14 GB | 100 GB |
| Standard\_D12 | 4 | 28 GB | 200 GB |
| Standard\_D13 | 8 | 56 GB | 400 GB |
| Standard\_D14 | 16 | 112 GB | 800 GB |

## Dv2 系列：优化计算

Dv2 系列实例是下一代的 D 系列实例，可以作为虚拟机或云服务使用。Dv2 系列实例将附带更强大的 CPU，比 D 系列实例平均快 35% 左右，并附带与 D 系列相同的内存和磁盘配置。Dv2 系列实例基于最新一代的 2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) 处理器，通过 Intel Turbo Boost Technology 2.0 可以达到 3.2 GHz。Dv2 系列和 D 系列非常适合需求更快的 CPU、更好的本地磁盘的性能或者更高内存的应用程序，并为许多企业级应用程序提供强大的组合。

## 常规用途 (Dv2)

用于网站、中小型数据库，以及其他日常应用程序。

| 大小 (id) | 核心数 | RAM | 总磁盘大小 |
| --------------- | :-------: | ------: | ------: |
| Standard\_D1\_v2 | 1 | 3\.5 GB | 50 GB |
| Standard\_D2\_v2 | 2 | 7 GB | 100 GB |
| Standard\_D3\_v2 | 4 | 14 GB | 200 GB |
| Standard\_D4\_v2 | 8 | 28 GB | 400 GB |
| Standard\_D5\_v2 | 16 | 56 GB | 800 GB |

## 内存密集型 (Dv2)

用于大型数据库、SharePoint 服务器场，以及高吞吐量应用程序

| 大小 (id) | 核心数 | RAM | 总磁盘大小 |
| --------------- | :-------: | ------: | ------: |
| Standard\_D11\_v2 | 2 | 14 GB | 100 GB |
| Standard\_D12\_v2 | 4 | 28 GB | 200 GB |
| Standard\_D13\_v2 | 8 | 56 GB | 400 GB |
| Standard\_D14\_v2 | 16 | 112 GB | 800 GB |

## 配置云服务的大小

你可以指定角色实例的虚拟机大小作为[服务定义文件](/documentation/articles/cloud-services-model-and-package/#csdef)描述的服务模型的一部分。角色大小确定了 CPU 核心数目、内存容量，以及分配给正在运行的实例的本地文件系统大小。根据应用程序的资源要求选择角色大小。

下面是一个将 Web 角色实例的角色大小设置为 [Standard\_D2](#general-purpose-d) 的示例：


	<WebRole name="WebRole1" vmsize="<mark>Standard_D2</mark>">

	</WebRole>


<!---HONumber=Mooncake_0503_2016-->
