<properties linkid="develop-ruby-how-to-twilio-sms-voice-service" urlDisplayName="Twilio Voice/SMS Service" pageTitle="How to Use Twilio for Voice and SMS (Ruby) - Azure" metaKeywords="Azure Ruby Twilio, Azure phone calls, Azure phone calls, Azure twilio, Azure SMS, Azure SMS, Azure voice calls, azure voice calls, Azure text messages, Azure text messages" description="Learn how to make a phone call and send a SMS message with the Twilio API service on Azure. Code samples written in Ruby." metaCanonical="" services="" documentationCenter="Ruby" title="How to Use Twilio for Voice and SMS Capabilities in PHP" authors="" solutions="" manager="" editor="" />

# 如何通过 Ruby 使用 Twilio 实现语音和 SMS 功能

本指南演示如何在 Azure 中使用 Twilio API 服务执行常见编程任务。所涉及的任务包括发起电话呼叫和发送短信服务 (SMS) 消息。有关 Twilio 以及在应用程序中使用语音和 SMS 的详细信息，请参阅[后续步骤][后续步骤]部分。

## 目录

-   [什么是 Twilio？][什么是 Twilio？]
-   [Twilio 定价][Twilio 定价]
-   [概念][概念]
-   [创建 Twilio 帐户][创建 Twilio 帐户]
-   [创建 Ruby Sinatra 应用程序][创建 Ruby Sinatra 应用程序]
-   [将应用程序配置为使用 Twilio 库][将应用程序配置为使用 Twilio 库]
-   [如何：发起传出呼叫][如何：发起传出呼叫]
-   [如何：接收短信][如何：接收短信]
-   [如何：使用其他 Twilio 服务][如何：使用其他 Twilio 服务]
-   [后续步骤][后续步骤]

## <span id="WhatIs"></span></a>什么是 Twilio？

Twilio 是一种电话 Web 服务 API，它使你能够使用现有 Web 语言和技术生成语音和 SMS 应用程序。Twilio 属于第三方服务（而非 Azure 功能和 Microsoft 产品）。

利用 **Twilio 语音**，你的应用程序可以发起和接收电话呼叫。利用 **Twilio SMS**，你的应用程序可以发出和接收 SMS 消息。利用 **Twilio 客户端**，你的应用程序可以使用现有 Internet 连接（包括移动连接）启用语音通信。

## <span id="Pricing"></span></a>Twilio 定价和特别优惠

[Twilio 定价][1]中提供了有关 Twilio 定价的信息。Azure 客户可享受[特别优惠][特别优惠]：免费获得 1000 条信息和 1000 分钟拨入通话时间。若要注册此优惠或了解更多信息，请访问 [][特别优惠]<http://ahoy.twilio.com/azure></a>。

## <span id="Concepts"></span></a>概念

Twilio API 是一个为应用程序提供语音和 SMS 功能的 RESTful API。提供了多种语言版本的客户端库；有关列表，请参阅 [Twilio API 库][Twilio API 库]。

### <span id="TwiML"></span></a>TwiML

TwiML 是一组基于 XML 的指令，可指示 Twilio 如何处理呼叫或短信。

例如，以下 TwiML 会将文本 **Hello World** 转换为语音。

    <?xml version="1.0" encoding="UTF-8" ?>
    <Response>
       <Say>Hello World</Say>
    </Response>

所有 TwiML 文档都使用 `<Response>` 作为其根元素。你可以在根元素中使用 Twilio 谓词定义应用程序的行为。

### <span id="Verbs"></span></a>Twilio 谓词

Twilio 谓词是指示 Twilio 执行哪些**操作**的 XML 标记。例如，**\<Say\>** 谓词指示 Twilio 在呼叫时传递语音消息。

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

有关 Twilio 谓词、其属性和 TwiML 的详细信息，请参阅 [TwiML][TwiML]。有关 Twilio API 的其他信息，请参阅 [Twilio API][Twilio API]。

## <span id="CreateAccount"></span></a>创建 Twilio 帐户

准备好获取 Twilio 帐户后，请在[试用 Twilio][试用 Twilio] 上注册。可以先使用免费帐户，以后再升级你的帐户。

注册 Twilio 帐户时，你将收到适用于你的应用程序的一个免费电话号码。还会收到帐户 SID 和身份验证令牌。需要二者才能发起 Twilio API 呼叫。为了防止对你的帐户进行未经授权的访问，请保护身份验证令牌。你的帐户 SID 和身份验证令牌分别显示在 [Twilio 帐户页][Twilio 帐户页]上标记为“ACCOUNT SID”（帐户 SID）和“AUTH TOKEN”（身份验证令牌）的字段中。

### <span id="VerifyPhoneNumbers"></span></a>验证电话号码

除了 Twilio 提供的号码外，还可验证你控制为在应用程序中使用的号码（例如，手机号码或家庭电话号码）。

有关如何验证电话号码的信息，请参阅[管理号码（可能为英文页面）][管理号码（可能为英文页面）]。

## <span id="create_app"></span></a>创建 Ruby 应用程序

使用 Twilio 服务且在 Azure 中运行的 Ruby 应用程序与任何其他使用 Twilio 服务的 Ruby 应用程序之间没有任何差别。Twilio 服务基于 REST 并且可通过多种方法从 Ruby 中调用，本文将重点介绍如何将 Twilio 服务与[用于 Ruby 的 Twilio 帮助程序库][用于 Ruby 的 Twilio 帮助程序库]一起使用。

首先，[设置一台新的 Azure Linux 虚拟机][设置一台新的 Azure Linux 虚拟机]充当新 Ruby Web 应用程序的主机。忽略创建 Rails 应用程序所涉及的步骤，只设置虚拟机。确保创建的终结点的外部端口为 80，内部端口为 5000。

在以下示例中，我们将使用 [Sinatra][Sinatra]，一个非常简单的用于 Ruby 的 Web 框架。不过，你当然可以将用于 Ruby 的 Twilio 帮助程序库与任何其他 Web 框架（包括 Ruby on Rails）一起使用。

通过 SSH 登录到你的新虚拟机并为新应用程序创建一个目录。在该目录内创建一个名为 Gemfile 的文件并将以下代码复制到该文件中：

    source 'https://rubygems.org'
    gem 'sinatra'
    gem 'thin'

在命令行上，运行`bundle install`. 这将安装上述依赖项。接下来，创建文件`web.rb`. 它将包含你的 Web 应用程序的代码。将以下代码粘贴到该文件中：

    require 'sinatra'

    get '/' do
        "Hello Monkey!"
    end

此时，你应该能够运行命令 `ruby web.rb -p 5000`. 这将在端口 5000 上启动一个小型 Web 服务器。你应该能够通过访问为 Azure VM 设置的 URL，在浏览器中浏览到此应用程序。在浏览器中打开 Web 应用程序后，即可开始生成 Twilio 应用程序。

## <span id="configure_app"></span></a>将应用程序配置为使用 Twilio

可以通过将 `Gemfile` 更新为包含以下行，将 Web 应用程序配置为使用 Twilio 库：

    gem 'twilio-ruby'

在命令行上，运行 `bundle install`. 现在，请打开 `web.rb` 并在顶部添加以下行：

    require 'twilio-ruby'

现在你已经可以在 Web 应用程序中使用用于 Ruby 的 Twilio 帮助程序库。

## <span id="howto_make_call"></span></a>如何：发起传出呼叫

下面演示如何发起传出呼叫。主要概念包括使用用于 Ruby 的 Twilio 帮助程序调用 REST API 以及呈现 TwiML。用你的值替换“呼叫方”和“被呼叫方”电话号码，并确保在运行代码之前验证 Twilio 帐户的“呼叫方”电话号码。

将此函数添加到 `web.md`:

    # Set your account ID and authentication token.
    sid = "your_twilio_account_sid";
    token = "your_twilio_authentication_token";

    # The number of the phone initiating the the call.
    # This should either be a Twilio number or a number that you've verified
    from = "NNNNNNNNNNN";

    # The number of the phone receiving call.
    to = "NNNNNNNNNNN";

    # Use the Twilio-provided site for the TwiML response.
    url = "http://yourdomain.cloudapp.net/voice_url";
      
    get '/make_call' do
      # Create the call client.
      client = Twilio::REST::Client.new(sid, token);
      
      # Make the call
      client.account.calls.create(to: to, from: from, url: url)
    end

    post '/voice_url' do
      "<Response>
         <Say>Hello Monkey!</Say>
       </Response>"
    end

如果你在浏览器中打开 `http://yourdomain.cloudapp.net/make_call` ，将会触发 Twilio API 调用以发起电话呼叫。元素 `client.account.calls.create` 中的前两个参数很容易理解：主叫号码为 `from` ，被叫号码为 `to`.

第三个参数 (`url`) 是 Twilio 请求的 URL，用于获取在连接呼叫后要执行的操作的说明。在本例中，我们设置的 URL (`http://yourdomain.cloudapp.net`) 会返回一个简单 TwiML 文档，并使用 `<Say>` 谓词执行某种形式的文本到语音转换，以便向被呼叫方讲出“Hello Monkey”。

## <span id="howto_recieve_sms"></span></a>如何：接收 SMS 消息

在前面的示例中，我们发起了一个**传出**电话呼叫。这一次，我们使用在注册过程中 Twilio 提供的电话号码来处理**传入的** SMS 消息。

首先，请登录到 [Twilio 仪表板][Twilio 帐户页]。单击顶部导航栏中的“Numbers”（号码），然后单击提供给你的 Twilio 号码。你将看到两个可以配置的 URL。一个语音请求 URL 和一个短信请求 URL。每当有人拨打你的号码或向你的号码发送短信时，Twilio 就会调用这些 URL。这些 URL 也称为“Web 挂钩”。

我们想要处理传入的 SMS 消息，因此需要将 URL 更新为`http://yourdomain.cloudapp.net/sms_url`. 继续操作并单击页面底部的“Save Changes”（保存更改）。现在，请返回到 `web.rb` ，然后对应用程序进行编程以处理此更改：

    post '/sms_url' do
      "<Response>
         <Sms>Hey, thanks for the ping! Twilio and Azure rock!</Sms>
       </Response>"
    end

进行更改后，确保重新启动 Web 应用程序。现在，拿出手机，向你的 Twilio 号码发送一条短信。你应该会立即收到短信响应，内容为“Hey, thanks for the ping!Twilio and Azure rock!”。

## <span id="additional_services"></span></a>如何：使用其他 Twilio 服务

除了此处所示的示例之外，Twilio 还提供了基于 Web 的 API，可通过这些 API 从 Azure 应用程序中使用其他 Twilio 功能。有关完整详细信息，请参阅 [Twilio API 文档][Twilio API]。

### <span id="NextSteps"></span></a> 后续步骤

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
  [创建 Ruby Sinatra 应用程序]: #create_app
  [将应用程序配置为使用 Twilio 库]: #configure_app
  [如何：发起传出呼叫]: #howto_make_call
  [如何：接收短信]: #howto_recieve_sms
  [如何：使用其他 Twilio 服务]: #additional_services
  [1]: http://www.twilio.com/pricing
  [特别优惠]: http://ahoy.twilio.com/azure
  [Twilio API 库]: https://www.twilio.com/docs/libraries
  [TwiML]: http://www.twilio.com/docs/api/twiml
  [Twilio API]: http://www.twilio.com/api
  [试用 Twilio]: https://www.twilio.com/try-twilio
  [Twilio 帐户页]: https://www.twilio.com/user/account
  [管理号码（可能为英文页面）]: https://www.twilio.com/user/account/phone-numbers/verified#
  [用于 Ruby 的 Twilio 帮助程序库]: https://www.twilio.com/docs/ruby/install
  [设置一台新的 Azure Linux 虚拟机]: http://www.windowsazure.com/zh-cn/develop/ruby/tutorials/web-app-with-linux-vm/
  [Sinatra]: http://www.sinatrarb.com/
  [Twilio 安全准则]: http://www.twilio.com/docs/security
  [Twilio 操作方法和示例代码]: http://www.twilio.com/docs/howto
  [Twilio 快速入门教程]: http://www.twilio.com/docs/quickstart
  [GitHub 上的 Twilio]: https://github.com/twilio
  [与 Twilio 技术支持人员交流]: http://www.twilio.com/help/contact
