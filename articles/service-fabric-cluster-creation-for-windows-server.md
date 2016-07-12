<properties
   pageTitle="创建本地或任意云 Azure Service Fabric 群集 | Azure"
   description="了解如何在运行 Windows Server 的任何本地或任意云中计算机上创建 Azure Service Fabric 群集（物理或虚拟）。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/14/2016"
   wacn.date="07/04/2016"/>


# 在本地或云中创建 Azure Service Fabric 群集

Azure Service Fabric 允许在运行 Windows Server 的任何虚拟机或计算机上创建 Service Fabric 群集。这意味着你可以在具有一组相互连接的 Windows Server 计算机的任何环境（无论是本地环境还是任何云提供商所提供的）中部署和运行 Service Fabric 应用程序。Service Fabric 提供了一个安装程序包以供你创建此类 Service Fabric 群集。

本文将引导你使用 Service Fabric的独立包在本地完成创建群集的步骤，不过，你也可以针对其他任何环境（例如其他云）轻松地调整。

>[AZURE.NOTE] 此独立产品目前以预览版提供。如果你想要立即下载一份 EULA，请[单击此处](http://go.microsoft.com/fwlink/?LinkID=733084)。

<a id="downloadpackage"></a>
## 下载 Service Fabric 独立包


[下载适用于 Windows Server 2012 R2 的 Service Fabric 独立包](http://go.microsoft.com/fwlink/?LinkId=730690)，其名为 *Microsoft.Azure.ServiceFabric.WindowsServer.&lt;版本&gt;.zip*。

在下载包中，你将看到以下文件：

|**文件名**|**简短说明**|
|-----------------------|--------------------------|
|MicrosoftAzureServiceFabric.cab|包含要部署到群集中每台计算机的二进制文件的 CAB 文件。|
|ClusterConfig.JSON|包含群集的所有设置（包括群集所属每台计算机的相关信息）的群集配置文件。|
|EULA.txt|有关使用 Azure Service Fabric Service Fabric 独立包的许可条款。如果你想要立即下载一份 EULA，请[单击此处](http://go.microsoft.com/fwlink/?LinkID=733084)。|
|Readme.txt|发行说明和基本安装说明的链接。这是你可以在此页上看到的一部分说明。|
|CreateServiceFabricCluster.ps1|用于通过 ClusterConfig.JSON 文件中的设置创建群集的 PowerShell 脚本。|
|RemoveServiceFabricCluster.ps1|用于通过 ClusterConfig.JSON 中的设置删除群集的 PowerShell 脚本。|

## 规划和准备群集部署
在创建群集之前，需要执行以下步骤。

### 步骤 1：规划群集基础结构
你将在你拥有的计算机上创建 Service Fabric 群集，因此必须规划好群集可以幸免于哪种故障。例如，是否需要为这些计算机单独提供电源线或 Internet 连接？ 此外，还应该考虑这些计算机的物理安全性。物理位置在哪里？谁需要访问这些计算机？ 一旦做出这些决定，就要以逻辑方式将计算机映射到多个容错域（有关详细信息，请参阅下文）。相比于测试群集，生产群集的基础结构规划要复杂得多。

### 步骤 2：准备符合先决条件的计算机
要添加到群集的每台计算机的先决条件：

- 建议至少提供 2 GB 内存
- 网络连接 – 确保计算机位于一个或多个安全网络中
- Windows Server 2012 R2 或 Windows Server 2012（需要为此系统安装 KB2858668）。
- .NET Framework 4.5.1 或更高版本的完整安装版
- Windows PowerShell 3.0
- 部署和配置群集的群集管理员必须拥有每台计算机的管理员权限。
- 应在所有计算机上运行 RemoteRegistry 服务。

### 步骤 3：确定初始群集大小
每个节点都包含完整的 Service Fabric 堆栈，并且是 Service Fabric 群集的独立成员。在典型的 Service Fabric 部署中，每个 OS 实例（物理或虚拟）都有一个节点。群集大小取决于业务要求；但是，一个群集必须至少包含三个节点（计算机/VM）。请注意，出于开发目的，可以在一台指定的计算机上创建多个节点。在生产环境中，对于每个物理机或虚拟机，Service Fabric 只支持一个节点。

### 步骤 4：确定容错域和升级域的数目
**容错域 (FD)** 是故障的物理单元，与数据中心内的物理基础结构直接相关。容错域由共享单一故障点的硬件组件（计算机、交换机等）组成。尽管容错域与机架之间没有 1:1 映射，但是大致上，可将每个机架视为一个容错域。在考虑群集中的节点时，强烈建议至少在三个容错域之间分布节点。

当你在 *ClusterConfig.JSON* 中指定 FD 时，必须选择 FD 的名称。Service Fabric 支持分层的 FD，因此，你可以在 FD 中反映基础结构拓扑。例如，允许以下配置：

- "faultDomain": "fd:/Room1/Rack1/Machine1"
- "faultDomain": "fd:/FD1"
- "faultDomain": "fd:/Room1/Rack1/PDU1/M1"


**升级域 (UD)** 是节点的逻辑单元。在 Service Fabric 协调式升级（应用程序升级或群集升级）期间，将关闭 UD 中的所有节点以执行升级，而其他 UD 中的节点仍可用来为请求提供服务。对计算机进行的固件升级将不遵循 UD，因此你每次必须在一台计算机上执行此操作。

领会这些概念的最简单方法是将 FD 视为非计划内故障的单元，将 UD 视为计划维护的单元。

当你在 *ClusterConfig.JSON* 中指定 UD 时，必须选择 UD 的名称。例如，允许以下所有配置：

- "upgradeDomain": "UD0"
- "upgradeDomain": "UD1A"
- "upgradeDomain": "DomainRed"
- "upgradeDomain": "Blue"

### 步骤 5：下载适用于 Windows Server 的 Service Fabric 独立包
下载[适用于 Windows Server 的 Service Fabric 独立包](http://go.microsoft.com/fwlink/?LinkId=730690)，并将包解压缩到不属于群集的部署计算机或将会属于群集的一个计算机中。

<a id="createcluster"></a>
## 创建群集

完成以上规划和准备部分中所述的步骤之后，可以开始创建群集。

### 步骤 1：修改群集配置
打开下载的包中的 *ClusterConfig.JSON*。可以使用选择的任何编辑器来修改以下设置：

|**配置设置**|**说明**|
|-----------------------|--------------------------|
|NodeTypes|节点类型可让你将群集节点划分到不同的组中。一个群集必须至少有一个节点类型。组中的所有节点具有以下共同特征。<br>*Name* - 这是节点类型名称。<br>*EndPoints* - 这是与此节点类型关联的各种命名终结点（端口）。你可以使用任何所需的端口号，前提是它们不与此清单中的其他任何端口冲突，并且计算机/VM 上的其他任何程序未使用该端口号 <br>*PlacementProperties* - 描述此节点类型的属性，你可以使用这些属性作为系统服务或服务的放置约束。这些属性是用户定义的键/值对，可为指定节点提供额外的元数据。节点属性的示例包括节点是否有硬盘或图形卡、其硬盘的轴数、核心和其他物理属性。<br>*Capacities* - 节点容量，定义特定节点可以使用的特定资源的名称和数量。例如，节点可以定义名为“MemoryInMb”的指标容量，而且默认有 2048 MB 的可用内存。这些容量在运行时使用，以确保将需要特定资源量的服务放在具有剩余可用资源的节点上。|
|Nodes|将属于群集的每个节点的详细信息（节点类型、节点名称、IP 地址、节点的容错域和升级域）。要在其上创建群集的计算机必须与其 IP 地址一起列在此处。<br>如果你对所有节点使用相同的 IP 地址，将会创建一个可用于测试的单机群集。不应将单机群集用于部署生产工作负荷。|

### 步骤 2：运行创建群集脚本
在 JSON 文档中修改群集配置并在其中添加所有节点信息之后，运行包文件夹中的群集创建 PowerShell 脚本，并传入配置文件的路径和包根目录的位置。

可以在对群集配置文件中列为节点的所有计算机具有管理访问权限的任何计算机上运行此脚本。运行此脚本的计算机不一定是群集的一部分。

```
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath C:\Microsoft.Azure.ServiceFabric.WindowsServer.5.0.135.9590\ClusterConfig.JSON -MicrosoftServiceFabricCabFilePath C:\Microsoft.Azure.ServiceFabric.WindowsServer.5.0.135.9590\MicrosoftAzureServiceFabric.cab
```

## 后续步骤

创建群集后，请务必对它进行保护：
- [群集安全性](/documentation/articles/service-fabric-cluster-security/)

若要开始开发和部署应用，请阅读以下文章：

- [Service Fabric SDK 及其入门](/documentation/articles/service-fabric-get-started/)
- [在 Visual Studio 中管理 Service Fabric 应用程序](/documentation/articles/service-fabric-manage-application-in-visual-studio/)。

阅读有关 Azure 群集和独立群集的详细信息：

- [Overview of the standalone cluster creation feature and a comparison with Azure-managed clusters（独立群集创建功能的概述及其与 Azure 托管群集的比较）](/documentation/articles/service-fabric-deploy-anywhere/)

<!---HONumber=Mooncake_0627_2016-->