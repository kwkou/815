<properties
    pageTitle="删除虚拟网络网关：PowerShell：Azure 经典 | Azure"
    description="在经典部署模型中使用 PowerShell 删除虚拟网络网关。"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="timlt"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic=""
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/20/2017"
    wacn.date="04/17/2017"
    ms.author="cherylmc"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="e37c67bd5a0319a8ce7a9e5f4c668e91cd2064fa"
    ms.lasthandoff="04/06/2017" />

# <a name="delete-a-virtual-network-gateway-using-powershell-classic"></a>使用 PowerShell 删除虚拟网络网关（经典）
> [AZURE.SELECTOR]
- [Resource Manager - PowerShell](/documentation/articles/vpn-gateway-delete-vnet-gateway-powershell/)
- [经典 - PowerShell](/documentation/articles/vpn-gateway-delete-vnet-gateway-classic-powershell/)

可在经典部署模型中使用 PowerShell 删除 VPN 网关。 删除虚拟网络网关后，修改网络配置文件以删除不再使用的元素。

## <a name="part-1-connect-to-azure"></a>第 1 部分： 连接到 Azure

### <a name="1-install-the-latest-powershell-cmdlets"></a>1.安装最新的 PowerShell cmdlet。

下载和安装最新版本的 Azure 服务管理 (SM) PowerShell cmdlet。 有关详细信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。

### <a name="2-connect-to-your-azure-account"></a>2.连接到你的 Azure 帐户。 

使用提升的权限打开 PowerShell 控制台，然后连接到帐户。 使用下面的示例来帮助连接：

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

检查该帐户的订阅。

    Get-AzureRmSubscription

如果有多个订阅，请选择要使用的订阅。

    Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"

接下来，使用以下 cmdlet 将 Azure 订阅添加到经典部署模型的 PowerShell。

    Add-AzureAccount -Environment AzureChinaCloud

## <a name="part-2-export-and-view-the-network-configuration-file"></a>第 2 部分： 导出并查看网络配置文件

在计算机上创建一个目录，然后将网络配置文件导出到该目录。 使用此文件查看当前配置信息并修改网络配置。

本例中，网络配置文件导出到 C:\AzureNet。

     Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml

使用文本编辑器打开文件，并查看经典 VNet 的名称。 在 Azure 门户预览中创建 VNet 时，Azure 使用的全名在 Azure 门户预览中不可见。 例如，在 Azure 门户预览中命名为“ClassicVNet1”的 VNet 可能在网络配置文件中具有更长的名称。 名称可能如下所示：“Group ClassicRG1 ClassicVNet1”。 VNet 名称被列为“VirtualNetworkSite name =”。<br>运行 PowerShell cmdlet 时，请使用网络配置文件中的名称。

## <a name="part-3-delete-the-virtual-network-gateway"></a>第 3 部分： 删除虚拟网络网关

删除虚拟网络网关时，通过该网关的所有 VNet 连接都将断开。 如果 P2S 客户端连接到 VNet，它们将断开连接且不发出警告。

此示例将删除虚拟网络网关。 运行此示例时，请使用网络配置文件中虚拟网络的全名。

    Remove-AzureVNetGateway -VNetName "Group ClassicRG1 ClassicVNet1"

如果成功，则返回显示：

    Status : Successful

## <a name="part-4-modify-the-network-configuration-file"></a>第 4 部分： 修改网络配置文件

删除虚拟网络网关时，cmdlet 不会修改网络配置文件。 需修改文件才可删除不再使用的元素。 以下部分可帮助你修改下载的网络配置文件。

### <a name="local-network-site-references"></a>本地网络站点引用

若要删除站点引用信息，请更改 ConnectionsToLocalNetwork/LocalNetworkSiteRef 的配置。 删除本地站点引用会触发 Azure 删除隧道。 根据你创建的配置，你可能没有列出 LocalNetworkSiteRef。

    <Gateway>
       <ConnectionsToLocalNetwork>
         <LocalNetworkSiteRef name="D1BFC9CB_Site2">
           <Connection type="IPsec" />
         </LocalNetworkSiteRef>
       </ConnectionsToLocalNetwork>
    </Gateway>

示例：

    <Gateway>
       <ConnectionsToLocalNetwork>
       </ConnectionsToLocalNetwork>
     </Gateway>

### <a name="local-network-sites"></a>本地网络站点

删除不再使用的所有本地站点。 根据你创建的配置，你可能没有列出本地网络站点。

    <LocalNetworkSites>
      <LocalNetworkSite name="Site1">
        <AddressSpace>
          <AddressPrefix>192.168.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>5.4.3.2</VPNGatewayAddress>
      </LocalNetworkSite>
      <LocalNetworkSite name="Site3">
        <AddressSpace>
          <AddressPrefix>192.168.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>57.179.18.164</VPNGatewayAddress>
      </LocalNetworkSite>
    </LocalNetworkSites>

本例中将仅删除 Site3。

    <LocalNetworkSites>
        <LocalNetworkSite name="Site1">
        <AddressSpace>
          <AddressPrefix>192.168.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>5.4.3.2</VPNGatewayAddress>
      </LocalNetworkSite>
    </LocalNetworkSites>

### <a name="client-addresspool"></a>客户地址池

如果你的 P2S 连接到 VNet，你将有一个 VPNClientAddressPool。 删除与所删除的虚拟网络网关对应的客户地址池。

    <Gateway>
       <VPNClientAddressPool>
         <AddressPrefix>10.1.0.0/24</AddressPrefix>
       </VPNClientAddressPool>
       <ConnectionsToLocalNetwork />
    </Gateway>

示例：

     <Gateway>
       <ConnectionsToLocalNetwork />
     </Gateway>

### <a name="gatewaysubnet"></a>GatewaySubnet

删除与 VNet 对应的 GatewaySubnet。

    <Subnets>
       <Subnet name="FrontEnd">
         <AddressPrefix>10.11.0.0/24</AddressPrefix>
       </Subnet>
       <Subnet name="GatewaySubnet">
         <AddressPrefix>10.11.1.0/29</AddressPrefix>
       </Subnet>
     </Subnets>

示例：

    <Subnets>
       <Subnet name="FrontEnd">
         <AddressPrefix>10.11.0.0/24</AddressPrefix>
       </Subnet>
     </Subnets>

## <a name="part-5-upload-the-network-configuration-file"></a>第 5 部分： 上传网络配置文件

保存所做的更改，并将网络配置文件上传到 Azure。 确保根据环境需要更改文件路径。

     Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml

如果成功，则返回显示类似于下例的内容：

     OperationDescription        OperationId                      OperationStatus                                                
     --------------------        -----------                      ---------------                                                
     Set-AzureVNetConfig        e0ee6e66-9167-cfa7-a746-7casb9    Succeeded