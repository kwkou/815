### 检索连接字符串
可以使用 **CloudStorageAccount** 类型来表示 
你的存储帐户信息。如果你使用了 
Azure 项目模板并且/或者引用了
Microsoft.WindowsAzure.CloudConfigurationManager 命名空间，你 
可以使用 **CloudConfigurationManager** 类型
从 Azure 服务配置检索你的存储连接字符串
和存储帐户信息：

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

如果你要创建应用程序而不引用 Microsoft.WindowsAzure.CloudConfigurationManager，并且你的连接字符串位于前面显示的  `web.config` 或  `app.config` 中，那么你可以使用 **ConfigurationManager** 来检索连接字符串。你需要将对 System.Configuration.dll 的引用添加到你的项目中，并为其添加另一个命名空间声明：

	using System.Configuration;
	...
	CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
		ConfigurationManager.ConnectionStrings["StorageConnectionString"]);

<!--HONumber=53-->