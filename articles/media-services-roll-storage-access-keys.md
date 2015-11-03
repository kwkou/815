<properties 
	pageTitle="轮转存储访问密钥后更新媒体服务" 
	description="此文将提供有关在轮转存储访问密钥后如何更新媒体服务的指导。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="09/07/2015"
	wacn.date="11/02/2015"/>

#如何：轮转存储访问密钥后更新媒体服务

当你创建新的 Azure 媒体服务帐户时，系统还会要求你选择用来存储媒体内容的 Azure 存储帐户。请注意，你可以将多个存储帐户添加到媒体服务帐户。

在创建新的存储帐户后，Azure 将生成两个 512 位存储访问密钥，用于对你存储帐户的访问进行身份验证。若要保持存储连接的安全性，我们建议定期重新生成并轮转你的存储访问密钥。将提供两个访问密钥（主密钥和辅助密钥），以便在你重新生成其中一个访问密钥时，始终能够使用另一个访问密钥连接到存储帐户。此过程也称为“轮转访问密钥”。

媒体服务依赖于其中的一个存储密钥（主密钥或辅助密钥）。具体而言，用于流式传输或下载资产的定位符依赖于访问密钥。在轮转存储访问密钥时，你还需要确保更新定位符，使流式传输服务不会中断。

>[AZURE.NOTE]在重新生成存储密钥后，必须确保与媒体服务同步更新。

本主题介绍了轮转存储密钥以及将媒体服务更新为使用适当存储密钥的步骤。请注意，如果你有多个存储帐户，请对每个存储帐户执行此过程。

>[AZURE.NOTE]在对生产帐户执行本主题中所述的步骤之前，请确保对生产前帐户测试这些步骤。


## 步骤 1：重新生成辅助存储访问密钥

首先，请重新生成辅助存储密钥。默认情况下，媒体服务不使用辅助密钥。有关如何轮转存储密钥的信息，请参阅[如何：查看、复制和重新生成存储访问密钥](/documentation/articles/storage-create-storage-account#view-copy-and-regenerate-storage-access-keys)。
  
##<a id="step2"></a>步骤 2：将媒体服务更新为使用新的辅助存储密钥

将媒体服务更新为使用辅助存储访问密钥。可以使用以下两种方法之一，将重新生成的存储密钥同步到媒体服务。

- 使用 Azure 门户：选择你的 Media Service 帐户，然后单击门户窗口底部的“管理密钥”图标。根据要与媒体服务同步的存储密钥，选择同步主密钥按钮或同步辅助密钥按钮。在本例中，我们将使用辅助密钥。

- 使用媒体服务管理 REST API。

	以下代码示例演示了如何构造 https://endpoint/<subscriptionId>/services/mediaservices/Accounts/<accountName>/StorageAccounts/<storageAccountName>/Key 请求，以便将指定的存储密钥与媒体服务同步。在本例中，我们将使用辅助存储密钥值。有关详细信息，请参阅[如何：使用媒体服务管理 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn167656.aspx)。
 
		public void UpdateMediaServicesWithStorageAccountKey(string mediaServicesAccount, string storageAccountName, string storageAccountKey)
		{
		    var clientCert = GetCertificate(CertThumbprint);
		
		    HttpWebRequest request = (HttpWebRequest)WebRequest.Create(string.Format("{0}/{1}/services/mediaservices/Accounts/{2}/StorageAccounts/{3}/Key",
		        Endpoint, SubscriptionId, mediaServicesAccount, storageAccountName));
		    request.Method = "PUT";
		    request.ContentType = "application/json; charset=utf-8";
		    request.Headers.Add("x-ms-version", "2011-10-01");
		    request.Headers.Add("Accept-Encoding: gzip, deflate");
		    request.ClientCertificates.Add(clientCert);
		
		
		    using (var streamWriter = new StreamWriter(request.GetRequestStream()))
		    {
		        streamWriter.Write("\"");
		        streamWriter.Write(storageAccountKey);
		        streamWriter.Write("\"");
		        streamWriter.Flush();
		    }
		
		    using (var response = (HttpWebResponse)request.GetResponse())
		    {
		        string jsonResponse;
		        Stream receiveStream = response.GetResponseStream();
		        Encoding encode = Encoding.GetEncoding("utf-8");
		        if (receiveStream != null)
		        {
		            var readStream = new StreamReader(receiveStream, encode);
		            jsonResponse = readStream.ReadToEnd();
		        }
		    }
		}

然后，更新现有（依赖于旧存储密钥）的定位符。

>[AZURE.NOTE]在对媒体服务执行任何操作（例如，创建新定位符）之前等待 30 分钟，以防止对挂起的作业产生任何影响。

##步骤 3：更新定位符 

在 30 分钟后，你可以更新现有的定位符，使其依赖于新的辅助存储密钥。

若要更新定位符的过期日期，请使用 [REST](https://msdn.microsoft.com/zh-CN/library/azure/hh974308.aspx#update_a_locator) 或 [.NET](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mediaservices.client.ilocator.update(v=azure.10).aspx) API。请注意，当你更新 SAS 定位符的过期日期时，URL 会发生变化。

##步骤 5：重新生成主存储访问密钥

重新生成主存储访问密钥。有关如何轮转存储密钥的信息，请参阅[如何：查看、复制和重新生成存储访问密钥](/documentation/articles/storage-create-storage-account#view-copy-and-regenerate-storage-access-keys)。

##步骤 6：将媒体服务更新为使用新的主存储密钥
	
按照[步骤 2](#step2) 中所述的相同过程操作，不过此次将新的主存储访问密钥与媒体服务帐户同步。

>[AZURE.NOTE]在对媒体服务执行任何操作（例如，创建新定位符）之前等待 30 分钟，以防止对挂起的作业产生任何影响。

##步骤 7：更新定位符  

在 30 分钟后，你可以更新现有的定位符，使其依赖于新的主存储密钥。

若要更新定位符的过期日期，请使用 [REST](https://msdn.microsoft.com/zh-CN/library/azure/hh974308.aspx#update_a_locator) 或 <a href="https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mediaservices.client.ilocator.update(v=azure.10).aspx">.NET</a> API。请注意，当你更新 SAS 定位符的过期日期时，URL 会发生变化。

 

<!---HONumber=76-->