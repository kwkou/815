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
	ms.date="04/25/2016" 
	wacn.date="06/20/2016"/>

# 使用 Azure 资源管理器锁定资源

作为管理员，你可能需要锁定订阅、资源组或资源，以防止组织中的其他用户意外删除关键资源。锁定后，经过授权的用户仍可以读取和修改资源，但不能删除资源。

不同于基于角色的访问控制，你可以使用管理锁对所有用户和角色应用限制。在父作用域应用锁时，所有子资源将继承同一个锁。

## 谁可以在组织中创建或删除锁

若要创建或删除管理锁，你必须有权访问 **Microsoft.Authorization/\*** 或 **Microsoft.Authorization/locks/\*** 操作。在内置角色中，只有**所有者**和**用户访问管理员**有权执行这些操作。

## 在模板上创建锁

以下示例演示在存储帐户上创建锁的模板。要对其应用锁的存储帐户将以参数形式提供。锁名是通过将包含 **/Microsoft.Authorization/** 的资源名称与锁名连接起来创建的（本例中为 **myLock**）。

提供的类型特定于资源类型。对于存储，此类型是“Microsoft.Storage/storageaccounts/providers/locks”。

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

    PUT https://management.chinacloudapi.cn/{scope}/providers/Microsoft.Authorization/locks/{lock-name}?api-version={api-version}

作用域可能是订阅、资源组或资源。锁名称可以是您想要对该锁使用的任何称谓。对于 api 版本，请使用 **2015-01-01**。

在请求中，包括指定锁属性的 JSON 对象。

    {
      "properties": {
        "level": "CanNotDelete",
        "notes": "Optional text notes."
      }
    } 

有关示例，请参阅[管理锁的 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt204563.aspx)。

## 使用 Azure PowerShell 创建一个锁

你可以通过使用 **New-AzureRmResourceLock** 在 Azure PowerShell 中锁定已部署的资源，如下所示。

    New-AzureRmResourceLock -LockLevel CanNotDelete -LockName LockSite -ResourceName examplesite -ResourceType Microsoft.Web/sites

Azure PowerShell 提供了其他用于使用锁的命令，如 **Set-AzureRmResourceLock** 用来更新锁，而 **Remove-AzureRmResourceLock** 用来删除锁。

## 后续步骤

- 有关使用资源锁的详细信息，请参阅 [向下锁定你的 Azure 资源](http://blogs.msdn.com/b/cloud_solution_architect/archive/2015/06/18/lock-down-your-azure-resources.aspx)
- 若要了解有关使用逻辑方式组织资源的信息，请参阅[使用标记来组织资源](/documentation/articles/resource-group-using-tags/)。
- 若要更改资源位于哪个资源组，请参阅[将资源移到新的资源组](/documentation/articles/resource-group-move-resources/)

<!---HONumber=Mooncake_0425_2016-->