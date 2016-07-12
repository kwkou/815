<properties
   pageTitle="使用经典部署配置应用程序网关以进行 SSL 卸载 | Azure"
   description="本文提供有关使用 Azure 经典部署模型创建支持 SSL 卸载的应用程序网关的说明。"
   documentationCenter="na"
   services="application-gateway"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn"/>
<tags
   ms.service="application-gateway"
   ms.date="04/05/2016"
   wacn.date="06/30/2016"/>

# 使用经典部署模型配置应用程序网关以进行 SSL 卸载

> [AZURE.SELECTOR]
-[Azure 经典 PowerShell](/documentation/articles/application-gateway-ssl/)
-[Azure 资源管理器 PowerShell](/documentation/articles/application-gateway-ssl-arm/)

可将 Azure 应用程序网关配置为在网关上终止安全套接字层 (SSL) 会话，以避免 Web 场中出现开销较高的 SSL 解密任务。SSL 卸载还简化了 Web 应用程序的前端服务器设置与管理。


## 开始之前

1. 使用 Web 平台安装程序安装最新版本的 Azure PowerShell cmdlet。可以从[下载页面](/downloads)的“Windows PowerShell”部分下载并安装最新版本。
2. 请确认你已创建包含有效子网、可正常运行的虚拟网络。请确保没有虚拟机或云部署正在使用子网。应用程序网关必须单独位于虚拟网络子网中。
3. 要配置为使用应用程序网关的服务器必须存在，或者在虚拟网络中为其创建终结点，或者为其分配公共 IP/VIP。

若要在应用程序网关上配置 SSL 卸载，请按所列顺序执行以下步骤：

1. [创建新的应用程序网关](#create-a-new-application-gateway)
2. [上载 SSL 证书](#upload-ssl-certificates)
3. [配置网关](#configure-the-gateway)
4. [设置网关配置](#set-the-gateway-configuration)
5. [启动网关](#start-the-gateway)
6. [验证网关状态](#verify-the-gateway-status)


##<a name="create-a-new-application-gateway"></a> 创建新的应用程序网关

若要创建网关，请使用 **New-AzureApplicationGateway** cmdlet，并将值替换为你自己的值。请注意，此时不会开始计收网关的费用。计费将在后面已成功启动网关时开始。

此示例在第一行显示 cmdlet，接着显示输出。

	PS C:\> New-AzureApplicationGateway -Name AppGwTest -VnetName testvnet1 -Subnets @("Subnet-1")

	VERBOSE: 4:31:35 PM - Begin Operation: New-AzureApplicationGateway
	VERBOSE: 4:32:37 PM - Completed Operation: New-AzureApplicationGateway
	Name       HTTP Status Code     Operation ID                             Error
	----       ----------------     ------------                             ----
	Successful OK                   55ef0460-825d-2981-ad20-b9a8af41b399

若要验证是否已创建网关，可以使用 **Get-AzureApplicationGateway** cmdlet。

在此示例中，Description、InstanceCount 和 GatewaySize 是可选参数。InstanceCount 的默认值为 2，最大值为 10。GatewaySize 的默认值为 Medium。其他可用值为 Small 和 Large。VirtualIPs 和 DnsName 显示为空白，因为网关尚未启动。这些值将在网关进入运行状态后立即创建。

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


##<a name="upload-ssl-certificates"></a> 上载 SSL 证书

使用 **Add-AzureApplicationGatewaySslCertificate** 将 pfx 格式的服务器证书上载到应用程序网关。证书名称是用户选择的名称，在应用程序网关中必须唯一。在应用程序网关上执行所有证书管理操作时，将按此名称引用此证书。

此示例在第一行显示 cmdlet，接着显示输出。将示例中的值替换为你自己的值。

	PS C:\> Add-AzureApplicationGatewaySslCertificate  -Name AppGwTest -CertificateName GWCert -Password <password> -CertificateFile <full path to pfx file>

	VERBOSE: 5:05:23 PM - Begin Operation: Get-AzureApplicationGatewaySslCertificate
	VERBOSE: 5:06:29 PM - Completed Operation: Get-AzureApplicationGatewaySslCertificate
	Name       HTTP Status Code     Operation ID                             Error
	----       ----------------     ------------                             ----
	Successful OK                   21fdc5a0-3bf7-2c12-ad98-192e0dd078ef

接下来，验证证书上载。使用 **Get-AzureApplicationGatewayCertificate** cmdlet。

此示例在第一行显示 cmdlet，接着显示输出。

	PS C:\> Get-AzureApplicationGatewaySslCertificate AppGwTest

	VERBOSE: 5:07:54 PM - Begin Operation: Get-AzureApplicationGatewaySslCertificate
	VERBOSE: 5:07:55 PM - Completed Operation: Get-AzureApplicationGatewaySslCertificate
	Name           : SslCert
	SubjectName    : CN=gwcert.app.test.contoso.com
	Thumbprint     : AF5ADD77E160A01A6......EE48D1A
	ThumbprintAlgo : sha1RSA
	State..........: Provisioned

>[AZURE.NOTE] 证书密码的长度必须是 4 到 12 个字符，可包含字母或数字。不接受特殊字符。

##<a name="configure-the-gateway"></a> 配置网关

应用程序网关配置由多个值组成。这些值可将绑定在一起以构造配置。

有效值为：

- **后端服务器池：**后端服务器的 IP 地址列表。列出的 IP 地址应属于虚拟网络子网，或者是公共 IP/VIP。
- **后端服务器池设置：**每个池都有一些设置，例如端口、协议和基于 Cookie 的关联性。这些设置绑定到池，并会应用到池中的所有服务器。
- **前端端口：**此端口是应用程序网关上打开的公共端口。客户流量将抵达此端口，然后重定向到后端服务器之一。
- **侦听器：**侦听器具有前端端口、协议（Http 或 Https，区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。
- **规则：**规则将会绑定侦听器和后端服务器池，并定义当流量抵达特定侦听器时应定向到的后端服务器池。目前仅支持基本规则。基本规则是一种轮循负载分发模式。

**其他配置说明**

对于 SSL 证书配置，**HttpListener** 中的协议应更改为 Https（区分大小写）。需要将 **SslCert** 元素添加到 **HttpListener**，后者的值设置为上述上载 SSL 证书部分中使用的名称。前端端口应更新为 443。

**启用基于 Cookie 的相关性**：可以配置应用程序网关，以确保来自客户端会话的请求始终定向到 Web 场中的同一 VM。这是通过注入允许网关适当定向流量的会话 Cookie 来实现的。若要启用基于 Cookie 的相关性，请在 **BackendHttpSettings** 元素中将 **CookieBasedAffinity** 设置为 Enabled。



可以通过创建配置对象或使用配置 XML 文件来构造配置。
若要使用配置 XML 文件构造配置，请使用以下示例。

**配置 XML 示例**


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


##<a name="set-the-gateway-configuration"></a> 设置网关配置

接下来，你将设置应用程序网关。可以对配置对象或配置 XML 文件使用 **Set-AzureApplicationGatewayConfig** cmdlet。


	PS C:\> Set-AzureApplicationGatewayConfig -Name AppGwTest -ConfigFile D:\config.xml

	VERBOSE: 7:54:59 PM - Begin Operation: Set-AzureApplicationGatewayConfig
	VERBOSE: 7:55:32 PM - Completed Operation: Set-AzureApplicationGatewayConfig
	Name       HTTP Status Code     Operation ID                             Error
	----       ----------------     ------------                             ----
	Successful OK                   9b995a09-66fe-2944-8b67-9bb04fcccb9d

##<a name="start-the-gateway"></a> 启动网关

配置网关后，使用 **Start-AzureApplicationGateway** cmdlet 来启动网关。成功启动网关后，将开始计收应用程序网关的费用。


**注意：****Start-AzureApplicationGateway** cmdlet 可能需要 15-20 分钟的时间才能完成。


	PS C:\> Start-AzureApplicationGateway AppGwTest

	VERBOSE: 7:59:16 PM - Begin Operation: Start-AzureApplicationGateway
	VERBOSE: 8:05:52 PM - Completed Operation: Start-AzureApplicationGateway
	Name       HTTP Status Code     Operation ID                             Error
	----       ----------------     ------------                             ----
	Successful OK                   fc592db8-4c58-2c8e-9a1d-1c97880f0b9b


##<a name="verify-the-gateway-status"></a> 验证网关状态

使用 **Get-AzureApplicationGateway** cmdlet 检查网关的状态。如果前一步骤中的 **Start-AzureApplicationGateway** 成功，则 State 应为 Running，VirtualIPs 和 DnsName 应包含有效的条目。

此示例演示了一个正常运行并已准备好接收流量的应用程序网关。

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

<!--- [Azure 负载平衡器](/documentation/services/load-balancer)-->
- [Azure 流量管理器](/documentation/services/traffic-manager)

<!---HONumber=Mooncake_0425_2016-->
