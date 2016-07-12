<properties 
	pageTitle="连接到 SQL Server 虚拟机（经典）| Azure"
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
	wacn.date="05/24/2016"/>

# 连接到 Azure 上的 SQL Server 虚拟机（经典部署）

## 概述

连接到 Azure 虚拟机上运行的 SQL Server 与连接到本地 SQL Server 所要执行的配置步骤差别不大。你仍然需要完成涉及防火墙、身份验证和数据库登录的配置步骤。

但在 SQL Server 连接方面，有一些 Azure VM 特定的设置。本文将介绍一些[常规连接方案](#connection-scenarios)，并提供[在 Azure VM 中配置 SQL Server 连接的详细步骤](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)。

本文的重点在于如何连接已存在的使用经典模型部署的 SQL 服务器虚拟机。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

##<a name="connection-scenarios"></a> 连接方案

客户端连接虚拟机上运行的 SQL Server 的方式取决于客户端的位置与计算机/网络配置。这些方案包括：

- [连接到同一云服务中的 SQL Server](#connect-to-sql-server-in-the-same-cloud-service)
- [通过 Internet 连接到 SQL Server](#connect-to-sql-server-over-the-internet)
- [连接到同一虚拟网络中的 SQL Server](#connect-to-sql-server-in-the-same-virtual-network)

###<a name="connect-to-sql-server-in-the-same-cloud-service"></a> 连接到同一云服务中的 SQL Server

可以在同一云服务中创建多个虚拟机。若要了解此虚拟机方案，请参阅[如何将虚拟机连接到虚拟网络或云服务](/documentation/articles/virtual-machines-windows-classic-connect-vms/)。

首先，根据[此文章中的步骤来配置连接](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)。请注意，如果要连接同一云服务中的计算机，则不需要设置公共终结点。

可以在客户端连接字符串中使用 VM **主机名**。主机名是你在创建 VM 期间为 VM 指定的名称。例如，如果 SQL VM 名为 **mysqlvm**，云服务 DNS 名称为 **mycloudservice.chinacloudapp.cn**，则同一云服务中的客户端 VM 可以使用以下连接字符串进行连接：

	"Server=mysqlvm;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

###<a name="connect-to-sql-server-over-the-internet"></a> 通过 Internet 连接到 SQL Server

如果你想要通过 Internet 连接到 SQL Server 数据库引擎，则必须创建虚拟机终结点以进行传入 TCP 通信。此 Azure 配置步骤将传入 TCP 端口通信定向到虚拟机可以访问的 TCP 端口。

首先，根据[此文章中的步骤来配置连接](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)。然后，可访问 Internet 的任何客户端可通过指定云服务 DNS 名称（例如 **mycloudservice.chinacloudapp.cn**）和 VM 终结点（例如 **57500**）来连接到 SQL Server 实例。

	"Server=mycloudservice.chinacloudapp.cn,57500;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

尽管客户端可通过 Internet 进行连接，但这并不意味着任何人都可以连接到 SQL Server。外部客户端必须有正确的用户名和密码。为了提高安全性，请不要对公共虚拟机终结点使用常用的 1433 端口。如果可能，请考虑在终结点上添加 ACL 以将流量限制为你允许的客户端。有关在终结点上使用 ACL 的说明，请参阅[管理终结点上的 ACL](/documentation/articles/virtual-machines-windows-classic-setup-endpoints/#manage-the-acl-on-an-endpoint)。

>[AZURE.NOTE] 请务必注意，使用这种方法与 SQL Server 进行通信时，所有返回的数据被将视为数据中心的传出流量。这需要按一般的[出站数据传输价格](/pricing/details/data-transfer)付费。即使在同一 Azure 数据中心内的另一个计算机或云服务中使用这种方法，也是如此，因为流量还是会通过 Azure 的公共负载平衡器。

###<a name="connect-to-sql-server-in-the-same-virtual-network"></a> 连接到同一虚拟网络中的 SQL Server

[虚拟网络](/documentation/articles/virtual-networks-overview/)支持其他方案。你可以连接同一虚拟网络中的 VM，即使这些 VM 位于不同的云服务中。

虚拟网络还可让你将 Azure VM 加入域。这是对 SQL Server 使用 Windows 身份验证的唯一方式。其他连接方案需要使用用户名和密码进行 SQL 身份验证。

首先，根据[此文章中的步骤来配置连接](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)。如果你要配置域环境和 Windows 身份验证，则不需要使用本文中的步骤来配置 SQL 身份验证和登录。此外，在此方案中，公共终结点不是必需的。

假设你已配置 DNS，你可以在连接字符串中指定 SQL Server VM 的主机名来连接 SQL Server 实例。以下示例假设同时已配置 Windows 身份验证，并且用户已获得访问 SQL Server 实例的权限。

	"Server=mysqlvm;Integrated Security=true" 

请注意，在此方案中，你还可以指定 VM 的 IP 地址。

##<a name="steps-for-configuring-sql-server-connectivity-in-an-azure-vm"></a> 在 Azure VM 中配置 SQL Server 连接的步骤

以下步骤演示如何使用 SQL Server Management Studio (SSMS) 通过 Internet 连接到 SQL Server 实例。但是，这些步骤同样适用于使你的 SQL Server 虚拟机可以通过本地和 Azure 中运行的应用程序访问。

你必须先完成下列各部分中描述的下列任务，然后才能从其他 VM 或 Internet 连接到 SQL Server 的实例：

- [为虚拟机创建 TCP 终结点](#create-a-tcp-endpoint-for-the-virtual-machine)
- [在 Windows 防火墙中打开 TCP 端口](#open-tcp-ports-in-the-windows-firewall-for-the-default-instance-of-the-database-engine)
- [将 SQL Server 配置为侦听 TCP 协议](#configure-sql-server-to-listen-on-the-tcp-protocol)
- [配置混合模式的 SQL Server 身份验证](#configure-sql-server-for-mixed-mode-authentication)
- [创建 SQL Server 身份验证登录名](#create-sql-server-authentication-logins)
- [确定虚拟机的 DNS 名称](#determine-the-dns-name-of-the-virtual-machine)
- [从其他计算机连接到数据库引擎](#connect-to-the-database-engine-from-another-computer)

下图中概述了此连接路径：

![连接到 SQL Server 虚拟机](./media/virtual-machines-sql-server-connection-steps/SQLServerinVMConnectionMap.png)

[AZURE.INCLUDE [连接到 VM 中的 SQL Server（经典 TCP 终结点）](../includes/virtual-machines-sql-server-connection-steps-classic-tcp-endpoint.md)]

[AZURE.INCLUDE [连接到 VM 中的 SQL Server](../includes/virtual-machines-sql-server-connection-steps.md)]

[AZURE.INCLUDE [连接到 VM 中的 SQL Server（经典步骤）](../includes/virtual-machines-sql-server-connection-steps-classic.md)]

## 后续步骤

如果你还打算针对高可用性和灾难恢复使用 AlwaysOn 可用性组，你应该考虑实施侦听器。数据库客户端将连接到侦听器，而不是直接连接到一个 SQL Server 实例。侦听器将客户端路由到可用性组中的主副本。有关详细信息，请参阅[在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器](/documentation/articles/virtual-machines-windows-classic-ps-sql-int-listener/)。

请务必查看 Azure 虚拟机上运行的 SQL Server 的所有安全最佳实践。有关详细信息，请参阅 [Azure 虚拟机中 SQL Server 的安全注意事项](/documentation/articles/virtual-machines-windows-sql-security/)。

有关其他与在 Azure VM 中运行 SQL Server 相关的主题，请参阅 [Azure 虚拟机上的 SQL Server](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

<!---HONumber=Mooncake_0215_2016-->