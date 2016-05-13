#如何使用针对 Mac 和 Linux 的 Azure 命令行工具

本指南介绍如何使用适用于 Mac 和 Linux 的 Azure 命令行工具创建和管理 Azure 中的服务。所涉及的方案包括**安装工具**、**导入发布设置**、**创建和管理 Azure 网站**以及**创建和管理 Azure 虚拟机**。有关完整的参考文档，请参阅[适用于 Mac 和 Linux 的 Azure 命令行工具文档][reference-docs]。

##目录
* [什么是适用于 Mac 和 Linux 的 Azure 命令行工具](#Overview)
* [如何安装面向 Mac 和 Linux 的 Azure 命令行工具](#Download)
* [如何创建 Azure 帐户](#CreateAccount)
* [如何下载和导入发布设置](#Account)
* [如何创建和管理 Azure 网站](#WebSites)
* [如何创建和管理 Azure 虚拟机](#VMs)


<h2><a id="Overview"></a>什么是适用于 Mac 和 Linux 的 Azure 命令行工具</h2>

适用于 Mac 和 Linux 的 Azure 命令行工具是一组用于部署和管理 Azure 服务的命令行工具。
 
支持的任务包括：

* 导入发布设置。
* 创建和管理 Azure 网站。
* 创建和管理 Azure 虚拟机。

有关支持命令的完整列表，请在这些工具安装后在命令行处键入 `azure -help`，或参见[参考文档][reference-docs]。

<h2><a id="Download">如何安装面向 Mac 和 Linux 的 Azure 命令行工具</a></h2>

以下列表包含有关安装命令行工具的信息（具体取决于你的操作系统）：

* **Mac**：下载 [Azure SDK 安装程序][mac-installer]。打开下载的 .pkg 文件并根据提示完成安装步骤。

* **Linux**：安装最新版本的 [Node.js][nodejs-org]（请参阅[通过程序包管理器安装 Node.js][install-node-linux]），然后运行以下命令：

		npm install azure-cli -g

	**注意**：你可能需要使用提升的权限才能运行此命令：

		sudo npm install azure-cli -g

* **Windows**：运行此处 [Azure 命令行工具][windows-installer]提供的 Windows 安装程序（.msi 文件）。


若要测试安装，请在命令提示符下键入 `azure`。如果安装成功，你将会看到所有可用 `azure` 命令的列表。

<h2><a id="CreateAccount"></a>如何创建 Azure 帐户</h2>

若要使用适用于 Mac 和 Linux 的 Azure 命令行工具，你需要一个 Azure 帐户。

打开 Web 浏览器，浏览到 [http://www.azure.cn][azuredotcn] 并单击右上角的**免费试用**。

![Azure 网站][Azure Web Site]

遵循有关创建帐户的说明。

<h2><a id="Account"></a>如何下载和导入发布设置</h2>

若要开始操作，您需要先下载并导入您的发布设置。这将允许您使用这些工具来创建和管理 Azure 服务。若要下载你的发布设置，请使用 `account download` 命令：

	azure account download

这将打开默认浏览器，并提示您登录到管理门户。登录后，将会下载你的 `.publishsettings` 文件。记下此文件的保存位置。

接下来，通过运行以下命令导入 `.publishsettings` 文件，并将 `{path to .publishsettings file}` 替换为 `.publishsettings` 文件的路径：

	azure account import {path to .publishsettings file}

可以使用 <code>account clear</code> 命令来删除通过 <code>import</code> 命令存储的所有信息：

	azure account clear

若要查看 `account` 命令的选项列表，请使用 `-help` 选项：

	azure account -help

导入发布设置后，为安全起见，应删除 `.publishsettings` 文件。

> [AZURE.NOTE] 导入发布设置时，用于访问 Azure 订阅的凭据将存储在你的 `user` 文件夹中。你的 `user` 文件夹受操作系统保护。但是，建议你采取其他措施来对你的 `user` 文件夹进行加密。你可通过以下方式完成此操作：
> 
> - 在 Windows 上，修改文件夹属性或使用 BitLocker。
> - 在 Mac 上，为文件夹启用 FileVault。
> - 在 Ubuntu 上，使用“加密主目录”功能。其他 Linux 分发提供了等效功能。

此时你已准备好创建和管理 Azure 网站和 Azure 虚拟机。

<h2><a id="WebSites"></a>如何创建和管理 Azure 网站</h2>

###创建网站

若要创建 Azure 网站，请先创建一个名为 `MySite` 的空目录，然后浏览到该目录。

然后，运行以下命令：

	azure site create MySite --git

此命令的输出将包含新创建的网站的默认 URL。选项 `--git` 允许你通过在本地应用程序目录和网站的数据中心创建 git 存储库来使用 git 发布到网站。请注意，如果本地文件夹已是 git 存储库，则此命令会将新的远程添加到现有存储库，并指向网站数据中心内的存储库。

请注意，你可将 `azure site create` 命令与下列任一选项一起执行：

* `--location [location name]`。此选项允许你指定创建网站的数据中心的位置（例如“美国西部”）。如果忽略此选项，系统将提示你选择一个位置。
* `--hostname [custom host name]`。此选项允许你指定网站的自定义主机名。

然后，你可以将内容添加到网站目录。使用常规 git 流（`git add`，`git commit`）来提交你的内容。使用以下 git 命令可将网站内容推送到 Azure：

	git push azure master

###设置从 GitHub 的发布

若要从 GitHub 存储库设置连续发布，请在创建站点时使用 `--GitHub` 选项：

	auzre site create MySite --github --githubusername username --githubpassword password --githubrepository githubuser/reponame

如果您拥有 GitHub 存储库的本地克隆，或者您拥有带对 GitHub 存储库的单个远程引用的存储库，则此命令会自动将 GitHub 存储库中的代码发布到您的站点。自此，任何推送到 GitHub 存储库的更改都将自动发布到您的站点。

在设置从 GitHub 的发布时，所使用的默认分支为 master 分支。若要指定其他分支，请从本地存储库执行以下命令：

	azure site repository <branch name>

###配置应用设置

应用设置是在运行时可供应用程序使用的键值对。在针对 Azure 网站进行设置时，应用设置值将重写你网站的 Web.config 文件中定义的具有相同键的设置。对于 Node.js 和 PHP 应用程序，应用设置可用作环境变量。下面的示例演示如何设置键值对：

	azure site config add <key>=<value> 

若要查看所有键值对的列表，请使用以下命令：

	azure site config list 

或者，如果您知道该键并希望查看值，则可使用：

	azure site config get <key> 

若要更改现有键的值，则必须先清除现有键，然后重新添加该键。清除命令为：

	azure site config clear <key> 

###列出并显示站点

若要列出你的网站，请使用以下命令：

	azure site list

若要获取有关站点的详细信息，请使用 `site show` 命令。下面的示例演示了 `MySite` 的详细信息：

	azure site show MySite

###停止、启动或重新启动站点

你可以使用 `site stop`、`site start` 或 `site restart` 命令停止、启动或重新启动站点：

	azure site stop MySite
	azure site start MySite
	azure site restart MySite

###删除站点

最后，你可使用 `site delete` 命令删除站点：

	azure site delete MySite

请注意，如果你从运行 `site create` 的文件夹中运行上述任一命令，则无需将站点名称 `MySite` 指定为最后一个参数。

若要查看 `site` 命令的完整列表，请使用 `-help` 选项：

	azure site -help 

<h2><a id="VMs"></a>如何创建和管理 Azure 虚拟机</h2>

从你提供的或映像库中可用的虚拟机映像（.vhd 文件）创建 Azure 虚拟机。若要查看可用的映像，请使用 `vm image list` 命令：

	azure vm image list

你可使用 `vm create` 命令从可用映像之一设置和启动虚拟机。下面的示例演示了如何从映像库 (CentOS 6.2) 中的映像创建 Linux 虚拟机（名为 `myVM`）。虚拟机的根用户名和密码分别为 `myusername` 和 `Mypassw0rd`。（请注意，`--location` 参数指定在其中创建虚拟机的数据中心。如果忽略 `--location` 参数，则系统将提示你选择一个位置。）

	azure vm create myVM OpenLogic__OpenLogic-CentOS-62-20120509-en-us-30GB.vhd myusername --location "West US"

你可以考虑将 `--ssh` 标志 (Linux) 或 `--rdp` 标志 (Windows) 传递到 `vm create` 以支持与新创建的虚拟机的远程连接。

如果你更愿意从自定义映像设置虚拟机，则可使用 `vm image create` 命令从 .vhd 文件创建映像，然后使用 `vm create` 命令设置虚拟机。下面的示例演示了如何从本地 .vhd 文件创建 Linux 映像（名为 `myImage`）。（`--location` 参数指定在其中存储映像的数据。）

	azure vm image create myImage /path/to/myImage.vhd --os linux --location "West US"

你可以从存储在 Azure Blob 存储中的 .vhd 创建映像，而不用从本地 .vhd 创建映像。你可使用 `blob-url` 参数做到这一点：

	azure vm image create myImage --blob-url <url to .vhd in Blob Storage> --os linux

创建映像后，你可使用 `vm create` 从映像处设置虚拟机。下面的命令将从上面创建的映像（`myImage`） 创建名为 `myVM` 的虚拟机。

	azure vm create myVM myImage myusername --location "China East"

设置虚拟机后，您可能需要创建终结点以允许对虚拟机进行远程访问（见下例）。下面的示例使用 `vm create endpoint` 命令打开 `myVM` 上的外部端口 22 和本地端口 22：

	azure vm endpoint create myVM 22 22

你可使用 `vm show` 命令获取有关虚拟机的详细信息（包括 IP 地址、DNS 名称和终结点信息）：

	azure vm show myVM

若要关闭、启动或重新启动虚拟机，请使用下列命令之一：

	azure vm shutdown myVM
	azure vm start myVM
	azure vm restart myVM

最后，若要删除 VM，请使用 `vm delete` 命令：

	azure vm delete myVM

有关用于创建和管理虚拟机的命令的完整列表，请使用 `-h` 选项：

	azure vm -h

<!-- IMAGES -->
[Azure Web Site]: ./media/crossplat-cmd-tools/freetrial.png

<!-- LINKS -->
[nodejs-org]: http://nodejs.org/
[install-node-linux]: https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager
[mac-installer]: http://go.microsoft.com/fwlink/?LinkId=252249
[windows-installer]: http://go.microsoft.com/fwlink/?LinkID=275464
[reference-docs]: http://go.microsoft.com/fwlink/?LinkId=252246
[azuredotcn]: http://www.azure.cn

<!---HONumber=Mooncake_0307_2016-->