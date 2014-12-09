<properties linkid="dev-net-how-to-sendgrid-email-service" urlDisplayName="SendGrid Email Service" pageTitle="How to use the SendGrid email service (.NET) - Azure" metaKeywords="Azure SendGrid, Azure email service, Azure SendGrid .NET, Azure email .NET, Azure SendGrid C#, Azure email C#" description="Learn how send email with the SendGrid email service on Azure. Code samples written in C# and use the .NET API." metaCanonical="" services="" documentationCenter=".NET" title="How to Send Email Using SendGrid with Azure" authors="" solutions="" manager="" editor="" />

# 如何在 Azure 中使用 SendGrid 发送电子邮件

本指南演示如何在 Azure 中使用SendGrid 电子邮件服务执行常见编程任务。相关示例用 C# 编写且使用 .NET API。涉及的任务包括**创建电子邮件**、**发送电子邮件**、**添加附件**和**使用筛选器**。有关 SendGrid 和发送电子邮件的详细信息，请参阅[后续步骤][后续步骤]部分。

## <a name="toc"></a>目录

[什么是 SendGrid 电子邮件服务？][什么是 SendGrid 电子邮件服务？]
[创建 SendGrid 帐户][创建 SendGrid 帐户]
[引用 SendGrid .NET 类库][引用 SendGrid .NET 类库]
[如何：创建电子邮件][如何：创建电子邮件]
[如何：发送电子邮件][如何：发送电子邮件]
[如何：添加附件][如何：添加附件]
[如何：使用筛选器启用页脚、跟踪和分析][如何：使用筛选器启用页脚、跟踪和分析]
[如何：使用其他 SendGrid 服务][如何：使用其他 SendGrid 服务]
[后续步骤][后续步骤]

## <a name="whatis"></a><span class="short-header">什么是 SendGrid 电子邮件服务？</span>什么是 SendGrid 电子邮件服务？

SendGrid 是一项[基于云的电子邮件服务][基于云的电子邮件服务]，该服务提供了可靠的[事务性电子邮件传递][事务性电子邮件传递]、可缩放性、实时分析以及可用于简化自定义集成的灵活的 API。常见 SendGrid 使用方案包括：

-   自动向客户发送收据。
-   管理用于每月向客户发送电子传单和特惠产品/服务的
    通讯组列表。
-   收集诸如已阻止的电子邮件和客户响应性等项目的
    实时度量值。
-   生成用于帮助确定趋势的报告。
-   转发客户查询。

有关详细信息，请参阅 [][]<http://sendgrid.com></a>。

## <a name="createaccount"></a><span class="short-header">创建 SendGrid 帐户</span>创建 SendGrid 帐户

[WACOM.INCLUDE [sendgrid-sign-up][sendgrid-sign-up]]

## <a name="reference"></a><span class="short-header">引用 SendGrid .NET 类库</span>引用 SendGrid .NET 类库

SendGrid NuGet 包是获取 SendGrid API 和使用所有依赖项配置应用程序的最简单方法。NuGet 是Microsoft Visual Studio 2012 随附的一个 Visual Studio 扩展，可让你轻松安装和更新库与工具。

<div class="dev-callout">
<b>说明</b>
<p>如果
你运行的 Visual Studio 版本低于 Visual Studio 2012，并且想要安装 NuGet，那么，请访问 <a href="http://www.nuget.org">http://www.nuget.org</a>，然后单击&ldquo;安装<b>
</b>NuGet&rdquo;按钮。</p>
</div>

若要在应用程序中安装 SendGrid NuGet 包，请执行以下操作：

1.  在“解决方案资源管理器”中，右键单击“引用”，然后单击“管理 NuGet 包”。

2.  在“管理 NuGet 包”对话框的左窗格中，单击“联机”。

3.  搜索 **SendGrid**，然后在结果列表中选择“SendGrid”项。

    ![SendGrid NuGet 包][SendGrid NuGet 包]

4.  单击“安装”以完成安装，然后关闭此对话框。

SendGrid 的 .NET 类库名为 **SendGridMail**。它包含以下命名空间：

-   **SendGridMail**：用于创建和处理电子邮件项。
-   **SendGridMail.Transport**：用于通过
    **SMTP** 协议或采用 **Web/REST** 的 HTTP 1.1 协议发送电子邮件。

将以下代码命名空间声明添加到你想要在其中以编程方式访问 SendGrid 电子邮件服务的任何 C# 文件的顶部。
**System.Net** 和 **System.Net.Mail** 是 .NET Framework 命名空间，之所以提供这些命名空间，是因为它们包含你通常要与 SendGrid API 结合使用的类型。

    using System.Net;
    using System.Net.Mail;
    using SendGridMail;
    using SendGridMail.Transport;

## <a name="createemail"></a><span class="short-header">如何：创建电子邮件</span>如何：创建电子邮件

使用静态 **SendGrid.GetInstance** 方法可以创建 **SendGrid** 类型的电子邮件。创建邮件后，你可以使用 **SendGrid** 属性和方法来设置值，包括电子邮件发件人、电子邮件收件人以及电子邮件的主题和正文。

以下示例演示了如何创建完全填充的电子邮件对象：

    // Setup the email properties.
    var from = new MailAddress("john@contoso.com");
    var to   = new MailAddress[] { new MailAddress("jeff@contoso.com") };
    var cc   = new MailAddress[] { new MailAddress("anna@contoso.com") };
    var bcc  = new MailAddress[] { new MailAddress("peter@contoso.com") };
    var subject = "Testing the SendGrid Library";
    var html    = "<p>Hello World!</p>";
    var text = "Hello World plain text!";
    var transport = SendGridMail.TransportType.SMTP;

    // Create an email, passing in the the eight properties as arguments.
    SendGrid myMessage = SendGrid.GetInstance(from, to, cc, bcc, subject, html, text, transport);

以下示例演示了如何创建空电子邮件对象：

    // Create the email object first, then add the properties.
    SendGrid myMessage = SendGrid.GetInstance();
     
    // Add the message properties.
    MailAddress sender = new MailAddress(@"John Smith <john@contoso.com>");
    myMessage.From = sender;
     
    // Add multiple addresses to the To field.
    List<String> recipients = new List<String>
    {
        @"Jeff Smith <jeff@contoso.com>",
        @"Anna Lidman <anna@contoso.com>",
        @"Peter Saddow <peter@contoso.com>"
    };
     
    foreach (string recipient in recipients)
    {
        myMessage.AddTo(recipient);
    }
     
    // Add a message body in HTML format.
    myMessage.Html = "<p>Hello World!</p>";

    // Add the subject.
    myMessage.Subject = "Testing the SendGrid Library";

有关 **SendGrid** 类型支持的所有属性和方法的详细信息，请参阅 GitHub 上的 [sendgrid-csharp][sendgrid-csharp]。

## <a name="sendemail"></a><span class="short-header">如何：发送电子邮件</span>如何：发送电子邮件

创建电子邮件后，可以使用 SendGrid 提供的 SMTP 或Web API 发送该邮件。有关每个 API 的优点和缺点的详细信息，请参阅 SendGrid 文档中的 [SMTP 与 Web API][SMTP 与 Web API]。

使用任一协议发送电子邮件都需要你提供SendGrid 帐户凭据（用户名和密码）。以下代码演示了如何在 **NetworkCredential**对象中包装凭据：

    // Create network credentials to access your SendGrid account.
    var username = "your_sendgrid_username";
    var pswd = "your_sendgrid_password";

    var credentials = new NetworkCredential(username, pswd);

若要发送电子邮件，请在**SMTP** 类（使用 SMTP 协议）或 **REST** 传输类（调用 SendGrid Web API）上使用 **Deliver** 方法。以下示例演示了如何同时使用 SMTP 和 Web API 发送邮件。

### SMTP

    // Create the email object first, then add the properties.
    SendGrid myMessage = SendGrid.GetInstance();
    myMessage.AddTo("anna@contoso.com");
    myMessage.From = new MailAddress("john@contoso.com", "John Smith");
    myMessage.Subject = "Testing the SendGrid Library";
    myMessage.Text = "Hello World!";

    // Create credentials, specifying your user name and password.
    var credentials = new NetworkCredential("username", "password");

    // Create an SMTP transport for sending email.
    var transportSMTP = SMTP.GetInstance(credentials);

    // Send the email.
    transportSMTP.Deliver(myMessage);

### Web API

    // Create the email object first, then add the properties.
    SendGrid myMessage = SendGrid.GetInstance();
    myMessage.AddTo("anna@contoso.com");
    myMessage.From = new MailAddress("john@contoso.com", "John Smith");
    myMessage.Subject = "Testing the SendGrid Library";
    myMessage.Text = "Hello World!";

    // Create credentials, specifying your user name and password.
    var credentials = new NetworkCredential("username", "password");

    // Create a REST transport for sending email.
    var transportREST = Web.GetInstance(credentials);

    // Send the email.
    transportREST.Deliver(myMessage);

## <a name="addattachment"></a><span class="short-header">如何：添加附件</span>如何：添加附件

可以通过调用 **AddAttachment**方法并指定要附加的文件的名称和路径，将附件添加到邮件。针对想要附加的每个文件调用此方法一次
就能包含多个附件。以下示例演示了如何将附件添加到邮件：

    SendGrid myMessage = SendGrid.GetInstance();
    myMessage.AddTo("anna@contoso.com");
    myMessage.From = new MailAddress("john@contoso.com", "John Smith");
    myMessage.Subject = "Testing the SendGrid Library";
    myMessage.Text = "Hello World!";

    myMessage.AddAttachment(@"C:\file1.txt");

## <a name="usefilters"></a><span class="short-header">如何：使用筛选器启用页脚、跟踪和分析</span>如何：使用筛选器启用页脚、跟踪和分析

SendGrid 可通过使用筛选器来提供其他电子邮件功能。可将这些设置添加到电子邮件以启用特定功能（例如单击跟踪、Google 分析、
订阅跟踪等）。有关筛选器的完整列表，请参阅[筛选器设置][筛选器设置]。

可以使用作为 **SendGrid** 类的一部分实现的方法将筛选器应用到 **SendGrid** 电子邮件。必须先调用 **InitalizeFilters** 方法初始化可用筛选器的列表，然后才能对电子邮件启用筛选器。

以下示例演示了脚注和单击跟踪筛选器：

### 脚注

    // Create the email object first, then add the properties.
    SendGrid myMessage = SendGrid.GetInstance();
    myMessage.AddTo("anna@contoso.com");
    myMessage.From = new MailAddress("john@contoso.com", "John Smith");
    myMessage.Subject = "Testing the SendGrid Library";
    myMessage.Text = "Hello World!";

    myMessage.InitializeFilters();
    // Add a footer to the message.
    myMessage.EnableFooter("PLAIN TEXT FOOTER", "<p><em>HTML FOOTER</em></p>");

### 单击跟踪

    // Create the email object first, then add the properties.
    SendGrid myMessage = SendGrid.GetInstance();
    myMessage.AddTo("anna@contoso.com");
    myMessage.From = new MailAddress("john@contoso.com", "John Smith");
    myMessage.Subject = "Testing the SendGrid Library";
    myMessage.Html = "<p><a href=\"http://www.windowsazure.com\">Hello World Link!</a></p>";
    myMessage.Text = "Hello World!";

    myMessage.InitializeFilters();
    // true indicates that links in plain text portions of the email 
    // should also be overwritten for link tracking purposes. 
    myMessage.EnableClickTracking(true);

## <a name="useservices"></a><span class="short-header">如何：使用其他 SendGrid 服务</span>如何：使用其他 SendGrid 服务

SendGrid 提供了基于 Web 的 API，可通过这些 API从 Azure 应用程序中使用其他 SendGrid 功能。有关完整详细信息，请参阅 [SendGrid API 文档][SendGrid API 文档]。

## <a name="nextsteps"></a><span class="short-header">后续步骤</span>后续步骤

在了解 SendGrid 电子邮件服务的基础知识后，请访问以下链接以了解更多信息。

-   SendGrid C# 库存储库：[sendgrid-csharp][sendgrid-csharp]
-   SendGrid API 文档：<http://docs.sendgrid.com/documentation/api/>
-   面向 Azure 客户的 SendGrid 特惠产品/服务：[][]<http://sendgrid.com></a>

  [后续步骤]: #nextsteps
  [什么是 SendGrid 电子邮件服务？]: #whatis
  [创建 SendGrid 帐户]: #createaccount
  [引用 SendGrid .NET 类库]: #reference
  [如何：创建电子邮件]: #createemail
  [如何：发送电子邮件]: #sendemail
  [如何：添加附件]: #addattachment
  [如何：使用筛选器启用页脚、跟踪和分析]: #usefilters
  [如何：使用其他 SendGrid 服务]: #useservices
  [基于云的电子邮件服务]: http://sendgrid.com/solutions
  [事务性电子邮件传递]: http://sendgrid.com/transactional-email
  []: http://sendgrid.com
  [sendgrid-sign-up]: ../includes/sendgrid-sign-up.md
  [SendGrid NuGet 包]: ./media/sendgrid-dotnet-how-to-send-email/sendgrid01.png
  [sendgrid-csharp]: https://github.com/sendgrid/sendgrid-csharp
  [SMTP 与 Web API]: http://docs.sendgrid.com/documentation/get-started/integrate/examples/smtp-vs-rest/
  [筛选器设置]: http://docs.sendgrid.com/documentation/api/smtp-api/filter-settings/
  [SendGrid API 文档]: http://docs.sendgrid.com/documentation/api/
