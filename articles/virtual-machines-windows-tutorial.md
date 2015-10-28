<properties
	pageTitle="在 Azure 预览门户中创建运行 Windows 的虚拟机 | Windows Azure"
	description="了解如何使用 Azure 预览门户中的 Azure 应用商店创建运行 Windows 的 Azure 虚拟机"
	services="virtual-machines"
	documentationCenter=""
	authors="KBDAzure"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>
<tags
	ms.service="virtual-machines"
	ms.date="08/14/2015"
	wacn.date="09/18/2015"/>

# 在 Azure 预览门户中创建运行 Windows 的虚拟机#

> [AZURE.SELECTOR]
- [Azure preview portal](/documentation/articles/virtual-machines-windows-tutorial)
- [Azure portal](/documentation/articles/virtual-machines-windows-tutorial-classic-portal)
- [PowerShell: Resource Manager deployment](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell)
- [PowerShell: Classic deployment](/documentation/articles/virtual-machines-ps-create-preconfigure-windows-vms)

本教程演示如何在预览门户中快速轻松地创建 Azure 虚拟机。我们将使用 Windows Server 2012 R2 数据中心映像作为示例来创建虚拟机，但这只是 Azure 提供的众多映像的其中一个。映像选择取决于订阅。例如，桌面映像可能适用于 MSDN 订户。

你还可以使用自己的映像，配合资源管理器模板或自动化工具来创建虚拟机。若要了解不同的方法，请参阅[创建 Windows 虚拟机的不同方式](/documentation/articles/virtual-machines-windows-choices-create-vm)。

本教程使用资源管理器部署模型来创建虚拟机。我们建议使用上述模型，而不要使用基于服务管理 API 的经典部署模型。有关资源管理器的详细信息，请参阅 [Azure 资源管理器概述](/documentation/articles/resource-group-overview)。若要了解为虚拟机使用资源管理器的优点，请参阅 [Azure 资源管理器下的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-azurerm-versus-azuresm)。

## 选择映像

1. 登录到[门户](https://manage.windowsazure.cn)。

2. 在“中心”菜单上，单击“新建”\>“计算”\>“Windows Server 2012 R2 数据中心”。

	![应用商店](./media/virtual-machines-windows-tutorial/marketplace_new.png)

	>[AZURE.TIP]若要查找其他映像，请单击“应用商店”，然后搜索或筛选可用的项。

3. 在“Windows Server 2012 R2 数据中心”页上，在“选择部署模型”下选择“资源管理器”。单击“创建”。

	![应用商店中的搜索](./media/virtual-machines-windows-tutorial/marketplace_search_select.png)

## 创建虚拟机

选择映像后，可以对大多数配置使用 Azure 的默认设置并快速创建虚拟机。

1. 在“创建虚拟机”边栏选项卡上，单击“基本信息”。输入虚拟机的所需**名称**、管理**用户名**，以及强**密码**。如果有多个订阅，请为新虚拟机指定一个订阅，并指定一个新的或现有的**资源组**以及 Azure 数据中心的**位置**。

	![配置 VM 基本设置](./media/virtual-machines-windows-tutorial/create_vm_basics.PNG)

	>[AZURE.NOTE]“用户名”指用于管理服务器的管理帐户。创建一个很难让人猜出，但你自己可以记住的密码。**你需要使用用户名和密码登录到虚拟机**。

2. 单击“大小”并选择适合你需要的虚拟机大小。每种大小都指定了计算核心的数量、内存以及其他功能，比如对高级存储的支持，这将对价格产生影响。根据所选的映像，Azure 将自动推荐特定的大小。

	![配置 VM 大小](./media/virtual-machines-windows-tutorial/create_vm_size.PNG)

	>[AZURE.NOTE]某些地区的 DS 系列虚拟机提供高级存储。高级存储是数据密集型工作负荷（如数据库）的最佳存储选项。有关详细信息，请参阅[高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage-preview-portal)。

3. 单击“设置”以查看新虚拟机的存储和网络设置。对于第一个虚拟机，一般可以接受默认设置。如果选择了支持它的虚拟机大小，则可以通过选择“磁盘类型”下的“高级\(SSD\)”来试用高级存储。

	![配置 VM 设置](./media/virtual-machines-windows-tutorial/create_vm_settings.PNG)

6. 单击“摘要”以查看你的配置选择。查看或更新完设置后，单击“创建”。

	![配置 VM 设置](./media/virtual-machines-windows-tutorial/create_vm_summary.PNG)

8. 当 Azure 创建虚拟机时，你可以在“中心”菜单中的“通知”中跟踪进度。当 Azure 创建虚拟机后，你将在启动板上看到它，除非你清除了“创建虚拟机”边栏选项卡中的“固定到启动板”。

## 登录到虚拟机

创建虚拟机后，你需要登录其中，以便可以管理其设置以及要在其上运行的应用程序。

>[AZURE.NOTE]如需了解相关要求和疑难解答提示，请参阅[使用 RDP 或 SSH 连接到 Azure 虚拟机](https://msdn.microsoft.com/zh-cn/library/azure/dn535788.aspx)。

1. 如果你尚未登录[门户](https://manage.windowsazure.cn)，请先登录。

2. 在启动板上单击你的虚拟机。如果需要查找虚拟机，请单击“浏览所有”\>“最近”，或“浏览所有”\>“虚拟机”。然后从列表中选择你的虚拟机。

3. 在虚拟机边栏选项卡上，单击“连接”。

	![登录到虚拟机](./media/virtual-machines-windows-tutorial/connect_vm_portal.png)

4. 单击“打开”以使用自动为 Windows Server 虚拟机创建的远程桌面协议文件。

5. 单击“连接”。

6. 键入你在创建虚拟机时指定的用户名和密码，然后单击“确定”。

7. 单击“是”以验证虚拟机的标识。

	你现在可以像使用任何其他服务器一样使用该虚拟机。

## 后续步骤

* 使用 Azure PowerShell 和 Azure CLI 来[查找和选择虚拟机映像](/documentation/articles/resource-groups-vm-searching)。
* 使用 [Azure 资源管理器](/documentation/articles/virtual-machines-how-to-automate-azure-resource-manager)自动部署和管理虚拟机与工作负荷。
