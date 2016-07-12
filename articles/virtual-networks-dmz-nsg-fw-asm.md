<properties
   pageTitle="外围网络示例 – 构建外围网络以通过防火墙和 NSG 保护应用程序 | Azure"
   description="使用防火墙和网络安全组 (NSG) 构建外围网络"
   services="virtual-network"
   documentationCenter="na"
   authors="tracsman"
   manager="rossort"
   editor=""/>

<tags
	ms.service="virtual-network"
	ms.date="02/01/2016"
	wacn.date="03/17/2016"/>

# 示例 2 – 构建外围网络以通过防火墙和 NSG 保护应用程序

本示例将创建一个带防火墙的外围网络、四个 Windows 服务器和网络安全组。本示例还将演练每个相关命令，让你更加深入地了解每个步骤。另外还提供了“流量方案”部分，让你逐步深入了解流量如何流经外围网络的各个防御层。最后的“参考”部分提供了完整的代码，并说明如何构建此环境来测试和试验各种方案。

![使用 NVA 和 NSG 的入站外围网络][1]

## 环境描述
此示例中，有一个订阅包含以下项：

- 两个云服务：“FrontEnd001”和“BackEnd001”
- 一个虚拟网络“CorpNetwork”，其中包含两个子网：“FrontEnd”和“BackEnd”
- 应用到这两个子网的单个网络安全组
- 一个与前端子网连接的网络虚拟设备（在本示例中为 Barracuda NextGen Firewall）
- 一个代表应用程序 Web 服务器的 Windows Server（“IIS01”）
- 两个代表应用程序后端服务器的 Windows Server（“AppVM01”、“AppVM02”）
- 一个代表 DNS 服务器的 Windows Server（“DNS01”）

>[AZURE.NOTE] 虽然本示例使用了 Barracuda NextGen Firewall，但你也可以使用多种不同的网络虚拟设备。

下面的“参考”部分提供了可用于构建上述大多数环境的 PowerShell 脚本。尽管该示例脚本还完成了 VM 和虚拟网络的构建，但本文未对其进行详细描述。
 
构建环境：

  1.	保存“参考”部分中包含的网络配置 xml 文件（更新名称、位置和 IP 地址以符合给定的方案）
  2.	更新脚本中的用户变量，以符合用于运行脚本的环境（订阅、服务名称等）
  3.	在 PowerShell 中执行脚本

**注意**：PowerShell 脚本中提到的区域必须与网络配置 xml 文件中提到的区域相匹配。

成功运行脚本后，可执行以下脚本后续步骤：

1.	设置防火墙规则，下面的“防火墙规则”部分中对此进行了介绍。
2.	（可选）“参考”部分中提供了两个脚本，用于设置 Web 服务器和包含简单 Web 应用程序的应用服务器，以便能使用此外围网络配置进行测试。

下一部分介绍与网络安全组相关的大部分脚本语句。

## 网络安全组 (NSG)
本示例将构建一个 NSG 组，然后加载六个规则。

>[AZURE.TIP] 一般而言，应该先创建特定的“允许”规则，最后再创建一般的“拒绝”规则。分配的优先级确定了要先评估哪些规则。发现要向流量应用哪个特定规则后，不需要评估后续规则。可以朝入站或出站方向（从子网的角度看）应用 NSG 规则。

以声明性的方式为入站流量构建以下规则：

1.	允许内部 DNS 流量（端口 53）
2.	允许从 Internet 到任何 VM 的 RDP 流量（端口 3389）
3.	允许从 Internet 到 NVA（防火墙）的 HTTP 流量（端口 80）
4.	允许从 IIS01 到 AppVM1 的任何流量（所有端口）
5.	拒绝从 Internet 到整个 VNet（两个子网）的任何流量（所有端口）
6.	拒绝从前端子网到后端子网的任何流量（所有端口）

将这些规则绑定到每个子网后，如果有从 Internet 到 Web 服务器的入站 HTTP 请求，将应用规则 3（允许）和规则 5（拒绝），但由于规则 3 具有较高的优先级，因此只应用规则 3 并忽略规则 5。这样就会允许 HTTP 请求传往防火墙。如果相同的流量尝试传往 DNS01 服务器，则会先应用规则 5（拒绝），因此不允许该流量传递到服务器。规则 6（拒绝）阻止前端子网与后端子网对话（规则 1 和 4 允许的流量除外），这可在攻击者入侵前端上的 Web 应用程序时保护后端网络，攻击者只能对后端的“受保护”网络进行有限度的访问（只能访问 AppVM01 服务器上公开的资源）。

有一个默认出站规则可允许流量外流到 Internet。在此示例中，我们允许出站流量，且未修改任何出站规则。如果两个方向的流量都要锁定，则需要用户定义的路由。

上述 NSG 规则非常类似于[示例 1 - 使用 NSG 构建简单的外围网络][Example1]中的 NSG 规则。请查看该文档中的 NSG 说明，以详细了解每个 NSG 规则及其属性。

## 防火墙规则
电脑上必须安装管理客户端才能管理防火墙和创建所需的配置。有关如何管理设备的信息，请参阅防火墙（或其他 NVA）供应商提供的文档。本部分的余下内容将介绍如何通过供应商的管理客户端（即，不使用 Azure 经典管理门户或 PowerShell）来配置防火墙本身。

有关下载客户端和连接到本示例所用 Barracuda 的说明，可在以下位置找到：[Barracuda NG Admin](https://techlib.barracuda.com/NG61/NGAdmin)

需要在防火墙上创建转发规则。本示例只将 Internet 流量入站路由到防火墙，再传送到 Web 服务器，因此只需要一个转发 NAT 规则。在本示例使用的 Barracuda NextGen Firewall 上，这个规则是目标 NAT 规则（“Dst NAT”），将由它传递此流量。

若要创建以下规则（或验证现有的默认规则），请先访问 Barracuda NG Admin 客户端仪表板，导航到“设置”选项卡，在“操作配置”部分中单击“规则集”。此时将出现名为“主规则”的网格，其中显示了该防火墙的现有活动规则和已停用规则。此网格右上角有一个绿色“+”小按钮，单击此按钮即可创建新规则（注意：防火墙可能会“禁止”更改，如果你看到标记为“锁定”的按钮且无法创建或编辑规则，请单击此按钮以“解除锁定”规则集并允许编辑）。如果你想要编辑现有规则，请选择该规则，右键单击并选择“编辑规则”。

创建新规则并提供名称，例如“WebTraffic”。

目标 NAT 规则图标类似于：![目标 NAT 图标][2]

规则本身外观类似于：

![防火墙规则][3]

任何到达防火墙并尝试连接到 HTTP（端口 80，若为 HTTPS 则为 443）的入站地址将从防火墙的“DHCP1 本地 IP”接口发出，并重定向到 IP 地址为 10.0.1.5 的 Web 服务器。由于流量在端口 80 上传入，并在端口 80 上流向 Web 服务器，因此不需要更改端口。但是，如果在端口 8080 上侦听 Web 服务器，因而要将防火墙上的入站端口 80 转换为 Web 服务器上的入站端口 8080，则目标列表应为 10.0.1.5:8080。

另外，还应该针对“来自 Internet 的目标规则”指明连接方法，其中“动态 SNAT”是最合适的。

尽管只创建了一个规则，但正确设置其优先级仍然很重要。如果在防火墙上所有规则的网格中，此新规则位于底部（在“BLOCKALL”规则之下），它将永远不会派上用场。请确保针对 Web 流量新建的规则在 BLOCKALL 规则之上。

创建规则后，必须将其推送到防火墙并激活，如果不这么做，规则更改将不会生效。以下部分介绍了推送和激活过程。

## 规则激活
修改规则集以添加此规则后，必须将规则集上载到防火墙并激活。

![防火墙规则激活][4]

管理客户端右上角是按钮群集。单击“发送更改”按钮将修改后的规则发送到防火墙，然后单击“激活”按钮。

激活防火墙规则集后，此示例环境即构建完成。（可选）运行“参考”部分中的后续构建脚本，将应用程序添加到此环境以测试下面的流量方案。

>[AZURE.IMPORTANT] 你不会直接连接 Web 服务器，了解这一点很重要。当浏览器从 FrontEnd001.CloudApp.Net 请求 HTTP 页面时，HTTP 终结点（端口 80）会将此流量传递到防火墙而不是 Web 服务器。然后，防火墙根据上面创建的规则，通过 NAT 将该请求传送到 Web 服务器。

## 流量方案

#### （允许）通过防火墙从 Web 访问 Web 服务器
1.	Internet 用户从 FrontEnd001.CloudApp.Net（面向 Internet 的云服务）请求 HTTP 页面
2.	云服务通过端口 80 上的开放终结点将流量传递到 10.0.1.4:80 上的防火墙本地接口
3.	前端子网开始处理入站规则：
  1.	NSG 规则 1 (DNS) 不适用，将转到下一规则
  2.	NSG 规则 2 (RDP) 不适用，将转到下一规则
  3.	NSG 规则 3（Internet 到防火墙）适用，允许流量，停止处理规则
4.	流量抵达防火墙的内部 IP 地址 (10.0.1.4)
5.	防火墙转发规则将此流量视为端口 80 流量，并将它重定向到 Web 服务器 IIS01
6.	IIS01 正在侦听 Web 流量，将接收此请求并开始处理请求
7.	IIS01 请求 AppVM01 上的 SQL Server 提供信息
8.	前端子网上没有出站规则，允许流量
9.	后端子网开始处理入站规则：
  1.	NSG 规则 1 (DNS) 不适用，将转到下一规则
  2.	NSG 规则 2 (RDP) 不适用，将转到下一规则
  3.	NSG 规则 3（Internet 到防火墙）不适用，将转到下一规则
  4.	NSG 规则 4（IIS01 到 AppVM01）适用，允许流量，停止规则处理
10.	AppVM01 接收 SQL 查询并做出响应
11.	后端子网上没有出站规则，因此允许响应
12.	前端子网开始处理入站规则：
  1.	后端子网到前端子网的入站流量没有适用的 NSG 规则，因此不会应用任何 NSG 规则
  2.	允许子网间流量的默认系统规则允许此流量，因此允许流量。
13.	IIS 服务器接收 SQL 响应，完成 HTTP 响应并发送给请求方
14.	由于这是来自防火墙的 NAT 会话，因此响应目标（最初）针对防火墙
15.	防火墙接收来自 Web 服务器的响应，并将其转发回到 Internet 用户
16.	前端子网上没有出站规则，因此允许响应，Internet 用户将收到请求的网页。

#### （允许）通过 RDP 访问后端
1.	Internet 上的服务器管理员在 BackEnd001.CloudApp.Net:xxxxx 上请求与 AppVM01 的 RDP 会话，其中 xxxxx 是通过 RDP 访问 AppVM01 所用的随机分配端口号（在 Azure 经典管理门户上或通过 PowerShell 可以找到分配的端口）
2.	防火墙只在 FrontEnd001.CloudApp.Net 地址上侦听，因此不参与此流量的传送
3.	后端子网开始处理入站规则：
  1.	NSG 规则 1 (DNS) 不适用，将转到下一规则
  2.	NSG 规则 2 (RDP) 适用，允许流量，停止规则处理
4.	由于没有出站规则，将应用默认规则并允许返回流量
5.	已启用 RDP 会话
6.	AppVM01 提示输入用户名和密码

#### （允许）在 DNS 服务器上执行 Web 服务器 DNS 查找
1.	Web 服务器 IIS01 需要 www.data.gov 上的数据源，但需要解析地址。
2.	VNet 的网络配置将 DNS01（后端子网上的 10.0.2.4）列为主 DNS 服务器，IIS01 将 DNS 请求发送到 DNS01
3.	前端子网上没有出站规则，允许流量
4.	后端子网开始处理入站规则：
  1.	 NSG 规则 1 (DNS) 适用，允许流量，停止规则处理
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

#### （允许）Web 服务器访问 AppVM01 上的文件
1.	IIS01 请求 AppVM01 上的文件
2.	前端子网上没有出站规则，允许流量
3.	后端子网开始处理入站规则：
  1.	NSG 规则 1 (DNS) 不适用，将转到下一规则
  2.	NSG 规则 2 (RDP) 不适用，将转到下一规则
  3.	NSG 规则 3（Internet 到防火墙）不适用，将转到下一规则
  4.	NSG 规则 4（IIS01 到 AppVM01）适用，允许流量，停止规则处理
4.	AppVM01 接收请求并以文件做出响应（假设已获得访问授权）
5.	后端子网上没有出站规则，因此允许响应
6.	前端子网开始处理入站规则：
  1.	后端子网到前端子网的入站流量没有适用的 NSG 规则，因此不会应用任何 NSG 规则
  2.	允许子网间流量的默认系统规则允许此流量，因此允许流量。
7.	IIS 服务器接收文件

#### （拒绝）从 Web 直接访问 Web 服务器
Web 服务器、IIS01 和防火墙都在相同的云服务中，因此共享相同的公开 IP 地址。因此，任何 HTTP 流量都会定向到防火墙。尽管可以成功处理请求，但不能将其直接定向到 Web 服务器，而是按照设计首先通过防火墙。请参阅本部分第一个方案中所述的流量传送模式。

#### （拒绝）通过 Web 访问后端服务器
1.	Internet 用户尝试通过 BackEnd001.CloudApp.Net 服务访问 AppVM01 上的文件
2.	由于没有为文件共享开放终结点，此流量不会通过云服务抵达服务器
3.	如果出于某种原因而开放了终结点，NSG 规则 5（Internet 到 VNet）将阻止此流量

#### （拒绝）在 DNS 服务器上执行 Web DNS 查找
1.	Internet 用户尝试通过 BackEnd001.CloudApp.Net 服务查找 DNS01 上的内部 DNS 记录
2.	由于没有为 DNS 开放终结点，此流量不会通过云服务抵达服务器
3.	如果出于某种原因而开放了终结点，NSG 规则 5（Internet 到 VNet）将阻止此流量（注意：有两个原因导致规则 1 (DNS) 不适用：首先，源地址是 Internet，此规则只适用于以本地 VNet 作为源；其次，这是允许规则，因此永远不会拒绝流量）

#### （拒绝）从 Web 通过防火墙访问 SQL
1.	Internet 用户从 FrontEnd001.CloudApp.Net（面向 Internet 的云服务）请求 SQL 数据
2.	由于没有为 SQL 开放终结点，此流量不会通过云服务抵达防火墙
3.	如果出于某种原因而开放了终结点，前端子网将开始处理入站规则：
  1.	NSG 规则 1 (DNS) 不适用，将转到下一规则
  2.	NSG 规则 2 (RDP) 不适用，将转到下一规则
  3.	NSG 规则 2（Internet 到防火墙）适用，允许流量，停止处规则处理
4.	流量抵达防火墙的内部 IP 地址 (10.0.1.4)
5.	防火墙上没有 SQL 的任何转发规则，将丢弃流量

## 结束语
这种使用防火墙保护应用程序并隔离后端子网与输入流量的方式相当直截了当。

## 参考
### 主脚本和网络配置
将完整脚本保存在 PowerShell 脚本文件中。将网络配置保存到名为“NetworkConf2.xml”的文件中。如有需要，请修改用户定义的变量。运行脚本，然后根据上面的防火墙规则设置说明操作。

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

>[AZURE.IMPORTANT] 此此脚本运行时，PowerShell 中可能会弹出警告或其他参考性消息。只有以红色字体显示的错误消息才需要引以关注。


	<# 
	 .SYNOPSIS
	  Example of DMZ and Network Security Groups in an isolated network (Azure only, no hybrid connections)
	
	 .DESCRIPTION
	  This script will build out a sample DMZ setup containing:
	   - A default storage account for VM disks
	   - Two new cloud services
	   - Two Subnets (FrontEnd and BackEnd subnets)
	   - A Network Virtual Appliance (NVA), in this case a Barracuda NextGen Firewall
	   - One server on the FrontEnd Subnet (plus the NVA on the FrontEnd subnet)
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
	   myFirewall - 10.0.1.4
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
	    $NetworkConfigFile = "C:\Scripts\NetworkConf2.xml"
	
	  # VM Base Disk Image Details
	    $SrvImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Windows Server 2012 R2 Datacenter'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}
	    $FWImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Barracuda NextGen Firewall'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}
	
	  # NSG Details
	    $NSGName = "MyVNetSG"
	
	# User Defined VM Specific Config
	    # Note: To ensure proper NSG Rule creation later in this script:
	    #       - The Web Server must be VM 1
	    #       - The AppVM1 Server must be VM 2
	    #       - The DNS server must be VM 4
	    #
	    #       Otherwise the NSG rules in the last section of this
	    #       script will need to be changed to match the modified
	    #       VM array numbers ($i) so the NSG Rule IP addresses
	    #       are aligned to the associated VM IP addresses.
	
	    # VM 0 - The Network Virtual Appliance (NVA)
	      $VMName += "myFirewall"
	      $ServiceName += $FrontEndService
	      $VMFamily += "Firewall"
	      $img += $FWImg
	      $size += "Small"
	      $SubnetName += $FESubnet
	      $VMIP += "10.0.1.4"
	
	    # VM 1 - The Web Server
	      $VMName += "IIS01"
	      $ServiceName += $FrontEndService
	      $VMFamily += "Windows"
	      $img += $SrvImg
	      $size += "Standard_D3"
	      $SubnetName += $FESubnet
	      $VMIP += "10.0.1.5"
	
	    # VM 2 - The First Appliaction Server
	      $VMName += "AppVM01"
	      $ServiceName += $BackEndService
	      $VMFamily += "Windows"
	      $img += $SrvImg
	      $size += "Standard_D3"
	      $SubnetName += $BESubnet
	      $VMIP += "10.0.2.5"
	
	    # VM 3 - The Second Appliaction Server
	      $VMName += "AppVM02"
	      $ServiceName += $BackEndService
	      $VMFamily += "Windows"
	      $img += $SrvImg
	      $size += "Standard_D3"
	      $SubnetName += $BESubnet
	      $VMIP += "10.0.2.6"
	
	    # VM 4 - The DNS Server
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
	    Set-AzureSubscription -SubscriptionId $subID -ErrorAction Stop
	    Select-AzureSubscription -SubscriptionId $subID -Current -ErrorAction Stop
	
	  # Create Storage Account
	    If (Test-AzureName -Storage -Name $StorageAccountName) { 
	        Write-Host "Fatal Error: This storage account name is already in use, please pick a diffrent name." -ForegroundColor Red
	        Return}
	    Else {Write-Host "Creating Storage Account" -ForegroundColor Cyan 
	          New-AzureStorageAccount -Location $DeploymentLocation -StorageAccountName $StorageAccountName}
	
	  # Update Subscription Pointer to New Storage Account
	    Write-Host "Updating Subscription Pointer to New Storage Account" -ForegroundColor Cyan 
	    Set-AzureSubscription -SubscriptionId $subID -CurrentStorageAccountName $StorageAccountName -ErrorAction Stop
	
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
	        If ($VMFamily[$i] -eq "Firewall") 
	            { 
	            New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] -InstanceSize $size[$i] | `
	                Add-AzureProvisioningConfig -Linux -LinuxUser $LocalAdmin -Password $LocalAdminPwd  | `
	                Set-AzureSubnet  -SubnetNames $SubnetName[$i] | `
	                Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
	                New-AzureVM -ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
	            # Set up all the EndPoints we'll need once we're up and running
	            # Note: Web traffic goes through the firewall, so we'll need to set up a HTTP endpoint.
	            #       Also, the firewall will be redirecting web traffic to a new IP and Port in a
	            #       forwarding rule, so the HTTP endpoint here will have the same public and local
	            #       port and the firewall will do the NATing and redirection as declared in the
	            #       firewall rule.
	            Add-AzureEndpoint -Name "MgmtPort1" -Protocol tcp -PublicPort 801  -LocalPort 801  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
	            Add-AzureEndpoint -Name "MgmtPort2" -Protocol tcp -PublicPort 807  -LocalPort 807  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
	            Add-AzureEndpoint -Name "HTTP"      -Protocol tcp -PublicPort 80   -LocalPort 80   -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
	            # Note: A SSH endpoint is automatically created on port 22 when the appliance is created.
	            }
	        Else
	            {
	            New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] -InstanceSize $size[$i] | `
	                Add-AzureProvisioningConfig -Windows -AdminUsername $LocalAdmin -Password $LocalAdminPwd  | `
	                Set-AzureSubnet  -SubnetNames $SubnetName[$i] | `
	                Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
	                Set-AzureVMMicrosoftAntimalwareExtension -AntimalwareConfiguration '{"AntimalwareEnabled" : true}' | `
	                Remove-AzureEndpoint -Name "PowerShell" | `
	                New-AzureVM -ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
	                # Note: A Remote Desktop endpoint is automatically created when each VM is created.
	            }
	        $i++
	    }
	
	# Configure NSG
	    Write-Host "Configuring the Network Security Group (NSG)" -ForegroundColor Cyan
	    
	  # Build the NSG
	    Write-Host "Building the NSG" -ForegroundColor Cyan
	    New-AzureNetworkSecurityGroup -Name $NSGName -Location $DeploymentLocation -Label "Security group for $VNetName subnets in $DeploymentLocation"
	
	  # Add NSG Rules
	    Write-Host "Writing rules into the NSG" -ForegroundColor Cyan
	    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internal DNS" -Type Inbound -Priority 100 -Action Allow `
	        -SourceAddressPrefix VIRTUAL_NETWORK -SourcePortRange '*' `
	        -DestinationAddressPrefix $VMIP[4] -DestinationPortRange '53' `
	        -Protocol *
	
	    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable RDP to $VNetName VNet" -Type Inbound -Priority 110 -Action Allow `
	        -SourceAddressPrefix INTERNET -SourcePortRange '*' `
	        -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '3389' `
	        -Protocol *
	
	    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internet to $($VMName[0])" -Type Inbound -Priority 120 -Action Allow `
	        -SourceAddressPrefix Internet -SourcePortRange '*' `
	        -DestinationAddressPrefix $VMIP[0] -DestinationPortRange '*' `
	        -Protocol *
	
	    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable $($VMName[1]) to $($VMName[2])" -Type Inbound -Priority 130 -Action Allow `
	        -SourceAddressPrefix $VMIP[1] -SourcePortRange '*' `
	        -DestinationAddressPrefix $VMIP[2] -DestinationPortRange '*' `
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
	  # Configure Firewall
	  # Install Test Web App (Run Post-Build Script on the IIS Server)
	  # Install Backend resource (Run Post-Build Script on the AppVM01)
	  Write-Host
	  Write-Host "Build Complete!" -ForegroundColor Green
	  Write-Host
	  Write-Host "Optional Post-script Manual Configuration Steps" -ForegroundColor Gray
	  Write-Host " - Configure Firewall" -ForegroundColor Gray
	  Write-Host " - Install Test Web App (Run Post-Build Script on the IIS Server)" -ForegroundColor Gray
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
[1]: ./media/virtual-networks-dmz-nsg-fw-asm/example2design.png "使用 NSG 的入站外围网络"
[2]: ./media/virtual-networks-dmz-nsg-fw-asm/dstnaticon.png "目标 NAT 图标"
[3]: ./media/virtual-networks-dmz-nsg-fw-asm/firewallrule.png "防火墙规则"
[4]: ./media/virtual-networks-dmz-nsg-fw-asm/firewallruleactivate.png "防火墙规则激活"

<!--Link References-->
[HOME]: /documentation/articles/best-practices-network-security/
[SampleApp]: /documentation/articles/virtual-networks-sample-app/
[Example1]: /documentation/articles/virtual-networks-dmz-nsg-asm/

<!---HONumber=Mooncake_0307_2016-->