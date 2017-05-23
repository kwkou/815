<properties
    pageTitle="使用 URL 路由规则创建应用程序网关 | Azure"
    description="本页提供有关使用 URL 路由规则创建、配置 Azure 应用程序网关的说明"
    documentationcenter="na"
    services="application-gateway"
    author="georgewallace"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="d141cfbb-320a-4fc9-9125-10001c6fa4cf"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="04/03/2017"
    wacn.date="05/22/2017"
    ms.author="gwallace"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="72201c01098c7a9fbc49934487fce8ee6ed3d8d2"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="create-an-application-gateway-using-path-based-routing"></a>使用基于路径的路由创建应用程序网关
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/application-gateway-create-url-route-portal/)
- [Azure Resource Manager PowerShell](/documentation/articles/application-gateway-create-url-route-arm-ps/)

借助基于 URL 路径的路由，可根据 Http 请求的 URL 路径来关联路由。 它将检查是否有路由连接到为应用程序网关中显示的 URL 配置的后端池，并将网络流量发送到定义的后端池。 基于 URL 的路由的常见用法是将不同内容类型的请求负载均衡到不同的后端服务器池。

基于 URL 的路由将新的规则类型引入应用程序网关。 应用程序网关有两种规则类型：基本和 PathBasedRouting。 基本规则类型针对后端池提供轮循机制服务，而 PathBasedRouting 除了轮循机制分发以外，还在选择后端池时考虑请求 URL 的路径模式。

## <a name="scenario"></a>方案

在以下示例中，应用程序网关使用两个后端服务器池来为 contoso.com 提供流量：视频服务器池和图像服务器池。

对 http://contoso.com/image* 的请求会路由到图像服务器池 (pool1)，对 http://contoso.com/video* 的请求会路由到视频服务器池 (pool2)。 如果没有任何路径模式匹配，则选择默认的服务器池 (pool1)。

![url 路由](./media/application-gateway-create-url-route-arm-ps/figure1.png)

## <a name="before-you-begin"></a>准备阶段

1. 使用 Web 平台安装程序安装最新版本的 Azure PowerShell cmdlet。 可以从[下载页](/downloads/)的“Windows PowerShell”部分下载并安装最新版本。
2. 将为应用程序网关创建虚拟网络和子网。 请确保没有虚拟机或云部署正在使用子网。 应用程序网关必须单独位于虚拟网络子网中。
3. 为使用应用程序网关而添加到后端池的服务器必须存在，或者在虚拟网络中为其创建终结点，或者为其分配公共 IP/VIP。

## <a name="what-is-required-to-create-an-application-gateway"></a>创建应用程序网关需要什么？

* **后端服务器池：** 后端服务器的 IP 地址列表。 列出的 IP 地址应属于虚拟网络子网，或者是公共 IP/VIP。
* **后端服务器池设置：** 每个池都有一些设置，例如端口、协议和基于 Cookie 的关联性。 这些设置绑定到池，并会应用到池中的所有服务器。
* **前端端口：** 此端口是应用程序网关上打开的公共端口。 流量将抵达此端口，然后重定向到后端服务器之一。
* **侦听器：**侦听器具有前端端口、协议（Http 或 Https，这些值区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。
* **规则：** 规则将会绑定侦听器和后端服务器池，并定义当流量抵达特定侦听器时应定向到的后端服务器池。

## <a name="create-an-application-gateway"></a>创建应用程序网关

使用 Azure 经典部署和 Azure Resource Manager 部署的差别在于创建应用程序网关的顺序和需要配置的项。

使用 Resource Manager 时，组成应用程序网关的所有项都将分开配置，然后结合在一起来创建应用程序网关资源。

以下是创建应用程序网关所需执行的步骤：

1. 创建 Resource Manager 的资源组。
2. 创建应用程序网关的虚拟网络、子网和公共 IP。
3. 创建应用程序网关配置对象。
4. 创建应用程序网关资源。

## <a name="create-a-resource-group-for-resource-manager"></a>创建 Resource Manager 的资源组

确保使用最新版本的 Azure PowerShell。 [将 Windows PowerShell 与 Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)中提供了详细信息。

### <a name="step-1"></a>步骤 1

登录 Azure

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

系统会提示使用凭据进行身份验证。<BR>

### <a name="step-2"></a>步骤 2

检查该帐户的订阅。

    Get-AzureRmSubscription

### <a name="step-3"></a>步骤 3

选择要使用的 Azure 订阅。 <BR>

    Select-AzureRmSubscription -Subscriptionid "GUID of subscription"

### <a name="step-4"></a>步骤 4

创建资源组（如果要使用现有的资源组，请跳过此步骤）。

    $resourceGroup = New-AzureRmResourceGroup -Name appgw-RG -Location "China North"

或者，可以为应用程序网关的资源组创建标记：

    $resourceGroup = New-AzureRmResourceGroup -Name appgw-RG -Location "China North" -Tags @{Name = "testtag"; Value = "Application Gateway URL routing"} 

Azure Resource Manager 要求所有资源组指定一个位置。 此位置将用作该资源组中的资源的默认位置。 请确保用于创建应用程序网关的所有命令都使用相同的资源组。

在上面的示例中，我们在位置“中国北部”创建了名为“appgw-RG”的资源组。

> [AZURE.NOTE]
> 如果需要为应用程序网关配置自定义探测，请参阅 [Create an application gateway with custom probes by using PowerShell](/documentation/articles/application-gateway-create-probe-ps/)（使用 PowerShell 创建带自定义探测的应用程序网关）。 有关详细信息，请查看 [custom probes and health monitoring](/documentation/articles/application-gateway-probe-overview/)（自定义探测和运行状况监视）。
> 
> 

## <a name="create-a-virtual-network-and-a-subnet-for-the-application-gateway"></a>为应用程序网关创建虚拟网络和子网

以下示例演示如何使用 Resource Manager 创建虚拟网络。 该示例创建适用于应用程序网关的 VNET。 应用程序网关需要其自己的子网，因此，为应用程序网关创建的子网小于 VNET 地址空间。 这允许在同一 VNET 中配置其他资源（包括但不限于 Web 服务器）。

### <a name="step-1"></a>步骤 1

将地址范围 10.0.0.0/24 分配给用于创建虚拟网络的子网变量。  这会为应用程序网关创建子网配置对象，该对象将用在下一示例中。

    $subnet = New-AzureRmVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24

### <a name="step-2"></a>步骤 2

使用前缀 10.0.0.0/16 和子网 10.0.0.0/24，在中国北部区域的 **appgw-rg** 资源组中创建名为 **appgwvnet** 的虚拟网络。 这会完成 VNET 的配置，该 VNET 有一个可供应用程序网关驻留的子网。

    $vnet = New-AzureRmVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-RG -Location "China North" -AddressPrefix 10.0.0.0/16 -Subnet $subnet

### <a name="step-3"></a>步骤 3

为后续步骤分配子网变量，将在以后的步骤中将该变量传递给 `New-AzureRMApplicationGateway` cmdlet。

    $subnet=$vnet.Subnets[0]

## <a name="create-a-public-ip-address-for-the-front-end-configuration"></a>创建前端配置的公共 IP 地址

在中国北部区域的 **appgw-rg** 资源组中创建公共 IP 资源 **publicIP01**。 应用程序网关可以使用公共 IP 地址和/或内部 IP 地址来接收负载均衡请求。  此示例仅使用公共 IP 地址。 在以下示例中，没有为创建公共 IP 地址而配置任何 DNS 名称。  应用程序网关不支持在公共 IP 地址上使用自定义 DNS 名称。  如果公共终结点需要自定义名称，则应创建一个 CNAME 记录，使之指向针对公共 IP 地址自动生成的 DNS 名称。

    $publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-RG -name publicIP01 -location "China North" -AllocationMethod Dynamic

服务启动时，会将一个 IP 地址分配到应用程序网关。

## <a name="create-application-gateway-configuration"></a>创建应用程序网关配置

在创建应用程序网关之前，必须设置所有配置项目。 以下步骤将创建应用程序网关资源所需的配置项目。

### <a name="step-1"></a>步骤 1

创建名为“gatewayIP01” 的应用程序网关 IP 配置。 当应用程序网关启动时，它会从配置的子网获取 IP 地址，再将网络流量路由到后端 IP 池中的 IP 地址。 请记住，每个实例需要一个 IP 地址。

    $gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet

### <a name="step-2"></a>步骤 2

分别配置名为 **pool01** 和 **pool2** 的后端 IP 地址池，且 **pool1** 和 **pool2** 具有相应的 IP 地址。 这些 IP 地址是托管 Web 应用程序的资源的 IP 地址，而这些 Web 应用程序受应用程序网关保护。 这些后端池成员都经探测（不管是基本探测还是自定义探测）验证为运行正常。  然后，当请求进入应用程序网关时，系统就会将流量路由到这些成员。 可以通过应用程序网关中的多个规则来使用后端池，这意味着，一个后端池可以用于驻留在同一主机上的多个 Web 应用程序。

    $pool1 = New-AzureRmApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 134.170.185.46, 134.170.188.221, 134.170.185.50

    $pool2 = New-AzureRmApplicationGatewayBackendAddressPool -Name pool02 -BackendIPAddresses 134.170.186.47, 134.170.189.222, 134.170.186.51

在本示例中，会有两个后端池根据 URL 路径路由网络流量。 一个池接收来自 URL 路径“/video”的流量，另一个池接收来自路径“/image”的流量。 替换上述 IP 地址，添加自己的应用程序 IP 地址终结点。 

### <a name="step-3"></a>步骤 3

为后端池中进行了负载均衡的网络流量配置应用程序网关设置“poolsetting01”和“poolsetting02”。 在本示例中，将为后端池配置不同的后端池设置。 每个后端池可有自身的后端池设置。  可以通过规则使用后端 HTTP 设置，以便将流量路由到正确的后端池成员。 将流量发送到后端池成员时，所用的协议和端口由此设置决定。 基于 Cookie 的会话也由后端 HTTP 设置决定。  在启用的情况下，基于 Cookie 的会话相关性会将流量发送到与每个数据包的前述请求相同的后端。

    $poolSetting01 = New-AzureRmApplicationGatewayBackendHttpSettings -Name "besetting01" -Port 80 -Protocol Http -CookieBasedAffinity Disabled -RequestTimeout 120

    $poolSetting02 = New-AzureRmApplicationGatewayBackendHttpSettings -Name "besetting02" -Port 80 -Protocol Http -CookieBasedAffinity Enabled -RequestTimeout 240

### <a name="step-4"></a>步骤 4

使用公共 IP 终结点配置前端 IP。 侦听器使用前端 IP 配置对象将外向 IP 地址与侦听器相关联。

    $fipconfig01 = New-AzureRmApplicationGatewayFrontendIPConfig -Name "frontend1" -PublicIPAddress $publicip

### <a name="step-5"></a>步骤 5

配置应用程序网关的前端端口。 侦听器使用前端端口配置对象来定义应用程序网关所侦听的具体端口，以便确定侦听器上的流量。

    $fp01 = New-AzureRmApplicationGatewayFrontendPort -Name "fep01" -Port 80

### <a name="step-6"></a>步骤 6

配置侦听器。 此步骤针对用于接收传入网络流量的公共 IP 地址和连接端口配置侦听器。 以下示例使用以前配置的前端 IP 配置、前端端口配置以及协议（http 或 https）对侦听器进行配置。 在此示例中，侦听器侦听端口 80 上的 HTTP 流量，该端口位于此前创建的公共 IP 地址上。

    $listener = New-AzureRmApplicationGatewayHttpListener -Name "listener01" -Protocol Http -FrontendIPConfiguration $fipconfig01 -FrontendPort $fp01

### <a name="step-7"></a>步骤 7

配置后端池的 URL 规则路径。 此步骤配置应用程序网关用于定义 URL 路径间映射的相对路径，并会分配到后端池来处理传入流量。

> [AZURE.IMPORTANT]
> 每个路径必须以 / 开头，“\*”只允许放在末尾处。 有效示例包括 /xyz、/xyz* 或 /xyz/*。 发送到路径匹配器的字符串不会在第一个“?”或“#”之后包含任何文本，不允许使用这些字符。 

以下示例将创建两个规则：一个用于将流量路由到后端“pool1”的“/image/”路径，另一个用于将流量路由到后端“pool2”的“/video/”路径。 这些规则可确保将每个 URL 集的流量路由到后端。 例如，http://contoso.com/image/figure1.jpg 将转到 pool1，而 http://contoso.com/video/example.mp4 将转到 pool2。

    $imagePathRule = New-AzureRmApplicationGatewayPathRuleConfig -Name "pathrule1" -Paths "/image/*" -BackendAddressPool $pool1 -BackendHttpSettings $poolSetting01

    $videoPathRule = New-AzureRmApplicationGatewayPathRuleConfig -Name "pathrule2" -Paths "/video/*" -BackendAddressPool $pool2 -BackendHttpSettings $poolSetting02

如果路径不符合任何预定义的路径规则，规则路径映射配置也会配置默认的后端地址池。 例如，http://contoso.com/shoppingcart/test.html 将转到 pool1，因为它定义为非匹配流量的默认池。

    $urlPathMap = New-AzureRmApplicationGatewayUrlPathMapConfig -Name "urlpathmap" -PathRules $videoPathRule, $imagePathRule -DefaultBackendAddressPool $pool1 -DefaultBackendHttpSettings $poolSetting02

### <a name="step-8"></a>步骤 8

创建规则设置。 此步骤将应用程序网关配置为使用基于 URL 路径的路由。 之前步骤中定义的 `$urlPathMap` 变量现用于创建基于路径的规则。 在此步骤中，将该规则与侦听器以及之前创建的 URL 路径映射关联。

    $rule01 = New-AzureRmApplicationGatewayRequestRoutingRule -Name "rule1" -RuleType PathBasedRouting -HttpListener $listener -UrlPathMap $urlPathMap

### <a name="step-9"></a>步骤 9

配置实例数目和应用程序网关的大小。

    $sku = New-AzureRmApplicationGatewaySku -Name "Standard_Small" -Tier Standard -Capacity 2

## <a name="create-application-gateway"></a>创建应用程序网关

创建包含前述步骤中所有配置对象的应用程序网关。

    $appgw = New-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-RG -Location "China North" -BackendAddressPools $pool1,$pool2 -BackendHttpSettingsCollection $poolSetting01, $poolSetting02 -FrontendIpConfigurations $fipconfig01 -GatewayIpConfigurations $gipconfig -FrontendPorts $fp01 -HttpListeners $listener -UrlPathMaps $urlPathMap -RequestRoutingRules $rule01 -Sku $sku

## <a name="get-application-gateway-dns-name"></a>获取应用程序网关 DNS 名称

创建网关后，下一步是配置用于通信的前端。 使用公共 IP 时，应用程序网关需要动态分配的 DNS 名称，这会造成不方便。 若要确保最终用户能够访问应用程序网关，可以使用指向应用程序网关的公共终结点的 CNAME 记录。 若要配置前端 IP CNAME 记录，可使用附加到应用程序网关的 PublicIPAddress 元素检索应用程序网关及其关联的 IP/DNS 名称的详细信息。 应使用应用程序网关的 DNS 名称来创建 CNAME 记录，使两个 Web 应用程序都指向此 DNS 名称。 不建议使用 A 记录，因为重新启动应用程序网关后 VIP 可能会变化。

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

如果要了解安全套接字层 (SSL) 卸载，请参阅[配置应用程序网关以进行 SSL 卸载](/documentation/articles/application-gateway-ssl-arm/)。

<!--Update_Description: wording update-->