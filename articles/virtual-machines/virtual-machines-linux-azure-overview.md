 <properties
   pageTitle="Azure 和 Linux | Azure"
   description="介绍 Linux 虚拟机上的 Azure 计算、存储和网络服务。"
   services="virtual-machines-linux"
   documentationCenter="virtual-machines-linux"
   authors="vlivech"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="virtual-machines-linux"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="vm-linux"
   ms.workload="infrastructure"
   ms.date="09/14/2016"
   wacn.date="10/24/2016"
   ms.author="v-livech"/>  


# Azure 和 Linux

Azure 正在不断集结各种集成的公有云服务，包括分析、虚拟机、数据库、移动、网络、存储和 Web，因此很适合用于托管你的解决方案。Azure 提供可缩放的计算平台，允许你即用即付，而无需投资购买本地硬件。Azure 允许你根据客户端所需的任何规模，随时扩展和缩减你的解决方案。

## 可用性
为了使部署符合 99.95 的 VM 服务级别协议，必须部署两个或更多个在可用性集中运行工作负荷的 VM。这可确保 VM 分布在我们数据中心内的多个容错域，并使用不同的维护时段部署到主机。完整 [Azure SLA](/support/sla/virtual-machines/) 说明了 Azure 作为整体的保证可用性。

## Azure 虚拟机和实例
Azure 支持运行由多家合作伙伴提供和维护的众多热门 Linux 分发版。可以在 Azure 应用商店中找到 Red Hat Enterprise、CentOS、Debian、Ubuntu、CoreOS、RancherOS 和 FreeBSD 等分发版。我们积极与各大 Linux 社区合作以便为 [Azure 认可的 Linux 分发版](/documentation/articles/virtual-machines-linux-endorsed-distros/)列表添加更多成员。

如果首选的 Linux 分发版目前不在库中，可以通过[在 Azure 中创建和上载 Linux VHD](/documentation/articles/virtual-machines-linux-create-upload-generic/) 来“自带 Linux”VM。

借助 Azure 虚拟机，用户可以采用灵活的方式部署各种计算解决方案。几乎可以在任何操作系统（Windows、Linux 或从我们不断增长的合作伙伴列表中的任一合作伙伴自定义创建的操作系统）上部署几乎任何工作负荷和任何语言。没有找到所需的映像？ 别担心，你也可以使用本地的自有映像。

## VM 大小
在 Azure 中部署 VM 时，将从一系列大小中选择一个适合工作负荷的 VM 大小。大小还会影响虚拟机的处理能力、内存和存储容量。收费的依据是 VM 的运行时长及其消耗的分配资源量。[虚拟机大小](/documentation/articles/virtual-machines-linux-sizes/)的完整列表。

下面是从我们提供的系列（A、D、DS、G 和 GS）之一中选择 VM 大小的基本指导原则。

* A 系列 VM 是高性价比的入门级 VM，适用于轻度工作负荷和开发/测试方案。所有区域都广泛提供此系列 VM，它们可用于连接和使用虚拟机可用的所有标准资源。
* D 系列的 VM 旨在运行需要更高计算能力和临时磁盘性能的应用程序。D 系列 VM 为临时磁盘提供更快的处理器、更高的内存内核比和固态驱动器 (SSD)。
* Dv2 系列是 D 系列的最新版本，具有更强大的 CPU。Dv2 系列 CPU 比 D 系列 CPU 快大约 35%。该系列基于最新一代的 2.4 GHz Intel Xeon® E5-2673 v3 (Haskell) 处理器，通过 Intel Turbo Boost Technology 2.0 可以达到 3.2 GHz。Dv2 系列的内存和磁盘配置与 D 系列相同。

注意：DS 系列 VM 可以访问高级存储 - 适用于 I/O 密集型工作负荷的以 SSD 为后盾的高性能低延迟存储。高级存储只在某些区域可用。有关详细信息，请参阅：

- [高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)

## 自动化
若要实现适当的 DevOps 区域性，所有基础结构都必须是代码。当所有基础结构都存在于代码中时，便可以轻松重新创建它（Phoenix 服务器）。Azure 可与所有主要自动化工具（如 Ansible、Chef、SaltStack 和 Puppet）配合使用。Azure 也有自己的自动化工具：

- [Azure 模板](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)

- [Azure VMAccess](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)

Azure 正在支持它的大多数 Linux 发行版中推出 [cloud-init](http://cloud-init.io/) 支持。目前，默认情况下 Canonical Ubuntu VM 在启用 cloud-init 的情况下进行部署。RedHats RHEL、CentOS 和 Fedora 支持 cloud-init，但 RedHat 维护的 Azure 映像未安装 cloud-init。若要在 RedHat 系列 OS 上使用 cloud-init，必须创建安装了 cloud-init 的自定义映像。

- [在 Azure Linux VM 上使用 cloud-init](/documentation/articles/virtual-machines-linux-using-cloud-init/)

## 配额
每个 Azure 订阅都有默认的配额限制，此限制会在为项目部署大量 VM 时造成影响。每个订阅的当前限制是每区域 20 个 VM。若要提高配额限制，可以开具支持票证来请求提高限制。有关配额限制的更多详细信息，请参阅：

- [Azure 订阅服务限制](/documentation/articles/azure-subscription-service-limits/)


## 合作伙伴

Microsoft 与合作伙伴紧密合作，以确保可用映像进行更新并针对 Azure 运行时进行优化。有关合作伙伴的详细信息，请在下面查看其应用商店页。

- [Azure 认可的分发中的 Linux](/documentation/articles/virtual-machines-linux-endorsed-distros/)

## 在 Azure 上获取安装程序
若要开始使用 Azure，需要 Azure 帐户、已安装 Azure CLI 和一对 SSH 公钥和私钥。

## 注册帐户
使用 Azure 云的第一步是注册 Azure 帐户。若要开始，请转到 [Azure 帐户注册](/pricing/1rmb-trial/)页。

## 安装 CLI
使用新的 Azure 帐户，可以立即开始使用 Azure 门户预览（一个基于 Web 的管理面板）。若要通过命令行管理 Azure 云，请安装 `azure-cli`。在 Mac 或 Linux 工作站上安装 [Azure CLI](/documentation/articles/xplat-cli-install/)。

## 创建 SSH 密钥对
现在已有 Azure 帐户、Azure Web 门户和 Azure CLI。下一步是创建 SSH 密钥对，使用它可以通过 SSH 连接到 Linux 而无需使用密码。[在 Linux 和 Mac 上创建 SSH 密钥](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)可启用无密码登录和更高的安全性。

## 在 Azure 中开始使用 Linux
设置 Azure 帐户、安装 Azure CLI 并创建 SSH 密钥后，现在便可以开始在 Azure 云中构建出基础结构。第一个任务是创建几个 VM。

## 在 CLI 上创建 VM
在 CLI 上创建 Linux VM 是一种部署 VM 的快速方法，无需离开正在使用的终端。通过命令行标志或开关提供可以在 Web 门户上指定的所有内容。

- [使用 CLI 创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)

## 在门户中创建 VM
通过在 Azure Web 门户上创建 Linux VM，可以轻松地指向和单击用于访问部署的各个选项。不是使用命令行标记或开关，而是能够查看各种选项和设置的良好 Web 布局。通过命令行界面提供的所有功能也都在门户中提供。

- [使用门户创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/)

## 不使用密码通过 SSH 登录
VM 现在正在 Azure 上运行，用户可以登录。使用密码通过 SSH 登录既不安全耗时也长。使用 SSH 密钥是最安全且最快捷的登录方式。通过门户或 CLI 创建 Linux VM 时，有两种身份验证选择。如果为 SSH 选择密码，则 Azure 将 VM 配置为允许通过密码登录。如果选择使用 SSH 公钥，则 Azure 将 VM 配置为只允许通过 SSH 密钥登录，并禁止密码登录。若要通过只允许 SSH 密钥登录来保护 Linux VM，请在门户或 CLI 中创建 VM 的过程中使用 SSH 公钥选项。

- [通过配置 SSHD 禁用 Linux VM 上的 SSH 密码](/documentation/articles/virtual-machines-linux-mac-disable-ssh-password-usage/)

## 相关 Azure 组件

## 存储

- [Azure 存储简介](/documentation/articles/storage-introduction/)

- [使用 azure-cli 将磁盘添加到 Linux VM](/documentation/articles/virtual-machines-linux-add-disk/)

- [如何在 Azure 门户预览中将数据磁盘附加到 Linux VM](/documentation/articles/virtual-machines-linux-attach-disk-portal/)

## 联网

- [虚拟网络概述](/documentation/articles/virtual-networks-overview/)

- [Azure 中的 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/)

- [在 Azure 中打开 Linux VM 的端口](/documentation/articles/virtual-machines-linux-nsg-quickstart/)

- [在 Azure 门户预览中创建完全限定的域名](/documentation/articles/virtual-machines-linux-portal-create-fqdn/)

## 后续步骤

现在已概要了解 Azure 上的 Linux。下一步是进一步的研究，并创建一些 VM 组件！

- [使用门户在 Azure 上创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/)

- [使用 CLI 在 Azure 上创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)

<!---HONumber=Mooncake_1017_2016-->