<properties pageTitle="关于 Chef 和 Azure 虚拟机" description="介绍在 Azure 中的 VM 上安装和配置 Chef" services="virtual machines" documentationCenter="" authors="kathydav" manager="timlt"  />
<tags  
	ms.service="virtual-machines"
	ms.date="05/20/2015" 
	wacn.date="08/29/2015"/>

#关于 Chef 和 Azure 虚拟机

Chef 提供了构建、部署和管理基础结构的自动化系统。资源都是使用“配方”进行管理，所谓配方就是可重复使用的定义，可提供针对配置 Web 服务器等任务的说明。

Chef 是一种客户端服务器系统。要了解使用 Chef 服务器的选项，请参阅[选择你的安装](http://www.getchef.com/chef/choose-your-version/)。设置代理时，需了解有关 Chef 服务器的信息。

要在 Azure 虚拟机上安装 Chef 客户端，你可选择以下选项：

- 使用 Azure 管理门户，在创建运行 Windows Server 2012 或 Windows Server 2012 R2 的虚拟机时，安装 Chef 客户端。有关说明，请参阅 [Windows Azure 门户](https://docs.chef.io/azure_portal.html)。
- 使用 Azure PowerShell，在现有虚拟机上安装 Chef 客户端。GitHub 上提供了一个[脚本](https://gist.github.com/kaustubh-d/cea1aa75baebd3615609)示例。
- 使用 Chef 插件 [knife-azure](http://docs.getchef.com/plugin_knife_azure.html)，创建虚拟机示例并安装 Chef 客户端。   


##其他资源
[Chef 和 Windows Azure]

[如何登录到运行 Windows Server 的虚拟机]

[如何登录到运行 Linux 的虚拟机]

[管理扩展]

<!--Link references-->
[Chef 和 Windows Azure]: http://www.getchef.com/solutions/azure/
[如何登录到运行 Windows Server 的虚拟机]: /zh-cn/documentation/articles/virtual-machines-log-on-windows-server/
[如何登录到运行 Linux 的虚拟机]: /zh-cn/documentation/articles/virtual-machines-linux-how-to-log-on
[管理扩展]: https://msdn.microsoft.com/zh-cn/library/dn606311.aspx

<!---HONumber=67-->