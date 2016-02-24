<properties
   pageTitle="Azure AD Connect：先决条件和硬件 | Microsoft Azure"
   description="将显示在登录页和大多数搜索结果中的文章说明"
   services="active-directory"
   documentationCenter=""
   authors="andkjell"
   manager="stevenpo"
   editor="curtand"/>

<tags
   ms.service="active-directory"
   ms.date="11/16/2015"
   wacn.date="01/29/2016"/>

# Azure AD Connect 的先决条件
本主题介绍 Azure AD Connect 的先决条件和硬件要求。

## 安装 Azure AD Connect 之前
在安装 Azure AD Connect 之前，需要准备好以下项目。

**Azure AD**

- Azure 订阅或 [Azure 试用版订阅](pricing/free-trial/)。这只是用来访问 Azure 门户，而不是用于 Azure AD Connect。如果你正在使用 PowerShell 或 Office 365，则无需 Azure 订阅即可使用 Azure AD Connect。如果你有 Office 365 许可证，则还可以使用 Office 365 门户。使用付费的 Office 365 许可证，还可以从 Office 365 门户访问 Azure 门户。
- [添加并验证](/documentation/articles/active-directory-add-domain)要在 Azure AD 中使用的域。例如，如果你计划让用户使用 contoso.com，请确保此域已经过验证，并且不是直接使用 contoso.onmicrosoft.com 默认域。
- Azure AD 目录默认允许 5 万个对象。在验证域后，该限制将增加到 30 万个对象。如果在 Azure AD 中需要更多的对象，则需要开具支持案例来请求增大此限制。如果需要 50 万个以上的对象，则需要购买 Office 365、Azure AD Basic、Azure AD Premium 或 Enterprise Mobility Suite 等许可证。

**本地服务器和环境**

- AD 架构版本与林功能级别必须是 Windows Server 2003 或更高版本。只要符合架构和林级别的要求，域控制器就能运行任何版本。
- 如果你打算使用**密码写回**功能，必须在 Windows Server 2008（包含最新的 SP）或更高版本上安装域控制器。
- 不能在 Small Business Server 或 Windows Server Essentials 上安装 Azure AD Connect。该服务器必须使用 Windows Server Standard 或更高版本。
- Azure AD Connect 必须安装在 Windows Server 2008 或更高版本上。如果使用快速设置，此服务器可以是域控制器或成员服务器。如果使用自定义设置，服务器也可以是独立服务器，并且不需要加入域。
- 如果在 Windows Server 2008 上安装 Azure AD Connect，请确保从 Windows Update 应用最新的修补程序。无法在未修补的服务器上启动安装。
- 如果你打算使用**密码同步**功能，必须在 Windows Server 2008 R2 SP1 或更高版本上安装 Azure AD Connect 服务器。
- Azure AD Connect 服务器上必须安装了 [.Net 4.5.1](#component-prerequisites) 或更高版本和 [PowerShell 3.0](#component-prerequisites) 或更高版本。
- 如果你正在部署 Active Directory 联合身份验证服务，则要安装 AD FS 或 Web 应用程序代理的服务器必须是 Windows Server 2012 R2 或更高版本。必须在这些服务器上启用 [Windows 远程管理](#windows-remote-management)才能进行远程安装。
- 如果要部署 Active Directory 联合身份验证服务，你需要使用 [SSL 证书](#ssl-certificate-requirements)。
- Azure AD Connect 要求使用 SQL Server 数据库来存储标识数据。默认情况下，将会安装 SQL Server 2012 Express LocalDB（轻量版 SQL Server Express），并在本地计算机上创建服务的服务帐户。SQL Server Express 有 10 GB 的大小限制，允许你管理大约 100000 个对象。如果你需要管理更多的目录对象，则需要将安装过程指向不同版本的 SQL Server。Azure AD Connect 支持从 SQL Server 2008（装有 SP4）到 SQL Server 2014 的各种 Microsoft SQL Server。

**帐户**

- 你要集成的 Azure AD 目录的 Azure AD 全局管理员帐户。
- 如果使用快速设置或者从 DirSync 升级，则需要本地 Active Directory 的企业管理员帐户。
- 如果使用自定义设置安装路径，[帐户将是 Active Directory](/documentation/articles/active-directory-aadconnect-accounts-permissions)。

**连接**

- 如果你正在使用出站代理连接到 Internet，则必须在 **C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config** 文件中添加以下设置，才能将安装向导和 Azure AD 同步连接到 Internet 和 Azure AD。

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

如果代理服务器需要身份验证，则该部分应如下所示。

```
    <system.net>
        <defaultProxy enabled="true" useDefaultCredentials="true">
            <proxy
            usesystemdefault="true"
            proxyaddress="http://<PROXYADDRESS>:<PROXYPORT>"
            bypassonlocal="true"
            />
        </defaultProxy>
    </system.net>
```

在 machine.config 中进行此更改之后，安装向导和同步引擎将响应来自代理服务器的身份验证请求。在所有安装向导页中（“配置”页除外）都使用了已登录用户的凭据。在安装向导结束时的“配置”页上，上下文将切换到已创建的[服务帐户](active-directory-aadconnect-accounts-permissions#azure-ad-connect-sync-service-account)。

有关[默认代理元素](https://msdn.microsoft.com/library/kd3cf2ex.aspx)的详细信息，请参阅 MSDN。

- 如果代理限制了可访问的 URL，则必须在代理中打开 [Office 365 URL 和 IP 地址范围](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)中所述的 URL。

**其他**

- 可选：一个用于验证同步的测试用户帐户。

## 组件先决条件

Azure AD Connect 依赖于 PowerShell 和 .Net 4.5.1。请根据 Windows Server 版本执行以下操作：


- Windows Server 2012R2
  - 已按默认安装 PowerShell，因此不需要执行任何操作。
  - .Net 4.5.1 和更高版本将通过 Windows 更新提供。请确保已在控制面板中安装 Windows Server 的最新更新。
- Windows Server 2008R2 和 Windows Server 2012
  - 可从 [Microsoft 下载中心](http://www.microsoft.com/downloads)获取的 **Windows Management Framework 4.0** 中包含最新的 PowerShell 版本。
  - .Net 4.5.1 和更高版本可从 [Microsoft 下载中心](http://www.microsoft.com/downloads)获取。
- Windows Server 2008
  - 可从 [Microsoft 下载中心](http://www.microsoft.com/downloads)获取的 **Windows Management Framework 3.0** 中包含最新的受支持 PowerShell 版本。
 - .Net 4.5.1 和更高版本可从 [Microsoft 下载中心](http://www.microsoft.com/downloads)获取。

## Windows 远程管理

 当你使用 Azure AD Connect 部署 Active Directory 联合身份验证服务或 Web 应用程序代理时，请检查以下要求，以确保连接和配置成功：

 - 如果目标服务器已加入域，请确保已启用“Windows 远程托管”
    - 在权限提升的 PSH 命令窗口中，使用命令 `Enable-PSRemoting –force`
 - 如果目标服务器是未加入域的 WAP 计算机，则需要满足一些额外的要求
 	- 在目标计算机（WAP 计算机）上：
         - 确保 winrm（Windows 远程管理/WS-Management）服务正在通过“服务”管理单元运行
         - 在权限提升的 PSH 命令窗口中，使用命令 `Enable-PSRemoting –force`
    - 在运行向导的计算机上（如果目标计算机未加入域或者是不受信任的域）：
        - 在权限提升的 PSH 命令窗口中，使用命令 `Set-Item WSMan:\localhost\Client\TrustedHosts –Value <DMZServerFQDN> -Force –Concatenate`
 	    - 在服务器管理器中：
 		     - 将外围网络 WAP 主机添加到计算机池（“服务器管理器”->“管理”->“添加服务器”...使用 DNS 选项卡）
 		     - 服务器管理器中的“所有服务器”选项卡：右键单击 WAP 服务器并选择“以下列身份进行管理...”，然后输入 WAP 计算机的本地（非域）凭据
 		     - 若要验证远程 PSH 连接，请在服务器管理器的“所有服务器”选项卡中：右键单击 WAP 服务器，然后选择“Windows PowerShell”。此时应会打开远程 PSH 会话，以确保可以建立远程 PowerShell 会话。

## <a name="ssl-certificate-requirements"></a> SSL 证书要求

**重要说明：**强烈建议在你的 AD FS 场的所有节点中以及所有 Web 应用程序代理服务器中使用相同的 SSL 证书。

- 该证书必须是 X509 证书。
- 在测试实验室环境中，你可以在联合服务器上使用自签名证书。不过，对于生产环境，建议你从某个公共 CA 获取证书。
    - 如果使用未公开受信任的证书，请确保每个 Web 应用程序代理服务器上安装的证书同时受本地服务器和所有联合服务器的信任
- 证书的标识必须与联合身份验证服务名称（例如 fs.contoso.com）匹配。
    - 标识是类型为 dNSName 的使用者备用名称 (SAN) 扩展，或者是指定为公用名的使用者名称（当不存在 SAN 条目时）。  
    - 证书中可以存在多个 SAN 条目，但是它们中必须有一个与联合身份验证服务名称匹配。
    - 如果计划使用“工作区加入”，则还额外需要一个 SAN，它具有值 **enterpriseregistration.**，后接你的组织的用户主体名称 (UPN) 后缀（例如 **enterpriseregistration.contoso.com**）。
- 不支持基于 CryptoAPI 下一代 (CNG) 密钥和密钥存储提供者的证书。这意味着，你必须使用基于 CSP（加密服务提供者）而非 KSP（密钥存储提供者）的证书。
- 支持通配符证书。

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

以下是运行 AD FS 或 Web 应用程序服务器的计算机的最低要求：

- CPU：双核 1.6 GHz 或更高
- 内存：2GB 或更高
- Azure VM：A2 配置或更高


## 后续步骤
了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)的详细信息。

<!---HONumber=Mooncake_0118_2016-->