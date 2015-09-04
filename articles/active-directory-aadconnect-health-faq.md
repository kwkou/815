<properties 
	pageTitle="Azure AD Connect Health 常见问题" 
	description="此常见问题对 Azure AD Connect Health 的相关问题进行了解答。此常见问题介绍有关使用该服务的问题，包括计费模型、功能、限制和支持。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="swadhwa" 
	editor="curtand"/>

<tags 
	ms.service="active-directory"  
	ms.date="07/12/2015"
	wacn.date="08/29/2015"/>


# Azure AD Connect Health 常见问题 (FAQ)

此常见问题对 Azure AD Connect Health 的相关问题进行了解答。此常见问题介绍有关使用该服务的问题，包括计费模型、功能、限制和支持。

## 常见问题



**问：我在 Azure Active Directory 中有多个租户。如何切换到使用 Azure Active Directory Premium 的租户？**

可以通过选择左侧导航栏上的“主页”，然后选择右上角的当前登录“用户名”并选择正确的租户帐户来切换 Azure AD 租户。如果此处未列出“租户”帐户，则选择注销，然后使用 Azure Active Directory Premium 租户的全局租户管理员凭据中进行登录。


## 安装问题



**问：在单个服务器上安装 Azure AD Connect Health 代理有什么影响？**

在 ADFS 服务器上安装 Microsoft Identity Health 代理对 CPU、内存使用量、网络带宽和存储方面的影响非常小。

以下数值是近似值。

- CPU 占用率：增加约 1%
- 内存使用量：最高达系统总内存的 10%
- 网络带宽使用量：约 1 MB/1000 ADFS 请求
>[AZURE.NOTE]如果代理无法与 Azure 通信，则会将数据存储在本地，最高可达系统总内存的 10%。当达到总物理内存的 10% 之后，如果代理仍未能将数据上载到该服务，则新的 ADFS 事务将基于“最近最少提供服务”覆盖任何“缓存的”事务。

- AD Health 代理的本地缓冲存储区：约 20 MB
- 审核通道所需的数据存储


建议为 AD FS 审核通道预配 1024 MB (1 GB) 的磁盘空间，以便 AD Health 代理处理所有数据。

**问：Azure AD Connect Health 代理安装过程中必须重新启动服务器吗？**

否。安装代理不需要重新启动服务器。但是，一些先决条件步骤的安装可能需要重新启动服务器。

例如，在 Windows Server 2008 R2 上安装 .Net 4.5 Framework 需要重新启动服务器。


**问：Azure AD Connect Health 服务是否通过直通 http 代理运行？**

是的，注册过程和正常操作均可通过显式设置代理将出站 http 请求转发。这种情况下，“Netsh WinHttp set Proxy”不会运行，因为代理使用 System.Net 而非 Microsoft Windows HTTP 服务来发出 Web 请求。

在运行 Register-AdHealthAgent（安装的最后一步）之前的任何时间开始执行


- 步骤 1 – 添加条目到 machine.config 文件


找到 machine.config 文件。该文件位于 in%windir%\\Microsoft.NET\\Framework64[version]\\config\\machine.config</li>

添加 machine.config 文件中 <configuration></configuration> 元素下的以下条目。
 
		
	<system.net>  
			<defaultProxy useDefaultCredentials="true">
       		<proxy 
        usesystemdefault="true" 
        proxyaddress="http://YOUR.PROXY.HERE.com"  
        bypassonlocal="true"/>
		</defaultProxy>
	</system.net> 

 

可在 [此处] (https://msdn.microsoft.com/zh-cn/library/kd3cf2ex(v=vs.110).aspx)) 找到更多的 <defaultProxy> 信息。

此设置会配置整个系统中的 .NET 应用程序，以便在进行 http .NET 请求时使用显式定义的代理。建议不要修改每个单独的 app.config，因为自动更新过程中将撤销此操作。只需更改一个文件，并且如果只修改 machine.config，则它会在更新后保留下来。

- 步骤 2 - 在 Internet 选项中配置代理

打开“Internet Explorer”->“设置”->“Internet 选项”->“连接”->“局域网设置”。

为你的 LAN 选择“使用代理服务器”

如果 HTTP 和 HTTPS/Secure 对应不同代理端口，则选择“高级”




**问：连接到 Http 代理时，Azure AD Connect Health 服务是否支持基本身份验证？**

否。目前暂不支持指定任意用户名/密码进行基本身份验证的机制。





## 操作问题



**问：是否需要在我的 AD FS 应用程序代理服务器或 Web 应用程序代理服务器上启用审核？**

否，不需要在 AD FS 应用程序代理服务器或 Web 应用程序代理服务器上启用审核。只需要在 AD FS 联合服务器上启用审核。



**问：如何解决 Azure AD Connect Health 警报？**

借助 success 条件解决 Azure AD Connect Health 警报。Azure AD Connect Health 代理定期检测并向服务报告 success 条件。对于少数几个警报而言，需基于时间进行抑制。即，如果从警报生成的 48 小时内未观察到相同的错误条件，则警报会自动得到解决。




**问：为了让 Azure AD Connect Health 代理运行，我需要打开哪些防火墙端口？**

你需要打开 TCP/UDP 端口 80 和 443，以便 Azure AD Connect Health 代理能够与 Azure AD Health 服务终结点进行通信。

<!---HONumber=67-->