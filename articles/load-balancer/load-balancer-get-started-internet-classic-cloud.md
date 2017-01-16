<properties
    pageTitle="开始在经典部署模型中为云服务创建面向 Internet 的负载均衡器 | Azure"
    description="了解如何在经典部署模型中为云服务创建面向 Internet 的负载均衡器"
    services="load-balancer"
    documentationcenter="na"
    author="kumudd"
    manager="timlt"
    tags="azure-service-management" />
<tags
    ms.assetid="0bb16f96-56a6-429f-88f5-0de2d0136756"
    ms.service="load-balancer"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/17/2016"
    wacn.date="01/13/2017"
    ms.author="kumud" />

# 开始为云服务创建面向 Internet 的负载均衡器
> [AZURE.SELECTOR]
- [Azure 经典管理门户](/documentation/articles/load-balancer-get-started-internet-classic-portal/)
- [PowerShell](/documentation/articles/load-balancer-get-started-internet-classic-ps/)
- [Azure CLI](/documentation/articles/load-balancer-get-started-internet-classic-cli/)
- [Azure 云服务](/documentation/articles/load-balancer-get-started-internet-classic-cloud/)

[AZURE.INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

> [AZURE.IMPORTANT]
在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：Azure Resource Manager 部署模型和经典部署模型。在使用任何 Azure 资源前，请确保了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍经典部署模型。你还可以[了解如何使用 Azure Resource Manager 创建面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)。

将自动为云服务配置负载均衡器，并可以通过服务模型自定义云服务。

## 使用服务定义文件创建负载均衡器

你可以利用用于 .NET 2.5 的 Azure SDK 来更新云服务。云服务的终结点设置在[服务定义](https://msdn.microsoft.com/zh-cn/library/azure/gg557553.aspx) .csdef 文件中进行。

下面的示例演示如何配置云部署的 servicedefinition.csdef 文件：

检查云部署生成的 .csdef 文件的代码片段，可以看到配置的外部终结点，以便在端口 10000、10001 和 10002 上使用端口 HTTP。

    <ServiceDefinition name="Tenant">
        <WorkerRole name="FERole" vmsize="Small">
    <Endpoints>
        <InputEndpoint name="FE_External_Http" protocol="http" port="10000" />
        <InputEndpoint name="FE_External_Tcp"  protocol="tcp"  port="10001" />
        <InputEndpoint name="FE_External_Udp"  protocol="udp"  port="10002" />

        <InputEndpointname="HTTP_Probe" protocol="http" port="80" loadBalancerProbe="MyProbe" />

        <InstanceInputEndpoint name="InstanceEP" protocol="tcp" localPort="80">
            <AllocatePublicPortFrom>
                <FixedPortRange min="10110" max="10120"  />
            </AllocatePublicPortFrom>
        </InstanceInputEndpoint>
        <InternalEndpoint name="FE_InternalEP_Tcp" protocol="tcp" />
    </Endpoints>
        </WorkerRole>
    </ServiceDefinition>

## <a name="check-load-balancer-health-status-for-cloud-services"></a> 检查云服务的负载均衡器运行状况状态

下面是运行状况探测器的示例：

    <LoadBalancerProbes>
        <LoadBalancerProbe name="MyProbe" protocol="http" path="Probe.aspx" intervalInSeconds="5" timeoutInSeconds="100" />
    </LoadBalancerProbes>

负载均衡器组合了终结点信息和探测器信息，以便以 `http://{DIP of VM}:80/Probe.aspx` 格式创建可用于查询服务的运行状况的 URL。

该服务通过同一 IP 地址检测定期探测。这是来自正在运行虚拟机的节点主机的运行状况探测请求。
该服务必须对负载均衡器以 HTTP 200 状态代码进行响应，以假设该服务处于正常状态。任何其他 HTTP 状态代码（例如 503）会直接将虚拟机从轮转列表中删除。

探测器定义还控制探测的频率。在上面的示例中，负载均衡器每隔 5 秒探测一次终结点。如果在 10 秒（两个探测时间间隔）内未收到肯定应答，则认为探测器关闭，并且虚拟机将从轮转列表中删除。同样，如果该服务已退出轮转列表并且收到肯定应答，则该服务将立即回到轮转列表。如果该服务在正常和不正常之间波动，负载均衡器可以决定推迟将该服务重新引入到轮转列表，直到多次探测的结果为正常。

有关详细信息，请查看[运行状况探测](https://msdn.microsoft.com/zh-cn/library/azure/jj151530.aspx)的服务定义架构。

## 后续步骤

[开始配置内部负载均衡器](/documentation/articles/load-balancer-get-started-ilb-arm-ps/)

[配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载均衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0109_2017-->