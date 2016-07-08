<properties
	pageTitle="Office 365 和 Azure AD 用户证书续订指南。| Azure"
	description="本文向 Office 365 用户说明了如何解决向其发送证书续订通知的电子邮件的问题。"
	services="active-directory"
	documentationCenter=""
	authors="billmath"
	manager="stevenpo"
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="05/03/2016"
	wacn.date="06/14/2016"/>


# 续订 Office 365 和 Azure AD 的联合身份验证证书

如果你收到电子邮件或门户通知，要求你续订 Office 365 的证书，则可参阅本文，了解如何解决此问题并防止其再次发生。本文假定你使用 AD FS 作为联合服务器。

>[AZURE.IMPORTANT] 请注意，执行下列操作之一后，在 Windows Server 2012 或 Windows Server 2008 R2 上通过代理进行身份验证可能会失败：
>
- 证书在 AD FS 中滚动更新后，你的代理续订了信任令牌
- 你手动替换了 AD FS 证书
>     
可以使用一个修补程序来解决此问题。请参阅[在 Windows Server 2012 或 Windows 2008 R2 SP1 中通过代理进行身份验证失败](http://support.microsoft.com/kb/3094446)

## 查看是否需要执行任何操作

如果你使用的是 AD FS 2.0 或更高版本，Office 365 和 Azure AD 会在你的证书过期之前自动对其进行更新。你不需要执行任何手动步骤，也不需要以计划任务的形式运行脚本。为此，你必须让以下两项默认的 AD FS 配置设置生效：

- AD FS 属性 AutoCertificateRollover 必须设置为 True，表示 AD FS 会在旧的令牌签名和令牌解密证书过期之前自动生成新的。
	- 如果该值为 False，则表示你使用的是自定义证书设置。如需全面指导，请转到[此处](https://msdn.microsoft.com/library/azure/JJ933264.aspx#BKMK_NotADFSCert)。
- 联合身份验证元数据必须可以通过公共 Internet 进行使用。

	下面是如何进行检查：

	- 通过在主联合服务器的 PowerShell 命令窗口中执行以下命令，验证你的 AD FS 安装使用的是自动证书滚动更新：

	`PS C:\> Get-ADFSProperties`

（请注意，如果你使用的是 AD FS 2.0，则需先运行 Add-Pssnapin Microsoft.Adfs.Powershell）。

在生成的输出中，检查以下设置：
	
	AutoCertificateRollover :True

从公共 Internet（企业网络之外）上的计算机导航到以下 URL，查看你的联合身份验证元数据是否可以公开访问：


https://(your_FS_name)/federationmetadata/2007-06/federationmetadata.xml

其中，`(your_FS_name) ` 将替换为你组织使用的联合身份验证服务主机名，例如 fs.contoso.com。如果你能够成功验证这两项设置，则无需执行任何其他操作。

示例：https://fs.contoso.com/federationmetadata/2007-06/federationmetadata.xml

## 如果你决定手动更新证书
每当你要手动更新 AD FS 证书时，必须根据[此部分](#if-your-metadata-is-not-publicly-accessible)中的“手动更新 Office 365 联合身份验证信任属性”下的步骤所述，使用 PowerShell 命令 Update-MsolFederatedDomain 更新你的 Office 365 域

## 如果你的 AutoCertificateRollover 属性设置为 False

如果你的 AutoCertificateRollover 属性设置为 False，则你使用的是非默认 AD FS 证书设置。这种情况最常见的原因是，你的组织通过组织证书颁发机构来管理注册的 AD FS 证书。在这种情况下，你需要自行续订和更新证书。使用[此处](https://msdn.microsoft.com/library/azure/JJ933264.aspx#BKMK_NotADFSCert)的指南。

## 如果你的元数据不可公开访问
如果 AutocertificateRollover 设置为 True 而联合身份验证元数据不可公开使用，则可执行以下步骤以确保在本地和云中对证书进行了更新：

### 确保 AD FS 系统已生成新的证书。

- 确保你已登录主 AD FS 服务器。
- 通过打开 PowerShell 命令窗口并运行以下命令，检查 AD FS 中的当前签名证书：

`PS C:\>Get-ADFSCertificate –CertificateType token-signing.`

（请注意，如果你使用的是 AD FS 2.0，则需先运行 Add-Pssnapin Microsoft.Adfs.Powershell）


- 查看命令输出中是否存在任何已列出的证书。如果 AD FS 已生成新证书，你应该会在输出中看到两个证书：一个证书的 IsPrimary 值为 True，NotAfter 日期为 5 天内；另一个证书的 IsPrimary 为 False，NotAfter 大约为未来的 1 年。
	
- 如果你只看到一个证书，且 NotAfter 日期在 5 天内，则需执行以下步骤以生成新的证书。

- 若要生成新的证书，请在 PowerShell 命令提示符下执行以下命令：`PS C:\>Update-ADFSCertificate –CertificateType token-signing`。

- 通过再次运行以下命令来验证更新：PS C:\>Get-ADFSCertificate –CertificateType token-signing
	- 此时会列出两个证书，其中一个的 NotAfter 日期大约为未来的 1 年，其 IsPrimary 值为 False。

- 接下来，若要手动更新 Office 365 联合身份验证信任属性，请按以下步骤操作。




### 手动更新 Office 365 联合身份验证信任属性，按以下步骤操作。

1.	打开用于 Windows PowerShell 的 Microsoft Azure Active Directory 模块。
2.	运行 $cred=Get-Credential。当此 cmdlet 提示你输入凭据时，键入你的云服务管理员帐户凭据。
3.	运行 Connect-MsolService –Credential $cred。此 cmdlet 会将你连接到云服务。通过工具运行任何其他已安装的 cmdlet 之前，必须创建将你连接到云服务的上下文。
4.	如果你在并非用作 AD FS 主联合服务器的计算机上运行这些命令，请运行 Set-MSOLAdfscontext -Computer <AD FS primary server>，其中 <AD FS primary server> 是主 AD FS 服务器的内部 FQDN 名称。此 cmdlet 生成将你连接到 AD FS 的上下文。
5.	运行 Update-MSOLFederatedDomain –DomainName <domain>。此 cmdlet 会将 AD FS 中的设置更新到云服务中，并配置两者之间的信任关系。

>[AZURE.NOTE] 如果你需要支持多个顶级域（例如 contoso.com 和 fabrikam.com），则必须将 SupportMultipleDomain 开关用于任何 cmdlet。有关详细信息，请参阅“支持多个顶级域”。最后，请确保所有 Web 应用程序代理服务器都通过 [Windows Server May 2014](http://support.microsoft.com/kb/2955164) 汇总进行了更新，否则代理可能无法使用新证书自行进行更新，导致服务中断。

<!---HONumber=Mooncake_0606_2016-->