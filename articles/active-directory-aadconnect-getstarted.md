<properties 
	pageTitle="Azure AD Connect 入门" 
	description="了解如何下载、安装和运行 Azure AD Connect 的设置向导。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="terrylan" 
	editor="lisatoft"/>

<tags 
	ms.service="azure-active-directory-connect"  
	ms.date="04/02/2015"
	wacn.date="06/16/2015"/>

# Azure AD Connect 入门

以下文档将会帮助你开始使用 Azure Active Directory Connect。

## 下载 Azure AD Connect



若要开始使用 Azure AD Connect，可以使用以下链接下载最新版本：[下载 Azure AD Connect 公共预览版](http://connect.microsoft.com/site1164/program8612)

## 安装 Azure AD Connect 之前
在安装 Azure AD Connect 之前，需要准备好以下项目。

- Azure 订阅或 [Azure 试用版订阅](/pricing/1rmb-trial/)
- Azure AD 高级版或 [Azure AD 高级试用版](http://aka.ms/aadptrial)
- 你要集成的 Azure AD 租户的 Azure AD 全局管理员帐户
- 装有 Windows Server 2008 或更高版本的 AD 域控制器或成员服务器
- 本地 Active Directory 的企业管理员帐户
- 可选：一个用于验证同步的测试用户帐户。 

如果你想要配合 AD FS 选项使用 SSO，则还需要以下项目：

- 目标联合服务器上的本地管理员凭据。
- 要部署 Web 应用程序代理角色的任何工作组（未加入域）服务器上的本地管理员凭据。
- 用于联合服务器的 Windows Server 2012 R2 服务器。
- 用于 Web 应用程序代理的 Windows Server 2012 R2 服务器。
-  一个 .pfx 文件，其中包含目标联合服务名称（例如 fs.contoso.com）的 SSL 证书。
- 执行向导的计算机必须能够通过 Windows 远程管理连接到要安装 AD FS 或 Web 应用程序代理的任何其他计算机。 


## 安装 Azure AD Connect

下载 Azure AD Connect 后，使用以下步骤安装 Azure AD Connect。

1. 以企业管理员身份登录到要安装 Azure AD Connect 的服务器。
2. 导航到 AzureADConnect.msi 并双击它
3. 使用“快速”或“自定义”设置完成向导中的每个步骤
4. 可选：使用测试用户帐户登录到云服务（如 Office 365）以进行测试。

![欢迎使用 Azure AD Connect](./media/active-directory-aadconnect-getstarted/aadConnect_Welcome.png)

## 同步服务可选配置  

在安装同步服务时，可以将可选配置部分保留未选中状态，Azure AD Connect 会自动完成所有设置。这包括设置 SQL Server 2012 Express 实例，以及创建相应的组并为其分配权限。如果你想要更改默认设置，可以使用下表来了解可用的可选配置选项。

可选配置 | 说明 
------------- | ------------- |
SQL Server 名称 |用于指定 SQL Server 名称和实例名称。如果你已有一个要使用的数据库服务器，请选择此选项。
服务帐户 |默认情况下，Azure AD Connect 将为同步服务创建要使用的服务帐户。问题在于，密码是自动生成的，而安装 Azure AD Connect 的人员并不知道该密码。在大多数情况下，使用默认设置不会有问题，但是，如果你想要执行一些高级配置（例如，划归已同步的组织单位的范围），则需要创建一个帐户并选择你自己的密码。 |
权限 | 默认情况下，在安装同步服务时，Azure AD Connect 将创建四个组。这些组是：管理员组、操作员组、浏览组和密码重置组。如果你想要指定自己的组，可在此处指定。
导入设置 |如果你要从 Azure AD Sync 的 DirSync 中导入配置信息，请使用此选项。|



## 快速安装  

“快速设置”是默认选项，也是最常用的设置方案之一。使用此选项时，Azure AD Connect 将部署使用密码哈希同步选项的同步。这适用于单一林，它允许你的用户使用本地密码登录到云。如果使用快速安装，则在完成安装后，会立即触发同步。使用此选项时，只需单击六下鼠标就能将本地目录扩展到云中。

![快速安装](./media/active-directory-aadconnect-getstarted/express.png)

## 自定义安装

使用自定义安装可以选择多个不同的选项。下表描述了选择自定义安装选项时可用的向导页。

页面名称 | 说明
-------------------    | ------------- | 
用户登录|在此页上，可以选择是要使用密码同步、结合 AD FS 的联合身份验证，还是都不使用。
连接到目录|在此页上，可以添加你要同步的一个或多个目录。
同步筛选| 在这里，你可以确定是否要同步所有用户和组，或者是否要为每个目录指定一个组并只同步这些组。
本地标识|在这里，你可以指定用户只能在“连接到目录”页上添加的所有目录中出现一次，或者可以出现在多个目录中。如果用户出现在多个目录中，则你必须选择用于在目录中唯一标识这些用户的属性。例如，mail 属性、ObjectSID 或 SAMAccountName 都是用来唯一标识用户的常见属性。
Azure 标识|在此页上指定你要用于联合身份验证的源定位点。
选项功能|下表简要描述了你可以选择的可选功能。

![快速安装](./media/active-directory-aadconnect-getstarted/of.png)


可选功能 | 说明
-------------------    | ------------- | 
Exchange 混合部署 |Exchange 混合部署功能通过将 Azure AD 中的特定属性集同步回到本地目录，使 Exchange 邮箱能够在本地和 Azure 中共存。
Azure AD 应用程序和属性筛选|通过启用 Azure AD 应用程序和属性筛选，可以根据向导后续页面上的特定集定制同步属性集。这会在向导中打开两个附加的配置页。  
密码写回|通过启用密码写回，源自 Azure AD 的密码更改将写回到本地目录。
用户写回|通过启用用户写回，在 Azure AD 中创建的用户将写回到本地目录。这会在向导中打开一个附加的配置页。  
设备同步|通过启用设备同步，可将设备配置写入 Azure AD。
目录扩展属性同步|通过启用目录扩展属性同步，可将指定的属性同步到 Azure AD。这会在向导中打开一个附加的配置页。  

有关其他配置选项（例如，更改默认配置、使用同步规则编辑器和声明性设置），请参阅[管理 Azure AD Connect](/documentation/articles/active-directory-aadconnect-manage)

## Azure AD Connect 支持组件

下面列出了 Azure AD Connect 将在设置 Azure AD Connect 的服务器上安装的必备组件支持组件。此列表针对基本快速安装。如果你在“安装同步服务”页上选择使用其他 SQL Server，则不会安装下面列出的 SQL Server 2012 组件。

- Forefront Identity Manager Azure Active Directory Connector
- Microsoft SQL Server 2012 命令行实用工具
- Microsoft SQL Server 2012 本机客户端
- Microsoft SQL Server 2012 Express LocalDB
- 用于 Windows PowerShell 的 Azure Active Directory 模块
- 面向 IT 专业人员的 Microsoft Online Services 登录助手
- Microsoft Visual C++ 2013 Redistribution Package


**其他资源**

* [在云中使用本地标识基础结构](/documentation/articles/active-directory-aadconnect)
* [Azure AD Connect 工作原理](/documentation/articles/active-directory-aadconnect-howitworks)
* [管理 Azure AD Connect](/documentation/articles/active-directory-aadconnect-manage)
* [MSDN 上的 Azure AD Connect](https://msdn.microsoft.com/zh-cn/library/azure/dn832695.aspx)

<!---HONumber=60-->