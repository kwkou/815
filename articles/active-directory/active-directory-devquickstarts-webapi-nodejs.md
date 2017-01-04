<properties
	pageTitle="Azure AD NodeJS 入门 | Azure"
	description="如何生成一个与 Azure AD 集成以进行身份验证的 Node.js REST Web API。"
	services="active-directory"
	documentationCenter="nodejs"
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>  


<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="javascript"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="brandwe"
	wacn.date="01/03/2017"/>  


# 节点 WEB API 入门

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

**Passport** 是 Node.js 的身份验证中间件。Passport 极其灵活并且采用模块化结构，可以在不造成干扰的情况下放入任何基于 Express 的应用程序或 Resitify Web 应用程序。一套综合性策略支持使用用户名和密码、Facebook、Twitter 等进行身份验证。我们针对 Azure Active directory 开发了一个策略。我们将安装此模块，然后添加 Azure Active Directory `passport-azure-ad` 插件。

为此，你需要：

1. 将一个应用程序注册到 Azure AD
2. 将应用设置为使用 Passport 的 azure-ad-passport 插件。
3. 配置一个客户端应用程序用于调用待办事项列表 Web API

本教程的代码[在 GitHub 上](https://github.com/Azure-Samples/active-directory-node-webapi)维护。

> [AZURE.NOTE] 本文未涵盖如何使用 Azure AD B2C 来实施登录、注册和配置文件管理，而是着重介绍如何在用户已通过身份验证后调用 Web API。如果尚未开始，应该先从[如何与 Azure Active Directory 集成文档](/documentation/articles/active-directory-how-to-integrate/)入手，了解 Azure Active Directory 的基础知识。


我们已在 GitHub 中的 MIT 许可证下发布了此运行示例的所有源代码，你可以任意克隆（甚至分发！）这些代码，并提供反馈和发出请求。

## 关于 Node.js 模块

在本演练中，我们将使用 Node.js 模块。模块是可加载的 JavaScript 包，可为你的应用程序提供特定功能。你通常会使用 Node.js NPM 命令行工具在 NPM 安装目录中安装模块，但一些模块（如 HTTP 模块）是作为核心 Node.js 包的一部分提供的。安装的模块保存在 Node.js 安装目录的根目录下的 node\_modules 目录中。node\_modules 目录中的每个模块都保留自己的 node\_modules 目录，其中包含它依赖的所有模块，每个必需的模块都有一个 node\_modules 目录。这种递归的目录结构表示了依赖关系链。

这种依赖关系链结构会导致应用程序占用空间变大，但保证满足所有依赖项的要求，并且开发中使用的模块版本也将在生产中使用。这使得生产应用程序的行为更有预测性，并防止出现影响用户的版本控制问题。

## 1\.注册 Azure AD 租户

若要使用本示例，你需要一个 Azure Active Directory 租户。如果你不确定什么是租户或者如何获取租户，请参阅[如何获取 Azure AD 租户](/documentation/articles/active-directory-howto-tenant/)。

## 2\.创建应用程序

你现在需要在目录中创建应用，以便为 Azure AD 提供一些必要信息，让它与应用安全地通信。在此案例中，因为客户端应用和 Web API 会组成一个逻辑应用，所以将由单一**应用程序 ID** 表示。若要创建应用，请遵循[这些说明](/documentation/articles/active-directory-how-applications-are-added/)。如果要生成业务线应用，[这些附加说明可能很有用](/documentation/articles/active-directory-applications-guiding-developers-for-lob-applications/)。

请务必：

- 登录到 Azure 管理门户。
- 在左侧的导航栏中单击“Active Directory”。
- 选择你要在其中注册应用程序的租户。
- 单击“应用程序”选项卡，然后在底部抽屉中单击“添加”。
- 根据提示创建一个新的 **Web 应用程序和/或 WebAPI**。
    - 应用程序的**名称**向最终用户描述你的应用程序
    - “登录 URL”是应用程序的基本 URL。示例代码的默认值是 `https://localhost:8080`。
    - “应用程序 ID URI”是应用程序的唯一标识符。约定是使用 `https://<tenant-domain>/<app-name>`，例如 `https://contoso.partner.onmschina.cn/my-first-aad-app`
- 完成注册后，AAD 将为应用程序分配唯一的客户端标识符。在后面的部分中将会用到此值，因此，请从“配置”选项卡复制此值。

- 提醒：为你的应用程序创建一个**应用程序密码**并复制下来。稍后您将需要它。
- 提醒：复制分配给应用的**应用程序 ID**。稍后也会用到。


## 3\.下载适用于平台的 node.js
若要成功使用本示例，你必须正确安装 Node.js。

请从 [http://nodejs.org](http://nodejs.org) 安装 Node.js。

## 4\.在平台上安装 MongoDB

若要成功使用本示例，你必须正确安装 MongoDB。我们将使用 MongoDB 来使 REST API 持久保留在服务器实例之间。

从 [http://mongodb.org](http://www.mongodb.org) 安装 MongoDB。

> [AZURE.NOTE] 本演练假定为 MongoDB 使用默认的安装与服务器终结点，在编写本文时，该终结点为：mongodb://localhost


## 5\.将 Restify 模块安装到 Web API 中

我们将使用 Resitfy 来生成 REST API。Restify 是从 Express 派生的精简灵活 Node.js 应用程序框架，它提供一套可靠的功能用于在 Connect 顶层生成 REST API。

### 安装 Restify

在命令行中，将目录切换到 azuread 目录。如果 **azuread** 目录不存在，请创建该目录。

`cd azuread - or- mkdir azuread; cd azuread`

输入以下命令：

`npm install restify`

此命令将安装 Restify。

#### 遇到了错误吗？

在某些操作系统上使用 npm 时，你可能会收到一条错误“错误: EPERM，chmod '/usr/local/bin/express' 和一个尝试以管理员身份运行帐户的请求。如果发生这种情况，请使用 sudo 命令以更高的权限级别运行 npm。

#### 遇到了有关 DTRACE 的错误吗？

在安装 Restify 时，你可能会看到类似于下面的内容：

Shell
		
	clang: error: no such file or directory: 'HD/azuread/node_modules/restify/node_modules/dtrace-provider/libusdt'
	make: *** [Release/DTraceProviderBindings.node] Error 1
	gyp ERR! build error
	gyp ERR! stack Error: `make` failed with exit code: 2
	gyp ERR! stack     at ChildProcess.onExit (/usr/local/lib/node_modules/npm/node_modules/node-gyp/lib/build.js:267:23)
	gyp ERR! stack     at ChildProcess.EventEmitter.emit (events.js:98:17)
	gyp ERR! stack     at Process.ChildProcess._handle.onexit (child_process.js:789:12)
	gyp ERR! System Darwin 13.1.0
	gyp ERR! command "node" "/usr/local/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js" "rebuild"
	gyp ERR! cwd /Volumes/Development HD/azuread/node_modules/restify/node_modules/dtrace-provider
	gyp ERR! node -v v0.10.11
	gyp ERR! node-gyp -v v0.10.0
	gyp ERR! not ok
	npm WARN optional dep failed, continuing dtrace-provider@0.2.8



Restify 提供强大的机制来使用 DTrace 跟踪 REST 调用。但是，许多操作系统不提供 DTrace。你可以安全地忽略这些错误。


此命令的输出看上去应如下所示：


	restify@2.6.1 node_modules/restify
	├── assert-plus@0.1.4
	├── once@1.3.0
	├── deep-equal@0.0.0
	├── escape-regexp-component@1.0.2
	├── qs@0.6.5
	├── tunnel-agent@0.3.0
	├── keep-alive-agent@0.0.1
	├── lru-cache@2.3.1
	├── node-uuid@1.4.0
	├── negotiator@0.3.0
	├── mime@1.2.11
	├── semver@2.2.1
	├── spdy@1.14.12
	├── backoff@2.3.0
	├── formidable@1.0.14
	├── verror@1.3.6 (extsprintf@1.0.2)
	├── csv@0.3.6
	├── http-signature@0.10.0 (assert-plus@0.1.2, asn1@0.1.11, ctype@0.5.2)
	└── bunyan@0.22.0 (mv@0.0.5)


## 6\.将 Passport.js 安装到 Web API 中

[Passport](http://passportjs.org) 是 Node.js 的身份验证中间件。Passport 极其灵活并且采用模块化结构，可以在不造成干扰的情况下放入任何基于 Express 的应用程序或 Resitify Web 应用程序。一套综合性策略支持使用用户名和密码、Facebook、Twitter 等进行身份验证。我们针对 Azure Active directory 开发了一个策略。我们将安装此模块，然后添加 Azure Active Directory 策略插件。

在命令行中，将目录切换到 azuread 目录。

输入以下命令以安装 passport.js

`npm install passport`  


该命令的输出应如下所示：

	passport@0.1.17 node_modules\passport
	├── pause@0.0.1
	└── pkginfo@0.2.3

## 7\.将 Passport-Azure-AD 添加到 Web API

接下来，我们将使用 passport-azuread 来添加 OAuth 策略，这是一套将 Azure Active Directory 连接到 Passport 的策略。在此 Rest API 示例中，我们将针对持有者令牌使用此策略。

> [AZURE.NOTE] 尽管 OAuth2 提供了可以颁发任何已知令牌类型的框架，但只有一部分令牌类型已得到广泛使用。用于保护终结点的令牌是持有者令牌。持有者令牌是 OAuth2 中最广泛颁发的令牌，许多实现假定持有者令牌是唯一颁发的令牌类型。

在命令行中，将目录切换到 azuread 目录

键入以下命令以安装 Passport.js passport-azure-ad 模块：

`npm install passport-azure-ad`  


该命令的输出应如下所示：

	passport-azure-ad@1.0.0 node\_modules/passport-azure-ad
	├── xtend@4.0.0 
	├── xmldom@0.1.19 
	├── passport-http-bearer@1.0.1 (passport-strategy@1.0.0) 
	├── underscore@1.8.3 
	├── async@1.3.0 
	├── jsonwebtoken@5.0.2 
	├── xml-crypto@0.5.27 (xpath.js@1.0.6) 
	├── ursa@0.8.5 (bindings@1.2.1, nan@1.8.4) 
	├── jws@3.0.0 (jwa@1.0.1, base64url@1.0.4) 
	├── request@2.58.0 (caseless@0.10.0, aws-sign2@0.5.0, forever-agent@0.6.1, stringstream@0.0.4, tunnel-agent@0.4.1, oauth-sign@0.8.0, isstream@0.1.2, extend@2.0.1, json-stringify-safe@5.0.1, node-uuid@1.4.3, qs@3.1.0, combined-stream@1.0.5, mime-types@2.0.14, form-data@1.0.0-rc1, http-signature@0.11.0, bl@0.9.4, tough-cookie@2.0.0, hawk@2.3.1, har-validator@1.8.0)
	└── xml2js@0.4.9 (sax@0.6.1, xmlbuilder@2.6.4)

## 8\.将 MongoDB 模块添加到 Web API

我们将使用 MongoDB 作为数据存储。为此，我们需要安装这两个广泛使用的插件来管理模型和称为 Mongoose 的架构，以及 MongoDB 的数据库驱动程序（也称为 MongoDB）。


- `npm install mongoose`  


## 9\.安装其他模块

接下来，我们将安装剩余的所需模块。


在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

`cd azuread`  



输入以下命令，以在 node\_modules 目录中安装以下模块：

- `npm install assert-plus`  

- `npm install bunyan`
- `npm update`  



## 10\.创建包含依赖项的 server.js

server.js 文件将提供 Web API 服务器的大多数功能。我们要将大部分代码添加到此文件。用于生产目的，需要将功能重构为较小的文件，例如单独的路由和控制器。在本演示中，我们将为此功能使用 server.js。

在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

`cd azuread`  


在偏好的编辑器中创建 `server.js` 文件，然后添加以下信息：

Javascript

	'use strict';

	/**
 	* Module dependencies.
 	*/

	var fs = require('fs');
	var path = require('path');
	var util = require('util');
	var assert = require('assert-plus');
	var bunyan = require('bunyan');
	var getopt = require('posix-getopt');
	var mongoose = require('mongoose/');
	var restify = require('restify');
	var passport = require('passport');
	var BearerStrategy = require('passport-azure-ad').BearerStrategy;


保存文件。稍后我们将会使用该文件。

## 11\.创建一个配置文件用于存储 Azure AD 设置

此代码文件会将配置参数从 Azure Active Directory 门户传递到 Passport.js。当你在本演练的第一部分中向门户添加 Web API 时，已经创建了这些配置值。我们将解释在复制代码后，要输入其中的哪些参数值。


在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

`cd azuread`  


在偏好的编辑器中创建 `config.js` 文件，然后添加以下信息：

Javascript

	 exports.creds = {
	     mongoose_auth_local: 'mongodb://localhost/tasklist', // Your mongo auth uri goes here
	     clientID: 'your client ID',
	     audience: 'your application URL',
	    // you cannot have users from multiple tenants sign in to your server unless you use the common endpoint
	  // example: https://login.microsoftonline.com/common/.well-known/openid-configuration
	     identityMetadata: 'https://login.microsoftonline.com/<your tenant id>/.well-known/openid-configuration', 
	     validateIssuer: true, // if you have validation on, you cannot have users from multiple tenants sign in to your server
	     passReqToCallback: false,
	     loggingLevel: 'info' // valid are 'info', 'warn', 'error'. Error always goes to stderr in Unix.

	 };



保存文件。

## 12\.将配置添加到 server.js 文件

我们需要在应用程序中，从你刚刚创建的配置文件读取这些值。为此，我们只需在应用程序中添加 .config 文件作为所需的资源，然后将全局变量设置为 config.js 文档中的值

在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

`cd azuread`  


在偏好的编辑器中打开 `server.js` 文件，并添加以下信息：

Javascript

	var config = require('./config');

然后，在 `server.js` 中替换包含以下代码的新节：

Javascript

	var options = {
	    // The URL of the metadata document for your app. We will put the keys for token validation from the URL found in the jwks_uri tag of the in the metadata.
	    identityMetadata: config.creds.identityMetadata,
	    clientID: config.creds.clientID,
	    validateIssuer: config.creds.validateIssuer,
	    audience: config.creds.audience,
	    passReqToCallback: config.creds.passReqToCallback,
	    loggingLevel: config.creds.loggingLevel

	};

	// array to hold logged in users and the current logged in user (owner)
	var users = [];
	var owner = null;

	// Our logger
	var log = bunyan.createLogger({
	    name: 'Azure Active Directory Bearer Sample',
	         streams: [
	        {
	            stream: process.stderr,
	            level: "error",
	            name: "error"
	        }, 
	        {
	            stream: process.stdout,
	            level: "warn",
	            name: "console"
	        }, ]
	});

	  // if logging level specified, switch to it.
	  if (config.creds.loggingLevel) { log.levels("console", config.creds.loggingLevel); }

	// MongoDB setup
	// Setup some configuration
	var serverPort = process.env.PORT || 8080;
	var serverURI = (process.env.PORT) ? config.creds.mongoose_auth_mongohq : config.creds.mongoose_auth_local;


保存文件。



## 13\.使用 Moongoose 添加 MongoDB 模型和架构信息

现在，我们已将这三个文件统一放在 REST API 服务中，接下来让我们的准备工作发挥作用。

对于本演练，我们将使用 MongoDB 来存储***步骤 4*** 中所述的任务。

回顾我们在 ***步骤 11*** 中创建的 `config.js` 文件，我们将数据库称为 `tasklist`，因为这是我们在 mogoose\_auth\_local 连接 URL 的末尾放置的内容。你无需事先在 MongoDB 中创建此数据库，当你首次运行服务器应用程序时，系统将创建此数据库（假定它不存在）。

现在，我们已告诉服务器要使用哪个 MongoDB 数据库，接下来我们需要编写一些附加的代码，以便为服务器任务创建模型和架构。

#### 模型介绍

我们的架构模型非常简单，你可以根据需要对其进行扩展。

NAME - 分配到任务的用户名。一个***字符串***。

TASK - 任务本身。一个***字符串***。

DATE - 任务截止日期。一个***日期时间***

COMPLETED - 任务是否已完成。一个***布尔值***

#### 在代码中创建架构


在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

`cd azuread`  


在偏好的编辑器中打开 `server.js` 文件，并在配置条目下面添加以下信息：

Javascript
		
	// Connect to MongoDB
	global.db = mongoose.connect(serverURI);
	var Schema = mongoose.Schema;
	log.info('MongoDB Schema loaded');

	// Here we create a schema to store our tasks and users. Pretty simple schema for now.
	var TaskSchema = new Schema({
	    owner: String,
	    task: String,
	    completed: Boolean,
	    date: Date
	});

	// Use the schema to register a model
	mongoose.model('Task', TaskSchema);
	var Task = mongoose.model('Task');

从该代码中可以看到，我们将会创建架构，然后创建在定义***路由***时，将在整个代码中用于存储数据的模型对象。

## 14\.为任务 REST API 服务器添加路由

现在，我们已经创建了一个可用的数据库模型，接下来让我们添加用于 REST API 服务器的路由。

### 关于 Restify 中的路由

Restify 中路由的工作原理，与使用 Express 堆栈时的路由工作原理完全相同。你可以使用客户端应用程序应该调用的 URI 定义路由。通常，需要在单独的文件中定义路由。对于本演练，我们将在 server.js 文件中定义路由。对于生产用途，我们建议你在各自的文件中定义路由。

Restify 路由的典型模式是：

Javascript

	function createObject(req, res, next) {

	// do work on Object

	 _object.name = req.params.object; // passed value is in req.params under object

	 ///...

	return next(); // keep the server going
	}

	....

	server.post('/service/:add/:object', createObject); // calls createObject on routes that match this.




这是最基本级别的模式。Resitfy（和 Express）提供了更深层的功能，例如，定义应用程序类型，以及跨不同的终结点执行复杂路由。对于本演练，我们会保持这些路由的简炼性。

### 1\.将默认路由添加到服务器

现在，我们将添加 Create、Retrieve、Update 和 Delete 的基本 CRUD 路由。

在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

`cd azuread`  


在偏好的编辑器中打开 `server.js` 文件，并在前面创建的数据库条目下面添加以下信息：

Javascript

	/**
	 *
	 * APIs for our REST Task server
	 */

	// Create a task

	function createTask(req, res, next) {

	    // Resitify currently has a bug which doesn't allow you to set default headers
	    // This headers comply with CORS and allow us to mongodbServer our response to any origin

	    res.header("Access-Control-Allow-Origin", "*");
	    res.header("Access-Control-Allow-Headers", "X-Requested-With");

	    // Create a new task model, fill it up and save it to Mongodb
	    var _task = new Task();

	    if (!req.params.task) {
	        req.log.warn('createTodo: missing task');
	        next(new MissingTaskError());
	        return;
	    }

	    _task.owner = owner;
	    _task.task = req.params.task;
	    _task.date = new Date();

	    _task.save(function(err) {
	        if (err) {
	            req.log.warn(err, 'createTask: unable to save');
	            next(err);
	        } else {
	            res.send(201, _task);

	        }
	    });

	    return next();

	}


	// Delete a task by name

	function removeTask(req, res, next) {

	    Task.remove({
	        task: req.params.task,
	        owner: owner
	    }, function(err) {
	        if (err) {
	            req.log.warn(err,
	                'removeTask: unable to delete %s',
	                req.params.task);
	            next(err);
	        } else {
	            log.info('Deleted task:', req.params.task);
	            res.send(204);
	            next();
	        }
	    });
	}

	// Delete all tasks

	function removeAll(req, res, next) {
	    Task.remove();
	    res.send(204);
	    return next();
	}


	// Get a specific task based on name

	function getTask(req, res, next) {

	    log.info('getTask was called for: ', owner);
	    Task.find({
	        owner: owner
	    }, function(err, data) {
	        if (err) {
	            req.log.warn(err, 'get: unable to read %s', owner);
	            next(err);
	            return;
	        }

	        res.json(data);
	    });

	    return next();
	}

	/// Simple returns the list of TODOs that were loaded.

	function listTasks(req, res, next) {
	    // Resitify currently has a bug which doesn't allow you to set default headers
	    // This headers comply with CORS and allow us to mongodbServer our response to any origin

	    res.header("Access-Control-Allow-Origin", "*");
	    res.header("Access-Control-Allow-Headers", "X-Requested-With");

	    log.info("listTasks was called for: ", owner);

	    Task.find({
	        owner: owner
	    }).limit(20).sort('date').exec(function(err, data) {

	        if (err) {
	            return next(err);
	        }

	        if (data.length > 0) {
	            log.info(data);
	        }

	        if (!data.length) {
	            log.warn(err, "There is no tasks in the database. Did you initalize the database as stated in the README?");
	        }

	        if (!owner) {
	            log.warn(err, "You did not pass an owner when listing tasks.");
	        } else {

	            res.json(data);

	        }
	    });

	    return next();
	}



### 2\.接下来，让我们在 API 中添加一些错误处理方式：

		

	///--- Errors for communicating something interesting back to the client

	function MissingTaskError() {
	    restify.RestError.call(this, {
	        statusCode: 409,
	        restCode: 'MissingTask',
	        message: '"task" is a required parameter',
	        constructorOpt: MissingTaskError
	    });

	    this.name = 'MissingTaskError';
	}
	util.inherits(MissingTaskError, restify.RestError);


	function TaskExistsError(owner) {
	    assert.string(owner, 'owner');

	    restify.RestError.call(this, {
	        statusCode: 409,
	        restCode: 'TaskExists',
	        message: owner + ' already exists',
	        constructorOpt: TaskExistsError
	    });

	    this.name = 'TaskExistsError';
	}
	util.inherits(TaskExistsError, restify.RestError);


	function TaskNotFoundError(owner) {
	    assert.string(owner, 'owner');

	    restify.RestError.call(this, {
	        statusCode: 404,
	        restCode: 'TaskNotFound',
	        message: owner + ' was not found',
	        constructorOpt: TaskNotFoundError
	    });

	    this.name = 'TaskNotFoundError';
	}

	util.inherits(TaskNotFoundError, restify.RestError);



## 15\.创建服务器！

我们已经定义了数据库和路由，最后一件事就是添加用于管理调用的服务器实例。

Restify（和 Express）允许你对 REST API 执行大量的深度自定义，但同样，我们在本演练中将使用最基本的设置。

Javascript
		
	/**
	 * Our Server
	 */


	var server = restify.createServer({
	    name: "Azure Active Directroy TODO Server",
	    version: "2.0.1"
	});

	// Ensure we don't drop data on uploads
	server.pre(restify.pre.pause());

	// Clean up sloppy paths like //todo//////1//
	server.pre(restify.pre.sanitizePath());

	// Handles annoying user agents (curl)
	server.pre(restify.pre.userAgentConnection());

	// Set a per request bunyan logger (with requestid filled in)
	server.use(restify.requestLogger());

	// Allow 5 requests/second by IP, and burst to 10
	server.use(restify.throttle({
	    burst: 10,
	    rate: 5,
	    ip: true,
	}));

	// Use the common stuff you probably want
	server.use(restify.acceptParser(server.acceptable));
	server.use(restify.dateParser());
	server.use(restify.queryParser());
	server.use(restify.gzipResponse());
	server.use(restify.bodyParser({
	    mapParams: true
	})); // Allows for JSON mapping to REST


## 16\.将路由添加到服务器（目前不包括身份验证）

Javascript
		
	/// Now the real handlers. Here we just CRUD
	/**
	/*
	/* Each of these handlers are protected by our OIDCBearerStrategy by invoking 'oidc-bearer'
	/* in the pasport.authenticate() method. We set 'session: false' as REST is stateless and
	/* we don't need to maintain session state. You can experiement removing API protection
	/* by removing the passport.authenticate() method like so:
	/*
	/* server.get('/tasks', listTasks);
	/*
	**/
	server.get('/tasks', listTasks);
	server.get('/tasks', listTasks);
	server.get('/tasks/:owner', getTask);
	server.head('/tasks/:owner', getTask);
	server.post('/tasks/:owner/:task', createTask);
	server.post('/tasks', createTask);
	server.del('/tasks/:owner/:task', removeTask);
	server.del('/tasks/:owner', removeTask);
	server.del('/tasks', removeTask);
	server.del('/tasks', removeAll, function respond(req, res, next) {
	res.send(204);
	next();
	});
	// Register a default '/' handler
	server.get('/', function root(req, res, next) {
	var routes = [
	'GET /',
	'POST /tasks/:owner/:task',
	'POST /tasks (for JSON body)',
	'GET /tasks',
	'PUT /tasks/:owner',
	'GET /tasks/:owner',
	'DELETE /tasks/:owner/:task'
	];
	res.send(200, routes);
	next();
	});
	server.listen(serverPort, function() {
	var consoleMessage = '\n Azure Active Directory Tutorial';
	consoleMessage += '\n +++++++++++++++++++++++++++++++++++++++++++++++++++++';
	consoleMessage += '\n %s server is listening at %s';
	consoleMessage += '\n Open your browser to %s/tasks\n';
	consoleMessage += '+++++++++++++++++++++++++++++++++++++++++++++++++++++ \n';
	consoleMessage += '\n !!! why not try a $curl -isS %s | json to get some ideas? \n';
	consoleMessage += '+++++++++++++++++++++++++++++++++++++++++++++++++++++ \n\n';
	});


## 17\.在添加 OAuth 支持之前，让我们先运行服务器。

添加身份验证之前，请先测试服务器

最简单的检查方法是在命令行中使用 curl。在这样做之前，我们需要一个用于分析 JSON 输出的简单实用工具。为此，请安装 json 工具，因为下面的所有示例都要使用该工具。

`$npm install -g jsontool`  


这将全局安装 JSON 工具。现在，我们已安装了工具，让我们试运行服务器：

首先，请确保 monogoDB 实例正在运行。

`$sudo mongod`  


然后，切换到目录并开始运行。

`$ cd azuread` `$ node server.js`

`$ curl -isS http://127.0.0.1:8080 | json`  


Shell

	HTTP/1.1 200 OK
	Connection: close
	Content-Type: application/json
	Content-Length: 171
	Date: Tue, 14 Jul 2015 05:43:38 GMT
	[
	"GET /",
	"POST /tasks/:owner/:task",
	"POST /tasks (for JSON body)",
	"GET /tasks",
	"PUT /tasks/:owner",
	"GET /tasks/:owner",
	"DELETE /tasks/:owner/:task"
	]


然后，我们按如下所示添加一个任务：

`$ curl -isS -X POST http://127.0.0.1:8080/tasks/brandon/Hello`  


响应应为：

Shell
		
	HTTP/1.1 201 Created
	Connection: close
	Access-Control-Allow-Origin: *
	Access-Control-Allow-Headers: X-Requested-With
	Content-Type: application/x-www-form-urlencoded
	Content-Length: 5
	Date: Tue, 04 Feb 2014 01:02:26 GMT
	Hello

我们可以按如下所示列出 Brandon 的任务：

`$ curl -isS http://127.0.0.1:8080/tasks/brandon/`  


如果一切正常，我们可以将 OAuth 添加到 REST API 服务器。

**你已有一台装有 MongoDB 的 REST API 服务器！**


## 18\.将身份验证添加到 REST API 服务器

现在，我们已经运行了 REST API（顺便祝贺你！），接下来，让我们在 Azure AD 中利用它。

在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

`cd azuread`  


### 1：使用 passport-azure-ad 随附的 OIDCBearerStrategy

到目前为止，我们已构建一个典型的 REST TODO 服务器，其中不包含任何授权种类。这是我们将其结合在一起的起点。

首先，需指出要使用 Passport。在其他服务器配置之后紧接着执行此操作：

Javascript
		
	// Let's start using Passport.js

	server.use(passport.initialize()); // Starts passport
	server.use(passport.session()); // Provides session support


> [AZURE.TIP]
编写 API 时，应始终将数据链接到用户无法证明其在令牌中是唯一的项目。当此服务器存储 TODO 项目时，会根据我们放在“所有者”字段的令牌（通过 token.oid 调用）中的用户对象 ID 来存储。这可确保只有该用户可以访问其 TODO，其他任何人都不可以访问输入的 TODO。“所有者”API 中不公开任何信息，因此，外部用户可以请求其他的 TODO，即使它们已经过身份验证也一样。

接下来，我们将使用 passport-azure-ad 随附的 Bearer 策略。先看看下面的代码，稍后我将进行解释。将此代码放在上面粘贴的内容后面：

Javascript

	/**
	/*
	/* Calling the OIDCBearerStrategy and managing users
	/*
	/* Passport pattern provides the need to manage users and info tokens
	/* with a FindorCreate() method that must be provided by the implementor.
	/* Here we just autoregister any user and implement a FindById().
	/* You'll want to do something smarter.
	**/

	var findById = function(id, fn) {
	    for (var i = 0, len = users.length; i < len; i++) {
	        var user = users[i];
	        if (user.sub === id) {
	            log.info('Found user: ', user);
	            return fn(null, user);
	        }
	    }
	    return fn(null, null);
	};


	var bearerStrategy = new BearerStrategy(options,
	    function(token, done) {
	        log.info('verifying the user');
	        log.info(token, 'was the token retreived');
	        findById(token.sub, function(err, user) {
	            if (err) {
	                return done(err);
	            }
	            if (!user) {
	                // "Auto-registration"
	                log.info('User was added automatically as they were new. Their sub is: ', token.sub);
	                users.push(token);
	                owner = token.sub;
	                return done(null, token);
	            }
	            owner = token.sub;
	            return done(null, user, token);
	        });
	    }
	);

	passport.use(bearerStrategy);


Passport 使用适用于它的所有策略（Twitter、Facebook 等），所有策略写入器都依循类似的模式。查看该策略，你会发现，我们已将它作为 function() 来传递，其中包含一个令牌和一个用作参数的 done。策略完成所有工作之后，便尽责地返回。完成后，我们需要存储用户并隐藏令牌，因此不需要再次请求它。

> [AZURE.IMPORTANT]
上述代码使用了正好地服务器上进行身份验证的任何用户。这就是所谓的自动注册。在生产服务器中，你希望所有人都必须先经历你确定的注册过程。这通常是在使用者应用中看到的模式，可让向 Facebook 注册，但接着请求填写其他信息。如果这不是命令行程序，我们就只能从返回的令牌对象中提取电子邮件，然后请求他们填写其他信息。由于这是测试服务器，因此，我们直接将它们加入到内存中的数据库。

### 2\.最后保护一些终结点

通过结合要使用的协议指定 `passport.authenticate()` 调用来保护终结点。

让我们在服务器代码编辑路由，以做一些更有趣的事：

Javascript

	server.get('/tasks', passport.authenticate('oauth-bearer', {
	session: false
	}), listTasks);
	server.get('/tasks', passport.authenticate('oauth-bearer', {
	session: false
	}), listTasks);
	server.get('/tasks/:owner', passport.authenticate('oauth-bearer', {
	session: false
	}), getTask);
	server.head('/tasks/:owner', passport.authenticate('oauth-bearer', {
	session: false
	}), getTask);
	server.post('/tasks/:owner/:task', passport.authenticate('oauth-bearer', {
	session: false
	}), createTask);
	server.post('/tasks', passport.authenticate('oauth-bearer', {
	session: false
	}), createTask);
	server.del('/tasks/:owner/:task', passport.authenticate('oauth-bearer', {
	session: false
	}), removeTask);
	server.del('/tasks/:owner', passport.authenticate('oauth-bearer', {
	session: false
	}), removeTask);
	server.del('/tasks', passport.authenticate('oauth-bearer', {
	session: false
	}), removeTask);
	server.del('/tasks', passport.authenticate('oauth-bearer', {
	session: false
	}), removeAll, function respond(req, res, next) {
	res.send(204);
	next();
	});


## 19\.再次运行服务器应用程序并确保它拒绝你

再次使用 `curl` 来查看是否针对终结点提供了 OAuth2 保护。应该在针对此终结点运行任何客户端 SDK 之前执行此操作。返回的标头足以说明一切正常运作。

首先，请确保 monogoDB 实例正在运行：

	$sudo mongod

然后，切换到目录并开始运行。

	$ cd azuread $ node server.js

试用基本 POST：

	$ cd azuread
	$ node server.js


Shell
		
	HTTP/1.1 401 Unauthorized
	Connection: close
	WWW-Authenticate: Bearer realm="Users"
	Date: Tue, 14 Jul 2015 05:45:03 GMT
	Transfer-Encoding: chunked


401 在这里是正常的响应，表明 Passport 层正在尝试重定向到授权终结点，这正是你所希望的。

## 祝贺你！ 你已经创建了一个使用 OAuth2 的 REST API 服务！

这就是在不使用 OAuth2 兼容客户端的情况下，此服务器能够为你带来的最大优势。你需要学习其他演练。

如果你只要想要了解如何使用 Restify 和 OAuth2 实现 REST API，此示例中的代码足以帮助你学习开发和生成服务。

如果你对 ADAL 学习历程中的后续步骤感兴趣，我们建议你了解下面这些支持的 ADAL 客户端：

只需将这些代码克隆到开发人员计算机，然后根据演练中所述进行配置。

[ADAL for iOS](https://github.com/MSOpenTech/azure-activedirectory-library-for-ios)

[ADAL for Android](https://github.com/MSOpenTech/azure-activedirectory-library-for-android)


[AZURE.INCLUDE [active-directory-devquickstarts-additional-resources](../../includes/active-directory-devquickstarts-additional-resources.md)]

<!---HONumber=Mooncake_Quality_Review_1230_2016-->
