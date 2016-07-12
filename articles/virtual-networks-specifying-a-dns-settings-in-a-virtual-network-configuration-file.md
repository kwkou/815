<properties 
   pageTitle="在虚拟网络配置文件中指定 DNS 设置 | Azure"
   description="在经典部署模型中，如何使用虚拟网络配置文件更改虚拟网络中的 DNS 服务器设置"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn" 
   tags="azure-service-management" />
<tags
	ms.service="virtual-network"
	ms.date="02/23/2016"
	wacn.date="04/25/2016"/> 


# 在虚拟网络配置文件中指定 DNS 设置

网络配置文件有两个可用于指定域名系统 (DNS) 设置的元素：**DnsServers** 和 **DnsServerRef**。可以通过指定服务器的 IP 地址和 **DnsServers** 元素的引用名添加 DNS 服务器列表。然后可以使用 **DnsServerRef** 元素指定 DnsServers 元素中的哪些 DNS 服务器实体用于虚拟网络内的不同网络站点。

[AZURE.INCLUDE [azure-arm-classic-important-include](../includes/azure-arm-classic-important-include.md)]

网络配置文件可能包含以下元素。每个元素的标题链接到提供有关元素值设置的其他信息的页中。

>[AZURE.IMPORTANT] 有关如何配置网络配置文件的信息，请参阅[使用网络配置文件配置虚拟网络](/documentation/articles/virtual-networks-using-network-configuration-file/)。有关网络配置文件中包含的每个元素的信息，请参阅 [Azure 虚拟网络配置架构](https://msdn.microsoft.com/zh-cn/library/azure/jj157100.aspx)。

[Dns 元素](https://msdn.microsoft.com/zh-cn/library/azure/jj157100)

    <Dns>
      <DnsServers>
        <DnsServer name="ID1" IPAddress="IPAddress1" />
        <DnsServer name="ID2" IPAddress="IPAddress2" />
        <DnsServer name="ID3" IPAddress="IPAddress3" />
      </DnsServers>
    </Dns>

>[AZURE.WARNING] **DnsServer** 元素中的 **name** 属性仅用作 **DnsServerRef** 元素的引用。它不表示 DNS 服务器的主机名。每个 **DnsServer** 属性值必须在整个 Azure 订阅中是唯一的

[虚拟网络站点元素](https://msdn.microsoft.com/zh-cn/library/azure/jj157100)

	<DnsServersRef>
	  <DnsServerRef name="ID1" />
	  <DnsServerRef name="ID2" />
	  <DnsServerRef name="ID3" />
	</DnsServersRef>

>[AZURE.NOTE] 为了指定虚拟网络站点元素的此设置，它必须先前就在 DNS 元素中进行定义。虚拟网络站点元素中的 DnsServerRef *name* 必须引用 DNS 元素中针对 DnsServer *name* 指定的名称值。

## 后续步骤

- 了解 [Azure 虚拟网络配置架构](https://msdn.microsoft.com/zh-cn/library/azure/jj157100)。
- 了解 [Azure 服务配置架构](https://msdn.microsoft.com/zh-cn/library/azure/ee758710)。
- [使用网络配置文件配置虚拟网络](/documentation/articles/virtual-networks-using-network-configuration-file/)。

<!---HONumber=Mooncake_0418_2016-->