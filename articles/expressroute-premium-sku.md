<properties 
   pageTitle="如何启用或禁用 ExpressRoute 高级版外接程序 |Windows Azure"
   description="如何为 ExpressRoute 线路启用或禁用 ExpressRoute 高级版外接程序。ExpressRoute 高级版允许你为公共和私有对等互连添加最多 10,000 个路由，并最多向 ExpressRoute 线路添加 10 个虚拟网络。你还可以将一个区域中的虚拟网络链接到另一个区域中的 ExpressRoute 线路。"
   services="expressroute"
   documentationCenter="na"
   authors="cherylmc"
   manager="jdial"
   editor="tysonn" />
<tags 
   ms.service="expressroute"
   ms.date="06/02/2015"
   wacn.date="" />

# 为 ExpressRoute 线路配置 ExpressRoute 高级版外接程序

ExpressRoute 高级版是下面列出的功能的集合。

 - 对于公共对等互连和专用对等互连，将路由表限制从 4000 个路由提升为 10,000 个路由。
 - 增加了可连接到 ExpressRoute 线路的虚拟网络 (VNet) 数量（默认值为 10）。 
 - 通过 21vnet 核心网络建立全局连接。现在，你可以将一个地缘政治区域中 VNet 链接到另一个区域中的 ExpressRoute 线路。**示例：**可以将欧洲西部创建的 VNet 链接到硅谷创建的 ExpressRoute 线路。

有关 ExpressRoute 高级版外接程序的详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs)页。有关费用，请参阅[定价详细信息](/home/features/expressroute/#price)页。

下面的这些说明将帮助你完成以下操作：

- 创建启用了高级版外接程序的 ExpressRoute 线路。
- 更新现有的 ExpressRoute 线路以启用高级版外接程序。
- 为线路禁用 ExpressRoute 高级版外接程序。


## 创建启用了高级版外接程序功能的 ExpressRoute 线路

###  开始之前

在开始配置之前，请确认你具备以下先决条件：

- Azure 订阅
- 最新版本的 Azure PowerShell

###  1\.为 ExpressRoute 导入 PowerShell 模块

Windows PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。有关详细信息，请参阅 [MSDN](https://msdn.microsoft.com/zh-cn/library/azure/jj156055.aspx) 中的 PowerShell 文档。

使用下面的 cmdlet 为 ExpressRoute 导入 PowerShell 模块：


	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'


### 2\.配置新的 ExpressRoute 线路并为其启用高级版外接程序功能

可以创建新的 ExpressRoute 线路并在创建时为其启用高级版外接程序。按照有关如何通过 [NSP](/documentation/articlesexpressroute-configuring-nsps) 或 [EXP](/documentation/articlesexpressroute-configuring-exps) 创建 ExpressRoute 线路的说明进行操作。我们在 New-AzureDedicatedCircuit cmdlet 中添加了一个新的可选参数，让你指定 SKU。SKU 可以是 Standard 或 Premium。默认值为“standard”。在 SKU 上传递为 Premium 将为线路启用高级版外接程序功能。


		New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location -Sku Premium


### 3\.验证 ExpressRoute 高级版外接程序是否已启用
你可以检查是否为你的线路启用了 ExpressRoute 高级版外接程序。在下面的示例中，ExpressRoute 线路未启用 ExpressRoute 高级版外接程序功能。如果启用了该外接程序，SKU 将显示为 ***Premium***。

		PS C:\> Get-AzureDedicatedCircuit -ServiceKey *********************************

		Bandwidth                        : 200
		CircuitName                      : TestCircuit
		Location                         : Silicon Valley
		ServiceKey                       : *********************************
		ServiceProviderName              : equinix
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled




## 为现有 ExpressRoute 线路启用 ExpressRoute 高级版外接程序
你可以为任何已创建的 ExpressRoute 线路启用 ExpressRoute 高级外接程序功能。


1. **获取 ExpressRoute 线路的详细信息**

	可以使用以下 PowerShell cmdlet 获取 ExpressRoute 线路的详细信息：
		

    	PS C:\> Get-AzureDedicatedCircuit
	
	此命令将返回你已在订阅中创建的所有线路的列表。如果你有服务密钥，可以使用以下命令获取特定 ExpressRoute 线路的详细信息

		 PS C:\> Get-AzureDedicatedCircuit -ServiceKey <skey>

	将 <skey> 替换为实际的服务密钥。
	
		PS C:\> Get-AzureDedicatedCircuit -ServiceKey *********************************

		Bandwidth                        : 200
		CircuitName                      : TestCircuit
		Location                         : Silicon Valley
		ServiceKey                       : *********************************
		ServiceProviderName              : equinix
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled


2. **为线路启用 ExpressRoute 高级版外接程序**


	可以使用以下 PowerShell cmdlet 为现有线路启用 ExpressRoute 高级版外接程序：
	
		PS C:\> Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Premium
		
		Bandwidth                        : 1000
		CircuitName                      : TestCircuit
		Location                         : Silicon Valley
		ServiceKey                       : *********************************
		ServiceProviderName              : equinix
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Premium
		Status                           : Enabled

	你的线路现已启用 ExpressRoute 高级版外接程序功能。请注意，该命令成功运行后，我们将立即开始为你对高级版外接程序功能计费。


## 为 ExpressRoute 线路禁用 ExpressRoute 高级版外接程序

可以为已启用高级版外接程序的 ExpressRoute 线路禁用 ExpressRoute 高级版外接程序。

**注意**：在这样做之前，必须确保已停止使用在高级版外接程序功能列表中列出的功能。如果你有超过 10 个 VNet 链接到你的线路，我们将使你的禁用高级版外接程序的请求失败。你还将看到，如果在你禁用高级版外接程序后，你向我们播发超过 5000 个路由，我们将终止 BGP 会话。

1. **获取 ExpressRoute 线路的详细信息**

	可以使用以下 PowerShell cmdlet 获取 ExpressRoute 线路的详细信息：
		

    	PS C:\> Get-AzureDedicatedCircuit
	
	此命令将返回你已在订阅中创建的所有线路的列表。如果你有服务密钥，可以使用以下命令获取特定 ExpressRoute 线路的详细信息

		 PS C:\> Get-AzureDedicatedCircuit -ServiceKey <skey>

	将 <skey> 替换为实际的服务密钥。
	
		PS C:\> Get-AzureDedicatedCircuit -ServiceKey *********************************

		Bandwidth                        : 200
		CircuitName                      : TestCircuit
		Location                         : Silicon Valley
		ServiceKey                       : *********************************
		ServiceProviderName              : equinix
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Premium
		Status                           : Enabled


3. **为线路禁用 ExpressRoute 高级版外接程序**


	可以使用以下 PowerShell cmdlet 为现有线路禁用 ExpressRoute 高级版外接程序：
	
		PS C:\> Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Standard
		
		Bandwidth                        : 1000
		CircuitName                      : TestCircuit
		Location                         : Silicon Valley
		ServiceKey                       : *********************************
		ServiceProviderName              : equinix
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Premium
		Status                           : Standard

	你的线路现已禁用 ExpressRoute 高级版外接程序功能。


 

<!---HONumber=69-->