<properties
   pageTitle="外围网络示例 – 构建外围网络以通过防火墙、UDR 和 NSG 保护网络 | Azure"
   description="构建包含防火墙、用户定义的路由 (UDR) 和网络安全组 (NSG) 的外围网络"
   services="virtual-network"
   documentationCenter="na"
   authors="tracsman"
   manager="rossort"
   editor=""/>

<tags
	ms.service="virtual-network"
	ms.date="02/01/2016"
	wacn.date="03/17/2016"/>

# 示例 3 – 构建外围网络以通过防火墙、UDR 和 NSG 保护网络

本示例将创建一个外围网络，其中包含防火墙、四个 Windows 服务器、用户定义的路由、IP 转发和网络安全组。本示例还将演练每个相关命令，让你更加深入地了解每个步骤。另外还提供了“流量方案”部分，让你逐步深入了解流量如何流经外围网络的各个防御层。最后的“参考”部分提供了完整的代码，并说明如何构建此环境来测试和试验各种方案。

![使用 NVA、NSG 和 UDR 的双向外围网络][1]

## 环境设置
此示例中，有一个订阅包含以下项：

- 三个云服务：“SecSvc001”、“FrontEnd001”和“BackEnd001”
- 一个虚拟网络“CorpNetwork”，其中包含三个子网：“SecNet”、“FrontEnd”和“BackEnd”
- 一个与 SecNet 子网连接的网络虚拟设备（在本示例中为防火墙）
- 一个代表应用程序 Web 服务器的 Windows Server（“IIS01”）
- 两个代表应用程序后端服务器的 Windows Server（“AppVM01”、“AppVM02”）
- 一个代表 DNS 服务器的 Windows Server（“DNS01”）

下面的“参考”部分提供了可用于构建上述大多数环境的 PowerShell 脚本。尽管该示例脚本还完成了 VM 和虚拟网络的构建，但本文未对其进行详细描述。

构建环境：

  1.	保存“参考”部分中包含的网络配置 xml 文件（更新名称、位置和 IP 地址以符合给定的方案）
  2.	更新脚本中的用户变量，以符合用于运行脚本的环境（订阅、服务名称等）
  3.	在 PowerShell 中执行脚本

**注意**：PowerShell 脚本中提到的区域必须与网络配置 xml 文件中提到的区域相匹配。

成功运行脚本后，可执行以下脚本后续步骤：

1.	设置防火墙规则，下面的“防火墙规则描述”部分中做了介绍。
2.	（可选）“参考”部分中提供了两个脚本，用于设置 Web 服务器和包含简单 Web 应用程序的应用服务器，以便能使用此外围网络配置进行测试。

成功运行脚本后，需要完成防火墙规则。“防火墙规则”部分中对此做了介绍。

## 用户定义的路由 (UDR)
默认情况下，以下系统路由定义为：

        Effective routes : 
         Address Prefix    Next hop type    Next hop IP address Status   Source     
         --------------    -------------    ------------------- ------   ------     
         {10.0.0.0/16}     VNETLocal                            Active   Default    
         {0.0.0.0/0}       Internet                             Active   Default    
         {10.0.0.0/8}      Null                                 Active   Default    
         {100.64.0.0/10}   Null                                 Active   Default    
         {172.16.0.0/12}   Null                                 Active   Default    
         {192.168.0.0/16}  Null                                 Active   Default

VNETLocal 始终是该特定网络的 VNet 的已定义地址前缀（也就是说，根据每个特定 VNet 的定义方式，不同的 VNet 有不同前缀）。其余系统路由是静态的，上面列出了其默认值。

在优先级方面，由于路由通过最长前缀匹配 (LPM) 方法进行处理，因此路由表中最具体的路由将应用于给定的目标地址。

因此，发往局域网 (10.0.0.0/16) 的流量（例如发往 DNS01 服务器 10.0.2.4）将由于 10.0.0.0/16 路由而穿过 VNet 路由到目标。换而言之，对于 10.0.2.4，10.0.0.0/16 路由是最具体的路由，尽管 10.0.0.0/8 和 0.0.0.0/0 也可能适用，但它们较不具体，因此不影响此流量。发往 10.0.2.4 的流量有本地 VNet 的下一跃点，因而直接路由到目标。

例如，如果流量发往 10.1.1.1，则 10.0.0.0/16 路由就不适用，而 10.0.0.0/8 最具体的，因此将丢弃此流量（“黑洞化”），因为下一跃点为 Null。

如果目标不适用于任何 Null 前缀或 VNETLocal 前缀，则遵循最不具体路由 0.0.0.0/0 并路由出 Internet 作为下一跃点，因此将发送到 Azure 的 Internet 边缘之外。

如果路由表中有两个相同的前缀，则根据路由的“source”属性，其优先顺序如下：

1.	<blank> = 手动添加到路由表中的用户定义路由
2.	“VPNGateway”= 由动态网络协议添加的动态路由（与混合网络配合使用时为 BGP），这些路由随时间改变，因为动态协议会自动反映对等网络中的更改
3.	“Default”= 上述路由表中所示的系统路由、本地 VNet 和静态条目。

>[AZURE.NOTE] 由于 Azure 虚拟网关上使用的动态路由相当复杂，因此用户定义的路由 (UDR) 和 ExpressRoute 在使用上存在限制。如果子网与提供 ExpressRoute 连接的 Azure 网关通信，则不应该应用 UDR。此外，Azure 网关不能是用于其他 UDR 绑定子网的 NextHop 设备。将来的 Azure 版本将启用完全集成 UDR 和 ExpressRoute 的功能。

#### 创建本地路由

本示例需要两个路由表，分别用于前端和后端子网。已在每个表中加载了适用于给定子网的静态路由。对于本示例而言，每个数据表包含三个路由：

1. 定义了不带下一跃点的本地子网流量以允许本地子网流量绕过防火墙
2. 下一跃点定义为防火墙的虚拟网络流量，这会覆盖允许本地 VNet 流量直接路由的默认规则
3. 将带有下一跃点的所有剩余流量 (0/0) 定义为防火墙

路由表在创建后会绑定到其子网。前端子网路由表在创建并绑定到子网后应如下所示：

        Effective routes : 
         Address Prefix    Next hop type    Next hop IP address Status   Source     
         --------------    -------------    ------------------- ------   ------     
         {10.0.1.0/24}     VNETLocal                            Active 
		 {10.0.0.0/16}     VirtualAppliance 10.0.0.4            Active    
         {0.0.0.0/0}       VirtualAppliance 10.0.0.4            Active


本示例将使用以下命令来构建路由表、添加用户定义的路由，然后将路由表绑定到子网（注意：下面以货币符号开头的项（例如 $BESubnet）均为本文“参考”部分的脚本中的用户定义变量）：

1.	首先必须创建基础路由表。此代码段演示如何创建后端子网的路由表。该脚本还为前端子网创建了对应的路由表。

		New-AzureRouteTable -Name $BERouteTableName `
		    -Location $DeploymentLocation `
		    -Label "Route table for $BESubnet subnet"

2.	创建路由表后，可以添加特定的用户定义路由。在此代码段中，将通过虚拟设备路由所有流量 (0.0.0.0/0)（前面在脚本中创建虚拟设备时，已使用变量 $VMIP[0] 传入了分配的 IP 地址）。该脚本还在前端路由表中创建了对应的规则。

		Get-AzureRouteTable $BERouteTableName |`
		    Set-AzureRoute -RouteName "All traffic to FW" -AddressPrefix 0.0.0.0/0 `
		    -NextHopType VirtualAppliance `
		    -NextHopIpAddress $VMIP[0]

3. 上述路由条目将覆盖默认的“0.0.0.0/0”路由，但默认的 10.0.0.0/16 规则仍然存在，以允许 VNet 中的流量直接路由到目标，而不是路由到网络虚拟设备。若要纠正此行为，必须添加以下规则。

	    Get-AzureRouteTable $BERouteTableName | `
	        Set-AzureRoute -RouteName "Internal traffic to FW" -AddressPrefix $VNetPrefix `
	        -NextHopType VirtualAppliance `
	        -NextHopIpAddress $VMIP[0]

4. 此时要做出选择。在上述两个路由中，所有流量都会路由到防火墙进行评估，甚至单个子网中的流量也是如此。这可能是你想要的结果，但若要允许子网中的流量直接在本地路由而不要防火墙的介入，则你可以添加第三个特定规则。此路由指明，目标为本地子网的地址可以直接路由到该位置 (NextHopType = VNETLocal)。

	    Get-AzureRouteTable $BERouteTableName | `
	        Set-AzureRoute -RouteName "Allow Intra-Subnet Traffic" -AddressPrefix $BEPrefix `
	        -NextHopType VNETLocal

5.	最后，在创建路由表并填入用户定义的路由后，必须立即将路由表绑定到子网。在脚本中，前端路由表还绑定到了前端子网。下面是后端子网的绑定脚本。

		Set-AzureSubnetRouteTable -VirtualNetworkName $VNetName `
		   -SubnetName $BESubnet `
		   -RouteTableName $BERouteTableName

## IP 转发
UDR 随附 IP 转发功能。这是虚拟设备上的一项设置，使虚拟设备能够接收不是要专门传送到该设备的流量，再将流量转发到其最终目标。

例如，如果来自 AppVM01 的流量对 DNS01 服务器发出请求，UDR 会将此流量路由到防火墙。在启用 IP 转发后，目标为 DNS01 (10.0.2.4) 的流量被设备 (10.0.0.4) 所接受，然后转发到其最终目标 (10.0.2.4)。如果防火墙上未启用 IP 转发，则即使路由表将防火墙用作下一跃点，流量也不会被设备所接受。

>[AZURE.IMPORTANT] 必须记得一同启用 IP 转发和用户定义的路由。

设置 IP 转发是单个命令，可在创建 VM 时运行。在本示例的流程中，这个代码段靠近脚本末尾处，与 UDR 命令放在一起：

1.	调用代表虚拟设备的 VM 实例（在本例中为防火墙），并启用 IP 转发（注意：以货币符号开头的任何红色项（例如 $VMName[0]）均为本文“参考”部分的脚本中的用户定义变量。以方括号括住的零 [0] 代表 VM 阵列中的第一个 VM，为了使示例脚本无须修改即可运行，第一个 VM (VM 0) 必须是防火墙）：

		Get-AzureVM -Name $VMName[0] -ServiceName $ServiceName[0] | `
		   Set-AzureIPForwarding -Enable

## 网络安全组 (NSG)
此示例将构建一个 NSG 组，然后加载单个规则。此组接着只绑定到前端和后端子网（不绑定到 SecNet）。以声明性的方式构建以下规则：

1.	拒绝从 Internet 到整个 VNet（所有子网）的任何流量（所有端口）

尽管本示例使用了 NSG，但它的主要用途是作为防止人为配置错误的第二道防线。我们想要阻止由 Internet 流往前端或后端子网的所有入站流量，流量只应流经 SecNet 子网前往防火墙（并于合适时流往前端或后端子网）。此外，在配置 UDR 规则后，确实能够流往前端或后端子网的流量都将被定向到防火墙（得益于 UDR）。防火墙将此流量视为非对称流量，并且会丢弃出站流量。因此，前端和后端子网共有三个安全防护层：1) FrontEnd001 和 BackEnd001 云服务上没有开放的终结点，2) NSG 拒绝来自 Internet 的流量，3) 防火墙丢弃非对称流量。

本示例中的网络安全组有一个有趣的特点，那就是它只包含一个规则（如下所示），用于拒绝由 Internet 流往整个虚拟网络（包含安全子网）的流量。

	Get-AzureNetworkSecurityGroup -Name $NSGName | `
	    Set-AzureNetworkSecurityRule -Name "Isolate the $VNetName VNet `
	    from the Internet" `
	    -Type Inbound -Priority 100 -Action Deny `
	    -SourceAddressPrefix INTERNET -SourcePortRange '*' `
	    -DestinationAddressPrefix VIRTUAL_NETWORK `
	    -DestinationPortRange '*' `
	    -Protocol *

不过，由于 NSG 只绑定到前端和后端子网，因此不对流往安全子网的入站流量处理此规则。这样，即使 NSG 规则指出因为 NSG 永远不绑定到安全子网，所以没有由 Internet 流往 VNet 上任何地址的流量，流量也还是会流向安全子网。

	Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName `
	    -SubnetName $FESubnet -VirtualNetworkName $VNetName
	
	Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName `
	    -SubnetName $BESubnet -VirtualNetworkName $VNetName

## 防火墙规则
需要在防火墙上创建转发规则。由于防火墙会阻止或转发所有入站、出站和 VNet 内部流量，因此需要配置许多防火墙规则。此外，所有入站流量都抵达安全服务的公共 IP 地址（在不同端口上），并由防火墙进行处理。最佳实践是先绘制逻辑流量图，再设置子网和防火墙规则，以避免事后进行修改。下图是此示例中防火墙规则的逻辑视图：
 
![防火墙规则的逻辑视图][2]

>[AZURE.NOTE] 根据所用的网络虚拟设备，管理端口会有所不同。本示例中提到的 Barracuda NextGen 防火墙使用端口 22、801 和 807。请参阅设备供应商的文档来查找用于管理所用设备的确切端口。

### 逻辑规则描述
安全子网上只有防火墙这个资源，因此上述逻辑视图未显示该子网，而且这个视图显示防火墙规则以及这些规则在逻辑上是如何允许或拒绝流量的流动，而不显示实际的路由路径。此外，针对 RDP 流量选择的外部端口均为较高范围的端口 (8014-8026)，并且选择时在一定程度上遵循了本地 IP 地址的最后两个八位字节以方便阅读（例如，本地服务器地址 10.0.1.4 与外部端口 8014 相关联），不过可以使用任何更高范围的端口，只要端口不冲突即可。

在此示例中，我们需要 7 种类型的规则，下面描述了这些规则类型：

- 外部规则（针对入站流量）：
  1.	防火墙管理规则：此应用重定向规则允许流量传递到网络虚拟设备的管理端口。
  2.	RDP 规则（针对每个 Windows 服务器）：这四个规则（每台服务器一个）允许通过 RDP 管理单个服务器。根据所用的网络虚拟设备功能，也可将其捆绑为一个规则。
  3.	应用程序流量规则：应用程序流量规则有两个，第一个针对前端 Web 流量，第二个针对后端流量（例如 Web 服务器流往数据层）。这些规则的配置取决于网络体系结构（服务器的放置位置）和流量流动行为（流量的流动方向，以及使用的端口）。
      - 第一个规则允许实际的应用程序流量抵达应用程序服务器。其他规则所考虑的是安全、管理等方面的事项，应用程序规则用于允许外部用户或服务访问应用程序。就本示例而言，端口 80 上有单个 Web 服务器，因此单个防火墙应用程序规则将流往外部 IP 的入站流量，并重定向到 Web 服务器的内部 IP 地址。重定向的流量会话经过 NAT 后流往内部服务器。
      - 第二个应用程序流量规则是后端规则，用于允许 Web 服务器通过任何端口与 AppVM01 服务器（而非 AppVM02）对话。
- 内部规则（针对 VNet 内部流量）
  4.	出站到 Internet 规则：此规则允许来自任何网络的流量传递到选定的网络。此规则通常是防火墙上已有的但处于禁用状态的默认规则。对于本示例，应启用此规则。
  5.	DNS 规则：此规则只允许 DNS（端口 53）流量传递到 DNS 服务器。在此环境中，阻止了大部分由前端流往后端的流量，而此规则专门允许来自任何本地子网的 DNS。
  6.	子网到子网规则：此规则允许后端子网上的任何服务器连接到前端子网上的任何服务器（但不允许反向连接）。
- 防故障规则（针对不符合上述任一规则的流量）：
  7.	拒绝所有流量规则：请始终将此规则用作最终规则（在优先级方面），这样，如果流量的流动行为无法符合上述任何规则，此规则就会将其丢弃。这是默认规则且通常已激活，一般而言并不需要修改。

>[AZURE.TIP] 为简化本示例，允许第二个应用程序流量规则上的任何端口，但在真实方案中，则应使用具体的端口和地址范围，以降低此规则的攻击面。

<br />

>[AZURE.IMPORTANT] 创建好上述所有规则后，请务必检查每个规则的优先级，以确保可根据需要允许或拒绝流量。在本示例中，规则设置了优先顺序。如果规则顺序错误，则很容易被锁定在防火墙外面。至少应该确保防火墙本身的管理绝对属于优先级最高的规则。

### 规则先决条件
运行防火墙的虚拟机的先决条件之一是公共终结点。为了使防火墙能够处理流量，必须开放相应的公共终结点。本示例包含三种类型的流量：1) 管理流量，用于控制防火墙和防火墙规则；2) RDP 流量，用于控制 Windows 服务器；3) 应用程序流量。这些流量是上述防火墙规则的逻辑视图上半部分中的三排流量类型。

>[AZURE.IMPORTANT] 本内容的重点是要记住**所有**流量都会通过防火墙传送。通过远程桌面访问 IIS01 服务器也是如此，即使是在前端云服务和前端子网上，若要访问此服务器，也仍然需要通过 RDP 连接到端口 8014 上的防火墙，然后允许防火墙在内部将 RDP 请求路由到 IIS01 RDP 端口。由于没有直接通往 IIS01 的 RDP 路径（就经典管理门户上所见），Azure 经典管理门户的“连接”按钮将不起作用。这意味着，所有来自 Internet 的连接将连往安全服务和端口，例如 secscv001.chinacloudapp.cn:xxxx（请参考上图中外部端口与内部 IP 和端口的映射）。

可以在创建 VM 或后续构建时开放终结点，示例脚本和以下代码段中演示了具体的操作（注意：以货币符号开头的任何项（例如 $VMName[$i]）均为本文“参考”部分的脚本中的用户定义变量。以方括号括住的“$i”[$i] 代表 VM 阵列中特定 VM 的阵列编号）：

	Add-AzureEndpoint -Name "HTTP" -Protocol tcp -PublicPort 80 -LocalPort 80 `
	    -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | `
	    Update-AzureVM

尽管此处因为使用了变量而未明确显示，但实际上**只会**打开安全云服务上的终结点。这是为了确保所有入站流量都将由防火墙处理（路由、进行 NAT 处理、丢弃）。

电脑上必须安装管理客户端才能管理防火墙和创建所需的配置。有关如何管理设备的信息，请参阅防火墙（或其他 NVA）供应商提供的文档。本部分的余下内容和下一部分“创建防火墙规则”将介绍如何通过供应商的管理客户端（即，不使用 Azure 经典管理门户或 PowerShell）来配置防火墙本身。

有关下载客户端和连接到本示例所用 Barracuda 的说明，可在以下位置找到：[Barracuda NG Admin](https://techlib.barracuda.com/NG61/NGAdmin)

在登录防火墙之后、创建防火墙规则之前，你可以借助以下两个必备的对象类来方便创建规则：网络对象和服务对象。

在本示例中，应定义三个命名网络对象（前端子网和后端子网各一个，还有一个网络对象对应于 DNS 服务器的 IP 地址）。若要创建命名网络，请先访问 Barracuda NG Admin 客户端仪表板，导航到“配置”选项卡，在“操作配置”部分中单击“规则集”，单击“防火墙对象”菜单下面的“网络”，然后在“编辑网络”菜单中单击“新建”。现在，可以通过添加名称和前缀来创建网络对象：

![创建前端网络对象][3]
 
这将会创建前端子网的命名网络，同时为后端子网创建类似的对象。现在你可以更轻松地在防火墙规则中按名称引用子网。

对于 DNS 服务器对象：

![创建 DNS 服务器对象][4]

本文稍后将在 DNS 规则中使用这个 IP 地址引用。

第二个必备对象是服务对象。这些对象代表每台服务器的 RDP 连接端口。由于现有 RDP 服务对象已绑定到默认 RDP 端口 3389，因此可以创建新服务以允许来自外部端口 (8014-8026) 的流量。还可将新端口添加到现有的 RDP 服务，但为了便于演示，我们可为每台服务器创建一个规则。若要为服务器创建新的 RDP 规则，请先访问 Barracuda NG Admin 客户端仪表板，导航到“配置”选项卡，在“操作配置”部分中单击“规则集”，单击“防火墙对象”菜单下面的“服务”，向下滚动服务列表，然后选择“RDP”服务。右键单击并选择“复制”，然后右键单击并选择“粘贴”。现已创建一个可编辑的 RDP-Copy1 服务对象。右键单击 RDP-Copy1 并选择“编辑”，此时将弹出如下所示的“编辑服务对象”窗口：

![默认 RDP 规则的副本][5]

接下来，可以编辑代表特定服务器 RDP 服务的值。对于 AppVM01，应该修改上述默认 RDP 规则以反映图 8 的图表中所用的新服务名称、说明和外部 RDP 端口（注意：端口已从 RDP 默认值 3389 更改为要用于此特定服务器的外部端口，在 AppVM01 示例中，外部端口为 8025）。修改后的服务如下所示：

![AppVM01 规则][6]
 
必须重复此过程来创建其余服务器的 RDP 服务：AppVM02、DNS01 和 IIS01。创建这些服务可让下一部分中的规则创建操作变得更简单明了。

>[AZURE.NOTE] 防火墙不需要 RDP 服务的原因有两个：1) 首先，防火墙 VM 是基于 Linux 的映像，因此将在端口 22 上使用 SSH 来管理 VM，而不是使用 RDP，2) 下面所述的第一个管理规则允许端口 22 和另外两个管理端口，以允许进行管理连接。

### 创建防火墙规则
本示例使用三种类型的防火墙规则，它们各有不同的图标：

应用程序重定向规则：![应用程序重定向图标][7]

目标 NAT 规则：![目标 NAT 图标][8]

传递规则：![传递图标][9]

可以在 Barracuda 网站中找到有关这些规则的详细信息。

若要创建以下规则（或验证现有的默认规则），请先访问 Barracuda NG Admin 客户端仪表板，导航到“设置”选项卡，在“操作配置”部分中单击“规则集”。此时将出现名为“主规则”的网格，其中显示了此防火墙的现有活动规则和已停用规则。此网格右上角有一个绿色“+”小按钮，单击此按钮即可创建新规则（注意：防火墙可能会“禁止”更改，如果你看到标记为“锁定”的按钮且无法创建或编辑规则，请单击此按钮以“解除锁定”规则集并允许编辑）。如果你想要编辑现有规则，请选择该规则，右键单击并选择“编辑规则”。

创建和/或修改规则后，必须将其推送到防火墙并激活，如果不这么做，规则更改将不会生效。下面的详细规则说明描述了推送和激活过程。

完成本示例所需的每个规则的具体说明如下：

- **防火墙管理规则**：此应用重定向规则允许流量传递到网络虚拟设备的管理端口，在本示例中为 Barracuda NextGen 防火墙。管理端口是 801 和 807，以及 22（可选）。外部和内部端口相同（即没有端口转换）。此规则 (SETUP-MGMT-ACCESS) 是默认规则，已按默认启用（在 Barracuda NextGen 防火墙版本 6.1 中）。

	![防火墙管理规则][10]

>[AZURE.TIP] 此规则中的源地址空间是“任何”，如果管理 IP 地址范围已知，则减少此范围还可降低管理端口的攻击面。

- **RDP 规则**：这些目标 NAT 规则将允许通过 RDP 管理单个服务器。
创建此规则需要四个关键字段：
  1.	源 – 允许从任何位置进行 RDP 访问，“源”字段中使用了“任何”引用值。
  2.	服务 – 使用前面创建的相应服务对象，在本例中为“AppVM01 RDP”；外部端口将重定向到服务器本地 IP 地址和端口 3386（默认 RDP 端口）。此特定规则用于对 AppVM01 进行 RDP 访问。
  3.	目标 – 应该是*防火墙上的本地端口*“DCHP 1 本地 IP”，如果使用静态 IP，则为 eth0。如果网络设备有多个本地接口，序号（eth0、eth1 等）可能不同。这是防火墙用于向外发送的端口（可能与接收端口相同），实际的路由目标在“目标列表”字段中列出。
  4.	重定向 – 此部分告知虚拟设备最终要将此流量重定向到何处。最简单的重定向是在“目标列表”字段中放置 IP 和端口（可选）。如果不使用端口，则使用入站请求中的目标端口（即未转换）；如果指定了端口，端口将连同 IP 地址一起进行 NAT 处理。

	![防火墙 RDP 规则][11]

    总共需要创建四个 RDP 规则：

    | 规则名称 | 服务器 | 服务 | 目标列表 |
    |----------------|---------|-------------|---------------|
    | RDP-to-IIS01 | IIS01 | IIS01 RDP | 10\.0.1.4:3389 |
    | RDP-to-DNS01 | DNS01 | DNS01 RDP | 10\.0.2.4:3389 |
    | RDP-to-AppVM01 | AppVM01 | AppVM01 RDP | 10\.0.2.5:3389 |
    | RDP-to-AppVM02 | AppVM02 | AppVm02 RDP | 10\.0.2.6:3389 |
  
>[AZURE.TIP] 缩减“源”和“服务”字段的范围可降低攻击面。请使用可让功能正常运行的最小范围。

- **应用程序流量规则**：应用程序流量规则有两个，第一个针对前端 Web 流量，第二个针对后端流量（例如 Web 服务器流往数据层）。这些规则取决于网络体系结构（服务器的放置位置）和流量流动行为（流量的流动方向，以及使用的端口）。

	首先讨论 Web 流量的前端规则：

	![防火墙 Web 规则][12]
 
	此目标 NAT 规则允许实际的应用程序流量抵达应用程序服务器。其他规则所考虑的是安全、管理等方面的事项，应用程序规则用于允许外部用户或服务访问应用程序。就本示例而言，端口 80 上有单个 Web 服务器，因此单个防火墙应用程序规则将流往外部 IP 的入站流量，并重定向到 Web 服务器的内部 IP 地址。

	**注意**：“目标列表”字段中未指定端口，因此入站端口 80（对于所选服务为 443）将用于 Web 服务器的重定向。如果 Web 服务器正在侦听不同的端口（例如端口 8080），“目标列表”字段可更新为“10.0.1.4:8080”以同时允许端口重定向。

	下一个应用程序流量规则是后端规则，用于允许 Web 服务器通过任何服务与 AppVM01 服务器（而非 AppVM02）对话：

	![防火墙 AppVM01 规则][13]

	此传递规则允许前端子网上的任何 IIS 服务器在任何端口上连接到 AppVM01（IP 地址 10.0.2.5），使用任何协议来访问 Web 应用程序所需的数据。

	在此屏幕截图中，“目标”字段使用“<explicit-dest>”来表示目标为 10.0.2.5。这可能是如图所示的明确地址或命名网络对象（如 DNS 服务器先决条件中所述）。至于使用哪种表示法，将由防火墙管理员来决定。若要将 10.0.2.5 添加为明确目标，请双击 <explicit-dest> 下面的第一个空白行，然后在弹出窗口中输入地址。

    使用此传递规则时不需要 NAT，因为这是内部流量，“连接方法”可设置为“不使用 SNAT”。

	**注意**：此规则的“源”网络是前端子网上的任何资源，如果只有一个或已知特定数目的 Web 服务器，则可以将网络对象资源创建为更与这些确切 IP 地址相关，而不是针对整个前端子网。

>[AZURE.TIP] 此规则使用“任何”服务让你更轻松地设置和使用示例应用程序，此外，还允许在单个规则中使用 ICMPv4 (ping)。不过，这不是建议的做法。端口和协议（“服务”）应尽可能缩小到允许应用程序操作的程度，以降低跨越此边界的攻击面。

<br />

>[AZURE.TIP] 尽管此规则显示正在使用 explicit-dest 引用，但整个防火墙配置应使用一致的方法。建议在整个配置中使用命名网络对象，以方便阅读和支持。在此处使用 explicit-dest 只是为了显示备选的引用方法，一般并不建议这样做（尤其是在复杂配置中）。

- **出站到 Internet 规则**：此传递规则允许来自任何源网络的流量传递到选定的目标网络。此规则通常是 Barracuda NextGen 防火墙上已有的但处于禁用状态的默认规则。右键单击此规则可以访问“激活规则”命令。此处所示的规则已经过修改，添加了本文先决条件部分中为了参考而创建的，连接到此规则的“源”属性的两个本地子网。

	![防火墙出站规则][14]

- **DNS 规则**：此传递规则只允许 DNS（端口 53）流量传递到 DNS 服务器。在此环境中，阻止了大部分由前端流往后端的流量，而此规则专门允许 DNS。

	![防火墙 DNS 规则][15]

	**注意**：此屏幕截图中包含了“连接方法”。此规则用于内部 IP 到内部 IP 的地址流量，不需要 NAT，因此传递规则的“连接方法”设置为“不使用 SNAT”。

- **子网到子网规则**：此传递规则是经过激活和修改的默认规则，允许后端子网上的任何服务器连接到前端子网上的任何服务器。此规则完全针对内部流量，因此可将“连接方法”设置为“不使用 SNAT”。

	![防火墙 VNet 内部规则][16]

	**注意**：未选中“双向”复选框（也未签入大部分规则），这一点对于此规则很重要，因为这可以让此规则只能“单向”进行，即，可以从后端子网连接到前端网络，但不能反向连接。如果选中该复选框，此规则将启用双向流量，从逻辑图来看并不需要这么做。

- **拒绝所有流量规则**：请始终将此规则用作最终规则（在优先级方面），这样，如果流量的流动行为无法符合上述任何规则，此规则就会将其丢弃。这是默认规则且通常已激活，一般而言并不需要修改。

	![防火墙拒绝规则][17]

>[AZURE.IMPORTANT] 创建好上述所有规则后，请务必检查每个规则的优先级，以确保可根据需要允许或拒绝流量。在本示例中，规则的排列顺序是它们应在 Barracuda 管理客户端的转发规则主网格中显示的顺序。

## 规则激活
根据逻辑图中的规范修改规则集后，必须将规则集上载到防火墙并激活。

![防火墙规则激活][18]
 
管理客户端右上角是按钮群集。单击“发送更改”按钮将修改后的规则发送到防火墙，然后单击“激活”按钮。
 
激活防火墙规则集后，此示例环境即构建完成。

## 流量方案
>[AZURE.IMPORTANT] 本部分的重点是要记住**所有**流量都会通过防火墙传送。通过远程桌面访问 IIS01 服务器也是如此，即使是在前端云服务和前端子网上，若要访问此服务器，也仍然需要通过 RDP 连接到端口 8014 上的防火墙，然后允许防火墙在内部将 RDP 请求路由到 IIS01 RDP 端口。由于没有直接通往 IIS01 的 RDP 路径（就经典管理门户上所见），Azure 经典管理门户的“连接”按钮将不起作用。这意味着，所有来自 Internet 的连接将连往安全服务和端口，例如 secscv001.chinacloudapp.cn:xxxx。

对于这些方案，应准备好以下防火墙规则：

1.	防火墙管理
2.	通过 RDP 访问 IIS01
3.	通过 RDP 访问 DNS01
4.	通过 RDP 访问 AppVM01
5.	通过 RDP 访问 AppVM02
6.	到 Web 的应用流量
7.	到 AppVM01 的应用流量
8.	出站到 Internet
9.	前端到 DNS01
10.	子网内部流量（仅限后端到前端）
11.	全部拒绝

实际的防火墙规则集很可能包含上述规则以外的其他许多规则，任何给定防火墙上的规则还包含与此处所列编号不同的优先级编号。此列表和关联的编号只是提供这 11 个规则之间的相关性，以及它们彼此之间的相对优先级。换而言之，在实际的防火墙上，“通过 RDP 访问 IIS01”可能是规则编号 5，但只要它低于“防火墙管理”规则并高于“通过 RDP 访问 DNS01”规则，就符合此列表的目的。该列表还有助于使以下方案变得简单明了，例如“转发规则 9 (DNS)”。同样为了简单起见，当流量方案与 RDP 无关时，四个 RDP 规则将统称为“RDP 规则”。

另请记住，已创建了网络安全组用于前端和后端子网上的入站 Internet 流量。

#### （允许）从 Internet 访问 Web 服务器
1.	Internet 用户从 SecSvc001.CloudApp.Net（面向 Internet 的云服务）请求 HTTP 页面
2.	云服务通过端口 80 上的开放终结点将流量传递到 10.0.0.4:80 上的防火墙接口
3.	未对安全子网分配 NSG，因此系统 NSG 规则允许流量发往防火墙
4.	流量抵达防火墙的内部 IP 地址 (10.0.1.4)
5.	防火墙开始处理规则：
  1.	转发规则 1 (FW Mgmt) 不适用，将转到下一规则
  2.	转发规则 2-5（RDP 规则）不适用，将转到下一规则
  3.	转发规则 6（应用：Web）适用，允许流量，防火墙通过 NAT 将流量发送到 10.0.1.4 (IIS01)
6.	前端子网开始处理入站规则：
  1.	NSG 规则 1（阻止 Internet）不适用（此流量由防火墙进行 NAT 处理，因此源地址现在是位于安全子网上的防火墙，被前端子网 NSG 认为是“本地”流量，因此受到允许），将转到下一个规则
  2.	默认 NSG 规则允许子网到子网的流量，允许流量，停止处理 NSG 规则
7.	IIS01 正在侦听 Web 流量，将接收此请求并开始处理请求
8.	IIS01 尝试在后端子网上启动连往 AppVM01 的 FTP 会话
9.	前端子网上的 UDR 路由使防火墙成为下一跃点
10.	前端子网上没有出站规则，允许流量
11.	防火墙开始处理规则：
  1.	转发规则 1 (FW Mgmt) 不适用，将转到下一规则
  2.	转发规则 2-5（RDP 规则）不适用，将转到下一规则
  3.	转发规则 6（应用：Web）不适用，将转到下一规则
  4.	转发规则 7（应用：后端）适用，允许流量，防火墙将流量转发到 10.0.2.5 (AppVM01)
12.	后端子网开始处理入站规则：
  1.	NSG 规则 1（阻止 Internet）不适用，将转到下一规则
  2.	默认 NSG 规则允许子网到子网的流量，允许流量，停止处理 NSG 规则
13.	 AppVM01 接收请求，启动会话并做出响应
14.	后端子网上的 UDR 路由使防火墙成为下一跃点
15.	后端子网上没有出站 NSG 规则，因此允许响应
16.	由于这会返回已建立的会话上的流量，防火墙将响应传回给 Web 服务器 (IIS01)
17.	前端子网开始处理入站规则：
  1.	NSG 规则 1（阻止 Internet）不适用，将转到下一规则
  2.	默认 NSG 规则允许子网到子网的流量，允许流量，停止处理 NSG 规则
18.	IIS 服务器接收响应，完成 AppVM01 的事务，然后完成构建 HTTP 响应，此 HTTP 响应将发送给请求方
19.	前端子网上没有出站 NSG 规则，因此允许响应
20.	HTTP 响应抵达防火墙，由于这是对已建立的 NAT 会话的响应，因此被防火墙接受
21.	防火墙随后将响应重定向回给 Internet 用户
22.	前端子网上没有出站 NSG 规则或 UDR 跃点，因此允许响应，Internet 用户将收到请求的网页。

#### （允许）通过 RDP 从 Internet 访问后端
1.	Internet 上的服务器管理员请求通过 SecSvc001.CloudApp.Net:8025 启动 AppVM01 的 RDP 会话，其中 8025 是用户针对“通过 RDP 访问 AppVM01”防火墙规则分配的端口号
2.	云服务通过端口 8025 上的开放终结点将流量传递到 10.0.0.4:8025 上的防火墙接口
3.	未对安全子网分配 NSG，因此系统 NSG 规则允许流量发往防火墙
4.	防火墙开始处理规则：
  1.	转发规则 1 (FW Mgmt) 不适用，将转到下一规则
  2.	转发规则 2 (RDP IIS) 不适用，将转到下一规则
  3.	转发规则 3 (RDP DNS01) 不适用，将转到下一规则
  4.	转发规则 4 (RDP AppVM01) 适用，允许流量，防火墙通过 NAT 将流量发送到 10.0.2.5:3386（AppVM01 上的 RDP 端口）
5.	后端子网开始处理入站规则：
  1.	NSG 规则 1（阻止 Internet）不适用（此流量由防火墙进行 NAT 处理，因此源地址现在是位于安全子网上的防火墙，被后端子网 NSG 认为是“本地”流量，因此受到允许），将转到下一个规则
  2.	默认 NSG 规则允许子网到子网的流量，允许流量，停止处理 NSG 规则
6.	AppVM01 侦听 RDP 流量并做出响应
7.	由于没有出站 NSG 规则，将应用默认规则并允许返回流量
8.	UDR 将出站流量路由到作为下一跃点的防火墙
9.	由于这会返回已建立的会话上的流量，防火墙将响应传回给 Internet 用户
10.	已启用 RDP 会话
11.	AppVM01 提示输入用户名和密码

#### （允许）在 DNS 服务器上执行 Web 服务器 DNS 查找
1.	Web 服务器 IIS01 需要 www.data.gov 上的数据源，但需要解析地址。
2.	VNet 的网络配置将 DNS01（后端子网上的 10.0.2.4）列为主 DNS 服务器，IIS01 将 DNS 请求发送到 DNS01
3.	UDR 将出站流量路由到作为下一跃点的防火墙
4.	没有出站 NSG 规则绑定到前端子网，允许流量
5.	防火墙开始处理规则：
  1.	转发规则 1 (FW Mgmt) 不适用，将转到下一规则
  2.	转发规则 2-5（RDP 规则）不适用，将转到下一规则
  3.	转发规则 6-7（应用规则）不适用，将转到下一规则
  4.	转发规则 8（到 Internet）不适用，将转到下一规则
  5.	转发规则 9 (DNS) 适用，允许流量，防火墙将流量转发到 10.0.2.4 (DNS01)
6.	后端子网开始处理入站规则：
  1.	NSG 规则 1（阻止 Internet）不适用，将转到下一规则
  2.	默认 NSG 规则允许子网到子网的流量，允许流量，停止处理 NSG 规则
7.	DNS 服务器接收请求
8.	DNS 服务器没有缓存的地址，请求 Internet 上的根 DNS 服务器
9.	UDR 将出站流量路由到作为下一跃点的防火墙
10.	后端子网上没有出站 NSG 规则，允许流量
11.	防火墙开始处理规则：
  1.	转发规则 1 (FW Mgmt) 不适用，将转到下一规则
  2.	转发规则 2-5（RDP 规则）不适用，将转到下一规则
  3.	转发规则 6-7（应用规则）不适用，将转到下一规则
  4.	 转发规则 8（到Internet）适用，允许流量，通过 SNAT 将会话发送到 Internet 上的根 DNS 服务器
12.	Internet DNS 服务器做出响应，由于此会话是从防火墙发起的，因此响应被防火墙接受
13.	由于这是建立的会话，防火墙将响应转发到发起端服务器 DNS01
14.	后端子网开始处理入站规则：
  1.	NSG 规则 1（阻止 Internet）不适用，将转到下一规则
  2.	默认 NSG 规则允许子网到子网的流量，允许流量，停止处理 NSG 规则
15.	DNS 服务器接收并缓存响应，然后将初始请求响应给 IIS01
16.	后端子网上的 UDR 路由使防火墙成为下一跃点
17.	后端子网上没有出站 NSG 规则，允许流量
18.	这是防火墙上建立的会话，防火墙将响应转发回到 IIS 服务器
19.	前端子网开始处理入站规则：
  1.	后端子网到前端子网的入站流量没有适用的 NSG 规则，因此不会应用任何 NSG 规则
  2.	允许子网间流量的默认系统规则允许此流量，因此允许流量
20.	IIS01 从 DNS01 接收响应

#### （允许）后端服务器到前端服务器
1.	通过 RDP 登录 AppVM02 的管理员直接通过 Windows 文件资源管理器从 IIS01 服务器请求文件
2.	后端子网上的 UDR 路由使防火墙成为下一跃点
3.	后端子网上没有出站 NSG 规则，因此允许响应
4.	防火墙开始处理规则：
  1.	转发规则 1 (FW Mgmt) 不适用，将转到下一规则
  2.	转发规则 2-5（RDP 规则）不适用，将转到下一规则
  3.	转发规则 6-7（应用规则）不适用，将转到下一规则
  4.	转发规则 8（到 Internet）不适用，将转到下一规则
  5.	转发规则 9 (DNS) 不适用，将转到下一规则
  6.	转发规则 10（子网内部）适用，允许流量，防火墙将流量传递到 10.0.1.4 (IIS01)
5.	前端子网开始处理入站规则：
  1.	NSG 规则 1（阻止 Internet）不适用，将转到下一规则
  2.	默认 NSG 规则允许子网到子网的流量，允许流量，停止处理 NSG 规则
6.	假设有适当的身份验证和授权，IIS01 将接受请求和响应
7.	前端子网上的 UDR 路由使防火墙成为下一跃点
8.	前端子网上没有出站 NSG 规则，因此允许响应
9.	这是防火墙上现有的会话，因此允许此响应，防火墙将响应返回给 AppVM02
10.	后端子网开始处理入站规则：
  1.	NSG 规则 1（阻止 Internet）不适用，将转到下一规则
  2.	默认 NSG 规则允许子网到子网的流量，允许流量，停止处理 NSG 规则
11.	AppVM02 接收响应

#### （拒绝）从 Internet 直接访问 Web 服务器
1.	Internet 用户尝试通过 FrontEnd001.CloudApp.Net 服务访问 Web 服务器 IIS01
2.	由于没有为 HTTP 流量开放终结点，此流量不会通过云服务抵达服务器
3.	如果出于某种原因而开放了终结点，前端子网上的 NSG（阻止 Internet）将阻止此流量
4.	最后，前端子网 UDR 路由将来自 IIS01 的任何出站流量发送到作为下一跃点的防火墙，防火墙将此流量视为非对称流量，并丢弃出站响应。因此，Internet 和 IIS01 之间至少有三个独立防御层，可通过其云服务防止未经授权/不当的访问。

#### （拒绝）通过 Internet 访问后端服务器
1.	Internet 用户尝试通过 BackEnd001.CloudApp.Net 服务访问 AppVM01 上的文件
2.	由于没有为文件共享开放终结点，此流量不会通过云服务抵达服务器
3.	如果出于某种原因而开放了终结点，NSG（阻止 Internet）将阻止此流量
4.	最后，UDR 路由将来自 AppVM01 的任何出站流量发送到作为下一跃点的防火墙，防火墙将此流量视为非对称流量，并丢弃出站响应。因此，Internet 和 AppVM01 之间至少有三个独立防御层，可通过其云服务防止未经授权/不当的访问。

#### （拒绝）前端服务器到后端服务器
1.	假设 IIS01 遭到入侵，正在运行恶意代码以尝试扫描后端子网服务器。
2.	前端子网 UDR 路由将任何来自 IIS01 的出站流量发送到作为下一跃点的防火墙。遭到入侵的 VM 无法改变这一点。
3.	如果请求是针对 AppVM01 发出的，或者是为了执行 DNS 查找而对 DNS 服务器发出的，则防火墙可能会允许并处理该流量（由于转发规则 7 和 9）。其他所有流量将被转发规则 11（全部拒绝）阻止。
4.	如果防火墙上已启用高级威胁检测（本文未涵盖这方面的内容，请参阅供应商的文档，以了解特定网络设备的高级威胁功能），且流量包含带有高级威胁规则标志的已知签名或模式，即使是本文所述的基本转发规则所允许的流量也可能还会遭到阻止。

#### （拒绝）在 DNS 服务器上执行 Internet DNS 查找
1.	Internet 用户尝试通过 BackEnd001.CloudApp.Net 服务查找 DNS01 上的内部 DNS 记录 
2.	由于没有为 DNS 流量开放终结点，此流量不会通过云服务抵达服务器
3.	如果出于某种原因而开放了终结点，前端子网上的 NSG 规则（阻止 Internet）将阻止此流量
4.	最后，后端子网 UDR 路由将来自 DNS01 的任何出站流量发送到作为下一跃点的防火墙，防火墙将此流量视为非对称流量，并丢弃出站响应。因此，Internet 和 DNS01 之间至少有三个独立防御层，可通过其云服务防止未经授权/不当的访问。

#### （拒绝）从 Internet 通过防火墙访问 SQL
1.	Internet 用户从 SecSvc001.CloudApp.Net（面向 Internet 的云服务）请求 SQL 数据
2.	由于没有为 SQL 开放终结点，此流量不会通过云服务抵达防火墙
3.	如果出于某种原因而开放了 SQL 终结点，防火墙将开始处理规则：
  1.	转发规则 1 (FW Mgmt) 不适用，将转到下一规则
  2.	转发规则 2-5（RDP 规则）不适用，将转到下一规则
  3.	转发规则 6-7（应用程序规则）不适用，将转到下一规则
  4.	转发规则 8（到 Internet）不适用，将转到下一规则
  5.	转发规则 9 (DNS) 不适用，将转到下一规则
  6.	转发规则 10（子网内部）不适用，将转到下一规则
  7.	转发规则 11（全部拒绝）适用，阻止流量，停止处理规则


## 参考
### 主脚本和网络配置
将完整脚本保存在 PowerShell 脚本文件中。将网络配置保存到名为“NetworkConf2.xml”的文件中。
如有需要，请修改用户定义的变量。运行脚本，然后根据上面的防火墙规则设置说明操作。

#### 完整脚本
此脚本基于用户定义的变量执行以下操作：

1.	连接到 Azure 订阅
2.	新建存储帐户
3.	根据网络配置文件中的定义创建新的 VNet 和三个子网
4.	构建五个虚拟机：1 个防火墙和 4 个 Windows Server VM
5.	配置 UDR，包括：
  1.	创建两个新的路由表
  2.	将路由添加到表
  3.	将表绑定到相应的子网
6.	在 NVA 上启用 IP 转发
7.	配置 NSG，包括：
  1.	创建 NSG
  2.	添加规则
  3.	将 NSG 绑定到相应的子网

此 PowerShell 脚本应该在连接到 Internet 的电脑或服务器上本地运行。

>[AZURE.IMPORTANT] 此此脚本运行时，PowerShell 中可能会弹出警告或其他参考性消息。只有以红色字体显示的错误消息才需要引以关注。

	<# 
	 .SYNOPSIS
	  Example of DMZ and User Defined Routing in an isolated network (Azure only, no hybrid connections)
	
	 .DESCRIPTION
	  This script will build out a sample DMZ setup containing:
	   - A default storage account for VM disks
	   - Three new cloud services
	   - Three Subnets (SecNet, FrontEnd, and BackEnd subnets)
	   - A Network Virtual Appliance (NVA), in this case a Barracuda NextGen Firewall
	   - One server on the FrontEnd Subnet
	   - Three Servers on the BackEnd Subnet
	   - IP Forwading from the FireWall out to the internet
	   - User Defined Routing FrontEnd and BackEnd Subnets to the NVA
	
	  Before running script, ensure the network configuration file is created in
	  the directory referenced by $NetworkConfigFile variable (or update the
	  variable to reflect the path and file name of the config file being used).
	
	 .Notes
	  Everyone's security requirements are different and can be addressed in a myriad of ways.
	  Please be sure that any sensitive data or applications are behind the appropriate
	  layer(s) of protection. This script serves as an example of some of the techniques
	  that can be used, but should not be used for all scenarios. You are responsible to
	  assess your security needs and the appropriate protections needed, and then effectively
	  implement those protections.
	
	  Security Service (SecNet subnet 10.0.0.0/24)
	   myFirewall - 10.0.0.4
	 
	  FrontEnd Service (FrontEnd subnet 10.0.1.0/24)
	   IIS01      - 10.0.1.4
	 
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
	    $SecureService = "SecSvc001"
	    $FrontEndService = "FrontEnd001"
	    $BackEndService = "BackEnd001"
	
	  # Network Details
	    $VNetName = "CorpNetwork"
	    $VNetPrefix = "10.0.0.0/16"
	    $SecNet = "SecNet"
	    $FESubnet = "FrontEnd"
	    $FEPrefix = "10.0.1.0/24"
	    $BESubnet = "BackEnd"
	    $BEPrefix = "10.0.2.0/24"
	    $NetworkConfigFile = "C:\Scripts\NetworkConf3.xml"
	
	  # VM Base Disk Image Details
	    $SrvImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Windows Server 2012 R2 Datacenter'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}
	    $FWImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Barracuda NextGen Firewall'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}
	
	  # UDR Details
	    $FERouteTableName = "FrontEndSubnetRouteTable"
	    $BERouteTableName = "BackEndSubnetRouteTable"
	
	  # NSG Details
	    $NSGName = "MyVNetSG"
	
	# User Defined VM Specific Config
	    # Note: To ensure UDR and IP forwarding is setup
	    # properly this script requires VM 0 be the NVA.
	
	    # VM 0 - The Network Virtual Appliance (NVA)
	      $VMName += "myFirewall"
	      $ServiceName += $SecureService
	      $VMFamily += "Firewall"
	      $img += $FWImg
	      $size += "Small"
	      $SubnetName += $SecNet
	      $VMIP += "10.0.0.4"
	
	    # VM 1 - The Web Server
	      $VMName += "IIS01"
	      $ServiceName += $FrontEndService
	      $VMFamily += "Windows"
	      $img += $SrvImg
	      $size += "Standard_D3"
	      $SubnetName += $FESubnet
	      $VMIP += "10.0.1.4"
	
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
	
	If (Test-AzureName -Service -Name $SecureService) { 
	    Write-Host "The SecureService service name is already in use, please pick a different service name." -ForegroundColor Yellow
	    $FatalError = $true}
	Else { Write-Host "The FrontEndService service name is valid for use." -ForegroundColor Green}
	
	If (Test-AzureName -Service -Name $FrontEndService) { 
	    Write-Host "The FrontEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
	    $FatalError = $true}
	Else { Write-Host "The FrontEndService service name is valid for use" -ForegroundColor Green}
	
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
	    New-AzureService -Location $DeploymentLocation -ServiceName $SecureService -ErrorAction Stop
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
	            # Note: All traffic goes through the firewall, so we'll need to set up all ports here.
	            #       Also, the firewall will be redirecting traffic to a new IP and Port in a forwarding
	            #       rule, so all of these endpoint have the same public and local port and the firewall
	            #       will do the mapping, NATing, and/or redirection as declared in the firewall rules.
	            Add-AzureEndpoint -Name "MgmtPort1" -Protocol tcp -PublicPort 801  -LocalPort 801  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
	            Add-AzureEndpoint -Name "MgmtPort2" -Protocol tcp -PublicPort 807  -LocalPort 807  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
	            Add-AzureEndpoint -Name "HTTP"      -Protocol tcp -PublicPort 80   -LocalPort 80   -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
	            Add-AzureEndpoint -Name "RDPWeb"    -Protocol tcp -PublicPort 8014 -LocalPort 8014 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
	            Add-AzureEndpoint -Name "RDPApp1"   -Protocol tcp -PublicPort 8025 -LocalPort 8025 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
	            Add-AzureEndpoint -Name "RDPApp2"   -Protocol tcp -PublicPort 8026 -LocalPort 8026 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
	            Add-AzureEndpoint -Name "RDPDNS01"  -Protocol tcp -PublicPort 8024 -LocalPort 8024 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
	            # Note: A SSH endpoint is automatically created on port 22 when the appliance is created.
	            }
	        Else
	            {
	            New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] -InstanceSize $size[$i] | `
	                Add-AzureProvisioningConfig -Windows -AdminUsername $LocalAdmin -Password $LocalAdminPwd  | `
	                Set-AzureSubnet  -SubnetNames $SubnetName[$i] | `
	                Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
	                Set-AzureVMMicrosoftAntimalwareExtension -AntimalwareConfiguration '{"AntimalwareEnabled" : true}' | `
	                Remove-AzureEndpoint -Name "RemoteDesktop" | `
	                Remove-AzureEndpoint -Name "PowerShell" | `
	                New-AzureVM -ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
	            }
	        $i++
	    }
	
	# Configure UDR and IP Forwarding
	    Write-Host "Configuring UDR" -ForegroundColor Cyan
	
	  # Create the Route Tables
	    Write-Host "Creating the Route Tables" -ForegroundColor Cyan 
	    New-AzureRouteTable -Name $BERouteTableName `
	        -Location $DeploymentLocation `
	        -Label "Route table for $BESubnet subnet"
	    New-AzureRouteTable -Name $FERouteTableName `
	        -Location $DeploymentLocation `
	        -Label "Route table for $FESubnet subnet"
	
	  # Add Routes to Route Tables
	    Write-Host "Adding Routes to the Route Tables" -ForegroundColor Cyan 
	    Get-AzureRouteTable $BERouteTableName `
	        |Set-AzureRoute -RouteName "All traffic to FW" -AddressPrefix 0.0.0.0/0 `
	        -NextHopType VirtualAppliance `
	        -NextHopIpAddress $VMIP[0]
	    Get-AzureRouteTable $BERouteTableName `
	        |Set-AzureRoute -RouteName "Internal traffic to FW" -AddressPrefix $VNetPrefix `
	        -NextHopType VirtualAppliance `
	        -NextHopIpAddress $VMIP[0]
	    Get-AzureRouteTable $BERouteTableName `
	        |Set-AzureRoute -RouteName "Allow Intra-Subnet Traffic" -AddressPrefix $BEPrefix `
	        -NextHopType VNETLocal
	    Get-AzureRouteTable $FERouteTableName `
	        |Set-AzureRoute -RouteName "All traffic to FW" -AddressPrefix 0.0.0.0/0 `
	        -NextHopType VirtualAppliance `
	        -NextHopIpAddress $VMIP[0]
	    Get-AzureRouteTable $FERouteTableName `
	        |Set-AzureRoute -RouteName "Internal traffic to FW" -AddressPrefix $VNetPrefix `
	        -NextHopType VirtualAppliance `
	        -NextHopIpAddress $VMIP[0]
	    Get-AzureRouteTable $FERouteTableName `
	        |Set-AzureRoute -RouteName "Allow Intra-Subnet Traffic" -AddressPrefix $FEPrefix `
	        -NextHopType VNETLocal
	
	  # Assoicate the Route Tables with the Subnets
	    Write-Host "Binding Route Tables to the Subnets" -ForegroundColor Cyan 
	    Set-AzureSubnetRouteTable -VirtualNetworkName $VNetName `
	        -SubnetName $BESubnet `
	        -RouteTableName $BERouteTableName
	    Set-AzureSubnetRouteTable -VirtualNetworkName $VNetName `
	        -SubnetName $FESubnet `
	        -RouteTableName $FERouteTableName
	
	 # Enable IP Forwarding on the Virtual Appliance
	    Get-AzureVM -Name $VMName[0] -ServiceName $ServiceName[0] `
	        |Set-AzureIPForwarding -Enable
	
	# Configure NSG
	    Write-Host "Configuring the Network Security Group (NSG)" -ForegroundColor Cyan
	
	  # Build the NSG
	    Write-Host "Building the NSG" -ForegroundColor Cyan
	    New-AzureNetworkSecurityGroup -Name $NSGName -Location $DeploymentLocation -Label "Security group for $VNetName subnets in $DeploymentLocation"
	
	  # Add NSG Rule
	    Write-Host "Writing rules into the NSG" -ForegroundColor Cyan
	    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Isolate the $VNetName VNet from the Internet" -Type Inbound -Priority 100 -Action Deny `
	        -SourceAddressPrefix INTERNET -SourcePortRange '*' `
	        -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '*' `
	        -Protocol *
	
	  # Assign the NSG to two Subnets
	    # The NSG is *not* bound to the Security Subnet. The result
	    # is that internet traffic flows only to the Security subnet
	    # since the NSG bound to the Frontend and Backback subnets
	    # will Deny internet traffic to those subnets.
	    Write-Host "Binding the NSG to two subnets" -ForegroundColor Cyan
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
	          <Subnet name="SecNet">
	            <AddressPrefix>10.0.0.0/24</AddressPrefix>
	          </Subnet>
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
[1]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/example3design.png "使用 NVA、NSG 和 UDR 的双向外围网络"
[2]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/example3firewalllogical.png "防火墙规则的逻辑视图"
[3]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectfrontend.png "创建前端网络对象"
[4]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectdns.png "创建 DNS 服务器对象"
[5]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectrdpa.png "默认 RDP 规则的副本"
[6]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectrdpb.png "AppVM01 规则"
[7]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/iconapplicationredirect.png "应用程序重定向图标"
[8]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/icondestinationnat.png "目标 NAT 图标"
[9]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/iconpass.png "传递图标"
[10]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/rulefirewall.png "防火墙管理规则"
[11]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/rulerdp.png "防火墙 RDP 规则"
[12]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleweb.png "防火墙 Web 规则"
[13]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleappvm01.png "防火墙 AppVM01 规则"
[14]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleoutbound.png "防火墙出站规则"
[15]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruledns.png "防火墙 DNS 规则"
[16]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleintravnet.png "防火墙 VNet 内部规则"
[17]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruledeny.png "防火墙拒绝规则"
[18]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/firewallruleactivate.png "防火墙规则激活"

<!--Link References-->
[HOME]: /documentation/articles/best-practices-network-security/
[SampleApp]: /documentation/articles/virtual-networks-sample-app/

<!---HONumber=Mooncake_0307_2016-->