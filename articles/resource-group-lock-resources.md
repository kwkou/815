<properties 
	pageTitle="使用 Azure 资源管理器锁定资源" 
	description="用于防止用户更新或删除某些资源的锁资源。" 
	services="azure-resource-manager" 
	documentationCenter="" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="azure-resource-manager" 
	ms.date="08/10/2015" 
	wacn.date="10/3/2015"/>

# 使用 Azure 资源管理器锁定资源

作为管理员，在以下情况下，您将需要在资源或资源组上放置锁定，以防止您组织中的其他用户提交写操作或意外删除关键资源。

Azure 资源管理器通过资源管理锁提供对资源限制操作的功能。资源锁是能够在特定范围内强制实施锁级别的策略。锁级别标识强制实施策略的类型，目前有两个值 - **CanNotDelete** 和 **ReadOnly**。可将范围表示为一个 URI，并且可以是一个资源或一个资源组。

锁不同于向用户分配用来执行某些操作的权限。若要了解有关用户和角色的设置权限，请参阅<!--[-->预览门户中的基于角色的访问控制<!--](/documentation/articles/role-based-access-control-configure)-->和[管理和审核对资源的访问权限](/documentation/articles/resource-group-rbac)。

## 常见方案

一个常见方案是在您的资源组中有一些是在关闭和打开模式下使用的资源。对 VM 资源开启以给定时间间隔定期处理数据然后关闭的功能。在此方案中，您想要启用关闭 VM，但又不得删除存储帐户。在此方案中，您将使用的资源锁对存储帐户采用的锁级别为 **CanNotDelete**。

在另一个方案中，您的业务可能有一些周期，在这些周期内您不想更新到生产环境。**ReadOnly** 锁级别停止创建或更新。如果是零售公司，您可能不想在假日购物期间进行更新。如果是金融服务公司，您可能必须在特定市场时间内对部署进行相关约束。资源锁可以视情况提供一个用来锁定资源的策略。这可以仅应用到特定资源或应用到整个资源组。

## 在模板上创建锁

以下示例演示在存储帐户上创建锁的模板。对其应用锁的存储帐户将作为参数提供，并配合 concat () 函数一起使用。结果是附有 ¡®Microsoft.Authorization¡¯ 的资源名称，然后是该锁的名称（本例中为 **myLock**）。

提供的类型特定于资源类型。对于存储，此类型是“Microsoft.Storage/storageaccounts/providers/locks”。

使用资源的 **level** 属性设置作用域级别。因为该示例侧重帮助避免意外删除，所以该级别被设置为 **CannotDelete**。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "lockedResource": {
                "type": "string"
            }
        },
        "resources": [
            {
                "name": "[concat(parameters('lockedResource'), '/Microsoft.Authorization/myLock')]",
                "type": "Microsoft.Storage/storageAccounts/providers/locks",
                "apiVersion": "2015-01-01",
                "properties": {
	                "level": "CannotDelete"
                }
            }
        ]
    }

## 使用 REST API 创建锁

您可以使用[管理锁的 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt204563.aspx) 锁定已部署的资源。REST API 使您可以创建和删除锁，并且检索有关现有锁的信息。

若要创建一个锁，请运行：

    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/locks/{lock-name}?api-version={api-version}

作用域可能是订阅、资源组或资源。锁名称可以是您想要对该锁使用的任何称谓。对于 api 版本，请使用 **2015-01-01**。

在请求中，包括指定锁属性的 JSON 对象。

    {
        "properties": {
            "level": {lock-level},
            "notes": "Optional text notes."
        }
    } 

对于锁级别，指定 **CanNotDelete** 或 **ReadOnly**。

有关示例，请参阅[管理锁的 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt204563.aspx)。

## 使用 Azure PowerShell 创建一个锁

您可以通过使用 **New-AzureResourceLock** 在 Azure PowerShell 中锁定已部署的资源，如下所示。通过 PowerShell，只能将 **LockLevel** 设置为 **CanNotDelete**。

    PS C:\> New-AzureResourceLock -LockLevel CanNotDelete -LockName LockSite -ResourceName examplesite -ResourceType Microsoft.Web/sites -ResourceGroupName ExampleGroup

PowerShell 提供了其他用于使用锁的命令，如 **Set-AzureResourceLock** 用来更新锁，而 **Remove-AzureResourceLock** 用来删除锁。

## 后续步骤

- 有关使用资源锁的详细信息，请参阅 [向下锁定您的 Azure 资源](http://blogs.msdn.com/b/cloud_solution_architect/archive/2015/06/18/lock-down-your-azure-resources.aspx)
- 若要了解有关使用逻辑方式组织资源的信息，请参阅[使用标记来组织资源](/documentation/articles/resource-group-using-tags)。
- 若要更改资源位于哪个资源组，请参阅[将资源移到新的资源组](/documentation/articles/resource-group-move-resources)

<!---HONumber=71-->