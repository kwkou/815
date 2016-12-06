<properties 
	pageTitle="如何为 Azure Service Bus 和 Azure IoT Hub 生成 SharedAccessSignature" 
	description="如何为 Azure Service Bus 和 Azure IoT Hub 生成 SharedAccessSignature" 
	services="" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="service-bus-aog" ms.date="" wacn.date="12/05/2016"/>
# 如何为 Azure Service Bus 和 Azure IoT Hub 生成 SharedAccessSignature #

很多服务在做验证的时候都会用到 SharedAccessSignature，例如 Azure Service Bus, Azure IoT Hub 等。这篇文章主要讨论不同服务下生成的 SharedAccessSignature 的区别。
首先看下 SharedAccessSignature 的格式：

    SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}

从以上格式得知，针对不同的服务只要正确指定 policyname,resourceURI 就可以计算出相应的 SAS。但实际上，对于不同的服务，如何利用密钥生成 signature 是存在细微差别的。
接下来以 Azure Service Bus 和 Azure IoT Hub 为例，首先是 Azure IoT Hub, 针对如何生成 signature 官方文档说明如下：

    {signature} ：An HMAC-SHA256 signature string of the form: {URL-encoded-resourceURI} + "\n" + expiry. Important: The key is decoded from >base64 and used as key to perform the HMAC-SHA256 computation.

其次关于 Azure Service Bus 的官方文档说明如下： 

    The signature for the SAS token is computed using the HMAC-SHA256 hash of a string-to-sign with the PrimaryKey property of an authorization rule.

从以上描述可知，两者的区别主要在于是否需要对秘钥进行 decode。代码分别如下：

    // HMACSHA256 hmac = new HMACSHA256(Convert.FromBase64String(key)); // decode the key
    HMACSHA256 hmac = new HMACSHA256(Encoding.UTF8.GetBytes(key)); // don't decode the key

如果您是使用 C# 做开发，可以通过以下方法生成 SAS Token:


	class SASTokenGenerator
	{
		public static string GetSASToken(string resourceUri, string keyName, string key, TimeSpan ttl)
	    {
		    var expiry = GetExpiry(ttl);
		    string stringToSign = HttpUtility.UrlEncode(resourceUri) + "\n" + expiry;
		    // HMACSHA256 hmac = new HMACSHA256(Convert.FromBase64String(key)); for IoT Hub Service
		    HMACSHA256 hmac = new HMACSHA256(Encoding.UTF8.GetBytes(key)); //for service bus Service
		    var signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign)));
		    var sasToken = String.Format(CultureInfo.InvariantCulture, "SharedAccessSignature sr={0}&sig={1}&se={2}&skn={3}", 
		    HttpUtility.UrlEncode(resourceUri), HttpUtility.UrlEncode(signature), expiry, keyName);
		    return sasToken;
	    }

	    private static string GetExpiry(TimeSpan ttl)
	    {
	    	TimeSpan expirySinceEpoch = DateTime.UtcNow - new DateTime(1970, 1, 1) + ttl;
	    	return Convert.ToString((int)expirySinceEpoch.TotalSeconds);
	    }
	}





