<properties title="Azure 网站 Web 托管计划深入概述" pageTitle="Azure 网站 Web 托管计划深入概述 - Windows Azure 功能指南" description="了解 Web 托管计划为 Azure 网站的工作原理以及如何使您的管理体验的获益。" metaKeywords="Azure Web Sites, Azure Websites, WHP, Web Hosting Plan, Web Hosting Plans, Resource Groups" services="web-sites" solutions="web" documentationCenter="Infrastructure" authors="Byron Tardif and Yochay Kiryaty" videoId="" scriptId="" />

# Azure 网站 Web 托管计划深入概述

</br>
Web 托管计划 (WHP) 表示一组可在你的网站间共享的功能和容量。Web 托管计划支持 4 种 Azure 网站定价层（免费、共享、基本和标准），其中每个层都有自己的功能和容量。同一订阅、资源组和地理位置中的网站可以共享一个 Web 宿主计划。共享 web 托管计划的所有站点都可以利用 web 托管计划层定义的所有容量和功能。与给定的 web 托管计划相关联的所有网站在 web 托管计划定义的资源上运行。例如，如果您的 web 托管计划配置为使用两个“小”虚拟机，则与该 web 托管计划相关联的所有站点将都在这两个虚拟机上运行。Azure 网站始终如此，您的网站在其上运行的虚拟机受到完全管理且高度可用。
</br>
在这篇文章中，我们将探讨如层和 Web 托管计划的主要特征以及它们如何在管理您的网站时发挥作用。
</br>

## 网站、Web 托管计划和资源组

</br>
网站可在任何给定时间和唯一一个 web 托管计划关联。Web 托管计划都与资源组相关联。资源组是 Azure 中的新概念，可充当该组中每个资源的生命周期界限。资源组允许您一起管理应用程序的所有片段。
</br>
资源组中可有多个 web 托管计划，并且每个托管计划将具有按其关联的站点使用的自己的特性和功能集。下图阐释了此关系：
</br>
</br>
![Resource Groups and Web Hosting Plans][Resource Groups and Web Hosting Plans]
</br>
</br>
单个资源组中的功能允许您将不同的站点分配给不同的资源，主要是在您的网站上运行的虚拟机。例如，这一功能允许在开发、测试和生产站点之间分离资源，您可能要为自己的生产站点分配一个具有专用资源组的 web 托管计划，为您的开发和测试站点分配第二个 web 托管计划。
</br>
在单个资源组中有多个 web 托管计划还允许您定义跨区域的一个应用程序。例如，在两个区域中运行的高度可用网站将包括两个网站，每个区域一个，并且每个网站和每个 web 托管计划相关联。在这种情况下，所有站点都将与单个资源组相关联，它定义一个应用程序。让带有多个 web 托管计划和多个站点的资源组有一个单独视图，让它易于管理、控制和查看网站的运行状况。为给定应用程序管理网站资源和相应站点最重要是您可关联任何相关的 Azure 资源，例如 SQL Azure 数据库和团队项目。
</br>

## 应在何时创建新的资源组和创建新的 web 托管计划？

</br>
创建新网站时，当您要创建的网站代表新的 web 应用程序时，应考虑创建新的资源组。在此情况，创建新的资源组、关联的 web 托管计划和网站是正确的选择。通过新的 Azure 门户预览创建此类新的网站时，使用库或新网站 + SQL 选项，门户将默认为您的新站点创建新的资源组和 web 托管计划。但如果需要，您可以覆盖这些默认值。
</br>
</br>
![Creating a new Web Hosting Plan][Creating a new Web Hosting Plan]
</br>
</br>
您始终可以向现有资源组添加新的网站或任何其他资源。如果从现有资源组的上下文中创建新的网站，新建网站向导将默认为现有资源和 web 托管计划。此处也可根据需要覆盖这些默认设置。将新的网站添加到现有的资源组时，可选择将该站点添加到现有的 web 托管计划（这是新的 Azure 门户预览中的默认选项），也可创建要添加站点的新的 web 托管计划。
</br>
创建新的托管计划允许您为自己的网站分配一组新的资源，并为您提供对资源分配更好的控制，因为每个 web 托管计划都获取其自己的虚拟机集。由于您可以在 web 托管计划之间移动网站，假定 web 托管计划在同一区域中，则决定是否创建新的 web 托管计划不太重要。如果给定的网站上开始消耗太多资源或只是需要分开几个网站，您可以创建新的 web 托管计划，并将您的网站移入其中。
</br>
如果您想在不同区域中创建一个新网站，且该区域没有现有的 web 托管计划，您将必须在该区域中创建新的 web 托管计划，以便网站与之关联。
</br>
需要牢记的重要一点是您不能移动 web 托管计划或资源组之间的网站。此外，您不能在两个单独区域中的两个 web 托管计划之间移动网站。
</br>

## Azure 预览门户中的现有资源组

</br>
如果您在 Azure 网站上已有现有网站，将注意到所有网站都显示在 Azure 预览门户中。可以通过单击左侧导航窗格上的**浏览**按钮并选择**网站**查看您的所有网站：
</br>
</br>
![See all your web site as a flat list][See all your web site as a flat list]
</br>
</br>
您还可以查看为您创建的所有资源组，方法是单击左侧导航窗格上的**浏览**按钮并选择**资源组**：
</br>
</br>
![See all the resource groups that have been created][See all the resource groups that have been created]
</br>
</br>
您还会注意到自己在已有网站的每个区域中有一个自动生成的默认资源组。网站的自动生成资源组名称是 *Default-Web-<location name>*，这里的位置名称代表 Azure 区域（例如 *Default-Web-WestUS*)。在每个资源组中，您会发现用于组区域的全部现有站点。在完整 Azure 门户或 Azure 预览门户中过去创建以及将来创建的每个站点将在这两个门户网站上可用。
</br>
由于每个网站都必须与 web 托管计划关联，我们已根据以下约定，在每个区域中为您的现有站点创建默认 web 托管计划：
</br>

-   您的所有**免费**网站都与**默认** web 托管计划关联，并且其定价层设置为**免费**。
    </br>
-   您的所有**共享**网站都与**默认** web 托管计划关联，并且其定价层设置为**共享**
    </br>
-   您的所有**标准**网站都与默认 web 托管计划关联，并且其定价层设置为**标准**。
    </br>
    此 web 托管计划的名称是 **DefaultServerFarm**。选择此名称的目的是支持旧 API。名称 **ServerFarm** 有些误导，因为它是指 **Web 托管计划**，但注意它是 web 托管计划的名称而非其自身的一个实体，这点很重要。
    </br>

    ## Web 托管计划常见问题

    </br>
    **问题**：如何创建 Web 托管计划？
    </br>
    **回答**：Web 托管计划是一个容器，因此您不能创建空的 Web 托管计划。但是，新 Web 托管计划在网站创建过程中显式创建。
    </br>
    要执行此操作，请使用 **Azure 门户预览**中的 UI，单击**新建**，然后选择**网站**，此时将打开网站创建边栏选项卡。在下面的第一个图像中，可以看到左下角的**新建**图标，在第二个图像中可以看到**网站**创建边栏选项卡、**Web 托管计划**边栏选项卡和**定价层**边栏选项卡：
    </br>
    </br>
    ![Create a new website][Create a new website]
    </br>
    </br>
    ![Website, Web Hosting Plan and pricing tier blades][Website, Web Hosting Plan and pricing tier blades]
    </br>
    </br>
    对于此示例，我们选择创建名为 **contosomarketing** 的新网站，并将其放在名为 **contoso** 的新的 web 托管计划中。为此 Web 托管计划选择的“定价层”是 **Small Standard**。有关 Webhosting 计划定价层及其功能的详细信息，每个中提供的定价和扩展选项，请访问 [Azure 网站 Web 托管计划规范][Azure 网站 Web 托管计划规范]。
    </br>
    还应注意 Web 托管计划也可在现有的 Azure 门户中创建。通过从**WEB 托管计划**下拉列表选择**创建新的 web 托管计划**将其作为**快速创建**的组成部分：
    </br>
    </br>
    ![Create new web hosting plan in the existing portal][Create new web hosting plan in the existing portal]
    </br>
    </br>
    对于本示例，我们创建名为 **northwind** 的新站点，并选择创建新的 web 托管计划。此操作的结果将是名为 **default0** 的新的 web 托管计划，其中包含 **northwind** 网站。通过这种体验创建的所有 web 托管计划都遵守这种命名约定，并且在创建后不能重命名 Web 托管计划。此外，将在**免费**定价层中创建通过此过程创建的 Web 托管计划。
    </br>
    **问题**：如何将站点分配到 **Web 托管计划**？
    </br>
    **回答**：站点将作为创建过程的一部分分配给 Web 托管计划站点。要执行此操作，使用新的 **Azure 门户预览**中的用户界面，单击**新建**，然后选择**网站**：
    </br>
    </br>
    ![Create a new website][1]
    </br>
    </br>
    然后在网站创建边栏选项卡中，选择托管计划：
    </br>
    </br>
    ![Select a hosting plan][Select a hosting plan]
    </br>
    </br>
    使用现有的 Azure 门户，一个站点也可创建到特定的 Web 托管计划中。这作为**快速创建**向导的一部分。键入网站 URL 后，使用 **WEB 托管计划**下拉列表来选择要添加到站点的计划：
    </br>
    </br>
    ![Select a hosting plan in the existing portal][Select a hosting plan in the existing portal]
    </br>
    </br>
    **问题**：如何将一个站点移动到不同 Web 托管计划？
    </br>
    **回答**：目前不支持使用 Azure 门户预览或完整 Azure 门户将一个站点移动到不同 Web 托管计划。但是，使用 [Azure PowerShell 工具][Azure PowerShell 工具]，可在不同的 web 托管计划之间移动站点。要执行此操作，请安装 Azure PowerShell 工具，打开一个 power shell 提示符。然后，使用 *Switch-AzureMode AzureResourceManager* Cmdlet 切换到新的 **Azure 资源管理器**模式并使用 *Add-AzureAccount* Cmdlet 进行身份验证。
    </br>
    对于此示例，假定已创建一个带有 2 个 Web 托管计划的名为 **powershell** (**whp1** & **whp2**) 的“资源组”和 2 个网站 (**pstest** & **pstest2**)。要获取资源组中的内容，可使用 Get-AzureResourceGroup*Get-AzureResourceGroup* Cmdlet：
    </br>
    </br>
    ![Get the content of a Resource group][Get the content of a Resource group]
    </br>
    </br>
    要获取每项的详细配置，可使用 *Get-AzureResource* Cmdlet。在下面的两个快照中，该命令正被用于显示 **pstest** 和 **pstest2** 网站的详细信息：
    </br>
    </br>
    ![Get the detailed configuration for website pstest][Get the detailed configuration for website pstest]
    </br>
    </br>
    ![Get the detailed configuration for website pstest2][Get the detailed configuration for website pstest2]
    </br>
    </br>
    您从 Cmdlet 的输出可以看到，这两个站点当前都与 **whp1** web 托管计划关联。您还将注意到 **ServerFarm** 属性也指向 **whp1**，并且这是因为 **ServerFarm** 是一个传统概念，要被 Web 托管计划的概念替换。
    </br>
    接下来，我们要使用 *Set-AzureResource* Cmdlet 将 **pstest2** 网站移动到 **whp2** web 托管计划。*Set-AzureResource* Cmdlet 将哈希表作为 **PropertyObject** 参数的输入。通过定义哈希表中并将该对象传递给此 Cmdlet 这种方式，可更改任何资源配置。
    </br>
    对此示例我们将定义一个名为 **$whp** 的变量，包含 **name** 和 **value** 对组成的哈希表，用于 Web 托管计划属性以及我们将设置其的值：
    </br>
    </br>
    ![Create a hash table for a hosting plan][Create a hash table for a hosting plan]
    </br>
    </br>
    下一步，我们使用已定义的哈希表 **$whp** 设置站点的 Web 托管计划属性：
    </br>
    </br>
    ![Use a Hash Table to set the Web Hosting Plan property of the site][Use a Hash Table to set the Web Hosting Plan property of the site]
    </br>
    </br>
    有关使用 Powershell Cmdlet 的详细信息，请访问[此文章][此文章]。
    </br>
    请注意每个 web 托管计划都有自己的定价层。当您将网站从**免费**层 web 托管计划移动到**标准** web 托管计划，您的网站将能够利用标准层的所有功能和资源。
    </br>

</br>
**问题**：如何扩展 Web 托管计划？
</br>
**回答**：有两种方法可扩展 Web 托管计划。一种方法是来向上扩展您的 web 托管计划，以及和该 web 托管计划相关联的所有站点。通过更改 web 托管计划的定价层，该 web 托管计划中的所有站点都将受定价层定义的功能和资源的影响。
</br>
下图中可以看到 **Web 托管计划**边栏选项卡以及**定价层**边栏选项卡。单击 **Web 托管计划**边栏选项卡中的**定价层**部分，展开您可为 web 托管计划更改定价层的**定价层**边栏选项卡：
</br>
</br>
![The Web Hosting Plan blade and the Pricing Tier][The Web Hosting Plan blade and the Pricing Tier]
</br>
</br>
扩展计划的第二种方法是通过增加您的 Web 托管计划中的实例数向上扩展。下图中可以看到 **Web 托管计划**边栏选项卡以及**扩展**边栏选项卡。单击 **Web 托管计划**边栏选项卡中的“扩展”区域展开它，并允许更改该计划的实例计数：
</br>
</br>
![Changing the instance count of a hosting plan][Changing the instance count of a hosting plan]
</br>
</br>
因为上图中的 Web 托管计划配置为使用**标准**定价层，因此会启用**自动扩展**选项。
</br>
在完整 Azure 门户中执行此操作可以在**扩展**选项卡中进行，如下所示：
</br>
</br>
![Changing the instance count of a hosting plan in the existing portal][Changing the instance count of a hosting plan in the existing portal]
</br>
</br>
**问题**：如何删除 Web 托管计划？
</br>
**回答**：要删除 Web 托管计划，必须先删除与之关联的所有网站。一旦删除 Web 托管计划中的所有网站，可以从 Web 托管计划边栏选项卡删除 Web 托管计划：
</br>
</br>
![Deleting a web hosting plan][Deleting a web hosting plan]
</br>
</br>
在完整 Azure 门户删除 web 托管计划中的最后一个网站，将自动删除关联的 web 托管计划。
</br>
**问题**：如何监视 Web 托管计划？
</br>
**回答**：可使用 Web 托管计划边栏选项卡中的“监视”部分监视 Web 托管计划：
</br>
</br>
![Monitoring a web hosting plan][Monitoring a web hosting plan]
</br>
</br>
右键单击控件并选择**编辑查询**可定制监视控件：
</br>
</br>
![Editing the monitoring controls][Editing the monitoring controls]
</br>
</br>
公开的度量值是：
</br>

-   CPU 百分比
    </br>
-   内存百分比
    </br>
-   磁盘队列长度
    </br>
-   HTTP 队列长度。
    </br>
    此指标表示属于 Web 托管计划的实例之间的平均使用率。所有这些度量值均可用于设置警报，以及“自动扩展”规则。
    </br>

    ## 最终结论

    </br>
    Web 托管计划表示一组您可跨自己的网站间共享的功能和容量。Web 托管计划让您灵活地将特定站点分配到给定资源组 - 虚拟机，并进一步优化您 Azure 资源分配和网站的使用。

  [Resource Groups and Web Hosting Plans]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview01.png
  [Creating a new Web Hosting Plan]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview02.png
  [See all your web site as a flat list]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview03.png
  [See all the resource groups that have been created]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview04.png
  [Create a new website]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview05.png
  [Website, Web Hosting Plan and pricing tier blades]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview06.png
  [Azure 网站 Web 托管计划规范]: http://go.microsoft.com/?linkid=9845586
  [Create new web hosting plan in the existing portal]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview07.png
  [1]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview08.png
  [Select a hosting plan]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview09.png
  [Select a hosting plan in the existing portal]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview10.png
  [Azure PowerShell 工具]: http://www.windowsazure.com/zh-cn/documentation/articles/install-configure-powershell/
  [Get the content of a Resource group]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview11.png
  [Get the detailed configuration for website pstest]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview12.png
  [Get the detailed configuration for website pstest2]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview13.png
  [Create a hash table for a hosting plan]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview14.png
  [Use a Hash Table to set the Web Hosting Plan property of the site]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview15.png
  [此文章]: http://msdn.microsoft.com/library/azure/jj156055.aspx
  [The Web Hosting Plan blade and the Pricing Tier]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview16.png
  [Changing the instance count of a hosting plan]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview17.png
  [Changing the instance count of a hosting plan in the existing portal]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview18.png
  [Deleting a web hosting plan]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview19.png
  [Monitoring a web hosting plan]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview20.png
  [Editing the monitoring controls]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview21.png
