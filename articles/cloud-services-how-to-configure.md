<properties 
	pageTitle="如何配置云服务（管理门户）| Azure" 
	description="了解如何在 Azure 中配置云服务。了解如何更新云服务配置以及配置对角色实例的远程访问。" 
	services="cloud-services" 
	documentationCenter="" 
	authors="Thraka" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="cloud-services" 
	ms.date="04/22/2016"
	wacn.date="05/31/2016"/>




# 如何配置云服务

> [AZURE.SELECTOR]
- [Azure管理门户](/documentation/articles/cloud-services-how-to-configure/)

你可以在 Azure 管理门户中配置最常使用的云服务设置。或者，如果你希望直接更新配置文件，则可以下载要更新的服务配置文件，然后上载更新文件并使用配置更改更新云服务。无论使用哪种方法，配置更新都将应用于所有角色实例。

Azure 管理门户还允许你[为 Azure 云服务中的角色启用远程桌面连接](/documentation/articles/cloud-services-role-enable-remote-desktop/)

如果每个角色至少具有两个角色实例，那么 Azure 在配置更新期间只能确保 99.95% 的服务可用性。这使得一台虚拟机可以在另一台虚拟机正更新时处理客户端请求。有关详细信息，请参阅[服务级别协议](/support/legal/sla)。

## 更改云服务

1. 在 [Azure 管理门户](http://manage.windowsazure.cn)中依次单击“云服务”、云服务的名称、“配置”。

    ![配置页](./media/cloud-services-how-to-configure/CloudServices_ConfigurePage1.png)
    
    在“配置”页上，你可以配置监视、更新角色设置，并选择角色实例的来宾操作系统和系列。

2. 在“监视”中，将监视级别设置为“详细”或“最少”，并配置详细监视所需的诊断连接字符串。

3. 对于服务角色（按角色分组），你可以更新下列设置：
    
    >**设置**：  
    >修改服务配置 (.cscfg) 文件的 *ConfigurationSettings* 元素中指定的其他配置设置的值。
    >
    >**证书**：  
    >更改 SSL 加密中用于角色的证书指纹。若要更改证书，你必须首先上载新证书（在“证书”页上）。然后更新角色设置中显示的证书字符串中的指纹。

4. 在“操作系统”中，你可以更改角色实例的操作系统系列或版本，或选择“自动”以启用当前操作系统版本的自动更新。操作系统设置将应用到 Web 角色和辅助角色，但不影响虚拟机。

    部署期间，所有角色实例都将安装最新版本的操作系统，并且默认情况下这些操作系统会自动更新。
    
    如果由于你代码中的兼容性要求而需要云服务在不同的操作系统版本上运行，则可选择操作系统系列和版本。当你选择一个特定的操作系统版本时，云服务的自动操作系统更新便挂起。你将需要确保操作系统接收更新。
    
    如果你使用最新版本的操作系统解决了应用程序中的所有兼容性问题，则可通过将操作系统版本设置成“自动”来启用自动操作系统更新。
    
    ![操作系统设置](./media/cloud-services-how-to-configure/CloudServices_ConfigurePage_OSSettings.png)

5. 若要保存你的配置设置，并将其推送至角色实例，请单击“保存”。（单击“丢弃”可取消更改。） 更改设置后，命令栏中会出现“保存”和“丢弃”。

## 更新云服务配置文件

1. 下载包含当前配置的云服务配置文件 (.cscfg)。在云服务的“配置”页上，单击“下载”。然后单击“保存”或单击“另存为”以保存文件。

2. 更新服务配置文件后，上载并应用配置更新：

    1. 在“配置”页上，单击“上载”。
    
        ![上载配置](./media/cloud-services-how-to-configure/CloudServices_UploadConfigFile.png)
    
    2. 在“配置文件”中，使用“浏览”选择已更新的 .cscfg 文件。
    
    3. 如果云服务包含任何只有一个实例的角色，请选中“即使一个或多个角色包含单个实例也应用配置”复选框以使这些角色的配置更新继续进行。
    
        除非每个角色至少定义两个实例，否则服务配置更新期间，Azure 无法保证至少 99.95% 的云服务可用性。有关详细信息，请参阅[服务级别协议](/support/legal/sla)。
    
    4. 单击“确定”（复选标记）。


## 后续步骤

* 了解如何[部署云服务](/documentation/articles/cloud-services-how-to-create-deploy/)。
* 配置[自定义域名](/documentation/articles/cloud-services-custom-domain-name/)。
* [管理云服务](/documentation/articles/cloud-services-how-to-manage/)。
* [为 Azure 云服务中的角色启用远程桌面连接](/documentation/articles/cloud-services-role-enable-remote-desktop/)
* 配置 [SSL 证书](/documentation/articles/cloud-services-configure-ssl-certificate/)。

<!---HONumber=Mooncake_0523_2016-->