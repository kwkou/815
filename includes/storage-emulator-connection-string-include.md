存储模拟器支持单一固定的帐户和众所周知的用于共享密钥身份验证的身份验证密钥。 此帐户和密钥是允许用于存储模拟器的唯一共享密钥凭据。 它们具有以下特点：

    Account name: devstoreaccount1
    Account key: Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==

> [AZURE.NOTE]
> 存储模拟器支持的身份验证密钥仅用于测试客户端身份验证代码的功能。 它没有任何安全用途。 不能在存储模拟器中使用生产存储帐户和密钥。 不应将开发帐户用于生产数据。
> 
> 存储模拟器仅支持通过 HTTP 进行连接。 但是，若要访问生产性 Azure 存储帐户中的资源，建议使用 HTTPS 协议。
> 

#### <a name="connect-to-the-emulator-account-using-a-shortcut"></a>使用快捷方式连接到模拟器帐户
从应用程序连接到存储模拟器的最简单方式是在应用程序的配置文件内配置一个引用快捷方式 `UseDevelopmentStorage=true` 的连接字符串。 以下是 *app.config* 文件中指向存储模拟器的连接字符串示例： 

    <appSettings>
      <add key="StorageConnectionString" value="UseDevelopmentStorage=true" />
    </appSettings>

#### <a name="connect-to-the-emulator-account-using-the-well-known-account-name-and-key"></a>使用从众所周知的帐户名称和密钥连接到存储模拟器
若要创建引用存储模拟器帐户名称和密钥的连接字符串，必须在连接字符串中为你希望从模拟器中使用的每个服务指定终结点。 这是必须的，这样连接字符串将引用与生产存储帐户中的终结点不同的模拟器终结点。 例如，你的连接字符串的值将如下所示：

	DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;
	AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;
    BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;
    TableEndpoint=http://127.0.0.1:10002/devstoreaccount1;
    QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1; 

此值等同于如上所示的快捷方式 `UseDevelopmentStorage=true`。

#### <a name="specify-an-http-proxy"></a>指定 HTTP 代理
你还可以指定一个 HTTP 代理，以便在针对存储模拟器测试服务时进行使用。 当你针对存储服务调试操作时，这对观察 HTTP 请求和响应很有用。 若要指定代理，请将 `DevelopmentStorageProxyUri` 选项添加到连接字符串，并将它的值设置为代理 URI。 例如，下面是一个指向存储模拟器并配置 HTTP 代理的连接字符串：

    UseDevelopmentStorage=true;DevelopmentStorageProxyUri=http://myProxyUri
