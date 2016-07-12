<properties
	pageTitle="为 Azure 中的 Web 应用设置过渡环境"
	description="了解如何对 Azure 中的 Web 应用使用分阶段发布。"
	services="app-service"
	documentationCenter=""
	authors="cephalin"
	writer="cephalin"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="app-service"
	ms.date="03/09/2016"
	wacn.date="04/18/2016"/>

# 为 Azure 中的 Web 应用设置过渡环境
<a name="Overview"></a>

将 Web 应用部署到 [Azure Web 应用](/documentation/services/web-sites/)时，如果在“标准”App Service 计划模式下运行，则你可以部署到单独的部署槽而不是默认的生产槽。部署槽实际上是具有自身主机名的实时 Web 应用。两个部署槽（包括生产槽）之间的 Web 应用内容与配置元素可以交换。将应用程序部署到部署槽具有以下优点：

- 你可以在分阶段部署槽中验证 Web 应用更改，然后将其与生产槽交换。

- 首先将 Web 应用部署到槽，然后将其交换到生产，这确保槽的所有实例都预热，然后交换到生产。部署你的 Web 应用时，这消除了停机时间。流量重定向是无缝的，且不会因交换操作而删除任何请求。

- 交换后，具有以前分阶段 Web 应用的槽现在具有以前的生产 Web 应用。如果交换到生产槽的更改与您的预期不同，您可以立即执行同一交换来收回“上一已知的良好站点”。

仅“标准”App Service 计划模式支持分阶段部署。若要缩放你的 Web 应用，请参阅[在 Azure 中缩放 Web 应用](/documentation/articles/web-sites-scale/)。

- 如果你的 Web 应用具有多个槽，则你无法更改模式。

- 缩放不适用于非生产槽。

- 非生产槽不支持链接的资源管理。在 [Azure 经典管理门户](https://manage.windowsazure.cn/)中，你可以通过暂时将非生产槽移到其他 App Service 计划模式来避免对生产槽造成这种潜在影响。请注意，非生产槽必须先再次与生产槽共享相同的模式，才能交换这两个槽。


<a name="Add"></a>
## 将部署槽添加到 Web 应用 ##

Web 应用必须在“标准”模式下运行才能启用多个部署槽。

1. 在“快速启动”页面或 Web 应用“仪表板”页面的“速览”部分，单击“添加新部署槽”。 
	
	![添加新部署槽][QGAddNewDeploymentSlot]
	
	> [AZURE.NOTE]
	如果 Web 应用不在“标准”模式下，你将收到“必须在标准模式下才能启用分阶段发布”消息。此时，你可以选择“升级”，并导航到 Web 应用的“缩放”选项卡，然后再继续。
	
2. 在“添加新部署槽”对话框中，为槽提供一个名称，并选择是否要从另一个现有部署槽克隆 Web 应用配置。单击复选标记以继续。
	
	![配置源][ConfigurationSource1]
	
	第一次，创建一个插槽，您只有两种选择：从生产中的默认插槽克隆配置或者完全不进行克隆。
	
	创建多个插槽后，您将能够从生产槽以外的槽克隆配置：
	
	![配置源][MultipleConfigurationSources]

3. 在 Web 应用列表中，展开 Web 应用名称左侧的标记以显示部署槽。它将具有 Web 应用名称，然后是部署槽名称。
	
	![带有部署槽的应用列表][SiteListWithStagedSite]
	
4. 当你单击部署槽名称时，将打开一个包含一组选项卡的页面，这与其他 Web 应用一样。<strong>你的 Web 应用名称（部署槽名称）</strong>将显示在经典管理门户页面顶部，提醒你正在查看部署槽。
	
	![部署槽标题][StagingTitle]
	

创建部署槽后没有任何内容。您可以从其他存储库分支或完全不同的存储库部署到槽。您还可以更改此槽的配置。使用与内容更新部署槽关联的发布配置文件或部署凭证。例如，你可以[使用 git 发布到此槽](/documentation/articles/web-sites-publish-source-control/)。

<a name="AboutConfiguration"></a>
## 部署槽的配置 ##
从另一个部署槽克隆配置时，可以编辑克隆的配置。此外，某些配置元素在交换时遵循内容（不特定于位置），而其他配置元素将在交换之后保留在同一个位置（特定于位置）。以下列表显示交换槽时将更改的配置。

**已交换的设置**：

- 常规设置 - 例如 Framework 版本、32/64 位、Web 套接字
- 应用设置（可以配置为停在槽中）
- 连接字符串（可以配置为停在槽中）
- 处理程序映射
- 监视和诊断设置
- WebJobs 内容

**不交换的设置**：

- 发布终结点
- 自定义域名
- SSL 证书和绑定
- 缩放设置
- Web 作业计划程序


<a name="Swap"></a>
## 交换部署槽的步骤 ##

>[AZURE.IMPORTANT] 在将 Web 应用从部署槽交换到生产槽之前，请确保所有非槽特定的设置已完全根据你希望它在交换目标中的位置明确地进行配置。

1. 若要交换部署槽，请在 Web 应用命令栏或部署槽命令栏中单击“交换”按钮。确保正确设置交换源和交换目标。交换目标通常是生产槽。  

	![“交换”按钮][SwapButtonBar]

3. 单击“确定”完成操作。操作完成后，即已交换部署槽。

<a name="Multi-Phase"></a>
## 针对 Web 应用使用多阶段交换 ##

在设计为停在槽中的配置元素（例如连接字符串）的上下文中，多阶段交换可以简化验证。在这些情况下，从交换目标将这类配置元素应用到交换源，以及在交换实际生效之前进行验证，都是有用的做法。将交换目标配置元素应用到交换源后，可以执行的操作是完成交换，或者还原为交换源的原始配置，而还原还有取消交换的作用。“适用于部署槽的 Azure PowerShell cmdlet”部分中提供了可用于多阶段交换的 Azure PowerShell cmdlet 示例。

<a name="Rollback"></a>
## 在交换后回滚生产应用的步骤 ##

如果在槽交换后在生产中发现任何错误，请立即通过交换相同的两个槽来将槽回滚到交换前状态。

<a name="Warm-up"></a>
## 交换前的自定义准备工作 ##

某些应用可能需要自定义的准备操作。web.config 中的 applicationInitialization 配置元素可以指定收到请求之前要执行的自定义初始化操作。交换操作将等待此自定义准备操作完成。以下是 web.config 片段的示例。

    <applicationInitialization>
        <add initializationPage="/" hostName="[web app hostname]" />
        <add initializationPage="/Home/About" hostname="[web app hostname]" />
    </applicationInitialization>

<a name="Delete"></a>
## 删除部署槽##

在部署槽的边栏选项卡中，单击命令栏上的“删除”。

![删除部署槽][DeleteStagingSiteButton]

<!-- ======== AZURE POWERSHELL CMDLETS =========== -->

<a name="PowerShell"></a>
## 适用于部署槽的 Azure PowerShell cmdlet

Azure PowerShell 是一个模块，可提供通过 Windows PowerShell 管理 Azure 的 cmdlet，包括对管理 Azure Web 应用中 Web 应用部署槽的支持。

[AZURE.INCLUDE [AzureRm PowerShell 与中国 Azure 云](../includes/azurerm-azurechinacloud-environment-parameter.md)]

- 有关安装和配置 Azure PowerShell 的信息以及使用 Azure 订阅对 Azure PowerShell 进行身份验证的信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。  


----------

### 创建 Web 应用

	New-AzureRmWebApp -ResourceGroupName [resource group name] -Name [web app name] -Location [location] -AppServicePlan [app service plan name]

----------

### 为 Web 应用创建部署槽

	New-AzureRmWebAppSlot -ResourceGroupName [resource group name] -Name [web app name] -Slot [deployment slot name] -AppServicePlan [app service plan name]

----------

### 启动多阶段交换并将目标槽配置应用到源槽

	$ParametersObject = @{targetSlot  = "[slot name - e.g. "production"]"}
	Invoke-AzureRmResourceAction -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots -ResourceName [web app name]/[slot name] -Action applySlotConfig -Parameters $ParametersObject -ApiVersion 2015-07-01

----------

### 还原多阶段交换的第一个阶段并还原源槽配置

	Invoke-AzureRmResourceAction -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots -ResourceName [web app name]/[slot name] -Action resetSlotConfig -ApiVersion 2015-07-01

----------

### 交换部署槽

	$ParametersObject = @{targetSlot  = "[slot name - e.g. "production"]"}
	Invoke-AzureRmResourceAction -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots -ResourceName [web app name]/[slot name] -Action slotsswap -Parameters $ParametersObject -ApiVersion 2015-07-01

----------

### 删除部署槽

	Remove-AzureRmResource -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots -Name [web app name]/[slot name] -ApiVersion 2015-07-01

----------

<!-- ======== Azure CLI =========== -->

<a name="CLI"></a>
## 用于部署槽的 Azure 命令行界面 (Azure CLI) 命令

Azure CLI 提供了适用于 Azure 的跨平台命令，包括对 Web 应用部署槽的管理支持。

[AZURE.INCLUDE [Azure CLI 与中国 Azure 云](../includes/azure-cli-azurechinacloud-environment-parameter.md)]

- 有关安装和配置 Azure CLI 的说明（包括有关如何将 Azure CLI 连接到 Azure 订阅的信息），请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)。

-  若要在 Azure CLI 中列出可用于 Azure 的命令，请调用 `azure site -h`。

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
要删除不再需要的部署槽，请使用 **azure site delete** 命令，如以下示例所示。

`azure site delete webappslotstest --slot staging`

----------

## 后续步骤 ##
[Azure Web 应用 – 阻止对非生产部署槽的 Web 访问](http://ruslany.net/2014/04/azure-web-sites-block-web-access-to-non-production-deployment-slots/)

[Azure 试用版](/pricing/1rmb-trial/)

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
 

<!---HONumber=Mooncake_0411_2016-->