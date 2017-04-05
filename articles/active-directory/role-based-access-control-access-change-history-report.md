<properties
    pageTitle="访问报告 - Azure RBAC | Azure"
    description="生成一个列出过去 90 天内使用基于角色的访问控制对 Azure 订阅的访问权限进行的所有更改的报告。"
    services="active-directory"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="" />
<tags
    ms.assetid="2bc68595-145e-4de3-8b71-3a21890d13d9"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="02/27/2017"
    wacn.date="04/05/2017"
    ms.author="kgremban" />  


# 为基于角色的访问控制创建访问报告
无论何时授予或撤销订阅中的访问权限，更改都将记录在 Azure 事件中。可创建访问权限更改历史记录报告来查看过去 90 天的所有更改。

## 使用 Azure PowerShell 创建报告
若要在 PowerShell 中创建访问权限更改历史记录报告，请使用 `Get-AzureRMAuthorizationChangeLog` 命令。有关此 cmdlet 的更多详细信息可在 [PowerShell 库](https://www.powershellgallery.com/packages/AzureRM.Storage/1.0.6/Content/ResourceManagerStartup.ps1)中找到。

调用此命令时，可指定你要列出的分配的属性，包括以下方面：

| 属性 | 说明 |
| --- | --- |
| **操作** |是授予还是撤销访问权限 |
| **调用方** |负责访问权限更改的所有者 |
| **日期** |访问权限更改的日期和时间 |
| **DirectoryName** |Azure Active Directory 目录 |
| **PrincipalName** |用户、组或应用程序的名称 |
| **PrincipalType** |分配是针对用户、组还是应用程序 |
| **RoleId** |已授予或已撤销角色的 GUID |
| **RoleName** |已授予或已撤销的角色 |
| **ScopeName** |订阅、资源组或资源的名称 |
| **ScopeType** |分配是在订阅、资源组中还是在资源范围内 |
| **SubscriptionId** |Azure 订阅的 GUID |
| **SubscriptionName** |Azure 订阅的名称 |

此示例命令列出过去 7 天内订阅中所有访问权限的更改：

	
	Get-AzureRMAuthorizationChangeLog -StartTime ([DateTime]::Now - [TimeSpan]::FromDays(7)) | FT Caller,Action,RoleName,PrincipalType,PrincipalName,ScopeType,ScopeName


![PowerShell Get-AzureRMAuthorizationChangeLog - 屏幕快照](./media/role-based-access-control-configure/access-change-history.png)

## 使用 Azure CLI 创建报告
若要在 Azure 命令行界面 (CLI) 中创建访问权限更改历史记录报告，请使用 `azure role assignment changelog list` 命令。

## 导出到电子表格
若要保存报告或处理数据，请将访问权限更改导出至 .csv 文件。可在电子表格中查看该报告以进行审阅。

![更改日志被视为电子表格 - 屏幕快照](./media/role-based-access-control-configure/change-history-spreadsheet.png)  


## 后续步骤
- 使用 [Azure RBAC 中的自定义角色](/documentation/articles/role-based-access-control-custom-roles/)
- 了解如何[使用 powershell 管理 Azure RBAC](/documentation/articles/role-based-access-control-manage-access-powershell/)

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->