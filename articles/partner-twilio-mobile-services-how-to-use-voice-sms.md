<properties linkid="develop-mobile-tutorials-twilio-for-voice-and-sms" pageTitle="Use Twilio for Voice and SMS Capabilities | Mobile Dev Center" metaKeywords="" description="Learn how to perform common tasks using the Twilio API with Azure Mobile Services." metaCanonical="" services="" documentationCenter="Mobile" title="How to use Twilio for voice and SMS capabilities from Mobile Services" authors="twilio" solutions="" manager="" editor="" />

# 如何在移动服务中使用 Twilio 实现语音和短信功能

本主题说明如何在 Azure 移动服务中使用 Twilio API 执行常见任务。在本教程中，你将要了解如何创建自定义 API 脚本，以使用 Twilio API 发起通话以及发送短信服务 (SMS) 消息。

## <span id="WhatIs"></span></a>什么是 Twilio？

Twilio 为将来的商业沟通提供强大支持，并使开发人员能够将语音、VoIP 和消息传送嵌入到应用程序中。它们对基于云的全球环境中所需的所有基础结构进行虚拟化，并通过 Twilio 通信 API 平台将其公开。可轻松构建和扩展应用程序。享受现用现付定价所带来的灵活性，并从云可靠性中受益。

利用 **Twilio 语音**，你的应用程序可以发起和接收电话呼叫。**Twilio SMS** 使你的应用程序能够发送和接收 SMS 消息。利用 **Twilio 客户端**，你可以从任何手机、平板电脑或浏览器发起 VoIP 呼叫并支持 WebRTC。

## <span id="Pricing"></span></a>Twilio 定价和特别优惠

Azure 客户可享受[特别优惠][特别优惠]：升级 Twilio 帐户即可免费获得 10 美元的 Twilio 信用。此 Twilio 信用可应用于任何 Twilio 使用（10 美元信用等价于发送多达 1,000 条 SMS 消息或接收长达 1000 分钟的入站语音，具体取决电话号码和消息或呼叫目标的位置）。兑换此 Twilio 信用额度并从 [ahoy.twilio.com/azure][特别优惠] 开始。

Twilio 是一种现用现付服务。没有设置费用，并且你可以随时关闭你的帐户。你可以在 [Twilio 定价][Twilio 定价]上找到更多详细信息。

## <span id="Concepts"></span></a>概念

Twilio API 是一个为应用程序提供语音和 SMS 功能的 RESTful API。提供了多种语言版本的客户端库；有关列表，请参阅 [Twilio API 库][Twilio API 库]。其他教程介绍了如何使用 Twilio 和以 [.NET][.NET]、[node.js][node.js]、[Java][Java]、[PHP][PHP]、[Python][Python] 或 [Ruby][Ruby] 编写的任何 Azure 应用程序。

Twilio API 的关键方面是 Twilio 谓词和 Twilio 标记语言 (TwiML)。

### <span id="Verbs"></span></a>Twilio 谓词

API 利用了 Twilio 谓词；例如，**\<Say\>** 谓词指示 Twilio 在呼叫时传递语音消息。

下面是 Twilio 谓词的列表。通过 [Twilio 标记语言文档][Twilio 标记语言文档]了解其他谓词和功能。

-   **\<Dial\>**：将呼叫方连接到其他电话。
-   **\<Gather\>**：收集通过电话按键输入的数字。
-   **\<Hangup\>**：结束呼叫。
-   **\<Play\>**：播放音频文件。
-   **\<Pause\>**：安静地等待指定时间（以秒为单位）。
-   **\<Record\>**：录制呼叫方的声音并返回包含该录音的文件的 URL。
-   **\<Redirect\>**：将对呼叫或 SMS 的控制转移到其他 URL 上的 TwiML。
-   **\<Reject\>**：拒绝对 Twilio 号码的传入呼叫而不向你收费
-   **\<Say\>**：将文本转换为呼叫中生成的语音。
-   **\<Sms\>**：发送 SMS 消息。

### <span id="TwiML"></span></a>TwiML

TwiML 是一组基于 XML 的指令，这些指令以用于指示 Twilio 如何处理呼叫或 SMS 的 Twilio 谓词为基础。

例如，以下 TwiML 会将文本 **Hello World** 转换为语音。

    <?xml version="1.0" encoding="UTF-8" ?>
    <Response>
       <Say>Hello World</Say>
    </Response>

当应用程序调用 Twilio API 时，某个 API 参数将为返回 TwiML 响应的 URL。在开发过程中，可以使用 Twilio 提供的 URL 来提供应用程序所使用的 TwiML 响应。还可以托管你自己的 URL 来生成 TwiML 响应，也可以选择使用 **TwiMLResponse** 对象。

有关 Twilio 谓词、其属性和 TwiML 的详细信息，请参阅 [TwiML][Twilio 标记语言文档]。有关 Twilio API 的其他信息，请参阅 [Twilio API][Twilio API]。

## <span id="CreateAccount"></span></a>创建 Twilio 帐户

准备好获取 Twilio 帐户后，请在[试用 Twilio][试用 Twilio] 上注册。可以先使用免费帐户，以后再升级你的帐户。

注册 Twilio 帐户时，你将收到帐户 ID 和身份验证令牌。需要二者才能发起 Twilio API 呼叫。为了防止对你的帐户进行未经授权的访问，请保护身份验证令牌。你的帐户 ID 和身份验证令牌会分别显示在 [Twilio 帐户页][Twilio 帐户页]上标记为“帐户 SID”和“身份验证令牌”的字段中。

## <span id="VerifyPhoneNumbers"></span></a>验证电话号码

需要使用 Twilio 为你的帐户验证各个电话号码。例如，若要发起出站电话呼叫，必须使用 Twilio 验证电话号码是否为出站呼叫方 ID。同样，若要使电话号码接收 SMS 消息，则必须使用 Twilio 验证接收电话号码。有关如何验证电话号码的信息，请参阅[管理号码（可能为英文页面）][管理号码（可能为英文页面）]。下面的一些代码依赖于需要使用 Twilio 进行验证的电话号码。

作为对应用程序使用现有号码的替代方法，你可以购买 Twilio 电话号码。有关购买 Twilio 电话号码的信息，请参阅 [Twilio 电话号码帮助][Twilio 电话号码帮助]。

## <span id="create_app"></span></a>创建移动服务

托管启用 Twilio 的应用程序的移动服务与其他任何移动服务没有区别。只需添加 Twilio node.js 库，就能从移动服务自定义 API 脚本引用该库。有关创建初始移动服务的信息，请参阅[移动服务入门][移动服务入门]。

## <span id="VerifyPhoneNumbers"></span></a>配置移动服务以使用 Twilio Node.js 库

Twilio 提供了可包装 Twilio 各个方面的 Node.js 库，使你能够以简单且轻松的方式与 Twilio REST API 和 Twilio 客户端进行交互，从而生成 TwiML 响应。

若要在移动服务中使用 Twilio node.js 库，你必须利用移动服务 npm 模块支持（将脚本存储在源代码管理中即可）。教程[在源代码管理中存储脚本][在源代码管理中存储脚本]将指导你完成在移动服务中首次设置源代码管理，以及在 Git 存储库中存储服务器脚本的整个过程。

设置移动服务的源代码管理后，请在“移动服务”仪表板上打开“配置”选项卡，然后找到并复制 Git URL

将此 URL 粘贴到浏览器中，并将存储库名称替换为 */DebugConsole/index.html*

例如，将

    https://twilioSample.scm.azure-mobile.net/twilioSample.git

更改为

    https://twilioSample.scm.azure-mobile.net/DebugConsole/index.html

出现提示时，请输入你为服务设置源代码管理时使用的凭据。登录后，你将会看到 Azure 移动服务控制台。

![移动服务控制台][移动服务控制台]

在控制台中，将目录切换到 scripts 文件夹：

    cd site\wwwroot\App_Data\config\scripts

在 api 文件夹中，可以通过执行以下命令来安装 Twilio 节点模块：

    npm install twilio

现在，你可以在自定义 API 和表脚本中引用并使用 Twilio 库了。

## <span id="howto_make_call"></span></a>如何：发起传出呼叫

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

有关传入到 **client.makeCall** 函数中的参数的详细信息，请参阅 [][]<http://www.twilio.com/docs/api/rest/making-calls></a>。

如前所述，此代码使用 Twilio 提供的网站返回 TwiML 响应。可以改用你自己的站点来提供 TwiML 响应。有关更多信息，请参见[如何：从你自己的网站提供 TwiML 响应][如何：从你自己的网站提供 TwiML 响应]。

## <span id="howto_send_sms"></span></a>如何：发送 SMS 消息

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

## <span id="howto_provide_twiml_responses"></span></a>如何：从你自己的网站提供 TwiML 响应

当你的应用程序开始调用 Twilio API 时（例如通过 client.InitiateOutboundCall 方法），Twilio 会将你的请求发送到预期返回 TwiML 响应的 URL。“如何：发起传出呼叫”中的示例使用 Twilio 提供的 URL <http://twimlets.com/message> 返回该响应。

<div class="dev-callout">
<b>说明</b>
<p>虽然 TwiML 专供 Web 服务使用，但你可以在浏览器中查看 TwiML。例如，单击 <a href="http://twimlets.com/message">twimlet_message_url</a> 可查看空 &lt;Response&gt; 元素；又如，单击 <a href="http://twimlets.com/message?Message%5B0%5D=Hello%20World">twimlet_message_url_hello_world</a> 可查看包含 &lt;Say&gt; 元素的 &lt;Response&gt; 元素。</p>
</div>

你可以创建自己的返回 HTTP 响应的 URL 网站，而不用依赖 Twilio 提供的 URL。你可以使用任何语言创建返回 HTTP 响应的站点。本主题假设你将从 ASP.NET 泛型处理程序托管该 URL。

以下脚本将生成在呼叫中念出 Hello World 的 TwiML 响应。

    var twilio = require('twilio');

    exports.post = function(request, response) {
        var resp = new twilio.TwimlResponse();
        resp.say({voice:'woman'}, 'ahoy hoy! Testing Twilio and node.js');
        response.set('Content-Type', 'text/xml');
        response.send(200, resp.toString());
    };

有关 TwiML 的详细信息，请参阅 [][1]<https://www.twilio.com/docs/api/twiml></a>。

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

[WACOM.INCLUDE [twilio\_的更多服务和后续步骤](../includes/twilio_additional_services_and_next_steps.md)]

  [特别优惠]: http://ahoy.twilio.com/azure
  [Twilio 定价]: http://www.twilio.com/pricing
  [Twilio API 库]: https://www.twilio.com/docs/libraries
  [.NET]: /zh-cn/develop/net/how-to-guides/twilio-voice-and-sms-service/
  [node.js]: /zh-cn/develop/nodejs/how-to-guides/twilio-voice-and-sms-service/
  [Java]: /zh-cn/develop/java/how-to-guides/twilio-voice-and-sms-service/
  [PHP]: /zh-cn/develop/php/how-to-guides/twilio-voice-and-sms-service/
  [Python]: /zh-cn/develop/python/how-to-guides/twilio-voice-and-sms-service/
  [Ruby]: /zh-cn/develop/ruby/how-to-guides/twilio-voice-and-sms-service/
  [Twilio 标记语言文档]: http://www.twilio.com/docs/api/twiml
  [Twilio API]: http://www.twilio.com/api
  [试用 Twilio]: https://www.twilio.com/try-twilio
  [Twilio 帐户页]: https://www.twilio.com/user/account
  [管理号码（可能为英文页面）]: https://www.twilio.com/user/account/phone-numbers/verified#
  [Twilio 电话号码帮助]: https://www.twilio.com/help/faq/phone-numbers
  [移动服务入门]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/get-started/
  [在源代码管理中存储脚本]: http://www.windowsazure.com/zh-cn/develop/mobile/tutorials/store-scripts-in-source-control/
  [移动服务控制台]: ./media/partner-twilio-mobile-services-how-to-use-voice-sms/twilio-kuduconsole.png
  []: http://www.twilio.com/docs/api/rest/making-calls
  [如何：从你自己的网站提供 TwiML 响应]: #howto_provide_twiml_responses
  [twimlet\_message\_url]: http://twimlets.com/message
  [twimlet\_message\_url\_hello\_world]: http://twimlets.com/message?Message%5B0%5D=Hello%20World
  [1]: https://www.twilio.com/docs/api/twiml
  [twilio\_的更多服务和后续步骤]: ../includes/twilio_additional_services_and_next_steps.md
