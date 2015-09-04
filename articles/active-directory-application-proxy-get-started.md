<properties
	pageTitle="如何提供对本地应用的安全远程访问"
	description="介绍如何使用 Azure AD 应用程序代理提供对本地应用的安全远程访问。"
	services="active-directory"
	documentationCenter=""
	authors="rkarlin"
	manager="terrylan"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="07/07/2015"
	wacn.date="08/29/2015"/>

# 如何提供对本地应用程序的安全远程访问

想要为远程用户（拥有所有类型的设备 – 托管、非托管；平板电脑、智能手机和便携式计算机）提供访问权限。提供对大量资源的安全访问权限可能会很复杂。在最近几年，反向代理是一种提供安全远程访问的流行方式，但它们需要位于防火墙后，这将难以确保并获得高度可用性。

## 云中的安全远程访问
在现代云环境中，我们将使用 Windows Azure Active Directory 中的应用程序代理，使远程访问达到下一级别。应用程序代理是作为一种服务提供的 Azure AD 的一项功能，也就是说它易于部署和使用。它与 Azure Active Directory（Office 365 使用的同一标识平台）集成。

## 什么是 Azure Active Directory 应用程序代理？
应用程序代理提供单一登录并对本地托管的 Web 应用程序提供安全远程访问（例如 SharePoint 站点和 Outlook Web Access）。现在可以以访问 Azure Active Directory 中 SaaS 应用程序的方式访问本地 Web 应用程序，而无需使用 VPN 或更改网络基础结构。应用程序代理允许发布应用程序。员工可以在家中使用其自己的设备登录到你的应用，并可以通过此基于云的代理进行身份验证。

## 它是如何工作的？
### 启用访问权限
应用程序代理通过在网络内安装名为连接器的精简的 Windows Server 服务进行工作。连接器不需要打开任何入站端口，并且无需在外围网络中放置任何内容。如果有大量流量涌入应用，可以添加多个连接器，该服务将负责负载平衡。连接器无状态并将根据需要从云中请求所有内容。用户从任何设备远程访问应用程序时，他将由 Azure Active Directory 进行身份验证，然后访问该应用程序。

### 单一登录
Azure AD 应用程序代理提供单一登录 (SSO) 到使用 IWA 的应用程序或声明感知应用程序。如果应用程序使用集成 Windows 身份验证，应用程序代理将模拟用户使用 Kerberos 约束委派来提供 SSO。已实现单一登录 (SSO) 到声明感知应用程序，因为用户已由 Azure Active Directory 进行了身份验证。

## 如何快速入门
请确保你具有 Azure AD 基本或高级订阅和一个你作为全局管理员的 Azure AD 目录。还需要 Azure AD 基本或高级许可证，用于目录管理员和用户访问应用。请查看以下有关详细信息。

### 开始启用对本地应用程序的远程访问
设置应用程序代理需要两个步骤：

1. [启用应用程序代理和配置连接器](/documentation/articles/active-directory-application-proxy-enable)<br>
2. [发布应用程序](/documentation/articles/active-directory-application-proxy-publish) - 使用快速而简单的向导，对你在本地的应用进行远程发布和访问。

## 下一步是什么？
使用应用程序代理可以执行更多的操作：


- [使用你自己的域名发布应用程序](https://msdn.microsoft.com/zh-cn/library/azure/mt210927.aspx)
- [启用单一登录](https://msdn.microsoft.com/zh-cn/library/azure/dn879065.aspx)
- [使用声明感知应用程序](https://msdn.microsoft.com/zh-cn/library/azure/mt210926.aspx)
- [启用条件性访问](https://msdn.microsoft.com/zh-cn/library/azure/dn931796.aspx)


### 了解有关应用程序代理的更多信息
- [请查看此处的联机帮助](https://msdn.microsoft.com/zh-cn/library/azure/dn768219.aspx)
- [请查看应用程序代理博客](http://blogs.technet.com/b/applicationproxyblog/)
- [观看第 9 频道上我们的视频！](http://channel9.msdn.com/events/Ignite/2015/BRK3864)

## 其他资源
* [以组织身份注册 Azure](/documentation/articles/sign-up-organization)
* [Azure 标识](/documentation/articles/fundamentals-identity)

<!---HONumber=67-->