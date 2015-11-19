<properties 
	pageTitle="Azure AD Connect Health 常见问题" 
	description="此常见问题回答了关于 Azure AD Connect Health 的问题。其中涵盖了服务使用方面的问题，包括计费模式、功能、限制和支持。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory"  
	ms.date="08/14/2015"
	wacn.date="11/02/2015"/>


# Azure AD Connect Health 常见问题 (FAQ)

此常见问题回答了关于 Azure AD Connect Health 的问题。其中涵盖了服务使用方面的问题，包括计费模式、功能、限制和支持。

## 一般问题



**问：我在 Azure Active Directory 中有多个租户。如何切换到使用 Azure Active Directory Premium 的租户？**

切换 Azure AD 租户时，你可以先在左侧导航栏中选择“主页”，然后在右上角选择当前已登录用户的“用户名”，最后再选择合适的租户帐户。如果租户帐户未列在此处，可选择“注销”，然后使用 Azure Active Directory Premium 租户的全局租户管理凭据登录。


## 安装问题



**问：在单个服务器上安装 Azure AD Connect Health 代理有何影响？**

在 ADFS 服务器上安装 Microsoft Identity Health 代理对 CPU、内存消耗、网络带宽和存储的影响很小。

下面的数字为近似值。

- CPU 占用率：增加约 1%
- 内存消耗：最多为系统总内存的 10%
- 网络带宽使用量：约 1 MB/1000 ADFS 请求
>[AZURE.NOTE]如果无法与 Azure 通信，代理会将数据存储在本地，最大限制为系统总内存的 10%。达到物理总内存的 10% 时，如果代理仍无法将数据上载到服务，新的 ADFS 事务将会根据“最近向其提供的服务最少”这一标准覆盖任何“缓存”的事务。

- AD Health 代理的本地缓存存储：约 20 MB
- 审核通道所需的数据存储


建议你预配一个 1024 MB (1 GB) 大小的磁盘空间，供 AD Health 代理的 AD FS 审核通道处理所有数据。

**问：在 Azure AD Connect Health 代理安装期间，我必须重新启动我的服务器吗？**

否。安装代理时不需要重新启动服务器。但是，安装某些必备组件时可能需要重新启动服务器。

例如，在 Windows Server 2008 R2 上安装 .Net 4.5 Framework 需要重启服务器。


**问：Azure AD Connect Health Services 是否通过直通型 http 代理进行工作？**

是的，注册过程和正常操作都可通过已设置为转发出站 http 请求的显式代理来完成。这种情况下不能使用“Netsh WinHttp set Proxy”，因为该代理使用 System.Net 而非 Microsoft Windows HTTP Services 来发出 Web 请求。

可以在运行 Register-AdHealthAgent（安装的最后步骤）之前随时执行


- 步骤 1 – 将条目添加到 machine.config 文件


找到 machine.config 文件。该文件位于 %windir%\\Microsoft.NET\\Framework64[version]\\config\\machine.config</li>

将以下条目添加到 machine.config 文件中的 <configuration></configuration> 元素下。
 
		
	<system.net>  
			<defaultProxy useDefaultCredentials="true">
       		<proxy 
        usesystemdefault="true" 
        proxyaddress="http://YOUR.PROXY.HERE.com"  
        bypassonlocal="true"/>
		</defaultProxy>
	</system.net> 

 

如需更多 <defaultProxy> 信息，请访问[此处](https://msdn.microsoft.com/zh-cn/library/kd3cf2ex(v=vs.110).aspx)。

此设置在系统范围内将 .NET 应用程序配置为在进行 http .NET 请求时使用显式定义代理。不建议修改每个单独的 app.config，因为在自动更新时会撤消该操作。只需更改一个文件，该文件在更新后会继续保留（如果你只修改 machine.config）。

- 步骤 2 - 在“Internet 选项”中配置代理

打开“Internet Explorer -> 设置 -> Internet 选项 -> 连接 -> LAN 设置”。

选择“为 LAN 使用代理服务器”

如果你有不同的针对 HTTP 和 HTTPS/Secure 的代理端口，则请选择“高级”




**问：Azure AD Connect Health Services 在连接到 Http 代理时是否支持基本身份验证？**

否。目前不支持指定任意用户名/密码进行基本身份验证这种机制。





## 操作问题



**问：我是否需要对 AD FS 应用程序代理服务器或 Web 应用程序代理服务器启用审核？**

否，不需要对 AD FS 应用程序代理服务器或 Web 应用程序代理服务器启用审核。只需对 AD FS 联合服务器启用审核。



**问：如何解除 Azure AD Connect Health 警报？**

在成功的情况下，将解除 Azure AD Connect Health 警报。Azure AD Connect Health 代理将定期检测并向服务报告成功的情况。某些警报的解除取决于时间。也就是说，如果在警报生成后的 48 小时内未观察到相同的错误条件，则警报会自动解除。




**问：若要确保 Azure AD Connect Health 代理正常使用，需要打开哪些防火墙端口？**

需要打开 TCP/UDP 端口 80 和 443，才能让 Azure AD Connect Health 代理与 Azure AD Health 服务终结点进行通信。

## 相关链接

* [Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health)
* [适用于 AD FS 的 Azure AD Connect Health 代理安装](/documentation/articles/active-directory-aadconnect-health-agent-install-adfs)
* [在 AD FS 中使用 Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health-adfs)
* [Azure AD Connect Health 操作](/documentation/articles/active-directory-aadconnect-health-operations)

<!---HONumber=76-->