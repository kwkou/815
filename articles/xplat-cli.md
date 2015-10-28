<properties
	pageTitle="适用于 Mac、Linux 和 Windows 的 Azure CLI"
	description="安装和配置适用于 Mac、Linux 和 Windows 的 Azure CLI 来管理 Azure 服务"
	editor="tysonn"
	manager="timlt"
	documentationCenter=""
	authors="squillace"
	services=""/>

<tags
	ms.service="multiple"
	ms.date="03/10/2015"
	wacn.date="10/3/2015"/>

# 安装和配置 Azure CLI

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/powershell-install-configure)
- [Azure CLI](/documentation/articles/xplat-cli)

Azure CLI 提供了一组开源且跨平台的命令，这些命令可以用于 Azure 平台。该 Azure CLI 提供了很多与 Azure 管理门户中提供的功能相同的功能，例如用于管理网站、虚拟机、移动服务、SQL 数据库以及 Azure 平台提供的其他服务的功能。

Azure CLI 以 JavaScript 编写，并且需要 Node.js。它是使用 Azure SDK for Node.js 实现的，并根据 Apache 2.0 许可证发布。项目存储库位于 [https://github.com/azure/azure-xplat-cli](https://github.com/azure/azure-xplat-cli)。

本文将介绍如何安装和配置适用于 Mac、Linux 和 Windows 的 Azure CLI，以及如何使用它通过 Azure 平台执行基本任务。

<a id="install"></a>
## 如何安装 Azure CLI

若要了解 Azure CLI 的安装步骤，请阅读[安装 Azure CLI](/documentation/articles/xplat-cli-install) 页。


<a id="configure"></a>
## 如何连接到 Azure 订阅

尽管 Azure CLI 提供的某些命令无需 Azure 订阅也可以生效，但大多数命令都要求订阅。若要了解如何配置 Azure CLI 以使用您的订阅，请访问[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect)。


<a id="use"></a>
## 如何使用 Azure CLI

可以使用 `azure` 命令访问 Azure CLI。若要查看可用命令的列表，请使用 `azure` 命令且不带任何参数。您应该看到如下帮助信息：

	info:             _    _____   _ ___ ___
	info:            /_\  |_  / | | | _ \ __|
	info:      _ ___/ _ \__/ /| |_| |   / _|___ _ _
	info:    (___  /_/ \_\/___|\___/|_|_\___| _____)
	info:       (_______ _ _)         _ ______ _)_ _
	info:              (______________ _ )   (___ _ _)
	info:
	info:    Microsoft Azure: Microsoft's Cloud Platform
	info:
	info:    Tool version 0.8.10
	help:
	help:    Display help for a given command
	help:      help [options] [command]
	help:
	help:    Opens the portal in a browser
	help:      portal [options]
	help:
	help:    Commands:
	help:      account        Commands to manage your account information and publish settings
	help:      config         Commands to manage your local settings
	help:      hdinsight      Commands to manage your HDInsight accounts
	help:      mobile         Commands to manage your Mobile Services
	help:      network        Commands to manage your Networks
	help:      sb             Commands to manage your Service Bus configuration
	help:      service        Commands to manage your Cloud Services
	help:      site           Commands to manage your Web Sites
	help:      sql            Commands to manage your SQL Server accounts
	help:      storage        Commands to manage your Storage objects
	help:      vm             Commands to manage your Virtual Machines
	help:
	help:    Options:
	help:      -h, --help     output usage information
	help:      -v, --version  output the application version

上面列出的顶级命令包含用于使用 Azure 特定领域的命令。例如，`azure account` 命令包含与 Azure 订阅相关的命令，如之前使用的 `download` 和 `import` 设置。有关可用的命令和选项的详细信息，请参阅 [使用适用于 Mac、Linux 和 Windows 的 Azure CLI]。

大多数命令的格式为 `azure <command> <operation> [parameters]`，并且对服务或对象（如帐户配置）执行操作。其他命令提供子命令并且遵循格式 `azure <command> <subcommand> <operation> [parameters]`。使用帐户配置的示例命令如下：

* 若要查看您已导入的订阅，请使用：

		azure account list

* 如果您已导入了订阅，请使用以下命令将其中一个订阅设置为默认订阅：

		azure account set <subscription>

可以使用 `--help` 或 `-h` 参数来查看特定命令的帮助。或者，也可以使用 `azure help [command] [options]` 格式返回相同信息。例如，以下命令都返回相同信息：

	azure account set --help

	azure account set -h

	azure help account set

当您对某一命令所需的参数有疑问时，请使用 `--help`、`-h` 或 `azure help [command]` 来查看帮助。

### 设置配置模式

Azure CLI 可用于对各个_资源_（即用户管理的实体，如数据库服务器、数据库或网站）执行管理操作。这是 Azure CLI 的默认操作模式，称为“**Azure 服务管理**”。但是，当您有一个包含多个资源的复杂解决方案时，能够将整个解决方案作为单个单元进行管理很有用。

要支持将一组资源作为单个逻辑单元进行管理，或管理_资源组_，我们发布了**资源管理器**的预览版，作为管理 Azure 资源的新方法。

>[AZURE.NOTE]资源管理器当前为预览版，不能提供与 Azure 服务管理相同级别的管理功能。

为了支持新的资源管理器，Azure CLI 允许您使用 `azure config mode` 命令在这些管理模式之间切换。

Azure CLI 默认使用 Azure 服务管理模式。要切换到 Resource Manager 模式，请使用以下命令来启用命令：

	azure config mode arm

要更改回 Azure 服务管理模式，请使用以下命令：

	azure config mode asm

>[AZURE.NOTE]资源管理器模式与 Azure 服务管理模式互斥。即在一种模式下创建的资源不能从另一种模式进行管理。

有关通过 Azure CLI 使用资源管理器的详细信息，请参阅 [结合使用 Azure CLI 和资源管理器][/documentation/articles/cliarm]。

### 使用 Azure 服务管理模式下的服务

借助 Azure CLI，您可以轻松地管理 Azure 服务。在这个示例中，您将了解如何使用 Azure CLI 管理 Azure 网站。

1. 使用以下命令创建一个新的 Azure 网站将 **mywebsite** 替换为唯一名称。

		azure site create mywebsite

	系统将提示您指定将在其中创建网站的区域。选择在地理上靠近您的区域。完成此命令后，将在 http://mywebsite.azurewebsites.net （将 **mywebsite** 替换为您指定的名称）上提供网站。

	> [AZURE.NOTE]如果您使用 Git 进行项目源代码管理，则可以指定 `--git` 参数以便在 Azure 上为此网站创建一个 Git 存储库。这还将在该命令从其运行的目录中创建一个 Git 存储库，如果该存储库尚不存在。它还将创建一个名为 __azure__ 的 Git remote，用于通过 `git push azure master` 命令将部署推送到 Azure 网站。

	> [AZURE.NOTE]如果您收到提示“站点”不是 Azure 命令的错误，则 Azure CLI 很可能不是资源组模式。要更改回为资源模式，请使用 `azure config mode asm` 命令。

2. 使用以下命令可为您的订阅列出网站：

		azure site list

	该列表应包含在之前步骤中创建的网站。

2. 使用以下命令可停止网站。将 **mywebsite** 替换为站点名称。

		azure site stop mywebsite

	在命令执行完毕后，您可以刷新浏览器以便验证该网站已停止。

3. 使用以下命令可启动该网站。将 **mywebsite** 替换为站点名称。

		azure site start mywebsite

	在命令执行完毕后，您可以刷新浏览器以便验证该网站已启动。

4. 使用以下命令可删除该网站。将 **mywebsite** 替换为站点名称。

		azure site delete mywebsite

	在命令执行完毕后，使用 `azure site list` 命令确认网站是否已不再存在。

<a id="script"></a>
## 如何撰写适用于 Mac、Linux 和 Windows 的 Azure CLI 的脚本

尽管您可以使用 Azure CLI 手动发布命令，但也可以通过利用命令行解释程序的功能以及系统上提供的其他命令行实用工具，来创建复杂的自动化工作流。例如，以下命令将停止所有正在运行的 Azure 网站：

	azure site list | grep 'Running' | awk '{system("azure site stop "$2)}'

该示例将网站列表传输到 `grep` 命令，该命令将检查每一行是否有字符串“Running”。然后将匹配的任何行传输到 `awk` 命令，该命令将调用 `azure site stop` 并使用传递给它的第二列（正在运行的站点名称）作为要停止的站点名称。

尽管此过程演示了如何将命令链接在一起，但您也可以使用命令行解释程序提供的脚本撰写功能创建更复杂的脚本。不同的命令行解释程序具体不同的脚本撰写功能和语法。Bash 可能是针对基于 UNIX 的系统（包括 Linux 和 OS X）的最常用的命令行解释程序。

有关使用 Bash 撰写脚本的信息，请参阅[高级 Bash 脚本撰写指南][advanced-bash]。

有关撰写基于 OS X 或 Linux 的系统的脚本的更常规的信息，请参阅 [Shell 脚本][script]。

有关使用批处理文件撰写基于 Windows 的系统的脚本的信息，请参阅[命令行参考 A-Z][batch]。

### 理解结果

在创建脚本时，您通常需要捕获命令输出，并且或者将此输出传递给其他命令，或者对此输出执行某一操作，例如检查特定值。Azure CLI 将输出生成到 STDOUT 和 STDERR。每一行都以字符串 `info:` 为前缀以便用于信息状态消息，或者以 `data:` 为前缀以便用于针对某一服务返回的数据；但是，您可以通过使用 `--verbose` 或 `-v` 参数指示 Azure CLI 返回更详细信息。此操作将返回以字符串 `verbose:` 为前缀的其他信息。

例如，从 `azure site list` 命令返回以下输出：

	info:    Executing command site list
	+ Enumerating sites
	data:    Name           Status   Mode  Host names
	data:    -------------  -------  ----  -------------------------------
	data:    myawesomesite  Running  Free  myawesomesite.chinacloudsites.cn
	info:    site list command OK

如果指定了 `--verbose` 或 `-v` 参数，将返回以下类似信息：

	info:    Executing command site list
	verbose: Subscription ####################################
	verbose: Enumerating sites
	verbose: [
	verbose:     {
	verbose:         ComputeMode: 'Shared',
	verbose:         HostNameSslStates: {
	verbose:             HostNameSslState: [
	verbose:                 {
	verbose:                     IpBasedSslResult: {
	verbose:                         $: { i:nil: 'true' }
	verbose:                     },
	verbose:                     SslState: 'Disabled',
	verbose:                     ToUpdateIpBasedSsl: {
	verbose:                         $: { i:nil: 'true' }
	verbose:                     },
	verbose:                     VirtualIP: {
	verbose:                         $: { i:nil: 'true' }
	verbose:                     },
	verbose:                     Thumbprint: {
	verbose:                         $: { i:nil: 'true' }
	verbose:                     },
	verbose:                     ToUpdate: {
	verbose:                         $: { i:nil: 'true' }
	verbose:                     },
	verbose:                     Name: 'myawesomesite.chinacloudsites.cn'
	verbose:                 },
	...
	verbose:     }
	verbose: ]
	data:    Name           Status   Mode  Host names
	data:    -------------  -------  ----  -------------------------------
	data:    myawesomesite  Running  Free  myawesomesite.chinacloudsites.cn
	info:    site list command OK

请注意，`verbose:` 信息显示为 JSON 格式的数据。如果您在使用本身理解 JSON 的实用工具，例如 [jsawk](https://github.com/micha/jsawk) 或 [jq](http://stedolan.github.io/jq/)，则可以使用 `--json` 参数返回 JSON 格式的信息。例如：

	azure site list --json | jsawk -n 'out(this.Name)' | xargs -L 1 azure site delete -q

上面的命令将网站列表作为 JSON 检索，然后使用 jsawk 检索网站名称，并且最后使用 xargs 为每个网站运行站点删除命令，并且将网站名称作为参数传递。

>[AZURE.NOTE]`--json` 参数将阻止生成状态或数据信息（以 `info:` 和 `data:` 为前缀的字符串）。例如，如果将 `--json` 参数用于 `azure site create`，则不会返回任何输出，因为此命令不返回 `info:` 以外的任何数据。

### 处理错误

尽管 Azure CLI 将错误信息记录到 STDERR，但与错误有关的其他信息也可以记录到用于运行脚本的目录下的 **azure.err** 文件中。如果信息记录到此文件，则以下内容将写入 STDOUT：

	info:    Error information has been recorded to azure.err

### 退出状态

如果缺少必需的参数，则某些 Azure CLI 命令将不会返回非零退出状态。这些命令而是将会提示用户输入。例如，在使用 `azure site create` 命令创建新网站时，如果没有指定站点名称或 `--location` 参数，系统将提示您提供这些值。

如果您在撰写依赖于退出状态的脚本，请确认您在使用的 Azure CLI 命令不会提示用户输入。

<a id="additional-resources">

## 其他资源

* [使用带服务管理的 Azure CLI][Using the Azure CLI]

* [将 Azure CLI 用于资源管理器][cliarm]

* 有关 Azure CLI、下载源代码、报告问题或贡献项目的详细信息，请访问[适用于 Azure CLI 的 GitHub 存储库](https://github.com/azure/azure-xplat-cli)。

* 如果你在使用 Azure CLI 或 Azure 时遇到问题，请访问 [Azure 论坛](http://social.msdn.microsoft.com/Forums/windowsazure/home)。

* 有关 Azure 的详细信息，请参阅 [http://www.windowsazure.cn/](http://www.windowsazure.cn)。




[mac-installer]: http://go.microsoft.com/fwlink/?LinkId=252249
[windows-installer]: http://go.microsoft.com/fwlink/?LinkID=275464&clcid=0x409
[authandsub]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh531793.aspx#BKMK_AccountVCert

[Azure Web Site]: ../media/freetrial.png
[select a preview feature]: ../media/antares-iaas-preview-02.png
[select subscription]: ../media/antares-iaas-preview-03.png
<!--[free-trial]: http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A7171371E-->
[advanced-bash]: http://tldp.org/LDP/abs/html/
[script]: http://en.wikipedia.org/wiki/Shell_script
[batch]: http://technet.microsoft.com/zh-cn/library/bb490890.aspx
[xplatarm]: /zh-cn/documentation/articles/xplat-cli-azure-resource-manager
[portal]: https://manage.windowsazure.cn
[signuporg]: http://azure.microsoft.com/zh-cn/documentation/articles/sign-up-organization/
[Using the Azure CLI]: /documentation/articles/virtual-machines-command-line-tools
<!---HONumber=71-->