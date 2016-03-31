<properties 
	pageTitle="将自己的域名添加到 Azure AD | Azure" 
	description="介绍如何将自己的域名添加到 Azure Active Directory，并提供其他相关信息。"
	services="active-directory" 
	documentationCenter="" 
	authors="curtand"
	manager="stevenpo"
	editor=""/>

<tags 
	ms.service="active-directory" 
	ms.date="12/01/2015"
	wacn.date="01/29/2016" />

# 将自己的域名添加到 Azure AD

当你注册云服务时，系统将向你颁发一个采用以下格式的域名：contoso.partner.onmschina.cn。你可以继续使用该初始域名，也可以将自己的自定义域名添加到云服务。本主题介绍如何添加你自己的域名和相关信息。

Office 365 用户可能对下列相关主题感兴趣：

- [DNS 基础知识](https://support.office.com/article/854b6b2b-0255-4089-8019-b765cff70377)
- [查找域注册机构或 DNS 托管提供商](https://support.office.com/article/Find-your-domain-registrar-or-DNS-hosting-provider-b5b633ba-1e56-4a98-8ff5-2acaac63a5c8/)
- [在 Office 365 中处理域名](https://support.office.com/article/Work-with-domain-names-in-Office-365-4f1bd681-337a-4bd3-a7b7-cf77b18d0870/)

## 关于 partner.onmschina.cn 域

你可以将 partner.onmschina.cn 域用于其他服务。例如，你可以将该域用于 Exchange Online 和 Lync Online 以创建通讯组列表和登录帐户，以便用户可以访问 SharePoint Online 和 Web 应用集。

如果你将自己的域名添加到目录，则可以继续使用 partner.onmschina.cn domain 域。

在注册过程中选择要用于云服务的名称（例如 contoso.partner.onmschina.cn）后，将无法更改该名称。

## 如何添加自己的域？

如果你的组织已有自定义域名，作为管理员，你可以将此域添加到 Azure AD 目录，用于你订阅的所有 Microsoft Online Services。将域名添加到 Azure AD 后，即可将你的域名与各项云服务相关联。

你可以使用下列方式之一，将最多 900 个域名添加到 Azure AD 租户：

- Azure 管理门户、Office 365 门户或 Windows Intune 门户。
- 用于 Windows PowerShell 的 Azure Active Directory 模块。有关可以使用哪个 cmdlet 来执行此操作的详细信息，请参阅[管理域](https://msdn.microsoft.com/zh-cn/library/azure/dn919677.aspx)。

你必须已经注册了域名，并拥有域名注册机构所需的登录凭据（例如 Go Daddy 或 Register.com）。

可以将多个域添加到目录。但是，不能将同一个域添加到不同的目录。例如，在将域添加到目录时，你不能创建另一个 Azure AD 目录，然后向其添加同一个域名。

如果你打算对云服务使用单一登录，我们建议你通过运行 Microsoft 部署准备工具来帮助你准备 Active Directory 环境。此工具将检查你的 Active Directory 环境并提供一份报告，其中包含有关你是否已做好单一登录设置准备的信息。如果你尚未做好准备，该报告将列出准备单一登录所需进行的更改。例如，该工具将检查用户是否具有 UPN，以及这些 UPN 是否采用了正确的格式。若要下载该工具，请参阅 [Microsoft 部署准备工具](http://community.office365.com/zh-cn/f/183/t/2285.aspx#8155)。

> [AZURE.NOTE]是否使用 Office 365？ 设置域后，你可以开始创建使用自定义域名的电子邮件地址、Lync Online 帐户和分发组。还可以将你的域名用于 SharePoint Online 上托管的面向公众的 Web 应用。

- [使用 Azure 管理门户添加和验证域](#add-and-verify-a-domain-using-the-azure-management-portal)
- [编辑云服务的 DNS 记录](#edit-dns-records-for-your-cloud-services)
- [在任一域名注册机构验证域](#verify-a-domain-at-any-domain-name-registrar)

### <a name="add-and-verify-a-domain-using-the-azure-management-portal" ></a> 使用 Azure 管理门户添加和验证域

1. 在管理门户中，单击“Active Directory”，然后单击你所在组织的目录的名称。你可以执行以下操作之一：
    1. 在默认目录页上，单击“改进用户登录体验”部分中的“添加域”。
    2. 单击“域”选项卡，然后单击“添加客户域”或“添加”按钮。
2. 在“添加域”页上，键入你要添加的域名，然后执行以下操作之一：
    1. 如果你不打算将本地 Active Directory 与 Azure AD 集成，请执行以下操作：
        1. 保留“我计划配置此域为使用本地 Active Directory 进行单点登录”复选框的未选中状态，然后单击“添加”按钮。
        2. 在看到已成功向 Azure AD 添加域的消息后，单击箭头移到下一页，以便验证你的域。
        3. 遵照下一页上的指示验证你在前面步骤中添加的域名是否属于你。有关分步指导，请参阅“在任一域名注册机构验证域”。
    2. 如果你要将本地 Active Directory 与 Azure AD 集成，请执行以下操作：
        1. 确保选中“我计划配置此域为使用本地 Active Directory 进行单点登录”复选框，然后单击“添加”按钮。
        2. 在看到已成功向 Azure AD 添加域的消息后，单击箭头移到下一页，然后遵照该页上的指示配置你为单一登录添加的域。

> [AZURE.NOTE]
> 将域名添加到 Azure AD 并验证此域后，可以更改新电子邮件地址的默认域名。有关详细信息，请参阅[如何更改用户的主域名？](#how-can-i-change-the-primary-domain-name-for-users?) 还可以编辑现有用户帐户的配置文件，以将电子邮件地址（同时也是你的用户 ID）更新为使用自定义域名而不是 partner.onmschina.cn 域。

### <a name="edit-dns-records-for-your-cloud-services" ></a>编辑云服务的 DNS 记录

> [AZURE.NOTE]是否使用 Microsoft Intune？ 不需要编辑 Windows Intune 云服务的 DNS 记录。

添加并验证自定义域名后，下一步就是编辑你的域注册机构或 DNS 托管提供商保留的用于将流量指向云服务的 DNS 记录。Azure AD 提供了所需的 DNS 信息。

如果你刚刚完成了“添加域”向导，请单击“配置 DNS 记录”。否则，请执行以下步骤。

1. 在门户上的左窗格中单击“域”。
2. 根据你正在使用的门户，选择相应选项：单击你要设置的域名，然后单击“DNS 设置”或“查看 DNS 设置”。“DNS 设置”页列出了云服务的 DNS 记录。

    如果你要配置的服务未显示在“DNS 设置”选项卡中，请检查你所做的域服务选择，以确保选择的服务与此域名相关。若要更改设置，例如，若要添加 Lync Online，请参阅“指定要与你的域结合使用的服务”。

3. 在域名注册机构 Web 应用上，将所需的记录添加到 DNS 文件中。

通常，所做的更改需要在大约 15 分钟后才能生效。但是，最长可能需要花费 72 小时才能将你创建的 DNS 记录传播到整个 DNS 系统中。如果需要重新查看这些记录设置，请在“域”页上单击该域，然后在“域属性”页上单击“DNS 设置”选项卡。

若要查看域的状态，请在“域”页上单击该域，然后在“域属性”页上单击“域故障排除”。

###<a name="verify-a-domain-at-any-domain-name-registrar"></a> 在任一域名注册机构验证域

如果你的域已在某个域名注册机构注册，并且你想要将它配置为用于 Azure AD，则需要进行域验证，以确认你拥有该域。若要验证你的域，请在域名注册机构或 DNS 托管商那里创建一条 DNS 记录，然后，Azure AD 将使用该记录来确认你拥有该域。

在验证域之前，必须先将自定义域添加到 Azure AD。如果已添加自定义域但尚未对其进行验证，该域的状态为“单击以验证域”或“未验证”。

#### 收集域信息 

根据用于管理 Azure AD 租户的门户的要求，你需要收集一些有关域的信息，以便稍后可以创建一条要在验证过程中使用的 DNS 记录。

如果使用 Microsoft Intune 或 Azure 帐户门户：

1. 请在“域”页上的域名列表中，找到你要验证的域。在“状态”列中，单击“单击以验证域”。
2. 在“验证域”页的“查看执行此步骤的说明:”下拉列表中，选择你的 DNS 托管提供商。如果你的提供商未出现在列表中，请选择“常规说明”。
3. 在“选择验证方法:”下拉列表中，选择“添加 TXT 记录(首选方法)”或“添加 MX 记录(备选方法)”。

    如果 DNS 托管提供商允许你创建 TXT 记录，则我们建议你使用 TXT 记录进行验证。为什么？ TXT 记录易于创建，并且即使意外输入错误值，也不会妨碍电子邮件的传送。

4. 从表中复制或记录“目标地址或指向的地址”信息。

如果使用管理门户：

1. 在门户中，依次单击“Active Directory”、目录名称和“域”。 
2. 在“域”页上的域名列表中，单击你要验证的域，然后单击“验证”。
2. 在“验证”页上的“记录类型”下拉列表中，选择“TXT 记录”或“MX 记录”。
3. 复制或记录其中的信息。

#### 在域名注册机构添加 DNS 记录 

Azure AD 使用你在域名注册机构创建的 DNS 记录来确认你拥有该域。按照以下说明为在注册机构注册的域创建 TXT 或 MX 记录类型。

如果你的域注册机构不接受“@”作为主机名，请联系你的域注册机构以了解如何表示“当前区域的父级”。

Office 365 提供了[针对公共域注册机构的具体说明](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)。

对于一般情况，请按照以下步骤添加 TXT 或 MX 记录：

1. 登录到域名注册机构的 Web 应用，然后选择你要验证的域。
2. 在帐户的 DNS 管理区域中，选择用来为域添加 TXT 或 MX 记录的选项。
3. 在域对应的“TXT”或“MX”框中，键入 @
4. 在“完全限定的域名(FQDN)”或“指向”框中，键入或粘贴你在执行前一步骤时记录的“目标地址或指向的地址”。
5. 为 TXT 记录，系统会要求提供 **TTL** 信息，此时请键入 **1** 以将 TTL 设置为一小时。 

    对于 MX 记录，系统会提示指定优先级（或优先顺序），此时请键入一个数字，该数字应大于你为现有 MX 记录指定的数字。这有助于防止新的 MX 记录干扰域的邮件路由。你可以看到以下选项替代了优先级：“低”、、“中”、“高”。在本例中，我们选择“低”。

6. 保存更改，然后从域名注册机构的 Web 应用中注销。

创建 TXT 记录或 MX 记录并从 Web 应用注销后，请返回到云服务以验证域。通常，所做的更改需要在大约 15 分钟后才能生效。但是，最长可能需要花费 72 小时才能将你创建的记录传播到整个 DNS 系统中。

#### 验证域 

成功地将你为域创建的记录传播到整个 DNS 系统后，请执行以下操作，完成在 Azure AD 中对域的验证。

1. 在门户中单击“域”。
2. 在“域”列表中，找到你要验证的域，然后根据所用的门户，单击“单击以验证域”或“验证”。
3. 遵照提供的说明完成验证过程。
    - 如果域验证成功，系统会通知你已将该域添加到你的帐户。
    - 如果域验证失败，可能需要更多的时间来传播你在域注册机构所做的更改。请取消验证过程，稍后再返回以重试验证。

如果对域做出更改的时间超过了 72 个小时，请登录域注册机构 Web 应用并验证输入的别名信息是否正确。如果输入的信息不正确，则必须删除不正确的 DNS 记录，然后根据本主题中的过程使用正确的信息创建一条新记录。

在验证域后，可将你的域配置为用于你的帐户。

## <a name="how-can-i-change-the-primary-domain-name-for-users?"></a>如何更改用户的主域名？

将域名添加到 Azure AD 后，可以更改当你创建新用户帐户时应显示为默认域名的域名。为此，请执行以下步骤：

1. 在门户页的左上角，单击你的组织名称。
2. 单击“编辑”。
3. 选择一个新的默认域名，例如，你添加的自定义域名。

## 如何删除域？

在删除域名之前，我们建议你阅读以下信息：

- 无法删除当你注册时为目录提供的原始 contoso.partner.onmschina.cn 域名。 
- 对于具有关联子域的任何顶级域，只有在删除这些子域后，才能删除这些顶级域。例如，如果 corp.adatum.com 或其他子域使用了顶级域名 adatum.com，则你无法删除该域名。有关详细信息，请参阅此[支持文章](https://support.microsoft.com/zh-cn/kb/2787792/)。
- 是否已激活目录同步？ 如果已激活，则会将类似于下面的域自动添加到你的帐户：contoso.mail.partner.onmschina.cn。此域名不可删除。
- 在删除某个域名之前，必须先从与该域关联的所有用户或电子邮件帐户中删除该域名。你可以删除所有帐户，或者批量编辑用户帐户以更改其域名信息和电子邮件地址。有关详细信息，请参阅[在 Azure AD 中创建或编辑用户](/documentation/articles/active-directory-create-users)。
- 如果你在正用于 SharePoint Online 网站集的域名中托管 SharePoint Online 站点，则必须先删除网站集，然后才能删除该域名。

若要删除域名，请执行以下步骤：

1. 在 Azure 经典门户左窗格中的“Azure AD”中，单击“域”。
2. 在“域”页上，选择要删除的域名，然后单击“删除域”。
3. 在“删除域”页上，单击“是”。

如果此时无法删除域名，则该域名的状态在“域”页上会显示为“等待删除”。如果你看到的仍是这种状态，请重试删除该域名。

## 更改域名后排查问题 

### 我对域进行了更改，但它尚未反映这些更改。

由于更新操作需要遍历整个域名系统 (DNS)，在域注册机构或托管提供商处所做的更改最长可能需要 72 个小时才能完全传播到 Internet 中，随后你才可以开始将域名用于你的服务。

此外，你在域注册机构进行的编辑必须完全正确。如果你回来更正错误，则可能需要在数天后，更新的设置才出现在云服务门户站点上。

需要多长时间？ 这在一定程度上取决于你为要替换或更新的 DNS 记录指定的生存时间 (TTL) 设置。在 TTL 过期之前，已缓存先前数据的 Internet 服务器不会查询权威名称服务器以请求新值。

### 我添加并验证了一个域，然后在域注册机构站点上配置了 DNS 记录。为何新的电子邮件帐户还不能接收邮件？ 

在完成为域添加或更新 DNS 记录的操作后，所做的更改最长可能要在 72 小时后生效。

此外，域注册机构站点上的设置信息必须完全正确。请复查你的设置，并确保留出足够的时间，让更改后的 DNS 记录传播到整个系统中。

### 我无法验证我的域名。如何找出问题的原因？ 

查明问题的方法之一是使用域故障排除向导。若要启动该向导，请执行以下操作：在 Azure 管理门户的“管理员”页上，单击“域”，然后双击要验证的域名。接下来，在“故障排除”下，单击“域故障排除”。

故障排除向导将要求你提供信息，说明自己处于验证过程中的哪个阶段，然后将向你提供相关信息以帮助你完成验证。

### 我添加并验证了域，但新域名无法用于现有用户的电子邮件地址。 

如果在添加用户帐户之后将自定义域名添加到云服务，则可能必须进行更新才能使用新域名。例如，你需要编辑用户的帐户以将其电子邮件地址设置为使用你的自定义域。

## 后续步骤

- [Azure AD 论坛](https://social.msdn.microsoft.com/Forums/home?forum=WindowsAzureAD)
- [堆栈溢出](http://stackoverflow.com/questions/tagged/azure)
- [以组织身份注册 Azure](/documentation/articles/sign-up-organization)
- [在 Azure AD 中管理域](https://msdn.microsoft.com/zh-cn/library/azure/dn919677.aspx)

<!---HONumber=Mooncake_0118_2016-->