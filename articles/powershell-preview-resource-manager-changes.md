<properties
	pageTitle="Azure PowerShell 1.0 预览版资源管理器更改 | Azure"
	description="介绍针对 Azure PowerShell 1.0 预览版设计的资源管理器 cmdlet 所发生的更改。"
	services="azure-resource-manager"
	documentationCenter="na"
	authors="ravbhatnagar"
	manager="ryjones"
	editor=""/>

<tags
	ms.service="azure-resource-manager"
	ms.date="10/09/2015"
	wacn.date="11/12/2015"/>

#对 Azure 资源管理器管理 PowerShell cmdlet 的更改

在 Azure PowerShell 1.0 预览版中，我们对管理 cmdlet 做了一些改善。这些改善不包括 cmdlet 名称更改，因为 1.0 预览版本身就包括这些更改。其中有些改善涉及到重大更改，这可能会破坏现有的脚本。我们希望这些更改能够改善你的体验。我们乐意听取大家对这些更改的意见，并会将意见纳入 Azure PowerShell 1.0。请试用新的 cmdlet 并向我们提供反馈。

##已从资源组中分离模板部署

在 0.9.8 版本中，所有模板部署参数也都会出现在资源组 cmdlet 中。因此，在 New-AzureResourceGroup 中无法创建新的资源组，以及提供想要部署的模板的详细信息。相同的模板部署功能也出现在 New-AzureResourceGroupDeployment 中。在 1.0 预览版中，我们已将此功能分离出来。现在，New-AzureRMResourceGroup 将提供创建新资源组的功能，而 New-AzureRmResourceGroupDeployment 将提供部署模板的功能。可以使用管道来结合使用此两者。这使得 cmdlet 更易于理解和使用。

##合并了审核日志 cmdlet

对于审核日志，我们提供许多 cmdlet 来获取不同范围的日志；例如，Get-AzureResourceProviderLog、Get-AzureResourceGroupLog、Get-AzureSubscriptionIdLog 和 Get-AzureResourceLog。若要获取日志，通常需要运行日志 cmdlet 的组合。这种体验不是最理想的。我们已将此功能合并到单个 cmdlet，可在不同范围中通过使用参数来调用它。现在，你可以结合适当的参数调用 Get-AzureRmLog 来指定范围。

在版本 0.9.8 中，若要获取特定资源组的日志，你需要运行如下所示的命令：

    Get-AzureResourceGroupLog -ResourceGroup <resource-group-name>

若要获取特定资源的日志，你需要执行以下命令：

     Get-AzureResourceLog -ResourceId
     /subscriptions/#######-####-####-####-############/resourcegroups/<resource-group-name>/providers/<provider-namespace>/
     <resource-name>

在 1.0 预览版中，运行以下 cmdlet 就能获取相同的信息。若要获取资源组的日志，你可以运行：

     Get-AzureRmLog -ResourceGroup <resource-group-name>
     
若要获取特定资源的日志，你可以运行：

     Get-AzureRmLog -ResourceId /subscriptions/#######-####-####-####-############/resourcegroups/<resource-group-name>
     /providers/<provider-namespace>/<resource-name>

##对于适用于资源和资源组的 Get cmdlet 的更改

我们已合并针对 Get-AzureRmResource 和 Get-AzureRmResourceGroup cmdlet 的更改，因此，这些 cmdlet 现在只会针对资源提供程序直接进行查询。我们已分离了针对缓存进行查询的功能，并设计出了名为 Find-AzureRmResource* 的新 cmdlet。如果你想要查找具有特定标记的资源组，可以使用新的 Find-AzureRmResourceGroup cmdlet。由于这种更改，我们已从 Get-AzureRmResource 和 Get-AzureRmResourceGroup cmdlet 中删除了针对标记进行查询的参数。

在 0.9.8 中，若要查找包含特定标记的所有资源组，你需要运行：

    Get-AzureResourceGroup -Tag <tag-object>

在 1.0 预览版中，运行以下 cmdlet 即可实现上述方案：

    Find-AzureRmResourceGroup -Tag <tag-object>
    
##已删除 Test-AzureResource 和 Test-AzureResourceGroup

这些 cmdlet 无法针对每种方案和资源类型可靠运行。我们正在努力针对此功能寻找更好的解决方案。在此同时，我们不希望你继续使用不可靠的 cmdlet。我们已在此版本中删除这些 cmdlet，将来的版本中会添加可靠的解决方案。

##Get-AzureRmResourceProvider 的改善

现在，你可以使用 Get-AzureRmResourceProvider cmdlet 来获取资源提供程序的位置信息。它将告诉你某个特定区域中可以使用哪些提供程序和类型。此外，你可以使用此 cmdlet 来获取特定提供程序的可用位置列表。我们已删除 Get-AzureLocation cmdlet，因为现在可以通过 Get-AzureRmResourceProvider cmdlet 来获取所有位置信息。

在 0.9.8，若要获取所有受支持位置的列表，需要运行：

     Get-AzureLocation

若要获取提供程序的注册状态，需要运行：

     Get-AzureProvider -ListAvailable

在 1.0 预览版中，只需按如下所示使用一个 cmdlet，就能实现上述目的：

     Get-AzureRmResourceProvider -Location <location>

上述 cmdlet 将筛选结果，以便只显示指定位置中可用的提供程序和类型。

或者，你可以按如下所示将结果筛选为特定的提供程序：

     Get-AzureRmResourceProvider -ProviderNamespace <provider-namespace>

上述 cmdlet 将筛选结果，以便只显示指定提供程序支持的位置。

##策略 cmdlet

我们已添加对 Azure 资源管理器的策略支持。此版本中已添加可用于管理策略的 PowerShell cmdlet。

<!---HONumber=79-->