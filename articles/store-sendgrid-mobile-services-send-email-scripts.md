<properties 
	pageTitle="使用 SendGrid 发送电子邮件 | Microsoft Azure" 
	description="了解如何使用 SendGrid 服务从 Azure 移动服务应用程序发送电子邮件。" 
	services="mobile-services" 
	documentationCenter="" 
	authors="Erikre" 
	manager="sendgrid" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="11/30/2015" 
	wacn.date="03/21/2016"/>


# 使用 SendGrid 从移动服务发送电子邮件

[AZURE.INCLUDE [mobile-service-note-mobile-apps](../includes/mobile-services-note-mobile-apps.md)]

&nbsp;


此主题演示如何将电子邮件功能添加到你的移动服务中。在本教程中，你将添加服务器端脚本以使用 SendGrid 发送电子邮件。完成后，每次插入记录时，你的移动服务即会发送一封电子邮件。

SendGrid 是一项[基于云的电子邮件服务]，该服务提供了可靠的[事务性电子邮件传递]、可缩放性、实时分析以及可用于简化自定义集成的灵活的 API。有关详细信息，请参阅 <http://sendgrid.com>。

本教程将指导你完成以下启用电子邮件功能的基本步骤：

1. [创建 SendGrid 帐户]
2. [添加脚本以发送电子邮件]
3. [插入数据以接收电子邮件]

本教程基于移动服务快速入门。在开始本教程之前，必须先[完成移动服务入门]。

## <a name="sign-up"></a>创建新的 SendGrid 帐户

[AZURE.INCLUDE [sendgrid-sign-up](../includes/sendgrid-sign-up.md)]

## <a name="add-script"></a>注册发送电子邮件的新脚本

1. 登录到 [Azure 经典门户]，单击“移动服务”，然后单击你的移动服务。

2. 在 Azure 经典门户中，单击“数据”选项卡，然后单击“TodoItem”表。

	![][1]

3. 在 **todoitem** 中，单击“脚本”选项卡，然后选择“插入”。
   
	![][2]

	将显示当 **TodoItem** 表中发生插入时所调用的函数。

4. 将 insert 函数替换为以下代码：

        var SendGrid = require('sendgrid').SendGrid;
        
        function insert(item, user, request) {    
            request.execute({
                success: function() {
                    // After the record has been inserted, send the response immediately to the client
                    request.respond();
                    // Send the email in the background
                    sendEmail(item);
                }
            });

            function sendEmail(item) {
                var sendgrid = new SendGrid('**username**', '**password**');       
                
                sendgrid.send({
                    to: '**email-address**',
                    from: '**from-address**',
                    subject: 'New to-do item',
                    text: 'A new to-do was added: ' + item.text
                }, function(success, message) {
                    // If the email failed to send, log it as an error so we can investigate
                    if (!success) {
                        console.error(message);
                    }
                });
            }
        }

5. 将上面脚本中的占位符替换为适当的值：

	- **_username_ 和 _password_**：你在[创建 SendGrid 帐户]中确定的 SendGrid 凭据。

	- **_email-address_**：用于接收电子邮件的地址。在实际应用程序中，你可以使用表来存储和检索电子邮件地址。在测试应用程序时，请仅使用你自己的电子邮件地址。

	- **_from-address_**：用于发送电子邮件的地址。可考虑使用属于你组织的已注册域地址。

     >[AZURE.NOTE]如果你没有注册的域，则可以使用移动服务的域，其格式为 *notifications@_your-mobile-service_.azure-mobile.net*。但是，将忽略发送至你的移动服务域的消息。

6. 单击“保存”按钮。你现在已配置了一个脚本，每当将记录插入 **TodoItem** 表时都会发送电子邮件。

## <a name="insert-data"></a>插入测试数据以接收电子邮件

1. 在客户端应用程序项目中，运行快速入门应用程序。 

	此主题演示快速入门应用程序的 Windows 应用商店版本。

2. 在应用程序中的“插入 TodoItem”内键入文本，然后单击“保存”。

	![][3]

3. 请注意，你将收到一封电子邮件，如以下通知所示。

	![][4]

	恭喜，你已成功配置移动服务，可使用 SendGrid 发送电子邮件。

## <a name="nextsteps"></a>后续步骤

现在，你已了解结合使用 SendGrid 电子邮件服务和移动服务是多么简单，单击下面的链接可了解有关 SendGrid 的详细信息。

-   SendGrid API 文档：<https://sendgrid.com/docs>
-   面向 Azure 客户的 SendGrid 特惠产品/服务：<https://sendgrid.com/windowsazure.html>

<!-- Anchors. -->
[创建 SendGrid 帐户]: #sign-up
[添加脚本以发送电子邮件]: #add-script
[插入数据以接收电子邮件]: #insert-data

<!-- Images. -->
[1]: ./media/store-sendgrid-mobile-services-send-email-scripts/mobile-portal-data-tables.png
[2]: ./media/store-sendgrid-mobile-services-send-email-scripts/mobile-insert-script-push2.png
[3]: ./media/store-sendgrid-mobile-services-send-email-scripts/mobile-quickstart-push1.png
[4]: ./media/store-sendgrid-mobile-services-send-email-scripts/mobile-receive-email.png

<!-- URLs. -->
[完成移动服务入门]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/
[sign up page]: https://sendgrid.com/windowsazure.html
[Multiple User Credentials page]: https://sendgrid.com/credentials
[Azure 经典门户]: https://manage.windowsazure.cn/
[基于云的电子邮件服务]: https://sendgrid.com/email-solutions
[事务性电子邮件传递]: https://sendgrid.com/transactional-email

 

<!---HONumber=Mooncake_0118_2016-->