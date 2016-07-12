## 设置存储连接字符串

用于 .NET 的 Azure 存储空间客户端库支持使用存储连接字符串来配置终结点和用于访问存储服务的凭据。维护存储连接字符串的最佳方法在配置文件中。

有关连接字符串的详细信息，请参阅[配置 Azure 存储空间的连接字符串](/documentation/articles/storage-configure-connection-string/)。

> [AZURE.NOTE] 您的存储帐户密钥类似于您的存储帐户的根密码。始终要小心保护存储帐户密钥。避免将其分发给其他用户、对其进行硬编码或将其保存在其他人可以访问的纯文本文件中。如果认为密钥可能已泄漏，请使用 Azure 管理门户重新生成密钥。

### 确定目标环境

可从两个环境中选择用于运行本指南中示例的环境：

- 可针对 Azure 存储模拟器运行代码。存储模拟器是模拟云中 Azure 存储帐户的本地环境。应用程序处于开发阶段时，可以选择使用模拟器免费测试和调试代码。模拟器使用已知帐户和密钥。有关详细信息，请参阅[使用 Azure 存储模拟器进行开发和测试](/documentation/articles/storage-use-emulator/)
- 可针对云中的 Azure 存储帐户运行代码。 

如果你以云中的存储帐户为目标，请从 Azure 门户复制存储帐户的主访问密钥。有关详细信息，请参阅[查看和复制存储访问密钥](/documentation/articles/storage-create-storage-account/#view-and-copy-storage-access-keys)。

> [AZURE.NOTE] 您可以指向存储模拟器以避免引发与 Azure 存储空间有关的任何费用。但是，如果您确实选择指向云中的 Azure 存储帐户，则执行此教程的费用将会忽略不计。
	
### 使用 .NET 配置来配置连接字符串

如果应用程序在台式计算机或移动设备、Azure 虚拟机或 Azure Web 应用中运行，请使用 .NET 配置保存连接字符串（例如，保存在应用程序的 `web.config` 或 `app.config` 文件中）。使用 `<appSettings>` 元素存储连接字符串，如下所示。将 `account-name` 替换为您的存储帐户名称，将 `account-key` 替换为您的存储帐户密钥：

	<configuration>
  		<appSettings>
    		<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key;EndpointSuffix=core.chinacloudapi.cn" />
  		</appSettings>
	</configuration>

例如，配置设置将类似于：

	<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=account-key;EndpointSuffix=core.chinacloudapi.cn" />

若要以存储模拟器为目标，可使用映射到已知帐户名称和密钥的快捷方式。在这种情况下，连接字符串设置将如下：

	<add key="StorageConnectionString" value="UseDevelopmentStorage=true;" />

### 配置 Azure 云服务的连接字符串

如果应用程序在 Azure 云服务中运行，请使用 Azure 服务配置文件（`*.csdef` 和 `*.cscfg` 文件）来保存连接字符串。请请参阅[如何创建和部署云服务](/documentation/articles/cloud-services-how-to-create-deploy/)以了解有关 Azure 云服务配置的详细信息。

<!---HONumber=Mooncake_0516_2016-->