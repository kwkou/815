<properties pageTitle="适用于 Azure 虚拟机的安全产品/服务" description="Azure VM 的关键安全功能的快速概述以及指向详细信息的链接" services="virtual machines" documentationCenter="" authors="kathydav" manager="timlt"  />
<tags  
	ms.service="virtual-machines"
	ms.date="01/23/2015" 
	wacn.date="08/29/2015"/>

#适用于 Azure 虚拟机的安全产品/服务

<p>在大多数情况下，Azure 虚拟机的安全任务和配置都是与其他任何虚拟机或物理计算机相同的。这其中包括使用强密码、不使用众所周知的用户名以及制定更新操作系统和应用程序的政策和计划等基本任务。此外，我们建议采取以下做法，并提供了可用于实施这些任务的功能：

- 运行防病毒软件/反恶意软件保护

- 对终结点使用网络访问控制列表 (ACL)
 
##运行防病毒软件/反恶意软件保护

Azure 针对防病毒解决方案提供了多个选项，但由你进行管理。例如，你将需要决定何时运行扫描和安装更新。你可以在创建虚拟机时添加防病毒支持，也可以稍后添加。目前提供了 3 种解决方案作为安全扩展，既可以安装在新虚拟机上，也可以安装在现有虚拟机上：

- [如何在 Azure VM 上安装和配置 Symantec Endpoint Protection](/zh-cn/documentation/articles/virtual-machines-install-symantec/)
- [如何在 Azure VM 上安装和配置 Trend Micro Deep Security 即服务](/zh-cn/documentation/articles/virtual-machines-install-trend/)
- [在 Azure 虚拟机上部署反恶意软件解决方案](http://azure.microsoft.com/blog/2014/05/13/deploying-antimalware-solutions-on-azure-virtual-machines/)
 

##对虚拟机终结点使用网络访问控制列表 (ACL)

利用网络访问控制列表 (ACL)，你可以有选择性地允许或拒绝传入虚拟机终结点的入站流量。此数据包筛选功能额外提供了一层安全性。有关此操作原理的详细信息以及指向说明的链接，请参阅<!--[-->关于网络访问控制列表 (ACL)<!--](http://msdn.microsoft.com/zh-cn/library/azure/dn376541.aspx)-->。

##其他资源
关于 Windows Azure 信任中心的[资源](/support/trust-center/resources)

<!---HONumber=67-->