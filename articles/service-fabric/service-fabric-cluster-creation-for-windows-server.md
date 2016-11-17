<properties
   pageTitle="创建和管理独立 Azure Service Fabric 群集 | Azure"
   description="了解如何在运行 Windows Server 的任何本地或任意云计算机上创建和管理 Azure Service Fabric 群集（物理或虚拟）。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/26/2016"
   wacn.date="10/24/2016"
   ms.author="dkshir;chackdan"/>


# 创建和管理在 Windows Server 上运行的群集

Azure Service Fabric 允许在运行 Windows Server 的任何虚拟机或计算机上创建 Service Fabric 群集。这意味着您可以在包含一组相互连接的 Windows Server 计算机的任何环境（无论是本地环境还是任何云提供商所提供的环境）中部署和运行 Service Fabric 应用程序。Service Fabric 提供了一个安装程序包，用于创建名为“Windows Server 独立包”的 Service Fabric 群集。

本文将引导你使用 Service Fabric 的独立包在本地完成创建群集的步骤，不过，你也可以针对其他任何环境（例如其他云提供商）轻松地进行调整。

>[AZURE.NOTE] 此独立 Windows Server 包中的功能目前为预览版，不支持用于商用目的。若要预览版功能列表，请向下滚动到本文档末尾。如果你想要立即下载一份 EULA，请[单击此处](http://go.microsoft.com/fwlink/?LinkID=733084)。

<a id="downloadpackage"></a>
## 下载 Service Fabric 独立包


[下载适用于 Windows Server 2012 R2 及更高版本的 Service Fabric 独立包](http://go.microsoft.com/fwlink/?LinkId=730690)，其名为 *Microsoft.Azure.ServiceFabric.WindowsServer.&lt;版本&gt;.zip*。

>[AZURE.NOTE] 如果使用 Internet Explorer 或 Microsoft Edge 浏览器下载此包，必须将 [http://download.microsoft.com](http://download.microsoft.com) 添加到 Intranet 中的受信任站点，防止在下载 zip 时，将“区域”标记写入文件

 ![TustedZone][TrustedZone]  


在下载的包可以看到以下文件：

|**文件名**|**简短说明**|
|-----------------------|--------------------------|
|MicrosoftAzureServiceFabric.cab|包含部署到群集中每台计算机的二进制文件的 CAB 文件。|
|ClusterConfig.Unsecure.DevCluster.json|群集配置示例文件包含的设置适用于非安全型三节点式单 VM/计算机开发群集，这些设置包括群集中每个节点的信息。 |
|ClusterConfig.Unsecure.MultiMachine.json|群集配置示例文件包含的设置适用于非安全型多 VM/计算机群集，这些设置包括群集中每个计算机的信息。|
|ClusterConfig.Windows.DevCluster.json|群集配置示例文件，包含的所有设置适用于安全型三节点式单 VM/计算机开发群集，这些设置包括群集中每个节点的信息。此群集使用 [Windows 标识](https://msdn.microsoft.com/zh-cn/library/ff649396.aspx)进行保护。|
|ClusterConfig.Windows.MultiMachine.json|群集配置示例文件，包含的所有设置适用于使用 Windows 安全性措施的安全型多 VM/计算机群集，这些设置包括安全群集中每个计算机的信息。此群集使用 [Windows 标识](https://msdn.microsoft.com/zh-cn/library/ff649396.aspx)进行保护。|
|ClusterConfig.x509.DevCluster.json|群集配置示例文件，包含的所有设置适用于安全型三节点式单 VM/计算机开发群集，这些设置包括群集中每个节点的信息。此群集使用 x509 证书进行保护。|
|ClusterConfig.x509.MultiMachine.json|群集配置示例文件，包含的所有设置适用于安全型多 VM/计算机群集，这些设置包括安全群集中每个节点的信息。此群集使用 x509 证书进行保护。|
|EULA.txt|有关使用 Microsoft Azure Service Fabric Windows Server 独立包的许可条款。如果你想要立即下载一份 EULA，请[单击此处](http://go.microsoft.com/fwlink/?LinkID=733084)。|
|Readme.txt|发行说明和基本安装说明的链接。这是可以在此页上找到的一部分说明。|
|CreateServiceFabricCluster.ps1|PowerShell 脚本，用于通过 ClusterConfig.json 文件中的设置创建群集。|
|RemoveServiceFabricCluster.ps1|PowerShell 脚本，用于通过 ClusterConfig.json 中的设置删除群集。|
|AddNode.ps1|PowerShell 脚本，用于将节点添加到现有的部署群集。|
|RemoveNode.ps1|PowerShell 脚本，用于将节点从现有的部署群集中删除。|


## 规划和准备群集部署
在创建群集之前，请执行以下步骤。

### 步骤 1：规划群集基础结构
您需要在所拥有的计算机上创建 Service Fabric 群集，以便确定群集需应对的故障类型。例如，是否需要为这些计算机单独提供电源线或 Internet 连接？ 此外，还应考虑这些计算机的物理安全性。物理位置在哪里？谁需要访问这些计算机？ 做出这些决定后，以逻辑方式将计算机映射到多个容错域（有关详细信息，请向下滚动）。相比于测试群集，生产群集的基础结构规划要更复杂。

<a id="preparemachines"></a>
### 步骤 2：准备符合先决条件的计算机
要添加到群集的每台计算机的先决条件：

- 建议至少提供 2 GB 内存
- 网络连接 – 确保计算机位于一个或多个安全网络中。
- Windows Server 2012 R2 或 Windows Server 2012（需要系统安装 [KB2858668](https://support.microsoft.com/zh-cn/kb/2858668)）。
- [.NET Framework 4.5.1 或更高版本](https://www.microsoft.com/download/details.aspx?id=40773)的完整安装版。
- [Windows PowerShell 3.0](https://msdn.microsoft.com/zh-cn/powershell/scripting/setup/installing-windows-powershell)。
- 部署和配置群集的群集管理员必须拥有每台计算机的[管理员权限](https://social.technet.microsoft.com/wiki/contents/articles/13436.windows-server-2012-how-to-add-an-account-to-a-local-administrator-group.aspx)。
- 应在所有计算机上运行 [RemoteRegistry 服务](https://technet.microsoft.com/zh-cn/library/cc754820)。

### 步骤 3：确定初始群集大小
独立 Service Fabric 群集中的每个节点都应部署 Service Fabric 运行时，并且都是该群集的成员。在典型的生产部署中，每个 OS 实例（物理或虚拟）都有一个节点。群集大小取决于业务要求；但是，一个群集必须至少包含三个节点（计算机/VM）。出于开发目的，可以在一台指定的计算机上创建多个节点。在生产环境中，对于每个物理机或虚拟机，Service Fabric 只支持一个节点。

### 步骤 4：确定容错域和升级域的数目
**容错域 (FD)** 是故障的物理单元，与数据中心的物理基础结构直接相关。容错域由共享单一故障点的硬件组件（计算机、交换机、网络等）组成。尽管容错域与机架之间没有 1:1 映射，但是大致上，可将每个机架视为一个容错域。在考虑群集中的节点时，强烈建议至少在三个容错域之间分布节点。

当在 *ClusterConfig.json* 中指定 FD 时，可以选择每个 FD 的名称。Service Fabric 支持分层的 FD，因此，你可以在 FD 中反映基础结构拓扑。例如，以下 FD 是有效的：

- "faultDomain": "fd:/Room1/Rack1/Machine1"
- "faultDomain": "fd:/FD1"
- "faultDomain": "fd:/Room1/Rack1/PDU1/M1"


**升级域 (UD)** 是节点的逻辑单元。在 Service Fabric 协调式升级（应用程序升级或群集升级）期间，将关闭 UD 中的所有节点以执行升级，而其他 UD 中的节点仍可用来为请求提供服务。对计算机进行的固件升级将不遵循 UD，因此每次必须在一台计算机上执行此操作。

领会这些概念的最简单方法是将 FD 视为非计划内故障的单元，将 UD 视为计划维护的单元。

当在 *ClusterConfig.json* 中指定 UD 时，可以选择每个 UD 的名称。例如，以下配置是有效的：

- "upgradeDomain": "UD0"
- "upgradeDomain": "UD1A"
- "upgradeDomain": "DomainRed"
- "upgradeDomain": "Blue"

有关升级域和容错域的更多详细信息，请阅读[Service Fabric 群集介绍](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)一文。

### 步骤 5：下载适用于 Windows Server 的 Service Fabric 独立包
下载[适用于 Windows Server 的 Service Fabric 独立包](http://go.microsoft.com/fwlink/?LinkId=730690)，并将包解压缩到群集外的一台部署计算机中或解压缩到群集内的其中一台计算机中。

<a id="createcluster"></a>
## 创建群集

完成以上规划和准备部分中所述的步骤之后，可以开始创建群集。

### 步骤 1：修改群集配置
*ClusterConfig.json* 文件对群集进行了描述。有关此文件中相关部分的详细信息，请阅读 [Windows 独立群集的配置设置](/documentation/articles/service-fabric-cluster-manifest/)一文。从已下载的程序包中打开某个 *ClusterConfig.json* 文件，然后修改以下设置：

|**配置设置**|**说明**|
|-----------------------|--------------------------|
|**NodeTypes**|节点类型可让你将群集节点划分到不同的组中。一个群集必须至少有一个节点类型。组中的所有节点具有以下共同特征：<br>**名称** - 节点类型名称。<br>**终结点端口** - 即与此节点类型关联的各种命名终结点（端口）。可以使用任何端口号，只要它们不会与此清单中的其他部分发生冲突，并且未被计算机/VM 上运行的其他应用程序使用。<br>**放置属性** - 即此节点类型的相应属性，可用作系统服务或您拥有的服务的放置约束。这些属性是用户定义的键/值对，可为指定节点提供额外的元数据。节点属性的示例包括节点是否有硬盘或图形卡、其硬盘的轴数、核心和其他物理属性。<br>**容量** - 节点容量，定义特定节点提供的特定资源的名称和数量。例如，节点可以定义名为“MemoryInMb”的指标容量，而且默认有 2048 MB 的可用内存。这些容量可在运行时使用，以确保将需要特定数量资源的服务放置在具有所需的可用资源量的节点上。<br>**IsPrimary** - 如果已定义多个 NodeType，请确保只将一个 NodeType（在其中运行系统服务）设置为主 NodeType（使用值 *true* 进行设置）。应将所有其他节点类型设置为 *false* 值|
|**Nodes**|这些是群集内的每个节点的详细信息（节点类型、节点名称、IP 地址、节点的容错域和升级域）。要在其上创建群集的计算机必须与其 IP 地址一起列在此处。<br>如果对所有节点使用相同的 IP 地址，则会创建一个可用于测试的单机群集。不要将单机群集用于部署生产工作负荷。|

### 步骤 2：运行创建群集脚本
在 JSON 文档中修改群集配置并在其中添加所有节点信息之后，运行包文件夹中用于创建群集的 *CreateServiceFabricCluster.ps1* PowerShell 脚本，并传入 JSON 配置文件的路径和包 CAB 文件的位置。

可以在对群集配置文件中列为节点的所有计算机具有管理员访问权限的任何计算机上运行此脚本。运行此脚本的计算机不一定是群集的一部分。


	#Create an unsecured local development cluster

	.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab -AcceptEULA $true


	#Create an unsecured multi-machine cluster

	.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab -AcceptEULA $true


>[AZURE.NOTE] 部署日志可在 VM/计算机（已在其上运行 CreateServiceFabricCluster Powershell）上本地使用。您可以在名为“DeploymentTraces”的子文件夹中找到这些日志，该子文件夹位于运行过 Powershell 命令的文件夹中。另外，若要查看 Service Fabric 是否已正确部署到某个计算机，可在 C:\\ProgramData 目录中查找已安装文件，并可在任务管理器中查看正在运行的 FabricHost.exe 和 Fabric.exe 进程。

### 步骤 3：连接到群集

有关连接到安全群集的说明，请参阅[此文档](/documentation/articles/service-fabric-connect-to-secure-cluster/)。

若要连接到不安全的群集，请运行以下 PowerShell 命令



	Connect-ServiceFabricCluster -ConnectionEndpoint <*IPAddressofaMachine*>:<Client connection end point port>

	Connect-ServiceFabricCluster -ConnectionEndpoint 192.13.123.2345:19000


### 步骤 4：打开 Service Fabric Explorer

您现在可以通过 Service Fabric Explorer 连接到群集，既可以直接从装有 http://localhost:19080/Explorer/index.html 的某台计算机进行连接，也可以通过 http://<*IPAddressofaMachine*>:19080/Explorer/index.html 进行远程连接



## 向群集添加节点或删除群集的节点

当业务需要改变时，您可以向独立 Service Fabric 群集添加或删除节点。阅读[向 Service Fabric 独立群集添加或删节点](/documentation/articles/service-fabric-cluster-windows-server-add-remove-nodes/)以了解详细步骤。


## 删除群集

若要删除群集，请运行包文件夹中的 *RemoveServiceFabricCluster.ps1* Powershell 脚本，并传入 JSON 配置文件的路径以及包 CAB 文件所在的位置。

可以在对群集配置文件中列为节点的所有计算机具有管理员访问权限的任何计算机上运行此脚本。运行此脚本的计算机不一定是群集的一部分


	.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json   -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab

## 此包中的预览版功能

整个包目前为预览版。

## 后续步骤
- [Windows 独立群集的配置设置](/documentation/articles/service-fabric-cluster-manifest/)
- [向独立 Service Fabric 群集添加或删除节点](/documentation/articles/service-fabric-cluster-windows-server-add-remove-nodes/)
- [使用运行 Windows 的 Azure VM 创建独立 Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-with-windows-azure-vms/)
- [使用 Windows 安全性保护 Windows 上的独立群集](/documentation/articles/service-fabric-windows-cluster-windows-security/)
- [使用 X509 证书保护 Windows 上的独立群集](/documentation/articles/service-fabric-windows-cluster-x509-security/)


<!--Image references-->

[TrustedZone]: ./media/service-fabric-cluster-creation-for-windows-server/TrustedZone.png

<!---HONumber=Mooncake_1017_2016-->