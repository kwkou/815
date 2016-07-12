<properties
	pageTitle="使用 PowerShell 管理服务总线 | Azure"
	description="使用 PowerShell 脚本管理服务总线"
	services="service-bus"
	documentationCenter=".net"
	authors="sethmanheim"
	manager="timlt"
	editor=""/>

<tags
	ms.service="service-bus"
	ms.date="05/02/2016"
	wacn.date="06/21/2016"/>

# 使用 PowerShell 管理服务总线

## 概述

Azure PowerShell 是一个脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。本文介绍如何通过本地 Azure PowerShell 控制台使用 PowerShell 来设置和管理 Service Bus 实体（如命名空间、队列和事件中心）。

## 先决条件

在开始阅读本文前，你必须具有：

- Azure 订阅。Azure 是基于订阅的平台。有关获取订阅的详细信息，请参阅[购买选项]、[成员优惠]或[试用]。

- 配备 Azure PowerShell 的计算机。有关说明，请参阅[安装和配置 Azure PowerShell]。

- 大致了解 PowerShell 脚本、NuGet 包和 .NET Framework。

## 包含对适用于服务总线的 .NET 程序集的引用

通过有限数量的 PowerShell cmdlet 可以管理 Service Bus。若要预配未通过现有 cmdlet 公开的实体，可以对[服务总线 NuGet 包]中的服务总线使用 .NET 客户端。

首先，请确保该脚本可以找到随 NuGet 包一起安装的 **Microsoft.ServiceBus.dll** 程序集。为了灵活起见，该脚本执行以下步骤：

1. 确定调用它的路径。
2. 遍历路径直到找到名为 `packages` 的文件夹为止。该文件夹是在安装 NuGet 包时创建的。
3. 以递归方式在 `packages` 文件夹中搜索名为 **Microsoft.ServiceBus.dll** 的程序集。
4. 引用该程序集，以便类型可供以后使用。

下面说明如何在 PowerShell 脚本中实现这些步骤：

```
try
{
    # WARNING: Make sure to reference the latest version of Microsoft.ServiceBus.dll
    Write-Output "Adding the [Microsoft.ServiceBus.dll] assembly to the script..."
    $scriptPath = Split-Path (Get-Variable MyInvocation -Scope 0).Value.MyCommand.Path
    $packagesFolder = (Split-Path $scriptPath -Parent) + "\packages"
    $assembly = Get-ChildItem $packagesFolder -Include "Microsoft.ServiceBus.dll" -Recurse
    Add-Type -Path $assembly.FullName

    Write-Output "The [Microsoft.ServiceBus.dll] assembly has been successfully added to the script."
}

catch [System.Exception]
{
    Write-Error("Could not add the Microsoft.ServiceBus.dll assembly to the script. Make sure you build the solution before running the provisioning script.")
}
```

## 设置 Service Bus 命名空间

两个 PowerShell cmdlet 支持服务总线命名空间操作。你可以使用 [Get-AzureSBNamespace][] 和 [New-AzureSBNamespace][] 来代替 .NET SDK API。

本示例在脚本中创建几个本地变量：`$Namespace` 和 `$Location`。

- `$Namespace` 是我们要使用的 Service Bus 命名空间的名称。
- `$Location` 标识脚本要在其中预配命名空间的数据中心。
- `$CurrentNamespace` 将存储脚本检索（或创建）的引用命名空间。

在实际脚本中，`$Namespace` 和 `$Location` 可作为参数传递。

此部分脚本执行以下任务：

1. 尝试使用提供的名称检索 Service Bus 命名空间。
2. 如果找到该命名空间，则报告它找到的内容。
3. 如果找不到该命名空间，则会创建该命名空间，然后检索新创建的命名空间。


    	$Namespace = "MyServiceBusNS"
    	$Location = "China East"
    	
    	# Query to see if the namespace currently exists
    	$CurrentNamespace = Get-AzureSBNamespace -Name $Namespace
    	
    	# Check if the namespace already exists or needs to be created
    	if ($CurrentNamespace)
    	{
    	    Write-Output "The namespace [$Namespace] already exists in the [$($CurrentNamespace.Region)] region."
    	}
    	else
    	{
    	    Write-Host "The [$Namespace] namespace does not exist."
    	    Write-Output "Creating the [$Namespace] namespace in the [$Location] region..."
    	    New-AzureSBNamespace -Name $Namespace -Location $Location -CreateACSNamespace $false -NamespaceType Messaging
    	    $CurrentNamespace = Get-AzureSBNamespace -Name $Namespace
    	    Write-Host "The [$Namespace] namespace in the [$Location] region has been successfully created."
    	}


若要预配其他服务总线实体，请从 SDK 创建 [NamespaceManager][] 类的实例。可以使用 [Get-AzureSBAuthorizationRule][] cmdlet 来检索用于提供连接字符串的授权规则。我们将在 `$NamespaceManager` 变量中存储对 `NamespaceManager` 实例的引用。我们稍后将在脚本中使用 `$NamespaceManager` 来预配其他实体。


    $sbr = Get-AzureSBAuthorizationRule -Namespace $Namespace
    # Create the NamespaceManager object to create the event hub
    Write-Output "Creating a NamespaceManager object for the [$Namespace] namespace..."
    $NamespaceManager = [Microsoft.ServiceBus.NamespaceManager]::CreateFromConnectionString($sbr.ConnectionString);
    Write-Output "NamespaceManager object for the [$Namespace] namespace has been successfully created."


## 设置其他 Service Bus 实体

若要预配其他实体（如队列、主题和事件中心），可以使用[适用于服务总线的 .NET API][]。本文着重于事件中心，但对其他实体的步骤是类似的。此外，本文末尾提到了更多详细示例（包括其他实体）。

此部分脚本将再创建四个本地变量。这些变量用于实例化 `EventHubDescription` 对象。此脚本执行以下任务：

1. 使用 `NamespaceManager` 对象检查由 `$Path` 标识的事件中心是否存在。
2. 如果不存在，将创建 `EventHubDescription` 并将其传递到 `NamespaceManager` 类的 `CreateEventHubIfNotExists` 方法。
3. 确定事件中心可用后，请使用 `ConsumerGroupDescription` 和 `NamespaceManager` 创建使用者组。


    	$Path  = "MyEventHub"
    	$PartitionCount = 12
    	$MessageRetentionInDays = 7
    	$UserMetadata = $null
    	$ConsumerGroupName = "MyConsumerGroup"
    		
    	# Check to see if the Event Hub already exists
    	if ($NamespaceManager.EventHubExists($Path))
    	{
    	    Write-Output "The [$Path] event hub already exists in the [$Namespace] namespace."  
    	}
    	else
    	{
    	    Write-Output "Creating the [$Path] event hub in the [$Namespace] namespace: PartitionCount=[$PartitionCount] MessageRetentionInDays=[$MessageRetentionInDays]..."
    	    $EventHubDescription = New-Object -TypeName Microsoft.ServiceBus.Messaging.EventHubDescription -ArgumentList $Path
    	    $EventHubDescription.PartitionCount = $PartitionCount
    	    $EventHubDescription.MessageRetentionInDays = $MessageRetentionInDays
    	    $EventHubDescription.UserMetadata = $UserMetadata
    	    $EventHubDescription.Path = $Path
    	    $NamespaceManager.CreateEventHubIfNotExists($EventHubDescription);
    	    Write-Output "The [$Path] event hub in the [$Namespace] namespace has been successfully created."
    	}
    		
    	# Create the consumer group if it doesn't exist
    	Write-Output "Creating the consumer group [$ConsumerGroupName] for the [$Path] event hub..."
    	$ConsumerGroupDescription = New-Object -TypeName Microsoft.ServiceBus.Messaging.ConsumerGroupDescription -ArgumentList $Path, $ConsumerGroupName
    	$ConsumerGroupDescription.UserMetadata = $ConsumerGroupUserMetadata
    	$NamespaceManager.CreateConsumerGroupIfNotExists($ConsumerGroupDescription);
    	Write-Output "The consumer group [$ConsumerGroupName] for the [$Path] event hub has been successfully created."

## 将命名空间迁移到另一个 Azure 订阅

通过运行以下顺序的命令，可在 Azure 订阅之间移动命名空间。若要执行此操作，命名空间必须已经处于活动状态，而且运行 PowerShell 命令的用户必须既是源订阅又是目标订阅的管理员。

```
# Create a new resource group in target subscription
Select-AzureRmSubscription -SubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff'
New-AzureRmResourceGroup -Name 'targetRP' -Location 'China East'

# Move namespace from source subscription to target subscription
Select-AzureRmSubscription -SubscriptionId 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa'
$res = Find-AzureRmResource -ResourceNameContains mynamespace -ResourceType 'Microsoft.ServiceBus/namespaces'
Move-AzureRmResource -DestinationResourceGroupName 'targetRP' -DestinationSubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff' -ResourceId $res.ResourceId
```

## 后续步骤

本文提供使用 PowerShell 预配服务总线实体的基本概述。使用 .NET 客户端库可以执行的任何操作同样可以在 PowerShell 脚本中完成。

以下博客文章提供了更多详细示例：

- [如何使用 PowerShell 脚本创建 Service Bus 队列、主题和订阅](http://blogs.msdn.com/b/paolos/archive/2014/12/02/how-to-create-a-service-bus-queues-topics-and-subscriptions-using-a-powershell-script.aspx)
- [如何使用 PowerShell 脚本创建 Service Bus 命名空间和事件中心](http://blogs.msdn.com/b/paolos/archive/2014/12/01/how-to-create-a-service-bus-namespace-and-an-event-hub-using-a-powershell-script.aspx)

一些现成的脚本也可供下载：

- [服务总线 PowerShell 脚本](https://code.msdn.microsoft.com/windowsazure/Service-Bus-PowerShell-a46b7059)

<!--Link references-->
[购买选项]: http://azure.microsoft.com/zh-cn/pricing/purchase-options/
[成员优惠]: http://azure.microsoft.com/zh-cn/pricing/member-offers/
[试用]: /pricing/1rmb-trial/
[安装和配置 Azure PowerShell]: /documentation/articles/powershell-install-configure/
[服务总线 NuGet 包]: http://www.nuget.org/packages/WindowsAzure.ServiceBus/
[Get-AzureSBNamespace]: https://msdn.microsoft.com/zh-cn/library/azure/dn495122.aspx
[New-AzureSBNamespace]: https://msdn.microsoft.com/zh-cn/library/azure/dn495165.aspx
[Get-AzureSBAuthorizationRule]: https://msdn.microsoft.com/zh-cn/library/azure/dn495113.aspx
[适用于服务总线的 .NET API]: https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.aspx
[NamespaceManager]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx

<!---HONumber=82-->