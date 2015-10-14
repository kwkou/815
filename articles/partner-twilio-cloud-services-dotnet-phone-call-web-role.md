<properties 
	pageTitle="如何从 Twilio (.NET) 发起电话呼叫 | Microsoft Azure" 
	description="了解如何在 Azure 中使用 Twilio API 服务发起电话呼叫和发送短信。采用 .NET 编写的代码示例。" 
	services="" 
	documentationCenter=".net" 
	authors="devinrader" 
	manager="twilio" 
	editor=""/>

<tags 
	ms.service="cloud-services" 
	ms.date="08/02/2015" 
	wacn.date="10/3/2015"/>




# 如何在 Azure 的 Web 角色中使用 Twilio 发起电话呼叫

本指南演示如何使用 Twilio 从 Azure 中托管的网页发起呼叫。生成的应用程序将提示用户输入电话呼叫值，如以下屏幕截图中所示。

![使用 Twilio 和 ASP.NET 的 Azure 呼叫窗体][twilio_dotnet_basic_form]

## <a name="twilio-prereqs"></a>先决条件

你将需要执行以下操作才能使用本主题中的代码：

1. 获取 Twilio 帐户和身份验证令牌。若要开始使用 Twilio，请在 [https://www.twilio.com/try-twilio][try_twilio] 上进行注册。可以在 [http://www.twilio.com/pricing][twilio_pricing] 上评估定价。有关 Twilio 提供的 API 的信息，请参阅 [http://www.twilio.com/voice/api][twilio_api]。
2. 将 Twilio .NET 库添加到你的 Web 角色。请参阅本主题后面的“将 Twilio 库添加到 Web 角色项目”。

你应该熟悉如何在 Azure 中创建基本 Web 角色。

## <a name="howtocreateform"></a>如何：创建用于发起呼叫的 Web 窗体

<a id="use_nuget"></a>将 Twilio 库添加到 Web 角色项目：

1.  在 Visual Studio 中打开您的解决方案。
2.  右键单击“引用”。
3.  单击“管理 NuGet 包”。
4.  单击“联机”。
5.  在联机搜索框中，键入 *twilio*。
6.  单击 Twilio 程序包对应的“安装”。

以下代码演示了如何创建 Web 窗体来检索用于发起呼叫的用户数据。在此示例中，将创建名为 **TwilioCloud** 的 ASP.NET Web 角色。

    <%@ Page Title="Home Page" Language="C#" MasterPageFile="~/Site.master"
        AutoEventWireup="true" CodeBehind="Default.aspx.cs"
        Inherits="WebRole1._Default" %>

    <asp:Content ID="HeaderContent" runat="server" ContentPlaceHolderID="HeadContent">
    </asp:Content>
    <asp:Content ID="BodyContent" runat="server" ContentPlaceHolderID="MainContent">
        <div>
            <asp:BulletedList ID="varDisplay" runat="server" BulletStyle="NotSet">
            </asp:BulletedList>
        </div>
        <div>
            <p>Fill in all fields and click <b>Make this call</b>.</p>
            <div>
                To:<br /><asp:TextBox ID="toNumber" runat="server" /><br /><br />
                Message:<br /><asp:TextBox ID="message" runat="server" /><br /><br />
                <asp:Button ID="callpage" runat="server" Text="Make this call"
                    onclick="callpage_Click" />
            </div>
        </div>
    </asp:Content>

## <a id="howtocreatecode"></a>如何：创建用于发起呼叫的代码
在用户完成窗体后将调用以下代码，以创建呼叫消息并生成呼叫。在此示例中，该代码在窗体上按钮的 Onclick 事件处理程序中运行。（使用 Twilio 帐户和身份验证令牌，而不是分配给以下代码中的 **accountSID** 和 **authToken** 的占位符值。）

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;
    using System.Web.UI;
    using System.Web.UI.WebControls;
    using Twilio;

    namespace WebRole1
    {
        public partial class _Default : System.Web.UI.Page
        {
            protected void Page_Load(object sender, EventArgs e)
            {

            }

            protected void callpage_Click(object sender, EventArgs e)
            {
                // Call porcessing happens here.

                // Use your account SID and authentication token instead of
                // the placeholders shown here.
                string accountSID = "ACNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN";
                string authToken =  "NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN";

                // Instantiate an instance of the Twilio client.
                TwilioRestClient client;
                client = new TwilioRestClient(accountSID, authToken);

                // Retrieve the account, used later to retrieve the
                Twilio.Account account = client.GetAccount();
                string APIversuion = client.ApiVersion;
                string TwilioBaseURL = client.BaseUrl;

                this.varDisplay.Items.Clear();
                if (this.toNumber.Text == "" || this.message.Text == "")
                {
                    this.varDisplay.Items.Add(
                            "You must enter a phone number and a message.");
                }
                else
                {
                    // Retrieve the values entered by the user.
                    string to = this.toNumber.Text;
                    string myMessage = this.message.Text;

                    // Create a URL using the Twilio message and the user-entered
                    // text. You must replace spaces in the user's text with '%20'
                    // to make the text suitable for a URL.
                    String Url = "http://twimlets.com/message?Message%5B0%5D="
                            + myMessage.Replace(" ", "%20");

                    // Display the endpoint, API version, and the URL for the message.
                    this.varDisplay.Items.Add("Using Twilio endpoint "
                        + TwilioBaseURL);
                    this.varDisplay.Items.Add("Twilioclient API Version is "
                        + APIversuion);
                    this.varDisplay.Items.Add("The URL is " + Url);

                    // Instantiate the call options that are passed
                    // to the outbound call.
                    CallOptions options = new CallOptions();

                    // Set the call From, To, and URL values.                    
                    options.From = "+14155992671";
                    options.To = to;
                    options.Url = Url;

                    // Place the call.
                    var call = client.InitiateOutboundCall(options);
                    this.varDisplay.Items.Add("Call status: " + call.Status);
                }
            }
        }
    }

将发起呼叫，并显示 Twilio 终结点、API 版本和呼叫状态。以下屏幕截图显示了示例运行的输出。

![使用 Twilio 和 ASP.NET 的 Azure 呼叫响应][twilio_dotnet_basic_form_output]

有关 TwiML 的详细信息可在 [http://www.twilio.com/docs/api/twiml][twiml] 中找到。有关 &lt;Say&gt; 和其他 Twilio 谓词的详细信息可在 [http://www.twilio.com/docs/api/twiml/say][twilio_say] 中找到。

## <a id="nextsteps"></a>后续步骤
提供此代码是为了演示在 Azure 上的 ASP.NET Web 角色中使用 Twilio 的基本功能。在生产中部署到 Azure 之前，你可能希望添加更多错误处理功能或其他功能。例如：

* 您可以使用 Azure Blob 存储或 Azure SQL 数据库实例存储电话号码和呼叫文本，而不使用 Web 窗体。有关在 Azure 中使用 Blob 的信息，请参阅[如何在 .NET 中使用 Azure Blob 存储服务][howto_blob_storage_dotnet]。有关使用 SQL 数据库的信息，请参阅[如何在 .NET 应用程序中使用 Azure SQL 数据库][howto_sql_azure_dotnet]。
* 你可以使用 RoleEnvironment.getConfigurationSettings 从部署的配置设置中检索 Twilio 帐户 ID 和身份验证令牌，而不是在窗体中对这些值进行硬编码。有关 RoleEnvironment 类的信息，请参阅 [Microsoft.WindowsAzure.ServiceRuntime 命名空间][azure_runtime_ref_dotnet]。
* 阅读 [https://www.twilio.com/docs/security][twilio_docs_security] 上的 Twilio 安全准则。
* 在 [https://www.twilio.com/docs][twilio_docs] 上了解有关 Twilio 的详细信息。

##<a name="seealso"></a>另请参阅
* [如何在 Azure 中使用 Twilio 实现语音和短信功能](/documentation/articles/twilio-dotnet-how-to-use-for-voice-sms)

[twilio_pricing]: http://www.twilio.com/pricing
[try_twilio]: http://www.twilio.com/try-twilio
[twilio_api]: http://www.twilio.com/voice/api
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#

[twilio_dotnet_basic_form]: ./media/partner-twilio-cloud-services-dotnet-phone-call-web-role/WA_twilio_dotnet_basic_form.png
[twilio_dotnet_basic_form_output]: ./media/partner-twilio-cloud-services-dotnet-phone-call-web-role/WA_twilio_dotnet_basic_form_output.png

[twiml]: http://www.twilio.com/docs/api/twiml



[howto_twilio_voice_sms_dotnet]: /develop/net/how-to-guides/twilio/

[howto_blob_storage_dotnet]: https://www.windowsazure.cn/develop/net/how-to-guides/blob-storage/

[howto_sql_azure_dotnet]: https://www.windowsazure.cn/develop/net/how-to-guides/sql-database/


[twilio_docs_security]: http://www.twilio.com/docs/security
[twilio_docs]: http://www.twilio.com/docs
[twilio_say]: http://www.twilio.com/docs/api/twiml/say


[azure_runtime_ref_dotnet]: http://msdn.microsoft.com/library/windowsazure/microsoft.windowsazure.serviceruntime.aspx

<!---HONumber=71-->