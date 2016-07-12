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
	ms.date="03/21/2016"
	wacn.date="04/06/2016"/>


# Azure AD Connect Health 常见问题 (FAQ)

此常见问题回答了关于 Azure AD Connect Health 的问题。其中涵盖了服务使用方面的问题，包括计费模式、功能、限制和支持。

## 一般问题



**问：我要管理多个 Azure AD 目录。如何切换到使用 Azure Active Directory Premium 的租户？**

若要切换 Azure AD 目录，你可以在右上角选择当前登录的用户名，然后选择相应的帐户。如果此处未列出该帐户，你可选择“注销”，然后使用已启用 Azure Active Directory Premium 的目录的全局管理员凭据来登录。

## 安装问题



**问：在单个服务器上安装 Azure AD Connect Health 代理有何影响？**

在 ADFS 服务器上安装 Microsoft Identity Health 代理对 CPU、内存消耗、网络带宽和存储的影响很小。

下面的数字为近似值。

- CPU 占用率：增加约 1%
- 内存消耗：最多为系统总内存的 10%
- 网络带宽使用量：约 1 MB/1000 ADFS 请求
>[AZURE.NOTE]如果代理无法与 Azure 通信，则代理会按照定义的上限将数据存储在本地。当代理达到该限制时，如果仍无法将数据上载到服务，新的 ADFS 事务将会根据“最近向其提供的服务最少”这一标准覆盖任何“缓存”的事务。

- AD Health 代理的本地缓存存储：约 20 MB
- 审核通道所需的数据存储


建议你预配一个 1024 MB (1 GB) 大小的磁盘空间，供 AD Health 代理的 AD FS 审核通道处理所有数据。

**问：在 Azure AD Connect Health 代理安装期间，我必须重新启动我的服务器吗？**

否。安装代理时不需要重新启动服务器。但是，安装某些必备组件时可能需要重新启动服务器。

例如，在 Windows Server 2008 R2 上安装 .Net 4.5 Framework 需要重启服务器。


**问：Azure AD Connect Health Services 是否通过直通型 http 代理进行工作？**

是的。对于正在进行的操作，你可以将 Health 代理配置为使用 HTTP 代理转发出站 http 请求。

如果要在代理注册过程中配置代理，需要修改 Internet Explorer 代理设置。<br>
打开“Internet Explorer -> 设置 -> Internet 选项 -> 连接 -> LAN 设置”。<br> 
选择“为 LAN 使用代理服务器”。<br> 
如果你有不同的针对 HTTP 和 HTTPS/Secure 的代理端口，则请选择“高级”。<br>


**问：Azure AD Connect Health Services 在连接到 Http 代理时是否支持基本身份验证？**

否。目前不支持指定任意用户名/密码进行基本身份验证这种机制。



## 操作问题



**问：我是否需要对 AD FS 应用程序代理服务器或 Web 应用程序代理服务器启用审核？**

否，不需要对 AD FS 应用程序代理服务器或 Web 应用程序代理服务器启用审核。只需对 AD FS 联合服务器启用审核。



**问：如何解除 Azure AD Connect Health 警报？**

在成功的情况下，将解除 Azure AD Connect Health 警报。Azure AD Connect Health 代理将定期检测并向服务报告成功的情况。某些警报的解除取决于时间。也就是说，如果在警报生成后的 48 小时内未观察到相同的错误条件，则警报会自动解除。




**问：若要确保 Azure AD Connect Health 代理正常使用，需要打开哪些防火墙端口？**

需要打开 TCP/UDP 端口 80、443 和 5671，才能让 Azure AD Connect Health 代理与 Azure AD Health 服务终结点进行通信。


**问：Azure AD Connect Health 门户中为何有两个同名的服务器？**

当你从某个服务器中删除代理时，该服务器不会自动从 Azure AD Connect 门户中删除。因此，如果你手动从服务器中删除代理或删除服务器本身，需要从 Azure AD Connect Health 门户中手动删除该服务器条目。有关详细信息，请参阅[删除服务器或服务实例](/documentation/articles/active-directory-aadconnect-health-operations/#delete-a-server-or-service-instance)。
此外，如果你重建了服务器的映像或者创建了具有相同详细信息（如计算机名称）的新服务器，但没有从 Azure AD Connect Health 门户中删除原有服务器，而是在新服务器上安装了代理，则你现在将看到两个服务器条目。在这种情况下，你应手动删除属于原有服务器的条目。此条目中的数据通常已过时。

## 相关链接

* [Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health/)
* [在 AD FS 中使用 Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health-adfs/)
* [Azure AD Connect Health 操作](/documentation/articles/active-directory-aadconnect-health-operations/)

<!---HONumber=Mooncake_0405_2016-->