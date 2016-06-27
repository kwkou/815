<properties
   pageTitle="适用于应用程序的品牌准则 | Azure"
   description="介绍面向开发人员的 Azure Active Directory 资源的综合性指南"
   services="active-directory"
   documentationCenter="dev-center-name"
   authors="msmbaldwin"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="01/21/2016"
   wacn.date="05/18/2016"/>


# 适用于应用程序的品牌准则


本主题讨论在使用 Azure Active Directory 开发应用程序时应使用的品牌准则。在客户需要使用 Azure AD 中托管的工作或学校帐户进行注册和登录到应用程序时，这些准则将帮助指导客户进行相关操作。

## 个人帐户与 Microsoft 中的工作或学校帐户

Microsoft 管理两种类型的用户帐户：

- **个人帐户**（以前称为 Windows Live ID）。这些帐户表示个人用户与 Microsoft 之间的关系，用于访问客户设备和 Microsoft 中的服务。这些帐户专供个人使用。

- **工作或学校帐户。** 这些帐户由 Microsoft 代表使用 Azure Active Directory 的组织进行管理。这些帐户用于登录 Office 365 和 Microsoft 的其他业务服务。

Microsoft 工作或学校帐户帐户通常由组织（公司、学校、政府机构）分配给最终用户（员工、学生、联邦雇员）。这些帐户可以直接在云中或Azure AD 中直接控制，也可以从本地目录（如 Windows Server Active Directory）同步到 Azure AD。Microsoft 是工作或学校帐户的监管员，但这些帐户由组织所有和控制。

## 在应用程序中引用 Azure AD 帐户

Microsoft 不会向最终用户显示 Azure 或 Active Directory 品牌名称，你也应该如此。

- 在用户登录后，你应尽量使用组织的名称和徽标。这比使用“你的组织”等常用词语来好。

- 如果用户未登录，你应该将他们的帐户称为“工作或学校帐户”，并使用 Microsoft 徽标来表明这些帐户由 Microsoft 管理。请勿使用“企业帐户”、“业务帐户”或“公司帐户”等词语，这会给用户造成混淆。

## 用户帐户象形图
在先前版本的准则中，我们建议使用“蓝色徽章”象形图。根据用户和开发人员的反馈，我们现在建议改用 Microsoft 徽标。这会帮助用户了解，他们可以重用其用于 Office 365 或其他 Microsoft 业务服务的帐户来登录你的应用程序。

## 使用 Azure AD 注册和登录

你的应用程序可以为注册和登录提供不同的路径，以下部分提供了这两种应用场景的可视指南。

**如果你的应用程序支持最终用户注册（例如免费试用或免费增值模式）**：你可以显示**登录**按钮，让用户使用他们在 Microsoft 中的工作或学校帐户访问你的应用程序。当用户首次访问你的应用程序时，Azure AD 将显示许可提示。

**如果应用程序需要只有管理员才能授予的权限，或者应用程序需要组织许可**：应该将管理员请求与用户登录区别开来。**“获取此应用程序”按钮**会将管理员重定向到登录页，然后要求他们代表其组织中的用户授权。带来的额外好处是，应用程序中不会显示最终用户许可提示。

## 有关获取应用程序的可视指南

“获取应用程序”链接必须将用户重定向到 Azure AD 的访问权限授予（授权）页，以方便组织的管理员对你的应用程序进行授权，使其有权访问 Microsoft 托管的组织数据。

管理员许可你的应用程序后，可以选择将应用程序添加到其用户的 Office 365 应用程序启动器体验（可从 waffle 和 [https://portal.office.com/myapps](https://portal.office.com/myapps) 访问）。如果你想要广告此功能，可以使用类似于“将此应用程序添加到你的组织”词语，并显示类似于下面的按钮：

![应用程序类型和方案](./media/active-directory-branding-guidelines/add-to-my-org.png)
  
但是，我们建议你编写说明性的文本而不要依赖于按钮。例如：
> 如果你已使用 Office 365 或 Microsoft 的其他业务服务，则只需授予 <your_app_name> 对你的组织数据的访问权限。这样，你的用户便可以使用其现有工作帐户访问 <your_app_name>。


## 有关登录的可视指南
你的应用程序应显示登录按钮，用于将用户重定向到对应于用来与 Azure AD 集成的协议的登录终结点。以下部分详细描述了该按钮的外观。

### 象形图和“工作或学校帐户”
将 Microsoft 徽标和通用的“工作或学校”词语关联之后，即可在你的应用程序支持的其他标识提供程序中以唯一方式来表示 Azure AD。如果没有为“工作或学校帐户”留出足够的空间，可以将它缩短为“工作帐户”。

![应用程序类型和方案](./media/active-directory-branding-guidelines/work-or-school-account.png)

![应用程序类型和方案](./media/active-directory-branding-guidelines/work-account.png)

你还可以向最终用户提供一些附加说明，帮助他们确认是否可以使用此按钮：

![应用程序类型和方案](./media/active-directory-branding-guidelines/work-account-with-explaination.png)

## 品牌注意事项
**要**将“工作或学校帐户”与 Microsoft 徽标结合使用来表示通过 Azure AD 进行的登录如果空间短缺，可以显示为“工作帐户”，但**不要**使用其他词语（例如“企业帐户”、“业务帐户”或“公司帐户”）。

**不要**使用“Office 365 ID”或“Azure ID”。Office 365 也是 Microsoft 的消费型产品名称，它不使用 Azure AD 进行身份验证。

**不要**更改 Microsoft 徽标。

**不要**向最终用户显示 Azure 或 Active Directory 品牌。但是，可以对开发人员、IT 专业人员和管理员使用这些词语。

## 导航注意事项

**要**提供让用户注销以及切换到其他用户帐户的方法。虽然大多数人员只有一个由 Microsoft/Facebook/Google/Twitter 提供的个人帐户，但这些人员通常与多个组织相关联。即将推出支持多个登录用户的功能。

## 在你的应用程序中同时支持 Azure AD 帐户和 Microsoft 帐户

如果你的应用程序同时支持 Azure AD 帐户和 Microsoft 帐户，则需要在应用程序中提供两个单独的登录按钮。我们正在积极处理更新，以便你只需集成一次，就能支持个人帐户和 Microsoft 中的工作帐户。在使用此更新后，你便可以在应用程序中显示“使用 Microsoft 登录”按钮。

<!---HONumber=Mooncake_0613_2016-->