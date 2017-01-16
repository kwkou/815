<properties
    pageTitle="配置负载均衡器的 TCP 空闲超时 | Azure"
    description="配置负载均衡器的 TCP 空闲超时"
    services="load-balancer"
    documentationcenter="na"
    author="kumudd"
    manager="timlt" />
<tags
    ms.assetid="4625c6a8-5725-47ce-81db-4fa3bd055891"
    ms.service="load-balancer"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="10/24/2016"
    wacn.date="01/13/2017"
    ms.author="kumud" />  


# 为 Azure Load Balancer 配置 TCP 空闲超时设置

在默认配置中，Azure Load Balancer 的空闲超时设置为 4 分钟。如果处于非活动状态的时间超过超时值，则不能保证在客户端和云服务之间保持 TCP 或 HTTP 会话。

当连接关闭时，客户端应用程序可能收到以下错误消息：“基础连接已关闭: 应保持连接状态的连接已由服务器关闭”。

常见的做法是使用 TCP 保持连接状态。这种做法可以将连接状态保持更长时间。有关详细信息，请参阅 [.NET 示例](https://msdn.microsoft.com/zh-cn/library/system.net.servicepoint.settcpkeepalive.aspx)。在启用保持连接状态的情况下，在连接处于非活动状态时发送数据包。这些用于保持连接状态的数据包可以确保始终达不到空闲超时值，于是就可以长时间维持连接。

此设置仅适用于入站连接。为了避免断开连接，必须将 TCP 保持连接的时间间隔配置为小于空闲超时设置，或者增加空闲超时值。我们已允许对空闲超时进行配置，以便支持这样的情况。现在，你可以将空闲超时的持续时间设置为 4 到 30 分钟。

TCP 保持连接状态非常适用于不受电池寿命限制的情况。不建议将其用于移动应用程序。在移动应用程序中使用 TCP 保持连接状态可能会加快设备电池的耗尽速度。

![TCP 超时](./media/load-balancer-tcp-idle-timeout/image1.png)  


以下章节介绍了如何更改虚拟机和云服务中的空闲超时设置

## 将实例级公共 IP 的 TCP 超时值配置为 15 分钟

    Set-AzurePublicIP -PublicIPName webip -VM MyVM -IdleTimeoutInMinutes 15

`IdleTimeoutInMinutes` 是可选项。如果未设置，默认超时为 4 分钟。可接受的超时范围为 4 到 30 分钟。

## 在虚拟机上创建 Azure 终结点时设置空闲超时

若要更改终结点的超时设置，请执行以下命令：

    Get-AzureVM -ServiceName "mySvc" -Name "MyVM1" | Add-AzureEndpoint -Name "HttpIn" -Protocol "tcp" -PublicPort 80 -LocalPort 8080 -IdleTimeoutInMinutes 15| Update-AzureVM

若要检索空闲超时配置，请执行以下命令：

    PS C:\> Get-AzureVM -ServiceName "MyService" -Name "MyVM" | Get-AzureEndpoint
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

## 在负载均衡的终结点集上设置 TCP 超时

如果终结点是负载均衡的终结点集的一部分，则必须在负载均衡的终结点集上设置 TCP 超时。例如：

    Set-AzureLoadBalancedEndpoint -ServiceName "MyService" -LBSetName "LBSet1" -Protocol tcp -LocalPort 80 -ProbeProtocolTCP -ProbePort 8080 -IdleTimeoutInMinutes 15

## 更改云服务的超时设置

可以使用 Azure SDK 来更新云服务。在 .csdef 文件中进行云服务的终结点设置。更新云服务部署的 TCP 超时需要升级部署。例外情况是如果仅针对公共 IP 指定 TCP 超时，则无需升级。公共 IP 设置位于 .cscfg 文件中，可以通过部署更新和升级进行更新。

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

## REST API 示例

可以通过使用服务管理 API 配置 TCP 空闲超时。请确保 `x-ms-version` 标头设置为版本 `2014-06-01` 或更高版本。通过一次部署，在所有虚拟机上更新指定的负载均衡的输入终结点的配置。

### 请求

    POST https://management.core.chinacloudapi.cn/<subscription-id>/services/hostedservices/<cloudservice-name>/deployments/<deployment-name>

### 响应

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

[Internal load balancer overview（内部负载均衡器概述）](/documentation/articles/load-balancer-internal-overview/)

[开始配置面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)

[配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

<!---HONumber=Mooncake_0109_2017-->