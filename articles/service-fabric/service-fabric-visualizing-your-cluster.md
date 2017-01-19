<properties
    pageTitle="使用 Service Fabric Explorer 可视化群集 | Azure"
    description="Service Fabric Explorer 是一个用于检验和管理 Azure Service Fabric 群集中云应用程序和节点的基于 Web 的工具。"
    services="service-fabric"
    documentationcenter=".net"
    author="seanmck"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="c875b993-b4eb-494b-94b5-e02f5eddbd6a"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="12/13/2016"
    wacn.date="01/17/2017"
    ms.author="seanmck" />  


# 使用 Service Fabric Explorer 可视化群集
Service Fabric Explorer 是一个用于检验和管理 Azure Service Fabric 群集中应用程序和节点的基于 Web 的工具。Service Fabric Explorer 直接托管在群集内，因此，无论群集在何处运行，它都始终可供使用。

## 连接到 Service Fabric Explorer

如果你已根据说明[准备开发环境](/documentation/articles/service-fabric-get-started/)，则可以通过导航到 http://localhost:19080/Explorer 启动本地群集上的 Service Fabric Explorer。

>[AZURE.NOTE] 如果你结合使用 Internet Explorer 与 Service Fabric Explorer 管理远程群集，需要配置一些 Internet Explorer 设置。转到“工具”>“兼容性视图设置”，然后取消选中“在兼容性视图中显示 Intranet 站点”，以确保正确加载所有信息。

## 了解 Service Fabric Explorer 布局
可以使用左侧的树导航 Service Fabric Explorer。在树根中，群集仪表板提供了群集的概述，包括应用程序和节点运行状况的摘要。

![Service Fabric Explorer 群集仪表板][sfx-cluster-dashboard]

### 查看群集的布局
Service Fabric 群集中的节点横跨容错域和升级域的二维网格放置。这种放置可确保应用程序在发生硬件故障及应用程序升级时仍然可用。你可以使用群集图查看当前群集的布局方式。

![Service Fabric Explorer 群集图][sfx-cluster-map]

### 查看应用程序和服务
群集包含两个子树：一个用于应用程序，另一个用于节点。

你可以使用应用程序视图导航 Service Fabric 的逻辑层次结构：应用程序、服务、分区和副本。

在以下示例中，应用程序 **MyApp** 由两个服务 **MyStatefulService** 与 **WebService** 组成。由于 **MyStatefulService** 是有状态的，因此它包含一个分区，其中有一个主要副本和两个次要副本。相反，WebSvcService 是无状态的，只包含单个实例。

![Service Fabric Explorer 应用程序视图][sfx-application-tree]

在树的每个级别，主窗格显示有关项目的信息。例如，你可以看到特定服务的运行状况状态和版本。

![Service Fabric Explorer 基本信息窗格][sfx-service-essentials]

### 查看群集的节点

节点视图显示群集的物理布局。对于给定的节点，你可以检查已在该节点上部署代码的应用程序。更具体地说，你可以看到当前在那里运行的副本。

## 操作

Service Fabric Explorer 提供用于对群集中的节点、应用程序和服务快速调用操作的方式。

例如，若要删除某个应用程序实例，只需从左侧树中选择该应用程序，然后依次选择“操作”>“删除应用程序”。

![在 Service Fabric Explorer 中删除应用程序][sfx-delete-application]

>[AZURE.TIP] 可以通过单击每个元素旁边的省略号执行相同的操作。

下表列出了可对每个实体执行的操作：

| **实体** | **操作** | **说明** |
| --- | --- | --- |
| 应用程序类型 |取消预配类型 |从群集的映像存储中删除应用程序包。需要先删除该类型的所有应用程序。 |
| 应用程序 |删除应用程序 |删除应用程序，包括其所有的服务以及状态（如果有）。 |
| 服务 |删除服务 |删除服务及其状态（如果有）。 |
| 节点 |激活 |激活节点。 |
| 停用（暂停） |在其当前的状态中暂停节点。服务继续运行，但 Service Fabric 并不主动在节点上移入或移出任何项目，除非需要避免服务中断或数据不一致的问题。此操作通常用于在特定节点上启用调试服务，以确保它们不在检查期间移动。 | |
| 停用（重新启动） |从节点中安全删除所有内存中服务，并关闭永久性服务。通常在需要重新启动主机进程或计算机时使用。 | |
| 停用（删除数据） |在生成足够的备用副本之后，安全关闭节点上运行的所有服务。通常在永久性淘汰某个节点（或至少其存储）时使用。 | |
| 删除节点状态 |从群集中删除节点副本的信息。通常在发生故障的节点肯定无法恢复时使用。 | |

由于许多操作都具有破坏性，因此在完成该操作之前，系统可能会请求你确认意图。

>[AZURE.TIP] 可以通过 Service Fabric Explorer 执行的每个操作也可以通过 PowerShell 或 REST API 执行，以实现自动化。

你还可以使用 Service Fabric Explorer 为指定应用程序类型和版本创建新的应用程序实例。在树视图中选择应用程序类型，然后在右侧窗格中单击所需版本旁边的“创建应用实例”链接。

![在 Service Fabric Explorer 中创建应用程序实例][sfx-create-app-instance]

>[AZURE.NOTE] 当前无法对通过 Service Fabric Explorer 创建的应用程序实例进行参数化。它们是使用默认参数值创建的。

## 连接到远程 Service Fabric 群集
由于 Service Fabric Explorer 是基于 Web 的工具并且在群集内部运行，因此你只要知道群集的终结点且有足够的访问权限，就可以从任何浏览器访问它。

### 发现远程群集的 Service Fabric Explorer 终结点
若要连接到给定群集的 Service Fabric Explorer，只需将浏览器指向：

http://&lt;your-cluster-endpoint&gt;:19080/Explorer  


对于 Azure 群集，Azure 门户的群集基本信息窗格中也提供了完整 URL。

### 连接到安全群集
可以使用证书或 Azure Active Directory (AAD) 控制客户端对 Service Fabric 群集的访问。

如果尝试在安全群集上连接到 Service Fabric Explorer，则需提供客户端证书或使用 AAD 登录，具体取决于群集的配置。

## 后续步骤
- [可测试性概述](/documentation/articles/service-fabric-testability-overview/)。
- [在 Visual Studio 中管理 Service Fabric 应用程序](/documentation/articles/service-fabric-manage-application-in-visual-studio/)。
- [使用 PowerShell 部署 Service Fabric 应用程序](/documentation/articles/service-fabric-deploy-remove-applications/)

<!--Image references-->
[sfx-cluster-dashboard]: ./media/service-fabric-visualizing-your-cluster/SfxClusterDashboard.png
[sfx-cluster-map]: ./media/service-fabric-visualizing-your-cluster/SfxClusterMap.png
[sfx-application-tree]: ./media/service-fabric-visualizing-your-cluster/SfxApplicationTree.png
[sfx-service-essentials]: ./media/service-fabric-visualizing-your-cluster/SfxServiceEssentials.png
[sfx-delete-application]: ./media/service-fabric-visualizing-your-cluster/SfxDeleteApplication.png
[sfx-create-app-instance]: ./media/service-fabric-visualizing-your-cluster/SfxCreateAppInstance.png

<!---HONumber=Mooncake_Quality_Review_0117_2017-->