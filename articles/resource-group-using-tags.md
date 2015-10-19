<properties 
	pageTitle="使用标记来组织 Azure 资源" 
	description="演示如何应用标记来组织资源进行计费和管理。" 
	services="azure-resource-manager" 
	documentationCenter="" 
	authors="tfitzmac"
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="azure-resource-manager" 
	ms.date="08/07/2015" 
	wacn.date="10/3/2015"/>


# 使用标记来组织 Azure 资源

资源管理器使您可以通过应用标记来按照逻辑组织资源。标记由通过您所定义的属性标识资源的键值对组成。若要将资源标记为属于同一类别，请将相同的标记应用到这些资源。

查看具有特定标记的资源时，您会看到所有资源组中的资源。您并不一定只能在相同资源组中组织资源，还能够以独立于部署关系的方式组织资源。当您需要组织资源以进行计费或管理时，标记会特别有用。

您添加到资源或资源组的每个标记都会自动添加到订阅范围的分类。您也可以将标记名称预先填入订阅的分类，而且您想要作为资源使用的值会在未来加以标记。

> [AZURE.NOTE]您只能将标记应用到支持资源管理器操作的资源。如果您是通过经典部署模型（如通过 Azure 门户或[服务管理 API](https://msdn.microsoft.com/zh-cn/library/azure/dn948465.aspx)）创建的虚拟机、虚拟网络或存储，则无法将标记应用到该资源。您必须通过资源管理器重新部署这些资源才能支持标记。所有其他资源均支持标记。


## 预览门户中的标记

在预览门户中标记资源和资源组非常简单。使用“浏览”中心导航到要标记的资源或资源组，然后在边栏选项卡顶部的“概览”部分中单击“标记”部件。

![资源和资源组边栏选项卡上的“标记”部件](./media/resource-group-using-tags/tag-icon.png)

此时将打开一个边栏选项卡，其中包含已应用的标记列表。如果这是你的第一个标记，该列表将为空。若要添加标记，只需指定名称和值，然后按 Enter。添加若干个标记后，系统会根据预先存在的标记名称和值提供自动填充选项，以更好地确保各项资源的分类保持一致，并避免常见错误，如拼写错误。

![使用名称/值对标记资源](./media/resource-group-using-tags/tag-resources.png)

若要在门户中查看标记的分类，请使用“浏览”中心查看“所有内容”，然后选择“标记”。

![通过“浏览”中心查找标记](./media/resource-group-using-tags/browse-tags.png)

将最重要的标记固定启动板以便快速访问，准备工作到此完成。尽情使用吧！

![将标记固定到启动板](./media/resource-group-using-tags/pin-tags.png)

## 使用 PowerShell 进行标记

如果你以前没有对资源管理器使用过 Azure PowerShell，请参阅[将 Azure PowerShell 与 Azure 资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager)。在本文中，我们假设您已添加一个帐户，并选择了包含您要标记的资源的订阅。

标记功能只适用于[资源管理器](http://msdn.microsoft.com/zh-cn/library/azure/dn790568.aspx)中提供的资源和资源组，因此要做的下一步是改为使用资源管理器。

    Switch-AzureMode AzureResourceManager

标记直接存在于资源和资源组中，因此，若要查看已应用了哪些标记，只需分别使用 `Get-AzureResource` 或 `Get-AzureResourceGroup` 获取资源或资源组。让我们从一个资源组着手。

    PS C:\> Get-AzureResourceGroup tag-demo

    ResourceGroupName : tag-demo
    Location          : southcentralus
    ProvisioningState : Succeeded
    Tags              :
    Permissions       :
                    Actions  NotActions
                    =======  ==========
                    *

    Resources         :
                    Name                             Type                                  Location
                    ===============================  ====================================  ==============
                    CPUHigh ExamplePlan              microsoft.insights/alertrules         chinaeast
                    ForbiddenRequests tag-demo-site  microsoft.insights/alertrules         chinaeast
                    LongHttpQueue ExamplePlan        microsoft.insights/alertrules         chinaeast
                    ServerErrors tag-demo-site       microsoft.insights/alertrules         chinaeast
                    ExamplePlan-tag-demo             microsoft.insights/autoscalesettings  chinaeast
                    tag-demo-site                    microsoft.insights/components         chinaeast
                    ExamplePlan                      Microsoft.Web/serverFarms             chinaeast
                    tag-demo-site                    Microsoft.Web/sites                   chinaeast


此 cmdlet 将返回有关资源组的元数据的多个片段，包括已应用了哪些标记（如果有）。若要标记资源组，只需使用 `Set-AzureResourceGroup` 命令并指定标记名称和值。

    PS C:\> Set-AzureResourceGroup tag-demo -Tag @( @{ Name="project"; Value="tags" }, @{ Name="env"; Value="demo"} )

    ResourceGroupName : tag-demo
    Location          : southcentralus
    ProvisioningState : Succeeded
    Tags              :
                    Name     Value
                    =======  =====
                    project  tags
                    env      demo

标记作为一个整体进行更新，因此，如果要将一个标记添加到已标记的资源，需要使用一个数组，其中包含您要保留的所有标记。为此，可以首先选择现有标记，然后添加一个新标记。

    PS C:\> $tags = (Get-AzureResourceGroup -Name tag-demo).Tags
    PS C:\> $tags += @{Name="status";Value="approved"}
    PS C:\> Set-AzureResourceGroup tag-demo -Tag $tags

    ResourceGroupName : tag-demo
    Location          : southcentralus
    ProvisioningState : Succeeded
    Tags              :
                    Name     Value
                    =======  ========
                    project  tags
                    env      demo
                    status   approved


若要删除一个或多个标记，只需保存不包含您要删除的标记的数组。

该过程对于资源是相同，不过，使用的是 `Get-AzureResource` 和 `Set-AzureResource` cmdlet。若要获取具有特定标记的资源或资源组，请结合 `-Tag` 参数使用 `Get-AzureResource` 或 `Get-AzureResourceGroup` cmdlet。

    PS C:\> Get-AzureResourceGroup -Tag @{ Name="env"; Value="demo" } | %{ $_.ResourceGroupName }
    rbacdemo-group
    tag-demo
    PS C:\> Get-AzureResource -Tag @{ Name="env"; Value="demo" } | %{ $_.Name }
    rbacdemo-web
    rbacdemo-docdb
    ...

若要使用 PowerShell 获取订阅中所有标记的列表，请使用 `Get-AzureTag` cmdlet。

    PS C:/> Get-AzureTag
    Name                      Count
    ----                      ------
    env                       8
    project                   1

你可能会看到，有些标记以“hidden-”和“link:”开头。这属于内部标记，你应忽略这些标记并避免更改。

使用 `New-AzureTag` cmdlet 可将新标记添加到分类。即使这些标记尚未应用到任何资源或资源组，也会包含在自动填充内容中。若要删除某个标记名称/值，请先从它可能已应用到的所有资源中将它删除，然后使用 `Remove-AzureTag` cmdlet 从分类中将它删除。

## 使用 REST API 进行标记

门户和 PowerShell 在幕后都使用[资源管理器 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dn790568.aspx)。如果需要将标记集成到另一个环境中，可以对资源 ID 使用 GET 以获取标记，并使用 PATCH 调用更新标记集。


## 标记和计费

对于受支持的服务，您可以使用标记对计费数据进行分组。例如，[与 Azure 资源管理器集成的虚拟机](/documentation/articles/virtual-machines-azurerm-versus-azuresm)可让您定义并应用标签来组织虚拟机的计费使用情况。如果您针对不同组织运行多个虚拟机，可以使用标记根据成本中心对使用情况进行分组。您还可以使用标记根据运行时环境对成本进行分类；例如，在生产环境中运行的虚拟机的计费使用情况。

您可以通过[使用情况 API](/documentation/articles/billing-usage-rate-card-overview) 或者可以从 [Azure 帐户门户](https://account.windowsazure.com/)或 [EA 门户](https://ea.azure.com)下载的使用情况逗号分隔值 (CSV) 文件来检索有关标记的信息。有关以编程方式访问计费信息的详细信息，请参阅[深入了解您的 Microsoft Azure 资源消耗](/documentation/articles/billing-usage-rate-card-overview)。

在您为支持包含计费的标记的服务下载使用情况 CSV 时，标记将显示在**标记**列中。有关更多详细信息，请参阅[了解 Microsoft Azure 的计费](/documentation/articles/billing-understand-your-bill)。

![在计费中查看标记](./media/resource-group-using-tags/billing_csv.png)

## 后续步骤

- 有关部署资源时使用 Azure PowerShell 的说明，请参阅[将 Azure PowerShell 与 Azure 资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager)。
- 有关部署资源时使用 Azure CLI 的说明，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理配合使用](/documentation/articles/xplat-cli-azure-resource-manager)。
<!-- - 有关使用预览门户的说明，请参阅[使用 Azure 预览门户管理 Azure 资源](/documentation/articles/resource-group-portal)。-->  
  

  

<!---HONumber=71-->