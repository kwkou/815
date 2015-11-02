<properties 
   pageTitle="VM 和角色实例的解析"
   description="Azure IaaS、混合解决方案、不同的云服务之间、Active Directory 和使用自己的 DNS 服务器的名称解析方案"
   services="virtual-network"
   documentationCenter="na"
   authors="joaoma"
   manager="jdial"
   editor="tysonn"/>
<tags 
   ms.service="virtual-network"
   ms.date="09/02/2015"
   wacn.date="10/17/2015" />

# VM 和角色实例的名称解析

具体取决于如何使用 Azure 托管 IaaS、PaaS 和混合解决方案，你可能需要允许 VM 和创建的角色实例彼此进行通信。尽管这种通信可以通过使用 IP 地址完成，但使用容易记住且不会更改的名称要简单得多。

当 Azure 中托管的角色实例和 VM 需要将域名解析到内部 IP 地址时，它们可以使用两种方法之一：

- [Azure 提供的名称解析](#azure-provided-name-resolution)

- [使用你自己的 DNS 服务器的名称解析](#name-resolution-using-your-own-dns-server)

使用的名称解析类型取决于 VM 和角色实例需要彼此进行通信的方式。

**下表说明了方案和相应的名称解析解决方案：**

| **方案** | **提供的名称解析：** | **有关详细信息，请参阅：** |
|--------------|----------------------------------|-------------------------------|
| 位于相同云服务中的角色实例或 VM 之间的名称解析 | Azure 提供的名称解析 | [Azure 提供的名称解析](#azure-provided-name-resolution)|
| 位于相同虚拟网络中的 VM 或角色实例之间的名称解析 | Azure 提供的名称解析（仅限基于 ARM 的部署）或者使用你自己的 DNS 服务器的名称解析 | [Azure 提供的名称解析](#azure-provided-name-resolution)、[使用你自己的 DNS 服务器的名称解析](#name-resolution-using-your-own-dns-server)和 [DNS 服务器要求](#dns-server-requirements) |
| 位于不同虚拟网络中的 VM 和角色实例之间的名称解析 | 使用你自己的 DNS 服务器的名称解析| [使用你自己的 DNS 服务器的名称解析](#name-resolution-using-your-own-dns-server)和 [DNS 服务器要求](#dns-server-requirements)|
| 跨界：Azure 和本地计算机中的角色实例或 VM 之间的名称解析| 使用你自己的 DNS 服务器的名称解析| [使用你自己的 DNS 服务器的名称解析](#name-resolution-using-your-own-dns-server)和 [DNS 服务器要求](#dns-server-requirements)|
| 反向查找的内部 IP| 使用你自己的 DNS 服务器的名称解析| [使用你自己的 DNS 服务器的名称解析](#name-resolution-using-your-own-dns-server)和 [DNS 服务器要求](#dns-server-requirements)|
| 非公共域（例如 Active Directory 域）的名称解析| 使用你自己的 DNS 服务器的名称解析| [使用你自己的 DNS 服务器的名称解析](#name-resolution-using-your-own-dns-server)和 [DNS 服务器要求](#dns-server-requirements)|
| 位于不同云服务（而非虚拟网络）中的角色实例之间的名称解析| 不适用。不同云服务中的 VM 和角色实例之间的连接在虚拟网络外部不受支持。| 不适用。|


## Azure 提供的名称解析

除公共 DNS 名称解析之外，Azure 还为驻留在相同虚拟网络或云服务中的 VM 和角色实例提供内部名称解析。一个云服务中的 VM/实例共享同一个 DNS 后缀，因此只需主机名就足够了。在经典虚拟网络中，不同云服务具有不同的 DNS 后缀，因此需要 FQDN。在基于 ARM 的虚拟网络中，整个虚拟网络的 DNS 后缀是相同的，因此不需要 FQDN，并且可以将 DNS 名称分配到 NIC 或虚拟机。虽然 Azure 提供的名称解析不需要任何配置，但并不适合所有部署方案，如上表所示。

> [AZURE.NOTE]在 Web 和辅助角色的情况下，还可以基于使用 Azure 服务管理 REST API 的角色名称和实例数访问内部 IP 地址。有关详细信息，请参阅[服务管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx)。

### 功能和注意事项

**功能：**

- 易于使用：不需要配置就能使用 Azure 提供的名称解析。

- Azure 提供的名称解析服务高度可用，你无需创建和管理你自己的 DNS 服务器的群集。

- 在相同云服务中的角色实例或 VM 之间提供名称解析，无需 FQDN。

- 在基于 ARM 的虚拟网络中的 VM 之间提供名称解析，无需 FQDN，经典虚拟网络在解析不同云服务中的名称时需要 FQDN。

- 你可以创建最能描述你的部署的主机名，而不是使用自动生成的名称。

**注意事项：**

- 在虚拟网络之间以及在 Azure 和本地计算机之间无法使用名称解析。

- 不能修改 Azure 创建的 DNS 后缀。

- 不能手动注册你自己的记录。

- 不支持 WINS 和 NetBIOS。（不能在 Windows 资源管理器中的网络浏览器中列出虚拟机。）

- 主机名必须与 DNS 兼容（它们必须仅使用 0-9、a-z 和“-”，并且不能以“-”开始或结束。请参见 RFC 3696 第 2 节。）

- DNS 查询流量按照 VM 进行限制。这不应影响大部分应用程序。如果遵循请求限制，请确保启用客户端缓存。有关详细信息，请参阅[充分利用 Azure 提供的名称解析](#Getting-the-most-from-Azure-provided-name-resolution)。

- 每个经典虚拟网络仅注册前 180 个云服务中的 VM。这不适用于基于 ARM 的虚拟网络。


### 充分利用 Azure 提供的名称解析
**客户端缓存：**

不是每个 DNS 查询都需要跨网络发送。通过解析本地缓存中的重复性 DNS 查询，客户端缓存有助于减少延迟和提高网络信号的恢复能力。DNS 记录包含生存时间 (TTL)，这允许缓存尽可能长时间存储记录，而不影响记录刷新，因此客户端缓存适用于大多数情况。

默认 Windows DNS 客户端具有内置的 DNS 缓存。默认情况下，某些 Linux 发行版不包括缓存，建议每个 Linux VM 都添加缓存。有许多不同 DNS 缓存包可用，例如 dnsmasq，下面是在最常见的发行版上安装 dnsmasq 的步骤：

- **Ubuntu（使用 resolvconf）**：
	- 只安装 dnsmasq 包（“sudo apt-get install dnsmasq”）。
- **SUSE（使用 netconf）**：
	- 安装 dnsmasq 包（“sudo zypper install dnsmasq”） 
	- 启用 dnsmasq 服务（“systemctl enable dnsmasq.service”） 
	- 启动 dnsmasq 服务（“systemctl start dnsmasq.service”） 
	- 编辑“/etc/sysconfig/network/config”，并将 NETCONFIG\_DNS\_FORWARDER="" 更改为“dnsmasq”
	- 更新 resolv.conf（“netconfig update”），以将缓存设置为本地 DNS 解析程序
- **OpenLogic（使用 NetworkManager）**：
	- 安装 dnsmasq 包（“sudo yum install dnsmasq”）
	- 启用 dnsmasq 服务（“systemctl enable dnsmasq.service”）
	- 启动 dnsmasq 服务（“systemctl start dnsmasq.service”）
	- 将“prepend domain-name-servers 127.0.0.1;”添加到“/etc/dhclient-eth0.conf”
	- 重新启动网络服务（“service network restart”），以将缓存设置为本地 DNS 解析程序

[AZURE.NOTE]：该“dnsmasq”包只是适用于 Linux 的众多 DNS 缓存中的一个。在使用之前，请检查其是否适合你的特定需求，并且确认你没有安装其他缓存。

**客户端重试：**

DNS 主要是一个 UDP 协议。因为 UDP 协议无法保证消息传递，所以重试逻辑在DNS 协议本身中处理。每个 DNS 客户端（操作系统）可能会表现出不同的重试逻辑，具体取决于创建者偏好：

 - Windows 操作系统在 1 秒后重试，然后在再 2 秒后、再 4 秒后和另一个 4 秒后再次重试。 
 - 默认 Linux 设置在 5 秒后重试。建议将此更改为重试 5 次，每次间隔 1 秒。  

若要检查 Linux VM 上的当前设置，“cat /etc/resolv.conf”并查看“options”行，例如：

	options timeout:1 attempts:5

resolv.conf 文件通常是自动生成的，不应进行编辑。添加“options”行的具体步骤因发行版而异：

- **Ubuntu**（使用 resolvconf）：
	- 将“options”行添加到“/etc/resolveconf/resolv.conf.d/head” 
	- 运行“resolvconf -u”以更新
- **SUSE**（使用 netconf）：
	- 将“timeout:1 attempts:5”添加到“/etc/sysconfig/network/config”中的 NETCONFIG\_DNS\_RESOLVER\_OPTIONS="" 参数 
	- 运行“netconfig update”以更新
- **OpenLogic**（使用 NetworkManager）：
	- 将“echo "options timeout:1 attempts:5"”添加到“/etc/NetworkManager/dispatcher.d/11-dhclient” 
	- 运行“service network restart”以更新

## 使用你自己的 DNS 服务器的名称解析

如果名称解析要求超出 Azure 提供的功能，则可以选择使用你自己的 DNS 服务器。当你使用自己的 DNS 服务器时，你负责管理 DNS 记录。

> [AZURE.NOTE]建议避免使用外部 DNS 服务器，除非部署方案需要。

## DNS 服务器要求

如果计划使用不由 Azure 提供的名称解析，指定的 DNS 服务器则必须支持以下情况：

- DNS 服务器应通过动态 DNS (DDNS) 协议接受动态的 DNS 注册，或者你必须创建所需的记录。

- 如果依赖动态 DNS，则在 DNS 服务器上应关闭记录清理。在 Azure 中，IP 地址具有长期 DHCP 租约，这可能会导致在清理过程中删除 DNS 服务器上的记录。

- DNS 服务器必须已启用递归，以允许解析外部域名。

- DNS 服务器必须通过请求名称解析的客户端和通过将注册其名称的服务和虚拟机可访问（TCP/UDP 端口 53 上）。

- 另外，当许多机器人扫描打开的递归 DNS 解析器时，建议保护 DNS 服务器免于通过 Internet 访问。

- 为了获得最佳性能，在将 Azure VM用作 DNS 服务器时，应禁用 IPv6，并且[实例层级公共 IP](/documentation/articles/virtual-networks-instance-level-public-ip) 应分配给每个 DNS 服务器 VM。

## 指定 DNS 服务器

可以指定 VM 和角色实例使用的多个 DNS 服务器。对于每个 DNS 查询，客户端将首先尝试使用首选 DNS 服务器，并且仅当首选 DNS 服务器没有响应时才尝试使用备用服务器，即 DNS 查询在不同 DNS 服务器间不是负载平衡的。为此，验证是否按照正确顺序为环境列出 DNS 服务器。

> [AZURE.NOTE]如果更改已部署的虚拟网络的网络配置文件上的 DNS 设置，则需要重新启动每个 VM，所做的更改才会生效。

### 在管理门户中指定 DNS 服务器

在管理门户中创建虚拟网络时，可以指定想要使用的 DNS 服务器的 IP 地址和名称。创建完虚拟网络后，你部署到虚拟网络的虚拟机和角色实例会使用指定的 DNS 设置自动配置。为特定云服务（Azure 经典）指定的 DNS 服务器或网络接口卡（基于 ARM 的部署）优先于为虚拟网络指定的 DNS 服务器。请参阅[关于在管理门户中配置虚拟网络](/documentation/articles/virtual-networks-settings)。

### 使用配置文件指定 DNS 服务器（Azure 经典）

对于经典虚拟网络，可以使用两个不同的配置文件指定 DNS 设置：*网络配置*文件和*服务配置*文件。

网络配置文件描述你订阅中的虚拟网络。当你将角色实例或 VM 添加到虚拟网络中的云服务中时，网络配置文件的 DNS 设置则应用到每个角色实例或 VM，除非已指定特定于云服务的 DNS 服务器。

为添加到 Azure 的每个云服务创建服务配置文件。当你将角色实例或 VM 添加到云服务中时，服务配置文件的 DNS 设置则应用到每个角色实例或 VM。

> [AZURE.NOTE]服务配置文件中的 DNS 服务器重写网络配置文件中的设置。


## 后续步骤

[Azure 服务配置架构](https://msdn.microsoft.com/zh-cn/library/azure/ee758710)

[虚拟网络配置架构](https://msdn.microsoft.com/zh-cn/library/azure/jj157100)

[关于在管理门户中配置虚拟网络设置](/documentation/articles/virtual-networks-settings)

[使用网络配置文件配置虚拟网络](/documentation/articles/virtual-networks-using-network-configuration-file)

<!---HONumber=74-->