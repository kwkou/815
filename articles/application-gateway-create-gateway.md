<properties 
   pageTitle="创建、启动或删除应用程序网关 | Windows Azure"
   description="此页提供有关创建、配置、启动和删除 Azure 应用程序网关的说明"
   documentationCenter="na"
   services="application-gateway"
   authors="cherylmc"
   manager="jdial"
   editor="tysonn"/>
<tags 
   ms.service="application-gateway" 
   ms.date="06/23/2015"
   wacn.date="07/19/2015"/>

# 创建、启动或删除应用程序网关

在此版本中，你可以使用 PowerShell 或 REST API 调用创建应用程序网关。即将推出的版本将会提供门户和 CLI 支持。

本文将指导你完成创建、配置、启动和删除应用程序网关的步骤。

## 开始之前

1. 使用 Web 平台安装程序安装最新版本的 Azure PowerShell cmdlet。可以从[下载页面](/downloads)的 **Windows PowerShell** 部分下载和安装最新的 PowerShell cmdlet。
2. 请确认你已创建包含有效子网、可正常运行的虚拟网络。
3. 请确认后端服务器位于虚拟网络中，或者为后端服务器分配了公共 IP/VIP。


若要创建新的应用程序网关，请按所列顺序执行以下步骤。步骤的下面是你要使用的每个 cmdlet 的示例。

1. [创建新的应用程序网关](#create-a-new-application-gateway)
2. [配置网关](#configure-the-application-gateway)
3. [设置网关](#set-the-application-gateway)
4. [启动网关](#start-the-application-gateway)
4. [验证网关](#verify-the-application-gateway-status)

如果你要删除应用程序网关，请转到[删除应用程序网关](#delete-an-application-gateway)。

## 创建新的应用程序网关

**若要创建网关**，请使用 `New-AzureApplicationGateway` cmdlet，并将值替换为你自己的值。

此示例在第一行显示 cmdlet，接着显示输出。
    
	PS C:\> New-AzureApplicationGateway -Name AppGwTest -VnetName testvnet1 -Subnets @("Subnet-1")

	VERBOSE: 4:31:35 PM - Begin Operation: New-AzureApplicationGateway 
	VERBOSE: 4:32:37 PM - Completed Operation: New-AzureApplicationGateway
	Name       HTTP Status Code     Operation ID                             Error 
	----       ----------------     ------------                             ----
	Successful OK                   55ef0460-825d-2981-ad20-b9a8af41b399

**若要验证**是否已创建网关，可以使用 `Get-AzureApplicationGateway` cmdlet。

请注意，在此示例中，*Description*、*InstanceCount* 和 *GatewaySize* 是可选参数。*InstanceCount* 的默认值为 2，最大值为 10。*GatewaySize* 的默认值为 Medium。其他可用值为 Small 和 Large。*Vip* 和 *DnsName* 显示为空白，因为网关尚未启动。这些值将在网关进入运行状态后立即创建。此时不会开始计收应用程序网关的费用。创建网关后，将开始计费。

此示例在第一行显示 cmdlet，接着显示输出。

	PS C:\> Get-AzureApplicationGateway AppGwTest
	Name          : AppGwTest
	Description   : 
	VnetName      : testvnet1
	Subnets       : {Subnet-1}
	InstanceCount : 2
	GatewaySize   : Medium
	State         : Stopped
	VirtualIPs    : {}
	DnsName       :


## 配置应用程序网关

应用程序网关配置包括多个值，可将这些值绑定在一起以构造配置。

有效值为：

- **后端服务器池：**后端服务器的 IP 地址列表。此 IP 应属于 VNET 子网，或者应为公共 IP/VIP。 
- **后端服务器池设置：**每个池具有端口、协议和基于 Cookie 的相关性等设置。这些设置绑定到池，并会应用到池中的所有服务器。
- **前端端口：**此端口是应用程序网关上打开的公共端口。客户流量将抵达此端口，然后重定向到后端服务器之一。
- **侦听器：**侦听器具有前端端口、协议（Http 或 Https，区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。 
- **规则：**规则将会绑定侦听器和后端服务器池，并定义当流量抵达特定侦听器时应定向到的后端服务器池。目前仅支持“基本”规则，即轮循负载分发。可以通过创建配置对象或使用配置 XML 文件来构造配置。 

若要使用配置 XML 文件构造配置，请使用以下示例。

### 配置 XML 示例

	<?xml version="1.0" encoding="utf-8"?>
	<ApplicationGatewayConfiguration xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/windowsazure">
	    <FrontendPorts>
	        <FrontendPort>
	            <Name>FrontendPort1</Name>
	            <Port>80</Port>
	        </FrontendPort>
	    </FrontendPorts>
	    <BackendAddressPools>
	        <BackendAddressPool>
	            <Name>BackendPool1</Name>
	            <IPAddresses>
	                <IPAddress>10.0.0.1</IPAddress>
	                <IPAddress>10.0.0.2</IPAddress>
	            </IPAddresses>
	        </BackendAddressPool>
	    </BackendAddressPools>
	    <BackendHttpSettingsList>
	        <BackendHttpSettings>
	            <Name>BackendSetting1</Name>
	            <Port>80</Port>
	            <Protocol>Http</Protocol>
	            <CookieBasedAffinity>Enabled</CookieBasedAffinity>
	        </BackendHttpSettings>
	    </BackendHttpSettingsList>
	    <HttpListeners>
	        <HttpListener>
	            <Name>HTTPListener1</Name>
	            <FrontendPort>FrontendPort1</FrontendPort>
	            <Protocol>Http</Protocol>
	        </HttpListener>
	    </HttpListeners>
	    <HttpLoadBalancingRules>
	        <HttpLoadBalancingRule>
	            <Name>HttpLBRule1</Name>
	            <Type>basic</Type>
	            <BackendHttpSettings>BackendSetting1</BackendHttpSettings>
	            <Listener>HTTPListener1</Listener>
	            <BackendAddressPool>BackendPool1</BackendAddressPool>
	        </HttpLoadBalancingRule>
	    </HttpLoadBalancingRules>
	</ApplicationGatewayConfiguration>

## 设置应用程序网关

将 `Set-AzureApplicationGatewayConfig` cmdlet 与配置对象或配置 XML 文件配合使用，并将值替换为你自己的值。

此示例在第一行显示 cmdlet，接着显示输出。

	PS C:\> Set-AzureApplicationGatewayConfig -Name AppGwTest -ConfigFile D:\config.xml

	VERBOSE: 7:54:59 PM - Begin Operation: Set-AzureApplicationGatewayConfig 
	VERBOSE: 7:55:32 PM - Completed Operation: Set-AzureApplicationGatewayConfig
	Name       HTTP Status Code     Operation ID                             Error 
	----       ----------------     ------------                             ----
	Successful OK                   9b995a09-66fe-2944-8b67-9bb04fcccb9d

## 启动应用程序网关

配置网关后，发出 `Start-AzureApplicationGateway` cmdlet 来启动网关。这是用于设置网关的 cmdlet。成功运行该 cmdlet 并设置网关后，也会开始计费。


**注意** `Start-AzureApplicationGateway` cmdlet 最长可能需要 15-20 分钟才能完成。成功启动网关后，应用程序网关计费将会开始。

此示例在第一行显示 cmdlet，接着显示输出。将示例中的值替换为你自己的值。


	PS C:\> Start-AzureApplicationGateway AppGwTest 

	VERBOSE: 7:59:16 PM - Begin Operation: Start-AzureApplicationGateway 
	VERBOSE: 8:05:52 PM - Completed Operation: Start-AzureApplicationGateway
	Name       HTTP Status Code     Operation ID                             Error 
	----       ----------------     ------------                             ----
	Successful OK                   fc592db8-4c58-2c8e-9a1d-1c97880f0b9b

## 验证应用程序网关状态

使用 `Get-AzureApplicationGateway` cmdlet 检查网关的状态。如果 *Start-AzureApplicationGateway* 成功，则 State 应为 *Running*，Vip 和 DnsName 应包含有效的条目。

此示例演示了一个正常运行并已准备好将流量定向到 `http://<generated-dns-name>.chinacloudapp.cn` 的应用程序网关。cmdlet 显示在第一行，其后为输出。

	PS C:\> Get-AzureApplicationGateway AppGwTest 

	VERBOSE: 8:09:28 PM - Begin Operation: Get-AzureApplicationGateway 
	VERBOSE: 8:09:30 PM - Completed Operation: Get-AzureApplicationGateway
	Name          : AppGwTest 
	Description   : 
	VnetName      : testvnet1 
	Subnets       : {Subnet-1} 
	InstanceCount : 2 
	GatewaySize   : Medium 
	State         : Running 
	Vip           : 138.91.170.26 
	DnsName       : appgw-1b8402e8-3e0d-428d-b661-289c16c82101.chinacloudapp.cn


## 删除应用程序网关

若要删除应用程序网关，需要按顺序执行以下操作：

1. 发出 `Stop-AzureApplicationGateway` 以停止该网关。 
2. 使用 `Remove-AzureApplicationGateway` cmdlet 删除该网关。
3. 使用 `Get-AzureApplicationGateway` cmdlet 验证是否已删除该网关。

此示例在第一行显示 `Stop-AzureApplicationGateway` cmdlet，接着显示输出。将示例中的值替换为你自己的值。

	PS C:\> Stop-AzureApplicationGateway AppGwTest 

	VERBOSE: 9:49:34 PM - Begin Operation: Stop-AzureApplicationGateway 
	VERBOSE: 10:10:06 PM - Completed Operation: Stop-AzureApplicationGateway
	Name       HTTP Status Code     Operation ID                             Error 
	----       ----------------     ------------                             ----
	Successful OK                   ce6c6c95-77b4-2118-9d65-e29defadffb8

应用程序网关进入停止状态后，请发出 `Remove-AzureApplicationGateway` cmdlet 删除服务。

此示例在第一行显示 cmdlet，接着显示输出。将示例中的值替换为你自己的值。

	PS C:\> Remove-AzureApplicationGateway AppGwTest 

	VERBOSE: 10:49:34 PM - Begin Operation: Remove-AzureApplicationGateway 
	VERBOSE: 10:50:36 PM - Completed Operation: Remove-AzureApplicationGateway
	Name       HTTP Status Code     Operation ID                             Error 
	----       ----------------     ------------                             ----
	Successful OK                   055f3a96-8681-2094-a304-8d9a11ad8301

若要验证是否已删除服务，可以使用 `Get-AzureApplicationGateway` cmdlet。此步骤不是必需的。

此示例在第一行显示 cmdlet，接着显示输出。将示例中的值替换为你自己的值。

	PS C:\> Get-AzureApplicationGateway AppGwTest 

	VERBOSE: 10:52:46 PM - Begin Operation: Get-AzureApplicationGateway 

	Get-AzureApplicationGateway : ResourceNotFound: The gateway does not exist. 
	.....

## 后续步骤

如果你要配置 SSL 卸载，请参阅[配置应用程序网关以进行 SSL 卸载](application-gateway-SSL)。

如需负载平衡选项的其他常规信息，请参阅：

<!--- [Azure Load Balancer](https://azure.microsoft.com/documentation/services/load-balancer/)-->
- [Azure 流量管理器](/documentation/services/traffic-manager)


<!---HONumber=HO63-->