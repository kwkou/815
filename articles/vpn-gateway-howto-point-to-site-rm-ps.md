<properties 
   pageTitle="配置与 Azure 虚拟网络的点到站点 VPN 网关连接 | Azure"
   description="通过创建点到站点 VPN 连接安全地连接到 Azure 虚拟网络。如果你需要从远程位置建立跨界连接且不使用 VPN 设备，则此配置非常有用，此外，它还可用于混合网络配置。本文包含适用于通过 Resource Manager 部署模型创建的 VNet 的 PowerShell 说明。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>
<tags 
   ms.service="vpn-gateway"
   ms.date="03/30/2016"
   wacn.date="05/10/2016" />

# 使用 PowerShell 配置与虚拟网络的点到站点连接

> [AZURE.SELECTOR]
- [PowerShell - 资源管理器](/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/)
- [经典管理门户](/documentation/articles/vpn-gateway-point-to-site-create/)

点到站点设置可让你从客户端计算机单独创建与虚拟网络的安全连接。可通过从客户端计算机启动连接来建立 VPN 连接。如果你要从远程位置（例如从家里或会议室）连接到 VNet，或者只有少数几个需要连接到虚拟网络的客户端，则点到站点连接是绝佳的解决方案。

点到站点连接不需要 VPN 设备或面向公众的 IP 地址即可运行。有关点到站点连接的详细信息，请参阅 [VPN Gateway FAQ（VPN 网关常见问题）](/documentation/articles/vpn-gateway-vpn-faq/#point-to-site-connections)和 [About cross-premises connections（关于跨界连接）](/documentation/articles/vpn-gateway-cross-premises-options/)。

本文适用于点到站点 VPN 网关连接，此类连接可连接到使用 **Resource Manager 部署模型**（服务管理）创建的虚拟网络。

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../includes/vpn-gateway-classic-rm-include.md)]

**用于点到站点连接的部署模型和工具**

[AZURE.INCLUDE [vpn-gateway-table-point-to-site](../includes/vpn-gateway-table-point-to-site-include.md)]

![点到站点连接示意图](./media/vpn-gateway-point-to-site-create/point2site.png "点到站点")


## 关于此配置

在此方案中，你将使用点到站点连接创建虚拟网络。点到站点连接由以下项组成：具有网关的 VNet、上载的根证书 .cer 文件、客户端证书和客户端上的 VPN 配置。

我们针对此配置使用以下值：

- 名称：**TestVNet**，使用地址空间 **192.168.0.0/16** 和 **10.254.0.0/16**。请注意，可为一个 VNet 使用多个地址空间。
- 子网名称：**FrontEnd**，使用 **192.168.1.0/24**
- 子网名称：**BackEnd**，使用 **10.254.1.0/24**
- 子网名称：**GatewaySubnet**，使用 **192.168.200.0/24**。要使网关正常工作，必须使用子网名称 *GatewaySubnet*。 
- VPN 客户端地址池：**172.16.201.0/24**。使用此点到站点连接连接到 VNet 的 VPN 客户端将会接收来自此池的 IP 地址。
- 订阅：如果你有多个订阅，请确保使用正确的订阅。
- 资源组：**TestRG**
- 位置：**中国东部**
- DNS 服务器：要用于名称解析的 DNS 服务器的 **IP 地址**。
- GW 名称：**GW**
- 公共 IP 名称：**GWIP**
- VpnType：**RouteBased**


## 开始之前

- 确保你拥有 Azure 订阅。如果你还没有 Azure 订阅，你可以注册一个[试用版](/pricing/1rmb-trial)。
	
- 你需要安装 Azure Resource Manager PowerShell cmdlet（1.0.2 或更高版本）。有关安装 PowerShell cmdlet 的详细信息，请参阅 [How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure/)。

## 为 Azure 配置点到站点连接

1. 在 PowerShell 控制台中，登录到你的 Azure 帐户。该 cmdlet 将提示你提供 Azure 帐户的登录凭据。登录后它会下载你的帐户设置，以便这些信息可供 Azure PowerShell 使用。

		Login-AzureRmAccount -Environment AzureChinaCloud

2. 获取 Azure 订阅的列表。

		Get-AzureRmSubscription

3. 指定要使用的订阅。

		Select-AzureRmSubscription -SubscriptionName "Name of subscription"

4. 在此配置中，以下 PowerShell 变量声明是使用你要使用的值声明的。声明的值将在示例脚本中使用。声明你要使用的值。使用下面的示例，并根据需要将值替换为你自己的值。

		$VNetName  = "TestVNet"
		$FESubName = "FrontEnd"
		$BESubName = "Backend"
		$GWSubName = "GatewaySubnet"
		$VNetPrefix1 = "192.168.0.0/16"
		$VNetPrefix2 = "10.254.0.0/16"
		$FESubPrefix = "192.168.1.0/24"
		$BESubPrefix = "10.254.1.0/24"
		$GWSubPrefix = "192.168.200.0/26"
		$VPNClientAddressPool = "172.16.201.0/24"
		$RG = "TestRG"
		$Location = "China East"
		$DNS = "8.8.8.8"
		$GWName = "GW"
		$GWIPName = "GWIP"
		$GWIPconfName = "gwipconf"
    	$P2SRootCertName = "ARMP2SRootCert.cer"

5. 创建新的资源组。

		New-AzureRmResourceGroup -Name $RG -Location $Location

6. 为虚拟网络创建子网配置，并将其命名为 FrontEnd、BackEnd 和 GatewaySubnet。请注意，这些前缀必须是上面声明的 VNet 地址空间的一部分。

		$fesub = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName -AddressPrefix $FESubPrefix
		$besub = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName -AddressPrefix $BESubPrefix
		$gwsub = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName -AddressPrefix $GWSubPrefix

7. 创建虚拟网络。请注意，指定的 DNS 服务器应该是可以解析所连接的资源名称的 DNS 服务器。对于本示例，我们使用了公共 IP 地址，但是你需要在此处放入自己的值。

		New-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG -Location $Location -AddressPrefix $VNetPrefix1,$VNetPrefix2 -Subnet $fesub, $besub, $gwsub -DnsServer $DNS

8. 指定刚刚创建的虚拟网络的变量。

		$vnet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG
		$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet

9. 请求动态分配的公共 IP 地址。网关需要此 IP 地址才能正常运行。稍后要将网关连接到网关 IP 配置。
		
		$pip = New-AzureRmPublicIpAddress -Name $GWIPName -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
		$ipconf = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
		
10. 将根证书 .cer 文件上载到 Azure。可以使用来自企业证书环境的根证书，或者使用自签名的根证书。最多可以上载 20 个根证书。有关使用 makecert 创建自签名根证书的说明，请参阅 [Working with self-signed root certificates for Point-to-Site configurations（为点到站点配置使用自签名根证书）](/documentation/articles/vpn-gateway-certificates-point-to-site/)。请注意，.cer 文件不应包含根证书的私钥。
	
	以下是其外观的示例。上载公开证书数据的较难部分是必须复制并粘贴整个字符串，且不包含空格。否则，上载将无法进行。你需要针对此步骤使用自己的证书 .cer 文件。请勿尝试从下面复制并粘贴示例。

		$MyP2SRootCertPubKeyBase64 = "MIIDUzCCAj+gAwIBAgIQRggGmrpGj4pCblTanQRNUjAJBgUrDgMCHQUAMDQxEjAQBgNVBAoTCU1pY3Jvc29mdDEeMBwGA1UEAxMVQnJrIExpdGUgVGVzdCBSb290IENBMB4XDTEzMDExOTAwMjQxOFoXDTIxMDExOTAwMjQxN1owNDESMBAGA1UEChMJTWljcm9zb2Z0MR4wHAYDVQQDExVCcmsgTGl0ZSBUZXN0IFJvb3QgQ0EwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC7SmE+iPULK0Rs7mQBO/6a6B6/G9BaMxHgDGzAmSG0Qsyt5e08aqgFnPdkMl3zRJw3lPKGha/JCvHRNrO8UpeAfc4IXWaqxx2iBipHjwmHPHh7+VB8lU0EJcUe7WBAI2n/sgfCwc+xKtuyRVlOhT6qw/nAi8e5don/iHPU6q7GCcnqoqtceQ/pJ8m66cvAnxwJlBFOTninhb2VjtvOfMQ07zPP+ZuYDPxvX5v3nd6yDa98yW4dZPuiGO2s6zJAfOPT2BrtyvLekItnSgAw3U5C0bOb+8XVKaDZQXbGEtOw6NZvD4L2yLd47nGkN2QXloiPLGyetrj3Z2pZYcrZBo8hAgMBAAGjaTBnMGUGA1UdAQReMFyAEOncRAPNcvJDoe4WP/gH2U+hNjA0MRIwEAYDVQQKEwlNaWNyb3NvZnQxHjAcBgNVBAMTFUJyayBMaXRlIFRlc3QgUm9vdCBDQYIQRggGmrpGj4pCblTanQRNUjAJBgUrDgMCHQUAA4IBAQCGyHhMdygS0g2tEUtRT4KFM+qqUY5HBpbIXNAav1a1dmXpHQCziuuxxzu3iq4XwnWUF1OabdDE2cpxNDOWxSsIxfEBf9ifaoz/O1ToJ0K757q2Rm2NWqQ7bNN8ArhvkNWa95S9gk9ZHZLUcjqanf0F8taJCYgzcbUSp+VBe9DcN89sJpYvfiBiAsMVqGPc/fHJgTScK+8QYrTRMubtFmXHbzBSO/KTAP5rBTxse88EGjK5F8wcedvge2Ksk6XjL3sZ19+Oj8KTQ72wihN900p1WQldHrrnbixSpmHBXbHr9U0NQigrJp5NphfuU5j81C8ixvfUdwyLmTv7rNA7GTAD"
		$p2srootcert = New-AzureRmVpnClientRootCertificate -Name $P2SRootCertName -PublicCertData $MyP2SRootCertPubKeyBase64

11. 创建 VNet 的虚拟网络网关。GatewayType 必须是 Vpn，VpnType 必须是 RouteBased。

		New-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG -Location $Location -IpConfigurations $ipconf -GatewayType Vpn -VpnType RouteBased -EnableBgp $false -GatewaySku Standard -VpnClientAddressPool $VPNClientAddressPool -VpnClientRootCertificates $p2srootcert

## 客户端配置

使用点到站点连接连接到 Azure 的每个客户端必须符合两个要点：必须配置 VPN 客户端以建立连接，并且客户端上必须安装了客户端证书。Windows 客户端有可用的 VPN 客户端配置包。有关详细信息，请参阅 [VPN Gateway FAQ（VPN 网关常见问题）](/documentation/articles/vpn-gateway-vpn-faq/#point-to-site-connections)。

1. 下载 VPN 客户端配置包。在此步骤中，请使用以下示例下载客户端配置包。

		Get-AzureRmVpnClientPackage -ResourceGroupName $RG -VirtualNetworkGatewayName $GWName -ProcessorArchitecture Amd64

	PowerShell cmdlet 将返回 URL 链接。复制并粘贴返回到 Web 浏览器的链接以将包下载到计算机。以下是返回的 URL 的外观示例。

    	"https://mdsbrketwprodsn1prod.blob.core.chinacloudapi.cn/cmakexe/4a431aa7-b5c2-45d9-97a0-859940069d3f/amd64/4a431aa7-b5c2-45d9-97a0-859940069d3f.exe?sv=2014-02-14&sr=b&sig=jSNCNQ9aUKkCiEokdo%2BqvfjAfyhSXGnRG0vYAv4efg0%3D&st=2016-01-08T07%3A10%3A08Z&se=2016-01-08T08%3A10%3A08Z&sp=r&fileExtension=.exe"
	
2. 生成并安装从客户端计算机上的根证书创建的客户端证书 (*.pfx)。可以使用你所习惯的任何安装方法。如果你使用自签名根证书但不熟悉如何执行此操作，可以参考 [Working with self-signed root certificates for Point-to-Site configurations（使用用于点到站点配置的自签名根证书）](/documentation/articles/vpn-gateway-certificates-point-to-site/)。

3. 若要连接到 VNet，请在客户端计算机上，导航到 VPN 连接，找到你刚创建的 VPN 连接。其名称将与虚拟网络的名称相同。单击“连接”。可能会出现与使用证书相关的弹出消息。如果出现此消息，请单击“继续”以使用提升的权限。

4. 在“连接”状态页上，单击“连接”以启动连接。如果你看到“选择证书”屏幕，请确保所显示的客户端证书就是你要用来连接的证书。如果不是，请使用下拉箭头选择正确的证书，然后单击“确定”。

5. 现在应已建立连接。你可以使用以下过程来验证连接。

## 验证连接

1. 若要验证你的 VPN 连接是否处于活动状态，请打开提升的命令提示符，然后运行 ipconfig/all。

2. 查看结果。请注意，你收到的 IP 地址是在配置中指定的点到站点 VPN 客户端地址池中的地址之一。结果应大致如下所示：
    
		PPP adapter VNetEast:
			Connection-specific DNS Suffix .:
			Description.....................: VNetEast
			Physical Address................:
			DHCP Enabled....................: No
			Autoconfiguration Enabled.......: Yes
			IPv4 Address....................: 172.16.201.3(Preferred)
			Subnet Mask.....................: 255.255.255.255
			Default Gateway.................:
			NetBIOS over Tcpip..............: Enabled

## 添加或删除根证书

证书用于对点到站点 VPN 的 VPN 客户端进行身份验证。以下步骤将引导你添加和删除根证书。

### 添加根证书

最多可以将 20 个根证书添加到 Azure。请遵循以下步骤来添加根证书。

1. 创建并准备要上载的新根证书。

		$P2SRootCertName2 = "ARMP2SRootCert2.cer"
		$MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"

2. 上载新根证书。请注意，一次只能添加一个根证书。

		Add-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayname $GWName -ResourceGroupName $RG -PublicCertData $MyP2SCertPubKeyBase64_2

3. 可以使用以下 cmdlet 来验证是否已已正确添加新证书。

		Get-AzureRmVpnClientRootCertificate -ResourceGroupName $RG -VirtualNetworkGatewayName $GWName

### 删除根证书

可以从 Azure 中删除根证书。删除某个根证书之后，从该根证书生成的客户端证书将不再能够通过点到站点连接到 Azure，直到它们安装 Azure 中的有效根证书所生成的客户端证书为止。

1. 删除根证书。

		Remove-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -PublicCertData "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"

 
2. 使用以下 cmdlet 来验证是否已已成功删除证书。

		Get-AzureRmVpnClientRootCertificate -ResourceGroupName $RG -VirtualNetworkGatewayName $GWName

## 管理吊销的客户端证书列表

你可以吊销客户端证书。证书吊销列表可让你选择性地拒绝基于单个客户端证书的点到站点连接。从 Azure 中删除根证书时，将吊销由吊销的根证书生成/签名的所有客户端证书的访问权限，因此吊销客户端证书可让你删除特定证书的访问权限。常见的做法是使用根证书管理团队或组织级别的访问权限，然后使用吊销的客户端证书针对单个用户进行精细的访问控制。

### 吊销客户端证书

1. 获取要吊销的客户端证书的指纹。

		$RevokedClientCert1 = "ClientCert1"
		$RevokedThumbprint1 = "‎ef2af033d0686820f5a3c74804d167b88b69982f"

2. 将指纹添加到吊销的指纹列表。

		Add-AzureRmVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -Thumbprint $RevokedThumbprint1

3. 确认指纹已添加到证书吊销列表。一次只能添加一个指纹。

		Get-AzureRmVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG

### 恢复客户端证书

可以通过从吊销的客户端证书列表中删除指纹来恢复客户端证书。

1.  从吊销的客户端证书指纹列表中删除指纹。

		Remove-AzureRmVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -Thumbprint $RevokedThumbprint1

2. 检查指纹是否已从吊销列表中删除。

		Get-AzureRmVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG

## 后续步骤

你可以将虚拟机添加到虚拟网络。请参阅[创建虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)以获取相关步骤。



<!---HONumber=Mooncake_0425_2016-->
