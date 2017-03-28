<properties
    pageTitle="配置 SSL 卸载 - Azure 应用程序网关 - PowerShell | Azure"
    description="本页提供有关使用 Azure Resource Manager 创建支持 SSL 卸载的应用程序网关的说明"
    documentationcenter="na"
    services="application-gateway"
    author="georgewallace"
    manager="carmonm"
    editor="tysonn" />  

<tags
    ms.assetid="3c3681e0-f928-4682-9d97-567f8e278e13"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="12/16/2016"
    wacn.date="03/28/2017"
    ms.author="gwallace" />

# 使用 Azure Resource Manager 配置应用程序网关以进行 SSL 卸载
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/application-gateway-ssl-portal/)
- [Azure Resource Manager PowerShell](/documentation/articles/application-gateway-ssl-arm/)
- [Azure 经典 PowerShell](/documentation/articles/application-gateway-ssl/)

 可将 Azure 应用程序网关配置为在网关上终止安全套接字层 (SSL) 会话，以避免 Web 场中出现开销较高的 SSL 解密任务。SSL 卸载还简化了 Web 应用程序的前端服务器设置与管理。

## 开始之前

1. 使用 Web 平台安装程序安装最新版本的 Azure PowerShell cmdlet。可以从“[下载](/downloads/)”页的“Windows PowerShell”部分下载并安装最新版本。
2. 为应用程序网关创建虚拟网络和子网。请确保没有虚拟机或云部署正在使用子网。应用程序网关必须单独位于虚拟网络子网中。
3. 必须存在配置为使用应用程序网关的服务器，或者必须在虚拟网络中为其创建终结点，或者必须为其分配公共 IP/VIP。

## 创建应用程序网关需要什么？

* **后端服务器池：**后端服务器的 IP 地址列表。列出的 IP 地址应属于虚拟网络子网，或者是公共 IP/VIP。
* **后端服务器池设置：**每个池都有一些设置，例如端口、协议和基于 Cookie 的关联性。这些设置绑定到池，并会应用到池中的所有服务器。
* **前端端口：**此端口是应用程序网关上打开的公共端口。流量将抵达此端口，然后重定向到后端服务器之一。
* **侦听器：**侦听器具有前端端口、协议（Http 或 Https，这些设置区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。
* **规则：**规则将会绑定侦听器和后端服务器池，并定义当流量抵达特定侦听器时应定向到的后端服务器池。目前仅支持*基本*规则。*基本*规则是一种轮循负载分发模式。

**其他配置说明**

对于 SSL 证书配置，**HttpListener** 中的协议应更改为 *Https*（区分大小写）。需要将 **SslCertificate** 元素添加到 **HttpListener**，其中包含针对 SSL 证书配置的变量值。前端端口应更新为 443。

**启用基于 Cookie 的相关性**：可以配置应用程序网关，以确保来自客户端会话的请求始终定向到 Web 场中的同一 VM。此方案是通过注入允许网关适当定向流量的会话 Cookie 来实现的。若要启用基于 Cookie 的相关性，请在 BackendHttpSettings 元素中将 CookieBasedAffinity 设置为 Enabled。

## 创建应用程序网关

使用 Azure 经典部署模型和 Azure Resource Manager 的差别在于创建应用程序网关的顺序和需要配置的项。

使用 Resource Manager 时，应用程序网关的所有组件都将分开配置，然后放在一起创建应用程序网关资源。

以下是创建应用程序网关所要执行的步骤：

1. 创建资源管理器的资源组
2. 创建应用程序网关的虚拟网络、子网和公共 IP。
3. 创建应用程序网关配置对象
4. 创建应用程序网关资源

## 创建资源管理器的资源组
确保切换 PowerShell 模式，以便使用 Azure Resource Manager cmdlet。[将 Windows PowerShell 与 Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)中提供了详细信息。

### 步骤 1

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

### 步骤 2

检查该帐户的订阅。

    Get-AzureRmSubscription

系统会提示使用凭据进行身份验证。

### 步骤 3

选择要使用的 Azure 订阅。

    Select-AzureRmSubscription -Subscriptionid "GUID of subscription"

### 步骤 4

创建资源组（如果要使用现有的资源组，请跳过此步骤）。

    New-AzureRmResourceGroup -Name appgw-rg -Location "China North"

Azure 资源管理器要求所有资源组指定一个位置。此设置用作该资源组中的资源的默认位置。请确保用于创建应用程序网关的所有命令都使用相同的资源组。

在上面的示例中，我们在位置“中国北部”创建了名为“appgw-RG”的资源组。

## 为应用程序网关创建虚拟网络和子网

以下示例演示如何使用 Resource Manager 创建虚拟网络：

### 步骤 1

    $subnet = New-AzureRmVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24

此示例将地址范围 10.0.0.0/24 分配给用于创建虚拟网络的子网变量。

### 步骤 2

    $vnet = New-AzureRmVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-rg -Location "China North" -AddressPrefix 10.0.0.0/16 -Subnet $subnet

此示例使用前缀 10.0.0.0/16 和子网 10.0.0.0/24，在中国北部区域的“appgw-rg”资源组中创建名为“appgwvnet”的虚拟网络。

### 步骤 3

    $subnet = $vnet.Subnets[0]

此示例将子网对象分配到变量 $subnet 以完成后续步骤。

## 创建前端配置的公共 IP 地址

    $publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-rg -name publicIP01 -location "China North" -AllocationMethod Dynamic

此示例将在中国北部区域的“appgw-rg”资源组中创建公共 IP 资源“publicIP01”。

## 创建应用程序网关配置对象

### 步骤 1

    $gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet

此示例创建名为“gatewayIP01”的应用程序网关 IP 配置。当应用程序网关启动时，它会从配置的子网获取 IP 地址，再将网络流量路由到后端 IP 池中的 IP 地址。请记住，每个实例需要一个 IP 地址。

### 步骤 2

    $pool = New-AzureRmApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 134.170.185.46, 134.170.188.221,134.170.185.50

此示例配置名为“pool01”的后端 IP 地址池，其 IP 地址为“134.170.185.46”、“134.170.188.221”、“134.170.185.50”。这些值是接收来自前端 IP 终结点的网络流量的 IP 地址。将上述示例中的 IP 地址替换为 Web 应用程序终结点的 IP 地址。

### 步骤 3

    $poolSetting = New-AzureRmApplicationGatewayBackendHttpSettings -Name poolsetting01 -Port 80 -Protocol Http -CookieBasedAffinity Enabled

此示例为后端池中负载均衡的网络流量配置应用程序网关设置“poolsetting01”。

### 步骤 4

    $fp = New-AzureRmApplicationGatewayFrontendPort -Name frontendport01  -Port 443

此示例为公共 IP 终结点配置名为“frontendport01”的前端 IP 端口。

### 步骤 5

    $cert = New-AzureRmApplicationGatewaySslCertificate -Name cert01 -CertificateFile <full path for certificate file> -Password "<password>"

此示例配置用于 SSL 连接的证书。该证书需采用 .pfx 格式，并且密码必须为 4 到 12 个字符。

### 步骤 6

    $fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name fipconfig01 -PublicIPAddress $publicip

此示例创建名为“fipconfig01”的前端 IP 配置，并将公共 IP 地址与前端 IP 配置相关联。

### 步骤 7

    $listener = New-AzureRmApplicationGatewayHttpListener -Name listener01  -Protocol Https -FrontendIPConfiguration $fipconfig -FrontendPort $fp -SslCertificate $cert

此示例创建名为“listener01”的侦听器；将前端端口与前端 IP 配置和证书相关联。

### 步骤 8

    $rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool

此示例创建名为“rule01”的负载均衡器路由规则，并配置负载均衡器的行为。

### 步骤 9

    $sku = New-AzureRmApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2

此示例配置应用程序网关的实例大小。

> [AZURE.NOTE]
*InstanceCount* 的默认值为 2，最大值为 10。*GatewaySize* 的默认值为 Medium。你可以在 Standard\_Small、Standard\_Medium 和 Standard\_Large 之间进行选择。
> 
> 

## 使用 New-AzureApplicationGateway 创建应用程序网关

    $appgw = New-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Location "China North" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku -SslCertificates $cert

此示例创建包含前述步骤中所有配置项的应用程序网关。示例中的应用程序网关名为“appgwtest”。

## 获取应用程序网关 DNS 名称

创建网关后，下一步是配置用于通信的前端。使用公共 IP 时，应用程序网关需要动态分配的 DNS 名称，这会造成不方便。若要确保最终用户能够访问应用程序网关，可以使用指向应用程序网关的公共终结点的 CNAME 记录。为此，可使用附加到应用程序网关的 PublicIPAddress 元素检索应用程序网关及其关联的 IP/DNS 名称的详细信息。应使用应用程序网关的 DNS 名称来创建 CNAME 记录，使两个 Web 应用程序都指向此 DNS 名称。不建议使用 A 记录，因为重新启动应用程序网关后 VIP 可能会变化。


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

## 后续步骤

如果要将应用程序网关配置为与内部负载均衡器 (ILB) 配合使用，请参阅 [Create an application gateway with an internal load balancer (ILB)](/documentation/articles/application-gateway-ilb/)（创建具有内部负载均衡器 (ILB) 的应用程序网关）。

如需负载均衡选项的其他常规信息，请参阅：

* [Azure Load Balancer](/documentation/services/load-balancer/)
* [Azure 流量管理器](/documentation/services/traffic-manager/)

<!---HONumber=Mooncake_Quality_Review_1230_2016-->