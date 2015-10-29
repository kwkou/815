<properties
	pageTitle="将 Windows 虚拟机从 Amazon Web Services 迁移到 Windows Azure"
	description="使用 Azure Site Recovery 将运行在 Amazon Web Services (AWS) 中的 Windows 虚拟机迁移到 Azure。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.date="08/26/2015"
	wacn.date="10/22/2015"/>

#  将 Amazon Web Services (AWS) 中的 Windows 虚拟机迁移到 Azure


## 概述

Azure Site Recovery 可在许多部署中安排虚拟机的复制、故障转移和恢复，为业务连续性和灾难恢复 (BCDR) 策略发挥作用。有关部署方案的完整列表，请参阅 [Azure Site Recovery 概述](/documentation/articles/site-recovery-overview)。

本文介绍如何使用 Site Recovery 将在 AWS 中运行的 Windows 实例迁移到 Azure，或进行故障转移。本文采用[在本地 VMware 虚拟机或物理服务器与 Azure 之间设置保护](/documentation/articles/site-recovery-vmware-to-azure)中描述的大多数步骤。我们建议你通读该文以获取有关部署中每一步骤的详细说明。

## 入门

以下是开始之前需要满足的条件：

- **配置服务器**：充当配置服务器的 Azure 虚拟机。配置服务器协调本地计算机和 Azure 服务器之间的通信。
- **主目标服务器**：充当主目标服务器的 Azure 虚拟机。此服务器接收并保留来自受保护计算机的复制数据。
- **进程服务器**：运行 Windows Server 2012 R2 的虚拟机。受保护的虚拟机向此服务器发送复制数据。
- **EC2 VM 实例**：你想要迁移然后再保护的实例。

- 在[我需要做什么？](/documentation/articles/site-recovery-vmware-to-azure#what-do-i-need)中了解有关这些组件的更多信息
- 你还应阅读[容量规划](/documentation/articles/site-recovery-vmware-to-azure#capacity-planning)准则并且确保在你开始之前满足所有[部署先决条件](/documentation/articles/site-recovery-vmware-to-azure#before-you-start)。

## 部署步骤

1. [创建保管库](/documentation/articles/site-recovery-vmware-to-azure#step-1-create-a-vault)
2. 作为 Azure 虚拟机[部署配置服务器](/documentation/articles/site-recovery-vmware-to-azure#step-2-deploy-a-configuration-server)。
3. 作为 Azure 虚拟机[部署主目标服务器](/documentation/articles/site-recovery-vmware-to-azure#step-2-deploy-a-configuration-server)。
4. [部署进程服务器](/documentation/articles/site-recovery-vmware-to-azure#step-4-deploy-the-on-premises-process-server)。请注意：

	- 你应将进程服务器部署在与 EC2 实例相同的子网/Amazon 虚拟私有云中。
		![EC2 实例](./media/site-recovery-migrate-aws-to-azure/ASR_AWSMigration1.png)

	- 在部署进程服务器之后，验证它能与你要迁移的 EC2 实例通信。
	- 你要保护的每个虚拟机需要安装移动服务。此服务将数据发送到进程服务器。可以手动安装移动服务，也可以在启用了虚拟机保护后由进程服务器自动推送并安装。要迁移的 EC2 实例上的防火墙规则应配置为允许此服务的推送安装。EC2 实例的安全组应具有以下规则：

		![防火墙规则](./media/site-recovery-migrate-aws-to-azure/ASR_AWSMigration2.png)

	- 在进程服务器部署完毕并在 Site Recovery 保管库中注册到配置服务器之后，它应该出现在 Site Recovery 控制台中的**配置服务器**选项卡下。此步骤可能最多需要 15 分钟。
	
		![进程服务器](./media/site-recovery-migrate-aws-to-azure/ASR_AWSMigration3.png)

5. [安装最新更新](/documentation/articles/site-recovery-vmware-to-azure#step-5-install-latest-updates)。确保已安装的所有组件服务器都是最新的。
6. [创建保护组](/documentation/articles/site-recovery-vmware-to-azure#step-7-create-a-protection-group)。要开始使用 Site Recovery 保护迁移的实例，你需要设置一个保护组。你指定保护组的复制设置，这些设置将被应用到添加到该组的所有实例。 
7. [设置虚拟机](/documentation/articles/site-recovery-vmware-to-azure#step-8-set-up-machines-you-want-to-protect)。你需要在每个实例上安装移动服务（自动或手动皆可）。
8. [为虚拟机启用保护](/documentation/articles/site-recovery-vmware-to-azure#step-9-enable-protection)。通过将实例添加到保护组来为这些实例启用保护。请注意：

	- 可以使用实例的专用 IP 地址发现你要迁移到 Azure 的 EC2 实例，该地址可以在 EC2 控制台中获取。
	-  在你创建的保护组的选项卡上，单击“添加计算机”>“物理计算机”
		![EC2 发现](./media/site-recovery-migrate-aws-to-azure/ASR_AWSMigration4.png)
	- 指定实例的专用 IP 地址。
		- ![EC2 发现](./media/site-recovery-migrate-aws-to-azure/ASR_AWSMigration5.png)
	- 系统将启用保护，并且依据保护组的初始复制设置运行初始复制
9. [运行非计划的故障转移](/documentation/articles/site-recovery-failover#run-an-unplanned-failover)。在初始复制完成后，可以运行非计划的从 AWS 到 Azure 的故障转移。（可选）你可以创建一个恢复计划并运行非计划的故障转移，从 AWS 向 Azure 迁移多个虚拟机。[详细了解](/documentation/articles/site-recovery-create-recovery-plans)恢复计划。
		
## 后续步骤

在 [Site Recovery 论坛](https://social.msdn.microsoft.com/forums/zh-cn/home?forum=hypervrecovmgr)中发布任何评论或问题

<!---HONumber=74-->