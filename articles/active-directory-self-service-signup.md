<properties 
	pageTitle="什么是 Azure 的自助注册？| Azure" 
	description="概述 Azure 的自助服务注册以及如何管理注册过程。" 
	services="active-directory" 
	documentationCenter="" 
	authors="curtand"
	manager="stevenpo"
	editor=""/>

<tags 
	ms.service="active-directory" 
	ms.date="05/10/2016"
	wacn.date="06/27/2016"/>


# 什么是 Azure 的自助服务注册？

本主题介绍自助注册过程以及如何接管 DNS 域名。

## 为何使用自助服务注册？

- 让客户更快获得所需的服务。
- 为服务创建基于电子邮件的（迅速传播的）促销。
- 创建基于电子邮件的注册流程，让用户使用易记的工作电子邮件别名快速创建标识。
- 以后可以将非托管 Azure 目录转换成托管目录，并重复用于其他服务。

## 术语和定义

+ **自助注册**：用户注册云服务并让系统根据其电子邮件域在 Azure Active Directory (Azure AD) 中自动为其创建标识的方法。
+ **非托管 Azure 目录**：在其中创建标识的目录。非托管目录是没有全局管理员的目录。
+ **电子邮件验证的用户**：Azure AD 中的一种用户帐户类型。在注册自助服务产品后自动创建标识的用户称为电子邮件验证的用户。电子邮件验证的用户是目录的常规成员，带有 creationmethod=EmailVerified 标记。

## 用户体验

假设电子邮件地址为 Dan@BellowsCollege.com 用户要通过电子邮件接收机密文件。这些文件将由 Azure 权限管理 (Azure RMS) 保护。但是，Dan 的组织 Bellows College 未注册 Azure RMS，并且未部署 Active Directory RMS。在这种情况下，Dan 可以个人注册 RMS 的免费订阅，以便能够阅读受保护的文件。

如果 Dan 是使用 BellowsCollege.com 的电子邮件地址注册此自助服务产品的第一个用户，那么，将在 Azure AD 中为 BellowsCollege.com 创建一个非托管目录。如果来自 BellowsCollege.com 域的其他用户注册了此产品或类似的自助服务产品，则也会在 Azure 的同一个非托管目录中创建电子邮件验证的用户帐户。

## 管理员体验

拥有非托管 Azure 目录 DNS 域名的管理员可以在证明所有权后接管或合并目录。以下部分更详细地介绍了管理员体验，不过本文只会提供摘要：

- 接管某个非托管 Azure 目录后，你就会成为该非托管目录的全局管理员。这有时称为内部接管。
- 在合并非托管 Azure 目录时，会将非托管目录的 DNS 域名添加到托管 Azure 目录，并创建用户到资源的映射，以便用户可以继续访问服务而不会遇到中断。这有时称为外部接管。

## 将在 Azure Active Directory 中创建哪些项目？

#### 目录

- 将为每个域创建一个 Azure Active Directory 目录。
- Azure AD 目录没有全局管理员。

#### 用户

- 对于注册的每个用户，将在 Azure AD 目录中创建一个用户对象。
- 每个用户对象均标记为外部。
- 将为每个用户授予访问其所注册的服务的权限。

### 如何对我拥有的域声明自助服务 Azure AD 目录？

可以通过执行域验证来声明自助服务 Azure AD 目录。域验证会通过创建 DNS 记录证明你拥有该域。

有两种方法可实现 Azure AD 目录的 DNS 接管：

- 内部接管（管理员发现非托管 Azure 目录，并想要转变为托管目录）
- 外部接管（管理员尝试将新域添加到其托管 Azure 目录）

你可能想要验证你是否拥有某个域，因为你要在用户执行自助注册后接管非托管目录；或者，你可能要向现有的托管目录添加新域。例如，你有一个名为 contoso.com 的域，并想要添加名为 contoso.co.uk 或 contoso.uk 的新域。

## 什么是域接管？  

本部分介绍如何验证你是否拥有某个域

### 什么是域验证，为何使用域验证？

为了对目录执行操作，Azure AD 要求你验证 DNS 域的所有权。验证域后，你可以声明目录，并将自助服务目录提升为托管目录，或者将自助服务目录合并到现有的托管目录。

## 域验证示例

有两种方法可以实现目录的 DNS 接管：

+ 内部接管（例如，管理员发现自助服务非托管目录，并想要转变为托管目录）
+ 外部接管（例如，管理员尝试将新域添加到托管目录）

### 内部接管 — 将自助服务非托管目录提升为托管目录

当你执行内部接管时，目录将从非托管目录转换为托管目录。你需要完成 DNS 域名验证，并在验证时在 DNS 区域中创建 MX 记录或 TXT 记录。该操作将会：

+ 验证你是否拥有该域
+ 将目录转变为托管目录
+ 将你设为目录的全局管理员

假设 Bellows College 的 IT 管理员发现该校的用户已注册自助服务产品。作为 DNS 名称 BellowsCollege.com 的注册所有者，该 IT 管理员可以在 Azure 中验证 DNS 名称的所有权，然后接管非托管目录。然后，目录将成为托管目录，并且 IT 管理员将获得 BellowsCollege.com 目录的全局管理员角色。

### 外部接管 — 将自助服务目录合并到现有的托管目录

在外部接管中，你已经有一个托管目录，并希望非托管目录中的所有用户和组都加入该托管目录，而不是拥有两个独立的目录。

作为托管目录的管理员，你可以添加一个域，而该域恰好有一个关联的非托管目录。

假设你是 IT 管理员并且已有 Contoso.com（你的组织注册的域名）的托管目录。你发现组织中的用户使用电子邮件域名 user@contoso.co.uk（你的组织拥有的另一个域名）自助注册了某个产品。这些用户当前在 contoso.co.uk 的非托管目录中拥有帐户。

你不希望管理两个独立的目录，因此，你要将 contoso.co.uk 的非托管目录合并到 contoso.com 的现有 IT 托管目录。

外部接管遵循的 DNS 验证过程与内部接管相同。差别在于：用户和服务将重新映射到 IT 托管目录。

#### 执行外部接管会造成什么影响？

执行外部接管时，会创建用户到资源的映射，以便用户可以继续访问服务而不会遇到中断。许多应用程序（包括面向个人的 RMS）可能很好地处理用户到资源的映射，用户可以像平常一样继续访问这些服务。如果应用程序不能有效地处理用户到资源的映射，你可以显式阻止外部接管以防止给用户带来不好的体验。

#### 服务支持的目录接管

目前，以下服务支持接管：

- RMS


以下服务即将支持接管：

- PowerBI

以下服务不支持接管，需要额外执行管理操作才能在外部接管后迁移用户数据。

- SharePoint/OneDrive


## 如何执行 DNS 域名接管

你可以使用几个选项来执行域验证（并根据需要执行接管）：

1.  Azure 经典管理门户

	通过添加域来触发接管。如果域已经存在一个目录，则你可以选择执行外部接管。

	使用你的凭据登录到 Azure 经典管理门户。导航到现有目录，然后选择“添加域”。

2.  Office 365

	在 Office 365 中，可以使用[管理域](https://support.office.com/article/Navigate-to-the-Office-365-Manage-domains-page-026af1f2-0e6d-4f2d-9b33-fd147420fac2/)页上的选项来处理域和 DNS 记录。请参阅[在 Office 365 中验证域](https://support.office.com/article/Verify-your-domain-in-Office-365-6383f56d-3d09-4dcb-9b41-b5f5a5efd611/)。

3.  Windows PowerShell

	需要执行以下步骤以使用 Windows PowerShell 执行验证。

	步骤 |	要使用的 Cmdlet
	-------	| -------------
	创建凭据对象 | Get-Credential
	连接到 Azure AD | Connect-MsolService
	获取域的列表 | Get-MsolDomain
	创建质询 | Get-MsolDomainVerificationDns
	创建 DNS 记录 | 在 DNS 服务器上执行此操作
	验证质询 | Confirm-MsolEmailVerifiedDomain

例如：

1. 使用用于响应自助服务产品的凭据连接到 Azure AD：
		import-module MSOnline
		$msolcred = get-credential
		connect-msolservice -credential $msolcred

2. 获取域的列表：

	Get-MsolDomain

3. 然后 Get-MsolDomainVerificationDns cmdlet 来创建质询：

	Get-MsolDomainVerificationDns –DomainName *your\_domain\_name* –Mode DnsTxtRecord

	例如：

	Get-MsolDomainVerificationDns –DomainName contoso.com –Mode DnsTxtRecord

4. 复制从此命令返回的值（质询）。

	例如：

	MS=32DD01B82C05D27151EA9AE93C5890787F0E65D9

5. 在公共 DNS 命名空间中，创建包含你在上一步复制的值的 DNS txt 记录。

	此记录的名称即是父域的名称，因此，如果你要使用 Windows Server 中的 DNS 角色创建此资源记录，请将记录名称保留空白，而只在文本框中粘贴该值

6. 运行 Confirm-MsolDomain cmdlet 以验证质询：

	Confirm-MsolEmailVerifiedDomain -DomainName *your\_domain\_name*

	例如：

	Confirm-MsolEmailVerifiedDomain -DomainName contoso.com

如果质询成功，你将返回到提示符，且不会显示错误。

## 如何控制自助服务设置？

目前，管理员有两种自助服务控制方式。他们可以控制：

- 用户是否可以通过电子邮件加入目录。
- 用户是否可以向应用程序和服务对自身授权。


### 如何控制这些功能？

管理员可以使用以下 Azure AD Set-MsolCompanySettings cmdlet 参数配置这些功能：

+ **AllowEmailVerifiedUsers** 控制用户是否可以创建或加入非托管目录。如果将该参数设置为 $false，则电子邮件验证的用户不可以加入目录。
+ **AllowAdHocSubscriptions** 控制用户执行自助服务注册的能力。如果将该参数设置为 $false，则用户不可以执行自助服务注册。


### 这些控制方式如何配合工作？

可以结合使用这两个参数，以更准确地定义如何控制自助服务注册。例如，以下命令允许用户执行自助服务注册，但前提是这些用户已在 Azure AD 中拥有一个帐户（换言之，需要创建电子邮件验证帐户的用户无法执行自助服务注册）：

	Set-MsolCompanySettings -AllowEmailVerifiedUsers $false -AllowAdHocSubscriptions $true

以下流程图解释了这些参数的所有不同组合，以及目录和自助注册的最终状态。

![][1]

有关示例和如何使用这些参数的详细信息，请参阅 [Set-MsolCompanySettings](https://msdn.microsoft.com/zh-cn/library/azure/dn194127.aspx)。

## 另请参阅

-  [如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)

-  [Azure PowerShell](https://msdn.microsoft.com/zh-cn/library/azure/jj156055.aspx)

-  [Azure Cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/azure/jj554330.aspx)

-  [Set-MsolCompanySettings](https://msdn.microsoft.com/zh-cn/library/azure/dn194127.aspx)

<!--Image references-->
[1]: ./media/active-directory-self-service-signup/SelfServiceSignUpControls.png


<!---HONumber=Mooncake_0620_2016-->