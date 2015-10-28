<properties 
	pageTitle="如何配置云服务 | Windows Azure" 
	description="了解如何在 Azure 中配置云服务。了解如何更新云服务配置以及配置对角色实例的远程访问。" 
	services="cloud-services" 
	documentationCenter="" 
	authors="Thraka" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="cloud-services" 
	ms.date="06/29/2015"
	wacn.date="10/17/2015"/>




# 如何配置云服务

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/cloud-services-how-to-configure)
- [Azure 门户](/documentation/articles/cloud-services-how-to-configure-portal)

你可以在 Azure 管理门户中配置最常使用的云服务设置。或者，如果你希望直接更新配置文件，则可以下载要更新的服务配置文件，然后上载更新文件并使用配置更改更新云服务。无论使用哪种方法，配置更新都将应用于所有角色实例。

你还可以针对在云服务中运行的一个或所有角色启用“远程桌面”连接。通过远程桌面，你可以在应用程序正在运行时访问其桌面，并排查和诊断问题。即使你在应用程序开发过程中没有配置远程桌面服务定义文件 (.csdef)，你也可以对角色启用远程桌面连接。你无需重新部署应用程序，即可启用远程桌面连接。

如果每个角色至少具有两个角色实例，那么 Azure 在配置更新期间只能确保 99.95% 的服务可用性。这使得一台虚拟机可以在另一台虚拟机正更新时处理客户端请求。有关详细信息，请参阅[服务级别协议](/support/legal/sla/)。

## 更改云服务

1. 在 [Azure 门户](https://manage.windowsazure.cn)中，导航到你的云服务。

2. 单击“设置”图标或“必备/全部”设置链接以打开“设置”边栏选项卡。

    ![“设置”页](./media/cloud-services-how-to-configure-portal/cloud-service.png)
    
    从这里可以查看“属性”、更改“配置”、管理“证书”并管理有权访问此云服务的“用户”。

2. 在“监视”节下你可以单击任何磁贴来配置警报。

    ![云服务监视](./media/cloud-services-how-to-configure-portal/cs-monitoring.png)
    
3. 在“角色和实例”节下你可以单击任何云服务角色来管理该实例。

    ![云服务实例](./media/cloud-services-how-to-configure-portal/cs-instance.png)
    
    您可以从此处远程连接、重新启动云服务或对云服务进行重置映像。
    
    ![云服务实例按钮](./media/cloud-services-how-to-configure-portal/cs-instance-buttons.png)

>[AZURE.NOTE]不能使用 **Azure 门户**更改用于云服务的操作系统，仅可通过 [Azure 门户](https://manage.windowsazure.cn)更改此设置。详细信息见[此处](/documentation/articles/cloud-services-how-to-configure#update-a-cloud-service-configuration-file)。

## 更新云服务配置文件

1. 首先，下载现有的云服务配置文件 (.cscfg)。

    1. 在 [Azure 门户](https://manage.windowsazure.cn)中，导航到你的云服务。

    2. 单击“设置”图标或“必备/全部”设置链接以打开“设置”边栏选项卡。

        ![“设置”页](./media/cloud-services-how-to-configure-portal/cloud-service.png)
    
    3. 单击“配置”项。

        ![“配置”边栏选项卡](./media/cloud-services-how-to-configure-portal/cs-settings-config.png)
    
    4. 单击“下载”按钮。

        ![下载](./media/cloud-services-how-to-configure-portal/cs-settings-config-panel-download.png)

2. 更新服务配置文件后，上载并应用配置更新：

    1. 遵循上面的前 3 个步骤打开云服务的“配置”边栏选项卡。
    
    2. 单击“上载”按钮。

        ![上载](./media/cloud-services-how-to-configure-portal/cs-settings-config-panel-upload.png)
    
    3. 选择 .cscfg 文件，然后单击“确定”。

## 配置对角色实例的远程访问

不能使用 **Azure 门户**配置远程访问，仅可通过 [Azure 门户](https://manage.windowsazure.cn)更改此设置。详细信息见[此处](/documentation/articles/cloud-services-how-to-configure#configure-remote-access-to-role-instances)。
			
 

<!---HONumber=74-->