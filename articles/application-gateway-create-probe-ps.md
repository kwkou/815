<properties
   pageTitle="使用资源管理器中的 PowerShell 创建应用程序网关的自定义探测 | Azure"
   description="了解如何使用资源管理器中的 PowerShell 创建应用程序网关的自定义探测"
   services="application-gateway"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags  
   ms.service="application-gateway"
   ms.date="03/22/2016"
   wacn.date="06/30/2016" />

# 使用适用于 Azure 资源管理器的 PowerShell 创建 Azure 应用程序网关的自定义探测

[AZURE.INCLUDE [azure-probe-intro-include](../includes/application-gateway-create-probe-intro-include.md)]


> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代[经典部署模型](/documentation/articles/application-gateway-create-probe-classic-ps/)。


[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../includes/azure-ps-prerequisites-include.md)]


### 步骤 1

使用 Login-AzureRmAccount 进行身份验证。

	Login-AzureRmAccount -EnvironmentName AzureChinaCloud

### 步骤 2

检查该帐户的订阅。

		get-AzureRmSubscription

系统将提示你使用凭据进行身份验证。<BR>

### 步骤 3

选择要使用的 Azure 订阅。<BR>


		Select-AzureRmSubscription -Subscriptionid "GUID of subscription"


### 步骤 4

创建新的资源组（如果要使用现有的资源组，请跳过此步骤）。

    New-AzureRmResourceGroup -Name appgw-rg -location "China North"

Azure 资源管理器要求所有资源组指定一个位置。此位置将用作该资源组中的资源的默认位置。请确保用于创建应用程序网关的所有命令都使用相同的资源组。

在上面的示例中，我们在位置“中国北部”创建了名为“appgw-RG”的资源组。

## 为应用程序网关创建虚拟网络和子网

以下步骤将为应用程序网关创建虚拟网络和子网。

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


## 创建使用自定义探测的应用程序网关配置对象

在创建应用程序网关之前，需要设置所有配置项。以下步骤将创建应用程序网关资源所需的配置项。


### 步骤 1

创建名为“gatewayIP01”的应用程序网关 IP 配置。当应用程序网关启动时，它会从配置的子网获取 IP 地址，再将网络流量路由到后端 IP 池中的 IP 地址。请记住，每个实例需要一个 IP 地址。

	$gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet


### 步骤 2


配置名为“pool01”的后端 IP 地址池，其 IP 地址为“134.170.185.46, 134.170.188.221,134.170.185.50”。这些 IP 地址将接收来自前端 IP 终结点的网络流量。你要替换上述 IP 地址，添加你自己的应用程序 IP 地址终结点。

	$pool = New-AzureRmApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 134.170.185.46, 134.170.188.221,134.170.185.50


### 步骤 3


在此步骤中配置自定义探测。

使用的参数为：

- **-Interval** - 配置探测检查间隔，以秒为单位。
- **-Timeout** - 定义 HTTP 响应检查的探测超时。
- **-Hostname 和 -path** - 应用程序网关为了确定实例运行状况而调用的完整 URL 路径。例如，如果你的网站为 http://contoso.com/ ，则可以为“http://contoso.com/path/custompath.htm” 配置自定义探测，使探测检查能够获得成功的 HTTP 响应。
- **-UnhealthyThreshold** - 将后端实例标记为 *不正常* 所需的失败 HTTP 响应数目。



	$probe = New-AzureRmApplicationGatewayProbeConfig -Name probe01 -Protocol Http -HostName "contoso.com" -Path "/path/path.htm" -Interval 30 -Timeout 120 -UnhealthyThreshold 8


### 步骤 4

为后端池中的流量配置应用程序网关设置“poolsetting01”。此步骤还包括针对应用程序网关请求配置后端池响应超时。当后端响应达到超时限制时，应用程序网关将取消请求。这与仅适用于探测检查的后端响应的探测超时不同。

	$poolSetting = New-AzureRmApplicationGatewayBackendHttpSettings -Name poolsetting01 -Port 80 -Protocol Http -CookieBasedAffinity Disabled -Probe $probe -RequestTimeout 80


### 步骤 5

为公共 IP 终结点配置名为“frontendport01”的前端 IP 端口。

	$fp = New-AzureRmApplicationGatewayFrontendPort -Name frontendport01  -Port 80

### 步骤 6

创建名为“fipconfig01”的前端 IP 配置，并将公共 IP 地址与前端 IP 配置相关联。


	$fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name fipconfig01 -PublicIPAddress $publicip

### 步骤 7

创建名为“listener01”的侦听器，并将前端端口与前端 IP 配置相关联。

	$listener = New-AzureRmApplicationGatewayHttpListener -Name listener01  -Protocol Http -FrontendIPConfiguration $fipconfig -FrontendPort $fp

### 步骤 8

创建名为“rule01”的负载平衡器路由规则，并配置负载平衡器的行为。

	$rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool

### 步骤 9

配置应用程序网关的实例大小。

	$sku = New-AzureRmApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2


>[AZURE.NOTE]  *InstanceCount* 的默认值为 2，最大值为 10。 *GatewaySize* 的默认值为 Medium。你可以在 Standard\_Small、Standard\_Medium 和 Standard\_Large 之间进行选择。

## 使用 New-AzureRmApplicationGateway 创建应用程序网关

创建包含上述步骤中所有配置项的应用程序网关。示例中的应用程序网关名为“appgwtest”。

	$appgw = New-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Location "China North" -BackendAddressPools $pool -Probes $probe -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku

## 将探测添加到现有应用程序网关

执行以下四个步骤可将自定义探测添加到现有应用程序网关。

### 步骤 1

使用 **Get-AzureRmApplicationGateway** 将应用程序网关资源载入 PowerShell 变量。

	$getgw =  Get-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg

### 步骤 2

将探测添加到现有网关配置。

	$probe = Add-AzureRmApplicationGatewayProbeConfig -ApplicationGateway $getgw -Name probe01 -Protocol Http -HostName "contoso.com" -Path "/path/custompath.htm" -Interval 30 -Timeout 120 -UnhealthyThreshold 8


在本示例中，自定义探测已配置为每隔 30 秒检查 URL 路径 contoso.com/path/custompath.htm。配置的超时阈值为 120 秒，最多只能有 8 个失败的探测请求。

### 步骤 3

使用 **-Set-AzureRmApplicationGatewayBackendHttpSettings** 将探测添加到后端池设置配置和超时。


	 $getgw = Set-AzureRmApplicationGatewayBackendHttpSettings -ApplicationGateway $getgw -Name $getgw.BackendHttpSettingsCollection.name -Port 80 -Protocol Http -CookieBasedAffinity Disabled -Probe $probe -RequestTimeout 120

### 步骤 4

使用 **Set-AzureRmApplicationGateway** 将配置保存到应用程序网关。

	Set-AzureRmApplicationGateway -ApplicationGateway $getgw -verbose

## 从现有应用程序网关中删除探测

以下是从现有应用程序网关中删除自定义探测的步骤。

### 步骤 1

使用 **Get-AzureRmApplicationGateway** 将应用程序网关资源载入 PowerShell 变量。

	$getgw =  Get-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg


### 步骤 2

使用 **Remove-AzureRmApplicationGatewayProbeConfig** 从应用程序网关中删除探测配置。

	$getgw = Remove-AzureRmApplicationGatewayProbeConfig -ApplicationGateway $getgw -Name $getgw.Probes.name

### 步骤 3

使用 **-Set-AzureRmApplicationGatewayBackendHttpSettings** 更新后端池设置，以删除探测与超时设置。


	 $getgw=Set-AzureRmApplicationGatewayBackendHttpSettings -ApplicationGateway $getgw -Name $getgw.BackendHttpSettingsCollection.name -Port 80 -Protocol http -CookieBasedAffinity Disabled

### 步骤 4

使用 **Set-AzureRmApplicationGateway** 将配置保存到应用程序网关。

	Set-AzureRmApplicationGateway -ApplicationGateway $getgw -verbose

<!---HONumber=Mooncake_0215_2016-->
