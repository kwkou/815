<properties linkid="script-xplat-intro" urlDisplayName="Azure Cross-Platform Command-Line Interface" pageTitle="Azure 跨平台命令行接口" title="Azure 跨平台命令行接口" metaKeywords="Azure cross-platform command-line interface, Azure command-line, azure command-line, azure cli" description="安装和配置 Azure 跨平台命令行接口以管理 Azure 服务" metaCanonical="http://www.windowsazure.cn/zh-cn/script/xplat-cli-intro" umbracoNaviHide="0" disqusComments="1" editor="mollybos" manager="paulettm" documentationCenter="" solutions="51Aspx" authors="larryfr" services="Haifeng Liu" />
<tags ms.service="Haifeng Liu"
    ms.date="03/10/2015"
    wacn.date="04/11/2015"
    />

# 安装和配置 Azure 跨平台命令行接口

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/install-configure-powershell/" title="PowerShell">PowerShell</a><a href="/zh-cn/documentation/articles/xplat-cli/" title="跨平台 CLI" class="current">跨平台 CLI</a></div>

Azure 跨平台命令行接口 (xplat-cli) 提供了一组开源的跨平台命令以便使用 Azure 平台。该 xplat-cli 提供了很多与 Azure 管理门户中提供的功能相同的功能，例如用于管理网站、虚拟机、移动服务、SQL数据库 以及 Azure 平台提供的其他服务的功能。

该 xplat-cli 是用 JavaScript 编写的，且需要 Node.js。它是使用 Azure SDK for Node.js 实现的，且基于 Apache 2.0 许可证发布。项目存储库位于 <https://github.com/WindowsAzure/azure-sdk-tools-xplat>。

本文将介绍如何安装和配置 Azure 跨平台命令行接口，以及如何使用它通过 Azure 平台执行基本任务。

## 文档内容

-   [如何安装 Azure 跨平台命令行接口][如何安装 Azure 跨平台命令行接口]
-   [如何连接到 Azure 订阅][如何连接到 Azure 订阅]
-   [如何使用 Azure 跨平台命令行接口][如何使用 Azure 跨平台命令行接口]
-   [如何撰写 Azure 跨平台命令行接口脚本][如何撰写 Azure 跨平台命令行接口脚本]
-   [其他资源][其他资源]

## <span id="install"></span></a>如何安装 Azure 跨平台命令行接口

xplat-cli 有两种安装方法：使用适用于 Windows 和 OS X 的安装程序包；或者如果您的系统上安装了 Node.js，则使用 **npm** 命令。

对于 Linux 系统，必须安装 Node.js，并如下所述使用 **npm** 来安装 xplat-cli，或者从源代码生成 xplat-cli。<http://go.microsoft.com/fwlink/?linkid=253472&clcid=0x409> 提供了源代码。有关从源代码生成的详细信息，请参阅存档中随附的 INSTALL 文件。

安装了 xplat-cli 之后，您将可以从命令行接口（Bash、终端、命令提示符）使用 **azure** 命令访问 xplat-cli 命令。

### 使用安装程序

提供以下安装程序包：

-   [Windows 安装程序][Windows 安装程序]

-   [OS X 安装程序][OS X 安装程序]

### 使用 npm

如果 Node.js 安装在您的系统上，则使用以下命令安装 xplat-cli：

    npm install azure-cli -g

> [WACOM.NOTE] 可能需要使用 `sudo` 才能成功运行 **npm** 命令。

这将安装 xplat-cli 及所需的依赖项。在安装结束时，您应该会看到如下内容：

    azure-cli@0.8.0 ..\node_modules\azure-cli
    |-- easy-table@0.0.1
    |-- eyes@0.1.8
    |-- xmlbuilder@0.4.2
    |-- colors@0.6.1
    |-- node-uuid@1.2.0
    |-- async@0.2.7
    |-- underscore@1.4.4
    |-- tunnel@0.0.2
    |-- omelette@0.1.0
    |-- github@0.1.6
    |-- commander@1.0.4 (keypress@0.1.0)
    |-- xml2js@0.1.14 (sax@0.5.4)
    |-- streamline@0.4.5
    |-- winston@0.6.2 (cycle@1.0.2, stack-trace@0.0.7, async@0.1.22, pkginfo@0.2.3, request@2.9.203)
    |-- kuduscript@0.1.2 (commander@1.1.1, streamline@0.4.11)
    |-- azure@0.7.13 (dateformat@1.0.2-1.2.3, envconf@0.0.4, mpns@2.0.1, mime@1.2.10, validator@1.4.0, xml2js@0.2.8, wns@0.5.3, request@2.25.0)

> [WACOM.NOTE] 可以从 <http://nodejs.org/> 安装 Node.js。

## <span id="Configure"></span></a>如何连接到 Azure 订阅

尽管 xplat-cli 提供的某些命令无需 Azure 订阅也可以生效，但大多数命令都要求订阅。要配置 xplat-cli 以使用您的订阅，您可以：

-   下载并使用发布设置文件。

或者

-   使用组织帐户登录到 Azure。登录后，将使用 Azure Active Directory 对凭据进行身份验证。

为了帮助您选择适合您的需要的身份验证方法，请考虑以下方面：

-   使用 login 方法可以更轻松地管理对订阅的访问，但可能会中断自动操作，因为凭据可能会超时，并要求您重新登录。

    > [WACOM.NOTE] login 方法仅适用于组织帐户。组织帐户是指受组织管理、并在组织 Azure Active Directory 租户中定义的用户。如果您当前没有组织帐户，且已使用 Microsoft 帐户登录到 Azure 订阅，则您可以按照以下步骤轻松地创建一个组织帐户。
    >
    > 1.  登录到“Azure 管理门户”[][]，然后单击“Active Directory”。
    >
    > 2.  如果目录不存在，请选择“创建目录”，并提供所需的信息。
    >
    > 3.  选择目录，并添加新用户。此新用户即为组织帐户。
    >
    >     创建用户时，系统将为您提供用户电子邮件地址和临时密码。保存此信息，因为此信息还要用于另一个步骤。
    >
    > 4.  从管理门户中，选择“设置”，然后选择“管理员”。选择“添加”，并将新用户添加为共同管理员。这样组织帐户即可管理 Azure 订阅。
    >
    > 5.  最后，从 Azure 门户注销，然后使用新的组织帐户重新登录。如果这是使用此帐户首次登录，系统将提示更改密码。
    >
    > 有关 Windows Azure 组织帐户的详细信息，请参阅[以组织身份注册 Windows Azure][以组织身份注册 Windows Azure]。

-   只要订阅和证书有效，您就可以通过发布设置文件方法安装的证书执行管理任务。该方法可便于将自动化用于长时间运行的任务。在您下载并导入信息后，无需再次提供它。但是，使用此方法后难以管理对订阅的访问，因为可以访问证书的任何人都可以管理订阅。

有关身份验证和订阅管理的详细信息，请参阅[“基于帐户的身份验证和基于证书的身份验证之间的区别是什么”][“基于帐户的身份验证和基于证书的身份验证之间的区别是什么”]。

<!--If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see [Azure Free Trial][free-trial].-->

### 使用 login 方法

要使用组织帐户登录，请使用以下命令：

    azure login -u [username] -p [password] -e AzureChinaCloud

> [WACOM.NOTE] 如果这是您首次使用这些凭据登录，您将收到一个提示，要求您确认是否要缓存身份验证令牌。如果您之前使用过如下所述的 `azure logout` 命令，也会出现此提示。要使自动化方案绕过此提示，请使用带有 `azure login` 命令的 `-q` 参数。

> [WACOM.NOTE] 使用组织帐户进行身份验证时，访问 Azure 订阅的信息存储在位于 `user` 目录下的 `.azure` 中。您的 `user` 目录受到操作系统的保护；但是，建议您采取附加措施对您的 `user` 目录进行加密。您可通过以下方式完成此操作：

要注销，请使用以下命令：

    azure logout [username]

> [WACOM.NOTE] 如果与帐户关联的订阅只使用 Active Directory 进行身份验证，则注销将从本地配置文件中删除订阅信息。但是，如果还为订阅导入了发布设置文件，则注销将仅从本地配置文件中删除与 Active Directory 相关的信息。

> [WACOM.NOTE] 使用帐户进行身份验证时，以下命令将不起作用：
>
> -   `azure vm`
> -   `azure network`
> -   `azure mobile`
>
> 如果您需要使用这些命令，请使用发布设置文件对 Azure 进行身份验证，如下一部分所述。

### 使用发布设置文件方法

若要下载针对您的帐户的发布设置，请使用以下命令：

    azure account download -e AzureChinaCloud

此操作将打开默认浏览器，并提示您登录到 Azure 管理门户。登录后，将会下载一个 `.publishsettings` 文件。记下此文件的保存位置。

> [WACOM.NOTE] 如果您的帐户与多个 Azure Active Directory 租户关联，系统将提示您选择要为哪个 Active Directory 下载发布设置文件。
>
> 使用下载页面进行选择或通过访问 Azure 管理门户进行选择之后，所选的 Active Directory 将成为门户和下载页面使用的默认值。设立默认值之后，您将在下载页面的顶部看到文本：“要返回选择页面，请单击此处”。使用提供的链接返回选择页面。

接下来，通过运行以下命令导入 `.publishsettings` 文件，并将 `[path to .publishsettings file]` 替换为 `.publishsettings` 文件的路径：

    azure account import [path to .publishsettings file]

> [WACOM.NOTE] 导入发布设置时，用于访问 Azure 订阅的信息存储在位于 `user` 目录下的 `.azure` 目录中。您的 `user` 目录受到操作系统的保护；但是，建议您采取附加措施对您的 `user` 目录进行加密。您可通过以下方式完成此操作：
>
> -   在 Windows 上，修改目录属性或使用 BitLocker。
> -   在 Mac 上，为目录启用 FileVault。
> -   在 Ubuntu 上，使用“加密主目录”功能。其他 Linux 分发提供了等效功能。

在导入发布设置后，应该删除 `.publishsettings` 文件，因为命令行工具不再需要该文件，并且该文件会产生安全风险，因为它可以用来访问您的订阅。

### 多个订阅

如果您有多个 Azure 订阅，则登录将授予访问与凭据关联的所有订阅的权限。如果使用发布设置文件，则 `.publishsettings` 文件将包含所有订阅的信息。使用任一种方法时，都可以选择一个订阅作为 xplat-cli 执行操作时使用的默认订阅。您可以查看这些订阅以及哪个订阅是默认订阅，但使用 `azure account list` 命令。此命令将返回如下信息：

    info:    Executing command account list
    data:    Name              Id                                    Current
    data:    ----------------  ------------------------------------  -------
    data:    Azure-sub-1       ####################################  true
    data:    Azure-sub-2       ####################################  false

在上述列表中，“Current”列表示当前默认订阅为 Azure-sub-1。要更改该默认订阅，请使用 `azure account select` 命令，并且指定要作为默认订阅的订阅。例如：

    azure account select Azure-sub-2

这会将默认订阅更改为 Azure-sub-2。

> [WACOM.NOTE] 更改默认订阅的操作会立即生效，并且是全局更改；新的 xplat 命令（无论是从同一命令行实例还是其他实例运行的）将使用新的默认订阅。

如果要将非默认订阅用于 xplat-cli，但不想更改当前默认订阅，则可以使用 `--subscription` 选项并提供要用于此操作的订阅名称。

## <span id="use"></span></a>如何使用 Azure 跨平台命令行接口

可以使用 `azure` 命令访问 xplat-cli。若要查看可用命令的列表，请使用 `azure` 命令且不带任何参数。您应该看到如下帮助信息：

    info:             _    _____   _ ___ ___
    info:            /_\  |_  / | | | _ \ __|
    info:      _ ___/ _ \__/ /| |_| |   / _|___ _ _
    info:    (___  /_/ \_\/___|\___/|_|_\___| _____)
    info:       (_______ _ _)         _ ______ _)_ _
    info:              (______________ _ )   (___ _ _)
    info:
    info:    Windows Azure: Microsoft's Cloud Platform
    info:
    info:    Tool version 0.8.0
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
    help:      sb             Commands to manage your 服务总线 configuration
    help:      service        Commands to manage your Cloud Services
    help:      site           Commands to manage your Web Sites
    help:      sql            Commands to manage your SQL Server accounts
    help:      storage        Commands to manage your Storage objects
    help:      vm             Commands to manage your Virtual Machines
    help:
    help:    Options:
    help:      -h, --help     output usage information
    help:      -v, --version  output the application version

上面列出的顶级命令包含用于使用 Azure 特定领域的命令。例如，`azure account` 命令包含与 Azure 订阅相关的命令，如之前使用的 `download` 和 `import` 设置。

大多数命令的格式为 `azure <command> <operation> [parameters]`，并且对服务或对象（如帐户配置）执行操作。其他命令提供子命令并且遵循格式 `azure <command> <subcommand> <operation> [parameters]`。使用帐户配置的示例命令如下：

-   若要查看您已导入的订阅，请使用：

        azure account list

-   如果您已导入了订阅，请使用以下命令将其中一个订阅设置为默认订阅：

        azure account set <subscription>

可以使用 `--help` 或 `-h` 参数来查看特定命令的帮助。或者，也可以使用 `azure help [command] [options]` 格式返回相同信息。例如，以下命令都返回相同信息：

    azure account set --help

    azure account set -h

    azure help account set

当您对某一命令所需的参数有疑问时，请使用 `--help`、`-h` 或 `azure help [command]` 来查看帮助。

### 设置配置模式

xplat-cli 可用于对各个*资源*（即用户管理的实体，如数据库服务器、数据库或网站）执行管理操作。这是 xplat-cli 的默认操作模式，称为“Azure 服务管理”。但是，当您有一个包含多个资源的复杂解决方案时，能够将整个解决方案作为单个单元进行管理很有用。

要支持将一组资源作为单个逻辑单元进行管理，或管理*资源组*，我们发布了 **Resource Manager** 的预览版，作为管理 Azure 资源的新方法。

> [WACOM.NOTE] Resource Manager 当前为预览版，不能提供与 Azure 服务管理相同级别的管理功能。

为了支持新的 Resource Manager，xplat-cli 允许您使用 `azure config mode` 命令在这些管理模式之间切换。

xplat-cli 默认使用 Azure 服务管理模式。要切换到 Resource Manager 模式，请使用以下命令来启用命令：

    azure config mode arm

要更改回 Azure 服务管理模式，请使用以下命令：

    azure config mode asm 

> [WACOM.NOTE] Resource Manager 模式和 Azure 服务管理模式互斥。即在一种模式下创建的资源不能从另一种模式进行管理。

有关通过 xplat-cli 使用 Resource Manager 的详细信息，请参阅[结合使用 Azure 跨平台命令行接口和 Resource Manager][结合使用 Azure 跨平台命令行接口和 Resource Manager]。

### 使用 Azure 服务管理模式下的服务

借助 xplat-cli，您可以轻松地管理 Azure 服务。在这个示例中，您将了解如何使用 xplat-cli 管理 Azure 网站。

1.  使用以下命令创建一个新的 Azure 网站。将 **mywebsite** 替换为唯一名称。

        azure site create mywebsite

    系统将提示您指定用于创建网站的区域。请选择在地理位置上接近您的区域。此命令完成后，将在 http://mywebsite.chinacloudsites.cn （将 **mywebsite** 替换为您指定的名称）上提供网站。

    > [WACOM.NOTE] 如果您使用 Git 进行项目源代码管理，则可以指定 `--git` 参数以便在 Azure 上为此网站创建一个 Git 存储库。这还将在该命令从其运行的目录中创建一个 Git 存储库，如果该存储库尚不存在。它还将创建一个名为 **azure** 的 Git remote，用于通过 `git push azure master` 命令将部署推送到 Azure 网站。

    > [WACOM.NOTE] 如果您收到提示“站点”不是 Azure 命令的错误，则 xplat-cli 很可能不是资源组模式。要更改回为资源模式，请使用 `azure config mode asm` 命令。

2.  使用以下命令可为您的订阅列出网站：

        azure site list

    该列表应包含在之前步骤中创建的网站。

3.  使用以下命令可停止网站。将 **mywebsite** 替换为站点名称。

        azure site stop mywebsite

    在命令执行完毕后，您可以刷新浏览器以便验证该网站已停止。

4.  使用以下命令可启动该网站。将 **mywebsite** 替换为站点名称。

        azure site start mywebsite

    在命令执行完毕后，您可以刷新浏览器以便验证该网站已启动。

5.  使用以下命令可删除该网站。将 **mywebsite** 替换为站点名称。

        azure site delete mywebsite

    在命令执行完毕后，使用 `azure site list` 命令确认网站是否已不再存在。

## <span id="script"></span></a>如何撰写 Azure 跨平台命令行接口脚本

尽管您可以使用 xplat-cli 手动发布命令，但也可以通过利用命令行解释程序的功能以及系统上提供的其他命令行实用工具，来创建复杂的自动化工作流。例如，以下命令将停止所有正在运行的 Azure 网站：

    azure site list | grep 'Running' | awk '{system("azure site stop "$2)}'

该示例将网站列表传输到 `grep` 命令，该命令将检查每一行是否有字符串“Running”。然后将匹配的任何行传输到 `awk` 命令，该命令将调用 `azure site stop` 并使用传递给它的第二列（正在运行的站点名称）作为要停止的站点名称。

尽管此过程演示了如何将命令链接在一起，但您也可以使用命令行解释程序提供的脚本撰写功能创建更复杂的脚本。不同的命令行解释程序具体不同的脚本撰写功能和语法。Bash 可能是针对基于 UNIX 的系统（包括 Linux 和 OS X）的最常用的命令行解释程序。

有关使用 Bash 撰写脚本的信息，请参阅[高级 Bash 脚本撰写指南][高级 Bash 脚本撰写指南]。

有关撰写基于 OS X 或 Linux 的系统的脚本的更常规的信息，请参阅 [Shell 脚本][Shell 脚本]。

有关使用批处理文件撰写基于 Windows 的系统的脚本的信息，请参阅[命令行参考 A-Z][命令行参考 A-Z]。

### 理解结果

在创建脚本时，您通常需要捕获命令输出，并且或者将此输出传递给其他命令，或者对此输出执行某一操作，例如检查特定值。xplat-cli 将输出生成到 STDOUT 和 STDERR。每一行都以字符串 `info:` 为前缀以便用于信息状态消息，或者以 `data:` 为前缀以便用于针对某一服务返回的数据；但是，您可以通过使用 `--verbose` 或 `-v` 参数指示 xplat-cli 返回更详细信息。此操作将返回以字符串 `verbose:` 为前缀的其他信息。

例如，从 `azure site list` 命令返回以下输出：

    info:    Executing command site list
    + Enumerating sites
    data:    Name           Status   Mode  Host names
    data:    -------------  -------  ----  -------------------------------
    data:    myawesomesite  Running  Free  myawesomesite.chinacloudsites.cn
    info:    site list command OK

如果指定了 `--verbose` 或 `-v` 参数，将返回以下信息：

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

请注意，`verbose:` 信息显示为 JSON 格式的数据。如果您在使用本身理解 JSON 的实用工具，例如 [jsawk][jsawk] 或 [jq][jq]，则可以使用 `--json` 参数返回 JSON 格式的信息。例如：

    azure site list --json | jsawk -n 'out(this.Name)' | xargs -L 1 azure site delete -q 

上面的命令将网站列表作为 JSON 检索，然后使用 jsawk 检索网站名称，并且最后使用 xargs 为每个网站运行站点删除命令，并且将网站名称作为参数传递。

> [WACOM.NOTE] `--json` 参数将阻止生成状态或数据信息（以 `info:` 和 `data:` 为前缀的字符串）。例如，如果将 `--json` 参数用于 `azure site create`，则不会返回任何输出，因为此命令不返回 `info:` 以外的任何数据。

### 处理错误

尽管 xplat-cli 将错误信息记录到 STDERR，但与错误有关的其他信息也可以记录到用于运行脚本的目录下的 **azure.err** 文件中。如果信息记录到此文件，则以下内容将写入 STDOUT：

    info:    Error information has been recorded to azure.err

### 退出状态

如果缺少必需的参数，则某些 xplat-cli 命令将不会返回非零退出状态。这些命令而是将会提示用户输入。例如，在使用 `azure site create` 命令创建新网站时，如果没有指定站点名称或 `--location` 参数，系统将提示您提供这些值。

如果您在撰写依赖于退出状态的脚本，请确认您在使用的 xplat-cli 命令不会提示用户输入。

## <span id="additional-resources"></span></a>其他资源

-   有关 xplat-cli、下载源代码、报告问题或贡献项目的详细信息，请访问[适用于 Azure 跨平台命令行接口的 GitHub 存储库][适用于 Azure 跨平台命令行接口的 GitHub 存储库]。

-   如果您在使用 xplat-cli 或 Azure 时遇到问题，请访问 [Azure 论坛][Azure 论坛]。

-   有关 Azure 的详细信息，请参阅 [http://www.windowsazure.cn/][http://www.windowsazure.cn/]。


  [如何安装 Azure 跨平台命令行接口]: #install
  [如何连接到 Azure 订阅]: #configure
  [如何使用 Azure 跨平台命令行接口]: #use
  [如何撰写 Azure 跨平台命令行接口脚本]: #script
  [其他资源]: #additional-resources
  [Windows 安装程序]: http://go.microsoft.com/fwlink/?LinkID=275464&clcid=0x409
  [OS X 安装程序]: http://go.microsoft.com/fwlink/?LinkId=252249
  
  [以组织身份注册 Windows Azure]: /zh-cn/documentation/articles/sign-up-organization/
  [“基于帐户的身份验证和基于证书的身份验证之间的区别是什么”]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh531793.aspx#BKMK_AccountVCert
  [结合使用 Azure 跨平台命令行接口和 Resource Manager]: /zh-cn/documentation/articles/xplat-cli-azure-resource-manager/
  [高级 Bash 脚本撰写指南]: http://tldp.org/LDP/abs/html/
  [Shell 脚本]: http://en.wikipedia.org/wiki/Shell_script
  [命令行参考 A-Z]: http://technet.microsoft.com/zh-cn/library/bb490890.aspx
  [jsawk]: https://github.com/micha/jsawk
  [jq]: http://stedolan.github.io/jq/
  [适用于 Azure 跨平台命令行接口的 GitHub 存储库]: https://github.com/WindowsAzure/azure-sdk-tools-xplat
  [Azure 论坛]: https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=windowsazurezhchs
  [http://www.windowsazure.cn/]: http://www.windowsazure.cn
