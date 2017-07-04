<properties
    pageTitle="在 Azure 中构建 Node.js 和 MongoDB Web 应用 | Azure"
    description="了解如何通过使用 MongoDB 连接字符串与 DocumentDB 数据库建立连接，在 Azure 中运行 Node.js 应用。"
    services="app-service\web"
    documentationcenter="nodejs"
    author="cephalin"
    manager="erikre"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="0b4d7d0e-e984-49a1-a57a-3c0caa955f0e"
    ms.service="app-service-web"
    ms.workload="web"
    ms.tgt_pltfrm="na"
    ms.devlang="nodejs"
    ms.topic="article"
    ms.date="03/30/2017"
    wacn.date="05/02/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="379165b13b1fb6c276a10c317e4bf3bfb31aeba3"
    ms.lasthandoff="04/22/2017" />

# <a name="build-a-nodejs-and-mongodb-web-app-in-azure"></a>在 Azure 中构建 Node.js 和 MongoDB Web 应用
本教程介绍如何在 Azure 中创建 Node.js Web 应用并将其连接到 MongoDB 数据库。 完成本教程后，你将获得一个在 [Azure 应用服务 Web 应用](/documentation/articles/app-service-web-overview/)中运行的 MEAN（MongoDB、Express、AngularJS 和 Node.js）应用程序。

![在 Azure 应用服务中运行的 MEAN.js 应用](./media/app-service-web-tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

## <a name="before-you-begin"></a>开始之前

开始学习本教程之前，请确保在计算机上[安装 Azure CLI](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)。 此外，还需要 [Node.js](https://nodejs.org/) 和 [Git](http://www.git-scm.com/downloads)。 将运行 `az`、`npm` 和 `git` 命令。

你应该具备 Node.js 的实践知识。 本教程并未介绍有关开发 Node.js 应用程序的一般信息。

## <a name="step-1---create-local-nodejs-application"></a>步骤 1 - 创建本地 Node.js 应用程序
在此步骤中，你将设置本地 Node.js 项目。

### <a name="clone-the-sample-application"></a>克隆示例应用程序

打开终端窗口，运行 `CD` 切换到工作目录。  

运行以下命令克隆示例存储库。 此示例存储库包含默认的 [MEAN.js](http://meanjs.org/) 应用程序。 

    git clone https://github.com/prashanthmadi/mean

### <a name="run-the-application"></a>运行应用程序

安装所需的包并启动应用程序。

    cd mean
    npm install
    npm start

除非你已有一个本地 MongoDB 数据库，否则 `npm start` 应会终止并返回类似于下面的错误消息：

    { [MongoError: failed to connect to server [localhost:27017] on first connect]
      ...
      name: 'MongoError',
      message: 'failed to connect to server [localhost:27017] on first connect' }

此时请不要设置本地 MongoDB 数据库，因为稍后将在 Azure 中创建。

## <a name="step-2---create-a-mongodb-database"></a>步骤 2 - 创建 MongoDB 数据库

在此步骤中，你要将应用程序连接到 MongoDB 数据库。 对于 MongoDB，本教程使用支持 MongoDB 客户端连接的 [Azure DocumentDB](/documentation/services/documentdb/)。 换而言之，Node.js 应用程序只知道它要连接到 MongoDB 数据库。 连接由 DocumentDB 数据库支持的事实对于应用程序是完全透明的。

### <a name="log-in-to-azure"></a>登录 Azure

现在，你要在终端窗口中使用 Azure CLI 2.0 创建所需的资源用于在 Azure 应用服务中托管 Node.js 应用程序。  使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。 

    az login 

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

### <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 创建[资源组](/documentation/articles/resource-group-overview/)。 Azure 资源组是在其中部署和管理 Azure 资源（例如 Web 应用、数据库和存储帐户）的逻辑容器。 

以下示例在中国北部区域中创建一个资源组。

    az group create --name myResourceGroup --location "China North"

若要查看适用于 `--location` 的可能值，请使用 `az appservice list-locations` Azure CLI 命令。

### <a name="create-a-documentdb-account"></a>创建 DocumentDB 帐户

使用 [az documentdb create](https://docs.microsoft.com/zh-cn/cli/azure/documentdb#create) 命令创建 DocumentDB 帐户。

在以下命令中，请将出现的 `<documentdb_name>` 占位符替换为你自己的唯一 DocumentDB 名称。 此唯一名称将用作 DocumentDB 终结点 (`https://<documentdb_name>.documents.azure.cn/`) 的一部分，因此需要在 Azure 中的所有 DocumentDB 帐户之间保持唯一。 

    az documentdb create --name <documentdb_name> --resource-group myResourceGroup --kind MongoDB

`--kind MongoDB` 参数启用 MongoDB 客户端连接。

创建 DocumentDB 帐户后，Azure CLI 将显示类似于以下示例的信息。 

    {
      "databaseAccountOfferType": "Standard",
      "documentEndpoint": "https://<documentdb_name>.documents.azure.cn:443/",
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Document
    DB/databaseAccounts/<documentdb_name>",
      "kind": "MongoDB",
      "location": "China North",
      "name": "<documentdb_name>",
      "readLocations": [
        {
          "documentEndpoint": "https://<documentdb_name>-chinanorth.documents.azure.cn:443/",
          "failoverPriority": 0,
          "id": "<documentdb_name>-chinanorth",
          "locationName": "China North",
          "provisioningState": "Succeeded"
        }
      ],
      "resourceGroup": "myResourceGroup",
      "type": "Microsoft.DocumentDB/databaseAccounts",
      "writeLocations": [
        {
          "documentEndpoint": "https://<documentdb_name>-chinanorth.documents.azure.cn:443/",
          "failoverPriority": 0,
          "id": "<documentdb_name>-chinanorth",
          "locationName": "China North",
          "provisioningState": "Succeeded"
        }
      ]
    } 

## <a name="step-3---connect-your-nodejs-application-to-the-database"></a>步骤 3 - 将 Node.js 应用程序连接到数据库

在此步骤中，你要使用 MongoDB 连接字符串将 MEAN.js 示例应用程序连接到刚刚创建的 DocumentDB 数据库。 

### <a name="retrieve-the-database-key"></a>检索数据库密钥

若要连接到 DocumentDB 数据库，需要数据库密钥。 使用 [az documentdb list-keys](https://docs.microsoft.com/zh-cn/cli/azure/documentdb#list-keys) 命令检索主密钥。

    az documentdb list-keys --name <documentdb_name> --resource-group myResourceGroup

Azure CLI 输出类似于以下示例的信息。 

    {
      "primaryMasterKey": "RUayjYjixJDWG5xTqIiXjC...",
      "primaryReadonlyMasterKey": "...",
      "secondaryMasterKey": "...",
      "secondaryReadonlyMasterKey": "..."
    }

将 `primaryMasterKey` 的值复制到文本编辑器。 下一步骤需要用到此信息。

### <a name="devconfig"></a> 在 Node.js 应用程序中配置连接字符串

在 MEAN.js 存储库中打开 `config/env/local-development.js`。

将此文件的内容替换为以下代码。 另外，请务必将两个 `<documentdb_name>` 占位符替换为你的 DocumentDB 数据库名称，将 `<primary_maste_key>` 占位符替换为在上一步骤中复制的密钥。

    'use strict';

    module.exports = {
      db: {
        uri: 'mongodb://<documentdb_name>:<primary_maste_key>@<documentdb_name>.documents.azure.cn:10250/mean-dev?ssl=true&sslverifycertificate=false'
      }
    };

> [AZURE.NOTE] 
> `ssl=true` 选项非常重要，因为 [Azure DocumentDB 需要 SSL](/documentation/articles/documentdb-connect-mongodb-account/#connection-string-requirements)。 
>
>

保存所做更改。

### <a name="run-the-application-again"></a>再次运行应用程序。

再次运行 `npm start`。 

    npm start

此时不会出现前面所示的错误消息，而应会出现一条控制台消息，指出开发环境已启动并正在运行。 

在浏览器中导航到 `http://localhost:3000`。 在顶部菜单中单击“注册”，然后尝试创建一个虚构的用户。 

MEAN.js 示例应用程序将用户数据存储在数据库中。 如果上述操作成功并且 MEAN.js 可自动登录到创建的用户，则表示 MongoDB 数据库连接可正常工作。 

![MEAN.js 成功连接到 MongoDB](./media/app-service-web-tutorial-nodejs-mongodb-app/mongodb-connect-success.png)

## <a name="step-4---deploy-the-nodejs-application-to-azure"></a>步骤 4 - 将 Node.js 应用程序部署到 Azure
在此步骤中，你要将 MongoDB 连接的 Node.js 应用程序部署到 Azure 应用服务。

### <a name="prepare-your-sample-application-for-deployment"></a>准备要部署的示例应用程序

你可能已注意到，前面更改的配置文件适用于开发环境 (`/config/env/local-development.js`)。 将应用程序部署到应用服务时，应用程序默认在生产环境中运行。 因此，现在需要对相应的配置文件进行相同的更改。

在 MEAN.js 存储库中打开 `config/env/production.js`。

在 `db` 对象中，如以下示例所示替换 `uri` 的值。 请务必按上文所述替换占位符。

    db: {
      uri: 'mongodb://<documentdb_name>:<primary_maste_key>@<documentdb_name>.documents.azure.cn:10250/mean?ssl=true&sslverifycertificate=false',
      ...
    },

在终端中，将所有更改提交到 Git。 可以复制这两个命令，以便同时运行。

    git add .
    git commit -m "configured MongoDB connection string"

Node.js 应用程序已准备就绪，可供部署。

### <a name="create-an-app-service-plan"></a>创建应用服务计划

使用 [az appservice plan create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#create) 命令创建应用服务计划。 

> [AZURE.NOTE] 
> 应用服务计划表示用于托管应用的物理资源集合。 分配到应用服务计划的所有应用程序将共享该计划定义的资源，在托管多个应用时可以节省成本。 
> <br/> 
> 应用服务计划定义： 
><p> * 区域（中国北部、中国东部） 
><p> * 实例大小（小、中、大） 
><p> * 规模计数（一个、两个、三个实例，等等） 
><p> * SKU（免费、共享、基本、标准、高级）

以下示例使用**免费**定价层创建名为 `myAppServicePlan` 的应用服务计划。

    az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku FREE

创建应用服务计划后，Azure CLI 将显示类似于以下示例的信息。 

    { 
        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan", 
        "kind": "app", 
        "location": "China North", 
        "sku": { 
        "capacity": 0, 
        "family": "F", 
        "name": "F1", 
        "tier": "Free" 
        }, 
        "status": "Ready", 
        "type": "Microsoft.Web/serverfarms" 
    } 

### <a name="create-a-web-app"></a>创建 Web 应用

创建应用服务计划后，请在 `myAppServicePlan` 应用服务计划中创建 Web 应用。 该 Web 应用提供托管空间用于部署代码，并提供一个 URL 用于查看已部署的应用程序。 使用 [az appservice web create](https://docs.microsoft.com/zh-cn/cli/azure/webapp#create) 命令创建该 Web 应用。 

在以下命令中，请将 `<app_name>` 占位符替换为你自己的唯一应用名称。 此唯一名称将用作 Web 应用的默认域名的一部分，因此，该名称需要在 Azure 中的所有应用之间保持唯一。 稍后，可以先将任何自定义 DNS 条目映射到 Web 应用，然后向用户公开该条目。 

    az appservice web create --name <app_name> --resource-group myResourceGroup --plan myAppServicePlan

创建 Web 应用后，Azure CLI 将显示类似于以下示例的信息。 

    { 
        "clientAffinityEnabled": true, 
        "defaultHostName": "<app_name>.chinacloudsites.cn", 
        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Web/sites/<app_name>", 
        "isDefaultContainer": null, 
        "kind": "app", 
        "location": "China North", 
        "name": "<app_name>", 
        "repositorySiteName": "<app_name>", 
        "reserved": true, 
        "resourceGroup": "myResourceGroup", 
        "serverFarmId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan", 
        "state": "Running", 
        "type": "Microsoft.Web/sites", 
    } 

### <a name="configure-local-git-deployment"></a>配置本地 Git 部署 

可以通过不同的方法将应用程序部署到 Azure 应用服务，包括 FTP、本地 Git，以及 GitHub、Visual Studio Team Services 和 Bitbucket。 对于 FTP 和本地 Git 部署，需在服务器上配置一个部署用户，用于对部署进行身份验证。 

使用 [az appservice web deployment user set](https://docs.microsoft.com/zh-cn/cli/azure/webapp/deployment/user#set) 命令创建帐户级别的凭据。 

> [AZURE.NOTE] 
> 在应用服务中进行 FTP 和本地 Git 部署时需要一个部署用户。 此部署用户是帐户级用户。 因此，它不同于 Azure 订阅帐户。 只需创建此部署用户一次。

    az appservice web deployment user set --user-name <specify-a-username> --password <mininum-8-char-captital-lowercase-number>

使用 [az appservice web source-control config-local-git](https://docs.microsoft.com/zh-cn/cli/azure/webapp/source-control#config-local-git) 命令配置对 Azure Web 应用的本地 Git 访问。 

    az appservice web source-control config-local-git --name <app_name> --resource-group myResourceGroup

配置部署用户后，Azure CLI 将使用以下格式显示 Azure Web 应用的部署 URL：

    https://<username>@<app_name>.scm.chinacloudsites.cn:443/<app_name>.git 

复制终端的输出，因为下一步骤将要用到。 

### <a name="push-to-azure-from-git"></a>从 Git 推送到 Azure

将 Azure 远程功能添加到本地 Git 存储库。 

    git remote add azure <paste_copied_url_here> 

推送到 Azure remote 以部署 Node.js 应用程序。 系统将提示你输入前面在创建部署用户期间提供的密码。 

    git push azure master

在部署期间，Azure 应用服务会向 Git 告知其进度。

    Counting objects: 5, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (5/5), done.
    Writing objects: 100% (5/5), 489 bytes | 0 bytes/s, done.
    Total 5 (delta 3), reused 0 (delta 0)
    remote: Updating branch 'master'.
    remote: Updating submodules.
    remote: Preparing deployment for commit id '6c7c716eee'.
    remote: Running custom deployment command...
    remote: Running deployment command...
    remote: Handling node.js deployment.
    .
    .
    .
    remote: Deployment successful.
    To https://<app_name>.scm.chinacloudsites.cn/<app_name>.git
     * [new branch]      master -> master

> [AZURE.NOTE]
> 你可能已注意到，部署过程先运行 `npm install`，再运行 [Gulp](http://gulpjs.com/)。 与其他某些 Node.js 应用程序一样，MEAN.js 使用 Gulp 来自动执行部署任务。 具体而言，它使用 Gulp 来缩减和捆绑用于生产的脚本。 在部署过程中，应用服务默认不会运行 Gulp 或 Grunt 任务，因此，此示例存储库的根目录中提供了两个附加文件来启用此功能： 
><p> - `.deployment` - 此文件告知应用服务要以自定义部署脚本的形式运行 `bash deploy.sh`。
><p> - `deploy.sh` - 自定义部署脚本。 查看该文件可以发现，它先运行 `npm install` 和 `bower install`，再运行 `gulp prod`。 
> <p>
> 可以使用此方法将任何步骤添加到基于 Git 的部署。
>

### <a name="browse-to-the-azure-web-app"></a>浏览到 Azure Web 应用 
使用 Web 浏览器浏览到已部署的 Web 应用。 

    http://<app_name>.chinacloudsites.cn 

在顶部菜单中单击“注册”，然后尝试创建一个虚构的用户。 

如果上述操作成功并且该应用可自动登录到创建的用户，则表示 Azure 中的 MEAN.js 应用已与 MongoDB (DocumentDB) 数据库建立连接。 

![在 Azure 应用服务中运行的 MEAN.js 应用](./media/app-service-web-tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

## <a name="step-5---store-sensitive-data-as-environment-variables"></a>步骤 5 - 以环境变量的形式存储敏感数据

在本教程前面的部分，你已将 MEAN.js 配置文件中的连接字符串硬编码。 实际上，这并不是最佳的安全做法。 当你向 Git 提交更改时，数据库密钥会立即透露给对 Git 存储库拥有读取访问权限的任何人。 在此步骤中，你将了解如何改为存储并访问连接字符串。

### <a name="configure-an-environment-variable-in-azure"></a>在 Azure 中配置环境变量

在应用服务中，使用 [az appservice web config appsettings update](https://docs.microsoft.com/zh-cn/cli/azure/webapp/config/appsettings#update) 命令将环境变量设置为_应用设置_。 

使用以下示例可在 Azure Web 应用中配置 `MONGODB_URI` 应用设置。 请务必按上文所述替换占位符。

    az appservice web config appsettings update --name <app_name> --resource-group myResourceGroup --settings MONGODB_URI="mongodb://<documentdb_name>:<primary_maste_key>@<documentdb_name>.documents.azure.cn:10250/mean?ssl=true&sslverifycertificate=false"

### <a name="access-the-environment-variable-in-nodejs-code"></a>访问 Node.js 代码中的环境变量

在 Node.js 代码中，使用 `process.env.MONGODB_URI` 访问此应用设置，就像访问任何环境变量一样。 

打开 `config/env/production.js` 并将 `db.uri` 设置为此应用设置。 例如：

    db: {
      uri: process.env.MONGODB_URI,
      ...
    },

### <a name="push-your-changes-to-azure"></a>将更改推送到 Azure

接下来，使用 `git push` 命令推送更改。

    git push azure master

现在，你的 Azure Web 应用将会访问此环境变量（代表该应用的 MongoDB 连接字符串）。