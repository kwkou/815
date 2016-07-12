<properties
	pageTitle="管理 Azure 备份保管库和服务器 | Azure"
	description="使用本教程来了解如何管理 Azure 备份保管库和服务器。"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="backup"
	ms.date="04/01/2016"
	wacn.date="05/18/2016"/>


# 管理 Azure 备份保管库和服务器
本文概述了可通过经典管理门户和 Microsoft Azure 备份代理完成的备份管理任务。

>[AZURE.NOTE] 本文提供经典部署模型中的操作过程。
## 经典管理门户任务
1. 登录到[经典管理门户](https://manage.windowsazure.cn)。
2. 单击“恢复服务”，然后单击备份保管库的名称以查看“快速启动”页。

    ![管理 Azure 备份选项卡](./media/backup-azure-manage-windows-server/rs-left-nav.png)

在“快速启动”页面顶部选择选项后，可以看到可用的管理任务。

![管理 Azure 备份选项卡](./media/backup-azure-manage-windows-server/qs-page.png)

### 仪表板
选择“仪表板”以查看服务器的使用概览。“使用概览”包括：

- 已在云中注册的 Windows Server 数目
- 云中受保护的 Azure 虚拟机数目
- 在 Azure 中消耗的存储空间总量
- 最近执行的作业的状态

在“仪表板”的底部，可以执行以下任务：

- **管理证书** - 如果使用证书注册了服务器，请使用此选项来更新证书。如果使用的是保管库凭据，请不要使用“管理证书”。
- **删除** - 删除当前备份保管库。如果不再使用备份保管库，则可将其删除以释放存储空间。仅在从保管库中删除所有注册的服务器后启用“删除”。

![备份仪表板任务](./media/backup-azure-manage-windows-server/dashboard-tasks.png)

## 已注册的项
选择“已注册的项”可查看已注册到此保管库的服务器的名称。

![已注册的项](./media/backup-azure-manage-windows-server/registered-items.png)

“类型”筛选器默认为 Azure 虚拟机。若要查看已注册到此保管库的服务器名称，可从下拉菜单中选择“Windows Server”。

可从该处执行以下任务：

- **允许重新注册** - 在为服务器选择该选项时，可使用本地 Microsoft Azure 备份代理中的**注册向导**再一次将服务器注册到备份保管库。由于证书中存在错误或者如果必须重新构建服务器，你可能需要重新注册。
- **删除** - 从备份保管库中删除服务器。将立即删除与服务器关联的所有已存储数据。

## 后续步骤
- [从 Azure 还原 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-restore-windows-server/)
- 若要了解有关 Azure 备份的详细信息，请参阅 [Azure 备份概述](/documentation/articles/backup-introduction-to-azure-backup/)
- 访问 [Azure 备份论坛](http://go.microsoft.com/fwlink/p/?LinkId=290933)

<!---HONumber=Mooncake_0530_2016-->