<properties
    pageTitle="配置 Azure 存储空间的连接字符串 | Azure"
    description="配置 Azure 存储帐户的连接字符串。连接字符串包含在运行时从应用程序访问 Azure 存储帐户所需的身份验证信息。"
    services="storage"
    documentationcenter=""
    author="mmacy"
    manager="timlt"
    editor="tysonn" />  

<tags
    ms.assetid="ecb0acb5-90a9-4eb2-93e6-e9860eda5e53"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/12/2016"
    wacn.date="12/29/2016"
    ms.author="marsma" />

# 配置 Azure 存储空间连接字符串
## 概述
连接字符串包含在运行时从应用程序访问 Azure 存储帐户中数据所需的身份验证信息。您可以将连接字符串配置为：

* 连接到 Azure 存储模拟器。
* 在 Azure 中访问存储帐户。
* 通过共享访问签名 (SAS) 访问 Azure 中的指定资源。

[AZURE.INCLUDE [storage-account-key-note-include](../../includes/storage-account-key-note-include.md)]

## 存储连接字符串
应用程序需要在运行时访问连接字符串，才能验证对 Azure 存储空间发出的请求。有几个不同的选项来存储连接字符串：

* 对于在桌面或设备上运行的应用程序，可以在 **app.config** 文件或 **web.config** 文件中存储连接字符串。将连接字符串添加到 **AppSettings** 节。
* 对于在 Azure 云服务中运行的应用程序，可以在 [Azure 服务配置架构 (.cscfg) 文件](https://msdn.microsoft.com/zh-cn/library/ee758710.aspx)中存储连接字符串。将连接字符串添加到服务配置文件的 **ConfigurationSettings** 部分。
* 也可以直接在代码中使用连接字符串。但是，在大部分情况下，建议在配置文件中存储配置字符串。

在一个配置文件中存储连接字符串可以轻松地更新连接字符串，从而在存储模拟器和云中的 Azure 存储帐户之间切换。你只需编辑连接字符串，使其指向你的目标环境。

你可以使用 [Azure Configuration Manager](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/) 类在运行时访问连接字符串，而不考虑你的应用程序在何处运行。

## 创建存储模拟器的连接字符串
[AZURE.INCLUDE [storage-emulator-connection-string-include](../../includes/storage-emulator-connection-string-include.md)]

请参阅[使用 Azure 存储模拟器进行开发和测试](/documentation/articles/storage-use-emulator/)以了解有关存储模拟器的更多信息。

## 创建 Azure 存储帐户的连接字符串
若要创建 Azure 存储帐户的连接字符串，请使用下面的连接字符串格式。该字符串指示是通过 HTTPS（推荐）还是 HTTP 连接到存储帐户，将 *myAccountName* 替换为存储帐户的名称，将 *myAccountKey* 替换为帐户访问密钥：

    DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey;EndpointSuffix=core.chinacloudapi.cn

例如，你的连接字符串将类似于以下示例连接字符串：
 
	DefaultEndpointsProtocol=https;
	AccountName=storagesample;
	AccountKey=<account-key>;
	EndpointSuffix=core.chinacloudapi.cn

> [AZURE.NOTE] Azure 存储空间连接字符串同时支持 HTTP 和 HTTPS，但强烈建议使用 HTTPS。

## 使用共享访问签名创建连接字符串
[AZURE.INCLUDE [storage-use-sas-in-connection-string-include](../../includes/storage-use-sas-in-connection-string-include.md)]

## 创建显式存储终结点的连接字符串
可以在连接字符串中显式指定服务终结点，而不使用默认终结点。若要创建指定显式终结点的连接字符串，请使用以下格式为每个服务指定完整的服务终结点，包括协议规范（HTTPS（建议）或 HTTP）：

	DefaultEndpointsProtocol=[http|https];
	BlobEndpoint=myBlobEndpoint;
	QueueEndpoint=myQueueEndpoint;
	TableEndpoint=myTableEndpoint;
	FileEndpoint=myFileEndpoint;
	AccountName=myAccountName;
	AccountKey=myAccountKey

如果你已将 Blob 存储终结点映射到自定义域，则可能需要指定显式终结点。在这种情况下，你可以在连接字符串中指定 Blob 存储的自定义终结点，并选择地指定另一个服务的默认终结点（如果应用程序使用这些终结点）。

下面是用于指定 Blob 服务的显式终结点的有效连接字符串的示例：

	# Blob endpoint only
	DefaultEndpointsProtocol=https;
	BlobEndpoint=www.mydomain.com;
	AccountName=storagesample;
	AccountKey=account-key

	# All service endpoints
	DefaultEndpointsProtocol=https;
	BlobEndpoint=www.mydomain.com;
	FileEndpoint=myaccount.file.core.chinacloudapi.cn;
	QueueEndpoint=myaccount.queue.core.chinacloudapi.cn;
	TableEndpoint=myaccount;
	AccountName=storagesample;
	AccountKey=account-key

连接字符串中列出的终结点值用于构造 BLOB 服务的请求 URI，还指示返回到代码的任何 URI 形式。

请注意，如果选择在连接字符串中省略服务终结点，则无法使用该连接字符串从代码访问该服务中的数据。

### 创建含终结点后缀的连接字符串
若要针对具有不同终结点后缀的地区或实例内的存储服务创建一个连接字符串，例如针对 Azure 中国或 Azure Governance，请使用以下连接字符串格式。该字符串指示是通过 HTTP 还是 HTTPS 连接到存储帐户，将 *myAccountName* 替换为存储帐户的名称，将 *myAccountKey* 替换为帐户访问密钥，以及将 *mySuffix* 替换为 URI 后缀：


	DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey;EndpointSuffix=mySuffix;


例如，你的连接字符串应类似于以下连接字符串：

	DefaultEndpointsProtocol=https;
	AccountName=storagesample;
	AccountKey=<account-key>;
	EndpointSuffix=core.chinacloudapi.cn;

## 分析连接字符串
[AZURE.INCLUDE [storage-cloud-configuration-manager-include](../../includes/storage-cloud-configuration-manager-include.md)]

## 后续步骤
* [使用 Azure 存储模拟器进行开发和测试](/documentation/articles/storage-use-emulator/)
* [Azure 存储资源管理器](/documentation/articles/storage-explorers/)
* [使用共享访问签名 (SAS)](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)

<!---HONumber=Mooncake_1226_2016-->