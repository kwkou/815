<properties
   pageTitle="将 ExpressRoute 线路从经典部署模型移动到 Resource Manager 部署模型：PowerShell | Azure"
   description="本页面介绍如何使用 PowerShell 将经典线路移动到 Resource Manager 部署模型。"
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"/>
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/03/2017"
   wacn.date="03/24/2017"
   ms.author="ganesr;cherylmc"/>  



# 使用 PowerShell 将 ExpressRoute 线路从经典部署模型移动到 Resource Manager 部署模型

若要将 ExpressRoute 线路同时用于经典部署模型和 Resource Manager 部署模型，必须将该线路移动到 Resource Manager 部署模型中。以下部分介绍使用 PowerShell 移动线路的步骤。

## 开始之前
- 确保拥有最新版本的 Azure PowerShell 模块（至少为 1.0 版）。
- 在开始配置之前，请务必查看[先决条件](/documentation/articles/expressroute-prerequisites/)、[路由要求](/documentation/articles/expressroute-routing/)和[工作流](/documentation/articles/expressroute-workflows/)。
- 请参阅[将 ExpressRoute 线路从经典部署模型移动到 Resource Manager 部署模型](/documentation/articles/expressroute-move/)中提供的信息。请确保对限制和局限性有全面的了解。
- 验证线路在经典部署模型中可完全正常运行。
- 确保拥有一个在 Resource Manager 部署模型中创建的资源组。

## 移动 ExpressRoute 线路

### 步骤 1：从经典部署模型收集线路详细信息
登录 Azure 经典环境并收集服务密钥。

1. 登录到你的 Azure 帐户。

    	Add-AzureAccount -Environment AzureChinaCloud

2. 选择相应的 Azure 订阅。

    	Select-AzureSubscription "<Enter Subscription Name here>"

3. 为 Azure 和 ExpressRoute 导入 PowerShell 模块。

	# Import the PowerShell modules for Azure and ExpressRoute
	Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

4. 使用下面的 cmdlet 获取所有 ExpressRoute 线路的服务密钥。检索密钥后，请复制要移动到 Resource Manager 部署模型的线路的**服务密钥**。

	# Get the service keys of all your ExpressRoute circuits
	Get-AzureDedicatedCircuit

### 步骤 2：登录并创建资源组
登录 Resource Manager 环境并创建新的资源组。

1. 登录 Azure Resource Manager 环境。

    	Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)

2. 选择相应的 Azure 订阅。

    	Get-AzureRmSubscription -SubscriptionName "<Enter Subscription Name here>" | Select-AzureRmSubscription

3. 如果还没有资源组，请修改下面的片段，创建新的资源组。

	#Create a new resource group if you don't already have one
	New-AzureRmResourceGroup -Name "DemoRG" -Location "chinaeast"

### 步骤 3：将 ExpressRoute 线路转移到 Resource Manager 部署模型
现在，可以将 ExpressRoute 线路从经典部署模型移动到 Resource Manager 部署模型。在继续下一步之前，请先参阅[将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型](/documentation/articles/expressroute-move/)中提供的信息。

若要移动线路，请修改并运行以下代码片段：

	Move-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "DemoRG" -Location "chinaeast" -ServiceKey "<Service-key>"

>[AZURE.NOTE] 转移完成之后，列在前一个 cmdlet 中的新名称用于处理资源。线路实质上已重命名。


## 修改线路访问

### 为两种部署模型启用 ExpressRoute 线路访问
将经典 ExpressRoute 线路移动到 Resource Manager 部署模型后，可以启用对两种部署模型的访问。运行以下 cmdlet 启用对两种部署模型的访问：

1. 获取线路详细信息。

    	$ckt = Get-AzureRmExpressRouteCircuit -Name "DemoCkt" -ResourceGroupName "DemoRG"

2. 将“允许经典操作”设置为“TRUE”。

    	$ckt.AllowClassicOperations = $true

3. 更新线路。成功完成此操作后，可以在经典部署模型中查看线路。

		Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

4. 运行以下 cmdlet 获取 ExpressRoute 线路的详细信息。服务密钥必须已列出。

    Get-AzureDedicatedCircuit

5. 现在，可以使用经典 VNet 的经典部署模型命令以及 Resource Manager VNet 的 Resource Manager 命令来管理 ExpressRoute 线路的链接。以下文章将引导你了解如何管理 ExpressRoute 线路的链接：

- [Link your virtual network to your ExpressRoute circuit in the Resource Manager deployment model](/documentation/articles/expressroute-howto-linkvnet-arm)（在 Resource Manager 部署模型中将虚拟网络链接到 ExpressRoute 线路）
- [Link your virtual network to your ExpressRoute circuit in the classic deployment model](/documentation/articles/expressroute-howto-linkvnet-classic)（在经典部署模型中将虚拟网络链接到 ExpressRoute 线路）

### 禁用对经典部署模型的 ExpressRoute 线路访问
运行以下 cmdlet 可禁止访问经典部署模型。

1. 获取 ExpressRoute 线路的详细信息。

		$ckt = Get-AzureRmExpressRouteCircuit -Name "DemoCkt" -ResourceGroupName "DemoRG"

2. 将“允许经典操作”设置为“FALSE”。

		$ckt.AllowClassicOperations = $false

3. 更新线路。成功完成此操作后，你将无法在经典部署模型中查看线路。

		Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

## 后续步骤

- [创建和修改 ExpressRoute 线路的路由](/documentation/articles/expressroute-howto-routing-arm/)
- [将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm/)

<!---HONumber=Mooncake_0320_2017-->