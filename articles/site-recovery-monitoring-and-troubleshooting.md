<properties
	pageTitle="VMM 和 Hyper-V 站点保护的监视和故障排除指南" 
	description="Azure Site Recovery 可以协调位于本地服务器中的虚拟机到 Azure 或辅助数据中心的复制、故障转移和恢复。参考本文来监视 VMM 或 Hyper-V 站点保护并对其进行故障排除。" 
	services="site-recovery" 
	documentationCenter="" 
	authors="anbacker" 
	manager="mkjain" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.date="07/20/2015" 
	wacn.date="10/03/2015"/>
	
# 监视 VMware、VMM、Hyper-V 和物理站点的保护以及对其进行故障排除

你可以参考本监视与故障排除指南来了解如何跟踪 Azure 站点恢复的复制运行状况，并掌握相关的故障排除方法。

## 了解组件

### 用于在本地与 Azure 之间复制的 VMware/物理站点部署。
要在本地 VMware/物理计算机之间设置灾难恢复；需要配置配置服务器、主目标和进程服务器。在启用对源服务器的保护时，Azure 站点恢复将安装移动服务。在源服务器故障转移到 Azure 发生本地中断后，客户需要在 Azure 中设置进程服务器和在本地设置主目标服务器，以防止源服务器在本地重新生成。

![用于在本地与 Azure 之间复制的 VMware/物理站点部署](./media/site-recovery-monitoring-and-troubleshooting/image18.png)

### 用于在本地站点之间复制的 VMM 站点部署。

在两个本地位置之间设置 DR 的一部分操作；需要下载 Azure 站点恢复提供者并将其安装在 VMM 服务器上。提供者需要 Internet 连接，以确保从 Azure 门户触发的所有操作都将转换成本地操作，例如启用保护、在故障排除过程中关闭主端虚拟机，等等。

![用于在本地站点之间复制的 VMM 站点部署](./media/site-recovery-monitoring-and-troubleshooting/image1.png)

### 用于在本地与 Azure 之间复制的 VMM 站点部署。

在本地和 Azure 之间设置 DR 的一部分操作；下载下载 Azure 站点恢复提供者并将其安装在 VMM 服务器上，此外还需要在每个 Hyper-V 主机上安装 Azure 恢复服务代理。

![用于在本地与 Azure 之间复制的 VMM 站点部署](./media/site-recovery-monitoring-and-troubleshooting/image2.png)

### 用于在本地与 Azure 之间复制的 Hyper-V 站点部署

与 VMM 部署相同 – 不同的是提供者和代理都安装在 Hyper-V 主机本身上。

## 监视配置、保护和恢复操作

ASR 中的每个操作都被审核，可在“作业”选项卡下面跟踪。如果发生任何配置、保护或恢复错误，请导航到“作业”选项卡，并查看是否发生了任何失败。

![监视配置、保护和恢复操作](./media/site-recovery-monitoring-and-troubleshooting/image3.png)

如果在“作业”视图下发现失败，请选择该作业，然后单击该作业对应的“错误详细信息”。

![监视配置、保护和恢复操作](./media/site-recovery-monitoring-and-troubleshooting/image4.png)

错误详细信息可帮助你识别可能的原因，并获得解决问题的建议。

![监视配置、保护和恢复操作](./media/site-recovery-monitoring-and-troubleshooting/image5.png)

在上例中，似乎有另一个操作正在进行，正是该操作导致保护配置失败。确保根据建议解决问题 – 然后单击“重新启动”以重新启动该操作。

![监视配置、保护和恢复操作](./media/site-recovery-monitoring-and-troubleshooting/image6.png)

“重新启动”选项并非适用于所有操作 – 对于那些没有“重新启动”选项的操作，请导航回到对象，然后再次重做操作。随时都可以使用“取消”按钮来取消正在进行的每个作业。

![监视配置、保护和恢复操作](./media/site-recovery-monitoring-and-troubleshooting/image7.png)

## 监视虚拟机的复制运行状况

可以通过 Azure 门户针对每个受保护实体进行 ASR 提供者中心监视和远程监视。在选择“VMM 云”或“保护组”之后，导航到“受保护的项”。“VMM 云”选项卡只适用于基于 VMM 的部署，以及在“保护组”选项卡下具有受保护实体的所有其他案例。

![监视虚拟机的复制运行状况](./media/site-recovery-monitoring-and-troubleshooting/image8.png)

然后在相应云或保护组下选择受保护的实体。在你选择受保护的实体后，所有允许的操作将显示在底部窗格中。

![监视虚拟机的复制运行状况](./media/site-recovery-monitoring-and-troubleshooting/image9.png)

如上所示，如果虚拟机的运行状况为关键 – 你可以单击底部的“错误详细信息”按钮以查看错误。根据提到的“可能原因”和“建议”来解决问题 – 在本例中，虚拟机需要重新同步，这可以通过单击“重新同步”按钮在门户本身中完成。

![监视虚拟机的复制运行状况](./media/site-recovery-monitoring-and-troubleshooting/image10.png)

![监视虚拟机的复制运行状况](./media/site-recovery-monitoring-and-troubleshooting/image11.png)

注意：如果有任何活动的操作正在进行或失败，请如前所述导航到“作业”视图，以查看特定于作业的错误。

## 事件查看器

| 方案 | 事件源 |
|-------------------------	|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| VMM 站点保护 | VMM 服务器 <ul><li> **应用程序和服务日志/Microsoft/VirtualMachineManager/Server/Admin** </li></ul> Hyper-V 主机 <ul><li> **应用程序和服务日志/MicrosoftAzureRecoveryServices/Replication**（适用于 Azure 为目标的情况）</li><li> **应用程序和服务日志/Microsoft/Windows/Hyper-V-VMMS/Admin** </li></ul> |
| Hyper-V 站点保护 | <ul><li> **应用程序和服务日志/MicrosoftAzureRecoveryServices/Replication** </li><li> **应用程序和服务日志/Microsoft/Azure Site Recovery/Provider/Operational** </li><li> **应用程序和服务日志/Microsoft/Windows/Hyper-V-VMMS/Admin** </li><ul>|


## 排查本地 Hyper-V 问题

连接到本地 Hyper-V 管理器控制台，选择虚拟机，然后查看复制运行状况。

![排查本地 Hyper-V 问题](./media/site-recovery-monitoring-and-troubleshooting/image12.png)

在本例中，“复制运行状况”指示为“关键”– 请使用“查看复制运行状况”查看详细信息。

![排查本地 Hyper-V 问题](./media/site-recovery-monitoring-and-troubleshooting/image13.png)

### Hyper-V 复制日志记录选项

与 Hyper-V 复本相关的所有事件都会记录在位于“应用程序和服务日志\\Microsoft\\Windows”下的 Hyper-V-VMMS\\Admin 日志中。此外，可以针对 Hyper-V-VMMS 启用分析日志。若要启用此日志，请先在“事件查看器”中显示“分析”与“调试”日志。开启事件查看器，然后在“查看”菜单中，单击“显示分析和调试日志”。

![排查本地 Hyper-V 问题](./media/site-recovery-monitoring-and-troubleshooting/image14.png)

分析日志将显示在 Hyper-V-VMMS 下

![排查本地 Hyper-V 问题](./media/site-recovery-monitoring-and-troubleshooting/image15.png)

然后，在“操作”窗格中，单击“启用日志”。启用后，它将在“性能监视器”中显示为“数据收集器集”下的一个事件跟踪会话。

![排查本地 Hyper-V 问题](./media/site-recovery-monitoring-and-troubleshooting/image16.png)

若要查看收集的信息，请先通过禁用日志来停止跟踪会话，然后保存日志并在“事件查看器”中重新将它打开，或者根据需要使用其他工具转换它。


## 了解 Hyper-V 虚拟机保护和恢复生命周期

![了解 Hyper-V 虚拟机保护和恢复生命周期](./media/site-recovery-monitoring-and-troubleshooting/image17.png)


## 联系 Microsoft 技术

### 日志收集

对于 VMM 站点保护，请参阅[使用支持诊断平台 (SDP) 工具进行 ASR 日志收集](http://social.technet.microsoft.com/wiki/contents/articles/28198.asr-data-collection-and-analysis-using-the-vmm-support-diagnostics-platform-sdp-tool.aspx)，以收集所需的日志。

对于 Hyper-V 分支机构和 SMB 站点保护，请下载[工具](https://dcupload.microsoft.com/tools/win7files/DIAG_ASRHyperV_global.DiagCab)，然后在 Hyper-V 主机上执行该工具以收集日志。

对于 VMware/物理站点方案，请参阅[针对 VMware 和物理站点保护进行 Azure 站点恢复日志收集](http://social.technet.microsoft.com/wiki/contents/articles/30677.azure-site-recovery-log-collection-for-vmware-and-physical-site-protection.aspx)，以收集所需的日志。

SDP 工具将在本地收集日志文件，您也可以在“%LocalAppData%\\ElevatedDiagnostics”下随机命名的子文件夹下找到该文件

### 创建支持票证

若要针对 ASR 开具支持票证，请使用 URL <http://aka.ms/getazuresupport> 联系 Azure 支持

## 知识库文章

-   [如何为已故障转移或迁移到 Azure 的
    > 受保护虚拟机保留驱动器号](http://support.microsoft.com/kb/3031135)

-   [如何排查 Azure 恢复服务
    > 问题](http://support.microsoft.com/kb/3005185)

-   [如何在 Hyper-V 站点保护中为 Azure 站点恢复启用调试
    > 日志记录](http://support.microsoft.com/kb/3033922)

-   [ASR：尝试为虚拟机启用保护时发生
    > “找不到群集资源”错误](http://support.microsoft.com/kb/3010979)
    
-   [了解 Hyper-V 副本及其故障排除
    > 指南](http://www.microsoft.com/en-in/download/details.aspx?id=29016)
	
	
	
## 常见 ASR 错误及其解决方法

以下是您可能遇到的常见错误及其解决方法。每个错误记录在单独的 WIKI 网页中。

### 设置

-   [无法验证所选的证书。请选择其他证书。](http://social.technet.microsoft.com/wiki/contents/articles/25569.the-selected-certificate-cannot-be-validated-please-select-a-different-certificate.aspx)

-   [由于出现内部错误，无法注册 VMM 服务器。请参阅站点恢复门户中的作业视图，了解有关错误的更多详细信息。再次运行安装程序以注册该服务器。](http://social.technet.microsoft.com/wiki/contents/articles/25570.the-vmm-server-cannot-be-registered-due-to-an-internal-error-please-refer-to-the-jobs-view-in-the-site-recovery-portal-for-more-details-on-the-error-run-setup-again-to-register-the-server.aspx)

-   [无法建立到 Hyper-V Recovery Manager 保管库的连接。请验证代理设置或稍后再试。](http://social.technet.microsoft.com/wiki/contents/articles/25571.a-connection-cant-be-established-to-the-hyper-v-recovery-manager-vault-verify-the-proxy-settings-or-try-again-later.aspx)

### 配置

-   [Hyper-V 主机群集包含至少一个静态网络适配器，或者将没有连接的适配器配置为使用 DHCP。](http://social.technet.microsoft.com/wiki/contents/articles/25498.hyper-v-host-cluster-contains-at-least-one-static-network-adapter-or-no-connected-adapters-are-configured-to-use-dhcp.aspx)

-   [Hyper-V 配置文件未在云功能配置文件中启用](http://social.technet.microsoft.com/wiki/contents/articles/25499.the-hyper-v-profile-isn-t-enabled-in-the-capability-profiles-for-cloud.aspx)

-   [无法为 ‘%CloudName;’ 应用保护配置。无法配置新添加的 Hyper-V 主机或群集，因为未配置云保护。](http://social.technet.microsoft.com/wiki/contents/articles/25500.protection-configuration-for-cloudname-couldn-t-be-applied-a-newly-added-hyper-v-host-or-cluster-couldn-t-be-configured-because-cloud-protection-isn-t-configured.aspx)

-   [VMM 不具有完成某项操作的权限](http://social.technet.microsoft.com/wiki/contents/articles/31110.vmm-does-not-have-permissions-to-complete-an-action.aspx)

### 保护

-   [操作系统不受支持](http://social.technet.microsoft.com/wiki/contents/articles/31103.the-operating-system-is-not-supported.aspx)

-   [启用保护失败，因为主机上未安装代理](http://social.technet.microsoft.com/wiki/contents/articles/31105.enable-protection-failed-since-agent-not-installed-on-host-machine.aspx)

-   [找不到适合副本虚拟机的主机 - 因为计算资源较少](http://social.technet.microsoft.com/wiki/contents/articles/25501.a-suitable-host-for-the-replica-virtual-machine-can-t-be-found-due-to-low-compute-resources.aspx)

-   [找不到适合副本虚拟机的主机 - 因为没有连接逻辑网络](http://social.technet.microsoft.com/wiki/contents/articles/25502.a-suitable-host-for-the-replica-virtual-machine-can-t-be-found-due-to-no-logical-network-attached.aspx)

-   [无法连接到副本主机 - 不能建立连接](http://social.technet.microsoft.com/wiki/contents/articles/31106.cannot-connect-to-the-replica-host-machine-connection-could-not-be-established.aspx)

### 恢复

-   VMM 无法完成主机操作 -

    -   [故障转移到虚拟机的所选恢复点：常规拒绝访问错误。](http://social.technet.microsoft.com/wiki/contents/articles/25504.fail-over-to-the-selected-recovery-point-for-virtual-machine-general-access-denied-error.aspx)

    -   [Hyper-V 无法故障转移到虚拟机的所选恢复点：操作中止。请尝试使用较新的恢复点。(0x80004004)](http://social.technet.microsoft.com/wiki/contents/articles/25503.hyper-v-failed-to-fail-over-to-the-selected-recovery-point-for-virtual-machine-operation-aborted-try-a-more-recent-recovery-point-0x80004004.aspx)

    -   不能建立与服务器的连接 (0x00002EFD)

        -   [Hyper-v 无法启用虚拟机的反向复制](http://social.technet.microsoft.com/wiki/contents/articles/25505.a-connection-with-the-server-could-not-be-established-0x00002efd-hyper-v-failed-to-enable-reverse-replication-for-virtual-machine.aspx)

        -   [Hyper-v 无法启用虚拟机的复制](http://social.technet.microsoft.com/wiki/contents/articles/25506.a-connection-with-the-server-could-not-be-established-0x00002efd-hyper-v-failed-to-enable-replication-for-virtual-machine-virtual-machine.aspx)

    -   [无法提交虚拟机的故障转移](http://social.technet.microsoft.com/wiki/contents/articles/25508.could-not-commit-failover-for-virtual-machine.aspx)

-   [恢复计划包含的虚拟机未准备好进行计划的故障转移](http://social.technet.microsoft.com/wiki/contents/articles/25509.the-recovery-plan-contains-virtual-machines-which-are-not-ready-for-planned-failover.aspx)

-   [虚拟机未准备好进行计划的故障转移](http://social.technet.microsoft.com/wiki/contents/articles/25507.the-virtual-machine-isn-t-ready-for-planned-failover.aspx)

-   [虚拟机未运行且未关闭](http://social.technet.microsoft.com/wiki/contents/articles/25510.virtual-machine-is-not-running-and-is-not-powered-off.aspx)

-   [虚拟机发生带外操作且](http://social.technet.microsoft.com/wiki/contents/articles/25507.the-virtual-machine-isn-t-ready-for-planned-failover.aspx)提交故障转移失败

-   测试故障转移

    -   [无法启动故障转移，因为正在进行测试故障转移](http://social.technet.microsoft.com/wiki/contents/articles/31111.failover-could-not-be-initiated-since-test-failover-is-in-progress.aspx)

-   特定于恢复到 Azure

    -   [无法启动故障转移，因为虚拟机名称中有不受支持的字符](http://social.technet.microsoft.com/wiki/contents/articles/31107.failover-could-not-be-started-due-to-unsupported-characters-in-the-vm-name.aspx)

### 配置服务器、进程服务器、主目标

配置服务器 (CS)、进程服务器 (PS)、主目标 (MT)

-   [PS/CS 作为虚拟机托管所在的 ESXi 主机发生故障，出现死机紫屏。](http://social.technet.microsoft.com/wiki/contents/articles/31107.vmware-esxi-host-experiences-a-purple-screen-of-death.aspx)

<!---HONumber=71-->