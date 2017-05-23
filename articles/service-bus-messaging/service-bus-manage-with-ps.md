<properties
    pageTitle="使用 PowerShell 管理 Azure 服务总线资源 | Azure"
    description="使用 PowerShell 模块创建和管理服务总线资源"
    services="service-bus"
    documentationcenter=".NET"
    author="sethmanheim"
    manager="timlt"
    editor="" />
<tags
    ms.assetid=""
    ms.service="service-bus"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="04/06/2017"
    wacn.date="05/22/2017"
    ms.author="sethm"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="b84ae9d37a62d7f9634652215ad5ecc082a3ae7b"
    ms.lasthandoff="05/12/2017" />

# <a name="use-powershell-to-manage-service-bus-resources"></a>使用 PowerShell 管理服务总线资源

Microsoft Azure PowerShell 是一个脚本编写环境，可用于控制和自动执行 Azure 服务的部署和管理。 本文介绍如何通过本地 Azure PowerShell 控制台或脚本，使用[服务总线 Resource Manager PowerShell 模块](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/?view=azurermps-3.7.0#service_bus)来预配和管理服务总线实体（命名空间、队列、主题和订阅）。

还可以使用 Azure Resource Manager 模板管理服务总线实体。 有关详细信息，请参阅[使用 Azure Resource Manager 模板创建服务总线资源](/documentation/articles/service-bus-resource-manager-overview/)一文。

## <a name="prerequisites"></a>先决条件

在开始之前，你需要具备以下项：

* Azure 订阅。 
* 配备 Azure PowerShell 的计算机。 有关说明，请参阅 [Azure PowerShell cmdlet 入门](/documentation/articles/powershell-install-configure/)。
* 大致了解 PowerShell 脚本、NuGet 包和 .NET Framework。

## <a name="get-started"></a>入门

第一步是使用 PowerShell 登录 Azure 帐户和 Azure 订阅。 按照 [Azure PowerShell cmdlet 入门](/documentation/articles/powershell-install-configure/)中的说明登录 Azure 帐户，检索并访问 Azure 订阅中的资源。

## <a name="provision-a-service-bus-namespace"></a>设置 Service Bus 命名空间

使用服务总线命名空间时，可以使用 [Get-AzureRmServiceBusNamespace](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/get-azurermservicebusnamespace?view=azurermps-4.0.0)、[New-AzureRmServiceBusNamespace](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/new-azurermservicebusnamespace?view=azurermps-4.0.0)、[Remove-AzureRmServiceBusNamespace](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/remove-azurermservicebusnamespace?view=azurermps-4.0.0) 和 [Set-AzureRmServiceBusNamespace](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/set-azurermservicebusnamespace?view=azurermps-4.0.0) cmdlet。

本示例在脚本中创建几个本地变量：`$Namespace` 和 `$Location`。

* `$Namespace` 是要使用的服务总线命名空间的名称。
* `$Location` 标识我们要在其中设置命名空间的数据中心。
* `$CurrentNamespace` 将存储我们检索（或创建）的引用命名空间。

在实际脚本中，`$Namespace` 和 `$Location` 可作为参数传递。

此部分脚本执行以下任务：

1. 尝试使用指定名称检索服务总线命名空间。
2. 如果找到该命名空间，则报告它找到的内容。
3. 如果找不到该命名空间，则会创建该命名空间，然后检索新创建的命名空间。

        # Query to see if the namespace currently exists
        $CurrentNamespace = Get-AzureRMServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
   
        # Check if the namespace already exists or needs to be created
        if ($CurrentNamespace)
        {
            Write-Host "The namespace $Namespace already exists in the $Location region:"
            # Report what was found
            Get-AzureRMServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
        }
        else
        {
            Write-Host "The $Namespace namespace does not exist."
            Write-Host "Creating the $Namespace namespace in the $Location region..."
            New-AzureRmServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace -Location $Location
            $CurrentNamespace = Get-AzureRMServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
            Write-Host "The $Namespace namespace in Resource Group $ResGrpName in the $Location region has been successfully created."
                
        }

### <a name="create-a-namespace-authorization-rule"></a>创建命名空间授权规则

下面的示例演示如何使用 [New-AzureRmServiceBusNamespaceAuthorizationRule](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/new-azurermservicebusnamespaceauthorizationrule?view=azurermps-4.0.0)、[Get-AzureRmServiceBusNamespaceAuthorizationRule](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/get-azurermservicebusnamespaceauthorizationrule?view=azurermps-4.0.0)、[Set-AzureRmServiceBusNamespaceAuthorizationRule](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/set-azurermservicebusnamespaceauthorizationrule?view=azurermps-4.0.0) 和 [Remove-AzureRmServiceBusNamespaceAuthorizationRule](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/remove-azurermservicebusnamespaceauthorizationrule?view=azurermps-4.0.0) cmdlet 管理命名空间授权规则。

    # Query to see if rule exists
    $CurrentRule = Get-AzureRmServiceBusNamespaceAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule

    # Check if the rule already exists or needs to be created
    if ($CurrentRule)
    {
        Write-Host "The $AuthRule rule already exists for the namespace $Namespace."
    }
    else
    {
        Write-Host "The $AuthRule rule does not exist."
        Write-Host "Creating the $AuthRule rule for the $Namespace namespace..."
        New-AzureRmServiceBusNamespaceAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule -Rights @("Listen","Send")
        $CurrentRule = Get-AzureRmServiceBusNamespaceAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule
        Write-Host "The $AuthRule rule for the $Namespace namespace has been successfully created."

        Write-Host "Setting rights on the namespace"
        $authRuleObj = Get-AzureRmServiceBusNamespaceAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule

        Write-Host "Remove Send rights"
        $authRuleObj.Rights.Remove("Send")
        Set-AzureRmServiceBusNamespaceAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj

        Write-Host "Add Send and Manage rights to the namespace"
        $authRuleObj.Rights.Add("Send")
        Set-AzureRmServiceBusNamespaceAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj
        $authRuleObj.Rights.Add("Manage")
        Set-AzureRmServiceBusNamespaceAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj

        Write-Host "Show value of primary key"
        $CurrentKey = Get-AzureRmServiceBusNamespaceKey -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule
        
        Write-Host "Remove this authorization rule"
        Remove-AzureRmServiceBusNamespaceAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule
    }

## <a name="create-a-queue"></a>创建队列

若要创建队列或主题，请使用上一部分中的脚本执行命名空间检查。 然后，创建队列：

    # Check if queue already exists
    $CurrentQ = Get-AzureRmServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName

    if($CurrentQ)
    {
        Write-Host "The queue $QueueName already exists in the $Location region:"
    }
    else
    {
        Write-Host "The $QueueName queue does not exist."
        Write-Host "Creating the $QueueName queue in the $Location region..."
        New-AzureRmServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName -EnablePartitioning $True
        $CurrentQ = Get-AzureRmServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName
        Write-Host "The $QueueName queue in Resource Group $ResGrpName in the $Location region has been successfully created."
    }

### <a name="modify-queue-properties"></a>修改队列属性

执行上一部分中的脚本后，可以使用 [Set-AzureRmServiceBusQueue](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/set-azurermservicebusqueue?view=azurermps-4.0.0) cmdlet 更新队列的属性，如以下示例所示：

    $CurrentQ.DeadLetteringOnMessageExpiration = $True
    $CurrentQ.MaxDeliveryCount = 7
    $CurrentQ.MaxSizeInMegabytes = 2048
    $CurrentQ.EnableExpress = $True

    Set-AzureRmServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName -QueueObj $CurrentQ

## <a name="provisioning-other-service-bus-entities"></a>设置其他 Service Bus 实体

可以使用[服务总线 PowerShell 模块](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/?view=azurermps-3.7.0#service_bus)预配其他实体，例如主题和订阅。 这些 cmdlet 在语法上与上一部分所示的队列创建 cmdlet 类似。

## <a name="next-steps"></a>后续步骤

- 有关服务总线 Resource Manager PowerShell 模块的完整文档，请参阅[此处](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/?view=azurermps-3.7.0#service_bus)。 此页列出所有可用的 cmdlet。
- 有关使用 Azure Resource Manager 模板的信息，请参阅[使用 Azure Resource Manager 模板创建服务总线资源](/documentation/articles/service-bus-resource-manager-overview/)一文。
- 有关[服务总线 .NET 管理库](/documentation/articles/service-bus-management-libraries/)的信息。

这些博客文章介绍管理服务总线实体的一些备选方法：

* [How to create Service Bus queues, topics and subscriptions using a PowerShell script（如何使用 PowerShell 脚本创建服务总线队列、主题和订阅）](http://blogs.msdn.com/b/paolos/archive/2014/12/02/how-to-create-a-service-bus-queues-topics-and-subscriptions-using-a-powershell-script.aspx)
* [如何使用 PowerShell 脚本创建 Service Bus 命名空间和事件中心](http://blogs.msdn.com/b/paolos/archive/2014/12/01/how-to-create-a-service-bus-namespace-and-an-event-hub-using-a-powershell-script.aspx)
* [服务总线 PowerShell 脚本](https://code.msdn.microsoft.com/Service-Bus-PowerShell-a46b7059)

<!--Anchors-->