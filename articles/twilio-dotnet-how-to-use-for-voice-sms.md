<properties 
	pageTitle="如何使用 Twilio 实现语音和短信功能 (.NET) | Microsoft Azure" 
	description="了解如何在 Azure 中使用 Twilio API 服务发起电话呼叫和发送短信。采用 .NET 编写的代码示例。" 
	services="" 
	documentationCenter=".net" 
	authors="devinrader" 
	manager="twilio" 
	editor=""/>

<tags 
	ms.service="multiple" 
	ms.date="04/24/2015" 
	wacn.date="10/3/2015"/>





# 如何在 Azure 中使用 Twilio 实现语音和短信功能

本指南演示如何在 Azure 中使用 Twilio API 服务执行常见编程任务。所涉及的任务包括发起电话呼叫和发送短信服务 (SMS) 消息。有关 Twilio 以及在应用程序中使用语音和短信的详细信息，请参阅[后续步骤](#NextSteps)部分。

## <a id="WhatIs"></a>什么是 Twilio？
Twilio 为将来的商业沟通提供强大支持，并使开发人员能够将语音、VoIP 和消息传送嵌入到应用程序中。它们对基于云的全球环境中所需的所有基础结构进行虚拟化，并通过 Twilio 通信 API 平台将其公开。可轻松构建和扩展应用程序。享受现用现付定价所带来的灵活性，并从云可靠性中受益。

利用 **Twilio 语音**，你的应用程序可以发起和接收电话呼叫。**Twilio SMS** 使你的应用程序能够发送和接收 SMS 消息。利用 **Twilio 客户端**，你可以从任何手机、平板电脑或浏览器发起 VoIP 呼叫并支持 WebRTC。

## <a id="Pricing"></a>Twilio 定价和特别优惠
Azure 客户在升级 Twilio 帐户后即可获得[特别优惠](http://www.twilio.com/azure)：10 美元的 Twilio 信用额度。此 Twilio 信用可应用于任何 Twilio 使用（10 美元信用等价于发送多达 1,000 条 SMS 消息或接收长达 1000 分钟的入站语音，具体取决电话号码和消息或呼叫目标的位置）。兑换此 Twilio 信用额度并从 [ahoy.twilio.com/azure](http://ahoy.twilio.com/azure) 开始。

Twilio 是一种现用现付服务。没有设置费用，并且您可以随时关闭您的帐户。你可以在 [Twilio 定价](http://www.twilio.com/voice/pricing)上找到更多详细信息。

## <a id="Concepts"></a>概念
Twilio API 是一个为应用程序提供语音和 SMS 功能的 RESTful API。提供了多种语言版本的客户端库；有关列表，请参阅 [Twilio API 库][twilio_libraries]。

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

## <a id="create_app"></a>创建 Azure 应用程序
托管启用 Twilio 的应用程序的 Azure 应用程序与其他任何 Azure 应用程序没有区别。您需添加 Twilio .NET 库并将角色配置为使用 Twilio .NET 库。有关创建初始 Azure 项目的信息，请参阅[使用 Visual Studio 创建 Azure 项目][vs_project]。

## <a id="configure_app"></a>将应用程序配置为使用 Twilio 库
Twilio 提供了一系列可包装 Twilio 各个方面的 .NET 帮助程序库，使您能够以简单且轻松地方式与 Twilio REST API 和 Twilio 客户端进行交互，从而生成 TwiML 响应。

Twilio 为 .NET 开发人员提供了五个库：库|说明 ---|--- Twilio.API|可在友好的 .NET 库中包装 Twilio REST API 的核心 Twilio 库。此库可用于 .NET、Silverlight 和 Windows Phone 7。Twilio.TwiML|可提供一种 .NET 友好方式来生成 TwiML 标记。Twilio.MVC|对于使用 ASP.NET MVC 的开发人员，此库包括 TwilioController、TwiML ActionResult 和请求验证特性。Twilio.WebMatrix|对于使用 Microsoft 免费 WebMatrix 开发工具的开发人员，此库包含适用于各种 Twilio 操作的 Razor 语法帮助程序。Twilio.Client.Capability|包含可用于 Twilio 客户端 JavaScript SDK 的 Capability 令牌生成器。

请注意，所有库都需要 .NET 3.5、Silverlight 4 或者 Windows Phone 7 或更高版本。

本指南中提供的示例使用 Twilio.API 库。

这些库可以[使用 NuGet 程序包管理器扩展进行安装](http://www.twilio.com/docs/csharp/install)，该扩展适用于 Visual Studio 2010 和 2012。源代码托管在 [GitHub][twilio_github_repo] 上，其中的 Wiki 包含有关使用这些库的完整文档。

默认情况下，Microsoft Visual Studio 2010 安装 1.2 版的 NuGet。安装 Twilio 库需要 1.6 或更高版本的 NuGet。有关安装或更新 NuGet 的信息，请参阅 [http://nuget.org/][nuget]。

> [AZURE.NOTE]若要安装 NuGet 的最新版本，必须首先使用 Visual Studio Extension Manager 卸载已加载的版本。为此，您必须以管理员的身份运行 Visual Studio。否则，“卸载”按钮将处于禁用状态。

### <a id="use_nuget"></a>向您的 Visual Studio 项目添加 Twilio 库：

1.  在 Visual Studio 中打开您的解决方案。
2.  右键单击“引用”。
3.  单击“管理 NuGet 程序包...”。
4.  单击“联机”。
5.  在联机搜索框中，键入 *twilio*。
6.  单击 Twilio 程序包对应的“安装”。


## <a id="howto_make_call"></a>如何：发起传出呼叫
下面演示了如何使用 **TwilioRestClient** 类发起传出呼叫。此代码还使用 Twilio 提供的网站返回 Twilio 标记语言 (TwiML) 响应。用你的值替换“呼叫方”和“被呼叫方”电话号码，并确保在运行代码之前验证 Twilio 帐户的“呼叫方”电话号码。

    // Use your account SID and authentication token instead
    // of the placeholders shown here.
    string accountSID = "your_twilio_account";
    string authToken = "your_twilio_authentication_token";

    // Create an instance of the Twilio client.
    TwilioRestClient client;
    client = new TwilioRestClient(accountSID, authToken);

    // Use the Twilio-provided site for the TwiML response.
    String Url="http://twimlets.com/message";
    Url = Url + "?Message%5B0%5D=Hello%20World";

    // Instantiate the call options that are passed
    // to the outbound call
    CallOptions options = new CallOptions();

    // Set the call From, To, and URL values to use for the call.
    // This sample uses the sandbox number provided by
    // Twilio to make the call.
    options.From = "+NNNNNNNNNN";
    options.To = "NNNNNNNNNN";
    options.Url = Url;

    // Make the call.
    var call = client.InitiateOutboundCall(options);

有关传入到 **client.InitiateOutboundCall** 方法中的参数的详细信息，请参阅 [http://www.twilio.com/docs/api/rest/making-calls][twilio_rest_making_calls]。

如前所述，此代码使用 Twilio 提供的网站返回 TwiML 响应。可以改用您自己的站点来提供 TwiML 响应。有关详细信息，请参阅[如何：从你自己的网站提供 TwiML 响应](#howto_provide_twiml_responses)。

## <a id="howto_send_sms"></a>如何：发送 SMS 消息
以下屏幕截图演示了如何使用 **TwilioRestClient** 类发送短信。“呼叫方”号码由 Twilio 提供，供试用帐户用来发送 SMS 消息。在运行代码前，必须验证你的 Twilio 帐户的“被呼叫方”号码。

        // Use your account SID and authentication token instead
        // of the placeholders shown here.
        string accountSID = "your_twilio_account";
        string authToken = "your_twilio_authentication_token";

        // Create an instance of the Twilio client.
        TwilioRestClient client;
        client = new TwilioRestClient(accountSID, authToken);

        // Send an SMS message.
        Message result = client.SendMessage(
            "+14155992671", "+12069419717", "This is my SMS message.");

        if (result.RestException != null)
        {
            //an exception occurred making the REST call
            string message = result.RestException.Message;
        }

## <a id="howto_provide_twiml_responses"></a>如何：从您自己的网站提供 TwiML 响应
当您的应用程序开始调用 Twilio API 时（例如通过 **client.InitiateOutboundCall** 方法），Twilio 会将您的请求发送到预期返回 TwiML 响应的 URL。[如何：发起传出呼叫](#howto_make_call)中的示例使用 Twilio 提供的 URL [http://twimlets.com/message][twimlet_message_url] 返回该响应。

> [AZURE.NOTE]虽然 TwiML 专供 Web 服务使用，但您可以在浏览器中查看 TwiML。例如，单击 [http://twimlets.com/message](twimlet_message_url) 可查看空 &lt;Response&gt; 元素；又如，单击 <!--[-->http://twimlets.com/message?Message%5B0%5D=Hello%20World<!--](twimlet_message_url_hello_world)--> 可查看包含 &lt;Say&gt; 元素的 &lt;Response&gt; 元素。

您可以创建自己的返回 HTTP 响应的 URL 网站，而不用依赖 Twilio 提供的 URL。您可以使用任何语言创建返回 HTTP 响应的站点。本主题假设您将从 ASP.NET 泛型处理程序托管该 URL。

以下 ASP.NET 处理程序将生成在呼叫时表述 **Hello World** 的 TwiML 响应。

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;

    namespace WebRole1
    {
        /// <summary>
        /// Summary description for Handler1
        /// </summary>
        public class Handler1 : IHttpHandler
        {

            public void ProcessRequest(HttpContext context)
            {
                context.Response.Clear();
                context.Response.ContentType = "text/xml";
                context.Response.ContentEncoding = System.Text.Encoding.UTF8;
                string twiMLResponse = "<Response><Say>Hello World.</Say></Response>";
                context.Response.Write(twiMLResponse);
                context.Response.End();
            }

            public bool IsReusable
            {
                get
                {
                    return false;
                }
            }
        }
    }

如上面的示例中所示，TwiML 响应只是一个 XML 文档。Twilio.TwiML 库包含将为您生成 TwiML 的类。下面的示例将生成与上面所示相同的响应，但该响应会使用 TwilioResponse 类。

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;

    namespace WebRole1
    {
        /// <summary>
        /// Summary description for Handler1
        /// </summary>
        public class Handler1 : IHttpHandler
        {

            public void ProcessRequest(HttpContext context)
            {
                var twiml = new Twilio.TwiML.TwilioResponse();
                twiml.Say("Hello World.");

                context.Response.Clear();
                context.Response.ContentType = "text/xml";
                context.Response.Write(twiml.ToString());
                context.Response.End();
            }

            public bool IsReusable
            {
                get
                {
                    return false;
                }
            }
        }
    }

有关 TwiML 的详细信息，请参阅 [https://www.twilio.com/docs/api/twiml](https://www.twilio.com/docs/api/twiml)。

在设置提供 TwiML 响应的方法后，可将此 URL 传入 **client.InitiateOutboundCall** 方法中。例如，如果将名为 MyTwiML 的 Web 应用程序部署到 Azure 云服务，则 ASP.NET 处理程序的名称将为 mytwiml.ashx，并且可将 URL 传递到 **client.InitiateOutboundCall**，如以下代码示例中所示：

    // Place the call From, To, and URL values into a hash map.
    // This sample uses the sandbox number provided by Twilio to make the call.
    options.From = "NNNNNNNNNN";
    options.To = "NNNNNNNNNN";
    options.Url = "http://<your_hosted_service>.cloudapp.net/MyTwiML/mytwiml.ashx";

    // Place the call.
    var call = client.InitiateOutboundCall(options);


有关通过 ASP.NET 在 Azure 上使用 Twilio 的其他信息，请参阅[如何在 Azure 的 Web 角色中使用 Twilio 发起电话呼叫][howto_phonecall_dotnet]。

[AZURE.INCLUDE [twilio\_additional\_services\_and\_next\_steps](../includes/twilio_additional_services_and_next_steps.md)]





[howto_phonecall_dotnet]: partner-twilio-cloud-services-dotnet-phone-call-web-role.md



[twimlet_message_url]: http://twimlets.com/message

[twilio_rest_making_calls]: http://www.twilio.com/docs/api/rest/making-calls

[vs_project]: http://msdn.microsoft.com/library/windowsazure/ee405487.aspx
[nuget]: http://nuget.org/
[twilio_github_repo]: https://github.com/twilio/twilio-csharp



[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api]: http://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_account]: https://www.twilio.com/user/account
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#

<!---HONumber=71-->