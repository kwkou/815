<properties 
	pageTitle="使用 PowerShell 部署和管理通知中心" 
	description="如何使用自动化 PowerShell 创建和管理通知中心" 
	services="notification-hubs" 
	documentationCenter="" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="notification-hubs" 
	ms.date="06/18/2015" 
	wacn.date="11/02/2015"/>

# 使用 PowerShell 部署和管理通知中心

## 概述

本文说明如何使用 PowerShell 创建和管理 Azure 通知中心。本主题将演示以下常见自动化任务。

+ 创建通知中心
+ 设置凭据

<!--如果您还需要为通知中心创建新的服务总线命名空间，请参阅[使用 PowerShell 管理服务总线](/documentation/articles/service-bus-powershell-how-to-provision)。-->

不支持直接使用 Azure PowerShell 随附的 cmdlet 来管理通知中心。在 PowerShell 中，最佳方法是引用 Microsoft.ServiceBus.dll 程序集。该程序集是随[服务总线 NuGet 包](http://www.nuget.org/packages/WindowsAzure.ServiceBus/)一起分发的。


## 先决条件

在开始阅读本文前，您必须具有：

- Azure 订阅。Azure 是基于订阅的平台。有关获取订阅的详细信息，请参阅[购买选项]、<!-- [成员优惠]或-->[免费试用]。

- 配备 Azure PowerShell 的计算机。有关说明，请参阅[安装和配置 Azure PowerShell]。

- 大致了解 PowerShell 脚本、NuGet 包和 .NET Framework。


## 包含对适用于服务总线的 .NET 程序集的引用

Azure PowerShell 中的 PowerShell cmdlet 尚不支持管理 Azure 通知中心。若要设置通知中心和其他不是通过现有 cmdlet 公开的服务总线实体，您可以使用[服务总线 NuGet 包](http://www.nuget.org/packages/WindowsAzure.ServiceBus/)中适用于服务总线的 .NET 客户端。

首先，请确保脚本可以找到 **Microsoft.ServiceBus.dll** 程序集，该程序集在 Visual Studio 项目中以 NuGet 包的形式安装。为了灵活起见，该脚本执行以下步骤：

1. 确定调用它的路径。
2. 遍历路径直到找到名为 `packages` 的文件夹为止。此文件夹是在为 Visual Studio 项目安装 NuGet 包时创建的。
3. 以递归方式在 `packages` 文件夹中搜索名为 **Microsoft.ServiceBus.dll** 的程序集。
4. 引用该程序集，以便类型可供以后使用。

下面说明如何在 PowerShell 脚本中实现这些步骤：

	powershell

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


## 创建 NamespaceManager 类

若要预配通知中心和其他服务总线实体，请从 SDK 创建 [NamespaceManager](http://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.namespacemanager.aspx) 类的实例。

你可以使用 Azure PowerShell 附带的 [Get-AzureSBAuthorizationRule] cmdlet 检索用于提供连接字符串的授权规则。 我们会将对 `NamespaceManager` 实例的引用还原到 `$NamespaceManager` 变量中。我们将使用 `$NamespaceManager` 来预配通知中心。

	powershell
	$sbr = Get-AzureSBAuthorizationRule -Namespace $Namespace
# 创建用于创建事件中心的 NamespaceManager 对象
	Write-Output "Creating a NamespaceManager object for the [$Namespace] namespace..."
	$NamespaceManager=[Microsoft.ServiceBus.NamespaceManager]::CreateFromConnectionString($sbr.ConnectionString);
	Write-Output "NamespaceManager object for the [$Namespace] namespace has been successfully created."

```


## 设置新通知中心 

<!--若要设置新的通知中心，请使用[服务总线的 .NET API]。本文只着重于通知中心。若要使用其他服务总线实体，请参阅[使用 PowerShell 管理服务总线](/documentation/articles/service-bus-powershell-how-to-provision)。-->

您将在脚本的这个部分设置四个本地变量。

1. `$Namespace`：将此变量设置为要创建通知中心的命名空间的名称。
2. `$Path`：将此路径设置为新通知中心的名称。例如“MyHub”。    
3. `$WnsPackageSid`：从 [Windows 开发人员中心](http://go.microsoft.com/fwlink/p/?linkid=266582&clcid=0x409)将此变量设置为 Windows 应用的包 SID。
4. `$WnsSecretkey`：从 [Windows 开发人员中心](http://go.microsoft.com/fwlink/p/?linkid=266582&clcid=0x409)将此变量设置为 Windows 应用的机密密钥。

这些变量可用于连接服务总线命名空间，以及创建配置为使用 Windows 应用 Windows 通知中心 (WNS) 凭据处理 WNS 通知的新通知中心。有关获取包 SID 和机密密钥的信息，请参阅[通知中心入门](/documentation/articles/notification-hubs-windows-store-dotnet-get-started)教程。

+ 脚本代码段使用 `NamespaceManager` 对象来检查 `$Path` 标识的通知中心是否存在。

+ 如果不存在，脚本将使用 WNS 凭据创建 `NotificationHubDescription`，并将其传递给 `NamespaceManager` 类 `CreateNotificationHub` 方法。

``` powershell

		$Namespace = "<Enter your namespace>
		$Path  = "<Enter a name for your notification hub>"
		$WnsPackageSid = "<your package sid>"
		$WnsSecretkey = "<enter your secret key>"

		$WnsCredential = New-Object -TypeName Microsoft.ServiceBus.Notifications.WnsCredential -ArgumentList $WnsPackageSid,$WnsSecretkey

# 查询命名空间
	$CurrentNamespace = Get-AzureSBNamespace -Name $Namespace

# 检查命名空间是否已存在  

	if ($CurrentNamespace)
	{
    Write-Output "The namespace [$Namespace] in the [$($CurrentNamespace.Region)] region was found."

    # Create the NamespaceManager object used to create a new notification hub
    $sbr = Get-AzureSBAuthorizationRule -Namespace $Namespace
    Write-Output "Creating a NamespaceManager object for the [$Namespace] namespace..."
    $NamespaceManager = [Microsoft.ServiceBus.NamespaceManager]::CreateFromConnectionString($sbr.ConnectionString);
    Write-Output "NamespaceManager object for the [$Namespace] namespace has been successfully created."

    # Check to see if the Notification Hub already exists
    if ($NamespaceManager.NotificationHubExists($Path))
    {
        Write-Output "The [$Path] notification hub already exists in the [$Namespace] namespace."  
    }
    else
    {
        Write-Output "Creating the [$Path] notification hub in the [$Namespace] namespace."
        $NHDescription = New-Object -TypeName Microsoft.ServiceBus.Notifications.NotificationHubDescription -ArgumentList $Path;
        $NHDescription.WnsCredential = $WnsCredential;
        $NamespaceManager.CreateNotificationHub($NHDescription);
        Write-Output "The [$Path] notification hub was created in the [$Namespace] namespace."
    }
	}
	else
	{
	Write-Host "The [$Namespace] namespace does not exist."
	}

```	




## 其他资源

- [使用 PowerShell 管理服务总线](/documentation/articles/service-bus-powershell-how-to-provision)
- [如何使用 PowerShell 脚本创建 服务总线队列、主题和订阅](http://blogs.msdn.com/b/paolos/archive/2014/12/02/how-to-create-a-service-bus-queues-topics-and-subscriptions-using-a-powershell-script.aspx)
- [如何使用 PowerShell 脚本创建服务总线命名空间和事件中心](http://blogs.msdn.com/b/paolos/archive/2014/12/01/how-to-create-a-service-bus-namespace-and-an-event-hub-using-a-powershell-script.aspx)

一些现成的脚本也可供下载：- [服务总线 PowerShell 脚本](https://code.msdn.microsoft.com/windowsazure/Service-Bus-PowerShell-a46b7059)
 

[购买选项]: http://www.windowsazure.cn/pricing/overview/
[成员优惠]: http://azure.microsoft.com/pricing/member-offers/
[免费试用]: /pricing/1rmb-trial/
[安装和配置 Azure PowerShell]: /documentation/articles/install-configure-powershell
[服务总线的 .NET API]: https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.aspx
[Get-AzureSBNamespace]: https://msdn.microsoft.com/zh-cn/library/azure/dn495122.aspx
[New-AzureSBNamespace]: https://msdn.microsoft.com/zh-cn/library/azure/dn495165.aspx
[Get-AzureSBAuthorizationRule]: https://msdn.microsoft.com/zh-cn/library/azure/dn495113.aspx
 

<!---HONumber=67-->