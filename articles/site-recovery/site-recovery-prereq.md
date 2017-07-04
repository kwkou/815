<properties
    pageTitle="使用 Azure Site Recovery 复制到 Azure 的先决条件 | Azure"
    description="本文总结了使用 Azure Site Recovery 服务将 VM 和物理计算机复制到 Azure 的先决条件。"
    services="site-recovery"
    documentationcenter=""
    author="rajani-janaki-ram"
    manager="jwhit"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="e24eea6c-50a7-4cd5-aab4-2c5c4d72ee2d"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="03/27/2017"
    wacn.date="05/02/2017"
    ms.author="rajanaki"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="cf8beb6e09b06edf48aeaa6f28c766a819f5373f"
    ms.lasthandoff="04/22/2017" />

#  <a name="prerequisites-for-replication-to-azure-by-using-azure-site-recovery"></a>使用 Azure Site Recovery 复制到 Azure 的先决条件


Azure Site Recovery 服务可协调本地物理服务器和虚拟机到云 (Azure) 或辅助数据中心的复制，进而促进业务连续性和灾难恢复 (BCDR) 策略。 当主要位置发生故障时，可以故障转移到辅助位置，使应用和工作负荷保持可用。 当主要位置恢复正常时，可以故障回复到主要位置。 有关 Site Recovery 的详细信息，请参阅[什么是 Site Recovery？](/documentation/articles/site-recovery-overview/)。

本文汇总了开始使用 Site Recovery 复制到 Azure 时所要满足的先决条件。

## <a name="azure-requirements"></a>Azure 要求

**要求** | **详细信息**
--- | ---
**Azure 帐户** | <p>一个 [Azure](https://www.azure.cn/) 帐户。</p><p> 可以从 [1 元试用](/pricing/1rmb-trial/)开始。</p>
**Site Recovery 服务** | 有关 Site Recovery 定价的详细信息，请参阅[站点恢复定价](/pricing/details/site-recovery/)。 |
**Azure 存储** | <p>需要一个 Azure 存储帐户来存储复制数据，且该帐户必须与恢复服务保管库位于同一区域。复制的数据存储在 Azure 存储中，发生故障转移时将创建 Azure VM</p><p> 根据要用于故障转移 Azure VM 的资源模型，可在 [Azure Resource Manager 模型](/documentation/articles/storage-create-storage-account/)或[经典模型](/documentation/articles/storage-create-storage-account-classic-portal/)下设置帐户。</p><p>可使用 [异地冗余存储](/documentation/articles/storage-redundancy/#geo-redundant-storage) 或本地冗余存储。建议使用异地冗余存储，以便在发生区域性故障或无法恢复主要区域时可复原数据。</p><p> 可以使用标准或高级存储。[高级存储](/documentation/articles/storage-premium-storage/)通常用于 IO 性能一贯较高且延迟一贯较低、托管 IO 密集型工作负荷的虚拟机。如果使用高级存储来存储复制数据，则还需要一个标准存储帐户来存储复制日志，以便捕获本地数据正在发生的更改。</p>
**存储限制** | 无法跨资源组、在订阅中或跨订阅移动 Site Recovery 中所用的存储帐户。
**Azure 网络** | <p>需要一个 Azure VM 故障转移后可连接的 Azure 网络，且该网络必须与恢复服务保管库位于同一区域。</p><p> 在 Azure 门户中，可使用 [Resource Manager 模型](/documentation/articles/virtual-networks-create-vnet-arm-pportal/)或[经典模型](/documentation/articles/virtual-networks-create-vnet-classic-pportal/)创建网络。</p><p> 如果从 System Center Virtual Machine Manager 复制到 Azure，则需要在 Virtual Machine Manager VM 网络与 Azure 网络之间设置网络映射，确保 Azure VM 在故障转移后连接到适当的网络。</p>
**网络限制** | 无法跨资源组、在订阅中或跨订阅移动 Site Recovery 中所用的网络帐户。
**网络映射** | 如果复制 Virtual Machine Manager 云中的 Hyper-V VM，则需要设置网络映射，以便在故障转移后创建 Azure VM 时，这些 VM 可以连接到相应的网络。

>[AZURE.NOTE]
>以下部分介绍客户环境中各组件的先决条件。 若要深入了解对特定配置的支持，请阅读[支持矩阵](/documentation/articles/site-recovery-support-matrix/)。
>

## <a name="disaster-recovery-of-physical-windows-or-linux-servers-to-azure"></a>通过灾难恢复将物理 Windows/Linux 服务器恢复到 Azure
下面是对物理 Windows/Linux 服务器进行灾难恢复所需的组件，[Azure 要求](#Azure requirements)中提到的组件除外。


### <a name="configuration-server-or-additional-process-server-you-will-need-to-set-up-an-on-premises-machine-as-the-configuration-server-to-coordinate-communications-between-the-on-premises-site-and-azure-and-to-manage-data-replication-brbr"></a>**配置服务器或附加的进程服务器**：需将一台本地计算机设置为配置服务器，用于协调本地站点与 Azure 之间的通信，以及管理数据复制。 <br></br>


1. **复制的计算机先决条件**


| **组件** | **要求** |
| --- | --- |
| **Windows 计算机**（物理） | <p>计算机应运行支持的 64 位操作系统：Windows Server 2012 R2、Windows Server 2012 或 Windows Server 2008 R2 SP1 及更高版本。</p><p> 操作系统应安装在驱动器 C 上。OS 磁盘应为 Windows 基本磁盘而不是动态磁盘。 数据磁盘可以是动态磁盘。</p>|
| **Linux 计算机**（物理） | <p>需要支持的 64 位操作系统：Red Hat Enterprise Linux 6.7、6.8、7.1 或 7.2；Centos 6.5、6.6、6.7、6.8、7.0、7.1 或 7.2；运行 Red Hat 兼容内核或 Unbreakable Enterprise Kern Release 3 (UEK3) 的 Oracle Enterprise Linux 6.4 或 6.5；SUSE Linux Enterprise Server 11 SP3；SUSE Linux Enterprise Server 11 SP4。</p><p>在受保护计算机上的 /etc/hosts 文件中，部分所含条目应将本地主机名映射到与所有网络适配器相关联的 IP 地址。</p><p>如果要在故障转移后使用安全外壳客户端 (ssh) 连接运行 Linux 的 Azure 虚拟机，请确保将受保护的计算机上的安全外壳服务设置为在系统启动时自动启动，并且防火墙规则允许与其建立 ssh 连接。</p><p>主机名、装载点、设备名称以及 Linux 系统路径和文件名（例如 /etc/ 和 /usr）只能采用英文形式。</p><p>以下目录（如果已设置为独立的分区/文件系统）必须都位于源服务器的同一磁盘（OS 磁盘）上：/（根）、/boot、/usr、/usr/local、/var 等<br/><br/>XFS 文件系统上的 ASR 目前不支持 XFS v5 功能（例如元数据校验和）。 请确保 XFS 文件系统不使用任何 v5 功能。 可以使用 xfs_info 实用工具来检查 XFS 超级块中是否存在分区。 如果 ftype 设置为 1，则会使用 XFSv5 功能。</p><p>在 Red Hat Enterprise Linux 7 和 CentOS 7 服务器上，必须安装 lsof 实用工具并使其可用。</p>|


## <a name="disaster-recovery-of-hyper-v-virtual-machines-to-azure-no-virtual-machine-manager"></a>将 Hyper-V 虚拟机灾难恢复到 Azure（无 Virtual Machine Manager）

除了 [Azure 要求](#Azure requirements)中提到的组件，还可参见下方了解为 Virtual Machine Manager 云中的 Hyper-V 虚拟机提供灾难恢复时所需的其他组件。

| **先决条件** | **详细信息** |
| --- | --- |
| **Hyper-V 主机** |<p>一台或多台本地服务器，运行具有最新更新的 Windows Server 2012 R2 并已启用 Hyper-V 角色，或者运行 Microsoft Hyper-V Server 2012 R2。</p><p>Hyper-V 服务器应包含一个或多个虚拟机。</p><p>Hyper-V 服务器应直接或通过代理连接到 Internet。</p><p>Hyper-V 服务器上应已安装 [KB2961977](https://support.microsoft.com/zh-cn/kb/2961977) 中提到的修补程序。</p>|
|**提供程序和代理**| <p>在 Azure Site Recovery 部署期间，将安装 Azure Site Recovery 提供程序。安装提供程序还将在每台运行要保护的虚拟机的 Hyper-V 服务器上安装 Azure 恢复服务代理。</p><p>Site Recovery 保管库中的所有 Hyper-V 服务器应有相同版本的提供程序和代理。</p><p>提供程序将需要通过 Internet 连接到 Azure Site Recovery。流量可以直接发送或通过代理发送。 不支持基于 HTTPS 的代理。代理服务器应允许访问：</p><p> [AZURE.INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)]</p><p>如果在服务器上设置了基于 IP 地址的防火墙规则，请确保这些规则允许与 Azure 通信。</p><p> 允许 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)和 HTTPS (443) 端口。</p>|


## <a name="disaster-recovery-of-hyper-v-virtual-machines-in-virtual-machine-manager-clouds-to-azure"></a>将 Virtual Machine Manager 云中的 Hyper-V 虚拟机灾难恢复到 Azure

除了 [Azure 要求](#Azure requirements)中提到的组件，还可参见下方了解为 Virtual Machine Manager 云中的 Hyper-V 虚拟机提供灾难恢复时所需的其他组件。

| **先决条件** | **详细信息** |
| --- | --- |
| **Virtual Machine Manager** |<p>在 **System Center 2012 R2 或更高版本**上运行的一个或多个 Virtual Machine Manager 服务器。每个 Virtual Machine Manager 服务器都应配置有一个或多个云。</p><p>云应当包含：<br>- 一个或多个 Virtual Machine Manager 主机组。</p><p>- 每个主机组中有一个或多个 Hyper-V 主机服务器或群集。</p><p>若要深入了解如何设置 Virtual Machine Manager 云，请参阅 [如何在 VMM 2012 中创建云](http://social.technet.microsoft.com/wiki/contents/articles/2729.how-to-create-a-cloud-in-vmm-2012.aspx)。</p> |
| **Hyper-V** |<p>Hyper-V 主机服务器必须至少运行具有 Hyper-V 角色或的 Windows Server 2012 R2 或 Microsoft Hyper-V Server 2012 R2，并且安装了最新更新。</p><p> Hyper-V 服务器应包含一个或多个 VM。</p><p> 必须在 Virtual Machine Manager 云中管理 Hyper-V 主机服务器或群集（其包含要复制的 VM）。</p><p>Hyper-V 服务器必须直接或通过代理连接到 Internet。</p><p>Hyper-V 服务器上必须已安装文章 [2961977](https://support.microsoft.com/zh-cn/kb/2961977) 中提到的修补程序。</p><p>Hyper-V 主机服务器需要访问 Internet 才可将数据复制到 Azure。</p> |
| **提供程序和代理** |<p>在 Azure Site Recovery 部署期间，需在 Virtual Machine Manager 服务器上安装 Azure Site Recovery 提供程序，并在 Hyper-V 主机上安装恢复服务代理。提供程序和代理需要通过 Internet 直接连接或通过代理连接到 Azure。不支持基于 HTTPS 的代理。 Virtual Machine Manager 服务器和 Hyper-V 主机上的代理服务器应允许访问：</p><p>[AZURE.INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)] </p><p>如果在 Virtual Machine Manager 服务器上设置了基于 IP 地址的防火墙规则，请确保这些规则允许与 Azure 通信。</p><p> 允许 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)和 HTTPS (443) 端口。</p>|

### <a name="replicated-machine-prerequisites"></a>复制的计算机先决条件
| **组件** | **详细信息** |
| --- | --- |
| **受保护的 VM** | <p>Site Recovery 支持 [Azure](https://technet.microsoft.com/zh-cn/library/cc794868%28v=ws.10%29.aspx) 支持的所有操作系统。</p><p>VM 应符合创建 Azure VM 的 [Azure 先决条件](/documentation/articles/site-recovery-support-matrix-to-azure/#failed-over-azure-vm-requirements)。计算机名称应包含 1 到 63 个字符（字母、数字和连字符）。名称必须以字母或数字开头，并以字母或数字结尾。</p><p>可以在启用 VM 的复制后修改此名称。</p><p>受保护计算机上单个磁盘的容量不应超过 1023 GB。一台 VM 最多可以有 16 个磁盘（因此最大容量为 16 TB）。</p>|


## <a name="disaster-recovery-of-hyper-v-virtual-machines-in-virtual-machine-manager-clouds-to-a-customer-owned-site"></a>将 Virtual Machine Manager 云中的 Hyper-V 虚拟机灾难恢复到客户自有站点

除了 [Azure 要求](#Azure requirements)中提到的组件，还可参见下方了解将 Virtual Machine Manager 云中的 Hyper-V 虚拟机灾难恢复到客户自有站点时所需的其他组件。

| **组件** | **详细信息** |
| --- | --- |
| **Virtual Machine Manager** |<p> 建议在主站点和辅助站点中各部署一个 Virtual Machine Manager 服务器。</p><p> 可以[在单个 VMM 服务器上的云之间复制](/documentation/articles/site-recovery-vmm-to-vmm/#prepare-for-single-server-deployment)。为此，至少需要在 Virtual Machine Manager 服务器上配置两个云。</p><p> Virtual Machine Manager 服务器应至少运行具有最新更新的 System Center 2012 SP1。</p><p> 每个 Virtual Machine Manager 服务器上必须至少有一个或多个云。必须为所有云设置 Hyper-V 容量配置文件。</p><p>云必须包含一个或多个 Virtual Machine Manager 主机组。有关设置 Virtual Machine Manager 云的详细信息，请参阅[准备 Azure Site Recovery 部署](/documentation/articles/site-recovery-support-matrix-to-azure/)。</p> |
| **Hyper-V** | <p>Hyper-V 服务器必须至少运行具有 Hyper-V 角色且安装了最新更新的 Windows Server 2012。</p><p> Hyper-V 服务器应包含一个或多个 VM。</p><p> Hyper-V 主机服务器应位于主要和辅助 VMM 云中的主机组内。</p><p> 如果在 Windows Server 2012 R2 上的群集中运行 Hyper-V，建议安装[更新 2961977](https://support.microsoft.com/zh-cn/kb/2961977)</p><p> 如果正在 Windows Server 2012 的群集中运行 Hyper-V，并且具有基于静态 IP 地址的群集，则不会自动创建群集代理。 必须手动配置群集代理。有关群集代理的详细信息，请参阅 [将副本代理角色群集配置为群集复制](http://social.technet.microsoft.com/wiki/contents/articles/18792.configure-replica-broker-role-cluster-to-cluster-replication.aspx)。</p> |
| **提供程序** | <p>在 Site Recovery 部署期间，在 Virtual Machine Manager 服务器上安装 Azure Site Recovery 提供程序。提供程序通过 HTTPS 443 与站点恢复通信，以协调复制。数据复制是通过 LAN 或 VPN 连接在主要和辅助 Hyper-V 服务器之间发生的。</p><p> 在 Virtual Machine Manager 服务器上运行的提供程序需要访问以下 URL：</p><p>[AZURE.INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)] </p><p>提供程序必须允许从 Virtual Machine Manager 服务器到 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)的防火墙通信，并允许 HTTPS (443) 协议。</p> |


## <a name="url-access"></a>URL 访问
应该可从 VMM 和 Hyper-V 主机服务器访问以下 URL。

|**URL** | **VMM 到 VMM** | **VMM 到 Azure** | **Hyper-V 到 Azure** |
|--- | --- | --- | --- |
|``*.accesscontrol.chinacloudapi.cn`` | ALLOW | 允许 | 允许 |
|``*.backup.windowsazure.cn`` | 不是必需 | 允许 | 允许 |
|``*.hypervrecoverymanager.windowsazure.cn`` | 允许 | 允许 | 允许 |
|``*.store.core.chinacloudapi.cn`` | 允许 | 允许 | 允许 |
|``*.blob.core.chinacloudapi.cn`` | 不是必需 | 允许 | 允许 |
|``https://dev.mysql.com/get/archives/mysql-5.5/mysql-5.5.37-win32.msi`` | 不是必需 | 不是必需 | 不是必需 |
<!--Update_Description:add "URL 访问" section;add anchors to sub titles-->