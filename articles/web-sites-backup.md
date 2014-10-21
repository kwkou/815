<properties linkid="web-sites-backup" urlDisplayName="Azure Web Sites Backups" pageTitle="Azure Web Sites Backups" metaKeywords="Azure Web Sites, Backups" description="Learn how to create backups of your Azure web sites." metaCanonical="" services="web-sites" documentationCenter="" title="Azure Web Sites Backups" authors="timamm" solutions="" manager="paulettm" editor="mollybos" />

# Azure 网站备份

利用 Azure 网站备份和还原功能，你可以轻松地手动或自动创建网站备份。你可以将网站还原到以前的状态，或者基于原始网站的备份之一创建新网站。

有关从备份中还原 Azure 网站的信息，请参阅[还原 Azure 网站][还原 Azure 网站]。

## 本文内容

-   [备份的内容][备份的内容]
-   [要求和限制][要求和限制]
-   [创建手动备份][创建手动备份]
-   [配置自动化的备份][配置自动化的备份]
-   [如何存储备份][如何存储备份]
-   [说明][说明]
-   [后续步骤][后续步骤]

    -   [有关存储帐户的详细信息][有关存储帐户的详细信息]

<a name="whatsbackedup"></a>

## 备份的内容

Azure 网站备份以下信息：

-   网站配置
-   网站文件内容
-   任何连接到你的网站的 SQL Server 或 MySQL 数据库（你可以选择备份中要包括哪些数据库）

这些信息会备份到你指定的 Azure 存储帐户。

> [WACOM.NOTE] 每个备份都是你的网站的完整脱机副本，而不是增量更新。

<a name="requirements"></a>

## 要求和限制

-   备份和还原功能要求网站为“标准”级别。有关使用“标准”级来缩放网站的详细信息，请参阅[如何缩放网站][如何缩放网站]。

-   备份和还原功能要求 Azure 存储帐户与要备份的网站必须同属一个订阅。如果你还没有存储帐户，可以创建一个，方法是：单击 Azure 门户左窗格中的“存储”按钮（网格图标），然后选择底部命令栏中的“新建”。有关 Azure 存储帐户的详细信息，请参阅本文结尾处的[链接][有关存储帐户的详细信息]。

<a name="manualbackup"></a>

## 创建手动备份

1.  在网站的 Azure 门户中，选择“备份”选项卡。

    ![“备份”页面][“备份”页面]

2.  选择网站要备份到其中的存储帐户。该存储帐户必须与要备份的网站同属一个订阅。

    ![选择存储帐户][选择存储帐户]

3.  在“包含的数据库”选项中，选择要备份的连接到你的网站的数据库（SQL Server 或 MySQL）。

    ![选择要包含的数据库][选择要包含的数据库]

    > [WACOM.NOTE] 若要使数据库显示在此列表中，其连接字符串必须存在于门户的“配置”选项卡的“连接字符串”部分中。

4.  在命令栏上，单击“立即备份”。

    ![BackUpNow 按钮][BackUpNow 按钮]

    在备份过程中你将看到进度消息：

    ![备份进度消息][备份进度消息]

你可以随时进行手动备份。在预览版期间，每 24 小时内的手动备份不能超过 2 个（此限制可能会变更）。

<a name="automatedbackups"></a>

## 配置自动化的备份

1.  在“备份”页面上，将“自动化的备份”设置为“开”。

    ![启用自动化的备份][启用自动化的备份]

2.  选择网站要备份到其中的存储帐户。该存储帐户必须与要备份的网站同属一个订阅。

    ![选择存储帐户][选择存储帐户]

3.  在“频率”框中，指定你希望多久进行一次自动化的备份。（在预览版期间，天数是唯一可用的时间单位。）

    ![选择备份频率][选择备份频率]

    天数必须介于 1 到 90 之间（含 1 和 90，从每天 1 次到每 90 天一次）。

4.  使用“开始日期”选项指定你希望自动化的备份开始的日期和时间。

    ![选择开始日期][选择开始日期]

    时间以半小时为增量。

    ![选择开始时间][选择开始时间]

    > [WACOM.NOTE] Azure 存储备份时间时采用的是 UTC 格式，但显示时根据的是用于显示门户的计算机上的系统时间。

5.  在“包含的数据库”部分中，选择要备份的连接到你的网站的数据库（SQL Server 或 MySQL）。若要使数据库显示在列表中，其连接字符串必须位于门户中“配置”选项卡的**“连接字符串”**部分。

    ![选择要包含的数据库][选择要包含的数据库]

    > [WACOM.NOTE] 如果你选择在备份中包含一个或多个数据库，并且指定了小于 7 天的频率，就会收到频繁备份会增加数据库成本的警告。

6.  在命令栏中，单击“保存”按钮以保存你的配置更改（或者，如果你决定不保存，请选择“放弃”）。

    ![“保存”按钮][“保存”按钮]

<a name="aboutbackups"></a>

## 如何存储备份

在完成了一个或多个备份后，在你的存储帐户的“容器”选项卡中可以看到这些备份。你的备份将在名为“websitebackups”的容器中。每个备份都由一个 .zip 文件和一个 .xml 文件组成，前者包含备份的数据，后者包含 .zip 文件内容的清单。

.zip 和 .xml 备份文件名中包含网站的名称，后面跟有下划线和备份生成时间的时间戳。时间戳包含 YYYYMMDD（数字，无空格）格式的日期加上 UTC 格式的 24 小时制时间（例如，fabrikam\_201402152300.zip）。如果你想在不实际执行网站还原的情况下访问备份，可将这些文件的内容解压缩，然后进行浏览。

> [WACOM.NOTE] 改动“websitebackups”容器中的任何文件都会导致备份无效，进而无法还原。

<a name="notes"></a>

## 说明

-   一定要在网站的“配置”选项卡上为你的每个数据库正确设置连接字符串，这样“备份和还原”功能才会包括这些数据库。
-   在预览版期间，你负责管理保存到你的存储帐户的备份的内容。如果你从存储帐户删除一个备份，且未在其他位置制作副本，则无法在以后还原该备份。
-   虽然你可以将多个网站备份到同一存储帐户，但是，为了易于维护，还是为每个网站都单独创建存储帐户比较好。
-   在预览版期间，备份和还原操作只能通过 Azure 管理门户完成。

<a name="nextsteps"></a>

## 后续步骤

有关从备份中还原 Azure 网站的信息，请参阅[还原 Azure 网站][1]。

若要开始使用 Azure，请参阅 [Microsoft Azure 免费试用版][Microsoft Azure 免费试用版]。

<a name="moreaboutstorage"></a>

### 有关存储帐户的详细信息

[什么是存储帐户？][什么是存储帐户？]

[如何：创建存储帐户][如何：创建存储帐户]

[如何监视存储帐户][如何监视存储帐户]

[了解 Winidows Azure 存储计费][了解 Winidows Azure 存储计费]

<!-- IMAGES -->

  [还原 Azure 网站]: http://www.windowsazure.cn/zh-cn/documentation/articles/web-sites-restore/
  [备份的内容]: #whatsbackedup
  [要求和限制]: #requirements
  [创建手动备份]: #manualbackup
  [配置自动化的备份]: #automatedbackups
  [如何存储备份]: #aboutbackups
  [说明]: #notes
  [后续步骤]: #nextsteps
  [有关存储帐户的详细信息]: #moreaboutstorage
  [如何缩放网站]: http://www.windowsazure.com/en-us/documentation/articles/web-sites-scale/
  [“备份”页面]: ./media/web-sites-backup/01ChooseBackupsPage.png
  [选择存储帐户]: ./media/web-sites-backup/02ChooseStorageAccount.png
  [选择要包含的数据库]: ./media/web-sites-backup/03IncludedDatabases.png
  [BackUpNow 按钮]: ./media/web-sites-backup/04BackUpNow.png
  [备份进度消息]: ./media/web-sites-backup/05BackupProgress.png
  [启用自动化的备份]: ./media/web-sites-backup/06SetAutomatedBackupOn.png
  [选择备份频率]: ./media/web-sites-backup/07Frequency.png
  [选择开始日期]: ./media/web-sites-backup/08StartDate.png
  [选择开始时间]: ./media/web-sites-backup/09StartTime.png
  [“保存”按钮]: ./media/web-sites-backup/10SaveIcon.png
  [1]: http://www.windowsazure.com/en-us/documentation/articles/web-sites-restore/
  [Microsoft Azure 免费试用版]: http://azure.microsoft.com/en-us/pricing/free-trial/
  [什么是存储帐户？]: http://www.windowsazure.com/en-us/documentation/articles/storage-whatis-account/
  [如何：创建存储帐户]: http://www.windowsazure.com/en-us/documentation/articles/storage-create-storage-account/
  [如何监视存储帐户]: http://www.windowsazure.com/en-us/documentation/articles/storage-monitor-storage-account/
  [了解 Winidows Azure 存储计费]: http://blogs.msdn.com/b/windowsazurestorage/archive/2010/07/09/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity.aspx
