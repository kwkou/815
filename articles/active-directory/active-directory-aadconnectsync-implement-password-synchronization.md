<properties
    pageTitle="使用 Azure AD Connect 同步实现密码同步 | Azure"
    description="介绍密码同步的原理及设置方法。"
    services="active-directory"
    documentationcenter=""
    author="MarkusVi"
    manager="femila"
    editor="" />
<tags
    ms.assetid="05f16c3e-9d23-45dc-afca-3d0fa9dbf501"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/21/2017"
    wacn.date="05/22/2017"
    ms.author="markvi"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="20ae8f99ef859bbd0ebd9cc934569993670a290d"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="implement-password-synchronization-with-azure-ad-connect-sync"></a>使用 Azure AD Connect 同步实现密码同步
本文提供将用户密码从本地 Active Directory 实例同步到基于云的 Azure Active Directory (Azure AD) 实例时所需的信息。

## <a name="what-is-password-synchronization"></a>什么是密码同步
因为忘记密码而无法完成工作的机率与需要记住的不同密码数目有关。 要记住的密码越多，忘记密码的机率就越高。 就密码重置和其他密码相关问题的疑问和来电占据了最多的技术服务资源。

密码同步是用于将用户密码从本地 Active Directory 实例同步到基于云的 Azure AD 实例的功能。
使用此功能可登录到各种 Azure AD 服务，例如 Office 365、Microsoft Intune、CRM Online 和 Azure Active Directory 域服务 (Azure AD DS)。 登录到该服务时使用的密码与登录到本地 Active Directory 实例时使用的密码相同。

![什么是 Azure AD Connect](./media/active-directory-aadconnectsync-implement-password-synchronization/arch1.png)

减少密码数目以后，用户只需维护一个密码。 密码同步有助于：

- 提升用户的生产力。
- 减少技术支持成本  

此外，如果你决定使用 Active Directory 联合身份验证服务 (AD FS) 进行联合身份验证，可以选择性地设置密码同步，作为在 AD FS 基础结构发生故障时的备用身份验证方式。

密码同步是由 Azure AD Connect Sync 实现的目录同步功能的扩展。 若要在环境中使用密码同步，需要：

- 安装 Azure AD Connect。  
- 配置本地 Azure Active Directory 实例与 Azure Active Directory 实例之间的目录同步。
- 启用密码同步。

有关详细信息，请参阅[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)。

> [AZURE.NOTE]
> 有关为 FIPS 和密码同步配置的 Azure Active Directory 域服务的更多详细信息，请参阅本文后面的“密码同步和 FIPS”。
>
>

## <a name="how-password-synchronization-works"></a>密码同步的工作原理
Active Directory 域服务以实际用户密码的哈希值表示形式存储密码。 哈希值是单向数学函数（*哈希算法*）的计算结果。 没有任何方法可将单向函数的结果还原为纯文本版本的密码。 无法使用密码哈希来登录本地网络。

为了同步密码，Azure AD Connect 同步将从本地 Active Directory 实例提取密码哈希。 同步到 Azure Active Directory 身份验证服务之前，已对密码哈希应用其他安全处理。 密码将基于每个用户按时间顺序同步。

密码同步过程的实际数据流等同于用户数据（例如 DisplayName 或电子邮件地址）的同步。 但是，密码的同步频率高于其他属性的标准目录同步窗口。 密码同步过程每隔 2 分钟运行一次。 无法修改此过程的运行频率。 同步某个密码时，该密码将覆盖现有的云密码。

首次启用密码同步功能时，它将对范围内的所有用户执行初始密码同步。 无法显式定义一部分要同步的用户密码。

当你更改本地密码时，更新后的密码将会同步，此操作基本上在几分钟内就可完成。
密码同步功能会自动重试失败的同步尝试。 如果尝试同步密码期间出现错误，该错误会被记录在事件查看器中。

同步密码对当前登录的用户没有任何影响。
当前的云服务会话不会立即受到已同步密码更改的影响，而是在你登录云服务时才受到影响。 但是，当云服务要求你再次身份验证时，就需要提供新的密码。

无论用户是否已登录到其公司网络，都必须第二次输入其公司凭据，以便向 Azure AD 进行身份验证。 但是，如果用户在登录时选中了“使我保持登录状态(KMSI)”复选框，则可以最大限度地避开这些模式。 这样选择可设置会话 Cookie 以在短时间内绕过身份验证。 Azure AD 管理员可以启用或禁用 KMSI 行为。

> [AZURE.NOTE]
> 只有 Active Directory 的对象类型用户才支持密码同步。 不支持 iNetOrgPerson 对象类型。

### <a name="detailed-description-of-how-password-synchronization-works"></a>密码同步工作原理的详细说明
下面将深入说明 Active Directory 与 Azure AD 之间的密码同步工作原理。

![详细的密码流程](./media/active-directory-aadconnectsync-implement-password-synchronization/arch3.png)


1. 每隔两分钟，AD Connect 服务器上的密码同步代理都会通过用于同步 DC 之间数据的标准 [MS-DRSR](https://msdn.microsoft.com/zh-cn/library/cc228086.aspx) 复制协议，从 DC 请求存储的密码哈希（unicodePwd 属性）。 服务帐户必须具有“复制目录更改”和“复制所有目录更改”AD 权限（默认情况下，在安装时授予），才能获取密码哈希。
2. 在发送前，DC 将使用密钥（即 RPC 会话密钥的 [MD5](http://www.rfc-editor.org/rfc/rfc1321.txt) 哈希）和 salt 对 MD4 密码哈希进行加密。 然后，它通过 RPC 将结果发送到密码同步代理。 DC 还使用 DC 复制协议将 salt 传递给同步代理，因此该代理将能够解密信封。
3.    密码同步代理获得加密的信封后，将使用 [MD5CryptoServiceProvider](https://msdn.microsoft.com/zh-cn/library/System.Security.Cryptography.MD5CryptoServiceProvider.aspx)和 salt 生成密钥，以便将收到的数据重新解密为其原始的 MD4 格式。 密码同步代理无需有权访问明文密码。 密码同步代理使用 MD5 完全是为了实现与 DC 的复制协议兼容性，并仅在本地 的 DC 和密码同步代理之间使用。
4.    密码同步代理通过先将哈希转换为 32 字节的十六进制字符串，然后使用 UTF-16 编码重新将此字符串转换为二进制，来将 16 字节的二进制密码哈希扩展为 64 字节。
5.    密码同步代理通过将 salt（包含 10 字节长度的 salt）添加到 64 字节的二进制字符串，来进一步保护原始哈希。
6.    然后，密码同步代理将 MD4 哈希与 salt 组合在一起，并将其输入到 [PBKDF2](https://www.ietf.org/rfc/rfc2898.txt) 函数。 使用 [HMAC-SHA256](https://msdn.microsoft.com/zh-cn/library/system.security.cryptography.hmacsha256.aspx) 键控哈希算法的 1000 次迭代。 
7.    密码同步代理获取生成的 32 字节哈希，将 salt 和 SHA256 迭代次数连接到它（以供 Azure AD 使用），然后通过 SSL 将该字符串从 Azure AD Connect 传输到 Azure AD。</br> 
8.    当用户尝试登录到 Azure AD 并输入其密码时，将通过同一 MD4+salt+PBKDF2+HMAC-SHA256 过程运行密码。 如果生成的哈希与 Azure AD 中存储的哈希匹配，则用户输入的密码正确并进行身份验证。 

>[AZURE.NOTE] 
>原始 MD4 哈希不会传送到 Azure AD。 与之相反，传输的是原始 MD4 哈希的 SHA256 哈希。 因此，如果获取了 Azure AD 中存储的哈希，将无法在本地“传递哈希”攻击中使用。

### <a name="how-password-synchronization-works-with-azure-active-directory-domain-services"></a>如何使用 Azure Active Directory 域服务进行密码同步
也可以使用密码同步功能将本地密码同步到 Azure Active Directory 域服务。 在此方案中，Azure Active Directory 域服务实例以本地 Active Directory 实例中所有可用的方法验证云中的用户。 此方案的体验类似于在本地环境中使用 Active Directory 迁移工具 (ADMT)。

### <a name="security-considerations"></a>安全注意事项
同步密码时，纯文本版本的密码既不能向密码同步功能公开，也不能向 Azure AD 或任何相关联的服务公开。

用户身份验证针对 Azure AD（而不是针对组织自己的 Active Directory 实例）进行。 如果组织对以任何形式离开本地的数据有任何顾虑，请考虑到这样一个事实：存储在 Azure AD 中的 SHA256 密码数据（原始 MD4 哈希的哈希）比 Active Directory 中存储的数据要安全得多。 而且，由于此 SHA256 哈希无法解密，因此无法将其带回到组织的 Active Directory 环境，并且在“传递哈希”攻击中显示为有效的用户密码。





### <a name="password-policy-considerations"></a>密码策略注意事项
有两种类型的密码策略受启用密码同步的影响：

- 密码复杂性策略
- 密码过期策略

#### <a name="password-complexity-policy"></a>密码复杂性策略  
启用密码同步时，本地 Active Directory 实例中的密码复杂性策略会替代云中可能为同步的用户定义的复杂性策略。 可以使用本地 Active Directory 实例的所有有效密码来访问 Azure AD 服务。

> [AZURE.NOTE]
> 直接在云中创建的用户的密码仍受到云中定义的密码策略的约束。

#### <a name="password-expiration-policy"></a>密码过期策略  
如果用户属于密码同步的范围，云帐户密码则设置为“永不过期”。

可以继续使用在本地环境中过期的同步密码来登录云服务。 当你下次在本地环境中更改密码时，云密码将会更新。

#### <a name="account-expiration"></a>帐户过期
如果组织在用户帐户管理中使用了 accountExpires 属性，请注意，此属性不会同步到 Azure AD。 因此，环境中为密码同步配置的过期 Active Directory 帐户仍将在 Azure AD 中处于活动状态。 我们建议，如果帐户已过期，工作流操作应触发一个 PowerShell 脚本以禁用用户的 Azure AD 帐户。 相反，在启用帐户后，Azure AD 实例应该开启。

### <a name="overwrite-synchronized-passwords"></a>覆盖已同步的密码
管理员可以使用 Windows PowerShell 手动重置密码。

在这种情况下，新密码会覆盖已同步密码，并且在云中定义的所有密码策略都会应用于新的密码。

如果再次更改本地密码，新密码则会同步到云，并会手动覆盖更新的密码。

同步密码对登录的 Azure 用户没有任何影响。 当前的云服务会话不会立即受到已同步密码更改的影响，而是在你登录云服务时才受到影响。 KMSI 会延长此差异的持续时间。 当云服务要求再次身份验证时，你需要提供新的密码。

### <a name="additional-advantages"></a>其他优点

- 通常情况下，密码同步比联合身份验证服务易于实现。 它不需要任何其他服务器，并且不依赖于高度可用的联合身份验证服务来对用户进行身份验证。 
- 除了联合身份验证，还可以启用密码同步。 如果联合身份验证服务发生了中断，它可以用作回退。












## <a name="enable-password-synchronization"></a>启用密码同步
使用“快速设置”选项安装 Azure AD Connect 时，会自动启用密码同步。 有关详细信息，请参阅[通过快速设置开始使用 Azure AD Connect](/documentation/articles/active-directory-aadconnect-get-started-express/)。

如果在安装 Azure AD Connect 时使用了自定义设置，可在用户登录页上使用密码同步。 有关详细信息，请参阅 [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom/)。

![启用密码同步](./media/active-directory-aadconnectsync-implement-password-synchronization/usersignin.png)

### <a name="password-synchronization-and-fips"></a>密码同步和 FIPS
如果已经根据美国联邦信息处理标准 (FIPS) 锁定服务器，则会禁用 MD5。

**若要为密码同步启用 MD5，请执行以下步骤：**

1. 转到 %programfiles%\Azure AD Sync\Bin。
2. 打开 miiserver.exe.config。
3. 转到文件末尾的 configuration/runtime 节点。
4. 添加以下节点： `<enforceFIPSPolicy enabled="false"/>`
5. 保存所做更改。

下面显示了此代码片段的大致情况，供参考：

	<configuration>
		<runtime>
			<enforceFIPSPolicy enabled="false"/>
		</runtime>
	</configuration>

有关安全性与 FIPS 的信息，请参阅 [AAD Password Sync, encryption and FIPS compliance](https://blogs.technet.microsoft.com/enterprisemobility/2014/06/28/aad-password-sync-encryption-and-fips-compliance/)（AAD 密码同步、加密和 FIPS 符合性）。

## <a name="troubleshoot-password-synchronization"></a>排查密码同步问题
如果遇到密码同步问题，请参阅[排查密码同步问题](/documentation/articles/active-directory-aadconnectsync-troubleshoot-password-synchronization/)。

## <a name="next-steps"></a>后续步骤
- [Azure AD Connect 同步：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis/)
- [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

<!---Update_Description: wording update -->