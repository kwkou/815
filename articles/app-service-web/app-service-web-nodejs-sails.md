<properties
	pageTitle="将 Sails.js Web 应用部署到 Azure App Service"
	description="学习如何将 Node.js 应用部署到 Azure App Service。本教程说明如何部署 Sails.js Web 应用。"
	services="app-service\web"
	documentationCenter="nodejs"
	authors="cephalin"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="07/19/2016"
	wacn.date=""/>

# 将 Sails.js Web 应用部署到 Azure App Service

本教程说明如何将 Sails.js 应用部署到 Azure App Service。在此过程中，可以搜集一些有关如何将 Node.js 应用配置为在应用服务中运行的一般知识。

## 先决条件

- [Node.js](https://nodejs.org/)。
- [Sails.js](http://sailsjs.org/get-started)。
- Sails.js 的实践知识。本教程并非旨在帮助你解决运行 Sail.js 相关的一般性问题。
- [Git](http://www.git-scm.com/downloads)。
- [Azure CLI](/documentation/articles/xplat-cli-install/)。
- 一个 Azure 帐户。如果你没有帐户，可以[注册试用版](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

## 步骤 1：在开发环境中创建 Sails.js 应用

首先，遵循以下步骤快速创建默认的 Sails.js 应用：

1. 打开所选的命令行终端，并使用 `CD` 切换到工作目录。

2. 创建新的 Sails.js 应用并将它运行：

        sails new <appname>
        cd <appname>
        sails lift

    确保可以导航到 http://localhost:1377 上的默认主页。

## 步骤 2：在 Azure 中创建应用服务应用资源。

接下来，创建 App Service 应用资源。稍后要将 Sails.js 应用部署到其中。

1. 如下所示，在同一个终端中跳转到 ASM 模式，并登录到 Azure：

        azure config mode asm
        azure login -e AzureChinaCloud

    根据提示，在浏览器中继续使用具有 Azure 订阅的 Microsoft 帐户登录。

2. 请确保你仍处于 Sails.js 项目的根目录中。在 Azure 中使用下一个命令创建具有唯一应用名称的应用服务应用资源。Web 应用的 URL 为 http://&lt;appname>.chinacloudsites.cn。

        azure site create --git <appname>

    根据提示选择要部署到的 Azure 区域。如果你从未设置 Azure 订阅的 Git/FTP 部署凭据，则系统还会提示你创建凭据。

    创建应用服务应用资源后：

    - Sails.js 应用已经过 Git 初始化，
    - 本地 Git 初始化的存储库已作为 Git 远程应用连接到新的应用服务应用（名称正好为“azure”），并且
    - 在根目录中创建了 iisnode.yml 文件。可以使用此文件来配置 [iisnode](https://github.com/tjanczuk/iisnode)，应用服务可以使用它来运行 Node.js 应用。

## 步骤 3：设置并部署 Sails.js 应用

 在应用服务中使用 Sails.js 应用需要执行三个主要步骤：

 - 配置应用，使其在应用服务中运行
 - 将其部署到应用服务
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

    接下来，需确保 [Grunt](https://www.npmjs.com/package/grunt) 与 Azure 的网络驱动器兼容。版本低于 1.0.0 的 Grunt 使用过时的 [glob](https://www.npmjs.com/package/glob) 包（版本低于 5.0.14），该包不支持网络驱动器。

3. 打开 package.json，将 `grunt` 的版本更改为 `1.0.0`，并删除所有 `grunt-*` 包。`dependencies` 属性应如下所示：

        "dependencies": {
            "ejs": "<leave-as-is>",
            "grunt": "1.0.0",
            "include-all": "<leave-as-is>",
            "rc": "<leave-as-is>",
            "sails": "<leave-as-is>",
            "sails-disk": "<leave-as-is>",
            "sails-sqlserver": "<leave-as-is>"
        },

6. 保存更改并测试更改，以确保应用仍在本地运行。若要执行此操作，请删除 `node_modules` 文件夹，然后运行：

        npm install
        sails lift

4. 现在，使用 git 将应用部署到 Azure：

        git add .
        git commit -m "<your commit message>"
        git push azure master

5. 最后，只需在浏览器中启动实时 Azure 应用：

        azure site browse

    现在，你应会看到相同的 Sails.js 主页。
    
    ![](./media/app-service-web-nodejs-sails/sails-in-azure.png)

## 排查部署问题

如果 Sails.js 应用在应用服务中由于某种原因而失败，请查找 stderr 日志，以帮助进行故障排除。有关详细信息，请参阅[从 iisnode 获取 stdout 和 stderr 日志](/documentation/articles/app-service-web-nodejs-get-started/#iisnodelog)。如果应用成功启动，stdout 日志应显示你熟悉的消息：

                .-..-.

    Sails              <|    .-..-.
    v0.12.3             |\
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

可以控制 [config/log.js](http://sailsjs.org/#!/documentation/concepts/Logging) 文件中 stdout 日志的粒度。

## 连接到 Azure 中的数据库

若要连接到 Azure 中的数据库，可以在 Azure 中创建所选的数据库，例如 Azure SQL 数据库、MySQL、MongoDB、Azure (Redis) 缓存等，并使用相应的[数据存储适配器](https://github.com/balderdashy/sails#compatibility)连接到该数据库。本部分中的步骤说明如何连接到 Azure SQL 数据库。

1. 遵循[此处](/documentation/articles/sql-database-get-started/)的教程，在新的 SQL Server 中创建空白的 Azure SQL 数据库。默认防火墙设置允许 Azure 服务（例如应用服务）连接到该数据库。

2. 从命令行终端安装 SQL Server 适配器：

        npm install sails-sqlserver --save

3. 打开 config/connections.js 并将以下连接对象添加到列表：

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

4. 需要在应用服务中设置每个环境变量 (`process.env.*`)。为此，请从终端运行以下命令：

        azure site appsetting add dbuser="<database server administrator>"
        azure site appsetting add dbpassword="<database server password>"
        azure site appsetting add sqlserver="<database server name>.database.chinacloudapi.cn"
        azure site appsetting add dbname="<database name>"
        
    将设置放在 Azure 应用设置中，可以使敏感数据不受源控件 (Git) 的控制。接下来，配置开发环境以便使用相同的连接信息。

4. 打开 config/local.js，并添加以下连接对象：

        connections: {
            sqlserver: {
                user: "<database server administrator>",
                password: "<database server password>",
                host: "<database server name>.database.chinacloudapi.cn", 
                database: "<database name>",
            },
        },
    
    此配置将覆盖本地环境的 config/connections.js 文件中的设置。项目中默认的 .gitignore 排除了此文件，因此该文件不会存储在 Git 中。现在，可以从 Azure Web 应用和本地开发环境中连接到 Azure SQL 数据库。

4. 打开 config/env/production.js 来配置生产环境，并添加以下 `models` 对象：

        models: {
            connection: 'sqlserver',
            migrate: 'safe'
        },

4. 打开 config/env/development.js 来配置开发环境，并添加以下 `models` 对象：

        models: {
            connection: 'sqlserver',
            migrate: 'alter'
        },

    通过 `migrate: 'alter'` 可以使用数据库迁移功能在 Azure SQL 数据库中轻松创建并更新数据库表。但是，在 Azure（生产）环境中使用 `migrate: 'safe'`，因为 Sails.js 不允许在生产环境中使用 `migrate: 'alter'`（请参阅 [Sails.js 文档](http://sailsjs.org/documentation/concepts/models-and-orm/model-settings)）。

4. 和往常一样在终端[生成](http://sailsjs.org/documentation/reference/command-line-interface/sails-generate) Sails.js 的[蓝图 API](http://sailsjs.org/documentation/concepts/blueprints)，然后运行 `sails lift` 以使用 Sails.js 数据库迁移功能创建数据库。例如：

         sails generate api mywidget
         sails lift

    由此命令生成的 `mywidget` 模型为空，但可以使用它来说明具有数据库连接。运行 `sails lift` 时可为应用使用的模型创建缺失的表。

6. 访问刚刚在浏览器中创建的蓝图 API。例如：

        http://localhost:1337/mywidget/create
    
    此 API 应在浏览器窗口中返回创建的条目，以表示已成功创建数据库。

        {"id":1,"createdAt":"2016-03-28T23:08:01.000Z","updatedAt":"2016-03-28T23:08:01.000Z"}

5. 现在将更改推送到 Azure，并浏览到应用以确保应用仍能正常工作。

        git add .
        git commit -m "<your commit message>"
        git push azure master
        azure site browse

6. 访问 Azure Web 应用的蓝图 API。例如：

        http://<appname>.chinacloudsites.cn/mywidget/create

    如果 API 返回另一个新条目，那么 Azure Web 应用正在和 Azure SQL 数据库通信。

## 更多资源

- [在 Azure App Service 中 Node.js Web 应用入门](/documentation/articles/app-service-web-nodejs-get-started/)
- [将 Node.js 模块与 Azure 应用程序一起使用](/documentation/articles/nodejs-use-node-modules-azure-apps/)

<!---HONumber=Mooncake_0919_2016-->