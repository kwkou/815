<properties
	pageTitle="使用 PowerShell 将应用程序连接到虚拟网络"
	description="有关如何使用 PowerShell 来连接和操作虚拟网络的说明"
	services="app-service"
	documentationCenter=""
	authors="ccompy"
	manager="wpickett"
	editor="cephalin"/>

<tags
	ms.service="app-service"
	ms.date="04/07/2016"
	wacn.date="05/30/2016"/>

# 使用 PowerShell 将应用程序连接到虚拟网络 #

## 概述 ##

在 Azure Web 应用中，可以将应用连接到订阅中的虚拟网络 (VNet)。此功能称为 VNet 集成。

本文介绍如何使用 PowerShell 来启用集成。

继续阅读本文之前，请确保满足以下条件：

- 安装最新的 Azure PowerShell SDK。可以使用 Web 平台安装程序来安装。
- 在标准或高级 SKU 中运行的 Azure 中的应用。

## 经典虚拟网络 ##

本部分针对使用经典部署模型的虚拟网络说明三项任务：

1. 将应用连接到包含网关且已针对点到站点连接进行配置的现有虚拟网络。
1. 更新应用的虚拟网络集成信息。
1. 从虚拟网络断开连接应用。

### 将应用连接到经典 VNet ###

若要将应用连接到虚拟网络，请遵循以下三个步骤：

1. 向 Web 应用声明它将加入特定的虚拟网络。应用将生成证书，该证书将提供给虚拟网络以建立点到站点连接。
1. 将 Web 应用证书上载到虚拟网络，然后检索点到站点 VPN 包 URI。
1. 使用点到站点包 URI 更新 Web 应用的虚拟网络连接。

第一和第三个步骤完全可以通过编写脚本来完成，但第二个步骤需要通过经典管理门户执行一次性的手动操作，或访问以在虚拟网络 Azure Resource Manager 终结点上执行 **PUT** 或 **PATCH** 操作。请联系 Azure 支持人员以启用此功能。开始之前，请确保经典虚拟网络已启用点到站点连接并已部署网关。若要创建网关并启用点到站点连接，需要根据 [Creating a VPN gateway（创建 VPN 网关）][createvpngateway]中所述使用经典管理门户。

经典虚拟网络需要与 App Service 计划位于相同的订阅中，其中保存了你要集成的应用。

##### 设置 Azure PowerShell SDK #####

打开 PowerShell 窗口，然后使用以下命令设置 Azure 帐户和订阅：

	Login-AzureRmAccount -EnvironmentName AzureChinaCloud

该命令将打开提示符以获取你的 Azure 凭据。登录后，使用以下命令来选择要用的订阅。请确保使用包含你的虚拟网络和 App Service 计划的订阅。

	Select-AzureRmSubscription -SubscriptionName [WebAppSubscriptionName]

或

	Select-AzureRmSubscription -SubscriptionId [WebAppSubscriptionId]

##### 本文中使用的变量 #####

为了简化命令，我们以特定配置设置 **$Configuration** PowerShell 变量。

在 PowerShell 中使用以下参数设置变量，如下所示：

	$Configuration = @{}
	$Configuration.WebAppResourceGroup = "[Your web app resource group]"
	$Configuration.WebAppName = "[Your web app name]"
	$Configuration.VnetSubscriptionId = "[Your vnet subscription id]"
	$Configuration.VnetResourceGroup = "[Your vnet resource group]"
	$Configuration.VnetName = "[Your vnet name]"

应用位置应该是不包含任何空格的位置。例如，中国北部输入为 chinanorth

	$Configuration.WebAppLocation = "[Your web app Location]"

下一个项是证书应写入到的位置。它应该是本地计算机上的可写路径。请务必在末尾包含 .cer。

	$Configuration.GeneratedCertificatePath = "[C:\Path\To\Certificate.cer]"

若要查看设置，请键入 **$Configuration**。

	> $Configuration

	Name                           Value
	----                           -----
	GeneratedCertificatePath       C:\vnetcert.cer
	VnetSubscriptionId             efc239a4-88f9-2c5e-a9a1-3034c21ad496
	WebAppResourceGroup            vnetdemo-rg
	VnetResourceGroup              testase1-rg
	VnetName                       TestNetwork
	WebAppName                     vnetintdemoapp
	WebAppLocation                 centralus

本部分的余下内容假设你已按前面所述创建了变量。

##### 向应用声明虚拟网络 #####

使用以下命令来告诉应用它要使用此特定虚拟网络。这会导致应用生成所需的证书：

	$vnet = New-AzureRmResource -Name "$($Configuration.WebAppName)/$($Configuration.VnetName)" -ResourceGroupName $Configuration.WebAppResourceGroup -ResourceType "Microsoft.Web/sites/virtualNetworkConnections" -PropertyObject @{"VnetResourceId" = "/subscriptions/$($Configuration.VnetSubscriptionId)/resourceGroups/$($Configuration.VnetResourceGroup)/providers/Microsoft.ClassicNetwork/virtualNetworks/$($Configuration.VnetName)"} -Location $Configuration.WebAppLocation -ApiVersion 2015-07-01

如果此命令成功，**$vnet** 中应该包含 **Properties** 变量。**Properties** 变量应该包含证书指纹和证书数据。

##### 将 Web 应用证书上载到虚拟网络 #####

需要针对订阅与虚拟网络的每个组合执行一次性的手动步骤。也就是说，如果将订阅 A 中的应用连接到虚拟网络 A，你只需执行此步骤一次，而不管配置了多少个应用。如果将新的应用添加到另一个虚拟网络，则需要再次执行此步骤。这是因为证书集是在 Azure Web 应用的订阅级别生成的，并且针对应用将要连接到的每个虚拟网络生成该集一次。

如果你遵循了这些步骤，或者已使用经典管理门户来与相同的虚拟网络集成，则证书已设置。

第一个步骤是生成 .cer 文件。第二个步骤是将 .cer 文件上载到虚拟网络。若要从前一步骤中的 API 调用生成 .cer 文件，请运行以下命令。

	$certBytes = [System.Convert]::FromBase64String($vnet.Properties.certBlob)
	[System.IO.File]::WriteAllBytes("$($Configuration.GeneratedCertificatePath)", $certBytes)

可在 **$Configuration.GeneratedCertificatePath** 指定的位置找到证书。

若要手动上载证书，请在 Azure 经典管理门户中，单击“网络”>“你的 Vnet”>“证书”>“上载”。

##### 获取点到站点包 #####

在 Web 应用上设置虚拟网络连接的下一个步骤是获取点到站点包，并将其提供给 Web 应用。

将以下模板保存到计算机上某个位置中的名为 GetNetworkPackageUri.json 的文件，例如：C:\\Azure\\Templates\\GetNetworkPackageUri.json。

	{
		"$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
		"contentVersion": "1.0.0.0",
		"parameters": {
			"certData": {
				"type": "string"
			},
			"certThumbprint": {
				"type": "string"
			},
			"networkName": {
				"type": "string"
			}
		},
		"variables": {
			"legacyVnetName": "[concat('Group ', resourceGroup().name, ' ', parameters('networkName'))]"
			},
			"resources": [
			],
		"outputs" : {
			"PackageUri" :
			{
			"value" : "[listPackage(resourceId('Microsoft.ClassicNetwork/virtualNetworks/gateways/clientRootCertificates', parameters('networkName'), 'primary', parameters('certThumbprint')), '2014-06-01').packageUri]", "type" : "string"
			}
		}
	}


设置输入参数：

	$parameters = @{"certData" = $vnet.Properties.certBlob ;
	certThumbprint = $vnet.Properties.certThumbprint ;
	"networkName" = $Configuration.VnetName }

调用脚本：

	$output = New-AzureRmResourceGroupDeployment -Name unused -ResourceGroupName $Configuration.VnetResourceGroup -TemplateParameterObject $parameters -TemplateFile C:\PATH\TO\GetNetworkPackageUri.json


变量 **$output.Outputs.packageUri** 现在会包含要提供给 Web 应用的包 URI。

##### 将点到站点包上载到应用 #####

最后一个步骤是将此包提供给应用。只需运行下一条命令：

	$vnet = New-AzureRmResource -Name "$($Configuration.WebAppName)/$($Configuration.VnetName)/primary" -ResourceGroupName $Configuration.WebAppResourceGroup -ResourceType "Microsoft.Web/sites/virtualNetworkConnections/gateways" -ApiVersion 2015-07-01 -PropertyObject @{"VnetName" = $Configuration.VnetName ; "VpnPackageUri" = $($output.Outputs.packageUri).Value } -Location $Configuration.WebAppLocation

如果有消息请求你确认是否覆写现有资源，请确保允许覆盖。

此命令成功之后，应用现在应会连接到虚拟网络。若要确认是否成功，请转到应用控制台，然后键入以下命令：

	SET WEBSITE_

如果存在名为 WEBSITE\_VNETNAME 的环境变量，并且其值与目标虚拟网络名称匹配，则表示所有配置都已成功。

### 更新经典 VNet 集成信息 ###

若要更新或重新同步信息，只需重复最初创建集成时所遵循的步骤。这些步骤如下：

1. 定义配置信息。
1. 向应用声明虚拟网络。
1. 获取点到站点包。
1. 将点到站点包上载到应用。

### 从经典 VNet 断开连接应用 ###

若要断开连接应用，需要使用在虚拟网络集成期间设置的配置信息。如果使用该信息，则只需使用一条命令就可以从虚拟网络断开连接应用。

	$vnet = Remove-AzureRmResource -Name "$($Configuration.WebAppName)/$($Configuration.VnetName)" -ResourceGroupName $Configuration.WebAppResourceGroup -ResourceType "Microsoft.Web/sites/virtualNetworkConnections" -ApiVersion 2015-07-01

## Resource Manager 虚拟网络 ##

Resource Manager 虚拟网络具有 Azure Resource Manager API，与经典虚拟网络相比，这些 API 可以简化某些过程。我们提供了脚本来帮助你完成以下任务：

- 创建 Resource Manager 虚拟网络并将你的应用与该虚拟网络集成。
- 创建网关，在现有 Resource Manager 虚拟网络中配置点到站点连接，然后将你的应用与该虚拟网络集成。
- 将应用与已启用网关和点到站点连接的现有 Resource Manager 虚拟网络集成。
- 从虚拟网络断开连接应用。

### Resource Manager VNet Azure Web 应用集成脚本 ###

复制以下脚本并将它保存到文件。如果你不想要使用该脚本，可以自由学习该脚本，以了解如何对 Resource Manager 虚拟网络进行设置。


    function ReadHostWithDefault($message, $default)
    {
    	$result = Read-Host "$message [$default]"
    	if($result -eq "")
	    {
			$result = $default
	    }
		    return $result
    	}

	function PromptCustom($title, $optionValues, $optionDescriptions)
	{
	    Write-Host $title
	    Write-Host
	    $a = @()
	    for($i = 0; $i -lt $optionValues.Length; $i++){
		    Write-Host "$($i+1))" $optionDescriptions[$i]
	    }
	    Write-Host

	    while($true)
	    {
		    Write-Host "Choose an option: "
		    $option = Read-Host
		    $option = $option -as [int]

		    if($option -ge 1 -and $option -le $optionValues.Length)
		    {
			    return $optionValues[$option-1]
		    }
	    }
    }

    function PromptYesNo($title, $message, $default = 0)
    {
	    $yes = New-Object System.Management.Automation.Host.ChoiceDescription "&Yes", ""
	    $no = New-Object System.Management.Automation.Host.ChoiceDescription "&No", ""
	    $options = [System.Management.Automation.Host.ChoiceDescription[]]($yes, $no)
	    $result = $host.ui.PromptForChoice($title, $message, $options, $default)
	    return $result
    }

    function CreateVnet($resourceGroupName, $vnetName, $vnetAddressSpace, $vnetGatewayAddressSpace, $location)
    {
	    Write-Host "Creating a new VNET"
	    $gatewaySubnet = New-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix $vnetGatewayAddressSpace
	    New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix $vnetAddressSpace -Subnet $gatewaySubnet
    }

    function CreateVnetGateway($resourceGroupName, $vnetName, $vnetIpName, $location, $vnetIpConfigName, $vnetGatewayName, $certificateData, $vnetPointToSiteAddressSpace)
    {
	    $vnet = Get-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
	    $subnet=Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet

	    Write-Host "Creating a public IP address for this VNET"
	    $pip = New-AzureRmPublicIpAddress -Name $vnetIpName -ResourceGroupName $resourceGroupName -Location $location -AllocationMethod Dynamic
	    $ipconf = New-AzureRmVirtualNetworkGatewayIpConfig -Name $vnetIpConfigName -Subnet $subnet -PublicIpAddress $pip

	    Write-Host "Adding a root certificate to this VNET"
	    $root = New-AzureRmVpnClientRootCertificate -Name "AppServiceCertificate.cer" -PublicCertData $certificateData

	    Write-Host "Creating Azure VNET Gateway. This may take up to an hour."
	    New-AzureRmVirtualNetworkGateway -Name $vnetGatewayName -ResourceGroupName $resourceGroupName -Location $location -IpConfigurations $ipconf -GatewayType Vpn -VpnType RouteBased -EnableBgp $false -GatewaySku Basic -VpnClientAddressPool $vnetPointToSiteAddressSpace -VpnClientRootCertificates $root
    }

    function AddNewVnet($subscriptionId, $webAppResourceGroup, $webAppName)
    {
	    Write-Host "Adding a new Vnet"
	    Write-Host
	    $vnetName = Read-Host "Specify a name for this Virtual Network"

	    $vnetGatewayName="$($vnetName)-gateway"
	    $vnetIpName="$($vnetName)-ip"
	    $vnetIpConfigName="$($vnetName)-ipconf"

	    # Virtual Network settings
	    $vnetAddressSpace="10.0.0.0/8"
	    $vnetGatewayAddressSpace="10.5.0.0/16"
	    $vnetPointToSiteAddressSpace="172.16.0.0/16"

	    $changeRequested = 0
	    $resourceGroupName = $webAppResourceGroup

	    while($changeRequested -eq 0)
	    {
		    Write-Host
		    Write-Host "Currently, I will create a VNET with the following settings:"
		    Write-Host
		    Write-Host "Virtual Network Name: $vnetName"
		    Write-Host "Resource Group Name:  $resourceGroupName"
		    Write-Host "Gateway Name: $vnetGatewayName"
		    Write-Host "Vnet IP name: $vnetIpName"
		    Write-Host "Vnet IP config name:  $vnetIpConfigName"
		    Write-Host "Address Space:$vnetAddressSpace"
		    Write-Host "Gateway Address Space:$vnetGatewayAddressSpace"
		    Write-Host "Point-To-Site Address Space:  $vnetPointToSiteAddressSpace"
		    Write-Host
		    $changeRequested = PromptYesNo "" "Do you wish to change these settings?" 1

		    if($changeRequested -eq 0)
		    {
			    $vnetName = ReadHostWithDefault "Virtual Network Name" $vnetName
			    $resourceGroupName = ReadHostWithDefault "Resource Group Name" $resourceGroupName
			    $vnetGatewayName = ReadHostWithDefault "Vnet Gateway Name" $vnetGatewayName
			    $vnetIpName = ReadHostWithDefault "Vnet IP name" $vnetIpName
			    $vnetIpConfigName = ReadHostWithDefault "Vnet IP configuration name" $vnetIpConfigName
			    $vnetAddressSpace = ReadHostWithDefault "Vnet Address Space" $vnetAddressSpace
			    $vnetGatewayAddressSpace = ReadHostWithDefault "Vnet Gateway Address Space" $vnetGatewayAddressSpace
			    $vnetPointToSiteAddressSpace = ReadHostWithDefault "Vnet Point-to-site Address Space" $vnetPointToSiteAddressSpace
		    }
	    }

	    $ErrorActionPreference = "Stop";

	    # We create the virtual network and add it here. The way this works is:
	    # 1) Add the VNET association to the App. This allows the App to generate certificates, etc. for the VNET.
	    # 2) Create the VNET and VNET gateway, add the certificates, create the public IP, etc., required for the gateway
	    # 3) Get the VPN package from the gateway and pass it back to the App.

	    $webApp = Get-AzureRmResource -ResourceName $webAppName -ResourceType "Microsoft.Web/sites" -ApiVersion 2015-08-01 -ResourceGroupName $webAppResourceGroup
	    $location = $webApp.Location

	    Write-Host "Creating App association to VNET"
	    $propertiesObject = @{
	     "vnetResourceId" = "/subscriptions/$($subscriptionId)/resourceGroups/$($resourceGroupName)/providers/Microsoft.Network/virtualNetworks/$($vnetName)"
	    }
	    $virtualNetwork = New-AzureRmResource -Location $location -Properties $PropertiesObject -ResourceName "$($webAppName)/$($vnetName)" -ResourceType "Microsoft.Web/sites/virtualNetworkConnections" -ApiVersion 2015-08-01 -ResourceGroupName $webAppResourceGroup -Force

	    CreateVnet $resourceGroupName $vnetName $vnetAddressSpace $vnetGatewayAddressSpace $location

	    CreateVnetGateway $resourceGroupName $vnetName $vnetIpName $location $vnetIpConfigName $vnetGatewayName $virtualNetwork.Properties.CertBlob $vnetPointToSiteAddressSpace

	    Write-Host "Retrieving VPN Package and supplying to App"
	    $packageUri = Get-AzureRmVpnClientPackage -ResourceGroupName $resourceGroupName -VirtualNetworkGatewayName $vnetGatewayName -ProcessorArchitecture Amd64

	    # Put the VPN client configuration package onto the App
	    $PropertiesObject = @{
	    "vnetName" = $VirtualNetworkName; "vpnPackageUri" = $packageUri
	    }

	    New-AzureRmResource -Location $location -Properties $PropertiesObject -ResourceName "$($webAppName)/$($vnetName)/primary" -ResourceType "Microsoft.Web/sites/virtualNetworkConnections/gateways" -ApiVersion 2015-08-01 -ResourceGroupName $webAppResourceGroup -Force

	    Write-Host "Finished!"
    }

    function AddExistingVnet($subscriptionId, $resourceGroupName, $webAppName)
    {
		$ErrorActionPreference = "Stop";

		# At this point, the gateway should be able to be joined to an App, but may require some minor tweaking. We will declare to the App now to use this VNET
		Write-Host "Getting App information"
		$webApp = Get-AzureRmResource -ResourceName $webAppName -ResourceType "Microsoft.Web/sites" -ApiVersion 2015-08-01 -ResourceGroupName $resourceGroupName
		$location = $webApp.Location

		$webAppConfig = Get-AzureRmResource -ResourceName "$($webAppName)/web" -ResourceType "Microsoft.Web/sites/config" -ApiVersion 2015-08-01 -ResourceGroupName $resourceGroupName
		$currentVnet = $webAppConfig.Properties.VnetName
		if($currentVnet -ne $null -and $currentVnet -ne "")
		{
			Write-Host "Currently connected to VNET $currentVnet"
		}

		# Display existing vnets
		$vnets = Get-AzureRmVirtualNetwork
		$vnetNames = @()
		foreach($vnet in $vnets)
		{
			$vnetNames += $vnet.Name
	    }

	    Write-Host
	    $vnet = PromptCustom "Select a VNET to integrate with" $vnets $vnetNames

	    # We need to check if this VNET is able to be joined to a App, based on following criteria
		    # If there is no gateway, we can create one.
		    # If there is a gateway:
			    # It must be of type Vpn
			    # It must be of VpnType RouteBased
			    # If it doesn't have the right certificate, we will need to add it.
			    # If it doesn't have a point-to-site range, we will need to add it.

	    $gatewaySubnet = $vnet.Subnets | Where-Object { $_.Name -eq "GatewaySubnet" }

	    if($gatewaySubnet -eq $null -or $gatewaySubnet.IpConfigurations -eq $null -or $gatewaySubnet.IpConfigurations.Count -eq 0)
	    {
			$ErrorActionPreference = "Continue";
		    # There is no gateway. We need to create one.
		    Write-Host "This Virtual Network has no gateway. I will need to create one."

		    $vnetName = $vnet.Name
		    $vnetGatewayName="$($vnetName)-gateway"
		    $vnetIpName="$($vnetName)-ip"
		    $vnetIpConfigName="$($vnetName)-ipconf"

		    # Virtual Network settings
		    $vnetAddressSpace="10.0.0.0/8"
		    $vnetGatewayAddressSpace="10.5.0.0/16"
		    $vnetPointToSiteAddressSpace="172.16.0.0/16"

		    $changeRequested = 0

		    Write-Host "Your VNET is in the address space $($vnet.AddressSpace.AddressPrefixes), with the following Subnets:"
		    foreach($subnet in $vnet.Subnets)
		    {
			    Write-Host "$($subnet.Name): $($subnet.AddressPrefix)"
		    }

		    $vnetGatewayAddressSpace = Read-Host "Please choose a GatewaySubnet address space"

		    while($changeRequested -eq 0)
		    {
			    Write-Host
			    Write-Host "Currently, I will create a VNET gateway with the following settings:"
			    Write-Host
			    Write-Host "Virtual Network Name: $vnetName"
			    Write-Host "Resource Group Name:  $($vnet.ResourceGroupName)"
			    Write-Host "Gateway Name: $vnetGatewayName"
			    Write-Host "Vnet IP name: $vnetIpName"
			    Write-Host "Vnet IP config name:  $vnetIpConfigName"
			    Write-Host "Address Space:$($vnet.AddressSpace.AddressPrefixes)"
			    Write-Host "Gateway Address Space:$vnetGatewayAddressSpace"
			    Write-Host "Point-To-Site Address Space:  $vnetPointToSiteAddressSpace"
			    Write-Host
			    $changeRequested = PromptYesNo "" "Do you wish to change these settings?" 1

			    if($changeRequested -eq 0)
			    {
				    $vnetGatewayName = ReadHostWithDefault "Vnet Gateway Name" $vnetGatewayName
				    $vnetIpName = ReadHostWithDefault "Vnet IP name" $vnetIpName
				    $vnetIpConfigName = ReadHostWithDefault "Vnet IP configuration name" $vnetIpConfigName
				    $vnetGatewayAddressSpace = ReadHostWithDefault "Vnet Gateway Address Space" $vnetGatewayAddressSpace
				    $vnetPointToSiteAddressSpace = ReadHostWithDefault "Vnet Point-to-site Address Space" $vnetPointToSiteAddressSpace
			    }
		    }

		    $ErrorActionPreference = "Stop";

		    Write-Host "Creating App association to VNET"
		    $propertiesObject = @{
		     "vnetResourceId" = "/subscriptions/$($subscriptionId)/resourceGroups/$($vnet.ResourceGroupName)/providers/Microsoft.Network/virtualNetworks/$($vnetName)"
		    }

		    $virtualNetwork = New-AzureRmResource -Location $location -Properties $PropertiesObject -ResourceName "$($webAppName)/$($vnet.Name)" -ResourceType "Microsoft.Web/sites/virtualNetworkConnections" -ApiVersion 2015-08-01 -ResourceGroupName $resourceGroupName -Force

		    # If there is no gateway subnet, we need to create one.
		    if($gatewaySubnet -eq $null)
		    {
			    $gatewaySubnet = New-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix $vnetGatewayAddressSpace
			    $vnet.Subnets.Add($gatewaySubnet);
			    Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
		    }

		    CreateVnetGateway $vnet.ResourceGroupName $vnetName $vnetIpName $location $vnetIpConfigName $vnetGatewayName $virtualNetwork.Properties.CertBlob $vnetPointToSiteAddressSpace

		    $gateway = Get-AzureRmVirtualNetworkGateway -ResourceGroupName $vnet.ResourceGroupName -Name $vnetGatewayName
	    }
	    else
	    {
		    $uriParts = $gatewaySubnet.IpConfigurations[0].Id.Split('/')
		    $gatewayResourceGroup = $uriParts[4]
		    $gatewayName = $uriParts[8]

		    $gateway = Get-AzureRmVirtualNetworkGateway -ResourceGroupName $vnet.ResourceGroupName -Name $gatewayName

		    # validate gateway types, etc.
		    if($gateway.GatewayType -ne "Vpn")
		    {
			    Write-Error "This gateway is not of the Vpn type. It cannot be joined to an App."
			    return
		    }

		    if($gateway.VpnType -ne "RouteBased")
		    {
			    Write-Error "This gateways Vpn type is not RouteBased. It cannot be joined to an App."
			    return
		    }

		    if($gateway.VpnClientConfiguration -eq $null -or $gateway.VpnClientConfiguration.VpnClientAddressPool -eq $null)
		    {
			    Write-Host "This gateway does not have a Point-to-site Address Range. Please specify one in CIDR notation, e.g. 10.0.0.0/8"
			    $pointToSiteAddress = Read-Host "Point-To-Site Address Space"
			    Set-AzureRmVirtualNetworkGatewayVpnClientConfig -VirtualNetworkGateway $gateway.Name -VpnClientAddressPool $pointToSiteAddress
		    }

		    Write-Host "Creating App association to VNET"
		    $propertiesObject = @{
		     "vnetResourceId" = "/subscriptions/$($subscriptionId)/resourceGroups/$($vnet.ResourceGroupName)/providers/Microsoft.Network/virtualNetworks/$($vnetName)"
		    }

		    $virtualNetwork = New-AzureRmResource -Location $location -Properties $PropertiesObject -ResourceName "$($webAppName)/$($vnet.Name)" -ResourceType "Microsoft.Web/sites/virtualNetworkConnections" -ApiVersion 2015-08-01 -ResourceGroupName $resourceGroupName -Force

		    # We need to check if the certificate here exists in the gateway.
		    $certificates = $gateway.VpnClientConfiguration.VpnClientRootCertificates

		    $certFound = $false
		    foreach($certificate in $certificates)
		    {
			    if($certificate.PublicCertData -eq $virtualNetwork.Properties.CertBlob)
			    {
				    $certFound = $true
				    break
			    }
		    }

		    if(-not $certFound)
		    {
			    Write-Host "Adding certificate"
			    Add-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName "AppServiceCertificate.cer" -PublicCertData $virtualNetwork.Properties.CertBlob -VirtualNetworkGatewayName $gateway.Name
		    }
	    }

	    # Now finish joining by getting the VPN package and giving it to the App
	    Write-Host "Retrieving VPN Package and supplying to App"
	    $packageUri = Get-AzureRmVpnClientPackage -ResourceGroupName $vnet.ResourceGroupName -VirtualNetworkGatewayName $gateway.Name -ProcessorArchitecture Amd64

	    # Put the VPN client configuration package onto the App
	    $PropertiesObject = @{
	    "vnetName" = $vnet.Name; "vpnPackageUri" = $packageUri
	    }

	    New-AzureRmResource -Location $location -Properties $PropertiesObject -ResourceName "$($webAppName)/$($vnet.Name)/primary" -ResourceType "Microsoft.Web/sites/virtualNetworkConnections/gateways" -ApiVersion 2015-08-01 -ResourceGroupName $resourceGroupName -Force

	    Write-Host "Finished!"
    }

    function RemoveVnet($subscriptionId, $resourceGroupName, $webAppName)
    {
	    $webAppConfig = Get-AzureRmResource -ResourceName "$($webAppName)/web" -ResourceType "Microsoft.Web/sites/config" -ApiVersion 2015-08-01 -ResourceGroupName $resourceGroupName
	    $currentVnet = $webAppConfig.Properties.VnetName
	    if($currentVnet -ne $null -and $currentVnet -ne "")
	    {
		    Write-Host "Currently connected to VNET $currentVnet"

		    Remove-AzureRmResource -ResourceName "$($webAppName)/$($currentVnet)" -ResourceType "Microsoft.Web/sites/virtualNetworkConnections" -ApiVersion 2015-08-01 -ResourceGroupName $resourceGroupName
	    }
	  	  else
	    {
	  	  Write-Host "Not connected to a VNET."
	    }
    }

    Write-Host "Please Login"
    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

    # Choose subscription. If there's only one we will choose automatically

    $subs = Get-AzureRmSubscription
    $subscriptionId = ""

    if($subs.Length -eq 0)
    {
	    Write-Error "No subscriptions bound to this account."
	    return
    }

    if($subs.Length -eq 1)
    {
	    $subscriptionId = $subs[0].SubscriptionId
    }
    else
    {
	    $subscriptionChoices = @()
	    $subscriptionValues = @()

	    foreach($subscription in $subs){
	    $subscriptionChoices += "$($subscription.SubscriptionName) ($($subscription.SubscriptionId))";
	    $subscriptionValues += ($subscription.SubscriptionId);
	    }

	    $subscriptionId = PromptCustom "Choose a subscription" $subscriptionValues $subscriptionChoices
    }

    Select-AzureRmSubscription -SubscriptionId $subscriptionId

    $resourceGroup = Read-Host "Please enter the Resource Group of your App"

    $appName = Read-Host "Please enter the Name of your App"

    $options = @("Add a NEW Virtual Network to an App", "Add an EXISTING Virtual Network to an App", "Remove a Virtual Network from an App");
    $optionValues = @(0, 1, 2)
    $option = PromptCustom "What do you want to do?" $optionValues $options

    if($option -eq 0)
    {
		AddNewVnet $subscriptionId $resourceGroup $appName
    }
    if($option -eq 1)
    {
	    AddExistingVnet $subscriptionId $resourceGroup $appName
    }
	if($option -eq 2)
    {
	    RemoveVnet $subscriptionId $resourceGroup $appName
    }

保存该脚本的副本。在本文中，其名称为 V2VnetAllinOne.ps1，但你可以使用其他名称。此脚本没有参数。你可以直接运行它。脚本执行所做的第一件事是提示你登录。登录后，脚本将获取有关你帐户的详细信息，并返回订阅列表。如果不考虑凭据请求部分，初始脚本执行的情况如下所示：

	PS C:\Users\ccompy\Documents\VNET> .\V2VnetAllInOne.ps1
	Please Login

	Environment           : AzureChinaCloud
	Account               : xxxxxxx@xxxxxxx.partner.onmschina.cn
	TenantId              : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
	SubscriptionId        : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
	CurrentStorageAccount :

	Choose a subscription

	1) Demo Subscription (xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)
	2) MS Test (xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)
	3) Purple Demo Subscription (xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)

	Choose an option:
	3

	Account      : xxxxxxx@xxxxxxx.partner.onmschina.cn
	Environment  : AzureChinaCloud
	Subscription : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
	Tenant       : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

	Please enter the Resource Group of your App: hcdemo-rg
	Please enter the Name of your App: v2vnetpowershell
	What do you want to do?

	1) Add a NEW Virtual Network to an App
	2) Add an EXISTING Virtual Network to an App
	3) Remove a Virtual Network from an App

本部分的余下内容将说明三个选项中的每一个。

### 创建 Resource Manager VNet 并与它集成 ###

若要创建使用 Resource Manager 部署模型的新虚拟网络并将它与应用集成，请选择“1) 将新的虚拟网络添加到应用”。随后系统将提示你输入虚拟网络的名称。在本例中，你可以看到，我在以下设置中使用了名称 v2pshell。

脚本将提供有关所创建的虚拟网络的详细信息。如果需要，我可以更改任何值。在此示例执行中，我创建了具有以下设置的虚拟网络：

	Virtual Network Name:         v2pshell
	Resource Group Name:          hcdemo-rg
	Gateway Name:                 v2pshell-gateway
	Vnet IP name:                 v2pshell-ip
	Vnet IP config name:          v2pshell-ipconf
	Address Space:                10.0.0.0/8
	Gateway Address Space:        10.5.0.0/16
	Point-To-Site Address Space:  172.16.0.0/12

	Do you wish to change these settings?
	[Y] Yes  [N] No  [?] Help (default is "N"):

如果你想要更改任何值，请键入 **Y**，然后进行更改。如果你对虚拟网络设置感到满意，请键入 **N**，或者在系统提示是否更改设置时按 Enter。从此处开始，脚本将告诉你它正在执行什么操作，直到开始创建虚拟网络网关为止。该步骤最多需要一个小时。在此阶段没有任何进度指示器，但创建好网关后，脚本会告诉你。

脚本完成时，会显示 **Finished**。此时，你已创建了一个具有所选名称和设置的 Resource Manager 虚拟网络。此新虚拟网络也将与你的应用集成。

### 将应用与现有的 Resource Manager VNet 集成 ###

与现有的虚拟网络集成时，如果提供的 Resource Manager 虚拟网络没有网关或点到站点连接，则脚本将进行设置。如果 VNET 上已设置这些项，脚本将直接进入应用集成环节。若要启动此过程，只需选择“2) 将现有的虚拟网络添加到应用”。

仅当现有 Resource Manager 虚拟网络与应用位于同一个订阅中时，此选项才可用。选择该选项后，你将看到 Resource Manager 虚拟网络的列表。

	Select a VNET to integrate with

	1) v2demonetwork
	2) v2pshell
	3) v2vnetintdemo
	4) v2asenetwork
	5) v2pshell2

	Choose an option:
	5

只需选择你要集成的虚拟网络。如果你已有一个启用了点到站点连接的网关，脚本只将应用与虚拟网络集成。如果你没有网关，则需要指定网关子网。网关子网必须位于虚拟网络地址空间，并且不能在任何其他子网中。如果虚拟网络没有网关但你要运行此步骤，则结果如下所示：

	This Virtual Network has no gateway. I will need to create one.
	Your VNET is in the address space 172.16.0.0/16, with the following Subnets:
	default: 172.16.0.0/24
	Please choose a GatewaySubnet address space: 172.16.1.0/26

在此示例中，我创建了具有以下设置的虚拟网络网关：

	Virtual Network Name:         v2pshell2
	Resource Group Name:          vnetdemo-rg
	Gateway Name:                 v2pshell2-gateway
	Vnet IP name:                 v2pshell2-ip
	Vnet IP config name:          v2pshell2-ipconf
	Address Space:                172.16.0.0/16
	Gateway Address Space:        172.16.1.0/26
	Point-To-Site Address Space:  172.16.0.0/12

	Do you wish to change these settings?
	[Y] Yes  [N] No  [?] Help (default is "N"):
	Creating App association to VNET

如果你想要更改上述任何设置，可以这样做。否则请按 Enter 键，然后脚本将创建网关，并将应用附加到虚拟网络。网关创建时间仍为一小时，因此请确保有心理准备。一切完成后，脚本将显示 **Finished**。

### 从 Resource Manager VNet 断开连接应用 ###

从虚拟网络断开连接应用不会关闭网关或禁用点到站点连接。你仍可以将网关或该连接用于其他目的。此外，除了与你提供的应用断开连接以外，不会与任何其他应用断开连接。若要执行此操作，请选择“3) 从应用中删除虚拟网络”。执行此操作时，将显示如下所示的屏幕：

	Currently connected to VNET v2pshell

	Confirm
	Are you sure you want to delete the following resource:
	/subscriptions/edcc99a4-b7f9-4b5e-a9a1-3034c51db496/resourceGroups/hcdemo-rg/providers/Microsoft.Web/sites/v2vnetpowers
	hell/virtualNetworkConnections/v2pshell
	[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):

尽管脚本中有 delete 字样，但实际上不会删除虚拟网络。它只是删除集成。在确认这是你想要执行的操作之后，命令将快速进行处理，并在完成时返回 **True**。

<!--Links-->
[createvpngateway]: /documentation/articles/vpn-gateway-point-to-site-create/
[azureportal]: http://portal.azure.cn

<!---HONumber=Mooncake_0523_2016-->