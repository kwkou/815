<properties 
   pageTitle="为 SQL AlwaysOn 配置负载均衡器 | Azure"
   description="介绍如何将负载均衡器配置为与 SQL AlwaysOn 配合工作，以及如何利用 powershell 为 SQL 实现创建负载均衡器"
   services="load-balancer"
   documentationCenter="na"
   author="sdwheeler"
   manager="carmonm" />  
<tags 
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="10/24/2016"
   wacn.date="12/05/2016"
   ms.author="joaoma" />

# 为 SQL AlwaysOn 配置负载均衡器

SQL Server AlwaysOn 可用性组现在可与 ILB 配合运行。可用性组是 SQL Server 用于实现高可用性和灾难恢复的旗舰解决方案。无论配置中的副本数目是多少，可用性组侦听器都可让客户端应用程序无缝连接到主副本。

侦听器 (DNS) 名称会映射到经过负载均衡的 IP 地址，Azure 的负载均衡器只会将传入流量定向到副本集中的主服务器。

你可以使用 SQL Server AlwaysOn（侦听器）终结点的 ILB 支持。现在，你可以控制侦听器的可访问性，并可以从虚拟网络 (VNet) 中的特定子网内选择负载均衡的 IP 地址。

如果在侦听器上使用 ILB，只有以下项可以访问 SQL Server 终结点（例如 Server=tcp:ListenerName,1433;Database=DatabaseName）：

* 同一虚拟网络中的服务和 VM
* 来自连接的本地网络的服务和 VM
* 来自互连的 VNet 的服务和 VM

![ILB\_SQLAO\_NewPic](./media/load-balancer-configure-sqlao/sqlao1.png)  


图 1 - 使用面向 Internet 的负载均衡器配置的 SQL AlwaysOn

## 将内部负载均衡器添加到服务

1. 在以下示例中，我们会配置包含名为“Subnet-1”的子网的虚拟网络：

	    Add-AzureInternalLoadBalancer -InternalLoadBalancerName ILB\_SQL\_AO -SubnetName Subnet-1 -ServiceName SqlSvc

2. 为每个 VM 上的 ILB 添加负载均衡终结点

	    Get-AzureVM -ServiceName SqlSvc -Name sqlsvc1 | Add-AzureEndpoint -Name "LisEUep" -LBSetName "ILBSet1" -Protocol tcp -LocalPort 1433 -PublicPort 1433 -ProbePort 59999 -ProbeProtocol tcp -ProbeIntervalInSeconds 10 – DirectServerReturn $true -InternalLoadBalancerName ILB\_SQL\_AO | Update-AzureVM

 	    Get-AzureVM -ServiceName SqlSvc -Name sqlsvc2 | Add-AzureEndpoint -Name "LisEUep" -LBSetName "ILBSet1" -Protocol tcp -LocalPort 1433 -PublicPort 1433 -ProbePort 59999 -ProbeProtocol tcp -ProbeIntervalInSeconds 10 –DirectServerReturn $true -InternalLoadBalancerName ILB\_SQL\_AO | Update-AzureVM

    在上述示例中，有 2 个分别名为“sqlsvc1”和“sqlsvc2”的 VM 在云服务“Sqlsvc”中运行。使用 `DirectServerReturn` 开关创建 ILB 后，可以将经过负载均衡的终结点添加到 ILB，以便 SQL 为可用性组配置侦听器。

有关 SQL AlwaysOn 的详细信息，请参阅[在 Azure 中为 AlwaysOn 可用性组配置内部负载均衡器](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-int-listener/)。

## 另请参阅
[开始配置面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)

[Get started configuring an Internal load balancer（开始配置内部负载均衡器）](/documentation/articles/load-balancer-get-started-ilb-arm-ps/)

[配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载均衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_1128_2016-->