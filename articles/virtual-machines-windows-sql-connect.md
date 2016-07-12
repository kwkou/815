<!-- ARM: tested -->

<properties 
	pageTitle="连接到 SQL Server 虚拟机 (Resource Manager) | Azure"
	description="本主题使用通过经典部署模型创建的资源并介绍了如何在 Azure 中连接到虚拟机上运行的 SQL Server。方案根据网络配置和客户端位置的不同而异。"
	services="virtual-machines-windows"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar"    
	tags="azure-service-management"/>
<tags 
	ms.service="virtual-machines-windows"
	ms.date="03/24/2016"
	wacn.date="06/29/2016"/>

# 连接到 Azure (Resource Manager) 上的 SQL Server 虚拟机

> [AZURE.SELECTOR]
- [资源管理器](/documentation/articles/virtual-machines-windows-sql-connect/)
- [经典](/documentation/articles/virtual-machines-windows-classic-sql-connect/)

## 概述

在 Resource Manager 中，连接到 Azure 虚拟机上运行的 SQL Server 与连接到本地 SQL Server 实例所要执行的配置步骤差别不大。你仍然需要完成涉及防火墙、身份验证和数据库登录的配置步骤。

但在 SQL Server 连接方面，有一些 Azure VM 特定的设置。本文将介绍一些[常规连接方案](#connection-scenarios)，并提供[在 Azure VM 中配置 SQL Server 连接的详细步骤](#steps-for-manually-configuring-sql-server-connectivity-in-an-azure-vm)。

本文的重点在于如何连接已存在的使用资源管理器模型部署的 SQL 服务器 VM。关于预配和连接的完整演练，请参阅[使用 Azure PowerShell 预配 SQL Server 虚拟机 (Resource Manager)](/documentation/articles/virtual-machines-windows-ps-sql-create/)。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-rm-include.md)]经典部署模型。

##<a name="connection-scenarios"></a> 连接方案

客户端连接虚拟机上运行的 SQL Server 的方式取决于客户端的位置与计算机/网络配置。这些方案包括：

- [通过 Internet 连接到 SQL Server](#connect-to-sql-server-over-the-internet)
- [连接到同一虚拟网络中的 SQL Server](#connect-to-sql-server-in-the-same-virtual-network)

###<a name="connect-to-sql-server-over-the-internet"></a> 通过 Internet 连接到 SQL Server

若要从 Internet 连接到 SQL Server 数据库引擎，需完成多个必要的步骤，例如配置防火墙、启用 SQL 身份验证以及配置网络安全组。必须设置网络安全组规则，以便在端口 1433 上进行 TCP 通信。

按照[本文中手动配置连接的步骤](#steps-for-manually-configuring-sql-server-connectivity-in-an-azure-vm)来手动配置 SQL Server 和虚拟机。

完成此步骤以后，任何可以访问 Internet 的客户端都可以连接到 SQL Server 实例，只需指定虚拟机的公共 IP 地址或分配到该 IP 地址的 DNS 标签即可。如果 SQL Server 端口为 1433，则不需在连接字符串中进行指定。

	"Server=sqlvmlabel.chinaeast.chinacloudapp.cn;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

尽管客户端可通过 Internet 进行连接，但这并不意味着任何人都可以连接到 SQL Server。外部客户端必须有正确的用户名和密码。为了提高安全性，可以不使用 1433 这个众所周知的端口。例如，如果你将 SQL Server 配置为在端口 1500 上进行侦听并制定了适当的防火墙和网络安全组规则，则可将端口号附加到服务器名称上进行连接，示例如下：

	"Server=sqlvmlabel.chinaeast.chinacloudapp.cn,1500;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

>[AZURE.NOTE] 请务必注意，使用这种方法与 SQL Server 进行通信时，所有返回的数据被将视为数据中心的传出流量。这需要按一般的[出站数据传输价格](/pricing/details/data-transfer/)付费。即使在同一 Azure 数据中心内的另一个计算机或云服务中使用这种方法，也是如此，因为流量还是会通过 Azure 的公共负载平衡器。

###<a name="connect-to-sql-server-in-the-same-virtual-network"></a> 连接到同一虚拟网络中的 SQL Server

[虚拟网络](/documentation/articles/virtual-networks-overview/)支持其他方案。你可以连接同一虚拟网络中的 VM，即使这些 VM 位于不同的资源组中。使用[站点到站点 VPN](/documentation/articles/vpn-gateway-site-to-site-create/)，可以创建连接 VM 与本地网络和计算机的混合体系结构。

虚拟网络还可让你将 Azure VM 加入域。这是对 SQL Server 使用 Windows 身份验证的唯一方式。其他连接方案需要使用用户名和密码进行 SQL 身份验证。

如果你使用门户预览通过 Resource Manager 来预配 SQL Server 虚拟机映像，则可选择“专用”作为 SQL 连接选项，以便在虚拟网络上设置通信所需的适当防火墙规则。如果不是在进行预配，则可按照[本文中手动配置连接的步骤](#steps-for-manually-configuring-sql-server-connectivity-in-an-azure-vm)来手动配置 SQL Server 和虚拟机。但如果你计划配置域环境和 Windows 身份验证，则不需要使用本文中的步骤来配置 SQL 身份验证和登录。你也不需要配置通过 Internet 进行访问的网络安全组规则。

假设你已在虚拟网络中配置 DNS，则可在连接字符串中指定 SQL Server VM 计算机名来连接 SQL Server 实例。以下示例还假设同时已配置 Windows 身份验证，并且用户已获得访问 SQL Server 实例的权限。

	"Server=mysqlvm;Integrated Security=true" 

请注意，在此方案中，你还可以指定 VM 的 IP 地址。

##<a name="steps-for-manually-configuring-sql-server-connectivity-in-an-azure-vm"></a> 在 Azure VM 中手动配置 SQL Server 连接的步骤

以下步骤演示了如何手动设置到 SQL Server 实例的连接，然后选择性地使用 SQL Server Management Studio (SSMS) 通过 Internet 进行连接。请注意，当你在门户预览中选择适当的 SQL Server 连接选项时，系统会为你完成下面的多项步骤。

你必须先完成下列各部分中描述的下列任务，然后才能从其他 VM 或 Internet 连接到 SQL Server 的实例：

- [在 Windows 防火墙中打开 TCP 端口](#open-tcp-ports-in-the-windows-firewall-for-the-default-instance-of-the-database-engine)
- [将 SQL Server 配置为侦听 TCP 协议](#configure-sql-server-to-listen-on-the-tcp-protocol)
- [配置混合模式的 SQL Server 身份验证](#configure-sql-server-for-mixed-mode-authentication)
- [创建 SQL Server 身份验证登录名](#create-sql-server-authentication-logins)
- [配置用于公共 IP 地址的 DNS 标签](#configure-a-dns-label-for-the-public-ip-address)
- [从其他计算机连接到数据库引擎](#connect-to-the-database-engine-from-another-computer)

[AZURE.INCLUDE [连接到 VM 中的 SQL Server](../includes/virtual-machines-sql-server-connection-steps.md)]

[AZURE.INCLUDE [连接到 VM Resource Manager 中的 SQL Server](../includes/virtual-machines-sql-server-connection-steps-resource-manager-nsg-rule.md)]

[AZURE.INCLUDE [连接到 VM Resource Manager 中的 SQL Server](../includes/virtual-machines-sql-server-connection-steps-resource-manager.md)]

## 后续步骤

若要查看预配说明以及这些连接步骤，请参阅 [使用 Azure PowerShell 预配 SQL Server 虚拟机 (Resource Manager)](/documentation/articles/virtual-machines-windows-ps-sql-create/)。

请务必查看 Azure 虚拟机上运行的 SQL Server 的所有安全最佳实践。有关详细信息，请参阅 [Azure 虚拟机中 SQL Server 的安全注意事项](/documentation/articles/virtual-machines-windows-sql-security/)。

有关其他与在 Azure VM 中运行 SQL Server 相关的主题，请参阅 [Azure 虚拟机上的 SQL Server](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

<!---HONumber=Mooncake_0411_2016-->