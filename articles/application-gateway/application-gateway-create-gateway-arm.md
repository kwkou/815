<properties
    pageTitle="创建和管理 Azure 应用程序网关 - PowerShell | Azure"
    description="本页提供有关使用 Azure Resource Manager 创建、配置、启动和删除 Azure 应用程序网关的说明"
    documentationcenter="na"
    services="application-gateway"
    author="georgewallace"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="866e9b5f-0222-4b6a-a95f-77bc3d31d17b"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="04/04/2017"
    wacn.date="05/22/2017"
    ms.author="gwallace"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="46533f3ab4305a8b319cd1d0f5640682ea0bd729"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="create-start-or-delete-an-application-gateway-by-using-azure-resource-manager"></a>使用 Azure Resource Manager 创建、启动或删除应用程序网关
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/application-gateway-create-gateway-portal/)
- [Azure Resource Manager PowerShell](/documentation/articles/application-gateway-create-gateway-arm/)
- [Azure 经典 PowerShell](/documentation/articles/application-gateway-create-gateway/)
- [Azure Resource Manager 模板](/documentation/articles/application-gateway-create-gateway-arm-template/)
- [Azure CLI](/documentation/articles/application-gateway-create-gateway-cli/)

Azure 应用程序网关是第 7 层负载均衡器。 它在不同服务器之间提供故障转移和性能路由 HTTP 请求，而不管它们是在云中还是本地。
应用程序网关提供许多应用程序传送控制器 (ADC) 功能，包括 HTTP 负载均衡、基于 Cookie 的会话相关性、安全套接字层 (SSL) 卸载、自定义运行状况探测、多站点支持，以及许多其他功能。

若要查找支持的功能的完整列表，请参阅[应用程序网关概述](/documentation/articles/application-gateway-introduction/)

本文将指导你完成创建、配置、启动和删除应用程序网关的步骤。

> [AZURE.IMPORTANT]
> 在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：Resource Manager 部署模型和经典部署模型。 在使用任何 Azure 资源之前，请确保了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。 可以通过单击本文顶部的选项卡来查看不同工具的文档。 本文档介绍如何使用 Azure Resource Manager 创建应用程序网关。 若要使用经典版本，请访问[使用 PowerShell 创建应用程序网关经典部署](/documentation/articles/application-gateway-create-gateway/)。

## <a name="before-you-begin"></a>开始之前

1. 使用 Web 平台安装程序安装最新版本的 Azure PowerShell cmdlet。 可以从[下载页](/downloads/)的“Windows PowerShell”部分下载并安装最新版本。
1. 如果有现有的虚拟网络，请选择现有的一个空子网，或者在现有虚拟网络中创建一个子网，专门供应用程序网关使用。 应用程序网关部署到的虚拟网络必须与要部署在应用程序网关后面的资源相同。
1. 必须存在配置为使用应用程序网关的服务器，或者必须在虚拟网络中为其创建终结点，或者必须为其分配公共 IP/VIP。

## <a name="what-is-required-to-create-an-application-gateway"></a>创建应用程序网关需要什么？

* **后端服务器池：**后端服务器的 IP 地址、FQDN 或 NIC 的列表。 如果使用了 IP 地址，则这些地址应该属于虚拟网络子网，或者应该是公共 IP/VIP。
* **后端服务器池设置：** 每个池都有一些设置，例如端口、协议和基于 Cookie 的关联性。 这些设置绑定到池，并会应用到池中的所有服务器。
* **前端端口：** 此端口是应用程序网关上打开的公共端口。 流量将抵达此端口，然后重定向到后端服务器之一。
* **侦听器：**侦听器具有前端端口、协议（Http 或 Https，这些值区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。
* **规则：** 规则将会绑定侦听器和后端服务器池，并定义当流量抵达特定侦听器时应定向到的后端服务器池。

## <a name="create-an-application-gateway"></a>创建应用程序网关

使用 Azure 经典部署和 Azure Resource Manager 部署的差别在于创建应用程序网关的顺序和需要配置的项。

使用 Resource Manager 时，组成应用程序网关的所有项都将分开配置，然后放在一起创建应用程序网关资源。

以下是创建应用程序网关所需执行的步骤。

## <a name="create-a-resource-group-for-resource-manager"></a>创建 Resource Manager 的资源组

确保使用最新版本的 Azure PowerShell。 [将 Windows PowerShell 与 Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)中提供了详细信息。

### <a name="step-1"></a>步骤 1

登录 Azure

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

系统会提示使用凭据进行身份验证。

### <a name="step-2"></a>步骤 2

检查该帐户的订阅。

    Get-AzureRmSubscription

### <a name="step-3"></a>步骤 3

选择要使用的 Azure 订阅。

    Select-AzureRmSubscription -Subscriptionid "GUID of subscription"

### <a name="step-4"></a>步骤 4

创建资源组（如果要使用现有的资源组，请跳过此步骤）。

    New-AzureRmResourceGroup -Name appgw-rg -Location "China North"

Azure Resource Manager 要求所有资源组指定一个位置。 此位置将用作该资源组中的资源的默认位置。 请确保用于创建应用程序网关的所有命令都使用相同的资源组。

在上面的示例中，我们在“中国北部”这个位置创建了名为“appgw-RG”的资源组。

> [AZURE.NOTE]
> 若需为应用程序网关配置自定义探测，请访问：[使用 PowerShell 创建带自定义探测的应用程序网关](/documentation/articles/application-gateway-create-probe-ps/)。 有关详细信息，请查看[自定义探测和运行状况监视](/documentation/articles/application-gateway-probe-overview/)。

## <a name="create-a-virtual-network-and-a-subnet-for-the-application-gateway"></a>为应用程序网关创建虚拟网络和子网

以下示例演示如何使用 Resource Manager 创建虚拟网络。 该示例创建适用于应用程序网关的 VNET。 应用程序网关需要其自己的子网，因此，为应用程序网关创建的子网小于 VNET 地址空间。 由于使用较小的子网，因此可以在同一 VNET 中配置其他资源（包括但不限于 Web 服务器）。

### <a name="step-1"></a>步骤 1

将地址范围 10.0.0.0/24 分配给用于创建虚拟网络的子网变量。 此步骤为应用程序网关创建子网配置对象，该对象将用在下一示例中。

    $subnet = New-AzureRmVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24

### <a name="step-2"></a>步骤 2

使用前缀 10.0.0.0/16 和子网 10.0.0.0/24，在中国北部区域的 **appgw-rg** 资源组中创建名为 **appgwvnet** 的虚拟网络。 此步骤完成 VNET 的配置，该 VNET 有一个可供应用程序网关驻留的子网。

    $vnet = New-AzureRmVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-rg -Location "China North" -AddressPrefix 10.0.0.0/16 -Subnet $subnet

### <a name="step-3"></a>步骤 3

为后续步骤分配子网变量，该变量将在以后的步骤中传递给 `New-AzureRMApplicationGateway` cmdlet。

    $subnet=$vnet.Subnets[0]

## <a name="create-a-public-ip-address-for-the-front-end-configuration"></a>创建前端配置的公共 IP 地址

在中国北部区域的 **appgw-rg** 资源组中创建公共 IP 资源 **publicIP01**。 应用程序网关可以使用公共 IP 地址和/或内部 IP 地址来接收负载均衡请求。  此示例仅使用公共 IP 地址。 在以下示例中，没有为创建公共 IP 地址而配置任何 DNS 名称。  应用程序网关不支持在公共 IP 地址上使用自定义 DNS 名称。  如果公共终结点需要自定义名称，则应创建一个 CNAME 记录，使之指向针对公共 IP 地址自动生成的 DNS 名称。

    $publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-rg -name publicIP01 -location "China North" -AllocationMethod Dynamic

> [AZURE.NOTE]
> 服务启动时，会将一个 IP 地址分配到应用程序网关。

## <a name="create-the-application-gateway-configuration-objects"></a>创建应用程序网关配置对象

在创建应用程序网关之前，必须设置所有配置项目。 以下步骤将创建应用程序网关资源所需的配置项目。

### <a name="step-1"></a>步骤 1

创建名为“gatewayIP01” 的应用程序网关 IP 配置。 当应用程序网关启动时，它会从配置的子网获取 IP 地址，再将网络流量路由到后端 IP 池中的 IP 地址。 请记住，每个实例需要一个 IP 地址。

    $gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet

### <a name="step-2"></a>步骤 2

配置名为 **pool01** 的后端 IP 地址池，其 IP 地址对应于 **pool1**。 这些 IP 地址是托管 Web 应用程序的资源的 IP 地址，而这些 Web 应用程序受应用程序网关保护。 这些后端池成员都经过探测（不管是基本探测还是自定义探测）的验证，其运行状况正常。  然后，当请求进入应用程序网关时，系统就会将流量路由到这些成员。 可以通过应用程序网关中的多个规则来使用后端池，这意味着，一个后端池可以用于驻留在同一主机上的多个 Web 应用程序。

    $pool = New-AzureRmApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 134.170.185.46, 134.170.188.221, 134.170.185.50

在本示例中，会有两个后端池根据 URL 路径路由网络流量。 一个池接收来自 URL 路径“/video”的流量，另一个池接收来自路径“/image”的流量。 替换上述 IP 地址，添加自己的应用程序 IP 地址终结点。

### <a name="step-3"></a>步骤 3

配置应用程序网关设置 **poolsetting01**，在后端池中实现网络流量的负载均衡。 每个后端池可有自身的后端池设置。  可以通过规则使用后端 HTTP 设置，将流量路由到正确的后端池成员。 将流量发送到后端池成员时，所用的协议和端口由后端 HTTP 设置决定。 基于 Cookie 的会话也由后端 HTTP 设置决定。  在启用的情况下，基于 Cookie 的会话相关性会将流量发送到与每个数据包的前述请求相同的后端。

    $poolSetting01 = New-AzureRmApplicationGatewayBackendHttpSettings -Name "besetting01" -Port 80 -Protocol Http -CookieBasedAffinity Disabled -RequestTimeout 120

### <a name="step-4"></a>步骤 4

配置应用程序网关的前端端口。 侦听器使用前端端口配置对象来定义应用程序网关所侦听的具体端口，以便确定侦听器上的流量。

    $fp = New-AzureRmApplicationGatewayFrontendPort -Name frontendport01  -Port 80

### <a name="step-5"></a>步骤 5

使用公共 IP 终结点配置前端 IP。 侦听器使用前端 IP 配置对象将外向 IP 地址与侦听器相关联。

    $fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name fipconfig01 -PublicIPAddress $publicip

### <a name="step-6"></a>步骤 6

配置侦听器。 此步骤针对用于接收传入网络流量的公共 IP 地址和连接端口配置侦听器。 以下示例使用以前配置的前端 IP 配置、前端端口配置以及协议（http 或 https）对侦听器进行配置。 在此示例中，侦听器侦听端口 80 上的 HTTP 流量，该端口位于此前创建的公共 IP 地址上。

    $listener = New-AzureRmApplicationGatewayHttpListener -Name listener01 -Protocol Http -FrontendIPConfiguration $fipconfig -FrontendPort $fp

### <a name="step-7"></a>步骤 7

创建名为 **rule01** 的负载均衡器路由规则，以便配置负载均衡器的行为。 在前述步骤中创建的后端池设置、侦听器和后端池构成了该规则。 根据定义的条件，流量会路由到相应的后端。

    $rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting01 -HttpListener $listener -BackendAddressPool $pool

### <a name="step-8"></a>步骤 8

配置实例数目和应用程序网关的大小。

    $sku = New-AzureRmApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2

> [AZURE.NOTE]
> **InstanceCount** 的默认值为 2，最大值为 10。 **GatewaySize** 的默认值为 Medium。 可以在 **Standard_Small**、**Standard_Medium** 和 **Standard_Large** 之间进行选择。

## <a name="create-an-application-gateway-by-using-new-azurermapplicationgateway"></a>使用 New-AzureRmApplicationGateway 创建应用程序网关

创建包含前述步骤中所有配置项的应用程序网关。 在此示例中，应用程序网关名为 **appgwtest**。

    $appgw = New-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Location "China North" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku

从附加到应用程序网关的公共 IP 资源中检索应用程序网关的 DNS 和 VIP 详细信息。

    Get-AzureRmPublicIpAddress -Name publicIP01 -ResourceGroupName appgw-rg  

## <a name="delete-an-application-gateway"></a>删除应用程序网关

若要删除应用程序网关，请执行以下步骤：

### <a name="step-1"></a>步骤 1

获取应用程序网关对象，并将其关联到变量 `$getgw`。

    $getgw = Get-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg

### <a name="step-2"></a>步骤 2

使用 `Stop-AzureRmApplicationGateway` 停止应用程序网关。

    Stop-AzureRmApplicationGateway -ApplicationGateway $getgw

应用程序网关进入停止状态后，请使用 `Remove-AzureRmApplicationGateway` cmdlet 删除该服务。

    Remove-AzureRmApplicationGateway -Name $appgwtest -ResourceGroupName appgw-rg -Force

> [AZURE.NOTE]
> 可以使用 **-force** 开关来禁止显示该删除的确认消息。

若要验证是否已删除服务，可以使用 `Get-AzureRmApplicationGateway` cmdlet。 此步骤不是必需的。

    Get-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg

## <a name="get-application-gateway-dns-name"></a>获取应用程序网关 DNS 名称

创建网关后，下一步是配置用于通信的前端。 使用公共 IP 时，应用程序网关需要动态分配的 DNS 名称，这会造成不方便。 若要确保最终用户能够访问应用程序网关，可以使用指向应用程序网关的公共终结点的 CNAME 记录。 若要查找动态创建的 DNS 名称，请使用附加到应用程序网关的 PublicIPAddress 元素检索应用程序网关及其关联的 IP/DNS 名称的详细信息。 应使用应用程序网关的 DNS 名称来创建 CNAME 记录，使两个 Web 应用程序都指向此 DNS 名称。 不建议使用 A 记录，因为重新启动应用程序网关后 VIP 可能会变化。

    Get-AzureRmPublicIpAddress -ResourceGroupName appgw-RG -Name publicIP01

<br/>

    Name                     : publicIP01
    ResourceGroupName        : appgw-RG
    Location                 : chinanorth
    Id                       : /subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/publicIPAddresses/publicIP01
    Etag                     : W/"00000d5b-54ed-4907-bae8-99bd5766d0e5"
    ResourceGuid             : 00000000-0000-0000-0000-000000000000
    ProvisioningState        : Succeeded
    Tags                     : 
    PublicIpAllocationMethod : Dynamic
    IpAddress                : xx.xx.xxx.xx
    PublicIpAddressVersion   : IPv4
    IdleTimeoutInMinutes     : 4
    IpConfiguration          : {
                                    "Id": "/subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/applicationGateways/appgwtest/frontendIP
                                Configurations/frontend1"
                                }
    DnsSettings              : {
                                    "Fqdn": "00000000-0000-xxxx-xxxx-xxxxxxxxxxxx.chinacloudapp.cn"
                                }

## <a name="next-steps"></a>后续步骤

若要配置 SSL 卸载，请访问：[配置应用程序网关以进行 SSL 卸载](/documentation/articles/application-gateway-ssl/)。

若要将应用程序网关配置为与内部负载均衡器配合使用，请访问：[创建具有内部负载均衡器 (ILB) 的应用程序网关](/documentation/articles/application-gateway-ilb/)。

如需大体上更详细地了解负载均衡选项，请访问：

* [Azure 负载均衡器](/documentation/services/load-balancer/)
* [Azure 流量管理器](/documentation/services/traffic-manager/)

<!--Update_Description: wording update-->