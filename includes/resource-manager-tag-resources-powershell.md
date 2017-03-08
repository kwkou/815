AzureRm.Resources 模块版本 3.0 包括在使用标记方式方面的重大更改。在继续之前，请查看版本：

    Get-Module -ListAvailable -Name AzureRm.Resources | Select Version

如果结果显示版本 3.0 或更高版本，则本主题中的示例可用于你的环境。如果没有版本 3.0 或更高版本，请使用 PowerShell 库或 Web 平台安装程序[更新版本](https://docs.microsoft.com/powershell/azureps-cmdlets-docs/)，然后再继续完成本主题。

    Version
    -------
    3.5.0

每次将标记应用到资源或资源组时，都会覆盖该资源或资源组上的现有标记。因此，必须根据资源或资源组是否具有要保留的现有标记，使用不同的方法。若要将标记添加到：

* 不带现有标记的资源组。

    Set-AzureRmResourceGroup -Name TagTestGroup -Tag @{ Dept="IT"; Environment="Test" }

* 带有现有标记的资源组。

    $tags = (Get-AzureRmResourceGroup -Name TagTestGroup).Tags
    $tags += @{Status="Approved"}
    Set-AzureRmResourceGroup -Tag $tags -Name TagTestGroup

* 不带现有标记的资源。

    Set-AzureRmResource -Tag @{ Dept="IT"; Environment="Test" } -ResourceName storageexample -ResourceGroupName TagTestGroup -ResourceType Microsoft.Storage/storageAccounts

* 带有现有标记的资源。

    $tags = (Get-AzureRmResource -ResourceName storageexample -ResourceGroupName TagTestGroup).Tags
    $tags += @{Status="Approved"}
    Set-AzureRmResource -Tag $tags -ResourceName storageexample -ResourceGroupName TagTestGroup -ResourceType Microsoft.Storage/storageAccounts

    若要将资源组中的所有标记应用于其资源，并且**不保留资源上的现有标记**，请使用以下脚本：

    $groups = Get-AzureRmResourceGroup
    foreach ($g in $groups) 
    {
        Find-AzureRmResource -ResourceGroupNameEquals $g.ResourceGroupName | ForEach-Object {Set-AzureRmResource -ResourceId $_.ResourceId -Tag $g.Tags -Force } 
    }

    若要将资源组中的所有标记应用于其资源，并且**保留资源上不重复的现有标记**，请使用以下脚本：

    $groups = Get-AzureRmResourceGroup
    foreach ($g in $groups) 
    {
        if ($g.Tags -ne $null) {
            $resources = Find-AzureRmResource -ResourceGroupNameEquals $g.ResourceGroupName 
            foreach ($r in $resources)
            {
                $resourcetags = (Get-AzureRmResource -ResourceId $r.ResourceId).Tags
                foreach ($key in $g.Tags.Keys)
                {
                    if ($resourcetags.ContainsKey($key)) { $resourcetags.Remove($key) }
                }
                $resourcetags += $g.Tags
                Set-AzureRmResource -Tag $resourcetags -ResourceId $r.ResourceId -Force
            }
        }
    }

若要删除所有标记，请传递一个空哈希表。

    Set-AzureRmResourceGroup -Tag @{} -Name TagTestGgroup

若要获取具有特定标记的资源组，请使用 `Find-AzureRmResourceGroup` cmdlet。

    (Find-AzureRmResourceGroup -Tag @{ Dept="Finance" }).Name 

若要获取具有特定标记和值的所有资源，请使用 `Find-AzureRmResource` cmdlet。

    (Find-AzureRmResource -TagName Dept -TagValue Finance).Name

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description:update meta properties; wording update; add the AzureRM.Resouce code with PowerShell version 3.5 -->