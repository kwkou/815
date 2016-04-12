### 检索连接字符串
可以使用 **CloudStorageAccount** 类型来表示你的存储帐户信息。如果你正在使用 Azure 项目模板，及/或引用 Microsoft.WindowsAzure.CloudConfigurationManager 命名空间，则可以使用 **CloudConfigurationManager** 类型从 Azure 服务配置中检索存储连接字符串和存储帐户信息：

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

如果你要创建应用程序而不引用 Microsoft.WindowsAzure.CloudConfigurationManager，并且你的连接字符串位于前面显示的 `web.config` 或 `app.config` 中，如上所示，那么你可以使用 **ConfigurationManager** 来检索连接字符串。你需要将对 System.Configuration.dll 的引用添加到你的项目中，并为其添加另一个命名空间声明：

	using System.Configuration;
	...
	CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
		ConfigurationManager.AppSettings["StorageConnectionString"]);

<!---HONumber=Mooncake_0405_2016-->