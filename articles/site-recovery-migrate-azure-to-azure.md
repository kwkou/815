<properties
	pageTitle="使用 Site Recovery 将 Azure IaaS 虚拟机从一个 Azure 区域迁移到另一个 Azure 区域 | Azure"
	description="使用 Azure Site Recovery 将 Azure IaaS 虚拟机从一个 Azure 区域迁移到另一个 Azure 区域。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="site-recovery"
	ms.date="03/16/2016"
	wacn.date="04/05/2016"/>

#  使用 Azure Site Recovery 在 Azure 区域之间迁移 Azure IaaS 虚拟机

## 概述

本文介绍如何使用 Site Recovery 在 Azure 区域之间迁移 Azure VM。开始之前，请注意：

- 目前只能迁移。这意味着，你可以将 VM 从一个 Azure 区域故障转移到另一个 Azure 区域，但不能重新对其进行故障回复。



请将任何评论或问题发布到本文底部，或者发布到 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)。


## 先决条件

以下是执行此部署所需的组件：

- **管理服务器**：运行 Windows Server 2012 R2 作为管理服务器的本地 VM。在此服务器上安装 Site Recovery 组件（包括配置服务器和进程服务器）。
- **IaaS 虚拟机**：你想要迁移的虚拟机。

## 部署步骤

1. 创建保管库。
2. 部署管理服务器。
3. 在部署管理服务器之后，验证它能与你要迁移的 VM 通信。
4. 创建保护组。保护组包含共享相同复制设置的受保护计算机。指定组的复制设置，这些设置将被应用到添加到该组的所有计算机。
5. 安装移动服务。你要保护的每个虚拟机需要安装移动服务。此服务将数据发送到进程服务器。可以手动安装移动服务，也可以在启用了虚拟机保护后由进程服务器自动推送并安装。请注意，要迁移的 IaaS 虚拟机上的防火墙规则应配置为允许此服务的推送安装。
6. 为计算机启用保护。将想要保护的计算机添加到复制组中。 
7. 你可以使用虚拟机的 IP 地址发现要迁移到 Azure 的 IaaS 虚拟机。在 Azure 中的虚拟机仪表板上找到此地址。
8. 在你创建的保护组的选项卡上，单击“添加计算机”>“物理计算机”。

	![EC2 发现](./media/site-recovery-migrate-azure-to-azure/migrate-add-machines.png)

9. 指定虚拟机的专用 IP 地址。

	![EC2 发现](./media/site-recovery-migrate-azure-to-azure/migrate-machine-ip.png)
	
	向组添加计算机之后，系统将启用保护，并且依据保护组设置运行初始复制。

10. [运行非计划的故障转移](/documentation/articles/site-recovery-failover/#run-an-unplanned-failover)。在初始复制完成之后，可以运行非计划的从一个 Azure 区域到另一个 Azure 区域的故障转移。（可选）你可以创建一个恢复计划并运行非计划的故障转移，在区域之间迁移多个虚拟机。[详细了解](/documentation/articles/site-recovery-create-recovery-plans/)恢复计划。
		
## 后续步骤

若要详细了解其他复制方案，请参阅[什么是 Azure Site Recovery？](/documentation/articles/site-recovery-overview/)

<!---HONumber=Mooncake_0328_2016-->