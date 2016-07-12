<properties
	pageTitle="在 Azure RemoteApp 中使用 SQL Server Management Studio 连接到 SQL 数据库 | Azure"
	description="通过本教程了解如何在连接到 SQL 数据库时使用 Azure RemoteApp 中的 SQL Server Management Studio 进行安全和性能操作"
	services="sql-database"
	documentationCenter=""
	authors="adhurwit"
	manager=""/>

<tags
	ms.service="sql-database"
	ms.date="04/12/2016"
	wacn.date="05/16/2016"/>

# 在 Azure RemoteApp 中使用 SQL Server Management Studio 连接到 SQL 数据库

## 介绍  
本教程演示了如何在 Azure RemoteApp 中使用 SQL Server Management Studio (SSMS) 连接到 SQL 数据库。本教程将指导你完成在 Azure RemoteApp 中设置 SQL Server Management Studio 这一流程，介绍相关好处，并说明可以在 Azure Active Directory 中使用的安全功能。

**估计完成时间：**45 分钟

## Azure RemoteApp 中的 SSMS

Azure RemoteApp 是 Azure 中的 RDS 服务，用于交付应用程序。你可以在此处了解其详细信息：[什么是 RemoteApp？](/documentation/articles/remoteapp-whatis/)

在 Azure RemoteApp 中运行 SSMS 所获得的体验与在本地运行 SSMS 相同。

![显示 SSMS 在 Azure RemoteApp 中运行的屏幕截图][1]



## 优点

在 Azure RemoteApp 中使用 SSMS 有许多好处，包括：

- 在 Azure SQL Server 上的端口 1433 不必对外（在 Azure 外部）公开。
- 不需要在 Azure SQL Server 防火墙中不断添加和删除 IP 地址。
- 所有 Azure RemoteApp 连接都通过 HTTPS 在端口 443 上发生，使用的是加密的远程桌面协议
- 它采用多用户模式，可以伸缩。
- 将 SSMS 与 SQL 数据库置于同一区域可以获得性能提升。
- 你可以通过提供用户活动报告的高级版 Azure Active Directory 审核 Azure RemoteApp 的使用情况。
- 你可以启用多重身份验证 (MFA)。
- 使用受支持的 Azure RemoteApp 客户端（包括 iOS、Android、Mac、Windows Phone 和 Windows 电脑）随处访问 SSMS。


## 创建 Azure RemoteApp 集合

下面是使用 SSMS 创建 Azure RemoteApp 集合的步骤：


### 1\.从映像创建新的 Windows VM
使用库中的“Windows Server 远程桌面会话主机 Windows Server 2012 R2”映像创建新的 VM。


### 2\.从 SQL Express 安装 SSMS

转到新的 VM，导航到此下载页面：
[Microsoft® SQL Server® 2014 Express](https://www.microsoft.com/zh-cn/download/details.aspx?id=42299)

有一个仅下载 SSMS 的选项。下载后，请进入安装目录，然后运行安装程序以安装 SSMS。

你还需安装 SQL Server 2014 Service Pack 1。可以在此处下载：[Microsoft SQL Server 2014 Service Pack 1 (SP1)](https://www.microsoft.com/zh-cn/download/details.aspx?id=46694)

SQL Server 2014 Service Pack 1 包括的基本功能适用于 Azure SQL 数据库。


### 3\.运行“验证”脚本和 Sysprep

在 VM 的桌面上是一个名为“验证”的 PowerShell 脚本。双击即可运行该脚本。该脚本将验证 VM 是否可以用于对应用程序进行远程托管。验证完成后，该脚本将要求你运行 sysprep - 选择运行它。

sysprep 在完成后会关闭 VM。

若要详细了解如何创建 Azure RemoteApp 映像，请参阅：[如何在 Azure 中创建 RemoteApp 模板映像](http://blogs.msdn.com/b/rds/archive/2015/03/17/how-to-create-a-remoteapp-template-image-in-azure.aspx)


### 4\.捕获映像

当 VM 停止运行以后，可在当前门户中查找并捕获它。

若要详细了解如何捕获映像，请参阅[捕获使用经典部署模型创建的 Azure Windows 虚拟机的映像](/documentation/articles/virtual-machines-windows-classic-capture-image/)


### 5\.添加到 Azure RemoteApp 模板映像

在当前门户的 Azure RemoteApp 部分，转到“模板映像”选项卡，然后单击“添加”。在弹出框中，选择“从虚拟机库导入映像”，然后选择刚创建的映像。



### 6\.创建云集合

在当前门户中，创建新的 Azure RemoteApp 云集合。选择刚导入的模板映像（其上安装了 SSMS）。

![创建新的云集合][2]


### 7\.发布 SSMS

在全新云集合的“发布”选项卡中，选择“从开始菜单发布应用程序”，然后从列表中选择“SSMS”。

![发布应用][5]

### 8\.添加用户

在“用户访问权限”选项卡中，你可以选择将有权访问此 Azure RemoteApp 集合（仅包括 SSMS）的用户。

![添加用户][6]


### 9\.安装 Azure RemoteApp 客户端应用程序

你可以下载并安装此处提供的 Azure RemoteApp 客户端：[下载 | Azure RemoteApp](https://www.remoteapp.windowsazure.com/en/clients.aspx)



## 配置 Azure SQL Server

所需的唯一配置是确保为防火墙启用 Azure 服务。如果你使用此解决方案，则不需添加任何 IP 地址即可打开防火墙。允许的网络流量是从其他 Azure 服务流到 SQL Server。


![Azure 允许][4]



## Multi-Factor Authentication (MFA)

可以为此应用程序专门启用 MFA。转到 Azure Active Directory 的“应用程序”选项卡。你会发现 Azure RemoteApp 的一个条目。如果你单击该应用程序然后进行配置，则会看到以下页面，你可以在其中针对此应用程序启用 MFA。

![启用 MFA][3]



## 审核 Azure Active Directory Premium 的用户活动

如果你没有 Azure AD Premium，则需在目录的“许可证”部分启用它。启用 Premium 以后，即可将用户分配到 Premium 级别。

转到 Azure Active Directory 中的用户以后，即可转到“活动”选项卡，查看登录 Azure RemoteApp 时的登录信息。



## 后续步骤

完成所有上述步骤以后，你就能够运行 Azure RemoteApp 客户端并使用分配的用户登录。你会看到作为应用程序之一显示的 SSMS，并可根据需要来运行它，就像它是安装在你的计算机上且具有 Azure SQL Server 访问权限一样。

有关如何连接到 SQL 数据库的详细信息，请参阅[使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例性的 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)。


这就是本文的全部内容。请尽情享受其中的乐趣！



<!--Image references-->
[1]: ./media/sql-database-ssms-remoteapp/ssms.png
[2]: ./media/sql-database-ssms-remoteapp/newcloudcollection.png
[3]: ./media/sql-database-ssms-remoteapp/mfa.png
[4]: ./media/sql-database-ssms-remoteapp/allowazure.png
[5]: ./media/sql-database-ssms-remoteapp/publish.png
[6]: ./media/sql-database-ssms-remoteapp/user.png

<!---HONumber=Mooncake_0509_2016-->
