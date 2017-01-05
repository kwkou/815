<properties
    pageTitle="使用 Resource Manager 部署模型配置与虚拟网络的点到站点 VPN 网关连接 | Azure"
    description="通过创建点到站点 VPN 网关连接安全地连接到 Azure 虚拟网络。"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="carmonm"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="3eddadf6-2e96-48c4-87c6-52a146faeec6"
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="10/17/2016"
    wacn.date="01/05/2017"
    ms.author="cherylmc" />  


# 使用 PowerShell 配置与 VNet 的点到站点连接
> [AZURE.SELECTOR]
- [Resource Manager - Azure 门户预览](/documentation/articles/vpn-gateway-howto-point-to-site-resource-manager-portal/)
- [Resource Manager - PowerShell](/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/)
- [经典 - Azure 门户预览](/documentation/articles/vpn-gateway-howto-point-to-site-classic-azure-portal/)

通过点到站点 (P2S) 配置，可以创建单台客户端计算机到虚拟网络的安全连接。如果要从远程位置（例如从家里或会议室）连接到 VNet，或者只有少数几个需要连接到虚拟网络的客户端，则 P2S 连接会很有用。

点到站点连接不需要 VPN 设备或面向公众的 IP 地址即可运行。可通过从客户端计算机启动连接来建立 VPN 连接。有关点到站点连接的详细信息，请参阅 [VPN Gateway FAQ](/documentation/articles/vpn-gateway-vpn-faq/#point-to-site-connections)（VPN 网关常见问题）及 [Planning and Design](/documentation/articles/vpn-gateway-plan-design/)（规划和设计）。

本文逐步讲解如何使用 PowerShell 在 Resource Manager 部署模型中创建具有点到站点连接的 VNet。

### P2S 连接的部署模型和方法
[AZURE.INCLUDE [部署模型](../../includes/vpn-gateway-deployment-models-include.md)]

下表显示了 P2S 配置的两种部署模型和可用的部署方法。当有配置步骤相关的文章发布时，我们会直接从此表格链接到该文章。

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-table-point-to-site-include.md)]

## 基本工作流
![点到站点连接示意图](./media/vpn-gateway-howto-point-to-site-rm-ps/p2srm.png "点到站点")  


在此方案中，你将使用点到站点连接创建虚拟网络。参考这些说明还可以生成此配置所需的证书。P2S 连接由以下项组成：具有 VPN 网关的 VNet、根证书 .cer 文件（公钥）、客户端证书和客户端上的 VPN 配置。

此配置将使用以下值。在本文的第 [1](#declare) 部分设置变量。可以使用这些步骤作为演练并按原样使用这些值，或者根据环境更改这些值。

### <a name="example"></a>示例值
* **名称：VNet1**
* **地址空间：192.168.0.0/16** 和 **10.254.0.0/16**<br>对于此示例，我们使用多个地址空间来演示此配置将使用多个地址空间。但是，多个地址空间不是此配置所必需的。
* **子网名称：FrontEnd**
  * **子网地址范围：192.168.1.0/24**
* **子网名称：BackEnd**
  * **子网地址范围：10.254.1.0/24**
* **子网名称：GatewaySubnet**<br>要使 VPN 网关正常工作，必须使用子网名称 *GatewaySubnet*。
  * **子网地址范围：192.168.200.0/24**
* **VPN 客户端地址池：172.16.201.0/24**<br>使用此点到站点连接连接到 VNet 的 VPN 客户端接收来自 VPN 客户端地址池的 IP 地址。
* **订阅：**如果你有多个订阅，请确保使用正确的订阅。
* **资源组：TestRG**
* **位置：中国东部**
* **DNS 服务器：要用于名称解析的 DNS 服务器的 IP 地址**。
* **网关名称：Vnet1GW**
* **公共 IP 名称：VNet1GWPIP**
* **VpnType：RouteBased**

## 开始之前
* 确保你拥有 Azure 订阅。如果你还没有 Azure 订阅，可以注册获取[1元帐户](/pricing/1rmb-trial)。
* 安装最新版本的 Azure Resource Manager PowerShell cmdlet。有关安装 PowerShell cmdlet 的详细信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。使用 PowerShell 进行此配置时，请确保以管理员身份运行。

## <a name="declare"></a>第 1 部分 - 登录并设置变量
在本部分，可以登录并声明用于此配置的值。声明的值将在示例脚本中使用。根据自己的环境更改值。或者，可以使用声明的值，以练习的形式逐步执行每个步骤。

1. 在 PowerShell 控制台中，登录到你的 Azure 帐户。该 cmdlet 将提示你提供 Azure 帐户的登录凭据。登录后它会下载你的帐户设置，以便这些信息可供 Azure PowerShell 使用。
   
        Login-AzureRmAccount -EnvironmentName AzureChinaCloud 
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
        $GWName = "VNet1GW"
        $GWIPName = "VNet1GWPIP"
        $GWIPconfName = "gwipconf"

## <a name="ConfigureVNet"></a>第 2 部分 - 配置 VNet
1. 创建资源组。
   
        New-AzureRmResourceGroup -Name $RG -Location $Location
2. 为虚拟网络创建子网配置，并将其命名为 *FrontEnd*、*BackEnd* 和 *GatewaySubnet*。这些前缀必须是声明的 VNet 地址空间的一部分。
   
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

## <a name="Certificates"></a>第 3 部分 - 证书
Azure 使用证书对点到站点 VPN 的 VPN 客户端进行身份验证。从企业证书解决方案生成的根证书或自签名根证书将公共证书数据（不是私钥）导出为 Base-64 编码 X.509 .cer 文件。然后从根证书将公共证书数据导入到 Azure。此外，需要从客户端的根证书生成客户端证书。每个要使用 P2S 连接连接到虚拟网络的客户端都必须已安装从根证书生成的客户端证书。

### <a name="cer"></a>1.获取根证书的 .cer 文件
需要为要使用的根证书获取公共证书数据。

* 如果使用企业证书系统，请获取要使用的根证书的 .cer 文件。
* 如果使用的不是企业证书解决方案，则需要生成自签名根证书。有关适用于 Windows 10 的步骤，请参阅 [Working with self-signed root certificates for Point-to-Site configurations](/documentation/articles/vpn-gateway-certificates-point-to-site/)（为点到站点配置使用自签名根证书）。

1. 若要从证书中获取 .cer 文件，请打开 **certmgr.msc** 并找到根证书。右键单击自签名根证书，单击“所有任务”，然后单击“导出”。此操作将打开“证书导出向导”。
2. 在向导中，单击“下一步”，选择“否，不导出私钥”，然后单击“下一步”。
3. 在“导出文件格式”页上，选择“Base-64 编码的 X.509 (.CER)”。 然后，单击“下一步”。
4. 在“要导出的文件”中，单击“浏览”并选择要导出证书的位置。在“文件名”中，为证书文件命名。然后单击“下一步”。
5. 单击“完成”以导出证书。

### <a name="generate"></a>2.生成客户端证书
接下来，生成客户端证书。可以为每个要连接的客户端生成唯一证书，也可以在多个客户端上使用相同的证书。生成唯一客户端证书的优势是能够根据需要吊销单个证书。否则，如果每个人都使用相同的客户端证书，在需要吊销某个客户端的证书时，必须为所有使用该证书进行身份验证的客户端生成并安装新证书。稍后将在本练习中的每台客户端计算机上安装客户端证书。

* 如果使用企业证书解决方案，请使用公用名称值格式“name@yourdomain.com”（而不是 NetBIOS“DOMAIN\\username” 格式）生成客户端证书。
* 如果使用自签名的证书解决方案，请参阅 [Working with self-signed root certificates for Point-to-Site configurations](/documentation/articles/vpn-gateway-certificates-point-to-site/)（为点到站点配置使用自签名根证书）生成客户端证书。

### <a name="exportclientcert"></a>3.导出客户端证书
身份验证时需要客户端证书。生成客户端证书后，将其导出。导出的客户端证书稍后将安装在每台客户端计算机上。

1. 若要导出客户端证书，可以使用 *certmgr.msc*。右键单击要导出的客户端证书，单击“所有任务”，然后单击“导出”。
2. 导出包含私钥的客户端证书。这是一个 *.pfx* 文件。请确保记录或记住为此证书设置的密码（密钥）。

### <a name="upload"></a>4.上载根证书 .cer 文件
声明证书名称的变量，并将值替换为你自己的值：

        $P2SRootCertName = "Mycertificatename.cer"

将根证书的公共证书数据添加到 Azure。最多可以上载 20 个根证书的文件。不要将根证书的私钥上载到 Azure。上载 .Cer 文件后，Azure 将使用它来对连接到虚拟网络的客户端进行身份验证。

将文件路径替换为你自己的值，然后运行这些 cmdlet。

        $filePathForCert = "C:\cert\Mycertificatename.cer"
        $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
        $CertBase64 = [system.convert]::ToBase64String($cert.RawData)
        $p2srootcert = New-AzureRmVpnClientRootCertificate -Name $P2SRootCertName -PublicCertData $CertBase64

## <a name="creategateway"></a>第 4 部分 - 创建 VPN 网关
配置并创建 VNet 的虚拟网络网关。*-GatewayType* 必须是 **Vpn**，*-VpnType* 必须是 **RouteBased**。此步骤最多需要 45 分钟才能完成。

        New-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
        -Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
        -VpnType RouteBased -EnableBgp $false -GatewaySku Standard `
        -VpnClientAddressPool $VPNClientAddressPool -VpnClientRootCertificates $p2srootcert

## <a name="clientconfig"></a>第 5 部分 - 下载 VPN 客户端配置包
使用 P2S 连接到 Azure 的客户端必须安装客户端证书和 VPN 客户端配置包。Windows 客户端有可用的 VPN 客户端配置包。VPN 客户端包中包含的信息可用于配置 Windows 内置的 VPN 客户端软件，与要连接到的特定 VPN 相关。该程序包不安装额外的软件。有关详细信息，请参阅 [VPN Gateway FAQ](/documentation/articles/vpn-gateway-vpn-faq/#point-to-site-connections)（VPN 网关常见问题）。

1. 创建网关后，可以下载客户端配置包。此示例为 64 位客户端下载程序包。如果要下载 32 位客户端，请将“Amd64”替换为“x86”。还可以使用 Azure 门户预览下载 VPN 客户端。
   
        Get-AzureRmVpnClientPackage -ResourceGroupName $RG `
        -VirtualNetworkGatewayName $GWName -ProcessorArchitecture Amd64
2. PowerShell cmdlet 返回 URL 链接。下面示范了返回的 URL 形式：
   
        "https://mdsbrketwprodsn1prod.blob.core.chinacloudapi.cn/cmakexe/4a431aa7-b5c2-45d9-97a0-859940069d3f/amd64/4a431aa7-b5c2-45d9-97a0-859940069d3f.exe?sv=2014-02-14&sr=b&sig=jSNCNQ9aUKkCiEokdo%2BqvfjAfyhSXGnRG0vYAv4efg0%3D&st=2016-01-08T07%3A10%3A08Z&se=2016-01-08T08%3A10%3A08Z&sp=r&fileExtension=.exe"
3. 复制并粘贴 Web 浏览器中返回的链接以下载包。然后在客户端计算机上安装该包。如果显示 SmartScreen 弹出窗口，请单击“更多信息”，然后单击“仍要运行”以安装该包。
4. 在客户端计算机上，导航到“网络设置”，然后单击“VPN”。此时将会列出连接。其中显示了要连接到的虚拟网络的名称，如以下示例所示：
   
    ![VPN 客户端](./media/vpn-gateway-howto-point-to-site-rm-ps/vpn.png "VPN 客户端")  


## <a name="clientcertificate"></a>第 6 部分 - 安装客户端证书
每个客户端计算机必须有一个客户端证书才能执行身份验证。安装客户端证书时，需要使用导出客户端证书时创建的密码。

1. 将 .pfx 文件复制到客户端计算机。
2. 双击 .pfx 文件将它安装。请勿修改安装位置。

## <a name="connect"></a>第 7 部分 - 连接到 Azure
1. 若要连接到 VNet，请在客户端计算机上导航到 VPN 连接，找到创建的 VPN 连接。其名称与虚拟网络的名称相同。单击“连接”。可能会出现与使用证书相关的弹出消息。如果出现此消息，请单击“继续”以使用提升的权限。
2. 在“连接”状态页上，单击“连接”开始连接。如果你看到“选择证书”屏幕，请确保所显示的客户端证书就是你要用来连接的证书。如果不是，请使用下拉箭头选择正确的证书，然后单击“确定”。
   
    ![VPN 客户端连接](./media/vpn-gateway-howto-point-to-site-rm-ps/clientconnect.png "VPN 客户端连接")  

3. 现在应已建立连接。
   
    ![已建立连接](./media/vpn-gateway-howto-point-to-site-rm-ps/connected.png "已建立连接")  


## <a name="verify"></a>第 8 部分 - 验证连接
1. 若要验证你的 VPN 连接是否处于活动状态，请打开提升的命令提示符，然后运行 *ipconfig/all*。
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

## <a name="addremovecert"></a>添加或删除受信任的根证书
证书用于对点到站点 VPN 的 VPN 客户端进行身份验证。以下步骤指导如何添加和删除根证书。在将 Base64 编码 X.509 (.cer) 文件添加到 Azure 时，则是在告诉 Azure 信任该文件所代表的根证书。

可以使用 PowerShell 或 Azure 门户预览添加或删除受信任的根证书。若要使用 Azure 门户预览执行此操作，请转到“虚拟网络网关”>“设置”>“点到站点配置”>“根证书”。以下步骤使用 PowerShell 完成相关任务。

### 添加受信任的根证书
最多可以将 20 个受信任的根证书 .cer 文件添加到 Azure。请遵循以下步骤来添加根证书。

1. 创建并准备好要添加到 Azure 的新根证书。将公钥导出为 Base-64 编码的 X.509 (.CER)，使用文本编辑器将它打开。然后，仅复制如下所示的部分。
   
    按以下示例中所示复制所需的值：
   
    ![证书](./media/vpn-gateway-howto-point-to-site-rm-ps/copycert.png "证书")  

2. 指定证书名称和密钥信息作为变量。将信息替换为自己的值，如以下示例中所示：
   
        $P2SRootCertName2 = "ARMP2SRootCert2.cer"
        $MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"
3. 添加新的根证书。一次只能添加一个证书。
   
        Add-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG" -PublicCertData $MyP2SCertPubKeyBase64_2
4. 可以使用以下 cmdlet 来验证是否已已正确添加新证书。
   
        Get-AzureRmVpnClientRootCertificate -ResourceGroupName "TestRG" `
        -VirtualNetworkGatewayName "VNet1GW"

### 删除受信任的根证书
可以从 Azure 中删除受信任的根证书。删除受信任的证书时，从根证书生成的客户端证书不再能够通过点到站点连接到 Azure。如果希望客户端能够连接，需要在客户端上安装新客户端证书，这些证书是从 Azure 信任的证书生成的。

1. 若要删除受信任的根证书，请修改以下示例：
   
    声明变量。
   
        $P2SRootCertName2 = "ARMP2SRootCert2.cer"
        $MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"
2. 删除证书。
   
        Remove-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -PublicCertData $MyP2SCertPubKeyBase64_2
3. 使用以下 cmdlet 来验证是否已已成功删除证书。
   
        Get-AzureRmVpnClientRootCertificate -ResourceGroupName "TestRG" `
        -VirtualNetworkGatewayName "VNet1GW"

## <a name="revoke"></a>管理已吊销客户端证书的列表
你可以吊销客户端证书。证书吊销列表可让你选择性地拒绝基于单个客户端证书的点到站点连接。如果从 Azure 中删除根证书 .cer，将吊销所有由吊销的根证书生成/签名的客户端证书的访问权限。如果需要，也可以吊销特定的客户端证书而不是根证书。这样，从根证书生成的证书仍然有效。

常见的做法是使用根证书管理团队或组织级别的访问权限，然后使用吊销的客户端证书针对单个用户进行精细的访问控制。

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

1. 从吊销的客户端证书指纹列表中删除指纹。
   
        Remove-AzureRmVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
        -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -Thumbprint $RevokedThumbprint1
2. 检查指纹是否已从吊销列表中删除。
   
        Get-AzureRmVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG

## 后续步骤
连接完成后，即可将虚拟机添加到虚拟网络。有关详细信息，请参阅[虚拟机](/documentation/services/virtual-machines/)。

<!---HONumber=Mooncake_Quality_Review_1230_2016-->