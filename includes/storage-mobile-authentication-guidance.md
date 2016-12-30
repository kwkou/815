## 创建用于访问 Azure 存储的应用程序

有两种方法可以对要访问存储服务的应用程序进行身份验证：

- 共享密钥：使用共享密钥仅用于测试目的
- 共享访问签名 (SAS)：对于生产应用程序使用 SAS

### 共享密钥
共享密钥身份验证意味着你的应用程序将使用帐户名和帐户密钥访问存储服务。为了快速说明如何使用此库，我们将在此入门指南中使用共享密钥身份验证。

> [AZURE.WARNING]**请只将共享密钥身份验证用于测试目的！** 为关联的存储帐户提供完全读/写访问权限的帐户名和帐户密钥将分发给下载你的应用的每个人。这**不**是好的做法，因为你会面临向不受信任的客户端泄露密钥的风险。

使用共享密钥身份验证时，将创建一个[连接字符串](/documentation/articles/storage-configure-connection-string/)。连接字符串由以下部分组成：

- **DefaultEndpointsProtocol** - 可以选择 HTTP 或 HTTPS。但是，强烈建议使用 HTTPS。
- **帐户名** - 存储帐户的名称
- **帐户密钥** - 在 [Azure 门户预览](https://portal.azure.cn)上，导航到你的存储帐户，然后单击“密钥”图标以查看此信息。
- **EndpointSuffix** - 用于区域中具有不同终结点后缀的存储服务，例如 Azure 中国, 需要指定 EndpointSuffix=core.chinacloudapi.cn。

以下是使用共享密钥身份验证的连接字符串示例：

`"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn"`

### 共享访问签名 (SAS)
对于移动应用程序，针对 Azure 存储服务对客户端请求进行身份验证的建议方法是使用共享访问签名 (SAS)。SAS 允许你使用指定的权限集向客户端授予在指定的时间内对资源的访问权限。作为存储帐户所有者，需要为移动客户端生成要使用的 SAS。若要生成 SAS，你可能需要编写单独的服务，该服务生成要分发给客户端的 SAS。出于测试目的，可以使用 [Azure 存储资源管理器](http://storageexplorer.com)或 [Azure 门户预览](https://portal.azure.cn)来生成 SAS。创建 SAS 时，可以指定 SAS 有效的时间间隔，以及 SAS 授予客户端的权限。

以下示例演示如何使用 Azure 存储资源管理器来生成 SAS。

1. [安装 Azure 存储资源管理器](http://storageexplorer.com)（如果尚未安装）

2. 连接到订阅

3. 单击你的存储帐户，然后单击左下方的“操作”选项卡。单击“获取共享访问签名”，生成 SAS 的连接字符串。

4. 下面是 SAS 连接字符串的示例，该字符串为存储帐户的 Blob 服务授予对服务、容器和对象级别的读取与写入权限。

  `"SharedAccessSignature=sv=2015-04-05&ss=b&srt=sco&sp=rw&se=2016-07-21T18%3A00%3A00Z&sig=3ABdLOJZosCp0o491T%2BqZGKIhafF1nlM3MzESDDD3Gg%3D;BlobEndpoint=https://youraccount.blob.core.chinacloudapi.cn"`

可以看到，使用 SAS 时，不会在应用程序中公开帐户密钥。可以查阅 [Shared Access Signature s: Understanding the SAS model](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)（共享访问签名：了解 SAS 模型）了解有关 SAS 和使用 SAS 的最佳实践的详细信息。

<!---HONumber=Mooncake_1226_2016-->