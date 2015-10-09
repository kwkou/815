<properties 
   pageTitle="使用 Azure 资源管理器创建、启动或删除应用程序网关 | Windows Azure"
   description="本页提供有关使用 Azure 资源管理器创建、配置、启动和删除 Azure 应用程序网关的说明"
   documentationCenter="na"
   services="application-gateway"
   authors="joaoma"
   manager="jdial"
   editor="tysonn"/>
<tags 
   ms.service="application-gateway"
   ms.date="08/07/2015"
   wacn.date="09/15/2015"/>


# 使用 Azure 资源管理器创建、启动或删除应用程序网关

> [AZURE.SELECTOR]
- [Azure classic steps](/documentation/articles/application-gateway-create-gateway)
- [Resource Manager Powershell steps](/documentation/articles/application-gateway-create-gateway-arm)

在此版本中，你可以使用 PowerShell 或 REST API 调用创建应用程序网关。即将推出的版本将会提供门户和 CLI 支持。本文将指导你完成创建、配置、启动和删除应用程序网关的步骤。

## 开始之前

1. 使用 Web 平台安装程序安装最新版本的 Azure PowerShell cmdlet。可以从[下载页面](/downloads/)的“Windows PowerShell”部分下载并安装最新版本。
2. 你将为应用程序网关创建虚拟网络和子网。请确保没有虚拟机或云部署正在使用子网。应用程序网关必须单独位于虚拟网络子网中。
3. 要配置为使用应用程序网关的服务器必须存在，或者在虚拟网络中为其创建终结点，或者为其分配公共 IP/VIP。

## 创建应用程序网关需要什么？
 

- **后端服务器池：**后端服务器的 IP 地址列表。列出的 IP 地址应属于虚拟网络子网，或者是公共 IP/VIP。 
- **后端服务器池设置：**每个池都有一些设置，例如端口、协议和基于 Cookie 的关联性。这些设置绑定到池，并会应用到池中的所有服务器。
- **前端端口：**此端口是应用程序网关上打开的公共端口。客户流量将抵达此端口，然后重定向到后端服务器之一。
- **侦听器：**侦听器具有前端端口、协议（Http 或 Https，区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。 
- **规则：**规则将会绑定侦听器和后端服务器池，并定义当流量抵达特定侦听器时应定向到的后端服务器池。目前仅支持*基本*规则。*基本*规则是一种轮循负载分发模式。


 
## 创建新的应用程序网关

使用 Azure 经典门户和 Azure 资源管理器的差别在于创建应用程序网关的顺序和需要配置的项。

使用资源管理器，组成应用程序网关的所有项都将分开配置，然后放在一起创建应用程序网关资源。


以下是创建应用程序网关所要执行的步骤：

1. 创建资源管理器的资源组
2. 创建应用程序网关的虚拟网络、子网和公共 IP
3. 创建应用程序网关配置对象
4. 创建应用程序网关资源


## 创建资源管理器的资源组

确保将 PowerShell 模式切换为使用 ARM cmdlet。[将 Windows PowerShell 与资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager)中提供了详细信息。

### 步骤 1

    Switch-AzureMode -Name AzureResourceManager

### 步骤 2

登录到你的 Azure 帐户。


    Add-AzureAccount

系统将提示你使用凭据进行身份验证。


### 步骤 3

选择要使用的 Azure 订阅。

    Select-AzureSubscription -SubscriptionName "MySubscription"

若要查看可用订阅的列表，请使用“Get-AzureSubscription”cmdlet。


### 步骤 4

创建新的资源组（如果要使用现有的资源组，请跳过此步骤）

    New-AzureResourceGroup -Name appgw-rg -location "China North"

Azure 资源管理器要求所有资源组指定一个位置。此位置将用作该资源组中的资源的默认位置。请确保用于创建应用程序网关的所有命令都使用相同的资源组。

在上面的示例中，我们在位置“中国北部”创建了名为“appgw-RG”的资源组。

## 为应用程序网关创建虚拟网络和子网

以下示例演示如何使用资源管理器创建虚拟网络：

### 步骤 1	
	
	$subnet = New-AzureVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24

将地址范围 10.0.0.0/24 分配给用于创建虚拟网络的子网变量

### 步骤 2	
	$vnet = New-AzurevirtualNetwork -Name appgwvnet -ResourceGroupName appgw-rg -Location "China North" -AddressPrefix 10.0.0.0/16 -Subnet $subnet

使用前缀 10.0.0.0/16 和子网 10.0.0.0/24，在中国北部区域的“appw-rg”资源组中创建名为“appgwvnet”的虚拟网络
	
## 创建前端配置的公共 IP 地址

	$publicip = New-AzurePublicIpAddress -ResourceGroupName appgw-rg -name publicIP01 -location "China North" -AllocationMethod Dynamic

在中国北部区域的“appw-rg”资源组中创建公共 IP 资源“publicIP01”。


## 创建应用程序网关配置对象

### 步骤 1

	$gipconfig = New-AzureApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet

创建名为“gatewayIP01”的应用程序网关 IP 配置。当应用程序网关启动时，它会从配置的子网获取 IP 地址，再将网络流量路由到后端 IP 池中的 IP 地址。请记住，每个实例需要一个 IP 地址。
 
### 步骤 2

	$pool = New-AzureApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 134.170.185.46, 134.170.188.221,134.170.185.50

此步骤将配置名为“pool01”的后端 IP 地址池，其 IP 地址为“134.170.185.46、134.170.188.221、134.170.185.50”。这些 IP 地址将接收来自前端 IP 终结点的网络流量。你要替换上述 IP 地址，添加你自己的应用程序 IP 地址终结点。

### 步骤 3

	$poolSetting = New-AzureApplicationGatewayBackendHttpSettings -Name poolsetting01 -Port 80 -Protocol Http -CookieBasedAffinity Disabled

为后端池中负载平衡的网络流量配置应用程序网关设置“poolsetting01”。

### 步骤 4

	$fp = New-AzureApplicationGatewayFrontendPort -Name frontendport01  -Port 80

在本例中，为公共 IP 终结点配置名为“frontendport01”的前端 IP 端口。

### 步骤 5

	$fipconfig = New-AzureApplicationGatewayFrontendIPConfig -Name fipconfig01 -PublicIPAddress $publicip

创建名为“fipconfig01”的前端 IP 配置，并将公共 IP 地址与前端 IP 配置相关联。

### 步骤 6

	$listener = New-AzureApplicationGatewayHttpListener -Name listener01  -Protocol Http -FrontendIPConfiguration $fipconfig -FrontendPort $fp

创建名为“listener01”的侦听器，并将前端端口与前端 IP 配置相关联。

### 步骤 7 

	$rule = New-AzureApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool

创建名为“rule01”的负载平衡器路由规则，并配置负载平衡器的行为。

### 步骤 8

	$sku = New-AzureApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2

配置应用程序网关的实例大小

>[AZURE.NOTE]*InstanceCount* 的默认值为 2，最大值为 10。*GatewaySize* 的默认值为 Medium。你可以在 Standard\_Small、Standard\_Medium 和 Standard\_Large 之间进行选择。

## 使用 New-AzureApplicationGateway 创建应用程序网关

	$appgw = New-AzureApplicationGateway -Name appgwtest -ResourceGroupName appw-rg -Location "China North" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku

创建包含上述步骤中所有配置项的应用程序网关。示例中的应用程序网关名为“appgwtest”。


## 启动应用程序网关

配置网关后，使用 `Start-AzureApplicationGateway` cmdlet 来启动网关。成功启动网关后，将开始计收应用程序网关的费用。


**注意：**`Start-AzureApplicationGateway` cmdlet 最多可能需要 15 到 20 分钟才能完成。

在以下示例中，应用程序网关名为“appgwtest”，资源组为“app-rg”：


### 步骤 1

获取应用程序网关对象，并将其关联到变量“$getgw”：
 
	$getgw =  Get-AzureApplicationGateway -Name appgwtest -ResourceGroupName app-rg

### 步骤 2
	 
使用 `Start-AzureApplicationGateway` 启动应用程序网关：

	 Start-AzureApplicationGateway -ApplicationGateway $getgw  

	

## 验证应用程序网关状态

使用 `Get-AzureApplicationGateway` cmdlet 检查网关的状态。如果前一步骤中的 *Start-AzureApplicationGateway* 成功，则 State 应为 *Running*，Vip 和 DnsName 应包含有效的条目。

此示例演示了一个正常运行并已准备好将流量定向到 `http://<generated-dns-name>.cloudapp.net` 的应用程序网关。

	Get-AzureApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg

	Sku                               : Microsoft.Azure.Commands.Network.Models.PSApplicationGatewaySku
	GatewayIPConfigurations           : {gatewayip01}
	SslCertificates                   : {}
	FrontendIPConfigurations          : {frontendip01}
	FrontendPorts                     : {frontendport01}
	BackendAddressPools               : {pool01}
	BackendHttpSettingsCollection     : {setting01}
	HttpListeners                     : {listener01}
	RequestRoutingRules               : {rule01}
	OperationalState                  : 
	ProvisioningState                 : Succeeded
	GatewayIpConfigurationsText       : [
                                      {
                                        "Subnet": {
                                          "Id": "/subscriptions/###############################/resourceGroups/appgw-rg
                                    /providers/Microsoft.Network/virtualNetworks/vnet01/subnets/subnet01"
                                        },
                                        "ProvisioningState": "Succeeded",
                                        "Name": "gatewayip01",
                                        "Etag": "W/"ddb0408e-a54c-4501-a7f8-8487c3530bd7"",
                                        "Id": "/subscriptions/###############################/resourceGroups/appgw-rg/p
                                    roviders/Microsoft.Network/applicationGateways/appgwtest/gatewayIPConfigurations/gatewayip
                                    01"
                                      }
                                    ]
	SslCertificatesText               : []
	FrontendIpConfigurationsText      : [
                                      {
                                        "PrivateIPAddress": null,
                                        "PrivateIPAllocationMethod": "Dynamic",
                                        "Subnet": null,
                                        "PublicIPAddress": {
                                          "Id": "/subscriptions/###############################/resourceGroups/appgw-rg
                                    /providers/Microsoft.Network/publicIPAddresses/publicip01"
                                        },
                                        "ProvisioningState": "Succeeded",
                                        "Name": "frontendip01",
                                        "Etag": "W/"ddb0408e-a54c-4501-a7f8-8487c3530bd7"",
                                        "Id": "/subscriptions/###############################/resourceGroups/appgw-rg/p
                                    roviders/Microsoft.Network/applicationGateways/appgwtest/frontendIPConfigurations/frontend
                                    ip01"
                                      }
                                    ]
	FrontendPortsText                 : [
                                      {
                                        "Port": 80,
                                        "ProvisioningState": "Succeeded",
                                        "Name": "frontendport01",
                                        "Etag": "W/"ddb0408e-a54c-4501-a7f8-8487c3530bd7"",
                                        "Id": "/subscriptions/###############################/resourceGroups/appgw-rg/p
                                    roviders/Microsoft.Network/applicationGateways/appgwtest/frontendPorts/frontendport01"
                                      }
                                    ]
	BackendAddressPoolsText           : [
                                      {
                                        "BackendAddresses": [
                                          {
                                            "Fqdn": null,
                                            "IpAddress": "134.170.185.46"
                                          },
                                          {
                                            "Fqdn": null,
                                            "IpAddress": "134.170.188.221"
                                          },
                                          {
                                            "Fqdn": null,
                                            "IpAddress": "134.170.185.50"
                                          }
                                        ],
                                        "BackendIpConfigurations": [],
                                        "ProvisioningState": "Succeeded",
                                        "Name": "pool01",
                                        "Etag": "W/"ddb0408e-a54c-4501-a7f8-8487c3530bd7"",
                                        "Id": "/subscriptions/###############################/resourceGroups/appgw-rg/p
                                    roviders/Microsoft.Network/applicationGateways/appgwtest/backendAddressPools/pool01"
                                      }
                                    ]
	BackendHttpSettingsCollectionText : [
                                      {
                                        "Port": 80,
                                        "Protocol": "Http",
                                        "CookieBasedAffinity": "Disabled",
                                        "ProvisioningState": "Succeeded",
                                        "Name": "setting01",
                                        "Etag": "W/"ddb0408e-a54c-4501-a7f8-8487c3530bd7"",
                                        "Id": "/subscriptions/###############################/resourceGroups/appgw-rg/p
                                    roviders/Microsoft.Network/applicationGateways/appgwtest/backendHttpSettingsCollection/set
                                    ting01"
                                      }
                                    ]
	HttpListenersText                 : [
                                      {
                                        "FrontendIpConfiguration": {
                                          "Id": "/subscriptions/###############################/resourceGroups/appgw-rg
                                    /providers/Microsoft.Network/applicationGateways/appgwtest/frontendIPConfigurations/fronte
                                    ndip01"
                                        },
                                        "FrontendPort": {
                                          "Id": "/subscriptions/###############################/resourceGroups/appgw-rg
                                    /providers/Microsoft.Network/applicationGateways/appgwtest/frontendPorts/frontendport01"
                                        },
                                        "Protocol": "Http",
                                        "SslCertificate": null,
                                        "ProvisioningState": "Succeeded",
                                        "Name": "listener01",
                                        "Etag": "W/"ddb0408e-a54c-4501-a7f8-8487c3530bd7"",
                                        "Id": "/subscriptions/###############################/resourceGroups/appgw-rg/p
                                    roviders/Microsoft.Network/applicationGateways/appgwtest/httpListeners/listener01"
                                      }
                                    ]
	RequestRoutingRulesText           : [
                                      {
                                        "RuleType": "Basic",
                                        "BackendAddressPool": {
                                          "Id": "/subscriptions/###############################/resourceGroups/appgw-rg
                                    /providers/Microsoft.Network/applicationGateways/appgwtest/backendAddressPools/pool01"
                                        },
                                        "BackendHttpSettings": {
                                          "Id": "/subscriptions/###############################/resourceGroups/appgw-rg
                                    /providers/Microsoft.Network/applicationGateways/appgwtest/backendHttpSettingsCollection/s
                                    etting01"
                                        },
                                        "HttpListener": {
                                          "Id": "/subscriptions/###############################/resourceGroups/appgw-rg
                                    /providers/Microsoft.Network/applicationGateways/appgwtest/httpListeners/listener01"
                                        },
                                        "ProvisioningState": "Succeeded",
                                        "Name": "rule01",
                                        "Etag": "W/"ddb0408e-a54c-4501-a7f8-8487c3530bd7"",
                                        "Id": "/subscriptions/###############################/resourceGroups/appgw-rg/p
                                    roviders/Microsoft.Network/applicationGateways/appgwtest/requestRoutingRules/rule01"
                                      }
                                    ]
	ResourceGroupName                 : appgw-rg
	Location                          : chinanorth
		Tag                               : {}
	TagsTable                         : 
	Name                              : appgwtest
	Etag                              : W/"ddb0408e-a54c-4501-a7f8-8487c3530bd7"
	Id                                : /subscriptions/###############################/resourceGroups/appgw-rg/providers/Microsoft.Network/applicationGateways/appgwtest




## 删除应用程序网关

若要删除应用程序网关，需要按顺序执行以下操作：

1. 使用 `Stop-AzureApplicationGateway` cmdlet 停止该网关。 
2. 使用 `Remove-AzureApplicationGateway` cmdlet 删除该网关。
3. 使用 `Get-AzureApplicationGateway` cmdlet 验证是否已删除该网关。


### 步骤 1

获取应用程序网关对象，并将其关联到变量“$getgw”：
 
	$getgw =  Get-AzureApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg

### 步骤 2
	 
使用 `Stop-AzureApplicationGateway` 停止应用程序网关：

	Stop-AzureApplicationGateway -ApplicationGateway $getgw  


应用程序网关进入停止状态后，请使用 `Remove-AzureApplicationGateway` cmdlet 删除该服务。


	Remove-AzureApplicationGateway -Name $appgwtest -ResourceGroupName appgw-rg -Force

	

>[AZURE.NOTE]可以使用“-force”开关来抑制删除确认消息
>

若要验证是否已删除服务，可以使用 `Get-AzureApplicationGateway` cmdlet。此步骤不是必需的。


	Get-AzureApplicationGateway -Name appgwtest-ResourceGroupName appgw-rg

	


## 后续步骤

如果你要配置 SSL 卸载，请参阅[配置应用程序网关以进行 SSL 卸载](/documentation/articles/application-gateway-ssl)。

如果你想要将应用程序网关配置为与 ILB 配合使用，请参阅[创建具有内部负载平衡器 (ILB) 的应用程序网关](/documentation/articles/application-gateway-ilb)。

如需负载平衡选项的其他常规信息，请参阅：

<!--- [Azure Load Balancer](https://azure.microsoft.com/documentation/services/load-balancer/)-->
- [Azure 流量管理器](/documentation/services/traffic-manager/)

<!---HONumber=69-->