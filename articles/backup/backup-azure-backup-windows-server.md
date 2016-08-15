<properties
   pageTitle="将 Windows Server 或 Windows 客户端文件和文件夹备份到 Azure | Azure"
   description="使用这个简单的过程将 Windows Server 或 Windows 客户端备份到 Azure。只需执行几个简单的步骤，就能将 Windows 文件和文件夹备份到云中。"
   services="backup"
   documentationCenter=""
   authors="Jim-Parker"
   manager="jwhit"
   editor=""
   keywords="windows server 备份; 备份 windows server"/>

<tags
   ms.service="backup"
   ms.date="02/05/2016"
   wacn.date="06/06/2016"/>

# 将 Windows Server 或 Windows 客户端文件和文件夹备份到 Azure
使用这个简单的过程，可以轻松将 Windows 文件和文件夹备份到 Azure。如果你尚未这样做，请完成[先决条件](/documentation/articles/backup-configure-vault/#before-you-start)部分，在环境中做好备份 Windows 计算机的准备，然后继续。

## 备份文件和文件夹
1. 注册计算机后，打开 Microsoft Azure 备份 mmc 管理单元。

    ![搜索结果](./media/backup-azure-backup-windows-server/result.png)

2. 单击“计划备份”

    ![计划 Windows Server 备份](./media/backup-azure-backup-windows-server/schedulebackup.png)

3. 选择你要备份的项。使用 Windows Server/Windows 客户端（不带 System Center Data Protection Manager）上的 Azure 备份可以保护文件和文件夹。

    ![Windows Server 备份项](./media/backup-azure-backup-windows-server/items.png)

4. 根据以下[文章](/documentation/articles/backup-azure-backup-cloud-as-tape/)中的详细说明，指定备份计划和保留策略。

5. 选择发送初始备份的方法。选择以哪种方法完成初始种子设定取决于你要备份的数据量，以及 Internet 上载链路的速度。如果你打算通过高延迟、低带宽连接备份若干 GB/TB 的数据，则建议你将磁盘寄送到最接近的 Azure 数据中心以完成初始备份。如果你有足够的带宽连接，则建议通过网络完成初始备份。

    ![初始 Windows Server 备份](./media/backup-azure-backup-windows-server/initialbackup.png)

6. 完成计划备份流程后，请返回 mmc 管理单元，然后单击“立即备份”以通过网络完成初始种子设定。

    ![立即备份 Windows Server](./media/backup-azure-backup-windows-server/backupnow.png)

7. 完成初始种子设定后，Azure 备份控制台中的“作业”视图将指示状态。

    ![IR 完成](./media/backup-azure-backup-windows-server/ircomplete.png)

## 后续步骤
- [管理 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-manage-windows-server/)
- [从 Azure 还原 Windows Server 或 Windows 客户端](/documentation/articles/backup-azure-restore-windows-server/)
- [Azure 备份常见问题](/documentation/articles/backup-azure-backup-faq/)
- 访问 [Azure 备份论坛](http://go.microsoft.com/fwlink/p/?LinkId=290933)

<!---HONumber=Mooncake_0530_2016-->