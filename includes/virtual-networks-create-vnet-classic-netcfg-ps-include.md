## 如何通过 PowerShell 使用网络配置文件创建 VNet

Azure 使用 xml 文件定义可用于订阅的所有 VNet。可以下载此文件，然后编辑它以修改或删除现有 VNet 并创建新 VNet。在本文档中，你将了解如何下载此文件（称为网络配置（或 netcgf）文件），并编辑它以创建新 VNet。查看 [Azure 虚拟网络配置架构](https://msdn.microsoft.com/zh-cn/library/azure/jj157100.aspx)以了解有关网络配置文件的详细信息。

若要通过 PowerShell 使用 netcfg 文件创建 VNet，请执行下面的步骤。

1. 如果你从未使用过 Azure PowerShell，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)，并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。
2. 从 Azure PowerShell 控制台中，通过运行以下命令使用 **Get-AzureVnetConfig** cmdlet 下载网络配置文件。 

		Get-AzureVNetConfig -ExportToFile c:\NetworkConfig.xml

	预期输出：

		XMLConfiguration                                                                                                     
		----------------                                                                                                     
		<?xml version="1.0" encoding="utf-8"?>...  

3. 使用任何 XML 或文本编辑器应用程序打开在前面步骤 2 中保存的文件，并查找 **<VirtualNetworkSites>** 元素。如果你已创建任何网络，每个网络将显示为其自身的 **<VirtualNetworkSite>** 元素。
4. 若要创建此方案中所述的虚拟网络，请在 **<VirtualNetworkSites>** 元素的正下方添加以下 XML：

		<VirtualNetworkSite name="TestVNet" Location="China North">
		  <AddressSpace>
		    <AddressPrefix>192.168.0.0/16</AddressPrefix>
		  </AddressSpace>
		  <Subnets>
		    <Subnet name="FrontEnd">
		      <AddressPrefix>192.168.1.0/24</AddressPrefix>
		    </Subnet>
		    <Subnet name="BackEnd">
		      <AddressPrefix>192.168.2.0/24</AddressPrefix>
		    </Subnet>
		  </Subnets>
		</VirtualNetworkSite>

9.  保存网络配置文件。
10. 从 Azure PowerShell 控制台中，通过运行以下命令使用 **Set-AzureVnetConfig** cmdlet 上载网络配置文件。请注意该命令下的输出，应在 **OperationStatus** 下看到 **Succeeded**。如果不是这样，请检查 xml 文件是否有错误。

		Set-AzureVNetConfig -ConfigurationPath c:\NetworkConfig.xml

	下面是上述命令的预期输出：

		OperationDescription OperationId                          OperationStatus
		-------------------- -----------                          ---------------
		Set-AzureVNetConfig  49579cb9-3f49-07c3-ada2-7abd0e28c4e4 Succeeded 
	
11. 从 Azure PowerShell 控制台中，通过运行以下命令使用 **Get-AzureVnetSite** cmdlet 验证是否已添加新网络。

		Get-AzureVNetSite -VNetName TestVNet

	下面是上述命令的预期输出：

		AddressSpacePrefixes : {192.168.0.0/16}
		Location             : China North
		AffinityGroup        : 
		DnsServers           : {}
		GatewayProfile       : 
		GatewaySites         : 
		Id                   : b953f47b-fad9-4075-8cfe-73ff9c98278f
		InUse                : False
		Label                : 
		Name                 : TestVNet
		State                : Created
		Subnets              : {FrontEnd, BackEnd}
		OperationDescription : Get-AzureVNetSite
		OperationId          : 3f35d533-1f38-09c0-b286-3d07cd0904d8
		OperationStatus      : Succeeded

<!---HONumber=76-->