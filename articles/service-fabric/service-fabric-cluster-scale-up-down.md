<properties
   pageTitle="扩展或缩减 Service Fabric 群集 | Azure"
   description="通过为每个节点类型/VM 规模集设置自动缩放规则，根据需要扩展或缩减 Service Fabric 群集。在 Service Fabric 群集中添加或删除节点"
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
   ms.date="09/09/2016"
   wacn.date="10/24/2016"
   ms.author="chackdan"/>  



# 使用自动缩放规则扩展可缩减 Service Fabric 群集

虚拟机规模集是一种 Azure 计算资源，可用于将一组 VM 作为一个集进行部署和管理。在 Service Fabric 群集中定义的每个节点类型将设置为不同的 VM 规模集。然后，每个节点类型可以独立扩展或缩减、打开不同的端口集，并可以有不同的容量指标。可在 [Service Fabric nodetypes](/documentation/articles/service-fabric-cluster-nodetypes/) 文档中了解有关详细信息。由于群集中的 Service Fabric 节点类型由后端的 VM 规模集构成，因此需要为每个节点类型/VM 规模集设置自动缩放规则。

>[AZURE.NOTE] 订阅必须有足够的核心用于添加构成此群集的新 VM。当前没有模型验证，因此如果达到任何配额限制，会遇到部署时故障。

## 选择要缩放的节点类型/VM 规模集

当前无法使用门户为 VM 规模集指定自动缩放规则，因此我们来使用 Azure PowerShell (1.0+) 列出节点类型，然后向它们添加自动缩放规则。

若要获取构成群集的 VM 规模集的列表，请运行以下 cmdlet：


	Get-AzureRmResource -ResourceGroupName <RGname> -ResourceType Microsoft.Compute/VirtualMachineScaleSets
	
	Get-AzureRmVmss -ResourceGroupName <RGname> -VMScaleSetName <VM Scale Set name>


## 为节点类型/VM 规模集设置自动缩放规则

如果群集具有多个节点类型，请为要缩放（扩展或缩减）的每个节点类型/VM 规模集重复此操作。在设置自动缩放之前请考虑必须具有的节点数。对于主节点类型所必须具有的最小节点数受所选择的可靠性级别影响。了解有关[可靠性级别](/documentation/articles/service-fabric-cluster-capacity/)的详细信息。

>[AZURE.NOTE]  将主节点类型减少到小于最小数量会使群集不稳定或使它关闭。这可能会对应用程序和系统服务造成数据丢失。

当前，自动缩放功能不受应用程序可能向 Service Fabric 报告的负荷所影响。因此当前你获得的自动缩放纯粹受每个 VM 规模集实例发出的性能计数器所影响。

按照以下说明[为每个 VM 规模集设置自动缩放](/documentation/articles/virtual-machine-scale-sets-autoscale-overview/)。

>[AZURE.NOTE] 在缩减方案中，除非节点类型具有金级或银级持续性级别，否则需要使用相应节点名称来调用 [Remove-ServiceFabricNodeState cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt125993.aspx)。

## 手动将 VM 添加节点类型/VM 规模集

根据[快速入门模板库](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-scale-existing)中的示例/说明更改每个 Nodetype 的 VM 数目。

>[AZURE.NOTE] 添加 VM 需要时间，请不要期待马上就有结果。因此请妥善规划增加容量的时间，在副本/服务实例可以使用 VM 容量之前有超过 10 分钟的时间让一切就绪。

## 手动从主节点类型/VM 规模集中删除 VM

>[AZURE.NOTE] Service Fabric 系统服务在群集中以主节点类型运行。因此请不要关闭该节点类型的实例，或者将该节点类型的实例数目缩减到少于可靠性层所需的数目。在[此处](/documentation/articles/service-fabric-cluster-capacity/)了解有关可靠性层的详细信息。

需以一次一个 VM 实例的形式执行以下步骤。这可让系统服务（以及有状态服务）在要删除的 VM 实例上正常关闭，并在其他节点上创建新副本。

1. 运行 [Disable-ServiceFabricNode](https://msdn.microsoft.com/zh-cn/library/mt125852.aspx) 加上“RemoveNode”可禁用要删除的节点（该节点类型的最高实例）。

2. 运行 [Get-ServiceFabricNode](https://msdn.microsoft.com/zh-cn/library/mt125856.aspx) 可确保节点确实转换为禁用。如果没有，请等到节点已禁用。此步骤不能一蹴而就。

2. 根据[快速入门模板库](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-scale-existing)中的示例/说明逐个更改 Nodetype 的 VM 数目。删除的实例是最高 VM 实例。

3. 根据需要重复步骤 1 到 3，但切勿将主节点类型的实例数目缩减到少于可靠性层所需的数目。在[此处](/documentation/articles/service-fabric-cluster-capacity/)了解有关可靠性层的详细信息。

## 手动从非主节点类型/VM 规模集中删除 VM

>[AZURE.NOTE] 对于有状态服务，需要一些始终启动的节点来保持可用性，以及保持服务的状态。至少需要与分区/服务的目标副本集计数相等的节点数目。

必须以一次一个 VM 实例的形式执行以下步骤。这可让系统服务（以及有状态服务）在要删除的 VM 实例上正常关闭，并在其他位置创建新副本。

1. 运行 [Disable-ServiceFabricNode](https://msdn.microsoft.com/zh-cn/library/mt125852.aspx) 加上“RemoveNode”可禁用要删除的节点（该节点类型的最高实例）。

2. 运行 [Get-ServiceFabricNode](https://msdn.microsoft.com/zh-cn/library/mt125856.aspx) 可确保节点确实转换为禁用。如果没有，请等到节点禁用。此步骤不能一蹴而就。

2. 根据[快速入门模板库](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-scale-existing)中的示例/说明逐个更改 Nodetype 的 VM 数目。现在，删除最高 VM 实例。

3. 根据需要重复步骤 1 到 3，但切勿将主节点类型的实例数目缩减到少于可靠性层所需的数目。在[此处](/documentation/articles/service-fabric-cluster-capacity/)了解有关可靠性层的详细信息。

## 可能会在 Service Fabric Explorer 中观察到的行为

增加群集时，Service Fabric Explorer 会反映属于群集一部分的节点（VM 规模集实例）数。但是在缩减群集时，会看到已删除的节点/VM 实例显示为不正常状态，除非使用相应节点名称来调用 [Remove-ServiceFabricNodeState cmd](https://msdn.microsoft.com/zh-cn/library/mt125993.aspx)。

下面是此行为的说明。

Service Fabric Explorer 中列出的节点是 Service Fabric 系统服务（特别是 FM）在群集具有的节点数方面所了解的信息的反映。减少 VM 规模集时，VM 已删除，但是 FM 系统服务仍然认为节点（映射到已删除的 VM）会恢复。因此 Service Fabric Explorer 会继续显示该节点（尽管运行状况状态可能是错误或未知）。

若要确保在删除 VM 时删除节点，有两个选项：

1) 为群集中的节点类型选择金级或银级（即将推出）持续性级别，这会提供基础结构集成。这随后会在你进行减少时自动从我们的系统服务 (FM) 状态中删除节点。在[此处](/documentation/articles/service-fabric-cluster-capacity/)了解有关持续性级别的详细信息

2) 减少 VM 实例之后，需要调用 [Remove-ServiceFabricNodeState cmdlet](https://msdn.microsoft.com/zh-cn/library/mt125993.aspx)。

>[AZURE.NOTE] Service Fabric 群集需要有一定数量的节点可随时启动，以保持可用性和状态 - 称为“维持仲裁”。 因此，除非已事先执行[状态的完整备份](/documentation/articles/service-fabric-reliable-services-backup-restore/)，否则关闭群集中的所有计算机通常是不安全的做法。

## 后续步骤
参阅以下文章以另外了解如何规划群集容量、升级群集以及对服务进行分区：

- [规划群集容量](/documentation/articles/service-fabric-cluster-capacity/)
- [群集升级](/documentation/articles/service-fabric-cluster-upgrade/)
- [为有状态服务分区以最大程度地实现缩放](/documentation/articles/service-fabric-concepts-partitioning/)

<!--Image references-->

[BrowseServiceFabricClusterResource]: ./media/service-fabric-cluster-scale-up-down/BrowseServiceFabricClusterResource.png
[ClusterResources]: ./media/service-fabric-cluster-scale-up-down/ClusterResources.png

<!---HONumber=Mooncake_1017_2016-->