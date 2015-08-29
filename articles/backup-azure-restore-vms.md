
<properties
	pageTitle="Azure 备份 - 恢复虚拟机"
	description="了解如何恢复 Azure 虚拟机"
	services="backup"
	documentationCenter=""
	authors="trinadhk"
	manager="shreeshd"
	editor=""/>

<tags
	ms.service="backup"

	ms.date="07/17/2015"
	wacn.date=""/>

# 还原虚拟机
你可以使用还原操作，通过存储在 Azure 备份保管库中的备份将虚拟机还原到新的 VM。

## 选择要还原的项

1. 导航到“受保护的项”选项卡，然后选择你要还原到新 VM 的虚拟机。

    ![受保护的项](./media/backup-azure-restore-vms/protected-items.png)

    “受保护的项”页中的“恢复点”字段显示了虚拟机的恢复点数目。“最新恢复点”字段显示了可从中还原虚拟机的最近备份。

2. 单击“还原”打开“还原项”向导。

    ![还原项](./media/backup-azure-restore-vms/restore-item.png)

## 选择恢复点

1. 在“选择恢复点”屏幕中，你可以从最新的恢复点进行恢复，或者从以前的某个时间点进行恢复。向导打开时默认选择的选项是“最新恢复点”。

    ![选择恢复点](./media/backup-azure-restore-vms/select-recovery-point.png)

2. 若要选择更早的时间点，请在下拉列表中选择“选择日期”选项，并通过单击**日历图标**，在日历控件中选择一个日期。在控件中，所有具有恢复点的日期以浅灰色阴影填充，并可供用户选择。

    ![选择日期](./media/backup-azure-restore-vms/select-date.png)

    在你单击日历控件中的日期后，该日可用的恢复点将显示在下面的恢复点表中。“时间”列指示生成快照的时间。“类型”列显示恢复点的[一致性](documentation/articles/backup-azure-vms)。表标题在括号中显示该日期可用的恢复点数目。

    ![恢复点](./media/backup-azure-restore-vms/recovery-points.png)

3. 从“恢复点”表中选择恢复点，然后单击“下一步”箭头转到下一个屏幕。

## 指定目标位置

1. 在“选择还原实例”屏幕中，指定有关要将虚拟机还原到何处的详细信息。

  - 指定虚拟机名称：在指定的云服务中，虚拟机名称应该是唯一的。如果你打算使用相同的名称替换现有的 VM，请先删除现有的 VM 和数据磁盘，然后从 Azure 备份还原数据。
  - 选择 VM 的云服务：这是创建 VM 的必要步骤。你可以选择使用现有的云服务，或创建新的云服务。

        所选云服务名称应全局唯一。通常，云服务名称以 [cloudservice].cloudapp.net 的形式与面向公众的 URL 相关联。如果该名称已被使用，Azure 不会允许你创建新的云服务。如果你选择创建新的云服务，它将被提供与虚拟机相同的名称，在这种情况下，所选 VM 名称将具备足够的唯一性，可以应用于关联的云服务。

        我们仅显示未与还原实例详细信息中的任何地缘组关联的云服务和虚拟网络。[了解详细信息](https://msdn.microsoft.com/zh-cn/library/azure/jj156085.aspx)。

2. 选择 VM 的存储帐户：这是创建 VM 的必要步骤。你可以选择与 Azure 备份保管库位于相同区域的现有存储帐户。不支持区域冗余或高级存储类型的存储帐户。

    如果没有受支持配置的存储帐户，请在启动还原操作之前创建一个具有受支持配置的存储帐户。

    ![选择虚拟网络](./media/backup-azure-restore-vms/restore-sa.png)

3. 选择虚拟网络：在创建 VM 时应该已经选择了虚拟机的虚拟网络 (VNET)。还原 UI 将显示此订阅中所有可用的 VNET。为已还原的 VM 选择 VNET 不是必要步骤 – 即使不应用 VNET，你也可以通过 Internet 连接到已还原的虚拟机。

    如果选择的云服务与虚拟网络关联，则你无法更改虚拟网络。

    ![选择虚拟网络](./media/backup-azure-restore-vms/restore-cs-vnet.png)

4. 选择子网：如果 VNET 有子网，默认选择的选项为第一个子网。从下拉选项中选择你想要的子网。有关子网详细信息，请转到[门户主页](https://manage.windowsazure.cn)中的“网络”扩展，然后转到“虚拟网络”并在选择虚拟网络后，向下钻取到“配置”以查看子网详细信息。

    ![选择子网](./media/backup-azure-restore-vms/select-subnet.png)

5. 在向导中单击“提交”图标以提交详细信息并创建还原作业。

## 跟踪还原操作
在还原向导中输入所有信息并提交后，Azure 备份将尝试创建一个作业来跟踪还原操作。

![创建还原作业](./media/backup-azure-restore-vms/create-restore-job.png)

如果成功创建作业，会出现一条 toast 通知，指出作业已创建。你可以单击“查看作业”按钮进入“作业”选项卡，以获取更多详细信息。

![已创建还原作业](./media/backup-azure-restore-vms/restore-job-created.png)

还原操作完成后，系统将在“作业”选项卡中将其标记为已完成。

![还原作业已完成](./media/backup-azure-restore-vms/restore-job-complete.png)

还原虚拟机后，你可能需要重新安装原始 VM 上的扩展，并在 Azure 门户中为虚拟机[修改终结点](/documentation/articles/virtual-machines-set-up-endpoints)。

## 排查错误
大部分的错误都可以根据“错误详细信息”中所建议的操作予以解决。下面是可以帮助进行故障排除的附加要点：

| 备份操作 | 错误详细信息 | 解决方法 |
| -------- | -------- | -------|
| 还原 | 发生云内部错误，还原失败 | <ol><li>使用 DNS 设置配置了你正在尝试还原的云服务。你可以检查 <br>$deployment = Get-AzureDeployment -ServiceName "ServiceName" -Slot "Production" Get-AzureDns -DnsSettings $deployment.DnsSettings<br>如果配置了地址，则表示配置了 DNS 设置。<br> <li>尝试还原的云服务配置了 ReservedIP，且云服务中现有的 VM 处于停止状态。<br>可以使用以下 PowerShell cmdlet 检查云服务是否有保留的 IP：<br>$deployment = Get-AzureDeployment -ServiceName "servicename" -Slot "Production" $dep.ReservedIPName</ol> |

## 后续步骤
- [管理虚拟机](/documentation/articles/backup-azure-manage-vms)

<!---HONumber=66-->