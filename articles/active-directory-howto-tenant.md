<properties
	pageTitle="如何获取 Azure AD 租户 | Microsoft Azure"
	description="如何获取用于注册和生成应用程序的 Azure Active Directory 租户。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="terrylan"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="09/28/2015"
	wacn.date="06/06/2016" />

# 如何获取 Azure Active Directory 租户

在 Azure Active Directory (Azure AD) 中，[租户](https://msdn.microsoft.com/zh-cn/library/azure/jj573650.aspx#BKMK_WhatIsAnAzureADTenant)表示组织。它是组织在注册 Azure、InTune 或 Office 365 等 Microsoft 云服务时接收并拥有的 Azure AD 服务专用实例。每个 Azure AD 租户都是独特的，独立于其他 Azure AD 租户。

租户包含公司中的用户以及有关这些用户的信息 - 他们的密码、用户配置文件数据、权限，等等。它还包含与某家组织及其安全性相关的组、应用程序和其他信息。

若要允许 Azure AD 用户登录到你的应用程序，必须在你自己的租户中注册你的应用程序。在 Azure AD 租户中发布应用程序是**完全免费的**。实际上，大多数开发人员都会针对试验、开发、过渡和测试目的创建多个租户和应用程序。注册和使用你的应用程序的组织如果想要利用高级目录功能，可以视需要选择购买许可证。

那么，怎样才能获得一个 Azure AD 租户呢？ 具体的过程根据你的以下情况而有所不同：

- [已有一个 Office 365 订阅](#use-an-existing-office-365-subscription)
- [已有一个与 Microsoft 帐户关联的现有 Azure 订阅](#use-an-msa-azure-subscription)
- [已有一个与组织帐户关联的现有 Azure 订阅](#use-an-organizational-azure-subscription)
- [没有上述任何订阅，想要从头开始](#start-from-scratch)

## <a name="use-an-existing-office-365-subscription"></a>使用现有的 Office 365 订阅
如果你已有一个 Office 365 订阅，但没有 Azure 订阅（因此也就无法登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)），请遵照[这些说明](https://technet.microsoft.com/zh-cn/library/dn832618.aspx)来访问你的 Azure AD 租户。

## <a name="use-an-msa-azure-subscription"></a>使用 MSA Azure 订阅
如果你以前使用个人 Microsoft 帐户注册过 Azure 订阅，则你已经有了一个租户！ 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中，你应该会在“所有项”和“Active Directory”的下面看到名为“默认租户”的租户。 你可以根据需要任意使用此租户 - 不过，有时你可能想要创建一个组织管理员帐户。

为此，请执行下列步骤。或者，你可能想要创建新的租户，然后遵循类似的过程中在该租户中创建管理员。

1.	使用你的个人帐户登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)
2.	导航到门户的“Active Directory”部分（在左侧导航栏中）
3.	在可用目录列表中选择“默认目录”条目
4.	单击页面顶部的“用户”链接。你将在列表的“源自”列中看到值为“Microsoft 帐户”的单个用户
5.	单击页面底部的“添加用户”
6.	在“添加用户表单”中提供以下详细信息：
    - 用户类型：组织中的新用户
    - 用户名：（选择此管理员的用户名）
    - 名字/姓氏/显示名称：（选择适当的值）
    - 角色：全局管理员
    - 备用电子邮件地址：（输入适当的值）
    - 可选：启用多因素身份验证
    - 最后，单击绿色的“创建”按钮以完成用户创建（并显示临时密码）。
7.	完成“添加用户表单”并收到新管理用户的临时密码后，请务必记下此密码，因为在更改密码时，你要以此新用户的身份登录。你还可以使用备用电子邮件直接向用户发送密码。
8.	若要更改临时密码，请使用此新用户帐户登录到 https://login.microsoftonline.com ，然后根据请求更改密码。


## <a name="use-an-organizational-azure-subscription"></a>使用组织 Azure 订阅
如果你以前使用组织帐户注册过 Azure 订阅，则你已经有了一个租户！ 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中，你应该会在“所有项”和“Active Directory”的下面看到一个租户。 你可以根据需要任意使用此租户。你还可以想要使用门户左下角的“新建”按钮创建新租户。


## <a name="tart-from-scratch"></a>从头开始
如果上述所有方法都不起作用，请不要担心。只需访问 [https://account.windowsazure.cn/organization](https://account.windowsazure.cn/organization)，以新组织的身份注册 Azure。完成注册过程后，你将会获得自己的 Azure AD 租户，该租户使用了你在注册期间选择的域名。在 [Azure 经典管理门户](https://manage.windowsazure.cn)中，可以通过导航到左侧导航栏中“Active Directory”来查找你的租户。

在注册 Azure 的过程中，你需要提供信用卡详细信息。你可以放心地继续注册 - 在 Azure AD 中发布应用程序或者创建新租户时，我们不会向你收费。

<!---HONumber=Mooncake_0418_2016-->