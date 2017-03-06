<properties
    pageTitle="适用于 Windows Server 的 Azure Service Fabric 独立包 | Azure"
    description="适用于 Windows Server 的 Azure Service Fabric 独立包的说明和内容。"
    services="service-fabric"
    documentationcenter=".net"
    author="maburlik"
    manager="timlt"
    editor="" />
<tags
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="2/15/2017"
    wacn.date="03/03/2017"
    ms.author="chackdan;maburlik" />  


# 打包适用于 Windows Server 的 Service Fabric 独立包的内容
在[已下载](http://go.microsoft.com/fwlink/?LinkId=730690)的 Service Fabric 独立包中，找到以下文件：

| **文件名** | **简短说明** |
| --- | --- |
| CreateServiceFabricCluster.ps1 |一个 PowerShell 脚本，它可以使用 ClusterConfig.json 中的设置创建群集。 |
| RemoveServiceFabricCluster.ps1 |一个 PowerShell 脚本，它可以使用 ClusterConfig.json 中的设置删除群集。 |
| AddNode.ps1 |一个 PowerShell 脚本，用于将节点添加到当前计算机上的现有部署群集。 |
| RemoveNode.ps1 |一个 PowerShell 脚本，用于将节点从当前计算机上的现有部署群集中删除。 |
| CleanFabric.ps1 |一个 PowerShell 脚本，用于从当前计算机中清除独立的 Service Fabric 安装。以前的 MSI 安装应使用其自身关联的卸载程序来删除。 |
| TestConfiguration.ps1 |一个 PowerShell 脚本，用于分析 Cluster.json 中指定的基础结构。 |
| DownloadServiceFabricRuntimePackage.ps1 |一个 PowerShell 脚本，在部署计算机未连接到 Internet 的方案中，用于在带外下载最新的运行时包。 |
| DeploymentComponentsAutoextractor.exe |自解压存档，包含独立包脚本使用的部署组件。 |
| EULA\_ENU.txt |有关使用 Microsoft Azure Service Fabric Windows Server 独立包的许可条款。可以立即[下载一份 EULA](http://go.microsoft.com/fwlink/?LinkID=733084)。 |
| Readme.txt |发行说明和基本安装说明的链接。本文档包含此文件中的一部分说明。 |
| ThirdPartyNotice.rtf |包中第三方软件的声明。 |

**模板**
| **文件名** | **简短说明** |
| --- | --- |
| ClusterConfig.Unsecure.DevCluster.json |一个群集配置示例文件，包含非安全型三节点式单计算机（或虚拟机）开发群集的设置，包括群集中每个节点的信息。 |
| ClusterConfig.Unsecure.MultiMachine.json |群集配置示例文件包含的设置适用于非安全多计算机（或虚拟机）群集，这些设置包括群集中每个计算机的信息。 |
| ClusterConfig.Windows.DevCluster.json |一个群集配置示例文件，包含安全三节点式单计算机（或虚拟机）开发群集的所有设置，包括群集中每个节点的信息。该群集使用 [Windows 标识](https://msdn.microsoft.com/zh-cn/library/ff649396.aspx)进行保护。 |
| ClusterConfig.Windows.MultiMachine.json |一个群集配置示例文件，包含使用 Windows 安全性的安全多计算机（或虚拟机）群集的所有设置，包括安全群集中每个计算机的信息。该群集使用 [Windows 标识](https://msdn.microsoft.com/zh-cn/library/ff649396.aspx)进行保护。 |
| ClusterConfig.x509.DevCluster.json |一个群集配置示例文件，包含安全三节点式单计算机（或虚拟机）开发群集的所有设置，包括群集中每个节点的信息。此群集使用 x509 证书进行保护。 |
| ClusterConfig.x509.MultiMachine.json |一个群集配置示例文件，包含安全多计算机（或虚拟机）群集的所有设置，包括安全群集中每个节点的信息。此群集使用 x509 证书进行保护。 |
| ClusterConfig.gMSA.Windows.MultiMachine.json |一个群集配置示例文件，包含安全多计算机（或虚拟机）群集的所有设置，包括安全群集中每个节点的信息。使用 [组托管服务帐户] 保护群集 (https://technet.microsoft.com/zh-cn/library/jj128431(v=ws.11).aspx)。 |

# 群集配置示例
最新版本的群集配置模板可以在以下 Github 页上找到：[独立群集配置示例](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)。

## 相关内容
* [创建独立 Azure Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)
* [Service Fabric 群集安全方案](/documentation/articles/service-fabric-windows-cluster-windows-security/)

<!---HONumber=Mooncake_0227_2017-->