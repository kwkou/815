<properties 
	pageTitle="Office 365 和 Azure AD 用户的证书续订指南。" 
	description="这篇文章向 Office 365 用户介绍如何解决通知他们续订证书的电子邮件的相关问题。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="swadhwa" 
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="07/10/2015" 
	wacn.date="08/29/2015"/>


# 续订 Office 365 和 Azure AD 的联合证书

如果你收到要求你续订 Office 365 证书的电子邮件或门户通知，这篇文章旨在帮助你解决此问题并防止其再次发生。本文假定你使用 AD FS 作为联合服务器。

## 检查你是否有执行任何操作

如果你正在使用 AD FS 2.0 或更高版本，Office 365 和 Azure AD 将在过期之前自动更新你的证书。不需要执行任何手动步骤或运行作为计划任务的脚本。要运行此工作，以下两个默认的 AD FS 配置设置都必须是有效的：

- AD FS 属性 AutoCertificateRollover 必须设置为 True，其指示 AD FS 将在旧的过期之前，自动生成新的标记签名和标记解密证书。
	- 如果值为 False，则你正在使用自定义证书设置。转到[此处](https://msdn.microsoft.com/zh-cn/library/azure/JJ933264.aspx#BKMK_NotADFSCert)以查看全面指导。
- 联合元数据必须可用于公共 internet。
	
	此处为如何检查：

	- 验证 AD FS 安装正在通过在主联合服务器上的 PowerShell 命令窗口中执行以下命令来使用自动证书滚动更新：

	`PS C:\> Get-ADFSProperties`

（请注意如果你正在使用 AD FS 2.0，你将需要运行 Add-pssnapin Microsoft.Adfs.Powershell）其他。

请通过从公共 internet（非企业网络）上的计算机导航到以下 URL 检查你的联合元数据是可以公开访问的：


https://(your_FS_name)/federationmetadata/2007-06/federationmetadata.xml

其中，`(your_FS_name) ` 将被替换为你的组织所使用联合服务主机名，例如 fs.contoso.com。如果你能成功够验证两个设置，则你无需执行任何其他操作。

示例：https://fs.contos.com/federationmetadata/2007-06/federationmetadata.xml

## 如果你的 AutoCertificateRollover 属性设置为 False

如果 AutoCertificateRollover 属性设置为 False，那么你正在使用非默认 AD FS 证书设置。最常见的原因是，你的组织管理的 AD FS 证书是从组织的证书颁发机构注册的。在这种情况下，你需要自行续订并更新你的证书。在[此处](https://msdn.microsoft.com/zh-cn/library/azure/JJ933264.aspx#BKMK_NotADFSCert)使用指南。

## 如果你的元数据不可公开访问
如果 AutocertificateRollover 设置为 True，但你的联合元数据未公开提供，使用下面的过程以确保在本地和云中更新你的证书：

### 验证你的 AD FS 系统已生成新证书。 

- 验证你登录到主 AD FS 服务器。
- 通过打开 PowerShell 命令窗口并运行以下命令，检查 AD FS 中的当前签名证书： 

`PS C:\>Get-ADFSCertificate –CertificateType token-signing.`

（请注意如果你正在使用 AD FS 2.0，你首先将需要运行 Add-pssnapin Microsoft.Adfs.Powershell）


- 查看任何列出的证书的命令输出。如果 AD FS 生成了一个新的证书，你在输出中应看到两个证书：一个的 IsPrimary 值为 True 且 NotAfter 日期是在 5 天内，另一个的 IsPrimary 为 False 且 NotAfter 是在将来的一年左右。
	
- 如果你只看到一个证书，且 NotAfter 日期是在 5 天内，你需要通过执行以下步骤来生成新证书。

- 若要生成新证书，请在 PowerShell 命令提示符处执行以下命令：`PS C:\>Update-ADFSCertificate –CertificateType token-signing`。

- 通过再次运行以下命令来验证更新：PS c: > Get ADFSCertificate –CertificateType token-signing
- 接下来，若要手动更新 Office 365 联合信任属性，请按照下列步骤。

现在应列出两个证书，其中一个的 NotAfter 日期为将来一年左右，且 IsPrimary 值为 False。


### 请按照下列步骤，手动更新 Office 365 联合信任属性。

1.	打开 Windows PowerShell 的 Windows Azure Active Directory 模块。
2.	运行 $cred=Get-Credential。当此 cmdlet 提示你键入凭据时，键入你的云服务管理员帐户凭据。
3.	运行 Connect-MsolService –Credential $cred。此 cmdlet 会将你连接到云服务。在运行任何由该工具安装的其他 cmdlet 之前，需要创建将你连接到云服务的上下文。
4.	如果你正在不是 AD FS 主联合服务器的计算机上运行这些命令，请运行 Set-MSOLAdfscontext -Computer <AD FS primary server>，其中 <AD FS primary server> 是主 AD FS 服务器的内部 FQDN 名称。此 cmdlet 会创建将你连接到 AD FS 的上下文。 
5.	运行 Update-MSOLFederatedDomain – DomainName <domain>。此 cmdlet 将设置从 AD FS 更新到云服务并配置两者之间的信任关系。

>[AZURE.NOTE]如果需要支持多个顶级域（例如 contoso.com 和 fabrikam.com），则必须对任何 cmdlet 使用 SupportMultipleDomain 开关。更多有关详细信息，请参阅多个顶级域的支持。最后，确保所有 Web 应用程序代理服务器都已采用 [Windows Server 2014](http://support.microsoft.com/kb/2955164) 汇总进行更新，否则这些代理可能无法使用新的证书进行自我更新，从而导致服务中断。

<!---HONumber=67-->