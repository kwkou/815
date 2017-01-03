<properties
    pageTitle="排查应用程序网关的网关无效 (502) 错误 | Azure"
    description="了解如何排查应用程序网关 502 错误"
    services="application-gateway"
    documentationcenter="na"
    author="amitsriva"
    manager="rossort"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="1d431ead-d318-47d8-b3ad-9c69f7e08813"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="12/16/2016"
    wacn.date="01/03/2017"
    ms.author="amsriva" />  


# 排查应用程序网关中的网关无效错误

## 概述

配置 Azure 应用程序网关之后，用户可能遇到的一个错误是“服务器错误: 502 - Web 服务器在作为网关或代理服务器时收到了无效响应”。此错误可能是以下主要原因造成的：

* Azure 应用程序网关的后端池未配置或为空。
* VM 规模集中没有正常运行的 VM 或实例。
* VM 规模集的后端 VM 或实例未响应默认的运行状况探测。
* 自定义运行状况探测的配置无效或不正确。
* 请求超时，或用户请求出现连接问题。

## BackendAddressPool 为空

### 原因

如果应用程序网关的后端地址池中未配置 VM 或 VM 规模集，则无法路由任何客户请求，并引发网关无效错误。

### 解决方案

确保后端地址池不为空。这可以通过 PowerShell、CLI 或门户来实现。

    Get-AzureRmApplicationGateway -Name "SampleGateway" -ResourceGroupName "ExampleResourceGroup"

上述 cmdlet 的输出应包含非空后端地址池。以下示例中返回了两个池，其中配置了后端 VM 的 FQDN 或 IP 地址。BackendAddressPool 的预配状态必须是 'Succeeded'。

BackendAddressPoolsText:

    [{
        "BackendAddresses": [{
            "ipAddress": "10.0.0.10",
            "ipAddress": "10.0.0.11"
        }],
        "BackendIpConfigurations": [],
        "ProvisioningState": "Succeeded",
        "Name": "Pool01",
        "Etag": "W/\"00000000-0000-0000-0000-000000000000\"",
        "Id": "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/applicationGateways/<application gateway name>/backendAddressPools/pool01"
    }, {
        "BackendAddresses": [{
            "Fqdn": "xyx.chinacloudapp.cn",
            "Fqdn": "abc.chinacloudapp.cn"
        }],
        "BackendIpConfigurations": [],
        "ProvisioningState": "Succeeded",
        "Name": "Pool02",
        "Etag": "W/\"00000000-0000-0000-0000-000000000000\"",
        "Id": "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/applicationGateways/<application gateway name>/backendAddressPools/pool02"
    }]

## BackendAddressPool 中存在运行不正常的实例

### 原因

如果 BackendAddressPool 的所有实例都运行不正常，则应用程序网关不包含任何要将用户请求路由到其中的后端。当后端实例运行正常但尚未部署所需的应用程序时，也可能会发生此情况。

### 解决方案

确定实例正常运行且已正确配置了应用程序。检查后端实例是否能够从同一个 VNet 中的另一个 VM 响应 ping。如果实例中配置了公共终结点，请确保能够为发送到 Web 应用程序的浏览器请求提供服务。

## 默认运行状况探测出现问题

### 原因

此外，出现 502 错误经常意味着默认的运行状况探测无法访问后端 VM。预配某个应用程序网关实例时，该实例会使用 BackendHttpSetting 的属性自动将默认的运行状况探测配置到每个 BackendAddressPool。无需用户输入即可设置此探测。具体而言，在配置负载均衡规则时，将在 BackendHttpSetting 与 BackendAddressPool 之间建立关联。默认探测是针对其中每个关联配置的，而应用程序网关将在 BackendHttpSetting 元素中指定的端口上，与 BackendAddressPool 中每个实例发起周期性运行状况检查连接。下表列出了与默认运行状况探测关联的值。

| 探测属性 | 值 | 说明 |
| --- | --- | --- |
| 探测 URL |http://127.0.0.1/  
 |URL 路径 |
| 间隔 |30 |探测间隔（秒） |
| 超时 |30 |探测超时（秒） |
| 不正常阈值 |3 |探测重试计数。连续探测失败计数达到不正常阈值后，将后端服务器标记为故障。 |

### 解决方案

* 确定默认站点已配置且正在侦听 127.0.0.1。
* 如果 BackendHttpSetting 指定的端口不是 80，则应将默认站点配置为侦听指定的端口。
* 对 http://127.0.0.1:port 的调用应返回 HTTP 结果代码 200。应在 30 秒超时期限内返回此代码。
* 确保配置的端口已打开，并且没有任何防火墙或 Azure 网络安全组在配置的端口上阻止传入或传出流量。
* 如果对 Azure 经典 VM 或云服务使用 FQDN 或公共 IP，请确保打开相应的[终结点](/documentation/articles/virtual-machines-windows-classic-setup-endpoints/)。
* 如果 VM 是通过 Azure Resource Manager 配置的并且位于应用程序网关部署所在的 VNet 的外部，则必须将[网络安全组](/documentation/articles/virtual-networks-nsg/)配置为允许在所需端口上进行访问。

## 自定义运行状况探测出现问题

### 原因

自定义运行状况探测能够对默认探测行为提供更大的弹性。使用自定义探测时，用户可以配置探测间隔、要测试的 URL 和路径，以及在将后端池实例标记为不正常之前可接受的失败响应次数。添加了以下附加属性。

| 探测属性 | 说明 |
| --- | --- |
| 名称 |探测的名称。此名称用于在后端 HTTP 设置中引用探测。 |
| 协议 |用于发送探测的协议。探测使用后端 HTTP 设置中定义的协议 |
| 主机 |用于发送探测的主机名。仅当应用程序网关上配置了多站点时才适用。这与 VM 主机名不同。 |
| 路径 |探测的相对路径。有效路径以“/”开头。探测将发送到 <协议>://<主机>:<端口><路径> |
| 时间间隔 |探测间隔（秒）。这是每两次连续探测之间的时间间隔。 |
| 超时 |探测超时（秒）。如果在此超时期间内未收到有效响应，则将探测标记为失败。 |
| 不正常阈值 |探测重试计数。连续探测失败计数达到不正常阈值后，将后端服务器标记为故障。 |

### 解决方案

根据上表验证是否已正确配置自定义运行状况探测。除了上述故障排除步骤以外，另请确保符合以下要求：

* 确保已根据[指南](/documentation/articles/application-gateway-create-probe-ps/)正确指定了探测。
* 如果在应用程序网关中设置了单站点，则默认情况下，除非已在自定义探测中进行配置，否则应将主机名指定为“127.0.0.1”。
* 确保对 http://\<主机>:<端口><路径> 的调用返回 HTTP 结果代码 200。
* 确保 Interval、Time-out 和 UnhealtyThreshold 都在可接受的范围内。

## 请求超时

### 原因

收到用户请求后，应用程序网关会将配置的规则应用到该请求，然后将其路由到后端池实例。应用程序网关将等待一段可配置的时间间隔，以接收后端实例做出的响应。默认情况下，此间隔为 **30 秒**。如果应用程序网关在此时间间隔内未收到后端应用程序的响应，则用户请求将出现 502 错误。

### 解决方案

应用程序网关允许用户通过 BackendHttpSetting 配置此设置，然后可将此设置应用到不同的池。不同的后端池可以有不同的 BackendHttpSetting，因此可配置不同的请求超时。

    New-AzureRmApplicationGatewayBackendHttpSettings -Name 'Setting01' -Port 80 -Protocol Http -CookieBasedAffinity Enabled -RequestTimeout 60

## 后续步骤

如果上述步骤无法解决问题，请开具[支持票证](/support/contact/)。

<!---HONumber=Mooncake_1226_2016-->