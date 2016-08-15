<properties
   pageTitle="使用 PowerShell 管理 Service Bus 和事件中心资源"
   description="使用 PowerShell 创建和管理 Service Bus 和事件中心资源"
   services="service-bus"
   documentationCenter=".NET"
   authors="sethmanheim"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-bus"
   ms.date="03/09/2016"
   wacn.date="01/14/2016"/>

# 使用 PowerShell 管理 Service Bus 和事件中心资源

Azure PowerShell 是一个脚本编写环境，可用于控制和自动执行 Azure 服务的部署和管理。本文介绍如何通过本地 Azure PowerShell 控制台使用 PowerShell 来设置和管理 Service Bus 实体（如命名空间、队列和事件中心）。

## 先决条件

在开始之前，你需要具备以下项：

- Azure 订阅。Azure 是基于订阅的平台。有关获取订阅的详细信息，请参阅[购买选项][试用版]。

- 配备 Azure PowerShell 的计算机。有关说明，请参阅[安装和配置 Azure PowerShell][]。

- 大致了解 PowerShell 脚本、NuGet 包和 .NET Framework。

## 包含对用于 Service Bus 的 .NET 程序集的引用

通过有限数量的 PowerShell cmdlet 可以管理 Service Bus。若要设置未通过现有 cmdlet 公开的实体，可以通过引用 [Service Bus NuGet 包]从 PowerShell 中使用用于 Service Bus 的 .NET 客户端。

首先，请确保该脚本可以找到随 NuGet 包一起安装的 **Microsoft.ServiceBus.dll** 程序集。为了灵活起见，该脚本执行以下步骤：

1. 确定调用它的路径。
2. 遍历路径直到找到名为 `packages` 的文件夹为止。该文件夹是在安装 NuGet 包时创建的。
3. 以递归方式在 `packages` 文件夹中搜索名为 **Microsoft.ServiceBus.dll** 的程序集。
4. 引用该程序集，以便类型可供以后使用。

下面说明如何在 PowerShell 脚本中实现这些步骤：

```powershell

try
{
    # IMPORTANT: Make sure to reference the latest version of Microsoft.ServiceBus.dll
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

使用 Service Bus 命名空间时，你可以使用两个 cmdlet：[Get-AzureSBNamespace] 和 [New-AzureSBNamespace]，而不是 .NET SDK。

本示例在脚本中创建几个本地变量：`$Namespace` 和 `$Location`。

- `$Namespace` 是我们要使用的 Service Bus 命名空间的名称。
- `$Location` 标识我们要在其中设置命名空间的数据中心。
- `$CurrentNamespace` 将存储我们检索（或创建）的引用命名空间。

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
		    New-AzureSBNamespace -Name $Namespace -Location $Location -CreateACSNamespace false -NamespaceType Messaging
		    $CurrentNamespace = Get-AzureSBNamespace -Name $Namespace
		    Write-Host "The [$Namespace] namespace in the [$Location] region has been successfully created."
		}

若要设置其他 Service Bus 实体，请从 SDK 创建 `NamespaceManager` 对象的一个实例。可以使用 [Get-AzureSBAuthorizationRule] cmdlet 来检索用于提供连接字符串的授权规则。此示例在 `$NamespaceManager` 变量中存储对 `NamespaceManager` 实例的引用。此脚本稍后将使用 `$NamespaceManager` 来设置其他实体。

	$sbr = Get-AzureSBAuthorizationRule -Namespace $Namespace
	# Create the NamespaceManager object to create the Event Hub
	Write-Output "Creating a NamespaceManager object for the [$Namespace] namespace..."
	$NamespaceManager = [Microsoft.ServiceBus.NamespaceManager]::CreateFromConnectionString($sbr.ConnectionString);
	Write-Output "NamespaceManager object for the [$Namespace] namespace has been successfully created."

## 设置其他 Service Bus 实体

若要设置其他实体（如队列、主题和事件中心），可以使用 [.NET API for Service Bus]。更多详细示例（包括其他实体）将在本文末尾提及。

### 创建事件中心

此部分脚本将再创建四个本地变量。这些变量用于实例化 `EventHubDescription` 对象。此脚本执行以下任务：

1. 使用 `NamespaceManager` 对象检查由 `$Path` 标识的事件中心是否存在。
2. 如果不存在，将创建 `EventHubDescription` 并将其传递到 `NamespaceManager` 类的 `CreateEventHubIfNotExists` 方法。
3. 确定事件中心可用后，请使用 `ConsumerGroupDescription` 和 `NamespaceManager` 创建使用者组。



    	$Path  = "MyEventHub"
    	$PartitionCount = 12
    	$MessageRetentionInDays = 7
    	$UserMetadata = $null
    	$ConsumerGroupName = "MyConsumerGroup"
    
    	# Check if the Event Hub already exists
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

### 创建队列

若要创建队列或主题，请按前一部分所述执行命名空间检查。

	if ($NamespaceManager.QueueExists($Path))
	{
	    Write-Output "The [$Path] queue already exists in the [$Namespace] namespace."
	}
	else
	{
	    Write-Output "Creating the [$Path] queue in the [$Namespace] namespace..."
	    $QueueDescription = New-Object -TypeName Microsoft.ServiceBus.Messaging.QueueDescription -ArgumentList $Path
	    if ($AutoDeleteOnIdle -ge 5)
	    {
	        $QueueDescription.AutoDeleteOnIdle = [System.TimeSpan]::FromMinutes($AutoDeleteOnIdle)
	    }
	    if ($DefaultMessageTimeToLive -gt 0)
	    {
	        $QueueDescription.DefaultMessageTimeToLive = [System.TimeSpan]::FromMinutes($DefaultMessageTimeToLive)
	    }
	    if ($DuplicateDetectionHistoryTimeWindow -gt 0)
	    {
	        $QueueDescription.DuplicateDetectionHistoryTimeWindow = [System.TimeSpan]::FromMinutes($DuplicateDetectionHistoryTimeWindow)
	    }
	    $QueueDescription.EnableBatchedOperations = $EnableBatchedOperations
	    $QueueDescription.EnableDeadLetteringOnMessageExpiration = $EnableDeadLetteringOnMessageExpiration
	    $QueueDescription.EnableExpress = $EnableExpress
	    $QueueDescription.EnablePartitioning = $EnablePartitioning
	    $QueueDescription.ForwardDeadLetteredMessagesTo = $ForwardDeadLetteredMessagesTo
	    $QueueDescription.ForwardTo = $ForwardTo
	    $QueueDescription.IsAnonymousAccessible = $IsAnonymousAccessible
	    if ($LockDuration -gt 0)
	    {
	        $QueueDescription.LockDuration = [System.TimeSpan]::FromSeconds($LockDuration)
	    }
	    $QueueDescription.MaxDeliveryCount = $MaxDeliveryCount
	    $QueueDescription.MaxSizeInMegabytes = $MaxSizeInMegabytes
	    $QueueDescription.RequiresDuplicateDetection = $RequiresDuplicateDetection
	    $QueueDescription.RequiresSession = $RequiresSession
	    if ($EnablePartitioning)
	    {
	        $QueueDescription.SupportOrdering = $False
	    }
	    else
	    {
	        $QueueDescription.SupportOrdering = $SupportOrdering
	    }
	    $QueueDescription.UserMetadata = $UserMetadata
	    $NamespaceManager.CreateQueue($QueueDescription);
	}

### 创建主题

	if ($NamespaceManager.TopicExists($Path))
	{
	    Write-Output "The [$Path] topic already exists in the [$Namespace] namespace."
	}
	else
	{
	    Write-Output "Creating the [$Path] topic in the [$Namespace] namespace..."
	    $TopicDescription = New-Object -TypeName Microsoft.ServiceBus.Messaging.TopicDescription -ArgumentList $Path
	    if ($AutoDeleteOnIdle -ge 5)
	    {
	        $TopicDescription.AutoDeleteOnIdle = [System.TimeSpan]::FromMinutes($AutoDeleteOnIdle)
	    }
	    if ($DefaultMessageTimeToLive -gt 0)
	    {
	        $TopicDescription.DefaultMessageTimeToLive = [System.TimeSpan]::FromMinutes($DefaultMessageTimeToLive)
	    }
	    if ($DuplicateDetectionHistoryTimeWindow -gt 0)
	    {
	        $TopicDescription.DuplicateDetectionHistoryTimeWindow = [System.TimeSpan]::FromMinutes($DuplicateDetectionHistoryTimeWindow)
	    }
	    $TopicDescription.EnableBatchedOperations = $EnableBatchedOperations
	    $TopicDescription.EnableExpress = $EnableExpress
	    $TopicDescription.EnableFilteringMessagesBeforePublishing = $EnableFilteringMessagesBeforePublishing
	    $TopicDescription.EnablePartitioning = $EnablePartitioning
	    $TopicDescription.IsAnonymousAccessible = $IsAnonymousAccessible
	    $TopicDescription.MaxSizeInMegabytes = $MaxSizeInMegabytes
	    $TopicDescription.RequiresDuplicateDetection = $RequiresDuplicateDetection
	    if ($EnablePartitioning)
	    {
	        $TopicDescription.SupportOrdering = $False
	    }
	    else
	    {
	        $TopicDescription.SupportOrdering = $SupportOrdering
	    }
	    $TopicDescription.UserMetadata = $UserMetadata
	    $NamespaceManager.CreateTopic($TopicDescription);
	}

## 后续步骤

本文提供使用 PowerShell 预配服务总线实体的基本概述。尽管通过引用 Microsoft.ServiceBus.dll 程序集有有限数量的 PowerShell cmdlet 可用于管理 Service Bus 消息实体，但可以使用 .NET 客户端库执行的任何操作几乎也可以在 PowerShell 脚本中执行。

以下博客文章提供了更多详细示例：

- [如何使用 PowerShell 脚本创建 Service Bus 队列、主题和订阅](http://blogs.msdn.com/b/paolos/archive/2014/12/02/how-to-create-a-service-bus-queues-topics-and-subscriptions-using-a-powershell-script.aspx)
- [如何使用 PowerShell 脚本创建 Service Bus 命名空间和事件中心](http://blogs.msdn.com/b/paolos/archive/2014/12/01/how-to-create-a-service-bus-namespace-and-an-event-hub-using-a-powershell-script.aspx)

一些现成的脚本也可供下载：
- [服务总线 PowerShell 脚本](https://code.msdn.microsoft.com/windowsazure/Service-Bus-PowerShell-a46b7059)

<!--Anchors-->

[购买选项]: /pricing/overview/
[试用版]: /pricing/1rmb-trial/
[Service Bus NuGet 包]: http://www.nuget.org/packages/WindowsAzure.ServiceBus/
[Get-AzureSBNamespace]: https://msdn.microsoft.com/zh-cn/library/azure/dn495122.aspx
[New-AzureSBNamespace]: https://msdn.microsoft.com/zh-cn/library/azure/dn495165.aspx
[Get-AzureSBAuthorizationRule]: https://msdn.microsoft.com/zh-cn/library/azure/dn495113.aspx
[.NET API for Service Bus]: https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.aspx
[安装和配置 Azure PowerShell]: /documentation/articles/powershell-install-configure/

<!---HONumber=Mooncake_0104_2016-->