<properties linkid="dev-nodejs-how-to-sendgrid-email-service" urlDisplayName="SendGrid Email Service" pageTitle="How to use the SendGrid email service (Node.js) - Azure" metaKeywords="Azure SendGrid, Azure email service, Azure SendGrid Node.js, Azure email Node.js" description="Learn how send email with the SendGrid email service on Azure. Code samples written using the Node.js API." metaCanonical="" services="" documentationCenter="Node.js" title="How to Send Email Using SendGrid from Node.js" authors="" solutions="" manager="" editor="" />

# 如何使用 SendGrid 从 Node.js 发送电子邮件

本指南演示如何在 Azure 中使用 SendGrid 电子邮件服务执行常见编程任务。相关示例是使用 Node.js API 编写的。涉及的任务包括**创建电子邮件**、**发送电子邮件**、**添加附件**、**使用筛选器**和**更新属性**。有关 SendGrid 和发送电子邮件的详细信息，请参[后续步骤][后续步骤]部分。

## 目录

-   [什么是 SendGrid 电子邮件服务？][什么是 SendGrid 电子邮件服务？]
-   [创建 SendGrid 帐户][创建 SendGrid 帐户]
-   [引用 SendGrid Node.js 模块][引用 SendGrid Node.js 模块]
-   [如何：创建电子邮件][如何：创建电子邮件]
-   [如何：发送电子邮件][如何：发送电子邮件]
-   [如何：添加附件][如何：添加附件]
-   [如何：使用筛选器启用页脚、跟踪和分析][如何：使用筛选器启用页脚、跟踪和分析]
-   [如何：更新电子邮件属性][如何：更新电子邮件属性]
-   [如何：使用其他 SendGrid 服务][如何：使用其他 SendGrid 服务]
-   [后续步骤][1]

## <a name="whatis"> </a>什么是 SendGrid 电子邮件服务？

SendGrid 是一项[基于云的电子邮件服务][基于云的电子邮件服务]，该服务提供了可靠的[事务性电子邮件传递][事务性电子邮件传递]、可缩放性、实时分析以及可用于简化自定义集成的灵活的 API。常见 SendGrid 使用方案包括：

-   自动向客户发送收据
-   管理用于每月向客户发送电子传单和特惠
    产品/服务的通讯组列表
-   收集诸如已阻止的电子邮件和客户响应性等项目的
    实时度量值
-   生成用于帮助确定趋势的报告
-   转发客户查询
-   以电子邮件的形式从应用程序发送通知

有关详细信息，请参阅 [][]<http://sendgrid.com></a>。

## <a name="createaccount"> </a>创建 SendGrid 帐户

[WACOM.INCLUDE [sendgrid-sign-up](../includes/sendgrid-sign-up.md)]

## <a name="reference"> </a>引用 SendGrid Node.js 模块

可使用以下命令通过 Node 包管理器 (npm) 安装用于 Node.js 的 SendGrid 模块：

    npm install sendgrid

安装之后，可使用以下代码要求你的应用程序中的模块：

    var SendGrid = require('sendgrid')

SendGrid 模块可导出 **SendGrid** 和 **Email** 函数。**SendGrid** 负责通过 SMTP 或 Web API 发送电子邮件，而 **Email** 负责封装电子邮件。

## <a name="createemail"> </a>如何：创建电子邮件

若要使用 SendGrid 模块创建电子邮件，需要先使用 Email 函数创建电子邮件，然后使用 SendGrid 函数发送该邮件。以下是使用 Email 函数创建新邮件的示例：

    var mail = new SendGrid.Email({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.'
    });

通过设置 html 属性，你可以为支持 HTML 邮件的客户端指定该邮件。例如：

    html: This is a sample <b>HTML<b> email message.

同时设置文本和 html 属性可以为无法支持 HTML 邮件的客户端提供文本内容的正常反馈。

有关 Email 函数支持的所有属性的详细信息，请参阅 [sendgrid-nodejs][sendgrid-nodejs]。

## <a name="sendemail"> </a>如何：发送电子邮件

使用 Email 函数创建电子邮件后，可使用 SendGrid 提供的 SMTP 或 Web API 发送该邮件。有关每个 API 的优点和差异的详细信息，请参阅 SendGrid 文档中的 SMTP 与 Web API[。][。]

使用 SMTP API 或 Web API 需要你先使用 SendGrid 帐户的用户和密钥初始化 SendGrid 函数，如下所示：

    var sender = new SendGrid('user','key');

现在可使用 SMTP 或 Web API 发送邮件。这些调用几乎是相同的：传递电子邮件和可选的回调函数；回调用于确定操作的成功与否。以下示例演示了如何同时使用 SMTP 和 Web API 发送邮件。

### SMTP

    sender.smtp(mail, function(success, err){
        if(success) console.log('Email sent');
        else console.log(err);
    });

### Web API

    sender.send(mail, function(success, err){
        if(success) console.log('Email sent');
        else console.log(err);
    });

<div class="dev-callout">
<strong>说明</strong>
<p>上面的示例演示传入电子邮件对象和
回调函数，你还可通过直接指定电子邮件属性来直接调用 send 和 smtp 函数。例如：</p>
<pre class="prettyprint">sender.send({
to:'john@contoso.com',
from:'anna@contoso.com',
subject:'test mail',
text:'This is a sample email message.'
});
</pre>
</div>

## <a name="addattachment"> </a>如何：添加附件

可通过在 **files** 属性中指定文件名和路径来将附件添加到邮件中。下面的示例演示如何发送附件：

    sender.send({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.',
        files: {
            'file1.txt': __dirname + '/file1.txt',
            'image.jpg': __dirname + '/image.jpg'
        }
    });

<div class="dev-callout">
<strong>说明</strong>
<p>在使用 <strong>files</strong> 属性时，必须可
通过 <a href="http://nodejs.org/docs/v0.6.7/api/fs.html#fs.readFile">fs.readFile</a> 访问文件。如果要附加的文件托管在 Azure 存储中（如 Blob 容器中），则必须先将该文件复制到本地存储或 Azure 驱动器，然后才能使用 <strong>files</strong> 属性将该文件作为附件发送。</p>
</div>

## <a name="usefilters"> </a>如何：使用筛选器启用脚注、跟踪和 Twitter

SendGrid 可通过使用筛选器来提供其他电子邮件功能。这些功能是一些可添加到电子邮件中以启用特定功能（如启用单击跟踪、Google 分
析、订阅跟踪等）的设置。有关筛选器的完整列表，请参阅[筛选器设置][筛选器设置]。

可使用 **filters** 属性将筛选器应用于邮件。每个筛选器均由一个包含特定于筛选器的设置的哈希指定。以下示例演示页脚、点击跟踪和Twitter 筛选器：

### 脚注

    sender.send({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.',
        filters: {
            'footer': {
                'settings': {
                    'enable': 1,
                    'text/plain': 'This is a text footer.'
                }
            }
        }
    });

### 单击跟踪

    sender.send({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.',
        filters: {
            'clicktrack': {
                'settings': {
                    'enable': 1
                }
            }
        }
    });

### Twitter

    sender.send({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.',
        filters: {
            'twitter': {
                'settings': {
                    'enable': 1,
                    'username': 'twitter_username',
                    'password': 'twitter_password'
                }
            }
        }
    });

## <a name="updateproperties"> </a>如何：更新电子邮件属性

可使用 **set *属性***替代一些电子邮件属性，或使用 **add *属性***追加一些电子邮件属性。例如，可使用以下命令添加更多收件人：

    email.addTo('jeff@contoso.com');

或使用以下命令设置筛选器：

    email.setFilterSetting({
      'footer': {
        'setting': {
          'enable': 1,
          'text/plain': 'This is a footer.'
        }
      }
    });

有关详细信息，请参阅 [sendgrid-nodejs][sendgrid-nodejs]。

## <a name="useservices"> </a>如何：使用其他 SendGrid 服务

SendGrid 提供了基于 Web 的 API，可通过这些 API从 Azure 应用程序中使用其他 SendGrid 功能。有关完整详细信息，请参阅 [SendGrid API 文档][SendGrid API 文档]。

## <a name="nextsteps"> </a> 后续步骤

在了解 SendGrid 电子邮件服务的基础知识后，请访问以下链接以了解更多信息。

-   SendGrid Node.js 模块存储库：[sendgrid-nodejs][sendgrid-nodejs]
-   SendGrid API 文档：
    <http://docs.sendgrid.com/documentation/api/>
-   面向 Azure 客户的 SendGrid 特惠产品/服务：
    [][2]<http://sendgrid.com/azure.html></a>

  [后续步骤]: http://www.windowsazure.com/zh-cn/develop/nodejs/how-to-guides/blob-storage/#next-steps
  [什么是 SendGrid 电子邮件服务？]: #whatis
  [创建 SendGrid 帐户]: #createaccount
  [引用 SendGrid Node.js 模块]: #reference
  [如何：创建电子邮件]: #createemail
  [如何：发送电子邮件]: #sendemail
  [如何：添加附件]: #addattachment
  [如何：使用筛选器启用页脚、跟踪和分析]: #usefilters
  [如何：更新电子邮件属性]: #updateproperties
  [如何：使用其他 SendGrid 服务]: #useservices
  [1]: #nextsteps
  [基于云的电子邮件服务]: http://sendgrid.com/solutions
  [事务性电子邮件传递]: http://sendgrid.com/transactional-email
  []: http://sendgrid.com
  [sendgrid-sign-up]: ../includes/sendgrid-sign-up.md
  [sendgrid-nodejs]: https://github.com/sendgrid/sendgrid-nodejs
  [。]: http://docs.sendgrid.com/documentation/get-started/integrate/examples/smtp-vs-rest/
  [fs.readFile]: http://nodejs.org/docs/v0.6.7/api/fs.html#fs.readFile
  [筛选器设置]: http://docs.sendgrid.com/documentation/api/smtp-api/filter-settings/
  [SendGrid API 文档]: http://docs.sendgrid.com/documentation/api/
  [2]: http://sendgrid.com/azure.html
