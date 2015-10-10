<properties
	pageTitle="管理 Azure 备份保管库和服务器 | Windows Azure"
	description="使用本教程来了解如何管理 Azure 备份保管库和服务器。"
	services="backup"
	documentationCenter=""
	authors="aashishr"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="backup"
	ms.date="08/13/2015"
	wacn.date="09/15/2015"/>


# 管理 Azure 备份保管库和服务器
本文概述了可通过管理门户完成的备份管理任务。

1. 登录到[管理门户](https://manage.windowsazure.cn)。
2. 单击“恢复服务”，然后单击备份保管库的名称以查看“快速启动”页。

    在“快速启动”页面顶部选择选项后，可以看到可用的管理任务。

    ![受保护的项](./media/backup-azure-manage-windows-server/RS_tabs.png)

## 仪表板
选择“仪表板”以查看服务器的使用概览。在“仪表板”的底部，可以执行以下任务：

- **管理证书**。如果使用证书注册了服务器，请使用此选项来更新证书。如果使用的是保管库凭据，请不要使用“管理证书”。
- **删除**。删除当前备份保管库。如果不再使用备份保管库，则可将其删除以释放存储空间。仅在从保管库中删除所有注册的服务器后启用“删除”。
- **保管库凭据**。使用此“速览”菜单项可以配置保管库凭据。

## 受保护的项
选择“受保护的项”可查看已从服务器备份的项。此列表仅供参考。

![受保护的项](./media/backup-azure-manage-windows-server/RS_protecteditems.png)

## 已注册的项
选择“已注册的项”可查看已注册到此保管库的服务器的名称。

![删除的服务器](./media/backup-azure-manage-windows-server/RS_deletedserver.png)

可从该处执行以下任务：- **允许重新注册** - 在为服务器选择该选项时，可使用代理中的**注册向导**再一次将服务器注册到备份保管库。由于证书中存在错误或者如果必须重新构建服务器，你可能需要重新注册。每个服务器名称只能重新注册一次。- **删除** - 从备份保管库中删除服务器。将立即删除与服务器关联的所有已存储数据。

## 后续步骤
- [从 Azure 还原 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-restore-windows-server)
- 若要了解有关 Azure 备份的详细信息，请参阅 [Azure 备份概述](/documentation/articles/backup-introduction-to-azure-backup)
- 访问 [Azure 备份论坛](http://go.microsoft.com/fwlink/p/?LinkId=290933)

<!---HONumber=69-->