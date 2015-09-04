<properties
	pageTitle="启用 Azure AD 应用程序代理"
	description="介绍了如何启动和运行 Azure AD 应用程序代理。"
	services="active-directory"
	documentationCenter=""
	authors="rkarlin"
	manager="terrylan"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="07/07/2015"
	wacn.date="08/29/2015"/>

# 启用 Azure AD 应用程序代理
> [AZURE.NOTE]应用程序代理是一项仅当升级到高级版或基本版的 Azure Active Directory 才可用的功能。有关详细信息，请参阅 [Azure Active Directory 版本](https://msdn.microsoft.com/zh-cn/library/azure/dn532272.aspx)。

使用 Windows Azure AD 应用程序代理可以在专用网络内部发布应用程序（例如 SharePoint 站点、Outlook Web Access 和基于 IIS 的应用），并为网络之外的用户提供安全访问权限。员工可以在家中使用其自己的设备登录到你的应用，并可以通过此基于云的代理进行身份验证

本部分将指导你完成在 Azure AD 中启用云目录的 Windows Azure AD 应用程序代理、在专用网络上安装应用程序代理连接器以及使用 Windows Azure AD 租户订阅注册该连接器。

##应用程序代理先决条件
在可以启用并使用应用程序代理服务之前，你必须有：

- 一个 Windows Azure 管理员帐户。如果没有，你可以在此处获取一个。
- 运行 Windows Server 2012 R2 或 Windows 8.1 或更高版本的服务器，可以在其上安装应用程序代理连接器。服务器必须能够将 HTTPS 请求发送到云中的应用程序代理服务，并且它必须与你打算发布的应用程序有一个 HTTPS 连接。 
- 如果路径中放置了防火墙，请确保防火墙处于打开状态，以允许产生于该连接器的对应用程序代理的 HTTPS (TCP) 请求。连接器将使用这些端口以及属于高级别域 msappproxy.net 的子域。请确保打开以下**所有**通往**出站**流量的端口：

端口号 | 描述
--- | ---
80 | 启用出站 HTTP 流量，以进行安全验证。
443 | 启用对 Azure AD 的用户身份验证（仅用于连接器注册过程）
10100 - 10120 | 启用发送回代理的 LOB HTTP 响应
9352, 5671 | 启用对传入请求的 Azure 服务的连接器之间的通信。
9350 | 可选。实现传入请求的更好性能。
8080 | 启用连接器启动序列并启用连接器自动更新
9090 | 启用连接器注册（仅用于连接器注册过程）
9091 | 启用连接器信任证书自动续订
 
如果防火墙根据原始用户强制实施流量，请打开这些端口，用于来自作为网络服务运行的 Windows 服务的流量。此外，请确保对 NT Authority\\System 启用端口 8080。


##步骤 1：在 Azure AD 中启用应用程序代理
1. 在 Azure 管理门户中，以管理员身份进行登录。
2. 转到 Active Directory 并选择你要在其中启用应用程序代理的目录。
3. 单击“配置”，向下滚动到“应用程序代理”，然后将此目录的“启用应用程序代理服务”切换到“启用”。

	![启用应用程序代理](http://i.imgur.com/87woFzq.png) <p>
4. 现在，单击屏幕底部的“下载”。将转到下载页面。阅读并接受许可条款，然后单击“下载”，保存用于应用程序代理连接器的 Windows Installer 文件 (.exe)。 

##步骤 2：安装和注册连接器
1. 在你准备的服务器上运行 AADApplicationProxyConnectorInstaller.exe（请参阅应用程序代理先决条件）。
2. 按照向导中的说明进行安装。
3. 安装过程中，将提示你使用活动的应用程序代理服务器帐户注册连接器。
<p> 提供 Azure AD 全局管理员凭据。- 请确保注册连接器的管理员位于启用应用程序代理服务的同一个目录中，例如，如果租户域为 contoso.com，则管理员应为 admin@contoso.com 或该域上的任何其他别名。并且你是 Azure AD 租户的全局管理员。全局管理员租户可能不同于 Windows Azure 凭据。- 如果 IE 增强的安全配置在安装 Azure AD 连接器的服务器上设置为“已打开”，则注册屏幕可能被阻止。如果发生这种情况，请按照错误消息中的说明进行操作，以允许访问。请确保 Internet Explorer 增强安全性为“已关闭”。- 如果连接器注册不成功，请参阅应用程序代理疑难解答。

4. 安装完成后，两个新的服务将添加到服务器，如下所示。这些是连接器服务，可以实现连接性和一个自动更新服务，该服务将定期检查连接器的新版本，并根据需要更新连接器。单击安装窗口中的“完成”以完成安装 ![应用程序代理连接器服务](http://i.imgur.com/zsVJKOz.png)<p>
5. 现在可以使用应用程序代理发布应用程序。

如果想要卸载连接器，则卸载连接器服务和更新程序服务之后，请确保重新启动计算机以完全删除该服务。<p> 为了获得高可用性，必须部署至少一个附加连接器。若要部署附加连接器，请重复以上步骤 2 和步骤 3 的操作。每个连接器必须分别进行注册。



## 其他资源

* [以组织身份注册 Azure](/documentation/articles/sign-up-organization)
* [Azure 标识](/documentation/articles/fundamentals-identity)

<!---HONumber=67-->