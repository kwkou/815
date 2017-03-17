<properties
    pageTitle="Azure Active Directory 基于证书的身份验证 - 入门 | Azure"
    description="了解如何在环境中配置基于证书的身份验证"
    author="MarkusVi"
    documentationcenter="na"
    manager="femila" />
<tags
    ms.assetid="c6ad7640-8172-4541-9255-770f39ecce0e"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="02/09/2017"
    wacn.date="03/07/2017"
    ms.author="markvi" />  


# Azure Active Directory 中基于证书的身份验证入门

如果使用基于证书的身份验证，则在 Windows、Android 或 iOS 设备上将 Exchange Online 帐户连接到以下对象时，可通过 Azure Active Directory 使用客户端证书进行身份验证：

- Office 移动应用程序，例如 Microsoft Outlook 和 Microsoft Word

- Exchange ActiveSync (EAS) 客户端

如果配置了此功能，就无需在移动设备上的某些邮件和 Microsoft Office 应用程序中输入用户名和密码组合。

本主题：

- 提供的步骤介绍如何为 Office 365 企业版、商业版和教育版的用户配置并使用基于证书的身份验证。此功能在 Office 365 中国版计划以预览版形式提供。

- 假设用户已配置[公钥基础结构 (PKI)](https://go.microsoft.com/fwlink/?linkid=841737) 和 [AD FS](/documentation/articles/active-directory-aadconnectfed-whatis/)。


## 要求

若要配置基于证书的身份验证，必须满足以下条件：

- 必须在 Azure Active Directory 中配置根证书颁发机构和任何中间证书颁发机构。

- 每个证书颁发机构必须有一个可通过面向 Internet 的 URL 引用的证书吊销列表 (CRL)。

- 必须已在 Azure Active Directory 中至少配置一个证书颁发机构。可以在[配置证书颁发机构](#step-2-configure-the-certificate-authorities)部分查找相关步骤。

- 对于 Exchange ActiveSync 客户端，客户端证书的“使用者可选名称”字段的主体名称或 RFC822 名称值必须为 Exchange Online 中用户的可路由电子邮件地址。Azure Active Directory 会将 RFC822 值映射到目录中的“代理地址”属性。

- 客户端设备必须能够访问至少一个颁发客户端证书的证书颁发机构。

- 必须已向客户端颁发用于客户端身份验证的客户端证书。




## 步骤 1：选择设备平台

第一步，用户需针对所关注的设备平台查看以下内容：

- Office 移动应用程序支持
- 特定的实现要求

存在以下设备平台的相关信息：

- [Android](/documentation/articles/active-directory-certificate-based-authentication-android/)
- [iOS](/documentation/articles/active-directory-certificate-based-authentication-ios/)


## 步骤 2：配置证书颁发机构 <a name="step-2-configure-the-certificate-authorities"></a>

若要在 Azure Active Directory 中配置证书颁发机构，请为每个证书颁发机构上载以下内容：

- 证书的公共部分，格式为 *.cer*
- 证书吊销列表 (CRL) 所在的面向 Internet 的 URL

证书颁发机构的架构如下所示：

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

对于此配置，可以使用 [Azure Active Directory PowerShell 版本 2](https://docs.microsoft.com/powershell/azuread/)：

1. 使用管理员特权启动 Windows PowerShell。
2. 安装 Azure AD 模块。需要安装 [2\.0.0.33 ](https://www.powershellgallery.com/packages/AzureAD/2.0.0.33) 或更高版本。
   
        Install-Module -Name AzureAD -RequiredVersion 2.0.0.33 

作为第一个配置步骤，需建立与租户的连接。建立到租户的连接以后，即可查看、添加、删除和修改在目录中定义的可信证书颁发机构。

### 连接

若要建立与租户的连接，请使用 [Connect-AzureAD](https://docs.microsoft.com/powershell/azuread/v2/connect-azuread) cmdlet：

    Connect-AzureAD 


### 检索 

若要检索在目录中定义的可信证书颁发机构，请使用 [Get-AzureADTrustedCertificateAuthority](https://docs.microsoft.com/powershell/azuread/v2/get-azureadtrustedcertificateauthority) cmdlet。

    Get-AzureADTrustedCertificateAuthority 
 

### 添加

若要创建可信证书颁发机构，请使用 [New-AzureADTrustedCertificateAuthority](https://docs.microsoft.com/powershell/azuread/v2/new-azureadtrustedcertificateauthority) cmdlet：
   
    $cert=Get-Content -Encoding byte "[LOCATION OF THE CER FILE]" 
    $new_ca=New-Object -TypeName Microsoft.Open.AzureAD.Model.CertificateAuthorityInformation 
    $new_ca.AuthorityType=0 
    $new_ca.TrustedCertificate=$cert 
    New-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $new_ca 


### 删除

若要删除可信证书颁发机构，请使用 [Remove-AzureADTrustedCertificateAuthority](https://docs.microsoft.com/powershell/azuread/v2/remove-azureadtrustedcertificateauthority) cmdlet：
   
    $c=Get-AzureADTrustedCertificateAuthority 
    Remove-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $c[2] 


### 修改

若要修改可信证书颁发机构，请使用 [Set-AzureADTrustedCertificateAuthority](https://docs.microsoft.com/powershell/azuread/v2/set-azureadtrustedcertificateauthority) cmdlet：

    $c=Get-AzureADTrustedCertificateAuthority 
    $c[0].AuthorityType=1 
    Set-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $c[0] 


## 步骤 3：配置吊销

若要吊销客户端证书，Azure Active Directory 会从作为证书颁发机构信息的一部分上传的 URL 中提取证书吊销列表 (CRL)，并将其缓存。CRL 中的上次发布时间戳（“生效日期”属性）用于确保 CRL 仍然有效。将定期引用 CRL，以撤销对该列表中证书的访问权限。

如果需要更即时的吊销（例如，如果用户丢失了设备），可以使用户的授权令牌失效。若要使授权令牌失效，请使用 Windows PowerShell 为此特定用户设置 **StsRefreshTokenValidFrom** 字段。必须为要撤销其访问权限的每个用户更新 **StsRefreshTokenValidFrom** 字段。

若要确保撤销仍然有效，必须将 CRL 的**生效日期**设置为晚于 **StsRefreshTokenValidFrom** 所设置的值，并确保相关的证书在 CRL 中。

以下步骤概述了通过设置 **StsRefreshTokenValidFrom** 字段更新授权令牌并使其失效的过程。

**若要配置吊销，请执行以下操作：**

1. 使用管理员凭据连接到 MSOL 服务：
   
        $msolcred = get-credential 
        connect-msolservice -credential $msolcred 

2. 检索用户的当前 StsRefreshTokensValidFrom 值：
   
        $user = Get-MsolUser -UserPrincipalName test@yourdomain.com` 
        $user.StsRefreshTokensValidFrom 

3. 将用户的新 StsRefreshTokensValidFrom 值配置为等于当前时间戳：
   
        Set-MsolUser -UserPrincipalName test@yourdomain.com -StsRefreshTokensValidFrom ("03/05/2016")

所设日期必须属于将来。如果日期不属于将来，则不会设置 **StsRefreshTokensValidFrom** 属性。如果日期属于将来，**StsRefreshTokensValidFrom** 会设置为当前时间（而不是由 Set-MsolUser 命令指示的日期）。


## 步骤 4：测试配置

### 测试证书

进行第一次配置测试时，应尝试使用**设备上的浏览器**登录到 [Outlook Web Access](https://outlook.office365.com) 或 [SharePoint Online](https://microsoft.sharepoint.com)。

如果登录成功，则可确定：

- 已为测试设备预配用户证书
- 已正确配置 AD FS


### 测试 Office 移动应用程序

**若要在 Office 移动应用程序上测试基于证书的身份验证，请执行以下操作：**

1. 在测试设备上，安装 Office 移动应用程序（例如 OneDrive）。
3. 启动应用程序。
4. 输入用户名，然后选择要使用的用户证书。

你应可以成功登录。

### 测试 Exchange ActiveSync 客户端应用程序

若要通过基于证书的身份验证访问 Exchange ActiveSync (EAS)，必须为应用程序提供包含客户端证书的 EAS 配置文件。

EAS 配置文件必须包含以下信息：

- 用于身份验证的用户证书

- EAS 终结点（例如 outlook.office365.com）

若要配置 EAS 配置文件并将其放置在设备上，可以使用移动设备管理 (MDM)，例如 Intune，也可以手动将 EAS 配置文件中的证书放置在设备上。

### 在 Android 上测试 EAS 客户端应用程序

**若要测试证书身份验证，请执行以下操作：**

1. 在应用程序中配置满足上述要求的 EAS 配置文件。
2. 打开应用程序，验证邮件是否正在同步。

<!---HONumber=Mooncake_0227_2017-->
