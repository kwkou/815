<properties
	pageTitle="配置 Azure 备份服务以准备备份 Windows Server | Windows Azure"
	description="使用本教程可了解如何在 Microsoft 的 Azure 云产品/服务中使用备份服务来将 Windows Server 备份到云中。"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor=""/>

<tags 
     ms.service="backup"
     ms.date="08/12/2015"
     wacn.date="09/15/2015"/>

# <span id="configure-a-backup-vault-tutorial"></span></a>配置 Azure 备份以便快速轻松地备份 Windows Server

<div class="dev-callout"> 
<strong>说明</strong>
 
<p>若要完成本教程，你需要一个 Azure 帐户。本教程将指导你完成启用 Azure 备份功能的整个过程。以前，若要注册备份服务器，你需要创建或获取一个 X.509 v3 证书。证书目前仍受支持，但是，为了简化将服务器注册到 Azure 保管库，你现在可以直接通过&ldquo;快速启动&rdquo;页生成保管库凭据。 </p>
<ul> 
<li>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 <a href="/pricing/1rmb-trial/">Azure 试用</a>。</li> 
 

</ul>
 

</div>

若要将文件和数据从 Windows Server 备份到 Azure，你必须在要存储数据的地理区域内创建一个备份保管库。本教程将指导你创建将用于存储备份的保管库、下载保管库凭据和安装备份代理，并概述了可通过管理门户完成的备份管理任务。

## <span id="create"></span></a>创建备份保管库

1.  登录到[管理门户][管理门户]。

2.  单击**“新建”**，依次指向**“数据服务”**和**“恢复服务”**，单击**“备份保管库”**，然后单击**“快速创建”**。

3.  在**“名称”**中，输入一个友好名称以标识此备份保管库。

4.  在**“区域”**中，为备份保管库选择地理区域。
    ![新备份保管库][新备份保管库]

5.  单击**“创建保管库”**。

    创建备份保管库可能需要一段时间。若要检查状态，你可以监视门户底部的通知。创建备份保管库后，将显示一条消息，告知你已成功创建保管库，并且该保管库将在恢复服务的资源中以**“活动”**状态列出。
    ![创建备份保管库][创建备份保管库]

6.  如果有多个订阅与你的组织帐户相关联，请选择要与备份保管库关联的正确帐户。

## <span id="upload"></span></a>下载保管库凭据

你可以不使用证书，而是使用保管库凭据向服务器注册 Azure 服务。你仍可以使用证书，不过，保管库凭据更易于使用，因为你可以使用 Azure 门户来生成和下载保管库凭据。

1.  登录到[管理门户][管理门户]。

2.  单击**“恢复服务”**，然后选择你要向其注册服务器的备份保管库。随后将显示该备份保管库的“快速启动”页。

3.  在“快速启动”页上，单击**“下载保管库凭据”**，以提示门户生成并下载你要用于向备份保管库注册服务器的保管库凭据。

4.  门户将使用保管库名称和当前日期的组合生成保管库凭据。单击**“保存”**将保管库凭据下载到本地帐户的 downloads 文件夹，或者从**“保存”**菜单中选择**“另存为”**以指定保管库凭据保存到的位置。你无法编辑保管库凭据，因此没有必要单击“打开”。下载凭据后，系统将提示你打开文件夹。单击**“x”**关闭此菜单。

## <span id="download"></span></a>下载和安装备份代理

1.  在[管理门户][管理门户]中操作。

2.  单击**“恢复服务”**，然后选择一个备份保管库以查看其“快速启动”页。

3.  在“快速启动”页上，选择你要下载的代理类型。可以选择**“下载 Azure Backup Agent”**、**“Windows Server and System Center Data Protection Manager”**或**“Windows Server Essentials”**。有关详细信息，请参阅：

    -   [安装 Azure Backup Agent for Windows Server 2012 and System Center 2012 SP1 - Data Protection Manager][安装 Azure Backup Agent for Windows Server 2012 and System Center 2012 SP1 - Data Protection Manager]
    -   [安装 Azure Backup Agent for Windows Server 2012 Essentials][安装 Azure Backup Agent for Windows Server 2012 Essentials]

安装代理后，你可使用适当的本地管理接口（如 Microsoft 管理控制台管理单元、System Center Data Protection Manager 控制台或 Windows Server Essentials 仪表板）配置服务器的备份策略。

## <span id="manage"></span></a>管理备份保管库和服务器

1.  登录到[管理门户][管理门户]。

2.  单击**“恢复服务”**，然后单击备份保管库的名称以查看“快速启动”页。

3.  单击**“仪表板”**以查看服务器的使用概览。在“仪表板”的底部，可以执行以下任务：

    -   **管理证书**。如果使用证书注册了服务器，请使用此选项来更新证书。如果使用的是保管库凭据，请不要使用**“管理证书”**。
    -   **删除**。删除当前备份保管库。如果不再使用备份保管库，则可将其删除以释放存储服务。仅在从保管库中删除所有注册的服务器后启用**“删除”**。
    -   **保管库凭据**。使用此“速览”菜单项可以配置保管库凭据。

4.  单击**“受保护的项”**可查看已从服务器备份的项。此列表仅供参考。

    ![受保护的项][受保护的项]

5.  单击**“服务器”**可查看注册到此保管库的服务器的名称。可从该处执行以下任务：

    -   **允许重新注册**。在为服务器选择该选项时，可使用代理中的注册向导再一次将服务器注册到备份保管库。由于证书中存在错误或者如果必须重新构建服务器，你可能需要重新注册。每个服务器名称只能重新注册一次。
    -   **删除**。从备份保管库中删除服务器。将立即删除与服务器关联的所有已存储数据。

        ![删除的服务器][删除的服务器]

## <span id="next"></span></a>后续步骤

-   若要了解有关 Azure 备份的详细信息，请参阅 [Azure 备份概述][Azure 备份概述]。

-   访问 [Azure 备份论坛][Azure 备份论坛]。

  [Azure 免费试用]: /pricing/1rmb-trial/
  [管理门户]: https://manage.windowsazure.cn
  [新备份保管库]: http://i.imgur.com/506c7ch.png
  [创建备份保管库]: http://i.imgur.com/grtLcKM.png
  [安装 Azure Backup Agent for Windows Server 2012 and System Center 2012 SP1 - Data Protection Manager]: http://technet.microsoft.com/zh-cn/library/hh831761.aspx#BKMK_installagent
  [安装 Azure Backup Agent for Windows Server 2012 Essentials]: http://technet.microsoft.com/zh-cn/library/jj884318.aspx
  [受保护的项]: ./media/backup-configure-vault/RS_protecteditems.png
  [删除的服务器]: ./media/backup-configure-vault/RS_deletedserver.png
  [Azure 备份概述]: https://technet.microsoft.com/zh-CN/library/hh831419.aspx
  [Azure 备份论坛]: http://social.msdn.microsoft.com/Forums/zh-CN/windowsazureonlinebackup/threads
