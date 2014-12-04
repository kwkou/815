<properties linkid="web-sites-staged-publishing" urlDisplayName="How to stage sites on Microsoft Azure" pageTitle="在 Microsoft Azure 网站上过渡部署" metaKeywords="Microsoft Azure Web Sites, Staged Deployment, Site Slots" description="了解如何在 Microsoft Azure 网站上使用过渡发布。" metaCanonical="" services="web-sites" documentationCenter="" title="在 Microsoft Azure 网站上过渡部署" authors="timamm"  solutions="" writer="timamm" manager="paulettm" editor="mollybos"  />

# 在 Microsoft Azure 网站上过渡部署

## 目录

-   [概述][概述]
-   [要向网站中添加部署槽][要向网站中添加部署槽]
-   [关于部署槽配置][关于部署槽配置]
-   [要交换部署槽][要交换部署槽]
-   [要将生产站点回滚到过渡环境][要将生产站点回滚到过渡环境]
-   [要删除站点槽][要删除站点槽]
-   [适用于站点槽的 Azure PowerShell cmdlet][适用于站点槽的 Azure PowerShell cmdlet]
-   [适用于站点槽的 Azure 跨平台命令行接口 (xplat-cli) 命令][适用于站点槽的 Azure 跨平台命令行接口 (xplat-cli) 命令]

<a name="Overview"></a>

## 概述

为在 Microsoft Azure 网站上运行的“标准”模式站点创建站点槽的选项允许使用过渡部署工作流。为每个默认生产站点（现在是生产槽）创建开发或过渡站点槽，并交换这些槽而无需停机。过渡部署对以下情况很有用：

-   **在部署前验证** - 向过渡站点槽部署内容或配置后，您可以在将这些更改交换到生产环境之前，对更改进行验证。

-   **构建和集成站点内容** - 您可以以增量方式将内容更新添加到过渡部署槽，然后在更新完成后将部署槽交换到生产环境。

-   **回滚生产站点** - 如果交换到生产环境的更改和预期不同，您可以立即将原始内容交换回生产环境。

在交换到生产环境之前，Microsoft Azure 会对源站点槽的所有实例进行预热，并防止部署内容时冷启动。流量重定向是无缝的，且不会因交换操作而删除任何请求。当前，除默认生产槽之外，每个标准网站只支持一个部署槽。

<a name="Add"></a>

## 要向网站中添加部署槽

要创建站点槽，站点必须在“标准”模式下运行。当前 Azure 每个生产槽只支持一个部署槽。

1.  在“快速入门”页面或网站“仪表板”页面的“速览”部分，单击“添加新部署槽”。

    ![添加新部署槽][添加新部署槽]

    > [WACOM.NOTE]
    > 如果网站不在“标准”模式下，您将收到消息“您必须在标准模式下才能启用过渡发布”。此时，您可以选择“升级”，并导航到网站的“缩放”选项卡，然后再继续。

2.  将显示“添加新部署槽”对话框。

    ![“添加新部署槽”对话框][“添加新部署槽”对话框]

    提供部署槽的名称。名称不能超过 60 个字母数字字符。不允许使用特殊字符或空格。

3.  在网站列表中，展开网站名称左侧的标记以显示部署槽。您提供的部署槽名称跟在生产站点名称后面，并用括号括住。

    ![带有部署槽的站点列表][带有部署槽的站点列表]

4.  选择部署站点槽名称时，将打开一个包含一组选项卡的页面，和其他网站一样。***您的网站名称*(*部署槽名称*)**将显示在门户页面顶部，提醒您正在查看部署站点槽。

    ![部署槽标题][部署槽标题]

您现在可以更新部署站点槽的内容和配置。使用与内容更新部署站点槽关联的发布配置文件或部署凭据。

<a name="AboutConfiguration"></a>

## 关于部署槽配置

创建部署槽之后，默认情况下，将从生产站点槽克隆部署槽的配置。所有站点槽的配置都可编辑。

**槽交换时可能会更改的配置**：

-   常规设置
-   连接字符串
-   处理程序映射
-   监视和诊断设置

**槽交换时不会更改的配置**：

-   发布终结点
-   自定义域名
-   SSL 证书和绑定
-   缩放设置

**注意**：

-   只有标准模式下的站点才可使用部署槽。

-   如果将站点更改为“免费”、“共享”或“基本”模式，站点将不再可更换。

-   要交换到生产环境的部署槽应完全按照生产环境中的预期设置进行配置。

-   默认情况下，部署槽与生产站点将指向同一个数据库。但是，您可以通过更改部署槽的数据库连接字符串，将部署槽配置为指向替代数据库。您随后可以在将部署槽交换到生产环境之前，还原部署槽的原始数据库连接字符串。

<a name="Swap"></a>

## 要交换部署槽

1.  要交换部署槽，请在网站列表中选择部署槽，然后在命令栏中单击“交换”按钮。

    ![“交换”按钮][“交换”按钮]

2.  将显示“交换部署”对话框。该对话框用于选择哪个站点槽应为源，哪个站点应为目标。

    ![“交换部署”对话框][“交换部署”对话框]

3.  单击复选标记完成操作。操作完成后，即已交换站点槽。

<a name="Rollback"></a>

## 要将生产站点回滚到过渡环境

如果交换到生产环境的内容或配置被标识任何错误，可以只将部署槽（之前的生产站点）交换回生产环境，然后对处于过渡模式下的新站点版本做进一步修复。

> [WACOM.NOTE]
> 要使回滚成功，部署站点槽必须仍包含之前的生产站点未更改的内容和配置。

<a name="Delete"></a>

## 要删除站点槽

在 Azure 网站门户页面底部的命令栏中，单击“删除”。您可以选择删除网站和所有部署槽，或仅删除部署槽。

![删除站点槽][删除站点槽]

对确认消息回答“是”之后，将删除一个或所有槽，具体取决于您选择的选项。

**注意**：

-   缩放不适用于非生产站点槽。仅适用于生产站点槽。

-   非生产站点槽不支持链接的资源管理。

-   但您仍可以根据需要直接发布到生产站点槽。

-   当前部署槽（站点）与生产槽（站点）共享相同的资源，并在同一个 VM 上运行。在过渡槽上运行压力测试时，生产环境将受到相当的压力负载。

<!-- ======== AZURE POWERSHELL CMDLETS =========== -->

<a name="PowerShell"></a>

## 适用于站点槽的 Azure PowerShell cmdlet

Azure PowerShell 是一个模块，可提供通过 Windows PowerShell 管理 Azure 的 cmdlet，包括对管理 Azure 网站部署槽的支持。

-   有关安装和配置 Azure PowerShell 的信息以及使用 Windows Azure 订阅对 Azure PowerShell 进行身份验证的信息，请参阅[如何安装和配置 Windows Azure PowerShell][如何安装和配置 Windows Azure PowerShell]。

-   要列出 PowerShell 中适用于 Azure 网站的 cmdlet，请调用 `help AzureWebsite`。

------------------------------------------------------------------------

### Get-AzureWebsite

**Get-AzureWebsite** cmdlet 提供当前订阅中有关 Azure 网站的信息，如以下示例所示。

`Get-AzureWebsite siteslotstest`

------------------------------------------------------------------------

### New-AzureWebsite

您可以通过使用 **New-AzureWebsite** cmdlet 并指定站点和槽的名称，为“标准”模式下的任何网站创建站点槽。还表示与用于创建部署槽的站点相同的区域，如以下示例所示。

`New-AzureWebsite siteslotstest –Slot staging –Location “West US”`

------------------------------------------------------------------------

### Publish-AzureWebsiteProject

您可以使用 **Publish-AzureWebsiteProject** cmdlet 进行内容部署，如以下示例所示。

`Publish-AzureWebsiteProject -Name siteslotstest -Slot staging -Package [path].zip`

------------------------------------------------------------------------

### Show-AzureWebsite

内容和配置更新应用到新的槽之后，您可以通过使用 **Show-AzureWebsite** cmdlet 浏览到该槽，对更新进行验证，如以下示例所示。

`Show-AzureWebsite -Name siteslotstest -Slot staging`

------------------------------------------------------------------------

### Switch-AzureWebsiteSlot

**Switch-AzureWebsiteSlot** cmdlet 可以执行交换操作，使更新的部署槽成为生产站点，如以下示例所示。生产站点将不会停机，也不会进行冷启动。

`Switch-AzureWebsiteSlot -Name siteslotstest`

------------------------------------------------------------------------

### Remove-AzureWebsite

如果不再需要部署槽，则可以使用 **Remove-AzureWebsite** cmdlet 删除该部署槽，如以下示例所示。

`Remove-AzureWebsite -Name siteslotstest -Slot staging`

------------------------------------------------------------------------

<!-- ======== XPLAT-CLI =========== -->

<a name="CLI"></a>

## 适用于站点槽的 Azure 跨平台命令行接口 (xplat-cli) 命令

Azure 跨平台命令行接口 (xplat-cli) 提供了用于与 Azure 结合使用的跨平台命令，包括对在 Azure 网站上管理部署槽的支持。

-   有关安装和配置 xplat-cli 的说明（包括有关如何将 xplat-cli 连接到 Azure 订阅的信息），请参阅[安装和配置 Azure 跨平台命令行接口][安装和配置 Azure 跨平台命令行接口]。

-   要列出 xplat-cli 中适用于 Azure 网站的命令，请调用 `azure site -h`。

------------------------------------------------------------------------

### azure site list

有关当前订阅中 Azure 网站的相关信息，请调用 **azure site list**，如以下示例所示。

`azure site list siteslotstest`

------------------------------------------------------------------------

### azure site create

要为“标准”模式下的任何网站创建站点槽，请调用 **azure site create**，并指定现有站点的名称和要创建的槽的名称，如以下示例所示。

`azure site create siteslotstest --slot staging`

要启用新槽源代码管理，请使用 **--git** 选项，如以下示例所示。

`azure site create –-git siteslotstest --slot staging`

------------------------------------------------------------------------

### azure site swap

要使更新的部署槽成为生产站点，请使用 **azure site swap** 命令执行交换操作，如以下示例所示。生产站点将不会停机，也不会进行冷启动。

`azure site swap siteslotstest`

------------------------------------------------------------------------

### azure site delete

要删除不再需要的部署槽，请使用 **azure site delete** 命令，如以下示例所示。

`azure site delete siteslotstest --slot staging`

------------------------------------------------------------------------

## 后续步骤

若要开始使用 Azure，请参阅 [Microsoft Azure 免费试用版][Microsoft Azure 免费试用版]。

<!-- IMAGES -->

  [概述]: #Overview
  [要向网站中添加部署槽]: #Add
  [关于部署槽配置]: #AboutConfiguration
  [要交换部署槽]: #Swap
  [要将生产站点回滚到过渡环境]: #Rollback
  [要删除站点槽]: #Delete
  [适用于站点槽的 Azure PowerShell cmdlet]: #PowerShell
  [适用于站点槽的 Azure 跨平台命令行接口 (xplat-cli) 命令]: #CLI
  [添加新部署槽]: ./media/web-sites-staged-publishing/QGAddNewDeploymentSlot.png
  [“添加新部署槽”对话框]: ./media/web-sites-staged-publishing/AddNewDeploymentSlotDialog.png
  [带有部署槽的站点列表]: ./media/web-sites-staged-publishing/SiteListWithStagedSite.png
  [部署槽标题]: ./media/web-sites-staged-publishing/StagingTitle.png
  [“交换”按钮]: ./media/web-sites-staged-publishing/SwapButtonBar.png
  [“交换部署”对话框]: ./media/web-sites-staged-publishing/SwapDeploymentsDialog.png
  [删除站点槽]: ./media/web-sites-staged-publishing/DeleteStagingSiteButton.png
  [如何安装和配置 Windows Azure PowerShell]: http://azure.microsoft.com/zh-cn/documentation/articles/install-configure-powershell/
  [安装和配置 Azure 跨平台命令行接口]: http://azure.microsoft.com/zh-cn/documentation/articles/xplat-cli/
  [Microsoft Azure 免费试用版]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
