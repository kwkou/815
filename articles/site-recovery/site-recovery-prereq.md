<properties
    pageTitle="使用 Azure Site Recovery 复制到 Azure 时所要满足的先决条件 | Azure"
    description="本文汇总了使用 Azure Site Recovery 服务将 VM 和物理机复制到 Azure 所要满足的先决条件。"
    services="site-recovery"
    documentationcenter=""
    author="rajani-janaki-ram"
    manager="jwhit"
    editor="tysonn" />
<tags
    ms.assetid="e24eea6c-50a7-4cd5-aab4-2c5c4d72ee2d"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="12/11/2016"
    wacn.date="02/10/2017"
    ms.author="rajanaki" />  


#  使用 Azure Site Recovery 复制到 Azure 时所要满足的先决条件

 Site Recovery 是一项 Azure 服务，可通过协调从本地物理服务器和虚拟机到云 (Azure) 或辅助数据中心的的复制，为 BCDR 策略提供辅助。当主要位置发生故障时，可以故障转移到辅助位置，使应用和工作负荷保持可用。当主要位置恢复正常时，可以故障回复到主要位置。有关详细信息，请参阅[什么是 Site Recovery？](/documentation/articles/site-recovery-overview/)

本文汇总了开始使用 Site Recovery 复制到 Azure 时所要满足的先决条件。



## Azure 要求

**要求** | **详细信息**
--- | ---
**Azure 帐户** | 一个 [Azure](https://www.azure.cn/) 帐户。<br/><br/> 你可以从 [1rmb 试用版](/pricing/1rmb-trial/)开始。
**Site Recovery 服务** | [详细了解](/pricing/details/site-recovery/) Site Recovery 定价。 |
**Azure 存储** | 需要使用一个 Azure 存储帐户来存储复制的数据，该帐户必须与恢复服务保管库位于同一区域。复制的数据存储在 Azure 存储中，Azure VM 在发生故障转移时创建。<br/><br/> 根据要用于故障转移 Azure VM 的资源模型，可在 [Resource Manager 模型](/documentation/articles/storage-create-storage-account/)或[经典模型](/documentation/articles/storage-create-storage-account-classic-portal/)中设置帐户。<br></br>可以使用 [GRS](/documentation/articles/storage-redundancy/#geo-redundant-storage) 或 LRS 存储。建议使用 GRS，以便在发生区域性故障或无法恢复主要区域时，能够复原数据。<br/><br/> 如果在 Azure 门户中复制物理服务器，则支持使用高级存储。[高级存储](/documentation/articles/storage-premium-storage/)通常用于 IO 性能一贯较高且延迟一贯较低、托管 IO 密集型工作负荷的虚拟机。如果使用高级存储来存储复制的数据，还需要使用一个标准存储帐户来存储复制日志，以便捕获对本地数据所做的更改。<br/><br/>
**存储限制** | 无法跨资源组移动 Site Recovery 中使用的存储帐户，不管是在订阅内部，还是跨不同的订阅。
**Azure 网络** | 需要创建一个 Azure 网络，故障转移后，Azure VM 将连接到该网络。该网络必须与恢复服务保管库位于同一区域。<br/><br/> 在 Azure 门户中，可在 [Resource Manager 模型](/documentation/articles/virtual-networks-create-vnet-arm-pportal/)或[经典模型](/documentation/articles/virtual-networks-create-vnet-classic-pportal/)中创建网络。<br/><br/> 如果从 VMM 复制到 Azure，需在 VMM VM 网络和 Azure 网络之间设置[网络映射](/documentation/articles/site-recovery-network-mapping/)，确保 Azure VM 在故障转移后连接到相应的网络。
**网络限制** | 无法跨资源组移动 Site Recovery 中使用的网络帐户，不管是在订阅内部，还是跨不同的订阅。
**网络映射** | 如果复制 VMM 云中的 Hyper-V VM，需要设置[网络映射](/documentation/articles/site-recovery-network-mapping/)，以便在故障转移之后创建 VM 时，Azure VM 可连接到相应的网络。

>[AZURE.NOTE]
以下部分介绍了客户环境中各个组件的先决条件。有关针对特定配置的支持的详细信息，请阅读[支持矩阵](/documentation/articles/site-recovery-support-matrix/)。
>

## 将物理 Windows/Linux 服务器灾难恢复到 Azure
下面是除了 [上述] 要求（#Azure 要求）以外，为物理 Windows/Linux 服务器提供灾难恢复所需的其他组件。


1. **配置服务器或附加的进程服务器**：需将一台本地计算机设置为配置服务器，用于协调本地站点与 Azure 之间的通信，以及管理数据复制。<br></br>

2\.**复制的计算机先决条件**


| **组件** | **要求** |
| --- | --- |
| **Windows 计算机（物理机）** | 计算机应运行支持的 64 位操作系统：Windows Server 2012 R2、Windows Server 2012 或 Windows Server 2008 R2 SP1 及更高版本。<br/><br/> 操作系统应安装在 C:\\ 驱动器上。OS 磁盘应是 Windows 基本磁盘而不是动态磁盘。数据磁盘可以是动态的。<br/><br/>|
| **Linux 计算机**（物理机） | 需要安装支持的 64 位操作系统：Red Hat Enterprise Linux 6.7、7.1、7.2；Centos 6.5、6.6、6.7、7.0、7.1、7.2；运行 Red Hat 兼容内核或 Unbreakable Enterprise Kern Release 3 (UEK3) 的 Oracle Enterprise Linux 6.4、6.5；SUSE Linux Enterprise Server 11 SP3。<br/><br/>受保护计算机上的 /etc/hosts 文件应该包含将本地主机名映射到所有网络适配器关联的 IP 地址的条目。<br/><br/>如果要在故障转移后使用 Secure Shell 客户端 (ssh) 连接到运行 Linux 的 Azure 虚拟机，请确保将受保护的计算机上的 Secure Shell 服务设置为在系统启动时自动启动，并且防火墙规则允许建立到它的 ssh 连接。<br/><br/>主机名、装载点、设备名称以及 Linux 系统路径和文件名（例如 /etc/；/usr）只能采用英文形式。<br/><br/>

## 将 Hyper-V 虚拟机灾难恢复到 Azure（不包含 VMM）

下面是除了 [上述] 要求（#Azure 要求）以外，为 VMM 云中的 Hyper-V 虚拟机提供灾难恢复所需的其他组件。

| **先决条件** | **详细信息** |
| --- | --- |
| **Hyper-V 主机** |一台或多台本地服务器，运行已启用最新更新和 Hyper-V 角色的 Windows Server 2012 R2，或者运行 Microsoft Hyper-V Server 2012 R2。<br></br>Hyper-V 服务器应包含一个或多个虚拟机。<br></br>Hyper-V 服务器应直接或通过代理连接到 Internet。<br></br>Hyper-V 服务器应安装 [KB2961977](https://support.microsoft.com/zh-cn/kb/2961977) 中提到的修补程序。
|**提供程序和代理**| 在 Azure Site Recovery 部署期间，将安装 Azure Site Recovery 提供程序。安装提供程序还将在运行所要保护的虚拟机的每台 Hyper-V 服务器上安装 Azure 恢复服务代理。<br></br>Site Recovery 保管库中的所有 Hyper-V 服务器都应当具有相同版本的提供程序和代理。<br></br>提供程序需要通过 Internet 连接到 Azure Site Recovery。流量可以直接发送或通过代理发送。请注意，不支持基于 HTTPS 的代理。代理服务器应该允许访问：<br/><br/> [AZURE.INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)]<br/><br/>如果在服务器上设置了基于 IP 地址的防火墙规则，请确保这些规则允许与 Azure 通信。<br></br> 允许 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)和 HTTPS (443) 端口。


## 将 VMM 云中的 Hyper-V 虚拟机灾难恢复到 Azure

下面是除了 [上述] 要求（#Azure 要求）以外，为 VMM 云中的 Hyper-V 虚拟机提供灾难恢复所需的其他组件。

| **先决条件** | **详细信息** |
| --- | --- |
| **VMM** |在 **System Center 2012 R2 或更高版本**上运行的一个或多个 VMM 服务器。应在每个 VMM 服务器上配置一个或多个云。<br></br>云应包含<br>- 一个或多个 VMM 主机组。<br/>- 每个主机组中包含一个或多个 Hyper-V 主机服务器或群集。<br/><br/>[详细了解](http://social.technet.microsoft.com/wiki/contents/articles/2729.how-to-create-a-cloud-in-vmm-2012.aspx)如何设置 VMM 云。 |
| **Hyper-V** |Hyper-V 主机服务器必须至少运行具有 Hyper-V 角色或 **Microsoft Hyper-V Server 2012 R2** 且安装了最新更新的 **Windows Server 2012 R2**。<br/><br/> Hyper-V 服务器应包含一个或多个 VM。<br/><br/> 必须在 VMM 云中管理包含要复制的 VM 的 Hyper-V 主机服务器或群集。<br/><br/>Hyper-V 服务器应直接或通过代理连接到 Internet。<br/><br/>Hyper-V 服务器上应已安装文章 [2961977](https://support.microsoft.com/zh-cn/kb/2961977) 中所述的修复程序。<br/><br/>Hyper-V 主机服务器需有 Internet 访问权限，以便将数据复制到 Azure。 |
| **提供程序和代理** |在 Azure Site Recovery 的部署期间，要在 VMM 服务器上安装 Azure Site Recovery 提供程序，在 Hyper-V 主机上安装恢复服务代理。该提供程序和代理需经由 Internet 直接连接或由代理连接到 Azure。不支持基于 HTTPS 的代理。VMM 服务器和 Hyper-V 主机上的代理服务器应该允许访问：<br/><br/>[AZURE.INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)] <br/><br/>如果在 VMM 服务器上设置了基于 IP 地址的防火墙规则，请确保这些规则允许与 Azure 通信。<br/><br/> 允许 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)和 HTTPS (443) 端口。|

### 复制的计算机先决条件
| **组件** | **详细信息** |
| --- | --- |
| **受保护的 VM** | [Azure](https://technet.microsoft.com/zh-cn/library/cc794868%28v=ws.10%29.aspx) 支持的所有操作系统都受 Site Recovery 的支持。<br></br>VM 应符合创建 Azure VM 的 [Azure 先决条件](/documentation/articles/site-recovery-best-practices/#azure-virtual-machine-requirements)。计算机名称应包含 1 到 63 个字符（字母、数字和连字符）。名称必须以字母或数字开头，并以字母或数字结尾。<br></br>可以在启用 VM 的复制后修改此名称。<br/><br/>受保护计算机上单个磁盘的容量不应超过 1023 GB。一台 VM 最多可以有 16 个磁盘（因此最大容量为 16 TB）。<br/><br/>


## 将 VMM 云中的 Hyper-V 虚拟机灾难恢复到客户拥有的站点

下面是除了 [上述] 要求（#Azure 要求）以外，将 VMM 云中的 Hyper-V 虚拟机灾难恢复到客户拥有的站点所需的其他组件。

| **组件** | **详细信息** |
| --- | --- |
| **VMM** | 我们建议在主站点和辅助站点中各部署一个 VMM 服务器。<br/><br/> 可[在单个 VMM 服务器上的云之间复制](/documentation/articles/site-recovery-single-vmm/)。为此，至少需要在 VMM 服务器上配置两个云。<br/><br/> VMM 服务器应当至少运行具有最新更新的 **System Center 2012 SP1**。<br/><br/> 每个 VMM 服务器必须有一个或多个云。必须在所有云中设置 Hyper-V 容量配置文件。<br/><br/>云必须包含一个或多个 VMM 主机组。[详细了解](https://msdn.microsoft.com/zh-cn/library/azure/dn469075.aspx#BKMK_Fabric)如何设置 VMM 云。<br/><br/> VMM 服务器需要 Internet 访问权限。 |
| **Hyper-V** | Hyper-V 服务器必须至少运行**具有 Hyper-V 角色且安装了最新更新的 Windows Server 2012**。<br/><br/> Hyper-V 服务器应包含一个或多个 VM。<br/><br/> Hyper-V 主机服务器应位于主要和辅助 VMM 云中的主机组内。<br/><br/> 如果正在 Windows Server 2012 R2 上的群集中运行 Hyper-V，则应安装[更新 2961977](https://support.microsoft.com/zh-cn/kb/2961977)<br/><br/> 如果正在 Windows Server 2012 上的群集中运行 Hyper-V，请注意，若使用基于静态 IP 地址的群集，则不会自动创建该群集中转站。需要手动配置群集中转站。[了解详细信息](http://social.technet.microsoft.com/wiki/contents/articles/18792.configure-replica-broker-role-cluster-to-cluster-replication.aspx)。 |
| **提供程序** | Site Recovery 部署期间，需要在 VMM 服务器上安装 Azure Site Recovery 提供程序。提供程序通过 HTTPS 443 与 Site Recovery 通信，协调复制。数据复制是通过 LAN 或 VPN 连接在主要和辅助 Hyper-V 服务器之间发生的。<br/><br/> 在 VMM 服务器上运行的提供程序需要访问以下 URL：<br/><br/>[AZURE.INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)] <br/><br/>允许从 VMM 服务器到 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)的防火墙通信，并允许 HTTPS (443) 协议。 |

<!---HONumber=Mooncake_0206_2017-->