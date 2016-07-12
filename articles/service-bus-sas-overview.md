<properties
   pageTitle="共享访问签名概述 | Microsoft Azure"
   description="共享访问签名是什么，其工作原理是怎样的，以及如何在 Node、PHP 和 C# 编程中使用它们。"
   services="service-bus,event-hubs"
   documentationCenter="na"
   authors="djrosanova"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-bus"
    ms.date="03/16/2016"
   wacn.date="04/05/2016"/>

# 共享访问签名

*共享访问签名* (SAS) 是服务总线的主要安全机制，包括事件中心、中转消息传送（队列和主题）和中继消息传送。本文介绍共享访问签名、其工作原理以及如何以平台无关的方式使用它们。

## SAS 概述

共享访问签名是基于 SHA-256 安全哈希或 URI 的身份验证机制。SAS 是所有服务总线服务使用的非常强大的机制。在实际应用中，SAS 有两个组件：*共享访问策略*和*共享访问签名*（通常称为*令牌*）。

你可以在[对服务总线进行共享访问签名身份验证](/documentation/articles/service-bus-shared-access-signature-authentication/)中找到有关共享访问签名与服务总线的更详细信息。

## 共享访问策略

对于 SAS，要了解的一个重点是，一切都从策略开始。对于每个策略，需要确定三个信息片段：**名称**、**范围**和**权限**。**名称**只是该范围内的唯一名称。范围也很简单：它是相关资源的 URI。对于服务总线命名空间，范围是完全限定的域名 (FQDN)，例如 **`https://<yournamespace>.servicebus.chinacloudapi.cn/`**。

策略的可用权限大多数都易于理解：

  + 发送
  + 侦听
  + 管理

在你创建策略后，系统将为它分配*主密钥*和*辅助密钥*。它们是加密形式的强密钥。请不要遗失或透漏这些密钥 - 在 [Azure 管理门户][]中总要用到它们。你可以使用其中一个生成的密钥，并且随时可以重新生成密钥。不过，如果你重新生成或更改策略中的主密钥，基于该密钥创建的所有共享访问签名都将失效。

当你创建服务总线命名空间时，系统将自动为整个命名空间创建名为 **RootManageSharedAccessKey** 的策略，此策略具有所有权限。你不会以 **root** 身份登录，因此除非有适合的理由，否则请勿使用此策略。可以在 Azure 管理门户上的命名空间“配置”选项卡中创建更多的策略。请务必注意，在服务总线中的单一树级别（命名空间、队列、事件中心等）中，最多只能附加 12 个策略。

## 共享访问签名（令牌）

策略本身不是服务总线的访问令牌。它是使用主密钥或辅助密钥生成访问令牌时所依据的对象。令牌是通过采用以下格式妥善编写一个字符串而生成的：

```
SharedAccessSignature sig=<signature-string>&se=<expiry>&skn=<keyName>&sr=<URL-encoded-resourceURI>
```

其中，`signature-string` 是令牌范围的 SHA-256 哈希（前一部分已介绍**范围**），后面附加了 CRLF 和过期时间（自纪元算起，以秒为单位：1970 年 1 月 1 日 `00:00:00 UTC`）。

哈希类似于以下虚构代码，它返回 32 个字节。

```
SHA-256('https://<yournamespace>.servicebus.chinacloudapi.cn/'+'\n'+ 1438205742)
```

非哈希值位于 **SharedAccessSignature** 字符串中，这样，接收方便可以使用相同的参数计算哈希，以确保它返回相同的结果。URI 指定范围，而密钥名称标识要用于计算哈希的策略。从安全性的立场来看，这非常重要。如果签名与接收方（服务总线）的计算结果不符，则拒绝访问。此时，我们可以确保发送方可访问密钥，并且应该被授予策略中指定的权限。

## 从策略生成签名

在代码中如何实际执行此操作？ 让我们探讨一下几个示例。

### NodeJS

```
function createSharedAccessToken(uri, saName, saKey) { 
    if (!uri || !saName || !saKey) { 
            throw "Missing required parameter"; 
        } 
    var encoded = encodeURIComponent(uri); 
    var now = new Date(); 
    var week = 60*60*24*7;
    var ttl = Math.round(now.getTime() / 1000) + week;
    var signature = encoded + '\n' + ttl; 
    var signatureUTF8 = utf8.encode(signature); 
    var hash = crypto.createHmac('sha256', saKey).update(signatureUTF8).digest('base64'); 
    return 'SharedAccessSignature sr=' + encoded + '&sig=' +  
        encodeURIComponent(hash) + '&se=' + ttl + '&skn=' + saName; 
}
``` 

### Java

```
private static String GetSASToken(String resourceUri, String keyName, String key)
  {
      long epoch = System.currentTimeMillis()/1000L;
      int week = 60*60*24*7;
      String expiry = Long.toString(epoch + week);

      String sasToken = null;
      try {
          String stringToSign = URLEncoder.encode(resourceUri, "UTF-8") + "\n" + expiry;
          String signature = getHMAC256(key, stringToSign);
          sasToken = "SharedAccessSignature sr=" + URLEncoder.encode(resourceUri, "UTF-8") +"&sig=" +
                  URLEncoder.encode(signature, "UTF-8") + "&se=" + expiry + "&skn=" + keyName;
      } catch (UnsupportedEncodingException e) {

          e.printStackTrace();
      }

      return sasToken;
  }


public static String getHMAC256(String key, String input) {
    Mac sha256_HMAC = null;
    String hash = null;
    try {
        sha256_HMAC = Mac.getInstance("HmacSHA256");
        SecretKeySpec secret_key = new SecretKeySpec(key.getBytes(), "HmacSHA256");
        sha256_HMAC.init(secret_key);
        Encoder encoder = Base64.getEncoder();

        hash = new String(encoder.encode(sha256_HMAC.doFinal(input.getBytes("UTF-8"))));

    } catch (InvalidKeyException e) {
        e.printStackTrace();
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
   } catch (IllegalStateException e) {
        e.printStackTrace();
    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
    }

    return hash;
}
```

### PHP

```
function generateSasToken($uri, $sasKeyName, $sasKeyValue) 
{ 
$targetUri = strtolower(rawurlencode(strtolower($uri))); 
$expires = time(); 	
$expiresInMins = 60; 
$week = 60*60*24*7;
$expires = $expires + $week; 
$toSign = $targetUri . "\n" . $expires; 
$signature = rawurlencode(base64_encode(hash_hmac('sha256', 			
 $toSign, $sasKeyValue, TRUE))); 

$token = "SharedAccessSignature sr=" . $targetUri . "&sig=" . $signature . "&se=" . $expires . 		"&skn=" . $sasKeyName; 
return $token; 
}
```
 
### C&#35;

```
private static string createToken(string resourceUri, string keyName, string key)
{
    TimeSpan sinceEpoch = DateTime.UtcNow - new DateTime(1970, 1, 1);
    var week = 60 * 60 * 24 * 7;
    var expiry = Convert.ToString((int)sinceEpoch.TotalSeconds + week);
    string stringToSign = HttpUtility.UrlEncode(resourceUri) + "\n" + expiry;
    HMACSHA256 hmac = new HMACSHA256(Encoding.UTF8.GetBytes(key));
    var signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign)));
    var sasToken = String.Format(CultureInfo.InvariantCulture, "SharedAccessSignature sr={0}&sig={1}&se={2}&skn={3}", HttpUtility.UrlEncode(resourceUri), HttpUtility.UrlEncode(signature), expiry, keyName);
    return sasToken;
}
```

## 使用共享访问签名（在 HTTP 级别）
 
在了解如何为服务总线中的任何实体创建共享访问签名后，便可以执行 HTTP POST 了：

```
POST https://<yournamespace>.servicebus.chinacloudapi.cn/<yourentity>/messages
Content-Type: application/json
Authorization: SharedAccessSignature sr=https%3A%2F%2F<yournamespace>.servicebus.chinacloudapi.cn%2F<yourentity>&sig=<yoursignature from code above>&se=1438205742&skn=KeyName
ContentType: application/atom+xml;type=entry;charset=utf-8
``` 
	
请记住，这适用于所有情况。你可以为队列、主题、订阅、事件中心或中继创建 SAS。如果对事件中心使用按发布者标识，只需附加 `/publishers/< publisherid>`。

如果你为发送方或客户端提供 SAS 令牌，它们不会直接获取密钥，并且他们无法逆向改编哈希来获取它。因此，你可以控制它们有权访问的项，以及可访问的时间长短。要记住的一个重点是，如果你更改策略中的主密钥，基于该密钥创建的所有共享访问签名都将失效。

## 使用共享访问签名（在 AMQP 级别）

在前一部分中，你已了解如何使用 SAS 令牌配合 HTTP POST 请求将数据发送到服务总线。如你所了解，你可以使用高级消息队列协议 (AMQP) 来访问服务总线。在许多方案中，都会出于性能原因而将该协议用作首选协议。文档[基于 AMQP 声明的安全性版本 1.0](https://www.oasis-open.org/committees/download.php/50506/amqp-cbs-v1%200-wd02%202013-08-12.doc)（自 2013 年以来以有效草案版推出，不过 Azure 现在能够很好地支持它）中介绍了如何配合 AMQP 使用 SAS 令牌。

开始将数据发送到服务总线之前，发布者必须将 AMQP 消息中的 SAS 令牌发送到正确定义的名为 **"$cbs"** 的 AMQP 节点（可以将它视为一个由服务使用的“特殊”队列，用于获取和验证所有 SAS 令牌）。发布者必须在 AMQP 消息中指定 **"ReplyTo"** 字段；这是服务向发布者回复令牌验证结果（发布者与服务之间的简单请求/回复模式）时所在的节点。根据 AMQP 1.0 规范中有关“动态创建远程节点”的论述，此回复节点是“在运行中”创建的。在检查 SAS 令牌有效之后，发布者可以继续将数据发送到服务。

下面的步骤演示如何使用 [AMQP.Net Lite](https://github.com/Azure/amqpnetlite) 库通过 AMQP 协议发送 SAS 令牌。如果你不能使用官方的服务总线 SDK（例如，在 WinRT、Net Compact Framework、.Net Micro Framework 和 Mono 中）进行 C&#35; 开发，则这很有用。当然，此库对于帮助了解基于声明的安全性如何在 AMQP 级别工作非常有用，就如同你可以了解它如何在 HTTP 级别工作一样（使用 HTTP POST 请求并在“Authorization”标头内部发送 SAS 令牌）。如果你不需要此类有关 AMQP 的深入知识，可以将官方的服务总线 SDK 用于 .Net Framework 应用程序，该 SDK 会为你执行此操作。

### C&#35;

```
/// <summary>
/// Send Claim Based Security (CBS) token
/// </summary>
/// <param name="shareAccessSignature">Shared access signature (token) to send</param>
private bool PutCbsToken(Connection connection, string sasToken)
{
    bool result = true;
    Session session = new Session(connection);

    string cbsClientAddress = "cbs-client-reply-to";
    var cbsSender = new SenderLink(session, "cbs-sender", "$cbs");
    var cbsReceiver = new ReceiverLink(session, cbsClientAddress, "$cbs");

    // construct the put-token message
    var request = new Message(sasToken);
    request.Properties = new Properties();
    request.Properties.MessageId = Guid.NewGuid().ToString();
    request.Properties.ReplyTo = cbsClientAddress;
    request.ApplicationProperties = new ApplicationProperties();
    request.ApplicationProperties["operation"] = "put-token";
    request.ApplicationProperties["type"] = "servicebus.chinacloudapi.cn:sastoken";
    request.ApplicationProperties["name"] = Fx.Format("amqp://{0}/{1}", sbNamespace, entity);
    cbsSender.Send(request);

    // receive the response
    var response = cbsReceiver.Receive();
    if (response == null || response.Properties == null || response.ApplicationProperties == null)
    {
        result = false;
    }
    else
    {
        int statusCode = (int)response.ApplicationProperties["status-code"];
        if (statusCode != (int)HttpStatusCode.Accepted && statusCode != (int)HttpStatusCode.OK)
        {
            result = false;
        }
    }

    // the sender/receiver may be kept open for refreshing tokens
    cbsSender.Close();
    cbsReceiver.Close();
    session.Close();

    return result;
}
```

`PutCbsToken()` 方法接收代表服务的 TCP 连接的 connection（[AMQP .NET Lite 库](https://github.com/Azure/amqpnetlite)提供的 AMQP Connection 类实例），以及表示要发送的 SAS 令牌的 sasToken 参数。

> [AZURE.NOTE] 请务必在 **SASL 身份验证机制设置为 EXTERNAL** 的情况下创建连接（而不是在不需要发送 SAS 令牌时使用的包含用户名与密码的默认 PLAIN）。

接下来，发布者创建两个 AMQP 链接来发送 SAS 令牌并接收来自服务的回复（令牌验证结果）。

AMQP 消息包含一组属性，比简单消息包含更多信息。SAS 令牌是消息的正文（使用其构造函数）。**"ReplyTo"** 属性设置为用于在接收方链接上接收验证结果的节点名称（可以根据需要更改其名称，该节点将由服务动态创建）。服务使用最后三个应用程序/自定义属性来指示它需要执行哪种类型的操作。如 CBS 草案规范中所述，这些属性必须是**操作名称** ("put-token")、**令牌类型**（在此例中为“servicebus.chinacloudapi.cn:sastoken”），以及要应用令牌的**受众的“名称”**（整个实体）。

在发送方链接上发送 SAS 令牌后，发布者需要在接收者链接上读取回复。回复是一个简单的 AMQP 消息，其中包含名为 **"status-code"** 的应用程序属性，这些属性可以包含与 HTTP 状态代码相同的值。

## 后续步骤

有关如何使用这些 SAS 令牌的详细信息，请参阅[服务总线 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/hh780717.aspx)。

有关服务总线身份验证的详细信息，请参阅[服务总线身份验证和授权](/documentation/articles/service-bus-authentication-and-authorization/)。

此[博客文章](http://developers.de/blogs/damir_dobric/archive/2013/10/17/how-to-create-shared-access-signature-for-service-bus.aspx)中介绍了更多关于 C# 和 Java 脚本中的 SAS 的示例。

[Azure 管理门户]: http://manage.windowsazure.cn

<!---HONumber=Mooncake_0104_2016-->