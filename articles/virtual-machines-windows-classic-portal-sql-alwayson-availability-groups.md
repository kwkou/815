<properties 
	pageTitle="在 Azure VM 中配置 AlwaysOn 可用性组 - 经典"
	description="在 Azure 虚拟机中创建 AlwaysOn 可用性组。本教程主要使用用户界面和工具而不是脚本。"
	services="virtual-machines-windows"
	documentationCenter="na"
	authors="MikeRayMSFT"
	manager="jhubbard"
	editor="" />
<tags 
	ms.service="virtual-machines-windows"
	ms.date="05/04/2016"
	wacn.date="06/29/2016" />

# 在 Azure 中配置 AlwaysOn 可用性组 - 经典

> [AZURE.SELECTOR]
- [Resource Manager: 手动](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/)
- [经典: UI](/documentation/articles/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/)
- [经典: PowerShell](/documentation/articles/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups/)

本端到端教程介绍如何使用 Azure 虚拟机上运行的 SQL Server AlwaysOn 实施可用性组。

完成本教程后，Azure 中的 SQL Server AlwaysOn 解决方案将包括以下要素：

- 一个包含多个子网（包括前端子网和后端子网）的虚拟网络

- 包含 Active Directory (AD) 域的域控制器

- 部署到后端子网并加入 AD 域的两个 SQL Server VM

- 具有节点多数仲裁模型的 3 节点 WSFC 群集

- 具有可用性数据库的两个同步提交副本的可用性组

下图显示了该解决方案的示意图。

![zure 中可用性组的测试实验体系结构](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC791912.png)

请注意，这是一个可能的配置。例如，你可最大程度减少 2 副本可用性组的虚拟机数目，以便通过将域控制器作为 2 节点 WSFC 群集中的仲裁文件共享见证服务器来节省 Azure 中的计算时间。通过此方法，上述配置中的 VM 数目可以减少一个。

本教程的假设条件如下：

- 你已有一个 Azure 帐户。

- 你已经知道如何使用 GUI 从虚拟机库预配经典 SQL Server VM。有关详细信息，

- 你已经深入了解 AlwaysOn 可用性组。有关详细信息，请参阅 [AlwaysOn 可用性组 (SQL Server)](https://msdn.microsoft.com/zh-cn/library/hh510230.aspx)。

>[AZURE.NOTE]如果你想将 AlwaysOn 可用性组与 SharePoint 结合使用，另请参阅[为 SharePoint 2013 配置 SQL Server 2012 AlwaysOn 可用性组](https://technet.microsoft.com/zh-cn/library/jj715261.aspx)。

## 创建虚拟网络和域控制器服务器

你将从一个新的 Azure 试用帐户开始。完成帐户设置后，你会进入 Azure 经典管理门户的主页屏幕。

1. 单击该页左下角的“新建”按钮，如下所示。

	![在经典管理门户中单击“新建”](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665511.gif)

1. 依次单击“网络服务”、“虚拟网络”、“自定义创建”，如下所示。

	![创建虚拟网络](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665512.gif)

1. 在“创建虚拟网络”对话框中，通过逐页完成以下设置创建新的虚拟网络。

	|Page|设置|
|---|---|
|虚拟网络详细信息|**名称 = ContosoNET**<br/>**区域 = 中国北部**|
|DNS 服务器和 VPN 连接|无|
|虚拟网络地址空间|设置显示在以下屏幕截图中：![创建虚拟网络](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784620.png)|

1. 接下来，你将创建用作域控制器 (DC) 的 VM。再次单击“新建”，然后依次单击“计算”、“虚拟机”、“从库中”，如下所示。

	![创建 VM](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784621.png)

1. 在“创建虚拟机”对话框中，逐页完成以下设置来配置新 VM。

	|Page|设置|
|---|---|
|选择虚拟机操作系统|Windows Server 2012 R2 Datacenter|
|虚拟机配置|**版本发布日期** =（最新）<br/>**虚拟机名称** = ContosoDC<br/>**层** = 基本<br/>**大小** = A2（双核）<br/>**新用户名** = AzureAdmin<br/>**新密码** = Contoso!000<br/>**确认密码** = Contoso!000|
|虚拟机配置|**云服务** = 创建新的云服务<br/>**云服务 DNS 名称** = 唯一的云服务名称<br/>**DNS 名称** = 唯一名称（例如：ContosoDC123）<br/>**区域/地缘组/虚拟网络** = ContosoNET<br/>**虚拟网络子网** = Back(10.10.2.0/24)<br/>**存储帐户** = 使用自动生成的存储帐户<br/>**可用性集** =（无）|
|虚拟机选项|使用默认值|

配置完新虚拟机后，请等待对 VM 设置完成。此过程要用一些时间才能完成，并且，如果你单击 Azure 经典管理门户中的“虚拟机”选项卡，则可以看到 ContosoDC 逐一经历状态“正在启动(预配)”、“已停止”、“正在启动”、“正在运行(预配)”和最后的“正在运行”。

现在已成功预配 DC 服务器。接下来，请在 DC 服务器上配置 Active Directory 域。

## 配置域控制器

在以下步骤中，你可以将 ContosoDC 计算机配置为 corp.contoso.com 的域控制器。

1. 在经典管理门户中，选择 **ContosoDC** 计算机。在“仪表板”选项卡上，单击“连接”以打开用于远程桌面访问 RDP 文件。

	![连接到虚拟机](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784622.png)

1. 使用已配置的管理员帐户 (**\\AzureAdmin**) 和密码 (**Contoso!000**) 登录。

1. 默认情况下，应显示“服务器管理器”仪表板。

1. 单击仪表板上的“添加角色和功能”链接。

	![服务器资源管理器中的“添加角色”](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784623.png)

1. 选择“下一步”，直到你到达“服务器角色”部分。

1. 选择“Active Directory 域服务”和“DNS 服务器”角色。出现提示时，添加这些角色所需的任何其他功能。

	>[AZURE.NOTE]你将收到无静态 IP 地址的验证警告。如果你要测试配置，请单击“继续”。对于生产方案，请[使用 PowerShell 设置域控制器计算机的静态 IP 地址](/documentation/articles/virtual-networks-reserved-private-ip/)。

	![添加角色对话框](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784624.png)

1. 单击“下一步”，直到你到达“确认”部分。选中“必要时自动重新启动目标服务器”复选框。

1. 单击“安装”。

1. 功能安装完毕后，返回到“服务器管理器”仪表板。

1. 选择左侧窗格中的新“AD DS”选项。

1. 单击黄色警告栏上的“更多”链接。

	![DNS 服务器 VM 上的 AD DS 对话框](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784625.png)

1. 在“所有服务器任务详细信息”对话框的“操作”栏中，单击“将此服务器提升为域控制器”。

1. 在“Active Directory 域服务配置向导”中，使用以下值：

	|Page|设置|
|---|---|
|部署配置|**添加新林** = 选定<br/>**根域名** = corp.contoso.com|
|域控制器选项|**密码** = Contoso!000<br/>**确认密码** = Contoso!000|

1. 单击“下一步”以浏览向导中的其他页。在“必备项检查”页上，确认你看到以下消息：“所有先决条件检查都成功通过”。请注意，你应查看任何适用的警告消息，但可以继续安装。

1. 单击“安装”。**ContosoDC** 虚拟机将自动重新启动。

## 配置域帐户

接下来的步骤用于配置 Active Directory (AD) 帐户以供稍后使用。

1. 重新登录到 **ContosoDC** 计算机。

1. 在“服务器管理器”中，选择“工具”，然后单击“Active Directory 管理中心”。

	![Active Directory 管理中心](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784626.png)

1. 在“Active Directory 管理中心”的左窗格中，选择“corp (本地)”。

1. 在右侧的“任务”窗格中，选择“新建”，然后单击“用户”。使用以下设置：

	|设置|值|
|---|---|
|**名字**|安装|
|**用户 SamAccountName**|安装|
|**密码**|Contoso!000|
|**确认密码**|Contoso!000|
|**其他密码选项**|选定|
|**密码永不过期**|已选中|

1. 单击“确定”以创建 **Install** 用户。此帐户将用于配置故障转移群集和可用性组。

1. 使用相同的步骤创建两个其他用户：**CORP\\SQLSvc1** 和 **CORP\\SQLSvc2**。这些帐户将用于 SQL Server 实例。接下来，你需要为 **CORP\\Install** 分配所需权限以配置 Windows Server 故障转移群集 (WSFC)。

1. 在“Active Directory 管理中心”的左窗格中，选择“corp (本地)”。然后，在右侧的“任务”窗格中，单击“属性”。

	![CORP 用户属性](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784627.png)

1. 选择“扩展”，然后单击“安全性”选项卡上的“高级”按钮。

1. 在“corp 的高级安全设置”对话框中，单击**“添加”**。

1. 单击“选择主体”。然后，搜索 **CORP\\Install**。单击**“确定”**。

1. 选择“读取全部属性”和“创建计算机对象”权限。

	![CORP 用户权限](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784628.png)

1. 单击“确定”，然后再次单击“确定”。关闭 corp 属性窗口。

现在你已完成 Active Directory 和用户对象的配置，接下来，你将创建三个 SQL Server VM 并将其加入到此域中。

## 创建 SQL Server VM

接下来，创建三个 VM，包括 WSFC 群集节点和两个 SQL Server VM。若要创建每个 VM，请返回到 Azure 经典管理门户，然后依次单击“新建”、“计算”、“虚拟机”和“从库中”。然后，使用下表中的模板来帮助创建 VM。

|Page|VM1|VM2|VM3|
|---|---|---|---|
|选择虚拟机操作系统|**Windows Server 2012 R2 Datacenter**|**SQL Server 2014 RTM Enterprise**|**SQL Server 2014 RTM Enterprise**|
|虚拟机配置|**版本发布日期** =（最新）<br/>**虚拟机名称** = ContosoWSFCNode<br/>**层** = 基本<br/>**大小** = A2（双核）<br/>**新用户名** = AzureAdmin<br/>**新密码** = Contoso!000<br/>**确认密码** = Contoso!000|**版本发布日期** =（最新）<br/>**虚拟机名称** = ContosoSQL1<br/>**层** = 基本<br/>**大小** = A3（4 核）<br/>**新用户名** = AzureAdmin<br/>**新密码** = Contoso!000<br/>**确认密码** = Contoso!000|**版本发布日期** =（最新）<br/>**虚拟机名称** = ContosoSQL2<br/>**层** = 基本<br/>**大小** = A3（4 核）<br/>**新用户名** = AzureAdmin<br/>**新密码** = Contoso!000<br/>**确认密码** = Contoso!000|
|虚拟机配置|**云服务** = 之前创建的唯一云服务 DNS 名称（例如：ContosoDC123）<br/>**区域/地缘组/虚拟网络** = ContosoNET<br/>**虚拟网络子网** = Back(10.10.2.0/24)<br/>**存储帐户** = 使用自动生成的存储帐户<br/>**可用性集** = 创建可用性集<br/>**可用性集名称** = SQLHADR|**云服务** = 之前创建的唯一云服务 DNS 名称（例如：ContosoDC123）<br/>**区域/地缘组/虚拟网络** = ContosoNET<br/>**虚拟网络子网** = Back(10.10.2.0/24)<br/>**存储帐户** = 使用自动生成的存储帐户<br/>**可用性集** = SQLHADR（也可以在创建计算机后配置可用性集。所有三个计算机都应分配到 SQLHADR 可用性集。）|**云服务** = 之前创建的唯一云服务 DNS 名称（例如：ContosoDC123）<br/>**区域/地缘组/虚拟网络** = ContosoNET<br/>**虚拟网络子网** = Back(10.10.2.0/24)<br/>**存储帐户** = 使用自动生成的存储帐户<br/>**可用性集** = SQLHADR（也可以在创建计算机后配置可用性集。所有三个计算机都应分配到 SQLHADR 可用性集。）|
|虚拟机选项|使用默认值|使用默认值|使用默认值|

在三个虚拟机预配完毕之后，你需要将它们加入到 **corp.contoso.com** 域中，并向这些虚拟机授予 CORP\\Install 管理权限。为此，请针对三个虚拟机中的每一个按以下步骤操作。

1. 首先，更改首选 DNS 服务器地址。通过在列表中选择 VM 并单击“连接”按钮，先将每个 VM 的远程桌面 (RDP) 文件下载到本地目录。若要选择 VM，请在该行中除第一个单元外的任意位置单击，如下所示。

	![下载 RDP 文件](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC664953.jpg)

1. 下载 RDP 文件启动下载的 RDP 文件，并使用已配置的管理员帐户 (**BUILTIN\\AzureAdmin**) 和密码 (**Contoso!000**) 登录到 VM 中。

1. 一旦登录，你应看到“服务器管理器”仪表板。单击左窗格中的“本地服务器”。

1. 选择“由 DHCP 分配的启用 IPv6 的 IPv4 地址”链接。

1. 在“网络连接”窗口中，选择网络图标。

	![更改 VM 的首选 DNS 服务器](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784629.png)

1. 在命令栏中，单击“更改此连接的设置”（根据你的窗口大小，你可能需要单击双右箭头才能看到此命令）。

1. 选择“Internet 协议版本 4 (TCP/IPv4)”，然后单击“属性”。

1. 选择“使用以下 DNS 服务器地址”，并在“首选 DNS 服务器”中指定 10.10.2.4。

1. 地址 **10.10.2.4** 是分配到 Azure 虚拟网络中 10.10.2.0/24 子网中的 VM 的地址，而该 VM 是 **ContosoDC**。若要验证 **ContosoDC** 的 IP 地址，请在命令提示符中使用 **nslookup contosodc**，如下所示。

	![使用 NSLOOKUP 查找 DC 的 IP 地址](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC664954.jpg)

1. 单击“确定”，然后“关闭”以提交更改。你现在能够将该 VM 加入到 **corp.contoso.com** 中。

1. 再次在“本地服务器”窗口中，单击“工作组”链接。

1. 在“计算机名”选项上，单击“更改”。

1. 选中“域”复选框并在文本框中键入 **corp.contoso.com**。单击“确定”。

1. 在“Windows 安全性”弹出对话框中，指定默认域管理员帐户 (**CORP\\AzureAdmin**) 和密码 (**Contoso!000**) 的凭据。

1. 在看到“欢迎使用 corp.contoso.com 域”消息时，请单击“确定”。

1. 单击“关闭”，然后单击弹出对话框中的“立即重新启动”。

### 接下来，添加 Corp\\Install 用户作为每个 VM 上的管理员：

1. 一直等到 VM 重新启动，然后再次启动 RDP 文件以使用 **BUILTIN\\AzureAdmin** 帐户登录到 VM 中。

1. 在“服务器管理器”中，选择“工具”，然后单击“计算机管理”。

	![计算机管理](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784630.png)

1. 在“计算机管理”窗口中，展开“本地用户和组”，然后选择“组”。

1. 双击“管理员”组。

1. 在“管理员属性”对话框中，单击“添加”按钮。

1. 输入用户 **CORP\\Install**，然后单击“确定”。当系统提示你输入凭据时，使用 **AzureAdmin** 帐户与 **Contoso!000** 密码。

1. 单击“确定”以关闭“管理员属性”对话框。

### 接下来，将**故障转移群集**功能功能添加到每个 VM。

1. 在“服务器管理器”仪表板中，单击“添加角色和功能”。

1. 在“添加角色和功能向导”中，单击“下一步”，直到出现“功能”页。

1. 选择“故障转移群集”。出现提示时，请添加任何其他相关功能。

	![向 VM 添加故障转移群集功能](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784631.png)

1. 单击“下一步”，然后单击“确认”页上的“安装”。

1. “故障转移群集”功能安装完成后，单击“关闭”。

1. 从 VM 注销。

1. 针对所有三个服务器（**ContosoWSFCNode**、**ContosoSQL1** 和 **ContosoSQL2**）重复本部分中的步骤。

这些 SQL Server VM 现在已预配并正在运行，但它们是使用默认选项与 SQL Server 一同安装的。

## 创建 WSFC 群集

在本部分中，你将创建托管要在以后创建的可用性组的 WSFC 群集。至此，你应该已针对将在 WSFC 群集中使用的三个 VM 中的每个 VM 完成了以下操作：

- 在 Azure 中进行了完全预配

- 已将 VM 加入到域中

- 已将 **CORP\\Install** 添加到本地管理员组

- 添加了故障转移群集功能

必须在每个 VM 上执行所有这些操作，才能将该 VM 加入到 WSFC 群集中。

另请注意，Azure 虚拟网络的行为与本地网络不同。你需要按以下顺序来创建群集：

1. 在其中一个节点 (**ContosoSQL1**) 上创建单节点群集。

1. 将群集 IP 地址修改为未使用的 IP 地址 (**10.10.2.101**)。

1. 将群集名称联机。

1. 添加其他节点（**ContosoSQL2** 和 **ContosoWSFCNode**）。

按照以下步骤完成完全配置群集的这些任务。

1. 启动 **ContosoSQL1** 的 RDP 文件，然后使用域帐户 **CORP\\Install** 登录。

1. 在“服务器管理器”仪表板中，选择“工具”，然后单击“故障转移群集管理器”。

1. 在左窗格中，右键单击“故障转移群集管理器”，然后单击“创建群集”，如下所示。

	![创建群集](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784632.png)

1. 在“创建群集向导”中，逐页完成以下设置来创建一个单节点群集：

	|Page|设置|
|---|---|
|开始之前|使用默认值|
|选择服务器|在“输入服务器名称”中键入 **ContosoSQL1**，然后单击“添加”|
|验证警告|选择**“否。不需要 Microsoft 对该群集的支持，因此不希望运行验证测试。单击‘下一步’时，继续创建群集。**”|
|用于管理群集的访问点|在“群集名称”中键入 **Cluster1**|
|确认|除非你使用的是存储空间，否则请使用默认值。请参阅此表后面的备注。|

	>[AZURE.WARNING]如果你使用[存储空间](https://technet.microsoft.com/zh-cn/library/hh831739)来将多个磁盘组合到存储池中，则必须取消选中“确认”页上的“将所有符合条件的存储添加到群集中”复选框。如果不取消选中该选项，则在群集过程中将分离虚拟磁盘。因此，这些虚拟磁盘也不会出现在磁盘管理器或资源管理器之中，除非从群集中删除存储空间，并使用 PowerShell 将其重新附加。

1. 在左窗格中，展开“故障转移群集管理器”，然后单击“Cluster1.corp.contoso.com”。

1. 在中心窗格中，向下滚动到“群集核心资源”部分并展开“名称: Clutser1”详细信息。你应看到“名称”和“IP 地址”资源都处于“已失败”状态。不能将 IP 地址资源联机，因为向该群集分配的 IP 地址与计算机本身的地址相同，地址重复。

1. 右键单击失败的**“IP 地址”**资源，然后单击**“属性”**。

	![群集属性](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784633.png)

1. 选择“静态 IP 地址”，然后在“地址”文本框中指定 **10.10.2.101**。然后，单击“确定”。

1. 在“群集核心资源”部分中，右键单击“名称: Cluster1”，然后单击“联机”。然后等待两个资源都已联机。当该群集名称资源联机时，它会用新的 AD 计算机帐户更新 DC 服务器。此 AD 帐户将在以后用于运行可用性组群集服务。

1. 最后，你需要向该群集添加剩余节点。在浏览器树中，右键单击“Cluster1.corp.contoso.com”，然后单击“添加节点”，如下所示。

	![向群集添加节点](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC784634.png)

1. 在“添加节点向导”中，单击“下一步”。然后，在“选择服务器”页上，向列表中添加“ContosoSQL2”和“ContosoWSFCNode”，方法是在“输入服务器名称”中键入服务器名称，然后单击“添加”。完成后，单击“下一步”。

1. 在“验证警告”页上，单击“否”（在生产方案中，你应执行验证测试）。然后，单击“下一步”。

1. 在“确认”页中，单击“下一步”以添加节点。

	>[AZURE.WARNING]如果你使用[存储空间](https://technet.microsoft.com/zh-cn/library/hh831739)来将多个磁盘组合到存储池中，则必须取消选中“将所有符合条件的存储添加到群集中”复选框。如果不取消选中该选项，则在群集过程中将分离虚拟磁盘。因此，这些虚拟磁盘也不会出现在磁盘管理器或资源管理器之中，除非从群集中删除存储空间，并使用 PowerShell 将其重新附加。

1. 向群集添加节点后，单击“完成”。“故障转移群集管理器”现在应显示你的群集具有三个节点，并将这些节点在“节点”容器中列出。

1. 从远程桌面会话注销。

## 为可用性组准备 SQL Server 实例

在本部分中，你将针对 **ContosoSQL1** 和 **contosoSQL2** 完成以下操作：

- 添加 **NT AUTHORITY\\System** 的一个登录名，该登录名具有对默认 SQL Server 实例的必要权限集

- 将 **CORP\\Install** 作为 sysadmin 角色添加到默认 SQL Server 实例

- 打开防火墙以便远程访问 SQL Server。

- 启用 AlwaysOn 可用性组功能

- 将 SQL Server 服务帐户分别更改为 **CORP\\SQLSvc1** 和 **CORP\\SQLSvc2**。

上述操作可按任意顺序执行。不过，以下步骤应按顺序进行。对 **ContosoSQL1** 和 **ContosoSQL2** 按步骤执行操作：

1. 如果你具有尚未登录了远程桌面会话的最大为 VM，请现在注销。

1. 启动 **ContosoSQL1** 和 **ContosoSQL2** 的 RDP 文件，然后以 **BUILTIN\\AzureAdmin** 身份登录。

1. 首先，向 SQL Server 登录帐户添加 **NT AUTHORITY\\System** 以及必要的权限。启动 **SQL Server Management Studio**。

1. 单击“连接”连接到默认 SQL Server 实例。

1. 在“对象资源管理器”，展开“安全性”，然后展开“登录名”。

1. 右键单击“NT AUTHORITY\\System”登录名，然后单击“属性”。

1. 在“安全对象”页上，针对本地服务器，选择“授予”以授予以下权限，然后单击“确定”。
	
	- 更改任何可用性组
	
	- 连接 SQL
	
	- 查看服务器状态

1. 接下来，将 **CORP\\Install** 作为 **sysadmin** 角色添加到默认 SQL Server 实例。在“对象资源管理器”中，右键单击“登录名”，然后单击“新建登录名”。

1. 在“登录名”中，键入 **CORP\\Install**。

1. 在“服务器角色”页上，选择“sysadmin”。然后，单击“确定”。创建登录名后，可通过在“对象资源管理器”中展开“登录名”来查看该登录名。

1. 接下来，为 SQL Server 创建防火墙规则。从“开始”屏幕启动“高级安全 Windows 防火墙”。

1. 在左窗格中，选择“入站规则”。在右窗格上，单击“新建规则”。

1. 在“规则类型”页上，选择“程序”，然后单击“下一步”。

1. 在“程序”页上，选择“此程序路径”，然后在文本框中键入 **%ProgramFiles%\\Microsoft SQL Server\\MSSQL12.MSSQLSERVER\\MSSQL\\Binn\\sqlservr.exe**（如果你遵循这些指示但使用的是 SQL Server 2012，SQL Server 目录则为 **MSSQL11.MSSQLSERVER**）。然后，单击“下一步”。

1. 在“操作”页面中，保持选中“允许连接”，然后单击“下一步”。

1. 在“配置文件”页面中，接受默认设置并单击“下一步”。

1. 在“名称”页上，在“名称”文本框中指定一个规则名称，如 **SQL Server (Program Rule)**，然后单击“完成”。

1. 接下来，启用 **AlwaysOn 可用性组**功能。从“开始”菜单，启动 **SQL Server 配置管理器**。

1. 在浏览器树中，单击“SQL Server 服务”，右键单击“SQL Server (MSSQLSERVER)”服务，然后单击“属性”。

1. 单击“AlwaysOn 高可用性组”选项卡，选择“启用 AlwaysOn 可用性组”（如下所示），然后单击“应用”。在弹出对话框中，单击“确定”，但不要关闭属性窗口。在更改服务帐户后，将重新启动 SQL Server 服务。

	![启用 AlwaysOn 可用性组](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665520.gif)

1. 接下来，更改 SQL Server 服务帐户。单击“登录”选项卡，在“帐户名”中键入 **CORP\\SQLSvc1**（对于 **ContosoSQL1**）或 **CORP\\SQLSvc2**（对于 **ContosoSQL2**），填入并确认密码，然后单击“确定”。

1. 在弹出窗口中，单击“是”重新启动该 SQL Server 服务。重新启动 SQL Server 服务后，在属性窗口中所做的更改即生效。

1. 从 VM 注销。

## 创建可用性组

此时，你已做好配置可用性组的准备。下面概括了将要执行的操作：

- 在 **ContosoSQL1** 上创建新数据库 (**MyDB1**)

- 获取数据库的完整备份和事务日志备份

- 使用 **NORECOVERY** 选项将完整备份和日志备份还原到 **ContosoSQL2**

- 通过同步提交、自动故障转移和可读辅助副本来创建可用性组 (**AG1**)

### 在 ContosoSQL1 上创建 MyDB1 数据库：

1. 如果你尚未从 **ContosoSQL1** 和 **ContosoSQL2** 的远程桌面会话注销，请现在注销。

1. 启动 **ContosoSQL1** 的 RDP 文件，然后以 **CORP\\Install** 身份登录。

1. 在“文件资源管理器”中的 C:\\ 之下，创建名为 **backup** 的目录。你将使用此目录来备份并还原数据库。

1. 右键单击新目录，指向“共享”，然后单击“特定用户”，如下所示。

	![创建备份文件夹](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665521.gif)

1. 创建备份文件夹添加“CORP\\SQLSvc1”并授予其“读/写”权限，添加“CORP\\SQLSvc2”并授予其“读取”权限（如下所示），然后单击“共享”。文件共享过程完成后，请单击“完成”。

	![授予对备份文件夹的权限](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665522.gif)

1. 接下来，你需要创建数据库。从“开始”菜单启动 **SQL Server Management Studio**，然后单击“连接”连接到默认 SQL Server 实例。

1. 在“对象资源管理器”中，右键单击“数据库”，然后单击“新建数据库”。

1. 在“数据库名称”中，键入 **MyDB1**，然后单击“确定”。

### 创建 MyDB1 的完整备份并在 ContosoSQL2 上还原该数据库：

1. 接下来，对数据库进行完整备份。在“对象资源管理器”中，展开“数据库”，右键单击“MyDB1”，指向“任务”，然后单击“备份”。

1. 在“源”部分中，将“备份类型”设置为“完整”。在“目标”部分中，单击“删除”以删除备份文件的默认文件路径。

1. 在“目标”部分中，单击“添加”。

1. 在“文件名”文本框中，键入 **\\ContosoSQL1\\backup\\MyDB1.bak**。然后，单击“确定”，然后再次单击“确定”以备份数据库。备份操作完成后，再次单击“确定”关闭对话框。

1. 接下来，对数据库进行事务日志备份。在“对象资源管理器”中，展开“数据库”，右键单击“MyDB1”，指向“任务”，然后单击“备份”。

1. 在“备份类型”中，选择“事务日志”。保持“目标”文件路径设置为此前指定的文件路径，然后单击“确定”。备份操作完成后，再次单击“确定”。

1. 接下来，在 **ContosoSQL2** 上还原完整备份和事务日志备份。启动 **ContosoSQL2** 的 RDP 文件，然后以 **CORP\\Install** 身份登录。将 **ContosoSQL1** 的远程桌面会话保持打开。

1. 从“开始”菜单启动 **SQL Server Management Studio**，然后单击“连接”连接到默认 SQL Server 实例。

1. 在“对象资源管理器”中，右键单击“数据库”，然后单击“还原数据库”。

1. 在“源”部分中，选择“设备”，然后单击“...”按钮。

1. 在“选择备份设备”中，单击“添加”。

1. 在“备份文件位置”中，键入 \\ContosoSQL1\\backup，单击“刷新”，选择“MyDB1.bak”，单击“确定”，然后再次单击“确定”。现在，你应在“要还原的备份集”窗格中看到完整备份和日志备份。

1. 转到“选项”页，在“恢复状态”中选择“RESTORE WITH NORECOVERY”，然后单击“确定”还原数据库。还原操作完成后，单击“确定”。

### 创建可用性组：

1. 返回到 **ContosoSQL1** 的远程桌面会话。在 SSMS 中的“对象资源管理器”中，右键单击“AlwaysOn 高可用性”，然后单击“新建可用性组向导”，如下所示。

	![启动新建可用性组向导](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665523.gif)

1. 在“简介”页上，单击“下一步”。在“指定可用性组名称”页上，在“可用性组名称”中键入 **AG1**，然后再次单击“下一步”。

	![新建可用性组向导，指定可用性组名称](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665524.gif)

1. 在“选择数据库”页上，选择“MyDB1”，然后单击“下一步”。这些数据库满足可用性组的先决条件，因为你针对预定主副本进行了至少一个完整备份。

	![新建可用性组向导，选择数据库](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665525.gif)

1. 在“指定副本”页上，单击“添加副本”。

	![新建可用性组向导，指定副本](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665526.gif)

1. 此时会弹出“连接到服务器”对话框。在“服务器名称”中键入 **ContosoSQL2**，然后单击“连接”。

	![新建可用性组向导，连接到服务器](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665527.gif)

1. 返回到“指定副本”页，此时应看到“可用的副本”中列出了 **ContosoSQL2**。配置这些副本，如下所示。完成后，单击“下一步”。

	![新建可用性组向导，指定副本（完整）](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665528.gif)

1. 在“选择初始数据同步”页上，选择“仅联接”，然后单击“下一步”。在对 **ContosoSQL1** 进行完整备份和事务备份并在 **ContosoSQL2** 上还原这些备份时，你已手动执行了数据同步。你也可以选择不对数据库执行备份和还原操作，并选择“完整”以便让“新建可用性组向导”为你执行数据同步。不过，对于某些企业中存在的超大型数据库，不建议这样做。

	![新建可用性组向导，选择初始数据同步](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665529.gif)

1. 在“验证”页中，单击“下一步”。此页应与以下页类似。因为你尚未配置可用性组侦听器，会出现一个侦听器配置警告。你可以忽略此警告，因为本教程不会配置侦听器。若要在完成本教程后配置侦听器，请参阅[在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器](/documentation/articles/virtual-machines-windows-classic-ps-sql-int-listener/)。

	![新建可用性组向导，验证](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665530.gif)

1. 在“摘要”页上，单击“完成”，然后等待向导配置完新的可用性组。在“进度”页上，可单击“更多详细信息”以查看详细进度。向导运行完成后，检查“结果”页以验证是否已成功创建可用性组（如下所示），然后单击“关闭”退出向导。

	![新建可用性组向导，结果](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665531.gif)

1. 在“对象资源管理器”中，展开“AlwaysOn 高可用性”，然后展开“可用性组”。此时，你应在此容器中看到新的可用性组。右键单击“AG1 (主)”，然后单击“显示仪表板”。

	![显示可用性组仪表板](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665532.gif)

1. **AlwaysOn 仪表板**应如下所示。你可以查看副本、每个副本的故障转移模式以及同步状态。

	![可用性组仪表板](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665533.gif)

1. 返回到“服务器管理器”，选择“工具”，然后启动“故障转移群集管理器”。

1. 展开 **Cluster1.corp.contoso.com**，然后展开“服务和应用程序”。选择“角色”，并注意已创建“AG1”可用性组角色。请注意，AG1 没有数据库客户端可按其连接到可用性组的 IP 地址，因为你未配置侦听器。你可以直接连接到主节点进行读写操作，直接连接到辅助节点进行只读查询。

	![故障转移群集管理器中的可用性组](./media/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/IC665534.gif)

>[AZURE.WARNING]请勿尝试从故障转移群集管理器对可用性组进行故障转移。所有故障转移操作都应在 SSMS 中的 **AlwaysOn 仪表板**内进行。有关详细信息，请参阅[将 WSFC 故障转移群集管理器用于可用性组的限制](https://msdn.microsoft.com/zh-cn/library/ff929171.aspx)。

## 后续步骤
现在，你已通过在 Azure 中创建可用性组，成功实施了 SQL Server AlwaysOn。若要为此可用性组配置侦听器，请参阅[在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器](/documentation/articles/virtual-machines-windows-classic-ps-sql-int-listener/)。

有关在 Azure 中使用 SQL Server 的其他信息，请参阅 [Azure 虚拟机上的 SQL Server](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

<!---HONumber=70-->