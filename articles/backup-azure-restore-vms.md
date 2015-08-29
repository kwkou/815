
<properties
	pageTitle="Azure 备份 - 恢复虚拟机"
	description="了解如何恢复 Azure 虚拟机"
	services="backup"
	documentationCenter=""
	authors="aashishr"
	manager="shreeshd"
	editor=""/>

<tags ms.service="backup" ms.date="04/30/2015" wacn.date="06/16/2015"/>  

# 恢复虚拟机


## 运行恢复
你可以使用恢复操作，通过存储在 Azure 备份保管库中的备份将虚拟机恢复为新的 VM。

1. 若要从备份恢复虚拟机，请在“受保护的项”页上，单击“恢复”以打开“恢复项”向导。  

	>[AZURE.NOTE]“受保护的项”页中的恢复点栏会显示你的虚拟机恢复点数目。最新恢复点栏会显示你可以将虚拟机恢复到的最新快照时间。

2. 在“选择恢复点”页上，你可以从最新的恢复点进行恢复，或者从以前的某个时间点进行恢复。可用的恢复点突出显示在日历上。

	![选择恢复点](./media/backup-azure-restore-vms/backup-recovery1.png)

3.  在“选择恢复实例”页上，指定你要将虚拟机恢复到何处。你需要恢复到一个替换位置。指定替换虚拟机名称以及现有的或新的云服务。根据需要指定目标网络和子网。

	![恢复虚拟机](./media/backup-azure-restore-vms/backup-recover2.png)

在恢复后，你将需要在 Azure 门户中为虚拟机重新配置扩展并重新创建终结点。
 
## 管理受保护的虚拟机

1. 若要查看和管理某个虚拟机的备份设置，请在“受保护的项”页上单击该虚拟机。

	- “备份详细信息”选项卡将显示有关最后一个备份的信息。

		![虚拟机备份](./media/backup-azure-restore-vms/backup-vmdetails.png)

	- “备份策略”选项卡将显示现有策略。你可以根据需要进行更改。如果需要创建新策略，请在“策略”页上单击“创建”。请注意，如果要删除某个策略，则它不应当具有与之关联的任何虚拟机。

		![虚拟机策略](./media/backup-azure-restore-vms/backup-vmpolicy.png)

2. 可以在“作业”页上获取有关虚拟机的操作或状态的更多信息。单击列表中的某个作业可获取更多详细信息，还可以筛选特定虚拟机的作业。

	![作业](./media/backup-azure-restore-vms/backup-job.png)

3. 在任何时刻，如果你希望停止保护某个虚拟机，请选择该虚拟机并在“受保护的项”页上单击“停止保护”。你可以指定是否为虚拟机删除当前位于“Azure 备份”中的备份，并且可以指定原因以备审核。虚拟机将以“保护已停止”状态显示。

	![禁用保护](./media/backup-azure-restore-vms/backup-disable-protection.png)

 请注意，如果你在为虚拟机停止备份时没有选择删除备份，可以在“受保护的项”页上选择该虚拟机，然后单击“删除”。如果希望从备份保管库中删除虚拟机，请将其停止，然后单击“取消注册”以将其完全删除。

###仪表板

在“仪表板”页上，你可以查看过去 24 小时内关于 Azure 虚拟机、其存储以及与之关联的作业的信息。你可以查看备份状态和任何关联的备份错误。

<!---HONumber=60-->