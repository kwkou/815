<properties
   pageTitle="配置独立群集 | Azure"
   description="本文介绍如何配置独立的或专用的 Service Fabric 群集。"
   services="service-fabric"
   documentationCenter=".net"
   authors="dsk-2015"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="06/23/2016"
   wacn.date="08/08/2016"
   ms.author="dkshir"/>


# Windows 独立群集的配置设置

本文介绍如何使用 **ClusterConfig.JSON** 文件来配置独立的 Service Fabric 群集。该文件将在你[下载 Service Fabric 独立包](/documentation/articles/service-fabric-cluster-creation-for-windows-server/#downloadpackage)时下载到你的工作计算机。ClusterConfig.JSON 文件允许你指定 Service Fabric 群集的各种信息，例如 Service Fabric 节点及其 IP 地址、群集上不同类型的节点、安全配置，以及使用容错域/升级域来定义的网络拓扑。

我们将在下面探讨此文件的各个部分。

## 常规群集配置
此部分介绍各种特定于群集的配置，如下面的 JSON 代码段所示。

    "name": "SampleCluster",
    "clusterManifestVersion": "1.0.0",
    "apiVersion": "2015-01-01-alpha",

你可以为 Service Fabric 群集提供任何友好名称，只需将其指定给 **name** 变量即可。你可以根据设置来更改 **clusterManifestVersion**；需先更新此项，然后才能升级 Service Fabric 配置。你可以让 **apiVersion** 保留默认值。


<a id="clusternodes"></a>
## 群集上的节点
你可以使用 **nodes** 部分来配置 Service Fabric 群集上的节点，如以下代码段所示。

    "nodes": [{
        "nodeName": "vm0",
        "iPAddress": "localhost",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r0",
        "upgradeDomain": "UD0"
    }, {
        "nodeName": "vm1",
        "iPAddress": "localhost",
        "nodeTypeRef": "NodeType1",
        "faultDomain": "fd:/dc1/r1",
        "upgradeDomain": "UD1"
    }, {
        "nodeName": "vm2",
        "iPAddress": "localhost",
        "nodeTypeRef": "NodeType2",
        "faultDomain": "fd:/dc1/r2",
        "upgradeDomain": "UD2"
    }],

一个 Service Fabric 群集需要至少 3 个节点。你可以根据设置向此部分添加更多节点。下表说明了每个节点的配置设置。

|**节点配置**|**说明**|
|-----------------------|--------------------------|
|nodeName|你可以为节点提供任何友好名称。|
|iPAddress|找出节点的 IP 地址，方法是打开一个命令窗口，然后键入 `ipconfig`。记下 IPV4 地址，将其分配给 **iPAddress** 变量。|
|nodeTypeRef|可以为每个节点分配不同的节点类型。[节点类型](#nodetypes)在以下部分定义。|
|faultDomain|容错域可让群集管理员定义可能因共享的物理依赖项而同时发生故障的物理节点。|
|upgradeDomain|升级域描述几乎在相同时间关闭以进行 Service Fabric 升级的节点集。你可以选择将哪些节点分配到哪些升级域，因为这不受任何物理要求的限制。| 


## 诊断配置
你可以通过 **diagnosticsFileShare** 部分将参数配置为允许进行诊断以及对节点和群集故障进行排查，如以下代码段所示。

    "diagnosticsFileShare": {
        "etlReadIntervalInMinutes": "5",
        "uploadIntervalInMinutes": "10",
        "dataDeletionAgeInDays": "7",
        "etwStoreConnectionString": "file:c:\ProgramData\SF\FileshareETW",
        "crashDumpConnectionString": "file:c:\ProgramData\SF\FileshareCrashDump",
        "perfCtrConnectionString": "file:c:\ProgramData\SF\FilesharePerfCtr"
    },

这些变量用于收集 ETW 跟踪日志、故障转储和性能计数器。有关 ETW 跟踪日志的详细信息，请阅读 [Tracelog](https://msdn.microsoft.com/zh-cn/library/windows/hardware/ff552994.aspx) 和 [ETW 跟踪](https://msdn.microsoft.com/zh-cn/library/ms751538.aspx)。可将 Service Fabric 节点和群集的[故障转储](https://blogs.technet.microsoft.com/askperf/2008/01/08/understanding-crash-dump-files/)定向到 **crashDumpConnectionString** 文件夹。可将群集的[性能计数器](https://msdn.microsoft.com/zh-cn/library/windows/desktop/aa373083.aspx)定向到计算机中的 **perfCtrConnectionString** 文件夹。


## 群集**属性**

ClusterConfig.JSON 中的“属性”部分用于配置群集，如下所示。

### **security** 
**security** 部分是确保 Service Fabric 独立群集安全性所必需的。以下代码段显示了该部分的一部分内容。

    "security": {
        "metadata": "This cluster is secured using X509 certificates.",
        "ClusterCredentialType": "X509",
        "ServerCredentialType": "X509",
		. . .
	}

**metadata** 用于描述安全群集，可以根据安装要求进行设置。**ClusterCredentialType** 和 **ServerCredentialType** 用于确定群集和节点需实现的安全类型。可以将这两项设置为 X509 以实现基于证书的安全性，或者设置为 Windows 以实现基于 Azure Active Directory 的安全性。**security** 部分的其余内容将取决于安全类型。若要了解如何填充 **security** 部分的其余内容，请阅读[独立群集中基于证书的安全性](/documentation/articles/service-fabric-windows-cluster-x509-security/)或[独立群集中的 Windows 安全性](/documentation/articles/service-fabric-windows-cluster-windows-security/)。

### **reliabilityLevel**
**reliabilityLevel** 定义可以在群集的主节点上运行的系统服务副本数。此项可提高这些服务以及此群集的可靠性。你可以将此变量设置为 Bronze、Silver、Gold 或 Platinum，这样即可分别运行这些服务的 3、5、7、9 个副本。请参阅以下示例。

	"reliabilityLevel": "Bronze",
	
请注意，由于一个主节点运行系统服务的一个副本，因此需要至少 3 个主节点来实现 Bronze 可靠性级别，至少 5 个主节点来实现 Silver 可靠性级别，至少 7 个主节点来实现 Gold 可靠性级别，至少 9 个主节点来实现 Platinum 可靠性级别。


<a id="nodetypes"></a>
### **nodeTypes**
**nodeTypes** 部分描述群集所拥有的节点的类型。一个群集必须指定至少一个节点类型，如以下代码段所示。

	"nodeTypes": [{
        "name": "NodeType0",
        "clientConnectionEndpointPort": "19000",
        "clusterConnectionEndpoint": "19001",
        "httpGatewayEndpointPort": "19080",
        "applicationPorts": {
			"startPort": "20001",
            "endPort": "20031"
        },
        "ephemeralPorts": {
            "startPort": "20032",
            "endPort": "20062"
        },
        "isPrimary": true
    }]

**name** 是此特定节点类型的友好名称。若要创建此节点类型的节点，需将此节点类型的友好名称分配给该节点的 **nodeTypeRef** 变量，如上面的[群集上的节点](#clusternodes)部分所述。对于每种节点类型，你可以定义各种终结点来连接到该群集。你可以为这些连接终结点选择任意端口号，只要不与此群集中的任何其他终结点冲突即可。在包含多个节点类型的群集中，将有一个主节点类型，其 **isPrimary** 设置为 true。其他节点的 **isPrimary** 将设置为 false。请阅读 [Service Fabric 群集容量规划注意事项](/documentation/articles/service-fabric-cluster-capacity/)，以详细了解与你的群集容量相关的 **nodeTypes** 和 **reliabilityLevel** 值，并了解主节点类型和非主节点类型的差异。


### **fabricSettings**
你可以通过此部分设置 Service Fabric 数据和日志的根目录。只能在初次创建群集的时候自定义这些设置。下面是此部分的代码段示例。

    "fabricSettings": [{
        "name": "Setup",
        "parameters": [{
            "name": "FabricDataRoot",
            "value": "C:\ProgramData\SF"
        }, {
            "name": "FabricLogRoot",
            "value": "C:\ProgramData\SF\Log"
    }]

请注意，如果只自定义数据根目录，则会将日志根目录放置在比数据根目录低一级的位置。


## 后续步骤

有了一个完整的根据独立群集设置配置的 ClusterConfig.JSON 文件以后，即可按照[在本地或云中创建 Azure Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)一文的说明部署群集，然后继续[使用 Service Fabric Explorer 可视化群集](/documentation/articles/service-fabric-visualizing-your-cluster/)。

<!---HONumber=Mooncake_0801_2016-->