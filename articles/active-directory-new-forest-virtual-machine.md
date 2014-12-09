<properties linkid="manage-services-networking-active-directory-forest" urlDisplayName="Active Directory forest" pageTitle="Install an Active Directory forest on an Azure virtual network" metaKeywords="" description="A tutorial that explains how to create a new Active Directory forest on a virtual machine (VM) on an Azure Virtual Network." metaCanonical="" services="active-directory,virtual-network" documentationCenter="" title="Install a new Active Directory forest in Azure" authors="Justinha"  solutions="" writer="Justinha" manager="TerryLan" editor="LisaToft"  />

# 在 Azure 虚拟网络中安装新的 Active Directory 林

本主题说明如何在 [Azure 虚拟网络][Azure 虚拟网络]上的虚拟机 (VM) 中创建新的 Windows Server Active Directory 环境。在此情况下，Azure 虚拟网络未连接到本地网络。

你也有可能对下列相关主题感兴趣：

-   你可以有选择性地[使用管理门户向导配置站点到站点 VPN][使用管理门户向导配置站点到站点 VPN]，然后安装新林，或者将本地林扩展到 Azure 虚拟网络。有关这些步骤的说明，请参阅[在 Azure 虚拟网络中安装副本 Active Directory 域控制器][在 Azure 虚拟网络中安装副本 Active Directory 域控制器]。
-   有关在 Azure 虚拟网络上安装 Active Directory 域服务 (AD DS) 的概念性指南，请参阅[在 Azure 虚拟机中部署 Windows Server Active Directory 的准则][在 Azure 虚拟机中部署 Windows Server Active Directory 的准则]。
-   有关在使用 AD DS 的 Azure 上创建测试实验室环境的分步指南，请参阅[测试实验室指南：Azure 中的 Windows Server 2012 R2 基本配置][测试实验室指南：Azure 中的 Windows Server 2012 R2 基本配置]。

## 目录

-   [在 Azure 上安装与在本地安装有什么不同？][在 Azure 上安装与在本地安装有什么不同？]
-   [步骤 1. 创建 Azure 虚拟网络][步骤 1. 创建 Azure 虚拟网络]
-   [步骤 2：创建 VM 以运行域控制器和 DNS 服务器角色][步骤 2：创建 VM 以运行域控制器和 DNS 服务器角色]
-   [步骤 3：安装 Windows Server Active Directory][步骤 3：安装 Windows Server Active Directory]
-   [步骤 4：为 Azure 虚拟网络设置 DNS 服务器][步骤 4：为 Azure 虚拟网络设置 DNS 服务器]
-   [步骤 5：为域成员创建 VM 并加入到域中][步骤 5：为域成员创建 VM 并加入到域中]

## <span id="differ"></span></a>在 Azure 上安装与在本地安装有什么不同？

在 Azure 上安装域控制器与在本地安装域控制器并没有太大的不同。下表列出了主要差别。

| 配置...                         | 本地                                                    | Azure 虚拟网络                                                                     |
|---------------------------------|---------------------------------------------------------|------------------------------------------------------------------------------------|
| **域控制器的 IP 地址**          | 在网络适配器属性中分配静态 IP 地址                      | 通过 DHCP 获取 IP 地址，然后运行 Set-AzureStaticVNetIP cmdlet 使其成为静态 IP 地址 |
| **DNS 客户端解析器**            | 在域成员的网络适配器属性中设置首选和备用 DNS 服务器地址 | 在虚拟网络属性中设置 DNS 服务器地址                                                |
| **Active Directory 数据库存储** | 选择性地更改默认存储位置 C:\\                           | 你需要更改默认存储位置 C:\\                                                        |

## <span id="createvnet"></span></a>步骤 1：创建 Azure 虚拟网络

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  创建虚拟网络。单击**“网络”**\>**“创建虚拟网络”**。使用下表中的值来完成向导操作。

    <table>
    <colgroup>
    <col width="50%" />
    <col width="50%" />
    </colgroup>
    <thead>
    <tr class="header">
    <th align="left">在此向导页上...</th>
    <th align="left">指定这些值</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td align="left"><strong>虚拟网络详细信息</strong></td>
    <td align="left"><p>Name:输入虚拟网络的名称</p>
    <p>区域：选择最靠近的区域</p>
    <p>地缘组：<strong>创建新的地缘组</strong></p>
    <p>地缘组名称：输入地缘组的名称</p></td>
    </tr>
    <tr class="even">
    <td align="left"><strong>DNS 和 VPN</strong></td>
    <td align="left"><p>将 DNS 服务器保留为空</p>
    <p>不要选择任一 VPN 选项</p></td>
    </tr>
    <tr class="odd">
    <td align="left"><strong>虚拟网络地址空间</strong></td>
    <td align="left"><p>子网名称：输入子网的名称</p>
    <p>起始 IP：<strong>10.0.0.0</strong></p>
    <p>CIDR： <strong>/24 (256)</strong></p></td>
    </tr>
    </tbody>
    </table>

## <span id="createvm"></span></a>步骤 2：创建 VM 以运行域控制器和 DNS 服务器角色

1.  单击**“新建”**\>**“计算”**\>**“虚拟机”**\>**“从库中”**。
2.  使用下表中的值来完成向导操作。

    <table>
    <colgroup>
    <col width="50%" />
    <col width="50%" />
    </colgroup>
    <thead>
    <tr class="header">
    <th align="left">在此向导页上...</th>
    <th align="left">指定这些值</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td align="left"><strong>操作系统</strong></td>
    <td align="left">选择<b>“Windows Server 2012 R2 Datacenter”</b>。</td>
    </tr>
    <tr class="even">
    <td align="left"><strong>虚拟机配置</strong></td>
    <td align="left"><p>发布日期：当天的日期</p>
    <p>计算机名称：指定唯一值</p>
    <p>层：标准</p>
    <p>大小：选择任意大小</p>
    <p>用户名：输入名称。此用户帐户将成为内置管理员组的成员。</p>
    <p>密码：必须至少为 8 个字符，可以包含以下类型的 3 个字符：</p>
    <ul>
    <li>大写字母</li>
    <li>小写字母</li>
    <li>数字</li>
    <li>特殊字符</li>
    </ul></td>
    </tr>
    <tr class="odd">
    <td align="left"><strong>云服务</strong></td>
    <td align="left"><p>云服务：<strong>创建新的云服务</strong></p>
    <p>云服务名称：接受默认值</p>
    <p>区域/地缘组/虚拟网络：选择你创建的虚拟网络</p>
    <p>虚拟网络子网：选择你创建的子网。</p>
    <p>存储帐户：<strong>使用自动生成的存储帐户</strong></p>
    <p>可用性集：<strong>无</strong></p>
    <p>终结点：接受默认值</p></td>
    </tr>
    <tr class="even">
    <td align="left"><strong>VM 代理</strong></td>
    <td align="left">选择<b>“安装 VM 代理”</b></td>
    </tr>
    </tbody>
    </table>

3.  按默认分配给 VM 的动态 IP 地址在云服务的工作持续时间内有效。但是，如果关闭 VM，该地址将发生更改。你可以通过[运行 Set-AzureStaticVNetIP Azure PowerShell cmdlet][运行 Set-AzureStaticVNetIP Azure PowerShell cmdlet] 来分配静态 IP 地址，这样，即使你关闭了 VM，该 IP 地址也会保留。
4.  将额外的磁盘附加到 VM，以便存储 Active Directory 数据库、日志和 SYSVOL。
  5.  单击**“VM”**\>**“附加”**\>**“附加空磁盘”**。
  6.  指定大小（例如 10 GB）并接受所有其他默认值。
7.  登录到 VM 并格式化该附加磁盘。
  8.  单击**“连接”**以登录到 VM，单击**“打开”**以创建 RDP 会话，然后再次单击**“连接”**。
  9.  将凭据更改为你指定的新用户名和密码。
  10. 在服务器管理器中，单击**“工具”**\>**“计算机管理”**。
  11. 单击**“磁盘管理”**，然后单击**“确定”**以初始化新磁盘。
  12. 右键单击磁盘名称，然后单击**“新建简单卷”**。完成向导操作以格式化新驱动器。

## <span id="installad"></span></a>步骤 3：安装 Windows Server Active Directory

使用你在本地使用的相同例程[安装 AD DS][安装 AD DS]（也就是说，你可以使用 UI、应答文件或 Windows PowerShell）。需要提供管理员凭据来安装新林。若要指定 Active Directory 数据库、日志和 SYSVOL 的位置，请将默认存储位置从操作系统驱动器更改为你已附加到 VM 的额外数据磁盘。

完成 DC 安装后，请再次连接到 VM 并登录到 DC。请记得指定域凭据。

</p>
## <span id="dns"></span></a>步骤 4：为 Azure 虚拟网络设置 DNS 服务器

1.  单击**“虚拟网络”**，双击你创建的虚拟网络，然后单击**“配置”**。
2.  在**“DNS 服务器”**下，键入 DC 的名称和 DIP，然后单击**“保存”**。
3.  选择 VM 并单击**“重新启动”**触发该 VM，以便使用新 DNS 服务器的 IP 地址配置 DNS 解析器设置。

## <span id="domainmembers"></span></a>步骤 5：为域成员创建 VM 并加入到域中

创建用于设置域成员计算机的附加 VMS。你可以使用 UI 或 Azure PowerShell。如果使用 UI，则只需遵循你在创建第一个 VM 时所用的相同步骤。然后，就像在本地操作一样，将 VM 加入到域中。如果使用 Azure PowerShell，则可以设置 VM，然后在首次启动 VM 时将其加入到域中，如以下示例中所示。
 '

    cls

    Set-AzureSubscription -SubscriptionName "Free Trial" -currentstorageaccountname 'constorageaccount'
    Select-AzureSubscription -SubscriptionName "Free Trial"

    #Deploy a new VM and join it to the domain
    #-------------------------------------------
    #Specify my DC's DNS IP (10.0.0.4)
    $myDNS = New-AzureDNS -Name 'DC-1' -IPAddress '10.0.0.4'

    # OS Image to Use
    $image = 'a699494373c04fc0bc8f2bb1389d6106__Win2K8R2SP1-Datacenter-201310.01-en.us-127GB.vhd'
    $service = 'ConApp1'
    $AG = 'YourAffinityGroup'
    $vnet = 'YourVirtualNetwork'
    $pwd = 'P@ssw0rd'
    $size = 'Small'

    #VM Configuration
    $vmname = 'ConApp1'
    $MyVM3 = New-AzureVMConfig -name $vmname -InstanceSize $size -ImageName $image |
    Add-AzureProvisioningConfig -AdminUserName 'PierreSettles' -WindowsDomain -Password $pwd -Domain 'Contoso' -DomainPassword 'P@ssw0rd' -DomainUserName 'PierreSettles' -JoinDomain 'contoso.com'|
    Set-AzureSubnet -SubnetNames 'FrontEnd' 

    New-AzureVM -ServiceName $service -AffinityGroup $AG -VMs $MyVM3 -DnsSettings $myDNS -VNetName $vnet   

如果你重新运行该脚本，则需要为 $service 提供唯一值。你可以运行 Test-AzureName –Service *服务名称*，以检查该名称是否已被使用。

## 另请参阅

-   [在 Azure 虚拟机中部署 Windows Server Active Directory 的准则][在 Azure 虚拟机中部署 Windows Server Active Directory 的准则]

-   [在管理门户中配置只使用云的虚拟机][在管理门户中配置只使用云的虚拟机]

-   [在管理门户中配置站点到站点 VPN][在管理门户中配置站点到站点 VPN]

-   [在 Azure 虚拟网络中安装副本 Active Directory 域控制器][在 Azure 虚拟网络中安装副本 Active Directory 域控制器]

-   [Windows Azure IT Pro IaaS：(01) 虚拟机基础知识][Windows Azure IT Pro IaaS：(01) 虚拟机基础知识]

-   [Windows Azure IT Pro IaaS：(05) 创建虚拟网络和跨界连接][Windows Azure IT Pro IaaS：(05) 创建虚拟网络和跨界连接]

-   [Azure 虚拟网络][1]

-   [如何安装和配置 Azure PowerShell][如何安装和配置 Azure PowerShell]

-   [Azure PowerShell][Azure PowerShell]

-   [Azure 管理 Cmdlet][Azure 管理 Cmdlet]

-   [设置 Azure VM 静态 IP 地址][设置 Azure VM 静态 IP 地址]

-   [如何向 Azure VM 分配静态 IP][如何向 Azure VM 分配静态 IP]

-   [安装新的 Active Directory 林][安装 AD DS]

-   [介绍 Active Directory 域服务 (AD DS) 虚拟化（级别 100）][介绍 Active Directory 域服务 (AD DS) 虚拟化（级别 100）]

-   [测试实验室指南：Azure 中的 Windows Server 2012 R2 基本配置][测试实验室指南：Azure 中的 Windows Server 2012 R2 基本配置]

  [Azure 虚拟网络]: http://msdn.microsoft.com/library/azure/jj156007.aspx
  [使用管理门户向导配置站点到站点 VPN]: http://msdn.microsoft.com/library/azure/dn133795.aspx
  [在 Azure 虚拟网络中安装副本 Active Directory 域控制器]: http://windowsazure.cn/zh-cn/documentation/articles/virtual-networks-install-replica-active-directory-domain-controller/?fb=zh-cn
  [在 Azure 虚拟机中部署 Windows Server Active Directory 的准则]: http://msdn.microsoft.com/library/azure/jj156090.aspx
  [测试实验室指南：Azure 中的 Windows Server 2012 R2 基本配置]: http://www.microsoft.com/en-us/download/details.aspx?id=41684
  [在 Azure 上安装与在本地安装有什么不同？]: #differ
  [步骤 1. 创建 Azure 虚拟网络]: #createvnet
  [步骤 2：创建 VM 以运行域控制器和 DNS 服务器角色]: #createvm
  [步骤 3：安装 Windows Server Active Directory]: #installad
  [步骤 4：为 Azure 虚拟网络设置 DNS 服务器]: #dns
  [步骤 5：为域成员创建 VM 并加入到域中]: #domainmembers
  [Azure 管理门户]: https://manage.windowsazure.cn
  [运行 Set-AzureStaticVNetIP Azure PowerShell cmdlet]: http://msdn.microsoft.com/library/azure/dn630228.aspx
  [安装 AD DS]: http://technet.microsoft.com/library/jj574166.aspx
  [在管理门户中配置只使用云的虚拟机]: http://msdn.microsoft.com/library/dn631643.aspx
  [在管理门户中配置站点到站点 VPN]: http://msdn.microsoft.com/library/dn133795.aspx
  [Windows Azure IT Pro IaaS：(01) 虚拟机基础知识]: http://channel9.msdn.com/Series/Windows-Azure-IT-Pro-IaaS/01
  [Windows Azure IT Pro IaaS：(05) 创建虚拟网络和跨界连接]: http://channel9.msdn.com/Series/Windows-Azure-IT-Pro-IaaS/05
  [1]: http://msdn.microsoft.com/library/windowsazure/jj156007.aspx
  [如何安装和配置 Azure PowerShell]: http://windowsazure.cn/zh-cn/documentation/articles/install-configure-powershell/?fb=zh-cn
  [Azure PowerShell]: http://msdn.microsoft.com/library/azure/jj156055.aspx
  [Azure 管理 Cmdlet]: http://msdn.microsoft.com/library/azure/jj152841
  [设置 Azure VM 静态 IP 地址]: http://windowsitpro.com/windows-azure/set-azure-vm-static-ip-address
  [如何向 Azure VM 分配静态 IP]: http://www.bhargavs.com/index.php/2014/03/13/how-to-assign-static-ip-to-azure-vm/
  [介绍 Active Directory 域服务 (AD DS) 虚拟化（级别 100）]: http://technet.microsoft.com/zh-cn/library/hh831734.aspx
