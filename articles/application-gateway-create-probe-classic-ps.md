<properties
   pageTitle="在经典部署模型中使用 PowerShell 创建应用程序网关的自定义探测 | Azure"
   description="了解如何使用经典部署模型中的 PowerShell 创建应用程序网关的自定义探测"
   services="application-gateway"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor=""
   tags="azure-service-management"
/>
<tags  
   ms.service="application-gateway"
   ms.date="03/22/2016"
   wacn.date="06/30/2016" />

# 使用 PowerShell 创建 Azure 应用程序网关（经典）的自定义探测


[AZURE.INCLUDE [azure-probe-intro-include](../includes/application-gateway-create-probe-intro-include.md)]。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用[资源管理器模型](/documentation/articles/application-gateway-create-probe-ps/)。


[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../includes/azure-ps-prerequisites-include.md)]


## 创建新的应用程序网关

创建应用程序网关：

1. 创建应用程序网关资源。
2. 创建配置 XML 文件或配置对象。
3. 将配置提交到新建的应用程序网关资源。

### 创建应用程序网关资源

若要创建网关，请使用 **New-AzureApplicationGateway** cmdlet，并将值替换为你自己的值。请注意，此时不会开始计收网关的费用。计费将在后面已成功启动网关时开始。

以下示例将使用名为“testvnet1”的虚拟网络和名为“subnet-1”的子网创建新的应用程序网关。


	PS C:\> New-AzureApplicationGateway -Name AppGwTest -VnetName testvnet1 -Subnets @("Subnet-1")

	VERBOSE: 4:31:35 PM - Begin Operation: New-AzureApplicationGateway
	VERBOSE: 4:32:37 PM - Completed Operation: New-AzureApplicationGateway
	Name       HTTP Status Code     Operation ID                             Error
	----       ----------------     ------------                             ----
	Successful OK                   55ef0460-825d-2981-ad20-b9a8af41b399


 *Description*、 *InstanceCount* 和 *GatewaySize* 是可选参数。


若要验证是否已创建网关，可以使用 **Get-AzureApplicationGateway** cmdlet。


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

>[AZURE.NOTE]  *InstanceCount* 的默认值为 2，最大值为 10。 *GatewaySize* 的默认值为 Medium。可以选择 Small、Medium 或 Large。


 *VirtualIPs* 和 *DnsName* 显示为空白，因为网关尚未启动。这些值将在网关进入运行状态后立即创建。

## 配置应用程序网关

可以使用 XML 或配置对象配置应用程序网关。

## 使用 XML 配置应用程序网关

在以下示例中，你将使用 XML 文件来配置所有应用程序网关设置，并将这些设置提交到应用程序网关资源。

### 步骤 1  

将以下文本复制到记事本中。


	<ApplicationGatewayConfiguration xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/windowsazure">
    <FrontendIPConfigurations>
        <FrontendIPConfiguration>
            <Name>fip1</Name>
            <Type>Private</Type>
        </FrontendIPConfiguration>
    </FrontendIPConfigurations>    
	<FrontendPorts>
        <FrontendPort>
            <Name>port1</Name>
            <Port>80</Port>
        </FrontendPort>
    </FrontendPorts>
    <Probes>
        <Probe>
            <Name>Probe01</Name>
            <Protocol>Http</Protocol>
            <Host>contoso.com</Host>
            <Path>/path/custompath.htm</Path>
            <Interval>15</Interval>
            <Timeout>15</Timeout>
            <UnhealthyThreshold>5</UnhealthyThreshold>
        </Probe>
      </Probes>
    <BackendAddressPools>
        <BackendAddressPool>
            <Name>pool1</Name>
            <IPAddresses>
                <IPAddress>1.1.1.1</IPAddress>
				<IPAddress>2.2.2.2</IPAddress>
            </IPAddresses>
        </BackendAddressPool>
    </BackendAddressPools>
    <BackendHttpSettingsList>
        <BackendHttpSettings>
            <Name>setting1</Name>
            <Port>80</Port>
            <Protocol>Http</Protocol>
            <CookieBasedAffinity>Enabled</CookieBasedAffinity>
            <RequestTimeout>120</RequestTimeout>
            <Probe>Probe01</Probe>
        </BackendHttpSettings>
    </BackendHttpSettingsList>
    <HttpListeners>
        <HttpListener>
            <Name>listener1</Name>
            <FrontendIP>fip1</FrontendIP>
	    <FrontendPort>port1</FrontendPort>
            <Protocol>Http</Protocol>
        </HttpListener>
    </HttpListeners>
    <HttpLoadBalancingRules>
        <HttpLoadBalancingRule>
            <Name>lbrule1</Name>
            <Type>basic</Type>
            <BackendHttpSettings>setting1</BackendHttpSettings>
            <Listener>listener1</Listener>
            <BackendAddressPool>pool1</BackendAddressPool>
        </HttpLoadBalancingRule>
    </HttpLoadBalancingRules>
	</ApplicationGatewayConfiguration>


编辑配置项的括号之间的值。使用扩展名 .xml 保存文件。

以下示例演示如何使用配置文件设置应用程序网关负载平衡公共端口 80 上的 HTTP 流量，然后使用自定义探测将网络流量发送到 2 个 IP 地址之间的后端端口 80。

>[AZURE.IMPORTANT] 协议项 Http 或 Https 区分大小写。


已添加用于配置自定义探测的新配置项 <Probe>。

配置参数为：

- **Name** - 自定义探测的引用名称。
- **Protocol** - 使用的协议（可能的值为 HTTP 或 HTTPS）。
- **Host** and **Path** - 应用程序网关为了确定实例运行状况而调用的完整 URL 路径。例如，如果你的网站为 http://contoso.com/ ，则可以为“http://contoso.com/path/custompath.htm” 配置自定义探测，使探测检查能够获得成功的 HTTP 响应。
- **Interval** - 配置探测检查间隔，以秒为单位。
- **Timeout** - 定义 HTTP 响应检查的探测超时。
- **UnhealthyThreshold** - 将后端实例标记为*不正常*所需的失败 HTTP 响应数目。

在 <BackendHttpSettings> 配置中引用探测名称，以分配使用自定义探测设置的后端池。

## 将自定义探测配置添加到现有应用程序网关

更改当前的应用程序网关配置需要完成三个步骤：获取当前的 XML 配置文件，对其进行修改以创建自定义探测，使用新的 XML 设置来配置应用程序网关。

### 步骤 1

使用 get-AzureApplicationGatewayConfig 获取 XML 文件。这会导出要修改的配置 XML 以添加探测设置。

	get-AzureApplicationGatewayConfig -Name <application gateway name> -Exporttofile "<path to file>"


### 步骤 2

在文本编辑器中打开 XML 文件。将 `<probe>` 节添加到 `<frontendport>` 的后面。

	<Probes>
        <Probe>
            <Name>Probe01</Name>
            <Protocol>Http</Protocol>
            <Host>contoso.com</Host>
            <Path>/path/custompath.htm</Path>
            <Interval>15</Interval>
            <Timeout>15</Timeout>
            <UnhealthyThreshold>5</UnhealthyThreshold>
        </Probe>

在 XML 的 backendHttpSettings 节中，添加以下示例中所示的探测名称：

        <BackendHttpSettings>
            <Name>setting1</Name>
            <Port>80</Port>
            <Protocol>Http</Protocol>
            <CookieBasedAffinity>Enabled</CookieBasedAffinity>
            <RequestTimeout>120</RequestTimeout>
            <Probe>Probe01</Probe>
        </BackendHttpSettings>

保存该 XML 文件。


### 步骤 3

使用 **Set-AzureApplicationGatewayConfig** 在新的 XML 文件中更新应用程序网关配置。这将以新的配置更新应用程序网关。

	set-AzureApplicationGatewayConfig -Name <application gateway name> -Configfile "<path to file>"


## 后续步骤

如果你要配置安全套接字层 (SSL) 卸载，请参阅[配置应用程序网关以进行 SSL 卸载](/documentation/articles/application-gateway-ssl/)。

如果你想要将应用程序网关配置为与内部负载平衡器配合使用，请参阅[创建具有内部负载平衡器 (ILB) 的应用程序网关](/documentation/articles/application-gateway-ilb/)。

<!---HONumber=Mooncake_0215_2016-->
