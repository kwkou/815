<properties
    pageTitle="Azure AD Connect：先决条件和硬件 | Azure"
    description="本主题介绍 Azure AD Connect 的先决条件和硬件要求"
    services="active-directory"
    documentationcenter=""
    author="andkjell"
    manager="femila"
    editor="" />
<tags
    ms.assetid="91b88fda-bca6-49a8-898f-8d906a661f07"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/17/2017"
    wacn.date="04/05/2017"
    ms.author="billmath" />  


# Azure AD Connect 的先决条件
本主题介绍 Azure AD Connect 的先决条件和硬件要求。

## 安装 Azure AD Connect 之前
在安装 Azure AD Connect 之前，需要准备好以下项目。

### Azure AD
- Azure 订阅或 [Azure 试用版订阅](/pricing/1rmb-trial/)。此订阅只是用来访问 Azure 门户预览，而不是用于 Azure AD Connect。如果使用 PowerShell 或 Office 365，则无需 Azure 订阅即可使用 Azure AD Connect。如果有 Office 365 许可证，还可以使用 Office 365 门户。使用付费的 Office 365 许可证，还可以从 Office 365 门户访问 Azure 门户预览。
  - 还可以使用 [Azure 门户预览](https://portal.azure.cn)中的 Azure AD 预览版功能。此门户不需要 Azure 许可证。
- [添加并验证](/documentation/articles/active-directory-add-domain/)要在 Azure AD 中计划使用的域。例如，如果计划让用户使用 contoso.com，请确保此域已经过验证，并且不是直接使用 contoso.partner.onmschina.cn 默认域。
- 默认情况下，一个 Azure AD 租户允许 5 万个对象。在验证域后，该限制将增加到 30 万个对象。如果在 Azure AD 中需要更多的对象，则需要开具支持案例来请求增大此限制。如果需要 50 万个以上的对象，则需要购买 Office 365、Azure AD Basic、Azure AD Premium 或 Enterprise Mobility Suite 等许可证。

### 准备本地数据
- 查看[可以在 Azure AD 中启用的可选同步功能](/documentation/articles/active-directory-aadconnectsyncservice-features/)并评估要启用哪些功能。

### 本地 Active Directory
- AD 架构版本与林功能级别必须是 Windows Server 2003 或更高版本。只要符合架构和林级别的要求，域控制器就能运行任何版本。
- 如果打算使用**密码写回**功能，必须在 Windows Server 2008（包含最新的 SP）或更高版本上安装域控制器。如果 DC 位于 2008（低于 R2）上，还必须应用[修补程序 KB2386717](http://support.microsoft.com/zh-cn/kb/2386717)。
- Azure AD 使用的域控制器必须可写。**不支持**使用 RODC（只读域控制器），并且 Azure AD Connect 不会遵循任何写重定向。
- **不支持**通过 SLD（单标签域）使用本地林/域。
- **不支持**通过“以点分隔的”（名称包含句点“.”）NetBios 名称使用本地林/域。

### Azure AD Connect 服务器
- 不能在 Small Business Server 或 Windows Server Essentials 上安装 Azure AD Connect。该服务器必须使用 Windows Server Standard 或更高版本。
- 必须在 Azure AD Connect 服务器上安装完整的 GUI。**不支持**在服务器核心上安装 GUI。
- Azure AD Connect 必须安装在 Windows Server 2008 或更高版本上。如果使用快速设置，此服务器可以是域控制器或成员服务器。如果使用自定义设置，服务器也可以是独立服务器，并且不需要加入域。
- 如果在 Windows Server 2008 上安装 Azure AD Connect，则请确保从 Windows Update 应用最新的修补程序。在未修补的服务器上无法启动安装。
- 如果打算使用**密码同步**功能，则必须在 Windows Server 2008 R2 SP1 或更高版本上安装 Azure AD Connect 服务器。
- Azure AD Connect 服务器必须安装 [.NET Framework 4.5.1](#component-prerequisites) 或更高版本和 [Microsoft PowerShell 3.0](#component-prerequisites) 或更高版本。
- 如果正在部署 Active Directory 联合身份验证服务，则要安装 AD FS 或 Web 应用程序代理的服务器必须是 Windows Server 2012 R2 或更高版本。必须在这些服务器上启用 [Windows 远程管理](#windows-remote-management)才能进行远程安装。
- 若要部署 Active Directory 联合身份验证服务，需要使用 [SSL 证书](#ssl-certificate-requirements)。
- 若要部署 Active Directory 联合身份验证服务，需要配置[名称解析](#name-resolution-for-federation-servers)。
- 如果全局管理员已启用 MFA，URL **https://secure.aadcdn.microsoftonline-p.com** 必须在受信任的站点列表中。在显示 MFA 质询提示之前，系统会先提示你将此 URL 添加到受信任的站点列表中（如果尚未添加）。可以使用 Internet Explorer 将它添加到受信任的站点。

### Azure AD Connect 所使用的 SQL Server
- Azure AD Connect 要求使用 SQL Server 数据库来存储标识数据。默认情况下，将会安装 SQL Server 2012 Express LocalDB（轻量版 SQL Server Express），并在本地计算机上创建服务的服务帐户。SQL Server Express 有 10GB 的大小限制，允许管理大约 100,000 个对象。如果需要管理更多的目录对象，则需要将安装向导指向不同的 SQL Server 安装。
- 如果使用独立的 SQL Server，则这些要求适用：
  - Azure AD Connect 支持从 SQL Server 2008（装有最新 Service Pack）到 SQL Server 2016 的各种 Microsoft SQL Server。**不支持**将 Azure SQL 数据库用作数据库。
  - 必须使用不区分大小写的 SQL 排序规则。这些排序规则通过其名称中的 \_CI_ 识别。**不支持**使用区分大小写的排序规则，可通过其名称中的 \_CS_ 识别。
  - 每个 SQL 实例只能有一个同步引擎。**不支持**与 FIM/MIM Sync、DirSync 或 Azure AD Sync 共享 SQL 实例。

### 帐户
- 要为将要集成的 Azure AD 租户准备 Azure AD 全局管理员帐户。该帐户必须是**学校或组织帐户**，而不能是 **Microsoft 帐户**。
- 如果使用快速设置或者从 DirSync 升级，必须创建本地 Active Directory 的企业管理员帐户。
- 如果使用自定义设置安装路径，[帐户应在 Active Directory 中](/documentation/articles/active-directory-aadconnect-accounts-permissions/)。

### 连接 <a name="connectivity"></a>
- Azure AD Connect 服务器需要 Intranet 和 Internet 的 DNS 解析。DNS 服务器必须能够将名称解析成本地 Active Directory 以及 Azure AD 终结点。
- 如果 Intranet 有防火墙，你需要开放 Azure AD Connect 服务器与域控制器之间的端口，请参阅 [Azure AD Connect 端口](/documentation/articles/active-directory-aadconnect-ports/)了解详细信息。
- 如果代理或防火墙限制了可访问的 URL，必须打开 [Office 365 URL 和 IP 地址范围](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)中所述的 URL。
  - 如果在德国使用 Microsoft 云或 Azure 政府版云，请参阅 [Azure AD Connect 同步服务实例注意事项](/documentation/articles/active-directory-aadconnect-instances/)获取 URL。
- Azure AD Connect 默认情况下使用 TLS 1.0 与 Azure AD 进行通信。可以遵循[为 Azure AD connect 启用 TLS 1.2](#enable-tls-12-for-azure-ad-connect) 中的步骤将此更改为 TLS 1.2。
- 如果使用出站代理连接到 Internet，则必须在 **C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\Config\\machine.config** 文件中添加以下设置，才能将安装向导和 Azure AD Connect 同步连接到 Internet 和 Azure AD。必须在文件底部输入此文本。在此代码中，&lt;PROXYADRESS&gt; 代表实际代理 IP 地址或主机名。

	
	    <system.net>
	        <defaultProxy>
	            <proxy
	            usesystemdefault="true"
	            proxyaddress="http://<PROXYADDRESS>:<PROXYPORT>"
	            bypassonlocal="true"
	            />
	        </defaultProxy>
	    </system.net>


- 如果代理服务器要求身份验证，则[服务帐户](/documentation/articles/active-directory-aadconnect-accounts-permissions/#azure-ad-connect-sync-service-account/)必须位于域中，且必须使用自定义的设置安装路径来指定[自定义服务帐户](/documentation/articles/active-directory-aadconnect-get-started-custom/#install-required-components/)。还需要对 machine.config 进行不同的更改。在 machine.config 中进行此更改之后，安装向导和同步引擎将响应来自代理服务器的身份验证请求。在所有安装向导页中（“配置”页除外）都使用已登录用户的凭据。在安装向导结束时的“配置”页上，上下文将切换到创建的[服务帐户](/documentation/articles/active-directory-aadconnect-accounts-permissions/#azure-ad-connect-sync-service-account/)。machine.config 节应如下所示。


	    <system.net>
	        <defaultProxy enabled="true" useDefaultCredentials="true">
	            <proxy
	            usesystemdefault="true"
	            proxyaddress="http://<PROXYADDRESS>:<PROXYPORT>"
	            bypassonlocal="true"
	            />
	        </defaultProxy>
	    </system.net>


有关[默认代理元素](https://msdn.microsoft.com/zh-cn/library/kd3cf2ex.aspx)的详细信息，请参阅 MSDN。在出现连接问题时，如需详细信息，请参阅[排查连接问题](/documentation/articles/active-directory-aadconnect-troubleshoot-connectivity/)。

### 其他
- 可选：一个用于验证同步的测试用户帐户。

## <a name="component-prerequisites"></a>组件先决条件

### PowerShell 和.Net Framework
Azure AD Connect 依赖于 Microsoft PowerShell 和 .NET Framework 4.5.1。服务器上需要安装此版本或更高版本。请根据 Windows Server 版本执行以下操作：

- Windows Server 2012R2
  - 已按默认安装 Microsoft PowerShell，因此不需要执行任何操作。
  - .NET Framework 4.5.1 和更高版本将通过 Windows 更新提供。请确保已在控制面板中安装 Windows Server 的最新更新。
- Windows Server 2008R2 和 Windows Server 2012
  - 可从 [Microsoft 下载中心](http://www.microsoft.com/downloads)获取的 **Windows Management Framework 4.0** 中获得最新的 Microsoft PowerShell 版本。
  - .NET Framework 4.5.1 和更高版本可从 [Microsoft 下载中心](http://www.microsoft.com/downloads)获取。
- Windows Server 2008
  - 可从 [Microsoft 下载中心](http://www.microsoft.com/downloads)获取的 **Windows Management Framework 3.0** 中包含最新的受支持 PowerShell 版本。
  - .NET Framework 4.5.1 和更高版本可从 [Microsoft 下载中心](http://www.microsoft.com/downloads)获取。

### 为 Azure AD Connect 启用 TLS 1.2  <a name="enable-tls-12-for-azure-ad-connect"></a>
Azure AD Connect 默认情况下使用 TLS 1.0 对同步引擎服务器和 Azure AD 之间的通信进行加密。可以通过配置 .Net 应用程序在服务器上默认使用 TLS 1.2 来更改此项。有关 TLS 1.2 的详细信息，请参阅 [Microsoft 安全通报 2960358](https://technet.microsoft.com/security/advisory/2960358)。

1. 无法在 Windows Server 2008 上启用 TLS 1.2。需要 Windows Server 2008R2 或更高版本。请确保已为操作系统安装了 .Net 4.5.1 修补程序，请参阅 [Microsoft 安全通报 2960358](https://technet.microsoft.com/security/advisory/2960358)。服务器上可能已经安装了此修补程序或更高版本的修补程序。
2. 如果使用 Windows Server 2008R2，请确保已启用 TLS 1.2。Windows Server 2012 服务器及更高版本上应该已经启用了 TLS 1.2。
	
		[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2]
		[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
		[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001

3. 对于所有操作系统，设置此注册表项并重新启动服务器。

		HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319
		"SchUseStrongCrypto"=dword:00000001

4. 如果还想要在同步引擎服务器和远程 SQL Server 之间启用 TLS 1.2，请确保为 [Microsoft SQL Server 的 TLS 1.2 支持](https://support.microsoft.com/zh-cn/kb/3135244)安装所需的版本。

## 联合身份验证安装和配置的先决条件
### <a name="windows-remote-management"></a>Windows 远程管理
在使用 Azure AD Connect 部署 Active Directory 联合身份验证服务或 Web 应用程序代理时，请检查以下要求：

- 如果目标服务器已加入域，则请确保已启用“Windows 远程托管”
  - 在权限提升的 PSH 命令窗口中，使用命令 `Enable-PSRemoting -force`
- 如果目标服务器是未加入域的 WAP 计算机，则需要满足一些额外的要求
  - 在目标计算机（WAP 计算机）上：
    - 确保 winrm（Windows 远程管理/WS-Management）服务正在通过“服务”管理单元运行
    - 在权限提升的 PSH 命令窗口中，使用命令 `Enable-PSRemoting -force`
      - 在运行向导的计算机上（如果目标计算机未加入域或者是不受信任的域）：
      - 在权限提升的 PSH 命令窗口中，使用命令 `Set-Item WSMan:\localhost\Client\TrustedHosts -Value <DMZServerFQDN> -Force -Concatenate`
    - 在服务器管理器中：
      - 将外围网络 WAP 主机添加到计算机池（“服务器管理器”->“管理”->“添加服务器”...使用 DNS 选项卡）
      - 服务器管理器中的“所有服务器”选项卡：右键单击 WAP 服务器并选择“以下列身份进行管理...”，然后输入 WAP 计算机的本地（非域）凭据
      - 若要验证远程 PSH 连接，请在服务器管理器的“所有服务器”选项卡中：右键单击 WAP 服务器，然后选择“Windows PowerShell”。此时应会打开远程 PSH 会话，以确保可以建立远程 PowerShell 会话。

### <a name="ssl-certificate-requirements"></a>SSL 证书要求
- 强烈建议在 AD FS 场的所有节点中以及所有 Web 应用程序代理服务器中使用相同的 SSL 证书。
- 该证书必须是 X509 证书。
- 在测试实验室环境中，可以在联合服务器上使用自签名证书。不过，对于生产环境，建议从某个公共 CA 获取证书。
  - 如果使用未公开受信任的证书，请确保每个 Web 应用程序代理服务器上安装的证书同时受本地服务器和所有联合服务器的信任
- 证书的标识必须与联合身份验证服务名称（例如 sts.contoso.com）匹配。
  - 标识是类型为 dNSName 的使用者备用名称 (SAN) 扩展，或者是指定为公用名的使用者名称（当不存在 SAN 条目时）。
  - 证书中可以存在多个 SAN 条目，但是它们中必须有一个与联合身份验证服务名称匹配。
  - 如果计划使用“工作区加入”，则还需要一个SAN，它的值为 **enterpriseregistration.**，后接组织的用户主体名称 (UPN) 后缀（例如 **enterpriseregistration.contoso.com**）。
- 不支持基于 CryptoAPI 下一代 (CNG) 密钥和密钥存储提供者的证书。这意味着，必须使用基于 CSP（加密服务提供者）而非 KSP（密钥存储提供者）的证书。
- 支持通配符证书。

### <a name="name-resolution-for-federation-servers"></a>联合服务器的名称解析
- 针对 Intranet（内部 DNS 服务器）和 Extranet（通过域注册机构注册的公共 DNS）设置 AD FS 联合身份验证服务名称（例如 sts.contoso.com）的 DNS 记录。对于 Intranet DNS 记录，请确保使用 A 记录而不是 CNAME 记录。只有这样，才能从加入域的计算机正常执行 Windows 身份验证。
- 如果要部署多个 AD FS 服务器或 Web 应用程序代理服务器，请确保配置负载均衡器，以及指向负载均衡器的 AD FS 联合身份验证服务名称（例如 sts.contoso.com）的 DNS 记录。
- 如果要将 Windows 集成身份验证用于 Intranet 中使用 Internet Explorer 的浏览器应用程序，请确保已将 AD FS 联合身份验证服务名称（例如 sts.contoso.com）添加到 IE 中的 Intranet 区域。此配置可以通过组策略进行控制，并可部署到所有已加入域的计算机中。

## <a name="enable-tls-12-for-azure-ad-connect"></a>Azure AD Connect 支持组件
下面列出了 Azure AD Connect 在要安装 Azure AD Connect 的服务器上安装的组件。此列表针对基本快速安装。如果在“安装同步服务”页上选择使用不同的 SQL Server，则不会在本地安装 SQL Express LocalDB。

- Azure AD Connect Health
- 面向 IT 专业人员的 Microsoft Online Services 登录助手（已安装，但不依赖它）
- Microsoft SQL Server 2012 命令行实用工具
- Microsoft SQL Server 2012 Express LocalDB
- Microsoft SQL Server 2012 本机客户端
- Microsoft Visual C++ 2013 再分发包

## Azure AD Connect 的硬件要求
下表显示了 Azure AD Connect 同步计算机的最低要求。

| Active Directory 中的对象数目 | CPU | 内存 | 硬盘驱动器大小 |
| --- | --- | --- | --- |
| 少于 10,000 个 |1\.6 GHz |4 GB |70 GB |
| 10,000-50,000 |1\.6 GHz |4 GB |70 GB |
| 50,000-100,000 |1\.6 GHz |16 GB |100 GB |
| 如果对象数超过 100,000 个，则需要使用完整版本的 SQL Server | | | |
| 100,000-300,000 |1\.6 GHz |32 GB |300 GB |
| 300,000-600,000 |1\.6 GHz |32 GB |450 GB |
| 超过 600,000 个 |1\.6 GHz |32 GB |500 GB |

以下是运行 AD FS 或 Web 应用程序服务器的计算机的最低要求：

- CPU：双核 1.6 GHz 或更高
- 内存：2 GB 或更高
- Azure VM：A2 配置或更高

## 后续步骤
了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->