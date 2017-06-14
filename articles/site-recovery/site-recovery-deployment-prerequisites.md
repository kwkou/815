<properties
    pageTitle="Azure Site Recovery 部署先决条件 | Azure"
    description="本文介绍通过 Azure Site Recovery 进行复制设置的先决条件。"
    services="site-recovery"
    documentationcenter=""
    author="rayne-wiselman"
    manager="jwhit"
    editor="tysonn" />  

<tags
    ms.assetid="e24eea6c-50a7-4cd5-aab4-2c5c4d72ee2d"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="12/04/2016"
    wacn.date="01/11/2017"
    ms.author="raynew" />  


# Site Recovery 部署先决条件

组织需要制定 BCDR 策略来确定应用、工作负荷和数据如何在计划和非计划停机期间保持运行和可用，并尽快恢复正常运行情况。Site Recovery 是一项 Azure 服务，可以通过协调从本地物理服务器和虚拟机到云 (Azure) 或辅助数据中心的复制，为 BCDR 策略提供辅助。当主要位置发生故障时，你可以故障转移到辅助位置，使应用和工作负荷保持可用。当主要位置恢复正常时，你可以故障回复到主要位置。有关详细信息，请参阅[什么是 Site Recovery？](/documentation/articles/site-recovery-overview/)

本文汇总了 Site Recovery 复制方案的部署先决条件。


## Azure 部署模型

Azure 提供两种不同的[部署模型](/documentation/articles/resource-manager-deployment-model/)用于创建和处理资源：Azure Resource Manager 模型和经典模型。Azure 还有两个门户 - 支持经典部署模型的 [Azure 经典管理门户](https://manage.windowsazure.cn/)，以及支持两种部署模型的 [Azure 门户](https://portal.azure.cn/)。

站点恢复在 Azure China 目前仅支持在经典管理门户中部署。可以在经典管理门户中维护保管库，但不能创建新的保管库。


## Azure 帐户要求

**要求** | **详细信息**
--- | --- 
**Azure 帐户** | <p>一个 [Azure](https://azure.cn/) 帐户。</p><p> 你可以从 [1rmb 试用版](/pricing/1rmb-trial/)开始。[详细了解](/pricing/details/site-recovery/) Site Recovery 定价。</p>


## Azure 存储要求

复制的数据存储在 Azure 存储中，Azure VM 在发生故障转移时创建。

**要求** | **详细信息**
--- | --- 
[Azure 存储帐户](/documentation/articles/storage-introduction/) | <p>可以使用 [GRS](/documentation/articles/storage-redundancy/#geo-redundant-storage) 或 LRS 存储。</p><p> 建议使用 GRS，以便在发生区域性故障或无法恢复主要区域时，能够复原数据。[了解详细信息](/documentation/articles/storage-redundancy)</p>
**Azure 门户** | 在 Azure 门户中，可以使用 Resource Manager 存储，也可使用经典存储帐户。
**高级存储** | <p> 高级存储通常用于 IO 性能一贯较高且延迟一贯较低、托管 IO 密集型工作负荷的虚拟机。</p><p> 如果使用高级存储，还需要一个标准存储帐户来存储复制日志，以便捕获对本地数据所做的更改。</p>
**存储限制** | <p>经典管理门户仅支持 GRS。</p><p> 在 Azure 门户中创建的存储帐户不能跨资源组移动。</p>。

## Azure 网络要求

需要一个 Azure 网络，以便故障转后创建的 Azure VM 连接到该网络。

**要求** | **详细信息**
--- | ---
**网络区域** | 该网络必须位于与保管库相同的区域中。
**Azure 门户** | 在 Azure 门户中，可以使用 Resource Manager 网络或经典网络。
**网络映射** | 如果从 VMM 复制到 Azure，需在 VMM VM 网络和 Azure 网络之间设置[网络映射](/documentation/articles/site-recovery-network-mapping/)，确保 Azure VM 在故障转移后连接到相应的网络。





### 物理机要求（复制到 Azure）

**组件** | **要求**
--- | ---
**运行 Windows 的物理服务器** | <p>Windows 计算机应运行[支持的](/documentation/articles/site-recovery-support-matrix/#support-for-replicated-machines) 64 位操作系统。</p><p> 操作系统应安装在 C:\\ 驱动器上。</p><p> OS 磁盘应是 Windows 基本磁盘而不是动态磁盘。数据磁盘可以是动态的。</p><p> Site Recovery 支持使用 RDM 磁盘的 VM。在故障回复期间，如果原始的源 VM 和 RDM 磁盘可用，Site Recovery 会重复使用 RDM 磁盘。如果这些磁盘不可用，则 Site Recovery 会在故障回复期间为每个磁盘创建一个新的 VMDK 文件。</p>
**运行 Linux 的物理服务器** | <p>Linux 计算机应运行[支持的操作系统](/documentation/articles/site-recovery-support-matrix/#support-for-replicated-machines)。</p><p>受保护计算机上的 /etc/hosts 文件包含的条目应将本地主机名映射到与所有网络适配器相关联的 IP 地址。</p><p> 主机名、装载点、设备名称，以及 Linux 系统路径和文件名（例如 /etc/；/usr）只能采用英文形式。</p><p> 应[支持](/documentation/articles/site-recovery-support-matrix/#support-for-replicated-machines)存储。</p>

## Hyper-V 复制要求（复制到 Azure）

**组件** | **要求**
--- | ---
**VMM（可选）** | <p>可以选择性地在 VMM 云中托管的 Hyper-V 主机上复制 VM。</p><p> 如果不使用 VMM，则可将一个或多个 Hyper-V 主机或群集收集到 Hyper-V 站点中，然后为站点设置复制。</p><p> 如果使用 VMM，它应在 System Center 2012 R2 上运行。</p><p> 如果使用 VMM，应在每个 VMM 服务器上配置一个或多个云。云应包含一个或多个 VMM 主机组，每个主机组中包含一个或多个 Hyper-V 主机服务器或群集。</p><p> VMM 服务器应连接到 Internet，并可访问[所需 URL](#requirements-for-url-access)。</p><p> 如果在 VMM 服务器上设置了基于 IP 地址的防火墙规则，请确保这些规则允许与 Azure 通信。</p><p> 允许 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)和 HTTPS (443) 端口。</p>
**Hyper-V 主机** | <p>Hyper-V 主机服务器必须至少运行具有 Hyper-V 角色或 **Microsoft Hyper-V Server 2012 R2** 且安装了最新更新的 **Windows Server 2012 R2**。</p><p> Hyper-V 服务器应包含一个或多个 VM。</p><p>Hyper-V 服务器应直接或通过代理连接到 Internet。</p><p>Hyper-V 服务器应已安装文章 [2961977](https://support.microsoft.com/zh-cn/kb/2961977) 中所述的修补程序。</p><p>Hyper-V 主机服务器需有将数据复制到 Azure 的 Internet 访问权限，包括[所需 URL](#requirements-for-url-access) 的访问权限。另外，允许 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)和 HTTPS (443) 端口。</p><p> 如果 Hyper-V 主机位于 VMM 云中，请确保安装 [KB 2961977](https://support.microsoft.com/zh-cn/kb/2961977)</p> 中所述的修补程序

### Hyper-V VM 要求（复制到 Azure）

**组件** | **要求**
--- | ---
**VM 合规性** |在故障转移 VM 之前，请确保分配到 Azure VM 的名称符合 [Azure 先决条件](/documentation/articles/site-recovery-best-practices/#azure-virtual-machine-requirements)。可以在启用 VM 的复制后修改此名称。
**磁盘** | <p>受保护计算机上单个磁盘的容量不应超过 1023 GB。一台 VM 最多可以有 64 个磁盘（因此最大容量为 64 TB）。</p><p> 不支持共享磁盘来宾群集。</p>
**UEFI** | 不支持统一可扩展固件接口 (UEFI)/可扩展固件接口 (EFI) 引导模式。
**NIC 组合** | 如果源 VM 具有 NIC 组合，该组合会在故障转移到 Azure 后转换为单一 NIC。
**Linux 静态** | 不支持保护运行 Linux 且使用静态 IP 地址的 Hyper-V VM。





## Hyper-V 复制要求（辅助站点）

**组件** | **要求**
--- | ---
**VMM** | 若要将 Hyper-V VM 复制到辅助站点，必须在 System Center VMM 云中管理 Hyper-V host 主机。<br/><br/> VMM 必须至少运行具有最新更新的 System Center 2012 SP1。<br/><br/> 建议在主站点和辅助站点中各有一个 VMM 服务器。在此方案中，用户可以在[同一 VMM 服务器的不同云](/documentation/articles/site-recovery-single-vmm/)之间复制，但需要进行一些手动配置。<br/><br/> VMM 服务器应连接到 Internet，并可访问[所需 URL](#requirements-for-url-access)。<br/><br/> 如果在 VMM 服务器上设置了基于 IP 地址的防火墙规则，请确保这些规则允许与 Azure 通信。<br/><br/> 允许 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)和 HTTPS (443) 端口。
**Hyper-V** | Hyper-V 服务器必须至少运行安装了 Hyper-V 角色和最新更新的 Windows Server 2012。<br/><br/> Hyper-V 服务器应包含一个或多个 VM<br/><br/> Hyper-V 主机应位于主 VMM 服务器和辅助 VMM 服务器上的主机组中。<br/><br/> 如果在 Windows Server 2012 R2 上的群集中运行 Hyper-V，应安装更新 [2961977](https://support.microsoft.com/zh-cn/kb/2961977)。如果用户在 Windows Server 2012 上有 Hyper-V 群集，并且有基于静态 IP 地址的群集，则不会自动创建群集代理。[了解更多](http://social.technet.microsoft.com/wiki/contents/articles/18792.configure-replica-broker-role-cluster-to-cluster-replication.aspx)有关手动配置的内容。

### Hyper-V VM 要求（辅助站点）

**组件** | **详细信息**
--- | ---
**VMM 云** | VM 必须位于 VMM 云中的 Hyper-V 主机上。






##<a name="requirements-for-url-access"></a> URL 访问要求

这些 URL 应该在 VMM 和 Hyper-V 主机服务器中提供。

**URL** | **VMM 到 VMM** | **VMM 到 Azure** | **Hyper-V 到 Azure** 
--- | --- | --- | --- 
``*.accesscontrol.chinacloudapi.cn``   | 允许 | 允许 | 允许 
``*.backup.windowsazure.cn`` | 不是必需 | 允许 | 允许 
``*.hypervrecoverymanager.windowsazure.cn`` | 允许 | 允许 | 允许 
``*.store.core.chinacloudapi.cn`` | 允许 | 允许 | 允许 
``*.blob.core.chinacloudapi.cn`` | 不是必需 | 允许 | 允许 
``https://www.msftncsi.com/ncsi.txt`` | 允许 | 允许 | 允许 
``https://dev.mysql.com/get/archives/mysql-5.5/mysql-5.5.37-win32.msi``   | 不是必需 | 不是必需 | 不是必需 
``time.windows.cn``   | 允许 | 允许 | 允许 
``time.nist.gov``   | 允许 | 允许 | 允许 

## 故障转移要求

**方案** | **测试** | **计划内** | **计划外**
--- | --- | --- | ---
物理机到 Azure | 支持 | 不支持 | 支持
Hyper-V (VMM) 到 Azure | 支持 | 支持 | 支持
Hyper-V（无 VMM）到 Azure | 支持 | 支持 | 不支持
Hyper-V（主 VMM/云）到辅助站点 | 支持 | 支持 | 支持

## 故障回复要求

**方案** | **测试** | **计划内** | **计划外**
--- | --- | --- | ---
**Azure 到 VMM** | 不支持 | 支持 | 不支持
**Azure 到 Hyper-V（无 VMM）** | 不支持 | 支持 | 不支持
**Hyper-V（辅助 VMM/云）到主站点** | 支持 | 支持 | 支持






## 部署优化

使用以下提示来优化和缩放部署。

- **操作系统卷大小**：将虚拟机复制到 Azure 时，操作系统卷必须小于 1TB。如果你的卷容量超过此值，可以在开始部署之前，手动将卷容量转移到另一个磁盘。
- **数据磁盘大小**：如果要复制到 Azure，一台虚拟机中最多可以包含 64 个数据磁盘，每个磁盘的最大大小为 1 TB。可以有效地复制和故障转移约 64 TB 的虚拟机。
- **恢复计划限制**：Site Recovery 可以扩展到数千个虚拟机。恢复计划旨在用作应一起故障转移的应用程序的模型，因此，我们可以将一个恢复计划中的计算机数目限制为 50。
- **Azure 服务限制**：每个 Azure 订阅在核心、云服务等方面附带了一组默认限制。建议运行测试故障转移，验证订阅中资源的可用性。可以通过 Azure 支持人员修改这些限制。
- **容量规划**：阅读 Site Recovery 的[容量规划](/documentation/articles/site-recovery-capacity-planner/)相关信息。
- **复制带宽**：如果你的复制带宽不足，请注意：
- **ExpressRoute**：可以配合 Azure ExpressRoute 和 WAN 优化器（如 Riverbed）来使用 Site Recovery。[详细了解](http://blogs.technet.com/b/virtualization/archive/2014/07/20/expressroute-and-azure-site-recovery.aspx)有关 ExpressRoute 的信息。
- **复制流量**：Site Recovery 用户只能使用数据块（而不是整个 VHD）执行智能初始复制。在复制过程中，只会复制更改。
- **网络流量**：你可以通过使用基于目标 IP 地址和端口的策略设置 [Windows QoS](https://technet.microsoft.com/zh-cn/library/hh967468.aspx)，来控制用于复制的网络流量。此外，如果要使用 Azure 备份代理复制到 Azure Site Recovery，可为该代理配置限制。[了解详细信息](https://support.microsoft.com/zh-cn/kb/3056159)。
- **RTO**：若要度量使用 Site Recovery 时预期的恢复时间目标 (RTO)，我们建议运行测试故障转移并查看 Site Recovery 作业，分析完成操作所花费的时间。如果你要故障转移到 Azure，为实现最佳 RTO，我们建议你通过与 Azure 自动化和恢复计划集成来自动化所有手动操作。
- **RPO**：当你复制到 Azure 时，Site Recovery 支持近乎同步的恢复点目标 (RPO)。这假设数据中心和 Azure 之间有足够的带宽。



## 后续步骤
查看常规部署要求之后，请阅读详细的先决条件并部署方案。

* [将 VMM 云中的 Hyper-V 服务器复制到 Azure](/documentation/articles/site-recovery-vmm-to-azure/)
* [将 Hyper-V 虚拟机（不使用 VMM）复制到 Azure](/documentation/articles/site-recovery-hyper-v-site-to-azure/)
* [将 Hyper-V VM 复制到辅助站点](/documentation/articles/site-recovery-vmm-to-vmm/)
* [使用 SAN 将 Hyper-V VM 复制到辅助站点](/documentation/articles/site-recovery-vmm-san/)
* [使用单个 VMM 服务器复制 Hyper-V VM](/documentation/articles/site-recovery-single-vmm/)

<!---HONumber=Mooncake_1226_2016-->