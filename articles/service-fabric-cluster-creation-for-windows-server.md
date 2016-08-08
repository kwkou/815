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
   ms.date="07/05/2016"
   wacn.date="08/08/2016"/>


# 创建在 Windows Server 上运行的群集

Azure Service Fabric 允许在运行 Windows Server 的任何虚拟机或计算机上创建 Service Fabric 群集。这意味着你可以在具有一组相互连接的 Windows Server 计算机的任何环境（无论是本地环境还是任何云提供商所提供的）中部署和运行 Service Fabric 应用程序。Service Fabric 提供了一个安装程序包以供你创建名为“Windows Server 独立包”的 Service Fabric 群集。

本文将引导你使用 Service Fabric 的独立包在本地完成创建群集的步骤，不过，你也可以针对其他任何环境（例如其他云提供商）轻松地进行调整。

>[AZURE.NOTE] 此 Windows Server 独立包目前为预览版，不支持用于生产工作负荷。如果你想要立即下载一份 EULA，请[单击此处](http://go.microsoft.com/fwlink/?LinkID=733084)。

<a id="downloadpackage"></a>
## 下载 Service Fabric 独立包


[下载适用于 Windows Server 2012 R2 及更高版本的 Service Fabric 独立包](http://go.microsoft.com/fwlink/?LinkId=730690)，其名为 Microsoft.Azure.ServiceFabric.WindowsServer.&lt;版本&gt;.zip。

在下载包中，你将看到以下文件：

|**文件名**|**简短说明**|
|-----------------------|--------------------------|
|MicrosoftAzureServiceFabric.cab|包含部署到群集中每台计算机的二进制文件的 CAB 文件。|
|ClusterConfig.Unsecure.DevCluster.json|群集配置示例文件，包含的设置适用于非安全型三节点式单 VM/计算机开发群集，这些设置包括群集中每个节点的信息。 |
|ClusterConfig.Unsecure.MultiMachine.json|群集配置示例文件，包含的设置适用于非安全型多 VM/计算机群集，这些设置包括群集中每个计算机的信息。|
|ClusterConfig.Windows.DevCluster.json|群集配置示例文件，包含的所有设置适用于安全型三节点式单 VM/计算机开发群集，这些设置包括群集中每个节点的信息。此群集使用 [Windows 标识](https://msdn.microsoft.com/zh-cn/library/ff649396.aspx)进行保护。|
|ClusterConfig.Windows.MultiMachine.json|群集配置示例文件，包含的所有设置适用于使用 Windows 安全性措施的安全型多 VM/计算机群集，这些设置包括安全群集中每个计算机的信息。此群集使用 [Windows 标识](https://msdn.microsoft.com/zh-cn/library/ff649396.aspx)进行保护。|
|ClusterConfig.x509.DevCluster.json|群集配置示例文件，包含的所有设置适用于安全型三节点式单 VM/计算机开发群集，这些设置包括群集中每个节点的信息。此群集使用 x509 证书进行保护。|
|ClusterConfig.x509.MultiMachine.json|群集配置示例文件，包含的所有设置适用于安全型多 VM/计算机群集，这些设置包括安全群集中每个节点的信息。此群集使用 x509 证书进行保护。|
|EULA.txt|有关使用 Microsoft Azure Service Fabric Windows Server 独立包的许可条款。如果你想要立即下载一份 EULA，请[单击此处](http://go.microsoft.com/fwlink/?LinkID=733084)。|
|Readme.txt|发行说明和基本安装说明的链接。这是你可以在此页上找到的一部分说明。|
|CreateServiceFabricCluster.ps1|PowerShell 脚本，用于通过 ClusterConfig.json 文件中的设置创建群集。|
|RemoveServiceFabricCluster.ps1|PowerShell 脚本，用于通过 ClusterConfig.json 中的设置删除群集。|
|AddNode.ps1|PowerShell 脚本，用于将节点添加到现有的部署群集。|
|RemoveNode.ps1|PowerShell 脚本，用于将节点从现有的部署群集中删除。|


## 规划和准备群集部署
在创建群集之前，需要执行以下步骤。

### 步骤 1：规划群集基础结构
你需要在所拥有的计算机上创建 Service Fabric 群集，因此必须确定群集需应对的故障类型。例如，是否需要为这些计算机单独提供电源线或 Internet 连接？ 此外，还应该考虑这些计算机的物理安全性。物理位置在哪里？谁需要访问这些计算机？ 一旦做出这些决定，就要以逻辑方式将计算机映射到多个容错域（有关详细信息，请参阅下文）。相比于测试群集，生产群集的基础结构规划要更复杂。

### 步骤 2：准备符合先决条件的计算机
要添加到群集的每台计算机的先决条件：

- 建议至少提供 2 GB 内存
- 网络连接。确保计算机位于一个或多个安全网络中
- Windows Server 2012 R2 或 Windows Server 2012（需要为此系统安装 KB2858668）。
- .NET Framework 4.5.1 或更高版本的完整安装版
- Windows PowerShell 3.0
- 部署和配置群集的群集管理员必须拥有每台计算机的管理员权限。
- 应在所有计算机上运行 RemoteRegistry 服务。

### 步骤 3：确定初始群集大小
每个节点均部署 Service Fabric 运行时，并且是群集的成员。在典型的生产部署中，每个 OS 实例（物理或虚拟）都有一个节点。群集大小取决于业务要求；但是，一个群集必须至少包含三个节点（计算机/VM）。
请注意，出于开发目的，可以在一台指定的计算机上创建多个节点。在生产环境中，对于每个物理机或虚拟机，Service Fabric 只支持一个节点。

### 步骤 4：确定容错域和升级域的数目
**容错域 (FD)** 是故障的物理单元，与数据中心的物理基础结构直接相关。容错域由共享单一故障点的硬件组件（计算机、交换机、网络等）组成。尽管容错域与机架之间没有 1:1 映射，但是大致上，可将每个机架视为一个容错域。在考虑群集中的节点时，强烈建议至少在三个容错域之间分布节点。

当你在 ClusterConfig.json 中指定 FD 时，必须选择每个 FD 的名称。Service Fabric 支持分层的 FD，因此，你可以在 FD 中反映基础结构拓扑。例如，以下 FD 是有效的：

- "faultDomain": "fd:/Room1/Rack1/Machine1"
- "faultDomain": "fd:/FD1"
- "faultDomain": "fd:/Room1/Rack1/PDU1/M1"


**升级域 (UD)** 是节点的逻辑单元。在 Service Fabric 协调式升级（应用程序升级或群集升级）期间，将关闭 UD 中的所有节点以执行升级，而其他 UD 中的节点仍可用来为请求提供服务。对计算机进行的固件升级将不遵循 UD，因此你每次必须在一台计算机上执行此操作。

领会这些概念的最简单方法是将 FD 视为非计划内故障的单元，将 UD 视为计划维护的单元。

当你在 ClusterConfig.json 中指定 UD 时，必须选择每个 UD 的名称。例如，以下配置是有效的：

- "upgradeDomain": "UD0"
- "upgradeDomain": "UD1A"
- "upgradeDomain": "DomainRed"
- "upgradeDomain": "Blue"

有关升级域和容错域的更多详细信息，请阅读[描述 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)一文。

### 步骤 5：下载适用于 Windows Server 的 Service Fabric 独立包
下载[适用于 Windows Server 的 Service Fabric 独立包](http://go.microsoft.com/fwlink/?LinkId=730690)，并将包解压缩到不属于群集的部署计算机或将会属于群集的一个计算机中。

<a id="createcluster"></a>
## 创建群集

完成以上规划和准备部分中所述的步骤之后，可以开始创建群集。

### 步骤 1：修改群集配置
ClusterConfig.json 文件对群集进行了描述。有关此文件中相关部分的详细信息，请阅读 [Windows 独立群集的配置设置](/documentation/articles/service-fabric-cluster-manifest/)一文。
从已下载的程序包中打开某个 ClusterConfig.json 文件，然后修改以下设置：

|**配置设置**|**说明**|
|-----------------------|--------------------------|
|**NodeTypes**|节点类型可让你将群集节点划分到不同的组中。一个群集必须至少有一个节点类型。组中的所有节点具有以下共同特征。<br> **名称** - 即节点类型名称。<br>**终结点端口** - 即与此节点类型关联的各种命名终结点（端口）。你可以使用任何所需的端口号，前提是它们不与此清单中的其他任何端口冲突，并且计算机/VM 上运行的其他任何应用程序未使用该端口号 <br> **放置属性** - 描述此节点类型的属性，你可以使用这些属性作为系统服务或你的服务的放置约束。这些属性是用户定义的键/值对，可为指定节点提供额外的元数据。节点属性的示例包括节点是否有硬盘或图形卡、其硬盘的轴数、核心和其他物理属性。<br> **容量** - 节点容量，定义特定节点提供的特定资源的名称和数量。例如，节点可以定义名为“MemoryInMb”的指标容量，而且默认有 2048 MB 的可用内存。这些容量可在运行时使用，以确保将需要特定数量资源的服务放置在始终可以使用这些资源的节点上。<br>**IsPrimary** - 如果已定义多个 NodeType，请确保只将一个 NodeType（在其中运行系统服务）设置为主 NodeType（使用值 true 进行设置）。应将所有其他节点类型设置为 false 值|
|**Nodes**|属于群集的每个节点的详细信息（节点类型、节点名称、IP 地址、节点的容错域和升级域）。要在其上创建群集的计算机必须与其 IP 地址一起列在此处。<br>如果你对所有节点使用相同的 IP 地址，将会创建一个可用于测试的单机群集。不应将单机群集用于部署生产工作负荷。|

### 步骤 2：运行创建群集脚本
在 JSON 文档中修改群集配置并在其中添加所有节点信息之后，运行包文件夹中用于创建群集的 CreateServiceFabricCluster.ps1 PowerShell 脚本，并传入 JSON 配置文件的路径和包 CAB 文件的位置。

可以在对群集配置文件中列为节点的所有计算机具有管理访问权限的任何计算机上运行此脚本。运行此脚本的计算机不一定是群集的一部分。


	#Create an unsecure local development cluster

	.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab -AcceptEULA $true


	#Create an unsecure multi-machine cluster

	.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab -AcceptEULA $true


>[AZURE.NOTE] 部署日志可在 VM/计算机（已在其上运行 CreateServiceFabricCluster Powershell）上本地使用。你可以在名为“DeploymentTraces”的子文件夹中找到这些日志，该子文件夹位于运行过 Powershell 命令的文件夹中。另外，若要查看 Service Fabric 是否已正确部署到某个计算机，可在 C:\\ProgramData 目录中查找已安装文件，并可在任务管理器中查看正在运行的 FabricHost.exe 和 Fabric.exe 进程。

### 步骤 3：连接到群集
你现在可以通过 Service Fabric Explorer 连接到群集，既可以直接从装有 http://localhost:19080/Explorer/index.html 的某个计算机进行连接，也可以通过 http://<IPAddressofaMachine>:19080/Explorer/index.html 进行远程连接

## 向群集添加节点

1. 准备要添加到群集的 VM/计算机（请参阅上述“规划和准备群集部署”部分的步骤 2）。
2. 规划要向其添加此 VM/计算机的容错域和升级域。
3. 复制或[下载适用于 Windows Server 的 Service Fabric 独立包](http://go.microsoft.com/fwlink/?LinkId=730690)，然后将该包解压缩到计划添加到群集的 VM/计算机。
4. 打开 Powershell 管理员提示符，导航到解压缩包所在的位置。
5. 运行 AddNode.ps1 Powershell，使用参数来描述要添加的新节点。以下示例将名为 VM5、类型为 NodeType0 且 IP 地址为 182.17.34.52 的新节点添加到 UD1 和 FD1 中。ExistingClusterConnectionEndPoint 是现有群集中已有节点的连接终结点。你可以在群集中针对此项选择任意节点 IP 地址。


	.\\AddNode.ps1 -MicrosoftServiceFabricCabFilePath .\\MicrosoftAzureServiceFabric.cab -NodeName VM5 -NodeType NodeType0 -NodeIPAddressorFQDN 182.17.34.52 -ExistingClusterConnectionEndPoint 182.17.34.50:19000 -UpgradeDomain UD1 -FaultDomain FD1


## 从群集中删除节点

1. 通过远程桌面 (RDP) 方式进入需要从群集中删除的 VM/计算机。
2. 复制或[下载适用于 Windows Server 的 Service Fabric 独立包](http://go.microsoft.com/fwlink/?LinkId=730690)，然后将该包解压缩到计划添加到群集的 VM/计算机。
3. 打开 Powershell 管理员提示符，导航到解压缩包所在的位置。
4. 运行 RemoveNode.ps1 Powershell。以下示例从群集中删除当前节点。ExistingClusterConnectionEndPoint 是现有群集中已有节点的连接终结点。你可以在群集中针对此项选择任意节点 IP 地址。


	.\\RemoveNode.ps1 -MicrosoftServiceFabricCabFilePath .\\MicrosoftAzureServiceFabric.cab -ExistingClusterConnectionEndPoint 182.17.34.50:19000


## 删除群集
若要删除群集，请运行包文件夹中的 RemoveServiceFabricCluster.ps1 Powershell 脚本，并传入 JSON 配置文件的路径以及包 CAB 文件所在的位置。

可以在对群集配置文件中列为节点的所有计算机具有管理访问权限的任何计算机上运行此脚本。运行此脚本的计算机不一定是群集的一部分


	.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json   -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab


## 如何：使用 Azure IaaS VM 创建三节点式群集
以下步骤描述如何在 Azure IaaS VM 上使用独立的 Windows Server 安装程序创建群集。请注意，此 IaaS 群集上的 Service Fabric 运行时完全由你管理，不像通过 Azure 门户创建的群集，后者通过 Service Fabric 资源提供程序进行升级。

1. 登录到 Azure 门户，在资源组中创建新的 Windows Server 2012 R2 Datacenter VM。
2. 向同一资源组再添加两个 Windows Server 2012 R2 Datacenter VM。确保每个 VM 在创建后都有相同的管理员用户名和密码。创建以后，你会在同一虚拟网络中看到所有三个 VM。
3. 使用服务器管理器的“本地服务器”仪表板连接到每个 VM 并关闭 Windows 防火墙。这确保网络流量可以在计算机之间通信。同时在每个计算机上，通过打开命令提示符并键入 ipconfig 来获取 IP 地址。另外，你也可以在 Azure 门户中选择资源组的虚拟网络资源，这样即可查看每个计算机的 IP 地址
4. 连接到其中一台计算机，测试是否可以成功地 ping 其他两台计算机。
5. 连接到其中一台 VM，将 Windows Server 独立包下载到此虚拟机的新文件夹中，然后解压缩该包。
6. 在记事本中打开 ClusterConfig.Unsecure.MultiMachine.json 文件，使用计算机的三个 IP 地址编辑每个节点，更改顶部的群集名称，然后保存文件。群集清单的部分示例如下所示，显示了每台计算机的 IP 地址。


	    {
	        "name": "TestCluster",
	        "clusterManifestVersion": "1.0.0",
	        "apiVersion": "2015-01-01-alpha",
	        "nodes": [
	        {
	            "nodeName": "vm0",
	        	"metadata": "Replace the localhost with valid IP address or FQDN below",
	            "iPAddress": "10.7.0.5",
	            "nodeTypeRef": "NodeType0",
	            "faultDomain": "fd:/dc1/r0",
	            "upgradeDomain": "UD0"
	        },
	        {
	            "nodeName": "vm1",
	        	"metadata": "Replace the localhost with valid IP address or FQDN below",
	            "iPAddress": "10.7.0.4",
	            "nodeTypeRef": "NodeType0",
	            "faultDomain": "fd:/dc2/r0",
	            "upgradeDomain": "UD1"
	        },
	        {
	            "nodeName": "vm2",
	        	"metadata": "Replace the localhost with valid IP address or FQDN below",
	            "iPAddress": "10.7.0.6",
	            "nodeTypeRef": "NodeType0",
	            "faultDomain": "fd:/dc3/r0",
	            "upgradeDomain": "UD2"
	        }
	    ],


7. 打开 Powershell ISE 窗口，导航到曾经下载和解压缩独立安装程序包并保存上述清单文件的文件夹。运行以下 Powershell 命令。


    	.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json -MicrosoftServiceFabricCabFilePath .\MicrosoftAzureServiceFabric.cab


8. 你会看到 Powershell 运行，此时请连接到每台计算机并创建群集。大约 1 分钟后，你可以通过某个计算机 IP 地址（例如 http://10.7.0.5:19080/Explorer/index.html）连接到 Service Fabric Explorer，以便测试群集是否正常运行。由于该群集是 IaaS VM 上的独立群集，因此若要确保其安全，则需将证书部署到 VM，或者将某台计算机设置为 Windows Server Active Directory (AD) 控制器，以便进行 Windows 身份验证，就像在本地执行相关操作一样。请参阅下面的“后续步骤”，了解如何设置安全群集。

## 后续步骤
- [在 Windows Server 或 Linux 上创建独立的 Service Fabric 群集](/documentation/articles/service-fabric-deploy-anywhere/)
- [Windows 独立群集的配置设置](/documentation/articles/service-fabric-cluster-manifest/)

- [使用 Windows 安全性保护 Windows 上的独立群集](/documentation/articles/service-fabric-windows-cluster-windows-security/)
- [使用 X509 证书保护 Windows 上的独立群集](/documentation/articles/service-fabric-windows-cluster-x509-security/)

<!---HONumber=Mooncake_0801_2016-->