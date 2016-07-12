<properties 
   pageTitle="在虚拟网络中创建和配置具有内部负载平衡器 (ILB) 的应用程序网关 | Azure"
   description="本页说明如何配置具有内部负载平衡终结点的 Azure 应用程序网关"
   documentationCenter="na"
   services="application-gateway"
   authors="joaoma"
   manager="jdial"
   editor="tysonn"/>
<tags 
   ms.service="application-gateway"
   ms.date="04/05/2016"
   wacn.date="06/30/2016"/>

# 创建具有内部负载平衡器 (ILB) 的应用程序网关

> [AZURE.SELECTOR]
- [Azure 经典步骤](/documentation/articles/application-gateway-ilb/)
- [资源管理器 Powershell 步骤](/documentation/articles/application-gateway-ilb-arm/)

可以配置使用面向 Internet 的虚拟 IP 或不向 Internet 公开的内部终结点（也称为内部负载平衡器 (ILB) 终结点）的应用程序网关。配置使用 ILB 的网关适用于不向 Internet 公开的内部业务线应用程序。对于位于不向 Internet 公开的安全边界内的多层应用程序中的服务/层也很有用，但仍需要执行循环负载分散、会话粘性或 SSL 终止。本文将引导你配置具有 ILB 的应用程序网关。

## 开始之前

1. 使用 Web 平台安装程序安装最新版本的 Azure PowerShell cmdlet。可以从[下载页面](/downloads)的“Windows PowerShell”部分下载并安装最新版本。
2. 请确认你已创建包含有效子网、可正常运行的虚拟网络。
3. 请确认后端服务器位于虚拟网络中，或者为后端服务器分配了公共 IP/VIP。


若要创建新的应用程序网关，请按所列顺序执行以下步骤。

1. [创建新的应用程序网关](#create-a-new-application-gateway)
2. [配置网关](#configure-the-gateway)
3. [设置网关配置](#set-the-gateway-configuration)
4. [启动网关](#start-the-gateway)
4. [验证网关](#verify-the-gateway-status)



##<a name="create-a-new-application-gateway"></a> 创建新的应用程序网关：

**若要创建网关**，请使用 `New-AzureApplicationGateway` cmdlet，并将值替换为你自己的值。请注意，此时不会开始计收网关的费用。计费将在后面已成功启动网关时开始。

	PS C:\> New-AzureApplicationGateway -Name AppGwTest -VnetName testvnet1 -Subnets @("Subnet-1")

	VERBOSE: 4:31:35 PM - Begin Operation: New-AzureApplicationGateway 
	VERBOSE: 4:32:37 PM - Completed Operation: New-AzureApplicationGateway
	Name       HTTP Status Code     Operation ID                             Error 
	----       ----------------     ------------                             ----
	Successful OK                   55ef0460-825d-2981-ad20-b9a8af41b399

**若要验证**是否已创建网关，可以使用 `Get-AzureApplicationGateway` cmdlet。

在此示例中， *Description*、 *InstanceCount* 和 *GatewaySize* 是可选参数。*InstanceCount* 的默认值为 2，最大值为 10。*GatewaySize* 的默认值为 Medium。其他可用值为 Small 和 Large。*Vip* 和 *DnsName* 显示为空白，因为网关尚未启动。这些值将在网关进入运行状态后立即创建。

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


##<a name="configure-the-gateway"></a> 配置网关

应用程序网关配置由多个值组成。这些值可将绑定在一起以构造配置。
 
有效值为：

- **后端服务器池：**后端服务器的 IP 地址列表。列出的 IP 地址应属于 VNet 子网，或者是公共 IP/VIP。 
- **后端服务器池设置：**每个池具有端口、协议和基于 Cookie 的相关性等设置。这些设置绑定到池，并会应用到池中的所有服务器。
- **前端端口：**此端口是应用程序网关上打开的公共端口。流量将抵达此端口，然后重定向到后端服务器之一。
- **侦听器：**侦听器具有前端端口、协议（Http 或 Https，区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。 
- **规则：**规则将会绑定侦听器和后端服务器池，并定义当流量抵达特定侦听器时应定向到的后端服务器池。目前仅支持 *基本* 规则。 *基本* 规则是一种轮循负载分发模式。

可以通过创建配置对象或使用配置 XML 文件来构造配置。
若要使用配置 XML 文件构造配置，请使用以下示例。



注意以下事项：


- *FrontendIPConfigurations* 元素描述有关配置具有 ILB 的应用程序网关的 ILB 详细信息。 

- 前端 IP *类型* 应设置为“专用”

- *StaticIPAddress* 应设置为所需的内部 IP，网关在该处接收流量。请注意，*StaticIPAddress* 元素是可选的。如果未设置，将选择来自已部署子网的可用内部 IP。

- 在 *FrontendIPConfiguration* 中指定的 *Name* 元素的值应该在 HTTPListener 的 *FrontendIP* 元素中使用，以引用 FrontendIPConfiguration。

 **配置 XML 示例**

 

		<?xml version="1.0" encoding="utf-8"?>
		<ApplicationGatewayConfiguration xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/windowsazure">
			<FrontendIPConfigurations>
				<FrontendIPConfiguration>
					<Name>fip1</Name> 
					<Type>Private</Type> 
					<StaticIPAddress>10.0.0.10</StaticIPAddress> 
				</FrontendIPConfiguration>
			</FrontendIPConfigurations>
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
					<FrontendIP>fip1</FrontendIP>
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
	


##<a name="set-the-gateway-configuration"></a> 设置网关配置

接下来，你将设置应用程序网关。可以对配置对象或配置 XML 文件使用 `Set-AzureApplicationGatewayConfig` cmdlet。

	PS C:\> Set-AzureApplicationGatewayConfig -Name AppGwTest -ConfigFile D:\config.xml

	VERBOSE: 7:54:59 PM - Begin Operation: Set-AzureApplicationGatewayConfig 
	VERBOSE: 7:55:32 PM - Completed Operation: Set-AzureApplicationGatewayConfig
	Name       HTTP Status Code     Operation ID                             Error 
	----       ----------------     ------------                             ----
	Successful OK                   9b995a09-66fe-2944-8b67-9bb04fcccb9d

##<a name="start-the-gateway"></a> 启动网关

配置网关后，使用 `Start-AzureApplicationGateway` cmdlet 来启动网关。成功启动网关后，将开始计收应用程序网关的费用。


**注意：**`Start-AzureApplicationGateway` cmdlet 最多可能需要 15 到 20 分钟才能完成。
   
	PS C:\> Start-AzureApplicationGateway AppGwTest 

	VERBOSE: 7:59:16 PM - Begin Operation: Start-AzureApplicationGateway 
	VERBOSE: 8:05:52 PM - Completed Operation: Start-AzureApplicationGateway
	Name       HTTP Status Code     Operation ID                             Error 
	----       ----------------     ------------                             ----
	Successful OK                   fc592db8-4c58-2c8e-9a1d-1c97880f0b9b

##<a name="verify-the-gateway-status"></a> 验证网关状态

使用 `Get-AzureApplicationGateway` cmdlet 检查网关的状态。如果前一步骤中的 *Start-AzureApplicationGateway* 成功，则 State 应为 *Running*，Vip 和 DnsName 应包含有效的条目。此示例在第一行显示 cmdlet，接着显示输出。在此示例中，网关正在运行并准备好接收流量。

**注意：**在此示例中，应用程序网关配置为在配置的 ILB 终结点 10.0.0.10 上接受流量。

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
	VirtualIPs    : {10.0.0.10}
	DnsName       : appgw-b2a11563-2b3a-4172-a4aa-226ee4c23eed.chinacloudapp.cn

## 后续步骤


如需负载平衡选项的其他常规信息，请参阅：

<!--- [Azure 负载平衡器](/documentation/services/load-balancer)-->
- [Azure 流量管理器](/documentation/services/traffic-manager)

<!---HONumber=Mooncake_0307_2016-->
