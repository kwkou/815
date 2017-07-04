### （可选）为 Azure Active Directory 配置 .NET 移动服务

>[AZURE.NOTE]这些步骤是可选的，因为它们仅适用于 Azure Active Directory 登录提供者。

1. 安装 [WindowsAzure.MobileServices.Backend.Security NuGet 包](https://www.nuget.org/packages/WindowsAzure.MobileServices.Backend.Security)。

2. 在 Visual Studio 中，展开 App_Start 并打开 WebApiConfig.cs。在顶部添加以下 `using` 语句：

        using Microsoft.WindowsAzure.Mobile.Service.Security.Providers;

3. 仍在 WebApiConfig.cs 中，在 `options` 实例化之后立即将以下代码添加到 `Register` 方法中：

        options.LoginProviders.Remove(typeof(AzureActiveDirectoryLoginProvider));
        options.LoginProviders.Add(typeof(AzureActiveDirectoryExtendedLoginProvider));

<!---HONumber=71-->