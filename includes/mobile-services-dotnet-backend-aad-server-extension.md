### （可选）配置 .NET 移动服务以进行 AAD 登录

>[WACOM.NOTE] 这些步骤是可选的，因为它们仅适用于 Azure Active Directory 登录提供者。

1. 安装 **WindowsAzure.MobileServices.Backend.Security** NuGet 软件包。

2. 在 Visual Studio 中，展开 App_Start，打开 WebApiConfig.cs 文件。在 `options`实例化之后立即将以下代码添加到 `Register`方法中：

        options.LoginProviders.Remove(typeof(AzureActiveDirectoryLoginProvider));
        options.LoginProviders.Add(typeof(AzureActiveDirectoryExtendedLoginProvider));

