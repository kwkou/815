<properties 
   pageTitle="使用 makecert 为点到站点 VPN 网关跨界配置创建自签名证书 | Azure"
   description="本文包含在 Windows 10 上使用 makecert 创建自签名根证书的步骤。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>
<tags 
   ms.service="vpn-gateway"
   ms.date="04/26/2016"
   wacn.date="06/24/2016" />

# 为点到站点配置使用自签名根证书

以下任务将帮助你使用 makecert 创建根证书，并从根证书生成客户端证书。以下步骤专为在 Windows 10 上使用 makecert 所编写。

## 创建自签名根证书

Makecert 是创建自签名根证书的方式之一。以下步骤将引导你使用 makecert 创建自签名根证书。这些步骤并非特定于部署模型。它们同样适用于 Resource Manager 和经典部署模型。

### 创建自签名根证书

1. 从运行 Windows 10 的计算机下载并安装适用于 [Windows 10 的 Windows 软件开发包 (SDK)](https://dev.windows.com/zh-cn/downloads/windows-10-sdk)。

2. 安装之后，可以在以下路径中找到 makecert.exe 实用工具：C:\\Program Files (x86)\\Windows Kits\\10\\bin\<arch>。
		
	示例：
	
		C:\Program Files (x86)\Windows Kits\10\bin\x64\makecert.exe

3. 接下来，在计算机上的“个人”证书存储中创建并安装根证书。以下示例创建稍后要上载的相应 .cer 文件。以管理员身份运行以下命令，其中 RootCertificateName 是要用于证书的名称。

	注意：如果不进行任何更改即运行以下示例，则会生成根证书和相应的文件 RootCertificateName.cer。可以在运行命令所在的目录中找到 .cer 文件。该证书位于“证书”-“当前用户\\个人\\证书”中。

    	makecert -sky exchange -r -n "CN=RootCertificateName" -pe -a sha1 -len 2048 -ss My "RootCertificateName.cer"

	>[AZURE.NOTE] 因为你创建了将从其生成客户端证书的根证书，可能需要导出此根证书以及私钥，并将它保存到一个可以恢复的安全位置。

## 生成、导出和安装客户端证书

可以从自签名根证书生成客户端证书，然后导出及安装客户端证书。以下并非部署模型特定的步骤。它们同样适用于 Resource Manager 和经典部署模型。

### 从自签名根证书生成客户端证书

以下步骤将引导进行从自签名根证书生成客户端证书的其中一种方法。可以从相同根证书生成多个客户端证书。然后每个客户端证书可以导出及安装在客户端计算机上。

1. 在用于创建自签名根证书的同一台计算机上，以管理员身份打开命令提示符。

2. 将目录更改为需要在其中保存客户端证书文件的位置。RootCertificateName 是指你生成的自签名根证书。如果运行以下示例（将 RootCertificateName 更改为根证书的名称），则会在个人证书存储中生成名为“ClientCertificateName”的客户端证书。

3. 输入以下命令：

    	makecert.exe -n "CN=ClientCertificateName" -pe -sky exchange -m 96 -ss My -in "RootCertificateName" -is my -a sha1

4. 所有证书都存储在你计算机上的“证书”-“当前用户\\个人\\证书”存储中。你可以按照此过程生成所需数目的客户端证书。

### 导出客户端证书

1. 若要导出客户端证书，请使用 certmgr.msc。右键单击要导出的客户端证书，单击“所有任务”，然后单击“导出”。此时将打开“证书导出向导”。

2. 在向导中，单击“下一步”，选择“是，导出私钥”，然后单击“下一步”。

3. 在“导出文件格式”页上，可以保留选择默认值。然后，单击“下一步”。
 
4. 在“安全性”页上，必须保护私钥。如果选择使用密码，请务必记下或牢记为此证书设置的密码。然后单击“下一步”。

5. 在“要导出的文件”中，单击“浏览”并选择要导出证书的位置。在“文件名”中，为证书文件命名。然后单击“下一步”。

6. 单击“完成”以导出证书。

### 安装客户端证书

想要使用点到站点连接与虚拟网络连接的每个客户端都必须安装客户端证书。除了必需的 VPN 配置包以外，还必须安装此证书。以下步骤将引导你手动安装客户端证书。

1. 找到 .pfx 文件并将其复制到客户端计算机。在客户端计算机上，双击 .pfx 文件以安装。将“存储位置”保留为“当前用户”，然后单击“下一步”。

2. 在“要导入的文件”页上，不要进行任何更改。单击“下一步”。

3. 在“私钥保护”页上，如果你使用了密码，请输入证书的密码，或验证安装证书的安全主体是否正确，然后单击“下一步”。

4. 在“证书存储”页上，保留默认位置，然后单击“下一步”。

5. 单击“完成”。在证书安装的“安全警告”上，单击“是”。现已成功导入证书。

## 后续步骤

继续使用点到站点配置。

- 有关 **Resource Manager** 部署模型步骤，请参阅 [Configure a Point-to-Site connection to a virtual network using PowerShell（使用 PowerShell 配置与虚拟网络的点到站点连接）](/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/)。 
- 有关**经典**部署模型步骤，请参阅 [Configure a Point-to-Site VPN connection to a VNet（配置连接到 VNet 的点到站点 VPN 连接）](/documentation/articles/vpn-gateway-point-to-site-create/)。

<!---HONumber=Mooncake_0425_2016-->
