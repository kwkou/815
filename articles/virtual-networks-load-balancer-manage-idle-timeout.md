<properties authors="danielceckert" documentationCenter="dev-center-name" editor="" manager="jefco" pageTitle="管理：负载平衡器空闲超时" description="Azure 负载平衡器空闲超时的管理功能" services="virtual-network" />

<tags ms.service="virtual-network" ms.date="02/20/2015" wacn.date="04/11/2015"/>

# 管理虚拟网络：负载平衡器 TCP 空闲超时

**TCP 空闲超时**允许开发人员指定一个确定的阈值，该阈值针对客户端-服务器会话期间出现的与 Azure 负载平衡器相关的非活动状态。TCP 空闲超时值为 4 分钟（Azure 负载平衡器的默认值）意味着，如果在客户端-服务器会话期间出现的与 Azure 负载平衡器相关的非活动状态的持续时间超过 4 分钟，则会关闭连接。

当客户端-服务器连接关闭时，客户端应用程序会获得一条类似于下面这样的错误消息："基础连接已关闭:本应保持活动的连接已被服务器关闭"。

长时间处于非活动状态时，通常使用 [TCP Keep-Alive](http://tools.ietf.org/html/rfc1122#page-101) 来保持连接 [（MSDN 示例）](https://msdn.microsoft.com/zh-CN/library/system.net.servicepoint.settcpkeepalive.aspx)。使用 TCP Keep-Alive 时，客户端会定期发送简单的数据包（其频率周期通常短于服务器的空闲超时阈值）。服务器会将这些传输内容视为存在连接活动的证据，即使除传输本身外并没有出现其他活动，这样就永远达不到空闲超时值，因此就可以长时间维持连接。

虽然 TCP Keep-Alive 很有效，但通常不适用于移动应用程序，因为它会消耗移动设备上有限的能源。使用 TCP Keep-Alive 的移动应用程序会更快地耗尽设备电池的能源，因为它会持续地汲取需要在网络上使用的能源。

Azure 负载平衡器支持对 TCP 空闲超时进行配置，因此支持移动设备方案。对于入站连接，开发人员可以将 TCP 空闲超时设置为 4 到 30 分钟（可配置的 TCP 空闲超时不适用于出站连接）。这样客户端就可以与服务器维持很长时间的会话，即使长时间处于非活动状态也不会断开连接。移动设备上的应用程序仍然可以选择利用 TCP Keep-Alive 技术来保留非活动时间预计会超过 30 分钟的连接，不过在使用这种空闲时间更长的 TCP 空闲超时时，应用程序发出 TCP Keep-Alive 请求的频率要比以前低得多，因此会大大降低移动设备能源的消耗。

## 实现

TCP 空闲超时可以针对以下情况进行配置： 

* [实例级公共 IP](https://msdn.microsoft.com/zh-CN/library/azure/dn690118.aspx)
* [负载平衡的终结点集](https://msdn.microsoft.com/zh-CN/library/azure/dn655055.aspx)
* [虚拟机终结点](/documentation/articles/virtual-machines-set-up-endpoints)
* [Web 角色](https://msdn.microsoft.com/zh-CN/library/windowsazure/ee758711.aspx)
* [辅助角色](https://msdn.microsoft.com/zh-CN/library/windowsazure/ee758711.aspx)

## 后续步骤
* TBD

## PowerShell 示例
请下载[最新版 Azure PowerShell](https://github.com/Azure/azure-sdk-tools/releases) 以获取最佳结果。

### 将你的实例级公共 IP 的 TCP 超时配置为 15 分钟

    Set-AzurePublicIP -PublicIPName webip -VM MyVM -IdleTimeoutInMinutes 15

IdleTimeoutInMinutes 为可选。在未设置的情况下，默认超时为 4 分钟。其值现在可以设置为 4 到 30 分钟。

### 在虚拟机上创建 Azure 终结点时设置空闲超时

    Get-AzureVM -ServiceName "mySvc" -Name "MyVM1" | Add-AzureEndpoint -Name "HttpIn" -Protocol "tcp" -PublicPort 80 -LocalPort 8080 -IdleTimeoutInMinutes 15| Update-AzureVM

### 检索空闲超时配置

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
    
### 在负载平衡终结点集上设置 TCP 超时

如果终结点是负载平衡终结点集的一部分，则必须在负载平衡终结点集上设置 TCP 超时：

    Set-AzureLoadBalancedEndpoint -ServiceName "MyService" -LBSetName "LBSet1" -Protocol tcp -LocalPort 80 -ProbeProtocolTCP -ProbePort 8080 -IdleTimeoutInMinutes 15

## 云服务示例

你可以利用 Azure SDK for .NET 来更新云服务

云服务的终结点设置在 .csdef 中进行。因此，若要更新云服务部署中的 TCP 超时，部署升级是必需的。例外情况是仅针对公共 IP 指定 TCP 超时。公共 IP 设置位于 .cscfg 中，可以通过部署更新和升级进行更新。

终结点设置的 .csdef 更改如下：

    <WorkerRole name="worker-role-name" vmsize="worker-role-size" enableNativeCodeExecution="[true|false]">
      <Endpoints>
        <InputEndpoint name="input-endpoint-name" protocol="[http|https|tcp|udp]" localPort="local-port-number" port="port-number" certificate="certificate-name" loadBalancerProbe="load-balancer-probe-name" idleTimeoutInMinutes="tcp-timeout" />
      </Endpoints>
    </WorkerRole>The .cscfg changes for the timeout setting on Public IPs are:
    
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
    
## API 示例

开发人员可以使用服务管理 API 来配置负载平衡器分布。请确保添加的 x-ms-version 标头设置为 2014-06-01 或更高版本。

### 通过一次部署，在所有虚拟机上更新指定的负载平衡输入终结点的配置

#### 请求

    POST https://management.core.chinacloudapi.cn/<subscription-id>/services/hostedservices/<cloudservice-name>/deployments/<deployment-name>

LoadBalancerDistribution 的值可以是 sourceIP（在 2 元组关联的情况下）、sourceIPProtocol（在 3 元组关联的情况下）或无（在没有关联的情况下，即 5 元组）

#### 响应

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

<!--HONumber=51-->