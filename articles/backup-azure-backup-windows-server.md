<properties
   pageTitle="将 Windows Server 或 Windows 客户端文件和文件夹备份到 Azure | Windows Azure"
   description="了解如何从 Windows Server 或 Windows 客户端备份到 Azure。"
   services="backup"
   documentationCenter=""
   authors="aashishr"
   manager="jwhit"
   editor=""/>

<tags
   ms.service="backup"
   ms.date="08/13/2015"
   wacn.date="09/15/2015"/>

# 将 Windows Server 或 Windows 客户端文件和文件夹备份到 Azure
本文介绍将 Windows Server 或 Windows 客户端文件和文件夹备份到 Azure 需要执行的步骤。

## 开始之前
在继续之前，请确保符合使用 Microsoft Azure 备份保护 Windows Server 和 Windows 客户端的所有[先决条件](/documentation/articles/backup-configure-vault#before-you-start)。先决条件包括如下任务：创建备份保管库、下载保管库凭据、安装 Azure 备份代理，以及向保管库注册计算机。

## 备份文件
1. 注册计算机后，打开 Microsoft Azure 备份 mmc 管理单元。

    ![搜索结果](./media/backup-azure-backup-windows-server/result.png)

2. 单击“计划备份”

    ![计划备份](./media/backup-azure-backup-windows-server/schedulebackup.png)

3. 选择你要备份的项。使用 Windows Server/Windows 客户端（不带 System Center Data Protection Manager）上的 Azure 备份可以保护文件和文件夹。

    ![要备份的项目](./media/backup-azure-backup-windows-server/items.png)

4. 根据以下[文章](/documentation/articles/backup-azure-backup-cloud-as-tape)中的详细说明，指定备份计划和保留策略。

5. 选择发送初始备份的方法。选择以哪种方法完成初始种子设定取决于你要备份的数据量，以及 Internet 上载链路的速度。如果你打算通过高延迟、低带宽连接备份若干 GB/TB 的数据，则建议你将磁盘寄送到最接近的 Azure 数据中心以完成初始备份。这称为“脱机备份”，此[文章](/documentation/articles/backup-azure-backup-import-export)中已做了详述。如果你有足够的带宽连接，则建议通过网络完成初始备份。

    ![初始备份](./media/backup-azure-backup-windows-server/initialbackup.png)

6. 完成计划备份流程后，请返回 mmc 管理单元，然后单击“立即备份”以通过网络完成初始种子设定。

    ![立即备份](./media/backup-azure-backup-windows-server/backupnow.png)

7. 完成初始种子设定后，Azure 备份控制台中的“作业”视图将指示状态。

    ![IR 完成](./media/backup-azure-backup-windows-server/ircomplete.png)

## 后续步骤
- [管理 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-manage-windows-server)
- [从 Azure 还原 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-restore-windows-server)
- [Azure 备份常见问题](/documentation/articles/backup-azure-backup-faq)

<!---HONumber=69-->