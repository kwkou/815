<properties
	pageTitle="在源代码管理中存储 JavaScript 后端项目代码 | Azure 移动服务"
	description="了解如何在您的计算机本地 Git 存储库中存储您的服务器脚本文件和模块。"
	services="mobile-services"
	documentationCenter=""
	authors="ggailey777"
	manager="dwrede"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="12/07/2015"
	wacn.date="01/29/2016"/>

# 在源代码管理中存储移动服务项目代码

> [AZURE.SELECTOR]
- [.NET backend](/documentation/articles/mobile-services-dotnet-backend-store-code-source-control/)
- [Javascript backend](/documentation/articles/mobile-services-store-scripts-source-control/)

本主题向您展示了如何使用 Azure 移动服务提供的源代码管理来存储服务器脚本。脚本和其他 JavaScript 后端代码文件可从您的本地 Git 存储库提升到生产移动服务。另外，本主题还说明了如何定义可在多个脚本之间使用的共享代码，以及如何使用 package.json 文件来为您的移动服务添加 Node.js 模块。

若要完成本教程，你必须事先参考[移动服务入门]教程创建一个移动服务。

## <a name="enable-source-control"></a>在移动服务中启用源代码管理

[AZURE.INCLUDE [mobile-services-enable-source-control](../includes/mobile-services-enable-source-control.md)]

## <a name="clone-repo"></a>安装 Git 并创建本地存储库

1. 在本地计算机上安装 Git。 

	安装 Git 所需的步骤因操作系统的不同而异。有关操作系统特定的分发和安装指南，请参阅[安装 Git]。

	> [AZURE.NOTE]在某些操作系统上，命令行和 GUI 版本的 Git 都可用。本文中提供的说明使用命令行版本。

2. 打开一个命令行，例如 **GitBash** (Windows) 或 **Bash** (Unix Shell)。在 OS X 系统上，可以通过 **Terminal** 应用程序访问命令行。

3. 在命令行中，切换到要在其中存储脚本的目录。例如，`cd SourceControl`。

4. 使用以下命令创建新 Git 存储库的本地副本，并将 `<your_git_URL>` 替换为移动服务的 Git 存储库的 URL：

		git clone <your_git_URL>

5. 出现提示时，请键入你在移动服务中启用源代码管理时设置的用户名和密码。身份验证成功后，你将看到类似于下面的一系列响应：

		remote: Counting objects: 8, done.
		remote: Compressing objects: 100% (4/4), done.
		remote: Total 8 (delta 1), reused 0 (delta 0)
		Unpacking objects: 100% (8/8), done.

6. 浏览到你从中运行了 `git clone` 命令的目录，并留意以下目录结构：

	![4][4]

	此时已创建了一个使用移动服务名称的新目录，即数据服务的本地存储库。

7. 打开 .\\service\\table 子文件夹，可以看到，它包含一个 TodoItem.json 文件，该文件是对 TodoItem 表的操作权限的 JSON 表示形式。

	如果在此表中定义了服务器脚本，则还会出现一个或多个名为 <code>TodoItem._&lt;operation&gt;_.js</code> 的文件，其中包含给定表操作的脚本。计划程序和自定义 API 脚本保留在与其名称相对应的不同文件夹中。有关详细信息，请参阅[源代码管理]。

创建本地存储库后，您可以更改服务器脚本，以及将更改推送回到移动服务。

## <a name="deploy-scripts"></a>将更新的脚本文件部署到移动服务

1. 浏览到 .\\service\\table 子文件夹，如果 todoitem.insert.js 文件不存在，现在请创建该文件。

2. 在文本编辑器中打开新文件 todoitem.insert.js 并在其中粘贴以下代码，然后保存更改：

		function insert(item, user, request) {
		    request.execute();
		    console.log(JSON.stringify(item, null, 4));
		}
	
	此代码只是将插入的项写入日志。如果此文件已包含代码，则你只需在此文件中添加一些有效的 JavaScript 代码（例如，对 `console.log()` 的调用），然后保存更改。

3. 在 Git 命令提示符下，键入以下命令以开始跟踪新脚本文件：

		$ git add .
	

4. 键入以下命令以提交更改：

		$ git commit -m "updated the insert script"

5. 键入以下命令以将更改上载到远程存储库：

		$ git push origin master
	
	您应该会看到一系列命令，指出已将提交的内容部署到移动服务。

6. 返回 [Azure 经典门户]，单击“数据”选项卡，单击“TodoItem”表，单击“脚本”，然后选择“插入”操作。可以看到，显示的插入操作脚本与您刚刚上载到存储库的 JavaScript 代码相同。

## <a name="use-npm"></a>在服务器脚本中利用共享代码和 Node.js 模块

移动服务允许你访问整个核心 Node.js 模块集，你可以通过 **require** 函数在代码中使用这些模块。移动服务还可使用不属于核心 Node.js 程序包的 Node.js 模块，您甚至可以定义自己的共享代码作为 Node.js 模块。有关创建模块的详细信息，请参阅 Node.js API 参考文档中的[模块]。

将 Node.js 模块添加到移动服务的建议方法为对服务的 package.json 文件添加引用。然后通过更新 package.json 文件为你的移动服务添加 [node-uuid] Node.js 模块。当更新推送到 Azure 时，移动服务重新启动，模块安装完成。然后，你可以使用此模块为已插入项的 **uuid** 属性生成新的 GUID 值。

2. 导航到本地 Git 存储库的 `.\service` 文件夹，在文本编辑器中打开 package.json 文件，然后将以下字段添加到 **dependencies** 对象：

		"node-uuid": "~1.4.3"

	>[AZURE.NOTE]对 package.json 文件执行这项更新会导致推送提交内容后重新启动移动服务。

2. 现在，请浏览到 .\\service\\table 子文件夹，打开 todoitem.insert.js 文件并按如下所示修改该文件：

		function insert(item, user, request) {
		    var uuid = require('node-uuid');
		    item.uuid = uuid.v1();
		    request.execute();
		    console.log(item);
		}

	此代码将在表中添加一个 uuid 列，并使用唯一的 GUID 标识符填充该列。

5. 像在前一部分中一样，在 Git 命令提示符下键入以下命令：

		$ git add .
		$ git commit -m "added node-uuid module"
		$ git push origin master
		
	这样就会添加新的文件，提交您的更改，并将新的 node-uuid 模块以及对 todoitem.insert.js 脚本所做的更改推送到您的移动服务。

##  <a name="next-steps"></a>后续步骤

完成本教程后，你便知道了如何在源代码管理中存储脚本。我们建议您了解有关如何使用服务器脚本和自定义 API 的详细信息：

+ [在移动服务中使用服务器脚本]
  <br/>说明如何使用服务器脚本、作业计划程序和自定义 API。

<!-- Anchors. -->
[Enable source control in your mobile service]: #enable-source-control
[Install Git and create the local repository]: #clone-repo
[Deploy updated script files to your mobile service]: #deploy-scripts
[Leverage shared code and Node.js modules in your server scripts]: #use-npm

<!-- Images. -->

[4]: ./media/mobile-services-store-scripts-source-control/mobile-source-local-repo.png
[5]: ./media/mobile-services-store-scripts-source-control/mobile-portal-data-tables.png
[6]: ./media/mobile-services-store-scripts-source-control/mobile-insert-script-source-control.png

<!-- URLs. -->
[Git website]: http://git-scm.com
[源代码管理]: http://msdn.microsoft.com/zh-cn/library/windowsazure/c25aaede-c1f0-4004-8b78-113708761643
[安装 Git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[移动服务入门]: /documentation/articles/mobile-services-ios-get-started/
[在移动服务中使用服务器脚本]: /documentation/articles/mobile-services-how-to-use-server-scripts/
[Azure 经典门户]: https://manage.windowsazure.cn/
[模块]: http://nodejs.org/api/modules.html
[node-uuid]: https://npmjs.org/package/node-uuid

<!---HONumber=Mooncake_0118_2016-->