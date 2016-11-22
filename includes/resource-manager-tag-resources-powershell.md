### 最新 PowerShell 版本中的标记 cmdlet 更改

2016 年 8 月版的 [Azure PowerShell 2.0][powershell] 对如何使用标记进行了重大更改。在继续之前，请检查 AzureRm.Resources 模块的版本。

    Get-Module -ListAvailable -Name AzureRm.Resources | Select Version

如果上次更新 Azure PowerShell 是在 2016 年 8 月之前，结果将显示版本低于 3.0。

    Version
    -------
    2.0.2

如果上次更新 Azure PowerShell 是在 2016 年 8 月之后，结果将显示版本为 3.0。

    Version
    -------
    3.0.1
    
如果模块版本为 3.0.1 或更高，将拥有处理标记的最新 cmdlet。使用 PowerShell 库、PowerShellGet 或 Web 平台安装程序安装或升级 Azure PowerShell 时，此版本的 Azure 资源模块会自动安装。如果版本低于 3.0.1，可以继续使用该版本，但你可以考虑更新到最新版本。最新版本包含可让你更加轻松地处理标记的更改。本主题对这两种方法都进行了说明。

### 更新脚本以获取最新版本的更改 

在最新版本中，**Tags** 参数名称更改为 **Tag**，而类型由 **Hashtable []** 更改为 **Hashtable**。不再需要为每个条目提供“名称”和“值”。而是提供**键 =“值”**格式的键/值对。

若要更新现有脚本，请将 **Tags** 参数更改为**Tag**，并更改标记格式，如以下示例所示。

    # Old
    New-AzureRmResourceGroup -Tags @{ Name = "testtag"; Value = "testval" } -Name $resourceGroupName -Location $location

    # New
    New-AzureRmResourceGroup -Tag @{ testtag = "testval" } -Name $resourceGroupName -Location $location 

但是，应该注意，资源组和资源在其元数据中仍将返回 **Tags** 属性。此属性不会更改。

### 3\.0.1 版本或更高版本

标记直接存在于资源和资源组中。若要查看现有标记，请使用 **Get-AzureRmResource** 查看资源，或使用 **Get-AzureRmResourceGroup** 查看资源组。

让我们从一个资源组着手。

    Get-AzureRmResourceGroup -Name testrg1

此 cmdlet 将返回有关资源组的元数据的多个片段，包括已应用了哪些标记（如果有）。

    ResourceGroupName : testrg1
    Location          : westus
    ProvisioningState : Succeeded
    Tags              :
                    Name         Value
                    ===========  ==========
                    Dept         Finance
                    Environment  Production

若要检索包含标记的资源元数据，请使用以下示例。

    Get-AzureRmResource -ResourceName tfsqlserver -ResourceGroupName testrg1

在结果中查看标记名称。

    Name              : tfsqlserver
    ResourceId        : /subscriptions/{guid}/resourceGroups/tag-demo-group/providers/Microsoft.Sql/servers/tfsqlserver
    ResourceName      : tfsqlserver
    ResourceType      : Microsoft.Sql/servers
    Kind              : v12.0
    ResourceGroupName : testrg1
    Location          : westus
    SubscriptionId    : {guid}
    Tags              : {Dept, Environment}

使用“标记”属性获取标记名称和值。

    (Get-AzureRmResource -ResourceName tfsqlserver -ResourceGroupName testrg1).Tags

这将返回以下结果：

    Name                   Value
    ----                   -----
    Dept                   Finance
    Environment            Production

建议检索具有特定标记和值的所有资源或资源组，而不是查看特定资源组或资源的标记。若要获取具有特定标记的资源组，请结合 **-Tag** 参数使用 **Find-AzureRmResourceGroup** cmdlet。

若要检索带有标记值的资源组，请使用以下格式。

    (Find-AzureRmResourceGroup -Tag @{ Dept="Finance" }).Name 

若要获取具有特定标记和值的所有资源，请使用 **Find-AzureRmResource** cmdlet。

    (Find-AzureRmResource -TagName Dept -TagValue Finance).Name
    
若要向当前没有标记的资源组添加标记，请使用 **Set-AzureRmResourceGroup** 命令并指定一个标记对象。

    Set-AzureRmResourceGroup -Name test-group -Tag @{ Dept="IT"; Environment="Test" }

该操作返回带有新的标记值的资源组。

    ResourceGroupName : test-group
    Location          : southcentralus
    ProvisioningState : Succeeded
    Tags              :
                    Name          Value
                    =======       =====
                    Dept          IT
                    Environment   Test
                    
可以使用 **Set-AzureRmResource** 命令向当前没有标记的资源添加标记。

    Set-AzureRmResource -Tag @{ Dept="IT"; Environment="Test" } -ResourceId /subscriptions/{guid}/resourceGroups/test-group/providers/Microsoft.Web/sites/examplemobileapp

标记作为一个整体更新。若要将一个标记添加到包含其他标记的资源，请使用数组，其中包含要保留的所有标记。首先选择现有标记，将其中一个标记添加到该集，然后重新应用所有标记。

    $tags = (Get-AzureRmResourceGroup -Name tag-demo).Tags
    $tags += @{Status="approved"}
    Set-AzureRmResourceGroup -Name test-group -Tag $tags

若要删除一个或多个标记，只需保存不包含您要删除的标记的数组。

该过程对于资源是相同的，不过，使用的是 **Get-AzureRmResource** 和 **Set-AzureRmResource** cmdlet。

若要使用 PowerShell 获取订阅中所有标记的列表，请使用 **Get-AzureRmTag** cmdlet。

    Get-AzureRmTag
    
将返回标记名称以及带有此标记的资源和资源组计数

    Name                      Count
    ----                      ------
    Dept                       8
    Environment                8

你可能会看到，有些标记以“hidden-”和“link:”开头。这些标记属于内部标记，应将其忽略并避免更改。

使用 **New-AzureRmTag** cmdlet 可将新标记添加到分类。即使这些标记尚未应用到任何资源或资源组，也会包含在自动填充内容中。若要删除某个标记名称/值，请先从它可能已应用到的所有资源中将它删除，然后使用 **Remove-AzureRmTag** cmdlet 从分类中将它删除。

### 低于 3.0.1 的版本

标记直接存在于资源和资源组中。若要查看现有标记，请使用 **Get-AzureRmResource** 查看资源，或使用 **Get-AzureRmResourceGroup** 查看资源组。

让我们从一个资源组着手。

    Get-AzureRmResourceGroup -Name testrg1

此 cmdlet 将返回有关资源组的元数据的多个片段，包括已应用了哪些标记（如果有）。

    ResourceGroupName : testrg1
    Location          : westus
    ProvisioningState : Succeeded
    Tags              :
                    Name         Value
                    ===========  ==========
                    Dept         Finance
                    Environment  Production
                    
若要检索资源元数据，请使用以下示例。资源元数据不会直接显示标记。

    Get-AzureRmResource -ResourceName tfsqlserver -ResourceGroupName testrg1

将在结果中看到，标记只作为 Hashtable 对象显示。

    Name              : tfsqlserver
    ResourceId        : /subscriptions/{guid}/resourceGroups/tag-demo-group/providers/Microsoft.Sql/servers/tfsqlserver
    ResourceName      : tfsqlserver
    ResourceType      : Microsoft.Sql/servers
    Kind              : v12.0
    ResourceGroupName : tag-demo-group
    Location          : westus
    SubscriptionId    : {guid}
    Tags              : {System.Collections.Hashtable}

可以通过检索 **Tags** 属性来查看实际标记。

    (Get-AzureRmResource -ResourceName tfsqlserver -ResourceGroupName tag-demo-group).Tags | %{ $_.Name + ": " + $_.Value }
   
该操作返回格式化结果：
    
    Dept: Finance
    Environment: Production
    
建议检索具有特定标记和值的所有资源或资源组，而不是查看特定资源组或资源的标记。若要获取具有特定标记的资源组，请结合 **-Tag** 参数使用 **Find-AzureRmResourceGroup** cmdlet。

若要检索带有标记值的资源组，请使用以下格式。

    Find-AzureRmResourceGroup -Tag @{ Name="Dept"; Value="Finance" } | %{ $_.Name }
    
若要获取具有特定标记和值的所有资源，请使用 Find-AzureRmResource cmdlet。

    Find-AzureRmResource -TagName Dept -TagValue Finance | %{ $_.ResourceName }

若要向当前没有标记的资源组添加标记，只需使用 Set-AzureRmResourceGroup 命令并指定一个标记对象。

    Set-AzureRmResourceGroup -Name test-group -Tag @( @{ Name="Dept"; Value="IT" }, @{ Name="Environment"; Value="Test"} )
    
该操作返回带有新的标记值的资源组。

    ResourceGroupName : test-group
    Location          : southcentralus
    ProvisioningState : Succeeded
    Tags              :
                Name          Value
                =======       =====
                Dept          IT
                Environment   Test

可以使用 Set-AzureRmResource 命令向当前没有标记的资源添加标记。

    Set-AzureRmResource -Tag @( @{ Name="Dept"; Value="IT" }, @{ Name="Environment"; Value="Test"} ) -ResourceId /subscriptions/{guid}/resourceGroups/test-group/providers/Microsoft.Web/sites/examplemobileapp

标记作为一个整体更新。若要将一个标记添加到包含其他标记的资源，请使用数组，其中包含要保留的所有标记。首先选择现有标记，将其中一个标记添加到该集，然后重新应用所有标记。

    $tags = (Get-AzureRmResourceGroup -Name tag-demo).Tags
    $tags += @{Name="status";Value="approved"}
    Set-AzureRmResourceGroup -Name test-group -Tag $tags

若要删除一个或多个标记，只需保存不包含您要删除的标记的数组。

该过程对于资源是相同的，除非使用的是 Get-AzureRmResource 和 Set-AzureRmResource cmdlet。

若要使用 PowerShell 获取订阅中所有标记的列表，请使用 **Get-AzureRmTag** cmdlet。

    Get-AzureRmTag
    
将返回标记名称以及带有此标记的资源和资源组计数

    Name                      Count
    ----                      ------
    Dept                       8
    Environment                8

你可能会看到，有些标记以“hidden-”和“link:”开头。这些标记属于内部标记，应将其忽略并避免更改。

使用 **New-AzureRmTag** cmdlet 可将新标记添加到分类。即使这些标记尚未应用到任何资源或资源组，也会包含在自动填充内容中。若要删除某个标记名称/值，请先从它可能已应用到的所有资源中将它删除，然后使用 **Remove-AzureRmTag** cmdlet 从分类中将它删除。


[powershell]: https://msdn.microsoft.com/zh-cn/library/mt619274(v=azure.200).aspx

<!---HONumber=Mooncake_0926_2016-->