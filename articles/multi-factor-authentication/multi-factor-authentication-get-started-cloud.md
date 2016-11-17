<properties 
	pageTitle="云中的 Azure 多重身份验证入门" 
	description="这是与 Azure 多重身份验证相关的页面，介绍如何在云中开始使用 Azure MFA。" 
	services="multi-factor-authentication" 
	documentationCenter="" 
	authors="billmath" 
	manager="femila" 
	editor="curtand"/>  


<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="08/15/2016"
	wacn.date="11/16/2016"
	ms.author="kgremban"/>

# 云中的 Azure 多重身份验证入门
以下文章介绍如何在云中开始使用 Azure 多重身份验证。

> [AZURE.NOTE]  以下文档介绍如何通过 **Azure 经典管理门户**启用用户。若要了解如何为 O365 用户设置 Azure 多重身份验证，请参阅 [Setup多重身份验证for Office 365](https://support.office.com/article/Set-up-multi-factor-authentication-for-Office-365-users-8f0454b2-f51a-4d9c-bcde-2c48e41621c6?ui=zh-cn&rs=zh-cn&ad=US)（为 Office 365 设置多重身份验证）。

![云中的 MFA](./media/multi-factor-authentication-get-started-cloud/mfa_in_cloud.png)

## 先决条件
若要为用户启用 Azure 多重身份验证，必须满足以下先决条件。




- [注册 Azure 订阅](/pricing/1rmb-trial-full/?form-type=identityauth/) - 如果没有 Azure 订阅，则需要注册一个订阅。对于只是在摸索如何使用 Azure MFA 的新手，可以使用试用版订阅
2. 创建 Azure 多重身份验证提供程序并将其分配到目录，或者将许可证分配给用户。

> [AZURE.NOTE]  许可证将提供给拥有 Azure MFA、Azure AD Premium 或企业移动性套件 (EMS) 的用户。Azure AD Premium 和 EMS 中包含 MFA。如果你有足够的许可证，则不需要创建 Auth 提供程序。
		

## 为用户启用多重身份验证
若要为某个用户启用多重身份验证，只需将该用户的状态从已禁用更改为已启用。有关用户状态的详细信息，请参阅 [User States in Azure 多重身份验证](/documentation/articles/multi-factor-authentication-get-started-user-states/)（Azure 多重身份验证中的用户状态）。

可以使用以下过程为用户启用 MFA。

### 启用多重身份验证
--------------------------------------------------------------------------------
1.  以管理员身份登录到 **Azure 经典管理门户**。
2.  在左侧单击“Active Directory”。
3.  在“目录”下单击要为其启用此功能的用户的目录。
![单击目录](./media/multi-factor-authentication-get-started-cloud/directory1.png)
4.  在顶部单击“用户”。
5.  在页面底部，单击“管理 Multi-Factor Auth”。
![单击目录](./media/multi-factor-authentication-get-started-cloud/manage1.png)
6.  此时将打开一个新的浏览器选项卡。找到要为其启用多重身份验证的用户。你可能需要在顶部切换视图。确保状态为“已禁用”。
![启用用户](./media/multi-factor-authentication-get-started-cloud/enable1.png)
7.  **勾选**其名称旁边的框。
7.  在右侧，单击“启用”。
![启用用户](./media/multi-factor-authentication-get-started-cloud/user1.png)
8.  单击“启用 Multi-Factor Auth”。
![启用用户](./media/multi-factor-authentication-get-started-cloud/enable2.png)
9.  你应会注意到，用户的状态已从“已禁用”更改为“已启用”。
![启用用户](./media/multi-factor-authentication-get-started-cloud/user.png)
10.  启用用户后，建议你通过电子邮件通知他们。它还应该通知他们如何使用其非浏览器应用以避免被锁定。


## 使用 PowerShell 自动启用多重身份验证

若要使用 Azure AD PowerShell 更改[状态](/documentation/articles/multi-factor-authentication-whats-next/)，可以使用以下代码。可以将 `$st.State` 更改为以下状态之一：


- Enabled
- 强制
- 已禁用

> [AZURE.IMPORTANT]  请注意，如果直接从“禁用”状态进入“强制”状态，非新式验证客户端将停止工作，因为用户未经历 MFA 注册并获取[应用密码](/documentation/articles/multi-factor-authentication-whats-next/#app-passwords/)。如果使用非新式验证客户端并且需要应用密码，建议从“已禁用”状态更改为“已启用”状态。这样，用户便可以注册并获取其应用密码。
		
		$st = New-Object -TypeName Microsoft.Online.Administration.StrongAuthenticationRequirement
		$st.RelyingParty = "*"
		$st.State = “Enabled”
		$sta = @($st)
		Set-MsolUser -UserPrincipalName bsimon@contoso.com -StrongAuthenticationRequirements $sta

使用 PowerShell 是批量启用用户的选项。Azure 门户预览中目前未提供批量启用功能，需要单独选择每个用户。如果用户数量众多，则这项任务就很繁琐。使用上述代码创建 PowerShell 脚本可以循环访问用户列表并启用这些用户。下面是一个示例：
    
    $users = "bsimon@contoso.com","jsmith@contoso.com","ljacobson@contoso.com"
    foreach ($user in $users)
    {
    	$st = New-Object -TypeName Microsoft.Online.Administration.StrongAuthenticationRequirement
    	$st.RelyingParty = "*"
    	$st.State = “Enabled”
    	$sta = @($st)
    	Set-MsolUser -UserPrincipalName $user -StrongAuthenticationRequirements $sta
    }


有关用户状态的详细信息，请参阅 [User States in Azure 多重身份验证](/documentation/articles/multi-factor-authentication-get-started-user-states/)（Azure 多重身份验证中的用户状态）。

## 后续步骤
现在你已在云中安装多重身份验证，可以配置和设置部署。请参阅 [Configuring Azure 多重身份验证.](/documentation/articles/multi-factor-authentication-whats-next/)（配置 Azure 多重身份验证）。

<!---HONumber=Mooncake_1107_2016-->
