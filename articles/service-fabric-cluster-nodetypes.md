
<!--Ibiza portal-->
<properties
   pageTitle="Service Fabric 节点类型和 VM 缩放集 | Azure"
   description="介绍 Service Fabric 节点类型如何与 VM 缩放集相关联，以及如何远程连接到 VM 缩放集实例或群集节点。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/02/2016"
   wacn.date="07/04/2016"/>


# Service Fabric 节点类型与虚拟机缩放集之间的关系

虚拟机缩放集是一种 Azure 计算资源，可用于将一组 VM 作为一个集进行部署和管理。在 Service Fabric 群集中定义的每个节点类型将设置为不同的 VM 缩放集。然后，每个节点类型可以独立扩展或缩减、打开不同的端口集，并可以有不同的容量指标。

以下屏幕截图显示了具有 FrontEnd 和 BackEnd 两个节点类型的群集。每个节点类型各有五个节点。

![显示具有两个节点类型的群集的屏幕截图][NodeTypes]

## 将 VM 缩放集实例映射到节点

如上所示，VM 缩放集实例从实例 0 开始逐渐递增。编号反映在名称中。例如，BackEnd\_0 是 BackEnd VM 缩放集的实例 0。此特定 VM 缩放集有五个实例，名称分别为 BackEnd\_0、BackEnd\_1、BackEnd\_2、BackEnd\_3、BackEnd\_4。

当你扩展 VM 缩放集时，将创建新的实例。新 VM 缩放集的名称通常是 VM 缩放集名称 + 下一个实例编号。在本示例中，即 BackEnd\_5。

<!--
## 将 VM 缩放集负载平衡器映射到每个节点类型/VM 缩放集

如果你已从门户或使用提供的示例 ARM 模板部署群集，则当你获取资源组下所有资源的列表时，将看到每个 VM 缩放集或节点类型的负载平衡器。

该名称类似于 **LB-&lt;NodeType name&gt;**。例如，以下屏幕截图中显示了 LB-sfcluster4doc-0：


![资源][Resources]


## 远程连接到 VM 缩放集实例或群集节点
在群集中定义的每个节点类型将设置为不同的 VM 缩放集。这意味着，节点类型可以独立扩展或缩减，且可由不同的 VM SKU 组成。不同于单实例 VM，VM 缩放集的实例没有自身的虚拟 IP 地址。因此你可能很难找到可用来远程连接到特定实例的 IP 地址和连接端口。

可以遵循以下步骤来查找 IP 地址和端口。

### 步骤 1：找出该节点类型的虚拟 IP 地址，然后找出 RDP 的入站 NAT 规则

若要获得此信息，必须获取 **Microsoft.Network/loadBalancers** 资源定义中定义的入站 NAT 规则值。

在门户中，导航到“负载平衡器”边栏选项卡并选择“设置”。

![LBBlade][LBBlade]


在“设置”中，单击“入站 NAT 规则”。随后可以获得可用来远程连接到第一个 VM 缩放集实例的 IP 地址和端口。以下屏幕截图中显示的是 **104.42.106.156** 和 **3389**。

![NATRules][NATRules]

### 步骤 2：找出可用来远程连接到特定 VM 缩放集实例/节点的端口

本文前面讨论了如何将 VM 缩放集实例映射到节点。我们将使用这种方法来找出确切的端口。

端口是以 VM 缩放集实例的递增顺序分配的。因此，在 FrontEnd 节点类型的示例中，五个实例的每个端口分别如下。现在你需要对 VM 缩放集实例执行相同的映射。

|**VM 缩放集实例**|**端口**|
|-----------------------|--------------------------|
|FrontEnd\_0|3389|
|FrontEnd\_1|3390|
|FrontEnd\_2|3391|
|FrontEnd\_3|3392|
|FrontEnd\_4|3393|
|FrontEnd\_5|3394|


### 步骤 3：远程连接到特定 VM 缩放集实例

在以下屏幕截图中，我使用了“远程桌面连接”来连接到 FrontEnd\_1：

![RDP][RDP]
-->
## 如何更改 RDP 端口范围值

### 群集部署之前

使用 ARM 模板设置群集时，可以在 **inboundNatPools** 中指定范围。

转到 **Microsoft.Network/loadBalancers** 的资源定义。在此处找到 **inboundNatPools** 的描述。替换 *frontendPortRangeStart* 和 *frontendPortRangeEnd* 值。

![InboundNatPools][InboundNatPools]


### 群集部署之后
这稍微要复杂一点，并且可能会导致 VM 设置被回收。你现在必须使用 Azure PowerShell 设置新值。请确保计算机上已安装 Azure PowerShell 1.0 或更高版本。如果尚未安装，我强烈建议你根据 [How to install and configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell）一文中所述的步骤安装。

登录到你的 Azure 帐户。如果此 PowerShell 命令由于某些原因而失败，你应该检查 Azure PowerShell 是否已正确安装。

```
Login-AzureRmAccount -EnvironmentName AzureChinaCloud
```

运行以下命令获取有关负载平衡器的详细信息，在值中，你将会找到 **inboundNatPools** 的描述：

```
Get-AzureRmResource -ResourceGroupName <RGname> -ResourceType Microsoft.Network/loadBalancers -ResourceName <load balancer name>
```

现在，将 *frontendPortRangeStart* 和 *frontendPortRangeEnd* 设置为所需的值。

```
$PropertiesObject = @{
	#Property = value;
}
Set-AzureRmResource -PropertyObject $PropertiesObject -ResourceGroupName <RG name> -ResourceType Microsoft.Network/loadBalancers -ResourceName <load Balancer name> -ApiVersion <use the API version that get returned> -Force
```


## 后续步骤

- [“随地部署”功能的概述及其与 Azure 托管群集的比较](/documentation/articles/service-fabric-deploy-anywhere/)
- [群集安全性](/documentation/articles/service-fabric-cluster-security/)
- [Service Fabric SDK 及其入门](/documentation/articles/service-fabric-get-started/)


<!--Image references-->
[NodeTypes]: ./media/service-fabric-cluster-nodetypes/NodeTypes.png
[Resources]: ./media/service-fabric-cluster-nodetypes/Resources.png
[InboundNatPools]: ./media/service-fabric-cluster-nodetypes/InboundNatPools.png
[LBBlade]: ./media/service-fabric-cluster-nodetypes/LBBlade.png
[NATRules]: ./media/service-fabric-cluster-nodetypes/NATRules.png
[RDP]: ./media/service-fabric-cluster-nodetypes/RDP.png

<!---HONumber=Mooncake_0523_2016-->