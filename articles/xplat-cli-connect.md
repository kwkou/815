<properties
	pageTitle="通过 CLI 登录到 Azure | Azure"
	description="从适用于 Mac、Linux 和 Windows 的 Azure 命令行界面 (Azure CLI) 连接到 Azure 订阅"
	editor="tysonn"
	manager="timlt"
	documentationCenter=""
	authors="dlepow"
	services="virtual-machines-linux,virtual-network,storage,azure-resource-manager"
	tags="azure-resource-manager,azure-service-management"/>  


<tags
	ms.service="multiple"
	ms.workload="multiple"
	ms.tgt_pltfrm="vm-multiple"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/04/2016"
	wacn.date="11/28/2016"
	ms.author="danlep"/>


# 通过 Azure CLI 登录 Azure

Azure CLI 是一组开源且跨平台的命令，可以用于 Azure 资源。本文介绍提供 Azure 帐户凭据以将 Azure CLI 连接到 Azure 订阅的不同方式：

* 运行 `azure login` CLI 命令，通过 Azure Active Directory 进行身份验证。借助此方法，可以在两个[命令模式](#CLI-command-modes)下访问 CLI 命令。运行命令时，如果没有其他选项，`azure login` 会提示通过 Web 门户继续以交互方式登录。有关其他 `azure login` 命令选项，请参阅本文中的方案，或键入 `azure login --help`。

* 如果只需使用 Azure 服务管理模式 CLI 命令（不建议用于大部分新部署），可以在计算机上下载并安装发布设置文件。

如果你尚未安装 CLI，请参阅[安装 Azure CLI](xplat-cli-install.md)。如果没有 Azure 订阅，只需要花费几分钟就能创建一个[免费帐户](http://azure.microsoft.com/free/)。

有关不同帐户的标识和 Azure 订阅的背景信息，请参阅[Azure 订阅如何与 Azure Active Directory 相关联](/documentation/articles/active-directory-how-subscriptions-associated-directory/)。






## 方案 1：使用交互式登录方法登录 Azure 

使用特定帐户时，CLI 会要求运行 `azure login`，然后通过 Web 门户使用 Web 浏览器继续完成登录过程，此过程称为*交互式登录*。常见原因是，拥有设置为要求多重身份验证的工作或学校帐户（也称为*组织帐户*）。想要使用 Resource Manager 模式命令时，也会使用交互式登录方法来登录 Microsoft 帐户。

交互式登录很简单：键入 `azure login` 即可，不需要任何选项，如以下示例所示：

```
azure login -e AzureChinaCloud
```                                                                                             

输出类似如下所示：

```         
info:    Executing command login
info:    To sign in, use a web browser to open the page http://aka.ms/devicelogin. Enter the code XXXXXXXXX to authenticate. 
```
复制命令输出中提供的代码，并打开浏览器访问 http://aka.ms/devicelogin（或指定的其他页面）。（可以在同一计算机上，也可以在其他计算机或设备上打开浏览器。） 输入代码，然后系统会提示你为要使用的标识输入用户名和密码。该过程完成后，命令外壳会完成登录过程。它的外观可能如下：

	info:    Added subscription Visual Studio Ultimate with MSDN
	info:    Added subscription Azure Free Trial
	info:    Setting subscription "Visual Studio Ultimate with MSDN" as default
	+
	info:    login command OK
    
>[AZURE.NOTE]  使用交互式登录时，会使用 Azure Active Directory 进行身份验证和授权。如果使用 Microsoft 帐户标识，登录过程会访问 Azure Active Directory 的默认域。（如果注册的是免费 Azure 帐户，Azure Active Directory 已为该帐户自动创建了默认域。）

## 方案 2：使用用户名和密码登录 Azure


想要使用不需要多重身份验证的工作或学校帐户时，使用包含用户名 (`-u`) 参数的 `azure login` 命令进行身份验证。系统会在命令行中提示输入密码（也可以选择将密码作为 `azure login` 命令的其他参数传递）。以下示例将传递组织帐户的用户名：

	azure login -e AzureChinaCloud -u myUserName@contoso.partner.onmschina.cn
	
然后，系统会提示输入密码：

	info:    Executing command login
	Password: *********
    
随后完成登录过程。

    info:    Added subscription Visual Studio Ultimate with MSDN
	+
	info:    login command OK

在系统提示时输入你的密码。

如果这是你首次使用这些凭据登录，系统将要求你确认是否希望缓存身份验证令牌。如果以前使用了 `azure logout` 命令，也会出现此提示（如本文稍后所述）。若要在自动化应用场景中绕过此提示，请使用 `-q` 参数运行 `azure login -e AzureChinaCloud`。

   

## 方案 3：使用服务主体登录 Azure

如果创建 Active Directory 应用程序的服务主体，并且服务主体拥有针对订阅的权限，就可以使用 `azure login -e AzureChinaCloud` 命令来对服务主体进行身份验证。根据方案，可以提供服务主体的凭据作为 `azure login` 命令的显式参数。例如，以下命令传递服务主体名称和 Active Directory 租户 ID：

    azure login -u https://www.contoso.org/example --service-principal --tenant myTenantID

随后，系统会提示提供密码。还可以通过 CLI 脚本或应用程序代码提供凭据，或使用证书针对自动化方案以非交互方式对服务主体进行身份验证。有关详细信息与示例，请参阅[使用 Azure Resource Manager 对服务主体进行身份验证](/documentation/articles/resource-group-authenticate-service-principal-cli/)。

## 方案 4：使用发布设置文件

如果你只需要使用 Azure 服务管理模式 CLI 命令（例如，在经典部署模型中部署 Azure VM），则可以使用发布设置文件进行连接。此方法会在本地计算机上安装证书，只要订阅和证书有效，就可以执行管理任务。

* **若要为帐户下载发布设置文件**，请通过键入 `azure config mode asm` 来确保 CLI 处于服务管理模式下。然后运行以下命令：

		azure account download

这将会打开默认浏览器，并提示你登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。登录后，将下载 `.publishsettings` 文件。记下此文件的保存位置。

>[AZURE.NOTE] 如果你的帐户与多个 Azure Active Directory 租户关联，系统将提示你选择要为哪个 Active Directory 下载发布设置文件。

使用下载页面进行选择或通过访问 Azure 经典门户进行选择之后，所选的 Active Directory 将成为经典管理门户和下载页面使用的默认值。设立默认值后，会在下载页面的顶部看到文本“__若要返回选择页面，请单击此处__”。使用提供的链接返回选择页面。

* **若要导入发布设置文件**，请运行以下命令：

		azure account import <path to your .publishsettings file>

>[AZURE.IMPORTANT]导入发布设置后，应删除 `.publishsettings` 文件。因为 Azure CLI 不再需要该文件，并且该文件会产生安全风险，因为它可以用来访问你的订阅。

<a name="CLI-command-modes"></a>
## CLI 命令模式

Azure CLI 提供两种命令模式让你使用不同的命令集来处理 Azure 资源：

* **Resource Manager 模式** - 用于在 Resource Manager 部署模型中处理 Azure 资源。若要设置此模式，请运行 `azure config mode arm`。

* **服务管理模式** - 用于在经典部署模型中处理 Azure 资源。若要设置此模式，请运行 `azure config mode asm`。

首次安装时，CLI 的当前版本处于 Resource Manager 模式下。

>[AZURE.NOTE]Resource Manager 模式与服务管理模式互斥。即，在一种模式下创建的资源不能通过另一种模式进行管理。

## 多个订阅

如果有多个 Azure 订阅，则连接到 Azure 即有权访问与凭据关联的所有订阅。将选择一个订阅作为默认订阅，由 Azure CLI 在执行操作时使用。可以使用 `azure account list` 命令查看这些订阅，包括当前默认订阅。此命令将返回如下信息：

	info:    Executing command account list
	data:    Name              Id                                    Current
	data:    ----------------  ------------------------------------  -------
	data:    Azure-sub-1       ####################################  true
	data:    Azure-sub-2       ####################################  false

在上述列表中，**当前**列指示当前的默认订阅为 Azure-sub-1。若要更改默认订阅，请使用 `azure account set` 命令，并指定你想要其成为默认订阅的订阅。例如：

	azure account set Azure-sub-2

这会将默认订阅更改为 Azure-sub-2。

> [AZURE.NOTE] 更改默认订阅将会立即生效，并且是全局更改；新的 Azure CLI 命令（无论是从同一命令行实例还是其他实例运行的）将使用新的默认订阅。

如果要将非默认订阅用于 Azure CLI，但不想更改当前默认订阅，则可以将 `--subscription` 选项用于命令并提供要用于此操作的订阅名称。

一旦连接到你的 Azure 订阅，即可使用 Azure CLI 命令来处理 Azure 资源。



## CLI 设置的存储

无论你是使用 `azure login -e AzureChinaCloud` 命令登录，还是导入发布设置，CLI 配置文件和日志都存储在 `.azure` 目录（位于你的 `user` 目录中）中。`user` 目录受操作系统保护。但是，建议采取其他措施来对 `user` 目录进行加密。你可通过以下方式完成此操作：

* 在 Windows 上，修改目录属性或使用 BitLocker。
* 在 Mac 上，为目录启用 FileVault。
* 在 Ubuntu 上，使用“加密主目录”功能。其他 Linux 分发提供了类似功能。

## 注销

若要注销，请使用以下命令：

	azure logout -u <username>

如果与帐户关联的订阅只使用 Active Directory 进行身份验证，则注销操作会从本地配置文件中删除订阅信息。但是，如果还为订阅导入了发布设置文件，则注销操作仅从本地配置文件中删除与 Active Directory 相关的信息。
## 后续步骤

* 若要使用 Azure CLI 命令，请参阅 [Resource Manager 模式中的 Azure CLI 命令](/documentation/articles/azure-cli-arm-commands/)和[服务管理模式中的 Azure CLI 命令](/documentation/articles/virtual-machines-command-line-tools/)。

* 若要了解有关 Azure CLI、下载源代码、报告问题或贡献项目的详细信息，请访问[适用于 Azure CLI 的 GitHub 存储库](https://github.com/azure/azure-xplat-cli)。

* 如果你在使用 Azure CLI 或 Azure 时遇到问题，请访问 [Azure 论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=azurescripting)。

<!---HONumber=Mooncake_1121_2016-->