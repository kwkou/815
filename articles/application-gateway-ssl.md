<properties 
   pageTitle="配置应用程序网关以进行 SSL 卸载 | Windows Azure"
   description="本文提供有关在 Azure 应用程序网关上配置 SSL 卸载的说明。"
   documentationCenter="na"
   services="application-gateway"
   authors="cherylmc"
   manager="jdial"
   editor="tysonn"/>
<tags 
   ms.service="application-gateway"
   ms.date="06/23/2015"
   wacn.date="07/19/2015"/>

# 配置应用程序网关以进行 SSL 卸载

可将应用程序网关配置为在网关上终止 SSL 会话，从而避免在 Web 场上执行开销较高的 SSL 解密。SSL 卸载还简化了应用程序的前端服务器设置与管理。

## 开始之前

1. 使用 Web 平台安装程序安装最新版本的 Azure PowerShell cmdlet。可以从[下载页面](/downloads)的 **Windows PowerShell** 部分下载和安装最新的 PowerShell cmdlet。
2. 请确认你已创建包含有效子网、可正常运行的虚拟网络。
3. 请确认后端服务器位于虚拟网络中，或者为后端服务器分配了公共 IP/VIP。

若要在应用程序网关上配置 SSL 卸载，请按所列顺序执行以下步骤。步骤的下面是你要使用的每个 cmdlet 的示例。

1. [创建新的应用程序网关](#create-a-new-application-gateway)
2. [上载 SSL 证书](#upload-ssl-certificates) 
3. [配置网关](#configure-the-application-gateway)
4. [设置网关配置](#set-the-application-gateway-configuration)
5. [启动网关](#start-the-application-gateway)


## 创建新的应用程序网关：

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

	VERBOSE: 4:39:39 PM - Begin Operation:
	Get-AzureApplicationGateway VERBOSE: 4:39:40 PM - Completed 
	Operation: Get-AzureApplicationGateway
	Name: AppGwTest	
	Description: 
	VnetName: testvnet1 
	Subnets: {Subnet-1} 
	InstanceCount: 2 
	GatewaySize: Medium 
	State: Stopped 
	VirtualIPs: 
	DnsName:


## 上载 SSL 证书 

使用 `Add-AzureApplicationGatewaySslCertificate` 将 pfx 格式的服务器证书上载到应用程序网关。证书名称是用户选择的名称，在应用程序网关中必须唯一。在应用程序网关上执行所有证书管理操作时，将按此名称引用此证书。

此示例在第一行显示 cmdlet，接着显示输出。将示例中的值替换为你自己的值。

	PS C:\> Add-AzureApplicationGatewaySslCertificate  -Name AppGwTest -CertificateName GWCert -Password <password> -CertificateFile <full path to pfx file> 
	
	VERBOSE: 5:05:23 PM - Begin Operation: Get-AzureApplicationGatewaySslCertificate 
	VERBOSE: 5:06:29 PM - Completed Operation: Get-AzureApplicationGatewaySslCertificate
	Name       HTTP Status Code     Operation ID                             Error 
	----       ----------------     ------------                             ----
	Successful OK                   21fdc5a0-3bf7-2c12-ad98-192e0dd078ef

使用 `Get-AzureApplicationGatewayCertificate` cmdlet 验证证书上载。

此示例在第一行显示 cmdlet，接着显示输出。

	PS C:\> Get-AzureApplicationGatewaySslCertificate AppGwTest 

	VERBOSE: 5:07:54 PM - Begin Operation: Get-AzureApplicationGatewaySslCertificate 
	VERBOSE: 5:07:55 PM - Completed Operation: Get-AzureApplicationGatewaySslCertificate
	Name           : SslCert 
	SubjectName    : CN=gwcert.app.test.contoso.com 
	Thumbprint     : AF5ADD77E160A01A6......EE48D1A 
	ThumbprintAlgo : sha1RSA
	State..........: Provisioned


## 配置应用程序网关

应用程序网关配置包含以下实体，可以组合这些实体以构造配置。
 
- **后端服务器池：**后端服务器的 IP 地址列表。此 IP 应属于 VNET 子网，或者应为公共 IP/VIP。 
- **后端服务器池设置：**每个池具有端口、协议和基于 Cookie 的相关性等设置。这些设置绑定到池，并会应用到池中的所有服务器。
- **前端端口：**此端口是应用程序网关上打开的公共端口。客户流量将抵达此端口，然后重定向到后端服务器之一。
- **侦听器：**侦听器具有前端端口、协议（Http 或 Https，区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。 
- **规则：**规则将会绑定侦听器和后端服务器池，并定义当流量抵达特定侦听器时应定向到的后端服务器池。目前仅支持“基本”规则，即轮循负载分发。 

可以通过创建配置对象或使用配置 XML 文件来构造配置。

对于 SSL 证书配置，**HttpListener** 中的协议应更改为 *Https*（区分大小写）。需要将 **SslCert** 元素添加到 **HttpListener**，后者的值设置为上述上载 SSL 证书部分中使用的名称。前端端口应更新为 443。

请注意，可以配置应用程序网关，以确保来自客户端会话的请求始终定向到 Web 场中的同一 VM。这是通过注入允许应用程序网关适当定向流量的会话 Cookie 来实现的。若要启用基于 Cookie 的相关性，请在 **BackendHttpSettings** 元素中将 **CookieBasedAffinity** 设置为 *Enabled*。


### 配置 XML 示例


	    <?xml version="1.0" encoding="utf-8"?>
	<ApplicationGatewayConfiguration xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/windowsazure">
	    <FrontendIPConfigurations />
	    <FrontendPorts>
	        <FrontendPort>
	            <Name>FrontendPort1</Name>
	            <Port>443</Port>
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
	            <Protocol>Https</Protocol>
	            <SslCert>GWCert</SslCert>
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


## 设置应用程序网关配置

可以配合配置对象或配置 XML 文件运行 `Set-ApplicationGatewayConfig` cmdlet。在以下示例中，我们将使用配置 XML 文件。

此示例在第一行显示 cmdlet，接着显示输出。将值替换为你自己的值。

	PS C:\> Set-AzureApplicationGatewayConfig -Name AppGwTest -ConfigFile D:\config.xml

	VERBOSE: 7:54:59 PM - Begin Operation: Set-AzureApplicationGatewayConfig 
	VERBOSE: 7:55:32 PM - Completed Operation: Set-AzureApplicationGatewayConfig
	Name       HTTP Status Code     Operation ID                             Error 
	----       ----------------     ------------                             ----
	Successful OK                   9b995a09-66fe-2944-8b67-9bb04fcccb9d

## 启动应用程序网关

配置网关后，使用 `Start-AzureApplicationGateway` cmdlet 来启动网关。这是用于设置网关的 cmdlet。成功运行该 cmdlet 并设置网关后，也会开始计费。


**注意** `Start-AzureApplicationGateway` cmdlet 可能最长可能需要 15-20 分钟才能完成。

此示例在第一行显示 cmdlet，接着显示输出。将示例中的值替换为你自己的值。
   
	PS C:\> Start-AzureApplicationGateway AppGwTest 

	VERBOSE: 7:59:16 PM - Begin Operation: Start-AzureApplicationGateway 
	VERBOSE: 8:05:52 PM - Completed Operation: Start-AzureApplicationGateway
	Name       HTTP Status Code     Operation ID                             Error 
	----       ----------------     ------------                             ----
	Successful OK                   fc592db8-4c58-2c8e-9a1d-1c97880f0b9b


使用 `Get-AzureApplicationGateway` cmdlet 检查网关的状态。如果 *Start-AzureApplicationGateway* 成功，则 State 应为“*Running*”，Vip 和 DnsName 应包含有效的条目。

此示例在第一行显示 cmdlet，接着显示输出。在此示例中，网关已启动并准备好接收流量。

	PS C:\> Get-AzureApplicationGateway AppGwTest 

	Name          : AppGwTest2
	Description   : 
	VnetName      : testvnet1
	Subnets       : {Subnet-1}
	InstanceCount : 2
	GatewaySize   : Medium
	State         : Running
	VirtualIPs    : {23.96.22.241}
	DnsName       : appgw-4c960426-d1e6-4aae-8670-81fd7a519a43.chinacloudapp.cn


## 后续步骤


如需负载平衡选项的其他常规信息，请参阅：

<!--- [Azure Load Balancer](https://azure.microsoft.com/documentation/services/load-balancer/)-->
- [Azure 流量管理器](/documentation/services/traffic-manager)


<!---HONumber=HO63-->