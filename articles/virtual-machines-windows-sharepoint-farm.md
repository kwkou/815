<!-- not suitable for Mooncake -->

<properties
	pageTitle="创建 SharePoint 服务器场 | Azure"
	description="使用 Azure 门户应用商店快速创建新的基本或高可用性 SharePoint Server 2013 场。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="02/03/2016"
	wacn.date=""/>

# 创建 SharePoint 服务器场

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model)。这篇文章介绍如何使用资源管理器部署模型，Microsoft 建议大多数新部署使用资源管理器模型替代经典模型。
 
使用 Azure 门户应用商店，可以快速创建预配置的 SharePoint Server 2013 场。当你需要基本或高可用性 SharePoint farm 以用于开发和测试环境，或者当你将 SharePoint Server 2013 评估作为组织的协作解决方案时，这可为你节约很多时间。

> [AZURE.NOTE] Azure 门户的 Azure 库中的“SharePoint 服务器场”项已删除。该项被替换为“SharePoint 2013 非 HA 场”和“SharePoint 2013 HA 场”。

基本 SharePoint 场在此配置中包括 3 个虚拟机。

![sharepointfarm](./media/virtual-machines-windows-sharepoint-farm/Non-HAFarm.png)

使用该场配置可为 SharePoint 应用程序开发或你的第一次 SharePoint 2013 评估简化设置。

若要创建基本（三服务器）SharePoint 场，请执行以下操作：

1. 单击[此处](https://azure.microsoft.com/marketplace/partners/sharepoint2013/sharepoint2013farmsharepoint2013-nonha/)。
2. 单击“部署”。
3. 在“SharePoint 2013 非 HA 场”窗格中，单击“创建”。
4. 在“创建 SharePoint 2013 非 HA 场”窗格的 7 个步骤中指定设置，然后单击“创建”。

高可用性 SharePoint 场在此配置中包括 9 个虚拟机。

![sharepointfarm](./media/virtual-machines-windows-sharepoint-farm/HAFarm.png)

使用该场配置可测试更高的客户端负载、外部 SharePoint 站点的高可用性以及 SharePoint 场的 SQL Server AlwaysOn。还可以将此配置用于高可用性环境中的 SharePoint 应用程序开发。

若要创建高可用性（九服务器）SharePoint 场，请执行以下操作：

1. 单击[此处](https://azure.microsoft.com/marketplace/partners/sharepoint2013/sharepoint2013farmsharepoint2013-ha/)。
2. 单击“部署”。
3. 在“SharePoint 2013 HA 场”窗格中，单击“创建”。
4. 在“创建 SharePoint 2013 HA 场”窗格的 7 个步骤中指定设置，然后单击“创建”。

> [AZURE.NOTE] 你可以使用 Azure 试用版创建 **SharePoint 2013 non-HA Farm** 或者 **SharePoint 2013 HA Farm**。

## 管理 SharePoint 场

你可以通过远程桌面连接来管理这些场的服务器。有关详细信息，请参阅[登录到虚拟机](/documentation/articles/virtual-machines-windows-tutorial-classic-portal#log-on-to-the-virtual-machine)。

在管理中心 SharePoint 站点中，可以配置“我的站点”、SharePoint 应用程序和其他功能。有关详细信息，请参阅[配置 SharePoint 2013](http://technet.microsoft.com/zh-cn/library/ee836142.aspx)。

> [AZURE.NOTE] Azure 门户可以在具有面向 Internet 的网络影响力、仅限云的虚拟网络中创建这两种场。没有站点到站点 VPN 或 ExpressRoute 连接重新连回到你的组织网络。

## 后续步骤

- 发现 Azure 基础结构服务中的其他 [SharePoint 2013](https://technet.microsoft.com/zh-cn/library/dn635309.aspx) 配置。

<!---HONumber=Mooncake_0411_2016-->