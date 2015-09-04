<properties pageTitle="关于 Puppet 和 Azure 虚拟机" description="介绍在 Azure 中的 VM 上安装和配置 Puppet" services="virtual machines" documentationCenter="" authors="kathydav" manager="timlt" />
<tags  
	ms.service="virtual-machines"
	ms.date="05/20/2015"
	wacn.date="08/29/2015"/>

#关于 Puppet 和 Azure 虚拟机

<p>Puppet Enterprise 是用于构建、部署和管理基础结构的自动化软件。可将其用于管理 IT 基础结构的生命周期，包括发现、预配、操作系统和应用程序配置管理、协调和报告。

Puppet 是一种客户端服务器系统。Puppet Master 和 Puppet Enterprise 代理都可通过 Windows Azure 进行安装：

- Puppet Master 可作用预配置的映像，安装在 Ubuntu 服务器上。也可以将 Puppet Enterprise 安装在现有服务器上，但使用映像是最简单的入门方式。设置代理时，需了解有关服务器的信息。

- Puppet Enterprise 代理可用作虚拟机扩展，在创建虚拟机时进行安装，或者安装在现有的虚拟机上。

有关说明，请从[](http://puppetlabs.com/solutions/microsoft)“Microsoft Windows 和 Azure”页下载“入门指南”。


##其他资源
[与 Windows Azure 和 Visual Studio 的新集成]

[如何登录到运行 Windows Server 的虚拟机]

[如何登录到运行 Linux 的虚拟机]

[管理扩展]

<!--Link references-->
[与 Windows Azure 和 Visual Studio 的新集成]: http://puppetlabs.com/blog/new-integrations-windows-azure-and-visual-studio
[如何登录到运行 Windows Server 的虚拟机]: /zh-cn/documentation/articles/virtual-machines-log-on-windows-server/
[如何登录到运行 Linux 的虚拟机]: /zh-cn/documentation/articles/virtual-machines-linux-how-to-log-on
[管理扩展]: http://msdn.microsoft.com/zh-cn/library/dn606311.aspx

<!---HONumber=67-->