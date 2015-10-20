<properties
   pageTitle="Azure 备份 - 从 Windows Server 或 Windows 客户端备份和还原"
   description="了解如何从 Windows Server 或 Windows 客户端备份和还原。本文还介绍了备选的服务器恢复方法"
   services="backup"
   documentationCenter=""
   authors="aashishr" 
   manager="jwhit" 
   editor=""/>

<tags
	ms.service="backup"
	ms.date="08/18/2015"
	wacn.date="09/15/2015"/>

# 从 Windows Server 或 Windows 客户端计算机备份和还原
本文介绍从 Windows server 或 Windows 客户端计算机进行备份所要执行的步骤。此外，还介绍了在同一台计算机还原已备份的文件，以及在任何其他计算机上还原备份的文件所要执行的步骤。

## 备份文件
1. 注册计算机后，打开 Windows Azure 备份 mmc 管理单元。<br/>
![搜索结果][1]

2. 单击“计划备份”<br/>
![计划备份][2]

3. 选择你要备份的项。使用 Windows Server/Windows 客户端（不带 System Center Data Protection Manager）上的 Azure 备份可以保护文件和文件夹。<br/>
![要备份的项目][3]

4. 根据以下[文章](/documentation/articles/backup-azure-backup-cloud-as-tape)中的详细说明，指定备份计划和保留策略

5. 选择发送初始备份的方法。选择以哪种方法完成初始种子设定取决于你要备份的数据量，以及 Internet 上载链路的速度。如果你打算通过高延迟、低带宽连接备份若干 GB/TB 的数据，则建议你将磁盘寄送到最接近的 Azure 数据中心以完成初始备份。这称为“脱机备份”，此[文章](https://msdn.microsoft.com/zh-cn/library/azure/dn894419.aspx)中已做了详述。如果你有足够的带宽连接，则建议通过网络完成初始备份。<br/>
![初始备份][4]

6. 完成工作流后，请返回 mmc 管理单元，然后单击“立即备份”以通过网络完成初始种子设定。<br/>
![立即备份][5]

7. 完成初始种子设定后，Azure 备份控制台中的“作业”视图将指示状态。<br/>
![IR 完成][6]

## 在同一台计算机上恢复数据
如果你意外删除了某个文件并想要在同一台计算机（备份位置）中还原文件/卷，以下步骤可帮助你恢复数据。

1. 打开“Windows Azure 备份”管理单元。

2. 单击“恢复数据”以启动工作流。<br/>
![恢复数据][7]

3. 选择“此服务器(*你的计算机名称*)”选项，因为你打算在同一台计算机上还原已备份的文件。<br/>
![同一台计算机][8]

4. 你可以选择“浏览文件”或“搜索文件”。如果你想要还原路径已知的一个或多个文件，请保留默认选项。如果你不确定文件夹的结构，但想要搜索文件，请选择“搜索文件”选项。对于本部分，我们将使用默认选项继续操作。<br/>
![浏览文件][9]

5. 选择你要从中还原文件的卷。在该屏幕中，你可以从任意时间点还原。日历控件中以“粗体”显示的日期指明了还原点的可用性。选择日期后，根据你的备份计划（和备份操作的成功与否），可以从“时间”下拉列表中选择一个时间点。<br/>
![卷和日期][10]

6. 选择你要恢复的项。可以选择多个要还原的文件夹/文件。<br/>
![选择文件][11]

7. 指定恢复参数。<br/>
![恢复选项][12]
  + 你可以选择还原到原始位置（将覆盖其中的文件/文件夹），或者同一台计算机中的另一个位置。

  + 如果要还原的文件/文件夹在目标位置存在，则你可以选择创建副本（同一文件的两个版本）、覆盖目标位置中的文件，或者跳过目标中存在的文件的恢复。

  + 强烈建议你保留所要恢复的文件中的 ACL 还原默认选项。

8. 提供这些输入后，恢复工作流将会启动，并将文件还原到此计算机。

## 恢复到备用计算机
如果整个服务器断开连接，你仍可以在另一台计算机中恢复文件/卷。下面的步骤演示了工作流。

步骤中使用的术语如下：
  + *源计算机* – 从中创建备份并且当前不可用的原始计算机。

  + *目标计算机* – 正在检索的数据所在的计算机。

  + *示例保管库* – *源计算机*和*目标计算机*注册到的备份保管库。<br/>

> [AZURE.NOTE]从一台计算机创建的备份无法在运行更低版本操作系统的计算机上还原。例如，如果备份是从 Windows 7 计算机创建的，它可以在 Windows 8 或更高版本的计算机上还原。但是，如果反过来则无法还原。

1. 在*目标计算机*中打开“Windows Azure 备份”管理单元。

2. 确保*目标计算机*和*源计算机*已还原到同一个备份保管库。

3. 单击“恢复数据”以启动工作流。<br/>
![恢复数据][7]

4. 选择“另一台服务器”<br/>
![另一台服务器][13]

5. 提供对应于*示例保管库*的保管库凭据文件。如果保管库凭据文件无效（或已过期），请在 Azure 门户中从*示例保管库*下载新的保管库凭据文件。提供保管库凭据文件后，系统将根据保管库凭据文件显示备份保管库。

6. 从显示的计算机列表中选择*源计算机*。<br/>
![计算机列表][14]

7. 如前所述，选择“搜索文件”或“浏览文件”选项。对于本部分，我们将使用“搜索文件”选项。<br/>
![搜索][15]

8. 在下一屏幕中选择卷和日期。搜索你要还原的文件夹/文件名称。<br/>
![搜索项][16]

9. 选择这些文件需要还原到的位置。<br/>
![还原位置][17]

10. 提供在将*源计算机*注册到*示例保管库*期间所用的加密通行短语。<br/>
![加密][18]

11. 提供输入后，单击“恢复”按钮，随即将会触发在提供的目标中还原备份文件的操作。


## 后续步骤
- [Azure 备份常见问题](/documentation/articles/backup-azure-backup-faq)

<!--Image references-->
[1]: ./media/backup-azure-backup-and-recover/result.png
[2]: ./media/backup-azure-backup-and-recover/schedulebackup.png
[3]: ./media/backup-azure-backup-and-recover/items.png
[4]: ./media/backup-azure-backup-and-recover/initialbackup.png
[5]: ./media/backup-azure-backup-and-recover/backupnow.png
[6]: ./media/backup-azure-backup-and-recover/ircomplete.png

[7]: ./media/backup-azure-backup-and-recover/recover.png
[8]: ./media/backup-azure-backup-and-recover/samemachine.png
[9]: ./media/backup-azure-backup-and-recover/browseandsearch.png
[10]: ./media/backup-azure-backup-and-recover/volanddate.png
[11]: ./media/backup-azure-backup-and-recover/selectfiles.png
[12]: ./media/backup-azure-backup-and-recover/recoveroptions.png

[13]: ./media/backup-azure-backup-and-recover/anotherserver.png
[14]: ./media/backup-azure-backup-and-recover/machinelist.png
[15]: ./media/backup-azure-backup-and-recover/search.png
[16]: ./media/backup-azure-backup-and-recover/searchitems.png
[17]: ./media/backup-azure-backup-and-recover/restorelocation.png
[18]: ./media/backup-azure-backup-and-recover/encryption.png

<!---HONumber=67-->