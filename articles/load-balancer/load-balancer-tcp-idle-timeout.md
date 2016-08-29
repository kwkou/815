<properties 
   pageTitle="配置负载平衡器的 TCP 空闲超时 | Azure"
   description="配置负载平衡器的 TCP 空闲超时"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn" />
<tags 
   ms.service="load-balancer"
   ms.date="03/03/2016"
   wacn.date="08/29/2016" />


# 如何更改负载平衡器的 TCP 空闲超时设置

在默认配置中，Azure Load Balancer 的“空闲超时”设置为 4 分钟。

这意味着，如果你的 tcp 或 http 会话处于非活动状态的时间超出超时值，则无法保证客户端和你的服务之间会维持连接状态。

当连接关闭时，你的客户端应用程序会收到一条类似于下面这样的错误消息：“基础连接已关闭: 应保持活动状态的连接已由服务器关闭”。

若要让连接处于活动状态的时间更长一些，常见的做法是使用 TCP Keep-alive（如需 .NET 示例，可单击[此处](https://msdn.microsoft.com/zh-cn/library/system.net.servicepoint.settcpkeepalive.aspx)）。

在连接上检测不到活动时，会发送数据包。让网络活动处于持续状态，则始终达不到空闲超时值，于是就可以长时间维持连接。

可以考虑在配置 TCP Keep-alive 时，让时间间隔短于默认的超时设置，以免连接掉线，也可以通过增加 TCP 连接会话的空闲超时值来保持连接。

虽然在不需要考虑 电池因素的情况下可以使用 TCP Keep-alive，但对于移动应用程序来说，这种选择通常无效。在移动应用程序中使用 TCP Keep-alive 可能会加快设备电池的耗尽速度。

我们已允许对空闲超时进行配置，以便支持这样的情况。现在，你可以将空闲超时的持续时间设置为 4 到 30 分钟。此设置仅适用于入站连接。

![tcptimeout](./media/load-balancer-tcp-idle-timeout/image1.png)


## 如何更改虚拟机和云服务中的空闲超时设置

- 通过 PowerShell 或服务管理 API 配置虚拟机上某个终结点的 TCP 超时
- 通过 PowerShell 或服务管理 API 配置负载平衡终结点集的 TCP 超时。
- 配置实例层级公共 IP 的 TCP 超时
- 通过服务模型配置 Web 角色/辅助角色的 TCP 超时。
 

>[AZURE.NOTE] 请记住，某些命令将只存在于最新 Azure PowerShell 包中。如果该 PowerShell 命令不存在，请下载最新 PowerShell 包。

 
### 将你的实例层级公共 IP 的 TCP 超时配置为 15 分钟。

	Set-AzurePublicIP –PublicIPName webip –VM MyVM -IdleTimeoutInMinutes 15

IdleTimeoutInMinutes 为可选。在未设置的情况下，默认超时为 4 分钟。

>[AZURE.NOTE] 可接受的超时范围为 4 到 30 分钟。
 
### 在虚拟机上创建 Azure 终结点时设置空闲超时

若要更改终结点的超时设置，请执行以下命令

	Get-AzureVM -ServiceName "mySvc" -Name "MyVM1" | Add-AzureEndpoint -Name "HttpIn" -Protocol "tcp" -PublicPort 80 -LocalPort 8080 -IdleTimeoutInMinutes 15| Update-AzureVM
 
检索空闲超时配置

	PS C:\> Get-AzureVM –ServiceName “MyService” –Name “MyVM” | Get-AzureEndpoint
	VERBOSE: 6:43:50 PM - Completed Operation: Get Deployment
	LBSetName : MyLoadBalancedSet
	LocalPort : 80
	Name : HTTP
	Port : 80
	Protocol : tcp
	Vip : 65.52.xxx.xxx
	ProbePath :
	ProbePort : 80
	ProbeProtocol : tcp
	ProbeIntervalInSeconds : 15
	ProbeTimeoutInSeconds : 31
	EnableDirectServerReturn : False
	Acl : {}
	InternalLoadBalancerName :
	IdleTimeoutInMinutes : 15
 
### 在负载平衡终结点集上设置 TCP 超时

如果终结点是负载平衡终结点集的一部分，则必须在负载平衡终结点集上设置 TCP 超时：

	Set-AzureLoadBalancedEndpoint -ServiceName "MyService" -LBSetName "LBSet1" -Protocol tcp -LocalPort 80 -ProbeProtocolTCP -ProbePort 8080 -IdleTimeoutInMinutes 15
 
### 更改云服务的超时设置

你可以利用用于 .NET 2.4 的 Azure SDK 来更新云服务。

云服务的终结点设置在 .csdef 中进行。若要更新云服务部署的 TCP 超时，需进行部署升级。例外情况是仅针对公共 IP 指定 TCP 超时。公共 IP 设置位于 .cscfg 中，可以通过部署更新和升级进行更新。

终结点设置的 .csdef 更改如下：

	<WorkerRole name="worker-role-name" vmsize="worker-role-size" enableNativeCodeExecution="[true|false]">
	  <Endpoints>
    <InputEndpoint name="input-endpoint-name" protocol="[http|https|tcp|udp]" localPort="local-port-number" port="port-number" certificate="certificate-name" loadBalancerProbe="load-balancer-probe-name" idleTimeoutInMinutes="tcp-timeout" />
	  </Endpoints>
	</WorkerRole>

进行公共 IP 的超时设置时，.cscfg 更改如下：

	<NetworkConfiguration>
 	 <VirtualNetworkSite name="VNet"/>
 	 <AddressAssignments>
    <InstanceAddress roleName="VMRolePersisted">
      <PublicIPs>
        <PublicIP name="public-ip-name" idleTimeoutInMinutes="timeout-in-minutes"/>
      </PublicIPs>
    </InstanceAddress>
 	 </AddressAssignments>
	</NetworkConfiguration>

## Rest API 示例

你可以使用服务管理 API 来配置 TCP 空闲超时。
请确保添加 x-ms-version 标头并将其设置为 2014-06-01 或更高版本。
 
通过一次部署，在所有虚拟机上更新指定的负载平衡输入终结点的配置
	
	Request

	POST https://management.core.chinacloudapi.cn/<subscription-id>/services/hostedservices/<cloudservice-name>/deployments/<deployment-name>
<BR>

	Response

	<LoadBalancedEndpointList xmlns="http://schemas.microsoft.com/windowsazure" xmlns:i="http://www.w3.org/2001/XMLSchema-instance">
	<InputEndpoint>
	<LoadBalancedEndpointSetName>endpoint-set-name</LoadBalancedEndpointSetName>
	<LocalPort>local-port-number</LocalPort>
	<Port>external-port-number</Port>
	<LoadBalancerProbe>
	<Path>path-of-probe</Path>
	<Port>port-assigned-to-probe</Port>
	<Protocol>probe-protocol</Protocol>
	<IntervalInSeconds>interval-of-probe</IntervalInSeconds>
	<TimeoutInSeconds>timeout-for-probe</TimeoutInSeconds>
	</LoadBalancerProbe>
	<LoadBalancerName>name-of-internal-loadbalancer</LoadBalancerName>
	<Protocol>endpoint-protocol</Protocol>
	<IdleTimeoutInMinutes>15</IdleTimeoutInMinutes>
	<EnableDirectServerReturn>enable-direct-server-return</EnableDirectServerReturn>
	<EndpointACL>
	<Rules>
	<Rule>
	<Order>priority-of-the-rule</Order>
	<Action>permit-rule</Action>
	<RemoteSubnet>subnet-of-the-rule</RemoteSubnet>
	<Description>description-of-the-rule</Description>
	</Rule>
	</Rules>
	</EndpointACL>
	</InputEndpoint>
	</LoadBalancedEndpointList>

## 后续步骤

[内部负载平衡器概述](/documentation/articles/load-balancer-internal-overview/)

[开始配置面向 Internet 的负载平衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)

[配置负载平衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

 

<!---HONumber=Mooncake_0822_2016-->