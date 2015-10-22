<properties 
	pageTitle="如何从 Twilio (PHP) 发起电话呼叫 | Microsoft Azure" 
	description="了解如何在 Azure 中使用 Twilio API 服务发起电话呼叫和发送短信。示例用于 PHP 应用程序。" 
	documentationCenter="php" 
	services="" 
	authors="devinrader" 
	manager="twilio" 
	editor="mollybos"/>

<tags 
	ms.service="multiple" 
	ms.date="11/25/2014" 
	wacn.date="10/3/2015"/>

# 如何在 Azure 的 PHP 应用程序中使用 Twilio 发起电话呼叫 

以下示例演示了如何使用 Twilio 从 Azure 中托管的 PHP 网页发起呼叫。生成的应用程序将提示用户输入电话呼叫值，如以下屏幕截图中所示。

![使用 Twilio 和 PHP 的 Azure 呼叫窗体][twilio_php]

你将需要执行以下操作来使用本主题中的代码：

1. 获取 Twilio 帐户和身份验证令牌。若要开始使用 Twilio，请在 [http://www.twilio.com/pricing][twilio_pricing] 上评估定价。您可以在 [https://www.twilio.com/try-twilio][try_twilio] 上注册试用帐户。有关 Twilio 提供的 API 的信息，请参阅 [http://www.twilio.com/api][twilio_api]。
2. 获取用于 PHP 的 Twilio 库。您可以从 Github ([https://github.com/twilio/twilio-php][twilio_php_github]) 中下载该库，或将其作为 PEAR 包安装。有关详细信息，请参阅 [https://github.com/twilio/twilio-php/blob/master/README.md][twilio_github_readme]。
3. 安装 Azure SDK for PHP。有关该 SDK 的概述及其安装说明，请参阅[设置 Azure SDK for PHP][setup_php_sdk]。

## 创建用于发起呼叫的 Web 窗体

以下 HTML 代码演示了如何生成检索用于发起呼叫的用户数据的网页 (**callform.html**)：

    <html>
	<head>
		<title>Automated call form</title>
	</head>
	<body>
	<h1>Automated Call Form</h1>
 	<p>Fill in all fields and click <b>Make this call</b>.</p>
  	<form action="makecall.php" method="post">
   	<table>
     	<tr>
       		<td>To:</td>
       		<td><input type="text" size=50 name="callTo" value=""></td>
     	</tr>
     	<tr>
       		<td>From:</td>
       		<td><input type="text" size=50 name="callFrom" value=""></td>
     	</tr>
     	<tr>
       		<td>Call message:</td>
       		<td><input type="text" size=100 name="callText" value="Hello. This is the call text. Good bye." /></td>
     	</tr>
     	<tr>
       		<td colspan=2><input type="submit" value="Make this call"></td>
     	</tr>
   	</table>
 	</form>
 	<br/>
	</body>
	</html>

## 创建用于发起呼叫的代码
以下代码演示如何生成在用户提交 **callform.html** 显示的窗体时调用的网页 (**makecall.php**)。下面显示的代码将创建呼叫消息和生成呼叫。（使用 Twilio 帐户和身份验证令牌，而不是分配给以下代码中的 **$sid** 和 **$token** 的占位符值）。

    <html>
	<head><title>Making call...</title></head>
	<body>
	<p>Your call is being made.</p>

	<?php
	require_once 'Services/Twilio.php';

	$sid = "your_account_sid";
	$token = "your_authentication_token";

	$from_number = $_POST['callFrom']; // Calls must be made from a registered Twilio number.
	$to_number = $_POST['callTo'];
	$message = $_POST['callText'];

	$client = new Services_Twilio($sid, $token, "2010-04-01");

	$call = $client->account->calls->create(
		$from_number, 
		$to_number,
  		'http://twimlets.com/message?Message='.urlencode($message)
	);

	echo "Call status: ".$call->status."<br />";
	echo "URI resource: ".$call->uri."<br />";
	?>
	</body>
	</html>

除了发起呼叫外，**makecall.php** 还可显示一些呼叫元数据（如以下屏幕截图中所示）。有关呼叫元数据的详细信息，请参阅 [https://www.twilio.com/docs/api/rest/call#instance-properties][twilio_call_properties]。

![使用 Twilio 和 PHP 的 Azure 呼叫响应][twilio_php_response]

## 运行应用程序
下一步是将应用程序部署到 Azure 网站。下列文章包含创建网站以及使用 Git、FTP 或 WebMatrix 部署代码的信息（尽管并非每篇文章中的所有信息都是相关的）：

* <!--[-->创建 PHP-MySQL Azure 网站并使用 Git 进行部署<!--][website-git]-->
* <!--[-->创建 PHP-MySQL Azure 网站并使用 FTP 进行部署<!--][website-ftp]-->
* <!--[-->使用 WebMatrix 创建和部署 PHP-MySQL Azure 网站<!--][website-webmatrix]-->

## 后续步骤
提供此代码是为了向你演示在 Azure 上通过 PHP 使用 Twilio 的基本功能。在生产中部署到 Azure 之前，你可能希望添加更多错误处理功能或其他功能。例如：

* 您可以使用 Azure 存储 Blob 或 SQL 数据库存储电话号码和呼叫文本，而不使用 Web 窗体。有关通过 PHP 使用 Azure 存储 Blob 的信息，请参阅[在 PHP 应用程序中使用 Azure 存储空间][howto_blob_storage_php]。有关通过 PHP 使用 SQL 数据库的信息，请参阅[在 PHP 应用程序中使用 SQL 数据库][howto_sql_azure_php]。
* **makecall.php** 代码使用 Twilio 提供的 URL ([http://twimlets.com/message][twimlet_message_url]) 提供了一个 Twilio 标记语言 (TwiML) 响应，指示 Twilio 如何继续进行呼叫。例如，返回的 TwiML 可能包含 `<Say>` 谓词，该谓词生成了与呼叫接收人的谈话的文本。您可以构建自己的服务来响应 Twilio 的请求，而不使用 Twilio 提供的 URL；有关详细信息，请参阅[如何通过 PHP 使用 Twilio 实现语音和 SMS 功能][howto_twilio_voice_sms_php]。有关 TwiML 的详细信息可在 [http://www.twilio.com/docs/api/twiml][twiml] 上找到，有关 `<Say>` 和其他 Twilio 谓词的信息可在 [http://www.twilio.com/docs/api/twiml/say][twilio_say] 上找到。
* 阅读 [https://www.twilio.com/docs/security][twilio_docs_security] 上的 Twilio 安全准则。

有关 Twilio 的更多信息，请参阅 [https://www.twilio.com/docs][twilio_docs]。

## 另请参阅
* [如何通过 PHP 使用 Twilio 实现语音和 SMS 功能](/documentation/articles/partner-twilio-php-how-to-use-voice-sms)

[twilio_pricing]: http://www.twilio.com/pricing
[try_twilio]: http://www.twilio.com/try-twilio
[twilio_api]: http://www.twilio.com/api
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#
[twilio_php]: https://github.com/twilio/twilio-php
[twilio_github_readme]: https://github.com/twilio/twilio-php/blob/master/README.md
[setup_php_sdk]: http://azurephp.interoperabilitybridges.com/articles/setup-the-windows-azure-sdk-for-php
[twimlet_message_url]: http://twimlets.com/message
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api_service]: http://api.twilio.com
[build_php_azure_app]: http://azurephp.interoperabilitybridges.com/articles/build-and-deploy-a-windows-azure-php-application
[howto_twilio_voice_sms_php]: /documentation/articles/partner-twilio-php-how-to-use-voice-sms
[howto_blob_storage_php]: http://www.windowsazure.cn/documentation/articles/storage-php-how-to-use-blobs/
[howto_sql_azure_php]: http://www.windowsazure.cn/documentation/articles/sql-database-php-how-to-use/
[twilio_call_properties]: https://www.twilio.com/docs/api/rest/call#instance-properties
[twilio_docs_security]: http://www.twilio.com/docs/security
[twilio_docs]: http://www.twilio.com/docs
[twilio_say]: http://www.twilio.com/docs/api/twiml/say
[ssl_validation]: http://readthedocs.org/docs/twilio-php/en/latest/usage/rest.html
[twilio_php]: ./media/partner-twilio-php-make-phone-call/WA_TwilioPHPCallForm.jpg
[twilio_php_response]: ./media/partner-twilio-php-make-phone-call/WA_TwilioPHPMakeCall.jpg
[website-git]: https://www.windowsazure.cn/develop/php/tutorials/website-w-mysql-and-git/
[website-ftp]: https://www.windowsazure.cn/develop/php/tutorials/website-w-mysql-and-ftp/
[website-webmatrix]: https://www.windowsazure.cn/develop/php/tutorials/website-w-mysql-and-webmatrix/
[twilio_php_github]: https://github.com/twilio/twilio-php

<!---HONumber=71-->