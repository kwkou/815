<!-- Remove authandsub -->
<properties
	pageTitle="通过 CLI 登录到 Azure | Azure"
	description="从适用于 Mac、Linux 和 Windows 的 Azure 命令行界面 (Azure CLI) 连接到 Azure 订阅"
	editor="tysonn"
	manager="timlt"
	documentationCenter=""
	authors="dlepow"
	services=""
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="multiple"
	ms.date="04/08/2016"
	wacn.date="06/20/2016"/>

# 从 Azure 命令行界面 (Azure CLI) 连接到 Azure 订阅

Azure CLI 是一组开源且跨平台的命令，可以用于 Azure 平台。本文介绍如何通过 Azure CLI 连接到 Azure 订阅以使用所有 CLI 命令。如果你尚未安装 CLI，请参阅[安装 Azure CLI](/documentation/articles/xplat-cli-install/)。



通过 Azure CLI 有两种方式可以连接到你的订阅：

* **使用工作或学校帐户或 Microsoft 帐户标识登录到 Azure** - 这可以使用其中任一种帐户标识进行身份验证。当前 CLI 还支持对已启用多重身份验证的帐户进行交互式身份验证。以交互方式登录后，你可以使用资源管理器或经典（服务管理）命令。

* **下载并使用发布设置文件** - 这将在本地计算机上安装证书，只要订阅和证书有效，你就可以执行管理任务。此方法只允许你使用经典（服务管理）命令。

<!-- 有关身份验证和订阅管理的详细信息，请参阅[基于帐户的身份验证和基于证书的身份验证之间的区别是什么][authandsub]。 -->

如果你没有 Azure 帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用][free-trial]。

>[AZURE.NOTE] 如果你使用的是早于版本 0.9.10 的 Azure CLI 版本，则只能将 `azure login -e AzureChinaCloud` 命令用于工作或学校帐户标识，对于 Microsoft 帐户标识不起作用。但是，使用 Azure CLI 版本 0.9.10 及更高版本，你可以通过交互式 `azure login -e AzureChinaCloud` 命令使用任何标识登录到你的帐户。
>
CLI 版本 0.9.9 及更高版本支持多重身份验证。



## 使用交互式登录方法

使用 `azure login -e AzureChinaCloud` 命令（不带任何参数）- 使用以下任一标识以交互方式进行身份验证：

- 工作或学校帐户标识（需要多重身份验证）或
- Microsoft 帐户标识（如果你想要访问资源管理器部署模式功能）

> [AZURE.NOTE]  在这两种情况下，都将使用 Azure Active Directory 进行身份验证和授权。如果你使用 Microsoft 帐户标识，登录过程将访问你的 Azure Active Directory 默认域。（如果你注册了免费试用版，你可能未意识到 Azure Active Directory 为你的帐户创建了默认域。）

以交互方式登录非常简单：键入 `azure login -e AzureChinaCloud` 并按照提示进行操作，如下所示：

	azure login -e AzureChinaCloud
	info:    Executing command login
	info:    To sign in, use a web browser to open the page http://aka.ms/devicelogin. Enter the code XXXXXXXXX to authenticate. If you're signing in as an Azure AD application, use the --username and --password parameters.

复制上面提供给你的代码，并打开浏览器访问 http://aka.ms/devicelogin。
输入代码，然后系统会提示你为要使用的标识输入用户名和密码。该过程完成后，命令行解释器将完成登录过程。它的外观可能如下：

	info:    Added subscription Visual Studio Ultimate with MSDN
	info:    Added subscription Azure Free Trial
	info:    Setting subscription "Visual Studio Ultimate with MSDN" as default
	+
	info:    login command OK

## 使用非交互式登录（使用工作或学校帐户）


非交互式登录方法仅适用于工作或学校帐户，也称为*组织帐户*。此帐户由你的组织管理，并在组织的 Azure Active Directory 中定义。你可以[创建组织帐户](#create-an-organizational-account)（如果你没有），也可以[基于 Microsoft 帐户 ID 创建工作或学校 ID](/documentation/articles/virtual-machines-windows-create-aad-work-id/)。这需要在 `azure login -e AzureChinaCloud` 命令中指定用户名或用户名和密码，如下所示：

	azure login -e AzureChinaCloud -u ahmet@contoso.partner.onmschina.cn
	info:    Executing command login
	Password: *********
	|info:    Added subscription Visual Studio Ultimate with MSDN
	+
	info:    login command OK

在系统提示时输入你的密码。

如果这是你首次使用这些凭据登录，系统将要求你确认是否希望缓存身份验证令牌。如果你以前使用了 `azure logout` 命令（下面进行说明），也会出现此提示。若要为自动化方案避开此提示，请运行带有 `-q` 参数的 `azure login -e AzureChinaCloud`。

* **若要注销**，请使用以下命令：

		azure logout -u <username>

	如果与帐户关联的订阅只使用 Active Directory 进行身份验证，则注销将从本地配置文件中删除订阅信息。但是，如果还为订阅导入了发布设置文件，则注销将仅从本地配置文件中删除与 Active Directory 相关的信息。

## 使用发布设置文件方法

如果你只需要使用经典（服务管理）CLI 命令，则可以使用发布设置文件进行连接。

* **若要为你的帐户下载发布设置文件**，请使用以下命令：

		azure account download

这将会打开默认浏览器，并提示你登录到 [Azure 门户][portal]。登录后，将下载 `.publishsettings` 文件。记下此文件的保存位置。

> [AZURE.NOTE] 如果你的帐户与多个 Azure Active Directory 租户关联，系统将提示你选择要为哪个 Active Directory 下载发布设置文件。
>
> 使用下载页面进行选择或通过访问 Azure 门户进行选择之后，所选的 Active Directory 将成为门户和下载页面使用的默认值。 设立默认值之后，你将在下载页面的顶部看到文本“若要返回选择页面，请单击此处”。使用提供的链接返回选择页面。

* **若要导入发布设置文件**，请运行以下命令：

		azure account import <path to your .publishsettings file>

	在导入发布设置后，应该删除 `.publishsettings` 文件，因为 Azure CLI 不再需要该文件，并且该文件会产生安全风险，因为它可以用来访问你的订阅。


## 多个订阅

如果你有多个 Azure 订阅，则连接到 Azure 即有权访问与凭据关联的所有订阅。将选择一个订阅作为默认订阅，由 Azure CLI 在执行操作时使用。你可以查看这些订阅以及哪个订阅是默认订阅，使用 `azure account list` 命令。此命令将返回如下信息：

	info:    Executing command account list
	data:    Name              Id                                    Current
	data:    ----------------  ------------------------------------  -------
	data:    Azure-sub-1       ####################################  true
	data:    Azure-sub-2       ####################################  false

在上面的列表中，**“当前”**列指示当前的默认订阅为 Azure-sub-1。若要更改默认订阅，请使用 `azure account set` 命令，并指定你想要其成为默认订阅的订阅。例如：

	azure account set Azure-sub-2

这会将默认订阅更改为 Azure-sub-2。

> [AZURE.NOTE] 更改默认订阅将会立即生效，并且是全局更改；新的 Azure CLI 命令（无论是从同一命令行实例还是其他实例运行的）将使用新的默认订阅。

如果要将非默认订阅用于 Azure CLI，但不想更改当前默认订阅，则可以将 `--subscription` 选项用于命令并提供要用于此操作的订阅名称。

一旦连接到你的 Azure 订阅，即可使用 Azure CLI 命令。

## CLI 设置的存储

无论你是使用工作或学校帐户登录，还是导入发布设置，CLI 配置文件和日志都存储在 `.azure` 目录（位于你的 `user` 目录中）中。你的 `user` 目录受到操作系统的保护；但是，建议你采取附加措施对你的 `user` 目录进行加密。你可通过以下方式完成此操作：

* 在 Windows 上，修改目录属性或使用 BitLocker。
* 在 Mac 上，为目录启用 FileVault。
* 在 Ubuntu 上，使用“加密主目录”功能。其他 Linux 分发提供了类似功能。

## 其他资源

* [将 Azure CLI 与服务管理（经典）命令配合使用][cliasm]

* 若要了解有关 Azure CLI、下载源代码、报告问题或贡献项目的详细信息，请访问[适用于 Azure CLI 的 GitHub 存储库](https://github.com/azure/azure-xplat-cli)。

* 如果你在使用 Azure CLI 或 Azure 时遇到问题，请访问 [Azure 论坛](http://social.msdn.microsoft.com/Forums/windowsazure/home)。





[authandsub]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh531793.aspx#BKMK_AccountVCert
[free-trial]: /pricing/1rmb-trial/
[portal]: https://manage.windowsazure.cn
[signuporg]: /documentation/articles/sign-up-organization/
[cliasm]: /documentation/articles/virtual-machines-command-line-tools/

<!---HONumber=Mooncake_0314_2016-->