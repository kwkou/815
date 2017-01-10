<properties
   pageTitle="Azure AD Connect：设计概念 | Azure"
   description="本主题详细说明某些实现设计方面的问题"
   services="active-directory"
   documentationCenter=""
   authors="AndKjell"
   manager="femila"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="Identity"
   ms.date="09/13/2016"
   ms.author="andkjell"
   wacn.date="01/09/2017"/>

# Azure AD Connect：设计概念
本主题旨在说明 Azure AD Connect 实现设计期间必须考虑到的各个方面。本主题是特定领域的深入探讨，其他主题中也简要描述了这些概念。

## sourceAnchor <a name="sourceanchor"></a>
sourceAnchor 属性定义为*在对象生存期内不会变化的属性*。它可将对象唯一标识为本地和 Azure AD 中的相同对象。该属性也称为 **immutableId**，这两个名称可以换用。

在本主题中，“不可变”（即无法更改）一词非常重要。由于此属性的值在设置之后就无法更改，因此请务必挑选可支持方案的设计。

该属性用于以下方案︰

- 构建新的同步引擎服务器，或者在灾难恢复方案后进行重建时，此属性会将 Azure AD 中的现有对象链接到本地对象。
- 如果从仅限云的标识转移到已同步的标识模型，则此属性可让对象将 Azure AD 中的现有对象与本地对象进行“硬匹配”。
- 如果使用联合，此属性将与 **userPrincipalName** 一起在声明中使用，以唯一标识用户。

本主题只讨论与用户相关的 sourceAnchor。相同的规则适用于所有对象类型，但只有用户才需要考虑这个问题。

### 选择良好的 sourceAnchor 属性
属性值必须遵循以下规则：

- 长度小于 60 个字符
    - 系统将 a-z、A-Z 或 0-9 以外的字符编码并计为 3 个字符
- 不包含特殊字符：&#92; ! # $ % & * + / = ? ^ &#96; { }| ~ < > ( ) ' ; : , [ ] " @ _
- 必须全局唯一
- 必须是字符串、整数或二进制数
- 不应基于用户的名称
- 不应区分大小写，避免使用可能因大小写而改变的值
- 应在创建对象时分配

如果选定的 sourceAnchor 不是字符串类型，Azure AD Connect 会将此属性值进行 Base64Encode 处理，确保不会出现特殊字符。如果使用除 ADFS 以外的其他联合服务器，请确保服务器也能将此属性进行 Base64Encode 处理。

sourceAnchor 属性区分大小写。“JohnDoe”与“johndoe”是不同的值。但是，两个对象的不同之处不能只是大小写不同。

如果有单个本地林，应使用属性 **objectGUID**。这也是在 Azure AD Connect 中使用快速设置时所用的属性，而且也是 DirSync 所用的属性。

如果有多个林，并且不在林和域之间移动用户，则 **objectGUID** 是适当的属性（即使在本例中）。

如果要在林和域之间移动用户，必须查找不会更改的属性或者在移动时可随用户移动的属性。建议的方法是引入合成属性。可以保存 GUID 等信息的属性也可能适用。在对象创建期间，将创建新的 GUID 创建并对用户加上戳记。可以在同步引擎服务器中创建自定义同步规则，根据 **objectGUID** 创建此值，然后在 ADDS 中更新选择的属性。当移动对象时，请务必同时复制此值的内容。

另一个解决方案是选择已知不会更改的现有属性。常用的属性包括 **employeeID**。如果打算使用包含字母的属性，请确保属性值的大小写（大写与小写）不会更改。例如，包含用户姓名的属性就是不应使用的不当属性。因为在结婚或离婚时，此姓名很可能会更改，所以不适用于此属性。这也是无法在 Azure AD Connect 安装向导中选择 **userPrincipalName**、**mail** 和 **targetAddress** 等属性的原因之一。这些属性还包含 @ 字符，sourceAnchor 中不允许此字符。

### 更改 sourceAnchor 属性
在 Azure AD 中创建对象并同步标识之后，无法更改 sourceAnchor 属性值。

出于此原因，Azure AD Connect 实施以下限制：

- 只能在初始安装期间设置 sourceAnchor 属性。如果重新运行安装向导，此选项是只读的。如果需要更改此设置，必须卸载然后重新安装。
- 如果你要安装其他 Azure AD Connect 服务器，则必须选择以前所用的同一 sourceAnchor 属性。如果以前使用 DirSync，现在想要迁移到 Azure AD Connect，则必须使用 **objectGUID**，因为这是 DirSync 所用的属性。
- 如果 sourceAnchor 值在对象导出到 Azure AD 之后发生更改，Azure AD Connect Sync 将引发错误，并且不允许在更正问题且在源目录中改回 sourceAnchor 之前，对此对象进行任何其他更改。

## Azure AD 登录
将本地目录与 Azure AD 集成时，请务必了解同步设置对用户身份验证的方式有何影响。Azure AD 使用 userPrincipalName (UPN) 对用户进行身份验证。但是，在同步用户时，必须小心选择要用于 userPrincipalName 值的属性。

### 选择 userPrincipalName 的属性
选择属性以便提供用于 Azure 的 UPN 值时，应确保

- 属性值符合 UPN 语法 (RFC 822)，其格式应为 username@domain
- 这些值的后缀符合 Azure AD 中其中一个已验证的自定义域

在快速设置中，属性的假设选择是 userPrincipalName。如果 userPrincipalName 属性不包含希望用户用于登录 Azure 的值，则必须选择“自定义安装”。

### 自定义域状态和 UPN
必须确保 UPN 后缀包含已验证的域。

John 是 contoso.com 中的用户。在将用户同步到 Azure AD 目录 contoso.partner.onmschina.cn 之后，希望 John 使用本地 UPN john@contoso.com 登录到 Azure。为此，需要将 contoso.com 添加为 Azure AD 中的自定义域并进行验证，才能开始同步用户。如果 John 的 UPN 后缀（例如 contoso.com）与 Azure AD 中已验证的域不匹配，Azure AD 会将该 UPN 后缀替换为 contoso.partner.onmschina.cn。

### 不可路由的本地域与 Azure AD 的 UPN
有些组织使用不可路由的域（例如 contoso.local）或简单的单标签域（例如 contoso）。在 Azure AD 中，无法验证不可路由的域。Azure AD Connect 只能同步到 Azure AD 中已验证的域。创建 Azure AD 目录时，将创建可路由的域，该域将成为 Azure AD 的默认域，例如 contoso.partner.onmschina.cn。因此，如果不想要同步到默认的 partner.onmschina.cn 域，必须在此类方案中验证任何其他可路由的域。

有关添加和验证域的详细信息，请阅读 [Add your custom domain name to Azure Active Directory](/documentation/articles/active-directory-add-domain/)（将自定义域名添加到 Azure Active Directory）。

Azure AD Connect 将检测你是否在不可路由的域环境中运行，并在适当的情况下警告你不要继续使用快速设置。如果在不可路由的域中操作，用户的 UPN 可能也包含不可路由的后缀。例如，如果在 contoso.local 下运行，Azure AD Connect 建议使用自定义设置而不是快速设置。使用自定义设置，可以在用户同步到 Azure AD 之后，指定要用作 UPN 以供登录 Azure 的属性。

## 后续步骤
了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_Quality_Review_0104_2017-->
