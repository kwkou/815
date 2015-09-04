<properties pageTitle="SharePoint Server 场的配置详细信息" description="介绍 SharePoint 场的默认配置" services="virtual-machines" documentationCenter="" authors="josephd" manager="timlt"/>
<tags  
	ms.service="virtual-machines"
	ms.date="07/07/2015"
	wacn.date="08/29/2015"/>

# SharePoint Server 场的配置详细信息

SharePoint Server 场是 Windows Azure 门户的一个功能，可自动为你创建预先配置的 SharePoint Server 2013 场。有两种场配置：

- 基本
- 高可用性

以下各节提供了每种场的配置详细信息。

有关更多信息，请参阅 [SharePoint Server 场](/zh-cn/documentation/articles/virtual-machines-sharepoint-farm-azure-preview/)。

## 基本 SharePoint 场

基本 SharePoint 场在以下配置中包括 3 个虚拟机：

![sharepointfarm](./media/virtual-machines-sharepoint-farm-config-azure-preview/SPFarm_Basic.png)

以下是配置详细信息：

-	Azure 订阅：在初始配置时指定。
-	Azure 域名（也称为云服务）：自动为每个虚拟机创建了单独的域名。
-	存储帐户：在初始配置时指定。
-	虚拟网络 	
	-   类型：仅限云	
    -	地址空间：10.0.0.0/26

- 虚拟机：
	-	*HostNamePrefix*-DC（AD DS 域控制器）
	-	*HostNamePrefix*-SQL（SQL Server 2014 服务器）
	-	*HostNamePrefix*-SP（SharePoint 2013 服务器）

- 域控制器
	-	虚拟机映像：Windows Server 2012 R2。
	-	主机名称前缀：在初始配置时指定。
	-	大小：A1（默认）
	-	域名：contoso.com（默认）
	-	域管理员帐户名称：在初始配置时指定。
	-	域管理员帐户密码：在初始配置时指定。

- 数据库服务器
	-	虚拟机映像：Windows Server 2012 R2 上的 SQL Server 2014 RTM Enterprise。
	-	主机名称前缀：在初始配置时指定。
	-	大小：A5（默认）
	-	数据库访问帐户名称：在初始配置时指定。
	-	数据库访问帐户密码：在初始配置时指定。
	-	SQL Server 服务帐户名称：sqlservice（默认）。
	-	SQL Server 服务帐户密码：在初始配置时指定。

- SharePoint Server
	-	虚拟机映像：SharePoint Server 2013 试用版。
	-	主机名称前缀：在初始配置时指定。
	-	大小：A2（默认）
	-	SharePoint 场帐户名称：sp\_farm（默认）。
	-	SharePoint 场帐户密码：在初始配置时指定。
	-	SharePoint 场通行短语：在初始配置时指定。


## 高可用性 SharePoint 场

高可用性 SharePoint 场在此配置中包括 9 个虚拟机：

![sharepointfarm](./media/virtual-machines-sharepoint-farm-config-azure-preview/SPFarm_HighAvail.png)
 
以下是配置详细信息：

-	Azure 订阅：在初始配置时指定。
-	Azure 域名（也称为云服务）：根据上图创建单独的域名。
-	存储帐户：在初始配置时指定。
-	虚拟网络：
	-	类型：仅限云
	-	地址空间：10.0.0.0/26

-	虚拟机：
	-	*HostNamePrefix*-DC1（AD DS 域控制器）
	-	*HostNamePrefix*-DC2（AD DS 域控制器）
	-	*HostNamePrefix*-SQL1（SQL Server 2014 服务器）
	-	*HostNamePrefix*-SQL2（SQL Server 2014 服务器）
	-	*HostNamePrefix*-SQL0（Windows Server 2012 R2 服务器）
	-	*HostNamePrefix*-WEB1（SharePoint 2013 服务器）
	-	*HostNamePrefix*-WEB2（SharePoint 2013 服务器）
	-	*HostNamePrefix*-APP1（SharePoint 2013 服务器）
	-	*HostNamePrefix*-APP2（SharePoint 2013 服务器）

-	域控制器：
	-	虚拟机映像：Windows Server 2012 R2。
	-	主机名称前缀：在初始配置时指定。
	-	大小：A1（默认）。
	-	域名：contoso.com（默认）。
	-	域管理员帐户名称：在初始配置时指定。
	-	域管理员帐户密码：在初始配置时指定。

-	数据库服务器：
	-	虚拟机映像：Windows Server 2012 R2 上的 SQL Server 2014 RTM Enterprise。
	-	主机名称前缀：在初始配置时指定。
	-	大小：数据库服务器为 A5（默认），文件共享见证为 A0（默认）。
	-	数据库访问帐户名称：在初始配置时指定。
	-	数据库访问帐户密码：在初始配置时指定。
	-	SQL Server 服务帐户名称：sqlservice（默认）。
	-	SQL Server 服务帐户密码：在初始配置时指定。

-	SharePoint Server：
	-	虚拟机映像：SharePoint Server 2013 试用版。
	-	主机名称前缀：在初始配置时指定。
	-	大小：A2（默认）。
	-	SharePoint 场帐户名称：sp\_farm（默认）。
	-	SharePoint 场帐户密码：在初始配置时指定。
	-	SharePoint 场通行短语：在初始配置时指定。

> [AZURE.NOTE]SharePoint Server 是从 SharePoint Server 2013 试用版映像中创建而来。若要在试用版到期后继续使用虚拟机，你需要将安装转换为使用 SharePoint Server 2013 Standard 版或 SharePoint Server 2013 Enterprise 版的“零售”或“批量许可证”密钥。

## Azure 资源管理器

Azure 预览门户的 SharePoint Server 场功能可在服务管理中创建虚拟机。若要在 Azure 资源管理器中创建 SharePoint Server 2013 场，请参阅[使用 Azure 资源管理器模板部署 SharePoint 场](/documentation/articles/virtual-machines-workload-template-sharepoint)。

## 其他资源

[SharePoint Server 场](/documentation/articles/virtual-machines-sharepoint-farm-azure-preview)

[Azure 虚拟机上的 SharePoint](http://msdn.microsoft.com/zh-cn/library/azure/dn275955.aspx)

[在混合云中设置 SharePoint Intranet 场用于测试](/documentation/articles/virtual-networks-setup-sharepoint-hybrid-cloud-testing)

<!---HONumber=67-->