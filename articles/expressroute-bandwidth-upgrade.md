<properties 
   pageTitle="动态升级 ExpressRoute 带宽 | Windows Azure"
   description="如何动态增加 ExpressRoute 线路的带宽大小而无需停机。"
   services="expressroute"
   documentationCenter="na"
   authors="cherylmc"
   manager="jdial"
   editor="tysonn" />
<tags 
   ms.service="expressroute"
   ms.date="06/03/2015"
   wacn.date="" />

# 动态升级 ExpressRoute 线路带宽而无需停机

无需任何停机时间即可增加 ExpressRoute 线路的大小。这些说明可帮助你使用 PowerShell 更新 ExpressRoute 线路的带宽。有关限制的详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs)页。

##  配置先决条件

在开始配置之前，请确认你具备以下先决条件：

- Azure 订阅
- 最新版本的 Azure PowerShell
- 已配置且正在运行的活动 ExpressRoute 线路


##  使用 PowerShell 配置设置

Windows PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。有关详细信息，请参阅 [MSDN](https://msdn.microsoft.com/zh-cn/library/windowsazure/jj156055.aspx) 中的 PowerShell 文档。

1. **为 ExpressRoute 导入 PowerShell 模块。**

	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

2. **获取 ExpressRoute 线路的详细信息**

	可以使用以下 PowerShell Commandlet 获取 ExpressRoute 线路的详细信息：
		

    	PS C:\> Get-AzureDedicatedCircuit
	
	此命令将返回你已在订阅中创建的所有线路的列表。如果你有服务密钥，可以使用以下命令获取特定 ExpressRoute 线路的详细信息：

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


3. **在 21vianet 一侧增加 ExpressRoute 线路的带宽**
	
	有关你的提供商支持的带宽选项，请查看 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs)页。你可以选取大于现有线路大小的任何大小。确定所需的大小后，可以使用以下命令调整线路的大小。

		PS C:\> Set-AzureDedicatedCircuitProperties -ServiceKey ********************************* -Bandwidth 1000
		
		Bandwidth                        : 1000
		CircuitName                      : TestCircuit
		Location                         : Silicon Valley
		ServiceKey                       : *********************************
		ServiceProviderName              : equinix
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

	已在 21vianet 一侧估计线路的大小。请注意，我们将从现在开始按照已更新的带宽选项为你计费。

4. **在服务提供商一侧增加 ExpressRoute 线路的带宽**

	请联系你的连接提供商 (NSP/EXP) 并为他们提供有关已更新的带宽的信息。按照你的服务提供商规定的订单更新过程完成任务。

 

<!---HONumber=69-->