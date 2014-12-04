<properties linkid="store-requestform-preview" urlDisplayName="Request Azure Store Integration" pageTitle="How to Send Email Using SendGrid from Java in an Azure Deployment" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="How to Send Email Using SendGrid from Java in an Azure Deployment" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" videoId="" scriptId="" />

# 如何在 Azure 部署中通过 Java 使用 SendGrid 发送电子邮件

以下示例演示如何能够使用 SendGrid 从在 Azure 中托管的网页上发送电子邮件。所得的应用程序将提示用户输入电子邮件值，如以下屏幕快照中所示。

![“电子邮件”窗体][“电子邮件”窗体]

所得的电子邮件将与以下屏幕快照类似。

![电子邮件][电子邮件]

你将需要执行以下操作来使用本主题中的代码：

1.  获取 javax.mail JAR（例如从 <http://www.oracle.com/technetwork/java/javamail/index.html> 获取）。
2.  将这些 JAR 添加到你的 Java 生成路径。
3.  如果你使用 Eclipse 创建此 Java 应用程序，则可使用 Eclipse 的部署程序集功能在应用程序部署文件 (WAR) 中加入 SendGrid 库。如果你不使用 Eclipse 创建此 Java 应用程序，请确保将这些库包含在与你的 Java 应用程序相同的 Azure 角色中，并将其添加到你的应用程序的类路径下。

你还必须有自己的 SendGrid 用户名和密码，才能发送电子邮件。若要开始使用 SendGrid，请参阅[如何通过 Java 使用 SendGrid 发送电子邮件][如何通过 Java 使用 SendGrid 发送电子邮件]。

此外，强烈建议你熟悉[在 Eclipse 中创建用于 Azure 的 Hello World 应用程序][在 Eclipse 中创建用于 Azure 的 Hello World 应用程序]上的信息，如果不使用 Eclipse，则强烈建议你熟悉在 Azure 中托管 Java 应用程序的其他方法。

## 创建用于发送电子邮件的 Web 窗体

以下代码演示如何创建 Web 窗体以检索用于发送电子邮件的用户数据。用于此内容的 JSP 文件的名称为 **emailform.jsp**。

    <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
        pageEncoding="ISO-8859-1" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
    <title>Email form</title>
    </head>
    <body>
     <p>Fill in all fields and click <b>Send this email</b>.</p>
     <br/>
      <form action="sendemail.jsp" method="post">
       <table>
         <tr>
           <td>To:</td>
           <td><input type="text" size=50 name="emailTo">
           </td>
         </tr>
         <tr>
           <td>From:</td>
           <td><input type="text" size=50 name="emailFrom">
           </td>
         </tr>
         <tr>
           <td>Subject:</td>
           <td><input type="text" size=100 name="emailSubject" value="My email subject">
           </td>
         </tr>
         <tr>
           <td>Text:</td>
           <td><input type="text" size=400 name="emailText" value="Hello,<p>This is my message.</p>Thank you." />
           </td>
         </tr>
         <tr>
           <td>SendGrid user name:</td>
           <td><input type="text" name="sendGridUser">
           </td>
         </tr>
         <tr>
           <td>SendGrid password:</td>
           <td><input type="password" name="sendGridPassword">
           </td>
         </tr>
         <tr>
           <td colspan=2><input type="submit" value="Send this email">
           </td>
         </tr>
       </table>
     </form>
     <br/>
    </body>
    </html>

## 创建用于发送电子邮件的代码

以下代码创建电子邮件并发送它，在填写 emailform.jsp 中的窗体时将调用这段代码。用于此内容的 JSP 文件的名称为 **sendemail.jsp**。

    <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
        pageEncoding="ISO-8859-1" import="javax.activation.*, javax.mail.*, javax.mail.internet.*, java.util.Date, java.util.Properties" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
    <title>Email processing happens here</title>
    </head>
    <body>
        <b>This is my send mail page.</b><p/>
     <%
     
     final String sendGridUser = request.getParameter("sendGridUser");
     final String sendGridPassword = request.getParameter("sendGridPassword");
     
     class SMTPAuthenticator extends Authenticator
     {
       public PasswordAuthentication getPasswordAuthentication()
       {
            String username = sendGridUser;
            String password = sendGridPassword;
          
            return new PasswordAuthentication(username, password);   
       }
     }
     try
     {
         
         // The SendGrid SMTP server.
         String SMTP_HOST_NAME = "smtp.sendgrid.net";

         Properties properties;
        
         properties = new Properties();
         
         // Specify SMTP values.
         properties.put("mail.transport.protocol", "smtp");
         properties.put("mail.smtp.host", SMTP_HOST_NAME);
         properties.put("mail.smtp.port", 587);
         properties.put("mail.smtp.auth", "true");
         
         // Display the email fields entered by the user. 
         out.println("Value entered for email Subject: " + request.getParameter("emailSubject") + "<br/>");        
         out.println("Value entered for email      To: " + request.getParameter("emailTo") + "<br/>");
         out.println("Value entered for email    From: " + request.getParameter("emailFrom") + "<br/>");
         out.println("Value entered for email    Text: " + "<br/>" + request.getParameter("emailText") + "<br/>");

         // Create the authenticator object.
         Authenticator authenticator = new SMTPAuthenticator();
         
         // Create the mail session object.
         Session mailSession;
         mailSession = Session.getDefaultInstance(properties, authenticator);
         
         // Display debug information to stdout, useful when using the
         // compute emulator during development.
         mailSession.setDebug(true);

         // Create the message and message part objects.
         MimeMessage message;
         Multipart multipart;
         MimeBodyPart messagePart; 
         
         message = new MimeMessage(mailSession);
         
         multipart = new MimeMultipart("alternative");
         messagePart = new MimeBodyPart();
         messagePart.setContent(request.getParameter("emailText"), "text/html");
         multipart.addBodyPart(messagePart);            

         // Specify the email To, From, Subject and Content. 
         message.setFrom(new InternetAddress(request.getParameter("emailFrom")));
         message.addRecipient(Message.RecipientType.TO, new InternetAddress(request.getParameter("emailTo")));
         message.setSubject(request.getParameter("emailSubject")); 
         message.setContent(multipart);
         
         // Uncomment the following if you want to add a footer.
         // message.addHeader("X-SMTPAPI", "{\"filters\": {\"footer\": {\"settings\": {\"enable\":1,\"text/html\": \"<html>This is my <b>email footer</b>.</html>\"}}}}");

         // Uncomment the following if you want to enable click tracking.
         // message.addHeader("X-SMTPAPI", "{\"filters\": {\"clicktrack\": {\"settings\": {\"enable\":1}}}}");
         
         Transport transport;
         transport = mailSession.getTransport();
         // Connect the transport object.
         transport.connect();
         // Send the message.
         transport.sendMessage(message,  message.getRecipients(Message.RecipientType.TO));
         // Close the connection.
         transport.close();
     
        out.println("<p>Email processing completed.</p>");
         
     }
     catch (Exception e)
     {
         out.println("<p>Exception encountered: " + 
                            e.getMessage()     +
                            "</p>");   
     }
    %>

    </body>
    </html>

除了发送电子邮件外，emailform.jsp 还向用户提供发送结果；以下屏幕快照就是一个例子：

![发送邮件结果][发送邮件结果]

## 后续步骤

将你的应用程序部署到计算模拟器，然后在浏览器中运行 emailform.jsp，在窗体中输入值，单击“发送此电子邮件”，然后在 sendemail.jsp 中查看结果。

提供这段代码是为了向你演示如何在 Azure 上通过 Java 使用 SendGrid。在生产中部署到 Azure 之前，你可能希望添加更多错误处理功能或其他功能。例如：

-   你可能会使用 Azure 存储 Blob 或 SQL Database 存储电子邮件地址和电子邮件，而不使用 Web 窗体。有关在 Java 中使用 Azure 存储 Blob 的信息，请参阅[如何从 Java 使用 Blob 存储服务][如何从 Java 使用 Blob 存储服务]。有关在 Java 中使用 SQL Database 的信息，请参阅[在 Java 中使用 SQL Database][在 Java 中使用 SQL Database]。
-   你可能会使用`RoleEnvironment.getConfigurationSettings` 从部署的配置设置中检索 SendGrid 用户名和密码，而不使用 Web 窗体检索这些值。有关`RoleEnvironment` 类的信息，请参阅[在 JSP 中使用 Azure 服务运行时库][在 JSP 中使用 Azure 服务运行时库]以及 <http://dl.windowsazure.com/javadoc> 上的 Azure 服务运行时程序包文档。
-   有关在 Java 中使用 SendGrid 的详细信息，请参阅[如何通过 Java 使用 SendGrid 发送电子邮件][如何通过 Java 使用 SendGrid 发送电子邮件]。

  [“电子邮件”窗体]: ./media/store-sendgrid-java-how-to-send-email-example/SendGridJavaEmailform.jpg
  [电子邮件]: ./media/store-sendgrid-java-how-to-send-email-example/SendGridJavaEmailSent.jpg
  [如何通过 Java 使用 SendGrid 发送电子邮件]: ../store-sendgrid-java-how-to-send-email
  [在 Eclipse 中创建用于 Azure 的 Hello World 应用程序]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh690944
  [发送邮件结果]: ./media/store-sendgrid-java-how-to-send-email-example/SendGridJavaResult.jpg
  [如何从 Java 使用 Blob 存储服务]: http://www.windowsazure.com/zh-cn/develop/java/how-to-guides/blob-storage/
  [在 Java 中使用 SQL Database]: http://www.windowsazure.com/zh-cn/develop/java/how-to-guides/using-sql-azure-in-java/
  [在 JSP 中使用 Azure 服务运行时库]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh690948
