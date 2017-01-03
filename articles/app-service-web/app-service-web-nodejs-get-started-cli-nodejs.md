<properties
    pageTitle="在 Azure App Service 中开始使用 Node.js Web 应用 | Azure"
    description="学习如何将 Node.js 应用程序部署到 Azure App Service 中的 Web 应用。"
    services="app-service\web"
    documentationcenter="nodejs"
    author="cephalin"
    manager="wpickett"
    editor="" />  

<tags
    ms.assetid="fb2b90c8-02b6-4700-929b-5de9a35d67cc"
    ms.service="app-service-web"
    ms.workload="web"
    ms.tgt_pltfrm="na"
    ms.devlang="nodejs"
    ms.topic="get-started-article"
    ms.date="12/16/2016"
    wacn.date="01/03/2017"
    ms.author="cephalin" />  


# 在 Azure App Service 中 Node.js Web 应用入门
[AZURE.INCLUDE [选项卡](../../includes/app-service-web-get-started-nav-tabs.md)]

本教程说明如何创建一个简单的 [Node.js] 应用程序，然后通过 cmd.exe 或 bash 等命令行环境将它部署到 [Azure App Service]。本教程中的说明适用于任何能够运行 Node.js 的操作系统。

## <a name="prereq"></a>用于完成任务的 CLI 版本

可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/app-service-web-nodejs-get-started-cli-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- [Azure CLI 2.0（预览版）](/documentation/articles/app-service-web-nodejs-get-started/)：用于资源管理部署模型的下一代 CLI

## 先决条件
* [Node.js]
* [Bower]
* [Yeoman]
* [Git]
* [Azure CLI]
* 一个 Azure 帐户。如果你没有帐户，可以[注册试用版]。

## 创建并部署简单的 Node.js Web 应用
1. 打开所选的命令行终端并安装[适用于 Yeoman 的 Express 生成器]。
   
        npm install -g generator-express
2. 通过 `CD` 进入工作目录，并使用下列语法生成 express 应用：
   
        yo express
   
    出现提示时选择以下选项：
   
    `? Would you like to create a new directory for your project?` **Yes**  
    `? Enter directory name` **{appname}**  
    `? Select a version to install:` **MVC**  
    `? Select a view engine to use:` **Jade**  
    `? Select a css preprocessor to use (Sass Requires Ruby):` **None**  
    `? Select a database to use:` **None**  
    `? Select a build tool to use:` **Grunt**
3. 通过 `CD` 进入新应用的根目录，并将它启动以确保它在开发环境中运行：
   
        npm start
   
    在浏览器中导航到 <http://localhost:3000> 以确保可以看到 Express 主页。确认应用正常运行后，请使用 `Ctrl-C` 将它停止。
4. 更改为 ASM 模式并登录 Azure（需要 [Azure CLI](#prereq)）：
   
        azure config mode asm
        azure login -e AzureChinaCloud
   
    根据提示，在浏览器中继续使用具有 Azure 订阅的 Microsoft 帐户登录。


3. 设置应用服务的部署用户。稍后使用凭据部署代码。
   
        azure site deployment user set --username <username> --pass <password>

5. 确保仍在应用的根目录中，然后使用下一个命令，以唯一的应用名称在 Azure 中创建应用服务应用资源。例如：http://{appname}.chinacloudsites.cn
   
        azure site create --git {appname}
   
    根据提示选择要部署到的 Azure 区域。
6. 从应用程序根目录打开 ./config/config.js 文件，并将生产端口更改为 `process.env.port`；`config` 对象中的 `production` 属性应类似下面的示例：
   
        production: {
            root: rootPath,
            app: {
                name: 'express1'
            },
            port: process.env.port,
        }
   
    > [AZURE.NOTE] 
    默认情况下，Azure App Service 使用 `production` 环境变量 (`process.env.NODE_ENV="production"`) 运行 Node.js 应用程序。因此，此处的配置让 Azure 中的 Node.js 应用能够在 iisnode 侦听的默认端口上响应 Web 请求。
    >
    >

7. 打开 ./package.json，添加 `engines` 属性，[指定所需的 Node.js 版本](#version)。
   
        "engines": {
            "node": "6.9.1"
        }, 
8. 保存更改，然后使用 git 将应用部署到 Azure。出现提示时，使用前面创建的用户凭据。
   
        git add .
        git add -f config
        git commit -m "{your commit message}"
        git push azure master
   
    Express 生成器已提供 .gitignore 文件，因此 `git push` 不会占用带宽来尝试上传 node\_modules/ 目录。
9. 最后，在浏览器中启动实时 Azure 应用：
   
        azure site browse
   
    现在，应该会看到 Node.js Web 应用在 Azure App Service 中实时运行。
   
    ![浏览到已部署应用程序的示例。][deployed-express-app]  


## 更新 Node.js Web 应用
若要更新应用服务中运行的 Node.js Web 应用，只需和首次部署 Web 应用时一样运行 `git add`、`git commit` 和 `git push`。

## 应用服务如何部署 Node.js 应用
Azure App Service 使用 [iisnode] 运行 Node.js 应用。Azure CLI 和 Kudu 引擎（Git 部署）合作，让你在通过命令行开发和部署 Node.js 应用时获得顺畅的体验。

* `azure site create --git` 可识别 server.js 或 app.js 的常见 Node.js 模式，并在根目录中创建 iisnode.yml。你可以使用此文件来自定义 iisnode。
* 在 `git push azure master` 中，Kudu 可自动完成以下部署任务：
  
  * 如果 package.json 位于存储库根目录中，请运行 `npm install --production`。
  * 在 package.json 中为指向启动脚本的 iisnode 生成 Web.config（例如 server.js 或 app.js）。
  * 自定义 Web.config 以让应用程序准备好使用 Node-Inspector 进行调试。

## 使用 Node.js 框架
如果使用流行的 Node.js 框架（例如 [Sails.js][SAILSJS] 或 [MEAN.js][MEANJS]）来开发应用，则可将这些应用部署到应用服务。流行的 Node.js 框架有其特定的行为模式，并且其包依赖性不断更新。但是，应用服务提供 stdout 和 stderr 日志，方便确切了解应用中发生了什么并做出相应更改。有关详细信息，请参阅[从 iisnode 获取 stdout 和 stderr 日志](#iisnodelog)。

以下教程说明如何在应用服务中使用特定框架：

* [将 Sails.js Web 应用部署到 Azure App Service]
* [在 Azure App Service 中使用 Socket.IO 创建 Node.js 聊天应用程序]
* [如何将 io.js 与 Azure App Service Web Apps 配合使用]

## <a name="version"></a>使用特定 Node.js 引擎
与平常在 package.json 中所做的一样，可以在典型工作流中告知应用服务使用特定的 Node.js 引擎。例如：

    "engines": {
        "node": "6.9.1"
    }, 

Kudu 部署引擎按以下顺序确定要使用哪个 Node.js 引擎：

* 首先，查看 iisnode.yml 以确认是否已指定 `nodeProcessCommandLine`。如果是，则使用它。
* 接下来，查看 package.json 以确认是否已在 `engines` 对象中指定 `"node": "..."`。如果是，则使用它。
* 默认情况下会选择默认的 Node.js 版本。

有关 Azure App Service 中所有受支持的 Node.js/NPM 版本的更新列表，请访问针对应用的以下 URL：

    https://<app_name>.scm.chinacloudsites.cn/api/diagnostics/runtime

> [AZURE.NOTE]
建议显式定义所需的 Node.js 引擎。默认的 Node.js 版本可以更改，并且可能会因为默认的 Node.js 版本不适合应用而在 Azure Web 应用中收到错误。
> 
> 

## <a name="iisnodelog"></a>从 iisnode 获取 stdout 和 stderr 日志
若要读取 iisnode 日志，请执行以下步骤。

> [AZURE.NOTE]
完成这些步骤之后，可能要等到发生错误时才会有日志文件。
> 
> 

1. 打开 Azure CLI 提供的 iisnode.yml 文件。
2. 设置以下两个参数：
   
        loggingEnabled: true
        logDirectory: iisnode
   
    结合这两个参数可告知应用服务中的 iisnode 将其 stdout 和 stderror 输出放在 D:\\home\\site\\wwwroot**iisnode** 目录中。
3. 保存更改，然后使用以下 Git 命令将更改推送到 Azure：
   
        git add .
        git commit -m "{your commit message}"
        git push azure master
   
    现已配置 iisnode。接下来的步骤演示如何访问这些日志。
4. 在浏览器中访问应用的 Kudu 调试控制台，位置为：
   
        https://{appname}.scm.chinacloudsites.cn/DebugConsole 
   
    此 URL 不同于 Web 应用 URL，它的 DNS 名称中包含“ *.scm* ”。如果在 URL 中省略此部分，你将收到 404 错误。
5. 导航到 D:\\home\\site\\wwwroot\\iisnode
   
    ![导航到 iisnode 日志文件所在的位置。][iislog-kudu-console-find]
6. 单击要读取的日志的“编辑”图标。如果需要，也可以单击“下载”或“删除”。
   
    ![打开 iisnode 日志文件。][iislog-kudu-console-open]  

   
    现在可以查看日志，帮助调试应用服务部署。
   
    ![检查 iisnode 日志文件。][iislog-kudu-console-read]  


## 使用 Node-Inspector 调试应用
如果使用 Node-Inspector 来调试 Node.js 应用，可将它用于实时应用服务应用。Node-Inspector 已预先安装在应用服务的 iisnode 安装中。如果你通过 Git 部署，则从 Kudu 自动生成的 Web.config 已包含启用 Node-Inspector 所需的所有配置。

若要启用 Node-Inspector，请遵循以下步骤：

1. 打开位于存储库根目录中的 iisnode.yml，并指定以下参数：
   
        debuggingEnabled: true
        debuggerExtensionDll: iisnode-inspector.dll
2. 保存更改，然后使用以下 Git 命令将更改推送到 Azure：
   
        git add .
        git commit -m "{your commit message}"
        git push azure master
3. 现在，只需在 URL 中添加 /debug 以导航到 package.json 中的启动脚本指定的应用启动文件。例如，
   
        http://{appname}.chinacloudsites.cn/server.js/debug
   
    或者，
   
        http://{appname}.chinacloudsites.cn/app.js/debug

## 更多资源
* [在 Azure 应用程序中指定 Node.js 版本](/documentation/articles/nodejs-specify-node-version-azure-apps/)
* [Azure 上的 Node.js 应用程序的最佳实践和故障排除指南](/documentation/articles/app-service-web-nodejs-best-practices-and-troubleshoot-guide/)
* [如何在 Azure App Service 中调试 Node.js Web 应用](/documentation/articles/web-sites-nodejs-debug/)
* [将 Node.js 模块与 Azure 应用程序一起使用](/documentation/articles/nodejs-use-node-modules-azure-apps/)
* [Azure App Service Web 应用：Node.js](http://blogs.msdn.com/b/silverlining/archive/2012/06/14/windows-azure-websites-node-js.aspx)
* [Node.js 开发人员中心](/develop/nodejs/)
* [在 Azure App Service 中开始使用 Web 应用](/documentation/articles/app-service-web-get-started/)
* [探索神秘无比的 Kudu 调试控制台]

<!-- URL List -->

[Azure CLI]: /documentation/articles/xplat-cli-install/
[Azure App Service]: /documentation/articles/app-service-value-prop-what-is/
[Bower]: http://bower.io/
[在 Azure App Service 中使用 Socket.IO 创建 Node.js 聊天应用程序]: /documentation/articles/web-sites-nodejs-chat-app-socketio/
[将 Sails.js Web 应用部署到 Azure App Service]: /documentation/articles/app-service-web-nodejs-sails/
[探索神秘无比的 Kudu 调试控制台]: /documentation/articles/aog-web-app-diagnostics-kudu/
[适用于 Yeoman 的 Express 生成器]: https://github.com/petecoop/generator-express
[Git]: http://www.git-scm.com/downloads
[如何将 io.js 与 Azure App Service Web Apps 配合使用]: /documentation/articles/web-sites-nodejs-iojs/
[iisnode]: https://github.com/tjanczuk/iisnode/wiki
[MEANJS]: http://meanjs.org/
[Node.js]: http://nodejs.org
[SAILSJS]: http://sailsjs.org/
[注册试用版]: /pricing/1rmb-trial/
[web app]: /documentation/articles/app-service-web-overview/
[Yeoman]: http://yeoman.io/

<!-- IMG List -->


[deployed-express-app]: ./media/app-service-web-nodejs-get-started/deployed-express-app.png
[iislog-kudu-console-find]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-navigate.png
[iislog-kudu-console-open]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-open.png
[iislog-kudu-console-read]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-read.png

<!---HONumber=Mooncake_1226_2016-->