<properties
    pageTitle="创建和导出点到站点证书：makecert : Azure | Azure"
    description="本文包含使用 makecert 创建自签名根证书、导出公钥和生成客户端证书的步骤。"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="05/03/2017"
    wacn.date="05/31/2017"
    ms.author="cherylmc"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="b2aca5a9c1182fa384b8a54ad47602962d8a8f0a"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="generate-and-export-certificates-for-point-to-site-connections-using-makecert"></a>使用 makecert 生成和导出点到站点连接证书

> [AZURE.NOTE]
> 仅当无权访问 Windows 10 计算机时使用这些说明，以生成点到站点连接的自签名证书。 Makecert 将被弃用。 此外，makecert 无法创建 SHA-2 证书，只能创建 SHA-1 证书（这对 P2S 仍然有效）。 出于这些原因，如果可能，建议使用 [PowerShell 步骤](/documentation/articles/vpn-gateway-certificates-point-to-site/)。 使用 PowerShell 或 makecert 创建的证书可以安装在任何[受支持的客户端操作系统](/documentation/articles/vpn-gateway-howto-point-to-site-resource-manager-portal/#faq)上，而不仅限于用来创建它们的操作系统。
>
>

本文将演示如何创建自签名根证书、导出公钥，以及生成客户端证书。 本文不包含点到站点配置说明或点到站点常见问题解答。 可通过从以下列表中选择一篇“配置点到站点”文章来查找该信息：
> [AZURE.SELECTOR]
- [创建自签名证书 - PowerShell](/documentation/articles/vpn-gateway-certificates-point-to-site/)
- [创建自签名证书 - Makecert](/documentation/articles/vpn-gateway-certificates-point-to-site-makecert/)
- [配置点到站点 - Resource Manager - Azure 门户](/documentation/articles/vpn-gateway-howto-point-to-site-resource-manager-portal/)
- [配置点到站点 - Resource Manager - PowerShell](/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/)
- [配置点到站点 - 经典 - Azure 门户](/documentation/articles/vpn-gateway-howto-point-to-site-classic-azure-portal/)

点到站点连接使用证书进行身份验证。 配置点到站点连接时，需要将根证书的公钥（.cer 文件）上传到 Azure。 此外，必须从根证书生成客户端证书，并将其安装在连接到 VNet 的每台客户端计算机上。 客户端证书允许客户端进行身份验证。

## <a name="rootcert"></a>创建自签名根证书

以下步骤将演示如何使用 makecert 创建自签名证书。 这些步骤并非特定于部署模型。 它们同样适用于 Resource Manager 和经典部署模型。

1. 下载并安装 [makecert](https://msdn.microsoft.com/zh-cn/library/windows/desktop/aa386968(v=vs.85).aspx)。
2. 安装后，可以在此路径中找到 makecert.exe 实用工具：“C:\Program Files (x86)\Windows Kits\10\bin\<arch>”。 以管理员身份打开命令提示符并导航到 makecert 实用程序所在的位置。 可以使用以下示例：

      cd C:\Program Files (x86)\Windows Kits\10\bin\x64

3. 在计算机上的“个人”证书存储中创建并安装证书。 以下示例将创建一个相应的 .cer 文件，在配置 P2S 时需要将此文件上传到 Azure。 使用想要用于证书的名称替换“P2SRootCert”和“P2SRootCert.cer”。 该证书将位于“证书”-“当前用户\个人\证书”中。

      makecert -sky exchange -r -n "CN=P2SRootCert" -pe -a sha1 -len 2048 -ss My "P2SRootCert.cer"

## <a name="cer"></a>导出公钥 (.cer)

[AZURE.INCLUDE [Export public key](../../includes/vpn-gateway-certificates-export-public-key-include.md)]

exported.cer 文件必须上传到 Azure。 请参阅[配置点到站点连接](/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/#upload)获取相关说明。

### <a name="export-the-self-signed-certificate-and-private-key-to-store-it-optional"></a>导出自签名证书和用于存储它的私钥（可选）

你可能想要导出自签名根证书并将它存储在安全位置。 如果需要，可以稍后在另一台计算机上安装此自签名证书，然后生成更多客户端证书，或导出另一个 .cer 文件。 若要将自签名根证书导出为 .pfx，请选择该根证书，然后使用[导出客户端证书](#clientexport)中所述的步骤导出。

## <a name="create-and-install-client-certificates"></a>创建和安装客户端证书

不要直接在客户端计算机上安装自签名的证书。 需要从自签名证书生成客户端证书。 然后将客户端证书导出并安装到客户端计算机上。 以下步骤并非特定于部署模型。 它们同样适用于 Resource Manager 和经典部署模型。

### <a name="clientcert"></a>生成客户端证书

在使用点到站点连接连接到 VNet 的每台客户端计算机上，必须安装客户端证书。 可以从自签名根证书生成客户端证书，然后导出并安装该客户端证书。 如果不安装客户端证书，身份验证将会失败。 

以下步骤引导你完成从自签名根证书生成客户端证书的过程。 可以从相同根证书生成多个客户端证书。 使用以下步骤生成客户端证书时，客户端证书将自动安装在用于生成该证书的计算机上。 如果想要在另一台客户端计算机上安装客户端证书，可以导出该证书。

1. 在用于创建自签名证书的同一台计算机上，以管理员身份打开命令提示符。
2. 修改并运行示例，生成客户端证书。
  * 将“P2SRootCert”更改为生成客户端证书所用的自签名根证书。 确保使用创建自签名根时指定的根证书名称（无论“CN=”的值为何）。
  * 将 P2SChildCert 更改为生成客户端证书所用的名称。

  如果未经修改就运行以下示例，个人证书存储中将有一个从根证书 P2SRootCert 生成的客户端证书，名为 P2SChildcert。

      makecert.exe -n "CN=P2SChildCert" -pe -sky exchange -m 96 -ss My -in "P2SRootCert" -is my -a sha1

### <a name="clientexport"></a>导出客户端证书

[AZURE.INCLUDE [Export client certificate](../../includes/vpn-gateway-certificates-export-client-cert-include.md)]

### <a name="install"></a>安装已导出的客户端证书

[AZURE.INCLUDE [Install client certificate](../../includes/vpn-gateway-certificates-install-client-cert-include.md)]

## <a name="next-steps"></a>后续步骤

继续使用点到站点配置。 

* 有关 **Resource Manager** 部署模型步骤，请参阅 [Configure a Point-to-Site connection to a VNet](/documentation/articles/vpn-gateway-howto-point-to-site-resource-manager-portal/)（配置与 VNet 的点到站点连接）。
* 有关**经典**部署模型步骤，请参阅 [Configure a Point-to-Site VPN connection to a VNet (classic)](/documentation/articles/vpn-gateway-howto-point-to-site-classic-azure-portal/)（配置与 VNet 的点到站点 VPN 连接（经典））。