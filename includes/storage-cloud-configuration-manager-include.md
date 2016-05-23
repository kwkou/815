[适用于 .NET 的 Microsoft Azure Configuration Manager 库](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/)提供用于分析配置文件中连接字符串的类。[CloudConfigurationManager 类](https://msdn.microsoft.com/zh-cn/library/azure/mt634650.aspx)分析配置设置，而不考虑客户端应用程序是在台式计算机、移动设备、Azure 虚拟机还是 Azure 云服务中运行。

下面的示例演示了如何检索配置文件中的连接字符串：

    // Parse the connection string and return a reference to the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
		CloudConfigurationManager.GetSetting("StorageConnectionString"));

不一定要使用 Azure Configuration Manager。还可以使用 API，如 .NET Framework 的 [ConfigurationManager 类](https://msdn.microsoft.com/zh-cn/library/system.configuration.configurationmanager.aspx)。

<!---HONumber=Mooncake_0516_2016-->