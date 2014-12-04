<properties linkid="dev-nodejs-how-to-twilio-voice-sms-service" urlDisplayName="Twilio Voice and SMS Service" pageTitle="在 Azure 中使用 Twilio for Voice、VoIP 和 SMS 消息传递" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="Node.js" title=" VoIP" authors="" solutions="" manager="" editor="" />

# 在 Azure 中使用 Twilio for Voice、VoIP 和 SMS 消息传递

本指南演示如何构建与 Azure 上的 Twilio 和 node.js 进行通信的应用程序。

## 目录

-   [什么是 Twilio？][什么是 Twilio？]
-   [注册 Twilio (Microsoft Discount)][注册 Twilio (Microsoft Discount)]
-   [创建和部署 node.js Azure 网站][创建和部署 node.js Azure 网站]
-   [配置 Twilio 模块][配置 Twilio 模块]
-   [发起传出呼叫][发起传出呼叫]
-   [发送 SMS 消息][发送 SMS 消息]
-   [后续步骤][后续步骤]

<span id="whatis"></span></a>

## 什么是 Twilio？

Twilio 是一种 API 平台，借助于该平台，开发人员能够轻松地收发电话呼叫、收发短信以及将 VoIP 呼叫嵌入到基于浏览器的应用程序和本机移动应用程序中。在我们深入探讨其功能之前先简要看一下它的工作方式。

### 接收呼叫和短信

通过 Twilio，开发人员可以[购买可编程电话号码][购买可编程电话号码]，此类电话号码可用于收发呼叫和短信。在某一 Twilio 号码接收到传入呼叫或文本时，Twilio 将向您的 Web 应用程序发送一个 HTTP POST 或 GET 请求，请求提供有关如何处理该呼叫或文本的说明。您的服务器将使用 [TwiML][TwiML] 响应 Twilio 的 HTTP 请求，TwiML 是一组简单的 XML 标记，包含有关如何处理呼叫或文本的说明。稍后将显示 TwiML 的示例。

### 发出呼叫和发送短信

通过向 Twilio Web 服务 API 发出 HTTP 请求，开发人员可以发送短信或发起传出电话呼叫。对于传出呼叫，开发人员必须还指定一个 URL，该 URL 返回有关在其连接后如何处理该传出呼叫的 TwiML 说明。

### 在 UI 代码中嵌入 VoIP 功能（JavaScript、iOS 或 Android）

Twilio 提供一个客户端 SDK，它可以将任何桌面 Web 浏览器、iOS 应用程序或 Android 应用程序转换为 VoIP 电话。在本文中，我们将重点介绍如何在浏览器中使用 VoIP 呼叫。除了在浏览器中运行的 Twilio JavaScript SDK 以外，还必须使用服务器端应用程序（我们的 node.js 应用程序）向 JavaScript 客户端发布一个“功能标记”。您还可以在 [Twilio 开发人员博客][Twilio 开发人员博客]上了解有关如何将 VoIP 用于 node.js 的更多信息。

<span id="signup"></span></a>

## 注册 Twilio (Microsoft Discount)

在使用 Twilio 服务之前，必须首先[注册帐户][注册帐户]。Microsoft Azure 客户可享受特别折扣 - [请务必在此处注册][注册帐户]!

<span id="azuresite"></span></a>

## 创建和部署 node.js Azure 网站

接下来，您将需要创建在 Azure 上运行的 node.js 网站。[此处提供有关如何进行创建的正式文档][此处提供有关如何进行创建的正式文档]。概括地说，您将执行以下操作：

-   如果您尚没有 Azure 帐户，则注册 Azure 帐户
-   使用 Azure 管理控制台创建一个新网站
-   添加源代码管理支持（假定您使用 Git）
-   使用一个简单的 node.js Web 应用程序创建文件 `server.js`
-   将这个简单的应用程序部署到 Azure

<span id="twiliomodule"></span></a>

## 配置 Twilio 模块

接下来，我们将开始编写一个可使用 Twilio API 的简单的 node.js 应用程序。在开始前，我们需要配置我们的 Twilio 帐户凭据。

### 在系统环境变量中配置 Twilio 凭据

为了向 Twilio 后端发出已验证了身份的请求，我们需要我们的帐户 SID 和授权标记，它起到为我们的 Twilio 帐户设置的用户名和密码的功能。将它们配置为可用于 Azure 中的节点模块的最安全方式是通过系统环境变量，可直接在 Azure 管理控制台中设置这些变量。

选择您的 node.js 网站，然后单击“配置”链接。如果您向下滚动一点，将会看到一个区域，您可以在该区域中为应用程序设置配置属性。按照显示输入您的 Twilio 帐户凭据（[可在您的 Twilio 仪表板上找到][可在您的 Twilio 仪表板上找到]）- 请确保将它们分别命名为“TWILIO\_ACCOUNT\_SID”和“TWILIO\_AUTH\_TOKEN”：

![Azure 管理控制台][Azure 管理控制台]

在您配置了这些变量后，在 Azure 控制台中重新启动您的应用程序。

### 在 package.json 中声明 Twilio 模块

接下来，我们需要创建一个 package.json，以便通过 [npm][npm] 管理我们的节点模块依赖项。在与您在 Azure/node.js 教程中创建的“server.js”文件所处的相同级别上，创建一个名为“package.json”的文件。在此文件中，放置以下代码：

{
 "name":"application-name",
 "version":"0.0.1",
 "private":true,
 "scripts":{
 "start":"node server"
 },
 "dependencies":{
 "express":"3.1.0",
 "ejs":"*",
 "twilio":"*"
 }
 }

这会将该 twilio 模块声明为一个依赖项，以及常见的 [Express Web 框架][Express Web 框架]和 EJS 模板引擎。好了，现在全都设置好了 - 我们就来编写一些代码吧！

<span id="makecall"></span></a>

## 发起传出呼叫

我们将创建一个简单的表单，它将向我们选择的号码发出呼叫。打开 server.js，然后输入以下代码。请注意，在出现“CHANGE\_ME”的地方放置您的 azure 网站的名称：

// Module dependencies
 var express = require('express'),
 path = require('path'),
 http = require('http'),
 twilio = require('twilio');

// Create Express web application
 var app = express();

// Express configuration
 app.configure(function(){
 app.set('port', process.env.PORT || 3000);
 app.set('views', \_\_dirname + '/views');
 app.set('view engine', 'ejs');
 app.use(express.favicon());
 app.use(express.logger('dev'));
 app.use(express.bodyParser());
 app.use(express.methodOverride());
 app.use(app.router);
 app.use(express.static(path.join(\_\_dirname, 'public')));
 });
 app.configure('development', function(){
 app.use(express.errorHandler());
 });

// Render an HTML user interface for the application's home page
 app.get('/', function(request, response) {
 response.render('index');
 });

// Handle the form POST to place a call
 app.post('/call', function(request, response) {
 var client = twilio();
 client.makeCall({
 // make a call to this number
 to:request.body.number,

          // Change to a Twilio number you bought - see:
          // https://www.twilio.com/user/account/phone-numbers/incoming
          from:'+15558675309',

          // A URL in our app which generates TwiML
          // Change "CHANGE_ME" to your app's name
          url:'https://CHANGE_ME.azurewebsites.net/outbound_call'
      }, function(error, data) {
          // Go back to the home page
          response.redirect('/');
      });

});

// Generate TwiML to handle an outbound call
 app.post('/outbound\_call', function(request, response) {
 var twiml = new twilio.TwimlResponse();

      // Say a message to the call's receiver 
      twiml.say('hello - thanks for checking out Twilio and Azure', {
          voice:'woman'
      });

      response.set('Content-Type', 'text/xml');
      response.send(twiml.toString());

});

// Start server
 http.createServer(app).listen(app.get('port'), function(){
 console.log("Express server listening on port " + app.get('port'));
 });

接下来，创建名为“views”的目录 - 在该目录内，创建包含以下内容的名为“index.ejs”的文件：

\<!DOCTYPE html\>
 \<html\>
 \<head\>
 \<title\>Twilio Test\</title\>
 \<style\>
 input { height:20px; width:300px; font-size:18px; margin:5px; padding:5px; }
 \</style\>
 \</head\>
 \<body\>
 \<h1\>Twilio Test\</h1\>
 \<form action="/call" method="POST"\>
 \<input placeholder="Enter a phone number" name="number"/\>
 \<br/\>
 \<input type="submit" value="Call the number above"/\>
 \</form\>
 \</body\>
 \</html\>

现在，将您的网站部署到 Azure 并且打开您的主页。您应该能够在文本字段中输入电话号码，并收到来自您的 Twilio 号码的呼叫！

<span id="sendmessage"></span></a>

## 发送 SMS 消息

现在，我们将设置一个用户界面并且构建用于发送短信的处理逻辑。打开“server.js”，然后将以下代码添加到对“app.post”的最后一个调用后：

app.post('/sms', function(request, response) {
 var client = twilio();
 client.sendSms({
 // send a text to this number
 to:request.body.number,

          // A Twilio number you bought - see:
          // https://www.twilio.com/user/account/phone-numbers/incoming
          from:'+15558675309',

          // The body of the text message
          body: request.body.message
          
      }, function(error, data) {
          // Go back to the home page
          response.redirect('/');
      });

});

在“views/index.ejs”中，在第一个表单下添加另一个表单，以便提交一个号码和一个短信：

\<form action="/sms" method="POST"\>
 \<input placeholder="Enter a phone number" name="number"/\>
 \<br/\>
 \<input placeholder="Enter a message to send" name="message"/\>
 \<br/\>
 \<input type="submit" value="Send text to the number above"/\>
 \</form\>

将您的应用程序重新部署到 Azure，现在您应该能够提交该表单并且亲自（或者您的任何好朋友）发送短信了！

<span id="nextsteps"></span></a>

## 后续步骤

您现在已经了解了使用 node.js 和 Twilio 生成进行通信的应用程序的基础知识。但这些示例只是对 Twilio 和 node.js 进行了蜻蜓点水般地介绍。有关将 Twilio 用于 node.js 的更多信息，请查看以下资源：

-   [正式模块文档][正式模块文档]
-   [有关具有 node.js 应用程序的 VoIP 的教程][Twilio 开发人员博客]
-   [Votr - 具有 node.js 和 CouchDB 的一个实时 SMS 投票应用程序（三个部分）][Votr - 具有 node.js 和 CouchDB 的一个实时 SMS 投票应用程序（三个部分）]
-   [在浏览器中将编程与 node.js 配合使用][在浏览器中将编程与 node.js 配合使用]

我们希望您喜欢在 Azure 上使用 node.js 和 Twilio！

  [什么是 Twilio？]: #whatis
  [注册 Twilio (Microsoft Discount)]: #signup
  [创建和部署 node.js Azure 网站]: #azuresite
  [配置 Twilio 模块]: #twiliomodule
  [发起传出呼叫]: #makecall
  [发送 SMS 消息]: #sendmessage
  [后续步骤]: #nextsteps
  [购买可编程电话号码]: https://www.twilio.com/user/account/phone-numbers/available/local
  [TwiML]: https://www.twilio.com/docs/api/twiml
  [Twilio 开发人员博客]: http://www.twilio.com/blog/2013/04/introduction-to-twilio-client-with-node-js.html
  [注册帐户]: http://ahoy.twilio.com/azure
  [此处提供有关如何进行创建的正式文档]: http://www.windowsazure.com/zh-cn/develop/nodejs/tutorials/create-a-website-(mac)/
  [可在您的 Twilio 仪表板上找到]: https://www.twilio.com/user/account
  [Azure 管理控制台]: ./media/partner-twilio-nodejs-how-to-use-voice-sms/twilio_1.png
  [npm]: http://npmjs.org
  [Express Web 框架]: http://expressjs.com
  [正式模块文档]: http://twilio.github.io/twilio-node/
  [Votr - 具有 node.js 和 CouchDB 的一个实时 SMS 投票应用程序（三个部分）]: http://www.twilio.com/blog/2012/09/building-a-real-time-sms-voting-app-part-1-node-js-couchdb.html
  [在浏览器中将编程与 node.js 配合使用]: http://www.twilio.com/blog/2013/06/pair-programming-in-the-browser-with-twilio.html
