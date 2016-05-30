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
	wacn.date=""/>

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

第一和第三个步骤完全可以通过编写脚本来完成，但第二个步骤需要通过门户执行一次性的手动操作，或访问以在虚拟网络 Azure Resource Manager 终结点上执行 **PUT** 或 **PATCH** 操作。请联系 Azure 支持人员以启用此功能。开始之前，请确保经典虚拟网络已启用点到站点连接并已部署网关。若要创建网关并启用点到站点连接，需要根据 [Creating a VPN gateway（创建 VPN 网关）][createvpngateway]中所述使用门户。

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

如果你遵循了这些步骤，或者已使用门户来与相同的虚拟网络集成，则证书已设置。

第一个步骤是生成 .cer 文件。第二个步骤是将 .cer 文件上载到虚拟网络。若要从前一步骤中的 API 调用生成 .cer 文件，请运行以下命令。

	$certBytes = [System.Convert]::FromBase64String($vnet.Properties.certBlob)
	[System.IO.File]::WriteAllBytes("$($Configuration.GeneratedCertificatePath)", $certBytes)

可在 **$Configuration.GeneratedCertificatePath** 指定的位置找到证书。

若要手动上载证书，请在 Azure 管理门户中，单击“网络”>“你的 Vnet”>“证书”>“上载”。

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

<!--Links-->
[createvpngateway]: /documentation/articles/vpn-gateway-point-to-site-create/
[azureportal]: http://portal.azure.cn

<!---HONumber=Mooncake_0523_2016-->