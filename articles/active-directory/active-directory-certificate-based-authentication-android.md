<properties 
    pageTitle="Android 上基于证书的身份验证入门 | Azure" 
    description="了解如何在 Android 设备的解决方案中配置基于证书的身份验证" 
    services="active-directory" 
    authors="markusvi"  
    documentationCenter="na" 
    manager="femila"/>
<tags 
    ms.service="active-directory" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.tgt_pltfrm="na" 
    ms.workload="identity" 
    ms.date="07/22/2016" 
    ms.author="markvi"
   wacn.date="10/11/2016"/>
    wacn.date="10/11/2016"/>



# Android 上基于证书的身份验证入门 — 公共预览版  

> [AZURE.SELECTOR]
- [iOS](/documentation/articles/active-directory-certificate-based-authentication-ios/)
- [Android](/documentation/articles/active-directory-certificate-based-authentication-android/)


本主题介绍如何在 Android 设备上为 Office 365 企业版、商业版和教育版计划中租户的用户配置并使用基于证书的身份验证 (CBA)。

如果使用 CBA，当你在 Android 或 iOS 设备上将 Exchange Online 帐户连接到以下对象时，将由 Azure Active Directory 使用客户端证书对你进行身份验证：

- Office 移动应用程序，例如 Microsoft Outlook 和 Microsoft Word
- Exchange ActiveSync (EAS) 客户端

如果配置了此功能，就无需在移动设备上的某些邮件和 Microsoft Office 应用程序中输入用户名和密码组合。
 

## 支持的方案和要求  



### 一般要求 


对于本主题中的所有方案，需要执行以下任务：

- 访问证书颁发机构以颁发客户端证书。

- 必须在 Azure Active Directory 中配置证书颁发机构。有关如何完成该配置的详细步骤，请参阅[入门](#getting-started)部分。

- 必须在 Azure Active Directory 中配置根证书颁发机构和任何中间证书颁发机构。

- 每个证书颁发机构必须有一个可通过面向 Internet 的 URL 引用的证书吊销列表 (CRL)。

- 必须颁发客户端证书以进行客户端身份验证。


- 仅对于 Exchange ActiveSync 客户端，客户端证书的"使用者可选名称"字段的主体名称或 RFC822 名称必须为 Exchange Online 中用户的可路由电子邮件地址。Azure Active Directory 会将 RFC822 值映射到目录中的"代理地址"属性。



### Office 移动应用程序支持 

| 应用 | 支持 |
| ---                       | ---          |
| Word/Excel/PowerPoint | ![勾选标记][1] |
| OneNote | 即将支持 |
| OneDrive | ![勾选标记][1] |
| Outlook | ![勾选标记][1] |
| Yammer | ![勾选标记][1] |
| Skype for Business | ![勾选标记][1] |


### 要求  

设备 OS 版本必须为 Android 5.0 (Lollipop) 及更高版本

必须配置联合服务器。


若要让 Azure Active Directory 吊销客户端证书，ADFS 令牌必须具有以下声明：

  - `http://schemas.microsoft.com/ws/2008/06/identity/claims/<serialnumber>`  
（客户端证书的序列号）

  - `http://schemas.microsoft.com/2012/12/certificatecontext/field/<issuer>`  
（客户端证书颁发者的相应字符串）

如果 ADFS 令牌（或任何其他 SAML 令牌）具有这些声明，Azure Active Directory 会将这些声明添加到刷新令牌中。当需要验证刷新令牌时，此信息可用于检查吊销。

作为最佳做法，你应该使用有关如何获取用户证书的说明来更新 ADFS 错误页。

有关更多详细信息，请参阅 [Customizing the AD FS Sign-in Pages](https://technet.microsoft.com/zh-cn/library/dn280950.aspx)（自定义 AD FS 登录页）。



### Exchange ActiveSync 客户端支持 


支持 Android 5.0 (Lollipop) 或更高版本上的某些 Exchange ActiveSync 应用程序。若要确定你的电子邮件应用程序是否支持此功能，请联系应用程序开发人员。



## 入门 


若要开始操作，你需要在 Azure Active Directory 中配置证书颁发机构。针对每个证书颁发机构，上传以下信息：

- 证书的公共部分，格式为 *.cer*

- 证书吊销列表 (CRL) 所在的面向 Internet 的 URL
 

下面是证书颁发机构的架构：

    class TrustedCAsForPasswordlessAuth 
    { 
       CertificateAuthorityInformation[] certificateAuthorities;    
    } 

    class CertificateAuthorityInformation 

    { 
        CertAuthorityType authorityType; 
        X509Certificate trustedCertificate; 
        string crlDistributionPoint; 
        string deltaCrlDistributionPoint; 
        string trustedIssuer; 
        string trustedIssuerSKI; 
    }                

    enum CertAuthorityType 
    { 
        RootAuthority = 0, 
        IntermediateAuthority = 1 
    } 


若要上传信息，可以通过 Windows PowerShell 使用 Azure AD 模块。  
下面是添加、删除或修改证书颁发机构的示例。



### 为 Azure AD 租户配置基于证书的身份验证 

1. 使用管理员特权启动 Windows PowerShell。

2. 安装 Azure AD 模块。需要安装 [1\.1.143.0](http://www.powershellgallery.com/packages/AzureADPreview/1.1.143.0) 或更高版本。

        Install-Module -Name AzureADPreview -RequiredVersion 1.1.143.0 

3. 连接到目标租户：

        Connect-AzureAD 

### 添加新证书颁发机构

1. 设置证书颁发机构的各种属性，并将其添加到 Azure Active Directory：

        $cert=Get-Content -Encoding byte "[LOCATION OF THE CER FILE]" 
        $new_ca=New-Object -TypeName Microsoft.Open.AzureAD.Model.CertificateAuthorityInformation 
        $new_ca.AuthorityType=0 
        $new_ca.TrustedCertificate=$cert 
        New-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $new_ca 

5. 获取证书颁发机构：

        Get-AzureADTrustedCertificateAuthority 


### 检索证书颁发机构列表

检索当前存储在 Azure Active Directory 中的租户的证书颁发机构：

        Get-AzureADTrustedCertificateAuthority 


### 删除证书颁发机构

1.	检索证书颁发机构：

		$c=Get-AzureADTrustedCertificateAuthority 


2. 删除证书颁发机构的证书：

		Remove-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $c[2] 


### 修改证书颁发机构 

1.	检索证书颁发机构：

		$c=Get-AzureADTrustedCertificateAuthority 


2. 修改证书颁发机构上的属性：

		$c[0].AuthorityType=1 

3. 设置**证书颁发机构**：

		Set-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $c[0] 




## 测试 Office 移动应用程序  

若要在 Office 移动应用程序上测试证书身份验证，请执行以下操作：

1.	在测试设备上，从 Google Play Store 中安装 Office 移动应用程序（例如 OneDrive）。

2.	确保已向测试设备预配用户证书。

3.	启动应用程序。

4.	输入用户名，然后选择要使用的用户证书。

你应可以成功登录。





## 测试 Exchange ActiveSync 客户端应用程序

若要通过基于证书的身份验证访问 Exchange ActiveSync，必须为应用程序提供包含客户端证书的 EAS 配置文件。EAS 配置文件必须包含以下信息：

- 用于身份验证的用户证书

- EAS 终结点必须是 outlook.office365.com（因为当前只有 Exchange Online 多租户环境才支持此功能）

通过使用 MDM（例如 Intune）或者手动将 EAS 配置文件中的证书放置在设备上，可以配置 EAS 配置文件并将其放置在设备上。


### 在 Android 上测试 EAS 客户端应用程序 

若要在 Android 5.0 (Lollipop) 或更高版本上对某个应用程序测试证书身份验证，请执行以下步骤：

1. 在应用程序中配置满足上述要求的 EAS 配置文件。


2. 正确配置该配置文件后，打开应用程序，确认邮件正在同步。




## 吊销

若要吊销客户端证书，Azure Active Directory 会从作为证书颁发机构信息的一部分上传的 URL 中提取证书吊销列表 (CRL)，并将其缓存。CRL 中的上次发布时间戳（"生效日期"属性）用于确保 CRL 仍然有效。将定期引用 CRL，以撤销对该列表中证书的访问权限。

如果需要更即时的吊销（例如，如果用户丢失了设备），可以使用户的授权令牌失效。若要使授权令牌失效，请使用 Windows PowerShell 为此特定用户设置 **StsRefreshTokenValidFrom** 字段。必须为要撤销其访问权限的每个用户更新 **StsRefreshTokenValidFrom** 字段。
 
若要确保撤销仍然有效，必须将 CRL 的**生效日期**设置为晚于 **StsRefreshTokenValidFrom** 所设置的值，并确保相关的证书在 CRL 中。
 
以下步骤概述了通过设置 **StsRefreshTokenValidFrom** 字段更新授权令牌并使其失效的过程。

1. 使用管理员凭据连接到 MSOL 服务：

		$msolcred = get-credential 
		connect-msolservice -credential $msolcred 

1.	检索用户的当前 StsRefreshTokensValidFrom 值：

		$user = Get-MsolUser -UserPrincipalName test@yourdomain.com` 
		$user.StsRefreshTokensValidFrom 


1.	将用户的新 StsRefreshTokensValidFrom 值配置为等于当前时间戳：

		Set-MsolUser -UserPrincipalName test@yourdomain.com -StsRefreshTokensValidFrom ("03/05/2016")


所设日期必须属于将来。如果日期不属于将来，则不会设置 **StsRefreshTokensValidFrom** 属性。如果日期属于将来，**StsRefreshTokensValidFrom** 会设置为当前时间（而不是由 Set-MsolUser 命令指示的日期）。


<!--Image references-->
[1]: ./media/active-directory-certificate-based-authentication-android/ic195031.png

<!---HONumber=Mooncake_0926_2016-->
