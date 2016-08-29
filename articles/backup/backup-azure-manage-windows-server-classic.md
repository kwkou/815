<properties
	pageTitle="使用 Azure 经典部署模型管理 Azure 备份保管库和服务器 | Azure"
	description="使用本教程来了解如何管理 Azure 备份保管库和服务器。"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="backup"
	ms.date="06/15/2016"
	wacn.date="08/29/2016"/>


# 使用经典部署模型管理 Azure 备份保管库和服务器

> [AZURE.SELECTOR]
- [资源管理器](/documentation/articles/backup-azure-manage-windows-server/)
- [经典](/documentation/articles/backup-azure-manage-windows-server-classic/)

本文概述了可通过 Azure 经典管理门户和 Azure 备份代理完成的备份管理任务。

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-classic-include.md)] 资源管理器部署模型。

## 管理门户任务
1. 登录到[管理门户](https://manage.windowsazure.cn)。

2. 单击“恢复服务”，然后单击备份保管库的名称以查看“快速启动”页。

    ![恢复服务](./media/backup-azure-manage-windows-server-classic/rs-left-nav.png)  


	在“快速启动”页面顶部选择选项后，可以看到可用的管理任务。

	![管理选项卡](./media/backup-azure-manage-windows-server-classic/qs-page.png)  


### 仪表板
选择“仪表板”以查看服务器的使用概览。“使用概览”包括：

- 已在云中注册的 Windows Server 数目
- 云中受保护的 Azure 虚拟机数目
- 在 Azure 中消耗的存储空间总量
- 最近执行的作业的状态

在“仪表板”的底部，可以执行以下任务：

- **管理证书** - 如果使用证书注册了服务器，请使用此选项来更新证书。如果使用的是保管库凭据，请不要使用“管理证书”。
- **删除** - 删除当前备份保管库。如果不再使用备份保管库，则可将其删除以释放存储空间。仅在从保管库中删除所有注册的服务器后启用“删除”。

![备份仪表板任务](./media/backup-azure-manage-windows-server-classic/dashboard-tasks.png)  


## 已注册的项
选择“已注册的项”可查看已注册到此保管库的服务器的名称。

![已注册的项](./media/backup-azure-manage-windows-server-classic/registered-items.png)

“类型”筛选器默认为 Azure 虚拟机。若要查看已注册到此保管库的服务器名称，可从下拉菜单中选择“Windows Server”。

可从该处执行以下任务：

- **允许重新注册** - 在为服务器选择该选项时，可使用本地 Azure 备份代理中的**注册向导**再一次将服务器注册到备份保管库。由于证书中存在错误或者如果必须重新构建服务器，你可能需要重新注册。
- **删除** - 从备份保管库中删除服务器。将立即删除与服务器关联的所有已存储数据。

    ![已注册的项任务](./media/backup-azure-manage-windows-server-classic/registered-items-tasks.png)

## 受保护的项
选择“受保护的项”可查看已从服务器备份的项。

![受保护的项](./media/backup-azure-manage-windows-server-classic/protected-items.png)

## 配置

从“配置”选项卡中，可以选择适当的存储冗余选项。选择存储冗余选项的最佳时机是在创建保管库之后、将任何计算机注册到保管库之前。

>[AZURE.WARNING] 将某个项注册到保管库后，存储冗余选项将会锁定且不能修改。

![配置](./media/backup-azure-manage-windows-server-classic/configure.png)

有关[存储冗余](/documentation/articles/storage-redundancy/)的详细信息，请参阅此文。

## Azure 备份代理任务

### 控制台

打开 **Azure 备份代理**（可以通过在计算机中搜索 *Microsoft Azure 备份*来找到它）。

![备份代理](./media/backup-azure-manage-windows-server-classic/snap-in-search.png)  


通过备份代理控制台右侧的“操作”，可以执行以下管理任务：

- 注册服务器
- 计划备份
- 立即备份
- 更改属性

![代理控制台操作](./media/backup-azure-manage-windows-server-classic/console-actions.png)

>[AZURE.NOTE] 若要**恢复数据**，请参阅 [Restore files to a Windows server or Windows client machine](/documentation/articles/backup-azure-restore-windows-server/)（将文件还原到 Windows Server 或 Windows 客户端计算机）。

### 修改现有备份

1. 在 Azure 备份代理中，单击“计划备份”。

    ![计划 Windows Server 备份](./media/backup-azure-manage-windows-server-classic/schedule-backup.png)  


2. 在计划备份向导中，将“更改备份项或时间”选项保留选中状态，然后单击“下一步”。

    ![修改计划的备份](./media/backup-azure-manage-windows-server-classic/modify-or-stop-a-scheduled-backup.png)

3. 如果你要添加或更改项，请在“选择要备份的项”屏幕中单击“添加项”。

    还可以在此向导页中设置“排除设置”。如果你要排除文件或文件类型，请阅读有关添加[排除设置](#exclusion-settings)的过程。

4. 选择要备份的文件和文件夹，然后单击“确定”。

    ![添加项](./media/backup-azure-manage-windows-server-classic/add-items-modify.png)

5. 指定“备份计划”并单击“下一步”。

    可以计划每日（一天最多 3 次）或每周备份。

    ![指定备份计划](./media/backup-azure-manage-windows-server-classic/specify-backup-schedule-modify-close.png)  


    >[AZURE.NOTE] [此文](/documentation/articles/backup-azure-backup-cloud-as-tape/)中详细介绍了如何指定备份计划。

6. 选择备份副本的“保留策略”，然后单击“下一步”。

    ![选择保留策略](./media/backup-azure-manage-windows-server-classic/select-retention-policy-modify.png)

7. 在“确认”屏幕上复查信息，然后单击“完成”。

8. 向导完成创建备份计划后，请单击“关闭”。

    修改保护设置后，可以通过转到“作业”选项卡并确认更改已反映在备份作业中，来确认可正确触发备份。

### 启用网络限制  
Azure 备份代理提供的“限制”选项卡可让你控制在数据传输期间使用网络带宽的方式。如果需要在上班时间内备份数据，但不希望备份程序干扰其他 Internet 流量，此控制机制很有帮助。数据传输的限制适用于备份和还原活动。

若要启用限制，请执行以下操作：

1. 在“备份代理”中，单击“更改属性”。

2. 选中“为备份操作启用 Internet 带宽使用限制”复选框。

    ![网络限制](./media/backup-azure-manage-windows-server-classic/throttling-dialog.png)  


3. 启用限制后，指定在“工作时间”和“非工作时间”允许使用多少带宽进行备份数据传输。

    带宽值从每秒 512 千字节 (Kbps) 开始，最高可为每秒 1023 兆字节 (Mbps)。你还可以指定“工作时间”的开始和结束时间，以及一周中有哪几天被视为工作日。在指定的工作时间之外的时间被视为非工作时间。

4. 单击**“确定”**。

## 排除设置

1. 打开 **Microsoft Azure 备份代理**（可以通过在计算机中搜索 *Microsoft Azure 备份*来找到它）。

    ![打开备份代理](./media/backup-azure-manage-windows-server-classic/snap-in-search.png)

2. 在 Azure 备份代理中，单击“计划备份”。

    ![计划 Windows Server 备份](./media/backup-azure-manage-windows-server-classic/schedule-backup.png)  


3. 在计划备份向导中，将“更改备份项或时间”选项保留选中状态，然后单击“下一步”。

    ![修改计划](./media/backup-azure-manage-windows-server-classic/modify-or-stop-a-scheduled-backup.png)

4. 单击“排除设置”。

    ![选择要排除的项](./media/backup-azure-manage-windows-server-classic/exclusion-settings.png)  


5. 单击“添加排除项”。

    ![添加排除项](./media/backup-azure-manage-windows-server-classic/add-exclusion.png)  


6. 选择位置，然后单击“确定”。

    ![选择要排除的位置](./media/backup-azure-manage-windows-server-classic/exclusion-location.png)

7. 在“文件类型”字段中添加文件扩展名。

    ![按文件类型排除](./media/backup-azure-manage-windows-server-classic/exclude-file-type.png)  


    添加 .mp3 扩展名

    ![文件类型示例](./media/backup-azure-manage-windows-server-classic/exclude-mp3.png)  


    若要添加其他扩展名，请单击“添加排除项”，然后输入另一个文件类型扩展名（添加 .jpeg 扩展名）。

    ![另一个文件类型示例](./media/backup-azure-manage-windows-server-classic/exclude-jpg.png)  


8. 添加所有扩展名之后，请单击“确定”。

9. 单击“下一步”继续运行计划备份向导，出现“确认”页时，请单击“完成”。

    ![确认排除](./media/backup-azure-manage-windows-server-classic/finish-exclusions.png)

## 后续步骤
- [从 Azure 还原 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-restore-windows-server/)
- 若要了解有关 Azure 备份的详细信息，请参阅 [Azure 备份概述](/documentation/articles/backup-introduction-to-azure-backup/)
- 访问 [Azure 备份论坛](http://go.microsoft.com/fwlink/p/?LinkId=290933)

<!---HONumber=Mooncake_0822_2016-->