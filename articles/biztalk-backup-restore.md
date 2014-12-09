<properties linkid="biztalk-backup-restore" urlDisplayName="BizTalk Services: Backup and Restore" pageTitle="BizTalk 服务：备份和还原 | Azure" metaKeywords="" description="BizTalk 服务包括备份和还原功能。在创建备份时，将拍下 BizTalk 服务配置的快照。" metaCanonical="" services="" documentationCenter="" title="BizTalk 服务：备份和还原" authors="mandia"  solutions="" writer="mandia" manager="paulettm" editor="cgronlun"  />

# BizTalk 服务：备份和还原

Azure BizTalk 服务包括备份和还原功能。本主题介绍如何使用 Azure 管理门户备份和还原 BizTalk 服务，其中包括：

-   [开始之前][开始之前]
-   [创建备份][创建备份]
-   [还原备份][还原备份]
-   [备份的内容][备份的内容]

您还可以使用 [BizTalk 服务 REST API][BizTalk 服务 REST API] 备份 BizTalk 服务。

## <a name="beforebackup"></a>开始之前

-   备份和还原可能不适用于所有版本。请参阅 [BizTalk 服务：版本图表][BizTalk 服务：版本图表]。

    **注**混合连接不会备份，不管是什么版本。

-   使用 Azure 管理门户，可以创建按需备份或创建计划的备份。

-   备份内容可以还原到同一个 BizTalk 服务或者新的 BizTalk 服务。若要使用相同名称还原 BizTalk 服务，必须删除现有的 BizTalk 服务，否则，还原将失败。

-   BizTalk 服务可以还原到相同版本或更高版本。不支持从执行备份时还原到更低版本的 BizTalk 服务。

    例如，使用基本版的备份可以还原到高级版。使用高级版的备份不能还原到标准版。

-   对 EDI 控制编号进行备份以便保持控制编号的连续性。如果消息是在最后的备份后处理的，则还原此备份内容可能会导致重复的控制编号。

-   如果批含有活动消息，请在运行备份**之前**处理该批。在创建备份（根据需要或计划）时，永远不会存储批中的消息。

    **如果对某一批次中的活动消息执行备份，将不会备份这些消息并且因此这些消息将丢失。**

-   可选：在 BizTalk 服务门户中，停止任何管理操作。

## <a name="createbu"></a>创建备份

备份可以随时进行并由您完全控制。本部分列出了使用 Azure 管理门户创建备份的步骤，包括：

[按需备份][按需备份]

[计划备份][计划备份]

#### <a name="backupnow"></a>按需备份

1.  在 Azure 管理门户中，选择 **BizTalk 服务**，然后选择要备份的 BizTalk 服务。
2.  在**仪表板**选项卡上，选择页面底部的**备份**。
3.  输入备份名称。例如，输入 *myBizTalkService*BU*Date*。
4.  选择 blob 存储帐户并选择复选标记以开始备份。

完成备份后，在存储帐户中创建一个使用您输入的备份名称的容器。此容器包含您的 BizTalk 服务备份配置。

#### <a name="backupschedule"></a>计划备份

1.  在 Azure 管理门户中，选择 **BizTalk 服务**，选择您要计划备份的 BizTalk 服务名称，然后选择**配置**选项卡。
2.  将**备份状态**设置为**自动**。
3.  选择**存储帐户**以存储备份，输入**频率**创建备份，以及保留备份的时间长度（**保留天数**）：

    ![][0]

    **说明**

-   在**保留天数**中，保持期必须大于备份频率。
-   选择**始终至少保留一个备份**，即使它过了保持期。

1.  选择**保存**。

当计划的备份作业运行时，它将在您指定的存储帐户中创建一个容器（用于存储备份数据）。容器的名称为 *BizTalk Service Name-date-time*。

如果 BizTalk 服务仪表板显示**失败**状态：

![上一次计划的备份状态][上一次计划的备份状态]

该链接将打开“管理服务操作日志”以帮助故障排除。请参阅 [BizTalk 服务：使用操作日志进行故障排除][BizTalk 服务：使用操作日志进行故障排除]。

## <a name="restore"></a>还原

您可以从 Azure 管理门户或从[还原 BizTalk 服务 REST API][还原 BizTalk 服务 REST API] 还原备份。本部分列出使用管理门户还原的步骤。

#### 还原备份前

-   还原 BizTalk 服务时，可以指定新的跟踪、存档和监视存储。

-   要还原使用相同名称的 BizTalk 服务，请在开始进行还原之前删除现有的 BizTalk 服务。否则，还原将失败。

-   还原相同的 EDI 运行时数据。EDI 运行时备份存储控制编号。还原的控制编号采用备份时的顺序。如果消息是在最后的备份后处理的，则还原此备份内容可能会导致重复的控制编号。

#### 还原备份

1.  在 Azure 管理门户中，选择**新建** \> **应用程序服务** \> **BizTalk 服务** \> **还原**：

    ![还原备份][1]

2.  在**备份 URL** 中，选择文件夹图标，然后展开存储 BizTalk 服务配置备份的 Azure 存储帐户。展开容器，然后在右窗格中选择相应的备份 .txt 文件。
    选择**打开**。

3.  在**还原 BizTalk 服务**页面上，为还原的 BizTalk 服务输入 **BizTalk 服务名称**并验证**域 URL**、**版本**和**区域**。**创建一个新的 SQL 数据库实例**用于跟踪数据库：

    ![][2]

    选择“下一步”箭头。

4.  验证 SQL 数据库的名称，输入将在其中创建 SQL 数据库的物理服务器，以及该服务器的用户名/密码。

    如果您想要配置 SQL 数据库版本、大小和其他属性，请选择**配置高级数据库设置**。

    选择“下一步”箭头。

5.  创建新的存储帐户，或者为 BizTalk 服务指定现有的存储帐户。

6.  单击复选标记以开始恢复。

一旦还原成功完成，新的 BizTalk 服务将列出在 Azure 管理门户的 BizTalk 服务页面上的挂起状态中。

### <a name="postrestore"></a>还原备份后

BizTalk 服务始终在**挂起**状态中还原。在此状态下，您可在新环境正常运行前进行任何配置更改，其中包括：

-   如果您使用 Azure BizTalk 服务 SDK 创建 BizTalk 服务应用程序，可能需要更新这些应用程序中的 ACS 凭据以使用还原的环境。

-   还原 BizTalk 服务以复制现有的 BizTalk 服务环境。在此情况中，如果有在使用源 FTP 文件夹的原始 BizTalk 服务门户中配置的协议，您可能需要在新还原的环境中更新协议，以使用不同的源 FTP 文件夹。否则，可能有两个不同的协议尝试提取同一条消息。

-   如果还原以具有多个 BizTalk 服务环境，请确保针对 Visual Studio 应用程序、PowerShell cmdlet、REST Api 或贸易合作伙伴管理 OM API 中正确的环境。

-   最好在新还原的 BizTalk 服务环境中配置自动化备份。

要在 Azure 管理门户中启动 BizTalk 服务，选择还原的 BizTalk 服务，然后选择任务栏中的**恢复**。

## <a name="budata"></a>备份的内容

在创建某一备份时，将备份以下项目：

<table border="1">
<tr bgcolor="FAF9F9">
<th>
</th>
<th>
备份的项目

</th>
</tr>
<tr>
<td colspan="2">
**Azure BizTalk 服务门户**

</td>
</tr>
<tr>
<td>
配置和运行时

</td>
<td>
-   合作伙伴和配置文件详细信息
-   合作伙伴协议
-   部署的自定义程序集
-   部署的桥
-   证书
-   部署的转换
-   管道
-   在 BizTalk 服务门户中创建和保存的模板
-   X12 ST01 和 GS01 映射
-   控制编号 (EDI)
-   AS2 消息 MIC 值

</td>
</tr>
<tr>
<td colspan="2">
**Azure BizTalk 服务**

</td>
</tr>
<tr>
<td>
SSL 证书

</td>
<td>
-   SSL 证书数据
-   SSL 证书密码

</td>
</tr>
<tr>
<td>
BizTalk 服务设置

</td>
<td>
-   缩放单位计数
-   版本
-   产品版本
-   区域/数据中心
-   Access Control 服务 (ACS) 命名空间和密钥
-   跟踪数据库连接字符串
-   存档存储帐户连接字符串
-   监视存储帐户连接字符串

</td>
</tr>
<tr>
<td colspan="2">
**其他项**

</td>
</tr>
<tr>
<td>
跟踪数据库

</td>
<td>
在创建 BizTalk 服务时，将输入跟踪数据库详细信息，包括 Azure SQL Database Server 和跟踪数据库名称。不会自动备份跟踪数据库。
 **重要说明**
 如果删除跟踪数据库，并且需要恢复该数据库，必须存在上一个备份。如果某一备份不存在，则跟踪数据库及其数据将无法恢复。在此情况下，应使用相同数据库名称创建新的跟踪数据库。建议执行地域复制。

</td>
</tr>
</table>
## 下一步

要在 Azure 管理门户中创建 Azure BizTalk 服务，请转到 [BizTalk 服务：使用 Azure 管理门户进行设置][BizTalk 服务：使用 Azure 管理门户进行设置]。若要开始创建应用程序，请转到 [Azure BizTalk 服务][Azure BizTalk 服务]。

## 另请参阅

-   [备份 BizTalk 服务][BizTalk 服务 REST API]
-   [从备份还原 BizTalk 服务][还原 BizTalk 服务 REST API]
-   [BizTalk 服务：开发人员版、基本版、标准版和高级版图表][BizTalk 服务：开发人员版、基本版、标准版和高级版图表]
-   [BizTalk 服务：使用 Azure 管理门户进行设置][BizTalk 服务：使用 Azure 管理门户进行设置]
-   [BizTalk 服务：设置状态图表][BizTalk 服务：设置状态图表]
-   [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡][BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]
-   [BizTalk 服务：限制][BizTalk 服务：限制]
-   [BizTalk 服务：颁发者名称和颁发者密钥][BizTalk 服务：颁发者名称和颁发者密钥]
-   [如何开始使用 Azure BizTalk 服务 SDK][如何开始使用 Azure BizTalk 服务 SDK]

  [开始之前]: #beforebackup
  [创建备份]: #createbu
  [还原备份]: #restore
  [备份的内容]: #budata
  [BizTalk 服务 REST API]: http://go.microsoft.com/fwlink/p/?LinkID=325584
  [BizTalk 服务：版本图表]: http://azure.microsoft.com/zh-cn/documentation/articles/biztalk-editions-feature-chart/
  [按需备份]: #backupnow
  [计划备份]: #backupschedule
  [0]: ./media/biztalk-backup-restore/AutomaticBU.png
  [上一次计划的备份状态]: ./media/biztalk-backup-restore/status-last-backup.png
  [BizTalk 服务：使用操作日志进行故障排除]: http://go.microsoft.com/fwlink/?LinkId=391211
  [还原 BizTalk 服务 REST API]: http://go.microsoft.com/fwlink/p/?LinkID=325582
  [1]: ./media/biztalk-backup-restore/restore-ui.png
  [2]: ./media/biztalk-backup-restore/RestoreBizTalkServiceWindow.png
  [BizTalk 服务：使用 Azure 管理门户进行设置]: http://go.microsoft.com/fwlink/p/?LinkID=302280
  [Azure BizTalk 服务]: http://go.microsoft.com/fwlink/p/?LinkID=235197
  [BizTalk 服务：开发人员版、基本版、标准版和高级版图表]: http://go.microsoft.com/fwlink/p/?LinkID=302279
  [BizTalk 服务：设置状态图表]: http://go.microsoft.com/fwlink/p/?LinkID=329870
  [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]: http://go.microsoft.com/fwlink/p/?LinkID=302281
  [BizTalk 服务：限制]: http://go.microsoft.com/fwlink/p/?LinkID=302282
  [BizTalk 服务：颁发者名称和颁发者密钥]: http://go.microsoft.com/fwlink/p/?LinkID=303941
  [如何开始使用 Azure BizTalk 服务 SDK]: http://go.microsoft.com/fwlink/p/?LinkID=302335
