<properties 
	pageTitle="在云中监视本地标识基础结构。" 
	description="本页介绍 Azure AD Connect Health 是什么，以及为何要使用它。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory"  
	ms.date="10/15/2015"
	wacn.date="11/12/2015"/>

# 在云中监视本地标识基础结构和同步服务

Azure AD Connect Health 可帮助你监视和深入了解本地标识基础结构以及通过 Azure AD Connect 提供的同步服务。它允许你查看警报、性能、使用模式、配置设置，使你能够维持与 Office 365 的可靠连接，等等。这种优势是使用目标服务器上安装的代理实现的。

![什么是 Azure AD Connect Health](./media/active-directory-aadconnect-health/aadconnecthealth2.png)


这些信息全部显示在 Azure AD Connect Health 门户中。使用 Azure AD Connect Health 门户可以查看警报、性能监视和使用情况分析。这些信息全部显示在一个易于使用的位置，你不必浪费时间寻找所需的信息。

![什么是 Azure AD Connect Health](./media/active-directory-aadconnect-health/usage.png)

将来对 Azure AD Connect Health 所做的更新将会包括附加的监视功能，以及深入分析其他标识组件和服务（例如 Azure AD Connect Sync 服务）的功能。因此，它将通过标识透视图提供单个仪表板，让你能拥有更稳健的集成环境，使用户能够充分利用该环境来增强工作能力。


<center>![什么是 Azure AD Connect Health](./media/active-directory-aadconnect-health/logo1.png)</center>




## 为何使用 Azure AD Connect Health

将本地目录与 Azure AD 集成可提供通用标识用于访问云和本地资源，从而提高用户的生产率。但是，这种集成带来的挑战是，必须确保此环境正常运行，才能让用户从任何设备可靠地访问本地和云中的资源。Azure AD Connect Health 提供简单的基于云的方法让你监视和深入分析用于访问 Office 365 或其他 Azure AD 应用程序的本地标识基础结构。这种方法就像在每个本地标识服务器上安装代理那样简单。

适用于 AD FS 的 Azure AD Connect Health 支持 Windows Server 2008/2008 R2 中的 AD FS 2.0，以及 Windows Server 2012/2012 R2 中的 AD FS。其中还包括为 Extranet 访问提供身份验证支持的任何 AD FS 代理或 Web 应用程序代理服务器。适用于 AD FS 的 Azure AD Connect Health 提供以下关键功能集：

- 查看警报并对其采取措施，以便可靠访问 AD FS 保护的应用程序（包括 Azure AD）
- 关键警报电子邮件通知
- 查看性能数据以确定容量规划
- AD FS 登录模式的详细视图可让你判断异常状况或建立容量规划基准

以下视频将概述 Azure AD Connect Health：





## 在 Azure 门户中开始
若要开始使用 Azure Active Directory Connect Health，请按以下步骤操作。

1. 登录到 [Microsoft Azure 门户](https://manage.windowsazure.cn/)。
2. 可以通过转到“应用商店”并进行搜索的方式来访问 Azure Active Directory Connect Health，也可以通过先选择“应用商店”，然后再选择“安全性 + 标识”的方式来访问它。
3. 在介绍性边栏选项卡（边栏选项卡是一种概述视图。你可以将边栏选项卡视为窗口或弹出式窗口）中，单击“创建”。此时会打开另一个包含目录信息的边栏选项卡。
4. 在目录边栏选项卡中，单击“创建”。如果你没有 Azure Active Directory Premium 许可证，则需获得一个才能使用 Azure AD Connect Health。有关 Azure AD Premium 的信息，请参阅“Azure AD Premium 入门”。

>[AZURE.NOTE]请记住在查看 Azure AD Connect Health 实例中的任何数据之前，你将需要在目标服务器上安装 Azure AD Connect Health 代理。若要下载 Azure AD Connect Health 代理，请通过第一个边栏选项卡选择“快速启动”和“获取工具”。你也可以直接使用下面的[链接](#download-the-agent)下载该代理。若要使用 Azure Active Directory Connect Health，请执行以下操作：



### Azure AD Connect Health 门户和服务
可以使用 Azure AD Connect Health 门户来查看警报、性能监视情况和使用情况分析。在你第一次访问 Azure AD Connect Health 之后，系统将会显示第一个边栏选项卡。你可以将边栏选项卡视为窗口。你看到的第一个边栏选项卡会显示“快速启动”、“服务”和“配置”。在屏幕快照下方是上述每个选项卡的简短说明。服务部分显示 Azure AD Connect Health 所监视的活动服务以及这些活动服务的实例。

![Azure AD Connect Health 门户](./media/active-directory-aadconnect-health/portal2.png)

- **快速启动** – 选择此项将打开“快速启动”边栏选项卡。在这里，你可以通过选择“获取工具”来下载 Azure AD Connect Health 代理，可以访问文档，还可以提供反馈。
- **Active Directory Federation Services** – 此项表示 Azure AD Connect Health 当前监视的所有 AD FS 服务。选择其中一个实例后，就会打开一个边栏选项卡，其中包含该服务实例的相关信息。该信息包括概述、属性、警报、监视情况，以及使用情况分析。
- 配置 – 使用此项可打开或关闭以下功能：
<ol>
1. 通过自动更新自动将 Azure AD Connect Health 代理更新到最新版本 - 这意味着，你将在 Azure AD Connect Health 代理发布时自动更新到最新版本的该软件。此项已默认启用。
2. 允许 Microsoft 访问你 Azure AD 目录的运行状况数据，但只能将其用于故障诊断目的 - 这意味着，如果启用此功能，Microsoft 将能够查看你所看到的数据。这有助于进行故障诊断，并帮助你解决问题。此项已默认禁用。





## 要求
下表是开始使用 Azure AD Connect Health 之前必须符合的要求列表。

| 要求 | 说明|
| ----------- | ---------- |
|你必须是 Azure AD 的全局管理员才能启用（创建）Azure AD Connect Health|默认情况下，只有全局管理员才能启用（创建）、访问所有信息，以及在 Azure AD Connect Health 中执行所有操作。有关更多信息，请参阅[管理 Azure AD 目录](/documentation/articles/active-directory-administer)。<br>使用基于角色的访问控制可以允许组织中的其他用户访问 Azure AD Connect Health。有关详细信息，请参阅 [Azure AD Connect Health 的基于角色的访问控制](/documentation/articles/active-directory-aadconnect-health-operations#manage-access-with-role-based-access-control)。</br></br>重要说明：在安装代理时使用的帐户必须是工作或组织帐户，而不能是 Microsoft 帐户。有关详细信息，请参阅[以组织身份注册 Azure](/documentation/articles/sign-up-organization)|
|对于 AD FS，必须启用 AD FS 审核才能使用使用情况分析| 如果你计划使用 AD FS 的使用情况分析，则必须启用 AD FS 审核。</br></br>请参阅[为 AD FS 启用审核](/documentation/articles/active-directory-aadconnect-health-agent-install-adfs#enable-auditing-for-ad-fs)。
|满足 Azure AD Connect Health 代理要求|参阅下表了解具体的代理要求。

下表是开始使用 Azure AD Connect Health 之前必须符合的代理要求列表。

| 要求 | 说明|
| ----------- | ---------- |
|Azure AD Connect Health 代理已安装在每台目标服务器上| Azure AD Connect Health 要求在目标服务器上安装代理，以提供可在门户中查看的数据。</br></br>例如，若要获取 AD FS 本地基础结构的相关数据，必须将代理安装在 AD FS 服务器上。这包括 AD FS 代理服务器和 Web 应用程序代理服务器。</br></br>有关安装代理的信息，请参阅 [Azure AD Connect Health 代理安装](/documentation/articles/active-directory-aadconnect-health-agent-install)。</br></br>重要说明：在安装代理时使用的帐户必须是工作或组织帐户，而不能是 Microsoft 帐户。有关详细信息，请参阅[以组织身份注册 Azure](/documentation/articles/sign-up-organization)|
|Azure 服务终结点的出站连接|在安装期间和运行时，代理需要连接到下面列出的 Azure AD Connect Health 服务终结点。如果你阻止了出站连接，请确保在允许列表中添加以下项：</br></br><li>**new**: &#42;.blob.core.windows.net </li><li>**new**: &#42;.queue.core.windows.net</li><li>&#42;.servicebus.windows.net - Port: 5671</li><li>https://&#42;.adhybridhealth.azure.com/</li><li>https://&#42;.table.core.windows.net/</li><li>https://policykeyservice.dc.ad.msft.net/</li><li>https://login.windows.net</li><li>https://login.microsoftonline.com</li><li>https://secure.aadcdn.microsoftonline-p.com</li> |
|运行代理的服务器上的防火墙端口。| 为了使代理能够与 Azure AD Health 服务终结点通信，代理要求打开以下防火墙端口。</br></br><li>TCP/UDP 端口 80</li><li>TCP/UDP 端口 443</li><li>TCP/UDP 端口 5671</li>
|如果启用了 IE 增强安全性，请允许以下网站|如果在要安装代理的服务器上启用了 IE 增强安全性，则必须允许以下网站。</br></br><li>https://login.microsoftonline.com</li><li>https://secure.aadcdn.microsoftonline-p.com</li><li>https://login.windows.net</li><li>Azure Active Directory 信任的组织联合服务器。例如：https://sts.contoso.com</li>
----------------------------------------------------------------------------------------------------------


##下载 Azure AD Connect Health 代理

若要开始使用 Azure AD Connect Health，可在此处下载最新版本的代理：[下载 Azure AD Connect Health 代理](http://go.microsoft.com/fwlink/?LinkID=518973)。 在安装代理之前，请确保已从应用商店添加了该服务。


## 相关链接

* [适用于 AD FS 的 Azure AD Connect Health 代理安装](/documentation/articles/active-directory-aadconnect-health-agent-install-adfs)
* [Azure AD Connect Health 操作](/documentation/articles/active-directory-aadconnect-health-operations)
* [在 AD FS 中使用 Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health-adfs)
* [Azure AD Connect Health 常见问题](/documentation/articles/active-directory-aadconnect-health-faq)
 

<!---HONumber=67-->