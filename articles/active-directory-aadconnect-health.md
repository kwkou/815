<properties
	pageTitle="在云中监视本地标识基础结构"
	description="本页介绍 Azure AD Connect Health 是什么，以及为何要使用它。"
	services="active-directory"
	documentationCenter=""
	authors="karavar"
	manager="stevenpo"
	editor="karavar"/>

<tags 
	ms.service="active-directory"
	ms.date="03/21/2016"
	wacn.date="04/06/2016"/>

# 在云中监视本地标识基础结构和同步服务

Azure AD Connect Health 可帮助你监视和深入了解本地标识基础结构和同步服务。它针对重要的标识组件 - 例如 AD FS 服务器、Azure AD Connect 服务器（即同步引擎）、Active Directory 域控制器等 - 提供监视功能，使你可以与 Office 365 和 Microsoft Online Services 建立可靠的连接。它还可以让你轻松访问有关这些组件的关键数据点，让你轻松获取使用情况信息和其他重要见解。

这些信息全部显示在 [Azure AD Connect Health 门户](https://aka.ms/aadconnecthealth)中。使用 Azure AD Connect Health 门户可以查看警报、性能监视、使用情况分析等信息。Azure AD Connect Health 在一个集中的位置提供重要标识组件的运行状况单一透视图。

![什么是 Azure AD Connect Health](./media/active-directory-aadconnect-health/aadconnecthealth2.png)

将来对 Azure AD Connect Health 所做的更新将会包括附加的监视功能，以及深入分析其他标识组件的功能。因此，它将通过标识透视图提供单个仪表板，让你能拥有更稳健的集成环境，使用户能够充分利用该环境来增强工作能力。

## 为何使用 Azure AD Connect Health

将本地目录与 Azure AD 集成可提供通用标识用于访问云和本地资源，从而提高用户的生产率。但是，这种集成带来的挑战是，必须确保此环境正常运行，才能让用户从任何设备可靠地访问本地和云中的资源。Azure AD Connect Health 提供简单的基于云的方法让你监视和深入分析用于访问 Office 365 或其他 Azure AD 应用程序的本地标识基础结构。这种方法就像在每个本地标识服务器上安装代理那样简单。

## [适用于 AD FS 的 Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health-adfs/)

适用于 AD FS 的 Azure AD Connect Health 支持 Windows Server 2008 R2 中的 AD FS 2.0，以及 Windows Server 2012 和 Windows Server 2012R2 中的 AD FS。其中还包括为 Extranet 访问提供身份验证支持的 AD FS 代理或 Web 应用程序代理服务器。适用于 AD FS 的 Azure AD Connect Health 的安装非常简单且费用低廉，同时提供以下重要功能集：

- 使用警报进行监视，以便在 AD FS 和 AD FS 代理服务器状况不正常时知道这一点
- 关键警报电子邮件通知
- 查看性能数据的趋势，这在规划 AD FS 的容量时非常有用
- 使用不同的透视图（应用、用户、网络位置等）了解有关 AD FS 登录的使用情况分析数据，这对你了解 AD FS 的使用情况非常有用。
- 针对 AD FS 的报告，例如，使用不当的用户名/密码尝试登录的前 50 个用户

以下视频将概述适用于 AD FS 的 Azure AD Connect Health



## Azure AD Connect Health 入门
Azure AD Connect Health 很容易入门。请遵循以下步骤进行配置：

1. [获取 Azure AD Premium](/documentation/articles/active-directory-get-started-premium/) 或[开始试用](https://azure.microsoft.com/trial/get-started-active-directory/)

2. 在标识服务器上[下载并安装 Azure AD Connect Health 代理](#download-and-install-azure-ad-connect-health-agent)。

3. 在 [https://aka.ms/aadconnecthealth](https://aka.ms/aadconnecthealth) 上查看 Azure AD Connect Health 仪表板

>[AZURE.NOTE]请记住在查看 Azure AD Connect Health 仪表板中的任何数据之前，你将需要在目标服务器上安装 Azure AD Connect Health 代理。

## Azure AD Connect Health 门户
可以使用 Azure AD Connect Health 门户来查看警报、性能监视情况和使用情况分析。单击 https://aka.ms/aadconnecthealth 可转到 Azure AD Connect Health 的主边栏选项卡。你可以将边栏选项卡视为窗口。在主边栏选项卡上，你将看到“快速启动”、“Azure AD Connect Health 中的服务”和其他配置选项。在屏幕快照下方是上述每个选项卡的简短说明。部署代理后，Azure AD Connect Health 将监视服务的服务标识符。

![Azure AD Connect Health 门户](./media/active-directory-aadconnect-health/portal2.png)

- **快速启动** – 选择此项将打开“快速启动”边栏选项卡。在这里，你可以通过选择“获取工具”来下载 Azure AD Connect Health 代理，可以访问文档，还可以提供反馈。

- **Active Directory 联合身份验证服务** – 此项表示 Azure AD Connect Health 当前监视的所有 AD FS 服务。选择其中一个实例后，就会打开一个边栏选项卡，其中包含该服务实例的相关信息。该信息包括概述、属性、警报、监视情况，以及使用情况分析。在[此处](/documentation/articles/active-directory-aadconnect-health-adfs/)阅读有关功能的详细信息。

- **配置** – 使用此项可打开或关闭以下功能：

	1. 通过自动更新自动将 Azure AD Connect Health 代理更新到最新版本 - 这意味着，你将在 Azure AD Connect Health 代理发布时自动更新到最新版本的该软件。此项已默认启用。

	2. 允许 Microsoft 访问你 Azure AD 目录的运行状况数据，但只能将其用于故障诊断目的 - 这意味着，如果启用此功能，Microsoft 将能够查看你所看到的数据。这有助于进行故障诊断，并帮助你解决问题。此项已默认禁用。


## 相关链接

* [Azure AD Connect Health 操作](/documentation/articles/active-directory-aadconnect-health-operations/)
* [在 AD FS 中使用 Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health-adfs/)
* [Azure AD Connect Health 常见问题](/documentation/articles/active-directory-aadconnect-health-faq/)
 

<!---HONumber=Mooncake_0509_2016-->