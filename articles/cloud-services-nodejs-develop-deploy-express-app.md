<properties linkid="dev-nodejs-basic-web-app-with-express" urlDisplayName="Web App with Express" pageTitle="Web App with Express (Node.js) - Azure Tutorial" metaKeywords="Azure Node.js hello world tutorial, Azure Node.js hello world, Azure Node.js Getting Started tutorial, Azure Node.js tutorial, Azure Node.js Express tutorial" description="An tutorial that builds on the cloud service tutorial, and demonstrates how to use the Express module." metaCanonical="" services="cloud-services" documentationCenter="Node.js" title="Build a Node.js web application using Express on an Azure Cloud Service" authors="larryfr" solutions="" manager="" editor="" />

# 使用 Express 在 Azure 云服务中生成 Node.js Web 应用程序

Node.js 包含核心运行时中最小的一个功能集。开发人员在开发 Node.js 应用程序时经常会使用第三方模块来提供额外功能。在本教程中，你将使用 [Express][Express] 模块新建一个应用程序，该模块可提供用于创建 Node.js Web 应用程序的 MVC 框架。

以下是已完成应用程序的屏幕快照：

![显示“Welcome to Express in Azure”的 Web 浏览器][显示“Welcome to Express in Azure”的 Web 浏览器]

## 创建云服务项目

执行下列步骤可新建一个名为“expressapp”的云服务项目：

1.  在“开始”菜单或“开始”屏幕中，搜索 **Azure PowerShell**。最后，右键单击“Azure PowerShell”并选择“以管理员身份运行”。

    ![Azure PowerShell 图标][Azure PowerShell 图标]

    [WACOM.INCLUDE [install-dev-tools][install-dev-tools]]

2.  将目录切换到 **c:\\node** 目录，然后输入下列命令以新建一个名为 **expressapp** 的解决方案和名为 **WebRole1** 的 Web 角色：

        PS C:\node> New-AzureServiceProject expressapp
        PS C:\Node> Add-AzureNodeWebRole

## 安装 Express

1.  通过发出以下命令安装 Express 生成器：

        PS C:\node\expressapp> npm install express-generator -g

    npm 命令的输出应与以下结果类似。

    ![Windows PowerShell 显示 npm install express 命令的输出。][Windows PowerShell 显示 npm install express 命令的输出。]

2.  将目录切换到 **WebRole1** 目录，然后使用 express 命令生成一个新的应用程序：

        PS C:\node\expressapp\WebRole1> express

    系统将提示你是否覆盖之前创建的应用程序。输入 **y** 或 **yes** 以继续操作。Express 将生成用于生成应用程序的 app.js 文件和一个文件夹结构。

    ![express 命令的输出][express 命令的输出]

3.  若要安装在 package.json 文件中定义的其他依赖项，请输入以下命令：

        PS C:\node\expressapp\WebRole1> npm install

    ![npm install 命令的输出][npm install 命令的输出]

4.  使用以下命令将 **bin/www** 文件复制到 **server.js**，这样，云服务便可以找到此应用程序的入口点。

        PS C:\node\expressapp\WebRole1> copy bin/www server.js

    完成此命令后，WebRole1 目录中应会生成一个 **server.js** 文件。

5.  修改 **server.js** 以删除以下行中的“.”字符之一。

        var app = require('../app');

    完成此项修改后，该行应如下所示。

        var app = require('./app');

    之所以需要进行此更改，是因为我们已将文件（以前的 **bin/www**）移到了所需应用程序文件所在的同一个目录。完成此更改后，请保存 **server.js** 文件。

6.  使用以下命令在 Windows Azure 模拟器中运行应用程序：

        PS C:\node\expressapp\WebRole1> Start-AzureEmulator -launch

    ![包含 Welcome to Express 的网页。][包含 Welcome to Express 的网页。]

## 修改视图

现在，你将修改视图以显示消息“Welcome to Express in Azure”。

1.  请输入以下命令来打开 index.jade 文件：

        PS C:\node\expressapp\WebRole1> notepad views/index.jade

    ![index.jade 文件的内容。][index.jade 文件的内容。]

    Jade 是 Express 应用程序使用的默认视图引擎。有关 Jade 视图引擎的详细信息，请参阅 [][]<http://jade-lang.com></a>。

2.  通过追加 **in Azure** 来修改最后一行文本。

    ![index.jade 文件，最后一行的内容为：p Welcome to \#{title} in Azure][index.jade 文件，最后一行的内容为：p Welcome to \#{title} in Azure]

3.  保存文件并退出记事本。

4.  刷新浏览器，你将看到所做的更改。

    ![浏览器窗口，其中的页面包含 Welcome to Express in Azure][浏览器窗口，其中的页面包含 Welcome to Express in Azure]

测试应用程序后，请使用 **Stop-AzureEmulator** cmdlet 停止模拟器。

## 将应用程序发布到 Azure

在 Azure PowerShell 窗口中，可使用 **Publish-AzureServiceProject** cmdlet 将应用程序部署到云服务

    PS C:\node\expressapp\WebRole1> Publish-AzureServiceProject -ServiceName myexpressapp -Location "East US" -Launch

在部署操作完成后，浏览器将打开并显示该网页。

![Web 浏览器中显示 Express 页。URL 指示它现在托管在 Azure 上。][显示“Welcome to Express in Azure”的 Web 浏览器]

  [Express]: http://expressjs.com/
  [显示“Welcome to Express in Azure”的 Web 浏览器]: ./media/cloud-services-nodejs-develop-deploy-express-app/node36.png
  [Azure PowerShell 图标]: ./media/cloud-services-nodejs-develop-deploy-express-app/azure-powershell-start.png
  [install-dev-tools]: ../includes/install-dev-tools.md
  [Windows PowerShell 显示 npm install express 命令的输出。]: ./media/cloud-services-nodejs-develop-deploy-express-app/express-g.png
  [express 命令的输出]: ./media/cloud-services-nodejs-develop-deploy-express-app/node23.png
  [npm install 命令的输出]: ./media/cloud-services-nodejs-develop-deploy-express-app/node26.png
  [包含 Welcome to Express 的网页。]: ./media/cloud-services-nodejs-develop-deploy-express-app/node28.png
  [index.jade 文件的内容。]: ./media/cloud-services-nodejs-develop-deploy-express-app/getting-started-19.png
  []: http://jade-lang.com
  [index.jade 文件，最后一行的内容为：p Welcome to \#{title} in Azure]: ./media/cloud-services-nodejs-develop-deploy-express-app/node31.png
  [浏览器窗口，其中的页面包含 Welcome to Express in Azure]: ./media/cloud-services-nodejs-develop-deploy-express-app/node32.png
