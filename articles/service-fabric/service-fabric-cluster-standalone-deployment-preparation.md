<properties
    pageTitle="Azure Service Fabric 独立群集部署准备 | Azure"
    description="在部署专用于处理生产工作负荷的群集之前要考虑的与准备环境和创建群集配置相关的文档。"
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
    ms.date="1/17/2016"
    wacn.date="03/03/2017"
    ms.author="dkshir;chackdan;maburlik" />  


<a id="preparemachines"></a>

## 规划和准备 Service Fabric 独立群集部署
在创建群集之前，请执行以下步骤。

### 步骤 1：规划群集基础结构
您需要在所拥有的计算机上创建 Service Fabric 群集，以便确定群集需应对的故障类型。例如，是否需要为这些计算机单独提供电源线或 Internet 连接？ 此外，还应考虑这些计算机的物理安全性。计算机位于何处？哪些人需要访问计算机？ 做出这些决定后，以逻辑方式将计算机映射到多个容错域（请参阅步骤 4）。与测试群集相比，生产群集的基础结构规划更加复杂。

### 步骤 2：准备符合先决条件的计算机
要添加到群集的每台计算机的先决条件：

* 建议至少提供 16 GB RAM。
* 建议至少提供 40 GB 可用磁盘空间。
* 建议使用 4 核或以上的 CPU。
* 与所有计算机的安全网络建立连接。
* Windows Server 2012 R2 或 Windows Server 2016。
* [.NET Framework 4.5.1 或更高版本](https://www.microsoft.com/download/details.aspx?id=40773)的完整安装版。
* [Windows PowerShell 3.0](https://msdn.microsoft.com/powershell/scripting/setup/installing-windows-powershell)。
* 应在所有计算机上运行 [RemoteRegistry 服务](https://technet.microsoft.com/zh-cn/library/cc754820)。

部署和配置群集的群集管理员必须拥有每台计算机的[管理员权限](https://social.technet.microsoft.com/wiki/contents/articles/13436.windows-server-2012-how-to-add-an-account-to-a-local-administrator-group.aspx)。无法在域控制器上安装 Service Fabric。

### 步骤 3：确定初始群集大小
独立 Service Fabric 群集中的每个节点都应部署 Service Fabric 运行时，并且都是该群集的成员。在典型的生产部署中，每个 OS 实例（物理或虚拟）都有一个节点。群集大小取决于业务要求。但是，一个群集必须至少包含三个节点（计算机或虚拟机）。出于开发目的，可以在一台指定的计算机上创建多个节点。在生产环境中，对于每个物理机或虚拟机，Service Fabric 只支持一个节点。

### 步骤 4：确定容错域和升级域的数目
*容错域* (FD) 是物理故障单元，与数据中心的物理基础结构直接相关。容错域由共享单一故障点的硬件组件（计算机、交换机、网络等）组成。尽管容错域与机架之间没有 1:1 映射，但是大致上，可将每个机架视为一个容错域。在考虑群集中的节点时，强烈建议至少在三个容错域之间分布节点。

在 ClusterConfig.json 中指定 FD 时，可以选择每个 FD 的名称。Service Fabric 支持分层的 FD，因此，你可以在 FD 中反映基础结构拓扑。例如，以下 FD 是有效的：

* "faultDomain": "fd:/Room1/Rack1/Machine1"
* "faultDomain": "fd:/FD1"
* "faultDomain": "fd:/Room1/Rack1/PDU1/M1"

*升级域* (UD) 是节点的逻辑单元。在 Service Fabric 协调式升级（应用程序升级或群集升级）期间，将关闭 UD 中的所有节点以执行升级，其他 UD 中的节点仍可用来为请求提供服务。对计算机进行的固件升级将不遵循 UD，因此每次必须在一台计算机上执行此操作。

领会这些概念的最简单方法是将 FD 视为非计划内故障的单元，将 UD 视为计划维护的单元。

在 ClusterConfig.json 中指定 UD 时，可以选择每个 UD 的名称。例如，以下名称是有效的：

* "upgradeDomain": "UD0"
* "upgradeDomain": "UD1A"
* "upgradeDomain": "DomainRed"
* "upgradeDomain": "Blue"

有关升级域和容错域的更多详细信息，请参阅[描述 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)一文。

### 步骤 5：下载适用于 Windows Server 的 Service Fabric 独立包
[下载链接 - Service Fabric 独立包 - Windows Server](http://go.microsoft.com/fwlink/?LinkId=730690)，将包解压缩到群集外的一台部署计算机中或解压缩到群集内的其中一台计算机中。

### 步骤 6：修改群集配置
若要创建独立群集，必须创建独立群集配置 ClusterConfig.json 文件，其中描述群集的规范。可以基于在以下链接中找到的模板创建配置文件。<br>[独立群集配置](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)

有关此文件中各个节的详细信息，请参阅 [Windows 独立群集的配置设置](/documentation/articles/service-fabric-cluster-manifest/)。

从已下载的包中打开某个 ClusterConfig.json 文件，然后修改以下设置：
| **配置设置** | **说明** |
| --- | --- |
| **NodeTypes** |<p>节点类型可让你将群集节点划分到不同的组中。一个群集必须至少有一个节点类型。组中的所有节点具有以下共同特征：</p><p>**名称** - 节点类型名称。</p><p>**终结点端口** - 即与此节点类型关联的各种命名终结点（端口）。可以使用任何端口号，只要它们不会与此清单中的其他部分发生冲突，并且未被计算机/VM 上运行的其他应用程序使用。</p><p>**放置属性** - 描述此节点类型的属性，可用作系统服务或服务的放置约束。这些属性是用户定义的键/值对，可为指定节点提供额外的元数据。节点属性的示例包括节点是否有硬盘或图形卡、其硬盘的轴数、核心和其他物理属性。</p><p>**容量** - 节点容量，定义特定节点提供的特定资源的名称和数量。例如，节点可以定义名为“MemoryInMb”的指标容量，而且默认有 2048 MB 的可用内存。这些容量可在运行时使用，以确保将需要特定数量资源的服务放置在具有所需的可用资源量的节点上。</p><p>**IsPrimary** - 如果已定义多个 NodeType，请确保只将一个 NodeType（在其中运行系统服务）设置为主 NodeType（使用值 *true* 进行设置）。应将所有其他节点类型设置为 *false* 值 </p>|
| **Nodes** |<p>这些是群集内的每个节点的详细信息（节点类型、节点名称、IP 地址、节点的容错域和升级域）。要在其上创建群集的计算机必须与其 IP 地址一起列在此处。</p><p>如果对所有节点使用相同的 IP 地址，则会创建一个可用于测试的单机群集。不要将单机群集用于部署生产工作负荷。 </p>|

群集配置的所有设置已配置到环境后，可以针对群集环境测试群集配置（步骤 7）。

<a id="environmentsetup"></a>

### 步骤 7.环境设置

群集管理员配置 Service Fabric 独立群集时，需要使用以下标准设置环境：<br>
1. 创建群集的用户应对群集配置文件中作为节点列出的所有计算机具有管理员级别的安全特权。
2. 从中创建群集的计算机以及每个群集节点计算机必须：
* 已卸载 Service Fabric SDK
* 已卸载 Service Fabric 运行时
* 已启用 Windows 防火墙服务 (mpssvc)
* 已启用删除注册表服务 (remoteregistry)
* 已启用文件共享 (SMB)
* 已基于群集配置端口打开了必要的端口
* 已为 Windows SMB 和远程注册表服务打开了必要的端口：135、137、138、139 和 445
* 已将网络彼此互连
3. 群集节点计算机不应为域控制器。
4. 如果要部署的群集是安全群集，请验证是否已具备必需的安全先决条件，以及是否已根据配置进行正确配置。
5. 如果群集计算机无法访问 Internet，请在群集配置中设置以下项：
* 禁用遥测：在“属性”下设置*“enableTelemetry”: false*
* 禁用自动 Fabric 版本下载和当前群集版本即将终止支持的通知：在“属性”下设置*“fabricClusterAutoupgradeEnabled”: true*
6. 设置适当的 Service Fabric 防病毒排除项：

| **防病毒排除目录** |
| --- |
| Program Files\\Microsoft Service Fabric |
| FabricDataRoot（从群集配置中） |
| FabricLogRoot（从群集配置中） |

| **防病毒排除进程** |
| --- |
| Fabric.exe |
| FabricHost.exe |
| FabricInstallerService.exe |
| FabricSetup.exe |
| FabricDeployer.exe |
| ImageBuilder.exe |
| FabricGateway.exe |
| FabricDCA.exe |
| FabricFAS.exe |
| FabricUOS.exe |
| FabricRM.exe |
| FileStoreService.exe |

### 步骤 8。使用 TestConfiguration 脚本验证环境
可以在独立包中找到 TestConfiguration.ps1 脚本。它作为最佳做法分析器，可验证上述某些条件，并应该用作健全性检查来验证是否可以在给定环境上部署群集。如果有任何失败，请参阅[环境设置](/documentation/articles/service-fabric-cluster-standalone-deployment-preparation/)下的列表进行故障排除。

可以在对群集配置文件中列为节点的所有计算机具有管理员访问权限的任何计算机上运行此脚本。运行此脚本的计算机不必要是群集的一部分。


	PS C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer> .\TestConfiguration.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json
	Trace folder already exists. Traces will be written to existing trace folder: C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer\DeploymentTraces
	Running Best Practices Analyzer...
	Best Practices Analyzer completed successfully.


	LocalAdminPrivilege        : True
	IsJsonValid                : True
	IsCabValid                 : True
	RequiredPortsOpen          : True
	RemoteRegistryAvailable    : True
	FirewallAvailable          : True
	RpcCheckPassed             : True
	NoConflictingInstallations : True
	FabricInstallable          : True
	Passed                     : True


当前此配置测试模块不验证安全配置，因此这必须独立完成。



## 后续步骤
* [创建在 Windows Server 上运行的独立群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)

<!---HONumber=Mooncake_0227_2017-->