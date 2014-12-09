<properties linkid="develop-java-how-to-twilio-sms-service" urlDisplayName="Twilio Voice/SMS Service" pageTitle="How to Use Twilio for Voice and SMS (Java) - Azure" metaKeywords="Twilio, Twilio API, phone calls, SMS message, TwiML responses, Azure Twilio Java" description="Learn how to make a phone call and send a SMS message with the Twilio API service on Azure. Code samples written in Java." metaCanonical="" services="" videoId="" scriptId="" documentationCenter="Java" title="How to Use Twilio for Voice and SMS Capabilities in Java" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" />

# 如何通过 Java 使用 Twilio 实现语音和短信功能

本指南演示如何在 Azure 中使用 Twilio API 服务执行常见编程任务。所涉及的任务包括发起电话呼叫和发送短信服务 (SMS) 消息。有关 Twilio 以及在应用程序中使用语音和 SMS 的详细信息，请参阅[后续步骤][后续步骤]部分。

## 目录

-   [什么是 Twilio？][什么是 Twilio？]
-   [Twilio 定价][Twilio 定价]
-   [概念][概念]
-   [创建 Twilio 帐户][创建 Twilio 帐户]
-   [验证电话号码][验证电话号码]
-   [创建 Java 应用程序][创建 Java 应用程序]
-   [将应用程序配置为使用 Twilio 库][将应用程序配置为使用 Twilio 库]
-   [如何：发起传出呼叫][如何：发起传出呼叫]
-   [如何：发送 SMS 消息][如何：发送 SMS 消息]
-   [如何：从你自己的网站提供 TwiML 响应][如何：从你自己的网站提供 TwiML 响应]
-   [如何：使用其他 Twilio 服务][如何：使用其他 Twilio 服务]
-   [后续步骤][后续步骤]

## <span id="WhatIs"></span></a>什么是 Twilio？

Twilio 是一种电话 Web 服务 API，它使你能够使用现有 Web 语言和技术生成语音和 SMS 应用程序。Twilio 属于第三方服务（而非 Azure 功能和 Microsoft 产品）。

利用 **Twilio 语音**，你的应用程序可以发起和接收电话呼叫。利用 **Twilio SMS**，你的应用程序可以发出和接收 SMS 消息。利用 **Twilio 客户端**，你的应用程序可以使用现有 Internet 连接（包括移动连接）启用语音通信。

## <span id="Pricing"></span></a>Twilio 定价和特别优惠

[Twilio 定价][1]中提供了有关 Twilio 定价的信息。Azure 客户可享受[特别优惠][特别优惠]：免费获得 1000 条信息和 1000 分钟拨入通话时间。若要注册此优惠或了解更多信息，请访问 [][特别优惠]<http://ahoy.twilio.com/azure></a>。

## <span id="Concepts"></span></a>概念

Twilio API 是一个为应用程序提供语音和 SMS 功能的 RESTful API。提供了多种语言版本的客户端库；有关列表，请参阅 [Twilio API 库][Twilio API 库]。

Twilio API 的关键方面是 Twilio 谓词和 Twilio 标记语言 (TwiML)。

### <span id="Verbs"></span></a>Twilio 谓词

API 利用了 Twilio 谓词；例如，**\<Say\>** 谓词指示 Twilio 在呼叫时传递语音消息。

下面是 Twilio 谓词的列表。

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

需要使用 Twilio 为你的帐户验证各个电话号码。例如，若要发起出站电话呼叫，必须使用 Twilio 验证电话号码是否为出站呼叫方 ID。同样，若要使电话号码接收 SMS 消息，则必须使用 Twilio 验证接收电话号码。有关如何验证电话号码的信息，请参阅[管理号码（可能为英文页面）][管理号码（可能为英文页面）]。下面的一些代码依赖于需要使用 Twilio 进行验证的电话号码。

作为对应用程序使用现有号码的替代方法，你可以购买 Twilio 电话号码。有关购买 Twilio 电话号码的信息，请参阅 [Twilio 电话号码帮助][Twilio 电话号码帮助]。

## <span id="create_app"></span></a>创建 Java 应用程序

1.  获取 Twilio JAR 并将其添加到 Java 生成路径和 WAR 部署程序集。在 [][]<https://github.com/twilio/twilio-java></a> 上，你可以下载 GitHub 源并创建自己的 JAR，或者下载预建的 JAR（带有或不带依赖项）。
2.  确保 JDK 的 **cacerts** 密钥库包含带有 MD5 指纹 67:CB:9D:C0:13:24:8A:82:9B:B2:17:1E:D1:1B:EC:D4（序列号为 35:DE:F4:CF，SHA1 指纹为 D2:32:09:AD:23:D3:14:23:21:74:E4:0D:7F:9D:62:13:97:86:63:3A）的 Equifax 安全证书颁发机构证书。这是 [][2]<https://api.twilio.com></a> 服务的证书颁发机构 (CA) 证书，在使用 Twilio API 时调用。有关如何确保 JDK 的 **cacerts** 密钥库包含正确 CA 证书的信息，请参阅[将证书添加到 Java CA 证书存储][将证书添加到 Java CA 证书存储]。

[如何在 Azure 上的 Java 应用程序中使用 Twilio 发起呼叫][如何在 Azure 上的 Java 应用程序中使用 Twilio 发起呼叫]中提供了有关使用适用于 Java 的 Twilio 客户端库的详细说明。

## <span id="configure_app"></span></a>将应用程序配置为使用 Twilio 库

在代码中，你可以在要在应用程序中使用的 Twilio 包或类的源文件的顶部添加 **import** 语句。

对于 Java 源文件：

    import com.twilio.*;
    import com.twilio.sdk.*;
    import com.twilio.sdk.resource.factory.*;
    import com.twilio.sdk.resource.instance.*;

对于 Java Server Page (JSP) 源文件：

    import="com.twilio.*"
    import="com.twilio.sdk.*"
    import="com.twilio.sdk.resource.factory.*"
    import="com.twilio.sdk.resource.instance.*"

根据要使用的 Twilio 包或类，你的 **import** 语句可能有差别。

## <span id="howto_make_call"></span></a>如何：发起传出呼叫

以下代码演示了如何使用 **CallFactory** 类发起传出呼叫。此代码还使用 Twilio 提供的网站返回 Twilio 标记语言 (TwiML) 响应。用你的值替换“呼叫方”和“被呼叫方”电话号码，并确保在运行代码之前验证 Twilio 帐户的“呼叫方”电话号码。

    // Use your account SID and authentication token instead
    // of the placeholders shown here.
    String accountSID = "your_twilio_account";
    String authToken = "your_twilio_authentication_token";

    // Create an instance of the Twilio client.
    TwilioRestClient client;
    client = new TwilioRestClient(accountSID, authToken);

    // Retrieve the account, used later to create an instance of the CallFactory.
    Account account = client.getAccount();

    // Use the Twilio-provided site for the TwiML response.
    String Url="http://twimlets.com/message";
    Url = Url + "?Message%5B0%5D=Hello%20World";

    // Place the call From, To and URL values into a hash map. 
    HashMap<String, String> params = new HashMap<String, String>();
    params.put("From", "NNNNNNNNNN"); // Use your own value for the second parameter.
    params.put("To", "NNNNNNNNNN");   // Use your own value for the second parameter.
    params.put("Url", Url);

    // Create an instance of the CallFactory class.
    CallFactory callFactory = account.getCallFactory();

    // Make the call.
    Call call = callFactory.create(params);

有关传入到 **CallFactory.create** 方法中的参数的详细信息，请参阅 [][3]<http://www.twilio.com/docs/api/rest/making-calls></a>。

如前所述，此代码使用 Twilio 提供的网站返回 TwiML 响应。你可以改用自己的网站来提供 TwiML 响应；有关详细信息，请参阅[如何在 Azure 上的 Java 应用程序中提供 TwiML 响应][如何：从你自己的网站提供 TwiML 响应]。

## <span id="howto_send_sms"></span></a>如何：发送 SMS 消息

以下代码演示了如何使用 **SmsFactory** 类发送 SMS 消息。“呼叫方”号码 **4155992671** 由 Twilio 提供，供试用帐户用来发送 SMS 消息。在运行代码前，必须为 Twilio 帐户验证“被呼叫方”号码。

    // Use your account SID and authentication token instead
    // of the placeholders shown here.
    String accountSID = "your_twilio_account";
    String authToken = "your_twilio_authentication_token";

    // Create an instance of the Twilio client.
    TwilioRestClient client;
    client = new TwilioRestClient(accountSID, authToken);

    // Retrieve the account, used later to create an instance of the SmsFactory.
    Account account = client.getAccount();

    // Send an SMS message.
    // Place the call From, To and Body values into a hash map. 
    HashMap<String, String> smsParams = new HashMap<String, String>();
    smsParams.put("From", "4155992671"); // The second parameter is a phone number provided
                                         // by Twilio for trial accounts.
    smsParams.put("To", "NNNNNNNNNN");   // Use your own value for the second parameter.
    smsParams.put("Body", "This is my SMS message.");

    // Create an instance of the SmsFactory class.
    SmsFactory smsFactory = account.getSmsFactory();

    // Send the message.
    Sms sms = smsFactory.create(smsParams);

有关传入到 **SmsFactory.create** 方法中的参数的详细信息，请参阅 [][4]<http://www.twilio.com/docs/api/rest/sending-sms></a>。

## <span id="howto_provide_twiml_responses"></span></a>如何：从你自己的网站提供 TwiML 响应

当你的应用程序启动对 Twilio API 的调用时（例如通过 **CallFactory.create** 方法），Twilio 会将你的请求发送到应该返回 TwiML 响应的 URL。上面的示例使用 Twilio 提供的 URL [][5]<http://twimlets.com/message></a>。（虽然 TwiML 专供 Web 服务使用，但你可以在浏览器中查看 TwiML。例如，单击 [][5]<http://twimlets.com/message></a> 可查看空 **\<Response\>** 元素；又如，单击 [][6]<http://twimlets.com/message?Message%5B0%5D=Hello%20World></a> 可查看包含 **\<Say\>** 元素的 **\<Response\>** 元素。）

你可以创建自己的返回 HTTP 响应的 URL 网站，而不用依赖 Twilio 提供的 URL。你可以用任何语言创建返回 HTTP 响应的网站；本主题假定你将在 JSP 页面中托管 URL。

以下 JSP 页面将生成在呼叫中念出 **Hello World** 的 TwiML 响应。

    <%@ page contentType="text/xml" %>
    <Response> 
        <Say>Hello World</Say>
    </Response>

以下 JSP 页面将生成一个 TwiML 响应，念出一些文字，中间停顿几次，然后念出有关 Twilio API 版本和 Azure 角色名称的信息。

    <%@ page contentType="text/xml" %>
    <Response> 
        <Say>Hello from Azure</Say>
        <Pause></Pause>
        <Say>The Twilio API version is <%= request.getParameter("ApiVersion") %>.</Say>
        <Say>The Azure role name is <%= System.getenv("RoleName") %>.</Say>
        <Pause></Pause>
        <Say>Good bye.</Say>
    </Response>

**ApiVersion** 参数以 Twilio 语音请求（非 SMS 请求）的形式提供。若要查看 Twilio 语音和 SMS 请求的可用请求参数，请分别参阅 <https://www.twilio.com/docs/api/twiml/twilio_request> 和 <https://www.twilio.com/docs/api/twiml/sms/twilio_request>。**RoleName** 环境变量作为 Azure 部署的一部分提供。（如果要添加自定义环境变量以便能够从 **System.getenv** 中选取它们，请参阅[其他角色配置设置][其他角色配置设置]中的环境变量部分。）

将 JSP 页面设置为提供 TwiML 响应后，请使用 JSP 页面的 URL 作为传入到 **CallFactory.create** 方法中的 URL。例如，如果将名为 MyTwiML 的 Web 应用程序部署到 Azure 托管服务，则 JSP 页面的名称将为 mytwiml.jsp，并且可将 URL 传递到 **CallFactory.create**，如下所示：

    // Place the call From, To and URL values into a hash map. 
    HashMap<String, String> params = new HashMap<String, String>();
    params.put("From", "NNNNNNNNNN");
    params.put("To", "NNNNNNNNNN");
    params.put("Url", "http://<your_hosted_service>.cloudapp.net/MyTwiML/mytwiml.jsp");

    CallFactory callFactory = account.getCallFactory();
    Call call = callFactory.create(params);

使用 TwiML 进行响应的另一个选项是通过 **TwiMLResponse** 类，可在 **com.twilio.sdk.verbs** 包中找到它。

有关将 Azure 中的 Twilio 与 Java 一起使用的更多信息，请参阅[如何在 Azure 上的 Java 应用程序中使用 Twilio 发起呼叫][如何在 Azure 上的 Java 应用程序中使用 Twilio 发起呼叫]。

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
  [创建 Java 应用程序]: #create_app
  [将应用程序配置为使用 Twilio 库]: #configure_app
  [如何：发起传出呼叫]: #howto_make_call
  [如何：发送 SMS 消息]: #howto_send_sms
  [如何：从你自己的网站提供 TwiML 响应]: #howto_provide_twiml_responses
  [如何：使用其他 Twilio 服务]: #AdditionalServices
  [1]: http://www.twilio.com/pricing
  [特别优惠]: http://ahoy.twilio.com/azure
  [Twilio API 库]: https://www.twilio.com/docs/libraries
  [TwiML]: http://www.twilio.com/docs/api/twiml
  [Twilio API]: http://www.twilio.com/api
  [试用 Twilio]: https://www.twilio.com/try-twilio
  [Twilio 帐户页]: https://www.twilio.com/user/account
  [管理号码（可能为英文页面）]: https://www.twilio.com/user/account/phone-numbers/verified#
  [Twilio 电话号码帮助]: https://www.twilio.com/help/faq/phone-numbers
  []: https://github.com/twilio/twilio-java
  [2]: https://api.twilio.com
  [将证书添加到 Java CA 证书存储]: ../java-add-certificate-ca-store
  [如何在 Azure 上的 Java 应用程序中使用 Twilio 发起呼叫]: ../partner-twilio-java-phone-call-example
  [3]: http://www.twilio.com/docs/api/rest/making-calls
  [4]: http://www.twilio.com/docs/api/rest/sending-sms
  [5]: http://twimlets.com/message
  [6]: http://twimlets.com/message?Message%5B0%5D=Hello%20World
  [其他角色配置设置]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh690945.aspx
  [Twilio 安全准则]: http://www.twilio.com/docs/security
  [Twilio 操作方法和示例代码]: http://www.twilio.com/docs/howto
  [Twilio 快速入门教程]: http://www.twilio.com/docs/quickstart
  [GitHub 上的 Twilio]: https://github.com/twilio
  [与 Twilio 技术支持人员交流]: http://www.twilio.com/help/contact
