<properties 
	pageTitle="在 Azure 中还原 Web 应用" 
	description="了解如何从备份还原 Web 应用。" 
	services="app-service\web" 
	documentationCenter="" 
	authors="cephalin" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service"
	ms.date="01/26/2016" 
	wacn.date="03/28/2016"/>

#还原 Azure Web 应用

本文介绍如何通过使用 Azure Web 应用备份功能来还原你先前备份的 Web 应用。有关详细信息，请参阅 [Azure Web 应用备份](/documentation/articles/web-sites-backup/)。

利用 Web 应用还原功能，可将 Web 应用还原到以前某个你自己需要的状态，或基于原有 Web 应用备份之一创建新的 Web 应用。创建与最新版本并行运行的新 Web 应用对于 A/B 测试会很有用。

Web 应用还原功能在 [Azure 经典管理门户](http://manage.windowsazure.cn)中的“备份”边栏选项卡上，只能用于“标准”模式。有关使用“标准”模式缩放应用的信息，请参阅[在 Azure 中缩放 Web 应用](/documentation/articles/web-sites-scale/)。

<a name="PreviousBackup"></a>
## 从以前制作的备份中还原 Web 应用

1. 在“备份”选项卡上，单击经典管理门户页底部的命令栏中的“立即还原”。此时将显示“立即还原”对话框。
	
	![选择备份源][ChooseBackupSource]
	
2. 在“选择备份源”下，选择“该 Web 应用的上一个备份”。
3. 选择要还原的备份的日期，然后单击向右箭头以继续。
4. 按本文后面[选择 Web 应用还原设置](#RestoreSettings)部分的步骤操作。

<a name="StorageAccount"></a>
##直接从存储帐户还原 Azure Web 应用

1. 在“备份”选项卡上，单击经典管理门户页底部的命令栏中的“立即还原”。此时将显示“立即还原”对话框。
	
	![选择备份源][ChooseBackupSource]
	
2. 在“选择备份源”下，选择“存储帐户文件”。在此可以直接指定存储帐户文件的 URL，或单击文件夹图标以导航到 blob 存储并指定备份文件。此示例选择文件夹图标。
	
	![存储帐户文件][StorageAccountFile]
	
3. 单击文件夹图标，以打开“浏览云存储”对话框。
	
	![浏览云存储][BrowseCloudStorage]
	

4. 展开要使用的存储帐户的名称，然后选择 **Websitebackups**，其中包含你的备份。
5. 选择包含要还原的备份的 zip 文件，然后单击“打开”。
6. 存储帐户文件已选好，并显示在存储帐户框中。单击向右箭头以继续。
	
	![已选择的存储帐户文件][StorageAccountFileSelected]
	
7. 继续学习[选择 Web 应用还原设置并开始还原操作](#RestoreSettings)部分。

<a name="RestoreSettings"></a>
##选择 Web 应用还原设置并开始还原操作
1. 在“选择 Web 应用还原设置”下的“还原到”中，选择“当前 Web 应用”或“新 Web 应用实例”。
	
	![选择 Web 应用还原设置][ChooseRestoreSettings]
	
	如果选择“当前 Web 应用”，现有 Web 应用将会被所选备份改写（破坏性还原）。所选备份创建时间之后对 Web 应用所做的所有更改都将被永久删除，而且还原操作是不可撤消的。还原操作期间，将暂时无法使用当前 Web 应用，你将会收到相关警告信息。
	
	如果选择“新 Web 应用实例”，将会在同一区域中使用你指定的名称创建一个新 Web 应用。（默认情况下，新名称是 **restored-***旧 Web 应用名称*。）
	
	你还原的 Web 应用所包含的内容和配置将与在经典管理门户中为原 Web 应用提供的内容和配置相同。它还将包括你在下一步中选择要包括的任何数据库。
2. 如果要连同 Web 应用一起还原数据库，请在“包含的数据库”下通过使用“还原到”下的下拉列表选择要将数据库还原到其中的数据库服务器的名称。你还可以选择创建要还原到其中的新数据库服务器，或者选择“不还原”，这是默认设置，这样就不会还原数据库。 
	
	选择了服务器名称后，在“数据库名称”框中为还原指定目标数据库的名称。
	
	如果你的还原包括一个或多个数据库，可选择“自动调整连接字符串”以将备份中存储的连接字符串更新为指向新数据库，或指向数据库服务器，视需要而定。还原完成后，应验证与数据库相关的所有功能都能正常使用。
	
	![选择数据库服务器主机][ChooseDBServer]
	
	> [AZURE.NOTE] 不能还原与同一 SQL Server 同名的 SQL 数据库。必须选择其他数据库名称或其他要将该数据库还原到其中的 SQL Server 主机。
	
	> [AZURE.NOTE] 您可以将同名的 MySQL 数据库还原到同一服务器，但请注意，这将清除出存储在 MySQL 数据库中的现有内容。
	
3. 如果选择还原现有数据库，需要提供用户名和密码。如果选择还原到新数据库，需要提供新数据库名称：
	
	![还原到新 SQL 数据库][RestoreToNewSQLDB]
	
	单击向右箭头以继续。	
4. 如果选择创建新数据库，需要在下一个对话框中提供该数据库的凭据和其他初始配置信息。这里的示例显示的是一个新的 SQL 数据库。（用于新 MySQL 数据库的选项会有所不同。）
	
	![新 SQL 数据库设置][NewSQLDBConfig]
	
5. 单击复选标记以开始还原操作。操作完成时，在经典管理门户 Web 应用的列表中将能够看到新 Web 应用实例（如果那是你选择的还原选项）。
	
	![还原的 Contoso Web 应用][RestoredContoso Website]

<a name="OperationLogs"></a>
##查看操作日志
	
1. 要查看有关 Web 应用还原操作成功或失败的详细信息，请转到 Web 应用的“仪表板”选项卡。在“速览”部分的“管理服务”下，单击“操作日志”。
	
	![仪表板 - 操作日志链接][DashboardOperationLogsLink]
	
2. 你会跳转到管理服务经典管理门户的“操作日志”页面，并可在里面的操作日志列表中看到还原操作日志：
	
	![“管理服务操作日志”页面][ManagementServicesOperationLogsList]
	
3. 若要查看有关该操作的详细信息，请在列表中选择该操作，然后单击命令栏上的“详细信息”按钮。
	
	![“详细信息”按钮][DetailsButton]
	
	执行此操作时，“操作详细信息”窗口会打开，显示日志文件的可复制内容：
	
	![操作详细信息][OperationDetails]
	

<!-- IMAGES -->
[ChooseBackupSource]: ./media/web-sites-restore/01ChooseBackupSource.png
[ChooseRestoreNow]: ./media/web-sites-restore/02ChooseRestoreNow.png
[ViewContainers]: ./media/web-sites-restore/03ViewContainers.png
[StorageAccountFile]: ./media/web-sites-restore/02StorageAccountFile.png
[BrowseCloudStorage]: ./media/web-sites-restore/03BrowseCloudStorage.png
[StorageAccountFileSelected]: ./media/web-sites-restore/04StorageAccountFileSelected.png
[ChooseRestoreSettings]: ./media/web-sites-restore/05ChooseRestoreSettings.png
[ChooseDBServer]: ./media/web-sites-restore/06ChooseDBServer.png
[RestoreToNewSQLDB]: ./media/web-sites-restore/07RestoreToNewSQLDB.png
[NewSQLDBConfig]: ./media/web-sites-restore/08NewSQLDBConfig.png
[RestoredContoso Website]: ./media/web-sites-restore/09RestoredContosoWebSite.png
[DashboardOperationLogsLink]: ./media/web-sites-restore/10DashboardOperationLogsLink.png
[ManagementServicesOperationLogsList]: ./media/web-sites-restore/11ManagementServicesOperationLogsList.png
[DetailsButton]: ./media/web-sites-restore/12DetailsButton.png
[OperationDetails]: ./media/web-sites-restore/13OperationDetails.png

<!---HONumber=76-->