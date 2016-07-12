<properties
	pageTitle="发生影响 Azure 虚拟机的 Azure 服务中断事件时该怎么办 | Azure"
	description="了解发生影响 Azure 虚拟机的 Azure 服务中断事件时该怎么办。"
	services="virtual-machines"
	documentationCenter=""
	authors="kmouss"
	manager="drewm"
	editor=""/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="05/16/2016"
	wacn.date="07/11/2016"/>

#发生影响 Azure 虚拟机的 Azure 服务中断事件时该怎么办

Azure.cn 的同仁兢兢业业，只为确保在任何时候都能提供你需要的服务。但有时候会因为不可抗力的影响，造成服务意外中断。

Azure.cn 为其服务提供服务级别协议 (SLA)，作为运行时间和连接承诺。你可以在 [Azure 服务级别协议](/support/legal/sla/)中找到各种 Azure 服务的 SLA。

Azure 已在平台中内置多种功能，用于支持高度可用的应用程序。

本文介绍了当整个区域因重大自然灾难或大规模服务中断而发生中断时的真实灾难恢复方案。这些都是极其罕见的情况，但你还是必须对整个区域发生中断的可能性有所准备。如果整个区域的服务中断，会暂时无法使用数据的本地冗余副本。如果启用了异地复制，则会在其他区域额外存储 Azure 存储空间 blob 和表的三个副本。如果发生全面性区域中断或发生主要区域无法恢复的灾难，Azure 会将所有 DNS 条目重新映射到异地复制区域。

>[AZURE.NOTE]注意，你对此过程无任何控制权，并且此过程仅适用于区域范围的服务中断。因此，还必须依靠应用程序特有的其他备份方法才能达到最高级别的可用性。

为帮助你处理这些罕见事件，我们提供以下 Azure 虚拟机指导，以应对 Azure 虚拟机应用程序部署所在的整个区域发生服务中断的情况。

##选项 1：等待恢复
在此情况下，你不需要采取任何操作。但你要知道，我们正在努力还原服务的可用性。

>[AZURE.NOTE]如果发生中断前尚未设置 Azure Site Recovery、虚拟机备份、读取访问权限异地冗余存储或异地冗余存储，这便是你的最佳选项。如果存储 VM 虚拟硬盘 (VHD) 的存储帐户已设置异地冗余存储或读取访问权限异地冗余存储，你可以指望恢复基本映像 VHD，并尝试用它预配新的 VM。因为除非使用 Azure VM 备份或 Azure Site Recovery，否则不保证能够同步处理数据，所以这不是首选的选项。因此，不保证此选项可行。

想要立即访问虚拟机的客户有以下两个选项可用。

>[AZURE.NOTE]注意，以下两个选项都有可能丢失部分数据。

##选项 2：从备份还原 VM
对于已配置 VM 备份的客户，你可以从备份和恢复点来还原 VM。

若要从 Azure 备份还原新的 VM，请参阅[还原 Azure 中的虚拟机](/documentation/articles/backup-azure-restore-vms/)。

请参阅[在 Azure 中计划 VM 备份基础结构](/documentation/articles/backup-azure-vms-introduction/)，帮助计划你的 Azure 虚拟机备份基础结构。

##选项 3：使用 Azure Site Recovery 启动故障转移
如果已将 Azure Site Recovery 配置为用于受影响的 Azure 虚拟机，则可以从副本还原 VM。这些副本可以位于 Azure 上，也可以位于本地。如此，你就可以用现有副本来创建新的 VM。若要从 Azure Site Recovery 副本还原 VM，请参阅[使用 Azure Site Recovery 在 Azure 区域之间迁移 Azure IaaS 虚拟机](/documentation/articles/site-recovery-migrate-azure-to-azure/)。

>[AZURE.NOTE]虽然 Azure 虚拟机的操作系统和数据磁盘会复制到次要 VHD，但如果它们位于异地冗余存储或读取访问权限异地冗余存储帐户中，则会各自复制每个 VHD。此级别的复制不保证已复制 VHD 的一致性。如果使用这些数据磁盘的应用程序和/或数据库彼此有依赖关系，则不保证所有 VHD 复制为一个快照。也不保证异地冗余存储或读取访问权限异地冗余存储中的 VHD 副本能生成应用程序一致的快照来引导 VM。

##后续步骤
若要了解如何备份 VM，请参阅[备份 Azure 虚拟机](/documentation/articles/backup-azure-vms/)。

若要了解如何使用 Azure Site Recovery 来安排和自动保护 VMWare 和 Hyper-V VM 上运行的物理（和虚拟）Windows 和 Linux 计算机，请参阅 [Azure Site Recovery](/documentation/services/site-recovery/)。

如果指示不清楚，或者如果希望 Azure.cn 代你执行操作，请联系[客户支持](/support/support-azure/)。

<!---HONumber=Mooncake_0704_2016-->