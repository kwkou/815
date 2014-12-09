<properties title="如何使用 SendGrid 电子邮件服务 (PHP) - Azure" pageTitle="如何使用 SendGrid 电子邮件服务 (PHP) - Azure" metaKeywords="Azure SendGrid, Azure email service, Azure SendGrid PHP, Azure email PHP" description="了解如何在 Azure 上使用 SendGrid 电子邮件服务发送电子邮件。采用 PHP 编写的代码示例。" documentationCenter="PHP" services="" manager="wpickett" editor="mollybos" authors="robmcm" scriptId="" videoId="" />

# 如何通过 PHP 使用 SendGrid 电子邮件服务

本指南演示了如何在 Azure 上使用 SendGrid 电子邮件服务执行常见编程任务。示例是采用 PHP 编写的。
所涉及的方案包括**创建电子邮件**、**发送电子邮件**和**添加附件**。有关 SendGrid 和发送电子邮件的详细信息，请参阅[后续步骤][后续步骤]部分。

## 目录

-   [什么是 SendGrid 电子邮件服务][什么是 SendGrid 电子邮件服务]
-   [创建 SendGrid 帐户][创建 SendGrid 帐户]
-   [从 PHP 应用程序中使用 SendGrid][从 PHP 应用程序中使用 SendGrid]
-   [如何：发送电子邮件][如何：发送电子邮件]
-   [如何：添加附件][如何：添加附件]
-   [如何：使用筛选器启用页脚、跟踪和分析][如何：使用筛选器启用页脚、跟踪和分析]
-   [后续步骤][后续步骤]

## <a name="bkmk_WhatIsSendGrid"> </a>什么是 SendGrid 电子邮件服务？

SendGrid 是一项[基于云的电子邮件服务][基于云的电子邮件服务]，该服务提供了可靠的
[事务性电子邮件传递][事务性电子邮件传递]、可缩放性、实时分析以及可用于简化自定义
集成的灵活的 API。常见 SendGrid 使用方案
包括：

-   自动向客户发送收据
-   管理用于每月向客户发送电子传单和特惠
    产品/服务的通讯组列表
-   收集诸如已阻止的电子邮件和客户响应性等项目的
    实时度量值
-   生成用于帮助确定趋势的报告
-   转发客户查询
-   以电子邮件的形式从应用程序发送通知

有关详细信息，请参阅 <http://sendgrid.com>。

## <a name="bkmk_CreateSendGrid"> </a>创建 SendGrid 帐户

[WACOM.INCLUDE [sendgrid-sign-up](../includes/sendgrid-sign-up.md)]

## <a name="bkmk_UsingSendGridfromPHP"> </a>从 PHP 应用程序中使用 SendGrid

在 Azure PHP 应用程序中使用 SendGrid 时不需要特殊
配置或编码。由于 SendGrid 是一项服务，因此可使用从
本地应用程序访问该服务的方式从云应用程序中
访问该服务。

## <a name="bkmk_HowToSendEmail"> </a>如何：发送电子邮件

可以使用 SMTP 或
由 SendGrid 提供的 Web API 发送电子邮件。

### SMTP API

若要使用 SendGrid SMTP API 发送电子邮件，请使用 *Swift Mailer*，
这是用于从 PHP 应用程序中发送电子邮件的基于组件的库。可以
从 <http://swiftmailer.org/download> 中
下载 *Swift Mailer* 库。利用库发送电子邮件
涉及创建
<span class="auto-style2">Swift\_SmtpTransport</span>、
<span class="auto-style2">Swift\_Mailer</span> 和
<span class="auto-style2">Swift\_Message</span> 类的实例、设置
适当的属性以及调用
<span class="auto-style2">Swift\_Mailer::send</span> 方法。

    <?php
     include_once "lib/swift_required.php";
     /*
      * Create the body of the message (a plain-text and an HTML version).
      * $text is your plain-text email
      * $html is your html version of the email
      * If the reciever is able to view html emails then only the html
      * email will be displayed
      */ 
     $text = "Hi!\nHow are you?\n";
     $html = <<<EOM
           <html>
           <head></head>
           <body>
               <p>Hi!<br>
                   How are you?<br>
               </p>
           </body>
           </html>
           EOM;
     // This is your From email address
     $from = array('someone@example.com' => 'Name To Appear');
     // Email recipients
     $to = array(
           'john@contoso.com'=>'Destination 1 Name',
           'anna@contoso.com'=>'Destination 2 Name'
     );
     // Email subject
     $subject = 'Example PHP Email';

     // Login credentials
     $username = 'yoursendgridusername';
     $password = 'yourpassword';
     
     // Setup Swift mailer parameters
     $transport = Swift_SmtpTransport::newInstance('smtp.sendgrid.net', 587);
     $transport->setUsername($username);
     $transport->setPassword($password);
     $swift = Swift_Mailer::newInstance($transport);

     // Create a message (subject)
     $message = new Swift_Message($subject);

     // attach the body of the email
     $message->setFrom($from);
     $message->setBody($html, 'text/html');
     $message->setTo($to);
     $message->addPart($text, 'text/plain');
     
     // send message 
     if ($recipients = $swift->send($message, $failures))
     {
         // This will let us know how many users received this message
         echo 'Message sent out to '.$recipients.' users';
     }
     // something went wrong =(
     else
     {
         echo "Something went wrong - ";
         print_r($failures);
     }

### Web API

使用 PHP 的 [curl 函数][curl 函数]以通过使用 SendGrid Web API 发送电子邮件。

    <?php

     $url = 'http://sendgrid.com/';
     $user = 'USERNAME';
     $pass = 'PASSWORD'; 

     $params = array(
          'api_user' => $user,
          'api_key' => $pass,
          'to' => 'john@contoso.com',
          'subject' => 'testing from curl',
          'html' => 'testing body',
          'text' => 'testing body',
          'from' => 'anna@sendgrid.com',
       );
       
     $request = $url.'api/mail.send.json';
     
     // Generate curl request
     $session = curl_init($request);
     
     // Tell curl to use HTTP POST
     curl_setopt ($session, CURLOPT_POST, true);
     
     // Tell curl that this is the body of the POST
     curl_setopt ($session, CURLOPT_POSTFIELDS, $params);
     
     // Tell curl not to return headers, but do return the response
     curl_setopt($session, CURLOPT_HEADER, false);
     curl_setopt($session, CURLOPT_RETURNTRANSFER, true);
     
     // obtain response
     $response = curl_exec($session);
     curl_close($session);
     
     // print everything out
     print_r($response);

SendGrid 的 Web API 与 REST API 非常相似，尽管它
不是真正的 RESTful API，这是因为在大多数调用中，GET 和 POST 谓词
是可互换使用的。

## <a name="bkmk_HowToAddAttachment"> </a>如何：添加附件

### SMTP API

使用 SMTP API 发送附件时将涉及在用于通过 Swift Mailer 发送电子邮件的示例脚本中额外添加一行
代码。

    <?php
     include_once "lib/swift_required.php";
     /*
      * Create the body of the message (a plain-text and an HTML version).
      * $text is your plain-text email
      * $html is your html version of the email
      * If the reciever is able to view html emails then only the html
      * email will be displayed
      */
     $text = "Hi!\nHow are you?\n";
      $html = <<<EOM
          <html>
          <head></head>
          <body>
             <p>Hi!<br>
                How are you?<br>
             </p>
          </body>
          </html>
     EOM;

     // This is your From email address
     $from = array('someone@example.com' => 'Name To Appear');
     
     // Email recipients
     $to = array(
          'john@contoso.com'=>'Destination 1 Name',
          'anna@contoso.com'=>'Destination 2 Name'
     );
     // Email subject
     $subject = 'Example PHP Email';
     
     // Login credentials
     $username = 'yoursendgridusername';
     $password = 'yourpassword';
     
     // Setup Swift mailer parameters
     $transport = Swift_SmtpTransport::newInstance('smtp.sendgrid.net', 587);
     $transport->setUsername($username);
     $transport->setPassword($password);
     $swift = Swift_Mailer::newInstance($transport);
     
     // Create a message (subject)
     $message = new Swift_Message($subject);
     
     // attach the body of the email
     $message->setFrom($from);
     $message->setBody($html, 'text/html');
     $message->setTo($to);
     $message->addPart($text, 'text/plain');
     $message->attach(Swift_Attachment::fromPath("path\to\file")->setFileName("file_name"));
     
     // send message 
     if ($recipients = $swift->send($message, $failures))
     {
          // This will let us know how many users received this message
          echo 'Message sent out to '.$recipients.' users';
     }
     // something went wrong =(
     else
     {
          echo "Something went wrong - "
          print_r($failures);
     }

额外的代码行如下所示：

     $message->attach(Swift_Attachment::fromPath("path\to\file")->setFileName('file_name'));

该代码行对
<span class="auto-style2">Swift\_Message</span> 对象调用 attach 方法，并对
<span class="auto-style2">Swift\_Attachment</span> 类使用静态
方法 <span class="auto-style2">fromPath</span>，以获取文件并
将其附加到邮件。

### Web API

使用 Web API 发送附件与使用 Web API 发送
电子邮件非常相似。但请注意，在以下示例中，
参数数组必须包含此元素：

     'files['.$fileName.']' => '@'.$filePath.'/'.$fileName

    <?php

     $url = 'http://sendgrid.com/';
     $user = 'USERNAME';
     $pass = 'PASSWORD';
     
     $fileName = 'myfile';
     $filePath = dirname(__FILE__);

     $params = array(
         'api_user' => $user,
         'api_key' => $pass,
         'to' =>'john@contoso.com',
         'subject' => 'test of file sends',
         'html' => '<p> the HTML </p>',
         'text' => 'the plain text',
         'from' => 'anna@sendgrid.com',
         'files['.$fileName.']' => '@'.$filePath.'/'.$fileName
     );
     
     print_r($params);
     
     $request = $url.'api/mail.send.json';
     
     // Generate curl request
     $session = curl_init($request);
     
     // Tell curl to use HTTP POST
     curl_setopt ($session, CURLOPT_POST, true);
     
     // Tell curl that this is the body of the POST
     curl_setopt ($session, CURLOPT_POSTFIELDS, $params);
     
     // Tell curl not to return headers, but do return the response
     curl_setopt($session, CURLOPT_HEADER, false);
     curl_setopt($session, CURLOPT_RETURNTRANSFER, true);
     
     // obtain response
     $response = curl_exec($session);
     curl_close($session);
     
     // print everything out
     print_r($response);

## <a name="bkmk_HowToUseFilters"> </a>如何：使用筛选器启用页脚、跟踪和分析

SendGrid 可通过使用
“筛选器”来提供其他电子邮件功能。这些功能是一些可添加到电子邮件中以
启用特定功能（如启用单击跟踪、Google 分
析、订阅跟踪等）的设置。

可使用 filters 属性将筛选器应用于邮件。每个
筛选器均由一个包含特定于筛选器的设置的哈希指定。
以下示例将启用页脚筛选器并
指定将附加到电子邮件底部的短信：

    <?php
     /*
      * This example is used for Swift Mailer V4
      */
     include "./lib/swift_required.php";
     include 'SmtpApiHeader.php';
     
     $hdr = new SmtpApiHeader();
     // The list of addresses this message will be sent to
     // [This list is used for sending multiple emails using just ONE request to 
     SendGrid]
     $toList = array('john@contoso.com', 'anna@contoso.com');
     
     // Specify the names of the recipients
     $nameList = array('Name 1', 'Name 2');
     
     // Used as an example of variable substitution
     $timeList = array('4 PM', '5 PM');
     
     // Set all of the above variables
     $hdr->addTo($toList);
     $hdr->addSubVal('-name-', $nameList);
     $hdr->addSubVal('-time-', $timeList);
     
     // Specify that this is an initial contact message
     $hdr->setCategory("initial");
     
     // You can optionally setup individual filters here, in this example, we have 
     enabled the footer filter
     $hdr->addFilterSetting('footer', 'enable', 1);
     $hdr->addFilterSetting('footer', "text/plain", "Thank you for your business");
     
     // The subject of your email
     $subject = 'Example SendGrid Email';
     
     // Where is this message coming from. For example, this message can be from support@yourcompany.com, info@yourcompany.com
     $from = array('someone@example.com' =&gt; 'Name Of Your Company');
     
     // If you do not specify a sender list above, you can specifiy the user here. If 
     // a sender list IS specified above, this email address becomes irrelevant.
     $to = array('john@contoso.com'=&gt;'Personal Name Of Recipient');
     
     # Create the body of the message (a plain-text and an HTML version). 
     # text is your plain-text email 
     # html is your html version of the email
     # if the receiver is able to view html emails then only the html
     # email will be displayed
     
     /*
     * Note the variable substitution here =)
     */
     $text = <<<EOM 
     Hello -name-,
     Thank you for your interest in our products. We have set up an appointment to call you at -time- EST to discuss your needs in more detail.
     Regards,
     Fred
     EOM;
     
     $html = <<<EOM
     < html> 
     <head></head>
     <body>
     <p>Hello -name-,<br>
     Thank you for your interest in our products. We have set up an appointment
     to call you at -time- EST to discuss your needs in more detail.
     
     Regards,
     
     Fred, How are you?<br>
     </p>
     </body>
     < /html>
     EOM;
     
     // Your SendGrid account credentials
     $username = 'sendgridusername@yourdomain.com';
     $password = 'example';
     
     // Create new swift connection and authenticate
     $transport = Swift_SmtpTransport::newInstance('smtp.sendgrid.net', 25);
     $transport ->setUsername($username);
     $transport ->setPassword($password);
     $swift = Swift_Mailer::newInstance($transport);
     
     // Create a message (subject)
     $message = new Swift_Message($subject);
     
     // add SMTPAPI header to the message
     // *****IMPORTANT NOTE*****
     // SendGrid's asJSON function escapes characters. If you are using Swift 
     Mailer's
     // PHP Mailer functions, the getTextHeader function will also escape characters.
     // This can cause the filter to be dropped.
     $headers = $message->getHeaders();
     $headers->addTextHeader('X-SMTPAPI', $hdr->asJSON());
     
     // attach the body of the email
     $message->setFrom($from);
     $message->setBody($html, 'text/html');
     $message->setTo($to);
     $message->addPart($text, 'text/plain');
     
     // send message
     if ($recipients = $swift->send($message, $failures))
     {
     // This will let us know how many users received this message
     // If we specify the names in the X-SMTPAPI header, then this will always be 1.
     echo 'Message sent out to '.$recipients.' users';
     }

     // something went wrong =(
     else
     {
     echo "Something went wrong - ";
     print_r($failures);
     }

## <a name="bkmk_NextSteps"> </a> 后续步骤

在了解 SendGrid 电子邮件服务的基础知识后，请访问以下链接
以了解更多信息。

-   SendGrid 文档：<https://sendgrid.com/docs>
-   面向 Azure 客户的 SendGrid 特惠产品/服务：<http://sendgrid.com/azure.html>

  [后续步骤]: #bkmk_NextSteps
  [什么是 SendGrid 电子邮件服务]: #bkmk_WhatIsSendGrid
  [创建 SendGrid 帐户]: #bkmk_CreateSendGrid
  [从 PHP 应用程序中使用 SendGrid]: #bkmk_UsingSendGridfromPHP
  [如何：发送电子邮件]: #bkmk_HowToSendEmail
  [如何：添加附件]: #bkmk_HowToAddAttachment
  [如何：使用筛选器启用页脚、跟踪和分析]: #bkmk_HowToUseFilters
  [基于云的电子邮件服务]: http://sendgrid.com/solutions
  [事务性电子邮件传递]: http://sendgrid.com/transactional-email
  [curl 函数]: http://php.net/curl
