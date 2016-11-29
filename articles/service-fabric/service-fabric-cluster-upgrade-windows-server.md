<properties
   pageTitle="在 Windows Server 上升级独立的 Service Fabric 群集 | Azure"
   description="升级运行独立 Service Fabric 群集的 Service Fabric 代码和/或配置，包括设置群集更新模式"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="10/10/2016"
   wacn.date="11/28/2016"
   ms.author="chackdan"/>  



# 在 Windows Server 上升级独立的 Service Fabric 群集

> [AZURE.SELECTOR]
- [Azure 群集](/documentation/articles/service-fabric-cluster-upgrade/)
- [独立群集](/documentation/articles/service-fabric-cluster-upgrade-windows-server/)

对于任何新式系统而言，为可升级性做好规划是实现产品长期成功的关键所在。Service Fabric 群集是你拥有的资源。本文介绍如何确保群集始终运行受支持的 Service Fabric 代码和配置版本。

## 控制群集中运行的结构版本

可以将群集设置为在 Microsoft 发布新版本时下载 Service Fabric 更新，或者选择要运行的受支持结构版本。

为此，可将“fabricClusterAutoupgradeEnabled”群集配置设置为 true 或 false。


>[AZURE.NOTE] 请确保群集始终运行受支持的 Service Fabric 版本。当我们宣布发行新版 Service Fabric 时，以前的版本标记为自发布日期开始算起的 60 天后结束支持。新版发布将在 [Service Fabric 团队博客](https://blogs.msdn.microsoft.com/azureservicefabric/)中通告。然后，便可以选择使用新版本。


仅当使用的是生产形式的节点配置（每个 Service Fabric 节点在独立的物理机或虚拟机上分配）时，才可以将群集升级到最新版本。如果使用的是开发群集（单个物理机或虚拟机上有多个 Service Fabric 节点），则必须先解除该群集，然后使用新版本重新创建该群集。


可以使用两种不同的工作流将群集升级到最新的或受支持的 Service Fabric 版本。一种工作流适用于已建立相应连接、可以自动下载最新版本的群集；第二种工作流适用于未建立相应连接、无法下载最新 Service Fabric 版本的群集。

### 升级已建立相应连接、可下载最新代码和配置的群集 

如果群集节点已与 [http://download.microsoft.com](http://download.microsoft.com) 建立 Internet 连接，可以使用以下步骤将群集升级到支持的版本

对于已连接到 [http://download.microsoft.com](http://download.microsoft.com) 的群集，我们会定期检查是否有新的 Service Fabric 版本可用。


推出新的 Service Fabric 版本时，相应的包将下载到群集本地，并为升级完成预配。此外，为了通知客户已推出新版本，系统会明确发出类似于下面的群集运行状况警告：

“对当前群集版本 [版本号] 的支持将在 [日期] 结束”。群集运行最新版本后，该警告将会消失。


#### 群集升级工作流。
 
看到群集运行状况警告后，需要执行以下操作：

1. 从对群集配置文件中列为节点的所有计算机拥有管理员访问权限的任何计算机连接到该群集。运行此脚本的计算机不必要是群集的一部分

	```powershell

	###### connect to the secure cluster using certs
	$ClusterName= "mysecurecluster.something.com:19000"
	$CertThumbprint= "70EF5E22ADB649799DA3C8B6A6BF7FG2D630F8F3" 
	Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
	    -X509Credential `
	    -ServerCertThumbprint $CertThumbprint  `
	    -FindType FindByThumbprint `
	    -FindValue $CertThumbprint `
	    -StoreLocation CurrentUser `
	    -StoreName My
	```

2. 获取可升级到的 Service Fabric 版本列表

	```powershell

	###### Get the list of available service fabric versions 
	Get-ServiceFabricRegisteredClusterCodeVersion
	```

	应会看到类似于下面的输出：

	![获取结构版本][getfabversions]  


3. 开始使用 [Start-ServiceFabricClusterUpgrade PowerShell 命令](https://msdn.microsoft.com/zh-cn/library/mt125872.aspx)，将群集升级到可用的版本之一

	```Powershell

	Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion <codeversion#> -Monitored -FailureAction Rollback

	###### Here is a filled out example

	Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion 5.3.301.9590 -Monitored -FailureAction Rollback
	
	```
可以在 Service Fabric Explorer 中或者通过运行以下 PowerShell 命令来监视升级进度

	```powershell

	Get-ServiceFabricClusterUpgrade
	```

	如果不符合现行的群集运行状况策略，则回滚升级。此时，可以指定自定义运行状况策略。有关 Start-ServiceFabricClusterUpgrade 命令的详细信息，请参阅[此文档](https://msdn.microsoft.com/zh-cn/library/mt125872.aspx)。

解决导致回滚的问题后，需要遵循前面相同的步骤再次启动升级。


### 升级<U>未建立相应连接</u>、无法下载最新代码和配置的群集

如果群集节点**未**与 [http://download.microsoft.com](http://download.microsoft.com) 建立 Internet 连接，可以使用以下步骤将群集升级到支持的版本


>[AZURE.NOTE]如果运行的群集未连接到 Internet，必须关注 Service Fabric 团队博客获取有关新版的通知。在此情况下，系统**不会**发出任何群集运行状况警告。

1. 请修改群集配置，将以下属性设置为 false。

        "fabricClusterAutoupgradeEnabled": false,

然后开始升级配置。有关用法详细信息，请参阅 [Start-ServiceFabricClusterUpgrade PS 命令](https://msdn.microsoft.com/zh-cn/library/mt125872.aspx)。群集清单版本是 clusterConfig.JSON 中指定的版本。请务必在开始升级配置之前更新该版本。

```powershell

	Start-ServiceFabricClusterUpgrade [-Config] [-ClusterConfigVersion] -FailureAction Rollback -Monitored 

```

#### 群集升级工作流。
 


1. 通过 [Create service fabric cluster for windows server](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)（创建适用于 Windows Server 的 Service Fabric 群集）下载最新的包版本


1. 从对群集配置文件中列为节点的所有计算机拥有管理员访问权限的任何计算机连接到该群集。运行此脚本的计算机不必要是群集的一部分

	```powershell

	###### connect to the cluster
	$ClusterName= "mysecurecluster.something.com:19000"
	$CertThumbprint= "70EF5E22ADB649799DA3C8B6A6BF7FG2D630F8F3" 
	Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
	    -X509Credential `
	    -ServerCertThumbprint $CertThumbprint  `
	    -FindType FindByThumbprint `
	    -FindValue $CertThumbprint `
	    -StoreLocation CurrentUser `
	    -StoreName My
	```

2. 将下载的包复制到群集映像存储中。

	```powershell

	###### Get the list of available service fabric versions 
	Copy-ServiceFabricClusterPackage -Code -CodePackagePath <name of the .cab file including the path to it> -ImageStoreConnectionString "fabric:ImageStore"

	###### Here is a filled out example
	Copy-ServiceFabricClusterPackage -Code -CodePackagePath .\MicrosoftAzureServiceFabric.5.3.301.9590.cab -ImageStoreConnectionString "fabric:ImageStore"


	```

2. 注册复制的包

	```powershell

	###### Get the list of available service fabric versions 
	Register-ServiceFabricClusterPackage -Code -CodePackagePath <name of the .cab file> 

	###### Here is a filled out example
	Register-ServiceFabricClusterPackage -Code -CodePackagePath MicrosoftAzureServiceFabric.5.3.301.9590.cab

	 ```


3. 开始将群集升级到可用的版本之一。

	```Powershell

	Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion <codeversion#> -Monitored -FailureAction Rollback

	###### Here is a filled out example
	Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion 5.3.301.9590 -Monitored -FailureAction Rollback
	
	```
可以在 Service Fabric Explorer 中或者通过运行以下 PowerShell 命令来监视升级进度

	```powershell

	Get-ServiceFabricClusterUpgrade
	```

	如果不符合现行的群集运行状况策略，则回滚升级。此时，可以指定自定义运行状况策略。有关 start-serviceFabricClusterUpgrade 命令的详细信息，请参阅[此文档](https://msdn.microsoft.com/zh-cn/library/mt125872.aspx)。

解决导致回滚的问题后，需要遵循前面相同的步骤再次启动升级。



## 后续步骤
- 了解如何自定义某些 [Service Fabric 群集结构设置](/documentation/articles/service-fabric-cluster-fabric-settings/)
- 了解如何[扩展和缩减群集](/documentation/articles/service-fabric-cluster-scale-up-down/)
- 了解[应用程序升级](/documentation/articles/service-fabric-application-upgrade/)

<!--Image references-->

[getfabversions]: ./media/service-fabric-cluster-upgrade-windows-server/getfabversions.PNG

<!---HONumber=Mooncake_1121_2016-->