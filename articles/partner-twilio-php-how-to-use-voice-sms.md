<properties title="How to Use Twilio for Voice and SMS (PHP) - Azure" pageTitle="How to Use Twilio for Voice and SMS (PHP) - Azure" metaKeywords="Azure PHP Twilio, Azure phone calls, Azure phone calls, Azure twilio, Azure SMS, Azure SMS, Azure voice calls, azure voice calls, Azure text messages, Azure text messages" description="Learn how to make a phone call and send a SMS message with the Twilio API service on Azure. Code samples written in PHP." documentationCenter="PHP" services="" authors="robmcm" manager="wpickett" editor="mollybos" videoId="" scriptId="" />

# 如何通过 PHP 使用 Twilio 实现语音和 SMS 功能

本指南演示如何在 Azure 中使用 Twilio API 服务执行常见编程任务。所涉及的任务包括发起电话呼叫和发送短信服务 (SMS) 消息。有关 Twilio 以及在应用程序中使用语音和 SMS 的详细信息，请参阅[后续步骤][后续步骤]部分。

## 目录

-   [什么是 Twilio？][什么是 Twilio？]
-   [Twilio 定价][Twilio 定价]
-   [概念][概念]
-   [创建 Twilio 帐户][创建 Twilio 帐户]
-   [验证电话号码][验证电话号码]
-   [创建 PHP 应用程序][创建 PHP 应用程序]
-   [将应用程序配置为使用 Twilio 库][将应用程序配置为使用 Twilio 库]
-   [如何：发起传出呼叫][如何：发起传出呼叫]
-   [如何：发送 SMS 消息][如何：发送 SMS 消息]
-   [如何：从你自己的网站提供 TwiML 响应][如何：从你自己的网站提供 TwiML 响应]

## <span id="WhatIs"></span></a>什么是 Twilio？

Twilio 为将来的商业沟通提供强大支持，并使开发人员能够将语音、VoIP 和消息传送嵌入到应用程序中。它们对基于云的全球环境中所需的所有基础结构进行虚拟化，并通过 Twilio 通信 API 平台将其公开。可轻松构建和扩展应用程序。享受现用现付定价所带来的灵活性，并从云可靠性中受益。

利用 **Twilio 语音**，你的应用程序可以发起和接收电话呼叫。**Twilio SMS** 使你的应用程序能够发送和接收文本消息。利用 **Twilio 客户端**，你可以从任何手机、平板电脑或浏览器发起 VoIP 呼叫并支持 WebRTC。

## <span id="Pricing"></span></a>Twilio 定价和特别优惠

Azure 客户可享受 [特别优惠][<http://www.twilio.com/azure>]：升级 Twilio 帐户即可免费获得 10 美元的 Twilio 信用。此 Twilio 信用可应用于任何 Twilio 使用（10 美元信用等价于发送多达 1,000 条 SMS 消息或接收长达 1000 分钟的入站语音，具体取决电话号码和消息或呼叫目标的位置）。兑换此 Twilio 信用并从以下网址开始：[][]<http://ahoy.twilio.com/azure></a>。

Twilio 是一种现用现付服务。没有设置费用，并且你可以随时关闭你的帐户。你可以在 [Twilio 定价][1]上找到更多详细信息。

## <span id="Concepts"></span></a>概念

Twilio API 是一个为应用程序提供语音和 SMS 功能的 RESTful API。提供了多种语言版本的客户端库；有关列表，请参阅 [Twilio API 库][Twilio API 库]。

Twilio API 的关键方面是 Twilio 谓词和 Twilio 标记语言 (TwiML)。

### <span id="Verbs"></span></a>Twilio 谓词

API 利用了 Twilio 谓词；例如，**\<Say\>** 谓词指示 Twilio 在呼叫时传递语音消息。

下面是 Twilio 谓词的列表。请通过 [Twilio 标记语言文档][<http://www.twilio.com/docs/api/twiml>] 了解其他谓词和功能。

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

有关 Twilio 谓词、其属性和 TwiML 的详细信息，请参阅 [TwiML][TwiML]。有关 Twilio API 的其他信息，请参阅 [Twilio API][Twilio API]。

## <span id="CreateAccount"></span></a>创建 Twilio 帐户

准备好获取 Twilio 帐户后，请在[试用 Twilio][试用 Twilio] 上注册。可以先使用免费帐户，以后再升级你的帐户。

注册 Twilio 帐户时，你将收到帐户 ID 和身份验证令牌。需要二者才能发起 Twilio API 呼叫。为了防止对你的帐户进行未经授权的访问，请保护身份验证令牌。你的帐户 ID 和身份验证令牌会分别显示在 [Twilio 帐户页][Twilio 帐户页]上标记为“帐户 SID”和“身份验证令牌”的字段中。

## <span id="VerifyPhoneNumbers"></span></a>验证电话号码

需要使用 Twilio 为你的帐户验证各个电话号码。例如，若要使用你的针对呼叫方 ID 的现有的电话号码发起出站电话呼叫，必须使用 Twilio 验证该电话号码。同样，在升级前若要将 SMS 消息发送到某一电话号码，必须使用 Twilio 验证该电话号码。升级后，你可以将 SMS 消息发送给任何号码而不必对其进行验证。有关如何验证电话号码的信息，请参阅[管理号码（可能为英文页面）][管理号码（可能为英文页面）]。下面的一些代码依赖于需要使用 Twilio 进行验证的电话号码。

作为对应用程序使用现有号码的替代方法，你可以购买 Twilio 电话号码。有关购买 Twilio 电话号码的信息，请参阅 [Twilio 电话号码帮助][Twilio 电话号码帮助]。

## <span id="create_app"></span></a>创建 PHP 应用程序

使用 Twilio 服务且在 Azure 中运行的 PHP 应用程序与任何其他使用 Twilio 服务的 PHP 应用程序之间没有任何差别。Twilio 服务是基于 REST 的且可通过几种方法从 PHP 中调用，本文将重点介绍如何将 Twilio 服务与 [Github 提供的用于 PHP 的 Twilio 库][Github 提供的用于 PHP 的 Twilio 库]一起使用。有关使用用于 PHP 的 Twilio 库的详细信息，请参阅 [][2]<http://readthedocs.org/docs/twilio-php/en/latest/index.html></a>。

有关如何生成 Twilio/PHP 应用程序并将其部署到 Azure 的详细说明，请参阅[如何在 Azure 中通过 PHP 应用程序使用 Twilio 发起电话呼叫][如何在 Azure 中通过 PHP 应用程序使用 Twilio 发起电话呼叫]。

## <span id="configure_app"></span></a>将应用程序配置为使用 Twilio 库

可以通过两种方式将应用程序配置为使用用于 PHP 的 Twilio 库：

1.  下载 Github 提供的用于 PHP 的 Twilio 库 ([][Github 提供的用于 PHP 的 Twilio 库]<https://github.com/twilio/twilio-php></a>) 并将 **Services** 目录添加到应用程序。

   -或-

2.  将用于 PHP 的 Twilio 库作为 PEAR 包安装。可使用以下命令安装它：

        $ pear channel-discover twilio.github.com/pear
        $ pear install twilio/Services_Twilio

安装用于 PHP 的 Twilio 库后，你可以在 PHP 文件的顶部添加 **require\_once** 语句来引用该库：

        require_once 'Services/Twilio.php';

有关详细信息，请参阅 [][3]<https://github.com/twilio/twilio-php/blob/master/README.md></a>。

## <span id="howto_make_call"></span></a>如何：发起传出呼叫

下面演示了如何使用 **Services\_Twilio** 类发起传出呼叫。此代码还使用 Twilio 提供的网站返回 Twilio 标记语言 (TwiML) 响应。用你的值替换“呼叫方”和“被呼叫方”电话号码，并确保在运行代码之前验证 Twilio 帐户的“呼叫方”电话号码。

    // Include the Twilio PHP library.
    require_once 'Services/Twilio.php';

    // Library version.
    $version = "2010-04-01";

    // Set your account ID and authentication token.
    $sid = "your_twilio_account_sid";
    $token = "your_twilio_authentication_token";

    // The number of the phone initiating the the call.
    // (Must be previously validated with Twilio.)
    $from_number = "NNNNNNNNNNN";

    // The number of the phone receiving call.
    $to_number = "NNNNNNNNNNN";

    // Use the Twilio-provided site for the TwiML response.
    $url = "http://twimlets.com/message";

    // The phone message text.
    $message = "Hello world.";

    // Create the call client.
    $client = new Services_Twilio($sid, $token, $version);

    //Make the call.
    try
    {
        $call = $client->account->calls->create(
            $from_number, 
            $to_number,
            $url.'?Message='.urlencode($message)
        );
    }
    catch (Exception $e) 
    {
        echo 'Error: ' . $e->getMessage();
    }

如前所述，此代码使用 Twilio 提供的网站返回 TwiML 响应。可以改用你自己的网站来提供 TwiML 响应；有关详细信息，请参阅[如何从你自己的网站提供 TwiML 响应][如何：从你自己的网站提供 TwiML 响应]。

-   **注意**：若要纠正 SSL 证书验证错误，请参阅 [][4]<http://readthedocs.org/docs/twilio-php/en/latest/usage/rest.html></a>

## <span id="howto_send_sms"></span></a>如何：发送 SMS 消息

下面演示如何使用 **Services\_Twilio** 类发送 SMS 消息。“呼叫方”号码由 Twilio 提供，供试用帐户用来发送 SMS 消息。在运行代码前，必须为 Twilio 帐户验证“被呼叫方”号码。

    // Include the Twilio PHP library.
    require_once 'Services/Twilio.php';

    // Library version.
    $version = "2010-04-01";

    // Set your account ID and authentication token.
    $sid = "your_twilio_account_sid";
    $token = "your_twilio_authentication_token";


    $from_number = "NNNNNNNNNNN"; // With trial account, texts can only be sent from your Twilio number.
    $to_number = "NNNNNNNNNNN";
    $message = "Hello world.";

    // Create the call client.
    $client = new Services_Twilio($sid, $token, $version);

    // Send the SMS message.
    try
    {
        $client->account->sms_messages->create($from_number, $to_number, $message);
    }
    catch (Exception $e) 
    {
        echo 'Error: ' . $e->getMessage();
    }

## <span id="howto_provide_twiml_responses"></span></a>如何：从你自己的网站提供 TwiML 响应

当你的应用程序发起对 Twilio API 的调用时，Twilio 会将你的请求发送到应返回 TwiML 响应的 URL。上面的示例使用 Twilio 提供的 URL [][5]<http://twimlets.com/message></a>。（虽然 TwiML 专供 Twilio 使用，但你可以在浏览器中查看它。例如，单击 [][5]<http://twimlets.com/message></a> 会看到一个空的`<Response>` 元素，又如，单击 [][6]<http://twimlets.com/message?Message%5B0%5D=Hello%20World></a> 会看到一个`<Response>` 元素，其中包含 `<Say>` 元素。）

你可以创建自己的返回 HTTP 响应的网站，而不用依赖 Twilio 提供的 URL。你可以使用任何语言创建返回 XML 响应的网站；本主题假定你将使用 PHP 创建 TwiML。

以下 PHP 页面将生成在呼叫时念出 **Hello World** 的 TwiML 响应。

    <?php    
        header("content-type: text/xml");    
        echo "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n";
    ?>
    <Response>    
        <Say>Hello world.</Say>
    </Response>

如上面的示例中所示，TwiML 响应只是一个 XML 文档。用于 PHP 的 Twilio 库包含将为你生成 TwiML 的类。以下示例将生成与上面所示相同的响应，但该响应会使用用于 PHP 的 Twilio 库中的 **Services\_Twilio\_Twiml** 类：

    require_once('Services/Twilio.php');

    $response = new Services_Twilio_Twiml();
    $response->say("Hello world.");
    print $response;

有关 TwiML 的详细信息，请参阅 [][7]<https://www.twilio.com/docs/api/twiml></a>。

将 PHP 页面设置为提供 TwiML 响应后，请使用 PHP 页面的 URL 作为传入到 `Services_Twilio->account->calls->create` 方法中的 URL。例如，如果已将名为 **MyTwiML** 的 Web 应用程序部署到 Azure 托管服务，且 PHP 页面的名称将为 **mytwiml.php**，则可将 URL 传递到 **Services\_Twilio-\>account-\>calls-\>create**，如以下示例所示：

    require_once 'Services/Twilio.php';

    $sid = "your_twilio_account_sid";
    $token = "your_twilio_authentication_token";
    $from_number = "NNNNNNNNNNN";
    $to_number = "NNNNNNNNNNN";
    $url = "http://<your_hosted_service>.cloudapp.net/MyTwiML/mytwiml.php";

    // The phone message text.
    $message = "Hello world.";

    $client = new Services_Twilio($sid, $token, "2010-04-01");

    try
    {
        $call = $client->account->calls->create(
            $from_number, 
            $to_number,
            $url.'?Message='.urlencode($message)
        );
    }
    catch (Exception $e) 
    {
        echo 'Error: ' . $e->getMessage();
    }

有关将 Azure 中的 Twilio 与 PHP 一起使用的更多信息，请参阅[如何在 Azure 上的 PHP 应用程序中使用 Twilio 发起呼叫][如何在 Azure 中通过 PHP 应用程序使用 Twilio 发起电话呼叫]。

## <span id="AdditionalServices"></span></a>如何：使用其他 Twilio 服务

除了此处所示的示例之外，Twilio 还提供了基于 Web 的 API，可通过这些 API 从 Azure 应用程序中使用其他 Twilio 功能。有关完整详细信息，请参阅 [Twilio API 文档][Twilio API]。

## <span id="NextSteps"></span></a> 后续步骤

现在，你已了解 Twilio 服务的基础知识，单击下面的链接可以了解更多信息：

-   [Twilio 安全准则][Twilio 安全准则]
-   [Twilio 操作方法和示例代码][Twilio 操作方法和示例代码]
-   [Twilio 快速入门教程][Twilio 快速入门教程]
-   [GitHub 上的 Twilio][GitHub 上的 Twilio]
-   [与 Twilio 技术支持人员交流][与 Twilio 技术支持人员交流]

  [后续步骤]: #NextSteps
  [什么是 Twilio？]: #WhatIs
  [Twilio 定价]: #Pricing
  [概念]: #Concepts
  [创建 Twilio 帐户]: #CreateAccount
  [验证电话号码]: #VerifyPhoneNumbers
  [创建 PHP 应用程序]: #create_app
  [将应用程序配置为使用 Twilio 库]: #configure_app
  [如何：发起传出呼叫]: #howto_make_call
  [如何：发送 SMS 消息]: #howto_send_sms
  [如何：从你自己的网站提供 TwiML 响应]: #howto_provide_twiml_responses
  []: http://ahoy.twilio.com/azure
  [1]: http://www.twilio.com/pricing
  [Twilio API 库]: https://www.twilio.com/docs/libraries
  [TwiML]: http://www.twilio.com/docs/api/twiml
  [Twilio API]: http://www.twilio.com/api
  [试用 Twilio]: https://www.twilio.com/try-twilio
  [Twilio 帐户页]: https://www.twilio.com/user/account
  [管理号码（可能为英文页面）]: https://www.twilio.com/user/account/phone-numbers/verified#
  [Twilio 电话号码帮助]: https://www.twilio.com/help/faq/phone-numbers
  [Github 提供的用于 PHP 的 Twilio 库]: https://github.com/twilio/twilio-php
  [2]: http://readthedocs.org/docs/twilio-php/en/latest/index.html
  [如何在 Azure 中通过 PHP 应用程序使用 Twilio 发起电话呼叫]: ../partner-twilio-php-make-phone-call
  [3]: https://github.com/twilio/twilio-php/blob/master/README.md
  [4]: http://readthedocs.org/docs/twilio-php/en/latest/usage/rest.html
  [5]: http://twimlets.com/message
  [6]: http://twimlets.com/message?Message%5B0%5D=Hello%20World
  [7]: https://www.twilio.com/docs/api/twiml
  [Twilio 安全准则]: http://www.twilio.com/docs/security
  [Twilio 操作方法和示例代码]: http://www.twilio.com/docs/howto
  [Twilio 快速入门教程]: http://www.twilio.com/docs/quickstart
  [GitHub 上的 Twilio]: https://github.com/twilio
  [与 Twilio 技术支持人员交流]: http://www.twilio.com/help/contact
