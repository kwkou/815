<properties
	pageTitle="Site Recovery 概述"
	description="Azure Site Recovery 可以协调位于本地的虚拟机和物理服务器到 Azure 或辅助本地站点的复制、故障转移和恢复。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.date="05/10/2015"
	wacn.date="10/03/2015"/>

#  Site Recovery 概述

Site Recovery 服务有助于构建稳健的业务连续性和灾难恢复 (BCDR) 解决方案，并通过协调和自动化到 Azure 或辅助本地数据中心的复制与故障转移，来保护本地物理服务器和虚拟机。

- **简化** - 使用 Site Recovery 可以轻松配置复制并针对本地工作负载及应用程序运行故障转移和恢复，因此可帮助简化 BCDR 策略。
- **复制** - 可将本地工作负载复制到 Azure 存储空间或辅助数据中心。
- **保管库** - 若要管理复制，可在选择的 Azure 区域中设置一个 Site Recovery 保管库。所有元数据将保留在该区域。
- **元数据** - 应用程序数据不会发送到 Azure。Site Recovery 只需要虚拟机名称和 VMM 云名称等元数据，以便协调复制和故障转移。
- **与 Azure 建立连接** - 管理服务器根据你的部署方案与 Azure 通信。例如，如果你要复制位于本地 VMM 云中的虚拟机，则 VMM 服务器将通过加密的出站 HTTPS 连接来与 Site Recovery 通信。不需要从虚拟机或 Hyper-V 主机建立连接。
- **Hyper-V 副本** - Azure Site Recovery 为复制过程使用 Hyper-V 副本；如果你要在两个本地 VMM 站点之间复制，则还可以使用 SAN 复制。执行初始复制时，Site Recovery 将使用智能复制 - 只复制数据块而不是整个 VHD。对于正在进行的复制，只会复制增量更改。Site Recovery 支持脱机数据传输，并可与 WAN 优化器配合工作。
- **价格** - 可以[了解更多](/home/features/site-recovery/#price)有关 Site Recovery 价格的信息。


## 部署方案

下表汇总了 Site Recovery 支持的复制方案。

**复制目标** | **复制源（本地）** | **详细信息** | **文章**
---|---|---|---
Azure | Hyper-V 站点 | 将定义为 Hyper-V 站点的一个或多个本地 Hyper-V 主机服务器上的虚拟机复制到 Azure。不需要 VMM 服务器。 | [了解详细信息](/documentation/articles/site-recovery-hyper-v-site-to-azure)
Azure| VMM 服务器 | 将 VMM 云中一个或多个本地 Hyper-V 主机服务器上的虚拟机复制到 Azure。 | [了解详细信息](/documentation/articles/site-recovery-vmm-to-azure)
Azure | 物理 Windows 服务器 | 将物理 Windows 或 Linux 服务器复制到 Azure | [了解详细信息](/documentation/articles/site-recovery-vmware-to-azure)
Azure | VMware 虚拟机 | 将 VMware 虚拟机复制到 Azure | [了解详细信息](/documentation/articles/site-recovery-vmware-to-azure)
辅助数据中心 | VMM 服务器 | 将 VMM 云中本地 Hyper-V 主机服务器上的虚拟机复制到另一数据中心内的辅助 VMM 服务器 | [了解详细信息](/documentation/articles/site-recovery-vmm-to-vmm)
辅助数据中心 | 使用 SAN 的 VMM 服务器 | 使用 SAN 复制将 VMM 云中本地 Hyper-V 主机服务器上的虚拟机复制到另一数据中心内的辅助 VMM 服务器| [了解详细信息](/documentation/articles/site-recovery-vmm-san)
辅助数据中心 | 单个 VMM 服务器 | 将 VMM 云中本地 Hyper-V 主机服务器上的虚拟机复制到同一 VMM 服务器上的辅助云 | [了解详细信息](/documentation/articles/site-recovery-single-vmm)


## 工作负载指南

ASR 复制技术与虚拟机中运行的任何应用程序兼容。我们与应用程序产品团队合作展开了更多测试，以进一步支持每个应用程序。

**工作负载** | <p>**复制 Hyper-V 虚拟机**</p> <p>**（目标为辅助站点）**</p> | <p>**复制 Hyper-V 虚拟机**</p> <p>**（目标为 Azure）**</p> | <p>**复制 VMware 虚拟机**</p> <p>**（目标为辅助站点）**</p> | <p>**复制 VMware 虚拟机**</p><p>**（目标为 Azure）**</p>  
---|---|---|---|---
Active Directory、DNS | Y | Y | Y | 即将支持
Web Apps（IIS、SQL） | Y | Y | Y | 即将支持
SCOM | Y | Y | Y | 即将支持  
SAP</b><p>将非群集 SAP 站点复制到 Azure</p> | Y（Microsoft 已测试） | Y（Microsoft 已测试） | Y（Microsoft 已测试） | 即将支持  
Exchange（非 DAG） | Y | 即将支持 | Y | 即将支持
远程桌面/VDI | Y | Y | Y | 即将支持
Linux</b><p>（操作系统和应用程序）</p> | Y（Microsoft 已测试） | Y（Microsoft 已测试） | Y（Microsoft 已测试） | 即将支持
Dynamics AX | Y | Y | Y | 即将支持
Dynamics CRM | 即将支持 | 即将支持 | Y | 即将支持
Oracle | 即将支持 | 即将支持 | Y（Microsoft 已测试） | 即将支持


## 功能和要求

下表汇总了 Site Recovery 的主要功能，以及在复制到 Azure 期间和使用默认 Hyper-V 副本复制与 SAN 复制到辅助站点期间，如何处理这些功能。

<table border="1">
<thead>
<tr>
	<th>功能</th><th>复制到 Azure</th>
	<th>复制到辅助站点（Hyper-V 副本）</th>
	<th>复制到辅助站点 (SAN)</th>
</tr>
</thead>

<tr>
<td>数据复制</td>
<td><p>有关本地服务器和虚拟机的元数据将存储在 Site Recovery 保管库中。</p> <p>复制的数据存储在 Azure 存储空间中。</p></td>
<td><p>有关本地服务器和虚拟机的元数据将存储在 Site Recovery 保管库中。</p> <p>复制的数据存储在目标 Hyper-V 服务器指定的位置。</p></td>
<td><p>有关本地服务器和虚拟机的元数据将存储在 Site Recovery 保管库中。</p> <p>复制的数据存储在目标数组存储中。</p></td>
</tr>

<tr>
<td>保管库要求</td>
<td><p>支持 Site Recovery 服务的 Azure 帐户</p></td>
<td><p>支持 Site Recovery 服务的 Azure 帐户</p>
</td><td><p>支持 Site Recovery 服务的 Azure 帐户</p></td>
</tr>

<tr>
<td>复制</td>
<td>将源 Hyper-V 主机中的虚拟机复制到 Azure 存储空间。故障回复到源位置。</td>
<td>将源 Hyper-V 主机中的虚拟机复制到目标 Hyper-V 主机。故障回复到源位置。</td>
<td>将源 SAN 存储设备中的虚拟机复制到目标 SAN 设备。故障回复到源位置。</td>
</tr>

<tr>
<td>虚拟机</td>
<td>Azure 存储空间中存储的虚拟机硬盘</td>
<td>Hyper-V 主机上存储的虚拟机硬盘</td>
<td>SAN 存储阵列上存储的虚拟机硬盘</td>
</tr>

<tr>
<td>Azure 存储空间</td>
<td>需要用来存储已复制的虚拟机硬盘</td>
<td>不适用</td>
<td>不适用</td>
</tr>

<tr>
<td>SAN 存储阵列</td>
<td><p>不适用</p></td>
<td>不适用</td>
<td>SAN 存储阵列在源和目标站点中都必须可用，并且必须由 VMM 管理</td>
</tr>

<tr>
<td>VMM 服务器</td>
<td>只要求源站点中有 VMM 服务器。</td>
<td>建议在源和目标站点中提供 VMM 服务器。可以在单个 VMM 服务器上的云之间复制。</td>
<td>要求源和目标 VMM 站点中有 VMM 服务器。云必须至少包含一个 Hyper-V 群集。</td>
</tr>

<tr>
<td>VMM 版本</td>
<td>System Center 2012 R2</td>
<td><p>System Center 2012 SP1</p><p>System Center 2012 R2</p></td>
<td><p>带有 VMM 更新汇总 5.0 的 System Center 2012 R2</p></td>
</tr>

<tr>
<td>VMM 配置</td>
<td><p>在源和目标站点中设置云</p><p>在源和目标站点中设置 VM 网络</p><p>在源和目标站点中设置存储分类 <p>在源和目标 VMM 服务器上安装提供程序</p></td>
<td><p>在源站点中设置云</p><p>设置 SAN 存储</p><p>在源站点中设置 VM 网络</p><p>在源 VMM 服务器上安装提供程序</p><p>启用虚拟机保护</p></td>
<td><p>在源和目标站点中设置云</p><p>在源和目标站点中设置 VM 网络</p><p>在源和目标 VMM 服务器上安装提供程序</p><p>启用虚拟机保护</p></td>
</tr>

<tr>
<td><p>Azure 站点恢复提供程序</p><p>用于通过 HTTPS 连接到 Site Recovery</p></td>
<td>在源 VMM 服务器上安装</td>
<td>在源和目标 VMM 服务器上安装</td>
<td>在源和目标 VMM 服务器上安装</td>
</tr>

<tr>
<td><p>Azure 恢复服务代理</p><p>用于通过 HTTPS 连接到 Site Recovery</p></td>
<td>在 Hyper-V 主机服务器上安装</td>
<td>不是必需</td>
<td>不是必需</td>
</tr>

<tr>
<td>虚拟机恢复点</td>
<td><p>按时间设置恢复点。</p> <p>指定恢复点应保留多久（0-24 小时）</p></td>
<td><p>按数量设置恢复点。</p> <p>指定应保留多少个附加恢复点 (0-15)。默认情况下，每隔一小时创建一个恢复点</p></td>
<td>在阵列存储设置中配置</td>
</tr>

<tr>
<td>网络映射</td>
<td><p>将虚拟机网络映射到 Azure 网络。</p> <p>网络映射可确保在同一源虚拟机网络中进行故障转移的所有虚拟机都可以在故障转移之后进行连接。另外，如果目标 Azure 网络上有网关，那么，虚拟机就可以连接到本地虚拟机。</p><p>如果不启用映射，则只有在同一恢复计划中进行故障转移的虚拟机可以在故障转移到 Azure 之后相互连接。</p></td>
<td><p>将源虚拟机网络映射到目标虚拟机网络。</p> <p>网络映射用于将复制的虚拟机置于最佳 Hyper-V 主机服务器上，并确保与源虚拟机网络相关联的虚拟机在故障转移之后与映射的目标网络相关联。</p><p>如果不启用映射，则不会将复制的虚拟机连接到网络。</p></td>
<td><p>将源虚拟机网络映射到目标虚拟机网络。</p> <p>网络映射确保与源虚拟机网络相关联的虚拟机在故障转移之后与映射的目标网络相关联。</p><p>如果不启用映射，则不会将复制的虚拟机连接到网络。</p></td>
</tr>

<tr>
<td>存储映射</td>
<td>不适用</td>
<td><p>将源 VMM 服务器上的存储分类映射到目标 VMM 服务器上的存储分类。</p> <p>通过映射，源存储分类中的虚拟机硬盘在故障转移后将存储在目标存储分类中。</p><p>如果未启用存储映射，复制的虚拟机硬盘将存储在目标 Hyper-v 主机服务器上的默认位置。</p></td>
<td><p>主辅站点中存储阵列和池之间的映射。</p></td>
</tr>

</table>

## 后续步骤

完成本概述后，请[阅读最佳实践](/documentation/articles/site-recovery-best-practices)，帮助开始进行部署规划。

<!---HONumber=71-->