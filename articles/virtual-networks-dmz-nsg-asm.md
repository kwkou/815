<properties
   pageTitle="外围网络示例 – 使用 NSG 构建简单的外围网络 | Azure"
   description="使用网络安全组 (NSG) 构建外围网络"
   services="virtual-network"
   documentationCenter="na"
   authors="tracsman"
   manager="rossort"
   editor=""/>

<tags
	ms.service="virtual-network"
	ms.date="02/01/2016"
	wacn.date="03/28/2016"/>

# 示例 1 – 使用 NSG 构建简单的外围网络

本示例将创建一个简单的外围网络，其中包含四个 Windows 服务器和网络安全组。本示例还将演练每个相关命令，让你更加深入地了解每个步骤。另外还提供了“流量方案”部分，让你逐步深入了解流量如何流经外围网络的各个防御层。最后的“参考”部分提供了完整的代码，并说明如何构建此环境来测试和试验各种方案。

![使用 NSG 的入站外围网络][1]

## 环境描述
此示例中，有一个订阅包含以下项：

- 两个云服务：“FrontEnd001”和“BackEnd001”
- 一个虚拟网络“CorpNetwork”，其中包含两个子网：“FrontEnd”和“BackEnd”
- 应用到这两个子网的网络安全组
- 一个代表应用程序 Web 服务器的 Windows Server（“IIS01”）
- 两个代表应用程序后端服务器的 Windows Server（“AppVM01”、“AppVM02”）
- 一个代表 DNS 服务器的 Windows Server（“DNS01”）

下面的“参考”部分提供了可用于构建上述大多数环境的 PowerShell 脚本。尽管 VM 和虚拟网络的构建也可以由本示例脚本来完成，但本文未予详细的描述。

构建环境：

  1.	保存“参考”部分中包含的网络配置 xml 文件（更新名称、位置和 IP 地址以符合给定的方案）
  2.	更新脚本中的用户变量，以符合用于运行脚本的环境（订阅、服务名称等）
  3.	在 PowerShell 中执行脚本

**注意**：PowerShell 脚本中提到的区域必须与网络配置 xml 文件中提到的区域相匹配。

成功运行脚本后，可以执行其他可选步骤。“参考”部分中提供了两个脚本，用于设置 Web 服务器和包含简单 Web 应用的应用服务器，以便能使用此外围网络配置进行测试。

以下部分通过演练 PowerShell 脚本的关键代码行，详细说明本示例的网络安全组及其运行方式。

## 网络安全组 (NSG)
本示例将构建一个 NSG 组，然后加载六个规则。

>[AZURE.TIP]一般而言，应该先创建特定的“允许”规则，最后再创建一般的“拒绝”规则。分配的优先级确定了要先评估哪些规则。发现要向流量应用哪个特定规则后，不需要评估后续规则。可以朝入站或出站方向（从子网的角度看）应用 NSG 规则。

以声明性的方式为入站流量构建以下规则：

1.	允许内部 DNS 流量（端口 53）
2.	允许从 Internet 到任何 VM 的 RDP 流量（端口 3389）
3.	允许从 Internet 到 Web 服务器 (IIS01) 的 HTTP 流量（端口 80）
4.	允许从 IIS01 到 AppVM1 的任何流量（所有端口）
5.	拒绝从 Internet 到整个 VNet（两个子网）的任何流量（所有端口）
6.	拒绝从前端子网到后端子网的任何流量（所有端口）

将这些规则绑定到每个子网后，如果有从 Internet 到 Web 服务器的入站 HTTP 请求，将应用规则 3（允许）和规则 5（拒绝），但由于规则 3 具有较高的优先级，因此只应用规则 3 并忽略规则 5。这样就会允许 HTTP 请求传往 Web 服务器。如果相同的流量尝试传往 DNS01 服务器，则会先应用规则 5（拒绝），因此不允许该流量传递到服务器。规则 6（拒绝）阻止前端子网与后端子网对话（规则 1 和 4 允许的流量除外），这可在攻击者入侵前端上的 Web 应用时保护后端网络，攻击者只能对后端的“受保护”网络进行有限度的访问（只能访问 AppVM01 服务器上公开的资源）。

有一个默认出站规则可允许流量外流到 Internet。在此示例中，我们允许出站流量，且未修改任何出站规则。如果两个方向的流量都要锁定，则需要用户定义的路由，下面的“示例 3”将对此进行介绍。

下面更详细讨论了每个规则（**注意**：以下列表中以货币符号开头的任何项（例如 $NSGName）均为本文“参考”部分的脚本中的用户定义变量）：

1. 首先需要构建一个网络安全组来保存规则：

	    New-AzureNetworkSecurityGroup -Name $NSGName `
            -Location $DeploymentLocation `
            -Label "Security group for $VNetName subnets in $DeploymentLocation"
 
2.	本示例中的第一个规则允许所有内部网络之间的 DNS 流量发往后端子网上的 DNS 服务器。该规则有一些重要参数：
  - “Type”表示此规则的流量作用方向；此方向是从子网或虚拟机的角度判断的（取决于此 NSG 绑定到了哪个位置）。因此，如果 Type 是“Inbound”并且流量进入子网，此规则将适用，而离开子网的流量则不受此规则影响。
  - “Priority”设置流量的评估顺序。编号越低，优先级就越高。只要将某个规则应用于特定的流量，就不再处理其他规则。因此，如果优先级为 1 的规则允许流量，优先级为 2 的规则拒绝流量，并将这两个规则同时应用于流量，则允许流量流动（规则 1 的优先级更高，因此将发生作用，并且不再应用其他规则）。
  - “Action”表示是要阻止还是允许受此规则影响的流量。

			Get-AzureNetworkSecurityGroup -Name $NSGName | `
                Set-AzureNetworkSecurityRule -Name "Enable Internal DNS" `
                -Type Inbound -Priority 100 -Action Allow `
                -SourceAddressPrefix VIRTUAL_NETWORK -SourcePortRange '*' `
                -DestinationAddressPrefix $VMIP[4] `
                -DestinationPortRange '53' `
                -Protocol *

3.	此规则允许 RDP 流量从 Internet 发往 VNET 中任一子网上任何服务器的 RDP 端口。此规则使用两种特殊地址前缀：“VIRTUAL\_NETWORK”和“INTERNET”。这样就可以轻松处理较大类别的地址前缀。

		Get-AzureNetworkSecurityGroup -Name $NSGName | `
	    	Set-AzureNetworkSecurityRule -Name "Enable RDP to $VNetName VNet" `
	    	-Type Inbound -Priority 110 -Action Allow `
	    	-SourceAddressPrefix INTERNET -SourcePortRange '*' `
	    	-DestinationAddressPrefix VIRTUAL_NETWORK `
	    	-DestinationPortRange '3389' `
	    	-Protocol *

4.	此规则允许入站 Internet 流量抵达 Web 服务器。这不会更改路由行为；只允许目标为 IIS01 的流量通过。因此，如果来自 Internet 的流量将 Web 服务器作为其目标，此规则将允许流量，并停止处理其他规则。（在优先级为 140 的规则中，其他所有入站 Internet 流量均被阻止）。如果你只要处理 HTTP 流量，可将此规则进一步限制为只允许目标端口 80。

		Get-AzureNetworkSecurityGroup -Name $NSGName | `
		    Set-AzureNetworkSecurityRule -Name "Enable Internet to $VMName[0]" `
    		-Type Inbound -Priority 120 -Action Allow `
    		-SourceAddressPrefix Internet -SourcePortRange '*' `
    		-DestinationAddressPrefix $VMIP[0] `
    		-DestinationPortRange '*' `
    		-Protocol *

5.	此规则允许流量从 IIS01 服务器传递到 AppVM01 服务器，后面的规则将阻止其他所有从前端到后端的流量。如果要添加的端口是已知的，则可以改善此规则。例如，如果 IIS 服务器只抵达 AppVM01 上的 SQL Server，并且 Web 应用曾遭到入侵，则目标端口范围应该从“*”（任何）更改为 1433（SQL 端口），以缩小 AppVM01 上的入站攻击面。

		Get-AzureNetworkSecurityGroup -Name $NSGName | `
		    Set-AzureNetworkSecurityRule -Name "Enable $VMName[1] to $VMName[2]" `
		    -Type Inbound -Priority 130 -Action Allow `
	    -SourceAddressPrefix $VMIP[1] -SourcePortRange '*' `
	    -DestinationAddressPrefix $VMIP[2] `
	    -DestinationPortRange '*' `
	    -Protocol *
 
6.	此规则将拒绝从 Internet 到网络上任何服务器的流量。结合优先级为 110 和 120 的规则，只允许入站 Internet 流量发往防火墙以及通过 RDP 端口发往其他服务器，除此之外的其他流量将被阻止。

		Get-AzureNetworkSecurityGroup -Name $NSGName | `
		    Set-AzureNetworkSecurityRule `
		    -Name "Isolate the $VNetName VNet from the Internet" `
		    -Type Inbound -Priority 140 -Action Deny `
		    -SourceAddressPrefix INTERNET -SourcePortRange '*' `
		    -DestinationAddressPrefix VIRTUAL_NETWORK `
		    -DestinationPortRange '*' `
		    -Protocol *
 
7.	最后一个规则拒绝从前端子网到后端子网的流量。这是仅限入站的规则，因此允许反向流量（从后端到前端）。

		Get-AzureNetworkSecurityGroup -Name $NSGName | `
    		Set-AzureNetworkSecurityRule `
    		-Name "Isolate the $FESubnet subnet from the $BESubnet subnet" `
    		-Type Inbound -Priority 150 -Action Deny `
    		-SourceAddressPrefix $FEPrefix -SourcePortRange '*' `
    		-DestinationAddressPrefix $BEPrefix `
    		-DestinationPortRange '*' `
    		-Protocol * 

## 流量方案
#### （*允许*）从 Web 访问 Web 服务器
1.	Internet 用户从 FrontEnd001.CloudApp.Net（面向 Internet 的云服务）请求 HTTP 页面
2.	云服务通过端口 80 上的开放终结点将流量传递到 IIS01（Web 服务器）
3.	前端子网开始处理入站规则：
  1.	NSG 规则 1 (DNS) 不适用，将转到下一规则
  2.	NSG 规则 2 (RDP) 不适用，将转到下一规则
  3.	NSG 规则 3（Internet 到 IIS01）适用，允许流量，停止处理规则
4.	流量抵达 Web 服务器 IIS01 的内部 IP 地址（10.0.1.5）
5.	IIS01 正在侦听 Web 流量，将接收此请求并开始处理请求
6.	IIS01 请求 AppVM01 上的 SQL Server 提供信息
7.	前端子网上没有出站规则，允许流量
8.	后端子网开始处理入站规则：
  1.	NSG 规则 1 (DNS) 不适用，将转到下一规则
  2.	NSG 规则 2 (RDP) 不适用，将转到下一规则
  3.	NSG 规则 3（Internet 到防火墙）不适用，将转到下一规则
  4.	NSG 规则 4（IIS01 到 AppVM01）适用，允许流量，停止处理规则
9.	AppVM01 接收 SQL 查询并做出响应
10.	后端子网上没有出站规则，因此允许响应
11.	前端子网开始处理入站规则：
  1.	后端子网到前端子网的入站流量没有适用的 NSG 规则，因此不会应用任何 NSG 规则
  2.	允许子网间流量的默认系统规则允许此流量，因此允许流量。
12.	IIS 服务器接收 SQL 响应，完成 HTTP 响应并发送给请求方
13.	前端子网上没有出站规则，因此允许响应，Internet 用户将收到请求的网页。

#### （*允许*）通过 RDP 访问后端
1.	Internet 上的服务器管理员在 BackEnd001.CloudApp.Net:xxxxx 上请求 AppVM01 的 RDP 会话，其中 xxxxx 是通过 RDP 访问 AppVM01 所用的随机分配端口号（在 Azure 经典管理门户上或通过 PowerShell 可以找到分配的端口）
2.	后端子网开始处理入站规则：
  1.	NSG 规则 1 (DNS) 不适用，将转到下一规则
  2.	NSG 规则 2 (RDP) 适用，允许流量，停止处理规则
3.	由于没有出站规则，将应用默认规则并允许返回流量
4.	已启用 RDP 会话
5.	AppVM01 提示输入用户名和密码

#### （*允许*）在 DNS 服务器上执行 Web 服务器 DNS 查找
1.	Web 服务器 IIS01 需要 www.data.gov 上的数据源，但需要解析地址。
2.	VNet 的网络配置将 DNS01（后端子网上的 10.0.2.4）列为主 DNS 服务器，IIS01 将 DNS 请求发送到 DNS01
3.	前端子网上没有出站规则，允许流量
4.	后端子网开始处理入站规则：
  1.	 NSG 规则 1 (DNS) 适用，允许流量，停止处理规则
5.	DNS 服务器接收请求
6.	DNS 服务器没有缓存的地址，请求 Internet 上的根 DNS 服务器
7.	后端子网上没有出站规则，允许流量
8.	Internet DNS 服务器做出响应，由于此会话是从内部发起的，因此允许响应
9.	DNS 服务器缓存响应，然后将初始请求响应给 IIS01
10.	后端子网上没有出站规则，允许流量
11.	前端子网开始处理入站规则：
  1.	后端子网到前端子网的入站流量没有适用的 NSG 规则，因此不会应用任何 NSG 规则
  2.	允许子网间流量的默认系统规则允许此流量，因此允许流量
12.	IIS01 从 DNS01 接收响应

#### （*允许*）Web 服务器访问 AppVM01 上的文件
1.	IIS01 请求 AppVM01 上的文件
2.	前端子网上没有出站规则，允许流量
3.	后端子网开始处理入站规则：
  1.	NSG 规则 1 (DNS) 不适用，将转到下一规则
  2.	NSG 规则 2 (RDP) 不适用，将转到下一规则
  3.	NSG 规则 3（Internet 到 IIS01）不适用，将转到下一规则
  4.	NSG 规则 4（IIS01 到 AppVM01）适用，允许流量，停止处理规则
4.	AppVM01 接收请求并以文件做出响应（假设已获得访问授权）
5.	后端子网上没有出站规则，因此允许响应
6.	前端子网开始处理入站规则：
  1.	后端子网到前端子网的入站流量没有适用的 NSG 规则，因此不会应用任何 NSG 规则
  2.	允许子网间流量的默认系统规则允许此流量，因此允许流量。
7.	IIS 服务器接收文件

#### （*拒绝*）通过 Web 访问后端服务器
1.	Internet 用户尝试通过 BackEnd001.CloudApp.Net 服务访问 AppVM01 上的文件
2.	由于没有为文件共享开放终结点，此流量不会通过云服务抵达服务器
3.	如果出于某种原因而开放了终结点，NSG 规则 5（Internet 到 VNet）将阻止此流量

#### （*拒绝*）在 DNS 服务器上执行 Web DNS 查找
1.	Internet 用户尝试通过 BackEnd001.CloudApp.Net 服务查找 DNS01 上的内部 DNS 记录
2.	由于没有为 DNS 开放终结点，此流量不会通过云服务抵达服务器
3.	如果出于某种原因而开放了终结点，NSG 规则 5（Internet 到 VNet）将阻止此流量（注意：有两个原因导致规则 1 (DNS) 不适用：首先，源地址是 Internet，此规则只适用于以本地 VNet 作为源；其次，这是允许规则，因此永远不会拒绝流量）

#### （*拒绝*）从 Web 通过防火墙访问 SQL
1.	Internet 用户从 FrontEnd001.CloudApp.Net（面向 Internet 的云服务）请求 SQL 数据
2.	由于没有为 SQL 开放终结点，此流量不会通过云服务抵达防火墙
3.	如果出于某种原因而开放了终结点，前端子网将开始处理入站规则：
  1.	NSG 规则 1 (DNS) 不适用，将转到下一规则
  2.	NSG 规则 2 (RDP) 不适用，将转到下一规则
  3.	NSG 规则 3（Internet 到 IIS01）适用，允许流量，停止处理规则
4.	流量抵达 IIS01 的内部 IP 地址 (10.0.1.5)
5.	IIS01 未侦听端口 1433，因此不会对请求做出响应

## 结束语
这种隔离后端子网与输入流量的方式相当直截了当。

<!-- 可以在[此处][HOME]找到更多示例和网络安全边界的概述。-->

## 参考
### 主脚本和网络配置
将完整脚本保存在 PowerShell 脚本文件中。将网络配置保存到名为“NetworkConf1.xml”的文件中。根据需要修改用户定义的变量。运行脚本，然后根据上面“示例 1”部分中所述的防火墙规则设置说明操作。

#### 完整脚本
此脚本基于用户定义的变量执行以下操作：

1.	连接到 Azure 订阅
2.	新建存储帐户
3.	根据网络配置文件中的定义创建新的 VNet 和两个子网
4.	构建 4 个 Windows Server VM
5.	配置 NSG，包括：
  -	创建 NSG
  -	填入规则
  -	将 NSG 绑定到相应的子网

此 PowerShell 脚本应该在连接到 Internet 的电脑或服务器上本地运行。

>[AZURE.IMPORTANT]此此脚本运行时，PowerShell 中可能会弹出警告或其他参考性消息。只有以红色字体显示的错误消息才需要引以关注。


	<# 
	 .SYNOPSIS
	  Example of Network Security Groups in an isolated network (Azure only, no hybrid connections)
	
	 .DESCRIPTION
	  This script will build out a sample DMZ setup containing:
	   - A default storage account for VM disks
	   - Two new cloud services
	   - Two Subnets (FrontEnd and BackEnd subnets)
	   - One server on the FrontEnd Subnet
	   - Three Servers on the BackEnd Subnet
	   - Network Security Groups to allow/deny traffic patterns as declared
	  
	  Before running script, ensure the network configuration file is created in
	  the directory referenced by $NetworkConfigFile variable (or update the
	  variable to reflect the path and file name of the config file being used).
	
	 .Notes
	  Security requirements are different for each use case and can be addressed in a
	  myriad of ways. Please be sure that any sensitive data or applications are behind
	  the appropriate layer(s) of protection. This script serves as an example of some
	  of the techniques that can be used, but should not be used for all scenarios. You
	  are responsible to assess your security needs and the appropriate protections
	  needed, and then effectively implement those protections.
	
	  FrontEnd Service (FrontEnd subnet 10.0.1.0/24)
	   IIS01      - 10.0.1.5
	 
	  BackEnd Service (BackEnd subnet 10.0.2.0/24)
	   DNS01      - 10.0.2.4
	   AppVM01    - 10.0.2.5
	   AppVM02    - 10.0.2.6
	
	#>
	
	# Fixed Variables
	    $LocalAdminPwd = Read-Host -Prompt "Enter Local Admin Password to be used for all VMs"
	    $VMName = @()
	    $ServiceName = @()
	    $VMFamily = @()
	    $img = @()
	    $size = @()
	    $SubnetName = @()
	    $VMIP = @()
	
	# User Defined Global Variables
	  # These should be changes to reflect your subscription and services
	  # Invalid options will fail in the validation section
	
	  # Subscription Access Details
	    $subID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
	
	  # VM Account, Location, and Storage Details
	    $LocalAdmin = "theAdmin"
	    $DeploymentLocation = "China North"
	    $StorageAccountName = "vmstore02"
	
	  # Service Details
	    $FrontEndService = "FrontEnd001"
	    $BackEndService = "BackEnd001"
	
	  # Network Details
	    $VNetName = "CorpNetwork"
	    $FESubnet = "FrontEnd"
	    $FEPrefix = "10.0.1.0/24"
	    $BESubnet = "BackEnd"
	    $BEPrefix = "10.0.2.0/24"
	    $NetworkConfigFile = "C:\Scripts\NetworkConf1.xml"
	
	  # VM Base Disk Image Details
	    $SrvImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Windows Server 2012 R2 Datacenter'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}
	  
	  # NSG Details
	    $NSGName = "MyVNetSG"
	
	# User Defined VM Specific Config
	    # Note: To ensure proper NSG Rule creation later in this script:
	    #       - The Web Server must be VM 0
	    #       - The AppVM1 Server must be VM 1
	    #       - The DNS server must be VM 3
	    #
	    #       Otherwise the NSG rules in the last section of this
	    #       script will need to be changed to match the modified
	    #       VM array numbers ($i) so the NSG Rule IP addresses
	    #       are aligned to the associated VM IP addresses.
	
	    # VM 0 - The Web Server
	      $VMName += "IIS01"
	      $ServiceName += $FrontEndService
	      $VMFamily += "Windows"
	      $img += $SrvImg
	      $size += "Standard_D3"
	      $SubnetName += $FESubnet
	      $VMIP += "10.0.1.5"
	
	    # VM 1 - The First Application Server
	      $VMName += "AppVM01"
	      $ServiceName += $BackEndService
	      $VMFamily += "Windows"
	      $img += $SrvImg
	      $size += "Standard_D3"
	      $SubnetName += $BESubnet
	      $VMIP += "10.0.2.5"
	
	    # VM 2 - The Second Application Server
	      $VMName += "AppVM02"
	      $ServiceName += $BackEndService
	      $VMFamily += "Windows"
	      $img += $SrvImg
	      $size += "Standard_D3"
	      $SubnetName += $BESubnet
	      $VMIP += "10.0.2.6"
	
	    # VM 3 - The DNS Server
	      $VMName += "DNS01"
	      $ServiceName += $BackEndService
	      $VMFamily += "Windows"
	      $img += $SrvImg
	      $size += "Standard_D3"
	      $SubnetName += $BESubnet
	      $VMIP += "10.0.2.4"

	# ----------------------------- #
	# No User Defined Varibles or   #
	# Configuration past this point #
	# ----------------------------- #	

	  # Get your Azure accounts
	    Add-AzureAccount
	    Set-AzureSubscription –SubscriptionId $subID -ErrorAction Stop
	    Select-AzureSubscription -SubscriptionId $subID -Current -ErrorAction Stop
	
	  # Create Storage Account
	    If (Test-AzureName -Storage -Name $StorageAccountName) { 
	        Write-Host "Fatal Error: This storage account name is already in use, please pick a diffrent name." -ForegroundColor Red
	        Return}
	    Else {Write-Host "Creating Storage Account" -ForegroundColor Cyan 
	          New-AzureStorageAccount -Location $DeploymentLocation -StorageAccountName $StorageAccountName}
	
	  # Update Subscription Pointer to New Storage Account
	    Write-Host "Updating Subscription Pointer to New Storage Account" -ForegroundColor Cyan 
	    Set-AzureSubscription –SubscriptionId $subID -CurrentStorageAccountName $StorageAccountName -ErrorAction Stop
	
	# Validation
	$FatalError = $false
	
	If (-Not (Get-AzureLocation | Where {$_.DisplayName -eq $DeploymentLocation})) {
	     Write-Host "This Azure Location was not found or available for use" -ForegroundColor Yellow
	     $FatalError = $true}
	
	If (Test-AzureName -Service -Name $FrontEndService) { 
	    Write-Host "The FrontEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
	    $FatalError = $true}
	Else { Write-Host "The FrontEndService service name is valid for use." -ForegroundColor Green}
	
	If (Test-AzureName -Service -Name $BackEndService) { 
	    Write-Host "The BackEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
	    $FatalError = $true}
	Else { Write-Host "The BackEndService service name is valid for use." -ForegroundColor Green}
	
	If (-Not (Test-Path $NetworkConfigFile)) { 
	    Write-Host 'The network config file was not found, please update the $NetworkConfigFile variable to point to the network config xml file.' -ForegroundColor Yellow
	    $FatalError = $true}
	Else { Write-Host "The network config file was found" -ForegroundColor Green
	        If (-Not (Select-String -Pattern $DeploymentLocation -Path $NetworkConfigFile)) {
	            Write-Host 'The deployment location was not found in the network config file, please check the network config file to ensure the $DeploymentLocation varible is correct and the netowrk config file matches.' -ForegroundColor Yellow
	            $FatalError = $true}
	        Else { Write-Host "The deployment location was found in the network config file." -ForegroundColor Green}}
	
	If ($FatalError) {
	    Write-Host "A fatal error has occured, please see the above messages for more information." -ForegroundColor Red
	    Return}
	Else { Write-Host "Validation passed, now building the environment." -ForegroundColor Green}
	
	# Create VNET
	    Write-Host "Creating VNET" -ForegroundColor Cyan 
	    Set-AzureVNetConfig -ConfigurationPath $NetworkConfigFile -ErrorAction Stop
	
	# Create Services
	    Write-Host "Creating Services" -ForegroundColor Cyan
	    New-AzureService -Location $DeploymentLocation -ServiceName $FrontEndService -ErrorAction Stop
	    New-AzureService -Location $DeploymentLocation -ServiceName $BackEndService -ErrorAction Stop
	
	# Build VMs
	    $i=0
	    $VMName | Foreach {
	        Write-Host "Building $($VMName[$i])" -ForegroundColor Cyan
	        New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] –InstanceSize $size[$i] | `
	            Add-AzureProvisioningConfig -Windows -AdminUsername $LocalAdmin -Password $LocalAdminPwd  | `
	            Set-AzureSubnet  –SubnetNames $SubnetName[$i] | `
	            Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
	            Set-AzureVMMicrosoftAntimalwareExtension -AntimalwareConfiguration '{"AntimalwareEnabled" : true}' | `
	            Remove-AzureEndpoint -Name "PowerShell" | `
	            New-AzureVM –ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
	            # Note: A Remote Desktop endpoint is automatically created when each VM is created.
	        $i++
	    }
	    # Add HTTP Endpoint for IIS01
	    Get-AzureVM -ServiceName $ServiceName[0] -Name $VMName[0] | Add-AzureEndpoint -Name HTTP -Protocol tcp -LocalPort 80 -PublicPort 80 | Update-AzureVM
	
	# Configure NSG
	    Write-Host "Configuring the Network Security Group (NSG)" -ForegroundColor Cyan
	    
	  # Build the NSG
	    Write-Host "Building the NSG" -ForegroundColor Cyan
	    New-AzureNetworkSecurityGroup -Name $NSGName -Location $DeploymentLocation -Label "Security group for $VNetName subnets in $DeploymentLocation"
	
	  # Add NSG Rules
	    Write-Host "Writing rules into the NSG" -ForegroundColor Cyan
	    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internal DNS" -Type Inbound -Priority 100 -Action Allow `
	        -SourceAddressPrefix VIRTUAL_NETWORK -SourcePortRange '*' `
	        -DestinationAddressPrefix $VMIP[3] -DestinationPortRange '53' `
	        -Protocol *
	
	    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable RDP to $VNetName VNet" -Type Inbound -Priority 110 -Action Allow `
	        -SourceAddressPrefix INTERNET -SourcePortRange '*' `
	        -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '3389' `
	        -Protocol *
	
	    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internet to $($VMName[0])" -Type Inbound -Priority 120 -Action Allow `
	        -SourceAddressPrefix Internet -SourcePortRange '*' `
	        -DestinationAddressPrefix $VMIP[0] -DestinationPortRange '*' `
	        -Protocol *
	
	    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable $($VMName[0]) to $($VMName[1])" -Type Inbound -Priority 130 -Action Allow `
	        -SourceAddressPrefix $VMIP[0] -SourcePortRange '*' `
	        -DestinationAddressPrefix $VMIP[1] -DestinationPortRange '*' `
	        -Protocol *
	    
	    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Isolate the $VNetName VNet from the Internet" -Type Inbound -Priority 140 -Action Deny `
	        -SourceAddressPrefix INTERNET -SourcePortRange '*' `
	        -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '*' `
	        -Protocol *
	
	    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Isolate the $FESubnet subnet from the $BESubnet subnet" -Type Inbound -Priority 150 -Action Deny `
	        -SourceAddressPrefix $FEPrefix -SourcePortRange '*' `
	        -DestinationAddressPrefix $BEPrefix -DestinationPortRange '*' `
	        -Protocol *
	
	    # Assign the NSG to the Subnets
	        Write-Host "Binding the NSG to both subnets" -ForegroundColor Cyan
	        Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName -SubnetName $FESubnet -VirtualNetworkName $VNetName
	        Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName -SubnetName $BESubnet -VirtualNetworkName $VNetName
	
	# Optional Post-script Manual Configuration
	  # Install Test Web 应用(Run Post-Build Script on the IIS Server)
	  # Install Backend resource (Run Post-Build Script on the AppVM01)
	  Write-Host
	  Write-Host "Build Complete!" -ForegroundColor Green
	  Write-Host
	  Write-Host "Optional Post-script Manual Configuration Steps" -ForegroundColor Gray
	  Write-Host " - Install Test Web 应用(Run Post-Build Script on the IIS Server)" -ForegroundColor Gray
	  Write-Host " - Install Backend resource (Run Post-Build Script on the AppVM01)" -ForegroundColor Gray
	  Write-Host
	  

#### 网络配置文件
使用更新的位置保存此 xml 文件，并将此文件的链接添加到上述脚本中的 $NetworkConfigFile 变量。
	
	<NetworkConfiguration xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
	  <VirtualNetworkConfiguration>
	    <Dns>
	      <DnsServers>
	        <DnsServer name="DNS01" IPAddress="10.0.2.4" />
	        <DnsServer name="Level3" IPAddress="209.244.0.3" />
	      </DnsServers>
	    </Dns>
	    <VirtualNetworkSites>
	      <VirtualNetworkSite name="CorpNetwork" Location="China North">
	        <AddressSpace>
	          <AddressPrefix>10.0.0.0/16</AddressPrefix>
	        </AddressSpace>
	        <Subnets>
	          <Subnet name="FrontEnd">
	            <AddressPrefix>10.0.1.0/24</AddressPrefix>
	          </Subnet>
	          <Subnet name="BackEnd">
	            <AddressPrefix>10.0.2.0/24</AddressPrefix>
	          </Subnet>
	        </Subnets>
	        <DnsServersRef>
	          <DnsServerRef name="DNS01" />
	          <DnsServerRef name="Level3" />
	        </DnsServersRef>
	      </VirtualNetworkSite>
	    </VirtualNetworkSites>
	  </VirtualNetworkConfiguration>
	</NetworkConfiguration>

#### 示例应用程序脚本
如果你想要为本示例和其他外围网络示例安装示例应用程序，以下链接便已提供了一个：[示例应用程序脚本][SampleApp]

<!--Image References-->
[1]: ./media/virtual-networks-dmz-nsg-asm/example1design.png "使用 NSG 的入站外围网络"

<!--Link References-->
[HOME]: /documentation/articles/best-practices-network-security/
[SampleApp]: /documentation/articles/virtual-networks-sample-app/

<!---HONumber=82-->