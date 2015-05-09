<properties 
	pageTitle="使用"Azure 备份"来备份 Azure 虚拟机" 
	description="使用此演练部署"Azure 备份"来备份虚拟机。" 
	services="backup" 
	documentationCenter="" 
	authors="raynew" 
	manager="jwhit" 
	editor="tysonn"/>

<tags ms.service="backup" ms.date="03/26/2015" wacn.date="04/11/2015"/>

# 使用"Azure 备份"来备份 Azure 虚拟机

使用"Azure 备份"服务保护你的本地服务器和 Azure 虚拟机上的数据免遭损失。此演练介绍了如何使用"Azure 备份"来备份你的 Azure 虚拟机。

## 概述

使用"Azure 备份"来备份 Azure 虚拟机上的数据提供了许多业务优势：

- "Azure 备份"可以自动执行虚拟机的备份和恢复
- 它创建与应用程序一致的备份来确保恢复的数据从一致的状态开始。
- 在虚拟机备份期间不需要停机。
- 你可以备份运行 Windows 或 Linux 的 Azure 虚拟机
- 恢复点可用来在"Azure 备份"保管库中轻松进行恢复。


## 准备工作

你将需要一个 Azure 帐户。如果没有，请使用 [1rmb 试用版](/pricing/1rmb-trial/)。你还可以阅读[关于 Azure 备份定价](/home/features/back-up/#home_back_pri)。


## 创建保管库

1. 登录到[管理门户](http://go.microsoft.com/fwlink/?LinkId=301902)。
2. 单击"恢复服务">"新建">"数据服务">"恢复服务">"备份保管库">"快速创建"。如果有多个订阅与你的组织帐户相关联，请选择要与备份保管库关联的正确订阅。
3. 在"名称"中，输入一个友好名称以标识此保管库。
4. 在"区域"中，为保管库选择地理区域。请注意，保管库应当与你要保护的虚拟机位于同一区域中。如果你的虚拟机位于不同的区域中，请在每个区域中创建一个保管库。

	![Create vault](./media/backup-azure-vms/Backup_VaultCreate.png)

3. 单击"创建保管库"。

 	创建备份保管库可能需要一段时间。可以在门户底部监视状态通知。将出现一条消息来确认保管库已成功创建，并且将在"恢复服务"页上将保管库列出为"活动"保管库。 

	![Vault list](./media/backup-azure-vms/Backup_VaultsList.png)

## 注册虚拟机

1. 在主"恢复服务"页上，双击保管库以将其打开。
2. 在"快速启动"页上，单击"保护 Azure 虚拟机">"搜索虚拟主机"。如果希望了解有关创建 Azure 虚拟机的更多信息，请参阅：
	- [创建运行 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-tutorial)
	- [创建运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-tutorial)

	![Register virtual machines](./media/backup-azure-vms/Backup_QuickStart.png)

3. 单击"注册的项">"注册">"注册的项"。选择虚拟机并单击复选标记以注册虚拟机。注册过程将在虚拟机上部署"恢复服务"扩展并启用备份。此扩展使得"备份"能够通过卷影复制服务 (VSS) 创建与应用程序一致的备份，且不需要关闭虚拟机。在注册完成后，虚拟机将出现在页面上且其状态为"已注册"。

	![Register virtual machines](./media/backup-azure-vms/Backup_RegisteredVM.png)

## 保护虚拟机

1. 在"注册的项"选项卡上，选择虚拟机名称 >"保护"以打开"保护项"向导。
2. 在"选择要保护的项"页上，选中你要为其启用保护的虚拟机。 

	![Select virtual machines](./media/backup-azure-vms/Backup_ProtectWizard1.png)	
 
3. 在"配置保护"页上，你将选择一个要应用于虚拟机的保护策略。策略定义了备份计划和备份在"Azure 备份"中保留的时长。如果要定义新策略，请指定名称、虚拟机的备份频率，以及备份的虚拟机数据应当在"Azure 备份"中保留的时长。最多可以将虚拟机保留 30 天。

	![Configure protection](./media/backup-azure-vms/Backup_ProtectWizard2.png)

4. 在配置保护后，虚拟机将出现在"受保护的项"选项卡上。它将显示为"受保护(初始备份未完成)"状态，直到初始备份完成，然后它将显示为"受保护"状态。

	![Protection status](./media/backup-azure-vms/Backup_ProtectionPending.png)
 
## 运行备份

在初始复制完成后，虚拟机将根据策略计划进行备份，你也可以单击"立即备份"来运行手动备份。
请注意，"立即备份"使用应用于虚拟机备份的保留策略并创建一个恢复点。
 
 
 
## 运行恢复

1. 若要从备份恢复虚拟机，请在"受保护的项"页上，单击"恢复"以打开"恢复项"向导。
2. 在"选择恢复点"页上，你可以从最新的恢复点进行恢复，或者从以前的某个时间点进行恢复。可用的恢复点突出显示在日历上。

	![Select recovery point](./media/backup-azure-vms/Backup_Recovery1.png)

3.  在"选择恢复实例"页上，指定你要将虚拟机恢复到何处。你需要恢复到一个替换位置。指定替换虚拟机名称以及现有的或新的云服务。根据需要指定目标网络和子网。 

	![Restore virtual machine](./media/backup-azure-vms/Backup_Recovery2.png)

在恢复后，你将需要在 Azure 门户中为虚拟机重新配置扩展并重新创建终结点。 
 
## 管理受保护的虚拟机

1. 若要查看和管理某个虚拟机的备份设置，请在"受保护的项"页上单击该虚拟机。

	- "备份详细信息"选项卡将显示有关最后一个备份的信息。

		![Virtual machine backup](./media/backup-azure-vms/Backup_VMDetails.png)	

	- "备份策略"选项卡将显示现有策略。你可以根据需要进行更改。如果需要创建新策略，请在"策略"页上单击"创建"。请注意，如果要删除某个策略，则它不应当具有与之关联的任何虚拟机。

		![Virtual machine policy](./media/backup-azure-vms/Backup_VMPolicy.png)

2. 可以在"作业"页上获取有关虚拟机的操作或状态的更多信息。单击列表中的某个作业可获取更多详细信息，还可以筛选特定虚拟机的作业。

	![Jobs](./media/backup-azure-vms/Backup_Jobs.png)

3. 在任何时刻，如果你希望停止保护某个虚拟机，请选择该虚拟机并在"受保护的项"页上单击"停止保护"。你可以指定是否为虚拟机删除当前位于"Azure 备份"中的备份，并且可以指定原因以备审核。虚拟机将以"保护已停止"状态显示。

	![Disable protection](./media/backup-azure-vms/Backup_DisableProtection.png)

 请注意，如果你在为虚拟机停止备份时没有选择删除备份，可以在"受保护的项"页上选择该虚拟机，然后单击"删除"。如果希望从备份保管库中删除虚拟机，请将其停止，然后单击"取消注册"以将其完全删除。 

### 仪表板

在"仪表板"页上，你可以查看过去 24 小时内关于 Azure 虚拟机、其存储以及与之关联的作业的信息。你可以查看备份状态和任何关联的备份错误。 






<!--HONumber=51-->