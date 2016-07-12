<properties
	pageTitle="Azure 中的 Node.js Web 应用入门"
	description="学习如何将 Node.js 应用程序部署到 Azure 中的 Web 应用。"
	services="app-service\web"
	documentationCenter="nodejs"
	authors="cephalin"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="06/01/2016"
	wacn.date="07/04/2016"/>

# Azure 中的 Node.js Web 应用入门

[AZURE.INCLUDE [选项卡](../includes/app-service-web-get-started-nav-tabs.md)]

本教程说明如何创建一个简单的 [Node.js][NODEJS] 应用程序，然后通过 cmd.exe 或 bash 等命令行环境将它部署到 [Azure Web 应用]。本教程中的说明适用于任何能够运行 Node.js 的操作系统。

<a name="prereq"></a>
## 先决条件

- **Node.js**（[单击此处进行安装][NODEJS]）
- **Bower**（[单击此处进行安装][BOWER]）
- **Yeoman**（[单击此处进行安装][YEOMAN]）
- **Git**（[单击此处进行安装][GIT]）
- **Azure CLI**（[单击此处进行安装][Azure CLI]）
- 一个 Azure 帐户。如果你没有帐户，可以[注册试用版]。

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
    
1. 如下所示登录到 Azure（为此需要 [Azure CLI](#prereq)）：

        azure login -e AzureChinaCloud -u <your username>

    根据提示，在浏览器中继续使用具有 Azure 订阅的 Microsoft 帐户登录。

2. 确保你仍在应用的根目录中，然后使用下一个命令，以唯一的应用名称在 Azure 中创建 Azure Web 应用资源；例如：http://{appname}.chinacloudsites.cn

        azure site create --git {appname}

    根据提示选择要部署到的 Azure 区域。如果你从未设置 Azure 订阅的 Git/FTP 部署凭据，则系统还会提示你创建凭据。

3. 从应用程序根目录打开 ./config/config.js 文件，并将生产端口更改为 `process.env.port`；`config` 对象中的 `production` 属性应类似下面的示例。

        production: {
            root: rootPath,
            app: {
                name: 'express1'
            },
            port: process.env.port,
        }

    这样，你的 Node.js 应用便可以在 iisnode 侦听的默认端口上响应 Web 请求。
    
4. 保存更改，然后使用 git 将应用部署到 Azure：

        git add .
        git commit -m "{your commit message}"
        git push azure master

    Express 生成器已提供 .gitignore 文件，因此 `git push` 不会占用带宽来尝试上载 node\_modules/ 目录。

5. 最后，在浏览器中启动实时 Azure 应用：

        azure site browse

    现在，你应会看到 Node.js Web 应用在 Azure Web 应用中实时运行。
    
    ![浏览到已部署应用程序的示例。][deployed-express-app]

## 更新 Node.js Web 应用

若要更新 Azure Web 应用中运行的 Node.js Web 应用，只需和首次部署 Web 应用时一样运行 `git add`、`git commit` 和 `git push`。
     
## Azure 如何部署 Node.js 应用

Azure 使用 [iisnode] 来运行 Node.js 应用。Azure CLI 和 Kudu 引擎（Git 部署）合作，让你在通过命令行开发和部署 Node.js 应用时获得顺畅的体验。

- `azure site create --git` 可识别 server.js 或 app.js 的常见 Node.js 模式，并在根目录中创建 iisnode.yml。你可以使用此文件来自定义 iisnode。
- 在 `git push azure master` 中，Kudu 可自动完成以下部署任务：

    - 如果 package.json 位于存储库根目录中，请运行 `npm install --production`。
    - 在 package.json 中为指向启动脚本的 iisnode 生成 Web.config（例如 server.js 或 app.js）。
    - 自定义 Web.config 以让应用程序准备好使用 Node-Inspector 进行调试。
    
## 使用 Node.js 框架

如果使用了流行的 Node.js 框架（例如 [Sails.js][SAILSJS] 或 [MEAN.js][MEANJS]）来开发应用，则可将这些应用部署到 Azure Web 应用。流行的 Node.js 框架有其特定的行为模式，并且其包依赖性不断更新。但是，Azure 为你提供 stdout 和 stderr 日志，让你确切地了解应用中发生了什么并做出相应的更改。有关详细信息，请参阅[从 iisnode 获取 stdout 和 stderr 日志](#iisnodelog)。

以下教程说明如何在 Azure Web 应用中使用特定框架：

- [Deploy a Sails.js web app to Azure（将 Sails.js Web 应用部署到 Azure）]
- [Create a Node.js chat application with Socket.IO in Azure Web App（在 Azure Web 应用使用 Socket.IO 创建 Node.js 聊天应用程序）]
- [How to use io.js with Azure Web Apps（如何将 io.js 与 Azure Web 应用配合使用）]

## 使用特定的 Node.js 引擎

与平常在 package.json 中所做的一样，你可以在典型工作流中告知 Azure 使用特定的 Node.js 引擎。例如：

    "engines": {
        "node": "5.5.0"
    }, 

Kudu 部署引擎按以下顺序确定要使用哪个 Node.js 引擎：

- 首先，查看 iisnode.yml 以确认是否已指定 `nodeProcessCommandLine`。如果是，则使用它。
- 接下来，查看 package.json 以确认是否已在 `engines` 对象中指定 `"node": "..."`。如果是，则使用它。
- 默认情况下会选择默认的 Node.js 版本。

##<a name="iisnodelog"></a> 从 iisnode 获取 stdout 和 stderr 日志

若要读取 iisnode 日志，请使用以下步骤。

> [AZURE.NOTE] 完成这些步骤之后，可能要等到发生错误时才会有日志文件。

1. 打开 Azure CLI 提供的 iisnode.yml 文件。

2. 设置以下两个参数：

        loggingEnabled: true
        logDirectory: iisnode
    
    这两个参数相结合，告诉 App Service 中的 iisnode 要将其 stdout 和 stderror 输出放在 D:\\home\\site\\wwwroot**iisnode** 目录中。

3. 保存更改，然后使用以下 Git 命令将更改推送到 Azure：

        git add .
        git commit -m "{your commit message}"
        git push azure master
   
   现已配置 iisnode。接下来的步骤演示如何访问这些日志。
     
4. 使用 FTP，并导航到 D:\\home\\site\\wwwroot\\iisnode

6. 单击要读取的日志的“编辑”图标。如果需要，也可以单击“下载”或“删除”。

    现在可以查看日志以帮助调试 Azure 部署。

## 使用 Node-Inspector 调试应用

如果你使用 Node-Inspector 来调试 Node.js 应用，可将它用于实时 Azure Web 应用。Node-Inspector 已预先安装在 Azure Web 应用的 iisnode 安装中。如果你通过 Git 部署，则从 Kudu 自动生成的 Web.config 已包含启用 Node-Inspector 所需的所有配置。

若要启用 Node-Inspector，请遵循以下步骤：

1. 打开位于存储库根目录中的 iisnode.yml，并指定以下参数： 

        debuggingEnabled: true
        debuggerExtensionDll: iisnode-inspector.dll

3. 保存更改，然后使用以下 Git 命令将更改推送到 Azure：

        git add .
        git commit -m "{your commit message}"
        git push azure master
   
4. 现在，只需在 URL 中添加 /debug 以导航到 package.json 中的启动脚本指定的应用启动文件。例如，

        http://{appname}.chinacloudsites.cn/server.js/debug
    
    或者，
    
        http://{appname}.chinacloudsites.cn/app.js/debug

## 更多资源

- [在 Azure 应用程序中指定 Node.js 版本](/documentation/articles/nodejs-specify-node-version-azure-apps/)
- [如何在 Azure 中调试 Node.js Web 应用](/documentation/articles/web-sites-nodejs-debug/)
- [将 Node.js 模块与 Azure 应用程序一起使用](/documentation/articles/nodejs-use-node-modules-azure-apps/)
- [Azure Web Apps: Node.js](http://blogs.msdn.com/b/silverlining/archive/2012/06/14/windows-azure-websites-node-js.aspx)
- [Node.js 开发人员中心](/develop/nodejs/)
- [Azure 中的 Web 应用入门](/documentation/articles/app-service-web-get-started/)

<!-- URL List -->

[Azure CLI]: /documentation/articles/xplat-cli-install/
[Azure Web 应用]: /documentation/services/web-sites
[activate your Visual Studio subscriber benefits]: /pricing/1rmb-trial/
[BOWER]: http://bower.io/
[Create a Node.js chat application with Socket.IO in Azure Web App（在 Azure Web 应用使用 Socket.IO 创建 Node.js 聊天应用程序）]: /documentation/articles/web-sites-nodejs-chat-app-socketio/
[Deploy a Sails.js web app to Azure（将 Sails.js Web 应用部署到 Azure）]: /documentation/articles/app-service-web-nodejs-sails/
[探索神秘无比的 Kudu 调试控制台]: /documentation/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[适用于 Yeoman 的 Express 生成器]: https://github.com/petecoop/generator-express
[GIT]: http://www.git-scm.com/downloads
[How to use io.js with Azure Web Apps（如何将 io.js 与 Azure Web 应用配合使用）]: /documentation/articles/web-sites-nodejs-iojs/
[iisnode]: https://github.com/tjanczuk/iisnode/wiki
[MEANJS]: http://meanjs.org/
[NODEJS]: http://nodejs.org
[SAILSJS]: http://sailsjs.org/
[注册试用版]: /pricing/1rmb-trial/
[Web 应用]: /home/features/web-site
[YEOMAN]: http://yeoman.io/

<!-- IMG List -->

[deployed-express-app]: ./media/app-service-web-nodejs-get-started/deployed-express-app.png
[iislog-kudu-console-find]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-navigate.png
[iislog-kudu-console-open]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-open.png
[iislog-kudu-console-read]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-read.png

<!---HONumber=Mooncake_0627_2016-->