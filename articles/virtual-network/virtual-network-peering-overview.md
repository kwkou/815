<properties
    pageTitle="Azure 虚拟网络对等互连 | Azure"
    description="了解 Azure 中的虚拟网络对等互连。"
    services="virtual-network"
    documentationcenter="na"
    author="NarayanAnnamalai"
    manager="jefco"
    editor="tysonn" />
<tags
    ms.assetid="eb0ba07d-5fee-4db0-b1cb-a569b7060d2a"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="10/17/2016"
    wacn.date="03/24/2017"
    ms.author="narayan" />  


# 虚拟网络对等互连
使用虚拟网络 (VNet) 对等互连可以通过 Azure 主干网络连接同一区域的两个 VNet。对等互连后，出于连接目的，两个 VNet 会显示为一个。这两个 VNet 仍作为单独资源管理，但这些对等 VNet 中的虚拟机 (VM) 可直接通过专用 IP 地址彼此通信。

对等 VNet 中 VM 之间的流量通过Azure 基础结构路由，与同一 VNet 中 VM 间流量的路由方式很像。使用 VNet 对等互连的部分好处有：

* 不同 VNet 中资源之间的连接延迟低且带宽高。
* 能够将诸如网络设备和 VPN 网关等资源用作对等 VNet 中的传输点。
* 能够将两个通过 Azure Resource Manager 部署模型创建的 VNet 对等互连，或者将一个通过 Resource Manager 部署模型创建的 VNet 对等互连到一个通过经典部署模型创建的 VNet。阅读[了解 Azure 部署模型](/documentation/articles/resource-manager-deployment-model/)一文，详细了解这两个 Azure 部署模型之间的差异。

VNet 对等互连的要求和关键方面：

* 对等 VNet 必须位于同一 Azure 区域。
* 对等 VNet 的 IP 地址空间不得重叠。
* VNet 对等互连存在于两个 VNet 之间，多个对等互连之间没有任何派生的可传递关系。例如，如果 VNetA 与 VNetB 对等互连，VNetB 与 VNetC 对等互连，但 VNetA *不* 与 VNetC 对等互连。
* 可以将存在于两个不同订阅中的 VNet 对等互连，只要两个订阅的特权用户授权予对等互连，并且订阅与同一个 Active Directory 租户关联即可。
* 如果两个 VNet 都是通过 Resource Manager 部署模型创建的，或者一个 VNet 是通过 Resource Manager 部署模型创建的，另一个 VNet 是通过经典部署模型创建的，则可以将 VNet 对等互连。但是，两个都是通过经典部署模型创建的 VNet 不能彼此对等互连。将通过不同部署模型创建的 VNet 对等互连时，两个 VNet 必须存在于*同一*订阅中。在**预览版**中，可以将通过不同部署模型创建且存在于*不同*订阅中的 VNet 对等互连。有关更多详细信息，请阅读[使用 Powershell 创建虚拟网络对等互连](/documentation/articles/virtual-networks-create-vnetpeering-arm-ps/)一文。
* 虽然在对等 VNet 中进行 VM 之间的通信没有其他带宽限制，但有一个最大网络带宽，具体取决于 VM 大小（仍适用）。若要详细了解不同 VM 大小的最大网络带宽，请阅读有关 [Windows](/documentation/articles/virtual-machines-windows-sizes/) 或 [Linux](/documentation/articles/virtual-machines-linux-sizes/) VM 大小的文章。

![基本 VNet 对等互连](./media/virtual-networks-peering-overview/figure01.png)  


## 连接
两个 VNet 对等互连以后，VNet 中的 VM 或云服务角色即可直接与其他连接到对等 VNet 的资源进行连接。这两个 VNet 具有完全 IP 级连接。

对等 VNet 中两个 VM 间的往返网络延迟与单个 VNet 中的往返网络延迟相同。网络吞吐量取决于可供 VM 使用的与其大小成比例的带宽。对等互连的带宽没有任何其他限制。

对等 VNet 中 VM 间的流量直接通过 Azure 后端基础结构路由，不通过网关。

连接到 VNet 的 VM 可以访问对等 VNet 中的内部负载均衡 (ILB) 终结点。可以根据需要将网络安全组 (NSG) 应用于 VNet（阻止对其他 VNet 的访问）或子网。

配置对等互连时，可以打开或关闭 VNet 之间的 NSG 规则。如果打开对等 VNet 之间的完全连接（默认选项），则可将 NSG 应用到特定的子网或 VM，以便阻止或拒绝特定的访问。阅读[网络安全组](/documentation/articles/virtual-networks-nsg/)一文，了解有关 NSG 的详细信息。

Azure 为 VM 提供的内部 DNS 名称解析在对等 VNet 中无效。VM 的内部 DNS 名称只能在本地 VNet 中解析。但是，可将连接到对等 VNet 的 VM 配置为 VNet 的 DNS 服务器。有关更多详细信息，请阅读[使用自己的 DNS 服务器进行名称解析](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/#name-resolution-using-your-own-dns-server)一文。

## 服务链
如本文后面的关系图所示，可将指向对等 VNet 中 VM 的用户定义路由 (UDR) 配置为“下一跃点”IP 地址。这样，便可以启用服务链，将一个 VNet 中的流量通过 UDR 引导到在对等 VNet 中运行的虚拟设备。

还可以有效构建中心辐射型环境，中心可在其中托管基础结构组件，如网络虚拟设备。然后，可以让所有分散 VNet 与之对等，以及与运行于中心 VNet 的设备的部分流量对等。简言之，VNet 对等互连使 UDR 上的下一跃点 IP 地址成为对等 VNet 中 VM 的 IP 地址。有关 UDR 的更多信息，请阅读[用户定义的路由](/documentation/articles/virtual-networks-udr-overview/)一文。

## 网关和本地连接
无论是否与另一个 VNet 对等，每个 VNet 仍可具有自己的网关，并使用它连接到本地网络。即使 VNet 对等，用户也可以使用网关配置 [VNet 到 VNet 连接](/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/)。

若已配置 VNet 互连的两个选项，则 VNet 之间的流量将通过对等配置（即通过 Azure 主干）流通。

VNet 对等时，用户还可以将对等 VNet 中的网关配置为本地网络的传输点。在这种情况下，使用远程网关的 VNet 不能有自己的网关。VNet 只能有一个网关。网关可能是本地网关或远程网关（对等 VNet 中），如下图所示：

![VNet 对等传输](./media/virtual-networks-peering-overview/figure02.png)  


通过不同部署模型创建的 VNet 之间的对等互连关系不支持网关传输。若要使用网关传输，对等互连关系中的两个 VNet 都必须通过 Resource Manager 创建。

正在共享单个 Azure ExpressRoute 连接的 VNet 对等时，它们之间的流量会通过对等关系（即通过 Azure 主干网）流通。仍可在各个 VNet 中使用本地网关连接到本地线路。或者，也可以使用共享网关，并为本地连接配置传输。

## 设置
VNet 对等互连是一项特权操作。它是 VirtualNetworks 命名空间下的独立功能。可授予用户特定权限来授权对等互连。具有虚拟网络读写访问权限的用户自动继承这些权限。

管理员或具有对等互连能力的特权用户可在另一个 VNet 上启动对等操作。如果另一端存在对等互连的匹配请求且也满足其他要求，就会建立对等互连。

请参阅本文[后续步骤](#next-steps)部分所列的文章，详细了解如何在两个 VNet 之间建立 VNet 对等互连。

## 限制
允许单个虚拟网络建立的对等互连数存在限制。有关详细信息，请参阅 [Azure 网络限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)。

## 定价
利用 VNet 对等互连的入口和出口流量会产生少许收费。有关详细信息，请参阅[定价页](/pricing/details/networking/)。

## <a name="next-steps"></a>后续步骤
了解如何通过以下方式创建 VNet 对等互连：

* [Azure 门户预览](/documentation/articles/virtual-networks-create-vnetpeering-arm-portal/)
* [Azure PowerShell](/documentation/articles/virtual-networks-create-vnetpeering-arm-ps/)
* [Azure Resource Manager 模板](/documentation/articles/virtual-networks-create-vnetpeering-arm-template-click/)

<!---HONumber=Mooncake_0320_2017-->