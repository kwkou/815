<properties linkid="develop-mobile-tutorials-store-scripts-in-source-control" urlDisplayName="Store server scripts in source control" pageTitle="Store server scripts in source control - Azure Mobile Services" metaKeywords="" description="Learn how to store your server script files and modules in a local Git repo on your computer." metaCanonical="" services="" documentationCenter="Mobile" title="Store server scripts in source control" authors="glenga" solutions="" manager="" editor="" />

# 在源代码管理中存储服务器脚本

本主题说明如何在 Azure 移动服务中首次设置源代码管理，以便在 Git 存储库中存储服务器脚本。可将脚本和其他 JavaScript 代码文件从本地存储库提升到生产移动服务。另外，本主题还说明了如何定义可在多个脚本之间使用的共享代码，以及如何上载 Node.js 模块。

本教程将指导你完成以下步骤：

1.  [在移动服务中启用源代码管理][]。
2.  [安装 Git 并创建本地存储库][]。
3.  [将更新的脚本文件部署到移动服务][]。
4.  [在服务器脚本中利用共享代码和 Node.js 模块][]。

若要完成本教程，你必须事先参考[移动服务入门][]或[数据处理入门][]教程创建一个移动服务。

## 启用源代码管理在移动服务中启用源代码管理

1.  登录到 [Azure 管理门户][]，单击“移动服务”，然后单击你的移动服务 。

    ![][]

2.  单击“仪表板”选项卡，在“速览”下单击“设置源代码管理”，然后单击“是”进行确认 。

    ![][1]

    > [WACOM.NOTE]
    > 源代码管理是一项预览功能。尽管你的脚本文件存储在移动服务中，但我们还是建议你定期备份这些文件。

3.  输入“用户名”和“新密码”，确认密码，然后单击勾选按钮 。

    ![][2]

    随后将在你的移动服务中创建 Git 存储库。记下你刚刚提供的凭据，访问此存储库时需要使用这些凭据。

4.  单击“配置”选项卡并留意新的“源代码管理”字段 。

    ![][3]

    其中显示了 Git 存储库的 URL。稍后你要使用此 URL 将该存储库克隆到本地计算机。

在移动服务中启用源代码管理后，你便可以使用 Git 将存储库克隆到本地计算机。

<a name="clone-repo"></a>
## 克隆存储库安装 Git 并创建本地存储库

1.  在本地计算机上安装 Git。

    安装 Git 需要执行的步骤根据操作系统的不同而异。有关操作系统特定的分发包和安装指导，请参阅[安装 Git][]。

    > [WACOM.NOTE]
    > 在某些操作系统上，同时提供了命令行和 GUI 版本的 Git。本文中提供的说明使用命令行版本。

2.  打开一个命令行，例如 "GitBash" (Windows) 或 "Bash" (Unix Shell)。在 OS X 系统上，可以通过 "Terminal" 应用程序访问命令行。

3.  在命令行中，切换到要在其中存储脚本的目录。例如 `cd SourceControl`。

4.  使用以下命令创建新 Git 存储库的本地副本，并将 `<your_git_URL>` 替换为移动服务的 Git 存储库的 URL：

        git clone <your_git_URL>

5.  出现提示时，请键入你在移动服务中启用源代码管理时设置的用户名和密码。身份验证成功后，你将看到类似于下面的一系列响应：

        remote:Counting objects:8, done.
        remote:Compressing objects:100% (4/4), done.
        remote:Total 8 (delta 1), reused 0 (delta 0)
        Unpacking objects:100% (8/8), done.

6.  浏览到你从中运行了 `git clone` 命令的目录，并留意以下目录结构：

    ![4][]

    此时已创建了一个使用移动服务名称的新目录，它也就是数据服务的本地存储库。

7.  打开 .\\service\\table 子文件夹，可以看到，它包含一个 TodoItem.json 文件，该文件是对 TodoItem 表的操作权限的 JSON 表示形式。

    如果在此表中定义了服务器脚本，则还会出现一个或多个名为 `TodoItem.<操作>.js` 的文件，其中包含给定表操作的脚本。计划程序和自定义 API 脚本保留在与其名称相对应的不同文件夹中。有关详细信息，请参阅[源代码管理][]。

创建本地存储库后，你可以更改服务器脚本，以及将更改推送回到移动服务。

<a name="deploy-scripts"></a>
## 部署脚本将更新的脚本文件部署到移动服务

1.  浏览到 .\\service\\table 子文件夹，如果 todoitem.insert.js 文件不存在，现在请创建该文件。

2.  在文本编辑器中打开新文件 todoitem.insert.js 并在其中粘贴以下代码，然后保存更改：

        function insert(item, user, request) {
        request.execute();
        console.log(item);
        }

    此代码只是将插入的项写入日志。如果此文件已包含代码，则你只需在此文件中添加一些有效的 JavaScript 代码（例如，对 `console.log()` 的调用），然后保存更改。

3.  在 Git 命令提示符下，键入以下命令以开始跟踪新脚本文件：

        $ git add .

4.  键入以下命令以提交更改：

        $ git commit -m "updated the insert script"

5.  键入以下命令以将更改上载到远程存储库：

        $ git push origin master

    你应该会看到一系列命令，指出已将提交的内容部署到移动服务。

6.  返回管理门户，单击“数据” 选项卡，然后单击“TodoItem”表 。

    ![][5]

7.  单击“脚本”，然后选择“插入”操作 。

    ![][6]

    可以看到，显示的插入操作脚本与你刚刚上载到存储库的 JavaScript 代码相同。

<a name="use-npm"></a>
## 共享代码和模块在服务器脚本中利用共享代码和 Node.js 模块

移动服务允许你访问整个核心 Node.js 模块集，你可以通过 "require" 函数在代码中使用这些模块。移动服务还可使用不属于核心 Node.js 程序包的 Node.js 模块，你甚至可以定义自己的共享代码作为 Node.js 模块。有关创建模块的详细信息，请参阅 Node.js API 参考文档中的[模块][]。

接下来，你将要使用源代码管理和 Node.js 程序包管理器 (NPM) 将 [node-uuid][] Node.js 模块添加到移动服务。然后，你可以使用此模块为已插入项的 "uuid" 属性生成新的 GUID 值。

1.  遵照 [Node.js 网站][]中的步骤在本地计算机上安装 Node.js（如果尚未这么做）。

2.  导航到本地 Git 存储库的 `.\service` 文件夹，然后在命令提示符下运行以下命令：

        npm install node-uuid

    NPM 将在当前位置创建 `node_modules` 目录，并在 `\node-uuid` 子目录中安装 [node-uuid][] 模块。

	<div class="dev-callout"><b>说明</b>

    <p>如果 `node_modules` 已在目录层次结构中存在，NPM 将在该目录中创建 `\node-uuid` 子目录，而不是在存储库中创建新的 `node_modules`。在此情况下，你只需删除现有的 `node_modules` 目录。</p>
	</div>

3.  现在，请浏览到 .\\service\\table 子文件夹，打开 todoitem.insert.js 文件并按如下所示修改该文件：

        function insert(item, user, request) {
        var uuid = require('node-uuid');
        item.uuid = uuid.v1();
        request.execute();
        console.log(item);
        }

    此代码将在表中添加一个 uuid 列，并使用唯一的 GUID 标识符填充该列。

4.  像在前一部分中一样，在 Git 命令提示符下键入以下命令：

        $ git add .
        $ git commit -m "added node-uuid module"
        $ git push origin master

    这样就会添加新的文件，提交你的更改，并将新的 node-uuid 模块以及对 todoitem.insert.js 脚本所做的更改推送到你的移动服务。

<a name="next-steps"> </a>
## 后续步骤

完成本教程后，你便知道了如何在源代码管理中存储脚本。我们建议你了解有关如何使用服务器脚本和自定义 API 的详细信息：

-   [在移动服务中使用服务器脚本][]
    说明如何使用服务器脚本、作业计划程序和自定义 API。

-   [定义支持拉取式通知的自定义 API][]
     说明如何使用自定义 API 来支持可在 Windows 应用商店应用程序中更新动态磁贴的定期通知。

  [在移动服务中启用源代码管理]: #enable-source-control
  [安装 Git 并创建本地存储库]: #clone-repo
  [将更新的脚本文件部署到移动服务]: #deploy-scripts
  [在服务器脚本中利用共享代码和 Node.js 模块]: #use-npm
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet
  [Azure 管理门户]: https://manage.windowsazure.cn/
  []: ./media/mobile-services-store-scripts-source-control/mobile-services-selection.png
  [1]: ./media/mobile-services-store-scripts-source-control/mobile-setup-source-control.png
  [2]: ./media/mobile-services-store-scripts-source-control/mobile-source-control-credentials.png
  [3]: ./media/mobile-services-store-scripts-source-control/mobile-source-control-configure.png
  [安装 Git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
  [4]: ./media/mobile-services-store-scripts-source-control/mobile-source-local-repo.png
  [源代码管理]: http://msdn.microsoft.com/zh-cn/library/windowsazure/c25aaede-c1f0-4004-8b78-113708761643
  [5]: ./media/mobile-services-store-scripts-source-control/mobile-portal-data-tables.png
  [6]: ./media/mobile-services-store-scripts-source-control/mobile-insert-script-source-control.png
  [模块]: http://nodejs.org/api/modules.html
  [node-uuid]: https://npmjs.org/package/node-uuid
  [Node.js 网站]: http://nodejs.org/
  [在移动服务中使用服务器脚本]: /zh-cn/develop/mobile/how-to-guides/work-with-server-scripts
  [定义支持拉取式通知的自定义 API]: /zh-cn/develop/mobile/tutorials/create-pull-notifications-dotnet
