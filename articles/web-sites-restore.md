<properties linkid="web-sites-restore" urlDisplayName="Restore a Microsoft Azure web site" pageTitle="Restore a Microsoft Azure web site" metaKeywords="Azure Web Sites, Restore, restoring" description="Learn how to restore your Azure web sites from backup." metaCanonical="" services="web-sites" documentationCenter="" title="Restore a Microsoft Azure web site" authors="timamm"  solutions="" writer="timamm" manager="paulettm" editor="mollybos"  />

# 还原 Microsoft Azure 网站

本文介绍如何还原以前通过使用 Azure 网站备份功能备份的网站。有关详细信息，请参阅 [Microsoft Azure 网站备份][Microsoft Azure 网站备份]。

利用 Azure 网站还原功能，可将网站还原到以前某个你自己需要的状态，或基于原有网站备份之一创建新网站。创建与最新版本并行运行的新网站对于 A/B 测试会很有用。

“还原”功能在 Azure 网站门户中的“备份”选项卡上，只能用于“标准”模式。

## 本文内容

-   [从以前制作的备份中还原 Azure 网站][从以前制作的备份中还原 Azure 网站]
-   [从存储帐户直接还原 Azure 网站][从存储帐户直接还原 Azure 网站]
-   [选择网站还原设置并开始还原操作][选择网站还原设置并开始还原操作]
-   [查看操作日志][查看操作日志]

<a name="PreviousBackup"></a>

## 从以前制作的备份中还原 Azure 网站

1.  在“备份”选项卡上，单击门户页底部的命令栏中的“立即还原”。此时将显示“立即还原”对话框。

    ![选择备份源][选择备份源]

2.  在“选择备份源”下，选择“该网站的上一个备份”。
3.  选择要还原的备份的日期，然后单击向右箭头以继续。
4.  按本文后面[选择网站还原设置][选择网站还原设置并开始还原操作]一节中的步骤操作。

<a name="StorageAccount"></a>

## 从存储帐户直接还原 Azure 网站

1.  在“备份”选项卡上，单击门户页底部的命令栏中的“立即还原”。此时将显示“立即还原”对话框。

    ![选择备份源][选择备份源]

2.  在“选择备份源”下，选择“存储帐户文件”。在此可以直接指定存储帐户文件的 URL，或单击文件夹图标以导航到 blob 存储并指定备份文件。此示例选择文件夹图标。

    ![存储帐户文件][存储帐户文件]

3.  单击文件夹图标，以打开“浏览云存储”对话框。

    ![浏览云存储][浏览云存储]

4.  展开要使用的存储帐户的名称，然后选择 **websitebackups**，它包含你的备份。
5.  选择包含要还原的备份的 zip 文件，然后单击“打开”。
6.  存储帐户文件已选好，并显示在存储帐户框中。单击向右箭头以继续。

    ![已选择的存储帐户文件][已选择的存储帐户文件]

7.  继续按下一节[选择网站还原设置并开始还原操作][选择网站还原设置并开始还原操作]中的说明操作。

<a name="RestoreSettings"></a>

## 选择网站还原设置并开始还原操作

1.  在“选择网站还原设置”下的“还原到”中，选择“当前网站”或“新网站实例”。

    ![选择网站还原设置][选择网站还原设置]

    如果选择“当前网站”，现有网站将会被所选备份改写（破坏性还原）。所选备份创建时间之后你对网站所做的所有更改都将被永久删除，而且还原操作是不可撤消的。还原操作期间，将暂时无法使用当前网站，你将会收到相关警告信息。

    如果选择“新网站实例”，将会在同一区域中使用你指定的名称创建一个新网站。（默认情况下，新名称是 **restored-** *oldWebSiteName*。）

    你还原的网站所包含的内容和配置将与在门户中为原网站提供的内容和配置相同。它还将包括你在下一步中选择要包括的任何数据库。

2.  如果要连同网站一起还原数据库，请在“包含的数据库”下通过使用“还原到”下的下拉列表选择要将数据库还原到其中的数据库服务器的名称。你还可以选择创建要还原到其中的新数据库服务器，或者选择“不还原”，这是默认设置，这样就不会还原数据库。

    选择了服务器名称后，在“数据库名称”框中为还原指定目标数据库的名称。

    如果你的还原包括一个或多个数据库，可选择“自动调整连接字符串”以将备份中存储的连接字符串更新为指向新数据库，或指向数据库服务器，视需要而定。还原完成后，应验证与数据库相关的所有功能都能正常使用。

    ![选择数据库服务器主机][选择数据库服务器主机]

    > [WACOM.NOTE] 不能还原与同一 SQL Server 同名的 SQL 数据库。必须选择其他数据库名称或其他要将该数据库还原到其中的 SQL Server 主机。

<!--     > [WACOM.NOTE] You can restore a MySQL database with the same name to the same server, but be aware that this will clear out the existing content stored in the MySQL database. -->

1.  如果选择还原现有数据库，需要提供用户名和密码。如果选择还原到新数据库，需要提供新数据库名称：

    ![还原到新 SQL 数据库][还原到新 SQL 数据库]

    单击向右箭头以继续。

2.  如果选择创建新数据库，需要在下一个对话框中提供该数据库的凭据和其他初始配置信息。这里的示例显示的是一个新的 SQL 数据库。（用于新 MySQL 数据库的选项会有所不同。）

    ![新 SQL 数据库设置][新 SQL 数据库设置]

3.  单击复选标记以开始还原操作。操作完成时，在门户网站的列表中将能够看到新网站实例（如果那是你选择的还原选项）。

    ![还原的 Contoso 网站][还原的 Contoso 网站]

<a name="OperationLogs"></a>

## 查看操作日志

1.  若要查看有关网站还原操作成功或失败的详细信息，请转到网站的“仪表板”选项卡。在“速览”部分的“管理服务”下，单击“操作日志”。

    ![仪表板 - 操作日志链接][仪表板 - 操作日志链接]

2.  你会跳转到“管理服务”门户“操作日志”页面，并可在里面的操作日志列表中看到还原操作日志：

    ![“管理服务操作日志”页面][“管理服务操作日志”页面]

3.  若要查看有关该操作的详细信息，请在列表中选择该操作，然后单击命令栏上的“详细信息”按钮。

    ![“详细信息”按钮][“详细信息”按钮]

    进行此操作时，“操作详细信息”窗口会打开，显示日志文件的可复制内容：

    ![操作详细信息][操作详细信息]

<!-- IMAGES -->

  [Microsoft Azure 网站备份]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-backup/
  [从以前制作的备份中还原 Azure 网站]: #PreviousBackup
  [从存储帐户直接还原 Azure 网站]: #StorageAccount
  [选择网站还原设置并开始还原操作]: #RestoreSettings
  [查看操作日志]: #OperationLogs
  [选择备份源]: ./media/web-sites-restore/01ChooseBackupSource.png
  [存储帐户文件]: ./media/web-sites-restore/02StorageAccountFile.png
  [浏览云存储]: ./media/web-sites-restore/03BrowseCloudStorage.png
  [已选择的存储帐户文件]: ./media/web-sites-restore/04StorageAccountFileSelected.png
  [选择网站还原设置]: ./media/web-sites-restore/05ChooseRestoreSettings.png
  [选择数据库服务器主机]: ./media/web-sites-restore/06ChooseDBServer.png
  [还原到新 SQL 数据库]: ./media/web-sites-restore/07RestoreToNewSQLDB.png
  [新 SQL 数据库设置]: ./media/web-sites-restore/08NewSQLDBConfig.png
  [还原的 Contoso 网站]: ./media/web-sites-restore/09RestoredContosoWebSite.png
  [仪表板 - 操作日志链接]: ./media/web-sites-restore/10DashboardOperationLogsLink.png
  [“管理服务操作日志”页面]: ./media/web-sites-restore/11ManagementServicesOperationLogsList.png
  [“详细信息”按钮]: ./media/web-sites-restore/12DetailsButton.png
  [操作详细信息]: ./media/web-sites-restore/13OperationDetails.png
