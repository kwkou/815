<properties 
	pageTitle="配置 Azure 连接字符串" 
	description="了解如何在 Azure 中配置存储帐户的连接字符串。" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor="cgronlun"/>
	
<tags ms.service="storage" ms.date="02/20/2015" wacn.date="04/11/2015"/>


# 配置 Azure 存储空间连接字符串

## 概述

连接字符串包含以编程方式访问 Azure 中的存储帐户所需的信息。你可以配置通过下列方式连接到 Azure 存储空间的连接字符串：

- 在本地测试服务或应用程序时连接到 Azure 存储模拟器。

- 使用存储服务的默认终结点连接到 Azure 中的存储帐户。

- 使用存储服务的显式终结点连接到 Azure 中的存储帐户。

如果你的应用程序是在 Azure 中运行的云服务，则通常会将连接字符串保存到 [Azure 服务配置架构 (.cscfg) 文件](https://msdn.microsoft.com/zh-CN/library/ee758710.aspx)中。如果你的应用程序在其他环境（例如，在桌面上）中运行，则通常会将连接字符串保存在 app.config 文件或其他配置文件中。你可以使用 Azure `CloudConfigurationManager` 类在运行时访问连接字符串，而不考虑它在何处运行。

## 创建存储模拟器的连接字符串

存储模拟器是使用已知名称和密钥的本地帐户。由于帐户名称和密钥对所有用户都是相同的，因此你可以使用快捷字符串格式来引用连接字符串中的存储模拟器。将连接字符串的值设为  `UseDevelopmentStorage=true`。

你还可以指定一个 HTTP 代理，以便在针对存储模拟器测试服务时进行使用。当你针对存储服务调试操作时，这对观察 HTTP 请求和响应很有用。若要指定代理，请将  `DevelopmentStorageProxyUri` 选项添加到连接字符串，并将它的值设置为代理 URI。例如，下面是一个指向存储模拟器并配置 HTTP 代理的连接字符串：

    UseDevelopmentStorage=true;DevelopmentStorageProxyUri=http://myProxyUri

## 创建 Azure 存储帐户的连接字符串

若要创建 Azure 存储帐户的连接字符串，请使用下面的连接字符串格式。指示要通过 HTTP 还是 HTTPS 连接到存储帐户，将  `myAccountName` 替换为存储帐户的名称，将  `myAccountKey` 替换为帐户访问密钥：

    DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey;EndpointSuffix=core.chinacloudapi.cn

例如，你的连接字符串将类似于以下示例连接字符串：

```        DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=KWPLd0rpW2T0U7K2pVpF8rYr1BgYtB7wYQw33AYiXeUoquiaY6o0TWqduxmPHlqeCNZ3LU0DHptbeIAy5l/Yhg==;EndpointSuffix=core.chinacloudapi.cn
```

> [AZURE.NOTE] Azure 存储空间连接字符串同时支持 HTTP 和 HTTPS，但强烈建议使用 HTTPS。
    
## 创建显式存储终结点的连接字符串

如果满足以下条件，你可以在连接字符串中显式指定服务终结点：

- 你已将存储帐户的自定义域名映射到 Blob 服务。
- 你拥有用于访问存储帐户中的存储资源的共享访问签名。

若要创建指定显式 Blob 终结点的连接字符串，请使用以下格式为每个服务指定完整的服务终结点，包括协议规范（HTTP 或 HTTPS）：

``` 
BlobEndpoint=myBlobEndpoint;QueueEndpoint=myQueueEndpoint;TableEndpoint=myTableEndpoint;FileEndpoint=myFileEndpoint;EndpointSuffix=core.chinacloudapi.cn;[credentials]
```

你必须至少指定一个服务终结点，但无需指定所有终结点。例如，如果要创建一个连接字符串以用于自定义 Blob 终结点，则可以选择指定队列和表终结点。请注意，如果你选择在连接字符串中省略队列和表终结点，则将无法使用该连接字符串从代码访问队列和表服务。

在连接字符串中显式指定服务终结点时，你有两个选项可在上述字符串中指定 `凭据`：

- 可以指定帐户名称和密钥： `AccountName=myAccountName;AccountKey=myAccountKey` 
- 可以指定共享访问签名： `SharedAccessSignature=base64Signature`

### 使用自定义域名指定 Blob 终结点 

如果你注册了一个自定义域名以用于 BLOB 服务，则可能希望显式配置连接字符串中的 Blob 终结点。连接字符串中列出的终结点值用于构造 BLOB 服务的请求 URI，还指示返回到代码的任何 URI 形式。 

例如，自定义域上的 Blob 终结点的连接字符串可能类似于：

```
DefaultEndpointsProtocol=https;BlobEndpoint=www.mydomain.com;AccountName=storagesample;AccountKey=KWPLd0rpW2T0U7K2pVpF8rYr1BgYtB7wYQw33AYiXeUoquiaY6o0TWqduxmPHlqeCNZ3LU0DHptbeIAy5l/Yhg==;EndpointSuffix=core.chinacloudapi.cn
```

### 使用共享访问签名指定 Blob 终结点 

你可以使用显式终结点创建连接字符串，以便通过共享访问签名访问存储资源。在这种情况下，你可以指定共享访问签名作为连接字符串的一部分，而不是指定帐户名称和密钥凭据。共享访问签名令牌将封装要访问的资源的相关信息、该签名可用的时间期限以及授予的权限。有关共享访问签名的详细信息，请参阅[使用共享访问签名委托访问权限](https://msdn.microsoft.com/zh-CN/library/ee395415.aspx)。

若要创建包含共享访问签名的连接字符串，请按以下格式指定该字符串：

    BlobEndpoint=myBlobEndpoint; QueueEndpoint=myQueueEndpoint;TableEndpoint=myTableEndpoint;SharedAccessSignature=base64Signature;EndpointSuffix=core.chinacloudapi.cn

终结点可以是默认服务终结点或自定义终结点。 `base64Signature` 与共享访问签名的签名部分对应。该签名是利用 SHA256 算法通过有效的"字符串到签名"和密钥计算的 HMAC，计算结果随后进行 Base64 编码。

<!--HONumber=51-->