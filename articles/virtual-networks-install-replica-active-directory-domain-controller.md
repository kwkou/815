<properties linkid="manage-services-networking-replica-domain-controller" urlDisplayName="Replica domain controller" pageTitle="Install a replica domain controller in Azure" metaKeywords="" description="A tutorial that teaches you how to install a domain controller from your Corp Active Directory forest on your Azure virtual machine." metaCanonical="" services="virtual-network" documentationCenter="" title="Install a Replica Active Directory Domain Controller in Azure Virtual Networks" authors="Justinha" solutions="" writer="Justinha" manager="TerryLan" editor="LisaToft" />

# 在 Azure 虚拟网络中安装 Active Directory 域控制器副本

本教程将指导你完成从 [Azure 虚拟网络][Azure 虚拟网络]上的虚拟机 (VM) 中的 Corp Active Directory 林安装附加域控制器的步骤。在本教程中，虚拟机的虚拟网络将连接到你公司的网络。有关在 Azure 虚拟网络上安装 Active Directory 域服务 (AD DS) 的概念性指导，请参阅[在 Azure 虚拟机上部署 Windows Server Active Directory 的指南][在 Azure 虚拟机上部署 Windows Server Active Directory 的指南]。

## 目录

-   [先决条件][先决条件]
-   [步骤 1：验证 YourPrimaryDC 的静态 IP 地址][步骤 1：验证 YourPrimaryDC 的静态 IP 地址]
-   [步骤 2：安装 Corp 林][步骤 2：安装 Corp 林]
-   [步骤 3：创建子网和站点][步骤 3：创建子网和站点]
-   [步骤 4：在 CloudSite 中安装附加域控制器][步骤 4：在 CloudSite 中安装附加域控制器]
-   [步骤 5：验证安装][步骤 5：验证安装]
-   [步骤 6：设置在启动时加入域的虚拟机][步骤 6：设置在启动时加入域的虚拟机]
-   [步骤 7：备份域控制器][步骤 7：备份域控制器]
-   [步骤 8：测试身份验证和授权][步骤 8：测试身份验证和授权]

## <span id="Prerequisites"></span></a> 先决条件

-   [在管理门户中配置站点到站点的 VPN][在管理门户中配置站点到站点的 VPN]，在 Azure 虚拟网络和 Corp 网络之间配置。
-   在虚拟网络中创建云服务。
-   在云服务中部署作为虚拟网络的一部分的两台虚拟机（指定要放置虚拟机的子网）。有关详细信息，请参阅[将虚拟机添加到虚拟网络（可能为英文页面）][将虚拟机添加到虚拟网络（可能为英文页面）]。一台虚拟机的大小必须为“大”或更大才能向其附加两个数据磁盘。需要数据磁盘才能存储以下内容：

    -   Active Directory 数据库和日志。
    -   系统状态备份。
-   包含两台虚拟机的 Corp 网络（YourPrimaryDC 和 FileServer）。
-   部署的域名系统 (DNS) 基础结构（如果你需要让外部用户解析 Active Directory 中的帐户名称）。在此情况下，你应在域控制器上安装 DNS 服务器前创建 DNS 区域委派，或允许 Active Directory 域服务安装向导创建该委派。有关创建 DNS 区域委派的详细信息，请参阅[创建区域委派][创建区域委派]。
-   在安装到 Azure 虚拟机上的 DC 中，配置 DNS 客户端解析器设置，如下所示：

    -   首选 DNS 服务器:本地 DNS 服务器
    -   备用 DNS 服务器:环回地址或（如果可能）在同一虚拟网络的 DC 上运行的其他 DNS 服务器。

<div class="dev-callout"> 
<b>说明</b>
<p>你需要提供自己的 DNS 基础结构来支持 Azure 虚拟网络上的 AD DS。此版本的 Azure 提供的 DNS 基础结构不支持 AD DS 需要的某些功能，如动态 SRV 资源记录注册。 </p>
</div>

<div class="dev-callout"> 
<b>说明</b>
<p>如果你已完成<a href="/en-us/manage/services/networking/active-directory-forest/">在 Azure 中安装新的 Active Directory 林</a>中的步骤，则在开始本教程前，你可能需要从 Azure 虚拟网络上的域控制器中删除 AD DS。有关如何删除 AD DS 的详细信息，请参阅<a href="http://technet.microsoft.com/en-us/library/cc771844(v=WS.10).aspx">从域中删除域控制器</a>。</p>
</div>

## <span id="verifystaticip"></span></a>步骤 1：验证 YourPrimaryDC 的静态 IP 地址

1.  登录到 Corp 网络上的 YourPrimaryDC。

2.  在服务器管理器中，单击“查看网络连接”。

3.  右键单击局域网连接，然后单击“属性”。

4.  单击“Internet 协议版本 4 (TCP/IPv4)”并单击“属性”。

5.  验证是否已为服务器分配静态 IP 地址。

    ![VerifystaticIPaddressyourPrimaryDC1][VerifystaticIPaddressyourPrimaryDC1]

## <span id="installforest"></span></a>步骤 2：安装 Corp 林

1.  在虚拟机的 RDP 会话中，单击“开始”，键入 **dcpromo**，然后按 ENTER 键。

    ![InstallCorpForest1][InstallCorpForest1]

2.  在“欢迎”页上，单击“下一步”。

    ![InstallCorpForest2][InstallCorpForest2]

3.  在“操作系统兼容性”页上，单击“下一步”。

    ![InstallCorpForest3][InstallCorpForest3]

4.  在“选择部署配置”页上，单击“在新林中新建域”，然后单击“下一步”。

    ![InstallCorpForest4][InstallCorpForest4]

5.  在“命名林根域”页上，键入“corp.contoso.com”作为林根域的完全限定域名 (FQDN)，然后单击“下一步”。

    ![InstallCorpForest5][InstallCorpForest5]

6.  在“设置林功能级别”页上，单击“Windows Server 2008 R2”，然后单击“下一步”。

    ![InstallCorpForest6][InstallCorpForest6]

7.  在“其他域控制器选项”页上，单击“DNS 服务器”，然后单击“下一步”。

    ![InstallCorpForest7][InstallCorpForest7]

8.  如果出现以下 DNS 委派警告，请单击“是”。

    ![InstallCorpForest8][InstallCorpForest8]

9.  在“Active Directory 数据库、日志文件和 SYSVOL 的位置”页上，键入或选择文件的位置，然后单击“下一步”。

    ![InstallCorpForest9][InstallCorpForest9]

10. 在“目录服务还原管理员”页上，键入并确认 DSRM 密码，然后单击“下一步”。

    ![InstallCorpForest10][InstallCorpForest10]

11. 在“摘要”页上，确认你的选择，然后单击“下一步”。

    ![InstallCorpForest11][InstallCorpForest11]

12. 在 Active Directory 安装向导完成后，单击“完成”，然后单击“立即重新启动”以完成安装。

    ![InstallCorpForest12][InstallCorpForest12]

## <span id="subnets"></span></a>步骤 3：创建子网和站点

1.  在 YourPrimaryDC 上，依次单击“开始”、“管理工具”和“Active Directory 站点和服务”。
2.  单击“站点”，右键单击“子网”，然后单击“新建子网”。

    ![CreateSubnetsandSites1][CreateSubnetsandSites1]

3.  在“前缀”中，键入 **10.1.0.0/24**，选中 **Default-First-Site-Name** 站点对象并单击“确定”。

    ![CreateSubnetsandSites2][CreateSubnetsandSites2]

4.  右键单击“站点”，并单击“新建站点”。

    ![CreateSubnetsandSites3][CreateSubnetsandSites3]

5.  在“名称”中，键入 **CloudSite**，选中 **DEFAULTIPSITELINK** 并单击“确定”。

    ![CreateSubnetsandSites4][CreateSubnetsandSites4]

6.  单击“确定”以确认该站点已创建。

    ![CreateSubnetsandSites5][CreateSubnetsandSites5]

7.  右键单击“子网”，然后单击“新建子网”。

    ![CreateSubnetsandSites6][CreateSubnetsandSites6]

8.  在“前缀:”中，键入 **10.4.2.0/24**，选中 **CloudSite** 站点对象并单击“确定”。

    ![CreateSubnetsandSites7][CreateSubnetsandSites7]

## <span id="cloudsite"></span></a>步骤 4：在 CloudSite 中安装附加域控制器

1.  登录到 YourVMachine，单击“开始”，键入 **dcpromo**，并按 ENTER 键。

    ![AddDC1][AddDC1]

2.  在“欢迎”页上，单击“下一步”。

    ![AddDC2][AddDC2]

3.  在“操作系统兼容性”页上，单击“下一步”。

    ![AddDC3][AddDC3]

4.  在“选择部署配置”页上，依次单击“现有林”、“将域控制器添加到现有域”和“下一步”。

    ![AddDC4][AddDC4]

5.  在“网络凭据”页上，确保你正在 **corp.contoso.com** 域中安装域控制器，并键入 Domain Admins 组的成员的凭据（或使用 corp\\administrator 凭据）。

    ![AddDC5][AddDC5]

6.  在“选择域”页上，单击“下一步”。

    ![AddDC6][AddDC6]

7.  在“选择站点”页上，确保已选定 CloudSite，然后单击“下一步”。

    ![AddDC7][AddDC7]

8.  在“其他域控制器选项”页上，单击“下一步”。

    ![AddDC8][AddDC8]

9.  在静态 IP 分配警告中，单击“是，该计算机将使用 DHCP 服务器自动分配的 IP 地址(不推荐)”。

    **重要说明**

    尽管 Azure 虚拟网络上的 IP 地址是动态的，但该地址的租赁在 VM 的持续时间内保持有效。因此，你无需在安装到虚拟网络上的域控制器上设置静态 IP 地址。在虚拟机中设置静态 IP 地址将导致通信失败。

    ![AddDC9][AddDC9]

10. 当系统给出了 DNS 委派警告提示时，请单击“是”。

    ![AddDC10][AddDC10]

11. 在“Active Directory 数据库、日志文件和 SYSVOL 的位置”页上，单击“浏览”并为 Active Directory 文件键入或选择数据磁盘上的位置，然后单击“下一步”。

    ![AddDC11][AddDC11]

12. 在“目录服务还原管理员”页上，键入并确认 DSRM 密码，然后单击“下一步”。

    ![AddDC12][AddDC12]

13. 在“摘要”页上，单击“下一步”。

    ![AddDC13][AddDC13]

14. 在 Active Directory 安装向导完成后，单击“完成”，然后单击“立即重新启动”以完成安装。

    ![AddDC14][AddDC14]

## <span id="validate"></span></a>步骤 5：验证安装

1.  重新连接到虚拟机。

2.  单击“开始”，右键单击“命令提示符”并单击“以管理员身份运行”。

3.  键入以下命令并按 Enter：'Dcdiag /c /v'

4.  验证测试是否已成功运行。

配置 DC 后，运行以下 Windows PowerShell cmdlet 来设置其他虚拟机并使其在设置后自动加入域。设置虚拟机时必须配置虚拟机的 DNS 客户端解析器设置。将你的域、虚拟机名称等项替换为正确的名称。

有关使用 Windows PowerShell 的详细信息，请参阅 [Azure PowerShell 入门][Azure PowerShell 入门]和 [Azure 管理 Cmdlet][Azure 管理 Cmdlet]。

## <span id="provisionvm"></span></a>步骤 6：设置在启动时加入域的虚拟机

1.  若要创建其他在首次启动时加入域的虚拟机，请打开 Azure PowerShell ISE，粘贴以下脚本，将占位符替换为你自己的值并运行该脚本。

    若要确定域控制器的内部 IP 地址，请单击运行该控制器的虚拟网络的名称。

    在以下示例中，域控制器的内部 IP 地址为 10.4.3.1。Add-AzureProvisioningConfig 还采用 -MachineObjectOU 参数，如果指定该参数（需要 Active Directory 中的完全可分辨名称），则可以在该容器中的所有虚拟机上设置组策略设置。

    设置虚拟机后，通过使用用户主体名称 (UPN) 格式指定域帐户（如 <administrator@corp.contoso.com>）进行登录。

        #Deploy a new VM and join it to the domain
        #-------------------------------------------
        #Specify my DC's DNS IP (10.4.3.1)
        $myDNS = New-AzureDNS -Name 'ContosoDC13' -IPAddress '10.4.3.1'

        # OS Image to Use
        $image = 'MSFT__Sql-Server-11EVAL-11.0.2215.0-08022012-en-us-30GB.vhd'
        $service = 'myazuresvcindomainM1'
        $AG = 'YourAffinityGroup'
        $vnet = 'YourVirtualNetwork'
        $pwd = 'p@$$w0rd'
        $size = 'Small'

        #VM Configuration
        $vmname = 'MyTestVM1'
        $MyVM1 = New-AzureVMConfig -name $vmname -InstanceSize $size -ImageName $image |
            Add-AzureProvisioningConfig -WindowsDomain -Password $pwd -Domain 'corp' -DomainPassword 'p@$$w0rd' -DomainUserName 'Administrator' -JoinDomain 'corp.contoso.com'|
            Set-AzureSubnet -SubnetNames 'BackEnd'

        New-AzureVM -ServiceName $service -AffinityGroup $AG -VMs $MyVM1 -DnsSettings $myDNS -VNetName $vnet

## <span id="backup"></span></a>步骤 7：备份域控制器

1.  连接到 YourVMachine。

2.  依次单击“开始”、“服务器管理器”和“添加功能”，然后选择“Windows Server Backup 功能”。按照说明进行操作来安装 Windows Server Backup。

3.  依次单击“开始”、“Windows Server Backup”和“一次性备份”。

4.  单击“其他选项”，然后单击“下一步”。

5.  单击“完全服务器”，然后单击“下一步”。

6.  单击“本地驱动器”，然后单击“下一步”。

7.  选择不托管操作系统文件或 Active Directory 数据库的目标驱动器，然后单击“下一步”。

    ![BackupDC][BackupDC]

8.  确认所选的备份设置，然后单击“备份”。

## <span id="test"></span></a>步骤 8：测试身份验证和授权

1.  若要测试身份验证和授权，请在 Active Directory 中创建域用户帐户。
    登录到每个站点中的客户端虚拟机，然后在该虚拟机上创建共享文件夹

2.  使用其他帐户、组和权限测试对共享文件夹的访问。

## 另请参阅

-   [Azure 虚拟网络][Azure 虚拟网络]

-   [Windows Azure IT Pro IaaS：(01) 虚拟机基础知识][Windows Azure IT Pro IaaS：(01) 虚拟机基础知识]

-   [Windows Azure IT Pro IaaS：(05) 创建虚拟网络和跨界连接][Windows Azure IT Pro IaaS：(05) 创建虚拟网络和跨界连接]

-   [Azure PowerShell][Azure PowerShell 入门]

-   [Azure 管理 Cmdlet][Azure 管理 Cmdlet]

  [Azure 虚拟网络]: http://msdn.microsoft.com/en-us/library/windowsazure/jj156007.aspx
  [在 Azure 虚拟机上部署 Windows Server Active Directory 的指南]: http://msdn.microsoft.com/en-us/library/windowsazure/jj156090.aspx
  [先决条件]: #Prerequisites
  [步骤 1：验证 YourPrimaryDC 的静态 IP 地址]: #verifystaticip
  [步骤 2：安装 Corp 林]: #installforest
  [步骤 3：创建子网和站点]: #subnets
  [步骤 4：在 CloudSite 中安装附加域控制器]: #cloudsite
  [步骤 5：验证安装]: #validate
  [步骤 6：设置在启动时加入域的虚拟机]: #provisionvm
  [步骤 7：备份域控制器]: #backup
  [步骤 8：测试身份验证和授权]: #test
  [在管理门户中配置站点到站点的 VPN]: http://msdn.microsoft.com/en-us/library/dn133795.aspx
  [将虚拟机添加到虚拟网络（可能为英文页面）]: http://windowsazure.cn/zh-cn/documentation/articles/virtual-networks-add-virtual-machine/
  [创建区域委派]: http://technet.microsoft.com/library/cc753500.aspx
  [在 Azure 中安装新的 Active Directory 林]: /en-us/manage/services/networking/active-directory-forest/
  [从域中删除域控制器]: http://technet.microsoft.com/en-us/library/cc771844(v=WS.10).aspx
  [VerifystaticIPaddressyourPrimaryDC1]: ./media/virtual-networks-install-replica-active-directory-domain-controller/VerifystaticIP.png
  [InstallCorpForest1]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest1.png
  [InstallCorpForest2]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest2.png
  [InstallCorpForest3]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest3.png
  [InstallCorpForest4]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest4.png
  [InstallCorpForest5]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest5.png
  [InstallCorpForest6]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest6.png
  [InstallCorpForest7]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest7.png
  [InstallCorpForest8]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest8.png
  [InstallCorpForest9]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest9.png
  [InstallCorpForest10]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest10.png
  [InstallCorpForest11]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest11.png
  [InstallCorpForest12]: ./media/virtual-networks-install-replica-active-directory-domain-controller/InstallCorpForest12.png
  [CreateSubnetsandSites1]: ./media/virtual-networks-install-replica-active-directory-domain-controller/CreateSubnetsandSites1.png
  [CreateSubnetsandSites2]: ./media/virtual-networks-install-replica-active-directory-domain-controller/CreateSubnetsandSites2.png
  [CreateSubnetsandSites3]: ./media/virtual-networks-install-replica-active-directory-domain-controller/CreateSubnetsandSites3.png
  [CreateSubnetsandSites4]: ./media/virtual-networks-install-replica-active-directory-domain-controller/CreateSubnetsandSites4.png
  [CreateSubnetsandSites5]: ./media/virtual-networks-install-replica-active-directory-domain-controller/CreateSubnetsandSites5.png
  [CreateSubnetsandSites6]: ./media/virtual-networks-install-replica-active-directory-domain-controller/CreateSubnetsandSites6.png
  [CreateSubnetsandSites7]: ./media/virtual-networks-install-replica-active-directory-domain-controller/CreateSubnetsandSites7.png
  [AddDC1]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC1.png
  [AddDC2]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC2.png
  [AddDC3]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC3.png
  [AddDC4]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC4.png
  [AddDC5]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC5.png
  [AddDC6]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC6.png
  [AddDC7]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC7.png
  [AddDC8]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC8.png
  [AddDC9]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC9.png
  [AddDC10]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC10.png
  [AddDC11]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC11.png
  [AddDC12]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC12.png
  [AddDC13]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC13.png
  [AddDC14]: ./media/virtual-networks-install-replica-active-directory-domain-controller/AddDC14.png
  [Azure PowerShell 入门]: http://msdn.microsoft.com/en-us/library/windowsazure/jj156055.aspx
  [Azure 管理 Cmdlet]: http://msdn.microsoft.com/en-us/library/windowsazure/jj152841
  [BackupDC]: ./media/virtual-networks-install-replica-active-directory-domain-controller/BackupDC.png
  [Windows Azure IT Pro IaaS：(01) 虚拟机基础知识]: http://channel9.msdn.com/Series/Windows-Azure-IT-Pro-IaaS/01
  [Windows Azure IT Pro IaaS：(05) 创建虚拟网络和跨界连接]: http://channel9.msdn.com/Series/Windows-Azure-IT-Pro-IaaS/05
