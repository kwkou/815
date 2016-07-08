<properties
	pageTitle="在 Azure Active Directory 中设置密码过期策略 | Azure"
	description="了解如何检查 Azure Active Directory 密码的过期策略，以及如何逐个或批量更改用户密码过期策略"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="msStevenPo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/16/2016"
	wacn.date="06/27/2016"/>


# 在 Azure Active Directory 中设置密码过期策略
> [AZURE.NOTE] 本主题为依赖于 Microsoft Azure Active Directory 提供标识和目录服务的云服务（如 Microsoft Intune 和 Office 365）提供联机帮助内容。

作为 Microsoft 云服务的全局管理员，你可以使用适用于 Windows PowerShell 的 Microsoft Azure Active Directory 模块将用户密码设置为永不过期。你还可以使用 Windows PowerShell cmdlet 删除永不过期配置，或者查看已将哪些用户密码设置为永不过期。

  > [AZURE.NOTE] 只能将未通过目录同步进行同步的用户帐户的密码配置为永不过期。有关目录同步的详细信息，请参阅[目录同步路线图](/documentation/articles/active-directory-aadconnect/)中的主题列表。

若要使用 Windows PowerShell cmdlet，首先必须安装它们。

## 要执行什么操作？

- [如何检查密码过期策略](#how-to-check-expiration-policy-for-a-password)

- [设置密码过期](#set-a-password-to-expire)

- [将密码设置为永不过期](#set-a-password-not-to-expire)

## <a name="how-to-check-expiration-policy-for-a-password"></a>如何检查密码过期策略

1.  使用公司管理员凭据连接到 Windows PowerShell。

2.  执行下列操作之一：

	- 若要查看单个用户的密码是否已设置为永不过期，请使用用户主体名称 (UPN)（例如 aprilr@contoso.onmicrosoft.com）或要查看的用户的用户 ID 运行以下 cmdlet：`Get-MSOLUser -UserPrincipalName <user ID> | Select PasswordNeverExpires`

	- 若要查看所有用户的“密码永不过期”设置，请运行以下 cmdlet：`Get-MSOLUser | Select UserPrincipalName, PasswordNeverExpires`

## <a name="set-a-password-to-expire"></a>设置密码过期

1.  使用公司管理员凭据连接到 Windows PowerShell。

2.  执行下列操作之一：

	- 若要将某一个用户的密码设置为会过期，请使用用户主体名称 (UPN) 或该用户的用户 ID 运行以下 cmdlet：`Set-MsolUser -UserPrincipalName <user ID> -PasswordNeverExpires \$false`
  	
	- 若要将组织中所有用户的密码设置为会过期，请使用以下 cmdlet：`Get-MSOLUser | Set-MsolUser -PasswordNeverExpires \$false`

## <a name="set-a-password-not-to-expire"></a>将密码设置为永不过期

1. 使用公司管理员凭据连接到 Windows PowerShell。

2.  执行下列操作之一：

	- 若要将某一个用户的密码设置为永不过期，请使用用户主体名称 (UPN) 或该用户的用户 ID 运行以下 cmdlet：`Set-MsolUser -UserPrincipalName <user ID> -PasswordNeverExpires \$true`

	- 若要将组织中所有用户的密码设置为永不过期，请运行以下 cmdlet：`Get-MSOLUser | Set-MsolUser -PasswordNeverExpires \$true`

<!---HONumber=Mooncake_0620_2016-->