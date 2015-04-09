<properties linkid="manage-windows-commontask-install-sql-server" urlDisplayName="Install SQL Server" pageTitle="设置在 Azure 中的 SQL Server 虚拟机 " metaKeywords="Azure tutorial creating SQL Server, SQL Server vm, configuring SQL Server" description="本教程演示如何创建和配置 SQL Server 虚拟机在 Azure 上。" metaCanonical="" services="virtual-machines" documentationCenter="" title="Provisioning a SQL Server Virtual Machine on Azure" authors="selcint" solutions="" manager="clairt" editor="tyson" />





# 设置 SQL Server 虚拟机在 Azure 的 #

Azure 虚拟机库包括若干含有 Microsoft SQL Server 的映像。您可以从库中选择一个虚拟机映像，并单击几下，您可以设置虚拟机到 Azure 环境。

在本教程中，您将：

* [连接到 Azure 管理门户并设置虚拟机从库](#Provision)
* [打开虚拟机使用远程桌面和完整安装](#RemoteDesktop)
* [将连接到虚拟机在另一台计算机上使用 SQL Server Management Studio 完成配置步骤](#SSMS)
* [接下来的步骤](#Optional)

##< id ="设置"> 连接到 Azure 管理门户和设置虚拟机从库</a>

1. 登录到[Azure 管理门户](http://manage.windowsazure.cn)使用你的帐户。 

2. 在 Azure 管理门户中，在左下角的网页上，单击**+ 新建**，单击**计算**，单击**虚拟机**，然后单击**从库**。

3. 在**创建虚拟机**页上，选择包含 SQL Server 的虚拟机映像，然后单击右下角的页上的下一步箭头。有关受支持的 SQL Server 映像在 Azure 上的最新信息，请参阅[入门 Azure 虚拟机中 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/dn133151.aspx)中的主题[Azure 虚拟机中 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/jj823132.aspx)文档集。 

    >[WACOM.NOTE] 如果必须通过使用平台映像 SQL Server 评估版创建的虚拟机，不能将其升级到库中的每分钟付费的安装映像中。您可以选择以下两个选项之一：
    
    > - 您可以通过按分钟付费的库中的 SQL Server 版本中创建新的虚拟机，并通过遵循的步骤，将数据库文件迁移到这个新的虚拟机[如何使用数据磁盘在 Azure 中的虚拟机之间迁移 SQL Server 数据库文件和架构](http://msdn.microsoft.com/zh-cn/library/azure/jj898505.aspx)。**Or**,

    > - 可以将 SQL Server 评估版的现有实例升级到另一版本的 SQL Server 下[Azure 上通过软件保障许可移动性](/zh-cn/pricing/license-mobility/) 通过遵循的步骤的协议[升级到不同版本的 SQL Server 2014](http://msdn.microsoft.com/zh-cn/library/cc707783(v=sql.120).aspx)。有关如何购买 SQL Server 的许可的副本的信息，请参阅[如何购买 SQL Server](http://www.microsoft.com/zh-cn/server-cloud/products/sql-server/buy.aspx#fbid=t8CT8yhDl9X)。
   

4. 在第一天**虚拟机配置**页上，提供以下信息：
	- 提供**虚拟机名称**。
	- 在**新用户名**框中，为虚拟机本地管理员帐户类型唯一用户名。
	- 在**新密码**框中，键入强密码。有关详细信息，请参阅[强密码](http://msdn.microsoft.com/zh-cn/library/ms161962.aspx)。
	- 在**确认密码**框中，再次键入该密码。
	- 选择适当的**大小**从下拉列表。 

	>[WACOM.NOTE] 在设置期间指定虚拟机的大小：
 	> A2 是为生产工作负荷推荐的最小大小。 
    > 使用 SQL Server Enterprise Edition 时，为虚拟机的最小建议的大小是 A3。
    > 选择 A3 或更高版本，在使用 SQL Server Enterprise Edition 时。
   	> 对于事务工作负荷映像使用 SQL Server 2012 或 2014年企业优化时，请选择 A4。  
   	> 对于数据仓库工作负荷映像使用 SQL Server 2012 或 2014年企业优化时，请选择 A7。 
   	> 所选的大小限制可以配置的数据磁盘数。有关可用的虚拟机大小以及可以附加到虚拟机的数据磁盘数的最新信息，请参阅[Azure 的虚拟机大小](http://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx)。

	单击右下角，若要继续下一步箭头。

	![VM Configuration](./media/virtual-machines-provision-sql-server/4VM-Config.png)


5. 在第二个**虚拟机配置**页上，配置网络、 存储和可用性的资源：
	- 在**云服务**框中，选择**创建新的云服务**。
	- 在**云服务 DNS 名称**框中，提供您的选择，一个 DNS 名称的第一部分，使它完成的格式名称**TESTNAME.cloudapp.net** 
	- 在**区域/地缘组/虚拟网络**框中，选择将承载此虚拟映像的区域。
	- 在**存储帐户**，选择一个现有的存储帐户，或选择一个自动生成。
	- 在**可用性集**框中，选择**(无)**。
	- 阅读并接受法律条款。
	

6. 单击下一步箭头以继续。


7. 单击右下角的对号标记以继续。

8. 请等候 Azure 准备你的虚拟机。预计虚拟机的状态将出现如下变化：

	- 启动 (设置)
	- 已停止
	- 启动 (设置)
	- 运行 (设置)
	- 正在运行
	

##<a id="RemoteDesktop">使用远程桌面和完整安装打开虚拟机</a>

1. 配置完成后，单击您要转到仪表板页的虚拟机的名称。在页面的底部，单击**连接**。

	
2. 选择要打开.rpd 文件使用 Windows 远程桌面程序 (%windir%\system32\mstsc.exe)。

	
3. 在**Windows 安全性**对话框框中，为您在前面的步骤中指定的本地管理员帐户提供密码。（您可能需要验证虚拟机的凭据。）

4. 第一次登录到此虚拟机，多个进程可能需要完成，包括设置桌面、 Windows 更新和完成 Windows 初始配置任务 (sysprep)。Windows sysprep 完成后，SQL Server 安装程序完成后配置任务。这些任务在其完成过程中可能会导致短时延迟。SQL Server 安装程序完成之前，SELECT @@SERVERNAME' 可能不会返回正确的名称。

一旦连接到通过 Windows 远程桌面的虚拟机，虚拟机的工作方式非常像任何其他计算机一样。以正常方式连接到 SQL Server 与 SQL Server Management Studio （在虚拟机上运行） 的默认实例。 

##<a id="SSMS">完成配置步骤以便在另一台计算机上使用 SQL Server Management Studio 连接到虚拟机</a>

可以从 Internet 连接到 SQL Server 实例之前，必须完成以下任务，如以下各节中所述：

- [创建虚拟机的 TCP 终结点](#Endpoint)
- [在 Windows 防火墙中打开 TCP 端口](#FW)
- [配置 SQL Server 以侦听 TCP 协议](#TCP)
- [将 SQL Server 配置为混合的模式身份验证](#Mixed)
- [创建 SQL Server 身份验证登录名](#Logins)
- [确定虚拟机的 DNS 名称](#DNS)
- [从另一台计算机连接到数据库引擎](#cde)
- [从您的应用程序连接到数据库引擎](#cdea)

连接路径是由以下关系图的摘要:

![Connecting to a SQL Server virtual machine][Image8b]

###<a id="Endpoint">为虚拟机创建 TCP 终结点</a>

虚拟机必须具有终结点以侦听传入的 TCP 通信。此 Azure 配置步骤将传入 TCP 端口通信定向到虚拟机可以访问的 TCP 端口。

1. 在 Azure 管理门户中，单击**虚拟机**。

	
2. 单击你新创建的虚拟机。将显示有关你的虚拟机的信息。
	

3. 在靠近页面顶部，选择**终结点**页，然后再在页面的底部，单击**添加**。
	

4. 在上**到虚拟机添加终结点**页上，单击**将独立终结点添加**，然后单击下一步箭头以继续。

	
5. 在上**指定终结点的详细信息**页上，提供以下信息。

	- 在**名称**框中，提供终结点的名称。
	- 在**协议**框中，选择**TCP**。您也可以输入**57500**中**公用端口**框。同样，也可以键入 SQL Server 的默认侦听端口**1433年**中**专用端口**框。请注意许多组织都会选择不同的端口号以免恶意的安全攻击。 


6. 单击复选标记以继续。终结点创建完成。
	

###<a id="FW">在 Windows 防火墙中为数据库引擎的默认实例打开 TCP 端口</a>

1. 通过 Windows 远程桌面连接到虚拟机。登录后，在开始菜单上单击**运行**，类型**WF.msc**，然后单击**确定**。如果运行 Windows Server 2012 或更高版本），您也可以输入**WF**开始菜单，然后再选择**具有高级安全性的 Windows 防火墙**。

	![Start the Firewall Program][Image12]
2. 在**具有高级安全性的 Windows 防火墙**，在左窗格中，右键单击**入站规则**，然后单击**新规则**操作窗格中。

	![New Rule][Image13]

3. 在**规则类型**对话框中，选择**端口**，然后单击**下一步**。

4. 在**协议和端口**对话框中，选择**TCP**。选择**特定本地端口**，然后键入数据库引擎的实例的端口号 (**1433年**对于默认实例或您所选终结点步骤中的专用端口）。 

	![TCP Port 1433][Image14]

5. 单击**下一步**。

6. 在**操作**对话框中，选择**允许连接**，然后单击**下一步**。

	**安全说明：**选择**允许连接，如果它是安全的**可以提供额外的安全性。如果您想要在您的环境中配置额外的安全选项，请选择此选项。

	![Allow Connections][Image15]

7. 在**配置文件**对话框中，选择**公共**，然后单击**下一步**。 

    **安全说明：**选择**公共**允许通过 internet 访问。只要有可能，请选择一个限制性更强的配置文件。

	![Public Profile][Image16]

8. 在**名称**对话框中，键入名称和该规则，说明然后单击**完成**。

	![Rule Name][Image17]

根据需要，请打开针对其他组件的其他端口。有关详细信息，请参阅[配置 Windows 防火墙以允许 SQL Server 访问](http://msdn.microsoft.com/zh-cn/library/cc646023.aspx)。


###<a id="TCP">将 SQL Server 配置为侦听 TCP 协议</a>

1. 在开始菜单上使用远程桌面连接到虚拟机，请单击**所有程序**，单击**Microsoft SQL Server** * 版本 * 单击**配置工具**，然后单击**SQL Server 配置管理器**。
	
	![Open SSCM][Image9]

2. 在 SQL Server 配置管理器中，在控制台窗格中，展开**SQL Server 网络配置**。

3. 在控制台窗格中，单击**协议 (_i) name_**。（默认实例是**MSSQLSERVER 的协议**。）

4. 在详细信息窗格中，右键单击 TCP，它应启用对于库映像默认情况下。有关自定义映像，请单击**启用**（如果其状态为已禁用。）

	![Enable TCP][Image10]

5. 在控制台窗格中，单击**SQL Server 服务**。（重新启动数据库引擎可以推迟到下一个步骤的完成。）

6. 在细节窗格中，右键单击**SQL Server ((_i) name_)** （默认实例是**SQL Server (MSSQLSERVER)**），然后单击**重新启动**以停止并重新启动 SQL Server 的实例。 

	![Restart Database Engine][Image11]

7. 关闭 SQL Server 配置管理器。

有关为 SQL Server 数据库引擎启用协议的详细信息，请参阅[启用或禁用服务器网络协议](http://msdn.microsoft.com/zh-cn/library/ms191294.aspx)。

###<a id="Mixed">配置混合模式的 SQL Server 身份验证</a>

如果没有域环境中，SQL Server 数据库引擎不能使用 Windows 身份验证。若要从另一台计算机连接到数据库引擎，配置为混合的模式身份验证的 SQL Server。混合模式身份验证同时允许 SQL Server 身份验证和 Windows 身份验证。（配置混合的模式身份验证可能不需要如果已配置 Azure 虚拟网络。有关详细信息，请参阅[Azure 虚拟机中 SQL Server 的连接性注意事项](http://msdn.microsoft.com/zh-cn/library/azure/dn133152.aspx)中的主题[Azure 虚拟机中 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/jj823132.aspx)文档集。

1. 在开始菜单上使用远程桌面连接到虚拟机，请单击**所有程序**，单击**Microsoft SQL Server _version_**，然后单击**SQL Server Management Studio**。 

	![Start SSMS][Image18]

	第一次打开 Management Studio 它必须创建用户 Management Studio 环境。这可能需要一小段时间。

2. Management Studio 打开时，显示**连接到服务器**对话框。在**服务器名称**框中，键入要连接到数据库引擎与对象资源管理器的虚拟机的名称。（而不是虚拟机名称还可以使用**(local)**或使用单个句点作为**服务器名称**。选择**Windows 身份验证**，并保留**_your_VM_name_\your_local_administrator**中**用户名**框。单击**连接**。

	![Connect to Server][Image19]

3. SQL Server Management Studio 对象资源管理器中，右键单击 SQL Server （虚拟机名称），实例的名称，然后依次**属性**。

	![Server Properties][Image20]

4. 在上**安全**页上，在**服务器身份验证**，选择**SQL Server 和 Windows 身份验证模式**，然后单击**确定**。

	![Select Authentication Mode][Image21]

5. 在 SQL Server Management Studio 对话框中，单击**确定**以确认重新启动 SQL Server 的要求。

6. 在对象资源管理器中，右键单击您的服务器，然后单击**重新启动**。（如果 SQL Server 代理正在运行，它必须还会重新启动。）

	![Restart][Image22]

7. 在 SQL Server Management Studio 对话框中，单击**是**以同意您想要重新启动 SQL Server。

###<a id="Logins">创建 SQL Server 身份验证登录名</a>

若要从另一台计算机连接到数据库引擎，必须创建至少一个 SQL Server 身份验证登录名。

1. SQL Server Management Studio 对象资源管理器中，展开您要在其中创建新的登录名的服务器实例的文件夹。

2. 右键单击**安全**文件夹，指向**新建**，然后选择**登录...**.

	![New Login][Image23]

3. 在**登录名-新建**对话框中，在**常规**页上，输入中的新用户的名称**登录名**框。

4. 选择**SQL Server 身份验证**。

5. 在**密码**框中，输入新用户的密码。输入该密码再次**确认密码**框。

6. 若要强制采用针对复杂和强制的密码策略选项，请选择**强制实施密码策略**（推荐）。这是选择 SQL Server 身份验证时的默认选项。

7. 若要强制采用针对过期的密码策略选项，请选择**强制密码过期**（推荐）。必须选择强制密码策略才能启用此复选框。这是选择 SQL Server 身份验证时的默认选项。

8. 若要强制用户在首次使用的登录名后创建一个新的密码，请选择**用户必须更改在下次登录密码**（建议使用此登录名是否为他人使用。如果登录名是供自己使用，不选择此选项。）必须选择强制密码过期才能启用此复选框。这是选择 SQL Server 身份验证时的默认选项。 

9. 从**默认数据库**列表中，选择登录名的默认数据库。**主**是此选项的默认值。如果尚未创建用户数据库，则保留此设置为**master**。

10. 在**默认语言**列表中，保留**默认**作为值。
    
	![Login Properties][Image24]

11. 如果这是您创建的第一个登录名，您可能想要将此登录名指定为 SQL Server 管理员。如果是这样，在**服务器角色**页上，选中**sysadmin**。 

	**安全说明：** sysadmin 固定的服务器角色的成员具有完全控制数据库引擎。您应慎重限制此角色的成员身份。

	![sysadmin][Image25]

12. 单击确定。

有关 SQL Server 登录名的详细信息，请参阅[创建登录名](http://msdn.microsoft.com/zh-cn/library/aa337562.aspx)。



###<a id="DNS">确定虚拟机的 DNS 名称</a>

若要从另一台计算机连接到 SQL Server 数据库引擎，必须知道虚拟机的域名系统 (DNS) 名称。（这是 internet 用于识别虚拟机的名称。您可以使用的 IP 地址，但在 Azure 移动资源以实现冗余或进行维护时，可能会更改的 IP 地址。DNS 名称将稳定因为它可以被重定向到新的 IP 地址。）  

1. 在 Azure 管理门户中 （或从上一步），选择**虚拟机**。 

2. 在上**虚拟机实例**页上，在**DNS 名称**列、 查找并复制显示的虚拟机的 DNS 名称前面有**http://**。（在用户界面上可能显示不出整个名称，不过没关系，你可以右键单击它，并选择"复制"。）
	

### <a id="cde">从其他计算机连接到数据库引擎</a>
 
1. 在计算机连接到 internet 上，打开 SQL Server Management Studio。

2. 在**连接到服务器**或**连接到数据库引擎**对话框中，在**服务器名称**框中，输入虚拟机 （上一个任务中确定） 和公共终结点端口号的 DNS 名称的格式 *DNSName,portnumber*如**tutorialtestVM.chinacloudapp.cn,57500**。

3. 在**身份验证**框中，选择**SQL Server 身份验证**。

4. 在**登录**框中，键入您在前一任务中创建的登录名的名称。

5. 在**密码**框中，键入您在前一任务中创建的登录名的密码。

6. 单击**连接**。

	![Connect using SSMS][Image33]

### <a id="cdea"> 从应用程序连接到数据库引擎</a>

如果您可以连接到 Azure 的虚拟机上运行使用 Management Studio 的 SQL Server 实例，您应能够通过使用类似于以下的连接字符串连接。

	connectionString="Server=<DNS_Name>;Integrated Security=false;User ID=<login_name>;Password=<your_password>;"providerName="System.Data.SqlClient"

有关详细信息，请参阅[如何解决连接到 SQL Server 数据库引擎](http://social.technet.microsoft.com/wiki/contents/articles/how-to-troubleshoot-connecting-to-the-sql-server-database-engine.aspx)。

##<a id="Optional">后续步骤</a>
你已经看到了如何使用平台映像在 Azure 虚拟机上创建和配置 SQL Server。当使用 SQL Server 在 Azure 虚拟机，我们建议您遵循提供的详细的指南[Azure 虚拟机中 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/jj823132.aspx)库中的文档。该文档集包含一系列提供详细指导的文章和教程。本系列包括以下各节：

[Azure 虚拟机中的 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/jj823132.aspx)

[Azure 虚拟机中的 SQL Server 入门](http://msdn.microsoft.com/zh-cn/library/azure/dn133151.aspx)

[准备迁移到 Azure 虚拟机中的 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/dn133142.aspx)

- 如何使用数据磁盘在 Azure 中的虚拟机之间迁移 SQL Server 数据库文件和架构

[Azure 虚拟机中的 SQL Server 部署](http://msdn.microsoft.com/zh-cn/library/azure/dn133141.aspx)

- 如何使用 CSUpload 将数据磁盘中的 SQL Server 数据和安装文件从本地复制到 Azure
- 如何使用 Hyper-V 在本地创建基础虚拟机
- 如何使用现有本地 SQL Server 磁盘在 Azure 中创建 SQL Server 虚拟机
- 如何使用现有本地 SQL Server 虚拟机在 Azure 中创建 SQL Server 虚拟机 
- 如何使用 PowerShell 在 Azure 中设置 SQL Server 虚拟机 
- 如何使用附加的数据磁盘存储数据库文件

[Azure 虚拟机中的 SQL Server 的连接性注意事项](http://msdn.microsoft.com/zh-cn/library/azure/dn133152.aspx)

- 教程：连接到同一云服务中的 SQL Server 
- 教程：连接到不同云服务中的 SQL Server 
- 教程：通过虚拟网络将 ASP.NET 应用程序连接到 Azure 中的 SQL Server 

[有关 Azure 虚拟机中的 SQL Server 的性能注意事项](http://msdn.microsoft.com/zh-cn/library/azure/dn133149.aspx)

[有关 Azure 虚拟机中的 SQL Server 的安全注意事项](http://msdn.microsoft.com/zh-cn/library/azure/dn133147.aspx)

[故障排除和监视 Azure 虚拟机中的 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/dn195883.aspx)

[高可用性和 Azure 虚拟机中的 SQL Server 的灾难恢复](http://msdn.microsoft.com/zh-cn/library/azure/jj870962.aspx)

- 教程：Azure 中的 AlwaysOn 可用性组 (GUI)
- 教程：Azure 中的 AlwaysOn 可用性组 (PowerShell)
- 教程：AlwaysOn 可用性组的侦听器配置
- 教程：添加 Azure 副本向导
- 教程：在 Azure 中进行数据库镜像以实现灾难恢复
- 教程：混合 IT 环境中用于实现灾难恢复的数据库镜像 
- 教程：在 Azure 中进行数据库镜像以实现高可用性
- 教程：在混合 IT 环境中进行日志传送以实现灾难恢复 
- Azure 中可用性组侦听器的故障排除

[Azure 虚拟机中的 SQL Server 的备份和还原](http://msdn.microsoft.com/zh-cn/library/azure/dn133143.aspx)

[Azure 虚拟机中的 SQL Server 商业智能](http://msdn.microsoft.com/zh-cn/library/azure/jj992719.aspx)

- 使用 PowerShell 创建运行 SQL Server BI 和 SharePoint 2010 的 Azure VM
- 使用 PowerShell 创建运行 SQL Server BI 和 SharePoint 2013 的 Azure VM
- 使用 PowerShell 创建运行本机模式报表服务器的 Azure VM

[SQL Server 数据仓库在 Azure 虚拟机中](http://msdn.microsoft.com/zh-cn/library/azure/dn387396.aspx)

[Azure 虚拟机中的 SQL Server 技术文章](http://msdn.microsoft.com/zh-cn/library/azure/dn248435.aspx)

- [白皮书：了解 Azure SQL Database 和 Azure 虚拟机中的 SQL Server](/zh-cn/documentation/articles/data-management-azure-sql-database-and-sql-server-iaas/)

- [白皮书：应用程序模式和 Azure 虚拟机中的 SQL Server 的开发策略](http://msdn.microsoft.com/library/azure/dn574746.aspx)

- [白皮书：部署 Azure 虚拟机中的 SQL Server Business Intelligence](http://msdn.microsoft.com/zh-cn/library/windowsazure/dn321998.aspx)

- [白皮书：Azure 虚拟机中的 SQL Server 的性能指南](http://msdn.microsoft.com/zh-cn/library/windowsazure/dn248436.aspx)

- [白皮书：Reporting Services 报表查看器控件和 Windows Azure 虚拟机基于的报表服务器](http://msdn.microsoft.com/library/azure/dn753698.aspx)

[Image5]: ./media/virtual-machines-provision-sql-server/5VM-Mode.png
[Image5b]: ./media/virtual-machines-provision-sql-server/5VM-Connect.png
[Image6]: ./media/virtual-machines-provision-sql-server/6VM-Options.png
[Image8b]: ./media/virtual-machines-provision-sql-server/SQLServerinVMConnectionMap.png
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
