
<properties
	pageTitle="Azure 备份 - 管理虚拟机"
	description="了解如何管理 Azure 虚拟机。"
	services="backup"
	documentationCenter=""
	authors="aashishr"
	manager="shreeshd"
	editor=""/>

<tags
	ms.service="backup" ms.date="07/17/2015" wacn.date="08/29/2015"/>

# 管理虚拟机


## 管理受保护的虚拟机

1. 若要查看和管理某个虚拟机的备份设置，请单击“受保护的项”选项卡。

  - 单击受保护项的名称可以查看“备份详细信息”选项卡，其中显示了有关上次备份的信息。

        ![Virtual machine backup](./media/backup-azure-manage-vms/backup-vmdetails.png)

2. 若要查看和管理某个虚拟机的备份策略设置，请单击“策略”选项卡。

  - “备份策略”选项卡将显示现有策略。你可以根据需要进行更改。如果需要创建新策略，请在“策略”页上单击“创建”。请注意，如果要删除某个策略，则它不应当具有与之关联的任何虚拟机。

        ![Virtual machine policy](./media/backup-azure-manage-vms/backup-vmpolicy.png)

3. 可以在“作业”页上获取有关虚拟机的操作或状态的更多信息。单击列表中的某个作业可获取更多详细信息，还可以筛选特定虚拟机的作业。

    ![作业](./media/backup-azure-manage-vms/backup-job.png)

## 虚拟机的按需备份
为虚拟机配置保护后，可以对它执行按需备份。如果虚拟机的初始备份已挂起，则按需备份将在 Azure 备份保管库中创建虚拟机的完整副本。如果已完成第一个备份，按需备份只会将以前备份的更改发送到 Azure 备份保管库。

若要执行虚拟机的按需备份，请执行以下操作：

1. 导航到“受保护的项”页，选择“Azure 虚拟机”作为“类型”（如果尚未选择），然后单击“选择”按钮。

    ![VM 类型](./media/backup-azure-manage-vms/vm-type.png)

2. 选择你要进行按需备份的虚拟机，然后单击页底部的“立即备份”按钮。

    ![立即备份](./media/backup-azure-manage-vms/backup-now.png)

    这会在选定的虚拟机上创建备份作业。通过此作业创建的恢复点保留范围与在虚拟机关联的策略中指定的保留范围相同。

    ![创建备份作业](./media/backup-azure-manage-vms/creating-job.png)

    >[AZURE.NOTE]若要查看与虚拟机关联的策略，请向下钻取到“受保护的项”页中的虚拟机，然后转到“备份策略”选项卡。

3. 创建作业后，可以单击 Toast 栏中的“查看作业”按钮，以在“作业”页中查看相应的作业。

    ![已创建备份作业](./media/backup-azure-manage-vms/created-job.png)

4. 成功完成作业后，即会创建可用来还原虚拟机的恢复点。这还会使“受保护的项”页中的恢复点列值递增 1。

## 停止保护虚拟机
可以使用以下选项来选择停止虚拟机的将来备份：

- 保留 Azure 备份保管库中与虚拟机关联的备份数据
- 删除与虚拟机关联的备份数据

如果你选择了第一个选项，则可以使用备份数据来还原虚拟机。有关此类虚拟机的定价详细信息，请单击[此处](http://www.windowsazure.cn/home/features/back-up/#price)。

若要停止保护虚拟机，请执行以下操作：

1. 导航到“受保护的项”页，选择“Azure 虚拟机”作为筛选类型（如果尚未选择），然后单击“选择”按钮。

    ![VM 类型](./media/backup-azure-manage-vms/vm-type.png)

2. 选择虚拟机，然后单击页底部的“停止保护”。

    ![停止保护](./media/backup-azure-manage-vms/stop-protection.png)

3. 默认情况下，Azure 备份不会删除与虚拟机关联的备份数据。

    ![确认停止保护](./media/backup-azure-manage-vms/confirm-stop-protection.png)

    如果你要删除备份数据，请选中该复选框。

    ![复选框](./media/backup-azure-manage-vms/checkbox.png)

    请选择停止备份的理由。尽管理由是选填的，但提供理由可帮助 Azure 备份处理反馈，并根据客户的情况设置处理优先级。

4. 单击“提交”按钮以提交“停止保护”作业。单击“查看作业”，在“作业”页中查看相应的作业。

    ![停止保护](./media/backup-azure-manage-vms/stop-protect-success.png)

    如果你未在“停止保护”向导中选择“删除关联的备份数据”选项，则在作业完成后，保护状态将更改为“已停止保护”。数据将会使用 Azure 备份保留，直到被显式删除。你随时都可通过在“受保护的项”页中选择虚拟机，然后单击“删除”来删除数据。

    ![已停止保护](./media/backup-azure-manage-vms/protection-stopped-status.png)

    如果你已选择“删除关联的备份数据”选项，则虚拟机将不会出现在“受保护的项”页中。

## 重新保护虚拟机
如果你未在“停止保护”中选择“删除关联的备份数据”选项，可以遵循类似于备份已注册虚拟机的步骤来重新保护虚拟机。受保护后，此虚拟机将在停止保护之前保留备份数据，并在重新保护后创建恢复点。

重新保护之后，如果有“停止保护”之前的恢复点，则虚拟机的保护状态将更改为“受保护”。

  ![重新保护 VM](./media/backup-azure-manage-vms/reprotected-status.png)

>[AZURE.NOTE]重新保护虚拟机时，你可以选择一个不同的策略，而不是最初用于保护虚拟机的策略。

## 取消注册虚拟机

如果你想要从备份保管库中删除虚拟机，请执行以下操作：

1. 单击页底部的“取消注册”按钮。

    ![禁用保护](./media/backup-azure-manage-vms/unregister-button.png)

    屏幕底部将会显示要求你进行确认的 Toast 通知。单击“是”继续。

    ![禁用保护](./media/backup-azure-manage-vms/confirm-unregister.png)

## 删除备份数据
你可以通过以下方法之一删除与虚拟机关联的备份数据：

- 在停止保护作业期间
- 在虚拟机上完成停止保护作业之后

对于在成功完成“停止备份”作业后处于“已停止保护”状态的虚拟机，若要删除其上的备份数据，请执行以下操作：

1. 导航到“受保护的项”页，选择“Azure 虚拟机”作为类型，然后单击“选择”按钮。

    ![VM 类型](./media/backup-azure-manage-vms/vm-type.png)

2. 选择虚拟机。虚拟机将显示为“已停止保护”状态。

    ![已停止保护](./media/backup-azure-manage-vms/protection-stopped-b.png)

3. 单击页底部的“删除”按钮。

    ![删除备份](./media/backup-azure-manage-vms/delete-backup.png)

4. 在“删除备份数据”向导中，选择删除备份数据的原因（强烈建议），然后单击“提交”。

    ![删除备份数据](./media/backup-azure-manage-vms/delete-backup-data.png)

5. 这将创建一个作业来删除选定虚拟机的备份数据。单击“查看作业”，在“作业”页中查看相应的作业。

    ![已成功删除数据](./media/backup-azure-manage-vms/delete-data-success.png)

    完成该作业后，与虚拟机对应的条目将从“受保护的项”页中删除。


###仪表板

在“仪表板”页中，可以查看有关 Azure 虚拟机、其存储和过去 24 小时内关联作业的信息。你可以查看备份状态和任何关联的备份错误。

  ![仪表板](./media/backup-azure-manage-vms/dashboard-protectedvms.png)

<!---HONumber=67-->