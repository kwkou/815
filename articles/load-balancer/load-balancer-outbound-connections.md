<properties
   pageTitle="了解 Azure 中的出站连接 | Microsoft"
   description="本文介绍了 Azure 如何使 VM 与公共 Internet 服务进行通信。"
  services="load-balancer"
   documentationCenter="na"
   authors="sdwheeler"
   manager="carmonm"
   editor="" />
<tags
   ms.assetid="5f666f2a-3a63-405a-abcd-b2e34d40e001"
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="10/31/2016"
   wacn.date="12/05/2016"
   ms.author="sewhee" />

# 了解 Azure 中的出站连接

Azure 中的虚拟机 (VM) 可以与 Azure 外部的公用 IP 地址空间中的终结点进行通信。当 VM 启动到公共 IP 地址空间中的目标的出站流时，Azure 将 VM 的专用 IP 地址映射到公共 IP 地址，并允许返回流量来访问 VM。

Azure 提供三种不同的方法来实现出站连接。每种方法都有自己的功能和限制。仔细查看每种方法以选择符合你的需求的方法。

| 方案 | 方法 | 注意 |
| --- | --- | --- |
| 独立 VM（无负载均衡器，无实例级公共 IP 地址） |默认 SNAT |Azure 关联用于 SNAT 的公共 IP 地址 |
| 负载均衡的 VM（VM 上没有实例级公共 IP 地址） |使用负载均衡器的 SNAT |Azure 对 SNAT 使用负载均衡器的公共 IP 地址 |
| 具有实例级公共 IP 地址的 VM（有或没有负载均衡器） |不使用 SNAT |Azure 使用分配给 VM 的公共 IP |

如果不希望 VM 与 Azure 外部的公共 IP 地址空间中的终结点通信，则可以使用网络安全组 (NSG) 来阻止访问。[阻止公共连接](#preventing-public-connectivity)中详细介绍了 NSG 的使用。

## 独立 VM（无实例级公共 IP 地址）

在此场景中，VM 不是 Azure Load Balancer 池的一部分，并且没有分配给它的实例级公共 IP (ILPIP) 地址。当 VM 创建出站流时，Azure 将此出站流的专用源 IP 地址转换为公共源 IP 地址。用于此出站流的公共 IP 地址是不可配置的。Azure 使用源网络地址转换 (SNAT) 来执行此功能。使用公共 IP 地址的临时端口区分由 VM 产生的各个流。创建流后 SNAT 动态分配临时端口。在此情况下，用于 SNAT 的临时端口被称为 SNAT 端口。

SNAT 端口是可能会被耗尽的有限资源。因此了解它们的使用方式很重要。每个到单个目标 IP 地址的流使用一个 SNAT 端口。对于到相同的目标 IP 地址的多个流，每个流使用一个 SNAT 端口。这可以确保源自相同的公共 IP 地址，并到相同的目标 IP 地址的流的唯一性。每个流到不同的目标 IP 地址的多个流对于每个目标使用一个 SNAT 端口。目标 IP 地址使流具有唯一性。

你可以使用[用于负载均衡器的 Log Analytics](/documentation/articles/load-balancer-monitor-log/) 和[针对 SNAT 端口耗尽消息要监视的警报事件日志](/documentation/articles/load-balancer-monitor-log/#alert-event-log)。如果 SNAT 端口资源已经耗尽，那么在现有流释放 SNAT 端口之前出站流将失败。负载均衡器对于回收 SNAT 端口使用 4 分钟的空闲超时时间。

## 负载均衡的 VM（无实例级公共 IP 地址）

在此场景中，VM 是 Azure Load Balancer 池的一部分。没有分配给 VM 的公共 IP 地址。当负载均衡的 VM 创建出站流时，Azure 将此出站流的专用源 IP 地址转换为公共负载均衡器前端的公共 IP 地址。Azure 使用源网络地址转换 (SNAT) 来执行此功能。使用负载均衡器的公共 IP 地址的临时端口区分由 VM 产生的各个流。创建出站流后 SNAT 动态分配临时端口。在此情况下，用于 SNAT 的临时端口被称为 SNAT 端口。

SNAT 端口是可能会被耗尽的有限资源。因此了解它们的使用方式很重要。每个到单个目标 IP 地址的流使用一个 SNAT 端口。对于到相同的目标 IP 地址的多个流，每个流使用一个 SNAT 端口。这可以确保源自相同的公共 IP 地址，并到相同的目标 IP 地址的流的唯一性。每个流到不同的目标 IP 地址的多个流对于每个目标使用一个 SNAT 端口。目标 IP 地址使流具有唯一性。

你可以使用[用于负载均衡器的 Log Analytics](/documentation/articles/load-balancer-monitor-log/) 和[针对 SNAT 端口耗尽消息要监视的警报事件日志](/documentation/articles/load-balancer-monitor-log/#alert-event-log)。如果 SNAT 端口资源已经耗尽，那么在现有流释放 SNAT 端口之前出站流将失败。负载均衡器对于回收 SNAT 端口使用 4 分钟的空闲超时时间。

## 具有实例级公共 IP 地址的 VM（有或没有负载均衡器）

在此场景中，向 VM 分配了实例级公共 IP (ILPIP)。VM 是否为负载均衡并不重要。如果使用 ILPIP，则不使用源网络地址转换 (SNAT)。VM 将 ILPIP 用于所有出站流。如果应用程序启动很多出站流，并且遇到 SNAT 耗尽的情况，则应考虑分配 ILPIP 以避免 SNAT 限制。

## 发现指定 VM 所使用的公共 IP

有多种方法来确定出站连接的公共源 IP 地址。OpenDNS 提供了一种服务可以向你显示 VM 的公共 IP 地址。使用 nslookup 命令，可以将名称 myip.opendns.com 的 DNS 查询发送到 OpenDNS 解析程序。该服务返回用于发送此查询的源 IP 地址。在 VM 中执行以下查询时，返回的是用于该 VM 的公共 IP。

    nslookup myip.opendns.com resolver1.opendns.com

## <a name="preventing-public-connectivity"></a> 阻止公共连接

有时允许 VM 创建出站流是不可取的，或者可能需要管理哪些目标可以通过出站流进行访问。在此情况下，使用[网络安全组 (NSG)](/documentation/articles/virtual-networks-nsg/) 来管理 VM 可以访问的目标。将 NSG 应用于负载均衡的 VM 时，需要注意[默认标记](/documentation/articles/virtual-networks-nsg/#default-tags)和[默认规则](/documentation/articles/virtual-networks-nsg/#default-rules)。

必须确保 VM 可以接收来自 Azure Load Balancer 的运行状况探测请求。如果 NSG 阻止来自 AZURE\_LOADBALANCER 默认标记的运行状况探测请求，那么 VM 的运行状况探测程序将失败，并且 VM 被标记为停机。负载均衡器停止向此 VM 发送新流。

<!---HONumber=Mooncake_1128_2016-->