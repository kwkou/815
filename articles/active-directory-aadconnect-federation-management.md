<properties
	pageTitle="使用 Azure AD Connect 进行 Active Directory 联合身份验证服务的管理和自定义 | Azure"
	description="使用 Azure AD Connect 管理 AD FS 并使用 Azure AD Connect 和 Powershell 自定义用户 AD FS 管理登录体验。"
	services="active-directory"
	documentationCenter=""
	authors="anandyadavmsft"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/04/2016"
	wacn.date="06/24/2016"/>

# 使用 Azure AD Connect 进行 Active Directory 联合身份验证服务的管理和自定义

本文详细介绍了各种 Active Directory 联合身份验证服务 (AD FS) 相关的且可以使用 Azure AD Connect 执行的任务，以及完整配置 AD FS 场可能需要的其他常见 AD FS 任务。

## AD FS 管理

Azure AD Connect 提供了各种 AD FS 相关的任务，通过使用 Azure AD Connect 向导，可以用最少的用户干预执行这些任务。在通过运行向导来完成安装 Azure AD Connect 后，你可以再次运行向导，以执行其他任务。

### 修复信任

Azure AD Connect 可以检查 AD FS 和 Azure ADtrust 的当前运行状况并采取适当措施来修复信任。请按照以下步骤来修复 Azure AD 和 AD FS 信任。

从可用任务列表选择“修复 AAD 和 ADFS 信任”。

![](media\active-directory-aadconnect-federation-management\RepairADTrust1.PNG)

在“连接到 Azure AD”页上，提供 Azure AD 的全局管理员凭据，然后单击“下一步”。

![](media\active-directory-aadconnect-federation-management\RepairADTrust2.PNG)

在“远程访问凭据”页上，提供域管理员的凭据

![](media\active-directory-aadconnect-federation-management\RepairADTrust3.PNG)

单击“下一步”时，Azure AD Connect 将检查证书运行状况，如果存在问题将显示出来。

![](media\active-directory-aadconnect-federation-management\RepairADTrust4.PNG)

“准备好配置”页将显示为修复信任将执行的操作的列表。

![](media\active-directory-aadconnect-federation-management\RepairADTrust5.PNG)

单击“安装”以继续操作并修复信任。

>[AZURE.NOTE] Azure AD Connect 只能对自签名的证书进行修复/采取措施。Azure AD connect 无法修复第三方证书。

### 添加新的 AD FS 服务器

> [AZURE.NOTE] Azure AD Connect 要求 PFX 证书文件添加 AD FS 服务器。因此，只有使用 Azure AD Connect 配置了 AD FS 场，才能够执行此操作。

选择“部署其他联合服务器”，然后单击“下一步”。

![](media\active-directory-aadconnect-federation-management\AddNewADFSServer1.PNG)

在“连接到 Azure AD”页上，提供 Azure AD 的全局管理员凭据，然后单击“下一步”。

![](media\active-directory-aadconnect-federation-management\AddNewADFSServer2.PNG)

在下一页上，提供域管理员凭据。

![](media\active-directory-aadconnect-federation-management\AddNewADFSServer3.PNG)

在下一页上，Azure AD Connect 将要求你提供在使用 Azure AD Connect 配置新的 AD FS 场时提供的 pfx 文件的密码。单击“输入密码”来输入 PFX 文件的密码。

![](media\active-directory-aadconnect-federation-management\AddNewADFSServer4.PNG)

![](media\active-directory-aadconnect-federation-management\AddNewADFSServer5.PNG)

在下一页上，提供其他服务器的名称或要添加到 AD FS 场的 IP 地址。

![](media\active-directory-aadconnect-federation-management\AddNewADFSServer6.PNG)

单击“下一步”并检查最终的“配置”页。Azure AD Connect 完成将服务器添加到 AD FS 场后，将提供验证连接性的选项。

![](media\active-directory-aadconnect-federation-management\AddNewADFSServer7.PNG)

![](media\active-directory-aadconnect-federation-management\AddNewADFSServer8.PNG)

### 添加新的 AD FS Web 应用程序代理服务器

> [AZURE.NOTE] Azure AD Connect 要求 PFX 证书文件添加 Web 应用程序代理服务器。因此，只有使用 Azure AD Connect 配置了 AD FS 场，才能够执行此操作。

从可用任务列表中选择“部署 Web 应用程序代理”。

![](media\active-directory-aadconnect-federation-management\WapServer1.PNG)

在下一页上，提供 Azure 全局管理员凭据。

![](media\active-directory-aadconnect-federation-management\wapserver2.PNG)

接下来，你将看到“指定 SSL 证书”页，你需要在其中提供在使用 Azure AD Connect 配置 AD FS 场时所提供的 PFX 文件的密码。

![](media\active-directory-aadconnect-federation-management\WapServer3.PNG)

![](media\active-directory-aadconnect-federation-management\WapServer4.PNG)

在下一页上，添加将作为 Web 应用程序代理添加的服务器。因为 Web 应用程序代理服务器可能加入或可能未加入到域，因此向导将要求向正被添加的服务器添加管理凭据。

![](media\active-directory-aadconnect-federation-management\WapServer5.PNG)

在“代理信任凭据”页上，提供管理凭据以配置代理信任并访问 AD FS 场中的主服务器。

![](media\active-directory-aadconnect-federation-management\WapServer6.PNG)

在“准备好配置”页上，向导显示了将执行的操作的列表。

![](media\active-directory-aadconnect-federation-management\WapServer7.PNG)

单击“安装”以完成配置。完成配置后，向导将提供验证到服务器的连接性的选项。单击“验证”以检查连接性。

![](media\active-directory-aadconnect-federation-management\WapServer8.PNG)

### 添加新的联合域

使用 Azure AD Connect 很容易添加与 Azure AD 联合的新域。Azure AD Connect 不仅添加新域用于联合身份验证，而且修改了声明规则，以便在你有多个域与 Azure AD 联合时，正确反映发布者。

若要添加的新的联合域，请选择任务“添加其他 Azure AD 域”。

![](media\active-directory-aadconnect-federation-management\AdditionalDomain1.PNG)

在向导的下一页上，提供 Azure AD 的全局管理员凭据。

![](media\active-directory-aadconnect-federation-management\AdditionalDomain2.PNG)

在远程访问凭据上提供域管理员凭据。

![](media\active-directory-aadconnect-federation-management\additionaldomain3.PNG)

在下一页上，向导将提供你想要与之联合本地目录的 Azure AD 域的列表。从列表中选择域。

![](media\active-directory-aadconnect-federation-management\AdditionalDomain4.PNG)

选择域后，向导将为你提供有关向导将采取的进一步操作以及配置产生的影响的适当信息。在某些情况下，如果你选择的域尚未在 Azure AD 中进行验证，则向导将为你提供帮助验证域的信息。有关验证域的详细信息，请参阅 [Add your custom domain name to Azure Active Directory（将自定义域名添加到 Azure Active Directory）](/documentation/articles/active-directory-add-domain/)。

单击“下一步”，“准备好配置”页将显示 Azure AD Connect 将执行的操作的列表。单击“安装”以完成配置。

![](media\active-directory-aadconnect-federation-management\AdditionalDomain5.PNG)

## AD FS 自定义

以下各节提供了有关如何执行一些常见任务的详细信息。为了自定义 AD FS 登录页，你可能必须完成这些任务。

### 添加自定义公司徽标或插图

若要更改在登录页上显示的公司徽标，请使用以下 PowerShell Windows PowerShell cmdlet 和语法。

> [AZURE.NOTE] 建议徽标维度为 260x35 @ 96 dpi，且文件大小不应超过 10 KB。

    Set-AdfsWebTheme -TargetName default -Logo @{path="c:\Contoso\logo.PNG"}

> [AZURE.NOTE] TargetName 参数是必需项。随 AD FS 一起发布的默认主题名为“默认”。
 

### 添加登录说明

若要将登录页描述添加到登录页上，请使用以下 Windows PowerShell PowerShell cmdlet 和语法。

    Set-AdfsGlobalWebContent -SignInPageDescriptionText "<p>Sign-in to Contoso requires device registration. Click <A href='http://fs1.contoso.com/deviceregistration/'>here</A> for more information.</p>"

### 修改 AD FS 声明规则

AD FS 提供指定用于发布声明的自定义规则的选项。它支持丰富的声明语言，让你可以使用可用的资源创建自定义声明规则。有关详细信息，你可以查看[此处](https://technet.microsoft.com/library/dd807118.aspx)的文章。

以下各节详细介绍了如何针对与 Azure AD 和 AD FS 联合身份验证有关的某些情况编写自定义规则。

#### 属性中存在的值上的不可变 ID 条件

当对象将同步到 Azure AD 时，通过 Azure AD Connect，你可以指定一个属性以用作源锚点。有可能你想要发布不可变 ID 声明，具体取决于自定义属性中的值是否为非空这个条件。例如，假设你已选择 ms-ds-consistencyguid 作为源锚点的属性，当该属性有一个针对它的值时，想要将 ImmutableID 发布为 ms-ds-consistencyguid；否则将 objectGuid 作为不可变 ID 发布。如下所示，你可以构建自定义声明规则集：

**规则 1 （查询属性）**

    c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"]
    => add(store = "Active Directory", types = ("http://contoso.com/ws/2016/02/identity/claims/objectguid", "http://contoso.com/ws/2016/02/identity/claims/msdsconcistencyguid"), query = "; objectGuid,ms-ds-consistencyguid;{0}", param = c.Value);

在此规则中，你只需对 Active Directory 的用户查询 ms-ds-consistencyguid 和 objectguid 的值。请将应用商店名称更改为 ADFS 部署中可用的适当应用商店名称，并将声明类型更改为在联合身份验证中为 objectGUID 和 ms-ds-consistencyguid 定义的正确声明类型。我在我的测试环境中定义自定义声明类型。

此外，通过使用“添加”和不“发布”，你可以避免添加实体的传出问题，只将这些值作为中间值。确定了要用作不可变 ID 的值后，就可以在稍后的规则中发布声明

**规则 2：（检查用户是否存在 ms ds consistencyguid）**

    NOT EXISTS([Type == "http://contoso.com/ws/2016/02/identity/claims/msdsconcistencyguid"])
    => add(Type = "urn:anandmsft:tmp/idflag", Value = "useguid");

如果没有为用户填充 ms-ds-concistencyguid，则此规则只是定义设置为“useguid”的临时标志“idflag”。这背后的逻辑在于 ADFS 不允许空的声明。因此，在规则 1 中添加声明 http://contoso.com/ws/2016/02/identity/claims/objectguid 和 http://contoso.com/ws/2016/02/identity/claims/msdsconcistencyguid 时，你可以最终以 msdsconsistencyguid 声明结尾的唯一条件是已经为用户填充了该值。如果未填充该值，在 ADFS 中它就会作为空值出现，然后立即删除。众所周知，所有对象都将含有 ObjectGuid，以便在规则 1 执行后，声明仍将始终存在

**规则 3：如果存在，将 ms-ds-consistencyguid 作为不可变 ID 发布**

    c:[Type == "http://contoso.com/ws/2016/02/identity/claims/msdsconcistencyguid"]
    => issue(Type = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier", Value = c.Value);

这是隐式的存在性检查。如果声明的值存在，则将其作为不可变 ID 发布。请注意我正在发布 nameidentifier 声明。你需要将其更改为你的环境中不可变 ID 的适当声明类型。

**规则 4：如果 ms-ds-consistencyGuid 不存在，则将 Object Guid 作为不可变 ID 发布**

    c1:[Type == "urn:anandmsft:tmp/idflag", Value =~ "useguid"]
    && c2:[Type == "http://contoso.com/ws/2016/02/identity/claims/objectguid"]
    => issue(Type = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier", Value = c2.Value);

在此规则中，你只需检查临时标志“idflag”，并根据该值决定是否发布声明。

> [AZURE.NOTE] 这些规则的顺序非常重要。

#### 具有子域 UPN 的 SSO

你可以添加多个域，以使用 Azure AD Connect 进行联合（[添加新的联合域](/documentation/articles/active-directory-aadconnect-federation-management/#add-a-new-federated-domain)）。UPN 声明将需要进行修改，以便发布者 ID 对应于根域而非子域，因为联合根域也涵盖子级。

默认情况下，发布者 ID 的声明规则设置为：


	c:[Type 
	== “http://schemas.xmlsoap.org/claims/UPN“]

	=> issue(Type = “http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid“, Value = regexreplace(c.Value, “.+@(?<domain>.+)“, “http://${domain}/adfs/services/trust/“));

![默认发布者 ID 声明](media\active-directory-aadconnect-federation-management\issuer_id_default.png)

默认规则只需使用 UPN 后缀，并将其用于发布者 ID 声明中。例如，John 是 sub.contoso.com 中的用户，而 contoso.com 与 Azure AD 联合。John 在登录 Azure AD 时输入 john@sub.contoso.com 作为用户名，则 AD FS 中的默认发布者 ID 声明规则将按以下方式对其进行处理：

c:[Type == “http://schemas.xmlsoap.org/claims/UPN“]

=> issue(Type = “http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid“, Value = regexreplace(john@sub.contoso.com, “.+@(?<domain>.+)“, “http://${domain}/adfs/services/trust/“));

**声明值：**http://sub.contoso.com/adfs/services/trust/

若要只是在发布者声明值中包含根域，请将声明规则更改为：

	c:[Type == “http://schemas.xmlsoap.org/claims/UPN“]

	=> issue(Type = “http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid“, Value = regexreplace(c.Value, “^((.*)([.|@]))?(?<domain>[^.]*[.].*)$”, “http://${domain}/adfs/services/trust/“));

## 后续步骤

请了解[用户登录选项](/documentation/articles/active-directory-aadconnect-user-signin/)的详细信息


<!---HONumber=Mooncake_0613_2016-->