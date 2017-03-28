<properties
    pageTitle="Azure 服务总线 API 调用示例"
    description="Azure 服务总线 API 调用示例"
    service=""
    resource=""
    authors="Allan Li"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Service Bus, API, SAS Token"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="sample-code-aog"
    ms.date=""
    wacn.date="03/27/2017" />
# Azure 服务总线 API 调用示例

编程调用 Azure 服务总线的相关服务，微软有提供基于 .NET 的 dll（[Nuget](https://www.nuget.org/packages/WindowsAzure.ServiceBus/)），也有基于 Java 的 Jar 包（[Mvnrepository](https://mvnrepository.com/artifact/com.microsoft.azure/azure-servicebus)），另外也可以直接调用服务总线 API。这片文章就提供 Azure 服务总线 API 调用示例，以供参考。主要包含以下内容：

1. 如何从策略生成共享访问令牌。
2. 调用 API 发送消息到服务总线队列。
3. 调用 API 从服务总线队列接收消息，包括 ReceiveAndDelete 模式和 PeekLock 模式。

## 如何从策略生成共享访问令牌

当调用服务总线 API 时，必须要提供共享访问令牌（SAS Token）作为 Authorization 头部，以作认证。那么如何生成这个 SAS Token 呢？

SAS Token 的具体构成是这样的：`SharedAccessSignature sr={资源地址}&sig={签名}&se={过期时间}&skn={策略名称}`。

- 资源地址：要访问资源的 URL 地址，比如 demo 命名空间下的 q1 队列 `https://demo.servicebus.chinacloudapi.cn/q1`

- 签名：对由资源地址，换行符和过期时间组成的字符串用策略秘钥通过 SHA-256 哈希后生成的 Base64String。

- 过期时间：自纪元算起，以秒为单位。

- 策略名称：相应策略的名称。

<br>

    static string createToken(string resourceUri, string keyName, string key)
    {
        TimeSpan sinceEpoch = DateTime.UtcNow - new DateTime(1970, 1, 1);
        var week = 60 * 60 * 24 * 7;
        var expiry = Convert.ToString((int)sinceEpoch.TotalSeconds + week);
        string stringToSign = HttpUtility.UrlEncode(resourceUri) + "\n" + expiry;
        HMACSHA256 hmac = new HMACSHA256(Encoding.UTF8.GetBytes(key));
        var signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign)));
        var sasToken = String.Format(CultureInfo.InvariantCulture, 
                    "SharedAccessSignature sr={0}&sig={1}&se={2}&skn={3}", 
                    HttpUtility.UrlEncode(resourceUri), 
                    HttpUtility.UrlEncode(signature), 
                    expiry, 
                    keyName);
        return sasToken;
    }

## 调用 API 发送消息到服务总线队列

发送消息的 API 具体描述可参照官方文档：[Send Message](https://docs.microsoft.com/en-us/rest/api/servicebus/send-message-to-queue)。

最关键的就是将 HTTP 请求的 Authorization 头部的值设置为根据拥有发送权限策略生成的 SAS Token。

    static async Task SendMessageAsync(ServiceBusHttpMessage message, string token)
    {
        var address = $"https://{serviceBusNamespace}.servicebus.chinacloudapi.cn/{queueName}/messages";

        HttpContent postContent = new ByteArrayContent(message.Body);

        // Serialize BrokerProperties. 
        var brokerProps = JsonConvert.SerializeObject(message.SystemProperties,
            Formatting.None,
            new JsonSerializerSettings { NullValueHandling = NullValueHandling.Ignore, 
                DefaultValueHandling = DefaultValueHandling.Ignore });

        postContent.Headers.Add("BrokerProperties", brokerProps);

        // Add custom properties. 
        foreach (string key in message.CustomProperties)
        {
            postContent.Headers.Add(key, message.CustomProperties.GetValues(key));
        }

        var httpClient = new HttpClient();
        httpClient.DefaultRequestHeaders.Add("Authorization", token);
        httpClient.DefaultRequestHeaders.Add("ContentType", "application/atom+xml;type=entry;charset=utf-8");

        // Send message. 
        HttpResponseMessage response = null;
        try
        {
            response = await httpClient.PostAsync($"{address}?timeout=60", postContent);
            response.EnsureSuccessStatusCode();
            Console.WriteLine("SendMessage successfully!");
        }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"SendMessage failed: {ex.Message}");
        }
        response.Dispose();
    }

## 调用 API 从服务总线队列接收消息

ReceiveAndDelete 模式 API：[Receive and Delete Message](https://docs.microsoft.com/en-us/rest/api/servicebus/receive-and-delete-message-destructive-read) 。

PeekLock 模式 API：[Peek-Lock Message](https://docs.microsoft.com/en-us/rest/api/servicebus/peek-lock-message-non-destructive-read) 。

两者的区别在于，ReceiveAndDelete 模式消息被接收后就自动从队列中删除了，而 PeekLock 模式只是把消息锁住，然后由客户端来删除。所以两种模式的 API URI 是一致的，只是 HTTP 方法不一样，前者是 DELETE，后者是 POST。

    static async Task<ServiceBusHttpMessage> ReceiveMessageAsync(string token, bool deleteMessage = true)
    {
        var address = $"https://{serviceBusNamespace}.servicebus.chinacloudapi.cn/{queueName}/messages/head";

        var httpClient = new HttpClient();
        httpClient.DefaultRequestHeaders.Add("Authorization", token);

        HttpResponseMessage response = null;

        try
        {
            if (deleteMessage)
            {
                // receive and delete
                response = await httpClient.DeleteAsync($"{address}?timeout=60");
            }
            else
            {
                // peek lock
                response = await httpClient.PostAsync($"{address}?timeout=60", new ByteArrayContent(new Byte[0]));
            }
            
            response.EnsureSuccessStatusCode();
            Console.WriteLine($"ReceiveMessage successfully!");
        }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"ReceiveMessage failed: {ex.Message}");
            return null;
        }

        var message = await ResolveMessageFromResponse(response);
        response.Dispose();
        return message;
    }

完整代码：[SBRestApiDemo](https://github.com/allenhula/azure-china-get-started/tree/master/ServiceBus/CSharp/SBRestApiDemo)。

## 更多资源

- [从策略生成共享访问令牌（SAS Token）](https://www.azure.cn/documentation/articles/service-bus-sas-overview/#bookmark-2)
- [服务总线REST API](https://docs.microsoft.com/en-us/rest/api/servicebus/service-bus-runtime-rest)
