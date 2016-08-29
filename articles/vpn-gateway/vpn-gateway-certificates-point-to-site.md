<properties 
   pageTitle="使用 makecert 为点到站点虚拟网络跨界连接创建自签名证书 | Azure"
   description="本文包含在 Windows 10 上使用 makecert 创建自签名证书的步骤。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>
<tags
	ms.service="vpn-gateway"
	ms.date="08/15/2016"
	wacn.date="08/29/2016"/>

# 为点到站点连接使用自签名证书

本文可帮助您使用 makecert 创建自签名证书，然后从中生成客户端证书。以下步骤专为在 Windows 10 上使用 makecert 所编写。已对 makecert 进行了验证，以创建与 P2S 连接相兼容的证书。

对于 P2S 连接，证书的首选方法是使用企业证书解决方案，确保颁发常见名称值格式为“name@yourdomain.com”的客户端证书，而不是“NetBIOS domain name\\username”格式。

如果没有企业解决方案，则需要自签名证书以允许 P2S 客户端连接到虚拟网络。我们知道虽然 makecert 已被弃用，但它仍然是创建与 P2S 连接相兼容的自签名证书的有效方法。

## 创建自签名证书

Makecert 是创建自签名证书的方式之一。以下步骤将演示如何使用 makecert 创建自签名证书。这些步骤并非特定于部署模型。它们同样适用于 Resource Manager 和经典部署模型。

### 创建自签名证书

1. 从运行 Windows 10 的计算机中下载并安装[用于 Windows 10 的 Windows 软件开发包 (SDK)](https://dev.windows.com/downloads/windows-10-sdk)。

2. 安装之后，可以在以下路径中找到 makecert.exe 实用工具：C:\\Program Files (x86)\\Windows Kits\\10\\bin<arch>。
		
	示例：
	
		C:\Program Files (x86)\Windows Kits\10\bin\x64\makecert.exe

3. 接下来，在计算机上的“个人”证书存储中创建并安装证书。以下示例将创建一个相应的 *.cer* 文件，在配置 P2S 时需要将此文件上传到 Azure。以管理员身份运行以下命令，其中 *CertificateName* 是该证书要使用的名称。<br><br>如果不进行任何更改就运行以下示例，将会生成证书和相应的 *CertificateName.cer* 文件。可以在运行命令所在的目录中找到 .cer 文件。该证书位于“证书”-“当前用户\\个人\\证书”中。

    	makecert -sky exchange -r -n "CN=CertificateName" -pe -a sha1 -len 2048 -ss My "CertificateName.cer"

4. 自签名证书用于创建客户端证书。在将自签名证书的 .cer 文件作为 P2S 配置的一部分上传时，则是在告诉 Azure 信任客户端计算机正在使用的证书。<br><br>任何安装了客户端证书，并配置有正确的 VPN 客户端设置的计算机都可以通过 P2S 连接到您的虚拟网络。因此，需要确保仅在需要时生成和安装客户端证书，并且需要安全地备份和存储此自签名证书。
 

## 创建和安装客户端证书

自签名证书不是安装在客户端上的内容。您需要从自签名证书生成客户端证书。然后导出客户端证书并安装到客户端计算机上。以下步骤并非特定于部署模型。它们同样适用于 Resource Manager 和经典部署模型。

### 第 1 部分 - 从自签名证书生成客户端证书

以下步骤将演示从自签名证书生成客户端证书的其中一种方法。可以从相同证书生成多个客户端证书。然后每个客户端证书可以导出及安装在客户端计算机上。

1. 在用于创建自签名证书的同一台计算机上，以管理员身份打开命令提示符。

2. 将目录更改为需要在其中保存客户端证书文件的位置。*CertificateName* 是指生成的自签名证书。如果运行以下示例（将 CertificateName 更改为证书的名称），则会在“个人”证书存储中生成名为“ClientCertificateName”的客户端证书。

3. 输入以下命令：

    	makecert.exe -n "CN=ClientCertificateName" -pe -sky exchange -m 96 -ss My -in "CertificateName" -is my -a sha1

4. 所有证书都存储在你计算机上的“证书”-“当前用户\\个人\\证书”存储中。你可以按照此过程生成所需数目的客户端证书。

### 第 2 部分 - 导出客户端证书

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

5. 单击“完成”。在证书安装的“安全警告”上，单击“是”。现已成功导入证书。

## 后续步骤

继续使用点到站点配置。

- 有关 **Resource Manager** 部署模型步骤，请参阅[使用 PowerShell 配置与虚拟网络的点到站点连接](/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/)。
- 有关**经典**部署模型步骤，请参阅[配置连接到 VNet 的点到站点 VPN 连接](/documentation/articles/vpn-gateway-point-to-site-create/)。

<!---HONumber=Mooncake_0822_2016-->