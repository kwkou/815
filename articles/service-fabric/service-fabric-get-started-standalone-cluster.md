<properties
    pageTitle="设置独立的 Azure Service Fabric 群集 | Azure"
    description="创建一个独立的开发群集，在同一计算机上运行三个节点。 完成此设置后，即可开始创建多机群集。"
    services="service-fabric"
    documentationcenter=".net"
    author="rwike77"
    manager="timlt"
    editor="" />
<tags
    ms.assetid=""
    ms.service="service-fabric"
    ms.devlang="dotNet"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="04/11/2017"
    wacn.date="05/15/2017"
    ms.author="ryanwi"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="a3176fd762dc96b5c43c0f80474995ffd279f4f1"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="create-your-first-service-fabric-standalone-cluster"></a>创建第一个独立的 Service Fabric 群集
可以在任何运行 Windows Server 2012 R2 或 Windows Server 2016（不管是在本地还是在云中）的虚拟机或计算机上创建独立的 Service Fabric 群集。 本快速入门介绍如何在数分钟内创建独立的开发群集。  完成后，你将有一个三节点群集运行在单台可以向其部署应用的计算机上。

## <a name="before-you-begin"></a>开始之前
Service Fabric 提供了一个安装程序包，用于创建独立的 Service Fabric 群集。  [下载安装程序包](http://go.microsoft.com/fwlink/?LinkId=730690)。  将安装程序包解压缩到计算机或虚拟机（需在其中设置开发群集）的文件夹中。  安装包的内容详见[此处](/documentation/articles/service-fabric-cluster-standalone-package-contents/)。

部署和配置群集的群集管理员必须拥有计算机的管理员权限。 不能在域控制器上安装 Service Fabric。

## <a name="validate-the-environment"></a>验证环境
独立包中的 *TestConfiguration.ps1* 脚本可以用作最佳做法分析器，验证群集能否在给定环境中部署。 [部署准备](/documentation/articles/service-fabric-cluster-standalone-deployment-preparation/)列出了先决条件和环境要求。 请运行以下脚本，验证你能否创建开发群集：

    .\TestConfiguration.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json

## <a name="create-the-cluster"></a>创建群集
可以通过安装包安装多个示例性的群集配置文件。 *ClusterConfig.Unsecure.DevCluster.json* 是最简单的群集配置：在单台计算机上运行一个不受保护的三节点群集。  其他配置文件描述的是受 X.509 证书或 Windows 安全措施保护的单机或多机群集。  就本教程来说，你无需修改任何默认的配置设置，但仍请查看配置文件，熟悉一下这些设置。  **nodes** 节描述了群集中的三个节点：名称、IP 地址、[节点类型、容错域和升级域](/documentation/articles/service-fabric-cluster-manifest/#nodes-on-the-cluster)。  **properties** 节定义群集的[安全性、可靠级别、诊断集合和节点类型](/documentation/articles/service-fabric-cluster-manifest/#cluster-properties)。

此群集是不安全的。  任何人都可以进行匿名连接并执行管理操作，因此生产型群集应始终使用 X.509 证书或 Windows 安全措施来确保安全性。  安全性只能在群集创建时配置，不能在创建群集以后启用安全性。  请阅读[保护群集](/documentation/articles/service-fabric-cluster-security/)，详细了解 Service Fabric 群集安全性。  

若要创建三节点开发群集，请通过管理员 PowerShell 会话运行 *CreateServiceFabricCluster.ps1* 脚本：

    .\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json -AcceptEULA

在创建群集时自动下载和安装 Service Fabric 运行时包。

## <a name="connect-to-the-cluster"></a>连接至群集
你的三节点开发群集现在正运行。 ServiceFabric PowerShell 模块与运行时一起安装。  可以通过 Service Fabric 运行时来验证：群集是从同一计算机运行的，还是从远程计算机运行的。  [Connect-ServiceFabricCluster](https://docs.microsoft.com/zh-cn/powershell/module/ServiceFabric/Connect-ServiceFabricCluster) cmdlet 可建立到群集的连接。   

    Connect-ServiceFabricCluster -ConnectionEndpoint localhost:19000

有关如何连接到群集的其他示例，请参阅[连接到安全群集](/documentation/articles/service-fabric-connect-to-secure-cluster/)。 连接到群集以后，请使用 [Get-ServiceFabricNode](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricnode) cmdlet 显示群集中节点的列表，以及每个节点的状态信息。 每个节点的 **HealthState** 应该为“正常”。

    PS C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer> Get-ServiceFabricNode |Format-Table

    NodeDeactivationInfo NodeName IpAddressOrFQDN NodeType  CodeVersion ConfigVersion NodeStatus NodeUpTime NodeDownTime HealthState
    -------------------- -------- --------------- --------  ----------- ------------- ---------- ---------- ------------ -----------
                         vm2      localhost       NodeType2 5.5.216.0   0                     Up 03:00:07   00:00:00              Ok
                         vm1      localhost       NodeType1 5.5.216.0   0                     Up 03:00:02   00:00:00              Ok
                         vm0      localhost       NodeType0 5.5.216.0   0                     Up 03:00:01   00:00:00              Ok

## <a name="visualize-the-cluster-using-service-fabric-explorer"></a>使用 Service Fabric Explorer 可视化群集
[Service Fabric Explorer](/documentation/articles/service-fabric-visualizing-your-cluster/) 是一项很好的工具，适用于可视化群集和管理应用程序。  Service Fabric Explorer 是一项在群集中运行的服务，使用浏览器导航到 [http://localhost:19080/Explorer](http://localhost:19080/Explorer) 即可访问该服务。 

群集仪表板提供了群集的概览，包括应用程序和节点运行状况的摘要。 节点视图显示群集的物理布局。 对于给定的节点，你可以检查已在该节点上部署代码的应用程序。

![Service Fabric Explorer][service-fabric-explorer]

## <a name="remove-the-cluster"></a>删除群集
若要删除群集，请运行包文件夹中的 *RemoveServiceFabricCluster.ps1* Powershell 脚本，并传入 JSON 配置文件的路径。 你可以选择指定删除的日志的位置。

    # Removes Service Fabric cluster nodes from each computer in the configuration file.
    .\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json -Force

若要从计算机中删除 Service Fabric 运行时，请运行包文件夹中的下述 PowerShell 脚本。

    # Removes Service Fabric from the current computer.
    .\CleanFabric.ps1

## <a name="next-steps"></a>后续步骤
设置单独的开发群集以后，可尝试阅读以下文件：
* [设置独立的多机群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)并启用安全性。
* [使用 PowerShell 部署应用](/documentation/articles/service-fabric-deploy-remove-applications/)

[service-fabric-explorer]: ./media/service-fabric-get-started-standalone-cluster/sfx.png