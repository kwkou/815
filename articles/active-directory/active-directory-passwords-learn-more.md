<properties 
	pageTitle="详细了解：Azure AD 密码管理 | Azure" 
	description="有关 Azure AD 密码管理的高级主题，内容包括密码写回的工作原理、密码写回安全性、密码重置门户的工作原理，以及密码重置使用的数据。" 
	services="active-directory" 
	documentationCenter="" 
	authors="asteen" 
	manager="kbrint" 
	editor="billmath"/>

<tags 
	ms.service="active-directory"  
	ms.date="02/16/2016" 
	wacn.date="06/27/2016"/>

# 了解有关密码管理的详细信息
如果你已部署密码管理，或只是想要在部署之前深入了解密码管理工作原理的技术细节，则本部分可让你大致了解该服务背后的技术概念。本部分将介绍以下内容：

* [**密码写回概述**](#password-writeback-overview)
  - [密码写回的工作原理](#how-password-writeback-works)
  - [密码写回支持的方案](#scenarios-supported-for-password-writeback)
  - [密码写回安全模型](#password-writeback-security-model)
* [**密码重置门户的工作原理**](#how-does-the-password-reset-portal-work)
  - [密码重置使用哪些数据？](#what-data-is-used-by-password-reset)
  - [如何访问用户的密码重置数据](#how-to-access-password-reset-data-for-your-users)

## <a name="password-writeback-overview"></a>密码写回概述
密码写回是一个 [Azure Active Directory Connect](/documentation/articles/active-directory-aadconnect/) 组件，Azure Active Directory Premium 的当前订户可以启用和使用该组件。有关详细信息，请参阅 [Azure Active Directory 版本](/documentation/articles/active-directory-editions/)。

使用此功能，你可将云租户配置为将密码写回本地 Active Directory。有了此功能，你无需再设置和管理复杂的本地自助服务密码重置解决方案。此功能提供了一种基于云的便利方法，让你的用户能够在任何位置重置本地密码。有关密码写回的一些主要功能，请继续阅读以下内容：

- **零延迟反馈。** 密码写回是一项同步操作。如果用户的密码不符合策略或因某种原因而无法重置或更改，用户会立即收到通知。
- **支持使用 AD FS 或其他联合技术的用户重置密码。** 使用密码写回功能，只要联合用户帐户同步到你的 Azure AD 租户，用户就能够从云中管理他们的本地 AD 密码。
- **支持使用密码哈希同步的用户重置密码。** 当密码重置服务检测到同步用户帐户已启用密码哈希同步时，我们会同时重置此帐户的本地密码和云密码。
- **支持从访问面板和 Office 365 更改密码。** 当联合或密码同步用户更改其过期或未过期的密码时，我们将这些密码写回到你的本地 AD 环境。
- **支持管理员从 [**Azure 经典管理门户**](https://manage.windowsazure.cn)重置密码时写回密码。**无论何时管理员在 [Azure 经典管理门户](https://manage.windowsazure.cn)中重置用户的密码，只要该用户已联合或密码已同步，我们便还会设置管理员在本地 AD 上选择的密码。Office 经典管理门户当前不支持此功能。
- **实施本地 AD 密码策略。** 当用户重置其密码时，在将重置提交到该目录之前，我们要确保它符合你的本地 AD 策略。这包括历史记录、复杂性、年龄、密码筛选器以及你在本地 AD 中定义的所有其他密码限制。
- **不需要任何入站防火墙规则。** 密码写回功能使用 Azure 服务总线中继作为底层通信通道，这意味着你无需打开防火墙上的任何入站端口即可使用此功能（仅限 443 出站）。
- **本地 Active Directory 的受保护组中存在的用户帐户不支持。** 有关受保护组的详细信息，请参阅 [Active Directory 中的受保护帐户和组](https://technet.microsoft.com/zh-cn/library/dn535499.aspx)。

### <a name="how-password-writeback-works"></a>密码写回的工作原理
密码写回有三个主要组件：

- 密码重置云服务（也集成到 Azure AD 的密码更改页面中）
- 租户特定的 Azure 服务总线中继
- 本地密码重置终结点

这三个组件按下图所示配合工作：

  ![][001]

当联合或密码哈希同步用户在云中重置或更改其密码时，将发生以下情况：

1.	我们检查用户具有何种类型的密码。如果我们看到密码在本地管理，则会确保写回服务启动且正在运行。如果是，我们让用户继续操作；如果不是，我们告知用户不能在此处重置其密码。
2.	接下来，用户通过适当的身份验证界面，到达重置密码屏幕。
3.	用户选择一个新密码并进行确认。
4.	单击提交时，我们使用写回设置过程中创建的公钥来加密纯文本密码。
5.	加密密码后，我们将其包括在一个负载中，该负载通过 HTTPS 通道发送到租户特定的服务总线中继（我们在写回设置过程中也为你设置了此项）。此中继受随机生成的密码保护，只有你的本地安装知道该密码。
6.	一旦消息到达服务总线，密码重置终结点便自动唤醒并看到有重置请求挂起。
7.	然后，该服务使用云定位点属性查找相关用户。要使查找成功，该用户对象必须存在于 AD 连接器空间中、必须链接到相应的 MV 对象，而且必须链接到相应的 AAD 连接器对象。最后，为使同步查找此用户帐户，AD 连接器对象到 MV 的链接必须具有同步规则 `Microsoft.InfromADUserAccountEnabled.xxx`。如此要求的原因是：从云中进行调用时，同步引擎使用 cloudAnchor 属性查找 AAD 连接器空间对象，然后遵循该链接依次回到 MV 对象和 AD 对象。由于同一用户可能有多个 AD 对象（多林），因此，同步引擎将依赖 `Microsoft.InfromADUserAccountEnabled.xxx` 链接选取正确的对象。
8.	一旦找到用户帐户，我们将尝试直接在相应的 AD 林中重置密码。
9.	如果密码设置操作成功，我们将告诉用户其密码已被修改，他们可以继续操作。
10.	如果密码设置操作失败，我们会将错误返回给用户，让他们再试一次。操作失败的可能原因是服务已关闭、用户选择的密码不符合组织策略、我们在本地 AD 中找不到该用户等。我们对于许多这类情况都有一个特定消息，并告知用户他们可以执行哪些操作来解决问题。

### <a name="scenarios-supported-for-password-writeback"></a>密码写回支持的方案
下表介绍同步功能的各个版本支持哪些方案。通常情况下，如果要使用密码写回，我们强烈建议安装最新版本的 [Azure AD Connect](/documentation/articles/active-directory-aadconnect/#download-azure-ad-connect)。

  ![][002]

### <a name="password-writeback-security-model"></a>密码写回安全模型
密码写回是高度安全、极其可靠的服务。为确保你的信息受保护，我们启用了一个 4 层安全模型，如下所述。

- **租户特定的服务总线中继** – 如果你设置该服务，我们将建立租户特定的总线服务中继，该中继受随机生成的强密码保护，而 Microsoft 绝不会接触此密码。
- **锁定的加密强密码加密密钥** – 创建服务总线中继后，我们创建一个强大的非对称密钥对，用于在经过线路时加密密码。此密钥仅驻留在公司在云中的密钥存储内，就像目录中的任何密码一样，将被牢牢锁住并接受审核。
- **行业标准 TLS** – 云中发生密码重置或更改操作时，我们将采用纯文本密码并用公钥对其进行加密。然后，我们将其置于 HTTPS 消息中，该消息使用 Microsoft 的 SSL 证书通过加密通道发送到服务总线中继。此消息到达服务总线后，你的本地代理将唤醒、使用先前生成的强密码对服务总线进行身份验证、选取加密消息、使用我们生成的私钥对消息进行解密，然后尝试通过 AD DS SetPassword API 设置密码。通过此步骤，我们可以在云中强制实施你的 AD 本地密码策略（复杂性、年龄、历史记录、筛选器等）。
- **消息过期策略** – 最后，如果由于某种原因而使消息位于服务总线中（因为本地服务关闭），消息则会超时并在几分钟后删除，以便进一步提高安全性。

## <a name="how-does-the-password-reset-portal-work"></a>密码重置门户的工作原理。
当某个用户导航到密码重置门户时，工作流将启动以确定此用户帐户是否有效、此用户所属的组织、此用户密码的管理位置以及用户是否已获许可使用该功能。阅读以下步骤，了解有关密码重置页面背后的逻辑。

1.	用户单击“无法访问你的帐户”链接或直接转到 [https://passwordreset.microsoftonline.com](https://passwordreset.microsoftonline.com)。
2.	用户输入用户 ID 并传递验证码。
3.	Azure AD 通过执行以下操作来验证用户是否能够使用此功能：
    - 检查用户是否启用了此功能并分配有 Azure AD 许可证。
        - 如果用户未启用此功能或未分配有许可证，则要求用户联系其管理员重置其密码。
    - 检查用户是否具有针对其帐户定义且符合管理员策略的正确质询数据。
        - 如果策略仅要求一个质询，则确保用户具有针对由管理员策略启用的至少一个质询定义的适当数据。
          - 如果未配置用户，则建议用户联系其管理员重置其密码。
        - 如果策略要求两个质询，则确保用户具有针对由管理员策略启用的至少两个质询定义的适当数据。
          - 如果未配置用户，则我们建议用户联系其管理员重置其密码。
    - 检查是否在本地管理用户密码（联合或密码哈希同步）。
       - 如果已部署写回且在本地管理用户密码，则允许用户继续进行身份验证并重置其密码。
       - 如果未部署写回且在本地管理用户密码，则要求用户联系其管理员重置其密码。
4.	如果确定用户能够成功重置其密码，则将指导用户完成重置过程。

有关如何部署密码写回的详细信息，请参阅[入门：Azure AD 密码管理](/documentation/articles/active-directory-passwords-getting-started/)。

### <a name="what-data-is-used-by-password-reset"></a>密码重置使用哪些数据？
- 下表概述密码重置期间此数据使用的位置和方式，旨在帮助你决定哪些身份验证选项适合你的组织。此表还显示当你代表用户从未验证此数据的输入路径提供数据时的格式要求。

> [AZURE.NOTE] 注册门户中不会显示办公电话，因为用户当前无法在目录中编辑此属性。

<table>
          <tbody><tr>
            <td>
              <p>
                <strong>联系方式名称</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>Azure Active Directory 数据元素</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>使用/设置位置。</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>格式要求</strong>
              </p>
            </td>
          </tr>
          <tr>
            <td>
              <p>办公电话</p>
            </td>
            <td>
              <p>PhoneNumber</p>
              <p>例如 Set-MsolUser -UserPrincipalName JWarner@contoso.com -PhoneNumber "+1 1234567890x1234"</p>
            </td>
            <td>
              <p>使用位置：</p>
              <p>密码重置门户</p>
              <p>设置位置：</p>
              <p>可从 PowerShell、DirSync、Azure 经典管理门户和 Office 经典管理门户设置 PhoneNumber</p>
            </td>
            <td>
              <p>+ccc xxxyyyzzzz（例如 +1 1234567890）</p>
              <ul>
                <li class="unordered">
										必须提供国家/地区代码<br><br></li>
              </ul>
              <ul>
                <li class="unordered">
										必须提供区号（如果适用）<br><br></li>
              </ul>
              <ul>
                <li class="unordered">
										必须在国家/地区代码前加上 +<br><br></li>
              </ul>
              <ul>
                <li class="unordered">
										必须在国家/地区代码与其余号码之间留一个空格<br><br></li>
              </ul>
              <ul>
                <li class="unordered">
										不支持分机号，如果你已指定分机号，我们将在调度电话呼叫前去掉分机号。<br><br></li>
              </ul>
            </td>
          </tr>
          <tr>
            <td>
              <p>移动电话</p>
            </td>
            <td>
              <p>AuthenticationPhone</p>
              <p>或者</p>
              <p>MobilePhone</p>
              <p>（如果存在数据，则使用身份验证电话，否则将退回到移动电话字段）。</p>
              <p>例如 Set-MsolUser -UserPrincipalName JWarner@contoso.com -MobilePhone "+1 1234567890x1234"</p>
            </td>
            <td>
              <p>使用位置：</p>
              <p>密码重置门户</p>
              <p>注册门户</p>
              <p>设置位置： </p>
              <p>可从密码重置注册门户或 MFA 注册门户设置 AuthenticationPhone。</p>
              <p>可从 PowerShell、DirSync、Azure 经典管理门户和 Office 经典管理门户设置 MobilePhone</p>
            </td>
            <td>
              <p>+ccc xxxyyyzzzz（例如 +1 1234567890）</p>
              <ul>
                <li class="unordered">
										必须提供国家/地区代码。<br><br></li>
              </ul>
              <ul>
                <li class="unordered">
										必须提供区号（如果适用）。<br><br></li>
              </ul>
              <ul>
                <li class="unordered">
										必须在国家/地区代码前加上 +。<br><br></li>
              </ul>
              <ul>
                <li class="unordered">
										必须在国家/地区代码与其余号码之间留一个空格。<br><br></li>
              </ul>
              <ul>
                <li class="unordered">
										不支持分机号，如果你已指定分机号，我们将在调度电话呼叫前去掉分机号。<br><br></li>
              </ul>
            </td>
          </tr>
          <tr>
            <td>
              <p>备用电子邮件</p>
            </td>
            <td>
              <p>AuthenticationEmail</p>
              <p>或者</p>
              <p>AlternateEmailAddresses[0] </p>
              <p>（如果存在数据，则使用身份验证电子邮件，否则将退回到“备用电子邮件”字段）。</p>
              <p>注意：在目录中备用电子邮件字段被指定为字符串数组。我们使用此数组中的第一个条目。</p>
              <p>例如 Set-MsolUser -UserPrincipalName JWarner@contoso.com -AlternateEmailAddresses "email@live.com"</p>
            </td>
            <td>
              <p>使用位置：</p>
              <p>密码重置门户</p>
              <p>注册门户</p>
              <p>设置位置： </p>
              <p>可从密码重置注册门户或 MFA 注册门户设置 AuthenticationEmail。</p>
              <p>可从 PowerShell、Azure 经典管理门户和 Office 经典管理门户设置 AlternateEmail</p>
            </td>
            <td>
              <p>
                <a href="mailto:user@domain.com">user@domain.com</a> 或 甲斐@黒川.日本</p>
              <ul>
                <li class="unordered">
										电子邮件应遵循标准格式。<br><br></li>
              </ul>
              <ul>
                <li class="unordered">
										支持 Unicode 电子邮件。<br><br></li>
              </ul>
            </td>
          </tr>
          <tr>
            <td>
              <p>安全问答</p>
            </td>
            <td>
              <p>不能直接在目录中修改。</p>
            </td>
            <td>
              <p>使用位置：</p>
              <p>密码重置门户</p>
              <p>注册门户 </p>
              <p>设置位置： </p>
              <p>只能通过 Azure 经典管理门户设置安全问题。</p>
              <p>只能通过注册门户设置给定用户的安全问题答案。</p>
            </td>
            <td>
              <p>安全问题最多有 200 个字符，最少有 3 个字符</p>
              <p>答案最多有 40 个字符，最少有 3 个字符</p>
            </td>
          </tr>
        </tbody></table>

### <a name="how-to-access-password-reset-data-for-your-users"></a>如何访问用户的密码重置数据
####可通过同步设置的数据
可以从本地同步以下字段：

* 移动电话
* 办公电话

####可使用 Azure AD PowerShell 设置的数据
可以使用 Azure AD PowerShell 和图形 API 访问以下字段：

* 备用电子邮件
* 移动电话
* 办公电话
* 身份验证电话
* 身份验证电子邮件

####只能使用注册 UI 设置的数据
只能通过 SSPR 注册 UI (https://aka.ms/ssprsetup) 访问以下字段：

* 安全问答

####当用户注册时会发生什么情况？
当用户注册时，注册页面**始终**设置以下字段：

* 身份验证电话
* 身份验证电子邮件
* 安全问答

如果你提供了“移动电话”或“备用电子邮件”的值，用户可以立即使用这些信息重置密码，即使他们尚未注册该服务。此外，用户在首次注册时将看到这些值，并可随意进行修改。但是，成功注册之后，这些值将分别保存在“身份验证电话”和“身份验证电子邮件”字段中。

这可能是解除阻止大量用户使用 SSPR，同时允许他们在整个注册过程中验证此信息的有效方式。

####使用 PowerShell 设置密码重置数据
可以使用 Azure AD PowerShell 设置以下字段的值。

* 备用电子邮件
* 移动电话
* 办公电话

若要开始，首先需要[下载并安装 Azure AD PowerShell 模块](https://msdn.microsoft.com/library/azure/jj151815.aspx#bkmk_installmodule)。安装后，你可以遵照以下步骤配置每个字段。

#####备用电子邮件
```
Connect-MsolService
Set-MsolUser -UserPrincipalName user@domain.com -AlternateEmailAddresses @("email@domain.com")
```

#####移动电话
```
Connect-MsolService
Set-MsolUser -UserPrincipalName user@domain.com -MobilePhone "+1 1234567890"
```

#####办公电话
```
Connect-MsolService
Set-MsolUser -UserPrincipalName user@domain.com -PhoneNumber "+1 1234567890"
```

####使用 PowerShell 读取密码重置数据
可以使用 Azure AD PowerShell 读取以下字段的值。

* 备用电子邮件
* 移动电话
* 办公电话
* 身份验证电话
* 身份验证电子邮件

若要开始，首先需要[下载并安装 Azure AD PowerShell 模块](https://msdn.microsoft.com/library/azure/jj151815.aspx#bkmk_installmodule)。安装后，你可以遵照以下步骤配置每个字段。

#####备用电子邮件
```
Connect-MsolService
Get-MsolUser -UserPrincipalName user@domain.com | select AlternateEmailAddresses
```

#####移动电话
```
Connect-MsolService
Get-MsolUser -UserPrincipalName user@domain.com | select MobilePhone
```

#####办公电话
```
Connect-MsolService
Get-MsolUser -UserPrincipalName user@domain.com | select PhoneNumber
```

#####身份验证电话
```
Connect-MsolService
Get-MsolUser -UserPrincipalName user@domain.com | select -Expand StrongAuthenticationUserDetails | select PhoneNumber
```

#####身份验证电子邮件
```
Connect-MsolService
Get-MsolUser -UserPrincipalName user@domain.com | select -Expand StrongAuthenticationUserDetails | select Email
```

## 密码重置文档的链接
以下是所有 Azure AD 密码重置文档页面的链接：

* [**重置自己的密码**](/documentation/articles/active-directory-passwords-update-your-own-password/) - 了解如何以系统用户的身份重置或更改自己的密码
* [**工作原理**](/documentation/articles/active-directory-passwords-how-it-works/) - 了解六个不同的服务组件及其功能
* [**入门**](/documentation/articles/active-directory-passwords-getting-started/) - 了解如何让用户重置及更改云密码或本地密码
* [**自定义**](/documentation/articles/active-directory-passwords-customize/) - 了解如何根据组织的需求自定义服务的外观和行为
* [**最佳实践**](/documentation/articles/active-directory-passwords-best-practices/) - 了解如何快速部署且有效管理组织的密码
* [**深入分析**](/documentation/articles/active-directory-passwords-get-insights/) - 了解集成式报告功能
* [**常见问题**](/documentation/articles/active-directory-passwords-faq/) - 获取常见问题的解答
* [**故障排除**](/documentation/articles/active-directory-passwords-troubleshoot/) - 了解如何快速排查服务的问题



[001]: ./media/active-directory-passwords-learn-more/001.jpg "Image_001.jpg"
[002]: ./media/active-directory-passwords-learn-more/002.jpg "Image_002.jpg"
 
<!---HONumber=Mooncake_0620_2016-->