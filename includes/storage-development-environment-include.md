## 设置开发环境

接下来在 Visual Studio 中设置开发环境，然后即可试用本指南中提供的代码示例。

### 创建 Windows 控制台应用程序项目

在 Visual Studio 中创建 Windows 控制台应用程序，如下所示：

![创建 Windows 控制台应用程序](./media/storage-development-environment-include/storage-development-environment-include-1.png)

本教程中的所有代码示例都可以添加到控制台应用程序中 `program.cs` 中的 **Main()** 方法。

请注意，你可以从任意类型的 .NET 应用程序中使用 Azure 存储客户端库，包括 Azure 云服务、Azure Web 应用、桌面应用程序或移动应用程序。为简单起见，我们在本指南中使用控制台应用程序。

### 使用 NuGet 安装所需包

为完成此教程，你需要将两个包安装到项目：

- [适用于 .NET 的 Microsoft Azure 存储客户端库](https://www.nuget.org/packages/WindowsAzure.Storage/)：此包提供以编程方式访问存储帐户中数据资源的权限。
- [适用于 .NET 的 Microsoft Azure Configuration Manager 库](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/)：此包提供用于分析配置文件中连接字符串的类，而不考虑应用程序在何处运行。

可以使用 NuGet 获取这两个包。执行以下步骤:

1. 在“解决方案资源管理器”中，右键单击您的项目并选择“管理 NuGet 包”。
2. 在线搜索“WindowsAzure.Storage”，然后单击“安装”以安装存储客户端库和依赖项。
3. 在线搜索“ConfigurationManager”，然后单击“安装”以安装 Azure 配置管理器。

>[AZURE.NOTE] [Azure SDK for .NET](/downloads/) 中也包含存储客户端库包。但是我们建议同时从 NuGet 安装存储客户端库，以确保始终使用客户端库的最新版本。

### 确定目标环境

可从两个环境中选择用于运行本指南中示例的环境：

- 可针对云中的 Azure 存储帐户运行代码。 
- 可针对 Azure 存储模拟器运行代码。存储模拟器是模拟云中 Azure 存储帐户的本地环境。应用程序处于开发阶段时，可以选择使用模拟器免费测试和调试代码。模拟器使用已知帐户和密钥。有关详细信息，请参阅[使用 Azure 存储模拟器进行开发和测试](/documentation/articles/storage-use-emulator/)

如果你以云中的存储帐户为目标，请从 Azure 门户复制存储帐户的主访问密钥。有关详细信息，请参阅[查看和复制存储访问密钥](/documentation/articles/storage-create-storage-account/#view-and-copy-storage-access-keys)。

> [AZURE.NOTE] 您可以指向存储模拟器以避免引发与 Azure 存储空间有关的任何费用。但是，如果您确实选择指向云中的 Azure 存储帐户，则执行此教程的费用将会忽略不计。

### 配置存储连接字符串

用于 .NET 的 Azure 存储空间客户端库支持使用存储连接字符串来配置终结点和用于访问存储服务的凭据。维护存储连接字符串的最佳方法在配置文件中。

有关连接字符串的详细信息，请参阅[配置 Azure 存储空间的连接字符串](/documentation/articles/storage-configure-connection-string/)。

> [AZURE.NOTE] 您的存储帐户密钥类似于您的存储帐户的根密码。始终要小心保护存储帐户密钥。避免将其分发给其他用户、对其进行硬编码或将其保存在其他人可以访问的纯文本文件中。如果认为密钥可能已泄漏，请使用 Azure 门户重新生成密钥。

若要配置连接字符串，请从 Visual Studio 中的解决方案资源管理器打开 `app.config` 文件。添加 `<appSettings>` 元素的内容，如下所示。将 `account-name` 替换为您的存储帐户名称，将 `account-key` 替换为您的存储帐户密钥：

	<configuration>
	    <startup> 
	        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2" />
	    </startup>
  		<appSettings>
    		<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key;EndpointSuffix=core.chinacloudapi.cn" />
  		</appSettings>
	</configuration>

例如，配置设置将类似于：

	<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=nYV0gln6fT7mvY+rxu2iWAEyzPKITGkhM88J8HUoyofvK7C6fHcZc2kRZp6cKgYRUM74lHI84L50Iau1+9hPjB==;EndpointSuffix=core.chinacloudapi.cn" />

若要以存储模拟器为目标，可使用映射到已知帐户名称和密钥的快捷方式。在这种情况下，连接字符串设置将如下：

	<add key="StorageConnectionString" value="UseDevelopmentStorage=true;" />


<!---HONumber=Mooncake_0516_2016-->