<properties 
	pageTitle="Azure AD Connect 的自定义安装" 
	description="本文档详细介绍了 Azure AD Connect 的自定义安装选项。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="swadhwa" 
	editor="curtand"/>

<tags 
	ms.service="active-directory"   
	ms.date="05/28/2015" 
	wacn.date="08/29/2015"/>

# Azure AD Connect 的自定义安装


以下文档提供了有关对 Azure AD Connect 使用自定义安装选项的信息。如果你要设置其他配置选项，或需要在快速安装中未包含可选功能，则可以使用此选项。

有关“快速安装”的信息，请参阅[快速安装](/documentation/articles/active-directory-aadconnect-get-started#express-installation-of-azure-ad-connect)。有关从 DirSync 升级到 Azure AD Connect 的信息，请参阅[将 DirSync 升级到 Azure Active Directory Connect。](/documentation/articles/active-directory-aadconnect-dirsync-upgrade-get-started)



## 安装所需的组件

在安装同步服务时，可以不勾选可选配置部分，然后 Azure AD Connect 将自动完成所有设置。这包括设置 SQL Server 2012 Express 实例以及创建相应的组并为其分配权限。如果想要更改默认值，则可以使用下表来了解可用的可选配置选项。

![所需的组件](./media/active-directory-aadconnect-get-started-custom/requiredcomponents.png)


可选配置 | 描述 
------------- | ------------- |
SQL Server 名称 |用于指定 SQL Server 名称和实例名称。如果你已拥有想要使用的 AD 数据库服务器，请选择此选项。
服务帐户 |默认情况下，Azure AD Connect 将为同步服务创建要使用的本地服务帐户。问题在于，密码是自动生成的，而安装 Azure AD Connect 的人员并不知道该密码。在大多数情况下，使用默认设置不会有问题，但是，如果你想要执行一些高级配置（例如，划归已同步的组织单位的范围），则需要创建一个帐户并选择你自己的密码。但如果你使用远程 SQL 服务器，则需要在域中设置一个服务帐户并知道密码。在这些情况下，请输入要使用的服务帐户。 |
权限 | 默认情况下，在安装同步服务时，Azure AD Connect 将在服务器本地创建四个组。这些组是：管理员组、操作员组、浏览组和密码重置组。如果你想要指定自己的组，可在此处指定。


## 用户登录
安装所需的组件后，系统会要求你指定用户要使用的单一登录方法。下表提供了可用选项的简短说明。

![用户登录](./media/active-directory-aadconnect-get-started-custom/usersignin.png)



单一登录选项 | 描述 
------------- | ------------- |
密码同步 |用户可以使用登录本地网络时所用的同一密码登录 Microsoft 云服务，例如 Office 365、Dynamics CRM 和 Windows InTune。用户密码将通过密码哈希同步到 Azure，并在云中进行身份验证。
使用 AD FS 进行联合身份验证|用户可以使用登录本地网络时所用的同一密码登录 Microsoft 云服务，例如 Office 365、Dynamics CRM 和 Windows InTune。用户将重定向到其本地 AD FS 实例以进行登录，并在本地完成身份验证。
不要配置| 不会对任一功能进行安装和配置。如果你已有第三方联合服务器或部署了另一个现有解决方案，请选择此选项。



## 连接到 Azure AD
在“连接到 Azure AD”屏幕中，输入全局管理员的帐户和密码。请确保此帐户未启用多重身份验证。否则会导致身份验证失败。请注意，此帐户只会用于在 Azure AD 中创建服务帐户，在向导完成后将不会使用。

![用户登录](./media/active-directory-aadconnect-get-started-custom/connectaad.png)


### 连接你的目录
若要连接到你的 Active Directory 域服务，Azure AD Connect 工具需要使用具有足够权限的帐户的凭据。此帐户可以是普通用户帐户，因为该账户只需要默认的读取权限。不过，根据你的方案，可能会需要其他权限。有关详细信息，请参阅 [Azure AD Connect 帐户摘要](/documentation/articles/active-directory-addconnect-account-summary)

![用户登录](./media/active-directory-aadconnect-get-started-custom/connectdir.png)


### 唯一标识你的用户

“跨林匹配”功能允许你定义如何在 Azure AD 中呈现你的 AD DS 林中的用户。一个用户可以在所有林中只呈现一次，也可以使用已启用和已禁用帐户的组合。

![用户登录](./media/active-directory-aadconnect-get-started-custom/unique.png)


设置 | 描述 
------------- | ------------- |
我的用户在所有林中只呈现一次| 将所有用户在 Azure AD 中创建为单独的对象。<br> 不会在 Metaverse 中联接对象。
邮件属性 | 如果邮件属性在不同的林中具有相同的值，则此选项将联接用户和联系人。当已使用 GALSync 创建了联系人时，建议使用此选项。
ObjectSID 和 msExchangeMasterAccountSID|此选项将帐户林中的已启用用户与 Exchange 资源林中的已禁用用户进行联接。这也称为 Exchange 中的链接邮箱。
sAMAccountName 和 MailNickName|此选项根据预期可以在其中找到用户登录 ID 的属性进行联接。
我自己的属性|此选项允许你选择自己的属性。**限制：**确保选择 Metaverse 中已存在的属性。如果你选择自定义属性，向导将无法完成。

- **源定位点** - sourceAnchor 属性是一个在用户对象的生命期内不会改变的属性。它是链接本地用户与 Azure AD 中用户的主密钥。由于无法更改该属性，因此你必须规划好要使用的合适属性。objectGUID 就是不错的候选项。除非在林/域之间移动用户帐户，否则此属性不会更改。在要在林间移动帐户的多林环境中，必须使用另一个属性，例如具有 employeeID 的属性。要避免某人结婚时会改变的属性，或会更改分配的属性。由于不可以使用带有 @ 符号的属性，因此无法使用 email 和 userPrincipalName。属性也区分大小写，因此在林间移动对象时，请务必保留大写/小写。对二进制属性而言，值采用 Base64 编码，但对其他属性类型而言，值会保留未编码状态。在联合方案和某些 Azure AD 接口中，此属性也称为 immutableID。

- **UserPrincipalName** - 属性 userPrincipalName 是用户登录到 Azure AD 和 Office 365 时使用的属性。应在同步处理用户前在 Azure AD 中对使用的域（也称为 UPN 后缀）进行验证。强烈建议保留默认属性 userPrincipalName。如果此属性不可路由且无法验证，则可以选择另一个属性，例如，选择 email 作为保存登录 ID 的属性。警告：使用替代 ID 将与所有 Office 365 工作负载不兼容。有关详细信息，请参阅 https://technet.microsoft.com/zh-cn/library/dn659436.aspx。





### 根据组同步筛选
根据组筛选功能可让你执行小型试验，试验中应该只会在 Azure AD 和 Office 365 内创建一小部分对象。若要使用这项功能，请在 Active Directory 中创建一个组，并添加应该以直接成员身份与 Azure AD 同步的用户和组。你稍后可以在此组中添加和删除用户，以维护应该要在 Azure AD 中显示的对象列表。若要使用这项功能，请在自定义路径中查看以下页面：

![同步筛选](./media/active-directory-aadconnect-get-started-custom/filter2.png)


### 可选功能

此屏幕可让你针对特定方案选择可选功能。以下是各项功能的简短说明。

![快速安装](./media/active-directory-aadconnect-get-started-custom/of.png)


可选功能 | 描述
-------------------    | ------------- | 
Exchange 混合部署 |“Exchange 混合部署”功能通过将 Azure AD 中的特定属性集同步回到本地目录，使 Exchange 邮箱能够在本地和 Azure 中共存。
Azure AD 应用和属性筛选|通过启用“Azure AD 应用和属性筛选”，可以根据向导后续页上的特定集定制同步属性集。这会在向导中打开两个附加的配置页。  
密码写回|通过启用“密码写回”，源自 Azure AD 的密码更改将写回到本地目录。
用户写回|通过启用“用户写回”，在 Azure AD 中创建的用户将写回到本地目录。这会在向导中打开一个附加的配置页。  
目录扩展属性同步|通过启用“目录扩展属性同步”，可将指定的属性同步到 Azure AD。这会在向导中打开一个附加的配置页。  

有关其他配置选项（例如，更改默认配置、使用同步规则编辑器和声明性设置），请参阅[管理 Azure AD Connect](/documentation/articles/active-directory-aadconnect-whats-next)




添加包含用户和组的组名称。只有此组的成员才会同步到 Azure AD。

## 目录扩展属性同步
借助目录扩展，你可以使用组织添加的自定义属性或 Active Directory 中的其他属性，在 Azure AD 中扩展架构。若要使用这项功能，请在“可选功能”页上选择“目录扩展属性同步”。这样，你便可以进入此页，并在其中选择其他属性。

![同步筛选](./media/active-directory-aadconnect-get-started-custom/extension2.png)


仅支持单值属性，且值不能超过 250 个字符。将使用选择的属性来扩展 Metaverse 和 Azure AD 架构。在 Azure AD 中，将添加具有这些属性的新应用程序。

![同步筛选](./media/active-directory-aadconnect-get-started-custom/extension3.png)


现在可以通过 Graph 提供这些属性：

![同步筛选](./media/active-directory-aadconnect-get-started-custom/extension4.png)

## 用户写回（预览）
“用户写回”可让你获取 Azure AD 中创建的用户（通过门户、图形、PowerShell 或任何其他方法），然后将用户写回本地 ADDS。若要启用该功能，请在“可选功能”页面上选择“用户写回”。现在，你会看到要在其中创建这些用户的位置。默认配置将在 AD DS 中的一个位置创建所有用户。

![同步筛选](./media/active-directory-aadconnect-get-started-custom/writeback2.png)

将创建使用随机密码的用户，因此你必须在 AD DS 中重置密码，才能使用户真正能够登录。

>[AZURE.NOTE]“密码同步”和“密码写回”功能与此预览功能不兼容。

## 组回写（预览）
可选功能中的“组回写”选项可使你将“Office 365 中组”写回到安装了 Exchange 的林中。这是始终在云中受控制的新组类型。可以在 outlook.office365.com 或 myapps.microsoft.com 中查找到此组类型，如下所示：


![同步筛选](./media/active-directory-aadconnect-get-started-custom/office365.png)



![同步筛选](./media/active-directory-aadconnect-get-started-custom/myapps.png)


此组将在本地 AD DS 中显示为通讯组。你的本地 Exchange 服务器必须位于 Exchange 2013 累计更新 8（2015 年 3 月发布）中以识别这种新的组类型。

**注意**

- 当前不填充通讯簿属性。最简单的方法是从组织中的另一个组中找到通讯簿属性并在同步引擎之外对此进行填充。**进行此操作的最简单的方法是使用 PowerShell cmdlet 的 update-recipient。**
- 只有使用 Exchange 架构的林才是组的有效目标。如果未检测到 Exchange，那么组回写将无法启用。
- “组回写”功能当前无法处理安全组或通讯组。

可在[此处](http://blogs.office.com/2014/09/25/delivering-first-chapter-groups-office-365/)查找详细信息。

## 设备回写（预览）

> [AZURE.WARNING]如果当前已激活 DirSync 或 Azure AD Sync，则请勿激活 Azure AD Connect 中任何写回功能。

“设备回写”功能将允许获取在云中注册的设备（如 Intune），并使其处于 AD DS 中以便条件性访问。若要启用该功能，则必须准备 AD DS。如果安装了 AD FS 和设备注册服务 (DRS)，DRS 将提供 PowerShell cmdlet 让你准备用于设备写回的 AD。如果未安装 DRS，则可以作为企业管理员运行 C:\\Program Files\\Windows Azure Active Directory Connect\\AdPrep\\AdSyncAdPrep.psm1。

在运行 PowerShell cmdlet 之前首先必须导入它。

	Import-Module 'C:\Program Files\Windows Azure Active Directory Connect\AdPrep\AdSyncPrep.psm1'

为此，你将需要在本地安装 Active Directory 和 MSOnline PowerShell。



## 暂存模式
在暂存模式下，可以同时设置新的同步服务器与现有的服务器。系统仅支持让一台同步服务器与云中的一个目录连接。但如果想要从另一台服务器（例如正在运行 DirSync 的服务器）中移动，则可以启用暂存模式下 Azure AD Connect。启用时，同步引擎将像平时一样导入并同步数据，但不会向 Azure AD 导出任何内容并将关闭密码同步和密码写回。

![同步筛选](./media/active-directory-aadconnect-get-started-custom/stagingmode.png)


在暂存模式下，可以对同步引擎进行所需的更改，并复查要导出的内容。如果配置看起来正常，请再次运行安装向导，并禁用暂存模式。这样即可将数据导出到 Azure AD。请确保同时禁用其他服务器，这样只有一台服务器在主动导出。

### 防止意外删除
安装 Azure AD Connect 时，会按默认启用防止意外删除的功能，并将其配置为不允许超过 500 个删除项目的导出。500 条为默认值，可以对此进行更改。启用此功能时，如果存在太多的删除项目，将不会继续导出并将收到一封内容如下的电子邮件：

![同步筛选](./media/active-directory-aadconnect-get-started-custom/email.png)


如果这是异常情况，则对此进行调查并采取任何纠正操作。

若要暂时禁用此保护并删除这些项目，请运行：Disable-ADSyncExportDeletionThreshold

若要重新启用保护或者更改默认阈值设置，请运行：Enable-ADSyncExportDeletionThreshold


## 配置使用 AD FS 进行联合身份验证
利用 Azure AD Connect 配置 AD FS 的步骤很简单，只需单击几下鼠标即可。在安装程序之前需要满足以下条件。

- 已启用远程管理的、用作联合服务器的 Windows Server 2012 R2 服务器
- 已启用远程管理的、用作 Web 应用程序代理服务器的 Windows Server 2012 R2 服务器
- 想要使用的联合身份验证服务名称（例如 adfs.contoso.com）的 SSL 证书

### 创建新的 AD FS 场或使用现有的 AD FS 场
你可以使用现有的 AD FS 场或选择创建新的 AD FS 场。如果选择创建一个新场，将需要提供 SSL 证书。如果 SSL 证书是使用密码进行保护的，系统将提示提供密码。

![AD FS 场](./media/active-directory-aadconnect-get-started-custom/adfs1.png)

 
**注意：**如果选择使用现有的 AD FS 场，将跳过几页并直接转到配置 AD FS 和 Azure AD 之间信任关系的界面。

### 指定 AD FS 服务器

你将在此输入想要在其中安装 AD FS 的特定服务器。你可以基于容量规划需求添加一个或多个服务器。在执行此配置之前，必须将这些服务器联接到 Active Directory 域。我们建议安装一台用于测试部署和试点部署的单个 AD FS 服务器并打开 Azure AD Connect 以部署其他服务器，然后将 AD FS 部署到其他服务器以满足扩展需求。


> [AZURE.NOTE]请确保在执行此配置之前所有的服务器都已联接到 AD 域。

![AD FS 服务器](./media/active-directory-aadconnect-get-started-custom/adfs2.png)


 
### 指定 Web 应用程序代理服务器
你将在此输入希望作为你的 Web 应用程序代理服务器的特定服务器。Web 应用程序代理服务器会部署在外围网络（面向 Extranet），并支持来自 Extranet 的身份验证请求。你可以基于容量规划需求添加一个或多个服务器。我们建议安装一台用于测试部署和试验部署的单个 Web 应用程序代理服务器并打开 Azure AD Connect 以部署其他服务器，然后将 Web 应用程序代理部署到其他服务器。我们通常建议拥有同等数量的代理服务器来满足来自 Intranet 的身份验证。

> [AZURE.NOTE]<li>如果用于安装 Azure AD Connect 的帐户不是 AD FS 服务器上的本地管理员，则系统将提示输入具有足够权限的帐户的凭据。</li><li>请确保在配置此步骤之前，Azure AD Connect 服务器和 Web 应用程序代理之间存在 HTTP/HTTPS 连接。</li><li> 此外，请确保 Web 应用程序服务器和 AD FS 服务器之间存在 HTTP/HTTPS 连接，以允许通过身份验证请求。</li>


![Web 应用](./media/active-directory-aadconnect-get-started-custom/adfs3.png)
 

系统将提示输入凭据，以便 Web 应用程序服务器可以建立与 AD FS 服务器的安全连接。这些凭据需要是 AD FS 服务器上的本地管理员。

![代理](./media/active-directory-aadconnect-get-started-custom/adfs4.png)
 
 
### 指定 AD FS 服务的服务帐户
AD FS 服务需要域服务帐户对用户进行身份验证并在 Active Directory 中查找用户信息。它可以支持两种服务帐户类型：

- **组托管服务帐户** - 这是一种在 Active Directory 域服务中随 Windows Server 2012 一并引入的服务帐户类型。此类型的帐户提供诸如 AD FS 等服务，使用单个帐户且无需定期更新帐户密码。如果将包含 AD FS 服务器的域中已经存在 Windows Server 2012 域控制器，则使用此选项。
- **域用户帐户** - 此类型的帐户将要求提供密码，并在密码更改时定期更新密码。仅在将包含 AD FS 服务器的域中不含有 Windows Server 2012 域控制器时，使用此选项。

如果你以域管理员身份登录，Azure AD Connect 将自动创建组托管服务帐户。
 
![AD FS 服务帐户](./media/active-directory-aadconnect-get-started-custom/adfs5.png)
 

### 选择想要联合的 Azure AD 域
使用此配置来设置 AD FS 与 Azure AD 之间的联合身份验证关系。它会配置 AD FS 以向 Azure AD 颁发安全令牌并配置 Azure AD 以信任来自此特定 AD FS 实例的令牌。此页仅允许你在首次体验中配置单个域。你可以通过再次打开 Azure AD Connect 并执行此任务以在任何时候配置其他域。

 
![Azure AD 域](./media/active-directory-aadconnect-get-started-custom/adfs6.png)
 
 
### 执行其他任务以完成联合身份验证配置
需要完成下面的附加任务来结束联合身份验证配置。

- 设置 AD FS 联合身份验证服务名称（例如 adfs.contoso.com）的 Intranet（你的内部 DNS 服务器）和 Extranet（通过你的域注册机构的公共 DNS）的 DNS 记录。对于 Intranet DNS 记录，请确保使用 A 记录而非 CNAME 记录。这对于从加入域的计算机中正确进行 Windows 身份验证是必需的。
- 如果正在部署多个 AD FS 服务器或 Web 应用程序代理服务器，请确保已配置了负载平衡器以及 AD FS 联合身份验证服务名称（例如 adfs.contoso.com）的 DNS 记录指向负载平衡器。
- 针对用于在 Intranet 中使用 Internet Explorer 的浏览器应用程序的集成 Windows 身份验证，请确保 AD FS 联合身份验证服务名称（例如 adfs.contoso.com）已添加到 IE 的 Intranet 区域。此配置可以通过组策略进行控制，并可部署到所有已加入域的计算机中。 

### 验证你的联合配置

当你单击“验证”按钮时，Azure AD Connect 将为你验证 DNS 设置。

![完成](./media/active-directory-aadconnect-get-started-custom/adfs7.png)
 
 
此外，请执行以下验证步骤：

- 在已加入域的计算机上通过 Intranet 中的 Internet Explorer 验证浏览器登录：连接到 https://myapps.microsoft.com，然后使用你登录的帐户验证登录。
- 通过 Extranet 中的任何设备验证浏览器登录：在家庭计算机或移动设备上连接到 https://myapps.microsoft.com，然后提供你的登录 ID 和密码凭据。
- 验证丰富客户端登录：连接到 https://testconnectivity.microsoft.com，选择“Office 365”选项卡，然后选择“Office 365 单一登录测试”。

### AD FS 服务中的可选配置
你可以自定义 AD FS 登录页的插图和徽标图像，方法是登录 AD FS，然后使用 PSH 进行这项配置。
	
	Set-AdfsWebTheme -TargetName default -Logo @{path="c:\Contoso\logo.png"} –Illustration @{path=”c:\Contoso\illustration.png”}

<!---HONumber=67-->