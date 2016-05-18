<properties
	pageTitle="配置 IIS Express 以便测试本地移动服务 | Azure 移动服务"
	description="了解如何配置 IIS Express，以便连接到本地移动服务项目进行测试。"
	authors="ggailey777"
	manager="dwrede"
	services="mobile-services"
	documentationCenter=""
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/07/2016"
	wacn.date="03/28/2016"/>

# 配置本地 Web 服务器以允许连接到本地移动服务 

通过 Azure 移动服务，你可以在 Visual Studio 中使用支持的 .NET 语言之一支持创建移动服务，然后将其发布到 Azure。针对移动服务使用 .NET 后端的主要优势之一在于，你可以先在本地计算机或虚拟机上运行、测试和调试移动服务，然后再将它发布到 Azure。

若要使用模拟器、虚拟机或独立工作站上运行的客户端在本地测试移动服务，你必须配置本地 Web 服务器和主机，以允许连接到工作站的 IP 地址和端口。本主题将说明如何配置 IIS Express，以允许连接到本地托管的移动服务。

[AZURE.INCLUDE [mobile-services-how-to-configure-iis-express](../includes/mobile-services-how-to-configure-iis-express.md)]

<!---HONumber=Mooncake_0118_2016-->