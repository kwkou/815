<properties
    pageTitle="Azure AD Connect：当你已拥有 Azure AD 时 | Azure"
    description="本主题介绍当存在现有的 Azure AD 租户时如何使用 Connect。"
    services="active-directory"
    documentationcenter=""
    author="andkjell"
    manager="femila"
    editor="" />
<tags
    ms.assetid=""
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/22/2017"
    wacn.date="04/05/2017"
    ms.author="billmath" />  


# 存在现有的租户时如何使用 Azure AD Connect
有关如何使用 Azure AD Connect 的大多数主题假设一开始使用的是新 Azure AD 租户，其中不包含任何用户或其他对象。但是，如果你一开始使用的 Azure AD 租户中填充了用户和其他对象，现在想要使用 Connect，那么，本主题适合你阅读。

## 基础知识
Azure AD 中的对象在云中 (Azure AD) 或本地掌控。对于单个对象而言，无法在本地管理一些属性，在 Azure AD 中管理另一些属性。每个对象都有一个标志，指示对象的管理位置。

可以在本地管理一些用户，在云中管理另一些用户。下面是此配置的常见应用情景：某家组织既有会计工作人员，也有销售工作人员。会计工作人员有本地 AD 帐户，但销售工作人员没有，他们在 Azure AD 中有帐户。这样，就需要在本地管理一些用户，在 Azure AD 中管理另一些用户。

如果你最初在 Azure AD 中管理用户，而这些用户同时又在本地 AD 中，后来你想要使用 Connect，那么，就需要考虑到其他一些因素。

## 与 Azure AD 中的现有用户同步
安装 Azure AD Connect 并开始同步时，Azure AD 同步服务（在 Azure AD 中）将针对每个新对象执行检查，尝试查找匹配的现有对象。此过程使用三个属性：**userPrincipalName**、**proxyAddresses** 和 **sourceAnchor**/**immutableID**。根据 **userPrincipalName** 和 **proxyAddresses** 执行的匹配称为**软匹配**。根据 **sourceAnchor** 执行的匹配称为**硬匹配**。对于 **proxyAddresses** 属性，只会将包含 **SMTP:**（即主要电子邮件地址）的值用于评估。

只会针对来自 Connect 的新对象评估匹配。如果更改现有对象，使它与其中的任一属性匹配，则看到的是错误。

如果 Azure AD 发现某个对象的属性值与来自 Connect 的某个对象的属性值相同，并且前一个对象已在 Azure AD 中存在，则 Azure AD 中的对象将被 Connect 取代。以前，云管理的对象已标记为在本地管理。Azure AD 中的所有属性如果在本地 AD 中具有值，这些属性将被本地值覆盖，但在本地具有 **NULL** 值的属性除外。在这种情况下，Azure AD 中的值将会保留，但是，仍然只能在本地将它更改为其他值。

> [AZURE.WARNING]
由于 Azure AD 中的所有属性将被本地值覆盖，因此请确保本地的数据正确。例如，如果只是在 Office 365 中管理电子邮件地址，而没有在本地 AD DS 中将它保持更新，则会丢失 Azure AD/Office 365 中的、在 AD DS 中不存在的所有值。

> [AZURE.IMPORTANT]
如果使用密码同步（快速设置始终会使用它），则 Azure AD 中的密码将被本地 AD 中的密码覆盖。如果你的用户经常管理不同的密码，则在安装 Connect 后，你需要告知他们使用本地密码。

在规划过程中，必须考虑上一部分的内容和警告。如果在 Azure AD 中做了大量更改，但这些更改未反映在本地 AD DS 中，则在使用 Azure AD Connect 同步对象之前，需要规划好如何在 AD DS 中填充更新的值。

如果使用软匹配匹配了对象，则 **sourceAnchor** 已添加到 Azure AD 中的对象，因此以后可以使用硬匹配。

### 硬匹配与软匹配
对于全新的 Connect 安装，软匹配与硬匹配之间没有实质的差别。主要差别在于灾难恢复情形。如果解除了装有 Azure AD Connect 的服务器，可以重新安装一个新实例，而不会丢失任何数据。在初始安装期间，会向 Connect 发送一个包含 sourceAnchor 的对象。然后，客户端 (Azure AD Connect) 便可以评估匹配，与在 Azure AD 中执行相同的操作相比，速度要快得多。硬匹配同时由 Connect 和 Azure AD 评估。软匹配只由 Azure AD 评估。

### 除用户以外的其他对象
用户通常具有 userPrincipalName 和 proxyAddresses，因此匹配很轻松。但其他对象（例如安全组）没有这些属性。在这种情况下，只能使用 sourceAnchor 执行硬匹配。sourceAnchor 始终是本地经过 Base64 转换的 **objectGUID**，因此，在需要两个对象匹配时，必须更新 Azure AD 中的值。只能使用 PowerShell 更新 sourceAnchor/immutableID，而不能通过门户更新。

## 基于 Azure AD 中的数据创建新的本地 Active Directory
某些客户最初在 Azure AD 中使用仅限云的解决方案，而没有构建本地 AD。后来，他们想要使用本地资源，并希望基于 Azure AD 数据构建本地 AD。对于这种情况，Azure AD Connect 无法起到作用。它不会创建本地用户，并且没有能力将本地密码设置为与 Azure AD 中的密码相同。

## 后续步骤
了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->