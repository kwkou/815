<properties 
	pageTitle="通过命令行管理移动服务 | Azure" 
	description="了解如何使用命令行工具创建、 部署和管理您的 Azure 移动服务。" 
	services="mobile-services" 
	documentationCenter="Mobile" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="01/27/2016"
	wacn.date="04/07/2016"/>

#  使用命令行工具自动操作移动服务 

##概述

本主题说明如何使用 Azure 命令行工具来自动创建和管理 Azure 移动服务。另外，还说明了如何安装和开始使用这些命令行工具，以及如何使用它们执行关键的移动服务。
 
将其中的每个命令合并到单个脚本或批处理文件后，这些命令可以自动完成移动服务的创建、验证和删除过程。

本主题有选择地介绍了 Azure 命令行工具支持的多个常见管理任务。有关详细信息，请参阅 [Azure 命令行工具文档][reference-docs]。

## 安装 Azure 命令行工具

以下列表包含有关安装命令行工具的信息（具体取决于你的操作系统）：

* **Windows**：下载 [Azure 命令行工具安装程序][windows-installer]。打开下载的 .msi 文件并根据提示完成安装步骤。

* **Mac**：下载 [Azure SDK 安装程序][mac-installer]。打开下载的 .pkg 文件并根据提示完成安装步骤。

* **Linux**：安装最新版本的 [Node.js][nodejs-org]（请参阅[通过程序包管理器安装 Node.js][install-node-linux]），然后运行以下命令：

		npm install azure-cli -g

若要测试安装，请在命令提示符下键入 `azure`。如果安装成功，则会显示所有可用 `azure` 命令的列表。

## 如何下载和导入发布设置

若要开始操作，必须先下载并导入你的发布设置。然后，你便可以使用这些工具来创建和管理 Azure 服务。若要下载你的发布设置，请使用 `account download` 命令：

		azure account download

这将会打开默认浏览器，并提示你登录到 Azure 经典门户。登录后，将会下载你的 `.publishsettings` 文件。请记下此文件的保存位置。

接下来，通过运行以下命令导入 `.publishsettings` 文件，并将 `<path-to-settings-file>` 替换为 `.publishsettings` 文件的路径：

		azure account import <path-to-settings-file>

可以使用 <code>account clear</code> 命令来删除通过 <code>import</code> 命令存储的所有信息：

		azure account clear

若要查看 `account` 命令的选项列表，请使用 `-help` 选项：

		azure account -help

导入发布设置后，为安全起见，应删除 `.publishsettings` 文件。有关详细信息，请参阅[如何安装适用于 Mac 和 Linux 的 Azure 命令行工具]。现在，你便可以通过命令行或者批处理文件开始创建和管理 Azure 移动服务了。

## 如何创建移动服务

可以使用命令行工具创建新的移动服务实例。在创建移动服务的同时，还会在新服务器中创建一个 SQL 数据库实例。

以下命令将在订阅中创建一个新的移动服务实例，其中`<service-name>`是新移动服务的名称，`<server-admin>`是新服务器的登录名，`<server-password>`是新登录名的密码：

		azure mobile create <service-name> <server-admin> <server-password>

当存在指定的移动服务时，`mobile create` 命令将会失败。在尝试重新创建某个移动服务之前，你应该在自动化脚本中尝试删除该移动服务。

## 如何列出订阅中的现有移动服务

> [AZURE.NOTE]CLI 中与“list”和“script”相关的命令只适用于 JavaScript 后端。

以下命令将返回 Azure 订阅中所有移动服务的列表：

		azure mobile list

此命令还会显示每个移动服务的当前状态和 URL。

## 如何删除现有的移动服务

可以使用命令行工具将现有的某个移动服务连同相关的 SQL 数据库和服务器一起删除。以下命令将删除移动服务，其中`<service-name>`是要删除的移动服务的名称：

		azure mobile delete <service-name> -a -q

如果包含 `-a` 和 `-q` 参数的话，此命令还会删除该移动服务使用的 SQL 数据库和服务器且不显示任何提示。

> [AZURE.NOTE]如果不随 <code>-a</code> 或 <code>-d</code> 一起指定 <code>-q</code> 参数，则执行将会暂停，并且系统会提示你针对 SQL 数据库选择删除选项。仅当没有其他任何服务使用该数据库或服务器时，才能使用 <code>-a</code> 参数；否则，请使用 <code>-d</code> 参数，以便只删除属于要删除的移动服务的数据。

## 如何在移动服务中创建表

以下命令将在指定的移动服务中创建一个表，其中，`<service-name>` 是移动服务的名称，`<table-name>` 是要创建的表的名称：

		azure mobile table create <service-name> <table-name>

这将会创建一个提供默认权限的新表 `application`，用于以下表操作：`insert`、`read`、`update` 和 `delete`。

以下命令将创建一个向公众提供 `read` 权限、但只向管理员授予 `delete` 权限的新表：

		azure mobile table create <service-name> <table-name> -p read=public,delete=admin

下面显示了脚本权限值与 [Azure 管理门户]中的权限值的对照表。

|脚本值|门户值|
|------|------|
|`public`|每个人|
|`application`（默认值）|拥有应用程序密钥的任何人|
|`user`|仅限经过身份验证的用户|
|`admin`|仅限脚本和管理员|

当存在指定的表时，`mobile table create` 命令将会失败。在尝试重新创建某个表之前，你应该在自动化脚本中尝试删除该表。

## 如何列出移动服务中的现有表

以下命令将返回移动服务中所有表的列表，其中`<service-name>`是移动服务的名称：

		azure mobile table list <service-name>

此命令还会显示每个表中的索引数，以及表中当前的数据行数。

## 如何删除移动服务中的现有表

以下命令将删除移动服务中的某个表，其中，`<service-name>` 是移动服务的名称，`<table-name>` 是要删除的表的名称：

		azure mobile table delete <service-name> <table-name> -q

在自动化脚本中，使用 `-q` 参数可以删除该表且不显示会阻碍执行的确认提示。

## 如何将脚本注册到表操作

以下命令将一个函数上载并注册到表中的某个操作，其中，`<service-name>` 是移动服务的名称，`<table-name>` 是表的名称，`<operation>` 是表操作（可以是 `read`、`insert`、`update` 或 `delete`）：

		azure mobile script upload <service-name> table/<table-name>.<operation>.js

请注意，此操作将从本地计算机上载 JavaScript (.js) 文件。该文件的名称必须由表和操作名称组成，并且该文件必须位于相对于命令执行位置的 `table` 子文件夹中。例如，以下操作将上载并注册属于 `TodoItems` 表的新 `insert` 脚本：

		azure mobile script upload todolist table/todoitems.insert.js

脚本文件中的函数声明也必须与注册的表操作相匹配。也就是说，对于 `insert` 脚本，上载的脚本应包含带有以下签名的函数：

		function insert(item, user, request) {
		    ...
		} 

有关注册脚本的详细信息，请参阅[移动服务服务器脚本参考]。

<!-- Anchors. -->
[Download and install the command-line tools]: #install
[Download and import publish settings]: #import
[Create a new mobile service]: #create-service
[Get the master key]: #get-master-key
[Create a new table]: #create-table
[Register a new table script]: #register-script
[Delete an existing table]: #delete-table
[Delete an existing mobile service]: #delete-service
[Test the mobile service]: #test-service
[List mobile services]: #list-services
[List tables]: #list-tables
[Next steps]: #next-steps

<!-- Images. -->











<!-- URLs. -->
[移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/p?LinkId=262293

[Azure 管理门户]: https://manage.windowsazure.cn/
[nodejs-org]: http://nodejs.org/
[install-node-linux]: https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager

[mac-installer]: http://go.microsoft.com/fwlink/p?LinkId=252249
[windows-installer]: http://go.microsoft.com/fwlink/p?LinkID=275464
[reference-docs]: /documentation/articles/virtual-machines-command-line-tools/#Commands_to_manage_mobile_services
[如何安装适用于 Mac 和 Linux 的 Azure 命令行工具]: /zh-cn/documentation/articles/xplat-cli-install

<!---HONumber=Mooncake_0118_2016-->