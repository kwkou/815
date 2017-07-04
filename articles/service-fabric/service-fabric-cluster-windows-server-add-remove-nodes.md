<properties
    pageTitle="向独立 Service Fabric 群集添加或删除节点 | Azure"
    description="了解如何向运行 Windows Server 的本地或任意云中物理计算机或虚拟机上的 Azure Service Fabric 群集添加节点。"
    services="service-fabric"
    documentationcenter=".net"
    author="rwike77"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="bc6b8fc0-d2af-42f8-a164-58538be38d02"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="2/02/2017"
    wacn.date="03/03/2017"
    ms.author="chackdan" />

# 向在 Windows Server 上运行的独立 Service Fabric 群集添加或删除节点

[在 Windows Server 计算机上创建独立 Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)之后，您的业务需求可能发生改变，因此可能需要向群集添加或删除多个节点。本文提供了实现此目标的详细步骤。

## 向群集添加节点

1. 按照[准备计算机以满足群集部署的先决条件](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)部分中所涉及的步骤，准备要添加到群集中的 VM/计算机。
2. 规划要向哪些容错域和升级域添加此 VM/计算机。
3. 通过远程桌面 (RDP) 方式进入需要向群集添加的 VM/计算机。
4. 向此 VM/计算机复制或[下载适用于 Windows Server 的 Service Fabric 独立包](http://go.microsoft.com/fwlink/?LinkId=730690)并解压缩该包。
5. 以管理员身份运行 Powershell，并导航到解压缩包所在的位置。
6. 运行 *AddNode.ps1* Powershell，使用参数来描述要添加的新节点。以下示例将名为 VM5、类型为 NodeType0 且 IP 地址为 182.17.34.52 的新节点添加到 UD1 和 fd:/dc1/r0 中。 *ExistingClusterConnectionEndPoint* 是现有群集中已有节点的连接终结点。对于此终结点，可以选择群集中 *任何* 节点的 IP 地址。


		.\AddNode.ps1 -NodeName VM5 -NodeType NodeType0 -NodeIPAddressorFQDN 182.17.34.52 -ExistingClientConnectionEndpoint 182.17.34.50:19000 -UpgradeDomain UD1 -FaultDomain fd:/dc1/r0 -AcceptEULA

运行 cmdlet [Get-servicefabricnode](https://docs.microsoft.com/powershell/servicefabric/vlatest/Get-ServiceFabricNode)，可检查是否添加了新节点。


## 从群集中删除节点
根据针对群集选择的可靠性级别，有时无法删除主节点类型的前 n (3/5/7/9) 个节点。另请注意，不支持在开发群集上运行 RemoveNode 命令。

1. 通过远程桌面 (RDP) 方式进入需要从群集中删除的 VM/计算机。
2. 复制或[下载适用于 Windows Server 的 Service Fabric 独立包](http://go.microsoft.com/fwlink/?LinkId=730690)并将该包解压缩到此 VM/计算机。
3. 以管理员身份运行 Powershell，并导航到解压缩包所在的位置。
4. 在 Powershell 中运行 *RemoveNode.ps1* 。以下示例从群集中删除当前节点。 *ExistingClientConnectionEndpoint* 是在群集中保留的任何节点的客户端连接终结点。选择群集中 *任何* **其他节点**的 IP 地址和终结点端口。这一**其他节点**反过来会更新已删除节点的群集配置。


		.\RemoveNode.ps1 -ExistingClientConnectionEndpoint 182.17.34.50:19000

> [AZURE.NOTE]
某些节点可能因系统服务依赖项的原因而不能删除。这些节点是主节点，可通过使用 `Get-ServiceFabricClusterManifest` 查询群集清单并查找使用 `IsSeedNode="true"` 标记的节点条目来进行识别。
> 
> 

即使在删除节点后，它也可能在查询和 SFX 中显示为停机，请注意，这是已知缺陷。我们会在即将发布的版本中解决此问题。


## 从群集中删除节点类型
删除节点类型时需要特别小心。删除节点类型之前，请仔细检查是否存在任何引用该节点类型的节点。


## 替换群集的主节点
应顺次执行主节点的替换，而不是成批删除，然后成批添加。


## 后续步骤
- [Windows 独立群集的配置设置](/documentation/articles/service-fabric-cluster-manifest/)
- [使用 X509 证书保护 Windows 上的独立群集](/documentation/articles/service-fabric-windows-cluster-x509-security/)
- [使用运行 Windows 的 Azure VM 创建独立 Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-with-windows-azure-vms/)

<!---HONumber=Mooncake_0227_2017-->
<!--update: add "从群集中删除节点类型", "替换群集的主节点" sections-->