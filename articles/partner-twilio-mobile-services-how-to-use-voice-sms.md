<properties
	pageTitle="使用 Twilio 实现语音和短信功能 | Windows Azure"
	description="了解如何在 Azure 移动服务中使用 Twilio API 执行常见任务。"
	services="mobile-services"
	documentationCenter=""
	authors="devinrader"
	manager="twilio"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="06/16/2015"
	wacn.date="10/22/2015"/>


# 如何在移动服务中使用 Twilio 实现语音和短信功能

本主题说明如何在 Azure 移动服务中使用 Twilio API 执行常见任务。在本教程中，你将要了解如何创建自定义 API 脚本，以使用 Twilio API 发起通话以及发送短信服务 (SMS) 消息。

## <a id="WhatIs"></a>什么是 Twilio？
Twilio 为将来的商业沟通提供强大支持，并使开发人员能够将语音、VoIP 和消息传送嵌入到应用程序中。它们对基于云的全球环境中所需的所有基础结构进行虚拟化，并通过 Twilio 通信 API 平台将其公开。可轻松构建和扩展应用程序。享受现用现付定价所带来的灵活性，并从云可靠性中受益。

利用 **Twilio 语音**，你的应用程序可以发起和接收电话呼叫。**Twilio SMS** 使你的应用程序能够发送和接收 SMS 消息。利用 **Twilio 客户端**，你可以从任何手机、平板电脑或浏览器发起 VoIP 呼叫并支持 WebRTC。

## <a id="Pricing"></a>Twilio 定价和特别优惠
Azure 客户在升级 Twilio 帐户后即可获得[特别优惠][special_offer]：10 美元的 Twilio 信用额度。此 Twilio 信用可应用于任何 Twilio 使用（10 美元信用等价于发送多达 1,000 条 SMS 消息或接收长达 1000 分钟的入站语音，具体取决电话号码和消息或呼叫目标的位置）。兑换此 Twilio 信用额度并从 [ahoy.twilio.com/azure][special_offer] 开始。

Twilio 是一种现用现付服务。没有设置费用，并且您可以随时关闭您的帐户。你可以在 [Twilio 定价][twilio_pricing]上找到更多详细信息。

## <a id="Concepts"></a>概念
Twilio API 是一个为应用程序提供语音和 SMS 功能的 RESTful API。提供了多种语言版本的客户端库；有关列表，请参阅 [Twilio API 库][twilio_libraries]。其他教程介绍了如何使用 Twilio 和以 <!--[-->.NET<!--][azure_twilio_howto_dotnet]-->、[node.js][azure_twilio_howto_node]、<!--[-->Java<!--][azure_twilio_howto_java]-->、[PHP][azure_twilio_howto_php]、[Python][azure_twilio_howto_python] 或 <!--[-->Ruby<!--][azure_twilio_howto_ruby]--> 编写的任何 Azure 应用程序。

Twilio API 的关键方面是 Twilio 谓词和 Twilio 标记语言 (TwiML)。

### <a id="Verbs"></a>Twilio 谓词
API 利用了 Twilio 谓词；例如，**&lt;Say&gt;** 谓词指示 Twilio 在呼叫时传递语音消息。

下面是 Twilio 谓词的列表。通过 [Twilio 标记语言文档](http://www.twilio.com/docs/api/twiml)了解其他谓词和功能。

* **&lt;Dial&gt;**：将呼叫方连接到其他电话。
* **&lt;Gather&gt;**：收集通过电话按键输入的数字。
* **&lt;Hangup&gt;**：结束呼叫。
* **&lt;Play&gt;**：播放音频文件。
* **&lt;Pause&gt;**：安静地等待指定时间（以秒为单位）。
* **&lt;Record&gt;**：录制呼叫方的声音并返回包含该录音的文件的 URL。
* **&lt;Redirect&gt;**：将对呼叫或 SMS 的控制转移到其他 URL 上的 TwiML。
* **&lt;Reject&gt;**：拒绝对 Twilio 号码的传入呼叫而不向你收费
* **&lt;Say&gt;**：将文本转换为呼叫中生成的语音。
* **&lt;Sms&gt;**：发送 SMS 消息。

### <a id="TwiML"></a>TwiML
TwiML 是一组基于 XML 的指令，这些指令以用于指示 Twilio 如何处理呼叫或 SMS 的 Twilio 谓词为基础。

例如，以下 TwiML 会将文本 **Hello World** 转换为语音。

    <?xml version="1.0" encoding="UTF-8" ?>
    <Response>
       <Say>Hello World</Say>
    </Response>

当应用程序调用 Twilio API 时，某个 API 参数将为返回 TwiML 响应的 URL。在开发过程中，可以使用 Twilio 提供的 URL 来提供应用程序所使用的 TwiML 响应。还可以托管你自己的 URL 来生成 TwiML 响应，也可以选择使用 **TwiMLResponse** 对象。

有关 Twilio 谓词、其属性和 TwiML 的详细信息，请参阅 [TwiML][twiml]。有关 Twilio API 的其他信息，请参阅 [Twilio API][twilio_api]。

## <a id="CreateAccount"></a>创建 Twilio 帐户
准备好获取 Twilio 帐户后，请在[试用 Twilio][try_twilio] 上注册。可以先使用免费帐户，以后再升级您的帐户。

注册 Twilio 帐户时，您将收到帐户 ID 和身份验证令牌。需要二者才能发起 Twilio API 呼叫。为了防止对您的帐户进行未经授权的访问，请保护身份验证令牌。你的帐户 ID 和身份验证令牌会分别显示在 [Twilio 帐户页][twilio_account]上标记为“帐户 SID”和“身份验证令牌”的字段中。

## <a id="create_app"></a>创建移动服务
托管启用 Twilio 的应用程序的移动服务与其他任何移动服务没有区别。只需添加 Twilio node.js 库，就能从移动服务自定义 API 脚本引用该库。有关创建初始移动服务的信息，请参阅[移动服务入门](/documentation/articles/mobile-services-ios-get-started)。

## <a id="ConfigureMobileService"></a>配置移动服务以使用 Twilio Node.js 库
Twilio 提供了可包装 Twilio 各个方面的 Node.js 库，使你能够以简单且轻松的方式与 Twilio REST API 和 Twilio 客户端进行交互，从而生成 TwiML 响应。

若要在移动服务中使用 Twilio node.js 库，你必须利用移动服务 npm 模块支持（将脚本存储在源代码管理中即可）。

1. 完成教程：[在源代码管理中存储脚本](/documentation/articles/mobile-services-store-scripts-source-control)。该教程将引导你为移动服务设置源代码管理，并将服务器脚本存储在 Git 存储库中。

2. 为移动服务设置源代码管理以后，在本地计算机中打开存储库，浏览到 `\services` 子文件夹，在文本编辑器中打开 package.json 文件，然后将以下字段添加到 **dependencies** 对象：

		"twilio": "~1.7.0"
 
3. 将 Twilio 程序包引用添加到 **dependencies** 对象以后，package.json 文件应如下所示：

		{
		  "name": "todolist",
		  "version": "1.0.0",
		  "description": "todolist - hosted on Azure Mobile Services",
		  "main": "server.js",
		  "engines": {
		    "node": ">= 0.8.19"
		  },
		  "dependencies": {
			"twilio": "~1.7.0" 
		  },
		  "devDependencies": {},
		  "scripts": {},
		  "author": "unknown",
		  "licenses": [],
		  "keywords":[]
		}

	>[AZURE.NOTE]Twilio 的依赖关系应添加成 `"twilio": "~1.7.0"` 形式，带 (~)。不支持带插入符号 (^) 的引用。

4. 提交此文件更新，并将更新推送回移动服务。

	对 package.json 文件进行这种更新会重启移动服务。
	
移动服务此时会安装和加载 Twilio 程序包，这样你就可以在自定义 API 和表脚本中引用和使用 Twilio 库。

## <a id="howto_make_call"></a>如何：发起传出呼叫
以下脚本演示了如何使用 **makeCall** 函数从移动服务发起传出呼叫。此代码还使用 Twilio 提供的网站返回 Twilio 标记语言 (TwiML) 响应。用你的值替换“呼叫方”和“被呼叫方”电话号码，并确保在运行代码之前验证 Twilio 帐户的“呼叫方”电话号码。

    var twilio = require('twilio');

    exports.post = function(request, response) {

        var client = new twilio.RestClient('[ACCOUNT_SID]', 'AUTH_TOKEN');

        client.makeCall({
            to:'+16515556677', 
            from: '+14506667788',
            url: 'http://www.example.com/twiml.php' 

        }, function(err, responseData) {
            console.log(responseData.from); 
            response.send(200, '');
        });
    };

有关传入到 **client.makeCall** 函数中的参数的详细信息，请参阅 [http://www.twilio.com/docs/api/rest/making-calls][twilio_rest_making_calls]。

如前所述，此代码使用 Twilio 提供的网站返回 TwiML 响应。可以改用您自己的站点来提供 TwiML 响应。有关详细信息，请参阅[如何：从你自己的网站提供 TwiML 响应](#howto_provide_twiml_responses)。

## <a id="howto_send_sms"></a>如何：发送 SMS 消息
以下代码演示了如何使用 **sendSms** 函数发送 SMS 消息。“呼叫方”号码由 Twilio 提供，供试用帐户用来发送 SMS 消息。在运行代码前，必须验证你的 Twilio 帐户的“被呼叫方”号码。

    var twilio = require('twilio');

    exports.post = function(request, response) {

        var client = new twilio.RestClient('[ACCOUNT_SID]', 'AUTH_TOKEN');
 
        client.sendSms({
            to:'[]',
            from:'[]',
            body:'ahoy hoy! Testing Twilio and node.js'
        }, function(error, message) {
    
            // The "error" variable will contain error information, if any.
            // If the request was successful, this value will be "false"
            if (!error) {
                console.log('Success! The SID for this SMS message is: ' + message.sid);
                console.log('Message sent on: ' + message.dateCreated);
            }
            else {
                console.log('Oops! There was an error.');
            }
            response.send(200, { error : error } );
        });
    };


## <a id="howto_provide_twiml_responses"></a>如何：从你自己的网站提供 TwiML 响应

当您的应用程序开始调用 Twilio API 时（例如通过 client.InitiateOutboundCall 方法），Twilio 会将您的请求发送到预期返回 TwiML 响应的 URL。“如何：发起传出呼叫”中的示例使用 Twilio 提供的 URL http://twimlets.com/message 返回该响应。

> [AZURE.NOTE]虽然 TwiML 专供 Web 服务使用，但您可以在浏览器中查看 TwiML。例如，单击 [twimlet\_message\_url](http://twimlets.com/message) 可查看空 &lt;Response&gt; 元素；又如，单击 [twimlet\_message\_url\_hello\_world](http://twimlets.com/message?Message%5B0%5D=Hello%20World) 可查看包含 &lt;Say&gt; 元素的 &lt;Response&gt; 元素。

您可以创建自己的返回 HTTP 响应的 URL 网站，而不用依赖 Twilio 提供的 URL。您可以使用任何语言创建返回 HTTP 响应的站点。本主题假设您将从 ASP.NET 泛型处理程序托管该 URL。

以下脚本将生成在呼叫中念出 Hello World 的 TwiML 响应。

    var twilio = require('twilio');

    exports.post = function(request, response) {
        var resp = new twilio.TwimlResponse();
        resp.say({voice:'woman'}, 'ahoy hoy! Testing Twilio and node.js');
        response.set('Content-Type', 'text/xml');
        response.send(200, resp.toString());
    };

有关 TwiML 的详细信息，请参阅 [https://www.twilio.com/docs/api/twiml](https://www.twilio.com/docs/api/twiml)。

在设置提供 TwiML 响应的方法后，可按以下代码示例中所示，将该 URL 传入 **client.makeCall** 方法：
    
    var twilio = require('twilio');

    exports.post = function(request, response) {

        var client = new twilio.RestClient('[ACCOUNT_SID]', 'AUTH_TOKEN');

        client.makeCall({
            to:'+16515556677', 
            from: '+14506667788',
            url: 'http://<your_mobile_service>.azure-mobile.net/api/makeCall' 

        }, function(err, responseData) {

            console.log(responseData.from);
            response.send(200, '');
        });
    };

[AZURE.INCLUDE [twilio\_additional\_services\_and\_next\_steps](../includes/twilio_additional_services_and_next_steps.md)]


[twilio_rest_making_calls]: http://www.twilio.com/docs/api/rest/making-calls

[twilio_pricing]: http://www.twilio.com/pricing
[special_offer]: http://ahoy.twilio.com/azure
[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api]: http://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_account]: https://www.twilio.com/user/account
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#


[azure_twilio_howto_dotnet]: /documentation/articles/twilio-dotnet-how-to-use-for-voice-sms/
[azure_twilio_howto_java]: /documentation/articles/partner-twilio-java-how-to-use-voice-sms/
[azure_twilio_howto_node]: /documentation/articles/how-to-guides/twilio-voice-and-sms-service/
[azure_twilio_howto_ruby]: /documentation/articles/partner-twilio-ruby-how-to-use-voice-sms/
[azure_twilio_howto_python]: /documentation/articles/partner-twilio-python-how-to-use-voice-sms/
[azure_twilio_howto_php]: /documentation/articles/partner-twilio-php-how-to-use-voice-sms/
 

<!---HONumber=74-->