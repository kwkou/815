<properties linkid="manage-windows-commontask-install-sql-server" urlDisplayName="Install SQL Server" pageTitle="Provision a SQL Server virtual machine in Azure " metaKeywords="Azure tutorial creating SQL Server, SQL Server vm, configuring SQL Server" description="A tutorial that teaches you how to create and configure a SQL Server virtual machine on Azure." metaCanonical="" services="virtual-machines" documentationCenter="" title="Provisioning a SQL Server Virtual Machine on Azure" authors="selcint" solutions="" manager="clairt" editor="tyson" />





# 在 Azure 上设置 SQL Server 虚拟机#

Azure 虚拟机库包括几种内含 Microsoft SQL Server 的映像。你可以从库中选择虚拟机映像之一，只需要单击几次，即可将虚拟机设置到你的 Azure 环境。

在本教程中，你将：

* [从库连接到 Azure 管理门户并设置虚拟机](#Provision)
* [使用远程桌面和完整安装打开虚拟机](#RemoteDesktop)
* [完成配置步骤以便在另一台计算机上使用 SQL Server Management Studio 连接到虚拟机](#SSMS)
* [后续步骤](#Optional)

## <span id="Provision"></span>从库连接到 Azure 管理门户并设置虚拟机

1.  使用你的帐户登录到 [Azure 管理门户](http://manage.windowsazure.cn)。

2.  在 Azure 管理门户中网页的左下角，依次单击“+新建”、“计算”、“虚拟机”、“从库中”。

3.  在“创建虚拟机”页面上，选择包含 SQL Server 的虚拟机映像，然后单击该页右下角的“下一步”箭头。有关在 Azure 上受支持的 SQL Server 映像的最新信息，请参阅 [Azure 虚拟机中的 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/dn133151.aspx) 文档集中的 [Azure 中的 SQL Server 虚拟机入门](http://msdn.microsoft.com/zh-cn/library/azure/jj823132.aspx)主题。

    > [WACOM.NOTE] 如果虚拟机是通过使用平台映像 SQL Server 评估版创建的，则无法将其升级到库中的按分钟付费版本的映像。可以选择以下两个选项之一：

    > -   你可以通过从库中使用按分钟付费 SQL Server 版本创建一台新虚拟机，并按[如何使用数据磁盘在 Azure 中的虚拟机之间迁移 SQL Server 数据库文件和架构](http://msdn.microsoft.com/zh-cn/library/azure/jj898505.aspx)中所述步骤将数据库文件迁移到这台新虚拟机。**或者**，

    > -   根据[在 Azure 上通过软件保证实现许可迁移](http://azure.microsoft.com/zh-cn/pricing/license-mobility/)协议，通过[升级到 SQL Server 2014 的不同版本](http://msdn.microsoft.com/zh-cn/library/cc707783(v=sql.120).aspx)中所述的步骤，将 SQL Server 评估版的现有实例升级到 SQL Server 的另一版本。有关如何购买 SQL Server 的许可副本的信息，请参阅[如何购买 SQL Server](http://www.microsoft.com/zh-cn/server-cloud/products/sql-server/buy.aspx#fbid=t8CT8yhDl9X)。

4.  在“虚拟机配置”页上，提供以下信息：
	-   提供一个“虚拟机名称”。
	-   在“新用户名”框中，键入用于虚拟机本地管理员帐户的唯一用户名。
	-   在**“新密码”**框中，键入一个强密码。有关详细信息，请参阅[强密码](http://msdn.microsoft.com/zh-cn/library/ms161962.aspx)。
	-   在**“确认密码”**框中，重新键入该密码。
	-   从下拉列表中选择合适的**“大小”**。

	> [WACOM.NOTE] 虚拟机的大小是在设置期间指定的：
	> A2 是推荐用于生产负载的最小大小。
    > 在使用 SQL Server Enterprise Edition 时推荐用于虚拟机的最小大小是 A3。
    > 在使用 SQL Server Enterprise Edition 时请选择 A3 或更高的值。
   	> 在使用 SQL Server 2012 Enterprise for Data Warehousing 映像时请选择 A6。
   	> 在使用 SQL Server 2014 for Data Warehousing 映像时请选择 A7。
   	> 所选的大小会限制你可以配置的数据磁盘的数量。有关可用虚拟机大小和可附加到虚拟机的数据磁盘数目的最新信息，请参阅[用于 Azure 的虚拟机大小](http://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx)。

	单击右下角的“下一步”箭头以继续。

	![VM 配置](./media/virtual-machines-provision-sql-server/4VM-Config.png)

5.  在**“虚拟机模式”**页上，提供以下信息：
	-   选择“独立虚拟机”。
	-   在“DNS 名称”框中，提供所选 DNS 名称的第一部分，让它完成时的格式是 **TESTNAME.cloudapp.net**
	-   在“区域/地缘组/虚拟网络”框中，选择将托管此虚拟映像的区域。

	单击下一步箭头以继续。

	![VM 模式][Image5]

6.  在“虚拟机选项”页上：
	-   在“可用性集”框中，选择“(无)”。
	-   阅读并接受法律条款。

	![VM 选项][Image6]

7.  单击右下角的对号标记以继续。

8.  请等候 Azure 准备你的虚拟机。预计虚拟机的状态将出现如下变化：

	-   正在启动（正在配置）
	-   已停止
	-   正在启动（正在配置）
	-   正在运行（正在配置）
	-   正在运行
	

## <span id="RemoteDesktop"></span>使用远程桌面和完整安装打开虚拟机

1.  当设置完成时，单击你的虚拟机的名称，以转到“仪表板”页面。在页面底部，单击“连接”。

	![选择仪表板页][Image5b]
2.  选择使用 Windows 远程桌面程序 (`%windir%\system32\mstsc.exe`) 打开 .rpd 文件。

	
3.  在“Windows 安全性”对话框中，提供你在前面步骤中指定的本地管理员帐户的密码。（系统可能会要求你验证虚拟机的凭据。）

4.  首次登录到此虚拟机时，可能需要完成若干过程，其中包括设置桌面、更新 Windows 和完成 Windows 初始配置任务 (sysprep)。Windows sysprep 完成后，SQL Server 安装程序将完成配置任务。这些任务在其完成过程中可能会导致短时延迟。`SELECT @@SERVERNAME` 可能要等 SQL Server 安装程序完成之后才会返回正确名称。

用 Windows 远程桌面连接到该虚拟机后，该虚拟机即可像任何其他计算机一样工作。以正常方式通过 SQL Server Management Studio（在虚拟机上运行）连接到 SQL Server 的默认实例。

## <span id="SSMS"></span>完成配置步骤以便在另一台计算机上使用 SQL Server Management Studio 连接到虚拟机

你必须先完成下列各节中描述的下列任务，然后才能通过 Internet 连接到 SQL Server 的实例：

-   [为虚拟机创建 TCP 终结点](#Endpoint)
-   [在 Windows 防火墙中打开 TCP 端口](#FW)
-   [将 SQL Server 配置为侦听 TCP 协议](#TCP)
-   [配置混合模式的 SQL Server 身份验证](#Mixed)
-   [创建 SQL Server 身份验证登录名](#Logins)
-   [确定虚拟机的 DNS 名称](#DNS)
-   [从其他计算机连接到数据库引擎](#cde)
-   [从应用程序连接到数据库引擎](#cdea)

下图中概述了此连接路径：

![连接到 SQL Server 虚拟机][Image8b]

### <span id="Endpoint"></span>为虚拟机创建 TCP 终结点

虚拟机必须具有终结点以侦听传入的 TCP 通信。此 Azure 配置步骤将传入 TCP 端口通信定向到虚拟机可以访问的 TCP 端口。

1.  在 Azure 管理门户上，单击**“虚拟机”**。

	
2.  单击你新创建的虚拟机。将显示有关你的虚拟机的信息。
	

3.  在靠近页面顶端的位置，选择“终结点”页面，然后在页面底部单击“添加终结点”。

	![单击添加终结点][Image28]

4.  在“给虚拟机添加终结点”页上，单击“添加终结点”，然后单击“下一步”箭头以继续。

	![单击添加终结点][Image29]

5.  在“指定终结点的详细信息”页面上，提供以下信息。

	-   在“名称”框中，为终结点提供名称。
	-   在“协议”框中，选择“TCP”。可在“专用端口”框中键入 SQL Server 的默认侦听端口 **1433**。同样，你可以在“公用端口”框中键入 **57500**。注意，许多组织选择其他端口号以避免恶意的安全攻击。


	![终结点屏幕][Image30]

6.  单击复选标记以继续。终结点创建完成。

	![带有终结点的 VM][Image31]

### <span id="FW"></span>在 Windows 防火墙中为数据库引擎的默认实例打开 TCP 端口

1.  通过 Windows 远程桌面连接到虚拟机。登录后，在“开始”菜单上单击“运行”，键入 **WF.msc**，然后单击“确定”。

	![启动防火墙程序][Image12]
2.  在**“高级安全 Windows 防火墙”**的左窗格中，右键单击**“入站规则”**，然后在操作窗格中单击**“新建规则”**。

	![新建规则][Image13]

3.  在**“规则类型”**对话框中，选择**“端口”**，然后单击**“下一步”**。

4.  在**“协议和端口”**对话框中，选择 **TCP**。选择“特定本地端口”，然后键入数据库引擎实例的端口号（即默认实例对应的端口号 **1433**，或你在终结点步骤中为专用端口选择的端口号）。

	![TCP 端口 1433][Image14]

5.  单击“下一步”。

6.  在**“操作”**对话框中，选择**“允许连接”**，然后单击**“下一步”**。

	**安全说明：** 选择“只允许安全连接”可增加安全性。如果你想在你的环境中配置其他安全性选项，请选择此选项。

	![允许连接][Image15]

7.  在“配置文件”对话框中，选择“公用”，然后单击**“下一步”**。

    **安全说明：** 选择“公用”允许通过 Internet 进行访问。只要有可能，就请选择更具限制性的配置文件。

	![公用配置文件][Image16]

8.  在“名称”对话框中，键入此规则的名称和说明，然后单击“完成”。

	![规则名称][Image17]

根据需要为其他组件打开附加端口。有关详细信息，请参阅[配置 Windows 防火墙以允许 SQL Server 访问](http://msdn.microsoft.com/zh-cn/library/cc646023.aspx)。


### <span id="TCP"></span>将 SQL Server 配置为侦听 TCP 协议

1.  使用远程桌面连接到虚拟机后，在“开始”菜单上，依次单击“所有程序”、Microsoft SQL Server *版本*、“配置工具”、“SQL Server 配置管理器”。

	![打开 SSCM][Image9]

2.  在“SQL Server 配置管理器”的控制台窗格中，展开“SQL Server 网络配置”。

3.  在控制台窗格中，单击“*实例名称* 的协议”。（默认实例为“MSSQLSERVER 的协议”。）

4.  在详细信息窗格中，右键单击“TCP”，默认情况下该协议对于库映像应为“已启用”状态。对于你的自定义映像，单击**“启用”**（如果其状态为“已禁用”）。

	![启用 TCP][Image10]

5.  在控制台窗格中，单击“SQL Server 服务”。（数据库引擎的重新启动可能会延迟到下一步完成之后。）

6.  在详细信息窗格中，右键单击“SQL Server (*实例名*)”（默认实例为“SQL Server (MSSQLSERVER)”），然后单击“重新启动”以停止并重新启动该 SQL Server 实例。

    ![重新启动数据库引擎][Image11]

7.  关闭 SQL Server 配置管理器。

有关启用 SQL Server 数据库引擎的协议的详细信息，请参阅[启用或禁用服务器网络协议](http://msdn.microsoft.com/zh-cn/library/ms191294.aspx)。

### <span id="Mixed"></span>配置混合模式的 SQL Server 身份验证

在没有域环境的情况下，SQL Server 数据库引擎无法使用 Windows 身份验证。若要从其他计算机连接到数据库引擎，请将 SQL Server 的身份验证模式配置为混合。混合模式身份验证同时允许 SQL Server 身份验证和 Windows 身份验证。（如果你已经配置了 Azure 虚拟网络，可能没有必要配置混合模式身份验证。有关详细信息，请参阅 [Azure 虚拟机中的 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/dn133152.aspx) 文档集中的 [Azure 虚拟机中 SQL Server 的连接性注意事项](http://msdn.microsoft.com/zh-cn/library/azure/jj823132.aspx)主题。

1.  使用远程桌面连接到虚拟机后，在“开始”菜单上，依次单击“所有程序”、Microsoft SQL Server *版本*、SQL Server Management Studio。

	![启动 SSMS][Image18]

	Management Studio 在首次打开时，一定会创建用户 Management Studio 环境。这可能需要一小段时间。

2.  打开时，Management Studio 会显示“连接到服务器”对话框。在“服务器名称”框中，键入要连接到对象资源管理器中的数据库引擎的虚拟机的名称。（你还可使用“(local)”或一个句点代替虚拟机名称作为“服务器名称”。选择“Windows 身份验证”，在“用户名”框中保留 ***your\_VM\_name*\\your\_local\_administrator**。单击“连接”。

	![连接到服务器][Image19]

3.  在 SQL Server Management Studio 的“对象资源管理器”中，右键单击 SQL Server 实例的名称（虚拟机名称），然后单击**“属性”**。

	![服务器属性][Image20]

4.  在“安全性”页上的“服务器身份验证”下，选择“SQL Server 和 Windows 身份验证模式”，然后单击“确定”。

	![选择身份验证模式][Image21]

5.  在 SQL Server Management Studio 对话框中，单击“确定”,以接受重新启动 SQL Server 的要求。

6.  在“对象资源管理器”中，右键单击你的服务器，然后单击“重新启动”。（如果 SQL Server 代理正在运行，它也必须重新启动。）

	![重新启动][Image22]

7.  在 SQL Server Management Studio 对话框中，单击“是”以同意重新启动 SQL Server。

### <span id="Logins"></span>创建 SQL Server 身份验证登录名

若要从其他计算机连接到数据库引擎，你必须创建至少一个 SQL Server 身份验证登录名。

1.  在 SQL Server Management Studio 对象资源管理器中，展开你要在其中创建新登录名的服务器实例所在的文件夹。

2.  右键单击“安全性”文件夹，指向“新建”，然后选择“登录名…”。

	![新建登录名][Image23]

3.  在“登录名 - 新建”对话框中的“常规”页上，在“登录名”框中输入新用户的名称。

4.  选择“SQL Server 身份验证”。

5.  在“密码”框中，输入新用户的密码。在“确认密码”框中再次输入该密码。

6.  若要强制实施针对复杂性和强制实施的密码策略选项，请选择“强制实施密码策略”（推荐）。这是选择 SQL Server 身份验证时的默认选项。

7.  若要强制实施针对过期的密码策略选项，请选择“强制密码过期”（推荐）。必须选择强制密码策略才能启用此复选框。这是选择 SQL Server 身份验证时的默认选项。

8.  若要强制用户在首次使用登录名后创建新密码，请选择“用户在下次登录时必须更改密码”（如果此登录名给其他人使用，推荐选择此选项。如果登录名是为了自用，请勿选择此选项。）必须选择强制密码过期才能启用此复选框。这是选择 SQL Server 身份验证时的默认选项。

9.  从“默认数据库”列表中，为该登录名选择默认数据库。**master** 是此选项的默认值。如果你尚未创建用户数据库，则保留此设置为 **master**。

10. 在“默认语言”列表中，保留“默认”值。

	![登录名属性][Image24]

11. 如果这是你创建的第一个登录名，可能会需要将此登录名指派为 SQL Server 管理员。这样的话，请在“服务器角色”页面上选中 **sysadmin**。

	**安全说明：** sysadmin 固定服务器角色的成员对数据库引擎具有完全控制权限。应谨慎限制此角色中的成员资格。

	![sysadmin][Image25]

12. 单击“确定”。

有关 SQL Server 登录名的详细信息，请参阅[创建登录名](http://msdn.microsoft.com/zh-cn/library/aa337562.aspx)。



### <span id="DNS"></span>确定虚拟机的 DNS 名称

若要从另一台计算机连接到 SQL Server 数据库引擎，必须知道虚拟机的域名系统 (DNS) 名称。（这是 Internet 用于识别虚拟机的名称）。可以使用 IP 地址，但 IP 地址在 Azure 为冗余或维护而移动资源时可能会变更。DNS 名称将保持不变，因为可将该名称重定向到新的 IP 地址。）

1.  在 Azure 管理门户（或在完成前一步后），选择“虚拟机”。

2.  在“虚拟机实例”页面上的“DNS 名称”列中，找到并复制带有前缀 **http://** 的虚拟机 DNS 名称。（在用户界面上可能显示不出整个名称，不过没关系，你可以右键单击它，并选择“复制”。）

	![DNS 名称][Image32]

### <span id="cde"></span>从其他计算机连接到数据库引擎

1.  在连接到 Internet 的计算机上，打开 SQL Server Management Studio。

2.  在“连接到服务器”或“连接到数据库引擎”对话框的“服务器名称”框中，按 *DNSName,portnumber* 的格式输入（在前一任务中确定的）虚拟机的 DNS 名称和公用终结点端口号，例如 **tutorialtestVM.chinacloudapp.cn,57500**。

3.  在“身份验证”框中，选择“SQL Server 身份验证”。

4.  在“登录名”框中，键入你在前面的任务中创建的登录名。

5.  在“密码”框中，键入你在前面的任务中创建的登录名的密码。

6.  单击“连接”。

	![使用 SSMS 进行连接][Image33]

### <span id="cdea"></span> 从应用程序连接到数据库引擎

如果你可以使用 Management Studio 连接到 Azure 虚拟机上运行的 SQL Server 的实例，就应该能够使用类似于下面这样的连接字符串来连接。

	connectionString="Server=<DNS_Name>;Integrated Security=false;User ID=<login_name>;Password=<your_password>;"providerName="System.Data.SqlClient"

有关详细信息，请参阅[如何解决 SQL Server 数据库引擎的连接问题](http://social.technet.microsoft.com/wiki/contents/articles/2102.how-to-troubleshoot-connecting-to-the-sql-server-database-engine.aspx)。

## <span id="Optional"></span>后续步骤

你已经看到了如何使用平台映像在 Azure 虚拟机上创建和配置 SQL Server。在使用 Azure 虚拟机中的 SQL Server 时，建议遵循库中 [Azure 虚拟机中的 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/jj823132.aspx) 文档中提供的详细指导。该文档集包含一系列提供详细指导的文章和教程。该系列包括以下各部分：

[Azure 虚拟机中的 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/jj823132.aspx)

[Azure 虚拟机中的 SQL Server 入门](http://msdn.microsoft.com/zh-cn/library/azure/dn133151.aspx)

[准备迁移到 Azure 虚拟机中的 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/dn133142.aspx)

-   如何使用数据磁盘在 Azure 中的虚拟机之间迁移 SQL Server 数据库文件和架构

[Azure 虚拟机中的 SQL Server 部署](http://msdn.microsoft.com/zh-cn/library/azure/dn133141.aspx)

-   如何使用 CSUpload 将数据磁盘中的 SQL Server 数据和安装文件从本地复制到 Azure
-   如何使用 Hyper-V 在本地创建基础虚拟机
-   如何使用现有本地 SQL Server 磁盘在 Azure 中创建 SQL Server 虚拟机
-   如何使用现有本地 SQL Server 虚拟机在 Azure 中创建 SQL Server 虚拟机
-   如何使用 PowerShell 在 Azure 中设置 SQL Server 虚拟机
-   如何使用附加的数据磁盘存储数据库文件

[Azure 虚拟机中的 SQL Server 的连接注意事项](http://msdn.microsoft.com/zh-cn/library/azure/dn133152.aspx)

-   教程：连接到同一云服务中的 SQL Server
-   教程：连接到不同云服务中的 SQL Server
-   教程：通过虚拟网络将 ASP.NET 应用程序连接到 Azure 中的 SQL Server

[Azure 虚拟机中的 SQL Server 的性能注意事项](http://msdn.microsoft.com/zh-cn/library/azure/dn133149.aspx)

[Azure 虚拟机中的 SQL Server 的安全注意事项](http://msdn.microsoft.com/zh-cn/library/azure/dn133147.aspx)

[Azure 虚拟机中 SQL Server 的故障排除和监视](http://msdn.microsoft.com/zh-cn/library/azure/dn195883.aspx)

[Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](http://msdn.microsoft.com/zh-cn/library/azure/jj870962.aspx)

- 教程：Azure 中的 AlwaysOn 可用性组 (GUI)
- 教程：Azure 中的 AlwaysOn 可用性组 (PowerShell)
- 教程：混合 IT 环境中的 AlwaysOn 可用性组 (PowerShell)
- 教程：Azure 中 AlwaysOn 可用性组的侦听器配置
- 教程：HybridIT 中 AlwaysOn 可用性组的侦听器配置
- 教程：在 Azure 中进行数据库镜像以实现灾难恢复
- 教程：混合 IT 环境中用于实现灾难恢复的数据库镜像
- 教程：在 Azure 中进行数据库镜像以实现高可用性
- 教程：在混合 IT 环境中进行日志传送以实现灾难恢复
- Azure 中可用性组侦听器的故障排除

[Azure 虚拟机中 SQL Server 的备份和还原](http://msdn.microsoft.com/zh-cn/library/azure/dn133143.aspx)

[Azure 虚拟机中的 SQL Server Business Intelligence](http://msdn.microsoft.com/zh-cn/library/azure/jj992719.aspx)

- 使用 PowerShell 创建运行 SQL Server BI 和 SharePoint 2010 的 Azure VM
- 使用 PowerShell 创建运行 SQL Server BI 和 SharePoint 2013 的 Azure VM
- 使用 PowerShell 创建运行本机模式报表服务器的 Azure VM

[Azure 虚拟机中的 SQL Server 数据仓库](http://msdn.microsoft.com/zh-cn/library/azure/dn387396.aspx)

[Azure 虚拟机中的 SQL Server 技术文章](http://msdn.microsoft.com/zh-cn/library/azure/dn248435.aspx)

- [白皮书：Azure 虚拟机中的 SQL Server 的应用程序模式和开发策略](http://msdn.microsoft.com/zh-cn/library/azure/dn574746.aspx)

- [白皮书：在 Azure 虚拟机中部署 SQL Server Business Intelligence](http://msdn.microsoft.com/zh-cn/library/windowsazure/dn321998.aspx)

- [白皮书：Azure 虚拟机中 SQL Server 的性能指南](http://msdn.microsoft.com/zh-cn/library/windowsazure/dn248436.aspx)

[Image5]: ./media/virtual-machines-provision-sql-server/5VM-Mode.png
[Image5b]: ./media/virtual-machines-provision-sql-server/5VM-Connect.png
[Image6]: ./media/virtual-machines-provision-sql-server/6VM-Options.png
[Image8b]: ./media/virtual-machines-provision-sql-server/SQLVMConnectionsOnAzure.GIF
[Image9]: ./media/virtual-machines-provision-sql-server/9Click-SSCM.png
[Image10]: ./media/virtual-machines-provision-sql-server/10Enable-TCP.png
[Image11]: ./media/virtual-machines-provision-sql-server/11Restart.png
[Image12]: ./media/virtual-machines-provision-sql-server/12Open-WF.png
[Image13]: ./media/virtual-machines-provision-sql-server/13New-FW-Rule.png
[Image14]: ./media/virtual-machines-provision-sql-server/14Port-1433.png
[Image15]: ./media/virtual-machines-provision-sql-server/15Allow-Connection.png
[Image16]: ./media/virtual-machines-provision-sql-server/16Public-Profile.png
[Image17]: ./media/virtual-machines-provision-sql-server/17Rule-Name.png
[Image18]: ./media/virtual-machines-provision-sql-server/18Start-SSMS.png
[Image19]: ./media/virtual-machines-provision-sql-server/19Connect-to-Server.png
[Image20]: ./media/virtual-machines-provision-sql-server/20Server-Properties.png
[Image21]: ./media/virtual-machines-provision-sql-server/21Mixed-Mode.png
[Image22]: ./media/virtual-machines-provision-sql-server/22Restart2.png
[Image23]: ./media/virtual-machines-provision-sql-server/23New-Login.png
[Image24]: ./media/virtual-machines-provision-sql-server/24Test-Login.png
[Image25]: ./media/virtual-machines-provision-sql-server/25sysadmin.png
[Image28]: ./media/virtual-machines-provision-sql-server/28Add-Endpoint.png
[Image29]: ./media/virtual-machines-provision-sql-server/29Add-Endpoint-to-VM.png
[Image30]: ./media/virtual-machines-provision-sql-server/30Endpoint-Details.png
[Image31]: ./media/virtual-machines-provision-sql-server/31VM-Connect.png
[Image32]: ./media/virtual-machines-provision-sql-server/32DNS-Name.png
[Image33]: ./media/virtual-machines-provision-sql-server/33Connect-SSMS.png