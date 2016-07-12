<properties
	pageTitle="创建访问权限更改历史记录报告 | Microsoft Azure"
	description="生成一个列出过去 90 天使用基于角色的访问控制访问 Azure 订阅时所有更改的报告。"
	services="active-directory"
	documentationCenter=""
	authors="kgremban"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="04/28/2016"
	wacn.date="07/05/2016"/>

# 创建访问权限更改历史记录报告

如果你不是 Azure 订阅或其中的资源和资源组的唯一所有者，则需要能够记录所有的访问权限更改。无论何时授予或撤销订阅中的访问权限，更改都将记录在 Azure 事件中。可创建访问权限更改历史记录报告来查看过去 90 天的所有更改。

## 使用 Azure PowerShell 创建报告
若要在 PowerShell 中创建访问权限更改历史记录报告，请使用下列命令：


		Get-AzureRMAuthorizationChangeLog


可指定你要列出的分配的属性，包括以下方面：

| 属性 | 说明 |
| -------- | ----------- |
| **操作** | 是授予还是撤销访问权限 |
| **调用方** | 负责访问权限更改的所有者 |
| **日期** | 访问权限更改的日期和时间 |
| **DirectoryName** | Azure Active Directory 目录 |
| **PrincipalName** | 用户、组或应用程序的名称 |
| **PrincipalType** | 分配是针对用户、组还是应用程序 |
| **RoleId** | 已授予或已撤销角色的 GUID |
| **RoleName** | 已授予或已撤销的角色 |
| **ScopeName** | 订阅、资源组或资源的名称 |
| **ScopeType** | 分配是在订阅、资源组还是在资源范围 |
| **SubscriptionId** | Azure 订阅的 GUID |
| **SubscriptionName** | Azure 订阅的名称 |

此示例命令列出过去 7 天订阅中所有访问权限的更改：

		
		Get-AzureRMAuthorizationChangeLog -StartTime ([DateTime]::Now - [TimeSpan]::FromDays(7)) | FT Caller,Action,RoleName,PrincipalType,PrincipalName,ScopeType,ScopeName


![PowerShell Get-AzureRMAuthorizationChangeLog - 屏幕快照](./media/role-based-access-control-configure/access-change-history.png)

## 使用 Azure CLI 创建报告
若要在 Azure 命令行界面 (CLI) 中创建访问权限变更历史记录报告，请使用下列命令：

		azure authorization changelog


## 导出到电子表格
若要保存报告或处理数据，请将访问权限更改导出至 .csv 文件。可在电子表格中查看该报告以进行审阅。

![更改日志被视为电子表格 - 屏幕快照](./media/role-based-access-control-configure/change-history-spreadsheet.png)

## 另请参阅
- 开始使用 [Azure 基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)
- 使用 [Azure RBAC 中的自定义角色](/documentation/articles/role-based-access-control-custom-roles/)

<!---HONumber=Mooncake_0627_2016-->