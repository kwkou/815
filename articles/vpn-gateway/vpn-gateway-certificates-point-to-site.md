<properties
    pageTitle="创建点到站点连接的自签名证书：Azure | Azure"
    description="本文包含在 Windows 10 上使用 makecert 创建自签名证书的步骤。"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="27b99f7c-50dc-4f88-8a6e-d60080819a43"
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/15/2017"
    wacn.date="03/03/2017"
    ms.author="cherylmc" />

# 针对点到站点连接使用自签名证书
本文介绍如何使用 **makecert** 创建自签名证书，然后从中生成客户端证书。对于 P2S 连接，证书的首选方法是使用企业证书解决方案。请确保使用公用名称值格式“name@yourdomain.com”（而不是“NetBIOS domain name\\username”格式）颁发客户端证书。

如果没有企业解决方案，则需要自签名证书以允许 P2S 客户端连接到虚拟网络。虽然可以使用 PowerShell 创建证书，但使用 PowerShell 生成的证书不包含 Azure 用于 P2S 身份验证所需的字段。已对 makecert 进行了验证，以创建与 P2S 连接相兼容的证书。使用 P2S 配置时并未弃用 makecert。

## 创建自签名证书
以下步骤将演示如何使用 makecert 创建自签名证书。这些步骤并非特定于部署模型。它们同样适用于 Resource Manager 和经典部署模型。

### 创建自签名证书
1. 从运行 Windows 10 的计算机中下载并安装[用于 Windows 10 的 Windows 软件开发包 (SDK)](https://dev.windows.com/downloads/windows-10-sdk)。
2. 安装后，可以在以下路径中找到 makecert.exe 实用程序：“C:\\Program Files (x86)\\Windows Kits\\10\\bin<arch>”。以管理员身份打开命令提示符并导航到 makecert 实用程序所在的位置。可以使用以下示例：

		cd C:\Program Files (x86)\Windows Kits\10\bin\x64

3. 在计算机上的“个人”证书存储中创建并安装证书。以下示例将创建一个相应的 *.cer* 文件，在配置 P2S 时需要将此文件上传到 Azure。将“ARMP2SRootCert”和“ARMP2SRootCert.cer”替换为要用于证书的名称。<br><br>该证书位于“证书”-“当前用户\\个人\\证书”中。
   
        makecert -sky exchange -r -n "CN=ARMP2SRootCert" -pe -a sha1 -len 2048 -ss My "ARMP2SRootCert.cer"

### <a name="rootpublickey"></a>获取公钥
根证书的公钥作为点到站点连接 VPN 网关配置的一部分上传到 Azure。

1. 若要获取证书 .cer 文件，请打开 **certmgr.msc**。找到自签名根证书（通常位于“证书”-“当前用户\\个人\\证书”中），然后右键单击。单击“所有任务”，然后单击“导出”。此操作将打开“证书导出向导”。
2. 在向导中，单击“下一步”。选择“否，不导出私钥”，然后单击“下一步”。
3. 在“导出文件格式”页上，选择“Base-64 编码的 X.509 (.CER)”，然后单击“下一步”。
4. 在“要导出的文件”中，单击“浏览”并选择要导出证书的位置。在“文件名”中，为证书文件命名。然后单击“下一步”。
5. 单击“完成”以导出证书。会看到“导出已成功”。单击“确定”关闭该向导。

### 导出自签名证书（可选）
可能需要导出自签名证书，并将它安全存储。如果需要，可以稍后在另一台计算机上安装此自签名证书，然后生成更多客户端证书，或导出另一个 .cer 文件。已安装客户端证书并配置适当的 VPN 客户端设置的任何计算机都可以通过 P2S 连接到虚拟网络。因此，需要确保仅在需要时生成和安装客户端证书，并且需要安全地存储此自签名证书。

若要将自签名证书导出为 .pfx，请选择根证书，然后使用[导出客户端证书](#clientkey)中所述的步骤导出。

## 创建和安装客户端证书
不要直接在客户端计算机上安装自签名的证书。需要从自签名证书生成客户端证书。然后将客户端证书导出并安装到客户端计算机上。以下步骤并非特定于部署模型。它们同样适用于 Resource Manager 和经典部署模型。

### 第 1 部分 - 从自签名证书生成客户端证书
以下步骤将演示从自签名证书生成客户端证书的其中一种方法。可以从相同证书生成多个客户端证书。然后每个客户端证书可以导出并安装在客户端计算机上。

1. 在用于创建自签名证书的同一台计算机上，以管理员身份打开命令提示符。
2. 修改并运行示例，生成客户端证书。
	* 将 *"ARMP2SRootCert"* 更改为生成客户端证书所用的自签名根证书。确保使用创建自签名根时指定的根证书名称（无论“CN=”的值为何）。
	* 将 *ClientCertificateName* 更改为要生成的客户端证书的名称。<br><br>如果未经修改就运行以下示例，个人证书存储中将有一个从根证书 ARMP2SRootCert 生成的客户端证书，名为 ClientCertificateName。

			makecert.exe -n "CN=ClientCertificateName" -pe -sky exchange -m 96 -ss My -in "ARMP2SRootCert" -is my -a sha1

### <a name="clientkey"></a>第 2 部分 - 导出客户端证书                                                                                                                        

1. 若要导出客户端证书，请打开 **certmgr.msc**。右键单击要导出的客户端证书，单击“所有任务”，然后单击“导出”。此操作将打开“证书导出向导”。
2. 在向导中，单击“下一步”，选择“是，导出私钥”，然后单击“下一步”。
3. 在“导出文件格式”页上，可以保留选择默认值。然后，单击“下一步”。
4. 在“安全性”页上，必须保护私钥。如果选择使用密码，请务必记下或牢记为此证书设置的密码。然后，单击“下一步”。
5. 在“要导出的文件”中，单击“浏览”并选择要导出证书的位置。在“文件名”中，为证书文件命名。然后单击“下一步”。
6. 单击“完成”以导出证书。

### 第 3 部分 - 安装客户端证书
想要使用点到站点连接与虚拟网络连接的每个客户端都必须安装客户端证书。除了必需的 VPN 配置包以外，还必须安装此证书。以下步骤将演示如何手动安装客户端证书。

1. 找到 *.pfx* 文件并将其复制到客户端计算机。在客户端计算机上，双击 *.pfx* 文件以进行安装。将“存储位置”保留为“当前用户”，然后单击“下一步”。
2. 在“要导入的文件”页上，不要进行任何更改。单击**“下一步”**。
3. 在“私钥保护”页上，如果使用了密码，请输入证书的密码，或验证安装证书的安全主体是否正确，然后单击“下一步”。
4. 在“证书存储”页上，保留默认位置，然后单击“下一步”。
5. 单击“完成”。在证书安装的“安全警告”上，单击“是”。现在可单击“是”，因为已生成证书。现已成功导入证书。

## 后续步骤
继续使用点到站点配置。

* 有关 **Resource Manager** 部署模型步骤，请参阅 [Configure a Point-to-Site connection to a VNet using PowerShell](/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/)（使用 PowerShell 配置与 VNet 的点到站点连接）。
* 有关**经典**部署模型步骤，请参阅 [Configure a Point-to-Site VPN connection to a VNet using the Classic Management Portal](/documentation/articles/vpn-gateway-point-to-site-create/)（使用经典管理门户配置与 VNet 的点到站点 VPN 连接）。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: wording update-->