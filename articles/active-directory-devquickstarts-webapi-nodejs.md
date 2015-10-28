<properties
	pageTitle="Azure AD NodeJS 入门 | Windows Azure"
	description="如何生成一个与 Azure AD 集成以进行身份验证的 Node.js Web API。"
	services="active-directory"
	documentationCenter="nodejs"
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="04/28/2015"
	wacn.date="06/16/2015"/>

# 节点 WEB API 入门

[AZURE.INCLUDE [active-directory-devguide](../includes/active-directory-devguide.md)]

本演练将演示如何快速轻松地设置一个与 Azure Active Directory 集成的 REST API 服务，以使用 OAuth2 协议进行 API 保护。下载内容中包含的示例服务器可在面向 OSX 和 Linux 的任何平面上运行。

在完成本演练后，你应该可以生成一个运行的并具有以下功能的 REST API 服务器：

* 运行 REST API 接口和 JSON，并使用 MongoDB 作为持久性存储的 node.js 服务器
* 利用 OAuth2 API 保护，并在 Azure Active Directory 中使用 Bearner 令牌的 REST API


我们已在 GitHub 中的 Apache 2.0 许可证下发布了此运行示例的所有源代码，你可以任意克隆（甚至分发！）这些代码，并提供反馈和发出请求。

## 关于 Node.js 模块

在本演练中，我们将使用 Node.js 模块。模块是可加载的 JavaScript 包，可为你的应用程序提供特定功能。你通常会使用 Node.js NPM 命令行工具在 NPM 安装目录中安装模块，但一些模块（如 HTTP 模块）是作为核心 Node.js 包的一部分提供的。安装的模块保存在 Node.js 安装目录的根目录下的 node_modules 目录中。node_modules 目录中的每个模块都保留自己的 node_modules 目录，其中包含它依赖的所有模块，每个必需的模块都有一个 node_modules 目录。这种递归的目录结构表示了依赖关系链。

这种依赖关系链结构会导致应用程序占用空间变大，但保证满足所有依赖项的要求，并且开发中使用的模块版本也将在生产中使用。这使得生产应用程序的行为更有预测性，并防止出现影响用户的版本控制问题。

## 步骤 1：注册 Azure AD 租户

若要使用本示例，你需要一个 Azure Active Directory 租户。如果你不确定什么是租户或者如何获取租户，请参阅[如何获取 Azure AD 租户](/documentation/articles/active-directory-howto-tenant)。

## 步骤 2：将 Web API 添加到租户

获取 Azure Active Directory 租户后，将此示例应用程序添加到你的租户，以便可以使用它来保护 API。

若要使应用程序对用户进行身份验证，你首先需要在租户中注册新的应用程序。

- 登录到 Azure 管理门户。
- 在左侧的导航栏中单击“Active Directory”。
- 选择你要在其中注册应用程序的租户。
- 单击“应用程序”选项卡，然后在底部抽屉中单击“添加”。
- 根据提示创建一个新的 **Web 应用程序和/或 WebAPI**。
    - 应用程序的“名称”向最终用户描述你的应用程序
    -	“登录 URL”是应用程序的基本 URL。框架的默认值为 `https://localhost:8888`。
    - “应用程序 ID URI”是应用程序的唯一标识符。约定是使用 `https://<tenant-domain>/<app-name>`，例如 `https://contoso.partner.onmschina.cn/my-first-aad-app`
- 完成注册后，AAD 将为应用程序分配唯一的客户端标识符。在后面的部分中将会用到此值，因此，请从“配置”选项卡复制此值。

## 步骤 3：下载平台的 node.js
若要成功使用本示例，你必须正确安装 Node.js。

请从 [http://nodejs.org](http://nodejs.org) 安装 Node.js。

## 步骤 4：在平台上安装 MongoDB

若要成功使用本示例，你必须正确安装 MongoDB。我们将使用 MongoDB 来使 REST API 持久保留在服务器实例之间。

从 [http://mongodb.org](http://www.mongodb.org) 安装 MongoDB。

**注意：**本演练假定你为 MongoDB 使用默认的安装与服务器终结点，在编写本文时，该终结点为：mongodb://localhost


## 步骤 5：将 Restify 模块安装到 Web API 中

我们将使用 Resitfy 来生成 REST API。Restify 是从 Express 派生的精简灵活 Node.js 应用程序框架，它提供一套可靠的功能用于在 Connect 顶层生成 REST API。

### 安装 Restify

在命令行中，将目录切换到 azuread 目录。如果 **azuread** 目录不存在，请创建该目录。

		cd azuread - or- mkdir azuread; cd azuread

输入以下命令：

		npm install restify

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


## 步骤 6：将在 Passport.js 安装到 Web API

[Passport](http://passportjs.org/) 是 Node.js 的身份验证中间件。Passport 极其灵活并且采用模块化结构，可以在不造成干扰的情况下放入任何基于 Express 的应用程序或 Resitify Web 应用程序。一套综合性策略支持使用用户名和密码、Facebook、Twitter 等进行身份验证。我们针对 Azure Active directory 开发了一个策略。我们将安装此模块，然后添加 Azure Active Directory 策略插件。

在命令行中，将目录切换到 azuread 目录。

输入以下命令以安装 passport.js

		npm install passport

该命令的输出应如下所示：

	passport@0.1.17 node_modules\passport
	├── pause@0.0.1
	└── pkginfo@0.2.3

## 步骤 7：将 Passport.js 持有者令牌支持添加到 Web API

接下来，我们将使用 passport-bearer-http（[Passport](http://passportjs.org/) 的持有者处理程序）添加持有者策略。我们还将使用 node-jwt 添加 JWT 令牌处理程序支持。

**注意：**尽管 OAuth2 提供了可以颁发任何已知令牌类型的框架，但只有一部分令牌类型已得到广泛使用。用于保护终结点的令牌是持有者令牌。持有者令牌是 OAuth2 中最广泛颁发的令牌，许多实现假定持有者令牌是唯一颁发的令牌类型。

在命令行中，将目录切换到 **azuread** 目录。

键入以下命令以安装 Passport.js 模块：

- `npm install passport-oauth`
- `npm install passport-http-bearer`
- `npm install node-jwt`

该命令的输出应如下所示：

	ms-passport-wsfed-saml2@0.3.8 node_modules\passport-oauth  
	├── xtend@2.0.3
	├── xml-crypto@0.0.9
	├── xmldom@0.1.13
	└── xml2js@0.1.14 (sax@0.5.2)


## 步骤 8：将 MongoDB 模块添加到 Web API

我们将使用 MongoDB 作为数据存储。为此，我们需要安装这两个广泛使用的插件来管理模型和称为 Mongoose 的架构，以及 MongoDB 的数据库驱动程序（也称为 MongoDB）。


* `npm install mongoose`
* `npm install mongodb`

## 步骤 9：安装其他模块

接下来，我们将安装剩余的所需模块。


在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

		cd azuread


输入以下命令，以在 node_modules 目录中安装以下模块：

* `npm install crypto`
* `npm install assert-plus`
* `npm install posix-getopt`
* `npm install util`
* `npm install path`
* `npm install connect`
* `npm install xml-crypto`
* `npm install xml2js`
* `npm install xmldom`
* `npm install async`
* `npm install request`
* `npm install underscore`
* `npm install grunt-contrib-jshint@0.1.1`
* `npm install grunt-contrib-nodeunit@0.1.2`
* `npm install grunt-contrib-watch@0.2.0`
* `npm install grunt@0.4.1`
* `npm install xtend@2.0.3`
* `npm install bunyan`
* `npm update`


## 步骤 10：创建包含依赖项的 server.js

server.js 文件将提供 Web API 服务器的大多数功能。我们要将大部分代码添加到此文件。用于生产目的，需要将功能重构为较小的文件，例如单独的路由和控制器。在本演示中，我们将为此功能使用 server.js。

在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

		cd azuread

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
	

保存文件。稍后我们将会使用该文件。

## 步骤 11：创建一个配置文件用于存储 Azure AD 设置

此代码文件会将配置参数从 Azure Active Directory 门户传递到 Passport.js。当你在本演练的第一部分中向门户添加 Web API 时，已经创建了这些配置值。我们将解释在复制代码后，要输入其中的哪些参数值。


在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

	cd azuread

在偏好的编辑器中创建 `config.js` 文件，然后添加以下信息：

	Javascript
	// Don't commit this file to your public repos
    exports.creds = {
    mongoose_auth_local: 'mongodb://localhost/tasklist', // Your mongo auth uri goes here
    openid_configuration: 'https://login.microsoftonline.com/common/.well-known/openid-configuration', // For using Microsoft you should never need to change this.
    openid_keys: 'https://login.microsoftonline.com/common/discovery/keys', // For using Microsoft you should never need to change this. If absent will attempt to get from openid_configuration
	}





**注意：**很有可能不需要更改这些值。

**注意：**我们会频繁滚动更新密钥。请确保始终从“openid_keys”URL 提取密钥，并且应用程序能够访问 Internet。


## 步骤 12：将配置添加到 server.js 文件

我们需要在应用程序中，从你刚刚创建的配置文件读取这些值。为此，我们只需在应用程序中添加 .config 文件作为所需的资源，然后将全局变量设置为 config.js 文档中的值

在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

	cd azuread

在偏好的编辑器中打开 `server.js` 文件，并添加以下信息：

	Javascript
	var config = require('./config');

然后，在 `server.js` 中替换包含以下代码的新节：

	Javascript
	/**
	* Setup some configuration
	*/
	var mongoose = require('mongoose/');
	var serverPort = process.env.PORT || 8888;
	var serverURI = ( process.env.PORT ) ? 	config.creds.mongoose_auth_mongohq : 	config.creds.mongoose_auth_local;


## 步骤 13：创建 metadata.js 帮助器文件以帮助分析元数据/令牌

由于目标是只在 server.js 文件中保留应用程序逻辑，因此最好是在一个单独的文件中输入一些帮助器方法。这些方法只会帮助我们分析 OpenID Connect 的元数据，而与核心方案无关。最好地将它保存在单独的位置。随着演练的进行，我们将在此文件中添加越来越多的信息。

***注意：***你会注意到，此 metadata.js 文件将会分析 SAML 和 WS-Fed 的 XML，以及 OpenID Connect 的 JSON。这是设计使然，你在其他示例中也会使用此文件。现在你可以安全地忽略此问题。

在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

	cd azuread

在偏好的编辑器中创建 `metadata.js` 文件，然后添加以下信息：

	Javascript

	'use strict';

	var xml2js = require('xml2js');
	var request = require('request');
	var aadutils = require('./aadutils');
	var async = require('async');

	// Logging

	var bunyan = require('bunyan');
	var log = bunyan.createLogger({name: 'Microsoft OpenID Connect Passport Strategy'});

	var Metadata = function (url, authtype) {


  	if(!url) {
		throw new Error("Metadata: url is a required argument");
  	}
  	if(!authtype) {
    	throw new Error('OIDCBearerStrategy requires an authentication type specified to metadata parser. Valid types are saml, wsfed, or odic"');
  	}

  	this.url = url;
  	this.metadata = null;
  	this.authtype = authtype;
  	log.info(authtype, 'Metadata requested for authentication type');
	};

	Object.defineProperty(Metadata, 'url', {
  	get: function () {
    	return this.url;
  	}
	});

	Object.defineProperty(Metadata, 'saml', {
  	get: function () {
    	return this.saml;
  	}
	});

	Object.defineProperty(Metadata, 'wsfed', {
  	get: function () {
    	return this.wsfed;
  	}
	});

	Object.defineProperty(Metadata, 'oidc', {
  	get: function () {
    	return this.oidc;
  	}
	});


	Object.defineProperty(Metadata, 'metadata', {
  	get: function () {
    	return this.metadata;
  	}
	});

	Metadata.prototype.updateSamlMetadata = function(doc, next) {
  	log.info('Request to update the SAML Metadata');
  	try {

    this.saml = {};

    var entity = aadutils.getElement(doc, 'EntityDescriptor');
    var idp = aadutils.getElement(entity, 'IDPSSODescriptor');
    var signOn = aadutils.getElement(idp[0], 'SingleSignOnService');
    var signOff = aadutils.getElement(idp[0], 'SingleLogoutService');
    var keyDescriptor = aadutils.getElement(idp[0], 'KeyDescriptor');
    this.saml.loginEndpoint = signOn[0].$.Location;
    this.saml.logoutEndpoint = signOff[0].$.Location;

    // copy the x509 certs from the metadata
    this.saml.certs = [];
    for (var j=0;j<keyDescriptor.length;j++) {
      this.saml.certs.push(keyDescriptor[j].KeyInfo[0].X509Data[0].X509Certificate[0]);
    }
    next(null);
  	} catch (e) {
    	next(new Error('Invalid SAMLP Federation Metadata ' + e.message));
  	}
	};

	Metadata.prototype.updateOidcMetadata = function(doc, next) {
  	log.info('Request to update the Open ID Connect Metadata');
  	try {
    	this.oidc = {};

    var issuer = doc['issuer'];
    var keyDescriptor = aadutils.getElement(idp[0], 'keys');

    // copy the x509 certs from the metadata
    this.oidc.certs = [];
    for (var j=0;j<keyDescriptor.length;j++) {
      this.oidc.certs.push(keyDescriptor[j].KeyInfo[0].X509Data[0].X509Certificate[0]);
    }
    next(null);
  	} catch (e) {
    	next(new Error('Invalid Open ID Connect Federation Metadata ' + e.message));
  	}
	};


	Metadata.prototype.updateWsfedMetadata = function(doc, next) {
  	log.info('Request to update the WS Federation Metadata');
  	try {
    	this.wsfed = {};
    	var entity = aadutils.getElement(doc, 'EntityDescriptor');
    	var roles = aadutils.getElement(entity, 'RoleDescriptor');
    	for(var i = 0; i < roles.length; i++) {
      	var role = roles[i];
      	if(role['fed:SecurityTokenServiceEndpoint']) {
        	var endpoint = role['fed:SecurityTokenServiceEndpoint'];
        	var endPointReference = aadutils.getFirstElement(endpoint[0],'EndpointReference');
        	this.wsfed.loginEndpoint = aadutils.getFirstElement(endPointReference,'Address');

        var keyDescriptor = aadutils.getElement(role, 'KeyDescriptor');
        // copy the x509 certs from the metadata
        this.wsfed.certs = [];
        for (var j=0;j<keyDescriptor.length;j++) {
          this.wsfed.certs.push(keyDescriptor[j].KeyInfo[0].X509Data[0].X509Certificate[0]);
        }
        break;
      }
    }

    return next(null);
  	} catch (e) {
    	next(new Error('Invalid WSFED Federation Metadata ' + e.message));
  	}
	};

	Metadata.prototype.fetch = function(callback) {
  	var self = this;
  	log.info("Fetching metadata from the provided metadata URL: " + self.url);
  	async.waterfall([
    	// fetch the Federation metadata for the AAD tenant
    	function(next){
      	request(self.url, function (err, response, body) {
        	if(err) {
          	next(err);
        	} else if(response.statusCode !== 200) {
          	next(new Error("Error:" + response.statusCode +  " Cannot get AAD Federation metadata from " + self.url));
        	} else {
          	log.info(body, "retreived");
          	next(null, body);
        	}
      	});
    	},
    	function(body, next){
      	// parse the AAD Federation metadata xml

      if(self.authtype == "saml" || self.authtype == "wsfed") {
      log.info(body, "Parsing XML retreived from the endpoint");
      var parser = new xml2js.Parser({explicitRoot:true});
      // Note: xml responses from Azure AAD have a leading \ufeff which breaks xml2js parser!
      parser.parseString(body.replace("\ufeff", ""), function (err, data) {
        self.metatdata = data;
        next(err);

      });
    } else if(self.authtype == "oidc") {
      log.info(body, "Parsing JSON retreived from the endpoint");
      JSON.parse(body, function (err, data) {
        self.metatdata = data;
        next(err);
      });

    } else {

       next(new Error("Error: No Authentication type specified to metadata parser. Valid types are saml, wsfed, or odic"));
    }

    },

    function(next){
      if(self.authtype = "saml") {
      // update the SAML SSO endpoints and certs from the metadata
      self.updateSamlMetadata(self.metatdata, next);
    }},
    function(next){
      if(self.authtype = "wsfed") {
      // update the SAML SSO endpoints and certs from the metadata
      self.updateWsfedMetadata(self.metatdata, next);
    }},
    function(next){
      if(self.authtype = "oidc") {
      self.updateOidcMetadata(self.metadata, next);
    }},
  	], function (err) {
    	// return err or success (err === null) to callback
    	callback(err);
  	});
	};

	exports.Metadata = Metadata;

如你在代码中所看到的，它只会提取你在 `config.js` 中传入的 openid URL，然后分析该 URL 以获取我们将在 `server.js` 文件 使用的信息。欢迎你探讨此代码，并根据需要对它进行补充。

### 在 server.js 中加载 metadata.js 文件

我们需要告诉服务器从何处获取你刚刚编写的方法。

在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

	cd azuread

在偏好的编辑器中打开 `server.js` 文件，并添加以下信息：

	Javascript
	var metadata = require('./metadata);

接下来，在 `Configuration` 节的末尾添加此调用，以将 `config.js` 中的元数据文档发送到刚刚编写的分析器：

	Javascript
	this.aadutils = new var Metadata = require('./metadata').Metadata;


## 步骤 14：使用 Moongoose 添加 MongoDB 模型和架构信息

现在，我们已将这三个文件统一放在 REST API 服务中，接下来让我们的准备工作发挥作用。

对于本演练，我们将使用 MongoDB 来存储***步骤 4*** 中所述的任务。

回顾我们在 ***步骤 11*** 中创建的 `config.js` 文件，我们将数据库称为 `tasklist`，因为这是我们在 mogoose_auth_local 连接 URL 的末尾放置的内容。你无需事先在 MongoDB 中创建此数据库，当你首次运行服务器应用程序时，系统将创建此数据库（假定它不存在）。

现在，我们已告诉服务器要使用哪个 MongoDB 数据库，接下来我们需要编写一些附加的代码，以便为服务器任务创建模型和架构。

#### 模型介绍

我们的架构模型非常简单，你可以根据需要对其进行扩展。

NAME - 分配到任务的用户名。一个***字符串***。

TASK - 任务本身。一个***字符串***。

DATE - 任务截止日期。一个***日期时间***

COMPLETED - 任务是否已完成。一个***布尔值***

#### 在代码中创建架构


在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

	cd azuread

在偏好的编辑器中打开 `server.js` 文件，并在配置条目下面添加以下信息：

	Javascript
	/**
	*
	* Connect to MongoDB
	*/

	global.db = mongoose.connect(serverURI);
	var Schema = mongoose.Schema;  

这将连接到 MongoDB 服务器，并向我们返回一个 Schema 对象。

#### 使用该架构在代码中创建模型

在上面编写的代码下面，添加以下代码：

	Javascript
	/**
	/ Here we create a schema to store our tasks. Pretty simple schema for now.
	*/

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

## 步骤 15：为任务 REST API 服务器添加路由

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

#### 将默认路由添加到服务器

现在，我们将添加 Create、Retrieve、Update 和 Delete 的基本 CRUD 路由。

在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

	cd azuread

在偏好的编辑器中打开 `server.js` 文件，并在前面创建的数据库条目下面添加以下信息：

	Javascript

	/**
 	*
 	* APIs
 	*/

	function createTask(req, res, next) {

	// Resitify currently has a bug which doesn't allow you to set default headers
  	// This headers comply with CORS and allow us to mongodbServer our response to any origin

  	res.header("Access-Control-Allow-Origin", "*");
  	res.header("Access-Control-Allow-Headers", "X-Requested-With");

    // Create a new task model, fill it up and save it to Mongodb
  	var _task = new Task();

        if (!req.params.task) {
                req.log.warn('createTask: missing task');
                next(new MissingTaskError());
                return;
        }


  	_task.owner = req.params.owner;
   	_task.task = req.params.task;
   	_task.date = new Date();

  	_task.save(function (err) {
  	if (err) {
        req.log.warn(err, 'createTask: unable to save');
        next(err);
    } else {
    res.send(201, _task);

			}
  	});

  	return next();

	}


	/**
 	* Deletes a Task by name
 	*/
	function removeTask(req, res, next) {

        Task.remove( { task:req.params.task }, function (err) {
                if (err) {
                        req.log.warn(err,
                                     'removeTask: unable to delete %s',
                                     req.params.task);
                        next(err);
                } else {
                        res.send(204);
                        next();
                }
        });
	}

	/**
 	* Deletes all Tasks. A wipe
 	*/
	function removeAll(req, res, next) {
        Task.remove();
        res.send(204);
        return next();
	}    });
	}


	/**
 	*
 	*
 	*
 	*/
	function getTask(req, res, next) {


        Task.find(req.params.name, function (err, data) {
                if (err) {
                        req.log.warn(err, 'get: unable to read %s', req.params.name);
                        next(err);
                        return;
                }

                res.json(data);
        });

        return next();
	}


	/**
 	* Simple returns the list of TODOs that were loaded.
 	*
 	*/

	function listTasks(req, res, next) {
  	// Resitify currently has a bug which doesn't allow you to set default headers
  	// This headers comply with CORS and allow us to mongodbServer our response to any origin

  	res.header("Access-Control-Allow-Origin", "*");
  	res.header("Access-Control-Allow-Headers", "X-Requested-With");

  	console.log("server getTasks");

  	Task.find().limit(20).sort('date').exec(function (err,data) {

    if (err)
      return next(err);

    if (data.length > 0) {
            console.log(data);
        }

    if (!data.length) {
            console.log('there was a problem');
            console.log(err);
            console.log("There is no tasks in the database. Did you initalize the database as stated in the README?");
        }

    else {

        res.json(data);

        }
  	});

  	return next();
	}


### 为路由添加一些错误处理

最好添加一些错误处理，以便能够以客户端理解的方式与它交流遇到的问题。

在前面编写的代码下面添加以下代码：

	Javascript
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


	function TaskExistsError(name) {
        	assert.string(name, 'name');

        	restify.RestError.call(this, {
                	statusCode: 409,
                	restCode: 'TaskExists',
                	message: name + ' already exists',
                	constructorOpt: TaskExistsError
        	});

        	this.name = 'TaskExistsError';
	}
	util.inherits(TaskExistsError, restify.RestError);


	function TaskNotFoundError(name) {
        	assert.string(name, 'name');

        	restify.RestError.call(this, {
                	statusCode: 404,
                	restCode: 'TaskNotFound',
                	message: name + ' was not found',
                	constructorOpt: TaskNotFoundError
        	});

        	this.name = 'TaskNotFoundError';
	}

	util.inherits(TaskNotFoundError, restify.RestError);



## 步骤 16：创建服务器！

我们已经定义了数据库和路由，最后一件事就是添加用于管理调用的服务器实例。

Restify（和 Express）允许你对 REST API 执行大量的深度自定义，但同样，我们在本演练中将使用最基本的设置。

	Javascript
	/**
 	* Our Server
 	*/


	var server = restify.createServer({
        	name: "Azure Active Directroy TODO Server",
    	version: "1.0.0",
    	formatters: {
        	'application/json': function(req, res, body){
            	if(req.params.callback){
                	var callbackFunctionName = req.params.callback.replace(/[^A-Za-z0-9_.]/g, '');
                	return callbackFunctionName + "(" + JSON.stringify(body) + ");";
            	} else {
                	return JSON.stringify(body);
            	}
        	},
        	'text/html': function(req, res, body){
            	if (body instanceof Error)
                        	return body.stack;

                      	if (Buffer.isBuffer(body))
                        	return body.toString('base64');

                	return util.inspect(body);
        	},
        	'application/x-www-form-urlencoded': function(req, res, body){
            	if (body instanceof Error) {
                    	res.statusCode = body.statusCode || 500;
                    	body = body.message;
            	} else if (typeof (body) === 'object') {
                	body = body.task || JSON.stringify(body);
            	} else {
                	body = body.toString();
            	}

        	res.setHeader('Content-Length', Buffer.byteLength(body));
        	return (body);
        	}
    	}
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

        	// This lets us push JSON to the REST API endpoint as well. Maps x: y as /name:value

        	server.use(restify.bodyParser({ mapParams: false }));

        	/// Now the real handlers. Here we just CRUD

        	server.get('/tasks', listTasks);
        	server.head('/tasks', listTasks);
        	server.get('/tasks/:name', getTask);
        	server.head('/tasks/:name', getTask);
        	server.post('/tasks/:name/:task', createTask);
        	server.del('/tasks/:name/:task', removeTask);
        	server.del('/tasks/:name', removeTask);
        	server.del('/tasks', removeAll, function respond(req, res, next) { res.send(204); next(); });


        	// Register a default '/' handler

        	server.get('/', function root(req, res, next) {
                	var routes = [
                        	'GET     /',
                        	'POST    /tasks/:name/:task',
                        	'GET     /tasks',
                        	'PUT     /tasks/:name',
                        	'GET     /tasks/:name',
                        	'DELETE  /tasks/:name/:task'
                	];
                	res.send(200, routes);
                	next();
        	});

  	server.listen(serverPort, function() {

  	var consoleMessage = '\n Azure Active Directory Tutorial'
  	consoleMessage += '\n +++++++++++++++++++++++++++++++++++++++++++++++++++++'
  	consoleMessage += '\n %s server is listening at %s';
  	consoleMessage += '\n Open your browser to %s/tasks\n';
  	consoleMessage += '+++++++++++++++++++++++++++++++++++++++++++++++++++++ \n'
  	consoleMessage += '\n !!! why not try a $curl -isS %s | json to get some ideas? \n'
  	consoleMessage += '+++++++++++++++++++++++++++++++++++++++++++++++++++++ \n\n'  

  	console.log(consoleMessage, server.name, server.url, server.url, server.url);

	});


## 步骤 17：在添加 OAuth 支持之前，让我们运行服务器。

在继续学习演练的 OAuth 部分之前，最好是确保我们前面的工作没有任何错误。

最简单的检查方法是在命令行中使用 `curl`。在这样做之前，我们需要一个用于分析 JSON 输出的简单实用工具。为此，请安装 [json](https://github.com/trentm/json) 工具，因为下面的所有示例都要使用该工具。

	$npm install -g jsontool

这将全局安装 JSON 工具。现在，我们已安装了工具，让我们试运行服务器：

首先，请确保 monogoDB 实例正在运行。

	$sudo mongod

然后，切换到目录并开始运行。

	$ cd azuread
	$ node server.js

	$ curl -isS http://127.0.0.1:8888 | json
	HTTP/1.1 200 OK
	Connection: close
	Content-Type: application/x-www-form-urlencoded
	Content-Length: 145
	Date: Wed, 29 Jan 2014 03:41:24 GMT

	[
  	"GET     /",
  	"POST    /tasks/:owner/:task",
  	"GET     /tasks",
  	"DELETE  /tasks",
  	"PUT     /tasks/:owner",
  	"GET     /tasks/:owner",
  	"DELETE  /tasks/:task"
	]

然后，我们按如下所示添加一个任务：

	$ curl -isS -X POST http://127.0.0.1:8888/tasks/brandon/Hello

响应应为：

	HTTP/1.1 201 Created
	Connection: close
	Access-Control-Allow-Origin: *
	Access-Control-Allow-Headers: X-Requested-With
	Content-Type: application/x-www-form-urlencoded
	Content-Length: 5
	Date: Tue, 04 Feb 2014 01:02:26 GMT

	Hello

我们可以按如下所示列出 Brandon 的任务：

	$ curl -isS http://127.0.0.1:8888/tasks/brandon/

如果一切正常，我们可以将 OAuth 添加到 REST API 服务器。

## 步骤 18：将 Passport.js 代码添加到 REST API 服务器

现在，我们已经运行了 REST API（顺便祝贺你！），接下来，让我们在 Azure AD 中利用它。

在命令行中，将目录切换到 **azuread** 文件夹（如果尚未进入）：

	cd azuread

### 步骤 1：添加 Passport 模块

在偏好的编辑器中打开 `server.js` 文件，并在前面指定的模块加载位置下面添加以下信息。该位置应靠近文件的顶部，紧接在 `var aadutils = require('./aadutils');` import 的后面。

	Javascript
	/*
	*
	* Load our old friend Passport for OAuth2 flows
	*/

	var passport = require('passport')
  	, OAuth2Strategy = require('passport-oauth').OAuth2Strategy;


### 2.告诉服务器我们要使用身份验证

在偏好的编辑器中打开 `server.js` 文件，并在定义路由的 **server.get() 下面**、**server.listen()** 方法的上面添加以下信息。


我们需要告诉 Restify 开始使用它的 `authorizationParser()`，并查看 Authorization 标头的内容。

	Javascript
        server.use(restify.authorizationParser());


### 3.将 Passport OAuth2 模块添加到代码

我们在此处使用前面已添加到 config.js 文件中的特定 OAuth2 参数。如果 `aadutils.js` 文件确实在分析联合元数据文档，则系统应会填充所有这些值，即使这些值在 config.js 文件中是空白的。

	Javascript
	// Now our own handlers for authentication/authorization
	// Here we only use Oauth2 from Passport.js

	passport.use('provider', new OAuth2Strategy({
    	authorizationURL: authEndpoint,
    	tokenURL: tokenEndpoint,
    	clientID: clientID,
    	clientSecret: clientSecret,
    	callbackURL: callbackURL
  	},
  	function(accessToken, refreshToken, profile, done) {
    	User.findOrCreate({ UserId: profile.id }, 	function(err, user) {
      	done(err, user);
    	});
  	}
	));

	// Let's start using Passport.js

	server.use(passport.initialize());


### 步骤 4：添加 OAuth 身份验证的路由

	Javascript
	// Redirect the user to the OAuth 2.0 provider for authentication.  When
	// complete, the provider will redirect the user back to the application at
	//     /auth/provider/callback

	server.get('/auth/provider', passport.authenticate('provider'));

	// The OAuth 2.0 provider has redirected the user back to the application.
	// Finish the authentication process by attempting to obtain an access
	// token.  If authorization was granted, the user will be logged in.
	// Otherwise, authentication has failed.

	server.get('/auth/provider/callback',
  	passport.authenticate('provider', { successRedirect: '/',
                                      failureRedirect: '/login' }));


### 步骤 5：添加路由的 IsAuthenticated() 帮助器方法

	Javascript
	// Simple route middleware to ensure user is authenticated.
	//   Use this route middleware on any resource that needs to be protected.  If
	//   the request is authenticated (typically via a persistent login session),
	//   the request will proceed.  Otherwise, the user will be redirected to the
	//   login page.

	var ensureAuthenticated = function(req, res, next) {
  	if (req.isAuthenticated()) {
    	return next();
  	}
  	res.redirect('/login');
	};



### 步骤 6：添加 Cookie 缓存机制

	Javascript
	// Passport session setup.
	//   To support persistent login sessions, Passport needs to be able to
	//   serialize users into and deserialize users out of the session.  Typically,
	//   this will be as simple as storing the user ID when serializing, and finding
	//   the user by ID when deserializing.
	passport.serializeUser(function(user, done) {
  	done(null, user.email);
	});

	passport.deserializeUser(function(id, done) {
  	findByEmail(id, function (err, user) {
    	done(err, user);
  	});
	});

### 步骤 7：最后保护一些终结点

通过结合你要使用的协议指定 passport.authenticate() 调用来保护终结点。

让我们在服务器代码编辑路由，以做一些更有趣的事：

	Javascript
	server.get('/tasks', passport.authenticate('provider', { session: false }), listTasks);


## 步骤 19：再次运行服务器应用程序并确保它拒绝你

再次使用 `curl` 来查看是否针对终结点提供了 OAuth2 保护。应该在针对此终结点运行任何客户端 SDK 之前执行此操作。返回的标头足以说明一切正常运作。

首先，请确保 monogoDB 实例正在运行。

	$sudo mongod

然后，切换到目录并开始运行。

	$ cd azuread
	$ node server.js

试用基本 GET：

	$ curl -isS http://127.0.0.1:8888/tasks/
	HTTP/1.1 302 Moved Temporarily
	Connection: close
	Location: https://login.windows.net/468a75f4-f9a7-4dc4-a527-4f4522734790/oauth2/authorize?response_type=code&redirect_uri=&client_id=123
	Content-Length: 0
	Date: Tue, 04 Feb 2014 02:15:14 GMT


302 在这里是正常的响应，表明 Passport 层正在尝试重定向到授权终结点，这正是你所希望的。

## 祝贺你！ 你已经创建了一个使用 OAuth2 的 REST API 服务！

这就是在不使用 OAuth2 兼容客户端的情况下，此服务器能够为你带来的最大优势。你需要学习其他演练。

如果你只要想要了解如何使用 Restify 和 OAuth2 实现 REST API，此示例中的代码足以帮助你学习开发和生成服务。

如果你对 ADAL 学习历程中的后续步骤感兴趣，我们建议你了解下面这些支持的 ADAL 客户端：

只需将这些代码克隆到开发人员计算机，然后根据演练中所述进行配置。

[ADAL for iOS](https://github.com/MSOpenTech/azure-activedirectory-library-for-ios)

[ADAL for Android](https://github.com/MSOpenTech/azure-activedirectory-library-for-android)

[ADAL for .Net](http://msdn.microsoft.com/zh-cn/library/azure/jj573266.aspx)

<!---HONumber=60-->