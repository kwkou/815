<properties 
   pageTitle="使用 Resource Manager 部署模型配置与虚拟网络的点到站点 VPN 网关连接 | Azure"
   description="通过创建点到站点 VPN 网关连接安全地连接到 Azure 虚拟网络。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>  

<tags 
   ms.service="vpn-gateway"
   ms.devlang="na"
   ms.topic="hero-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="08/31/2016"
   wacn.date="10/17/2016"
   ms.author="cherylmc" />  


# 使用 PowerShell 配置与 VNet 的点到站点连接

> [AZURE.SELECTOR]
- [PowerShell - Resource Manager](/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/)
- [门户 - 经典](/documentation/articles/vpn-gateway-point-to-site-create/)

使用点到站点 (P2S) 配置可以创建从单个客户端计算机到虚拟网络的安全连接。如果要从远程位置（例如从家里或会议室）连接到 VNet，或者只有少数几个需要连接到虚拟网络的客户端，则 P2S 连接会很有用。

本文逐步讲解如何在 **Resource Manager 部署模型**中创建具有点到站点连接的 VNet。这些步骤需要 PowerShell。目前无法在 Azure 门户中创建此端到端解决方案。

点到站点连接不需要 VPN 设备或面向公众的 IP 地址即可运行。可通过从客户端计算机启动连接来建立 VPN 连接。有关点到站点连接的详细信息，请参阅 [VPN Gateway FAQ](/documentation/articles/vpn-gateway-vpn-faq/#point-to-site-connections)（VPN 网关常见问题）及 [Planning and Design](/documentation/articles/vpn-gateway-plan-design/)（规划和设计）。



**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

**用于点到站点连接的部署模型和工具**

[AZURE.INCLUDE [vpn-gateway-table-point-to-site](../../includes/vpn-gateway-table-point-to-site-include.md)]

![点到站点连接示意图](./media/vpn-gateway-howto-point-to-site-rm-ps/p2srm.png "点到站点")


## 关于此配置

在此方案中，将使用点到站点连接创建虚拟网络。参考这些说明还可以生成此配置所需的证书。P2S 连接由以下项组成：具有 VPN 网关的 VNet、根证书 .cer 文件（公钥）、客户端证书和客户端上的 VPN 配置。

此配置将使用以下值。在本文的第 [1](#declare) 部分设置变量。可以使用这些步骤作为演练并按原样使用这些值，或者根据环境更改这些值。

- 名称：**VNet1**，使用地址空间 **192.168.0.0/16** 和 **10.254.0.0/16**。请注意，可为一个 VNet 使用多个地址空间。
- 子网名称：**FrontEnd**，使用 **192.168.1.0/24**
- 子网名称：**BackEnd**，使用 **10.254.1.0/24**
- 子网名称：**GatewaySubnet**，使用 **192.168.200.0/24**。要使网关正常工作，必须使用子网名称 *GatewaySubnet*。
- VPN 客户端地址池：**172.16.201.0/24**。使用此点到站点连接连接到 VNet 的 VPN 客户端接收来自此池的 IP 地址。
- 订阅：如果您有多个订阅，请确保使用正确的订阅。
- 资源组：**TestRG**
- 位置：**中国东部**
- DNS 服务器：要用于名称解析的 DNS 服务器的 **IP 地址**。
- GW 名称：**GW**
- 公共 IP 名称：**GWIP**
- VpnType：**RouteBased**


## 开始之前

- 确保你拥有 Azure 订阅。如果你还没有 Azure 订阅，你可以注册一个[试用版](/pricing/1rmb-trial)。
	
- 安装 Azure Resource Manager PowerShell cmdlet（1.0.2 或更高版本）。有关安装 PowerShell cmdlet 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。使用 PowerShell 进行此配置时，请确保以管理员身份运行。

## <a name="declare"></a>第 1 部分 - 登录并设置变量

在本部分，可以登录并声明用于此配置的值。声明的值将在示例脚本中使用。根据自己的环境更改值。或者，可以使用声明的值，以练习的形式逐步执行每个步骤。

1. 在 PowerShell 控制台中，登录到你的 Azure 帐户。该 cmdlet 将提示你提供 Azure 帐户的登录凭据。登录后它会下载你的帐户设置，以便这些信息可供 Azure PowerShell 使用。

		Login-AzureRmAccount -Environment AzureChinaCloud

2. 获取 Azure 订阅的列表。

		Get-AzureRmSubscription

3. 指定要使用的订阅。

		Select-AzureRmSubscription -SubscriptionName "Name of subscription"

4. 声明要使用的变量。使用以下示例，根据需要将值替换为自己的值。

		$VNetName  = "VNet1"
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


## 第 2 部分 - 配置 VNet 


1. 创建资源组。

		New-AzureRmResourceGroup -Name $RG -Location $Location

2. 为虚拟网络创建子网配置，并将其命名为 *FrontEnd* 、 *BackEnd* 和 *GatewaySubnet* 。这些前缀必须是上面声明的 VNet 地址空间的一部分。

		$fesub = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName -AddressPrefix $FESubPrefix
		$besub = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName -AddressPrefix $BESubPrefix
		$gwsub = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName -AddressPrefix $GWSubPrefix

3. 创建虚拟网络。指定的 DNS 服务器应该是可以解析所连接的资源名称的 DNS 服务器。在本示例中，我们使用了公共 IP 地址。请务必使用自己的值。
	
		New-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG -Location $Location -AddressPrefix $VNetPrefix1,$VNetPrefix2 -Subnet $fesub, $besub, $gwsub -DnsServer $DNS

4. 为创建的虚拟网络指定变量。

		$vnet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG
		$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet

5. 请求动态分配的公共 IP 地址。网关需要此 IP 地址才能正常运行。稍后要将网关连接到网关 IP 配置。
		
		$pip = New-AzureRmPublicIpAddress -Name $GWIPName -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
		$ipconf = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip

## 第 3 部分 - 添加受信任证书

Azure 使用证书来验证需要通过 P2S 进行连接的客户端。请将根证书的 .cer 文件（公钥）导入 Azure。在将 Base64 编码 X.509 (.cer) 文件添加到 Azure 时，则是在告诉 Azure 信任该文件所代表的根证书。

如果使用的是企业解决方案，可以使用现有的证书链。如果使用的不是企业 CA 解决方案，可以创建自签名根证书。创建自签名证书的方法之一是使用 Makecert。有关使用 *makecert* 创建自签名根证书的说明，请参阅[为点到站点配置使用自签名根证书](/documentation/articles/vpn-gateway-certificates-point-to-site/)。

无论以何种方式获取证书，都需要将证书的 .cer 文件上载到 Azure，同时生成要在连接的客户端上安装的客户端证书。不要直接在客户端上安装自签名的证书。稍后可以在本文的[客户端配置](#cc)部分中生成客户端证书。
		
最多可以将 20 个受信任的证书添加到 Azure。若要获取公钥，请将证书导出为 Base64 编码 X.509 (.cer) 文件。导出为 .cer 的方法之一是打开 **certmger.msc**，在“个人/证书”中找到该证书。单击右键，然后将证书导出为不包含私钥的“Base-64 编码 X.509 (.CER)”证书。请记下导出到.cer 文件的文件路径。以下是获取证书的 Base64 字符串表示形式的一个示例。

将受信任的证书添加到 Azure。在此步骤中，需要使用自己的 .cer 文件路径。请特别注意本文第 1 部分中设置的 $P2SRootCertName = "ARMP2SRootCert.cer" 变量。如果证书名称不同，请相应地调整变量。
    
		$filePathForCert = "pasteYourCerFilePathHere"
		$cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
		$CertBase64 = [system.convert]::ToBase64String($cert.RawData)
		$p2srootcert = New-AzureRmVpnClientRootCertificate -Name $P2SRootCertName -PublicCertData $CertBase64

## 第 4 部分 - 创建 VPN 网关

配置并创建 VNet 的虚拟网络网关。 *-GatewayType* 必须是 **Vpn**， *-VpnType* 必须是 **RouteBased**。此步骤最多需要 45 分钟才能完成。

		New-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
		-Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
		-VpnType RouteBased -EnableBgp $false -GatewaySku Standard `
		-VpnClientAddressPool $VPNClientAddressPool -VpnClientRootCertificates $p2srootcert

## 第 5 部分 - 下载 VPN 客户端配置包

使用 P2S 连接到 Azure 的客户端必须安装客户端证书和 VPN 客户端配置包。Windows 客户端有可用的 VPN 客户端配置包。VPN 客户端包中包含的信息可用于配置 Windows 内置的 VPN 客户端软件，与要连接到的特定 VPN 相关。该程序包不安装额外的软件。有关详细信息，请参阅 [VPN Gateway FAQ](/documentation/articles/vpn-gateway-vpn-faq/#point-to-site-connections)（VPN 网关常见问题）。

1. 创建网关后，请使用以下示例下载客户端配置包：

		Get-AzureRmVpnClientPackage -ResourceGroupName $RG `
		-VirtualNetworkGatewayName $GWName -ProcessorArchitecture Amd64

2. PowerShell cmdlet 返回 URL 链接。下面示范了返回的 URL 形式：

    	"https://mdsbrketwprodsn1prod.blob.core.chinacloudapi.cn/cmakexe/4a431aa7-b5c2-45d9-97a0-859940069d3f/amd64/4a431aa7-b5c2-45d9-97a0-859940069d3f.exe?sv=2014-02-14&sr=b&sig=jSNCNQ9aUKkCiEokdo%2BqvfjAfyhSXGnRG0vYAv4efg0%3D&st=2016-01-08T07%3A10%3A08Z&se=2016-01-08T08%3A10%3A08Z&sp=r&fileExtension=.exe"

3. 复制并粘贴 Web 浏览器中返回的链接以下载包。然后在客户端计算机上安装该包。

4. 在客户端计算机上，导航到“网络设置”，然后单击“VPN”。此时将会列出连接。其中显示了要连接到的虚拟网络的名称，如下所示：

	![VPN 客户端](./media/vpn-gateway-howto-point-to-site-rm-ps/vpn.png "VPN 客户端")  


## <a name="cc"></a>第 6 部分 - 生成客户端证书

接下来，生成客户端证书。可以为每个要连接的客户端生成唯一证书，也可以在多个客户端上使用相同的证书。生成唯一客户端证书的优势是能够根据需要吊销单个证书。否则，如果每个人都使用相同的客户端证书，在需要吊销某个客户端的证书时，必须为所有使用该证书进行身份验证的客户端生成并安装新证书。

- 如果使用的是企业证书解决方案，请使用通用名称值格式“name@yourdomain.com”生成客户端证书，而不要使用 NetBIOS“域\\用户名”格式。

- 如果使用自签名的证书解决方案，请参阅 [Working with self-signed root certificates for Point-to-Site configurations](/documentation/articles/vpn-gateway-certificates-point-to-site/)（为点到站点配置使用自签名根证书）生成客户端证书。

## 第 7 部分 - 安装客户端证书

在要连接到虚拟网络的每台计算机上安装客户端证书。身份验证时需要客户端证书。可以自动安装客户端证书，也可以手动安装。以下步骤指导如何手动导出和安装客户端证书。

1. 若要导出客户端证书，可以使用 *certmgr.msc* 。右键单击要导出的客户端证书，单击“所有任务”，然后单击“导出”。
2. 导出包含私钥的客户端证书。这是一个 *.pfx* 文件。请确保记录或记住为此证书设置的密码（密钥）。
3. 将 *.pfx* 文件复制到客户端计算机。在客户端计算机上，双击 *.pfx* 文件，安装该证书。系统请求你输入密码时，请输入相应的密码。请勿修改安装位置。


## 第 8 部分 - 连接到 Azure

1. 若要连接到 VNet，请在客户端计算机上导航到 VPN 连接，找到创建的 VPN 连接。其名称与虚拟网络的名称相同。单击“连接”。可能会出现与使用证书相关的弹出消息。如果出现此消息，请单击“继续”以使用提升的权限。

2. 在“连接”状态页上，单击“连接”开始连接。如果你看到“选择证书”屏幕，请确保所显示的客户端证书就是你要用来连接的证书。如果不是，请使用下拉箭头选择正确的证书，然后单击“确定”。

	![VPN 客户端 2](./media/vpn-gateway-howto-point-to-site-rm-ps/clientconnect.png "VPN 客户端连接")  


3. 现在应已建立连接。

	![VPN 客户端 3](./media/vpn-gateway-howto-point-to-site-rm-ps/connected.png "VPN 客户端连接 2")  


## 第 9 部分 - 验证连接

1. 若要验证你的 VPN 连接是否处于活动状态，请打开提升的命令提示符，然后运行 *ipconfig/all* 。

2. 查看结果。请注意，你收到的 IP 地址是在配置中指定的点到站点 VPN 客户端地址池中的地址之一。结果应大致如下所示：
    
		PPP adapter VNet1:
			Connection-specific DNS Suffix .:
			Description.....................: VNet1
			Physical Address................:
			DHCP Enabled....................: No
			Autoconfiguration Enabled.......: Yes
			IPv4 Address....................: 172.16.201.3(Preferred)
			Subnet Mask.....................: 255.255.255.255
			Default Gateway.................:
			NetBIOS over Tcpip..............: Enabled

## 添加或删除受信任的根证书

证书用于对点到站点 VPN 的 VPN 客户端进行身份验证。以下步骤指导如何添加和删除根证书。在将 Base64 编码 X.509 (.cer) 文件添加到 Azure 时，则是在告诉 Azure 信任该文件所代表的根证书。

可以使用 PowerShell 或 Azure 门户添加或删除受信任的根证书。若要使用 Azure 门户执行此操作，请转到“虚拟网络网关”>“设置”>“点到站点配置”>“根证书”。以下步骤使用 PowerShell 完成相关任务。

### 添加受信任的根证书

最多可以将 20 个受信任的根证书 .cer 文件添加到 Azure。请遵循以下步骤来添加根证书。

1. 创建并准备好要添加到 Azure 的新根证书。将公钥导出为 Base-64 编码的 X.509 (.CER)，使用文本编辑器将它打开。然后，仅复制如下所示的部分。
 
	按以下示例中所示复制所需的值。

	![证书](./media/vpn-gateway-howto-point-to-site-rm-ps/copycert.png "证书")  

	
2. 在以下示例中，请指定证书名称和密钥信息作为变量。将信息替换为自己的值。

		$P2SRootCertName2 = "ARMP2SRootCert2.cer"
		$MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"


3. 添加新的根证书。一次只能添加一个证书。

		Add-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayname "GW" -ResourceGroupName "TestRG" -PublicCertData $MyP2SCertPubKeyBase64_2


4. 可以使用以下 cmdlet 来验证是否已已正确添加新证书。

		Get-AzureRmVpnClientRootCertificate -ResourceGroupName "TestRG" `
		-VirtualNetworkGatewayName "GW"

### 删除受信任的根证书

可以从 Azure 中删除受信任的根证书。删除受信任的证书时，从根证书生成的客户端证书不再能够通过点到站点连接到 Azure。如果希望客户端能够连接，需要在客户端上安装新客户端证书，这些证书是从 Azure 信任的证书生成的。

1. 若要删除受信任的根证书，请修改以下示例：

	声明变量。

		$P2SRootCertName2 = "ARMP2SRootCert2.cer"
		$MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"

2. 删除证书

		Remove-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -PublicCertData $MyP2SCertPubKeyBase64_2
 
3. 使用以下 cmdlet 来验证是否已已成功删除证书。

		Get-AzureRmVpnClientRootCertificate -ResourceGroupName "TestRG" `
		-VirtualNetworkGatewayName "GW"

## 管理吊销的客户端证书列表

你可以吊销客户端证书。证书吊销列表可让你选择性地拒绝基于单个客户端证书的点到站点连接。如果从 Azure 中删除根证书 .cer，将吊销所有由吊销的根证书生成/签名的客户端证书的访问权限。如果需要，也可以吊销特定的客户端证书而不是根证书。这样，从根证书生成的证书仍然有效。常见的做法是使用根证书管理团队或组织级别的访问权限，然后使用吊销的客户端证书针对单个用户进行精细的访问控制。

### 吊销客户端证书

1. 获取要吊销的客户端证书的指纹。

		$RevokedClientCert1 = "ClientCert1"
		$RevokedThumbprint1 = "‎ef2af033d0686820f5a3c74804d167b88b69982f"

2. 将指纹添加到吊销的指纹列表。

		Add-AzureRmVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
		-VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -Thumbprint $RevokedThumbprint1

3. 确认指纹已添加到证书吊销列表。必须一次添加一个指纹。

		Get-AzureRmVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG

### 恢复客户端证书

可以通过从吊销的客户端证书列表中删除指纹来恢复客户端证书。

1.  从吊销的客户端证书指纹列表中删除指纹。

		Remove-AzureRmVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
		-VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -Thumbprint $RevokedThumbprint1

2. 检查指纹是否已从吊销列表中删除。

		Get-AzureRmVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG

## 后续步骤

你可以将虚拟机添加到虚拟网络。请参阅[创建虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)以获取相关步骤。

<!---HONumber=Mooncake_1010_2016-->