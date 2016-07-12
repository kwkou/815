<properties 
   pageTitle="使用 Azure 资源管理器配置应用程序网关以进行 SSL 卸载 | Azure"
   description="本页提供有关使用 Azure 资源管理器创建支持 SSL 卸载的应用程序网关的说明"
   documentationCenter="na"
   services="application-gateway"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn"/>
<tags
   ms.service="application-gateway"
   ms.date="03/03/2016"
   wacn.date="06/30/2016"/>

# 使用 Azure 资源管理器配置应用程序网关以进行 SSL 卸载

> [AZURE.SELECTOR]
-[Azure Classic PowerShell](/documentation/articles/application-gateway-ssl/)
-[Azure 资源管理器 PowerShell](/documentation/articles/application-gateway-ssl-arm/)

 可将 Azure 应用程序网关配置为在网关上终止安全套接字层 (SSL) 会话，以避免 Web 场中出现开销较高的 SSL 解密任务。SSL 卸载还简化了 Web 应用程序的前端服务器设置与管理。


## 开始之前

1. 使用 Web 平台安装程序安装最新版本的 Azure PowerShell cmdlet。可以从[下载页面](/downloads)的“Windows PowerShell”部分下载并安装最新版本。
2. 你将为应用程序网关创建虚拟网络和子网。请确保没有虚拟机或云部署正在使用子网。应用程序网关必须单独位于虚拟网络子网中。
3. 要配置为使用应用程序网关的服务器必须存在，或者在虚拟网络中为其创建终结点，或者为其分配公共 IP/VIP。

## 创建应用程序网关需要什么？


- **后端服务器池：**后端服务器的 IP 地址列表。列出的 IP 地址应属于虚拟网络子网，或者是公共 IP/VIP。
- **后端服务器池设置：**每个池都有一些设置，例如端口、协议和基于 Cookie 的关联性。这些设置绑定到池，并会应用到池中的所有服务器。
- **前端端口：**此端口是应用程序网关上打开的公共端口。客户流量将抵达此端口，然后重定向到后端服务器之一。
- **侦听器：**侦听器具有前端端口、协议（Http 或 Https，区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。
- **规则：**规则将会绑定侦听器和后端服务器池，并定义当流量抵达特定侦听器时应定向到的后端服务器池。目前仅支持*基本*规则。*基本*规则是一种轮循负载分发模式。

**其他配置说明**

对于 SSL 证书配置，**HttpListener** 中的协议应更改为 *Https*（区分大小写）。需要将 **SslCertificate** 元素添加到 **HttpListener**，后者包含针对 SSL 证书配置的变量值。前端端口应更新为 443。

**启用基于 Cookie 的相关性**：可以配置应用程序网关，以确保来自客户端会话的请求始终定向到 Web 场中的同一 VM。这是通过注入允许网关适当定向流量的会话 Cookie 来实现的。若要启用基于 Cookie 的相关性，请在 **BackendHttpSettings** 元素中将 **CookieBasedAffinity** 设置为 *Enabled* 。


## 创建新的应用程序网关

使用 Azure 经典部署模型和 Azure 资源管理器的差别在于创建应用程序网关的顺序和需要配置的项。

使用资源管理器，组成应用程序网关的所有项都将分开配置，然后放在一起创建应用程序网关资源。


以下是创建应用程序网关所要执行的步骤：

1. 创建资源管理器的资源组
2. 创建应用程序网关的虚拟网络、子网和公共 IP
3. 创建应用程序网关配置对象
4. 创建应用程序网关资源


## 创建资源管理器的资源组

确保切换 PowerShell 模式，以便使用 Azure Resource Manager cmdlet。[将 Windows PowerShell 与资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager/)中提供了详细信息。

### 步骤 1

		PS C:\> Login-AzureRmAccount -EnvironmentName AzureChinaCloud



### 步骤 2

检查该帐户的订阅。

		PS C:\> get-AzureRmSubscription

系统将提示你使用凭据进行身份验证。<BR>

### 步骤 3

选择要使用的 Azure 订阅。<BR>


		PS C:\> Select-AzureRmSubscription -Subscriptionid "GUID of subscription"


### 步骤 4

创建新的资源组（如果要使用现有的资源组，请跳过此步骤）。

    New-AzureRmResourceGroup -Name appgw-rg -location "China North"

Azure 资源管理器要求所有资源组指定一个位置。此位置将用作该资源组中的资源的默认位置。请确保用于创建应用程序网关的所有命令都使用相同的资源组。

在上面的示例中，我们在位置“中国北部”创建了名为“appgw-RG”的资源组。

## 为应用程序网关创建虚拟网络和子网

以下示例演示如何使用资源管理器创建虚拟网络：

### 步骤 1

	$subnet = New-AzureRmVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24

将地址范围 10.0.0.0/24 分配给用于创建虚拟网络的子网变量。

### 步骤 2
	$vnet = New-AzureRmVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-rg -Location "China North" -AddressPrefix 10.0.0.0/16 -Subnet $subnet

使用前缀 10.0.0.0/16 和子网 10.0.0.0/24，在中国北部区域的“appgw-rg”资源组中创建名为“appgwvnet”的虚拟网络。

### 步骤 3

	$subnet=$vnet.Subnets[0]

将子网对象分配到变量 $subnet 以完成后续步骤。

## 创建前端配置的公共 IP 地址

	$publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-rg -name publicIP01 -location "China North" -AllocationMethod Dynamic

将在中国北部区域的“appgw-rg”资源组中创建公共 IP 资源“publicIP01”。


## 创建应用程序网关配置对象

### 步骤 1

	$gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet

创建名为“gatewayIP01”的应用程序网关 IP 配置。当应用程序网关启动时，它会从配置的子网获取 IP 地址，再将网络流量路由到后端 IP 池中的 IP 地址。请记住，每个实例需要一个 IP 地址。

### 步骤 2

	$pool = New-AzureRmApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 134.170.185.46, 134.170.188.221,134.170.185.50

配置名为“pool01”的后端 IP 地址池，其 IP 地址为“134.170.185.46, 134.170.188.221,134.170.185.50”。 这些 IP 地址将接收来自前端 IP 终结点的网络流量。将上述示例中的 IP 地址替换为你的 Web 应用程序终结点的 IP 地址。

### 步骤 3

	$poolSetting = New-AzureRmApplicationGatewayBackendHttpSettings -Name poolsetting01 -Port 80 -Protocol Http -CookieBasedAffinity Enabled

为后端池中负载平衡的网络流量配置应用程序网关设置“poolsetting01”。

### 步骤 4

	$fp = New-AzureRmApplicationGatewayFrontendPort -Name frontendport01  -Port 443

为公共 IP 终结点配置名为“frontendport01”的前端 IP 端口。

### 步骤 5

	$cert = New-AzureRmApplicationGatewaySslCertificate -Name cert01 -CertificateFile <full path for certificate file> -Password ‘<password>’

配置用于 SSL 连接的证书。该证书需采用 .pfx 格式，并且密码必须为 4 到 12 个字符。

### 步骤 6

	$fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name fipconfig01 -PublicIPAddress $publicip

创建名为“fipconfig01”的前端 IP 配置，并将公共 IP 地址与前端 IP 配置相关联。

### 步骤 7

	$listener = New-AzureRmApplicationGatewayHttpListener -Name listener01  -Protocol Https -FrontendIPConfiguration $fipconfig -FrontendPort $fp -SslCertificate $cert


创建名为“listener01”的侦听器；将前端端口与前端 IP 配置和证书相关联。

### 步骤 8

	$rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool

创建名为“rule01”的负载平衡器路由规则，并配置负载平衡器的行为。

### 步骤 9

	$sku = New-AzureRmApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2

配置应用程序网关的实例大小。

>[AZURE.NOTE]  *InstanceCount* 的默认值为 2，最大值为 10。*GatewaySize* 的默认值为 Medium。你可以在 Standard\_Small、Standard\_Medium 和 Standard\_Large 之间进行选择。

## 使用 New-AzureApplicationGateway 创建应用程序网关

	$appgw = New-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Location "China North" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku -SslCertificates $cert

创建包含上述步骤中所有配置项的应用程序网关。示例中的应用程序网关名为“appgwtest”。

## 后续步骤

如果你想要将应用程序网关配置为与内部负载平衡器 (ILB) 配合使用，请参阅[创建具有内部负载平衡器 (ILB) 的应用程序网关](/documentation/articles/application-gateway-ilb/)。

如需负载平衡选项的其他常规信息，请参阅：

- [Azure 流量管理器](/documentation/services/traffic-manager)

<!---HONumber=Mooncake_0328_2016-->
