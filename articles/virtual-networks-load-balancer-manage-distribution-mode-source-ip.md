<properties authors="danielceckert" documentationCenter="dev-center-name" editor="" manager="jefco" pageTitle="管理：负载平衡器分发模式（源 IP 关联）" description="Azure 负载平衡器分发模式的管理功能" services="virtual-network"/>

<tags ms.service="virtual-network" ms.date="02/20/2015" wacn.date="04/11/2015"/>
   
# 管理虚拟网络：负载平衡器分发模式（源 IP 关联）

**源 IP 关联**（也称为**"会话关联"**或**"客户端 IP 关联"**）是一种 Azure 负载平衡器分发模式，它将来自单个客户端的连接绑定到 Azure 托管的单个服务器，而不是将每个客户端连接动态分发到 Azure 托管的不同服务器（默认负载平衡器行为）。

使用源 IP 关联，Azure 负载平衡器可以配置为使用 2 元组组合（源 IP、目标 IP）或 3 元组组合（源 IP、目标 IP、协议）将流量映射到 Azure 托管的可用服务器池。使用源 IP 关联时，从同一客户端计算机发起的连接由单个 DIP 终结点（Azure 托管的单个服务器）处理。

## 服务源

源 IP 关联解决了以前 [Azure 负载平衡器与 RD 网关 (DOC) 之间的不兼容](http://download.microsoft.com/download/E/A/7/EA75F19F-63F1-401A-8021-13AE2E6D8196/Microsoft%20Azure%20Desktop%20Hosting%20Reference%20Architecture%20Guide-Nov2014.docx)。

## 实现

可以为以下项配置源 IP 关联： 

* [虚拟机终结点](/documentation/articles/virtual-machines-set-up-endpoints)
* [负载平衡的终结点集](https://msdn.microsoft.com/zh-CN/library/azure/dn655055.aspx)
* [Web 角色](https://msdn.microsoft.com/zh-CN/library/azure/ee758711.aspx)
* [辅助角色](https://msdn.microsoft.com/zh-CN/library/azure/ee758711.aspx)

## 方案
1. 使用单个云服务的远程桌面网关群集
2. 媒体上载（即，UDP 用于数据，TCP 用于控制）
  * 客户端启动与 Azure 托管的负载平衡公共 IP 地址的 TCP 会话。
  * 客户端请求将由负载平衡器定向到某个 DIP；此通道将保持处于活动状态，以监视连接健康状况
  * 客户端启动与 Azure 托管的同一负载平衡公共 IP 地址的 UDP 会话
  * Azure 负载平衡器将请求定向到与 TCP 连接相同的 DIP 终结点
  * 客户端将使用更高的 UDP 吞吐量上载媒体，同时通过 TCP 维护控制通道，以保证可靠性
  
## 注意事项
* 如果更改了负载平衡集（即添加或删除虚拟机），客户端通道分布将重新计算；来自现有客户端的新连接可能由不是原来使用的服务器处理
* 使用源 IP 关联可能会导致跨 Azure 托管的服务器的流量不均匀分布
* 通过代理路由流量的客户端可能被 Azure 负载平衡器视为单个客户端

## PowerShell 示例
为获得最佳结果，请下载[最新版的 Azure PowerShell](https://github.com/Azure/azure-sdk-tools/releases)。

### 将 Azure 终结点添加到虚拟机并设置负载平衡器分发模式


    Get-AzureVM -ServiceName "mySvc" -Name "MyVM1" | Add-AzureEndpoint -Name "HttpIn" -Protocol "tcp" -PublicPort 80 -LocalPort 8080 -LoadBalancerDistribution "sourceIP"| Update-AzureVM  

LoadBalancerDistribution 可以设置为 sourceIP（用于 2 元组（源 IP、目标 IP）负载平衡）、sourceIPProtocol（用于 3 元组（源 IP、目标 IP、协议）负载平衡）或 none（如果想要默认行为（5 元组负载平衡））。  

### 检索终结点负载平衡器分发模式配置

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
    LoadBalancerDistribution : sourceIP

如果 LoadBalancerDistribution 元素不存在，则 Azure 负载平衡器使用默认的 5 元组算法

### 在负载平衡终结点集上设置分发模式

    Set-AzureLoadBalancedEndpoint -ServiceName "MyService" -LBSetName "LBSet1" -Protocol tcp -LocalPort 80 -ProbeProtocolTCP -ProbePort 8080 -LoadBalancerDistribution "sourceIP"
    
如果终结点是负载平衡终结点集的一部分，则必须在负载平衡终结点集上设置分发模式。

## 云服务示例

你可以利用 Azure SDK for .NET 来更新云服务

云服务的终结点设置在 .csdef 中进行。若要更新云服务部署的负载平衡器分发模式，需要进行部署升级。

下面是终结点设置的 .csdef 更改的示例：

    <WorkerRole name="worker-role-name" vmsize="worker-role-size" enableNativeCodeExecution="[true|false]">
      <Endpoints>
        <InputEndpoint name="input-endpoint-name" protocol="[http|https|tcp|udp]" localPort="local-port-number" port="port-number" certificate="certificate-name" loadBalancerProbe="load-balancer-probe-name" loadBalancerDistribution="sourceIP" />
      </Endpoints>
    </WorkerRole>
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

开发人员可以使用服务管理 API 配置负载平衡器分发。确保添加 x-ms-version 标头并将其设为版本 2014-09-01 或更高版本。

### 更新部署中指定的负载平衡集配置

#### 请求

    POST https://management.core.chinacloudapi.cn/<subscription-id>/services/hostedservices/<cloudservice-name>/deployments/<deployment-name>?comp=UpdateLbSet 
    
    x-ms-version: 2014-09-01 
    
    Content-Type: application/xml 
    
    <LoadBalancedEndpointList xmlns="http://schemas.microsoft.com/windowsazure" xmlns:i="http://www.w3.org/2001/XMLSchema-instance"> 
      <InputEndpoint> 
        <LoadBalancedEndpointSetName> endpoint-set-name </LoadBalancedEndpointSetName> 
        <LocalPort> local-port-number </LocalPort> 
        <Port> external-port-number </Port> 
        <LoadBalancerProbe> 
          <Port> port-assigned-to-probe </Port> 
          <Protocol> probe-protocol </Protocol> 
          <IntervalInSeconds> interval-of-probe </IntervalInSeconds> 
          <TimeoutInSeconds> timeout-for-probe </TimeoutInSeconds> 
        </LoadBalancerProbe> 
        <Protocol> endpoint-protocol </Protocol> 
        <EnableDirectServerReturn> enable-direct-server-return </EnableDirectServerReturn> 
        <IdleTimeoutInMinutes>idle-time-out</IdleTimeoutInMinutes> 
        <LoadBalancerDistribution>sourceIP</LoadBalancerDistribution> 
      </InputEndpoint> 
    </LoadBalancedEndpointList>

LoadBalancerDistribution 的值可以是 sourceIP（用于 2 元组关联）、sourceIPProtocol（用于 3 元组关联）或 none（用于无关联，即 5 元组）

#### 响应

    HTTP/1.1 202 Accepted 
    Cache-Control: no-cache 
    Content-Length: 0 
    Server: 1.0.6198.146 (rd_rdfe_stable.141015-1306) Microsoft-HTTPAPI/2.0 
    x-ms-servedbyregion: ussouth2 
    x-ms-request-id: 9c7bda3e67c621a6b57096323069f7af 
    Date: Thu, 16 Oct 2014 22:49:21 GMT

<!--HONumber=51-->