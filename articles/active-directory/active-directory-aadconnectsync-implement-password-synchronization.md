<properties
    pageTitle="使用 Azure AD Connect 同步实现密码同步 | Azure"
    description="提供有关密码同步工作原理以及如何启用密码同步的信息。"
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
    ms.date="01/13/2017"
    wacn.date="03/13/2017"
    ms.author="markvi" />

# 使用 Azure AD Connect 同步实现密码同步
本主题提供所需的信息，帮助将用户密码从本地 Active Directory (AD) 同步到基于云的 Azure Active Directory (Azure AD)。

## 什么是密码同步
因为忘记密码而无法完成工作的机率与需要记住的不同密码数目有关。要记住的密码越多，忘记密码的机率就越高。就密码重置和其他密码相关问题的疑问和来电占据了最多的技术服务资源。

密码同步是用于将用户密码从本地 Active Directory 同步到基于云的 Azure Active Directory (Azure AD) 的功能。
利用此功能，可以使用用于登录到本地 Active Directory 的同一个密码登录到 Azure Active Directory 服务（如 Office 365、Microsoft Intune、CRM Online 和 Azure AD 域服务）。

![什么是 Azure AD Connect](./media/active-directory-aadconnectsync-implement-password-synchronization/arch1.png)

密码同步可通过将用户需要维护的密码数目减少到只剩一个，帮助你：

- 提升用户的工作效率
- 减少技术支持成本


密码同步是由 Azure AD Connect Sync 实现的目录同步功能的扩展。若要在环境中使用密码同步，需要：

- 安装 Azure AD Connect
- 配置本地 AD 与 Azure Active Directory 之间的目录同步
- 启用密码同步

有关详细信息，请参阅[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

> [AZURE.NOTE]
有关为 FIPS 和密码同步配置的 Active Directory 域服务的更多详细信息，请参阅[密码同步和 FIPS](#password-synchronization-and-fips)。
>
>

## 密码同步的工作原理
Active Directory 域服务以实际用户密码的哈希值表示形式存储密码。哈希值是单向数学函数（“*哈希算法*”）的计算结果。没有任何方法可将单向函数的结果还原为纯文本版本的密码。无法使用密码哈希来登录本地网络。

为了同步密码，Azure AD Connect 同步将从本地 Active Directory 提取你的密码哈希。同步到 Azure Active Directory 身份验证服务之前，已对密码哈希应用其他安全处理。密码将基于每个用户按时间顺序同步。

密码同步过程的实际数据流等同于用户数据（例如 DisplayName 或电子邮件地址）的同步。但是，密码的同步频率高于其他属性的标准目录同步窗口。密码同步过程每隔 2 分钟运行一次。无法修改此过程的运行频率。同步某个密码时，该密码将覆盖现有的云密码。

首次启用密码同步功能时，它将对范围内的所有用户执行初始密码同步。你无法显式定义一部分要同步的用户密码。

当你更改本地密码时，更新后的密码将会同步，此操作基本上在几分钟内就可完成。
密码同步功能会自动重试失败的同步尝试。如果尝试同步密码期间出现错误，该错误会被记录在事件查看器中。

同步密码对当前登录的用户没有任何影响。
当前的云服务会话不会立即受到已同步密码更改的影响，而是在你登录云服务时才受到影响。但是，当云服务要求你再次身份验证时，就需要提供新的密码。

> [AZURE.NOTE]
只有 Active Directory 的对象类型用户才支持密码同步。不支持 iNetOrgPerson 对象类型。
>
>

### 密码同步在 Azure AD 域服务中的工作原理
也可以使用密码同步功能将本地密码同步到 Azure AD 域服务。此方案可让 Azure AD 域服务以本地 AD 中所有可用的方法验证云中的用户。此方案的体验类似于在本地环境中使用 Active Directory 迁移工具 (ADMT)。

### 安全注意事项
同步密码时，纯文本版本的密码既不能向密码同步功能公开，也不能向 Azure AD 或任何相关联的服务公开。

此外，对于本地 Active Directory 以可撤消的加密格式存储密码没有任何要求。Active Directory 密码哈希摘要用于本地 AD 和 Azure Active Directory 之间的传输。密码哈希摘要不能用于访问本地环境中的资源。

### 密码策略注意事项
有两种类型的密码策略受启用密码同步的影响：

1. 密码复杂性策略
2. 密码过期策略

**密码复杂性策略**

启用密码同步时，本地 Active Directory 中的密码复杂性策略会覆盖云中可能为同步的用户定义的复杂性策略。可以使用本地 Active Directory 的所有有效密码来访问 Azure AD 服务。

> [AZURE.NOTE]
直接在云中创建的用户的密码仍受到云中定义的密码策略的约束。
>
>

**密码过期策略**

如果用户属于密码同步的范围，云帐户密码则设置为“永不过期”。可以继续使用已在本地环境中过期的同步密码来登录云服务。当你下次在本地环境中更改密码时，云密码将会更新。

### 覆盖已同步的密码
管理员可以使用 Windows PowerShell 手动重置密码。

在这种情况下，新密码会覆盖你的已同步密码，并且在云中定义的所有密码策略都会应用于新的密码。

如果你再次更改本地密码，新密码则会同步到云，并会手动覆盖更新的密码。

## 启用密码同步
使用“快速设置”安装 Azure AD Connect 时，会自动启用密码同步。有关详细信息，请参阅[通过快速设置开始使用 Azure AD Connect](/documentation/articles/active-directory-aadconnect-get-started-express/)。

如果在安装 Azure AD Connect 时使用了自定义设置，则必须在用户登录页上启用密码同步。有关详细信息，请参阅 [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom/)。

![启用密码同步](./media/active-directory-aadconnectsync-implement-password-synchronization/usersignin.png)

### 密码同步和 FIPS <a name="password-synchronization-and-fips"></a>
如果已经根据美国联邦信息处理标准 (FIPS) 锁定服务器，则已禁用 MD5。

**若要为密码同步启用 MD5，请执行以下步骤：**

1. 转到 **%programfiles%\\Azure AD Sync\\Bin**。
2. 打开 **miiserver.exe.config**。
3. 转到 **configuration/runtime** 节点（位于文件末尾）。
4. 添加以下节点：`<enforceFIPSPolicy enabled="false"/>`
5. 保存所做更改。

下面显示了此代码片段的大致情况供参考：

    <configuration>
        <runtime>
            <enforceFIPSPolicy enabled="false"/>
        </runtime>
    </configuration>


有关安全性与 FIPS 的信息，请参阅 [AAD 密码同步、加密和 FIPS 合规性](https://blogs.technet.microsoft.com/enterprisemobility/2014/06/28/aad-password-sync-encryption-and-fips-compliance/)

## 排查密码同步问题  <a name="troubleshooting-password-synchronization"></a>
如果密码未按预期同步，请区分该密码是一部分用户的密码还是所有用户的密码。

- 如果单个对象出现问题，请参阅[排查一个对象的密码同步问题](#troubleshoot-one-object-that-is-not-synchronizing-passwords)。
- 如果遇到密码都未同步的问题，请参阅[排查未同步任何密码的问题](#troubleshoot-issues-where-no-passwords-are-synchronized)。

### 排查一个对象的密码同步问题  <a name="troubleshoot-one-object-that-is-not-synchronizing-passwords"></a>
可以通过检查对象的状态，轻松排查密码同步问题。

首先打开“Active Directory 用户和计算机”。找到用户并检查是否未选择“用户必须在下次登录时更改密码”。

![Active Directory 效率密码](./media/active-directory-aadconnectsync-implement-password-synchronization/adprodpassword.png)  


如果已选择，请要求该用户登录并更改密码。临时密码不会同步到 Azure AD。

如果 Active Directory 中的设置正确，下一步是在同步引擎中跟踪该用户。从本地 Active Directory 到 Azure AD 的路径中跟踪该用户，可以查看该对象是否出现描述性错误。

1. 启动**[同步服务管理器](/documentation/articles/active-directory-aadconnectsync-service-manager-ui/)**。
2. 单击“连接器”。
3. 选择用户所在的 **Active Directory 连接器**。
4. 选择“搜索连接器空间”。
5. 找到你要查找的用户。
6. 选择“沿袭”选项卡，确保至少有一个同步规则的“密码同步”显示为 **True**。在默认配置中，同步规则的名称为 **In from AD - User AccountEnabled**。

	![有关用户的沿袭信息](./media/active-directory-aadconnectsync-implement-password-synchronization/cspasswordsync.png)

7. 然后，从 Metaverse 到 Azure AD 连接器空间一直[跟踪用户](/documentation/articles/active-directory-aadconnectsync-service-manager-ui-connectors/)。连接器空间对象应存在一个“密码同步”设置为 **True** 的出站规则。在默认配置中，同步规则的名称为 **Out to AAD - User Join**。

	![用户的连接器空间属性](./media/active-directory-aadconnectsync-implement-password-synchronization/cspasswordsync2.png)

8. 若要查看对象在过去一周的密码同步详细信息，请单击“日志...”。
	
	![对象日志详细信息](./media/active-directory-aadconnectsync-implement-password-synchronization/csobjectlog.png)

	如果对象日志为空，则 Azure AD Connect 不能从 Active Directory 读取密码哈希。查看错误的事件日志。

状态列可能包含以下值：

| 状态 | 说明 |
| --- | --- |
| Success |已成功同步密码。 |
| FilteredByTarget |密码设置为“用户在下次登录时必须更改密码”。未同步密码。 |
| NoTargetConnection |Metaverse 或 Azure AD 连接器空间中没有任何对象。 |
| SourceConnectorNotPresent |在本地 Active Directory 连接器空间中找不到任何对象。 |
| TargetNotExportedToDirectory |尚未导出 Azure AD 连接器空间中的对象。 |
| MigratedCheckDetailsForMoreInfo |日志条目创建于版本 1.0.9125.0 之前，并且以其旧状态显示。 |

### 排查未同步任何密码的问题 <a name="troubleshoot-issues-where-no-passwords-are-synchronized"></a>
首先，请运行[“获取密码同步设置”](#get-the-status-of-password-sync-settings)部分中的脚本。这样就可以获得密码同步配置的概述。
![PowerShell 脚本从密码同步设置中返回的输出](./media/active-directory-aadconnectsync-implement-password-synchronization/psverifyconfig.png)
如果未在 Azure AD 中启用该功能，或者未启用同步通道状态，请运行 Connect 安装向导。选择“自定义同步选项”并取消选择密码同步。此项更改会暂时禁用该功能。然后再次运行向导并重新启用密码同步。再次运行脚本，验证配置是否正确。

如果脚本显示没有检测信号，请运行[“触发所有密码的完全同步”](#trigger-a-full-sync-of-all-passwords)中的脚本。在配置正确但密码不同步的其他一些情况下。也可以使用此脚本。

如果使用自定义设置安装 Azure AD Connect，请确保已将“复制目录更改”和“复制所有目录更改”权限授予 AD 连接器使用的帐户。请参阅[帐户和权限](/documentation/articles/active-directory-aadconnect-accounts-permissions/#create-the-ad-ds-account/)，了解此帐户所需的所有权限。如果没有这些权限，此帐户将无权读取 Active Directory 中的密码哈希。

然后查看应用程序事件日志。如果密码同步存在全局问题，并且服务正在运行，则如上述步骤所验证，应存在包含更多详细信息的错误。

#### 获取密码同步设置的状态 <a name="get-the-status-of-password-sync-settings"></a>

	Import-Module ADSync
	$connectors = Get-ADSyncConnector
	$aadConnectors = $connectors | Where-Object {$_.SubType -eq "Azure Active Directory (Microsoft)"}
	$adConnectors = $connectors | Where-Object {$_.ConnectorTypeName -eq "AD"}
	if ($aadConnectors -ne $null -and $adConnectors -ne $null)
	{
	    if ($aadConnectors.Count -eq 1)
	    {
	        $features = Get-ADSyncAADCompanyFeature -ConnectorName $aadConnectors[0].Name
	        Write-Host
	        Write-Host "Password sync feature enabled in your Azure AD directory: "  $features.PasswordHashSync
	        foreach ($adConnector in $adConnectors)
	        {
	            Write-Host
	            Write-Host "Password sync channel status BEGIN ------------------------------------------------------- "
	            Write-Host
	            Get-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector.Name
	            Write-Host
	            $pingEvents =
		                Get-EventLog -LogName "Application" -Source "Directory Synchronization" -InstanceId 654  -After (Get-Date).AddHours(-3) |
	                    Where-Object { $_.Message.ToUpperInvariant().Contains($adConnector.Identifier.ToString("D").ToUpperInvariant()) } |
	                    Sort-Object { $_.Time } -Descending
	            if ($pingEvents -ne $null)
	            {
	                Write-Host "Latest heart beat event (within last 3 hours). Time " $pingEvents[0].TimeWritten
	            }
	            else
	            {
	                Write-Warning "No ping event found within last 3 hours."
	            }
	            Write-Host
	            Write-Host "Password sync channel status END ------------------------------------------------------- "
	            Write-Host
	        }
	    }
	    else
	    {
	        Write-Warning "More than one Azure AD Connectors found. Please update the script to use the appropriate Connector."
	    }
	}
	Write-Host
	if ($aadConnectors -eq $null)
	{
	    Write-Warning "No Azure AD Connector was found."
	}
	if ($adConnectors -eq $null)
	{
	    Write-Warning "No AD DS Connector was found."
	}
	Write-Host

#### 触发所有密码的完全同步 <a name="trigger-a-full-sync-of-all-passwords"></a>
可以使用以下脚本对所有密码触发完全同步：

	$adConnector = "<CASE SENSITIVE AD CONNECTOR NAME>"
	$aadConnector = "<CASE SENSITIVE AAD CONNECTOR NAME>"
	Import-Module adsync
	$c = Get-ADSyncConnector -Name $adConnector
	$p = New-Object Microsoft.IdentityManagement.PowerShell.ObjectModel.ConfigurationParameter "Microsoft.Synchronize.ForceFullPasswordSync", String, ConnectorGlobal, $null, $null, $null
	$p.Value = 1
	$c.GlobalParameters.Remove($p.Name)
	$c.GlobalParameters.Add($p)
	$c = Add-ADSyncConnector -Connector $c
	Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $aadConnector -Enable $false
	Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $aadConnector -Enable $true


## 后续步骤
- [Azure AD Connect Sync：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis/)
- [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: wording update -->