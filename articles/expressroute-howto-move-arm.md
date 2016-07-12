<properties
   pageTitle="将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型 | Azure"
   description="本页面介绍如何将经典线路转移到 Resource Manager 部署模型。"
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>
<tags
   ms.service="expressroute"
   ms.date="05/05/2016"
   wacn.date="06/06/2016"/>


# 将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型

## 配置先决条件

- 需要最新版本的 Azure PowerShell 模块（至少 1.0 版）。
- 在开始配置之前，请务必查看[先决条件](/documentation/articles/expressroute-prerequisites/)、[路由要求](/documentation/articles/expressroute-routing/)和[工作流](/documentation/articles/expressroute-workflows/)。
- 在继续下一步之前，请先参阅[将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型](/documentation/articles/expressroute-move/)中提供的信息。确保你已完全了解可能的限制和局限性。
- 如果你要将 Azure ExpressRoute 线路从经典部署模型转移到 Azure Resource Manager 部署模型，必须完全配置好该线路，使它能够在经典部署模型中正常工作。
- 确保拥有一个在 Resource Manager 部署模型中创建的资源组。

## 将 ExpressRoute 线路转移到 Resource Manager 部署模型

只有将 ExpressRoute 线路转移到 Resource Manager 部署模型，才可以在经典部署模型和 Resource Manager 部署模型中使用它。可以运行以下 PowerShell 命令来实现此目的。

### 步骤 1：从经典部署模型收集线路详细信息

首先需要收集有关 ExpressRoute 线路的信息。

登录到 Azure 经典环境并收集服务密钥。可以使用以下 PowerShell 代码段来收集信息：

	# Sign in to your Azure account
	Add-AzureAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)

	# Select the appropriate Azure subscription
	Select-AzureSubscription "<Enter Subscription Name here>"

	# Import the PowerShell modules for Azure and ExpressRoute
	Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

	# Get the service keys of all your ExpressRoute circuits
	Get-AzureDedicatedCircuit

复制你要转移到 Resource Manager 部署模型的线路的**服务密钥**。

### 步骤 2：登录到 Resource Manager 环境并创建新的资源组

可以使用以下代码段创建新的资源组：

	# Sign in to your Azure Resource Manager environment
	Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)

	# Select the appropriate Azure subscription
	Get-AzureRmSubscription -SubscriptionName "<Enter Subscription Name here>" | Select-AzureRmSubscription

	#Create a new resource group if you don't already have one
	New-AzureRmResourceGroup -Name "DemoRG" -Location "China East"

你也可以使用现有的资源组（如果有）。

### 步骤 3：将 ExpressRoute 线路转移到 Resource Manager 部署模型

现在，你可以将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型。在继续下一步之前，请先参阅[将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型](/documentation/articles/expressroute-move/)中提供的信息。

可以运行以下代码段来实现此目的：

	Move-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "DemoRG" -Location "China East" -ServiceKey "<Service-key>"

>[AZURE.NOTE] 转移完成之后，列在前一个 cmdlet 中的新名称用于处理资源。线路实质上已重命名。

## 为两种部署模型启用 ExpressRoute 线路

必须先将 ExpressRoute 线路转移到 Resource Manager 部署模型，然后才可以控制对部署模型的访问。

运行以下 cmdlet 来启用对两种部署模型的访问：

    # Get details of the ExpressRoute circuit
    $ckt = Get-AzureRmExpressRouteCircuit -Name "DemoCkt" -ResourceGroupName "DemoRG"

    #Set "Allow Classic Operations" to TRUE
    $ckt.AllowClassicOperations = $true

    # Update circuit
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

成功完成此操作后，可以在经典部署模型中查看线路。

运行以下命令获取 ExpressRoute 线路的详细信息：

    Get-AzureDedicatedCircuit

服务密钥必须已列出。现在，你可以通过适用于经典 VNet 的标准经典部署模型命令以及适用于 ARM VNET 的标准 ARM 命令来管理到 ExpressRoute 线路的链接。以下文章将引导你了解如何管理 ExpressRoute 线路的链接：

- [在 Resource Manager 部署模型中将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm/)
- [在经典部署模型中将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic/)


## 在经典部署模型中禁用 ExpressRoute 线路

运行以下 cmdlet 可禁止访问经典部署模型：

    # Get details of the ExpressRoute circuit
    $ckt = Get-AzureRmExpressRouteCircuit -Name "DemoCkt" -ResourceGroupName "DemoRG"

    #Set "Allow Classic Operations" to FALSE
    $ckt.AllowClassicOperations = $false

    # Update circuit
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

成功完成此操作后，你将无法在经典部署模型中查看线路。

## 后续步骤

创建你的线路后，请确保执行以下操作：

- [创建和修改 ExpressRoute 线路的路由](/documentation/articles/expressroute-howto-routing-arm/)
- [将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm/)

<!---HONumber=Mooncake_0509_2016-->