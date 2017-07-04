<properties
    pageTitle="创建 Azure 内部负载均衡器 - PowerShell 经典 | Azure"
    description="了解如何在经典部署模型中使用 PowerShell 创建内部负载均衡器"
    services="load-balancer"
    documentationcenter="na"
    author="kumudd"
    manager="timlt"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="3be93168-3787-45a5-a194-9124fe386493"
    ms.service="load-balancer"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/23/2017"
    wacn.date="03/03/2017"
    ms.author="kumud" />

# 开始使用 PowerShell 创建内部负载均衡器（经典）
> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/load-balancer-get-started-ilb-classic-ps/)
- [Azure CLI](/documentation/articles/load-balancer-get-started-ilb-classic-cli/)
- [云服务](/documentation/articles/load-balancer-get-started-ilb-classic-cloud/)

[AZURE.INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

> [AZURE.IMPORTANT]
Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型的情况。Azure 建议大多数新部署使用 Resource Manager 模型。了解如何[使用 Resource Manager 模型执行这些步骤](/documentation/articles/load-balancer-get-started-ilb-arm-ps/)。

[AZURE.INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## 为虚拟机创建内部负载均衡器集

若要创建内部负载均衡器集和要将其流量发送到的服务器，必须执行以下操作：

1. 创建内部负载均衡实例，该实例将是要在负载均衡集的服务器上进行负载均衡的传入流量的终结点。
2. 添加与虚拟机对应的终结点以接收传入流量。
3. 配置将发送要进行负载均衡的流量的服务器，以将其流量发送到内部负载均衡实例的虚拟 IP (VIP) 地址。

### 步骤 1：创建内部负载均衡实例

对于现有的云服务或在区域虚拟网络下部署的云服务，可以使用以下 Windows PowerShell 命令创建内部负载均衡实例：

    $svc="<Cloud Service Name>"
    $ilb="<Name of your ILB instance>"
    $subnet="<Name of the subnet within your virtual network>"
    $IP="<The IPv4 address to use on the subnet-optional>"

    Add-AzureInternalLoadBalancer -ServiceName $svc -InternalLoadBalancerName $ilb -SubnetName $subnet -StaticVNetIPAddress $IP

请注意，这次使用的 [Add-AzureEndpoint](https://msdn.microsoft.com/zh-cn/library/dn495300.aspx) Windows PowerShell cmdlet 使用的是 DefaultProbe 参数集。有关其他参数集的详细信息，请参阅 [Add-AzureEndpoint](https://msdn.microsoft.com/zh-cn/library/dn495300.aspx)。

### 步骤 2：向内部负载均衡实例添加终结点

下面是一个示例：

    $svc="mytestcloud"
    $vmname="DB1"
    $epname="TCP-1433-1433"
    $lbsetname="lbset"
    $prot="tcp"
    $locport=1433
    $pubport=1433
    $ilb="ilbset"
    Get-AzureVM -ServiceName $svc -Name $vmname | Add-AzureEndpoint -Name $epname -Lbset $lbsetname -Protocol $prot -LocalPort $locport -PublicPort $pubport -DefaultProbe -InternalLoadBalancerName $ilb | Update-AzureVM

### 步骤 3：配置服务器，将其流量发送到新的内部负载均衡终结点

你必须将其流量要进行负载均衡的服务器配置为使用内部负载均衡实例的新 IP 地址 (VIP)。此地址是内部负载均衡实例正在侦听的地址。在大多数情况下，你只需添加或修改内部负载均衡实例的 VIP 的 DNS 记录。

如果在创建内部负载均衡实例期间指定了 IP 地址，则你已有 VIP。否则，你可以通过以下命令查看 VIP：

    $svc="<Cloud Service Name>"
    Get-AzureService -ServiceName $svc | Get-AzureInternalLoadBalancer

若要使用这些命令，请填充值并删除 < 和 >。下面是一个示例：

    $svc="mytestcloud"
    Get-AzureService -ServiceName $svc | Get-AzureInternalLoadBalancer

通过显示 Get-AzureInternalLoadBalancer 命令，请记下 IP 地址，并对服务器或 DNS 记录进行必要的更改，以确保将流量发送到 VIP。

> [AZURE.NOTE]
Azure Platform 对各种管理方案使用一个公开可路由的静态 IPv4 地址。该 IP 地址为 168.63.129.16。此 IP 地址不应被任何防火墙阻止，因为这会导致意外行为。对于 Azure 内部负载均衡，此 IP 地址用于监视负载均衡器中的探测器，以确定负载均衡集中虚拟机的运行状况状态。如果网络安全组用于将流量限制到内部负载均衡集中的 Azure 虚拟机或应用于虚拟网络子网，请确保添加网络安全规则以允许来自 168.63.129.16 的流量。

## 内部负载均衡的示例

若要指导你逐步了解为两个示例配置创建中负载均衡集的端到端过程，请参阅以下各节。

### 面向 Internet 的多层应用程序

你想要为一组面向 Internet 的 Web 服务器提供负载均衡数据库服务。这两组服务器托管在单个 Azure 云服务中。TCP 端口 1433 的 Web 服务器流量必须分布到数据库层的两个虚拟机上。图 1 显示了该配置。

![适用于数据库层的内部负载均衡集](./media/load-balancer-internal-getstarted/IC736321.png)

该配置由以下项组成：

* 托管虚拟机的现有云服务名为 mytestcloud。
* 两个现有的数据库服务器分别名为 DB1、DB2。
* Web 层中的 Web 服务器使用专用 IP 地址连接到数据库层中的数据库服务器。另一个选择是对虚拟网络使用你自己的 DNS，并手动为内部负载均衡器集注册 A 记录。

以下命令将配置名为 **ILBset** 的新内部负载均衡实例，并向对应于两个数据库服务器的虚拟机添加终结点：

    $svc="mytestcloud"
    $ilb="ilbset"
    Add-AzureInternalLoadBalancer -ServiceName $svc -InternalLoadBalancerName $ilb
    $prot="tcp"
    $locport=1433
    $pubport=1433
    $epname="TCP-1433-1433"
    $lbsetname="lbset"
    $vmname="DB1"
    Get-AzureVM -ServiceName $svc -Name $vmname | Add-AzureEndpoint -Name $epname -LbSetName $lbsetname -Protocol $prot -LocalPort $locport -PublicPort $pubport -DefaultProbe -InternalLoadBalancerName $ilb | Update-AzureVM

    $epname="TCP-1433-1433-2"
    $vmname="DB2"
    Get-AzureVM -ServiceName $svc -Name $vmname | Add-AzureEndpoint -Name $epname -LbSetName $lbsetname -Protocol $prot -LocalPort $locport -PublicPort $pubport -DefaultProbe -InternalLoadBalancerName $ilb | Update-AzureVM

## 删除内部负载均衡配置

若要从内部负载均衡器实例删除作为终结点的虚拟机，请使用以下命令：

    $svc="<Cloud service name>"
    $vmname="<Name of the VM>"
    $epname="<Name of the endpoint>"
    Get-AzureVM -ServiceName $svc -Name $vmname | Remove-AzureEndpoint -Name $epname | Update-AzureVM

若要使用这些命令，请填充值并删除 < 和 >。

下面是一个示例：

    $svc="mytestcloud"
    $vmname="DB1"
    $epname="TCP-1433-1433"
    Get-AzureVM -ServiceName $svc -Name $vmname | Remove-AzureEndpoint -Name $epname | Update-AzureVM

若要从云服务中删除内部负载均衡器实例，请使用以下命令：

    $svc="<Cloud service name>"
    Remove-AzureInternalLoadBalancer -ServiceName $svc

若要使用这些命令，请填充值并删除 < 和 >。

下面是一个示例：

    $svc="mytestcloud"
    Remove-AzureInternalLoadBalancer -ServiceName $svc

## 有关内部负载均衡器 cmdlet 的其他信息

若要获取有关内部负载均衡 cmdlet 的其他信息，请在 Windows PowerShell 提示符处运行以下命令：

    Get-Help New-AzureInternalLoadBalancerConfig -full
    Get-Help Add-AzureInternalLoadBalancer -full
    Get-Help Get-AzureInternalLoadbalancer -full
    Get-Help Remove-AzureInternalLoadBalancer -full

## 后续步骤

[使用源 IP 关联配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载均衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: update meta properties; wording update -->