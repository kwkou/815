<properties
    pageTitle="在两个虚拟网络之间配置 VPN 连接 | Azure"
    description="了解如何在两个虚拟网络之间配置 VPN 连接和域名解析，以及如何配置 HBase 异地复制。"
    services="hdinsight,virtual-network"
    documentationcenter=""
    author="mumian"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="1f47b85e-d5e8-4a4a-843e-1881d969e49f"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="06/28/2016"
    wacn.date="01/25/2017"
    ms.author="jgao" />  


# 在两个 Azure 虚拟网络之间配置 VPN 连接
> [AZURE.SELECTOR]
- [配置 VPN 连接](/documentation/articles/hdinsight-hbase-geo-replication-configure-VNets/)
- [配置 DNS](/documentation/articles/hdinsight-hbase-geo-replication-configure-DNS/)
- [配置 HBase 复制](/documentation/articles/hdinsight-hbase-geo-replication/)

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

Azure 虚拟网络站点到站点连接使用 VPN 网关通过 Ipsec/IKE 提供安全隧道。连接的 VNet 可位于不同的订阅和区域中。甚至可以将 VNet 到 VNet 通信与多站点配置组合使用。建立 VNet 到 VNet 连接有多个原因：

* 跨区域异地冗余和地区存在
* 具有强隔离边界的区域多层应用程序
* 在 Azure 中跨订阅进行组织间通信

有关详细信息，请参阅[配置 VNet 到 VNet 连接](/documentation/articles/virtual-networks-configure-vnet-to-vnet-connection/)。

若要通过视频了解它：

> [!VIDEO https://channel9.msdn.com/Series/Getting-started-with-Windows-Azure-HDInsight-Service/Configure-the-VPN-connectivity-between-two-Azure-virtual-networks/player]
> 
> 

本教程是有关创建 HBase 异地复制的[系列][hdinsight-hbase-replication]教程的一部分。

* 在两个虚拟网络之间配置 VPN 连接（本教程）
* [为虚拟网络配置 DNS][hdinsight-hbase-geo-replication-dns]
* [配置 HBase 异地复制][hdinsight-hbase-geo-replication]

下图说明将要在本教程中创建的两个虚拟网络：

![HDInsight HBase 复制虚拟网络示意图][img-vnet-diagram]  


## 先决条件
在开始阅读本教程前，你必须具有：

* **Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* **配备 Azure PowerShell 的工作站**。
  
    运行 PowerShell 脚本前，确保已使用以下 cmdlet 连接到 Azure 订阅：
  
        Add-AzureAccount -Environment AzureChinaCloud
  
    如果有多个 Azure 订阅，请使用以下 cmdlet 设置当前订阅：
  
        Select-AzureSubscription <AzureSubscriptionName>
  
    [AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

> [AZURE.NOTE]
Azure 服务名称和虚拟机名称均必须唯一。本教程中使用的名称是 Contoso-[Azure Service/VM name]-[CN/CE]。例如，Contoso-VNet-CN 是位于中国北部数据中心的 Azure 虚拟网络；Contoso-DNS-CE 是位于中国东部数据中心的 DNS 服务器 VM。必须选择适合自己的名称。
> 
> 

## 创建两个 Azure VNet
**在中国北部创建名为 Contoso-VNet-CN 的虚拟网络**

1. 登录 [Azure 经典管理门户][azure-portal]。
2. 依次单击“新建”、“网络服务”、“虚拟网络”和“自定义创建”。
3. 输入：
   
    * **名称**：Contoso-VNet-CN
    * **位置**：中国北部
     
        本教程使用中国北部和中国东部数据中心。
4. 输入：
   
    * **DNS 服务器**：（保留空白）
     
        需要使用自己的 DNS 服务器在虚拟网络中进行名称解析。有关何时使用 Azure 提供的名称解析和自己的 DNS 服务器的详细信息，请参阅[名称解析 (DNS)](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/)。有关在 VNet 之间配置名称解析的说明，请参阅[在两个 Azure 虚拟网络之间配置 DNS][hdinsight-hbase-dns]。
    * **配置点到站点 VPN**：（未选中）
     
        点到站点连接不适用于此方案。
    * **配置站点到站点 VPN**：（未选中）
     
        将配置与中国东部数据中心的 Azure 虚拟网络的站点到站点 VPN 连接。
5. 输入：
   
    * **地址空间起始 IP 地址**：10.1.0.0
    * **地址空间 CIDR**：/16
    * **Subnet-1 起始 IP 地址**：10.1.0.0
    * **Subnet-1 CIDR**：/24
   
    地址空间不能与美国虚拟网络重叠。

**在中国北部创建名为 Contoso-VNet-CN 的虚拟网络**

* 使用以下值重复上一过程：
  
    * **名称**：Contoso-VNet-CE
    * **位置**：中国东部
    * **DNS 服务器**：（保留空白）
    * **配置点到站点 VPN**：（未选中）
    * **配置站点到站点 VPN**：（未选中）
    * **地址空间起始 IP 地址**：10.2.0.0
    * **地址空间 CIDR**：/16
    * **Subnet-1 起始 IP 地址**：10.2.0.0
    * **Subnet-1 CIDR**：/24

## 在两个 VNet 之间配置 VPN 连接
### 创建本地网络
创建 VNet 到 VNet 配置时，需要配置每个 VNet 以便彼此标识为本地网络站点。在本部分中，要将每个 VNet 配置为本地网络。本地网络与相应 VNet 共享同一个 IP 地址空间。

![配置 Azure VPN 站点到站点配置 - Azure 本地网络][img-vnet-lnet-diagram]  


**创建与 Contoso-VNet-CN 网络地址空间匹配的名为 Contoso-LNet-CN 的本地网络**

1. 在 Azure 经典管理门户中，依次单击“新建”、“网络服务”、“虚拟网络”、“添加本地网络”。
2. 输入：
   
    * **名称**：Contoso-LNet-CN
    * **VPN 设备 IP 地址**：192.168.0.1（稍后将会更新此地址）
     
        通常，应该使用 VPN 设备的实际外部 IP 地址。对于 VNet 到 VNet 配置，将使用 VPN 网关 IP 地址。如果尚未为两个 VNet 创建 VPN 网关，请输入任意 IP 地址，然后返回进行修改。
3. 输入：
   
    * **地址空间起始 IP 地址**：10.1.0.0
    * **地址空间 CIDR**：/16
   
    此值必须完全对应前面为 Contoso-VNet-CN 指定的范围。

**创建与 Contoso-VNet-CE 网络地址空间匹配的名为 Contoso-LNet-CE 的本地网络**

* 使用以下参数重复上一过程：
  
    * **名称**：Contoso-LNet-CE
    * **VPN 设备 IP 地址**：192.168.0.1（稍后将会更新此地址）
    * **地址空间起始 IP 地址**：10.2.0.0
    * **地址空间 CIDR**：/16

### 创建 VPN 网关
此配置包括两个部分。首先需要配置与本地网络的 VNet 站点到站点连接，然后创建动态路由 VPN。VNet 到 VNet 连接需要具有动态路由 VPN 的 Azure VPN 网关。不支持 Azure 静态路由 VPN。

**在 Contoso-VNet-CN 与 Contoso-LNet-CE 之间配置站点到站点连接**

1. 在 Azure 经典管理门户中，单击左窗格中的“网络”，
2. 单击“Contoso-VNet-CN”。
3. 单击“配置”选项卡。
4. 选择“连接到本地网络”。
5. 在“本地网络”中，选择“Contoso-LNet-CE”。
6. 在虚拟网络地址空间部分中，单击“添加网关子网”。
7. 单击“保存”。
8. 单击“确定”确认。

**创建 Contoso-VNet-CN 的 VPN 网关**

1. 在 Azure 经典管理门户中，单击“仪表板”选项卡。
2. 单击页面底部的“创建网关”，然后单击“动态路由”。
3. 单击“是”确认。注意，页面上的网关图形将变为黄色，并显示“正在创建网关”。通常，创建网关需要大约 15 分钟时间。
   
    当网关状态更改为“正在连接”时，每个网关的 IP 地址将显示在仪表板上。写下对应每个 VNet 的 IP 地址，请注意不要混淆。在“本地网络”中编辑 VPN 设备的占位符 IP 地址时，将使用这些 IP 地址。
4. 创建“网关 IP 地址”的副本。在下一节中，将用于配置 Contoso-VNet-CN 的 VPN 网关 IP 地址。

**创建 Contoso-VNet-CN 的 VPN 网关**

* 重复上述两个过程，在 Contoso-VNet-CE 与 Contoso-LNet-CN 之间配置站点到站点连接，并创建 Contoso-Vnet-CE 的 VPN 网关。完成后，将获得 Contoso-VNet-CE 的 VPN 网关 IP 地址。

### 设置本地网络的 VPN 设备 IP 地址
在上一节中，已为每个 VNet 创建一个 VPN 网关。并已获得 VPN 网关的 IP 地址。现在，可以返回配置本地网络的 VPN 设备 IP 地址。

**配置 Contoso-LNet-CN 的 VPN 设备 IP 地址**

1. 在 Azure 经典管理门户中，单击左窗格中的“网络”。
2. 单击顶部的“本地网络”。
3. 单击“Contoso-LNet-CN”，然后单击底部的“编辑”。
4. 更新“VPN 设备 IP 地址”。该地址是从 Contoso-VNET-CN 的“仪表板”选项卡获取的地址。
5. 单击右侧按钮。
6. 单击勾选按钮。

**配置 Contoso-LNet-CE 的 VPN 设备 IP 地址**

* 重复上一过程，配置 Contoso-LNet-CE 的 VPN 设备 IP 地址。

### 设置 VNet 网关密钥
VNet 网关使用共享密钥对虚拟网络之间的连接进行身份验证。无法从 Azure 经典管理门户配置密钥。必须使用 PowerShell 或 .NET SDK 进行配置。

**设置密钥**

1. 在工作站中，打开 **Windows PowerShell ISE** 或 Windows PowerShell 控制台。
2. 更新以下脚本中的参数，然后运行该脚本：
   
        Add-AzureAccount -Environment AzureChinaCloud  
        Select-AzureSubscription -[AzureSubscriptionName]
        Set-AzureVNetGatewayKey -VNetName ContosoVNet-CN -LocalNetworkSiteName Contoso-LNet-CE -SharedKey A1b2C3D4
        Set-AzureVNetGatewayKey -VNetName ContosoVNet-CE -LocalNetworkSiteName Contoso-LNet-CN -SharedKey A1b2C3D4 

## 检查 VPN 连接
如果未将任何 VM 部署到 VNet，可使用 Azure 经典管理门户中“VNet 仪表板”页上的虚拟网络直观图表检查连接状态：

![HDInsight HBase 复制虚拟网络 VPN 连接状态][img-vpn-status]  


## 后续步骤
在本教程中，已学习如何在两个 Azure 虚拟网络之间配置 VPN 连接。本系列教程包括的另外两篇文章：

* [在两个 Azure 虚拟网络之间配置 DNS][hdinsight-hbase-geo-replication-dns]
* [配置 HBase 异地复制][hdinsight-hbase-geo-replication]

[hdinsight-hbase-geo-replication-dns]: /documentation/articles/hdinsight-hbase-geo-replication-configure-DNS/
[hdinsight-hbase-geo-replication]: /documentation/articles/hdinsight-hbase-geo-replication/

[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/
[azure-portal]: https://portal.azure.cn

[powershell-install]: /documentation/articles/powershell-install-configure/


[hdinsight-hbase-replication]: /documentation/articles/hdinsight-hbase-geo-replication/
[hdinsight-hbase-dns]: /documentation/articles/hdinsight-hbase-geo-replication-configure-DNS/

[img-vnet-diagram]: ./media/hdinsight-hbase-geo-replication-configure-vnets/hdinsight-hbase-vpn-diagram.png
[img-vnet-lnet-diagram]: ./media/hdinsight-hbase-geo-replication-configure-vnets/hdinsight-hbase-vpn-lnet-diagram.png
[img-vpn-status]: ./media/hdinsight-hbase-geo-replication-configure-vnets/hdinsight-hbase-vpn-status.png

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update meta properties & wording update-->