<properties
   pageTitle="来宾 OS 系列 1 停用通知 | Azure"
   description="提供有关 Azure 来宾 OS 系列 1 何时停用以及如何判断你是否受影响的信息"
   services="cloud-services"
   documentationCenter="na"
   authors="yuemlu"
   manager="timlt"
   editor=""/>

<tags
   ms.service="cloud-services"
   ms.date="04/19/2015"
   wacn.date="05/31/2016"/>



# 来宾 OS 系列 1 停用通知

我们已在 2013 年 6 月 1 日宣布停用 OS 系列 1。

**2014 年 9 月 2 日** - 基于 Windows Server 2008 操作系统的 Azure 来宾操作系统（来宾 OS）系列 1.x 正式停用。所有使用系列 1 部署新服务或升级现有服务的尝试均将失败，并显示错误消息“来宾 OS 系列 1 已停用“。

**2014 年 11 月 3 日** - 来宾 OS 系列 1 的延长支持结束，该系列完全停用。仍基于系列 1 的所有服务将受影响。我们随时可能会停止这些服务。除非你自己手动升级你的服务，否则无法保证你的服务将继续运行。

如果你还有其他疑问，请访问[云服务论坛](http://social.msdn.microsoft.com/Forums/home?forum=windowsazuredevelopment&filter=alltypes&sort=lastpostdesc)或[联系 Azure 支持人员](/support/contact)。




## 了解你是否受到影响

如果存在下列任一情况，则表示你的云服务已受到影响：

1. 你在云服务的 ServiceConfiguration.cscfg 文件中显式指定了值“osFamily = 1”。
2. 你未在云服务的 ServiceConfiguration.cscfg 文件中显式指定 osFamily 的值。当前，系统对此情况使用默认值“1”。
3. Azure 管理门户将你的来宾操作系统系列值列为“Windows Server 2008”。

若要了解你的哪个云服务在运行哪个 OS 系列，你可以在 Azure PowerShell 中运行以下脚本，但必须首先[设置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。有关该脚本的其他详细信息，请参阅 [Azure 来宾 OS 系列 1 生命周期终结：2014 年 6 月](http://blogs.msdn.com/b/ryberry/archive/2014/04/02/azure-guest-os-family-1-end-of-life-june-2014.aspx)。

	Powershell
	foreach($subscription in Get-AzureSubscription) {
	    Select-AzureSubscription -SubscriptionName $subscription.SubscriptionName 
	    
	    $deployments=get-azureService | get-azureDeployment -ErrorAction Ignore | where {$_.SdkVersion -NE ""} 
	
	    $deployments | ft @{Name="SubscriptionName";Expression={$subscription.SubscriptionName}}, ServiceName, SdkVersion, Slot, @{Name="osFamily";Expression={(select-xml -content $_.configuration -xpath "/ns:ServiceConfiguration/@osFamily" -namespace $namespace).node.value }}, osVersion, Status, URL
	}


如果脚本输出中的 osFamily 列为空或者包含“1”，则表示 OS 系列 1 的停用将影响到你的云服务。

## 如果你受到影响，请遵循以下建议

建议将你的云服务角色迁移到支持的来宾 OS 系列之一：

基于 Windows Server 2012 R2 的**来宾 OS 系列 4.x**（建议）

1. 确保你的应用程序使用了 SDK 2.1 或更高版本以及 .NET framework 4.0、4.5 或 4.5.1。
2. 在 ServiceConfiguration.cscfg 文件中将 osFamily 特性设置为“4”，然后重新部署云服务。


基于 Windows Server 2012 的**来宾 OS 系列 3.x**

1. 确保你的应用程序使用了 SDK 1.8 或更高版本以及 .NET framework 4.0 或 4.5。
2. 在 ServiceConfiguration.cscfg 文件中将 osFamily 特性设置为“3”，然后重新部署云服务。


基于 Windows Server 2008 R2 的**来宾 OS 系列 2.x**

1. 确保你的应用程序使用了 SDK 1.3 和更高版本以及 .NET framework 3.5 或 4.0。
2. 在 ServiceConfiguration.cscfg 文件中将 osFamily 特性设置为“2”，然后重新部署云服务。


## 对来宾 OS 系列 1 的延长支持已于 2014 年 11 月 3 日结束
不再支持来宾 OS 系列 1 上的云服务。请尽快迁出系列 1，以免服务中断。

## 后续步骤
查看最新的[来宾 OS 版本](/documentation/articles/cloud-services-guestos-update-matrix/)。
<!---HONumber=Mooncake_0523_2016-->
