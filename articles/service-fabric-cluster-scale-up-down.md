<properties
   pageTitle="扩展或缩减 Service Fabric 群集 | Azure"
   description="通过为每个节点类型/VM 缩放集设置自动缩放规则来增加或减少 Service Fabric 群集以满足需求。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/04/2016"
   wacn.date="07/04/2016"/>


# 使用自动缩放规则增加或减少 Service Fabric 群集

虚拟机缩放集是一种 Azure 计算资源，可用于将一组 VM 作为一个集进行部署和管理。在 Service Fabric 群集中定义的每个节点类型将设置为不同的 VM 缩放集。然后，每个节点类型可以独立扩展或缩减、打开不同的端口集，并可以有不同的容量指标。可在 [Service Fabric nodetypes](/documentation/articles/service-fabric-cluster-nodetypes/) 文档中了解有关详细信息。由于群集中的 Service Fabric 节点类型由后端的 VM 缩放集构成，因此需要为每个节点类型/VM 缩放集设置自动缩放规则。

>[AZURE.NOTE] 你的订阅必须有足够的核心用于添加构成此群集的新 VM。当前没有模型验证，因此如果达到任何配额限制，则你会遇到部署时间故障。

## 选择要缩放的节点类型/VM 缩放集

当前无法使用门户为 VM 缩放集指定自动缩放规则，因此我们来使用 Azure PowerShell (1.0+) 列出节点类型，然后向它们添加自动缩放规则。

若要获取构成群集的 VM 缩放集的列表，请运行以下 cmdlet：

```powershell
Get-AzureRmResource -ResourceGroupName <RGname> -ResourceType Microsoft.Network/VirtualMachineScaleSets

Get-AzureRmVmss -ResourceGroupName <RGname> -VMScaleSetName <VM Scale Set name>
```

## 为节点类型/VM 缩放集设置自动缩放规则

如果群集具有多个节点类型，则需要为要缩放（增加或减少）的每个节点类型/VM 缩放集执行此操作。在设置自动缩放之前请考虑必须具有的节点数。对于主节点类型所必须具有的最小节点数受所选择的可靠性级别影响。了解有关[可靠性级别](/documentation/articles/service-fabric-cluster-capacity/)的详细信息。

>[AZURE.NOTE]  将主节点类型减少到小于最小数量会使群集不稳定或使它关闭。这可能会对应用程序和系统服务造成数据丢失。

当前，自动缩放功能不受应用程序可能向 Service Fabric 报告的负荷所影响。因此当前你获得的自动缩放纯粹受每个 VM 缩放集实例发出的性能计数器所影响。


>[AZURE.NOTE] 在减少方案中，除非节点类型具有金级或银级持续性级别，否则需要使用相应节点名称来调用 [Remove-ServiceFabricNodeState cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt125993.aspx)。

## 可能会在 Service Fabric Explorer 中观察到的行为

增加群集时，Service Fabric Explorer 会反映属于群集一部分的节点（VM 缩放集实例）数。但是在减少群集时，你仍会看到已删除的节点/VM 实例显示为不正常状态，除非使用相应节点名称来调用 [Remove-ServiceFabricNodeState cmd](https://msdn.microsoft.com/zh-cn/library/mt125993.aspx)。

下面是此行为的说明。

Service Fabric Explorer 中列出的节点是 Service Fabric 系统服务（特别是 FM）在群集具有的节点数方面所了解的信息的反映。减少 VM 缩放集时，VM 已删除，但是 FM 系统服务仍然认为节点（映射到已删除的 VM）会恢复。因此 Service Fabric Explorer 会继续显示该节点（尽管运行状况状态可能是错误或未知）。

若要确保在删除 VM 时删除节点，有两个选项：

1) 为群集中的节点类型选择金级或银级（即将推出）持续性级别，这会为你提供基础结构集成。这随后会在你进行减少时自动从我们的系统服务 (FM) 状态中删除节点。请参阅[有关持续性级别的详细信息](/documentation/articles/service-fabric-cluster-capacity/)

2) 减少 VM 实例之后，需要调用 [Remove-ServiceFabricNodeState cmdlet](https://msdn.microsoft.com/zh-cn/library/mt125993.aspx)。

>[AZURE.NOTE] Service Fabric 群集需要有一定数量的节点可随时启动，以保持可用性和状态 - 称为“维持仲裁”。因此，除非你已事先执行[状态的完整备份](/documentation/articles/service-fabric-reliable-services-backup-restore/)，否则关闭群集中的所有计算机通常是不安全的做法。

## 后续步骤
参阅以下文章以另外了解如何规划群集容量、升级群集以及对服务进行分区：

- [规划群集容量](/documentation/articles/service-fabric-cluster-capacity/)
- [群集升级](/documentation/articles/service-fabric-cluster-upgrade/)
- [为有状态服务分区以最大程度地实现缩放](/documentation/articles/service-fabric-concepts-partitioning/)

<!--Image references-->
[BrowseServiceFabricClusterResource]: ./media/service-fabric-cluster-scale-up-down/BrowseServiceFabricClusterResource.png
[ClusterResources]: ./media/service-fabric-cluster-scale-up-down/ClusterResources.png

<!---HONumber=Mooncake_0523_2016-->