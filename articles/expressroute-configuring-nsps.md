<properties 
   pageTitle="通过 NSP 配置 Expressroute"
   description="本教程将指导你通过 NSP 设置 ExpressRoute。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="adinah"
   editor="tysonn"/>

<tags 
   ms.service="expressroute"
   ms.date="06/29/2015"
   wacn.date=""/>

#  通过网络服务提供商配置 ExpressRoute 连接

要通过网络服务提供商配置 ExpressRoute 连接，你需要按正确的顺序完成多个步骤。这些说明将帮助你完成以下操作：

- 创建和管理 ExpressRoute 线路
- 将虚拟网络连接到 ExpressRoute 线路

##  配置先决条件


在开始配置之前，请确认你满足以下先决条件：

- Azure 订阅
- 最新版本的 Windows PowerShell
- 以下虚拟网络要求：
	- 一组要在 Azure 的虚拟网络中使用的 IP 地址前缀。
	- 一组本地 IP 前缀（可以包含公共 IP 地址）。
	- 虚拟网络网关必须使用 /28 子网创建。
	- 虚拟网络外部的一组附加 IP 前缀 (/28)。这将用于配置路由。你将需要向网络服务提供商提供此项。
	- 你的网络的 AS 编号。你将需要向网络服务提供商提供此项。你可以使用私有 AS 编号。如果你选择这样做，则该编号必须大于 65000。有关 AS 编号的详细信息，请参阅“自治系统 (AS) 编号”。 

- 对于网络服务提供商：
	- 你必须使用 ExpressRoute 支持的 VPN 服务。请向网络服务提供商咨询现有的 VPN 服务是否符合条件。

##  部署工作流

![网络服务提供商工作流](./media/expressroute-configuring-nsps/expressroute-nsp-connectivity-workflow.png)

##  使用 PowerShell 配置设置
Windows PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。有关详细信息，请参阅 [MSDN](https://msdn.microsoft.com/zh-cn/library/windowsazure/jj156055.aspx) 中的 PowerShell 文档。



1. **为 ExpressRoute 导入 PowerShell 模块。**

		Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
		Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1' 

2. **获取支持的提供商、位置和带宽的列表。**

	在创建线路之前，你需要服务提供商、支持的位置和每个位置的带宽选项的列表。以下 PowerShell cmdlet 将返回此信息，你将在后面的步骤中使用该信息。

		PS C:\> Get-AzureDedicatedCircuitServiceProvider

	返回的信息将如下例所示：

		PS C:\> Get-AzureDedicatedCircuitServiceProvider
	
		Name                 DedicatedCircuitLocations      DedicatedCircuitBandwidths                                                                                                                                                                                   
		----                 -------------------------      --------------------------                                                                                                                                                                                   
		AT&T                 Silicon Valley,Washington DC   10Mbps:10, 50Mbps:50, 100Mbps:100, 500Mbps:500, 1Gbps:1000                                                                                                                                                   
		British Telecom      London,Amsterdam               10Mbps:10, 50Mbps:50, 100Mbps:100, 500Mbps:500, 1Gbps:1000                                                                                                                                                   
		Equinix              Amsterdam,Atlanta,Chicago,Dall 200Mbps:200, 500Mbps:500, 1Gbps:1000, 10Gbps:10000                                                                                                                                                           
		                     as,New York,Seattle,Silicon                                                                                                                                                                                                                 
		                     Valley,Washington                                                                                                                                                                                                                           
		                     DC,London,Hong                                                                                                                                                                                                                              
		                     Kong,Singapore,Sydney,Tokyo                                                                                                                                                                                                                 
		IIJ                  Tokyo                          10Mbps:10, 50Mbps:50, 100Mbps:100, 500Mbps:500, 1Gbps:1000                                                                                                                                                   
		Level 3              London,Silicon                 200Mbps:200, 500Mbps:500, 1Gbps:1000                                                                                                                                                                         
		Communications -     Valley,Washington DC                                                                                                                                                                                                                        
		Exchange                                                                                                                                                                                                                                                         
		Level 3              London,Silicon                 10Mbps:10, 50Mbps:50, 100Mbps:100, 500Mbps:500, 1Gbps:1000                                                                                                                                                   
		Communications -     Valley,Washington DC                                                                                                                                                                                                                        
		IPVPN                                                                                                                                                                                                                                                            
		Orange               Amsterdam,London               10Mbps:10, 50Mbps:50, 100Mbps:100, 500Mbps:500, 1Gbps:1000                                                                                                                                                   
		SingTel Domestic     Singapore                      10Mbps:10, 50Mbps:50, 100Mbps:100, 500Mbps:500, 1Gbps:1000                                                                                                                                                   
		SingTel              Singapore                      10Mbps:10, 50Mbps:50, 100Mbps:100, 500Mbps:500, 1Gbps:1000                                                                                                                                                   
		International                                                                                                                                                                                                                                                    
		TeleCity Group       Amsterdam,London               200Mbps:200, 500Mbps:500, 1Gbps:1000, 10Gbps:10000                                                                                                                                                           
		Telstra Corporation  Sydney                         10Mbps:10, 50Mbps:50, 100Mbps:100, 500Mbps:500, 1Gbps:1000                                                                                                                                                   
		Verizon              Silicon Valley,Washington DC   10Mbps:10, 50Mbps:50, 100Mbps:100, 500Mbps:500, 1Gbps:1000
		

3. **发出对服务密钥的请求，并将该请求传递给你的网络服务提供商。**

	你将使用 PowerShell cmdlet 发出此请求。对于此示例，我们将使用 AT&T Netbond 作为服务提供商，并将在硅谷指定一条 50 Mbps ExpressRoute 线路。如果你使用的是其他提供商和其他设置，请在发出请求时替换该信息。

	下面是请求新的服务密钥的示例：

		##Creating a new circuit
		$Bandwidth = 50
		$CircuitName = "NetBondSVTest"
		$ServiceProvider = "at&t"
		$Location = "Silicon Valley"
		
		New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location
		
		#Getting service key
		Get-AzureDedicatedCircuit

	响应将如下例所示：

		Bandwidth                        : 50
		CircuitName                      : NetBondSVTest
		Location                         : Silicon Valley
		ServiceKey                       : *********************************
		ServiceProviderName              : at&t
		ServiceProviderProvisioningState : NotProvisioned
		Status                           : Enabled

	你可以随时使用 Get-AzureCircuit cmdlet 检索此信息。进行不带任何参数的调用将列出所有线路。你的服务密钥将在 ServiceKey 字段中列出。

		PS C:\> Get-AzureDedicatedCircuit
		
		Bandwidth                        : 500
		CircuitName                      : NetBondSVTest
		Location                         : Silicon Valley
		ServiceKey                       : 00-0000-0000-0000-0000000000
		ServiceProviderName              : at&t
		ServiceProviderProvisioningState : NotProvisioned
		Status                           : Enabled

	完成此步骤后，你将建立与 Azure 存储空间和其他服务的连接。



4. **配置虚拟网络和网关。**

	请参阅[为 ExpressRoute 配置虚拟网络和网关](/documentation/articles/expressroute-configuring-vnet-gateway)。请注意，网关子网必须是 /28 才能用于 ExpressRoute 连接。

5. **将你的网络连接到线路。**

	只有在你确认线路已转为以下状态后，才继续执行下面的步骤
 
	- ServiceProviderState：已设置
	- 状态：已启用

	请确认你至少拥有一个已创建网关的 Azure 虚拟网络。网关必须处于运行状态。

		PS C:\> $Vnet = "MyTestVNet"
		New-AzureDedicatedCircuitLink -ServiceKey $ServiceKey -VNetName $Vnet
		
		Provisioned 

<!---HONumber=69-->