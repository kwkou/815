<properties
    pageTitle="将 ExpressRoute 关联的虚拟网络从经典部署模型迁移到 Resource Manager 部署模型：Azure：PowerShell | Azure"
    description="本页介绍如何在移动线路后将关联的虚拟网络迁移到 Resource Manager 部署模型。"
    documentationcenter="na"
    services="expressroute"
    author="ganesr"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="expressroute"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="04/11/2017"
    wacn.date="05/02/2017"
    ms.author="ganesr;cherylmc"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="3567f9d7926bb619e2e46ab0fde953bfdb306417"
    ms.lasthandoff="04/22/2017" />

# <a name="migrate-expressroute-associated-virtual-networks-from-classic-to-resource-manager"></a>将 ExpressRoute 关联的虚拟网络从经典部署模型迁移到 Resource Manager 部署模型

本文介绍如何在移动 ExpressRoute 线路后，将 Azure ExpressRoute 关联的虚拟网络从经典部署模型迁移到 Azure Resource Manager 部署模型。 


## <a name="before-you-begin"></a>开始之前
* 验证你是否有最新版本的 Azure PowerShell 模块。 有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。
* 在开始配置之前，请务必查看[先决条件](/documentation/articles/expressroute-prerequisites/)、[路由要求](/documentation/articles/expressroute-routing/)和[工作流](/documentation/articles/expressroute-workflows/)。
* 查看[将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型](/documentation/articles/expressroute-move/)中提供的信息。 请确保对限制和局限性有全面的了解。
* 验证线路在经典部署模型中可完全正常运行。
* 确保拥有一个在 Resource Manager 部署模型中创建的资源组。
* 查看以下资源迁移文档：

    * [平台支持的从经典部署模型到 Azure Resource Manager 的 IaaS 资源迁移](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)
    * [有关平台支持的从经典部署模型到 Azure Resource Manager 部署模型的迁移的技术深入探讨](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/)
    * [常见问题解答：平台支持的从经典部署模型到 Azure Resource Manager 部署模型的 IaaS 资源迁移](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)


## <a name="supported-and-unsupported-scenarios"></a>支持的和不支持的方案

* 可以在不停机的情况下，将 ExpressRoute 线路从经典环境移到 Resource Manager 环境。 可以在不停机的情况下，将 ExpressRoute 线路从经典环境转移到 Resource Manager 环境。 按照相关说明，[使用 PowerShell 将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型](/documentation/articles/expressroute-howto-move-arm/)。 这是将连接的资源转移到虚拟网络的先决条件。
* 可以在不停机的情况下，将同一订阅中的虚拟网络、网关，以及附加到 ExpressRoute 线路的虚拟网络中的关联部署迁移到 Resource Manager 环境。 可以按照后面描述的步骤来迁移各种资源，例如虚拟网络、网关以及部署在虚拟网络中的虚拟机。 必须确保虚拟网络配置正确，然后才能进行迁移。 
* 若要完成虚拟网络、网关以及处于虚拟网络中但与 ExpressRoute 线路不属同一订阅的关联部署的迁移，则需停机一段时间。 文档最后一部分介绍了在迁移资源时必须执行的步骤。

## <a name="move-an-expressroute-circuit-from-classic-to-resource-manager"></a>将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型
必须先将 ExpressRoute 线路从经典环境转移到 Resource Manager 环境，然后才能尝试迁移附加到 ExpressRoute 线路的资源。 若要完成此任务，请参阅以下文章：

* 查看[将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型](/documentation/articles/expressroute-move/)中提供的信息。
* [使用 Azure PowerShell 将线路从经典部署模型转移到 Resource Manager 部署模型](/documentation/articles/expressroute-howto-move-arm/)。
* 使用 Azure 服务管理门户。 可以按照工作流[创建新的 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-portal-resource-manager/)并选择导入选项。 

此操作不涉及停机。 只要迁移正在进行，就可以继续在本地和 Azure 之间传输数据。

## <a name="prepare-your-virtual-network-for-migration"></a>针对迁移准备虚拟网络
必须确保要迁移的虚拟网络的网络没有不必要的项目。 若要下载虚拟网络配置并根据需要对其进行更新，请运行以下 PowerShell cmdlet：

    Add-AzureAccount -Environment AzureChinaCloud
    Select-AzureSubscription -SubscriptionName <VNET Subscription>
    Get-AzureVNetConfig -ExportToFile C:\virtualnetworkconfig.xml

      
必须确保从要迁移的虚拟网络中删除对  &lt;ConnectionsToLocalNetwork&gt;  的所有引用。 以下代码片段演示了示例网络配置：

        <VirtualNetworkSite name="MyVNet" Location="East US">
            <AddressSpace>
                <AddressPrefix>10.0.0.0/8</AddressPrefix>
            </AddressSpace>
            <Subnets>
                <Subnet name="Subnet-1">
                    <AddressPrefix>10.0.0.0/11</AddressPrefix>
                </Subnet>
                <Subnet name="GatewaySubnet">
                    <AddressPrefix>10.32.0.0/28</AddressPrefix>
                </Subnet>
            </Subnets>
            <Gateway>
                <ConnectionsToLocalNetwork>
                </ConnectionsToLocalNetwork>
            </Gateway>
        </VirtualNetworkSite>

 
如果 &lt;ConnectionsToLocalNetwork&gt; 不为空，请删除其下的引用，然后重新提交网络配置。 可以运行以下 PowerShell cmdlet 来实现此目的：

    Set-AzureVNetConfig -ConfigurationPath c:\virtualnetworkconfig.xml

## <a name="migrate-virtual-networks-gateways-and-associated-deployments"></a>迁移虚拟网络、网关和关联的部署

进行迁移时，需要执行哪些步骤取决于资源是在同一订阅中，还是在不同的订阅中，或者二者都有。

### <a name="migrate-virtual-networks-gateways-and-associated-deployments-in-the-same-subscription-as-the-expressroute-circuit"></a>迁移与 ExpressRoute 线路属于同一订阅的虚拟网络、网关和关联的部署
此部分介绍了如何执行相关步骤，以便迁移与 ExpressRoute 线路属于同一订阅的虚拟网络、网关和关联的部署。 无需停机即可进行此迁移。 在整个迁移过程中，仍然可以使用所有资源。 正在进行迁移时，管理平面是锁定的。 

1. 确保已将 ExpressRoute 线路从经典环境转移到 Resource Manager 环境。
2. 确保已针对迁移进行了相应的虚拟网络准备。
3. 注册订阅，以便进行资源迁移。 若要针对资源迁移来注册订阅，请使用以下 PowerShell 代码片段：

        Select-AzureRmSubscription -SubscriptionName <Your Subscription Name>
        Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
        Get-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

4. 验证、准备和迁移。 若要转移虚拟网络，请使用以下 PowerShell 代码片段：

        Move-AzureVirtualNetwork -Prepare $vnetName  
        Move-AzureVirtualNetwork -Commit $vnetName

    也可通过运行以下 PowerShell cmdlet 来中止迁移：

        Move-AzureVirtualNetwork -Abort $vnetName

### <a name="migrate-virtual-networks-gateways-and-associated-deployments-in-a-different-subscription-from-that-of-the-expressroute-circuit"></a>迁移与 ExpressRoute 线路不属同一订阅的虚拟网络、网关和关联的部署

1. 确保已将 ExpressRoute 线路从经典环境转移到 Resource Manager 环境。
2. 确保已针对迁移进行了相应的虚拟网络准备。
3. 确保 ExpressRoute 线路可以同时在经典环境和 Resource Manager 环境中使用。 若要让线路能够同时在经典环境和 Resource Manager 环境中使用，请使用以下 PowerShell 脚本：

        Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)
        Select-AzureRmSubscription -SubscriptionName <My subscription>
        $circuit = Get-AzureRmExpressRouteCircuit -Name <CircuitName> -ResourceGroupName <ResourceGroup Name> 
        $circuit.AllowClassicOperations = $true
        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $circuit

4. 在 Resource Manager 环境中创建授权。 若要了解如何创建授权，请参阅[如何将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm/)。 若要创建授权，请使用以下 PowerShell 代码片段：

        circuit = Get-AzureRmExpressRouteCircuit -Name <CircuitName> -ResourceGroupName <ResourceGroup Name> 
        Add-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "AuthorizationForMigration"
        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $circuit
        $circuit = Get-AzureRmExpressRouteCircuit -Name MigrateCircuit -ResourceGroupName MigrateRGWest

        $id = $circuit.id 
        $auth1 = Get-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "AuthorizationForMigration"

        $key=$auth1.AuthorizationKey 

    请记下线路 ID 和授权密钥。 在迁移完成后，如果要将线路连接到虚拟网络，则需使用这些元素。
  
5. 删除与虚拟网络关联的专用线路链接。 若要在经典环境中删除线路链接，请使用以下 cmdlet：

        $skey = Get-AzureDedicatedCircuit | select ServiceKey
        Remove-AzureDedicatedCircuitLink -ServiceKey $skey -VNetName $vnetName

6. 注册订阅，以便进行资源迁移。 若要针对资源迁移来注册订阅，请使用以下 PowerShell 代码片段：

        Select-AzureRmSubscription -SubscriptionName <Your Subscription Name>
        Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
        Get-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

7. 验证、准备和迁移。 若要转移虚拟网络，请使用以下 PowerShell 代码片段：

        Move-AzureVirtualNetwork -Prepare $vnetName  
        Move-AzureVirtualNetwork -Commit $vnetName

    也可通过运行以下 PowerShell cmdlet 来中止迁移：

        Move-AzureVirtualNetwork -Abort $vnetName

8. 将虚拟网络连接回 ExpressRoute 线路。 以下 PowerShell 代码片段在特定订阅的上下文中运行，而虚拟网络在该订阅中创建。 不得在创建线路的订阅中运行此代码片段。 使用在步骤 4 中记下的线路 ID（充当 PeerID）和授权密钥。

        Select-AzureRMSubscription –SubscriptionName <customer subscription>  
        $gw = Get-AzureRmVirtualNetworkGateway -Name $vnetName-Default-Gateway -ResourceGroupName ($vnetName + "-Migrated")
        $vnet = Get-AzureRmVirtualNetwork -Name $vnetName -ResourceGroup  ($vnetName + "-Migrated")  

        New-AzureRmVirtualNetworkGatewayConnection -Name  ($vnetName + "-GwConn") -ResourceGroupName ($vnetName + "-Migrated")  -Location $vnet.Location -VirtualNetworkGateway1 $gw -PeerId $id -ConnectionType ExpressRoute -AuthorizationKey $key

## <a name="next-steps"></a>后续步骤
* [平台支持的从经典部署模型到 Azure Resource Manager 的 IaaS 资源迁移](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)
* [有关平台支持的从经典部署模型到 Azure Resource Manager 部署模型的迁移的技术深入探讨](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/)
* [常见问题解答：平台支持的从经典部署模型到 Azure Resource Manager 部署模型的 IaaS 资源迁移](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)
