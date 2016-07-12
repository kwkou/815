 <properties
   pageTitle="Azure 和 Linux | Azure"
   description="介绍 Linux 虚拟机上的 Azure 计算、存储和网络服务。"
   services="virtual-machines-linux"
   documentationCenter="virtual-machines-linux"
   authors="rickstercdn"
   manager="timlt"
   editor=""/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="02/01/2016"
	wacn.date="05/24/2016"/>

# Azure 和 Linux
Azure 正在不断集结各种集成的公有云服务，包括分析、虚拟机、数据库、移动、网络、存储和 Web，因此很适合用于托管你的解决方案。Azure 提供可缩放的计算平台，允许你即用即付，而无需投资购买本地硬件。Azure 允许你根据客户端所需的任何规模，随时扩展和缩减你的解决方案。
 
## Azure 虚拟机
借助 Azure 虚拟机，你可以采用灵活的方式部署各种计算解决方案。若要部署 Windows 或 Linux 虚拟机，你几乎可以在任何操作系统上（Windows, Linux）以几乎任何语言部署几乎任何工作负荷，或通过我们不断增加的合作伙伴列表内的任何成员所创建的自定义映像来进行。没有找到所需的映像？ 别担心，你也可以使用本地的自有映像。
 
## 在 Azure 中开始使用 Linux

同时使用 Azure 虚拟机、存储和网络可提供 Internet 规模的基础结构即服务以满足 Linux 计算需求。若要在 Azure 上开始使用 Linux，你需要：

1. 一个试用帐户。[获取帐户](/pricing/1rmb-trial/)。
2. 适用于 Linux、Mac 和 Windows 的 Azure 命令行界面 (Azure CLI)。[安装 CLI](/documentation/articles/xplat-cli-install/)。
3. 一个 Linux VM。[创建 VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)。
4. 有关 Linux 和 Azure 的详细信息，包括如何满足服务级别协议 (SLA)。**请阅读此文档，即使你不喜欢阅读法律文档。**

## 后勤：分发版、可用性、VM 大小和配额

### 分发版
Azure 支持运行由多家合作伙伴提供和维护的众多热门 Linux 分发版。可以在 Azure 库中找到 CentOS、Debian、Ubuntu 和 SUSE 等分发版。我们积极与各大 Linux 社区合作以便为认可的分发版列表添加更多成员。**[查看当前分发版](/documentation/articles/virtual-machines-linux-endorsed-distros/)**如果你首选的 Linux 分发版目前不在库中，可以遵循**[此页](/documentation/articles/virtual-machines-linux-create-upload-generic/)**上的指导来“自带 Linux” VM。

## 可用性和 Azure SLA
为了使部署符合 99.95 的 VM 服务级别协议，必须部署两个或更多个在可用性集中运行工作负荷的 VM。这可确保 VM 分布在我们数据中心内的多个容错域，并使用不同的维护时段部署到主机。

## VM 大小和配额
当你在 Azure 中部署 VM 时，可从一系列大小中选择一个适合工作负荷的 VM 大小。大小还会影响虚拟机的处理能力、内存和存储容量。收费的依据是 VM 的运行时长及其消耗的分配资源量。有关更完整列表，请参阅以下有关[虚拟机大小](/documentation/articles/virtual-machines-linux-sizes/)的文章。

下面是从我们提供的系列（A、D 和 Dv2）之一中选择 VM 大小的基本指导原则。

* A 系列 VM 是高性价比的入门级 VM，适用于轻度工作负荷和开发/测试方案。所有区域都广泛提供此系列 VM，它们可用于连接和使用虚拟机可用的所有标准资源。
* D 系列的 VM 旨在运行需要更高计算能力和临时磁盘性能的应用程序。D 系列 VM 为临时磁盘提供更快的处理器、更高的内存内核比和固态驱动器 (SSD)。 
* Dv2 系列是 D 系列的最新版本，具有更强大的 CPU。Dv2 系列 CPU 比 D 系列 CPU 快大约 35%。该系列基于最新一代的 2.4 GHz Intel Xeon® E5-2673 v3 (Haswell) 处理器，通过 Intel Turbo Boost Technology 2.0 可以达到 3.2 GHz。Dv2 系列的内存和磁盘配置与 D 系列相同。

注意：DS 系列 VM 可以访问高级存储 - 适用于 I/O 密集型工作负荷的以 SSD 为后盾的高性能低延迟存储。高级存储只在某些区域可用。有关详细信息，请参阅 **[Premium Storage: High-performance storage for Azure virtual machine workloads（高级存储：适用于 Azure 虚拟机工作负荷的高性能存储）](/documentation/articles/storage-premium-storage/)**。

每个 Azure 订阅都有默认的配额限制，此限制会在要部署大量 VM 供项目使用时造成影响。每个订阅的当前限制是每区域 20 个 VM。若要提高此配额限制，可以开具支持票证来请求提高限制。有关配额限制的详细信息，请参阅 **[Azure Subscription Service Limits（Azure 订阅服务限制）](/documentation/articles/azure-subscription-service-limits/)**

## 后续步骤

一个试用帐户。**[获取帐户。](/pricing/1rmb-trial/)** 如果你已有一个帐户并想要试用，请**[安装 Azure CLI](/documentation/articles/xplat-cli-install/)**。试用完毕后，请[创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)。

<!---HONumber=Mooncake_0503_2016-->