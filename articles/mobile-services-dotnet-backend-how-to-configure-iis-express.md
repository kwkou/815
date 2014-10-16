<properties pageTitle="Configure IIS Express for local testing of Mobile Service" metaKeywords="Azure Mobile Services, .NET Backend, IIS Express" description="Learn how to configure IIS Express to allow connections to a local mobile service project for testing." authors="glenga" title="Configure the local web server to allow connections to a local mobile service" />

# 配置本地 Web 服务器以允许连接到本地移动服务

通过 Azure 移动服务，你可以在 Visual Studio 中使用支持的 .NET 语言之一支持创建移动服务，然后将其发布到 Azure。针对移动服务使用 .NET 后端的主要优势之一在于，你可以先在本地计算机或虚拟机上运行、测试和调试移动服务，然后再将它发布到 Azure。

若要使用模拟器、虚拟机或独立工作站上运行的客户端在本地测试移动服务，你必须配置本地 Web 服务器和主机，以允许连接到工作站的 IP 地址和端口。本主题将说明如何配置 IIS Express，以允许连接到本机托管的移动服务。

[WACOM.INCLUDE [mobile-services-how-to-configure-iis-express][mobile-services-how-to-configure-iis-express]]

  [mobile-services-how-to-configure-iis-express]: ../includes/mobile-services-how-to-configure-iis-express.md
