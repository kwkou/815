<properties pageTitle="SharePoint Server 场" description="介绍 Azure 预览门户中可用的全新 SharePoint Server 场功能。" services="virtual-machines" documentationCenter="" authors="josephd" manager="timlt"/>
<tags  
	ms.service="virtual-machines"
	ms.date="07/07/2015"
	wacn.date="08/29/2015"/>

#SharePoint Server 场#

利用 SharePoint Server 场，Windows Azure 门户可自动为你创建预先配置的 SharePoint Server 2013 场。当你需要基本或高可用性 SharePoint farm 以用于开发和测试环境，或者当你将 SharePoint Server 2013 评估作为组织的协作解决方案时，这可为你节约很多时间。

基本 SharePoint 场在此配置中包括 3 个虚拟机。

![sharepointfarm](./media/virtual-machines-sharepoint-farm-azure-preview/SPFarm_Basic.png)

使用该场配置可为 SharePoint 应用程序开发或你的第一次 SharePoint 2013 评估简化设置。

高可用性 SharePoint 场在此配置中包括 9 个虚拟机。

![sharepointfarm](./media/virtual-machines-sharepoint-farm-azure-preview/SPFarm_HighAvail.png)

使用该场配置可测试更高的客户端负载、外部 SharePoint 站点的高可用性以及 SharePoint 场的 SQL Server AlwaysOn。还可以将此配置用于高可用性环境中的 SharePoint 应用程序开发。
 
有关这两种场的配置详细详细，请参阅 [SharePoint Server 场配置详细信息](/documentation/articles/virtual-machines-sharepoint-farm-config-azure-preview)。

## 逐步完成配置

若要使用 SharePoint Server 场模板创建 SharePoint 场，请执行以下操作：

1. 在[](https://manage.windowsazure.cn)“Windows Azure 预览门户”中，单击“新建”>“计算”>“SharePoint Server 场”。如果未出现“SharePoint Server 场”，请单击“新建”>“计算”>“应用商店”，在“搜索计算”中键入 **SharePoint**，然后单击“SharePoint Server 场”。在“SharePoint Server 场”窗格中，单击“创建”。
2. 在“创建 SharePoint 场”窗格中，键入资源组的名称。
3. 在场中的每个虚拟机上键入本地管理员帐户的用户名和密码。选择较难猜测的名称和密码，记录并存储在安全位置。
4. 如果想要高可用性场，请单击“启用高可用性”。
5. 若要配置你的域控制器，请单击箭头。你可以指定主机名称前缀（默认是资源组名称）、林的根域名称（默认是 contoso.com）以及域控制器的大小（默认是 A1）。
6. 若要配置你的 SQL Server，请单击箭头。你可以指定主机名称前缀（默认是资源组名称）、SQL Server 的大小（默认是 A5）、数据库访问帐户名称和密码（默认是使用管理员帐户）以及 SQL Server 服务帐户名称（默认是 sqlservice）和密码（默认是使用与管理员帐户相同的密码）。
7. 若要配置你的 SharePoint Server，请单击箭头。你可以指定主机名称前缀（默认是资源组名称）、SharePoint Server 的大小（默认是 A2）、SharePoint 用户帐户（默认是 sp\_setup）和密码、SharePoint 场帐户名称（默认是 sp\_farm）和密码，以及 SharePoint 场的通行短语。默认是对 SharePoint 用户帐户、场帐户和通行短语都使用管理员密码。
8. 若要为虚拟网络、存储帐户或诊断配置可选的配置设置，请单击相应的箭头。
9. 若要指定订阅，请单击箭头。
10. 完成后，单击“创建”。

> [AZURE.NOTE]默认情况下，域控制器中未安装 Active Directory 管理工具。若要安装这些工具，请从域控制器虚拟机上的管理员级别 Windows PowerShell 命令提示符中运行 **Install-WindowsFeature AD-Domain-Services -IncludeManagementTools** 命令。

## 访问和管理 SharePoint 场

SharePoint 场具有预先配置的终结点，可允许未经身份验证的 Web 流量（TCP 端口 80）传入 SharePoint Web 服务器，以用于连接 Internet 的客户端计算机。此终结点连接的是预先配置的工作组网站。若要访问此工作组网站，请访问以下操作：

1.	从 Azure 预览门户中，单击“浏览”，然后单击“资源组”。 
2.	在资源组列表中，单击 SharePoint 场资源组的名称。
3.	在 SharePoint 场资源组的窗格中，单击“部署历史记录”。 
4.	在“部署历史记录”窗格中，单击 **Microsoft.SharePointFarm**。
5.	在 **Microsoft.SharePointFarm** 窗格中，选择 **SHAREPOINTSITEURL** 字段中的 URL 并复制。
6.	从 Internet 浏览器中，将该 URL 粘贴到地址字段中。
7.	出现提示时，输入你在创建场时制定的用户帐户凭据。

在管理中心 SharePoint 站点中，可以配置“我的站点”、SharePoint 应用程序和其他功能。有关详细信息，请参阅[配置 SharePoint 2013](http://technet.microsoft.com/zh-cn/library/ee836142.aspx)。若要访问管理中心 SharePoint 站点，请执行以下操作：

1.	从 Azure 预览门户中，单击“浏览”，然后单击“资源组”。 
2.	在资源组列表中，单击 SharePoint 场资源组的名称。
3.	在 SharePoint 场资源组的窗格中，单击“部署历史记录”。 
4.	在“部署历史记录”窗格中，单击 **Microsoft.SharePointFarm**。
5.	在 **Microsoft.SharePointFarm** 窗格中，选择 **SHAREPOINTCENTRALADMINURL** 字段中的 URL 并复制。
6.	从 Internet 浏览器中，将该 URL 粘贴到地址字段中。
7.	出现提示时，输入你在创建场时制定的用户帐户凭据。


注意:

- Azure 预览门户可在指定订阅中创建这些虚拟机。
- Azure 预览门户可以在具有面向 Internet 的网络影响力、仅限云的虚拟网络中创建这两种场。没有站点到站点 VPN 或 ExpressRoute 连接重新连回到你的组织网络。
- 你可以通过远程桌面连接来管理这些服务器。有关详细信息，请参阅[如何登录到运行 Windows Server 的虚拟机](/documentation/articles/virtual-machines-log-on-windows-server)。

## Azure 资源管理器

Azure 预览门户的 SharePoint Server 场功能可在服务管理中创建虚拟机。若要在资源管理器中创建 SharePoint Server 2013 场，请参阅[使用 Azure 资源管理器模板部署 SharePoint 场](/documentation/articles/virtual-machines-workload-template-sharepoint)。

## 其他资源

[SharePoint Server 场的配置详细信息](/documentation/articles/virtual-machines-sharepoint-farm-config-azure-preview)

[Azure 基础结构服务上的 SharePoint](http://msdn.microsoft.com/zh-cn/library/azure/dn275955.aspx)

[在混合云中设置 SharePoint Intranet 场用于测试](/documentation/articles/virtual-networks-setup-sharepoint-hybrid-cloud-testing)

[Azure 基础结构服务中托管的 SharePoint 场](/documentation/articles/virtual-machines-sharepoint-infrastructure-services)

<!---HONumber=67-->