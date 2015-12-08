<properties
   pageTitle="Azure AD Connect 的先决条件 | Microsoft Azure"
   description="将显示在登录页和大多数搜索结果中的文章说明"
   services="active-directory"
   documentationCenter=""
   authors="andkjell"
   manager="stevenpo"
   editor="curtand"/>

<tags
   ms.service="active-directory"
   ms.date="10/13/2015"
   wacn.date="11/02/2015"/>

# Azure AD Connect 的先决条件
本主题介绍 Azure AD Connect 的先决条件和硬件要求。

## 安装 Azure AD Connect 之前
在安装 Azure AD Connect 之前，需要准备好以下项目。

**Azure AD**

- Azure 订阅或 [Azure 试用版订阅](/pricing/free-trial/) - 这只是用来访问 Azure 门户，而不是用于 Azure AD Connect。如果你正在使用 PowerShell 或 Office 365，则无需 Azure 订阅即可使用 Azure AD Connect。如果你有 Office 365 许可证，则还可以使用 Office 365 门户。使用付费的 Office 365 许可证，还可以从 Office 365 门户访问 Azure 门户。
- 请验证要在 Azure AD 中使用的域。例如，如果你计划让用户使用 contoso.com，请确保此域已经过验证，并且不是直接使用 contoso.onmicrosoft.com 默认域。
- Azure AD 目录默认允许 5 万个对象。在验证域后，该限制将增加到 30 万个对象。如果在 Azure AD 中需要更多的对象，则需要开具支持案例来请求增大此限制。如果需要 50 万个以上的对象，则需要购买 Office 365、Azure AD Basic、Azure AD Premium 或 Enterprise Mobility Suite 等许可证。

**本地服务器和环境**

- AD 架构版本与林功能级别必须是 Windows Server 2003 或更高版本。只要符合架构和林级别的要求，域控制器就能运行任何版本。
- Azure AD Connect 必须安装在 Windows Server 2008 或更高版本上。如果使用快速设置，此服务器可以是域控制器或成员服务器。如果使用自定义设置，服务器也可以是独立服务器，并且不需要加入域。
- 如果你打算使用密码同步功能，服务器必须是 Windows Server 2008 R2 SP1 或更高版本。
- 如果你正在部署 Active Directory 联合身份验证服务，则要安装 AD FS 或网站代理的服务器必须是 Windows Server 2012 R2 或更高版本。必须在这些服务器上启用 Windows 远程管理才能进行远程安装。
- Azure AD Connect 要求使用 SQL Server 数据库来存储标识数据。默认情况下，将会安装 SQL Server 2012 Express LocalDB（轻量版 SQL Server Express），并在本地计算机上创建服务的服务帐户。SQL Server Express 有 10 GB 的大小限制，允许你管理大约 100000 个对象。如果你需要管理更多的目录对象，则需要将安装过程指向不同版本的 SQL Server。Azure AD Connect 支持从 SQL Server 2008（装有 SP4）到 SQL Server 2014 的各种 Microsoft SQL Server。

**帐户**

- 你要集成的 Azure AD 目录的 Azure AD 全局管理员帐户。
- 如果使用快速设置，则需要本地 Active Directory 的企业管理员帐户。
- 如果使用自定义设置安装路径，[帐户将是 Active Directory](/documentation/articles/active-directory-aadconnect-accounts-permissions)。

**连接**

- 如果你正在使用出站代理连接到 Internet，则必须在 **C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\Config\\machine.config** 文件中添加以下设置，才能将安装向导和 Azure AD 同步连接到 Internet 和 Azure AD。

```
    <system.net>
        <defaultProxy>
            <proxy
            usesystemdefault="true"
            proxyaddress="http://<PROXYADDRESS>:<PROXYPORT>"
            bypassonlocal="true"
            />
        </defaultProxy>
    </system.net>
```

必须在文件底部输入此文本。在此代码中，&lt;PROXYADRESS&gt; 代表实际的代理 IP 地址或主机名。
- 如果代理限制了可访问的 URL，则必须在代理中打开 [Office 365 URL 和 IP 地址范围](https://support.office.com/zh-CN/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)中所述的 URL。

**其他**

- 可选：一个用于验证同步的测试用户帐户。

## 组件先决条件

Azure AD Connect 依赖于 PowerShell 和 .Net 4.5.1。请根据 Windows Server 版本执行以下操作：


- Windows Server 2012R2
  - 已按默认安装 PowerShell，因此不需要执行任何操作。
  - .Net 4.5.1 和更高版本将通过 Windows 更新提供。请确保已在控制面板中安装 Windows Server 的最新更新。
- Windows Server 2008R2 和 Windows Server 2012
  - 可从 [Microsoft 下载中心](/downloads)获取的 **Windows Management Framework 4.0** 中包含最新的 PowerShell 版本。
  - .Net 4.5.1 和更高版本可从 [Microsoft 下载中心](/downloads)获取。
- Windows Server 2008
  - 可从 [Microsoft 下载中心](/downloads)获取的 **Windows Management Framework 3.0** 中包含最新的受支持 PowerShell 版本。
 - .Net 4.5.1 和更高版本可从 [Microsoft 下载中心](/downloads)获取。

## Azure AD Connect 支持组件

下面列出了 Azure AD Connect 在要安装 Azure AD Connect 的服务器上安装的组件。此列表针对基本快速安装。如果在“安装同步服务”页上选择使用不同的 SQL Server，则不会在本地安装 SQL Express LocalDB。

- Microsoft SQL Server 2012 命令行实用工具
- Microsoft SQL Server 2012 本机客户端
- Microsoft SQL Server 2012 Express LocalDB
- 用于 Windows PowerShell 的 Azure Active Directory 模块
- 面向 IT 专业人员的 Microsoft Online Services 登录助手
- Microsoft Visual C++ 2013 Redistribution Package


## Azure AD Connect 的硬件要求
下表显示了 Azure AD Connect 同步计算机的最低要求。

| Active Directory 中的对象数目 | CPU | 内存 | 硬盘驱动器大小 |
| ------------------------------------- | --- | ------ | --------------- |
| 少于 10,000 个 | 1\.6 GHz | 4 GB | 70 GB |
| 10,000–50,000 | 1\.6 GHz | 4 GB | 70 GB |
| 50,000–100,000 | 1\.6 GHz | 16 GB | 100 GB |
| 如果对象数超过 100,000 个，则需要使用完全版本的 SQL Server| | | |
| 100,000–300,000 | 1\.6 GHz | 32 GB | 300 GB |
| 300,000–600,000 | 1\.6 GHz | 32 GB | 450 GB |
| 超过 600,000 个 | 1\.6 GHz | 32 GB | 500 GB |

以下是运行 AD FS 或网站服务器的计算机的最低要求：

- CPU：双核 1.6 GHz 或更高
- 内存：2GB 或更高
- Azure VM：A2 配置或更高


## 后续步骤
了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)的详细信息。

<!---HONumber=79-->