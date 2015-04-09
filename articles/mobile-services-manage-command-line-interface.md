<properties linkid="develop-mobile-tutorials-command-line-administration" urlDisplayName="命令行管理" pageTitle="在命令行管理移动服务 - Azure 教程" metaKeywords="" description="了解如何使用命令行工具创建、 部署和管理您的 Azure 移动服务。" metaCanonical="" services="" documentationCenter="Mobile" title="Automate mobile services with command-line tools" authors="glenga" solutions="" manager="" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-multiple" ms.devlang="multiple" ms.topic="article" ms.date="11/21/2014" ms.author="glenga" />

# 使用命令行工具自动操作移动服务 

本主题说明如何使用 Azure 命令行工具来自动创建和管理 Azure 移动服务。另外，还说明了如何安装和开始使用这些命令行工具，以及如何使用它们执行以下移动服务任务：

-	[创建新的移动服务] 
-	[创建新表]
-   [注册脚本以对表进行操作][注册新的表脚本]
-   [列出表]
- 	[删除现有表]
-	[列出移动服务]
-   [删除现有移动服务]
 
将其中的每个命令合并到单个脚本或批处理文件后，这些命令可以自动完成移动服务的创建、验证和删除过程。 

若要使用 Azure 命令行工具来管理移动服务，你需要有一个启用了 Azure 移动服务功能的 Azure 帐户。

+ 如果没有帐户，则可创建一个免费的试用帐户，只需几分钟即可完成。有关详细信息，请参阅： <a href="http://www.windowsazure.cn/zh-cn/pricing/1rmb-trial/" target="_blank">Azure 免费试用版</a>。

+ 如果您已有帐户但需要启用 Azure 移动服务预览，请参阅 <a href="/zh-cn/documentation/articles/php-create-account/#enable" target="_blank">启用 Azure 预览功能</a>。

本主题有选择地介绍了 Azure 命令行工具支持的多个常见管理任务。有关详细信息，请参阅[Azure 命令行工具文档][参考文档]。

<!--+  您必须在本地计算机上下载并安装 Azure 命令行工具。若要执行此操作，请按照本主题的第一个部分中的说明进行操作。 

+ （可选）若要能够直接从命令行执行 HTTP 请求，必须使用 cURL 或同等工具。cURL 可在各种平台上运行。针对您的平台查找和安装 cURL，请访问 <a href=http://go.microsoft.com/fwlink/p/?LinkId=275676 target="_blank">cURL 下载  页面</a>。-->

<h2><a name="install"></a>安装 Azure 命令行工具</h2>

以下列表包含有关安装命令行工具的信息（具体取决于你的操作系统）：

* **Windows**:下载[Azure 命令行工具安装程序][windows-安装程序]。打开下载的 .msi 文件并根据提示完成安装步骤。

* **Mac**:下载[Azure SDK 安装程序][mac-安装程序]。打开下载的 .pkg 文件并根据提示完成安装步骤。

* **Linux**:安装最新版本的[Node.js][nodejs-org] （请参阅[通过程序包管理器安装 Node.js][安装节点 linux]），然后运行以下命令：

		npm install azure-cli -g

若要测试安装，请在命令提示符下键入  `azure`。如果安装成功，则会显示所有可用  `azure` 命令的列表。
<h2><a name="import-account"></a>如何下载和导入发布设置</h2>

若要开始操作，必须先下载并导入你的发布设置。然后，你便可以使用这些工具来创建和管理 Azure 服务。若要下载您的发布设置，请使用 `account download`命令：

		azure account download

这将会打开默认浏览器，并提示你登录到管理门户。登录后，将会下载您的 `.publishsettings`文件。请记下此文件的保存位置。

接下来，通过运行以下命令导入 `.publishsettings`文件，用您的 `.publishsettings`文件路径来替换`<path-to-settings-file>`：

		azure account import <path-to-settings-file>

您可以使用导入命令删除 <code>import</code> ，使用 <code>account clear</code> 命令：

		azure account clear

若要查看 `account`命令的选项列表，请使用`-help`选项：

		azure account -help

导入发布设置后，为安全起见，您应删除 `.publishsettings`文件。有关详细信息，请参阅[如何安装适用于 Mac 和 Linux 的 Azure 命令行工具]。现在，你便可以通过命令行或者批处理文件开始创建和管理 Azure 移动服务了。  

<h2><a name="create-service"></a><span class="short-header">创建服务</span>如何创建移动服务</h2>

可以使用命令行工具创建新的移动服务实例。在创建移动服务的同时，还会在新服务器中创建一个 SQL Database 实例。 

以下命令将在订阅中创建一个新的移动服务实例，其中`<service-name>`是新移动服务的名称，`<server-admin>`是新服务器的登录名，`<server-password>`是新登录名的密码：

		azure mobile create <service-name> <server-admin> <server-password>

当存在指定的移动服务时， `mobile create`命令将会失败。在尝试重新创建某个移动服务之前，你应该在自动化脚本中尝试删除该移动服务。

<h2><a name="list-services"></a>如何列出订阅中的现有移动服务</h2>

以下命令将返回 Azure 订阅中所有移动服务的列表：

		azure mobile list

此命令还会显示每个移动服务的当前状态和 URL。

<h2><a name="delete-service"></a>如何删除现有的移动服务</h2>

可以使用命令行工具将现有的某个移动服务连同相关的 SQL Database 和服务器一起删除。以下命令将删除移动服务，其中`<service-name>`是要删除的移动服务的名称：

		azure mobile delete <service-name> -a -q

如果包含`-a`和`-q`参数的话，此命令还会删除该移动服务使用的 SQL Database 和服务器且不显示任何提示。

<div class="dev-callout"><strong>注意</strong> 
   <p>如果未指定 <code>-q</code> 参数与 <code>-a</code> 或 <code>-d</code>系统将暂停执行，并且提示您选择 SQL Database 的删除选项。当没有其他服务使用数据库或服务器时，只能使用 <code>-a</code> 参数；否则请使用 <code>-d</code> 参数，以便只删除属于要删除的移动服务的数据。</p>
</div>

<h2><a name="create-table"></a>如何在移动服务中创建表</h2>

下面的命令在指定的移动服务中创建一个表，其中`<service-name>`是移动服务的名称，`<table-name>`是要创建的表的名称：

		azure mobile table create <service-name> <table-name>

这将使用默认的权限创建一个新表， `application`，用于以下表操作： `insert`、 `read`、 `update`和 `delete`。 

下面的命令使用公共 `read`权限创建一个新表，但具有只向管理员授予的 `delete`权限：

		azure mobile table create <service-name> <table-name> -p read=public,delete=admin

下面显示了脚本权限值与[Azure 管理门户]中的权限值的对照表。

<table border="1" width="100%"><tr><th>脚本中的值</th><th>管理门户中的值</th></tr>
<tr><td><code>public</code></td><td>所有人</td></tr>
<tr><td><code>application</code> （默认）</td><td>具有应用程序密钥的任何人</td></tr>
<tr><td><code>user</code></td><td>仅经过身份验证的用户</td></tr>
<tr><td><code>admin	</code></td><td>仅脚本和管理员</td></tr></table>

当存在指定的表时， `mobile table create`命令将会失败。在尝试重新创建某个表之前，你应该在自动化脚本中尝试删除该表。

<h2><a name="list-tables"></a>如何列出移动服务中的现有表</h2>

以下命令将返回移动服务中所有表的列表，其中`<service-name>`是移动服务的名称：

		azure mobile table list <service-name>

此命令还会显示每个表中的索引数，以及表中当前的数据行数。

<h2><a name="delete-table"></a>如何删除移动服务中的现有表</h2>

下面的命令将删除移动服务中的表，其中`<service-name>`是移动服务的名称，`<table-name>`是要删除的表的名称：

		azure mobile table delete <service-name> <table-name> -q

在自动化脚本中，使用`-q`参数可以删除该表且不显示会阻碍执行的确认提示。

<h2><a name="register-script"></a>如何将脚本注册到表操作</h2>

下面的命令将上载并注册到表操作的函数，其中`<service-name>`是移动服务的名称，`<table-name>`是表的名称和，`<operation>`是表操作，可以是 `read`、 `insert`、 `update`或 `delete`：

		azure mobile script upload <service-name> table/<table-name>.<operation>.js

请注意，此操作将从本地计算机上载 JavaScript (.js) 文件。文件的名称必须由表和操作名称组成，并且文件必须位于和执行该命令的位置相对应的 `table`子文件夹中。例如，下面的操作将上载并注册一个新的 `insert`脚本，该脚本属于 `TodoItems`表：

		azure mobile script upload todolist table/todoitems.insert.js

脚本文件中的函数声明也必须与注册的表操作相匹配。也就是说，对于 `insert`脚本，上载的脚本应包含带有以下签名的函数：

		function insert(item, user, request) {
		    ...
		} 

有关注册脚本的详细信息，请参阅[移动服务服务器脚本参考]。

<!--<h2><a name="test-service"></a>测试新的移动服务</h2>

当您自动创建移动服务时，您可以选择使用 cURL 或另一个命令行请求生成器 

## <a name="nextsteps"> </a>后续步骤
后续步骤...
-->
<!-- Anchors. -->
[下载并安装命令行工具]: #install
[下载并导入发布设置]: #import
[创建新的移动服务]: #create-service
[获取主密钥]: #get-master-key
[创建新表]: #create-table
[注册新的表脚本]: #register-script
[删除现有表]: #delete-table
[删除现有移动服务]: #delete-service
[测试移动服务]: #test-service
[列出移动服务]: #list-services
[列出表]: #list-tables
[后续步骤]: #next-steps

<!-- Images. -->











<!-- URLs. -->
[移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/p?LinkId=262293

[Azure 管理门户]: https://manage.windowsazure.cn/
[nodejs-org]: http://nodejs.org/
[安装节点 linux]: https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager

[mac-安装程序]: http://go.microsoft.com/fwlink/p?LinkId=252249
[windows-安装程序]: http://go.microsoft.com/fwlink/p?LinkID=275464
[参考文档]: /zh-cn/documentation/articles/command-line-tools/#Commands_to_manage_mobile_services
[如何安装适用于 Mac 和 Linux 的 Azure 命令行工具]: /zh-cn/documentation/articles/xplat-cli

