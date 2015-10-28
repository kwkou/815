<properties
	pageTitle="Hyper-V 复制的容量规划"
	description="本文包含有关如何使用 Azure Site Recovery 容量规划器工具的说明，并提供了用于 Hyper-V 复制容量规划的资源"
	services="site-recovery"
	documentationCenter="na"
	authors="csilauraa"
	manager="jwhit"
	editor="tysonn"/>
<tags
	ms.service="site-recovery"
	ms.date="06/01/2015"
	wacn.date="08/29/2015"/>

# Hyper-V 复制的容量规划

Azure Site Recovery 使用 Hyper-V 副本在两个本地 VMM 站点之间或者在本地 VMM 服务器与 Azure 存储空间之间进行复制。本文包含有关使用 Azure Site Recovery (ASR) 的容量规划器 - Hyper-V 副本工具的分步说明。ASR 的容量规划器 - Hyper-V 副本工具可指导 IT 管理员设计出用于成功部署 Hyper-V 副本以及验证两个站点之间的网络连接所需的服务器、存储和网络基础结构。

## 系统要求
- 操作系统：Windows Server® 2012 或 Windows Server® 2012 R2
- 内存：20 MB（最低）
- CPU：5% 开销（最低）
- 磁盘空间：5 MB（最低）

## 教程步骤
- 步骤 1：准备主站点
- 步骤 2：准备恢复站点（如果恢复站点在本地）
- 步骤 3：运行容量规划器工具
- 步骤 4：解释结果

## 步骤 1：准备主站点
1. 创建需要为复制启用的所有 Hyper-V 虚拟机和对应主要 Hyper-V 主机/群集的列表。
2. 将主要 Hyper-V 主机和群集分组到下列其中一项：

  	- Windows Server® 2012 独立服务器
  	- Windows Server® 2012 群集
  	- Windows Server® 2012 R2 独立服务器
  	- Windows Server® 2012 R2 群集

3. 需要对每个独立服务器组和每个群集运行一次容量规划器。
4. 在所有主要主机和群集上启用 WMI 远程访问。确保已正确设置防火墙规则集和用户权限。

        netsh firewall set service RemoteAdmin enable

5. 在主要主机上启用性能监视。

  	- 打开“Windows 防火墙”下的“高级安全”管理单元，然后启用以下入站规则：
    	- COM+ 网络访问 (DCOM-IN)
    	- 远程事件日志管理组中的所有规则

## 步骤 2：准备恢复站点
如果您使用 Azure 作为恢复站点，或尚未准备好您的本地恢复站点，则可以跳过本部分。但如果跳过，您将无法测量两个站点之间的可用带宽。

1. 识别身份验证方法

	a.Kerberos：当主要和恢复 Hyper-V 主机位于相同域中或位于互相信任的域中时使用。

	b.证书：当主要和恢复 Hyper-V 主机位于不同的域中时使用。可以使用 makecert 创建证书。有关使用此方法部署证书所要执行的步骤的信息，请参阅[基于 Hyper-V 副本证书的身份验证 - makecert](http://blogs.technet.com/b/virtualization/archive/2013/04/13/hyper-v-replica-certificate-based-authentication-makecert.aspx) 博客文章。

2. 识别恢复站点中的“单个”恢复 Hyper-V 主机/群集。

	a.此恢复主机/群集将用于复制虚构的虚拟机，以及估计主要和辅助站点之间的可用带宽。

	b.**建议**：使用单个恢复 Hyper-V 主机来运行测试。

### 准备一台 Hyper-V 主机作为恢复服务器
1. 在 Hyper-V 管理器的“操作”窗格中，单击“Hyper-V 设置”。
2. 在“Hyper-V 设置”对话框中，单击“复制配置”。
3. 在“详细信息”窗格中，选择“启用这台计算机作为副本服务器”。
4. 在“身份验证和端口”部分中，选择前面选择的身份验证方法。对于任一身份验证方法，指定要使用的端口（对于通过 HTTP 进行的 kerberos 身份验证，默认端口为 80；对于通过 HTTPS 进行的基于证书的身份验证，默认端口为 443）。
5. 如果您使用基于证书的身份验证，请单击“选择证书”，然后提供证书信息。
6. 在“授权和存储”部分中，指定允许“任何”已验证（主要）服务器将复制数据发送到这个副本服务器。
7. 单击“确定”或“应用”。

	![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image1.png)

8. 运行以下命令，验证 https 侦听器是否正在运行：
        netsh http show servicestate
9. 打开防火墙端口：

        Port 443 (certificae-based authentication):
          Enable-Netfirewallrule -displayname "Hyper-V Replica HTTPS Listener (TCP-In)"


        Port 80 (Kerberos):
          Enable-Netfirewallrule -displayname "Hyper-V Replica HTTP Listener (TCP-In)"

### 准备一个 Hyper-V 群集作为恢复目标
如果您已准备了独立 Hyper-V 主机恢复服务器，则可以跳过本部分。

1. 配置 Hyper-V 副本代理：

	a.在“服务器管理器”中，打开“故障转移群集管理器”。

	b.在左窗格中，连接到群集，然后在群集名称突出显示时，单击“操作”窗格中的“配置角色”。此时将打开“高可用性向导”。

	c.在“选择角色”屏幕中，选择“Hyper-V 副本代理”。

	d.完成向导，提供“NetBIOS 名称”和“IP 地址”作为群集的连接点（称为客户端接入点）。“Hyper-V 副本代理”已配置，并生成了一个客户端接入点名称。请记下客户端接入点名称 - 稍后要用它来配置副本。

	e.验证 Hyper-V 副本代理角色是否已成功联机，并可以在群集的所有节点之间进行故障转移。为此，请右键单击该角色，指向“移动”，然后单击“选择节点”。选择节点 >“确定”。

	f.如果您使用基于证书的身份验证，请确保每个群集节点与 Hyper-V 副本代理的客户端接入点上都安装了该证书。

2. 配置副本设置：

	a.在“服务器管理器”中，打开“故障转移群集管理器”。
	
	b.在左窗格中，连接到群集，并在群集名称突出显示时，单击“详细信息”窗格的“导航”类别中的“角色”。
	
	c.右键单击该角色，然后选择“复制设置”。
	
	d.在“详细信息”窗格中，选择“启用这台计算机作为副本服务器”。

	e.在“身份验证和端口”部分中，选择前面选择的身份验证方法。对于任一身份验证方法，指定要使用的端口（对于通过 HTTP 进行的 Kerberos 身份验证，默认端口为 80；对于通过 HTTPS 进行的基于证书的身份验证，默认端口为 443）。

	f.如果您使用基于证书的身份验证，请单击“选择证书”，然后提供请求的证书信息。

	g.在“授权和存储”部分中，指定是要允许“任何”已验证（主要）服务器将复制数据发送到这个副本服务器，还是只允许来自特定主服务器的数据。您可以使用通配符来限制接受特定域中的服务器，而无需单独指定每个服务器（例如 *.contoso.com）。

	h.在所有恢复 Hyper-V 主机上打开防火墙端口：
          
          	Port 443 (Certificate auth):
             Get-ClusterNode | ForEach-Object {Invoke-command -computername \$\_.name -scriptblock {Enable-Netfirewallrule -displayname "Hyper-V Replica HTTPS Listener (TCP-In)"}}


          	Port 80 (Kerberos auth):
              Get-ClusterNode | ForEach-Object {Invoke-command -computername \$\_.name -scriptblock {Enable-Netfirewallrule -displayname "Hyper-V Replica HTTP Listener (TCP-In)"}}


## 步骤 3：运行容量规划器工具
1. 下载容量规划器工具。
2. 从某个主服务器（或主群集中的某个节点）运行该工具。右键单击 .exe 文件，然后选择“以管理员身份运行”。
3. 如果您选择该选项，请接受“许可条款”，然后单击“下一步”。
4. 选择“度量值收集持续时间”。强烈建议在生产期间运行该工具，以确保收集最有代表性的数据。度量值收集的建议持续时间为 30 分钟。如果您只想要验证网络连接，可以选择一分钟作为持续时间。

	![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image2.png)

5. 如下所示指定“主站点详细信息”，然后单击“下一步”。

	对于独立主机，请输入服务器名称或 FQDN。

	如果您的主要主机是群集的一部分，可以输入以下其中一项的 FQDN：

	a.Hyper-V 副本代理客户端接入点 (CAP)

	b.群集名称

	c.群集的任何节点

  	![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image3.png)


6. 输入“副本站点详细信息”（仅限本地站点到本地站点的复制）

  如果您想要启用 Azure 的复制或尚未准备 Hyper-V 主机或群集作为恢复服务器（如步骤 2 所述），则应该跳过涉及副本站点的测试。

  指定“副本站点”详细信息，然后单击“下一步”。


i.对于独立主机，请输入服务器名称或 FQDN

ii.如果您的主要主机是群集的一部分，可以输入以下其中一项的 FQDN：

a.Hyper-V 副本代理客户端接入点 (CAP)

b.群集名称

c.群集的任何节点  
![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image4.png)

7\.跳过涉及“扩展的副本站点”的测试。ASR 不支持这些测试。

8\.选择要分析的虚拟机。该工具将连接到“主站点详细信息”中指定的群集或独立服务器，并枚举正在运行的虚拟机。选择您要收集其度量值的虚拟机和虚拟磁盘。

将不会枚举或显示以下虚拟机：

-	已针对复制启用的虚拟机
-	未运行的虚拟机  

9\.输入“网络信息”（这仅适用于本地站点到本地站点复制并提供了副本站点详细信息的情况）。

指定请求的网络信息，然后单击“下一步”。

- 估计的 WAN 带宽
- 要用于身份验证的证书（可选）：如果您打算使用基于证书的身份验证，则应该在此页中提供所需的证书。

   ![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image5.png)

10\.在下一组屏幕上，单击“下一步”启动该工具。

![](./media/site-recovery-capacity-planning-for-hyper-v-replication/image6.png)

11\.该工具完成运行时，请单击“查看报告”以查看输出。

    Default report location:

    %systemdrive%\Users\Public\Documents\Capacity Planner

    Logs location:

    %systemdrive%\Users\Public\Documents\CapacityPlanner

## 步骤 4：解释结果
您可以忽略以下两种方案之一下未列出的度量值，因为这些度量值与此方案不相关。

### 本地站点到本地站点的复制
  - 复制对主要主机的计算资源和内存的影响
  - 复制对主要主机和恢复主机的存储磁盘空间、IOPS 的影响
  - 增量复制所需的总带宽 (Mbps)
  - 在主要主机和恢复主机之间观测到的网络带宽 (Mbps)
  - 两个主机/群集之间活动并行传输的理想数目建议

### 本地站点到 Azure 的复制
  - 复制对主要主机的计算资源和内存的影响
  - 复制对主要主机的存储磁盘空间、IOPS 的影响
  - 增量复制所需的总带宽 (Mbps)

在 [Hyper-V 副本容量规划器下载页](http://go.microsoft.com/?linkid=9876170)上可以找到更详细的指导。

## 其他资源
以下资源可帮助您完成 Hyper-V 复制的容量规划：

- [更新：Hyper-V 副本容量规划器](http://go.microsoft.com/fwlink/?LinkId=510891)—阅读此博客文章以了解这款新工具的概述。

- [Hyper-V 副本容量规划器](http://go.microsoft.com/fwlink/?LinkId=510892)—下载此工具的最新版本。

- [引导式动手实验](http://go.microsoft.com/fwlink/?LinkId=510893)—使用 Keith Mayer 的 TechNet 博客上所述的工具，完成有关容量规划的极佳演练。

- [性能和缩放测试：本地到本地](/documentation/articles/site-recovery-performance-and-scaling-testing-on-premises-to-on-premises)—阅读对本地到本地部署进行复制测试的结果。


## 后续步骤

开始部署 ASR：

- [设置本地 VMM 站点与 Azure 之间的保护](/documentation/articles/site-recovery-vmm-to-azure)
- [设置本地 Hyper-V 站点与 Azure 之间的保护](/documentation/articles/site-recovery-hyper-v-site-to-azure)
- [设置两个本地 VMM 站点之间的保护](/documentation/articles/site-recovery-vmm-to-vmm)
- [使用 SAN 在两个本地 VMM 站点之间设置保护](/documentation/articles/site-recovery-vmm-san)
- [使用单个 VMM 服务器设置保护](/documentation/articles/site-recovery-single-vmm)
 

<!---HONumber=67-->