<properties 
	pageTitle="通过 Azure Active Directory 认证 Azure 自动化服务" 
	description="如何在 Azure 自动化服务中使用 Azure AD 认证" 
	services="automation" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="automation-aog" ms.date="" wacn.date="02/01/2016"/>

#通过 Azure Active Directory 认证 Azure 自动化服务

关于 Azure 自动化的详细概念请阅读[这篇文章](/home/features/automation)。

Azure 自动化是通过 Windows PowserShell 工作流（也称为 Runbook）来处理 Azure 的资源和第三方应用的创建、部署、监视和维护工作的。在执行 Runbook 的时候自然需要认证是否拥有合理的身份来执行操作。本文介绍如何通过 Azure Active Directory 来授权。

首先去确认 Azure 订阅的目录名字（本文使用的订阅账户是"Microsoft"），可以在 Azure 经典管理门户网站设置里找到，如下图：

![](./media/aog-automation-connect-mooncake/get-directory.PNG)

之后去 Azure Active Directory 里找到对应的目录，如下图：

![](./media/aog-automation-connect-mooncake/find-active-directory.PNG)

点击进入"Microsoft"的 Active Diectory，点击用户，然后添加用户

![](./media/aog-automation-connect-mooncake/entry-user.PNG)

输入要创建的用户名：

![](./media/aog-automation-connect-mooncake/create-new-user.PNG)

输入名字、姓氏和显示名称：


![](./media/aog-automation-connect-mooncake/create-user2.PNG)

**注意：**不要勾选启用多重身份验证，目前在 Azure 自动化服务中还不支持多重身份用户。

![](./media/aog-automation-connect-mooncake/create-user3.PNG)

复制记录的密码，然后在 Azure 经典管理门户登录，第一次登陆需要修改密码。如果使用初始密码作为 Runbook 的凭证可能会导致操作失败。

然后进入自动化的资产，点击添加设置：

![](./media/aog-automation-connect-mooncake/entry-automation.PNG)

![](./media/aog-automation-connect-mooncake/add-config.PNG)

本文主要讨论使用 PowerShell 的方式来授权，所以我们选择“Windows PowerShell”凭据。

![](./media/aog-automation-connect-mooncake/define-config.PNG)

输入刚才创建的用户名和密码：

![](./media/aog-automation-connect-mooncake/input-user-information.PNG)

至此PowerShell的凭据已经创建完成，下面讨论如何来使用（本文主要是使用自动化服务实现开关虚拟机）。
首先我们进入 Runbook

![](./media/aog-automation-connect-mooncake/entry-runbook.PNG)

然后点击创作：

![](./media/aog-automation-connect-mooncake/edit-draft.PNG)

在创作的过程中使用下面的指令来实现授权，“Powercredential”为在定义凭据时使用的名称。

	$Cred = Get-AutomationPSCredential -Name "Powercredential"; 
    Add-AzureAccount -Credential $Cred -Environment AzureChinaCloud;
    Select-AzureSubscription -SubscriptionName "<subscription name>";  








