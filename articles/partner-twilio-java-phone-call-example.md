<properties linkid="develop-java-how-to-twilio-phone-call" urlDisplayName="How to Make a Phone Call from Twilio in Java" pageTitle="How to Make a phone call from Twilio (Java) - Azure" metaKeywords="Azure Twilio call, Twilio call website, Azure Twilio Java" description="Learn how to make a phone call from a web page using Twilio in a Java application on Azure." metaCanonical="" services="" documentationCenter="Java" title="How to Make a Phone Call Using Twilio in a Java Application on Azure" authors="robmcm" videoId="" scriptId="" solutions="" manager="wpickett" editor="mollybos" />

# 如何在 Azure 的 Java 应用程序中使用 Twilio 发起电话呼叫

以下示例演示了如何使用 Twilio 从 Azure 中托管的网页发起呼叫。生成的应用程序将提示用户输入电话呼叫值，如以下屏幕截图中所示。

![使用 Twilio 和 Java 的 Azure 呼叫窗体][使用 Twilio 和 Java 的 Azure 呼叫窗体]

你将需要执行以下操作来使用本主题中的代码：

1.  获取 Twilio 帐户和身份验证令牌。若要开始使用 Twilio，请在 [http://www.twilio.com/pricing][twilio_pricing] 上计算价格。你可以在 [https://www.twilio.com/try-twilio][try_twilio] 上注册。有关 Twilio 提供的 API 的信息，请参阅 [http://www.twilio.com/api][twilio_api]。
2.  使用 Twilio 验证你的电话号码是否为拨出呼叫方 ID。有关如何验证电话号码的信息，请参阅 [https://www.twilio.com/user/account/phone-numbers/verified#][verify_phone]。作为使用现有号码的替代方法，你可以购买 Twilio 电话号码。
    在本示例中，请将经验证的电话号码用作 callform.jsp（稍后介绍）的“呼叫方”值。
3.  获取 Twilio JAR。在 [https://github.com/twilio/twilio-java][twilio_java_github] 上，你可以下载 GitHub 源并创建自己的 JAR，或者下载预生成的 JAR（带有或不带依赖项）。
    本主题中的代码使用预生成的 TwilioJava-3.3.8-with-dependencies JAR 编写。
4.  将 JAR 添加到 Java 生成路径。
5.  如果你使用 Eclipse 创建此 Java 应用程序，请使用 Eclipse 的部署程序集功能将 Twilio JAR 包含在你的应用程序部署文件 (WAR) 中。如果你不使用 Eclipse 创建此 Java 应用程序，请确保将 Twilio JAR 包含在与你的 Java 应用程序相同的 Azure 角色中，并将其添加到你的应用程序的类路径下。
6.  确保 cacerts 密钥库包含带有 MD5 指纹 67:CB:9D:C0:13:24:8A:82:9B:B2:17:1E:D1:1B:EC:D4（序列号为 35:DE:F4:CF，SHA1 指纹为 D2:32:09:AD:23:D3:14:23:21:74:E4:0D:7F:9D:62:13:97:86:63:3A）的 Equifax 安全证书颁发机构证书。这是 [https://api.twilio.com][twilio_api_service]<https://api.twilio.com></a> 服务的证书颁发机构 (CA) 证书，在使用 Twilio API 时调用。有关将此 CA 证书添加到 JDK 的 cacert 存储的信息，请参阅[将证书添加到 Java CA 证书存储][将证书添加到 Java CA 证书存储]。

此外，如果你没有使用 Eclipse，我们强烈建议你熟悉[使用 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）创建 Hello World 应用程序][使用 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）创建 Hello World 应用程序]中的信息或熟悉用于在 Azure 中托管 Java 应用程序的其他方法。

## 创建用于发起呼叫的 Web 窗体

以下代码演示了如何创建 Web 窗体来检索用于发起呼叫的用户数据。在本示例中，将创建名为 **TwilioCloud** 的新的动态 Web 项目，并添加 **callform.jsp** 作为 JSP 文件。

    <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
        pageEncoding="ISO-8859-1" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
    <title>Automated call form</title>
    </head>
    <body>
     <p>Fill in all fields and click <b>Make this call</b>.</p>
     <br/>
      <form action="makecall.jsp" method="post">
       <table>
         <tr>
           <td>To:</td>
           <td><input type="text" size=50 name="callTo" value="" />
           </td>
         </tr>
         <tr>
           <td>From:</td>
           <td><input type="text" size=50 name="callFrom" value="" />
           </td>
         </tr>
         <tr>
           <td>Call message:</td>
           <td><input type="text" size=400 name="callText" value="Hello. This is the call text. Good bye." />
           </td>
         </tr>
         <tr>
           <td colspan=2><input type="submit" value="Make this call" />
           </td>
         </tr>
       </table>
     </form>
     <br/>
    </body>
    </html>

## 创建用于发起呼叫的代码

以下代码在用户填写 callform.jsp 显示的窗体后调用，用于创建呼叫消息并生成呼叫。在本示例中，JSP 文件被命名为 **makecall.jsp** 并添加到 **TwilioCloud** 项目。（使用 Twilio 帐户和身份验证令牌，而不是分配给以下代码中的 **accountSID** 和 **authToken** 的占位符值）。

    <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    import="java.util.*"
    import="com.twilio.*"
    import="com.twilio.sdk.*"
    import="com.twilio.sdk.resource.factory.*"
    import="com.twilio.sdk.resource.instance.*"
    pageEncoding="ISO-8859-1" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
    <title>Call processing happens here</title>
    </head>
    <body>
        <b>This is my make call page.</b><p/>
     <%
    try 
    {
         // Use your account SID and authentication token instead
         // of the placeholders shown here.
         String accountSID = "your_twilio_account";
         String authToken = "your_twilio_authentication_token";
     
         // Instantiate an instance of the Twilio client.     
         TwilioRestClient client;
         client = new TwilioRestClient(accountSID, authToken);

         // Retrieve the account, used later to retrieve the CallFactory.
         Account account = client.getAccount();

         // Display the client endpoint. 
         out.println("<p>Using Twilio endpoint " + client.getEndpoint() + ".</p>");
     
         // Display the API version.
         String APIVERSION = TwilioRestClient.DEFAULT_VERSION;
         out.println("<p>Twilio client API version is " + APIVERSION + ".</p>");

         // Retrieve the values entered by the user.
         String callTo = request.getParameter("callTo");  
         // The Outgoing Caller ID, used for the From parameter,
         // must have previously been verified with Twilio.
         String callFrom = request.getParameter("callFrom");
         String userText = request.getParameter("callText");
     
         // Replace spaces in the user's text with '%20', 
         // to make the text suitable for a URL.
         userText = userText.replace(" ", "%20");
     
         // Create a URL using the Twilio message and the user-entered text.
         String Url="http://twimlets.com/message";
         Url = Url + "?Message%5B0%5D=" + userText;
     
         // Display the message URL.
         out.println("<p>");
         out.println("The URL is " + Url);
         out.println("</p>");

         // Place the call From, To and URL values into a hash map. 
         HashMap<String, String> params = new HashMap<String, String>();
         params.put("From", callFrom);
         params.put("To", callTo);
         params.put("Url", Url);
     
         CallFactory callFactory = account.getCallFactory();
         Call call = callFactory.create(params);
         out.println("<p>Call status: " + call.getStatus()  + "</p>"); 
    } 
    catch (TwilioRestException e) 
    {
        out.println("<p>TwilioRestException encountered: " + e.getMessage() + "</p>");
        out.println("<p>StackTrace: " + e.getStackTrace().toString() + "</p>");
    }
    catch (Exception e) 
    {
        out.println("<p>Exception encountered: " + e.getMessage() + "");
        out.println("<p>StackTrace: " + e.getStackTrace().toString() + "</p>");
    }
    %>
    </body>
    </html>

除了发起呼叫外，makecall.jsp 还可显示 Twilio 终结点、API 版本和呼叫状态。如以下屏幕截图所示：

![使用 Twilio 和 Java 的 Azure 呼叫响应][使用 Twilio 和 Java 的 Azure 呼叫响应]

## 运行应用程序

下面是运行应用程序的概要步骤；这些步骤的详细信息可在[使用 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）创建 Hello World 应用程序][使用 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）创建 Hello World 应用程序]上找到。

1.  将 TwilioCloud WAR 导出到 Azure **approot** 文件夹。
2.  修改 **startup.cmd** 以解压缩 TwilioCloud WAR。
3.  针对计算模拟器编译应用程序。
4.  在计算模拟器中启动部署。
5.  打开浏览器，然后运行 **<http://localhost:8080/TwilioCloud/callform.jsp>**。
6.  在窗体中输入值，单击“发起此呼叫”，然后查看 makecall.jsp 中的结果。

准备好部署到 Azure 之后，请针对云部署重新进行编译，部署到 Azure，然后在浏览器中运行 http://*your_hosted_name*.cloudapp.net/TwilioCloud/callform.jsp（将 *your\_hosted\_name* 替换为你的值）。

## 后续步骤

提供此代码是为了向你演示在 Azure 上通过 Java 使用 Twilio 的基本功能。在生产中部署到 Azure 之前，你可能希望添加更多错误处理功能或其他功能。例如：

-   你可以使用 Azure 存储 Blob 或 SQL Database 存储电话号码和呼叫文本，而不使用 Web 窗体。有关在 Java 中使用 Azure 存储 Blob 的信息，请参阅[如何从 Java 使用 Blob 存储服务][如何从 Java 使用 Blob 存储服务]。有关在 Java 中使用 SQL Database 的信息，请参阅[在 Java 中使用 SQL Database][在 Java 中使用 SQL Database]。
-   你可以使用 **RoleEnvironment.getConfigurationSettings** 从部署的配置设置中检索 Twilio 帐户 ID 和身份验证令牌，而不用对 makecall.jsp 中的值进行硬编码。有关 **RoleEnvironment** 类的信息，请参阅[在 JSP 中使用 Azure 服务运行时库][在 JSP 中使用 Azure 服务运行时库]和 [http://dl.windowsazure.com/javadoc][azure_javadoc] 上的 Azure 服务运行时包文档。
-   makecall.jsp 代码将 Twilio 提供的 URL [http://twimlets.com/message][twimlet_message_url]<http://twimlets.com/message></a> 分配给 **Url** 变量。此 URL 提供了一个 Twilio 标记语言 (TwiML) 响应，指示 Twilio 如何继续进行呼叫。例如，返回的 TwiML 可能包含 **\<Say\>** 谓词，该谓词生成了与呼叫接收人的谈话的文本。你可以生成自己的服务来响应 Twilio 的请求，而不使用 Twilio 提供的 URL；有关详细信息，请参阅[如何通过 Java 使用 Twilio 实现语音和短信功能][如何通过 Java 使用 Twilio 实现语音和短信功能]。有关 TwiML 的详细信息可在 [http://www.twilio.com/docs/api/twiml][twiml] 上找到，有关 **\<Say\>** 和其他 Twilio 谓词的信息可在 [http://www.twilio.com/docs/api/twiml/say][twilio_say] 上找到。
-   阅读 [https://www.twilio.com/docs/security][twilio_docs_security] 上的 Twilio 安全准则。

有关 Twilio 的更多信息，请参阅 [https://www.twilio.com/docs][twilio_docs]。

## 另请参阅

-   [如何通过 Java 使用 Twilio 实现语音和短信功能][如何通过 Java 使用 Twilio 实现语音和短信功能]
-   [将证书添加到 Java CA 证书存储][将证书添加到 Java CA 证书存储]

  [使用 Twilio 和 Java 的 Azure 呼叫窗体]: ./media/partner-twilio-java-phone-call-example/WA_TwilioJavaCallForm.jpg
  
  [将证书添加到 Java CA 证书存储]: ../java-add-certificate-ca-store
  [使用 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）创建 Hello World 应用程序]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh690944.aspx
  [使用 Twilio 和 Java 的 Azure 呼叫响应]: ./media/partner-twilio-java-phone-call-example/WA_TwilioJavaMakeCall.jpg
  [如何从 Java 使用 Blob 存储服务]: http://www.windowsazure.com/zh-cn/develop/java/how-to-guides/blob-storage/
  [在 Java 中使用 SQL Database]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh749029.aspx
  [在 JSP 中使用 Azure 服务运行时库]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh690948.aspx
  [如何通过 Java 使用 Twilio 实现语音和短信功能]: ../partner-twilio-java-how-to-use-voice-sms

  [twilio_pricing]: http://www.twilio.com/pricing
  [try_twilio]: http://www.twilio.com/try-twilio
  [twilio_api]: http://www.twilio.com/api
  [verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#
  [twilio_java_github]: http://github.com/twilio/twilio-java
  [twimlet_message_url]: http://twimlets.com/message
  [twiml]: http://www.twilio.com/docs/api/twiml
  [twilio_api_service]: http://api.twilio.com
  [add_ca_cert]: ../java-add-certificate-ca-store
  [azure_java_eclipse_hello_world]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh690944.aspx
  [howto_twilio_voice_sms_java]: ../partner-twilio-java-how-to-use-voice-sms
  [howto_blob_storage_java]: http://www.windowsazure.com/zh-cn/develop/java/how-to-guides/blob-storage/
  [howto_sql_azure_java]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh749029.aspx
  [azure_runtime_jsp]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh690948.aspx
  [azure_javadoc]: http://dl.windowsazure.com/javadoc
  [twilio_docs_security]: http://www.twilio.com/docs/security
  [twilio_docs]: http://www.twilio.com/docs
  [twilio_say]: http://www.twilio.com/docs/api/twiml/say
  [twilio_java]: ./media/partner-twilio-java-phone-call-example/WA_TwilioJavaCallForm.jpg
  [twilio_java_response]: ./media/partner-twilio-java-phone-call-example/WA_TwilioJavaMakeCall.jpg