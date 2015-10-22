<properties 
	pageTitle="如何使用 Twilio 实现语音和 SMS (Java) | Microsoft Azure" 
	description="了解如何在 Azure 中使用 Twilio API 服务发起电话呼叫和发送短信。用 Java 编写的代码示例。" 
	services="" 
	documentationCenter="java" 
	authors="devinrader" 
	manager="twilio" 
	editor="mollybos"/>

<tags 
	ms.service="multiple" 
	ms.date="11/25/2014" 
	wacn.date="10/3/2015"/>

# 如何通过 Java 使用 Twilio 实现语音和短信功能

本指南演示如何在 Azure 中使用 Twilio API 服务执行常见编程任务。所涉及的任务包括发起电话呼叫和发送短信服务 (SMS) 消息。有关 Twilio 以及在应用程序中使用语音和短信的详细信息，请参阅[后续步骤](#NextSteps)部分。

## <a id="WhatIs"></a>什么是 Twilio？
Twilio 是一种电话 Web 服务 API，它使你能够使用现有 Web 语言和技术生成语音和 SMS 应用程序。Twilio 属于第三方服务（而非 Azure 功能和 Microsoft 产品）。

利用 **Twilio 语音**，你的应用程序可以发起和接收电话呼叫。利用 **Twilio SMS**，您的应用程序可以发出和接收 SMS 消息。利用 **Twilio 客户端**，您的应用程序可以使用现有 Internet 连接（包括移动连接）启用语音通信。

## <a id="Pricing"></a>Twilio 定价和特别优惠
Twilio 定价中提供了有关 [Twilio 定价][twilio_pricing]的信息。Azure 客户可享受[特别优惠][special_offer]：免费获得 1000 条信息或 1000 分钟的入站。若要注册此优惠或了解更多信息，请访问 [http://ahoy.twilio.com/azure][special_offer]。

## <a id="Concepts"></a>概念
Twilio API 是一个为应用程序提供语音和 SMS 功能的 RESTful API。提供了多种语言版本的客户端库；有关列表，请参阅 [Twilio API 库][twilio_libraries]。

Twilio API 的关键方面是 Twilio 谓词和 Twilio 标记语言 (TwiML)。

### <a id="Verbs"></a>Twilio 谓词
API 利用了 Twilio 谓词；例如，**&lt;Say&gt;** 谓词指示 Twilio 在呼叫时传递语音消息。

下面是 Twilio 谓词的列表。

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

## <a id="create_app"></a>创建 Java 应用程序
1. 获取 Twilio JAR 并将其添加到 Java 生成路径和 WAR 部署程序集。在 [https://github.com/twilio/twilio-java][twilio_java] 上，您可以下载 GitHub 源并创建自己的 JAR，或者下载预建的 JAR（带有或不带依赖项）。
2. 确保 JDK 的 **cacerts** 密钥库包含带有 MD5 指纹 67:CB:9D:C0:13:24:8A:82:9B:B2:17:1E:D1:1B:EC:D4（序列号为 35:DE:F4:CF，SHA1 指纹为 D2:32:09:AD:23:D3:14:23:21:74:E4:0D:7F:9D:62:13:97:86:63:3A）的 Equifax 安全证书颁发机构证书。这是 [https://api.twilio.com][twilio_api_service] 服务的证书颁发机构 (CA) 证书，在使用 Twilio API 时调用。有关如何确保 JDK 的 **cacerts** 密钥库包含正确 CA 证书的信息，请参阅[将证书添加到 Java CA 证书存储][add_ca_cert]。

[如何在 Azure 上的 Java 应用程序中使用 Twilio 发起呼叫][howto_phonecall_java]中提供了有关使用适用于 Java 的 Twilio 客户端库的详细说明。

## <a id="configure_app"></a>将应用程序配置为使用 Twilio 库
在代码中，您可以在要在应用程序中使用的 Twilio 包或类的源文件的顶部添加 **import** 语句。

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
根据要使用的 Twilio 包或类，您的 **import** 语句可能有差别。

## <a id="howto_make_call"></a>如何：发起传出呼叫
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

有关传入到 **CallFactory.create** 方法中的参数的详细信息，请参阅 [http://www.twilio.com/docs/api/rest/making-calls][twilio_rest_making_calls]。

如前所述，此代码使用 Twilio 提供的网站返回 TwiML 响应。您可以改用自己的网站来提供 TwiML 响应；有关详细信息，请参阅[如何在 Azure 上的 Java 应用程序中提供 TwiML 响应](#howto_provide_twiml_responses)。

## <a id="howto_send_sms"></a>如何：发送 SMS 消息
以下代码演示了如何使用 **SmsFactory** 类发送 SMS 消息。“**呼叫方**”号码 **4155992671** 由 Twilio 提供，供试用帐户发送 SMS 消息。在运行代码前，必须为 Twilio 帐户验证“**被呼叫方**”号码。

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
    MessageFactory messageFactory = account.getMessageFactory();
    
    List<NameValuePair> params = new ArrayList<NameValuePair>();
    params.add(new BasicNameValuePair("To", "+14159352345")); // Replace with a valid phone number for your account.
    params.add(new BasicNameValuePair("From", "+14158141829")); // Replace with a valid phone number for your account.
    params.add(new BasicNameValuePair("Body", "Where's Wallace?"));
    
    Message sms = messageFactory.create(params);
        
有关传入到 **SmsFactory.create** 方法中的参数的详细信息，请参阅 [http://www.twilio.com/docs/api/rest/sending-sms][twilio_rest_sending_sms]。

## <a id="howto_provide_twiml_responses"></a>如何：从您自己的网站提供 TwiML 响应
当您的应用程序启动对 Twilio API 的调用时（例如通过 **CallFactory.create** 方法），Twilio 会将您的请求发送到应该返回 TwiML 响应的 URL。上面的示例使用 Twilio 提供的 URL [http://twimlets.com/message][twimlet_message_url]。（虽然 TwiML 专供 Web 服务使用，但你可以在浏览器中查看 TwiML。例如，单击 [http://twimlets.com/message][twimlet_message_url] 可查看空 **&lt;Response&gt;** 元素；又如，单击 [http://twimlets.com/message?Message%5B0%5D=Hello%20World][twimlet_message_url_hello_world] 可查看包含 **&lt;Say&gt;** 元素的 **&lt;Response&gt;** 元素。）

您可以创建自己的返回 HTTP 响应的 URL 网站，而不用依赖 Twilio 提供的 URL。你可以用任何语言创建返回 HTTP 响应的网站；本主题假定你将在 JSP 页面中托管 URL。

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

**ApiVersion** 参数以 Twilio 语音请求（非 SMS 请求）的形式提供。若要查看 Twilio 语音和 SMS 请求的可用请求参数，请分别参阅 <https://www.twilio.com/docs/api/twiml/twilio_request> 和 <https://www.twilio.com/docs/api/twiml/sms/twilio_request>。**RoleName** 环境变量作为 Azure 部署的一部分提供。（如果要添加自定义环境变量以便能够从 **System.getenv** 中选取它们，请参阅[其他角色配置设置][misc_role_config_settings]中的环境变量部分。）

将 JSP 页面设置为提供 TwiML 响应后，请使用 JSP 页面的 URL 作为传入到 **CallFactory.create** 方法中的 URL。例如，如果将名为 MyTwiML 的 Web 应用程序部署到 Azure 托管服务，则 JSP 页面的名称将为 mytwiml.jsp，并且可将 URL 传递到 **CallFactory.create**，如下所示：

    // Place the call From, To and URL values into a hash map. 
    HashMap<String, String> params = new HashMap<String, String>();
    params.put("From", "NNNNNNNNNN");
    params.put("To", "NNNNNNNNNN");
    params.put("Url", "http://<your_hosted_service>.chinacloudapp.cn/MyTwiML/mytwiml.jsp");

    CallFactory callFactory = account.getCallFactory();
    Call call = callFactory.create(params);

使用 TwiML 进行响应的另一个选项是通过 **TwiMLResponse** 类，可在 **com.twilio.sdk.verbs** 包中找到它。

有关将 Azure 中的 Twilio 与 Java 一起使用的更多信息，请参阅[如何在 Azure 上的 Java 应用程序中使用 Twilio 发起电话呼叫][howto_phonecall_java]。

## <a id="AdditionalServices"></a>如何：使用其他 Twilio 服务
除了此处所示的示例之外，Twilio 还提供了基于 Web 的 API，可通过这些 API 从 Azure 应用程序中使用其他 Twilio 功能。有关完整详细信息，请参阅 [Twilio API 文档][twilio_api_documentation]。

## <a id="NextSteps"></a>后续步骤
现在，您已了解 Twilio 服务的基础知识，单击下面的链接可以了解详细信息：

* [Twilio 安全准则][twilio_security_guidelines]
* [Twilio 操作方法和示例代码][twilio_howtos]
* [Twilio 快速入门教程][twilio_quickstarts] 
* [GitHub 上的 Twilio][twilio_on_github]
* [与 Twilio 技术支持人员交流][twilio_support]

[twilio_java]: https://github.com/twilio/twilio-java
[twilio_api_service]: https://api.twilio.com
[add_ca_cert]: /documentation/articles/java-add-certificate-ca-store
[howto_phonecall_java]: /documentation/articles/partner-twilio-java-phone-call-example
[misc_role_config_settings]: http://msdn.microsoft.com/library/windowsazure/hh690945.aspx
[twimlet_message_url]: http://twimlets.com/message
[twimlet_message_url_hello_world]: http://twimlets.com/message?Message%5B0%5D=Hello%20World
[twilio_rest_making_calls]: http://www.twilio.com/docs/api/rest/making-calls
[twilio_rest_sending_sms]: http://www.twilio.com/docs/api/rest/sending-sms
[twilio_pricing]: http://www.twilio.com/pricing
[special_offer]: http://ahoy.twilio.com/azure
[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api]: http://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_account]: https://www.twilio.com/user/account
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#
[twilio_api_documentation]: http://www.twilio.com/api
[twilio_security_guidelines]: http://www.twilio.com/docs/security
[twilio_howtos]: http://www.twilio.com/docs/howto
[twilio_on_github]: https://github.com/twilio
[twilio_support]: http://www.twilio.com/help/contact
[twilio_quickstarts]: http://www.twilio.com/docs/quickstart

<!---HONumber=71-->