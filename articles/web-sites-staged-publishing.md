<properties
	pageTitle="为 Azure 网站中的 Web 应用设置过渡环境"
	description="了解如何对 Azure 网站中的 Web 应用使用分阶段发布。"
	services="app-service\web"
	documentationCenter=""
	authors="cephalin"
	writer="cephalin"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="app-service-web"
	ms.date="09/16/2015"
	wacn.date="11/02/2015"/>

# 为 Azure 网站设置过渡环境 #
<a name="Overview"></a>

将 Web 应用部署到 Azure 网站时，如果在“标准”Azure 网站计划模式下运行，则你可以部署到单独的部署槽而不是默认的生产槽。部署槽实际上是具有自身主机名的实时 Azure 网站。两个部署槽（包括生产槽）之间的 Azure 网站内容与配置元素可以交换。将应用程序部署到部署槽具有以下优点：

- 您可以在分阶段部署槽中验证网站更改，然后将其与生产槽交换。

- 首先将 Web 应用部署到槽，然后将其交换到生产，这确保槽的所有实例都预热，然后交换到生产。部署你的 Web 应用时，这消除了停机时间。流量重定向是无缝的，且不会因交换操作而删除任何请求。当不需要预交换验证时，可以通过配置[自动交换](#configure-auto-swap-for-your-web-app)来自动化这整个工作流。
- 交换后，具有以前分阶段站点的槽现在具有以前的生产站点。如果交换到生产槽的更改与您的预期不同，您可以立即执行同一交换来收回“上一已知的良好站点”。 
 
每种 Azure 网站计划模式支持不同数量的部署槽。

- 如果你的 Web 应用具有多个槽，则你无法更改模式。

- 缩放不适用于非生产槽。

- 非生产槽不支持链接的资源管理。


<a name="Add"></a>
## 将部署槽添加到 Web 应用 ##

网站必须在“标准”或“高级”模式下运行才能启用多个部署槽。

1. 在“快速启动”页面或网站“仪表板”页面的“速览”部分中，单击“添加新部署槽”。 
	
	![添加新部署槽][QGAddNewDeploymentSlot]
	
	> [AZURE.NOTE]
	> 如果网站不在“标准”模式下，你将收到消息“你必须在标准模式下才能启用过渡发布”。此时，你可以选择“升级”，导航到网站的“缩放”选项卡，然后继续。
	
2. 在“添加新的部署槽”对话框中，为槽提供一个名称，并选择是否要从另一个现有部署槽克隆网站配置。单击复选标记以继续。
	
	![配置源][ConfigurationSource1]
	
	第一次，创建一个插槽，您只有两种选择：从生产中的默认插槽克隆配置或者完全不进行克隆。
	
	创建多个插槽后，您将能够从生产槽以外的槽克隆配置：
	
	![配置源][MultipleConfigurationSources]

3. 在网站列表中，展开网站名称左侧的标记以显示部署槽。它将具有网站名称，然后是部署槽名称。
	
	![带有部署槽的站点列表][SiteListWithStagedSite]
	
4. 当你单击部署站点槽名称时，将打开一个包含一组选项卡的页面，和其他网站一样。<strong><i>你的网站名称</i>(<i>部署槽名称</i>)</strong>将显示在门户页面顶部，提醒你正在查看部署站点槽。
	
	![部署槽标题][StagingTitle]
	
5. 单击仪表板视图中的站点 URL。请注意部署槽具有其自己的主机名，也是活动站点。若要限制对部署槽的公共访问权限，请参阅 [Azure 网站 – 阻止对非生产部署槽的 Web 访问](http://ruslany.net/2014/04/azure-web-sites-block-web-access-to-non-production-deployment-slots/)。

	
没有任何内容。您可以从其他存储库分支或完全不同的存储库部署到槽。您还可以更改此槽的配置。使用与内容更新部署槽关联的发布配置文件或部署凭证。例如，你可以[使用 git 发布到此槽](/documentation/articles/web-sites-publish-source-control/)。

<a name="AboutConfiguration"></a>
## 部署槽的配置##
从另一个部署槽克隆配置时，可以编辑克隆的配置。此外，某些配置元素在交换时遵循内容（不特定于位置），而其他配置元素将在交换之后保留在同一个位置（特定于位置）。以下列表显示交换槽时将更改的配置。

**交换的设置**：
- 常规设置 - 例如 Framework 版本、32/64 位、Web 套接字
- 连接字符串（可以配置为停在槽中
- 处理程序映射
- 监视与诊断设置
- Web 作业内容

**不交换的设置**：

- 发布终结点
- 自定义域名
- SSL 证书和绑定
- 缩放设置
- Web 作业计划程序

若要将应用设置或连接字符串配置为停在某个槽中（不交换），请访问特定槽的“应用程序设置”边栏选项卡，然后针对应该停在该槽中的配置元素选中“槽设置”框。请注意，将配置元素标记为特定于槽会在将该元素建立为无法跨所有与该 Web 应用关联的部署槽进行交换时产生影响。

![槽设置][SlotSettings]

<a name="Swap"></a>
## 交换部署槽的步骤 ##

>[AZURE.IMPORTANT] 在将 Web 应用从部署槽交换到生产槽之前，请确保所有非槽特定的设置已完全根据你希望它在交换目标中的位置明确地进行配置。

1. 若要交换部署槽，请在 Web 应用命令栏或部署槽命令栏中单击“交换”按钮。确保正确设置交换源和交换目标。交换目标通常是生产槽。  

	![“交换”按钮][SwapButtonBar]

3. 单击“确定”完成操作。操作完成后，即已交换部署槽。

<a name="Rollback"></a>
## 在交换后回滚生产应用的步骤 ##
如果在槽交换后在生产中发现任何错误，请立即通过交换相同的两个槽来将槽回滚到交换前状态。

<a name="Delete"></a>
## 删除部署槽##

在部署槽的边栏选项卡中，单击命令栏上的“删除”。

![删除部署槽][DeleteStagingSiteButton]

<!-- ======== AZURE POWERSHELL CMDLETS =========== -->

<a name="PowerShell"></a>
## 适用于部署槽的 Azure PowerShell cmdlet

Azure PowerShell 是一个模块，可提供通过 Windows PowerShell 管理 Azure 的 cmdlet，包括对管理 Azure 网站部署槽的支持。

- 有关安装和配置 Azure PowerShell 的信息以及使用 Windows Azure 订阅对 Azure PowerShell 进行身份验证的信息，请参阅[如何安装和配置 Windows Azure PowerShell](/documentation/articles/install-configure-powershell)。  

- 若要列出 PowerShell 中可用于 Azure 网站的 cmdlet，请调用 `help Azure Website`。

----------

###Get-Azure Website
**Get-AzureWebsite** cmdlet 提供当前订阅中有关 Azure Web 应用的信息，如以下示例所示。

`Get-AzureWebsite webappslotstest`

----------

###New-Azure Website
可以通过使用 **New-AzureWebsite** cmdlet 并指定 Web 应用和槽的名称来创建部署槽。还表示与用于创建部署槽的 Web 应用相同的区域，如以下示例所示。

`New-Azure Website siteslotstest –Slot staging –Location “China East”`

----------

###Publish-Azure WebsiteProject
可以使用 **Publish-Azure WebsiteProject** cmdlet 进行内容部署，如以下示例所示。

`Publish-AzureWebsiteProject -Name webappslotstest -Slot staging -Package [path].zip`

----------

###Show-Azure Website
内容和配置更新应用到新的槽之后，你可以通过使用 **Show-Azure Website** cmdlet 浏览到该槽，对更新进行验证，如以下示例所示。

`Show-AzureWebsite -Name webappslotstest -Slot staging`

----------

### Switch-AzureWebsiteSlot
**Switch-AzureWebsiteSlot** cmdlet 可以执行交换操作，使更新的部署槽成为生产站点，如以下示例所示。生产应用将不会停机，也不会进行冷启动。

`Switch-AzureWebsiteSlot -Name webappslotstest`

----------

###Remove-Azure Website
如果不再需要部署槽，则可以使用 **Remove-Azure Website** cmdlet 删除该部署槽，如以下示例所示。

`Remove-AzureWebsite -Name webappslotstest -Slot staging`

----------

<!-- ======== Azure CLI =========== -->

<a name="CLI"></a>
## 用于部署槽的 Azure 命令行界面 (Azure CLI) 命令

Azure CLI 提供了适用于 Azure 的跨平台命令，包括对 Web 应用部署槽的管理支持。

- 有关安装和配置 Azure CLI 的说明（包括有关如何将 Azure CLI 连接到 Azure 订阅的信息），请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli)。

-  若要在 Azure CLI 中列出可用于 Azure 网站的命令，请调用 `azure site -h`。

----------
### azure site list
有关当前订阅中 Web 应用的相关信息，请调用 **azure site list**，如以下示例所示。

`azure site list webappslotstest`

----------
### azure site create
若要创建部署槽，请调用 **azure site create**，并指定现有 Web 应用的名称和要创建的槽的名称，如以下示例所示。

`azure site create webappslotstest --slot staging`

若要启用新槽源代码管理，请使用 **--git** 选项，如以下示例所示。

`azure site create --git webappslotstest --slot staging`

----------
### azure site swap
若要使更新的部署槽成为生产应用，请使用 **azure site swap** 命令执行交换操作，如以下示例所示。生产应用将不会停机，也不会进行冷启动。

`azure site swap webappslotstest`

----------
### azure site delete
若要删除不再需要的部署槽，请使用 **azure site delete** 命令，如以下示例所示。

`azure site delete webappslotstest --slot staging`

----------
## 后续步骤 ##
[Azure 网站 – 阻止对非生产部署槽的 Web 访问](http://ruslany.net/2014/04/azure-web-sites-block-web-access-to-non-production-deployment-slots/)

[Windows Azure 免费试用版](/zh-cn/pricing/1rmb-trial/)


<!-- IMAGES -->
[QGAddNewDeploymentSlot]: ./media/web-sites-staged-publishing/QGAddNewDeploymentSlot.png
[AddNewDeploymentSlotDialog]: ./media/web-sites-staged-publishing/AddNewDeploymentSlotDialog.png
[ConfigurationSource1]: ./media/web-sites-staged-publishing/ConfigurationSource1.png
[MultipleConfigurationSources]: ./media/web-sites-staged-publishing/MultipleConfigurationSources.png
[SiteListWithStagedSite]: ./media/web-sites-staged-publishing/SiteListWithStagedSite.png
[StagingTitle]: ./media/web-sites-staged-publishing/StagingTitle.png
[SwapButtonBar]: ./media/web-sites-staged-publishing/SwapButtonBar.png
[SwapConfirmationDialog]: ./media/web-sites-staged-publishing/SwapConfirmationDialog.png
[DeleteStagingSiteButton]: ./media/web-sites-staged-publishing/DeleteStagingSiteButton.png
[SwapDeploymentsDialog]: ./media/web-sites-staged-publishing/SwapDeploymentsDialog.png
[Autoswap1]: ./media/web-sites-staged-publishing/AutoSwap01.png
[Autoswap2]: ./media/web-sites-staged-publishing/AutoSwap02.png
[SlotSettings]: ./media/web-sites-staged-publishing/SlotSetting.png
 

<!---HONumber=76-->