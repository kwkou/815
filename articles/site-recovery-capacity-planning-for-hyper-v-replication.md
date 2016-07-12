<properties
	pageTitle="为站点恢复运行 Hyper-V Capacity Planner 工具 | Azure"
	description="本文提供 Azure Site Recovery 的 Hyper-V Capacity Planner 工具的用法说明"
	services="site-recovery"
	documentationCenter="na"
	authors="csilauraa"
	manager="jwhit"
	editor="tysonn"/>
<tags
	ms.service="site-recovery"
	ms.date="02/15/2016"
	wacn.date="04/05/2016"/>

# 为站点恢复运行 Hyper-V Capacity Planner 工具

作为 Azure Site Recovery 部署的一部分，你需要确定复制和带宽要求。站点恢复的 Hyper-V Capacity Planner 工具可帮助你确定 Hyper-V 虚拟机复制的复制要求和带宽要求。


本文介绍如何运行 Hyper-V Capacity Planner 工具。此工具应与[站点恢复容量计划](/documentation/articles/site-recovery-capacity-planner/)中所述的其他容量计划工具和信息一起使用。


## 开始之前

在主站点中的 Hyper-V 服务器或群集节点上运行该工具。若要运行 Hyper-V 主机服务器所需的工具，需满足以下要求：

- 操作系统：Windows Server® 2012 或 Windows Server® 2012 R2
- 内存：20 MB（最低）
- CPU：5% 开销（最低）
- 磁盘空间：5 MB（最低）

运行该工具之前，必须准备好主站点。如果在两个本地站点之间复制并想要检查带宽，则还需要准备好副本服务器。


## 步骤 1：准备主站点
1. 在主站点上生成要复制的所有 Hyper-V 虚拟机及其所在 Hyper-V 主机/群集的列表。该工具每次可针对多个独立主机或单个群集运行，但不能同时对此两者运行。还需要单独针对每个操作系统运行，因此你应该汇总并记下你的 Hyper-V 服务器，如下所示： 

  - Windows Server® 2012 独立服务器
  - Windows Server® 2012 群集
  - Windows Server® 2012 R2 独立服务器
  - Windows Server® 2012 R2 群集

3. 在所有 Hyper-V 主机和群集上启用 WMI 远程访问。在每个服务器/群集上运行以下命令，以确保设置防火墙规则和用户权限：

        netsh firewall set service RemoteAdmin enable

5. 在服务器和群集上启用性能监视，如下所示：

  - 使用“高级安全性”管理单元打开 Windows 防火墙，然后启用以下入站规则：“COM+ 网络访问(DCOM-IN)”，以及“远程事件日志管理组”中的所有规则。

## 步骤 2：准备副本服务器（本地到本地复制）

如果要复制到 Azure，则不需要执行此操作。

我们建议设置单个 Hyper-V 主机作为恢复服务器，以便可将虚构 VM 复制到该主机以检查带宽。你可以跳过此步骤，但如果不执行此操作，将无法测量带宽。

1. 如果想要使用群集节点作为副本，请配置 Hyper-V 副本代理：

	- 在“服务器管理器”中，打开“故障转移群集管理器”。
	- 连接到群集，突出显示群集名称，然后单击“操作”>“配置角色”以打开“高可用性”向导。
	- 在“选择角色”中，选择“Hyper-V 副本代理”。在向导中提供“NetBIOS 名称”和“IP 地址”作为群集的连接点（称为客户端接入点）。将配置“Hyper-V 副本代理”，并生成一个客户端接入点名称，你应该记下该名称。
	- 验证 Hyper-V 副本代理角色是否已成功联机，并可以在群集的所有节点之间故障转移。为此，请右键单击该角色，指向“移动”，然后单击“选择节点”。选择节点 >“确定”。
	- 如果你使用基于证书的身份验证，请确保每个群集节点和客户端访问点上都安装了证书。
2.  启用副本服务器：

	- 针对某个群集打开“故障群集管理器”，连接到该群集，然后单击“角色”> 选择角色 >“复制设置”>“启用此群集作为副本服务器”。请注意，如果使用群集作为副本，则还需要在主站点的群集上显示 Hyper-V 副本代理角色。
	- 对于独立服务器，请打开“Hyper-V 管理器”。在“操作”窗格中，单击想要启用的服务器的“Hyper-V 设置”，然后在“复制配置”中单击“启用这台计算机作为副本服务器”。
3. 设置身份验证：

	- 在“身份验证和端口”中选择如何对主服务器进行身份验证以及身份验证端口。如果你使用证书，请单击“选择证书”以选择一个证书。如果主服务器和恢复 Hyper-V 主机位于同一个域或信任的域中，请使用 Kerberos。对于不同的域或工作组部署使用证书。
	- 在“授权和存储”部分中，允许**任何**已经过身份验证的（主）服务器将复制数据发送到这个副本服务器。单击“确定”或“应用”。

	![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image1.png)

	- 运行 **netsh http show servicestate** 检查是否针对指定的协议/端口运行了侦听器：  
4. 设置防火墙。在 Hyper-V 安装期间创建防火墙规则，以允许默认端口上的流量（443 上的 HTTPS 流量，80 上的 Kerberos 流量）。按如下所示启用这些规则：
	
		- Certificate authentication on cluster (443): **Get-ClusterNode | ForEach-Object {Invoke-command -computername \$\_.name -scriptblock {Enable-Netfirewallrule -displayname "Hyper-V Replica HTTPS Listener (TCP-In)"}}**
		- Kerberos authentication on cluster (80): **Get-ClusterNode | ForEach-Object {Invoke-command -computername \$\_.name -scriptblock {Enable-Netfirewallrule -displayname "Hyper-V Replica HTTP Listener (TCP-In)"}}**
		- Certificate authentication on standalone server: **Enable-Netfirewallrule -displayname "Hyper-V Replica HTTPS Listener (TCP-In)"**
		- Kerberos authentication on standalone server: **Enable-Netfirewallrule -displayname "Hyper-V Replica HTTP Listener (TCP-In)"**

## 步骤 3：运行容量规划器工具

在准备好主站点并设置恢复服务器之后，可以运行该工具。

1. 从 Microsoft 下载中心[下载](https://www.microsoft.com/en-us/download/details.aspx?id=39057)该工具。
2. 从某个主服务器（或主群集中的某个节点）运行该工具。右键单击 .exe 文件，然后选择“以管理员身份运行”。
3. 在“开始之前”中指定收集数据的时间长度。建议在生产期间运行该工具，以确保数据具有代表性。如果你只想要验证网络连接，可以只收集一分钟的数据。

	![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image2.png)

4. 在“主站点详细信息”中，针对独立主机指定服务器名称或 FQDN，或针对群集指定客户端接受点的 FQDN、群集名称或群集中的任何节点，然后单击“下一步”。该工具将自动检测它所在的服务器的名称。该工具将选择你可以在其中监视指定服务器的 VM。

	![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image3.png)

5. 在“副本站点详细信息”中，如果你要复制到 Azure 或辅助数据中心且尚未设置副本服务器，请选择“跳过涉及副本站点的测试”。如果要复制到辅助数据中心并且已设置副本，请在“服务器名称(或) Hyper-V 副本代理 CAP”中键入独立服务器的 FQDN，或群集的客户端接入点。

	![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image4.png)

6. 在“扩展副本详细信息”中启用“跳过涉及扩展副本站点的测试”。这些测试不受站点恢复的支持。
7. 在“选择要复制的 VM”中，工具将根据在“主站点详细信息”页上指定的设置连接到服务器或群集，并显示主服务器上运行的 VM 和磁盘。请注意，将不会显示已针对复制启用的或者尚未运行的 VM。选择你要收集其度量值的 VM。选择 VHD 也会自动收集 VM 的数据。
9. 如果已配置副本服务器或群集，请在“网络信息”中指定你要用于主站点和副本站点之间的近似 WAN 带宽；如果已配置证书身份验证，请选择证书。

	![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image5.png)

10. 在“摘要”中检查设置，然后单击“下一步”开始收集度量值。工具的进度和状态将显示在“计算容量”页上。该工具完成运行时，请单击“查看报告”以查看输出。默认情况下，报告和日志存储在 **%systemdrive%\\Users\\Public\\Documents\\Capacity Planner** 中。

	![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image6.png)


## 步骤 4：解释结果
以下是重要度量值。你可以忽略未列在此处的度量值。它们与站点恢复无关。

### 本地到本地复制
  - 复制对主要主机的计算资源和内存的影响
  - 复制对主要主机和恢复主机的存储磁盘空间、IOPS 的影响
  - 增量复制所需的总带宽 (Mbps)
  - 在主要主机和恢复主机之间观测到的网络带宽 (Mbps)
  - 两个主机/群集之间活动并行传输的理想数目建议

### 本地到 Azure 复制
  - 复制对主要主机的计算资源和内存的影响
  - 复制对主要主机的存储磁盘空间、IOPS 的影响
  - 增量复制所需的总带宽 (Mbps)

## 更多资源

- 有关该工具的详细信息，请阅读随工具下载的文档。
- 观看 Keith Mayer 的 [TechNet 博客](http://blogs.technet.com/b/keithmayer/archive/2014/02/27/guided-hands-on-lab-capacity-planner-for-windows-server-2012-hyper-v-replica.aspx)中的工具演练。
- 获取本地到本地 Hyper-V 复制的性能测试[结果](/documentation/articles/site-recovery-performance-and-scaling-testing-on-premises-to-on-premises/)



## 后续步骤

你已完成容量计划，可以开始部署站点恢复了：

- [设置本地 VMM 站点与 Azure 之间的保护](/documentation/articles/site-recovery-vmm-to-azure/)
- [在本地 Hyper-V 站点与 Azure 之间设置保护](/documentation/articles/site-recovery-hyper-v-site-to-azure/)
- [设置两个本地 VMM 站点之间的保护](/documentation/articles/site-recovery-vmm-to-vmm/)
- [使用 SAN 在两个本地 VMM 站点之间设置保护](/documentation/articles/site-recovery-vmm-san/)
- [使用单个 VMM 服务器设置保护](/documentation/articles/site-recovery-single-vmm/)

<!---HONumber=Mooncake_0328_2016-->