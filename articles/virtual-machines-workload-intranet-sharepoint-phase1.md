<properties
	pageTitle="SharePoint Intranet 场工作负荷阶段 1：配置 Azure"
	description="在部署仅限 Intranet 的 SharePoint 2013 场（在 Azure 基础结构服务中通过 SQL Server AlwaysOn 可用性组进行）的这个第一阶段，你将创建 Azure 虚拟网络和其他 Azure 基础结构元素。"
	documentationCenter=""
	services="virtual-machines"
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="07/21/2015"
	wacn.date="09/18/2015"/>

# SharePoint Intranet 场工作负荷阶段 1：配置 Azure

在部署仅限 Intranet 的 SharePoint 2013 场（在 Azure 基础结构服务中通过 SQL Server AlwaysOn 可用性组进行）的这个阶段，你将构建出 Azure 网络和存储基础结构。你必须在转到[阶段 2](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2) 之前完成此阶段。请参阅[在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview) 以了解所有阶段的相关信息。

必须使用这些基本网络组件设置 Azure：

- 具有一个子网的跨界虚拟网络。
- 三个 Azure 云服务。
- 一个 Azure 存储帐户，用于存储 VHD 磁盘映像和额外的数据磁盘。

## 开始之前

在开始配置 Azure 组件之前，请填写下表。为了帮助你完成配置 Azure 的过程，请打印此部分并记下所需的信息，或者将此部分复制到一个文档并填写它。

对于虚拟网络 (VNet) 的设置，填写表 V。

项目 | 配置元素 | 说明 | 值
--- | --- | --- | ---
1\. | VNet 名称 | 要分配给 Azure 虚拟网络的名称（示例 SPFarmNet）。 | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
2\. | VNet 位置 | 将包含虚拟网络的 Azure 数据中心。 | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
3\. | 本地网络名称 | 要分配给你的组织网络的名称。 | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
4\. | VPN 设备 IP 地址 | 你的 VPN 设备在 Internet 上的接口的公共 IPv4 地址。与你的 IT 部门协作来确定此地址。 | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
5\. | VNet 地址空间 | 虚拟网络的地址空间（在单个专用地址前缀中定义）。与你的 IT 部门协作来确定此地址空间。 | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
6\. | 第一个最终 DNS 服务器 | 虚拟网络子网的地址空间的第四个可能的 IP 地址（请参阅表 S）。与你的 IT 部门协作来确定这些地址。 | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
7\. | 第二个最终 DNS 服务器 | 虚拟网络子网的地址空间的第五个可能的 IP 地址（请参阅表 S）。与你的 IT 部门协作来确定这些地址。 | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**表 V： 跨界虚拟网络配置**

针对此解决方案的子网，填写表 S。根据虚拟网络地址空间，为子网提供一个友好名称、单个 IP 地址空间和描述性目的。地址空间应采用无类别域际路由选择 (CIDR) 格式，也称为网络前缀格式。例如 10.24.64.0/20。与你的 IT 部门协作，在虚拟网络地址空间中确定此地址空间。

项目 | 子网名称 | 子网地址空间 | 目的
--- | --- | --- | ---
1\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**表 S：虚拟网络中的子网**

> [AZURE.NOTE]为简单起见，此预定义的体系结构使用单个子网。如果要覆盖一组流量筛选器来模拟子网隔离，则可以使用 Azure [网络安全组](/documentation/articles/virtual-networks-nsg)。

对于你最初在虚拟网络中设置域控制器时要使用的两个本地 DNS 服务器，填写表 D。为每个 DNS 服务器提供一个友好名称和单个 IP 地址。此友好名称无需与 DNS 服务器的主机名或计算机名称相匹配。请注意，列出了两个空白条目，但你可以添加更多。与你的 IT 部门协作来确定此列表。

项目 | DNS 服务器的友好名称 | DNS 服务器 IP 地址
--- | --- | ---
1\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
2\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**表 D：本地 DNS 服务器**

若要通过站点到站点 VPN 连接将数据包从跨界网络路由到你的组织网络，必须为虚拟网络配置一个本地网络，该网络包含你的组织的本地网络上的所有可访问位置的地址空间（采用 CIDR 表示法）列表。定义本地网络的地址空间列表不能包含用于其他虚拟网络或其他本地网络的地址空间或与之重叠。换而言之，配置的虚拟网络和本地网络的地址空间必须是唯一的。

对于本地网络地址空间集，填写表 L。请注意，列出了三个空白条目，但通常将需要更多。与你的 IT 部门协作来确定此地址空间列表。

项目 | 本地网络地址空间
--- | ---
1\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
2\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
3\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**表 L：本地网络的地址前缀**

若要使用表 V、S、D 和 L 中的设置创建虚拟网络，请使用[使用配置表创建跨界虚拟网络](/documentation/articles/virtual-machines-workload-deploy-vnet-config-tables)中的说明。

> [AZURE.NOTE]此过程将指导你逐步完成创建使用站点到站点 VPN 连接的虚拟网络。有关将 ExpressRoute 用于站点到站点连接的信息，请参阅 [ExpressRoute 技术概述](/documentation/articles/expressroute-introduction)。

创建 Azure 虚拟网络后，Azure 管理门户将确定以下项：

- 你的虚拟网络的 Azure VPN 网关的公共 IPv4 地址。
- 站点到站点 VPN 连接的 Internet 协议安全性 (IPsec) 预共享密钥。

若要在创建虚拟网络后，在 Azure 管理门户中查看这些项，请单击**“网络”**，单击虚拟网络的名称，然后单击**“仪表板”**菜单选项。

接下来，你将配置虚拟网络网关，以便创建安全的站点到站点 VPN 连接。有关说明，请参阅[在管理门户中配置虚拟网络网关](/documentation/articles/vpn-gateway-configure-vpn-gateway-mp)。

接下来，在新的虚拟网络与本地 VPN 设备之间创建站点到站点 VPN 连接。有关详细信息，请参阅[在管理门户中配置虚拟网络网关](/documentation/articles/vpn-gateway-configure-vpn-gateway-mp)中的说明。

接下来，请确保可从你的本地网络访问虚拟网络的地址空间。这通常是通过将与虚拟网络地址空间对应的路由添加到你的 VPN 设备，然后将该路由播发到你的组织网络的其余路由基础结构来完成的。与你的 IT 部门协作来确定执行此操作的方法。

接下来，请按[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell) 中的说明在本地计算机上安装 Azure PowerShell。打开 Azure PowerShell 命令提示符。

首先，使用以下命令选择相应的 Azure 订阅。将引号内的所有内容（包括 < and > 字符）替换为相应的名称。

	$subscr="<Subscription name>"
	Select-AzureSubscription -SubscriptionName $subscr –Current

你可以从 **Get-AzureSubscription** 命令输出的 **SubscriptionName** 属性获取订阅名称。

接下来，创建此 SharePoint 场所需的三个云服务。填写表 C。

项目 | 目的 | 云服务名称
--- | --- | ---
1\. | 域控制器 | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
2\. | SQL Server | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
3\. | SharePoint Server | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**表 C：云服务名称**

必须为每个云服务选取唯一名称。*云服务名称只能包含字母、数字和连字符。该字段中的第一个和最后一个字符必须是字母或数字。*

例如，你可以将第一个云服务命名为 DCs-*UniqueSequence*，其中 *UniqueSequence* 是你组织的缩写。例如，如果你的组织名为 Tailspin Toys，则可以将云服务命名为 DCs-Tailspin。

你可以使用以下 Azure PowerShell 命令在本地计算机上测试该名称的唯一性。

	Test-AzureName -Service <Proposed cloud service name>

如果此命令返回“False”，则你建议的名称是唯一的。然后，使用以下命令创建云服务。

	New-AzureService -Service <Unique cloud service name> -Location "<Table V – Item 2 – Value column>"

在表 C 中记录每个新创建的云服务的实际名称。

接下来，创建用于 SharePoint 场的存储帐户。*必须选择只包含小写字母和数字的唯一名称。* 你可以使用以下 Azure PowerShell 命令测试存储帐户名称的唯一性。

	Test-AzureName -Storage <Proposed storage account name>

如果此命令返回“False”，则你建议的名称是唯一的。然后，使用这些命令创建存储帐户并设置订阅以使用它。

	$staccount="<Unique storage account name>"
	New-AzureStorageAccount -StorageAccountName $staccount -Location "<Table V – Item 2 – Value column>"
	Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $subscr -CurrentStorageAccountName $staccount

接下来，定义四个可用性集的名称。填写表 A。

项目 | 目的 | 可用性集名称
--- | --- | ---
1\. | 域控制器 | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
2\. | SQL Server | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
3\. | SharePoint 应用程序服务器 | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
4\. | SharePoint 前端 Web 服务器 | \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**表 A：可用性集名称**

在阶段 2、阶段 3 和阶段 4 中创建虚拟机时将需要这些名称。

这是成功完成此阶段后生成的配置。

![](./media/virtual-machines-workload-intranet-sharepoint-phase1/workload-spsqlao_01.png)

## 后续步骤

若要继续配置此工作负荷，请转到[阶段 2：配置域控制器](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2)。

## 其他资源

[在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview)

[Azure 基础结构服务中托管的 SharePoint 场](/documentation/articles/virtual-machines-sharepoint-infrastructure-services)

[具有 SQL Server AlwaysOn 的 SharePoint 信息图](http://go.microsoft.com/fwlink/?LinkId=394788)

[适用于 SharePoint 2013 的 Windows Azure 体系结构](https://technet.microsoft.com/zh-cn/library/dn635309.aspx)

[Azure 基础结构服务实施准则](/documentation/articles/virtual-machines-infrastructure-services-implementation-guidelines)

[Azure 基础结构服务工作负荷：高可用性业务线应用程序](/documentation/articles/virtual-machines-workload-high-availability-LOB-application)

<!---HONumber=70-->