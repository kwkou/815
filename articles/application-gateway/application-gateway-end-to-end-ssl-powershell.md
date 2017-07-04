<properties
    pageTitle="通过应用程序网关配置 SSL 策略和端到端 SSL | Azure"
    description="本文介绍如何使用 Azure Resource Manager PowerShell 通过应用程序网关配置端到端 SSL"
    services="application-gateway"
    documentationcenter="na"
    author="georgewallace"
    manager="carmonm"
    editor="tysonn" />  

<tags
    ms.assetid="e6d80a33-4047-4538-8c83-e88876c8834e"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="12/14/2016"
    wacn.date="02/21/2017"
    ms.author="gwallace" />  


# 使用 PowerShell 通过应用程序网关配置 SSL 策略和端到端 SSL

## 概述

应用程序网关支持对流量进行端到端加密。应用程序网关通过在应用程序网关上终止 SSL 连接来完成此任务。网关随后将路由规则应用于流量、重新加密数据包，并根据定义的路由规则将数据包转发到适当的后端。来自 Web 服务器的任何响应都会经历相同的过程返回最终用户。

应用程序网关支持的另一个功能是禁用特定 SSL 协议版本。应用程序网关支持禁用以下协议版本：**TLSv1.0**、**TLSv1.1** 和 **TLSv1.2**。

> [AZURE.NOTE]
SSL 2.0 和 SSL 3.0 默认处于禁用状态且无法启用。这些版本被视为不安全的版本，不能用于应用程序网关
> 
> 

![方案图像][scenario]  


## 方案

在此方案中，可学习如何使用 PowerShell 创建使用端到端 SSL 的应用程序网关。

此方案将：

* 创建名为“appgw-rg”的资源组
* 创建名为“appgwvnet”且包含 10.0.0.0/16 保留 CIDR 块的虚拟网络。
* 创建名为“appgwsubnet”和“appsubnet”的两个子网。
* 创建支持端到端 SSL 加密并禁用特定 SSL 协议的小型应用程序网关。

## 开始之前

若要对应用程序网关配置端到端 SSL，需要网关证书和后端服务器证书。网关证书用于加密和解密通过 SSL 发送给网关的流量。网关证书需要采用个人信息交换 (pfx) 格式。此文件格式适用于导出私钥，后者是应用程序网关对流量进行加解密所必需的。

若要加密端到端 SSL，后端必须已加入应用程序网关的白名单。将后端的公用证书上载到应用程序网关即可完成此操作。这可确保应用程序网关仅与已知的后端实例通信，从而进一步保护端到端通信。

以下步骤介绍此过程：

## 创建资源组

本部分指导创建资源组，其中包含应用程序网关。

### 步骤 1

登录 Azure 帐户。

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

### 步骤 2

选择要用于此方案的订阅。

    Select-AzureRmsubscription -SubscriptionName "<Subscription name>"

### 步骤 3

创建资源组（如果要使用现有的资源组，请跳过此步骤）。

    New-AzureRmResourceGroup -Name appgw-rg -Location "China North"

## 为应用程序网关创建虚拟网络和子网

以下示例创建一个虚拟网络和两个子网。一个子网用于托管应用程序网关。另一个子网用于托管 Web 应用程序的后端。

### 步骤 1

分配要用于应用程序网关本身的子网地址范围。

    $gwSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name 'appgwsubnet' -AddressPrefix 10.0.0.0/24

> [AZURE.NOTE]
应适当调整为应用程序网关配置的子网的大小。最多可以为 10 个实例配置应用程序网关。每个实例从子网获取 1 个 IP 地址。子网太小可能会对应用程序网关的向外缩放造成负面影响。
> 
> 

### 步骤 2

分配要用于后端地址池的地址范围。

    $nicSubnet = New-AzureRmVirtualNetworkSubnetConfig  -Name 'appsubnet' -AddressPrefix 10.0.2.0/24

### 步骤 3

创建具有上述步骤中定义的子网的虚拟网络。

    $vnet = New-AzureRmvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg -Location "China North" -AddressPrefix 10.0.0.0/16 -Subnet $gwSubnet, $nicSubnet

### 步骤 4

检索要用于以下步骤的虚拟网络资源和子网资源：

    $vnet = Get-AzureRmvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg
    $gwSubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'appgwsubnet' -VirtualNetwork $vnet
    $nicSubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'appsubnet' -VirtualNetwork $vnet

## 创建前端配置的公共 IP 地址

创建要用于应用程序网关的公共 IP 资源。此公共 IP 地址会用于以后的步骤。

    $publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-rg -Name 'publicIP01' -Location "China North" -AllocationMethod Dynamic

> [AZURE.IMPORTANT]
> 应用程序网关不支持使用通过定义的域标签创建的公共 IP 地址。仅支持具有动态创建的域标签的公共 IP 地址。如果需要应用程序网关具有友好的 DNS 名称，建议使用 CNAME 记录作为别名。
> 
> 

## 创建应用程序网关配置对象

在创建应用程序网关前，必须设置所有配置项。以下步骤将创建应用程序网关资源所需的配置项。

### 步骤 1

创建应用程序网关 IP 配置，此设置配置应用程序网关使用的子网。应用程序网关启动时，它会从配置的子网获取 IP 地址，再将网络流量路由到后端 IP 池中的 IP 地址。请记住，每个实例需要一个 IP 地址。

    $gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name 'gwconfig' -Subnet $gwSubnet

### 步骤 2

创建前端 IP 配置，此设置将专用或公共 IP 地址映射到应用程序网关的前端。以下步骤将上述步骤中的公共 IP 地址与前端 IP 配置关联。

    $fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name 'fip01' -PublicIPAddress $publicip

### 步骤 3

使用后端 Web 服务器的 IP 地址配置后端 IP 地址池。这些 IP 地址是接收来自前端 IP 终结点的网络流量的 IP 地址。替换以下 IP 地址，添加自己的应用程序 IP 地址终结点。

    $pool = New-AzureRmApplicationGatewayBackendAddressPool -Name 'pool01' -BackendIPAddresses 1.1.1.1, 2.2.2.2, 3.3.3.3

> [AZURE.NOTE]
完全限定的域名 (FQDN) 也是可以通过 -BackendFqdns 开关替换后端服务器 IP 地址的有效值。
> 
> 

### 步骤 4

配置公共 IP 终结点的前端 IP 端口。此端口是最终用户连接到的端口。

    $fp = New-AzureRmApplicationGatewayFrontendPort -Name 'port01'  -Port 443

### 步骤 5

配置应用程序网关的证书。此证书用于加密和解密应用程序网关上的流量。

	$cert = New-AzureRmApplicationGatewaySslCertificate -Name cert01 -CertificateFile <full path to .pfx file> -Password <password for certificate file>

> [AZURE.NOTE]
此示例配置用于 SSL 连接的证书。该证书需采用 .pfx 格式，并且密码必须为 4 到 12 个字符。
> 
> 

### 步骤 6

创建应用程序网关的 HTTP 侦听器。分配要使用的前端 IP 配置、端口和 SSL 证书。

	$listener = New-AzureRmApplicationGatewayHttpListener -Name listener01 -Protocol Https -FrontendIPConfiguration $fipconfig -FrontendPort $fp -SslCertificate $cert

### 步骤 7

上传要在已启用 SSL 的后端池资源上使用的证书。

> [AZURE.NOTE]
默认探测从后端的 IP 地址上的**默认** SSL 绑定获取公钥，并将其收到的公钥值与用户在此处提供的公钥值进行比较。**如果**用户使用后端 的主机标头和 SNI，则检索到的公钥不一定是预期会将流量传输到其中的站点。如果有疑问，请访问后端的 https://127.0.0.1/，确认针对**默认** SSL 绑定所使用的证书。本部分使用该请求中的公钥。如果使用了 HTTPS 绑定上的主机标头和 SNI，但在手动向后端的 https://127.0.0.1/ 发送浏览器请求时没有收到响应和证书，则必须在后端设置默认 SSL 绑定。如果不这样做，探测会失败，系统就不会将后端列入允许名单。
> 
> 

    $authcert = New-AzureRmApplicationGatewayAuthenticationCertificate -Name 'whitelistcert1' -CertificateFile C:\users\gwallace\Desktop\cert.cer

> [AZURE.NOTE]
此步骤中提供的证书应该是后端中存在的 pfx 证书的公钥。以 .CER 格式导出后端服务器上安装的证书（不是根证书），将其用在此步骤。此步骤会将后端加入应用程序网关的白名单。
> 
> 

### 步骤 8

配置应用程序网关后端 http 设置。将上述步骤中上传的证书分配给 http 设置。

    $poolSetting = New-AzureRmApplicationGatewayBackendHttpSettings -Name 'setting01' -Port 443 -Protocol Https -CookieBasedAffinity Enabled -AuthenticationCertificates $authcert

### 步骤 9

创建配置负载均衡器行为的负载均衡器路由规则。在此示例中，创建基本轮循机制规则。

    $rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name 'rule01' -RuleType basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool

### 步骤 10

配置应用程序网关的实例大小。可用大小为 **Standard\_Small**、**Standard\_Medium** 和 **Standard\_Large**。对于容量，可用值为 1 到 10。

    $sku = New-AzureRmApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2

> [AZURE.NOTE]
进行测试时，可以选择 1 作为实例计数。必须知道的是，2 以下的实例计数不受 SLA 支持，因此不建议使用。小型网关用于开发/测试，不用于生产。
> 
> 

### 步骤 11

配置要在应用程序网关上使用的 SSL 策略。应用程序网关支持禁用特定 SSL 协议版本的功能。

以下值是可以禁用的协议版本的列表。

* **TLSv1\_0**
* **TLSv1\_1**
* **TLSv1\_2**

以下示例禁用 **TLSv1\_0**。

    $sslPolicy = New-AzureRmApplicationGatewaySslPolicy -DisabledSslProtocols TLSv1_0

## 创建应用程序网关

使用上述所有步骤创建应用程序网关。创建网关是需要长时间运行的过程。

    $appgw = New-AzureRmApplicationGateway -Name appgateway -SslCertificates $cert -ResourceGroupName "appgw-rg" -Location "China North" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku -SslPolicy $sslPolicy -AuthenticationCertificates $authcert -Verbose

## 禁用现有应用程序网关上的 SSL 协议版本

上述步骤指导创建具有端到端 SSL 并禁用特定 SSL 协议版本的应用程序。以下示例禁用现有应用程序网关上的特定 SSL 策略。

### 步骤 1

检索要更新的应用程序网关。

    $gw = Get-AzureRmApplicationGateway -Name AdatumAppGateway -ResourceGroupName AdatumAppGatewayRG

### 步骤 2

定义 SSL 策略。在以下示例中，禁用了 TLSv1.0 和 TLSv1.1。

    Set-AzureRmApplicationGatewaySslPolicy -DisabledSslProtocols TLSv1_0, TLSv1_1 -ApplicationGateway $gw

### 步骤 3

最后，更新网关。请务必注意，这个最后一步是需要长时间运行的任务。完成时，应用程序网关上即已配置端到端 SSL。

    $gw | Set-AzureRmApplicationGateway

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

请参阅 [Web 应用程序防火墙概述](/documentation/articles/application-gateway-web-application-firewall-overview/)，了解如何通过应用程序网关的 Web 应用程序防火墙强化 Web 应用程序的安全

[scenario]: ./media/application-gateway-end-to-end-ssl-powershell/scenario.png

<!---HONumber=Mooncake_1128_2016-->