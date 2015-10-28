<properties
	pageTitle="从 Azure 命令行界面 (Azure CLI) 登录 | Windows Azure"
	description="从 Azure 命令行界面 (Azure CLI) 连接到 Azure 订阅"
	editor="tysonn"
	manager="timlt"
	documentationCenter=""
	authors="dlepow"
	services=""/>

<tags
	ms.service="multiple"
	ms.date="06/09/2015"
	wacn.date="08/29/2015"/>

# 从 Azure 命令行界面 (Azure CLI) 连接到 Azure 订阅

Azure CLI 是一组开源且跨平台的命令，可以用于 Azure 平台。本文档介绍如何从 Azure CLI 连接到你的 Azure 订阅。有关安装说明，请参阅[安装 Azure CLI](/documentation/articles/xplat-cli-install)。

<a id="configure"></a>
## 如何连接到 Azure 订阅

Azure CLI 提供的大多数命令需要连接到 Azure 帐户。可以使用两种方式来配置 Azure CLI ，使其适用于你的订阅：

* 使用工作帐户或学校帐户（也称为组织帐户）登录 Azure。此时系统会使用 Azure Active Directory 来验证凭据。

或

* 下载并使用发布设置文件。此时会安装证书，有了证书，你就可以执行管理任务，前提是订阅和证书有效。

有关身份验证和订阅管理的详细信息，请参阅[“基于帐户的身份验证和基于证书的身份验证之间的区别是什么”][authandsub]。

如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用][free-trial]。

> [AZURE.NOTE]最灵活且最高级的 Azure 服务使用 Azure 资源管理器或 [ARM 模式](/documentation/articles/xplat-cli-azure-resource-manager)。ARM 模式需要使用工作或学校帐户连接到 Azure，所用登录方法介绍如下。

### 使用 login 方法

登录方法仅适用于工作或学校帐户。此帐户由你的组织管理，并在组织的 Azure Active Directory 中定义。

> [AZURE.NOTE]如果你当前没有工作或学校帐户，且已使用个人帐户登录到 Azure 订阅，则你可以按照以下步骤轻松地创建一个工作或学校帐户。
>
> 1. 登录到 [Azure 门户][portal]，然后选择 **Active Directory**。
>
> 2. 如果目录不存在，请选择“创建目录”，并提供所请求的信息。
>
> 3. 选择目录，并添加新用户。这个新用户是工作或学校帐户。
>
>     创建用户时，系统将为你提供用户电子邮件地址和临时密码。保存此信息，因为稍后需要它。
>
> 4. 从管理门户中，选择“设置”，然后选择“管理员”。选择“添加”，并将新用户添加为共同管理员。这样工作或学校帐户即可管理 Azure 订阅。
>
> 5. 最后，从 Azure 门户注销，然后使用新的工作或学校帐户重新登录。如果这是使用此帐户首次登录，系统将提示更改密码。
>
> 6. 请确保你在使用工作或学校帐户登录后能够看到你的订阅。
>
>有关工作或学校帐户的详细信息，请参阅[以组织身份注册 Windows Azure][signuporg]。

若要从 Azure CLI 使用工作或学校帐户登录，请使用以下命令：

	azure login -u <username>

并在系统提示时输入你的密码。

> [AZURE.NOTE]如果这是你首次使用这些凭据登录，你将收到一个提示，要求你确认是否要缓存身份验证令牌。如果你之前使用过如下所述的 `azure logout` 命令，也会出现此提示。若要使自动化方案绕过此提示，请使用带有 `azure login` 命令的 `-q` 参数。

若要注销，请使用以下命令：

	azure logout -u <username>

> [AZURE.NOTE]如果与帐户关联的订阅只使用 Active Directory 进行身份验证，则注销将从本地配置文件中删除订阅信息。但是，如果还为订阅导入了发布设置文件，则注销将仅从本地配置文件中删除与 Active Directory 相关的信息。

### 使用发布设置文件方法

> [AZURE.NOTE]如果使用此方法进行连接，则只能使用 Azure 服务管理（或 ASM 模式）命令。

若要下载针对你的帐户的发布设置，请使用以下命令：

	azure account download

此操作将打开默认浏览器，并提示你登录到 [Azure 门户][portal]。登录后，将会下载一个 `.publishsettings` 文件。记下此文件的保存位置。

> [AZURE.NOTE]如果你的帐户与多个 Azure Active Directory 租户关联，系统将提示你选择要为哪个 Active Directory 下载发布设置文件。
>
> 使用下载页面进行选择或通过访问 Azure 门户进行选择之后，所选的 Active Directory 将成为门户和下载页面使用的默认值。设立默认值之后，你将在下载页面的顶部看到文本“__若要返回选择页面，请单击此处__”。使用提供的链接返回选择页面。

接下来，通过运行以下命令导入 `.publishsettings` 文件：

	azure account import <path to your .publishsettings file>

在导入发布设置后，应该删除 `.publishsettings` 文件，因为 Azure CLI 不再需要该文件，并且该文件会产生安全风险，因为它可以用来访问你的订阅。

> [AZURE.NOTE]无论你是使用工作或学校帐户登录，还是导入发布设置，用于访问你的 Azure 订阅的信息都存储在 `.azure` 目录（位于你的 `user` 目录中）。你的 `user` 目录受到操作系统的保护；但是，建议你采取附加措施对你的 `user` 目录进行加密。你可通过以下方式完成此操作：
>
> * 在 Windows 上，修改目录属性或使用 BitLocker。
> * 在 Mac 上，为目录启用 FileVault。
> * 在 Ubuntu 上，使用“加密主目录”功能。其他 Linux 分发提供了等效功能。

### 多个订阅

如果你有多个 Azure 订阅，则连接到 Azure 即有权访问与凭据关联的所有订阅。将选择一个订阅作为默认订阅，由 Azure CLI 在执行操作时使用。你可以查看这些订阅以及哪个订阅是默认订阅，使用 `azure account list` 命令。此命令将返回如下信息：

	info:    Executing command account list
	data:    Name              Id                                    Current
	data:    ----------------  ------------------------------------  -------
	data:    Azure-sub-1       ####################################  true
	data:    Azure-sub-2       ####################################  false

在上面的列表中，“当前”列指示当前的默认订阅为 Azure-sub-1。若要更改默认订阅，请使用 `azure account set` 命令，并指定你想要其成为默认订阅的订阅。例如：

	azure account set Azure-sub-2

这会将默认订阅更改为 Azure-sub-2。

> [AZURE.NOTE]更改默认订阅将会立即生效，并且是全局更改；新的 Azure CLI 命令（无论是从相同命令行实例还是其他实例运行的）将使用新的默认订阅。

如果要将非默认订阅用于 Azure CLI，但不想更改当前默认订阅，则可以将 `--subscription` 选项用于命令并提供要用于此操作的订阅名称。

一旦连接到你的 Azure 订阅，即可使用 Azure CLI 命令。有关详细信息，请参阅[如何使用 Azure CLI](/documentation/articles/xplat-cli)。

<a id="additional-resources">
## 其他资源

* [使用带服务管理（或 ASM 模式）命令的 Azure CLI][cliasm]

* [使用带资源管理（或 ARM 模式）命令的 Azure CLI][cliarm]

* 有关 Azure CLI、下载源代码、报告问题或贡献项目的详细信息，请访问[适用于 Azure CLI 的 GitHub 存储库](https://github.com/azure/azure-xplat-cli)。

* 如果你在使用 Azure CLI 或 Azure 时遇到问题，请访问 [Azure 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home)。

* 有关 Azure 的详细信息，请参阅 [http://www.windowsazure.cn/](http://www.windowsazure.cn)。





[authandsub]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh531793.aspx#BKMK_AccountVCert
[free-trial]: http://www.windowsazure.cn/pricing/1rmb-trial/
[portal]: https://manage.windowsazure.cn
[signuporg]: /documentation/articles/sign-up-organization
[cliasm]: /documentation/articles/virtual-machines-command-line-tools
[cliarm]: /documentation/articles/xplat-cli-azure-resource-manager
<!---HONumber=67-->