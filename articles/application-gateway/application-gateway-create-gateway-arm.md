<properties
   pageTitle="使用 Azure 资源管理器创建、启动或删除应用程序网关 | Azure"
   description="本页提供有关使用 Azure 资源管理器创建、配置、启动和删除 Azure 应用程序网关的说明"
   documentationCenter="na"
   services="application-gateway"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn"/>
<tags
   ms.service="application-gateway"
   ms.date="04/05/2016"
   wacn.date="05/12/2016"/>


# 使用 Azure 资源管理器创建、启动或删除应用程序网关

Azure 应用程序网关是第 7 层负载平衡器。它在不同服务器之间提供故障转移和性能路由 HTTP 请求，而不管它们是在云中还是本地。应用程序网关具有以下应用程序传递功能：HTTP 负载平衡、基于 Cookie 的会话相关性和安全套接字层 (SSL) 卸载。


> [AZURE.SELECTOR]
- [Azure 经典 PowerShell 步骤](/documentation/articles/application-gateway-create-gateway/)
- [Azure 资源管理器 PowerShell](/documentation/articles/application-gateway-create-gateway-arm/)
- [Azure 资源管理器模板](/documentation/articles/application-gateway-create-gateway-arm-template/)

本文将指导你完成创建、配置、启动和删除应用程序网关的步骤。


>[AZURE.IMPORTANT] 在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文档将介绍如何使用 Azure 资源管理器创建应用程序网关。若要使用经典版本，请转到[使用 PowerShell 创建应用程序网关经典部署](/documentation/articles/application-gateway-create-gateway/)。



## 开始之前

1. 使用 Web 平台安装程序安装最新版本的 Azure PowerShell cmdlet。可以从[下载页面](/downloads)的“Windows PowerShell”部分下载并安装最新版本。
2. 你将为应用程序网关创建虚拟网络和子网。请确保没有虚拟机或云部署正在使用子网。应用程序网关必须单独位于虚拟网络子网中。
3. 要配置为使用应用程序网关的服务器必须存在，或者在虚拟网络中为其创建终结点，或者为其分配公共 IP/VIP。

## 创建应用程序网关需要什么？


- **后端服务器池：**后端服务器的 IP 地址列表。列出的 IP 地址应属于虚拟网络子网，或者是公共 IP/VIP。
- **后端服务器池设置：**每个池都有一些设置，例如端口、协议和基于 Cookie 的关联性。这些设置绑定到池，并会应用到池中的所有服务器。
- **前端端口：**此端口是应用程序网关上打开的公共端口。流量将抵达此端口，然后重定向到后端服务器之一。
- **侦听器：**侦听器具有前端端口、协议（Http 或 Https，区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。
- **规则：**规则将会绑定侦听器和后端服务器池，并定义当流量抵达特定侦听器时应定向到的后端服务器池。 



## 创建新的应用程序网关

使用 Azure 经典管理门户和 Azure 资源管理器的差别在于创建应用程序网关的顺序和需要配置的项。

使用资源管理器，组成应用程序网关的所有项都将分开配置，然后放在一起创建应用程序网关资源。


以下是创建应用程序网关所需执行的步骤：

1. 创建资源管理器的资源组。
2. 创建应用程序网关的虚拟网络、子网和公共 IP。
3. 创建应用程序网关配置对象。
4. 创建应用程序网关资源。


## 创建资源管理器的资源组

确保使用最新版本的 Azure PowerShell。[将 Windows PowerShell 与资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager/)中提供了详细信息。

### 步骤 1
登录到 Azure。

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

系统将提示你使用凭据进行身份验证。<BR>
### 步骤 2
检查该帐户的订阅。

		Get-AzureRmSubscription

### 步骤 3
选择要使用的 Azure 订阅。<BR>

		Select-AzureRmSubscription -Subscriptionid "GUID of subscription"

### 步骤 4
创建新的资源组（如果要使用现有的资源组，请跳过此步骤）。

    New-AzureRmResourceGroup -Name appgw-rg -location "China North"

Azure 资源管理器要求所有资源组指定一个位置。此位置将用作该资源组中的资源的默认位置。请确保用于创建应用程序网关的所有命令都使用相同的资源组。

在上面的示例中，我们在位置“中国北部”创建了名为“appgw-RG”的资源组。

>[AZURE.NOTE] 如果你需要为应用程序网关配置自定义探测，请参阅[使用 PowerShell 创建带自定义探测的应用程序网关](/documentation/articles/application-gateway-create-probe-ps/)。有关详细信息，请查看[自定义探测和运行状况监视](/documentation/articles/application-gateway-probe-overview/)。



## 为应用程序网关创建虚拟网络和子网

以下示例演示如何使用资源管理器创建虚拟网络。

### 步骤 1

将地址范围 10.0.0.0/24 分配给用于创建虚拟网络的子网变量。

	$subnet = New-AzureRmVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24


### 步骤 2

使用前缀 10.0.0.0/16 和子网 10.0.0.0/24，在中国北部区域的“appgw-rg”资源组中创建名为“appgwvnet”的虚拟网络。

	$vnet = New-AzureRmVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-rg -Location "China North" -AddressPrefix 10.0.0.0/16 -Subnet $subnet


### 步骤 3

分配子网变量，以完成后面的创建应用程序网关的步骤。

	$subnet=$vnet.Subnets[0]

## 创建前端配置的公共 IP 地址

在中国北部区域的“appgw-rg”资源组中创建公共 IP 资源“publicIP01”。

	$publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-rg -name publicIP01 -location "China North" -AllocationMethod Dynamic


## 创建应用程序网关配置对象

在创建应用程序网关之前，需要设置所有配置项。以下步骤将创建应用程序网关资源所需的配置项。

### 步骤 1

创建名为“gatewayIP01”的应用程序网关 IP 配置。当应用程序网关启动时，它会从配置的子网获取 IP 地址，再将网络流量路由到后端 IP 池中的 IP 地址。请记住，每个实例需要一个 IP 地址。


	$gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet


### 步骤 2

配置名为“pool01”的后端 IP 地址池，其 IP 地址为“134.170.185.46, 134.170.188.221,134.170.185.50”。这些 IP 地址将接收来自前端 IP 终结点的网络流量。你要替换上述 IP 地址，添加你自己的应用程序 IP 地址终结点。

	$pool = New-AzureRmApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 134.170.185.46, 134.170.188.221,134.170.185.50



### 步骤 3

为后端池中进行了负载平衡的网络流量配置应用程序网关设置“poolsetting01”。

	$poolSetting = New-AzureRmApplicationGatewayBackendHttpSettings -Name poolsetting01 -Port 80 -Protocol Http -CookieBasedAffinity Disabled


### 步骤 4

为公共 IP 终结点配置名为“frontendport01”的前端 IP 端口。

	$fp = New-AzureRmApplicationGatewayFrontendPort -Name frontendport01  -Port 80

### 步骤 5

创建名为“fipconfig01”的前端 IP 配置，并将公共 IP 地址与前端 IP 配置相关联。

	$fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name fipconfig01 -PublicIPAddress $publicip


### 步骤 6

创建名为“listener01”的侦听器，并将前端端口与前端 IP 配置相关联。

	$listener = New-AzureRmApplicationGatewayHttpListener -Name listener01  -Protocol Http -FrontendIPConfiguration $fipconfig -FrontendPort $fp

### 步骤 7

创建名为“rule01”的负载平衡器路由规则，并配置负载平衡器的行为。

	$rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool

### 步骤 8

配置应用程序网关的实例大小。

	$sku = New-AzureRmApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2

>[AZURE.NOTE]  InstanceCount 的默认值为 2，最大值为 10。GatewaySize 的默认值为 Medium。你可以在 Standard\_Small、Standard\_Medium 和 Standard\_Large 之间进行选择。

## 使用 New-AzureRmApplicationGateway 创建应用程序网关

创建包含上述步骤中所有配置项的应用程序网关。示例中的应用程序网关名为“appgwtest”。

	$appgw = New-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Location "China North" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku


## 删除应用程序网关

若要删除应用程序网关，请执行以下步骤：

1. 使用 **Stop-AzureRmApplicationGateway** cmdlet 停止网关。
2. 使用 **Remove-AzureRmApplicationGateway** cmdlet 删除网关。
3. 验证是否已使用 **Get-AzureRmApplicationGateway** cmdlet 删除网关。

### 步骤 1

获取应用程序网关对象，并将其关联到变量“$getgw”。

	$getgw =  Get-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg

### 步骤 2

使用 **Stop-AzureRmApplicationGateway** 停止应用程序网关。

	Stop-AzureRmApplicationGateway -ApplicationGateway $getgw  


应用程序网关进入停止状态后，请使用 **Remove-AzureRmApplicationGateway** cmdlet 删除该服务。


	Remove-AzureRmApplicationGateway -Name $appgwtest -ResourceGroupName appgw-rg -Force



>[AZURE.NOTE] 可以使用 **-force** 开关来禁止显示该删除的确认消息。


若要验证是否已删除服务，可以使用 **Get-AzureRmApplicationGateway** cmdlet。此步骤不是必需的。


	Get-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg


## 后续步骤

如果你要配置 SSL 卸载，请参阅[配置应用程序网关以进行 SSL 卸载](/documentation/articles/application-gateway-ssl/)。

如果你想要将应用程序网关配置为与内部负载平衡器配合使用，请参阅[创建具有内部负载平衡器 (ILB) 的应用程序网关](/documentation/articles/application-gateway-ilb/)。

如需负载平衡选项的其他常规信息，请参阅：

<!--- [Azure 负载平衡器](/documentation/services/load-balancer)-->
- [Azure 流量管理器](/documentation/services/traffic-manager)

<!---HONumber=Mooncake_0425_2016-->
