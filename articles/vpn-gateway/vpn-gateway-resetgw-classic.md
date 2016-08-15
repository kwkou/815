<properties
   pageTitle="重置 Azure VPN 网关 | Azure"
   description="本文将指导你完成重置 Azure VPN 网关。本文适用于使用典型部署模型创建的 VPN 网关。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>

<tags
   ms.service="vpn-gateway"
   ms.date="04/26/2016"
   wacn.date="06/24/2016"/>

# 使用 PowerShell 重置 Azure VPN 网关


本文将指导你完成使用 PowerShell cmdlet 重置 Azure VPN 网关。这些说明适用于经典部署模型。我们还没有 cmdlet 或 REST API 来为使用资源管理器部署模型创建的虚拟网络重置 VPN 网关。这些都正在开发中。你可以通过在 Azure 门户预览中查看虚拟网络来确定 VPN 网关是否是使用经典部署模型创建的。使用经典部署模型创建的虚拟网络显示在 Azure 门户预览的虚拟网络（经典）部分中。

重置 Azure VPN 网关，十分有助于丢失一个或多个 S2S VPN 隧道上的跨界 VPN 连接的情况。在此情况下，你的本地 VPN 设备都在正常工作，但却无法与 Azure VPN 网关建立 IPsec 隧道。当你使用 Reset-AzureVNetGateway cmdlet 时，它将重新启动网关，然后对其重新应用跨界配置。该网关将保持已有的公共 IP 地址。这意味着不需要使用 Azure VPN 网关的新公共 IP 地址更新 VPN 路由器配置。


在重置网关之前，请为每个 IPsec 站点到站点 (S2S) VPN 隧道验证下面列出的重要项目。如果项目中存在任何不匹配，将导致 S2S VPN 隧道断开连接。验证并更正你的本地和 Azure VPN 网关的配置，将能够避免网关上其他正在工作的连接出现不必要的重新启动和中断。

在重置网关之前，请验证以下各项。

- 在 Azure 和本地 VPN 策略中，为 Azure VPN 网关和本地 VPN 网关配置的 Internet IP 地址 (VIP) 正确。
- 在 Azure 和本地 VPN 网关上，预共享的密钥必须相同。
- 如果应用特定的 IPsec/IKE 配置，如加密、哈希算法和 PFS（完全向前保密），请确保 Azure 和本地 VPN 网关具有相同配置。


## 使用 PowerShell 重置 VPN 网关

用于重置 Azure VPN 网关的 PowerShell cmdlet 是 Reset-AzureVNetGateway。每个 Azure VPN 网关由运行在活动备用配置中的 VM 实例组成。一旦发出命令时，将立即重新启动 Azure VPN 网关的当前活动实例。从活动实例（正在重新启动）故障转移到备用实例期间将有短暂的间隔。该时间间隔应不超过 1 分钟。

下面的示例将重置名为“ContosoVNet”的虚拟网络的 Azure VPN 网关。
 
	Reset-AzureVNetGateway –VnetName “ContosoVNet” 
	
	Error          :
	HttpStatusCode : OK
	Id             : f1600632-c819-4b2f-ac0e-f4126bec1ff8
	Status         : Successful
	RequestId      : 9ca273de2c4d01e986480ce1ffa4d6d9
	StatusCode     : OK


如果在首次重新启动后未恢复连接，再次发出同一命令以重新启动第二个 VM 实例（新活动网关）。如果连续请求两次重新启动，则重新启动这两个 VM 实例（活动和备用）的时间可能会略长一些。这种情况将导致 VPN 连接出现较长的时间间隔，VM 需要最多 2 到 4 分钟才能完成重新启动。

在两次重新启动之后，如果你仍然遇到跨界连接问题，请从 Azure 经典管理门户开具支持票证以联系 Azure 支持。


## 后续步骤
	
有关此 cmdlet 的详细信息，请参阅 [PowerShell 参考](https://msdn.microsoft.com/zh-cn/library/azure/mt270366.aspx)。







<!---HONumber=Mooncake_0613_2016-->
