<properties 
	pageTitle="使用资源管理器锁定资源 | Azure" 
	description="通过对所有用户和角色应用限制，来防止用户更新或删除特定的资源。" 
	services="azure-resource-manager" 
	documentationCenter="" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="azure-resource-manager" 
	ms.date="10/28/2015" 
	wacn.date="12/31/2015"/>

# 使用 Azure 资源管理器锁定资源

作为管理员，在以下情况下，你将需要在订阅、资源组或资源上放置锁定，以防止你组织中的其他用户提交写操作或意外删除关键资源。

Azure 资源管理器通过资源管理锁提供对资源限制操作的功能。锁是能够在特定范围内强制实施锁级别的策略。范围可能是订阅、资源组或资源。锁级别标识强制实施策略的类型，目前有两个值 - **CanNotDelete** 和 **ReadOnly**。**CanNotDelete** 表示经过授权的用户仍可以读取和修改资源，但不能删除任何受限制的资源。**ReadOnly** 表示经过授权的用户只能从资源中读取，但无法修改或删除任何受限制的资源。

锁不同于使用基于角色的访问控制向用户分配用来执行特定操作的权限。若要了解有关用户和角色的设置权限，请参阅[管理和审核对资源的访问权限](/documentation/articles/resource-group-rbac)。与基于角色的访问控制不同，你可以使用管理锁来对所有用户和角色应用限制，并且通常只会在有限的持续时间内应用锁。

## 常见方案

一个常见方案是在您的资源组中有一些是在关闭和打开模式下使用的资源。对 VM 资源开启以给定时间间隔定期处理数据然后关闭的功能。在此方案中，您想要启用关闭 VM，但又不得删除存储帐户。在此方案中，你将使用的资源锁对存储帐户采用的锁级别为 **CanNotDelete**。

在另一个方案中，您的业务可能有一些周期，在这些周期内您不想更新到生产环境。**ReadOnly** 锁级别停止创建或更新。如果是零售公司，你可能不想在假日购物期间进行更新。如果是金融服务公司，你可能必须在特定市场时间内对部署进行相关约束。资源锁可以视情况提供一个用来锁定资源的策略。这可以仅应用到特定资源或应用到整个资源组。

## 谁可以在组织中创建或删除锁

若要创建或删除管理锁，你必须有权访问 **Microsoft.Authorization/*** 或 **Microsoft.Authorization/locks/*** 操作。在内置角色中，只有**所有者**和**用户访问管理员**有权执行这些操作。有关分配访问控制的详细信息，请参阅[管理对资源的访问权限](/documentation/articles/resource-group-rbac)。

## 锁继承

在父作用域应用锁时，所有子资源将继承同一个锁。

如果你将多个锁应用于一个资源，则限制性最强的锁将具有最高优先级。例如，如果你在父级（例如资源组）应用 **ReadOnly** 并对该组中的资源应用 **CanNotDelete**，将会优先应用父级中限制性较强的锁 (ReadOnly)。

## 在模板上创建锁

以下示例演示在存储帐户上创建锁的模板。对其应用锁的存储帐户将作为参数提供，并配合 concat () 函数一起使用。结果是附有“Microsoft.Authorization”的资源名称，然后是该锁的名称（本例中为 **myLock**）。

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

你可以使用[管理锁的 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt204563.aspx) 锁定已部署的资源。REST API 使您可以创建和删除锁，并且检索有关现有锁的信息。

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

[AZURE.INCLUDE [powershell-preview-inline-include](../includes/powershell-preview-inline-include.md)]

你可以通过使用 **New-AzureRmResourceLock** 在 Azure PowerShell 中锁定已部署的资源，如下所示。通过 PowerShell，只能将 **LockLevel** 设置为 **CanNotDelete**。

    PS C:\> New-AzureRmResourceLock -LockLevel CanNotDelete -LockName LockSite -ResourceName examplesite -ResourceType Microsoft.Web/sites

Azure PowerShell 提供了其他用于使用锁的命令，如 **Set-AzureRmResourceLock** 用来更新锁，而 **Remove-AzureRmResourceLock** 用来删除锁。

## 后续步骤

- 有关使用资源锁的详细信息，请参阅 [向下锁定你的 Azure 资源](http://blogs.msdn.com/b/cloud_solution_architect/archive/2015/06/18/lock-down-your-azure-resources.aspx)
- 若要了解有关使用逻辑方式组织资源的信息，请参阅[使用标记来组织资源](/documentation/articles/resource-group-using-tags)。
- 若要更改资源位于哪个资源组，请参阅[将资源移到新的资源组](/documentation/articles/resource-group-move-resources)
- 你可以使用自定义策略对订阅应用限制和约定。有关详细信息，请参阅[使用策略来管理资源和控制访问](/documentation/articles/resource-manager-policy)。

<!---HONumber=Mooncake_1221_2015-->