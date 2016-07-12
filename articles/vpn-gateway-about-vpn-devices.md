<properties 
   pageTitle="关于 Azure 虚拟网络的站点到站点 VPN 网关连接的 VPN 设备 | Azure"
   description="了解用于 S2S VPN 网关连接的 VPN 设备和 IPsec 参数。站点到站点连接可以用于混合配置。本文包含指向配置说明以及 VPN 网关设备示例的链接。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
  tags="azure-resource-manager, azure-service-management"/>
<tags 
   ms.service="vpn-gateway"
   ms.date="04/29/2016"
   wacn.date="06/30/2016" />

# 关于站点到站点 VPN 网关连接的 VPN 设备

配置站点到站点 (S2S) VPN 连接需要用到 VPN 设备。在创建混合解决方案时，或者每当你想要在本地网络与虚拟网络之间建立安全连接时，可以使用站点到站点连接。本文介绍兼容的 VPN 设备和配置参数。请注意，配置站点到站点连接时，需要为你的 VPN 设备提供面向公众的 IPv4 IP 地址。

如果你的设备没有出现在“已验证 VPN 设备”表中，请参阅本文的“非验证 VPN 设备”部分。你的设备仍可能兼容 Azure。如需 VPN 设备支持，请联系你的设备制造商。

**查看表时的注意事项：**

- 静态和动态路由的术语已更改。这两个术语你可能都会碰到。无功能性更改，只是名称进行了更改。
	- 静态路由 = 基于策略
	- 动态路由 = 基于路由 
- 除非另有说明，否则高性能 VPN 网关和基于路由的 VPN 网关的规范是相同的。例如，与基于路由的 VPN 网关兼容的已验证 VPN 设备也与新的 Azure 高性能 VPN 网关兼容。 


## 已验证的 VPN 设备 

我们在与设备供应商合作的过程中验证了一系列的标准 VPN 设备。下面的列表中包含的设备系列中的所有设备都应适用于 Azure VPN 网关。请参阅 [VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)这一文章，以确定你需要为所要配置的解决方案创建哪种网关类型。

若要获取配置 VPN 设备的帮助，请参考对应于各设备系列的链接。



| **供应商** | **设备系列** | **最低操作系统版本** | **基于策略** | **基于路由** |
|---------------------------------|----------------------------------------------------------|----------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Allied Telesis | AR 系列 VPN 路由器 | 2\.9.2 | 即将支持 | 不兼容 |
| Barracuda Networks, Inc. | Barracuda NextGen Firewall F-series | 基于策略：5.4.3，基于路由：6.2.0 | [配置说明](https://techlib.barracuda.com/NGF/AzurePolicyBasedVPNGW) | [配置说明](https://techlib.barracuda.com/NGF/AzureRouteBasedVPNGW) |
| Barracuda Networks, Inc. | Barracuda NextGen Firewall X-series | Barracuda Firewall 6.5 | [Barracuda Firewall](https://techlib.barracuda.com/BFW/ConfigAzureVPNGateway) | 不兼容 |
| Brocade | Vyatta 5400 vRouter | Virtual Router 6.6R3 GA | [配置说明](http://www1.brocade.com/downloads/documents/html_product_manuals/vyatta/vyatta_5400_manual/wwhelp/wwhimpl/js/html/wwhelp.htm#href=VPN_Site-to-Site%20IPsec%20VPN/Preface.1.1.html) | 不兼容 |
| 检查点 | 安全网关 | R75.40、R75.40VS | [配置说明](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk101275) | [配置说明](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk101275) |
| Cisco | ASA | 8\.3 | [Cisco 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASA) | 不兼容 |
| Cisco | ASR | IOS 15.1（基于策略）、IOS 15.2（基于路由） | [Cisco 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASR) | [Cisco 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASR) |
| Cisco | ISR | IOS 15.0（基于策略）、IOS 15.1（基于路由） | [Cisco 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ISR) | [Cisco 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ISR) |
| Citrix | CloudBridge MPX 设备或 VPX 虚拟设备 | 不适用 | [集成说明](https://www.citrix.com/welcome.html?resource=%2Fdownloads%2Fcloudbridge%2Fbetas-and-tech-previews%2Fcloudbridge-azure-integration) | 不兼容 |
| Dell SonicWALL | TZ 系列、NSA 系列、SuperMassive 系列、E 类 NSA 系列 | SonicOS 5.8.x、[SonicOS 5.9.x](http://documents.software.dell.com/sonicos/5.9/microsoft-azure-configuration-guide/supported-platforms?ParentProduct=850)、[SonicOS 6.x](http://documents.software.dell.com/sonicos/6.2/microsoft-azure-configuration-guide/supported-platforms?ParentProduct=646) | [说明 - SonicOS 6.2](http://documents.software.dell.com/sonicos/6.2/microsoft-azure-configuration-guide?ParentProduct=646) [说明 - SonicOS 5.9](http://documents.software.dell.com/sonicos/5.9/microsoft-azure-configuration-guide?ParentProduct=850) | [说明 - SonicOS 6.2](http://documents.software.dell.com/sonicos/6.2/microsoft-azure-configuration-guide?ParentProduct=646) [说明 - SonicOS 5.9](http://documents.software.dell.com/sonicos/5.9/microsoft-azure-configuration-guide?ParentProduct=850) |
| F5 | BIG-IP 系列 | 不适用 | [配置说明](https://devcentral.f5.com/articles/connecting-to-windows-azure-with-the-big-ip) | 不兼容 |
| Fortinet | FortiGate | FortiOS 5.0.7 | [配置说明](http://docs.fortinet.com/fortigate/admin-guides) | [配置说明](http://docs.fortinet.com/fortigate/admin-guides) |
| Internet Initiative Japan (IIJ) | SEIL 系列 | SEIL/X 4.60、SEIL/B1 4.60、SEIL/x86 3.20 | [配置说明](http://www.iij.ad.jp/biz/seil/ConfigAzureSEILVPN.pdf) | 不兼容 |
| Juniper | SRX | JunOS 10.2（基于策略）、JunOS 11.4（基于路由） | [Juniper 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SRX) | [Juniper 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SRX) |
| Juniper | J 系列 | JunOS 10.4r9（基于策略）、JunOS 11.4（基于路由） | [Juniper 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/JSeries) | [Juniper 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/JSeries) |
| Juniper | ISG | ScreenOS 6.3（基于策略和基于路由） | [Juniper 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/ISG) | [Juniper 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/ISG) |
| Juniper | SSG | ScreenOS 6.2（基于策略和基于路由） | [Juniper 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SSG) | [Juniper 示例](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SSG) |
| Microsoft | 路由和远程访问服务 | Windows Server 2012 | 不兼容 | [Microsoft 示例](http://go.microsoft.com/fwlink/p/?LinkId=717761) |
| 打开系统 AG | 任务控制安全网关 | 不适用 | [安装指南](https://www.open.ch/_pdf/Azure/AzureVPNSetup_Installation_Guide.pdf) | [安装指南](https://www.open.ch/_pdf/Azure/AzureVPNSetup_Installation_Guide.pdf) |
| Openswan | Openswan | 2\.6.32 | （即将支持） | 不兼容 |
| Palo Alto Networks | 运行 PAN-OS 的所有设备 | PAN-OS 6.1.5 或更高版本（基于策略）、PAN-OS 7.0.5 或更高版本（基于路由） | [配置说明](https://live.paloaltonetworks.com/t5/Configuration-Articles/How-to-Configure-VPN-Tunnel-Between-a-Palo-Alto-Networks/ta-p/59065) | [配置说明](https://live.paloaltonetworks.com/t5/Integration-Articles/Configuring-IKEv2-VPN-for-Microsoft-Azure-Environment/ta-p/60340) |
| Watchguard | 全部 | Fireware XTM v11.x | [配置说明](http://customers.watchguard.com/articles/Article/Configure-a-VPN-connection-to-a-Windows-Azure-virtual-network) | 不兼容 |


##<a name="devices-not-on-the-compatible-list"></a> 非验证的 VPN 设备

如果你没有看到你的设备列在“已验证的 VPN 设备”表（见上）中，该设备仍有可能适用于站点到站点连接。请确保你的 VPN 设备符合[关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/#gateway-requirements)一文“网关要求”部分列出的最低要求。满足最低要求的设备也应该兼容 VPN 网关。请联系你的设备制造商了解更多支持和配置说明。


## 编辑设备配置示例

在下载提供的 VPN 设备配置示例后，你需要替换一些值来反映你环境的设置。

**编辑示例的步骤：**

1. 使用记事本打开示例。 
1. 搜索所有 < text > 字符串并将其替换为与你的环境相关的值。请确保包含 < and >。指定名称时，你选择的名称应是唯一的。如果命令无效，请查看你的设备制造商文档。

| **示例文本** | **更改为** |
|----------------------------------|----------------------------------------------------------------------------------------------------------------------|
| &lt;RP\_OnPremisesNetwork&gt; | 你为此对象选择的名称。示例：myOnPremisesNetwork |
| &lt;RP\_AzureNetwork&gt; | 你为此对象选择的名称。示例：myAzureNetwork |
| &lt;RP\_AccessList&gt; | 你为此对象选择的名称。示例：myAzureAccessList |
| &lt;RP\_IPSecTransformSet&gt; | 你为此对象选择的名称。示例：myIPSecTransformSet |
| &lt;RP\_IPSecCryptoMap&gt; | 你为此对象选择的名称。示例：myIPSecCryptoMap |
| &lt;SP\_AzureNetworkIpRange&gt; | 指定范围。示例：192.168.0.0 |
| &lt;SP\_AzureNetworkSubnetMask&gt; | 指定子网掩码。示例：255.255.0.0 |
| &lt;SP\_OnPremisesNetworkIpRange&gt; | 指定本地范围。示例：10.2.1.0 |
| &lt;SP\_OnPremisesNetworkSubnetMask&gt; | 指定本地子网掩码。示例：255.255.255.0 |
| &lt;SP\_AzureGatewayIpAddress&gt; | 此信息特定于你的虚拟网络，位于经典管理门户的“网关 IP 地址”中。 |
| &lt;SP\_PresharedKey&gt; | 此信息特定于你的虚拟网络，位于经典管理门户的“管理密钥”中。 |



## IPsec 参数

>[AZURE.NOTE] 尽管 Azure VPN 网关支持下面列出的值，但你目前无法从 Azure VPN 网关中选择或指定特定的组合。你必须从本地 VPN 设备指定任何约束。

### IKE 阶段 1 设置

| **属性** | **基于策略** | **基于路由的标准或高性能 VPN 网关** |
|----------------------------------------------------|--------------------------------|------------------------------------------------------------------|
| SDK 版本 | IKEv1 | IKEv2 |
| Diffie-Hellman 组 | 组 2（1024 位） | 组 2（1024 位） |
| 身份验证方法 | 预共享密钥 | 预共享密钥 |
| 加密算法 | AES256 AES128 3DES | AES256 3DES |
| 哈希算法 | SHA1(SHA128) | SHA1(SHA128)、SHA2(SHA256) |
| 阶段 1 安全关联 (SA) 生命周期（时间） | 28,800 秒 | 10,800 秒 |


### IKE 阶段 2 设置

| **属性** | **基于策略** | **基于路由的标准或高性能 VPN 网关** |
|--------------------------------------------------------------------------|------------------------------------------------|--------------------------------------------------------------------|
| SDK 版本 | IKEv1 | IKEv2 |
| 哈希算法 | SHA1(SHA128) | SHA1(SHA128) |
| 阶段 2 安全关联 (SA) 生命周期（时间） | 3,600 秒 | 3,600 秒 |
| 阶段 2 安全关联 (SA) 生命周期（吞吐量）| 102,400,000 KB | - |
| IPsec SA 加密和身份验证产品（按偏好顺序列出）| 1.ESP-AES256 2.ESP-AES128 3.ESP-3DES 4.N/A | 请参阅基于路由的网关 IPsec 安全关联 (SA) 产品（见下）|
| 完全向前保密 (PFS) | 否 | 是 (DH Group1, 2, 5, 14, 24) |
| 对等体存活检测 | 不支持 | 支持 |

### 基于路由的网关 IPsec 安全关联 (SA) 产品

下表列出 IPsec SA 加密和身份验证产品。这些产品按提供或接受产品的偏好顺序列出。

| **IPsec SA 加密和身份验证产品** | **Azure 网关作为发起方** | **Azure 网关作为响应方** |
|---------------------------------------------------|--------------------------------------------------------------|--------------------------------------------------------------|
| 1 | ESP AES\_256 SHA | ESP AES\_128 SHA |
| 2 | ESP AES\_128 SHA | ESP 3\_DES MD5 |
| 3 | ESP 3\_DES MD5 | ESP 3\_DES SHA |
| 4 | ESP 3\_DES SHA | AH SHA1（具有含 null HMAC 的 ESP AES\_128） |
| 5 | AH SHA1（具有含 null HMAC 的 ESP AES\_256） | AH SHA1（具有含 null HMAC 的 ESP 3\_DES） |
| 6 | AH SHA1（具有含 null HMAC 的 ESP AES\_128） | AH MD5（具有含 null HMAC 的 ESP 3\_DES），不建议生命周期 |
| 7 | AH SHA1（具有含 null HMAC 的 ESP 3\_DES） | AH SHA1（具有 ESP 3\_DES SHA1），无生命周期 |
| 8 | AH MD5（具有含 null HMAC 的 ESP 3\_DES），不建议生命周期 | AH MD5（具有 ESP 3\_DES MD5），无生命周期 |
| 9 | AH SHA1（具有 ESP 3\_DES SHA1），无生命周期 | ESP DES MD5 |
| 10 | AH MD5（具有 ESP 3\_DES MD5），无生命周期 | ESP DES SHA1，无生命周期 |
| 11 | ESP DES MD5 | AH SHA1（具有 ESP DES null HMAC），不建议生命周期 |
| 12 | ESP DES SHA1，无生命周期 | AH MD5（具有 ESP DES null HMAC），不建议生命周期 |
| 13 | AH SHA1（具有 ESP DES null HMAC），不建议生命周期 | AH SHA1（具有 ESP DES SHA1），无生命周期 |
| 14 | AH MD5（具有 ESP DES null HMAC），不建议生命周期 | AH MD5（具有 ESP DES MD5），无生命周期 |
| 15 | AH SHA1（具有 ESP DES SHA1），无生命周期 | ESP SHA，无生命周期 |
| 16 | AH MD5（具有 ESP DES MD5），无生命周期 | ESP MD5，无生命周期 |
| 17 | - | AH SHA，无生命周期 |
| 18 | - | AH MD5，无生命周期 |


- 你可以对基于路由的高性能 VPN 网关指定 IPsec ESP NULL 加密。基于 Null 的加密不对传输中的数据提供保护，仅应在需要最大吞吐量和最小延迟时才使用。客户端可以在 VNet 到 VNet 通信方案中选择使用此方法，或者要在解决方案中的其他位置应用加密时使用此方法。

- adk 要通过 Internet 建立跨界连接，请使用默认的 Azure VPN 网关设置以及上表中列出的加密和哈希算法，以确保关键通信的安全性。






<!---HONumber=Mooncake_0613_2016-->
