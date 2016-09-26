<properties 
	pageTitle="将应用与 Azure 虚拟网络进行集成" 
	description="演示如何将 Azure App Service 中的应用连接到新的或现有的 Azure 虚拟网络" 
	services="app-service" 
	documentationCenter="" 
	authors="ccompy" 
	manager="wpickett" 
	editor="cephalin"/>

<tags
	ms.service="app-service"
	ms.date="08/11/2016"
	wacn.date=""/>

# 将应用与 Azure 虚拟网络进行集成 #

本文档将介绍 Azure App Service 虚拟网络集成功能，并说明如何在 [Azure App Service](/documentation/services/web-sites/) 中使用应用对其进行设置。如果你不熟悉 Azure 虚拟网络 (VNET)，则这里需要指出的是，该功能允许你将多个 Azure 资源置于你可以控制其访问权限但无法通过 Internet 路由的网络中。然后，你可以使用多种 VPN 技术将这些网络连接到本地网络。若要了解有关 Azure 虚拟网络的详细信息，请先了解以下信息：[Azure 虚拟网络概述][VNETOverview]。

在 Azure 中国，Azure App Service 只有一种形式。

1. 支持全部定价计划的多租户系统

VNET 集成功能允许你的 Web 应用访问虚拟网络中的资源，但不允许通过虚拟网络对你的 Web 应用进行专用访问。

使用 VNET 集成的常见情景是，你需要通过 Web 应用来访问在 Azure 虚拟网络的虚拟机中运行的数据库或 Web 服务。使用 VNET 集成时，你不需要公开 VM 中应用程序的公共终结点，可以改用无法通过 Internet 路由的专用地址。

VNET 集成功能：

- 需要标准或高级定价计划
- 适用于经典 (V1) 或 Resource Manager(V2) VNET
- 支持 TCP 和 UDP
- 适用于 Web、移动和 API 应用
- 允许应用一次只连接到 1 个 VNET
- 允许在 App Service 计划中一次最多集成 5 个 VNET
- 允许在 App Service 计划中由多个应用使用同一个 VNET
- 由于依靠 VNET 网关，因此可实现 99.9% 的 SLA

VNET 集成不支持某些功能，其中包括：

- 装载驱动器
- AD 集成
- NetBios
- 专用站点访问

### 入门 ###
将 Web 应用连接到虚拟网络之前，你需要牢记以下几点：

- VNET 集成仅适用于“标准”或“高级”定价计划中的应用。如果你在启用此功能后又将 App Service 计划扩展为不受支持的定价计划，你的应用会失去与所用 VNET 的连接。
- 如果你的目标虚拟网络已经存在，必须在连接到应用之前借助动态路由网关使网络处于点到站点 VPN 启用状态。如果网关由静态路由配置，那么你将无法启用点到站点虚拟专用网络 (VPN)。
- VNET 所在的订阅必须与应用服务计划 (ASP) 所在的订阅相同。
- 与 VNET 集成的应用将使用为该 VNET 指定的 DNS。
- 默认情况下，集成应用只会根据 VNET 中的已定义路由将流量路由到 VNET 中。


## 启用 VNET 集成 ##

本文档重点介绍如何使用 Azure 门户进行 VNET 集成。若要使用 PowerShell 为应用启用 VNET 集成，请遵循 [Connect your app to your virtual network by using PowerShell][IntPowershell]（使用 PowerShell 将应用连接到虚拟网络）中的指示进行操作。

你可以选择将应用连接到新的或现有的虚拟网络。如果你在集成过程中创建新网络，则除了创建 VNET，系统还会为你预先配置动态路由网关并启用点到站点 VPN。

>[AZURE.NOTE] 配置新的虚拟网络集成可能需要几分钟的时间。

若要启用 VNET 集成，请先打开应用的“设置”，然后选择“网络”。打开的 UI 提供三种网络选择。

如果应用的定价计划不正确，你可以通过该 UI 将计划扩展为所选的更高级别的定价计划。


![][1]
 
###允许与预先存在的 VNET 进行 VNET 集成###
你可以通过 VNET 集成 UI 从 VNET 列表中进行选择。经典 VNET 会在 VNET 名称旁边的括号中加入“经典”一词进行表示。对列表排序时，会首先列出 Resource Manager VNET。在下图中，你可以看到，只能选择一个 VNET。多种原因会导致 VNET 灰显，其中包括：

- 该 VNET 属于你的帐户有权访问的其他订阅
- 该 VNET 未启用点到站点连接
- 该 VNET 没有动态路由网关


![][2]

若要启用集成，请直接单击要集成的 VNET。选择 VNET 以后，你的应用会自动重新启动，以使所做的更改生效。

##### 在经典 VNET 中启用点到站点连接 #####
如果你的 VNET 既没有网关，也没有点到站点连接，则需先进行此方面的设置。若要对经典 VNET 进行此操作，请转到 [Azure 门户][AzurePortal]，此时会显示虚拟网络（经典）列表。在此处单击要集成的网络，然后单击 Essentials 下名为“VPN 连接”的大框。在此处，你可以创建点到站点 VPN，甚至可以让其创建网关。如果你有网关创建经验，则在完成点到站点连接以后，还需大约 30 分钟的时间才能让其就绪。

![][8]

##### 在 Resource Manager VNET 中启用点到站点连接 #####

若要为 Resource Manager VNET 配置网关和点到站点连接，需使用 PowerShell，如下所述：[Configure a Point-to-Site connection to a virtual network using PowerShell][V2VNETP2S]（使用 PowerShell 配置虚拟网络的点到站点连接）。尚未提供执行此功能的 UI。

### 创建预先配置的 VNET ###
若要创建配置了网关和点到站点连接的新 VNET，则可使用应用服务网络 UI 来执行该操作，但仅限于 Resource Manager VNET。若要创建配置了网关和点到站点连接的经典 VNET，则需通过“网络”用户界面手动执行该操作。

若要通过 VNET 集成 UI 创建 Resource Manager VNET，请直接选择“创建新的虚拟网络”，然后提供以下信息：

- 虚拟网络名称
- 虚拟网络地址块
- 子网名称
- 子网地址块
- 网关地址块
- 点到站点地址块

如果你希望此 VNET 连接到任何其他网络，则应避免选取与那些网络重叠的 IP 地址空间。

>[AZURE.NOTE] 创建带网关的 Resource Manager VNET 大约需要 30 分钟，目前不会将 VNET 与你的应用集成。创建带网关的 VNET 以后，你需要回到应用 VNET 集成 UI 并选择新的 VNET。

![][3]

Azure VNET 通常在专用网络地址中创建。默认情况下，VNET 集成功能会将任何流向那些 IP 地址范围的流量路由到你的 VNET 中。专用 IP 地址范围如下：

- 10\.0.0.0/8 - 相当于 10.0.0.0 - 10.255.255.255
- 172\.16.0.0/12 - 相当于 172.16.0.0 - 172.31.255.255
- 192\.168.0.0/16 - 相当于 192.168.0.0 - 192.168.255.255
 
VNET 地址空间需使用 CIDR 表示法来指定。如果你不熟悉 CIDR 表示法，则这里需要指出的是，该表示法是一种使用 IP 地址和整数（代表网络掩码）来指定地址块的方法。举例来说，10.1.0.0/24 表示 256 个地址，10.1.0.0/25 表示 128 个地址。IPv4 地址后接 /32 表示 1 个地址。

如果你在此处设置了 DNS 服务器信息，则会对你的 VNET 进行相应的设置。创建 VNET 以后，你可以根据 VNET 用户体验编辑此信息。

使用 VNET 集成 UI 创建经典 VNET 时，该 UI 会在与你的应用相同的资源组中创建 VNET。

## 系统的工作方式 ##
实际上，该功能基于点到站点 VPN 技术将你的应用连接到 VNET。Azure App Service 中的应用具有多租户系统体系结构，与虚拟机中一样，它可以阻止直接在 VNET 中预配应用。通过依托点到站点技术，我们将网络访问限制在托管应用的那台虚拟机。在那些应用主机上会进一步限制对网络的访问，因此你的应用只能访问配置好的允许访问的网络。

![][4]
 
如果尚未使用虚拟网络配置 DNS 服务器，你需要使用 IP 地址。使用 IP 地址时，请记住，此功能的主要好处是允许你在专用网络中使用专用地址。如果将应用设置为对其中一个 VM 使用公共 IP 地址，则你使用的不是 VNET 集成功能，而是通过 Internet 进行通信。


##管理 VNET 集成##

连接到 VNET 以及断开其连接的功能在应用级别执行。可能影响跨多个应用的 VNET 集成的操作在 ASP 级别执行。你可以通过在应用级别显示的 UI 获取 VNET 的详细信息。这些相同信息的大部分也会显示在 ASP 级别。

![][5]

在“网络功能状态”页中，可查看应用是否已连接到 VNET。如果 VNET 网关因某种原因而关闭，则会显示为未连接。

你现在在应用级别的 VNET 集成 UI 中获得的信息与你通过 ASP 获得的详细信息是相同的。下面是信息中的各项内容：

- VNET 名称 - 此链接打开网络 UI
- 位置 - 此项反映 VNET 的位置。可以在其他位置与 VNET 集成。
- 证书状态 - 可以使用证书来确保应用和 VNET 之间的 VPN 连接的安全性。此项所反映的测试用于确保证书处于同步状态。
- 网关状态 - 如果网关因某种原因而关闭，则应用无法访问 VNET 中的资源。
- VNET 地址空间 - 此项为 VNET 的 IP 地址空间。
- 点到站点地址空间 - 此项为 VNET 的点到站点 IP 地址空间。你的应用会在此地址空间中将通信显示成来自其中一个 IP。
- 站点到站点地址空间 - 你可以使用站点到站点 VPN 将 VNET 连接到本地资源或其他 VNET。配置完成以后，针对该 VPN 连接定义的 IP 范围会显示在此处。
- DNS 服务器 - 如果已为 DNS 服务器配置了 VNET，则这些服务器会列在此处。
- 路由到 VNET 的 IP - 此项为通过 VNET 定义了路由的 IP 地址的列表。这些地址会显示在此处。

在 VNET 集成的应用视图中，你能够执行的唯一操作是断开应用与当前所连接到的 VNET 的连接。为此，请直接单击顶部的“断开连接”。此操作不会更改你的 VNET。VNET 及其包括网关在内的配置都保持不变。如果你随后想要删除 VNET，则需先删除其中的资源（包括网关）。

“App Service 计划”视图包含许多其他操作。它还可以通过其他方式进行访问，通过应用进行访问除外。若要访问 ASP 网络 UI，请直接打开 ASP UI，然后向下滚动。有一个名为“网络功能状态”的 UI 元素。该元素会提供一些有关 VNET 集成的次要详细信息。单击此 UI 会打开网络功能状态 UI。如果你随后单击“单击此处进行管理”，则会打开一个 UI，其中会列出此 ASP 中的 VNET 集成。

![][6]

当你查看要集成的 VNET 的位置时，ASP 的位置很好记忆。当 VNET 位于其他位置时，出现延迟问题的可能性要大得多。

通过集成的 VNET 数，你可以了解到此 ASP 中与应用集成的 VNET 数，以及你到底可以集成多少。

若要查看每个 VNET 的更多详细信息，请直接单击你感兴趣的 VNET。除了前述详细信息，你还会看到此 ASP 中正在使用该 VNET 的应用的列表。

关于操作，一共有两个主要操作。第一个操作是允许添加驱使流量从应用进入 VNET 的路由。第二个操作是允许同步证书和网络信息。

![][7]

**路由** 
如前所述，在 VNET 中定义的路由用于将流量从应用导入 VNET。不过还有一些用处。当客户需要将额外的出站流量从应用发送到 VNET 中时，就需要用到此功能。此后对流量的控制取决于客户对其 VNET 的配置。

**证书** 
证书状态反映了应用服务所执行的检查的结果，该检查是为了验证用于 VPN 连接的证书是否仍然有效。启用 VNET 集成后，如果这是第一次从该 ASP 中的应用集成到该 VNET，则必须进行证书交换以确保连接的安全性。除了证书，我们还可以获得 DNS 配置、路由以及其他类似的用于描述网络的内容。
如果更改了这些证书或网络信息，则需单击“同步网络”。**注意**：单击“同步网络”会导致应用与 VNET 之间的连接出现短暂的中断。虽然应用不会重新启动，但失去连接会导致站点功能失常。

##访问本地资源##

VNET 集成功能的一大好处是，如果 VNET 通过站点到站点 VPN 连接到本地网络，则可通过应用访问本地资源。不过，你可能需要使用点到站点 IP 范围的路由来更新本地 VPN 网关才能这样做。先设置站点到站点 VPN 以后，接着应通过用于配置该 VPN 的脚本来设置包括点到站点 VPN 在内的路由。如果在创建站点到站点 VPN 后才添加点到站点 VPN，则需手动更新路由。具体操作信息取决于每个网关，在此不做说明。

>[AZURE.NOTE] 虽然可以在站点到站点 VPN 中使用 VNET 集成功能来访问本地资源，但目前仍无法在 ExpressRoute VPN 中使用该功能来执行同一操作。与经典或 Resource Manager VNET 集成时也是如此。

##定价详细信息##
使用 VNET 集成功能时，应了解某些定价方面的细节差异。使用此功能时，涉及到 3 种相关费用：

- ASP 定价层要求
- 数据传输费用
- VPN 网关费用。

应用必须是标准或高级 App Service 计划中的应用才能使用此功能。可通[应用服务定价][ASPricing]了解此方面费用的更多详细信息。

通过 VNET 集成连接进行传输的出站数据始终收费，即使该 VNET 是在同一数据中心。这是由系统对点到站点 VPN 的处理方式决定的。若要了解具体的收费情况，请参阅：[数据传输定价详细信息][DataPricing]。

最后一项是 VNET 网关费用。即使你不需使用网关来执行站点到站点 VPN 之类的其他功能，也仍需为网关付费，否则无法使用 VNET 集成功能。下面是此方面费用的详细信息：[VPN 网关定价][VNETPricing]。

##故障排除##

虽然此功能容易设置，但这并不意味着你就不会遇到问题。如果你在访问所需终结点时遇到问题，可以使用某些实用程序来测试从应用控制台发出的连接。你可以通过两种方式来体验控制台的使用。一种方式是使用 Kudu 控制台，另一种方式是访问 Azure 门户中的控制台。若要访问应用中的 Kudu 控制台，请转到“工具”->“Kudu”。这相当于访问 [sitename].scm.chinacloudsites.cn。打开后，直接转到“调试控制台”选项卡。若要访问 Azure 门户托管的控制台，请在应用中转到“工具”->“控制台”。


####工具####

由于存在安全约束，ping、nslookup 和 tracert 工具无法通过控制台来使用。为了填补这方面的空白，我们添加了两项单独的工具。我们添加了名为 nameresolver.exe 的工具，用于测试 DNS 功能。语法为：

    nameresolver.exe hostname [optional: DNS Server]

可以使用 nameresolver 来检查应用所需的主机名。你可以通过这种方式来测试 DNS 是否配置错误，或者测试你是否无权访问 DNS 服务器。

下一工具适用于测试与主机的 TCP 连接情况，以及端口组合情况。该工具名为 tcpping.exe，语法为：

    tcpping.exe hostname [optional: port]

该工具会告诉你是否可以访问特定的主机和端口，但不会执行基于 ICMP 的 ping 实用程序所执行的那种任务。ICMP ping 实用程序会告诉你主机是否已启动。使用 tcpping 可以了解你能否访问主机上的特定端口。


####针对 VNET 托管的资源进行访问权限调试####

许多因素会阻止应用访问特定的主机和端口。大多数情况下为以下三种情况：

- **存在防火墙** 若存在防火墙，则会发生 TCP 超时。本例中为 21 秒。使用 tcpping 工具测试连接性。除了防火墙外，还有多种原因可能导致 TCP 超时。
- **DNS 不可访问** DNS 超时时间为每个 DNS 服务器 3 秒。两个 DNS 服务器则为 6 秒。使用 nameresolver 查看 DNS 是否正常工作。请记住，无法使用 nslookup，因为没有使用为 VNET 配置的 DNS。
- **无效的 P2S IP 范围** 点到站点的 IP 范围必须在 RFC 1918 专用 IP 范围内 (10.0.0.0-10.255.255.255/172.16.0.0-172.31.255.255/192.168.0.0-192.168.255.255)，如果使用该范围外的 IP，则不起作用。

如果这些情况不能解决你的问题，请首先查看以下简单因素：

- 网关在门户中是否显示为已启动？
- 证书是否显示为处于同步状态？
- 是否有人更改了网络配置却没有在受影响的 ASP 中执行“同步网络”操作？

如果网关处于关闭状态，则将其重新启动。如果证书不同步，则请转到 VNET 集成的 ASP 视图，然后单击“同步网络”。如果你怀疑有人对 VNET 配置进行了更改且未使用 ASP 进行同步，则请转到 VNET 集成的 ASP 视图，然后单击“同步网络”。需提请注意的是，这会导致 VNET 连接和应用出现短暂中断。

如果这些都正常，则需进行更深入的故障诊断：

- 是否存在其他应用在使用 VNET 集成访问同一 VNET 中的资源？
- 是否能够转到应用控制台并使用 tcpping 来访问 VNET 中的任何其他资源？

如果上面这两个问题中有一个的回答为“是”，则说明 VNET 集成功能正常，问题出在其他地方。这种情况下，解决问题会更加困难，因为并没有简单的方法可以查看你为何无法访问主机:端口。部分原因包括：

- 你在主机上开启了防火墙，导致无法从点到站点 IP 范围访问应用程序端口。跨子网通常需要公共访问权限。
- 目标主机已关闭
- 应用程序已关闭
- 你的 IP 或主机名错误
- 你的应用程序所侦听的端口不同于你所期望的端口。若要查看是否属于这种情况，你可以转到该主机，然后在命令提示符处执行“netstat -aon”。此时会显示在相应端口上进行侦听的进程 ID。
- 你的网络安全组的配置方式导致无法从点到站点 IP 范围访问应用程序主机和端口

请记住，你不知道应用会使用点到站点 IP 范围中的哪个 IP，因此需允许整个范围中的 IP 进行访问。

其他调试步骤包括：

- 登录到 VNET 中的其他 VM，尝试在该处访问资源主机:端口。你可以使用某些 TCP ping 实用程序来进行此类操作，甚至还可以视需要使用 telnet。此处的目的就是确定是否可以从这个其他的 VM 进行连接。
- 在其他 VM 中启动应用程序，测试能否在应用的控制台中访问该主机和端口
####本地资源####
如果你无法访问本地资源，则首先应检查你是否能够访问 VNET 中的资源。如果能够访问，则后续步骤很容易。你需要尝试从 VNET 中的 VM 访问本地应用程序。你可以使用 telnet 或 TCP ping 实用程序。如果 VM 无法访问本地资源，首先请确保站点到站点 VPN 连接正常。如果正常，则请检查前述项目以及本地网关配置和状态。

现在，如果 VNET 托管的 VM 能够访问本地系统但应用无法访问，则可能是由于以下某个原因：
- 在本地网关中未使用点到站点 IP 范围对路由进行配置
- 网络安全组阻止点到站点 IP 范围中的 IP 进行访问
- 本地防火墙阻止来自点到站点 IP 范围的流量
- VNET 中的用户定义路由 (UDR) 阻止点到站点型流量访问本地网络

<!--Image references-->
[1]: ./media/web-sites-integrate-with-vnet/vnetint-upgradeplan.png
[2]: ./media/web-sites-integrate-with-vnet/vnetint-existingvnet.png
[3]: ./media/web-sites-integrate-with-vnet/vnetint-createvnet.png
[4]: ./media/web-sites-integrate-with-vnet/vnetint-howitworks.png
[5]: ./media/web-sites-integrate-with-vnet/vnetint-appmanage.png
[6]: ./media/web-sites-integrate-with-vnet/vnetint-aspmanage.png
[7]: ./media/web-sites-integrate-with-vnet/vnetint-aspmanagedetail.png
[8]: ./media/web-sites-integrate-with-vnet/vnetint-vnetp2s.png

<!--Links-->
[VNETOverview]: /documentation/articles/virtual-networks-overview/
[AzurePortal]: http://portal.azure.cn/
[ASPricing]: /pricing/overview/app-service/
[VNETPricing]: /pricing/overview/vpn-gateway/
[DataPricing]: /pricing/overview/data-transfers/
[V2VNETP2S]: /documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/
[IntPowershell]: /documentation/articles/app-service-vnet-integration-powershell/

<!---HONumber=Mooncake_0919_2016-->