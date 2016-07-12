<properties
	pageTitle="将 Sails.js Web 应用部署到 Azure"
	description="学习如何将 Node.js 应用程序部署到 Azure Web 应用。本教程说明如何部署 Sails.js Web 应用。"
	services="app-service\web"
	documentationCenter="nodejs"
	authors="cephalin"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="03/31/2016"
	wacn.date="05/16/2016"/>

# 将 Sails.js Web 应用部署到 Azure

本教程说明如何将 Sails.js 应用部署到 Azure Web 应用。在此过程中，你可以搜集一些有关如何配置 Node.js 应用以在 Azure Web 应用中运行的一般知识。

## 先决条件

- Node.js。安装二进制文件可从[此处](https://nodejs.org/)获取。
- Sails.js。[此处](http://sailsjs.org/get-started)提供了安装说明。
- Sails.js 的实践知识。本教程并非旨在帮助你解决运行 Sail.js 相关的一般性问题。
- Git。安装二进制文件可从[此处](http://www.git-scm.com/downloads)获取。
- Azure CLI。[此处](/documentation/articles/xplat-cli-install/)提供了安装说明。
- 一个 Azure 帐户。如果你没有帐户，可以[注册1元试用帐户](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

## 步骤 1：在开发环境中创建 Sails.js 应用

首先，遵循以下步骤快速创建默认的 Sails.js 应用：

1. 打开所选的命令行终端，并使用 `CD` 切换到工作目录。

2. 创建新的 Sails.js 应用并将它运行：

        sails new <appname>
        cd <appname>
        sails lift

    确保可以导航到 http://localhost:1377 上的默认主页。

## 步骤 2：在 Azure 中创建 Azure Web 应用资源

接下来，创建 App Service 应用资源。稍后要将 Sails.js 应用部署到其中。

1. 在同一个终端中登录到 Azure，如下所示：

        azure login -e AzureChinaCloud -u <your username>

    根据提示，在浏览器中继续使用具有 Azure 订阅的 Microsoft 帐户登录。

2. 请确保你仍处于 Sails.js 项目的根目录中。在 Azure 中以下一个命令创建具有唯一应用名称的 Azure Web 应用资源。Web 应用的 URL 是 http://&lt;appname>.chinacloudsites.cn。

        azure site create --git <appname>

    根据提示选择要部署到的 Azure 区域。如果你从未设置 Azure 订阅的 Git/FTP 部署凭据，则系统还会提示你创建凭据。

    创建 Azure Web 应用资源之后：

    - Sails.js 应用已经过 Git 初始化，
    - 本地 Git 初始化的存储库已作为 Git 远程应用连接到新的 Azure Web 应用（名称正好为“azure”），并且
    - 在根目录中创建了 iisnode.yml 文件。可以使用此文件来配置 [iisnode](https://github.com/tjanczuk/iisnode)，Azure 可以使用它来运行 Node.js 应用。

## 步骤 3：设置并部署 Sails.js 应用

 在 Azure 中使用 Sails.js 应用需要执行三个主要步骤：

 - 配置你的应用，使它在 Azure 中运行
 - 将应用部署到 Azure
 - 阅读 stderr 和 stdout 日志来排查任何部署问题

执行以下步骤:

1. 在根目录中打开新的 iisnode.yml 文件并添加以下两行：

        loggingEnabled: true
        logDirectory: iisnode

    现在已针对 iisnode 启用日志记录。有关具体操作的详细信息，请参阅[从 iisnode 获取 stdout 和 stderr 日志](/documentation/articles/app-service-web-nodejs-get-started/#iisnodelog)。

2. 打开 config/env/production.js 来配置生产环境，并设置 `port` 和 `hookTimeout`：

        module.exports = {

            // Use process.env.port to handle web requests to the default HTTP port
            port: process.env.port,
            // Increase hooks timout to 30 seconds
            // This avoids the Sails.js error documented at https://github.com/balderdashy/sails/issues/2691
            hookTimeout: 30000,

            ...
        };

    可以在 [Sails.js 文档](http://sailsjs.org/documentation/reference/configuration/sails-config)中找到这些配置设置的文档。

    接下来，需确保 [Grunt](https://www.npmjs.com/package/grunt) 与 Azure 的网络驱动器兼容。编写本文时，Grunt 可能会生成[“ENOTSUP: 套接字不支持操作”错误](https://github.com/isaacs/node-glob/issues/205)，因为它目前使用了网络驱动器不支持的已过时 [glob](https://www.npmjs.com/package/glob) 包 (v3.1.21)。[](https://github.com/isaacs/node-glob/issues/205)接下来的步骤演示如何让 Grunt 使用 [glob v5.0.14 或更高版本](https://github.com/isaacs/node-glob/commit/bf3381e90e283624fbd652835e1aefa55d45e2c7)。

3. 由于创建应用时 `npm install` 已在运行，因此，请在项目根目录中生成 npm-shrinkwrap.json：

        npm shrinkwrap

4. 打开 npm-shrinkwrap.json、找到 `"grunt":` 的 json，然后添加所需 glob 版本的依赖项。完成的 json 应如下所示：

        "grunt": {
            "version": "0.4.5",
            "from": "grunt@0.4.5",
            "resolved": "https://registry.npmjs.org/grunt/-/grunt-0.4.5.tgz",
            "dependencies": {
                "glob": {
                    "version": "5.0.14",
                    "from": "glob@5.0.14",
                    "resolved": "https://registry.npmjs.org/glob/-/glob-5.0.14.tgz"
                }
            }
        },

5. 通过搜索 `"glob":` 找到对 glob 的所有引用。如果任一引用为 v3.1.21 或更低版本，请将 json 更改为：

        "glob": {
            "version": "5.0.14",
            "from": "glob@5.0.14",
            "resolved": "https://registry.npmjs.org/glob/-/glob-5.0.14.tgz"
        }

6. 保存更改并测试更改，以确保应用仍在本地运行：

        npm install
        sails lift

4. 现在，使用 git 将应用部署到 Azure：

        git add .
        git commit -m "<your commit message>"
        git push azure master

5. 最后，只需在浏览器中启动实时 Azure 应用：

        azure site browse

    现在，你应会看到相同的 Sails.js 主页。
    
## 排查部署问题

如果 Sails.js 应用程序在 Azure Web 应用中由于某种原因而失败，请查找 stderr 日志，以帮助进行故障排除。
有关详细信息，请参阅[从 iisnode 获取 stdout 和 stderr 日志](/documentation/articles/app-service-web-nodejs-get-started/#iisnodelog)。
如果应用成功启动，stdout 日志应显示你熟悉的消息：

                .-..-.

    Sails              <|    .-..-.
    v0.12.1             |\
                        /|.\
                        / || \
                    ,'  |'  \
                    .-'.-==|/_--'
                    `--'-------' 
    __---___--___---___--___---___--___
    ____---___--___---___--___---___--___-__

    Server lifted in `D:\home\site\wwwroot`
    To see your app, visit http://localhost:\\.\pipe\a76e8111-663e-449d-956e-5c5deff2d304
    To shut down Sails, press <CTRL> + C at any time.

## 连接到 Azure 中的数据库

若要连接到 Azure 中的数据库，可以在 Azure 中创建所选的数据库，例如 Azure SQL 数据库、MySQL、MongoDB、Azure (Redis) 缓存等，并使用相应的[数据存储适配器](https://github.com/balderdashy/sails#compatibility)连接到该数据库。本部分中的步骤说明如何连接到 Azure SQL 数据库。

1. 遵循[此处](/documentation/articles/sql-database-get-started/)的教程，在新的 SQL Server 中创建空白的 Azure SQL 数据库。默认防火墙设置将允许 Azure 服务（例如 Azure Web 应用）连接到该数据库。

2. 从命令行终端安装 SQL Server 适配器：

        npm install sails-sqlserver --save

    由于已更改 package.json，因此必须重新生成 npm-shrinkwrap.json。接下来，请执行以下操作。
    
3. 删除 node\_modules/ 目录。

4. 运行 `npm shrinkwrap`。

5. 再次打开 npm-shrinkwrap.json，并按上一部分中所述更新 `glob` 包版本。

    现在，返回到主要任务。
        
3. 打开 config/connections.js 并将以下 json 添加到适配器列表：

        sqlserver: {
            adapter: 'sails-sqlserver',
            user: process.env.dbuser,
            password: process.env.dbpassword,
            host: process.env.sqlserver, 
            database: process.env.dbname,
            options: {
                encrypt: true   // use this for Azure databases
            }
        },

4. 需要在 Azure Web 应用中设置每个环境变量 (`process.env.*`)。为此，请从终端运行以下命令：

        azure site appsetting add dbuser="<database server administrator>"
        azure site appsetting add dbpassword="<database server password>"
        azure site appsetting add sqlserver="<database server name>.database.chinacloudapi.cn"
        azure site appsetting add dbname="<database name>"
        
4. 打开 config/env/production.js 来配置生产环境，并设置 `models` JSON对象中的 `connection` 和 `migrate`：

        models: {
            connection: 'sqlserver',
            migrate: 'alter'
        },

4. 像平时一样从终端[生成](http://sailsjs.org/documentation/reference/command-line-interface/sails-generate) Sails.js [蓝图 API](http://sailsjs.org/documentation/concepts/blueprints)。例如：

         sails generate api mywidget
     
5. 保存所有更改，将更改推送到 Azure，并浏览到应用以确保它仍能正常工作。

        git add .
        git commit -m "<your commit message>"
        git push azure master
        azure site browse

6. 现在，访问刚刚在浏览器中创建的蓝图 API。例如：

        http://<appname>.chinacloudsites.cn/widget/create
    
    此 API 应会在浏览器窗口中将创建的条目返回给你：
    
        {"id":1,"createdAt":"2016-03-28T23:08:01.000Z","updatedAt":"2016-03-28T23:08:01.000Z"}

## 更多资源

- [Azure 中的 Node.js Web 应用入门](/documentation/articles/app-service-web-nodejs-get-started/)
- [将 Node.js 模块与 Azure 应用程序一起使用](/documentation/articles/nodejs-use-node-modules-azure-apps/)

<!---HONumber=Mooncake_0509_2016-->