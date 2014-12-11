<properties urlDisplayName="Azure Site Recovery Overview" pageTitle="Azure 站点恢复概述" metaKeywords="Azure Site Recovery, on-premises, clouds, Azure, VMM, Hyper-V" description="Deploy Azure Site Recovery to protect virtual machines on Hyper-V host servers that are located in VMM clouds. You can deploy from one on-premises site to another, or from an on-premises site to Azure." metaCanonical="" umbracoNaviHide="0" disqusComments="1" title="Azure Site Recovery Overview" editor="jimbe" manager="johndaw" authors="raynew" />

<tags ms.service="site-recovery" ms.workload="backup-recovery" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="11/19/2014" ms.author="raynew" />

# Azure 站点恢复概述

<div class="dev-callout"> 
<p>Azure 站点恢复可在许多情形中安排复制和故障转移：</p>


<ul>
<li>**使用 Hyper-V 复制的本地 VMM 站点到本地 VMM 站点保护** - 安排本地 VMM 站点之间的复制、故障转移和恢复。虚拟机数据从源 Hyper-V 主机服务器复制到目标主机服务器。请阅读 <a href="http://www.windowsazure.cn/zh-cn/documentation/articles/hyper-v-recovery-manager-configure-vault/">Azure 站点恢复入门：使用 Hyper-V 复制的两个本地 VMM 站点之间的保护</a>。</li>

<li>**使用 SAN 复制的本地 VMM 站点到本地 VMM 站点保护** - 在承载源和目标本地站点中虚拟机数据的 SAN 设备之间使用基于存储阵列的复制安排端到端复制、故障转移和恢复。请阅读 <a href="http://www.windowsazure.cn/zh-cn/documentation/services/site-recovery/">Azure 站点恢复入门：: 使用 SAN 复制的两个本地 VMM 站点之间的保护</a>。</li>

<li>**本地 VMM 站点到 Azure 保护** - 在本地 VMM 站点和 Azure 之间安排复制、故障转移和恢复。复制的虚拟机数据存储在 Azure 存储空间中。请阅读 <a href="http://msdn.microsoft.com/zh-cn/library/dn788903.aspx">Azure 站点恢复入门：本地 VMM 站点和 Azure 之间的保护。</a></li>

<li>**使用 InMage 的本地 VMWare 站点到本地 VMWare 站点** - InMage Scout 是一家最近由 Microsoft 收购的公司，它提供本地 VMWare 站点之间的实时复制。目前 InMage 作为一个单独的产品提供，可通过订阅 Azure 站点恢复服务来获取。请阅读 <a href="http://www.windowsazure.cn/zh-cn/documentation/services/site-recovery/">Azure 站点恢复入门：使用 InMage 的本地 VMWare 站点之间的保护</a>。</li>
</ul>

<p>功能表总结了这些情形。</p>

<table border="1">
<thead>
<tr>
	<th>功能</th><th>本地到 Azure</th>
	<th>本地到本地 (Hyper-V)</th>
	<th>本地到本地 (SAN)</th>
</tr>
</thead>

<tr>
<td>数据到 Azure 站点恢复</td>
<td><p>云元数据将发送到 Azure 站点恢复。</p> <p>复制的数据存储在 Azure 存储空间中。</p></td>
<td><p>云元数据将发送到 Azure 站点恢复。</p> <p>复制的数据存储在目标 Hyper-V 服务器指定的位置。</p></td>
<td><p>云元数据将发送到 Azure 站点恢复。</p> <p>复制的数据存储在目标数组存储中。</p></td>
</tr>

<tr>
<td>保管库要求</td>
<td><p>你将需要一个 Azure 帐户来以设置 Azure 站点恢复保管库。请参阅 <a href="http://www.windowsazure.cn/pricing/1rmb-trial/">Azure 免费试用版</a>。<!--Get pricing information at <a href="http://go.microsoft.com/fwlink/?LinkId=378268">Azure 站点恢复管理器定价详细信息</a>。--></p></td>
<td><p>启用了 Azure 站点恢复的 Azure 帐户。</p>
</td><td><p>启用了 Azure 站点恢复的 Azure 帐户。</p></td>
</tr>

<tr>
<td>复制</td>
<td>从源本地 Hyper-V 服务器复制到目标 Azure 存储空间的虚拟机。你可以设置反向复制，以复制回源位置。</td>
<td>从源本地 Hyper-V 的服务器复制到另一个服务器的虚拟机。你可以设置反向复制，以复制回源位置。</td>
<td>从源 SAN 存储设备复制到目标 SAN 设备的虚拟机。你可以设置反向复制，以复制回源位置。</td>
</tr>

<tr>
<td>虚拟机存储</td>
<td>Azure 存储空间中存储的虚拟机硬盘</td>
<td>Hyper-V 主机上存储的虚拟机硬盘。</td>
<td>SAN 存储阵列上存储的虚拟机硬盘。</td>
</tr>

<tr>
<td>Azure 存储空间</td>
<td><p>需要 Azure 存储空间帐户。</p> <p>该帐户应该启用了地域复制，应位于 Azure 站点恢复服务所在的同一区域，且应与同一订阅关联。</p></td>
<td>不适用</td>
<td>不适用</td>
</tr>

<tr>
<td>SAN 存储阵列</td>
<td><p>不适用</p></td>
<td>不适用</td>
<td>SAN 存储阵列在源和目标站点中都必须可用，并且必须由 VMM 管理。有关详细信息，请参阅<a href="http://www.windowsazure.cn/zh-cn/documentation/services/site-recovery/">部署先决条件</a>。 </td>
</tr>

<tr>
<td>VMM 服务器</td>
<td>只在源站点中要求有一台 VMM 服务器。VMM 服务器必须至少有一个云，云中至少包含一个 Hyper-V 主机服务器或群集。</td>
<td>需要有源和目标两台 VMM 服务器，每台上至少有一个云，或者是一台有两个云的 VMM 服务器。云必须包含至少一个 Hyper-V 主机服务器或群集。</td>
<td>需要源和目标 VMM 服务器，每个服务器上至少要有一个云。云必须至少包含一个 Hyper-V 群集。</td>
</tr>

<tr>
<td>VMM System Center 版本</td>
<td>System Center 2012 R2</td>
<td><p>带有 SP1 的 System Center 2012</p><p>System Center 2012 R2</p></td>
<td><p>带有 VMM 更新汇总 5.0 的 System Center 2012 R2</p></td>
</tr>

<tr>
<td>VMM 配置</td>
<td><p>在源和目标站点中设置云</p><p>在源和目标站点中设置虚拟机网络</p><p>在源和目标站点中设置存储分类<p>在源和目标 VMM 服务器上安装提供程序</p></td>
<td><p>在源站点中设置云</p><p>设置 SAN 存储</p><p>在源站点中设置虚拟机网络</p><p>在源 VMM 服务器上安装提供程序</p><p>启用虚拟机保护</p></td>
<td><p>在源和目标站点中设置云</p><p>在源和目标站点中设置虚拟机网络</p><p>在源和目标 VMM 服务器上安装提供程序</p><p>启用虚拟机保护</p></td>
</tr>

<tr>
<td>Azure 站点恢复提供程序</td>
<td>在源 VMM 服务器上安装</td>
<td>在源和目标 VMM 服务器上安装</td>
<td>在源和目标 VMM 服务器上安装</td>
</tr>

<tr>
<td>Azure 恢复服务代理</td>
<td>在位于 VMM 云中的 Hyper-V 主机服务器上安装</td>
<td>不是必需</td>
<td>不是必需</td>
</tr>

<tr>
<td>虚拟机恢复点</td>
<td><p>按时间设置恢复点。</p> <p>指定恢复点应保留多久（0-24 小时）。在第一个小时中，使用公式 90/copy_frequency 创建一个恢复点，然后每小时一次。</p></td>
<td><p>按数量设置恢复点。</p> <p>指定应保留多少个附加恢复点 (0-15)。默认情况下，每隔一小时创建一个恢复点。当设置为零时，则只有最新的恢复点会存储在副本 Hyper-V 主机服务器上。</p></td>
<td>在数组存储设置中配置。</td>
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
<td><p>将源 VMM 服务器上的存储分类映射到目标 VMM 服务器上的存储分类。</p> <p>映射使得源存储分类中的虚拟机硬盘将来在故障转移之后能够位于目标存储分类中。</p><p>如果不启用存储映射，复制的虚拟硬盘将存储在目标 Hyper-V 主机服务器上的默认位置。</p></td>
<td><p>主辅站点中存储阵列和池之间的映射。</p></td>
</tr>

</table>

<!--p>如有问题，请访问 <a href="http://go.microsoft.com/fwlink/?LinkId=313628">Azure 恢复服务论坛</a>。</p-->
