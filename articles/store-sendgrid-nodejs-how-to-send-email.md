<properties 
	pageTitle="如何使用 SendGrid 电子邮件服务 (Node.js) | Microsoft Azure" 
	description="了解如何在 Azure 上使用 SendGrid 电子邮件服务发送电子邮件。代码示例用 Node.js API 编写。" 
	services="" 
	documentationCenter="nodejs" 
	authors="MikeWasson" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="multiple" 
	ms.date="10/30/2014" 
	wacn.date="10/3/2015"/>





# 如何使用 SendGrid 从 Node.js 发送电子邮件

本指南演示了如何在 Azure 上使用 SendGrid 电子邮件服务执行常见编程任务。相关示例是使用 Node.js API 编写的。涉及的任务包括**创建电子邮件**、**发送电子邮件**、**添加附件**、**使用筛选器**和**更新属性**。有关 SendGrid 和发送电子邮件的详细信息，请参阅[后续步骤][]部分。

## 目录

* [什么是 SendGrid 电子邮件服务？][]   
* [创建 SendGrid 帐户][]   
* [引用 SendGrid Node.js 模块][]   
* [如何：创建电子邮件][]   
* [如何：发送电子邮件][]   
* [如何：添加附件][]   
* [如何：使用筛选器启用页脚、跟踪和分析][]   
* [如何：更新电子邮件属性][]   
* [如何：使用其他 SendGrid 服务][]   
* [后续步骤][1]

## <a name="whatis"> </a>什么是 SendGrid 电子邮件服务？

SendGrid 是一项[基于云的电子邮件服务]，该服务提供了可靠的[事务性电子邮件传递]、可缩放性、实时分析以及可用于简化自定义集成的灵活的 API。常见 SendGrid 使用方案包括：

-   自动向客户发送收据
-   管理用于每月向客户发送电子传单和特惠产品/服务的通讯组列表
-   收集诸如已阻止的电子邮件和客户响应性等项目的实时度量值
-   生成用于帮助确定趋势的报告
-   转发客户查询
-   以电子邮件的形式从应用程序发送通知

有关详细信息，请参阅 [https://sendgrid.com](https://sendgrid.com)。

## <a name="createaccount"> </a>创建 SendGrid 帐户

[AZURE.INCLUDE [sendgrid-sign-up](../includes/sendgrid-sign-up.md)]

## <a name="reference"> </a>引用 SendGrid Node.js 模块

可使用以下命令通过 Node 包管理器 (npm) 安装用于 Node.js 的 SendGrid 模块：

    npm install sendgrid

安装之后，可使用以下代码要求您的应用程序中的模块：

    var sendgrid = require('sendgrid')(sendgrid_username, sendgrid_password);

SendGrid 模块可导出 **SendGrid** 和 **Email** 函数。**SendGrid** 负责通过 Web API 发送电子邮件，而 **Email** 负责封装电子邮件。

## <a name="createemail"> </a>如何：创建电子邮件

若要使用 SendGrid 模块创建电子邮件，需要先使用 Email 函数创建电子邮件，然后使用 SendGrid 函数发送该邮件。以下是使用 Email 函数创建新邮件的示例：

    var email = new sendgrid.Email({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.'
    });

通过设置 html 属性，您可以为支持 HTML 邮件的客户端指定该邮件。例如：

    html: This is a sample <b>HTML<b> email message.

同时设置文本和 html 属性可以为无法支持 HTML 邮件的客户端提供文本内容的正常反馈。

有关 Email 函数支持的所有属性的详细信息，请参阅 [sendgrid-nodejs][]。

## <a name="sendemail"> </a>如何：发送电子邮件

使用 Email 函数创建电子邮件后，可使用 SendGrid 提供的 Web API 发送该邮件。

### Web API

    sendgrid.send(email, function(err, json){
        if(err) { return console.error(err); }
        console.log(json);
    });

> [AZURE.NOTE]上面的示例演示传入电子邮件对象和回调函数，您还可通过直接指定电子邮件属性来直接调用 send 函数。例如：
>
>`````
sendgrid.send({
    to: 'john@contoso.com',
    from: 'anna@contoso.com',
    subject: 'test mail',
    text: 'This is a sample email message.'
});
`````

## <a name="addattachment"> </a>如何：添加附件

可通过在 **files** 属性中指定文件名和路径来将附件添加到邮件中。下面的示例演示如何发送附件：

    sendgrid.send({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.',
        files: [
            {
                filename:     '',           // required only if file.content is used.
                contentType:  '',           // optional
                cid:          '',           // optional, used to specify cid for inline content
                path:         '',           //
                url:          '',           // == One of these three options is required
                content:      ('' | Buffer) //
            }
        ],
    });

> [AZURE.NOTE]在使用 **files** 属性时，必须可通过[fs.readFile](http://nodejs.org/docs/v0.6.7/api/fs.html#fs.readFile) 访问文件。如果要附加的文件托管在 Azure 存储中（如 Blob 容器中），则必须先将该文件复制到本地存储或 Azure 驱动器，然后才能使用 **files** 属性将该文件作为附件发送。

## <a name="usefilters"> </a>如何：使用筛选器启用页脚和跟踪

SendGrid 可通过使用筛选器来提供其他电子邮件功能。可将这些设置添加到电子邮件以启用特定功能（例如，启用单击跟踪、Google 分析、订阅跟踪等）。有关筛选器的完整列表，请参阅[筛选器设置][]。

可使用 **filters** 属性将筛选器应用到邮件。每个筛选器均由一个包含特定于筛选器的设置的哈希指定。以下示例演示了脚注和单击跟踪筛选器：

### 脚注

    var email = new sendgrid.Email({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.'
    });
    
    email.setFilters({
        'footer': {
            'settings': {
                'enable': 1,
                'text/plain': 'This is a text footer.'
            }
        }
    });

    sendgrid.send(email);

### 单击跟踪

    var email = new sendgrid.Email({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.'
    });
    
    email.setFilters({
        'clicktrack': {
            'settings': {
                'enable': 1
            }
        }
    });
    
    sendgrid.send(email);

## <a name="updateproperties"> </a>如何：更新电子邮件属性

可使用 **set*Property*** 替代一些电子邮件属性，或使用 **add*Property*** 追加一些电子邮件属性。例如，可使用以下命令添加更多收件人：

    email.addTo('jeff@contoso.com');

或使用以下命令设置筛选器：

    email.addFilter('footer', 'enable', 1);
    email.addFilter('footer', 'text/html', '<strong>boo</strong>');

有关详细信息，请参阅 [sendgrid-nodejs][]。

## <a name="useservices"> </a>如何：使用其他 SendGrid 服务

SendGrid 提供了基于 Web 的 API，可通过这些 API 从 Azure 应用程序中使用其他 SendGrid 功能。有关完整详细信息，请参阅 [SendGrid API 文档][]。

## <a name="nextsteps"></a>后续步骤

此时，你已了解 SendGrid 电子邮件服务的基础知识，请访问以下链接以了解更多信息。

-   SendGrid Node.js 模块存储库：[sendgrid-nodejs][]
-   SendGrid API 文档：<https://sendgrid.com/docs>
-   面向 Azure 客户的 SendGrid 特惠产品/服务：[http://sendgrid.com/azure.html](https://sendgrid.com/windowsazure.html)

  [后续步骤]: http://www.windowsazure.com/develop/nodejs/how-to-guides/blob-storage/#next-steps
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

  
  
  [special offer]: https://sendgrid.com/windowsazure.html
  
  
  [sendgrid-nodejs]: https://github.com/sendgrid/sendgrid-nodejs
  
  [筛选器设置]: https://sendgrid.com/docs/API_Reference/SMTP_API/apps.html
  [SendGrid API 文档]: https://sendgrid.com/docs
  [基于云的电子邮件服务]: https://sendgrid.com/email-solutions
  [事务性电子邮件传递]: https://sendgrid.com/transactional-email

<!---HONumber=71-->