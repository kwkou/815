<properties
    pageTitle="有关用于跨界 Azure 连接的 VPN 设备 | Azure"
    description="本文讨论用于 S2S VPN 网关跨界连接的 VPN 设备和 IPsec 参数。 提供了指向配置说明和示例的链接。"
    services="vpn-gateway"
    documentationcenter="na"
    author="yushwang"
    manager="rossort"
    editor=""
    tags="azure-resource-manager, azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="ba449333-2716-4b7f-9889-ecc521e4d616"
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/03/2017"
    wacn.date="04/17/2017"
    ms.author="yushwang;cherylmc"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="f0f30889954cf2fdf6bed49f6d712785c5edcfbe"
    ms.lasthandoff="04/06/2017" />

# <a name="about-vpn-devices-and-ipsecike-parameters-for-site-to-site-vpn-gateway-connections"></a>关于用于站点到站点 VPN 网关连接的 VPN 设备和 IPsec/IKE 参数

通过 VPN 网关配置站点到站点 (S2S) 跨界 VPN 连接需要用到 VPN 设备。 在创建混合解决方案时，或者每当你想要在本地网络与虚拟网络之间建立安全连接时，可以使用站点到站点连接。 本文提供了适用于 Azure VPN 网关的 IPsec/IKE 参数的列表，以及连接到 Azure VPN 网关的有效 VPN 设备的列表。

> [AZURE.IMPORTANT]
> 如果遇到本地 VPN 设备与 Azure VPN 网关之间的连接问题，请参阅[已知的设备兼容性问题](#known)。 

###<a name="items-to-note-when-viewing-the-tables"></a>查看表时的注意事项：

* Azure VPN 网关的术语已更改。 这两个术语你可能都会碰到。 无功能性更改，只是名称进行了更改。
    * 静态路由 = PolicyBased
    * 动态路由 = RouteBased
* 除非另有说明，否则高性能 VPN 网关和 RouteBased VPN 网关的规范是相同的。 例如，经验证与 RouteBased VPN 网关兼容的 VPN 设备也与 Azure 高性能 VPN 网关兼容。

> [AZURE.NOTE]
> 配置站点到站点连接时，需要为你的 VPN 设备提供面向公众的 IPv4 IP 地址。                                                                                                                                                                               

## <a name="devicetable"></a>已验证的 VPN 设备
我们在与设备供应商合作的过程中验证了一系列标准 VPN 设备。 以下列表中包含的设备系列中的所有设备都应适用于 Azure VPN 网关。 请参阅 [关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/) 以确定需要为要配置的解决方案创建的网关类型。

若要获取配置 VPN 设备的帮助，请参考各设备系列相对应的链接。 如需 VPN 设备支持，请联系设备制造商。

|**供应商**          |**设备系列**     |**最低操作系统版本** |**PolicyBased** |**RouteBased** |
| ---                | ---                  | ---                   | ---            | ---           |
| A10 Networks, Inc. |Thunder CFW           |ACOS 4.1.1             |不兼容  |[配置指南](https://www.a10networks.com/resources/deployment-guides/a10-thunder-cfw-ipsec-vpn-interoperability-azure-vpn-gateways)|
| Allied Telesis     |AR 系列 VPN 路由器 |2.9.2                  |即将支持     |不兼容  |
| Barracuda Networks, Inc. |Barracuda NextGen Firewall F 系列 |PolicyBased：5.4.3<br>RouteBased：6.2.0 |[配置指南](https://techlib.barracuda.com/NGF/AzurePolicyBasedVPNGW) |[配置指南](https://techlib.barracuda.com/NGF/AzureRouteBasedVPNGW) |
| Barracuda Networks, Inc. |Barracuda NextGen Firewall X 系列 |Barracuda Firewall 6.5 |[配置指南](https://techlib.barracuda.com/BFW/ConfigAzureVPNGateway) |不兼容 |
| Brocade            |Vyatta 5400 vRouter   |Virtual Router 6.6R3 GA|[配置指南](http://www1.brocade.com/downloads/documents/html_product_manuals/vyatta/vyatta_5400_manual/wwhelp/wwhimpl/js/html/wwhelp.htm#href=VPN_Site-to-Site%20IPsec%20VPN/Preface.1.1.html) |不兼容 |
| 检查点 |安全网关 |R77.30 |[配置指南](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk101275) |[配置指南](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk101275) |
| Cisco              |ASA       |8.3 |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASA) |不兼容 |
| Cisco |ASR |PolicyBased：IOS 15.1<br>RouteBased：IOS 15.2 |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASR) |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASR) |
| Cisco |ISR |PolicyBased：IOS 15.0<br>RouteBased*：IOS 15.1 |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ISR) |[配置示例*](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ISR) |
| Citrix |NetScaler MPX、SDX、VPX |10.1 及以上 |[配置指南](https://docs.citrix.com/netscaler/11-1/system/cloudbridge-connector-introduction/cloudbridge-connector-azure.html) |不兼容 |
| Dell SonicWALL |TZ 系列、NSA 系列<br>SuperMassive 系列<br>E 类 NSA 系列 |SonicOS 5.8.x<br>SonicOS 5.9.x<br>SonicOS 6.x |[SonicOS 6.2 配置指南](http://documents.software.dell.com/sonicos/6.2/microsoft-azure-configuration-guide?ParentProduct=646)<br>[SonicOS 5.9 配置指南](http://documents.software.dell.com/sonicos/5.9/microsoft-azure-configuration-guide?ParentProduct=850) |[SonicOS 6.2 配置指南](http://documents.software.dell.com/sonicos/6.2/microsoft-azure-configuration-guide?ParentProduct=646)<br>[SonicOS 5.9 配置指南](http://documents.software.dell.com/sonicos/5.9/microsoft-azure-configuration-guide?ParentProduct=850) |
| F5 |BIG-IP 系列 |12.0 |[配置指南](https://devcentral.f5.com/articles/connecting-to-windows-azure-with-the-big-ip) |[配置指南](https://devcentral.f5.com/articles/big-ip-to-azure-dynamic-ipsec-tunneling) |
| Fortinet |FortiGate |FortiOS 5.4.2 |[配置指南](http://cookbook.fortinet.com/ipsec-vpn-microsoft-azure-54) |[配置指南](http://cookbook.fortinet.com/ipsec-vpn-microsoft-azure-54) |
| Internet Initiative Japan (IIJ) |SEIL 系列 |SEIL/X 4.60<br>SEIL/B1 4.60<br>SEIL/x86 3.20 |[配置指南](http://www.iij.ad.jp/biz/seil/ConfigAzureSEILVPN.pdf) |不兼容 |
| Juniper |SRX |PolicyBased：JunOS 10.2<br>Routebased：JunOS 11.4 |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SRX) |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SRX) |
| Juniper |J 系列 |PolicyBased：JunOS 10.4r9<br>RouteBased：JunOS 11.4 |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/JSeries) |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/JSeries) |
| Juniper |ISG |ScreenOS 6.3 |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/ISG) |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/ISG) |
| Juniper |SSG |ScreenOS 6.2 |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SSG) |[配置示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SSG) |
| Microsoft |路由和远程访问服务 |Windows Server 2012 |不兼容 |[配置示例](http://go.microsoft.com/fwlink/p/?LinkId=717761) |
| 打开系统 AG |任务控制安全网关 |不适用 |[配置指南](https://www.open.ch/_pdf/Azure/AzureVPNSetup_Installation_Guide.pdf) |[配置指南](https://www.open.ch/_pdf/Azure/AzureVPNSetup_Installation_Guide.pdf) |
| Openswan |Openswan |2.6.32 |（即将支持） |不兼容 |
| Palo Alto Networks |运行 PAN-OS 的所有设备 |PAN-OS<br>PolicyBased：6.1.5 或更高版本<br>RouteBased：7.1.4 |[配置指南](https://live.paloaltonetworks.com/t5/Configuration-Articles/How-to-Configure-VPN-Tunnel-Between-a-Palo-Alto-Networks/ta-p/59065) |[配置指南](https://live.paloaltonetworks.com/t5/Integration-Articles/Configuring-IKEv2-VPN-for-Microsoft-Azure-Environment/ta-p/60340) |
| WatchGuard |全部 |Fireware XTM<br> PolicyBased：v11.11.x<br>RouteBased：v11.12.x |[配置指南](http://watchguardsupport.force.com/publicKB?type=KBArticle&SFDCID=kA2F00000000LI7KAM&lang=en_US) |[配置指南](http://watchguardsupport.force.com/publicKB?type=KBArticle&SFDCID=kA22A000000XZogSAG&lang=en_US)|

(*) ISR 7200 系列路由器仅支持 PolicyBased VPN。

## <a name="additionaldevices"  id="devices-not-on-the-compatible-list"></a> 未验证的 VPN 设备
即使没有看到设备在“已验证的 VPN 设备”表列出中，该设备也有可能适用于站点到站点连接。 请联系设备制造商了解更多支持和配置说明。

## <a name="editing"></a>编辑设备配置示例
在下载提供的 VPN 设备配置示例后，你需要替换一些值来反映你环境的设置。

###<a name="to-edit-a-sample"></a>编辑示例的步骤：

1. 使用记事本打开示例。
2. 搜索所有 <*text*> 字符串并将其替换为与你的环境相关的值。 请确保包含 < 和 >。 指定名称时，你选择的名称应是唯一的。 如果命令无效，请查看设备制造商文档。

| **示例文本** | **更改为** |
| --- | --- |
| &lt;RP_OnPremisesNetwork&gt; |你为此对象选择的名称。 示例：myOnPremisesNetwork |
| &lt;RP_AzureNetwork&gt; |你为此对象选择的名称。 示例：myAzureNetwork |
| &lt;RP_AccessList&gt; |你为此对象选择的名称。 示例：myAzureAccessList |
| &lt;RP_IPSecTransformSet&gt; |你为此对象选择的名称。 示例：myIPSecTransformSet |
| &lt;RP_IPSecCryptoMap&gt; |你为此对象选择的名称。 示例：myIPSecCryptoMap |
| &lt;SP_AzureNetworkIpRange&gt; |指定范围。 示例：192.168.0.0 |
| &lt;SP_AzureNetworkSubnetMask&gt; |指定子网掩码。 示例：255.255.0.0 |
| &lt;SP_OnPremisesNetworkIpRange&gt; |指定本地范围。 示例：10.2.1.0 |
| &lt;SP_OnPremisesNetworkSubnetMask&gt; |指定本地子网掩码。 示例：255.255.255.0 |
| &lt;SP_AzureGatewayIpAddress&gt; |此信息特定于你的虚拟网络，并位于经典管理门户的“网关 IP 地址”中。 |
| &lt;SP_PresharedKey&gt; |此信息特定于你的虚拟网络，并作为“管理密钥”位于经典管理门户中。 |

## <a name="IPSec"></a>IPsec/IKE 参数
> [AZURE.NOTE]
> 尽管 Azure VPN 网关支持下表中列出的值，但你目前无法从 Azure VPN 网关中选择或指定算法或参数的特定组合。 你必须从本地 VPN 设备指定任何约束。
> 
> 此外，还必须将 **MSS** 固定在 **1350**。

在下表中：

* SA = 安全关联
* IKE 阶段 1 也称“主模式”
* IKE 阶段 2 也称“快速模式”

### <a name="ike-phase-1-main-mode-parameters"></a>IKE 阶段 1（主模式）参数
| **属性**          |**PolicyBased**    | **RouteBased**    |
| ---                   | ---               | ---               |
| SDK 版本           |IKEv1              |IKEv2              |
| Diffie-Hellman 组  |组 2（1024 位） |组 2（1024 位） |
| 身份验证方法 |预共享密钥     |预共享密钥     |
| 加密和哈希算法 |1.AES256、SHA256<br>2.AES256、SHA1<br>3.AES128、SHA1<br>4. 3DES、SHA1 |1.AES256、SHA1<br>2.AES256、SHA256<br>3.AES128、SHA1<br>4.AES128、SHA256<br>5. 3DES、SHA1<br>6. 3DES、SHA256 |
| SA 生存期           |28,800 秒     |28,800 秒     |

### <a name="ike-phase-2-quick-mode-parameters"></a>IKE 阶段 2（快速模式）参数
| **属性**                  |**PolicyBased**| **RouteBased**                              |
| ---                           | ---           | ---                                         |
| SDK 版本                   |IKEv1          |IKEv2                                        |
| 加密和哈希算法 |1.AES256、SHA256<br>2.AES256、SHA1<br>3.AES128、SHA1<br>4. 3DES、SHA1 |[RouteBased QM SA 产品/服务](#RouteBasedOffers) |
| SA 生存期（时间）            |3,600 秒  |27,000 秒                                |
| SA 生存期（字节数）           |102,400,000 KB | -                                           |
| 完全向前保密 (PFS) |否             |[RouteBased QM SA 产品/服务](#RouteBasedOffers) |
| 死对等体检测 (DPD)     |不支持  |支持                                    |

### <a name ="RouteBasedOffers"></a>RouteBased VPN IPsec 安全关联（IKE 快速模式 SA）产品/服务
下表列出了 IPsec SA（IKE 快速模式）产品/服务。 这些产品按提供或接受产品的偏好顺序列出。

#### <a name="azure-gateway-as-initiator"></a>Azure 网关作为发起方
|-  |**加密**|**身份验证**|**PFS 组**|
|---| ---          |---               |---          |
| 1 |GCM AES256    |GCM (AES256)      |无         |
| 2 |AES256        |SHA1              |无         |
| 3 |3DES          |SHA1              |无         |
| 4 |AES256        |SHA256            |无         |
| 5 |AES128        |SHA1              |无         |
| 6 |3DES          |SHA256            |无         |

#### <a name="azure-gateway-as-responder"></a>Azure 网关作为响应方
|-  |**加密**|**身份验证**|**PFS 组**|
|---| ---          | ---              |---          |
| 1 |GCM AES256    |GCM (AES256)      |无         |
| 2 |AES256        |SHA1              |无         |
| 3 |3DES          |SHA1              |无         |
| 4 |AES256        |SHA256            |无         |
| 5 |AES128        |SHA1              |无         |
| 6 |3DES          |SHA256            |无         |
| 7 |DES           |SHA1              |无         |
| 8 |AES256        |SHA1              |1            |
| 9 |AES256        |SHA1              |2            |
| 10|AES256        |SHA1              |14           |
| 11|AES128        |SHA1              |1            |
| 12|AES128        |SHA1              |2            |
| 13|AES128        |SHA1              |14           |
| 14|3DES          |SHA1              |1            |
| 15|3DES          |SHA1              |2            |
| 16|3DES          |SHA256            |2            |
| 17|AES256        |SHA256            |1            |
| 18|AES256        |SHA256            |2            |
| 19|AES256        |SHA256            |14           |
| 20|AES256        |SHA1              |24           |
| 21|AES256        |SHA256            |24           |
| 22|AES128        |SHA256            |无         |
| 23|AES128        |SHA256            |1            |
| 24|AES128        |SHA256            |2            |
| 25|AES128        |SHA256            |14           |
| 26|3DES          |SHA1              |14           |

* 可以使用 RouteBased 和高性能 VPN 网关指定 IPsec ESP NULL 加密。 基于 Null 的加密不对传输中的数据提供保护，仅应在需要最大吞吐量和最小延迟时才使用。  客户端可以在 VNet 到 VNet 通信方案中选择使用此方法，或者在解决方案中的其他位置应用加密时使用此方法。
* 若要通过 Internet 建立跨界连接，请使用默认的 Azure VPN 网关设置以及上表中列出的加密和哈希算法，确保关键通信的安全性。

## <a name="known"></a>已知的设备兼容性问题

> [AZURE.IMPORTANT]
> 这些是第三方 VPN 设备与 Azure VPN 网关之间的已知兼容性问题。 Azure 团队正积极与供应商合作解决此处列出的问题。 解决问题后，将使用最新的信息更新此页。 请定期查看。

###<a name="feb-16-2017"></a>2017 年 2 月 16 日

**早于 7.1.4 版的 Palo Alto Networks 设备**（针对 Azure 基于路由的 VPN）：如果通过 PAN-OS 版本早于 7.1.4 的 Palo Alto Networks 使用 VPN 设备，并且遇到 Azure 基于路由的 VPN 网关的连接问题，请执行以下步骤：

1. 检查 Palo Alto Networks 设备的固件版本。 如果 PAN-OS 版本低于 7.1.4，请升级到 7.1.4。
2. 连接到 Azure VPN 网关时，请在 Palo Alto Networks 设备上将阶段 2 SA（或快速模式 SA）生存期更改为 28,800 秒（8 小时）。
3. 如果仍然遇到连接问题，请从 Azure 门户预览提出支持请求。
<!--Update_Description: wording update-->